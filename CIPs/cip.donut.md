---
cip: 
title: Donut Hardfork 
author: Yaz Khoury <@YazzyYaz> 
discussions-to:  
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
* [EIP 665: Ed25519 Signature Verification](https://eips.ethereum.org/EIPS/eip-665)
* [EIP 2537: BLS Curve Operations on 12-381](https://eips.ethereum.org/EIPS/eip-2537)
* [EIP 2539: BLS Curve Operations on 12-377](https://eips.ethereum.org/EIPS/eip-2539)
* [CIP 20: Extensible Hash Function Precompile](https://github.com/celo-org/celo-proposals/pull/44)
* [CIP 21: Governable Lookback Window](https://github.com/celo-org/celo-proposals/pull/48)
* [CIP 22: Epoch Snark Data](https://github.com/celo-org/celo-proposals/pull/67/files#diff-57a5e04aa05b40794aa8beb293d2b68c967dab4d7f625546baec17a8e5bb568b)

## Rationale

__Optimization__: EVM improvements via simple subroutines will allow for a more efficient EVM.

__Interoperability__: Allowing compatibility between Celo and Ethereum EVM so developers can easily have an up-to-date EVM on both networks and easily be able to run their smart contracts on both networks.


## Implementation

Adoption of the content of this CIP requires a hardfork as it introduces changes that aren't backward compatible. The following clients with Celo support will implement the Celo Donut hardfork specification:
- celo-blockchain

## Copyright

This work is licensed under the Apache License, Version 2.0.
