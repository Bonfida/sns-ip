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

- **Right of association**: It is challenging to verify that the domain owner genuinely has the right to associate with the resources linked in the records. This ambiguity paves the way for impersonation and poses a security risk.

To tackle these issues, this proposal describes a system that specifically handles these issues, while remaining usable in both on and off-chain contexts.

## Specification

The record account contents will exactly match those described in SNS-IP-1.
The record derivation path will change and amends the initial SNS specification.
Records will stay subdomains, but will also become a `class` derived from the certification smart contract's central authority.

### Certification smart contract interface

TBD

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