## SNS-IP Number: 2

## Title: Record V2

## Author:

Bonfida

## Status: Draft

## Created: August 9th, 2023

## Abstract

This SNS-IP proposes an enhancement to the existing SNS records system to address concerns related to the staleness of records and the authenticity of linked resources. Specifically, it introduces mechanisms to verify the ownership of resources and handle outdated records. The related issue can be found here: https://github.com/Bonfida/sns-ip/issues/4

## Motivation

There are two significant issues currently with records:

- **Staleness of Records:** When domain names are traded, records associated with these domains can become outdated but remain accessible, leading to potential misinformation or misuse.

- **Resource Authenticity:** It is challenging to verify that the domain owner genuinely owns the resources linked in the records. This ambiguity paves the way for impersonation and can lead to potential security breaches.

To tackle these issues, there's a need for a system that enhances record integrity and assures the authenticity of resources linked in these records.

## Specification

A Borsh implementation in Javascript

```js
export enum GuardianSig {
  None = 0,
  Solana = 1,
  Ethereum = 2,
  Injective = 3,
}

export enum UserSig {
  None = 0,
  Solana = 1,
}

export class RecordV2Header {
  userSignature: UserSig;
  guardianSignature: GuardianSig;
  contentLength: number;

  static LEN: number = 2 + 2 + 4;

  static schema: Schema = new Map([
    [
      RecordV2Header,
      {
        kind: "struct",
        fields: [
          ["userSignature", "u16"],
          ["guardianSignature", "u16"],
          ["contentLength", "u32"],
        ],
      },
    ],
  ]);

  // ...
}

// ...

export class RecordV2 {
  header: RecordV2Header;
  buffer: Buffer;

  constructor(obj: { header: RecordV2Header, buffer: Buffer }) {
    this.header = obj.header;
    this.buffer = obj.buffer;
  }

  /**
   * This function deserializes a buffer into a `RecordV2`
   * @param buffer The buffer to deserialize into a `RecordV2`
   * @returns A RecordV2
   */
  static deserializeUnchecked(buffer: Buffer): RecordV2 {
    const header = deserializeUnchecked(
      RecordV2Header.schema,
      RecordV2Header,
      buffer
    );
    return new RecordV2({
      header,
      buffer: Buffer.from(buffer.slice(RecordV2Header.LEN)),
    });
  }

  // ...
}
```

The rest of the JS implementation can be found here: https://github.com/Bonfida/sns-sdk/pull/15

We are expecting the following signature types on launch:

- Solana (64 bytes)
- Ethereum (65 bytes)
- Injective (65 bytes)

When the record is not signed, the signature tag should be the `None` enum variant.

## Rationale

The rationale behind the specification above comes from the following design choices:

1. **Record Header Structure:** In designing the `RecordV2Header` class, we emphasized clarity and encapsulation. The header consists of three primary components: the user's signature (`userSignature`), the guardian's signature (`guardianSignature`), and the content length (`contentLength`).

   - **User and Guardian Signatures:** Represented as 16-bit unsigned integers (`u16`), these fields represent the enum type for the signatures.
   - **Content Length:** Stored as a 32-bit unsigned integer (`u32`), this indicates the size of record's content. This ensures that reading the buffer can be done efficiently, knowing exactly how many bytes to expect.

2. **Buffer Structure:** The `buffer` in the `RecordV2` class is a concatenated structure comprising:

   - **The user's signature:** Authenticated content by the record owner.
   - **The guardian's signature:** An optional but critical addition for verification by a trusted party.
   - **The content:** The main content of the record encoded using [SNS-IP 1 guidelines](https://github.com/Bonfida/sns-ip/blob/master/proposals/sns-ip-1.md)

3. **8-byte Alignment Constraint:** Abiding by the 8-byte alignment constraint of the Solana runtime is implicit but crucial. This adherence optimizes memory usage and ensures that the record's structure respects the underlying system's requirements, leading to efficient deserialization on-chain through type casting.

## Backwards Compatibility

To ensure this system remains backward-compatible, a new derivation prefix, `\x02`, will be added to the record key. This differentiation allows for the coexistence of old and new records, ensuring that applications and platforms can transition smoothly without compromising existing functionalities.

## Security Considerations

This proposal does not introduce new security considerations.

## Test Cases

Test cases will be developed for each record type to ensure that the new encoding formats are correctly implemented and that the SDKs can accurately encode and decode data according to the new standards. This changes will be implemented on all the different SDKs.

## Implementation

The changes will be implemented incrementally, starting with the specification JS SDK.
