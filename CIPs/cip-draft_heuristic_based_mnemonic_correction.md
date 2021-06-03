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

### Theoretical framework

#### Noisy channel model

Given a mnemonic phrase which is known to have errors (e.g. it contains one or more invalid words,
or the checksum does not verify), we might make a number of guesses as to what the original phrase
was. We can do this manually for small number of errors, such as guessing that "open void riple"
should actually be "open void ripple". Intuitively, this is an example of seeing the misspelled word
"riple" and, given a BIP-39 word list or simply from mental dictionary, guess that it is most likely
to have been an attempt at the word "ripple". Furthermore, it's obvious less likely that original
word was "hamster".

In general, and because our first guess is not always correct, this problem can formulated as
generating list of guesses at the source phrase ordered by probability, given an observed phrase
with errors.

Formally, the probability of a hypothesis phrase can be expressed as
![eq1](https://latex.codecogs.com/svg.latex?P%28%5Cbold%7Bh%7D%5Cmid%5Cbold%7Bo%7D%29). Where _h_ is
the hypothesis and _o_ is the observed phrase. Applying Bayes's Theorem, and the fact that mnemonic
phrases are chosen uniformly at random*, we can make the following adjustments.

![eq2](https://latex.codecogs.com/svg.latex?P%28%5Cbold%7Bh%7D%5Cmid%5Cbold%7Bo%7D%29%20%3D%20%5Cfrac%7BP%28%5Cbold%7Bo%7D%5Cmid%5Cbold%7Bh%7D%29P%28%5Cbold%7Bh%7D%29%7D%7BP%28%5Cbold%7Bo%7D%29%7D%20%3D%20%5Cfrac%7BP%28%5Cbold%7Bo%7D%5Cmid%5Cbold%7Bh%7D%29P%28%5Cbold%7Bh%7D%29%7D%7B%5Csum_%7B%5Cbold%7B%5Chat%7Bh%7D%7D%5Cin%5Cbold%7BH%7D%7D%7BP%28%5Cbold%7Bo%7D%5Cmid%5Cbold%7B%5Chat%7Bh%7D%7D%29%7DP%28%5Cbold%7B%5Chat%7Bh%7D%7D%29%7D%3D%5Cfrac%7BP%28%5Cbold%7Bo%7D%5Cmid%5Cbold%7Bh%7D%29%7D%7B%5Csum_%7B%5Cbold%7B%5Chat%7Bh%7D%7D%5Cin%5Cbold%7BH%7D%7D%7BP%28%5Cbold%7Bo%7D%5Cmid%5Cbold%7B%5Chat%7Bh%7D%7D%29%7D%7D)

Where _H_ is the set of all hypothesis (e.g. all valid mnemonic phrases).

With this we can see that the only quantity we need to estimate is the probability of the observed
phrase, assuming that a our hypothesis is correct. Furthermore, because the denominator in the
equation above is invariant to the hypothesis, it can be ignored for an ordering rule.

This is a commonly used method in [spelling correction](https://norvig.com/spell-correct.html), and
is known as the ["noisy channel" model](https://www.aclweb.org/anthology/C90-2036.pdf). Intuitively
this models the process of writing down, and then recalling the mnemonic phrase as a "channel" with
a number of common errors, "noise", that are introduced along the way.

Examples of errors that may be introduced in this process:

* Misspelling individual words
* Misreading a word as another common word
* Swapping the ordering of words
* Illegible handwriting rendering one or more words difficult to read

Because one or more of these errors may occur in any given phrase, at any location, the space of
hypotheses grows exponentially with the number of errors. It is therefore impossible to generate the
entire list of guesses. Instead the application must generate the guesses incrementally.

#### Word and phrase level errors

In order to facilitate modeling, break the error processes into two distinct categories:

1. Errors that effect an individual word (e.g. mispelling)
2. Errors that effect the phrase, but not individual words (e.g. word swapping, or duplication)

Under this factorization, and assuming errors are applied independently, the equation above can be rewritten as:

![eq3](https://latex.codecogs.com/svg.latex?P%28%5Cbold%7Bo%7D%5Cmid%5Cbold%7Bh%7D%29%20%3D%20P_w%28%5Cbold%7Bo%7D%5Cmid%5Cbold%7Bh%27%7D%29P_p%28%5Cbold%7Bh%27%7D%5Cmid%5Cbold%7Bh%7D%29%3DP_p%28%5Cbold%7Bh%27%7D%7C%5Cbold%7Bh%7D%29%5Cprod_%7Bi%3D1%7D%5EN%7BP_w%28%5Cbold%7Bo_i%7D%5Cmid%5Cbold%7Bh_i%27%7D%29%7D).

Where _P\_p_ is the model for phrase-level errors and _P\_w_ is the model for word-level errors and
_N_ is the number of words in the phrase.

#### Ordering rule

Most important to generating an ordered list of guesses is to have an ordering rule that can used
used to answer the question of question hypothesis _a_ is more likely than hypothesis _b_. Here we
rewrite the equation above as:

![eq4](https://latex.codecogs.com/svg.latex?P%28%5Cbold%7Ba%7D%5Cmid%5Cbold%7Bo%7D%29%20%3E%20P%28%5Cbold%7Bb%7D%5Cmid%5Cbold%7Bo%7D%29%5CLeftrightarrow%20%5Clog%7BP_p%28%5Cbold%7Ba%27%7D%7C%5Cbold%7Ba%7D%29&plus;%5Csum_%7Bi%3D1%7D%5EN%7B%5Clog%20P_w%28%5Cbold%7Bo_i%7D%5Cmid%5Cbold%7Ba_i%27%7D%29%7D%7D%3C%5Clog%7BP_p%28%5Cbold%7Bb%27%7D%7C%5Cbold%7Bb%7D%29&plus;%5Csum_%7Bi%3D1%7D%5EN%7B%5Clog%20P_w%28%5Cbold%7Bo_i%7D%5Cmid%5Cbold%7Bb_i%27%7D%29%7D%7D)

Taking the log probability is a common transform in language modeling, and makes computation easier.
It also simplifies the overall expressions when the underlying distribution is exponential, as is
often the case here.

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
