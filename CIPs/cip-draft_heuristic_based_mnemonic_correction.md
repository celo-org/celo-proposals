---
cip: <to be assigned>
title: Probabilistic error correction for mnemonic phrases
author: Victor Graf <@nategraf>
discussions-to: https://github.com/celo-org/celo-proposals/issues/227
status: Draft
type: Standards Track
category: 3
created: 2021-06-01
license: Apache 2.0
---

### Terminology

The key words "MUST", "MUST NOT", "SHOULD", and "MAY" in this document are to be interpreted as
described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.html).

## Simple Summary

A model-based approach to error correction in BIP-39 mnemonic phrases, enabling probabilistic
recovery of mnemonic phrases with common errors. Includes an edit-distance based instantiation and
theoretical framework for extension to include improved heuristics.

## Abstract

[BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic phrases are widely
used as the seed for account generation in the crypto ecosystem, and a pen-and-paper backup is the
most commonly recommended way to store these phrases to enable account recovery.

In the case that the user has a copy of their phrase with errors, it is possible to recovery the
original phrase as long as the errors are relatively small. This CIP introduces a proposed standard
and framework for correcting errors based on the "noisy channel" model which is commonly used in
[spelling correction programs](https://norvig.com/spell-correct.html) to generate suggested
correction. An instantiation of this standard is described using Levenshtein distance, and assuming
a number of independent random errors are introduced to the mnemonic phrase.

Complimentary to this proposal is CIP-#, which proposes a method for including error correcting
codes in mnemonic phrases.

## Motivation

Mnemonic phrase backups are a powerful tool, but also place a large responsibility on the user to
correctly record and, in the future when an account needs to be recovered, recall the phrase. In
practice, us humans often make mistakes which lead to a phrase being lost or unusable. When this
happens, it can be a catastrophic event for the user, losing any assets or identity held by accounts
derived from the phrase.

In the case that the user has a copy of their phrase with errors, introduced when copying the phrase
to paper, during storage (e.g. water damage, dog nibbled on it), or when typing the phrase into a
wallet application, the raw data (i.e. entropy) may still be available in the key, but has been
obscured by the introduced errors. Given expertise and time, a user may being able to manually
locate and remove any errors present in the phrase. Most users don't have the required expertise or
time. This proposal aims to provide a standard way to remove these errors automatically.

When information is lost from the phrase (e.g. some of the words are entirely missing), or when the
heuristic does not do appropriately model a particular error, heuristics based approaches may fail.
Proposed in CIP-# is the introduction of error correcting codes, which can recover phases with a
number of arbitrary errors, covering many cases that model based approaches cannot.

## Specification

WIP

## Rationale

WIP

## Backwards Compatibility

Model-based error correction described above does not introduce any concerns with backwards
compatibility. Implementations of this model also need not be deterministic, or otherwise return
results consistent with each other.

## Test Cases

WIP

## Implementation

* An implementation of edit-distance based mnemonic phrase correction is implemented in [celo-org/celo-monorepo/pull/8034](https://github.com/celo-org/celo-monorepo/pull/8034)

## Security Considerations

* As with any software that handles the root account secrets, bugs or compromise of applications
  which implement this proposal may lead to the compromise of the user's account.

## License
This work is licensed under the Apache License, Version 2.0.
