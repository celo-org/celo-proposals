---
cip: 53
title: Remove Minimum Client Version Check
author: Karl Bartel (@karlb)
discussions-to: https://github.com/celo-org/celo-proposals/discussions/361
status: Accepted
type: Standards Track
category: Ring 0
created: 2023-02-27
license: Apache 2.0
---

## Simple Summary

Remove the `minimumClientVersion` from the `BlockchainParameters` contract and all code using it.

## Abstract

The protocol contains a mechanism to store and update a minimum client version number on the blockchain. Celo clients should periodically read this version number and shut down if they are older than the specified version. This mechanism did not prove to be useful in practice, so this CIP suggests removing it.

## Motivation

The client version check has been added to avoid problems when incompatible client versions interact with each other. The past has shown that changes which cause such incompatibilities always happen along with a hard fork. The hard fork ensures that incompatible versions won't be on the same fork, making additional version checks unnecessary.

## Specification

Remove the following functions from the BlockchainParameters contract:

- `setMinimumClientVersion`
- `getMinimumClientVersion`

Celo clients will not call `getMinimumClientVersion` and continue operation regardless of their version number.

## Rationale

There is no specific need to remove this feature, but removal will simplify the code base and reduce the divergence from upstream geth, thereby making Celo clients more maintainable. Since there are no likely scenarios in which the feature would be helpful, the maintenance costs outweigh the benefits of having the feature.

## Backwards Compatibility

Removal of `getMinimumClientVersion` in the contract breaks backwards compatibility. The client can easily be changed not to call this function, but removing the functions from the contract must wait until the next hard fork to avoid breaking existing clients.

## Implementation

Remove the following elements from the BlockchainParameters contract:

- `function setMinimumClientVersion`
- `function getMinimumClientVersion`
- `struct ClientVersion`
- `ClientVersion private minimumClientVersion`
- Remove `major`, `minor`, `patch` parameters from constructor

PR: https://github.com/celo-org/celo-blockchain/pull/2026

Remove from `blockchain_parameters.go`:

- [SpawnCheck](https://github.com/karlb/celo-blockchain/blob/a0df3fb5d946524d1d34d8b182d1013223c84984/contracts/blockchain_parameters/blockchain_parameters.go#L139-L153)
- [checkMinimumVersion](https://github.com/karlb/celo-blockchain/blob/a0df3fb5d946524d1d34d8b182d1013223c84984/contracts/blockchain_parameters/blockchain_parameters.go#L113-L129)
- The [version check](https://github.com/karlb/celo-blockchain/blob/a0df3fb5d946524d1d34d8b182d1013223c84984/cmd/utils/flags.go#L643-L646) flag and [its usage](https://github.com/karlb/celo-blockchain/blob/a0df3fb5d946524d1d34d8b182d1013223c84984/cmd/geth/main.go#L458-L469)

PR: https://github.com/celo-org/celo-monorepo/pull/10204

## Security Considerations

No known security considerations.

## License

This work is licensed under the Apache License, Version 2.0.
