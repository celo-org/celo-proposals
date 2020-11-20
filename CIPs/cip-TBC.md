---
cip: TBC
title: Ed25519 precompile
author: Joe Bowman <joe@chorus.one>
discussions-to: https://github.com/celo-org/celo-proposals/issues/XX
status: Draft
type: Standards
category: Ring 0
created: 2020-11-18
license: Apache 2.0
---

### Terminology

The key words "MUST", "MUST NOT", "SHOULD", and "MAY" in this document are to be interpreted as described in 
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.html).


## Simple Summary

Support performant and cheap verification of Ed25519 cryptographic signatures in smart contracts in general by adding a precompiled contract for Ed25519 signature verification to the EVM.

## Abstract

Verification of Ed25519 cryptographic signatures is possible in EVM bytecode. However, the gas cost is very high, and computationally expensive, as such tight, wide word operations intensive code as required for Ed25519 is not a good fit for the EVM bytecode model.

The addition of a native compiled function, in a precompiled contract, to the EVM solves both cost and performance problems.

## Motivation

Ed25519 (that is, EdDSA using Curve25519) is an IETF recommendation (RFC7748) with some attractive properties:

- Ed25519 is intended to operate at around the 128-bit security level.
- EdDSA uses small public keys (32 octets) and signatures (64 octets) for Ed25519.
- Ed25519 are designed so that fast, constant-time (timing attack resistant) and generally side-channel resistant implementations are easier to produce.

With an Ed25519 precompile, we are able to verify Ed25519 signatures origination from external, Ed25519 proof-of-stake based blockchains within Celo EVM smart contracts, allowing for proof-based trustless bridges between Celo and other blockchains.

## Specification

If block.number >= FORK_HEIGHT (TBC), add a precompiled contract for Ed25519 signature verification (`ED25519VFY`).

The proposal adds a new precompiled function `ED25519VFY` with the following input and output.

`ED25519VFY` takes as input 128 octets:

- message: the 32-octet message that was signed
- public key: the 32-octet Ed25519 public key of the signer
- signature: the 64-octet Ed25519 signature

`ED25519VFY` returns as output 4 octets:

- `0x00000000` if signature is valid
- any non-zero value indicates a signature verification failure

### Address
The address of `ED25519VFY` is `0x9`.

### Gas costs
Gas cost for `ED25519VFY` is 2000.

## Rationale
The proposed `ED25519VFY` function takes the signer public key as a call parameter, as with Ed25519. Unlike Secp256K1, it isn't possible to derive the signers public key from the signature and message alone.

The proposed `ED25519VFY` function uses a zero return value to indicate success, since this allows for different errors to be distinguished by return value, as all non-zero return values signal a verification failure.

`ECRECOVER` has a gas cost of `3000`. Since Ed25519 is computationally cheaper, the gas price should be less.

## Backwards Compatibility
As the proposed precompiled contract is deployed at a reserved (<255) and previously unused address, an implementation of the proposal should not introduce any backward compatibility issues.

## Test Cases
Test vectors for Ed25519 can be found in this IETF ID https://tools.ietf.org/html/draft-josefsson-eddsa-ed25519-03#section-6.

Additional ZIP125-compliant test vectors can be found in the ed25519consensus repository (https://github.com/hdevalence/ed25519consensus/blob/main/zip215_test.go).

## Implementation

*ed25519consensus*

ed25519consensus is an implementation of the ed25519 DSA with strict validation criteria as set out in https://zips.z.cash/zip-0215, making it suitable for consensus-type scenarios.

The project can be found https://github.com/hdevalence/ed25519consensus.

## License
This work is licensed under the Apache License, Version 2.0.

## References

This CIP is based upon EIP-665 (https://github.com/ethereum/EIPs/blob/master/EIPS/eip-665.md) by Tobias Oberstein.

