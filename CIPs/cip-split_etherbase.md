---
cip: 28
title: Split etherbase
author: Andres di Giovanni <@andres-dg>
discussions-to: https://github.com/celo-org/celo-proposals/issues/95
status: Draft
type: Standards Track
category: Ring 0
created: 2020-11-19
license: Apache 2.0
---

## Simple Summary

Validators maintain the network and are, of course, rewarded for it. If elected during an epoch, they receive validator rewards by the end of it. Additionally, if they are chosen to create the next block, they receive transaction fees. These rewards go to a single configured address in the node, the etherbase. This proposal suggests splitting the etherbase into separate addresses, one for each function the node carries out in the network.

## Abstract

During an epoch, a validator is elected to sign every consensus message. This work is rewarded at the end of the epoch, through validator rewards. The validator is also chosen to process incoming transactions by mining a block. The block creator receives transaction fees for the work done. Currently, geth allows the user to configure a single address, etherbase, to receive both validator rewards and mining transaction fees.

## Motivation

Different rewards get transferred to the same address and the user may want to split them. 

## Specification

Add a flag for each required address:
    - tx-fee-recipient: the node's coinbase while creating blocks
    - miner.validator: validator address for epoch rewards

## Rationale

The miner will be committing new blocks with the coinbase set to tx-fee-recipient. Similarly, other nodes will stop verifying that the block's author corresponds to the block coinbase field in the header, since it might be different from now on.

## Backwards Compatibility

The change will work behind a check of the Donut hard fork. If the fork is not yet active, the provided miner.validator address will be used as the nodes coinbase, the legacy style. This check is done for every block so, when the activation block arrives, the coinbase will be set to the tx-fee-recipient address.

## Test Cases

Two test nets were set up for this.

 - Multiversion before activation: The idea behind this testnet is to check if the network stays healthy with old nodes and new nodes before Donut is active. The net is healthy.

  - New nodes after activation: The testnet activates Donut after migrations and incoming transactions are processed correctly. The blocks have the configured coinbase (tx-fee-recipient flag) as miners. Checking balances of these tx-fee-recipient wallets, transaction fees are indeed transferred to that address.

## License
This work is licensed under the Apache License, Version 2.0.
