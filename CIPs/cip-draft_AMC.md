---
cip:
title: A Mock CIP
author: ianmunge0 (@ianmunge0)
discussions-to: https://github.com/celo-org/celo-proposals/blob/master/CIPs/cip-template.md
status: Draft
type: Informational
created: 2022-10-15
requires: 
replaces: 
license: Apache 2.0
---
## Simple Summary
A mock proposal used for developing Governcelo

## Abstract
Governcelo being developed (here)[https://github.com/ianmunge0/make-crypto-mobile-hackathon/tree/governcelo] needs this proposal to test how it's created

## Motivation
Celo is a mobile-first blockchain. Governcelo will make the current method of making a proposal with pull requests easier.

## Specification
Governcelo will use OAuth for GitHub to enable the user to publish a proposal. The dapp will connect to Valora to make transactions involved with a proposal.

## Rationale
In OAuth for GitHub, the device flow method is used. In the GitHub OAuth app, have the device flow option enabled and generate a client secret. The alternate which is web flow works fine with web apps instead of mobile apps. To connect to Valora, the dapp will need the Celo dappkit.

## Backwards Compatibility
The introduction of making a proposal using pull requests from a mobile device will not change anything in the existing method of making a pull request from a command line interface or from a web browser.

## Test Cases
(test title)[https://github.com/ianmunge0/make-crypto-mobile-hackathon/pull/5]
(test title)[https://github.com/ianmunge0/make-crypto-mobile-hackathon/pull/4]

## Implementation


## Security Considerations
Celo uses blockchain technology which securely makes transactions. The dappkit will make transactions on Celo's blockchain. For the OAuth part, the session variables are secured with expo-secure-store.

## License
This work is licensed under the Apache License, Version 2.0.
