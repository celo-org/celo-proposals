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

Granda Mento is a mechanism to facilitate large CELO <-> stable token exchanges that aren't suitable via Mento or OTC. A new contract is created that has the authorization by Governance to exchange a limited amount of `CELO <-> stable token`. Exchanges via this contract must be approved by a multisig and can be vetoed by Governance.

## Abstract
> A short (~200 word) description of the technical issue being addressed.

## Motivation
> The motivation is critical for CIPs that want to change the Celo protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the CIP solves. CIP submissions without sufficient motivation may be rejected outright.

Users looking to make larger exchanges involving stable tokens are faced with 3 options:
1. Exchange `CELO <-> stable token` via Mento slowly over a longer period of time.
2. Arrange one or many `XXX <-> stable token` OTC trades.
3. Create a large limit order of `fiat <-> stable token` on a centralized exchange in hopes it gets filled over time.

The existing implementation of Mento is suitable for facilitating low volume exchanges, e.g. up to thousands of stable tokens at a time, with minimal slippage. However, significant slippage of over 2% starts to occur on both Mento and on centralized exchanges at trades sizes of ~$50k+. OTC trading satisfies the needs of medium volume exchanges, eg ~100k for cUSD and likely much less for stable tokens with lower total supply (like cEUR), but similarly takes a hit of around 2-3%. There are no existing avenues that are able to satisfy high volume stable token exchanges on the order of millions.

Increasing Mento bucket sizes permanently to allow larger exchanges via Mento will reduce slippage, but it puts a more significant portion of the Reserve at risk.

Building a process to exchange large (~$1 million+) amounts of CELO for stable tokens will enable large entities to make significant block purchases of stable tokens at scale. While minting of the stable token is expected to be the main use case, reversibility (ie exchanging `stable token -> CELO`) is desirable to instill higher confidence when exchanging large quantities.

While Granda Mento will help facilitate `CELO <-> stable token` exchanges, this is ultimately meant as a mechanism to help purchasers with fiat (eg USD) who are seeking to own Celo's corresponding stable token (eg cUSD). Purchasers would come to an agreement with a broker (eg the Celo Foundation) who owns an existing amount of CELO. The broker would be the one to make the `CELO -> cUSD` exchange on-chain.

The proposed implementation is being considered in the medium-term for large scale exchanges until Mento is reworked to support more efficient large-scale minting on a sustained basis.

## Specification
> The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow for an implementation on the Celo platform (go-celo).

At a high level, the design involves:
1. Anyone can submit an exchange proposal.
   * The amount of stable tokens being bought/sold in the exchange is required to be within a governable range.
   * The assets being sold in the exchange are deposited.
   * The current oracle price for the exchange is recorded.
2. The proposed exchange must be approved by a multisig that has previously been authorized by Governance. At this point, the exchange still cannot be executed.
3. A forced waiting period of X days must elapse before the exchange can be executed. During this time, Governance can choose to veto the exchange, refunding the exchange proposer.
4. After the waiting period, the exchange is executed.

### Specific Changes:

#### `GrandaMento.sol`

A new contract, `GrandaMento.sol`, is created. One deployment of `GrandaMento` exists per stable token. It must have the ability to mint/burn the corresponding stable token and be a spender of `Reserve.sol`.

The contract is owned Governance, is freezable, and has the following configurable parameters:

1. **`address public approver;`** - Set by governance. Intended to be a multisig and has the authority to approve exchanges.
2. **`uint256 public minStableExchangeAmount;`** - Set by governance. Minimum amount of the stable token an exchange can buy/sell.
3. **`uint256 public maxStableExchangeAmount;`** - Set by governance. Maximum amount of the stable token an exchange can buy/sell.
4. **`uint256 public exchangeWaitPeriod;`** - Set by governance. The minimum amount of time that must elapse between a proposed exchange being approved and when the exchange can be executed. Should give sufficient time for Governance to veto an approved exchange.
5. **`uint256 public spread;`** - Set by governance. The percent fee imposed upon the exchange.

The contract has the following functions:

1. **`function proposeExchange(uint256 amount, bool sellCelo) external returns (uint256)`** - Called by an exchange proposer to propose an exchange.
   * Callable by anyone.
   * Requires the amount of stable token being bought/sold to be within the range `[minStableExchangeAmount, maxStableExchangeAmount]`.
   * Deposits the full amount of the asset being sold into the contract.
   * Records:
     * A struct with the following info is stored in a mapping with an `id` key:
       * Which asset is being sold.
       * The amount of the asset being sold.
   * Returns:
     * The `id` of the struct in the mapping.

2. **`function approveExchangeProposal(uint256 id) external`** - Approves a proposed exchange.
   * Only callable by the approver address.
   * Marks the proposed exchange with the given `id` as approved.
   * Records:
     * The timestamp at which the approval occurred (ie `block.timestamp`).
3. **`function vetoExchangeProposal(uint256 id) external`** - Vetoes a previously approved proposed exchange.
   * Only callable by governance.
   * Refunds the proposed exchange's deposited sell asset to the proposer.
   * Records:
     * The exchange as vetoed.
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
     * The exchange as executed.

#### Modifications of existing contracts:

Currently, `StableToken.sol` only allows its `Exchange.sol` or `Validators.sol` to [mint](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L225) new stable tokens. Similarly, only its `Exchange.sol` can [burn](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L275) stable tokens.

The following modifications will be made:
1. **Set mint/burn allowance function** - Sets an allowance for an address to mint or burn an amount of tokens.
   * Only callable by governance.
   * Records:
   * The new mint/burn allowance for the specified address
2. **Modification to [mint()](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L224)**
   * If `msg.sender` is not the Exchange or `Validators.sol`:
     * Require `msg.sender` to have sufficient mint/burn allowance.
     * Subtract the mint/burn allowance of `msg.sender` by the amount minted.
3. **Modification to [burn()](https://github.com/celo-org/celo-monorepo/blob/master/packages/protocol/contracts/stability/StableToken.sol#L273)**
   * If `msg.sender` is not the Exchange:
     * Require `msg.sender` to have sufficient mint/burn allowance.
     * Subtract the mint/burn allowance of `msg.sender` by the amount burned.

## Rationale
>The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.

## Backwards Compatibility
>All CIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CIP must explain how the author proposes to deal with these incompatibilities. CIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Test Cases
>Test cases for an implementation are mandatory for CIPs that are affecting consensus changes. Other CIPs can choose to include links to test cases if applicable.

## Implementation
>The implementations must be completed before any CIP is given status "Final", but it need not be completed before the CIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.

## Security Considerations
>All CIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. CIP submissions missing the "Security Considerations" section will be rejected. A CIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

Implementation Risks:
* Modifications to logic in `StableToken.sol` around which addresses can mint/burn.
* `GrandaMento.sol`'s logic around minting/burning stable tokens or spending Reserve funds must be accurate.
* Exchanges in `GrandaMento.sol` must only be executable if they meet the required criteria.

Operational Risks:
* The multisig


## License
This work is licensed under the Apache License, Version 2.0.
