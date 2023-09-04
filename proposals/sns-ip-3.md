## SNS Improvement Proposal

### SNS-IP Number: 3

### Title: Record v2 (amended)

### Author(s): Bonfida

### Status: Draft

### Created: August 29th 2023

### Updated: -

## Abstract

This SNS-IP describes an amended version of the enhancement initally proposed in SNS-IP 2. Similary, it proposes a new specification for on-chain records which can address concerns related to the staleness of records, as well as right of association to the linked ressource when relevant. This SNS-IP specifically ensures that all records are created or edited through a certification smart contract which enforces those new constraints. This enables records to be trusted without validating their contents at every use, which would be prohibitively expensive to do on-chain.

## Motivation

As adapted from the rejected SNS-IP 2, there are two significant issues currently with records, the "motivating concerns":

- **Staleness of Records**: When domain names are traded, records associated with these domains can become outdated but remain accessible, leading to potential misinformation or misuse.

- **Right of association** (RoA): It is challenging to verify that the domain owner genuinely has the right to associate with the resources linked in the records. This ambiguity paves the way for impersonation and poses a security risk.

To tackle these issues, this proposal describes a system that specifically handles these issues, while remaining usable in both on and off-chain contexts.

## Specification

The record derivation path will change and amends the initial SNS specification.
Records will stay subdomains, but will also become a `class` derived from the certification smart contract's central authority.
The record's account verifications are described within its contents, which ensures that expected validation types for particular records can be changed in light of potentially arising security considerations.

```text

 0                   1                   2                   3                   4                   5                   6
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Staleness Validation Type   |      RoA Signature Type       |                        Content-Length                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                                                                                               |
+                                                   Staleness verification Id                                                   +
                                                               ...
+                                                          (S*8 bytes)                                                          +
|                                                                                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                                                                                               |
+                                                      RoA verification Id                                                      +
                                                               ...
+                                                          (G*8 bytes)                                                          +
|                                                                                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                                                                                               |
+                                                        Record Content                                                         +
                                                               ...
+                                                          (C*8 bytes)                                                          +
|                                                                                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Certification smart contract interface

All supported record types will have at most two associated validation steps (one for each motivating concern).
The first instruction will publish the pending record contents with `None` validation types.
Then each validation step will be performed in turn.
Any edit to the record contents will switch the validations to `None`.
In theory, the entire validation can be performed in one transaction containing three instructions (or more in the case of Ethereum validation).

One useful side-effect of using runtime Solana signing validation is that smart contract PDAs can be certification authorities, in which case a CPI call to `ValidateSolanaSignature` will perform the validation.

```rust

#[repr(u16)]
pub enum Validation {
    None,
    Solana,
    Ethereum,
    ..
}

struct RecordHeader {
    staleness_validation_type: Validation,
    right_of_association_validation_type: Validation,
    content_length: u32
}

pub enum CertificationInstruction {
    AllocateRecord { length: usize },
    AllocateAndPostRecord { contents: Vec<u8> },
    EditRecord { contents: Vec<u8>, offset: usize },
    // This instruction will check that the proper authority signed the transaction
    ValidateSolanaSignature,
    // This instruction will use the Secp256k1 validation flow natively supported by Solana
    // https://docs.solana.com/developing/runtime-facilities/programs#secp256k1-program
    ValidateEthereumSignature
}

```

### An example, the SOL record

In order to create a `SOL` record with value `JAUP6N2Jayt7nZJgqDX79zcHdcg4B5FuVDFbyt3LvbpP`, for domain `test.sol` which is owned by `CaN5H4fXGy1kJoJ6Mhgof1g9hxqppvoTpmzmx137rx2q`, we need to execute three instructions in sequence.

First we need to execute an `AllocateandPostRecord` instruction signed by the domain owner with an empty vector as the contents parameter.
The contents parameter is empty in this particular case because the SOL record will resolve to the RoA validation id.
Otherwise we would just be writing the `SOL` record address twice in the account, and adding an unnecessary verification step.

Then, we need to execute a `ValidateSolanaSignature` instruction signed by the domain owner to execute staleness validation.
This will write `CaN5H4fXGy1kJoJ6Mhgof1g9hxqppvoTpmzmx137rx2q` in byte vector format to the staleness validation id field in the the record account.

Finally, we need to execute another `ValidateSolanaSignature` instruction this time signed by `JAUP6N2Jayt7nZJgqDX79zcHdcg4B5FuVDFbyt3LvbpP` to execute RoA validation and, in this particular case, set the actual resolution.
This will write `JAUP6N2Jayt7nZJgqDX79zcHdcg4B5FuVDFbyt3LvbpP` in byte vector format to the RoA validation id field in the record account.

Then, any application which makes use of the record will validate it by checking that the staleness validation id field corresponds to the parent domain's current owner.
In the generic case, we would also have to check that the RoA validation id field is pertinent to the record's value, but this logic will be specific to each record type.

### Implementing new record types

This specification allows for the creation of custom record types as long as they just make use of already supported validation methods.
In order to avoid name space collisions, there will be a process to register new applications and officially allocate associated record types.
It will be possible to request native support for new validation methods as part of the SNS-IP process.

Once a record type is allocated to an application team, they have free reign over the kind of expected validation types to expect and enforce.
This specification allows these expectations to evolve over time, depending on the needs of the application itself.
Complex custom validation can be implemented as a separate smart contract, with an associated certification PDA.

## Rationale

TBD

## Backwards Compatibility

The new derivation path breaks initial compatiblity with existing implementations. However, the fact that the account contents stay the same means that all implementations will only need to change this derivation path. This proposal's advice is that implementations SHOULD deprecate use of the old insecure records as soon as posssible. The nature of this specification means that new record types will be implemented incrementally, which means that the v1 and v2 records will coexist. For applications in which the two motivating concerns are no issue, the v1 records may continue to coexist for a longer while, although all new implementations SHOULD make use of the new system.

## Security Considerations

All applications for which protection agains the motivating concerns is critical to security SHOULD switch to this specification as soon as possible.

## Test Cases

Test cases for each record type will be published as part of the reference implementation as it catches up to this specification.

## Implementation

The sns-sdk will serve as a reference implementation as new record types are introduced.

## References

- [SNS-IP 1](https://github.com/Bonfida/sns-ip/blob/master/proposals/sns-ip-1.md)
- [SNS-IP 2 (Rejected)](https://github.com/Bonfida/sns-ip/blob/master/proposals/sns-ip-2.md)
