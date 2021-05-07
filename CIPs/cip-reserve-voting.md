---
cip: TODO
title: Staking of Reserve CELO holdings
author: Asa Oines (@asaj)
discussions-to: <TODO>
status: Draft
type: Standards Track
category: Ring 1
created: 2021-05-06
requires: A not-yet-proposed CGP to increase `maxNumGroupsVotedFor`
license: Apache 2.0
---


## Simple Summary
This is a proposal to add the ability for the Reserve to vote for validator groups with a portion of its CELO holdings so that it can earn epoch rewards.

## Abstract
By modifying the Reserve smart contract to allow anyone to use a portion of its CELO holdings to vote for a validator group proportional to that group's share of the non-Reserve votes, we can ensure that the Reserve is able to earn [epoch rewards](https://docs.celo.org/celo-codebase/protocol/proof-of-stake/epoch-rewards) without biasing validator elections, further strengthening the [Celo price stability protocol](https://docs.celo.org/celo-codebase/protocol/stability).

## Motivation
The motivation behind this proposal is to ensure that the CELO assets held by the Reserve are able to earn inflationary epoch rewards. This bolsters the overcollateralization ratio and thus strengthens the price stability protocol.

These rewards could also be used to fund other activities. For example portion of these rewards could be used to subsidize sustainable yields on Celo stable assets (e.g. cUSD), furthering the adoption of these assets.

## Specification

A new contract `ReserveVoter.sol` should be added as a reserve spender and another reserve address, implementing the interface below.
Note that `Reserve.getReserveGoldBalance`, and similar functions, should be modified to include locked CELO balances.

```
  /*
   * Functions to govern the locking and unlocking of CELO to maintain a target as a function of
   * the reserve's overall CELO holdings.
   */

  /**
   * @notice Returns the amount of CELO that should be locked by the Reserve.
   * @dev This should be low enough to ensure sufficient Mento liquidity.
   */
  function getTargetLockedBalance() public view returns (uint256);

  /**
   * @notice Locks CELO held by the reserve up to the target locked balance.
   * @return The value of CELO locked by this transaction.
   * @dev This function should first consider re-locking pending withdrawals.
   */
  function lock() external returns (uint256);

  /**
   * @notice Unlocks CELO held by the reserve down to the target locked balance.
   * @return The value of CELO unlocked by this transaction.
   */
  function unlock() external returns (uint256);

  /**
   * @notice Withdraws CELO that has been unlocked after the unlocking period has passed.
   * @param index The index of the pending withdrawal to withdraw.
   */
  function withdraw(uint256 index) external;


  /*
   * Functions to govern voting with the Reserve's locked CELO to ensure that the top
   * `maxGroupsVotedFor` groups can receive votes from the reserve in proportion to their share of
   * votes not cast by the reserve.
   */


  /**
   * @notice Returns the amount of CELO that the Reserve should use to vote for `group`.
   * @dev This should be equal to the proportion of total votes not cast by the Reserve that
   *      `group` is receiving times the Reserve's current locked balance.
   */
  function getTargetVotes(address group) public view returns (uint256);

  /**
   * @notice Casts pending votes for `group` up to the target votes.
   * @param group The validator group to vote for.
   * @param lesser The group receiving fewer votes than `group`, or 0 if `group` has the
   *   fewest votes of any validator group.
   * @param greater The group receiving more votes than `group`, or 0 if `group` has the
   *   most votes of any validator group.
   * @return True upon success.
   */
  function vote(address group, address lesser, address greater) external returns (bool);

  /**
   * @notice Revokes pending votes for `group` if the current votes are greater than target.
   * @param group The validator group to revoke votes from.
   * @param lesser The group receiving fewer votes than the group for which the vote was revoked,
   *   or 0 if that group has the fewest votes of any validator group.
   * @param greater The group receiving more votes than the group for which the vote was revoked,
   *   or 0 if that group has the most votes of any validator group.
   * @param index The index of the group in the account's voting list.
   * @return True upon success.
   */
  function revokePending(
    address group,
    address lesser,
    address greater,
    uint256 index
  ) external nonReentrant returns (bool);

  /**
   * @notice Revokes active votes for `group` if the current votes are greater than target.
   * @param group The validator group to revoke votes from.
   * @param lesser The group receiving fewer votes than the group for which the vote was revoked,
   *   or 0 if that group has the fewest votes of any validator group.
   * @param greater The group receiving more votes than the group for which the vote was revoked,
   *   or 0 if that group has the most votes of any validator group.
   * @param index The index of the group in the account's voting list.
   * @return True upon success.
   * @dev Fails if there are pending votes that could be revoked instead.
   */
  function revokeActive(
    address group,
    address lesser,
    address greater,
    uint256 index
  ) external nonReentrant returns (bool);

  /**
   * @notice Transfers all votes from `groupA` to `groupB`.
   * @param groupA The validator group to revoke votes from.
   * @param groupB The validator group to cast votes for.
   * @param lesserA The group receiving fewer votes than `groupA`.
   * @param greaterA The group receiving more votes than `groupA`.
   * @param lesserB The group receiving fewer votes than `groupB`.
   * @param greaterB The group receiving more votes than `groupB`.
   * @param index The index of `groupA` in the account's voting list.
   * @return True upon success.
   * @dev This function exists to ensure that the reserve always casts its votes for the top
   *      `maxNumGroupsVotedFor`. It reverts if the reserve is already voting for groupB, if the
   *      has voted for fewer then `maxNumGroupsVotedFor` groups, or if `groupB` has received fewer
   *      votes than `groupA`, not counting votes already cast by the reserve.
   */
  function transfer(
    address groupA,
    address groupB,
    address lesserA,
    address greaterA,
    address lesserB,
    address greaterB,
    uint256 index
  ) external nonReentrant returns (bool);

  /**
   * @notice Activates the reserve's pending votes for `group`.
   * @param group The validator group being vote for.
   * @return True upon success.
   */
  activate(address group) external returns (bool);
```

## Rationale

The proposed implementation is designed such that anyone can rebalance the reserve's votes.
The protocol is designed to target a specific amount of CELO to be used for voting, and to allocate those votes proportionally across the top validator groups, such that the elected validator set is not impacted.
The functions specified allow users to perform actions that help the reserve converge towards these goals.

### Targeting a locked balance

The protocol should specify a target locked CELO balance that balances the need for CELO liquidity on Mento with the desire for as much CELO to be earning epoch rewards as possible. To start, simple heuristics such as "X% of the Reserve's CELO holdings", "ensure at least X CELO is unlocked", or even "lock X CELO" should be sufficient, where X is a governable parameter.

### Targeting votes

#### Maintaining impartiality
The implementation should ensure that the reserve stays impartial. We define impartiality as an implementation that meets the following criteria:
```
The votes cast by the reserve should not affect the elected validator set.
```

Celo uses the d'Hodnt algorithm to elect validators, which is an algorithm for proportional representation. For those familiar with this algorithm, it should be self evident that the reserve remains impartial so long as votes cast by the reserve do not change the total proportion of votes each group receives.

As the d'Hodnt algorithm is not influenced by the number of votes received by groups that do not elect a validator, if we agree with the statement above, we can agree that the reserve does not need to vote for groups that would otherwise not elect a validator in order to remain impartial.

Ultimately, the implementation is designed such that anyone can rebalance the Reserve's votes, such that the equilibrium point is the reserve voting for the maximum number of groups it is allowed to vote for, in proportion to those groups' share of votes not cast by the reserve.

So long as the maximum number of groups that an account can vote for is at least the size of the maximum validator set size, the proposed implementation should remain impartial.

#### Ensuring the top groups receive votes
The implementation is designed such that if more groups exist than the reserve can vote for, it only casts votes for the groups receiving the most votes, so as to remain impartial.

This is achieved by the `transfer` function, which allows votes to be transferred from `Alice` to `Bob` so long as:
1. The reserve is already voting for the maximum number of groups
2. The reserve is not already voting for `Bob`
3. `Bob` is receiving more non-reserve votes than `Alice`

This ensures that votes can always be transferred such that the reserve is voting for the top `maxNumGroupsVotedFor` validator groups.


## Backwards Compatibility
This should be fully backwards compatible. No changes to existing core contracts are needed.

## Test Cases
To follow.

## Implementation
To follow.

## Security Considerations
Increasing `maxNumGroupsVotedFor` results in a linear increase in the amount of gas a query for a user's locked CELO balance can consume.
Special care should be taken to audit all of these code paths to ensure that this value can be increased to greater than the max validator set size and guarantee that core operations, such as querying the Reserve's CELO balance when calculating epoch rewards, remain sufficiently performant.

## License
This work is licensed under the Apache License, Version 2.0.
