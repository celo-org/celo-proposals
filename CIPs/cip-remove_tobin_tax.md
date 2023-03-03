---
cip: <to be assigned>
title: Remove Tobin Tax
author: Paul Lange (@palango)
discussions-to: <URL>
status: Draft
type: Standards Track
category: Ring 0
created: 2023-03-03
license: Apache 2.0
---

## Simple Summary

Remove the [Tobin Tax](https://docs.celo.org/protocol/stability/tobin-tax) from the Celo protocol. This removes a source of potential confusion for users and simplifies the protocol.

## Abstract

This CIP proposes to remove the Tobin Tax from the Celo protocol. The Tobin Tax is a small fee that is levied on CELO transfers when the reserve ratio falls below a certain value. The Tobin tax has never been enabled on mainnet.

Currently the client includes a special transfer function that checks if a Tobin Tax has to be paid and pays it if necessary. This code is proposed to be removed.

## Motivation

The Tobin Tax creates two problems.

First, it can lead to confusion for users, when two equal transactions result in different outcome, depending on whether or not the Tobin Tax has to be paid.

Secondly, the implementation complicates both the Celo protocol smart contracts and the client software in several ways.

- It uses additional call stacks when executing EVM bytecode in the virtual machine, leading to  incompatible behaviour with Ethereum.
- It complicates tools that track CELO on the network and will require changes for every tool build on Ethereum to run on Celo.
- A CELO transaction that pays a Tobin Tax will still cost 21,000 gas, even though it does two transfers. This means more execution work for the client for the same gas budget.

Removing the Tobin Tax solves those problems.

Additionally the reserve is bound to stable coins that are developed by Mento, which is an independent entity now. Therefore separation of the core protocol from the reserve is desirable.

## Specification
### Blockchain node

Currently the Tobin Tax is injected in the block context [here](https://github.com/celo-org/celo-blockchain/blob/master/core/vm/vmcontext/context.go#L48). This can be replaced with default `Transfer` function.

This switch needs to be enabled during a hardfork.

Draft PR: https://github.com/celo-org/celo-blockchain/pull/2029

### Contracts

Once the hardfork is completed all smart contract functionality regarding the Tobin Tax can be deleted. This includes various fields in the `Reserve` smart contract. A draft PR is available at https://github.com/celo-org/celo-monorepo/pull/10206.

Once this work is done it can be put on-chain after evaluation of a corresponding governance proposal.

## Rationale

An alternative approach would be to keep all code for the Tobin Tax and not enable it. But given that it has never been enabled in the first place, removing it is preferable.

## Backwards Compatibility

The Tobin Tax was only paid for CELO transfers. As it was handled by the transfer function, no backwards incompatible changes are done by removing it.

## Implementation

There current code that calculates and pays the Tobin Tax can be disabled at the hardfork in which this CIP is enabled. It will be replaced by the default transfer function instead.

## Security Considerations

No security implications are known.

## License
This work is licensed under the Apache License, Version 2.0.
