---
cip: <to be assigned>
title: Processes Around Granda Mento Exchanges
author: Trevor Porter (@tkporter)
discussions-to: <URL>
status: Draft
type: Meta
created: 2021-8-13
requires (*optional): <CIP number(s)>
replaces (*optional): <CIP number(s)>
license: Apache 2.0
---

## Simple Summary
This is a description of the processes surrounding Granda Mento (see [CIP 38](cip-0038.md)) activity including:
* Creating an exchange proposal.
* Gathering community feedback with the intent of having an exchange proposal approved.
* Expected behavior of the approver.
* Vetoing an exchange proposal.

## Abstract
Granda Mento, as described in [CIP 38](cip-0038.md), is a mechanism for large CELO <-> stable token exchanges through "exchange proposals" that anyone can create. If rough consensus from the community is achieved, the approver (multisig of community member signers) is expected to approve an exchange proposal. After an exchange proposal is approved, a period of time exists where the community can veto the exchange proposal via a governance proposal. This document provides details on these processes.

## Motivation
While CIP 38 details the implementation of Granda Mento, there is no standard for the surrounding processes. Because Granda Mento relies upon community participation and approver multisig signers executing the rough consensus of the community, it's important for these processes to exist.

## Specification

### Creating a forum post

To field opinions from the community and arrive at a rough consensus to approve an exchange proposal, an exchange proposal must create a post at https://forum.celo.org. A forum post must receive sufficient community input before the approver multisig can determine rough consensus.

An on-chain exchange proposal does not need to be created when the forum post is created if the exchanger is hoping to gauge opinions before locking funds in GrandaMento. However, the exchange proposal should be submitted on-chain prior to achieving rough consensus to allow community members to verify the on-chain exchange proposal for themselves. 

#### Forum post title format

```
GrandaMento Exchange Proposal <on-chain proposal ID> Discussion: <sell amount> <sell token> for <buy amount> <buy token>
```
For example: `GrandaMento Exchange Proposal 1 Discussion: 500,000 CELO for 1,500,000 cUSD`.

If the proposal has not been created on-chain yet, the proposal ID and buy amount can be omitted for the time being, though it should be added later upon exchange proposal creation. For example: `GrandaMento Exchange Proposal <TBD> Discussion: 500,000 CELO for cUSD`.

#### Forum post contents format
```
1. What token and quantity are you selling, and what token are you buying?

2. Who are you?

3. Why do you want to execute the exchange?
// Include the intended use of bought funds.

4. What is the on-chain proposal ID and the on-chain buy amount?
// If you are planning to create the on-chain exchange proposal after
// fielding opinions from the community, please mention so here.

--- Only if selling CELO ---
5. How did you acquire the CELO?
// For example, via the Celo Foundation, Mento, a centralized exchange, OTC, etc.

6. At what price did you acquire the CELO?
```

For extra visibility, after creating the forum post, the exchanger should share the link to the forum post in the #granda-mento Discord channel.

### Achieving rough consensus

[Rough consensus](https://en.wikipedia.org/wiki/Rough_consensus) in favor of an exchange proposal is expected prior to the approver multisig approving an exchange proposal. 51% of discussion participants in agreement is seen as worse than rough consensus, while 99% is seen as better than rough consensus. Rough consensus is seen as sufficient for approval because community members are able to create a governance proposal to veto an approved exchange proposal. "Achieving" rough consensus is defined as the approver multisig deciding to approve the exchange proposal as a result of rough consensus being in favor of the exchange proposal.

A forum post must exist for at least one week prior to rough consensus being achieved. The on-chain proposal must be submitted and the corresponding on-chain proposal ID must be shared in the forum post for at least 4 days prior to rough consensus. This gives community members the opportunity to independently verify the on-chain exchange proposal.

The exchange proposer is expected to answer reasonable questions from the community related to the exchange proposal, motivations, and funds.

Signers of the approver multisig are expected to be active in the forum discussions, must signal their intent to approve at least two days before submitting the approval on-chain, and must post in the forum discussion when approval occurs. Approver multisig signers are able to wait longer than required before approving an exchange proposal, particularly in situations where an exchange proposal is controversial and determining rough consensus is not straightforward.

Below is a table showing the expected timeline:

| **Minimum # of days prior to achieving rough consensus** | **Actor**                 | **Action**                                                      |
|----------------------------------------------------------|---------------------------|-----------------------------------------------------------------|
| 7                                                        | Exchanger                 | Forum post created.                                             |
| 4                                                        | Exchanger                 | Exchange proposal submitted on-chain & shared.                  |
| 2                                                        | Approver multisig signers | On-chain proposal reviewed and intent for approval is signaled. |
| 0                                                        | Approver multisig signers | Approval is submitted on-chain & shared.                        |                      |

### Vetoing an exchange proposal

Exchange proposals that have been approved can be vetoed by the community via a governance proposal. This is most appropriate if the approver multisig has approved an exchange proposal that did not achieve rough consensus.

Anyone can submit a governance proposal by creating a CGP and proposing it on-chain. Passing a governance proposal takes multiple days, so it's important to create a veto governance proposal as early as possible to ensure the governance proposal will execute prior to the exchange proposal's veto period ending.

The CGP will need to include a transaction that calls the `cancelExchangeProposal` function with the correct on-chain proposal ID.

A human-readable template to include in the CGP:

1. Cancellation of the exchange proposal.
  - Destination: `GrandaMento`
  - Function: `cancelExchangeProposal`
  - Arguments: `["<proposal ID to cancel>"]`
  - Value: 0

JSON template to include in the CGP and to submit the governance proposal on-chain:

```
[{"contract":"GrandaMento","function":"cancelExchangeProposal","args":["<proposal ID to cancel>"],"value":"0"}]
```

## Rationale

Forum posts and questions around the acquisition of the funds being sold were originally proposed in a comment on CIP 38 [found here](https://github.com/celo-org/celo-proposals/pull/224#discussion_r639754818). A forum post may be created prior to the exchange proposal being created on-chain to give exchangers the opportunity to gauge the likelihood their proposal will be approved without needing to lock their funds.

The required periods of discussion, community review of the on-chain proposal, and approver signaling attempt to navigate a reasonable time frame for exchangers while still allowing sufficient community discourse.

## Backwards Compatibility

Fully backward compatible.

## Implementation

N/A

## Security Considerations

* An exchange proposer could attempt to create a more controversial proposal on-chain than the terms discussed in the forum post. It's important for any party, whether and approver multisig signer or a community member participating in discussions, to confirm the on-chain data matches the forum post.

## License
This work is licensed under the Apache License, Version 2.0.
