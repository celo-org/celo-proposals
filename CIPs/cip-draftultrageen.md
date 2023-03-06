---
cip: <to be assigned>
title: Ultragreen Celo
author: <a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s), e.g. (use with the parentheses or triangular brackets): FirstName LastName (@GitHubUsername), FirstName LastName <foo@bar.com>, FirstName (@GitHubUsername) and GitHubUsername (@GitHubUsername)>
discussions-to: <URL>
status: Draft
type: Standards Track
category (*only required for Standards Track): <Ring 0, 1, 2, 3>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
license: Apache 2.0
---

## Simple Summary
Celo has since its inception, in April 2020, had a focus on creating a positive impact on the plannet, becoming the first carbon-negative blockchain. This proposals wants to take that efford further and redistribute half of base fees of transactions to a new Green Fund (GF) and burn the other half, base fees are now sent fully to the Community Fund. The on-chain [Carbon Offsetting Fund](https://docs.celo.org/protocol/pos/epoch-rewards-carbon-offsetting-fund) also aims to be overhauled for a more transparent approach.

## Abstract
This proposals aims to oultine the requirements to make Celo fund on-chain ReFi initiatives with half of the base fees collected from transactions. The other half aims to improve the already deficitary tokenomics by burning it. The Carbon Offsetting Fund is meant to be transfered to an on-chain contract owned by Governance.

## Motivation
The motivation is to make Celo more efficient in how tokens are allocated and to bring more on-chain transparency about how the Carbon Offsetting Fund is run.

## Specification

### Current distribution of the base tx fees

100% of the base fees are getting allocated to the Community Fund (Governance Contract Address).

### Current distribution of Epoch Rewards

Thought the on-chain [Carbon Offsetting Fund](https://docs.celo.org/protocol/pos/epoch-rewards-carbon-offsetting-fund), the blockchain is allocating 0.1% of the daily epoch rewards. 

### Proposed distribution for base tx fees

* Re-routing 50% of the base fees in Celo to the address `0x000000000000000000000000000000000000dEaD`. This will in practice take the Celo out of circulation. The other half will be sent to the CF.
* 50% of the base fees paid in non-Celo tokens (currently Mento Token) will be send to a smart contract. That smart contract will use this tokens to buy Celo from on-chain markets and burn it. Examples of suitable on-chain markets are Mento, Uniswap and Ubeswap.

### Proposed allocation for Carbon Offsetting Fund

* Create a smart contract for the GF.
* Assign the parameter `carbonOffsettingPartner` in EpochRewards to the address of this contract.
* Re-route 50% of Celo and non-Celo base tx fees to the GF.

The GF will take the funds and exchange them for ReFi assets, like on-chain Carbon Credits. A first implementation of this contract may be a multisig for simplicity. A first implementation of the GF could be a multisig.

## Rationale

### Burn

The reason the implementation of Celo burn is done by sending it to a specific address is that as Celo is not an standard ERC20 token, as its balance is tracked natively by the blockchain and not in the usual Solidity storage, changing this would be a hardfork. Even though there are changes in this proposals that already require a hardfork, making the less amount of breaking changes is something that it's expected. On top of that, if we followed an implementation closer to what Ethereum did, by simply making the balance dissapear, it would be harder for indexers and from smart contract how much has been burned. If the burned balance it is in an address it is trivial to know how much the burn has been historically.

### Green Fund

The desition to add an smart contract as GF and re-route the Carbon Offsetting Fund to it is trivial, as this is easilty supported by Celo's Governance Process.

## Backwards Compatibility

Changing the allocation of the base fee is a hardfork. Chaing the `carbonOffsettingPartner` is already supported and can be done with a Governance proposal.

## Test Cases
Test cases for an implementation are mandatory for CIPs that are affecting consensus changes. Other CIPs can choose to include links to test cases if applicable.

## Implementation
* Implementation for a first version of a FeeBurner contract can be found [here](https://github.com/celo-org/celo-monorepo/commit/1bd83f68e8fd77ccfbe6b4f8051d64bde0d13ee7).

## Security Considerations
* Exchanging tokens in the open market has risk of front-running and liquidity.

## License
This work is licensed under the Apache License, Version 2.0.
