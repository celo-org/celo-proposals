---
cip: <to be assigned>
title: Extension to ODIS for Password Hashing
author: Victor Graf <victor@clabs.co>, Cody Born <cody@clabs.co>, Alec Schaefer <alec@clabs.co>
discussions-to: https://github.com/celo-org/celo-proposals/issues/233
status: Draft
type: Standards Track
category: 3
created: 2021-06-15
<!--
  TODO(victor): Once a CIP for the cryptography is ready, add that here as a dependency.
  requires:
-->
license: Apache 2.0
---

### Terminology

<!-- TODO(victor): Actually go through and add these keywords where appropriate -->
The key words "MUST", "MUST NOT", "SHOULD", and "MAY" in this document are to be interpreted as
described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.html).

## Simple Summary

An extension to the ODIS phone number privacy service to allow for its use for hardening passwords
such that they can be used as encryption and authorization keys.

## Abstract

<!-- TODO(victor): Once we have a roughly agreed upon proposal for the pOPRF, link it here -->
Given the required cryptographic functionality (to be discussed in more detail in a future
proposal), this proposal describes an interface to be exposed by the ODIS service to allow for
rate-limited hash queries against user-defined domains.

Central to the proposed interface are domains. A domain is a struct which specifies rate limiting
information and uniquely identifies the context of the secret to be hashed. 

Queries with distinct domain specifiers will receive uncorrelated output. For example, output from
ODIS when requesting `{ domain: "tel", message: "+18002738255" }` will be distinct from and
unrelated to the output when requesting `{ domain: "password", message: "+18002738255" }`.

Additionally, domain specifiers will control what rate limiting rules are applied. Combined with the
fact that distinct domains produce uncorrelated output, this allows for the domains to be used as
distinct OPRFs with custom rate limiting rules. For example the `tel` domain could use the current
ODIS rate limiting rules, while the `password` domain may use much stricter rules.

In order to make this scheme flexible, allowing for user-defined tuning of rate-limits and the
introduction of new rate limiting and authorization rules in the future, domains are defined as
serializeable structures.

## Motivation

Effective [offline password cracking](https://en.wikipedia.org/wiki/Password_cracking) techniques
limit the use of passwords to derive encryption or authentication keys. Rate-limited hashing can be
used to make it much more difficult to crack a password. ODIS, the phone number privacy service on
Celo, implements a rate-limited hash to protect users' phone numbers from being extracted from their
on-chain identifiers. With some extensions, ODIS can additionally be used to provide a rate-limited
hash for passwords, as well as many more use cases.

[Phone Number Privacy](https://docs.celo.org/celo-codebase/protocol/identity/phone-number-privacy)

ODIS' rate-limited hash function is implemented from an
[oblivious](http://www.cs.columbia.edu/~tal/4261/F19/hugo-columbia-oprf.pdf) [pseudorandom
function](https://en.wikipedia.org/wiki/Pseudorandom_function_family) (OPRF). In effect, this allows
the service to rate-limit invocations of the logical hash function without every seeing the users'
input (e.g. phone number or password). Unfortunately, it only allows for a single rate limiting
scheme.

In order to support secure password hashing, granular rate limits are invaluable. In order to apply
rate limits on a per-user basis (e.g. restricting the number of guesses on a user's password to a
constant number) ODIS' OPRF scheme must be extended to allow the inclusion of a domain specifier
(a.k.a salt) that is visible to the service.

## Specification

In order to support granular rate-limited hash functionality, ODIS will need to support a new
interface which accepts a domain specifiers, visible to the service, alongside the blinded input.

### Service APIs

[celo-org/celo-monorepo](https://github.com/celo-org/celo-monorepo/tree/master/packages/phone-number-privacy/signer)

#### Existing blinded requests

ODIS currently makes available its OPRF function with the following API:

```typescript
// POST to /getBlindedMessagePartialSig
// Headers:
//   Content-Type: application/json
//   Authorization: ${signature from authorized signer (e.g. DEK) for account}
interface GetBlindedMessageSigRequest {
  // Celo account address. Query is charge against this account's quota.
  account: string
  // Query message. A blinded elliptic curve point encoded in base64.
  blindedQueryPhoneNumber: string
  // Optional on-chain identifier. Unlocks additional quota if the
  // account is verified as an owner of the identifier.
  hashedPhoneNumber?: string
  // [Deprecated] Client-specified request timestamp, in milliseconds.
  // Used for request expiration.
  timestamp?: number
  // Client-specified session ID.
  sessionID?: string
}
```

#### Extension for domain-restricted requests

In order to support domain-restricted queries, a new API is proposed:

```typescript
// POST to /getDomainRestrictedPartialSig
// Headers:
//   Content-Type: application/json
interface GetDomainRestrictedSigRequest {
  // Domain specification.
  // Selects the PRF domain and rate limiting rules.
  domain: Domain
  // Domain-specific options.
  // Used for inputs relevant to the domain, but not part of the domain string.
  // Example: { "authorization": <signature> } for an account-restricted domain.
  domainOptions?: DomainOptions
  // Query message. A blinded elliptic curve point encoded in base64.
  blindedMessage: string
  // Client-specified session ID.
  sessionID?: string
}

interface Domain {
  // Unique name of the domain. (e.g. "ODIS Password Domain")
  name: string
  // Major version number. Allows for backwards incompatible changes.
  version: number
  // Arbitrary key-value pairs.
  // Must be serializable to EIP-712 encoding.
  [key: string]: EIP712Value
}

interface DomainOptions {
  // Arbitrary key-value pairs.
  // Valid keys and values depend on the domain specification.
  // Must be serializable to EIP-712 encoding.
  [key: string]: EIP712Value
}
```

### Domain specifiers

Domain specifiers serve two functions:

- Define the input domain, such that it is uncorrelated to the output in any other domain.
- Specify the rate limiting function to be used in deciding whether to serve the request.

In order to provide flexibility and support future extensions, the domain specifier is structured
and allows for the usage of multiple domain-specific rate limiting rules, as support by the ODIS
service.

In the domain specifiers, two fields are always required: `name` and `version`. Together, these
select which set of rules the service will apply to the request. Rulesets are included in the ODIS
implementation, and new rulesets may be added over time. Updating a ruleset in a way that would
change the rules applied to existing domains must be accompanied by an increase in the `version`
number.

With a given `name` and `version`, a number of additional fields may be specified which act as
parameters for the rate limiting ruleset. (e.g. in a linear backoff ruleset, a field may define the
refresh rate). In order to ensure deterministic serialization, each `name` and `version` MUST
correspond to a single type definition.

Changing any field in the domain specifier, results in a new domain with completely unrelated
output. (e.g. an attacker cannot simply dial down the security level of a domain. If they tried,
they would get useless output.)

Note that deprecation of a domain will make all secrets hashed under that domain inaccessible.
Depending on the application, it is likely to be very difficult to full deprecate a domain, as users
will need to actively migrate any usages from the deprecated domain.

In addition to the domain string, a ruleset may allow a number of "domain options". Domain options
are defined by and are relevant to a particular ruleset, but are not part of the domain (i.e. their
value does not effect the function output).

### Examples

The following examples are provided to help understand the scheme, and will likely not be
implemented as described.

**Linear backoff:** A rate limiting scheme allowing up to a given number of requests in a defined
time period could be implemented as a linear backoff with some amount of saved quota cap.

```typescript
type LinearBackoffDomain = {
  name: "ODIS Linear Backoff Domain"
  version: 1
  // Maximum number of saved quota.
  cap: number
  // Length of time, in milliseconds, to refresh a single unit of quota.
  // Undefined refesh time indicates that the quota never refreshes (hard cap).
  refresh?: number
  // Unique string to identify the "instance" of this rate limited domain.
  salt?: string
}
```

**Not before:** Useful in the lotteries and other random-selection based protocols, a domain could
be created to restrict requests before a particular time, but publicly available after.

```typescript
type NotBeforeDomain = {
  name: "ODIS Not Before Domain"
  version: 1
  // Unix time after which this domain can be queried.
  notBefore: number
}
```

**Peppered:** Useful in the context of hardening encryption keys for non-public ciphertexts, a
domain could be defined that requires a signature from a pepper-derived keypair. The pepper may be
stored alongside the ciphertext.

```typescript
type PepperedDomain = {
  name: "ODIS Peppered Domain"
  version: 1
  // Public key of a keypair derived from the pepper.
  publicKey: string
  // Maximum number of requests that can be made against this domain.
  cap?: number
}

type PepperedDomainOptions = {
  // Nonce value to include in the signature message to make it unique.
  nonce: number
  // EIP-712 signature over the request.
  signature: string
}
```

**Phone number hashing:** The current ODIS rate limiting scheme could be implemented as a particular
domain in the new API. A domain string and options could be written as follows:

```typescript
type PhoneNumberDomain = {
  name: "ODIS Phone Number Domain"
  version: 1
}

type PhoneNumberDomainOptions = {
  // Celo account address. Query is charge against this account's quota.
  account: string
  // Signature from the account, or authorized signer for the account.
  authorization: string
  // Optional on-chain identifier. Unlocks additional quota if the
  // account is verified as an owner of the identifier.
  hashedPhoneNumber?: string
}
```

**Smart contract access controlled:** Useful for expanding the set of use cases to community derived
ones. The address of a smart contract implementing the following interface will be passed in as a
parameter.

```solidity
interface AccessControlled {
  function shouldAllow(address requester) external view returns(string);
}
```

For example, this contract could check that a user owns a certain NFT or has burned a fee before
being allowed access to the signature. The signature would be applied to the domain and the returned
value from the contract function.

```typescript
type SmartContractDomain = {
  name: "ODIS Smart Contract Domain"
  version: 1
  // Specified to ensure an unambiguous smart contract is resolved.
  // An ODIS given service is likely to support only a single chain.
  chainId: number
  // Address of the smart contract on which `shouldAllow` will be called
  // to determine rate limiting. Call will occur as a view call at the
  // chain head upon receiving the request.
  contractAddress: string
}

type SmartContractDomainOptions = {
  // Optional parameter to avoid issues if the ODIS service node is behind.
  // When speciifed, the authorization check should not occur agsint a block
  // prior to the specified number.
  notBeforeBlock?: number
  // Celo account address of the requester.
  account: string
  // Signature from the account, or authorized signer for the account.
  authorization: string
}
```

### Domain serialization

<!-- TODO(victor): Refactor this section a bit to describe a function to map a subset of TS types to
EIP-712 types -->
Because any variation in the domain string, by design, results in an unpredictable change in the
output, there must be a canonical (i.e. deterministic) serialization for each given domain string.
Further, there must not be a collision in the encoded domain strings between two semantically
different domain specifiers (i.e. the encoding function must be injective). This matches the
requirements for the encoding function specified in
[EIP-712](https://eips.ethereum.org/EIPS/eip-712#specification). As a result, this proposal adopts
their encoding function for its domains specifiers.

#### Mapping TypeScript to EIP-712 types

In order to serialize a given `Domain` type, we must first define it as a EIP-712 object. As shown
above, domains may be represented as TypeScript types, in which case they can be mapped to an
equivalent EIP-712 object.

To form the EIP-712 domain, the required `name` and `version` fields of the `Domain` type are used
to populate the respective fields in an `EIP712Domain`. Additional fields of the `Domain` struct are
used to construct an EIP-712 type.

Other fields in the domain form the EIP-712 type. The EIP-712 type name should be equal to the
interface name. Fields within the interface are sorted by name. Values SHOULD be mapped from their
TypeScript types to the corresponding EIP-712 type.

Optional values are encoded by interpreting it as EIP-712 type `Optional<T>(T value,bool defined)`,
with `T` are the generic type name. When serialized, `value` MUST be set to the zero value of the
type when `defined` is true. Zero values for a type is the value when all atomic types are set to
zero, and dynamic types (`string` and `bytes`) are empty.

<!-- TODO(victor) Add an example here of converting LinearBackoffDomain to EIP-712 -->

### Signatures

Signatures are a useful primitive for authenticated rate limiting (e.g. allowing a higher rate limit
to the owners of verified accounts on Celo). In order to facilitate this, the entire request struct
should be deterministically serializeable, specifically using EIP-712.

Domains that make use of signatures for authentication of requests (e.g. the `PepperedDomain`
example below) should include a field in the domain options for specification of a signature.
Additional fields may also be added, such as a nonce to prevent replaying requests.

<!-- TOOD(victor): Add a specification of the EIP-712 domain to be used for here -->
Signatures should be verified over the entire request, excluding the signature itself. Specifically,
the message against which a signature is verified should be obtained by first removing the signature
from the full `GetDomainRestrictedSigRequest` object, then serializing it via EIP-712.

### HTTP Transport

When requested over HTTP, which is the primary proposed transport, the following HTTP semantics
should be followed.

If a given `name` and `version` are not recognized by an ODIS signer, they should return a `501: Not
Implemented` error. If a given `name` and `version` are recognized, but are deprecated, ODIS should
return a `405: Method Not Allowed` error. Signers should keep a list of all old domain names and
version, even if they are no longer supported.

When a users request is rejected for violating the rate limiting of a domain, the service should
respond with `429: Too Many Requests`. If the waiting would allow for additional requests against
the same domain, the service should include the `Retry-After` header to specify how long the client
must wait. ODIS servers may apply additional rate limiting, such as by IP address. Any rate limiting
rules outside of those applied per domain should be counted against first, such that a request
dropped by external rate limiting will not be counted against the domain quota.

### Key management

Because ODIS' current OPRF function operates with the same cryptographic primitives as the proposed
function below, and because its a bad idea in general to reuse keys, this proposed requires a new
set of keys to be generated and used in computing the pOPRF function.

Prior to the deployment of this new pOPRF functionality a DKG will need to be run on the nodes that
will participate in serving it. This new public key should be distributed along with client
implementations and used to verify the values returned from the new API. Note that the existing ODIS
OPRF key will continue to be used to verify output from the existing API.

### Cryptographic interface

Implementation of a domain-restricted hashing function is build on top of a partial
[oblivious](http://www.cs.columbia.edu/~tal/4261/F19/hugo-columbia-oprf.pdf) [pseudorandom
function](https://en.wikipedia.org/wiki/Pseudorandom_function_family) (pOPRF). It extends the
existing OPRF, currently implemented by ODIS, by adding a second input, which is not blinded, and
can be used by the pOPRF for rate limiting.

With respect the client, this functionality has the interface $F_K(d,m)$ where $d$, $m$, and the
output are binary strings, and $K$ is a public key which commits to the output of the function, and
allows for verification. $d$ is considered the "domain string" and is visible to the pOPRF service.
$m$ is the message string, and is hidden from the pOPRF service. Both strings are combined
cryptographically to form the output.

Multiple candidate implementations exist for this functionality. Linked below is the proposed
cryptographic design for use in the ODIS pOPRF service. Other implementation with the same interface
may also be used to implement the following service design.

[pOPRF Cryptography Proposal](https://www.notion.so/pOPRF-Cryptography-Proposal-493f1099460940f8a5d7dee4c78b4442)

## Backwards Compatibility

Backwards compatibility of for current users of ODIS is ensure because this proposal leaves in place
all existing functionality, including the public keys currently used to operate the service.

## Security Considerations

WIP

## License

This work is licensed under the Apache License, Version 2.0.
