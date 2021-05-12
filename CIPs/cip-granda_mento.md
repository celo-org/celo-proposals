---
cip: <to be assigned>
title: Granda Mento
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
> If you can't explain it simply, you don't understand it well enough. Provide a simplified and layman-accessible explanation of the CIP.

Granda Mento is a mechanism to facilitate large CELO <-> stable token trades that aren't suitable for Mento or OTC. A new contract is created that has the authorization by Governance to trade a limited amount of `CELO <-> stable token`. Trades via this contract must be approved by a multisig and can be vetoed by Governance.

## Abstract
> A short (~200 word) description of the technical issue being addressed.

## Motivation
> The motivation is critical for CIPs that want to change the Celo protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the CIP solves. CIP submissions without sufficient motivation may be rejected outright.

The existing implementation of Mento is suitable for facilitating low volume trades, e.g. up to tens of thousands at a time, without incurring slippage greater than a few percent. OTC trading satisfies the needs of medium volume trades, eg ~100k for cUSD and likely much less for stable tokens with lower total supply (like cEUR), but is unable to satisfy high volume CELO <-> stable token trades on the order of millions.

Mento is able to facilitate smaller `CELO <-> stable token` trades with minimal slippage for smaller trade sizes. However, significant slippage of over 2% starts to occur on both Mento and on centralized exchanges at trades sizes of ~$50k+. OTC trading satisfies the needs of medium volume trades, eg ~100k, but similarly takes a hit of around 2-3%.

Users looking to make larger trades are faced with 3 options:
1. Exchange via Mento slowly over a longer period of time
2. Arrange one or many OTC trades
3. Create a large limit order in hopes it gets filled over time

Increasing Mento bucket sizes permanently to allow larger trades via Mento will reduce slippage for (1), but it puts a more significant portion of the Reserve at risk.

Building a process to exchange large (~$1 million+) amounts of CELO for stable tokens will enable large entities to make significant block purchases of stable tokens at scale. While minting of the stable token is expected to be the main use case, reversibility (ie trading `stable token -> CELO`) is desirable to instill higher confidence when trading large quantities.

While Granda Mento will help facilitate `CELO <-> stable token` trades, this is ultimately meant as a mechanism to help purchasers with USD who are seeking to own cUSD. Purchasers would come to an agreement with a broker (eg the Celo Foundation) who owns an existing amount of CELO. The broker would be the one to make the `CELO -> cUSD` trade on-chain.

The proposed implementation is being considered in the short-term for large scale trades until Mento is reworked to support more efficient large-scale minting on a sustained basis.

## Specification
> The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow for an implementation on the Celo platform (go-celo).

At a high level, the design involves:
1. Anyone can submit a trade proposal.
   * The size of the trade is required to be sufficiently large.
   * The assets being sold in the trade are deposited.
   * The current oracle price for the trade is recorded.
2. The proposed trade must be approved by a multisig that has previously been authorized by Governance. At this point, the trade still cannot be executed.
3. A forced waiting period of X days must elapse before the trade can be executed. During this time, Governance can choose to veto the trade, refunding the trader.
4. After the waiting period, the trade is executed.

### Specific Changes:

#### `GrandaMento.sol`

A new contract, `GrandaMento.sol`, is created. One deployment of `GrandaMento` exists per stable token. It must have the ability to mint/burn the corresponding stable token and be a spender of `Reserve.sol`.

The contract has the following configurable parameters:

1. **Approver address** - Set by governance. Intended to be a multisig and has the authority to approve trades.
2. **Minimum stable token buy/sell quantity for trades** - Set by governance. Minimum amount of stable token a trade must buy/sell.
3. **Required waiting period for an approved trade before execution** - Set by governance. The minimum amount of time that must elapse between a proposed trade being approved and when the trade can be executed. Should give sufficient time for Governance to veto an approved trade.
4. **Spread imposed on trades** - Set by governance. The percent fee imposed upon the trade.

The contract has the following functions:

1. **Propose trade** - Called by a trader to initiate a trade.
   * Callable by anyone.
   * Requires the trade to be larger than the minimum stable token buy/sell quantity.
   * Deposits the full amount of the asset being sold into the contract.
   * Records:
     * Which asset is being sold
     * The amount of the asset being sold
2. **Approve trade proposal** - Approves a proposed trade.
   * Only callable by the approver address.
   * Marks a proposed trade as approved.
   * Records:
     * The timestamp at which the approval for the proposed trade occurred.
3. **Veto an approved trade proposal** - Vetoes a previously approved proposed trade.
   1. Only callable by governance.
   2. Refunds the proposed trade's deposited sell asset.
   3. Records:
     1. The trade as vetoed.
4. **Execute an approved and ready trade** - Executes a trade.
   * Callable by anyone.
   * Requires the trade to have been approved.
   * Requires the required waiting period for an approved trade to have elapsed since the time it was approved.
   * Makes the trade:
     * If selling CELO and purchasing stable token:
       * The CELO is sent to the Reserve.
       * Stable token is minted to the trader according to the rate originally recorded when the trade was proposed.
     * If selling stable token and purchasing CELO:
       * The stable token is burned.
       * CELO is transferred from the Reserve to the trader according to the rate originally recorded when the trade was proposed.
   * Records:
     * The trade as executed.

#### Modifications of existing contracts:

Currently, `StableToken.sol` only allows its `Exchange.sol` or `Validators.sol` to [mint](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L225) new stable tokens. Similarly, only its `Exchange.sol` can [burn](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L275) stable tokens.

The following modifications will be made:
1. **Set mint/burn allowance function** - Sets an allowance for an address to mint or burn an amount of tokens.
   1. Only callable by governance.
   2. Records:
   3. The new mint/burn allowance for the specified address
2. **Modification to [mint()](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L224)**
   * If `msg.sender` is not the Exchange or `Validators.sol`, require `msg.sender` to have sufficient mint/burn allowance.
   * Update the mint/burn allowance of `msg.sender` if appropriate
3. **Modification to [burn()](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L273)**
   * If `msg.sender` is not the Exchange, require `msg.sender` to have sufficient mint/burn allowance.
   * Update the mint/burn allowance of `msg.sender` if appropriate

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
