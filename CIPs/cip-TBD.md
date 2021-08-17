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
Paying transaction fees using currencies other than CELO should be feasible if a transactor has a balance greater than or equal to the fee.

## Abstract
When paying transaction fees using currencies other than CELO, the transaction should be deemed as valid if the transactor's balance is greater than or equal to the required fee, rather than strictly greater than it. This CIP seeks to modify the transaction fee check rule.

## Specification
When checking if the balance covers the transaction fee using other currencies than CELO in `state_transition` and `tx_pool`, it should pass if the currency balance `>=` transaction fee. The current behavior is to require the currency balance `>` transaction fee.

## Rationale
- Intuitively, anyone should not be required to have more than one dollar if they just intend to buy a one-dollar pizza.
- Transaction fee checks should be consistent across using ELO and other currencies. For now, the transaction validity check on CELO is `>=`, rather than `>`, which is what this CIP proposes for other currencies.

## Backwards Compatibility
This introduces a change in the transaction validity check.

## Test Cases
For each currency that is whitelisted:
- Send a Tx with balance = fee: :x: before HF block, :white_check_mark: after it.
- Send a Tx with balance > fee: :white_check_mark: before/after HF block.
- Send a Tx with balance < fee, :x: before/after HF block.

## Implementation
https://github.com/celo-org/celo-blockchain/pull/1664

## Security Considerations
Applying the CELO transaction validity check rule to other currencies should be secure.

## License
This work is licensed under the Apache License, Version 2.0.
