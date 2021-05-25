---
cip: <to be assigned>
title: Granda Mento
author: Trevor Porter (@tkporter), Albert Wang (@albertclabs)
discussions-to: https://forum.celo.org/t/discussion-on-granda-mento-enabling-larger-stablecoin-mints/966
status: Draft
type: Standards Track
category (*only required for Standards Track): <Ring 0, 1, 2, 3>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
license: Apache 2.0
---

## Simple Summary

Granda Mento is a mechanism to facilitate large CELO <-> stable token (e.g. cXXX) exchanges that aren't suitable via Mento or OTC. A new contract is created that can exchange `CELO <-> stable token` for any stable token Governance explicitly enables. Exchanges via this contract must be approved by a multisig and can be vetoed by Governance.

## Abstract

There are no existing solutions for making large exchanges ($1m+) involving stable tokens. Apart from cUSD minted for validator payments every epoch, stable tokens can only be minted via Mento. Large volume exchanges ($50k+) via Mento experience higher slippage (1-2%) due to the limited sizes of the constant product market maker buckets. Granda Mento is a proposed a mechanism for large volume stable token exchanges, primarily to enable minting of large quantities of stable tokens.

## Motivation

There are no existing avenues that are able to satisfy high volume stable token exchanges on the order of millions. Users looking to make larger exchanges involving stable tokens are faced with 3 options:
1. Exchange `CELO <-> stable token` via Mento slowly over a longer period of time.
2. Arrange one or many `XXX <-> stable token` OTC trades.
3. Create a large limit order of `fiat <-> stable token` on a centralized exchange in hope it gets filled over time.

The existing implementation of Mento is suitable for facilitating low volume exchanges, e.g. up to thousands of stable tokens at a time, with minimal slippage. For cUSD, slippage of over 2% starts to occur on both Mento and on centralized exchanges at trade sizes of ~$50k+. OTC trading satisfies the needs of medium volume exchanges, e.g. ~$100k, but similarly takes a hit of around 2-3%. For lower total supply stable tokens with lower liquidity, larger OTC trades or centralized exchange orders may not be possible.

Increasing Mento bucket sizes permanently to allow larger exchanges via Mento will reduce slippage, but it puts a more significant portion of the Reserve at risk.

Building a process to exchange large (~$1 million+) amounts of CELO for stable tokens will enable large entities to make significant block purchases of stable tokens at scale. While minting of the stable token is expected to be the main use case, reversibility (ie exchanging `stable token -> CELO`) is desirable to instill higher confidence when exchanging large quantities.

While Granda Mento will help facilitate `CELO <-> stable token` exchanges, this is ultimately meant as a mechanism to help purchasers with fiat (e.g. USD) who are seeking to own Celo's corresponding stable token (e.g. cUSD). Purchasers would come to an agreement with a broker (e.g. the Celo Foundation) who owns an existing amount of CELO. The broker would be the one to make the `CELO -> cUSD` exchange on-chain.

The proposed implementation is being considered in the medium-term for large scale exchanges until Mento is possibly reworked to support more efficient large-scale minting on a sustained basis.

## Specification

At a high level, the design involves:
1. Anyone can submit an exchange proposal.
   * The amount of stable tokens being bought/sold in the exchange is required to be within a governable range.
   * The assets being sold (ie either the stable token or CELO) in the exchange are deposited.
   * The current oracle price for the exchange is recorded.
2. The proposed exchange must be approved by a multisig that has previously been authorized by Governance. At this point, the exchange still cannot be executed.
3. A forced waiting period of X days must elapse before the exchange can be executed. During this time, Governance can choose to veto the exchange, refunding the exchange proposer.
4. After the waiting period, the exchange is executed according to the oracle price recorded in step 1. A fee is imposed upon the exchange through the use of a "spread".

An exchange can have the following states:
1. Proposed - The exchange has been proposed, but not yet approved by the approver. The proposer may still cancel their proposal and be refunded their deposit.
2. Approved - The exchange has been proposed and approved by the approver, but not yet executed or vetoed. The proposer may not cancel their proposal.
3. Executed - The exchange has been proposed, approved by the approver, and executed.
4. Cancelled - The exchange has been cancelled by the proposer when it was in the Proposed state, or it has been vetoed by Governance when it was in the Approved stage.

### Specific Changes:

#### Modifications of existing contracts:

##### `StableToken.sol`

Currently, `StableToken.sol` only allows its `Exchange.sol` or `Validators.sol` to [mint](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L225) new stable tokens. Similarly, only its `Exchange.sol` can [burn](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L275) stable tokens.

The following modifications will be made:
1. **Modification to [mint()](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L224)**
   * Allow `msg.sender` to be `GrandaMento` in addition to the existing permitted senders, Exchange and `Validators.sol`.
2. **Modification to [burn()](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L273)**
   * Allow `msg.sender` to be `GrandaMento` in addition to the existing permitted sender, the Exchange.

#### New contracts:

##### `GrandaMento.sol`

A new contract, `GrandaMento.sol`, is created, added to the Registry with the identifier `GrandaMento`, and owned by Governance. `GrandaMento` is a singleton contract that is capable of facilitating exchanges for multiple stable tokens-- that is, with the approval of Governance, `GrandaMento` could support `CELO <-> cUSD`, `CELO <-> cEUR`, and future `CELO <-> cXXX` exchanges. `GrandaMento` must be a spender of `Reserve.sol`, and supported stable tokens must allow `GrandaMento` to mint/burn them.

The contract has the following configurable parameters:

1. **`address public approver;`** - Set by Governance. Intended to be a multisig and has the authority to approve exchanges. The signers for the multisig are at the discretion of Governance, but it would comprise of community members or members on behalf of organizations that are aligned with the Celo network/ecosystem.
2. **`uint256 public exchangeWaitPeriodSeconds;`** - Set by Governance. The minimum amount of time in seconds that must elapse between a proposed exchange being approved and when the exchange can be executed. Should give sufficient time for Governance to veto an approved exchange.
3. **`FixidityLib.Fraction public spread;`** - Set by Governance. The percent fee imposed upon an exchange execution.
4. **`mapping(address => ExchangeLimits) stableTokenExchangeLimits`** - Set by Governance via `setStableTokenExchangeLimits`. A mapping of stable token address to a struct containing two `uint256`s, `minExchangeAmount` and `maxExchangeAmount`.

The contract has the following functions:

1. **`function proposeExchange(address stableToken, uint256 sellAmount, bool sellCelo) external returns (uint256)`** - Called by an exchange proposer to propose an exchange.
   * Callable by anyone.
   * If `sellCelo` is true, CELO is the asset being sold and `stableToken` is the asset being bought. If `sellCelo` is false, `stableToken` is the asset being sold and CELO is the asset being bought.
   * Requires the amount of `stableToken` being bought/sold to be within the token-specific range `[stableTokenExchangeLimits[stableToken].minExchangeAmount, stableTokenExchangeLimits[stableToken].maxExchangeAmount]` that is set by Governance via `setStableTokenExchangeLimits` (described later).
   * Deposits the full amount of the asset being sold into the contract.
   * Records:
     * The exchange as in the Proposed state.
     * A struct with the following info is stored in a mapping with an `id` key:
       * The stable token address.
       * The amount of the asset being sold.
       * Whether CELO is being sold.
   * Returns:
     * The `id` of the struct in the mapping.
2. **`function approveExchangeProposal(uint256 id) external`** - Approves a proposed exchange.
   * Only callable by the approver address.
   * Marks the proposed exchange with the given `id` as approved.
   * Records:
     * The exchange as in the Approved state.
     * The timestamp at which the approval occurred (ie `block.timestamp`).
3. **`function cancelExchangeProposal(uint256 id) external`** - Cancels a proposed exchange.
   * The permitted caller depends upon the state of the exchange proposal:
     * If in the Proposed state, can only be called by the proposer.
     * If in the Approved state, can only be called by Governance.
     * If in any other state, cannot be called.
   * Refunds the proposed exchange's deposited sell asset to the proposer.
   * Records:
     * The exchange as in the Cancelled state.
4. **`function executeExchange(uint256 id) external`** - Executes an exchange.
   * Callable by anyone.
   * Requires the exchange proposal to have been approved.
   * Requires the required waiting period for an approved exchange to have elapsed since the time it was approved (ie `block.timestamp - approval timestamp >= exchangeWaitPeriod`)
   * Makes the exchange:
     * If selling CELO and purchasing stable token:
       * The CELO is sent to the Reserve.
       * Stable token is minted to the exchange proposer according to the rate originally recorded when the exchange was proposed.
     * If selling stable token and purchasing CELO:
       * The stable token is burned.
       * CELO is transferred from the Reserve to the exchange proposer according to the rate originally recorded when the exchange was proposed.
   * Records:
     * The exchange as in the Executed state.
5. **`function setStableTokenExchangeLimits(address stableToken, uint256 minExchangeAmount, uint256 maxExchangeAmount) external`**
   * Only callable by Governance.
   * Sets the minimum and maximum amounts of stable token that can be minted/burned in an exchange by updating `stableTokenExchangeLimits[stableToken]`.
     * Effectively "enables" exchanging a stable token if the min & max amounts were previously 0.
     * Effectively "disables" exchanging a stable token if setting the min & max amounts to 0.

## Rationale

Granda Mento is not intended to undermine the market or provide a way for large players to "dump" CELO for cXXX. It's also not intended to be an exceptionally elegant or complicated solution-- instead, this is looking to be something that is simple, straightforward, and works alongside the existing implementation of Mento. Any larger-scale Mento design modifications to facilitate large stable token mints are out of scope, and should be considered for a medium/long term design change to the stability protocol. The proposed implementation attempts to navigate the tradeoffs of complexity, centralization, economic safety, and ease of use for all actors in the Celo ecosystem.

Some ideas other than the proposed approach include:

1. Exchanges have a delay and require a deposit of the sold asset. Any exchange must be explicitly approved by a governance proposal to be executed.
   * This relies upon the community as the arbiter of what exchanges are "safe."
   * This could exacerbate the already-present voter fatigue.
2. Large-volume exchanges could be made by anyone without Governance approval.
   * The most compelling approach to this would involve an auction. This is complicated to implement, requires careful design, and was intentionally removed from the original Mento design.

Approach (2) was deemed risky and complicated. Approach (1) felt more in line with the goal of having Granda Mento be simple and safe, but ultimately the involvement of Governance for every exchange felt unsustainable and exhausting for voters. The proposed implementation involving an exchange delay, a multisig for approvals, and Governance in the (likely rare) event the community wishes to veto an exchange, aims to relieve voter exhaustion while still providing the security of community scrutiny.

The proposed implementation prohibits the proposer from cancelling their own exchange proposal after it has been approved. This is to prevent Granda Mento from being used as a "free option," where the proposer can decide against the exchange late in the process if the price of the exchange is no longer favorable.

[Initial discussions around Granda Mento](https://forum.celo.org/t/discussion-on-granda-mento-enabling-larger-stablecoin-mints/966) showed concern around providing a "fair" price to ensure that large players do not have an unfair opportunity to "dump" CELO for stable tokens. Some ideas for pricing were considered:

1. Oracle price at the time of exchange proposal.
   * This is vulnerable to oracle attacks.
2. The price is included by the exchange proposer, and is required to be within X% of the current oracle price.
   * This is vulnerable to oracle attacks.
   * This puts more control in the hands of the exchange proposer.
3. Oracle price at the time of exchange execution.
   * This is vulnerable to oracle attacks.
   * Exchanger doesn't know the price at the proposal time.
4. TWAP between exchange proposal and exchange execution.
   * Involves smart contract changes to have the TWAP available on chain.
   * Exchanger doesn't know the price at the proposal time.

It's desirable for both the exchanger and for the Celo community to know what price will be used for the exchange-- this way, the exchanger knows what they're committing to, and the community can decide if they agree with the price. Approaches (1) and (2) the only options that involve knowledge of the price at the start of the trade. While these are both vulnerable to oracle attacks, the proposed implementation's approver and Governance veto serve as safeguards against an exchange with a manipulated price being executed. Because there is not a strong user need behind (2), (1) has been chosen. While the use of a TWAP for the price may be nice, the implementation complexity of implementing a TWAP on-chain is high, and the benefit is slim.

## Backwards Compatibility

The proposal is largely additive, and includes very small changes to existing contracts. Changes are expected to be entirely backward compatible.

## Test Cases
N/A

## Implementation
TBD

## Security Considerations

Implementation Risks:
* Modifications to logic in `StableToken.sol` around which addresses can mint/burn.
* `GrandaMento.sol`'s logic around minting/burning stable tokens or spending Reserve funds must be accurate.
* Exchanges in `GrandaMento.sol` must only be executable if they meet the required criteria.

Operational Risks:
* The multisig may become compromised or approve exchanges that are not favorable from the community's perspective. Governance must be on the lookout for exchange proposals to veto.

Economic Risks:
* The stability protocol has never dealt with large mints/burns apart from the initial allocations of cUSD and cEUR. An extremely large mint/burn quantity could potentially affect stability.

## License
This work is licensed under the Apache License, Version 2.0.
