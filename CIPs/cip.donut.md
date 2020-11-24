---
cip: 27
title: Donut Hardfork 
author: Yaz Khoury <@YazzyYaz>, James Prestwich <@prestwich>, Kobi Gurkan <@kobigurk>
discussions-to: https://github.com/celo-org/celo-proposals/issues/94 
status: Draft
type: Meta 
created: 2020-11-10
license: Apache 2.0
---

This is the Meta CIP for the technical specifications of the Celo Donut Hardfork.

## Simple Summary

Enable the activation of the Celo Donut Hardfork specification which includes Ethereum Berlin EIPs and Celo CIPs.

## Abstract

Add support for protocol-impacting changes introduced in the Ethereum Berlin specification for Yolov3 and Celo improvement proposals in a hardfork
named by the community as _Donut_.

This hardfork includes pre-Berlin EIPs like simple EVM subroutines and gas cost increases for state access opcodes, as well as BLS precompiles and extensible hash-function precompiles.

This document proposes the following blocks at which to implement these changes in the Celo networks:
- `TBD` on Baklava Testnet ()
- `TBD` on Alfajores Testnet ()
- `TBD` on Celo Mainnet ()
For more information on the opcodes and their respective EIPs and CIP implementations, please see the _Specification_
section of this document.

## Motivation

The enhance the EVM's capabilities and while pushing forward the Celo blockchain functionality in order to enable economic conditions for prosperity as well as adopting Ethereum's Berlin specifications.

## Specification

The Donut Hardfork specification meta document includes the following proposals:
* [CIP 25: Ed25519 Precompile](https://github.com/celo-org/celo-proposals/blob/master/CIPs/cip-0025.md)
* [EIP 2537: BLS Curve Operations on 12-381](https://eips.ethereum.org/EIPS/eip-2537)
* [EIP 2539: BLS Curve Operations on 12-377](https://eips.ethereum.org/EIPS/eip-2539)
* [CIP 20: Extensible Hash Function Precompile](https://github.com/celo-org/celo-proposals/pull/44)
* [CIP 21: Governable Lookback Window](https://github.com/celo-org/celo-proposals/pull/48)
* [CIP 22: Epoch Snark Data](https://github.com/celo-org/celo-proposals/pull/67/files#diff-57a5e04aa05b40794aa8beb293d2b68c967dab4d7f625546baec17a8e5bb568b)

## Rationale

Functionality: We add a number of new features and precompiles to enable new cryptographic protocols and support Plumo.

Validator Quality of life: CIPs 21 and (split etherbase?) address common validator needs wrt reward allocation.

## Implementation

Adoption of the content of this CIP requires a hardfork as it introduces changes that aren't backward compatible. The following clients with Celo support will implement the Celo Donut hardfork specification:
- celo-blockchain

## Copyright

This work is licensed under the Apache License, Version 2.0.
