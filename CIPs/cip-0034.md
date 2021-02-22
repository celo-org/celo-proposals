---
cip: 34
title: Implement "flash swap" support in Exchange.sol
author: Zviad Metreveli (@zviadm)
discussions-to: https://github.com/celo-org/celo-proposals/issues/167
status: Draft
type: Standards Track
category: Ring 1
created: 2021-02-17
license: Apache 2.0
---

## Simple Summary
Enable "flash swap" support similar to UniswapV2 implementation in Exchange.sol

## Abstract
UniswapV2 changed its core implementation from V1 to transfer output tokens first
optimistically, before having to supply the input in the same transaction. This has
unlocked opportunities to do "flash swaps" and zero capital on-chain arbitrages.

## Motivation

As Celo financial ecosystem grows, efficient on-chain arbitrage becomes critical to
its success. Allowing "flash swaps" decreases capital requirements for simple
on-chain arbitrages, making it less costly and more efficient.

Example: If you have a lending platform with CELO/cUSD liquidations, now a system can be
written that can arbitrage liquidations against core Mento exchange without needing any
initial capital to do so.

## Specification

UniswapV2 provides an example code for implementation:
https://github.com/Uniswap/uniswap-v2-core/blob/master/contracts/UniswapV2Pair.sol#L159

It is a bit more complex compared to doing it in a naive way, with slightly higher gas costs,
but it can bring large amount of benefits as the Celo platform grows.

## Alternative Options

An alternative option would be to allow "flash loan" style borrowing from the reserve with a fairly
high fee. (i.e 0.5-1% fee). That would also unlock the ability to do zero capital on-chain arbitrage
trades, but it would also introduce more unknowns and potentially more complexity in core contracts.


## Backwards Compatibility
This proposal doesn't change any existing behavior, it just adds new capabilities to Exchange.sol.

## Test Cases
TBD

## Implementation
TBD

## Security Considerations
There are no special security considerations, but there is always the risk of implementation mistakes.

## License
This work is licensed under the Apache License, Version 2.0.
