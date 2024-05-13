## SNS Improvement Proposal

### SNS-IP Number: 4

### Title: Subdomains

### Author(s): Bonfida

### Status: Draft

### Created: May 13th 2024

### Updated: -

## Abstract

This SNS-IP describes the derivation of SNS subdomains (eg. `a.b.sol`).
A subdomain is under the complete control of the parent domain's controlling entity. 
However, this control can be limited through the use of a community program called `sub-registrar`.
This protects subdomain ownership while allowing the parent domain owner more freedom in how registration proceeds.

## Motivation

### Sub-entities

An organization or community might find it useful to assign specific on-chain identities to their sub-entities. 
Theses sub-entities can be thought of as actors which represent the parent entity, while having to be fundamentally distinct.
A good example of this is a smart contract: it acts on behalf of an entity, but can be distinct from it, have its own funds, its own cross-chain identities, etc.

### Social identity

The Solana Name Service's mission is to represent and specify on-chain identity.
The singular `.sol` domain ensures broad compatibility across web3 and web2 applications, and provides an _absolute_ identity system.
However, identity can also be _relative_.
Users are also members of existing communities, and can occupy specific roles within those communities.

The current absolute identity system allows users to attach specific information to their own profile.
Thanks to SNS-IP-3, this information can be externally validated which means that users can't just say anything about themselves.
This does introduce a degree of relative truth: if one says that K is their Solana public key through the records system, then it means they have certified they are indeed the owner of that public key.
There is no doubt in this context that they are indeed a member of the Solana community under that _alias_.
However, the records system is designed to slowly incorporate new elements of identity, and cannot scale as fast as the growing numbers of community in the web3 ecosystem.

This means that communities and organizations should be given the means to administer their own relative identity systems.
We find that in practice these relative identity systems can be broadly split into two categories: _role-based_ and _username-based_.
Roles can be assigned and reassigned by the parent community or organization, which means that the owning entity retains full control over their _role-based_ domain.
Usernames are immutable and only the user can choose to abandon their username (banning is possible, but preventing impersonation means that a username can't be reused in the event of a ban).
This means that the owning entity relinquishes some of its control to an external smart contract which is responsible for enforcing _ownership immutability_ and preventing _impersonation_.


## Specification

### Valid subdomains and key derivation

| Term          | Definition                           | Example with `alpha.bonfida.sol`               |
|---------------|--------------------------------------|------------------------------------------------|
| subdomain id  | the first part of the subdomain url  | `alpha`                                        |
| domain id     | the second part of the subdomain url | `bonfida`                                      |
| domain key    | the parent domain's account key      | `Crf8hzfthWGbGbLTVCiqRqV5MVnbpHB1L9KQMd6gsinb` |
| subdomain key | the subdomain's account key          | <TODO>                                         |

A subdomain id has to be valid under the latest SNS guidelines for domain names.
In particular, this means that uppercase and invisible characters are invalid.

We compute `S` by prefixing the subdomain id with a null-byte. 
The subdomain is then defined as the child of the parent domain under the spl-name-service specification, with name `S`.

### Role-based identity system (RIS)

For role-based identity systems, the owner of the parent domain is free to create subdomains as they please, with a meaning defined within the organization. 
Due to this lack of restrictions, a username-based identity system (UIS) cannot be subordinated to a role-based identity system.
However, a RIS can be upgraded to a UIS at any time by transfering ownership through the `sub-registrar` community contract (as of writing under the supervision of the Bonfida organization).
A domain `a.sol` is said to be a _RIS_ when it is _not_ owned by the `sub-regsitrar` program.

### Username-based identity system (UIS)

In a user-based identity systems, users are free to create their own username subdomains as long as they meet a certain set of requirements which are defined by the parent organization.
The parent organization can at any time create new subdomains and bypass those restrictions using an admin authority.
The UIS domain's parent has to be the `.sol` root in order to reliably enforce immutability.
In the future, this requirement can be relaxed to also allow the parent domain to be a certified UIS itself.
However, it is impossible for an organization to delete an existing subdomain without the consent of its current owner.
Unless all subdomains registered through the `sub-registrar` UIS enforcer are closed, a domain cannot be down-graded to a RIS, and its ownership cannot be transfered.
This means that domain owners have to administer the domain through the UIS enforcer's primitives.

#### The `sub-registrar` reference UIS enforcer

Summary of `sub-registrar` instructions:

| Index | Instruction           | Description                                                                                                                               |
|-------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| 0     | CreateRegistrar       | Transfers ownership of a domain to the UIS enforcer, and defines subdomain registration, allowable admin action rules and admin authority |
| 1     | CreateRegistrar       | Edits subdomain registration rules                                                                                                        |
| 2     | CreateRegistrar       | Edits subdomain registration rules and admin authority                                                                                    |
| 2     | Register              | Endpoint through which users can create their own subdomains                                                                              |
| 3     | Register              | Endpoint through which users delete their own subdomains                                                                                  |
| 4     | CloseRegistrar        | Unwraps the domain and transfers ownership from the UIS enforcer. This is only possible if all subdomains are closed                      |
| 5     | AdminRegister         | With an admin signature, creates a subdomain and bypasses registration requirements                                                       |
| 6     | DeleteSubdomainRecord | Delete a subdomain record account                                                                                                         |
| 7     | AdminRevoke           | Revokes a subdomain with admin consent                                                                                                    |
| 8     | NftOwnerRevoke        | Revokes a subdomain with nft-holder consent, in the case of nft-based registration rules                                                  |


## Rationale

TBD

## Backwards Compatibility

While there are sporadic implementations of a subdomain system, most reference implementations do not officially support this.
This means that this SNS-IP is an extension of spec with little potential to break backwards compatibility in current production systems.

## Security Considerations

When resolving a subdomain, app developers MUST determine if the subdomain is of RIS or UIS type.
If a subdomain is UIS, then it can be trusted.
if a subdomain is RIS, the app developer SHOULD assert that the user trusts the parent entity, and MUST at least verify that they trust the parent entity.

## Test Cases

Test cases for will be published as part of the reference implementation as it catches up to this specification.

## Implementation

The sns-sdk and sub-registrar repos will serve as reference implementations of the RIS and UIS specs respectively

## References

- [SNS-IP 3](https://github.com/Bonfida/sns-ip/blob/master/proposals/sns-ip-3.md)