---
cip: TBD
title: Modify Transaction Fee Check
author: Tong Wang <@yoduyodu>
discussions-to: https://github.com/celo-org/celo-proposals/issues/274
status: Draft
type: Standards Track
category: Ring 0
created: 2021-08-16
license: Apache 2.0
---

## Simple Summary
Paying transaction fees using non-native currencies should be feasible if a transactor has a balance greater than or equal to the fee.

## Abstract
When paying transaction fees using non-native currencies, the transaction should be deemed as valid if the transactor's balance is greater than or equal to the transaction fee, rather than strictly greater than. This CIP seeks to modify the transaction fee check rule.

## Specification
When checking if the balance covers the transaction fee using non-native currencies in `state_transition` and `tx_pool`, it should pass if the balance `>=` transaction fee. The current behavior is to require the balance to be `>` transaction fee.

## Rationale
- Intuitively, anyone should not be required to have more than one dollar if they just intend to buy a one-dollar pizza.
- Transaction fee checks should be consistent across using native currency CELO and other non-native currencies. For now, the transaction validity check using CELO is `>=`, rather than `>`, which is what this CIP is proposing for non-native currencies.

## Backwards Compatibility
This introduces a change in the transaction validity check.

## Test Cases
TODO:

For each non-native currency that is whitelisted:
- Send a Tx with balance = fee: :x: before HF block, :white_check_mark: after.
- Send a Tx with balance > fee: :white_check_mark: before/after HF block.
- Send a Tx with balance < fee, :x: before/after HF block.

## Implementation
https://github.com/celo-org/celo-blockchain/pull/1664

## Security Considerations
Applying the CELO transaction validity check rule to other non-native currencies should be secure.

## License
This work is licensed under the Apache License, Version 2.0.
