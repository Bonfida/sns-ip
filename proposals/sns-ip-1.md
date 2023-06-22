## SNS-IP Number: 1

## Title: Formalizing Data Encoding for SNS Domain Records

## Author:

Bonfida

## Status: Accepted

## Created: June 1, 2023

## Abstract

This proposal aims to standardize the data encoding in each record type for the Solana Name Service. Currently, the content of each record lacks a standardized format, which can lead to inconsistencies and unpredictable behavior. By clarifying the encoding rules for each record content, we aim to improve predictability and ease of use.

## Motivation

Current implementations of SNS domain records show inconsistency in the encoding of record data. Standardizing these formats will streamline operations, reduce errors, and make the protocol more robust and predictable.

## Specification

For each record type, we propose to establish a standardized encoding format.

- **IPFS**: A UTF-8 IPFS CID string.
- **ARWV**: A UTF-8 Arweave hash string.
- **SOL**: A 96-byte array representing a concatenation of a public key (32 bytes) and a signature (64 bytes).
- **ETH**: Ethereum addresses should be represented as a 20-byte array.
- **BTC**: Bitcoin addresses should be represented as a UTF-8 encoded string. This approach simplifies client-side deserialization due to the multiple possible formats, accommodating all Bitcoin address types (P2PKH, P2SH, and Bech32).
- **LTC**: Litecoin addresses should also be represented as a UTF-8 encoded string. This streamlines client-side deserialization, taking into account the varying potential formats.
- **DOGE**: Dogecoin addresses should be represented as a UTF-8 encoded string as well. This ensures ease of client-side deserialization, considering the several possible formats.
- **Email**: A UTF-8 email address string.
- **URL**: A valid URL string.
- **Discord**: A UTF-8 Discord username string. Must not be prefixed by the @ character.
- **Github**: A UTF-8 Github username string. Must not be prefixed by the @ character.
- **Reddit**: A UTF-8 Reddit username string. Must not be prefixed by the @ character.
- **Twitter**: A UTF-8 Twitter username string. Must not be prefixed by the @ character.
- **Telegram**: A UTF-8 Telegram username string. Must not be prefixed by the @ character.
- **Pic**: An URL string pointing to a profile picture.
- **SHDW**: A Shadow drive address string.
- **POINT**: A Point network record string.
- **BSC**: BSC addresses should be represented as a 20-byte array.
- **Injective (INJ)**: Injective addresses should be represented as 20-byte array.
- **Backpack**: A UTF-8 Backpack username string. Must not be prefixed by the @ character.
- **A**: As defined in [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035)
- **AAAA**: As defined in [RFC 3596](https://datatracker.ietf.org/doc/rfc3596/)
- **CNAME**: As defined in [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035)
- **TXT**: As defined in [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035)

### Serialization and deserialization of records containing UTF-8 strings

These Records **SHOULD** be encoded as UTF-8 in a precisely sized account. These Records **MUST** be decoded as a UTF-8 string, truncating to the last non-null byte

## Rationale

The standardization of encoding for each record type will eliminate unpredictability and inconsistencies in the system. It will also enhance the robustness of the protocol by ensuring that the encoding and decoding processes are reliable and deterministic.

## Backwards Compatibility

The proposal is not fully backward compatible as it will require changes in the way data is stored in existing records. A migration plan will be developed to help existing record owners transition their records to the new format.

## Security Considerations

This proposal does not introduce new security considerations.

## Test Cases

Test cases will be developed for each record type to ensure that the new encoding formats are correctly implemented and that the SDKs can accurately encode and decode data according to the new standards. This changes will be implemented on all the different SDKs.

## Implementation

The changes will be implemented incrementally, starting with the specification and development of the new encoding formats for each record type, followed by the development of the migration plan, and finally, the actual migration.
