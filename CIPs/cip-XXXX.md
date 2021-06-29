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

This is the suggested template for new CIPs.

Note that a CIP number will be assigned by an editor. When opening a pull request to submit your CIP, please
use an abbreviated title in the filename, `cip-draft_title_abbrev.md`.

## Simple Summary
The goal of this CIP is to provide a specification for a cryptographic primitive, left/right constrained PRFs, for use in various subprotocols of the Celo blockchain. 

## Abstract
Celo is a mobile-first blockchain protocol that seeks to make it easy for users to make cryptocurrency payments with their mobile devices. As such, Celo has developed a series of subprotocols for enabling users to use their phone numbers for various aspects of the Celo protocol. However, due to phone numbers being directly tied to users' identities, several subprotocols have been developed to provide as much privacy as a possible to Celo's users. These subprotocols include ODIS, PNP and a few currently in development.

The use of a cryptographic primitive called constrained pseudorandom functions underpinned many of these privacy-enabling constructions on Celo. In particular, left-right constrained PRFs enable the construction of many privacy-preserving identity-based constructions that are useful for Celo users.

This CIP specifies the parameters and algorithms for using left-right constraints in the Celo ecosystem.

## Motivation
The motivation is critical for CIPs that want to change the Celo protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the CIP solves. CIP submissions without sufficient motivation may be rejected outright.

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
`e` is the optimal ate pairing for BLS12-377 as defined in (insert reference)
`H_1()` is the hash to `G_1` function defined in (insert reference)
`H_2()` is the has to `G_2` function defined in (insert reference)

The constrained keys for the predicates `p_wL` and `p_wR` are defined as:
`k_x* = H_1(x*)^k` and `k_y* = H_2(y*)^k` where `(x*, y*)` in `X^2`

TODOs:
- Define  `X`, `Y`
- Specify `e`, `H_1`, `H_2`



## Rationale
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.

## Backwards Compatibility
All CIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CIP must explain how the author proposes to deal with these incompatibilities. CIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Test Cases
Test cases for an implementation are mandatory for CIPs that are affecting consensus changes. Other CIPs can choose to include links to test cases if applicable.

## Implementation
The implementations must be completed before any CIP is given status "Final", but it need not be completed before the CIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.

## Security Considerations
All CIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. CIP submissions missing the "Security Considerations" section will be rejected. A CIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

## License
This work is licensed under the Apache License, Version 2.0.
