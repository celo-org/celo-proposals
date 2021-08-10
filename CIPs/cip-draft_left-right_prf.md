---
cip: <to be assigned>
title: Left/Right Constrained PRFs for use in Celo protocols
author: <a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s), e.g. (use with the parentheses or triangular brackets): FirstName LastName (@GitHubUsername), FirstName LastName <foo@bar.com>, FirstName (@GitHubUsername) and GitHubUsername (@GitHubUsername)>
discussions-to: <URL>
status: Draft
type: <Standards Track, Meta or Informational>
category (*only required for Standards Track): <Ring 0, 1, 2, 3>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
requires (*optional): <CIP number(s)>
replaces (*optional): <CIP number(s)>
license: Apache 2.0
---


## Simple Summary
The goal of this CIP is to provide a specification for a cryptographic primitive, left/right constrained PRFs, for use in various subprotocols of the Celo blockchain. 

## Abstract
Celo is a mobile-first blockchain protocol that seeks to make it easy for users to make cryptocurrency payments with their mobile devices. As such, Celo has developed a series of subprotocols for enabling users to use their phone numbers for various aspects of the Celo protocol. However, due to phone numbers being directly tied to users' identities, several subprotocols have been developed to provide as much privacy as a possible to Celo's users. These subprotocols include ODIS, PNP and a few currently in development.

The use of a cryptographic primitive called constrained pseudorandom functions underpinned many of these privacy-enabling constructions on Celo. In particular, left-right constrained PRFs enable the construction of many privacy-preserving identity-based constructions that are useful for Celo users.

This CIP specifies the parameters and algorithms for using left-right constraints in the Celo ecosystem.

## Motivation
The motivation for this CIP is to provide a secure scheme for identity-based encryption to allow for a variety of privacy-preserving protocols for the Celo blockchain.

## Specification
Constrained PRFs are defined as follows:
Given a key space `K`, a domain `X`, range `Y`, let `F` be a pseudorandom function that takes as inputs (`k`, `x`) for `k` in `K` and `x` in `X` and returns `F(k, x)` in `Y`. 

We say that `F` is constrained with respect to a set of constraints `S` if there is an new key space `K_c` such  that, we can define the following functions:
- `F.constrain(k, S)` which provides a constrained key `k_s` in `K_c`
- `F.eval(k_x, x)` which outputs `F(k,x)` if `x` is in `S`.

In practice, `S` is a set of predicates `PP` such that for any `p` in `PP`, we have `F.eval(k_p, x) = F(k,x)` when `p(x) = 1` and `p(x) = 0`, otherwise.

### Left/right constrained PRFs

Left/right constrained PRFs are defined as follows:
Let `F: K x X^2 -> Y` be a PRF. Then, `F` is a left/right PRF if `F` is constrained by the following predicates `P_LR = {(p_wL, p_wR), w in X}` where `p_wL(x,y) = 1` iff `x = w` and `p_wR(x,y) = 1` iff `y = w`. Left/right constrained PRFs have the property such that for all points in `(w, y)` in `X^2`, `F(k, (w,y)` can be evaluated at all the points in which the left side is fixed to `w`. Similarly for right constrained PRFs. 

We instantiate the following left/right constrained PRF construction to be used in the Celo ecosystem:

Define `F(k, (x,y))` to be `e(H_1(x), H_2(y))^k`,
where
- `e` is the optimal ate pairing for BLS12-377 as defined in [CIP30](CIP30), 
- `H_1()` is the hash to `G_1` function defined in [HASH_TO_CURVE](HASH_TO_CURVE)
- `H_2()` is the has to `G_2` function defined in [HASH_TO_CURVE](HASH_TO_CURVE)
- `K` is $Z_p$
- `X` is $G_1$ for the BLS12-377 curve
- `Y` is $G_2$ for the BLS12-377 curve

The constrained keys for the predicates `p_wL` and `p_wR` are defined as:
`k_x* = H_1(x*)^k` and `k_y* = H_2(y*)^k` where `(x*, y*)` in `X^2`


## Rationale
The main rational for this CIP is to provide a cryptographic primitive that can be used in identity-based contexts (phone number privacy, validator keys, etc) while still reusing Celo's fundamental cryptography toolkit (pairing-based and threshold-based cryptography). This would allow for minimal changes to the protocol level of how Celo works while still allowing for new subprotocols to be built to provide better privacy for Celo users.

## Backwards Compatibility

There are no backwards compatibility issues with this CIP.

## Test Cases

TBD

## Implementation

TBD

## Security Considerations
- Care must be taken when reusing BLS12-377 private keys for use with this CIP. 
- For threshold-variants of the proposed left-right constrained PRF, care must be taken as to not allow for rogue key attacks. 


## References

[CIP30](https://github.com/celo-org/celo-proposals/blob/master/CIPs/cip-0030.md): Precompile for BLS12-377 curve operations

[CIP22](https://github.com/celo-org/celo-proposals/blob/master/CIPs/cip-0022.md): Upgrade epoch SNARK data for flexibility, slashability and security

[HASH_TO_CURVE](https://github.com/celo-org/celo-bls-snark-rs/blob/eca15ffed3984601cb441600ba3002d12f339e5f/crates/bls-crypto/src/hash_to_curve/try_and_increment.rs#L81): Hash to curve implementation used in Celo

## License
This work is licensed under the Apache License, Version 2.0.
