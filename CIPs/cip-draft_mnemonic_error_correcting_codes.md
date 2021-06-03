---
cip: <to be assigned>
title: Mnemonic phrases with error correcting codes
author: Victor Graf <@nategraf>
discussions-to: https://github.com/celo-org/celo-proposals/issues/225
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

A scheme for mnemonic phrases with integrated error correction codes as a subset of BIP-39, enabling
guaranteed recovery of mnemonic phrases with a limited number of arbitrary errors.

## Abstract

[BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic phrases are widely
used as the seed for account generation in the crypto ecosystem, and a pen-and-paper backup is the
most commonly recommended way to store these phrases to enable account recovery.

Adding error correcting codes to the mnemonic phrase would allow for correction of a limited number
of arbitrary errors. This CIP proposes a fully-compatible extension to BIP-39 in which some of the
random words are replaced with "error correcting" words to produce an instance of
[Reed-Solomon](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction) coding. It is
constructed as a subset of BIP-39, allowing CIP-# and plain BIP-39 phrases to be used
interchangeable in existing and new applications.

Complimentary to this proposal is CIP-$, which proposes a method for and framework for probabilistic
error correction of mnemonic phrases.

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
time. CIP-$ aims to provide a standard way to remove these errors automatically.

When information is lost from the phrase (e.g. some of the words are entirely missing), or when the
heuristic does not do appropriately model a particular error, heuristics based approaches may fail.
Proposed here is the introduction of error correcting codes, which can recover phases with a number
of arbitrary errors, covering many cases that model based approaches cannot.

## Specification

### Error correcting codes

Our aim in adding error correction codes to BIP-39 phrases is to produce a valid BIP-39 phrase,
which has a number of the random words replaced with valid "error correcting" words, containing
redundant information to the remaining words.

A mnemonic phrase generated in this scheme is interpreted as a
[Reed-Solomon](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction) codeword with
`N` 11-bit symbols (i.e. BIP-39 words), of which `K` symbols form the message (i.e. the underlying
entropy).

<!-- TODO(victor): Is this the best instantiating to use here? It is very classical, but not
  necessarily efficient or commonly implemented.-->
Specifically, the Reed-Solomon codec in this proposal, `RS(N, K)`, is defined to as a bijective map from a `K`
symbol message to an `N` symbol codeword. Symbols are members of the finite field `GF(2^11)`.
Messages are viewed as a vector of coefficients to an `K-1` degree polynomial, `m = (m_0, m_1, ...
m_K-1) => p(X) m_0 + m_1X + ... + m_K-1X^K-1`. Code words are the evaluation of this polynomial at
points `a_0, a_1, ..., a_N-1 = 0, 1, ..., N-1` in `GF(2^11)`.

Each word in the phrase is coded as a symbol equal its index in the BIP-39 word list for the
phrase's language. Codewords are serialized as BIP-39 phrases and MUST have a valid BIP-39 checksum
to be considered valid under this proposal.

`N` and `K` MAY be chosen by the user or application developer when generating a phrase. `N` MUST be
a multiple of 3 between 12 and 24 inclusive (i.e. It must be a valid number of words for BIP-39).
`K` MUST be a multiple of 3 between 12 and 24 inclusive, less than or equal to `N`. Note that when
`N = K`, no error correction is provided and the scheme is equivalent to BIP-39 phrase with `N`
words.

With a given selection of `N` and `K`, it will be possible to correct `N-K` symbols at known
locations (i.e. when the provided token is not a valid BIP-39 word, or is indicated as missing) and
up to `floor(N-K/2)` symbols at known locations (e.g. when word orderings are swapped, or replaced
with another valid BIP-39 word). This is a result of the
[Singleton bound](https://en.wikipedia.org/wiki/Singleton_bound).

| N  | K  | entropy (bits) | correctable errors (words) | correctable erasures (words) |
| -- | -- | -------------- | -------------------------- | ---------------------------- |
| 15 | 12 |            127 |                          1 |                            3 |
| 18 | 12 |            126 |                          3 |                            6 |
| 18 | 15 |            159 |                          1 |                            3 |
| 21 | 12 |            125 |                          4 |                            9 |
| 21 | 15 |            158 |                          3 |                            6 |
| 21 | 18 |            191 |                          1 |                            3 |
| 24 | 12 |            124 |                          6 |                           12 |
| 24 | 15 |            157 |                          4 |                            9 |
| 24 | 18 |            190 |                          3 |                            6 |
| 24 | 21 |            223 |                          1 |                            3 |

Note that the entropy available for a given `N` and `K` is slightly less (up to 4 bits) than a
comparable BIP-39 phrase with `K` words. This is because of the reduction in message space given the
requirement that the resulting phrase much have a valid BIP-39 checksum.

#### Generation

Generation of a compliant mnemonic phrase with error correction may be done as follows:

1. Generate `11 * K` bits of entropy, splitting it into a vector `E` of `K` 11-bit values.
2. Encode the message `E` using the Reed-Solomon codec, `RS(N, K)`, described above. The resulting
   codeword `C` is a vector of `N` 11-bit symbols.
3. Serialize the codeword `C` to a BIP-39 phrase by using each element as an index into the BIP-39
   word list of the phrase language.
4. Verify the checksum of the resulting BIP-39 phrase, [as specified in that
   standard](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki#generating-the-mnemonic).
5. If the checksum verifies, return the resulting phrase. If not, return to step 1.

Note that implementations MAY use an alternate generation procedure as long as:
1. It results in a valid BIP-39 mnemonic phrase.
2. If re-encoded using the `RS(N, K)` codec described above, the same phrase is constructed.

#### Validity and error correction

<!-- TODO(victor): Include some thoughts on how to handle different choices of K -->

Error correction of a mnemonic phrase may be accomplished as follows:

1. Derive the codeword representation `C` as the vector of indices into the BIP-39 word list.
   * Any words in the given phrase that are not valid BIP-39 words should be marked as erasures.
2. Attempt to decode the codeword `C` to get the message `E`.
   * If decoding fails, return an error.
3. Re-encode `E` to obtain `C'`, and serialize the codeword as a phrase by using each element as an
   index into the BIP-39 wordlist of the phrase language.
4. Check the BIP-39 checksum of the resulting phrase.
   * If the checksum is invalid, return an error.
5. Return the corrected phrase.

This error correction procedure SHOULD also be used to determine is a given phrase is a valid
mnemonic with error correction:

* If the result of error correction is the same phrase, then the given phrase contains no errors.
* If the result of error correction is a different phrase, then the given phrase contained at least
  one error, but was recovered successfully. 
  * Note that error correction will change at most `floor((N - K) / 2)` unknown positions.
* If error correction fails, then the phrase is either not a CIP-# phrase, or has too many errors to
  fix.

<!-- TODO(victor) Analyse the probability of this kind of failure. Its likely similar to the issue
of mistakenly identifying a plain BIP-39 phrase as a CIP-# phrase, which is described below. -->
Note that if the given phrase contains more than `floor((N - K) / 2)`, then there is a small
probability that error correction will report success and return a phrase different than the true
source phrase. Given the chance of mistaken "correction" of a phrase, applications SHOULD inform
the user before proceeding with a phrase altered by this error correction procedure.

#### Acceptance of plain BIP-39 phrases by CIP-# applications

Applications that implement this proposal SHOULD also accept plain BIP-39 phrases. A phrase SHOULD
be treated as a plain BIP-39 phrase if error correction fails and the given phrase has a valid
BIP-39 checksum. Although it is technically possible that a plain BIP-39 phrase may pass error
correction, it is very unlikely.

### Heuristic-based correction

WIP

Heuristic based approaches can correct a errors which are well-modeled by the "noisy channel". In
cases where the error is not well modeled, or reduce the information content of the phrase to a
significant extent, it may not be possible to recovery the original phrase.

## Rationale

### Probability of mistaken identification of a plain BIP-39 phrase as a CIP-# phrase

Described above, error correction and a checksum validation are used to determine if error
correction is available. It is techically possible, but unlikely, for this to mistake a plain BIP-39
phrase for a CIP-# phrase. Specifically, given a plain BIP-39 phrase of `N` words, `e` of which are
invalid (i.e. erasures), the probability that the phrase will pass error correction is equal to the
probability that:

* A "quorum" subset of symbols (i.e. points) are consistent with some `K-1` degree polynomial.
* And the re-encoded codeword contains a valid BIP-39 checksum.

<!-- TOOD(victor) Refactor the math in this section. It is _not_ easy to read -->

In this case, the quorum of points required is the number of points available minus the number of
acceptable unknown errors :`(N - e) - floor((N - K - e) / 2) = ceil((N + K - e) / 2)`. Given a
particular quorum, it is consistent with some `K-1` degree polynomial if and only if, taking any `K`
points, the remaining `ceil((N - K - e) / 2)` points are consistent with the polynomial interpolated
from those `K` points. When  the points are uniformly random and independent of each other the
probability that this is true is `y = 2 ^ (-11 * ceil((N - K - e) / 2))`. (11 because the points are
in `GF(2^11)`)

In a given phrase, there are `(N - e) choose ceil((N + K - e) / 2)` quorums. When the words of the
phrase are chosen independently and at random, The overall chance of decoding a plain BIP-39 phrase
is `1 - (1 - y) ^ ((N - e) choose ceil((N + K - e) / 2))`, with `y` defined above.

If the phrase decodes, it will then be re-encoded and the checksum will be verified. Assuming the
phrase was modified by the re-encoding procedure, the probability that the checksum will verify is
`2^(-N/3)`. Overall the chance of mistaking a plain BIP-39 phrase for a CIP-# phrase is `(1 - (1 -
y) ^ ((N - e) choose ceil((N + K - e) / 2))) * (2^(-N/3))`.

Given a correct 15-word plain BIP-39 phrase, the probability of mistaking it for a CIP-# phrase is
roughly 1 in 10 million. If the 15-word phrase has an invalid word (i.e. an erasure), it will be
mistaken for a CIP-# phrase with probability roughly 2 in 10,000. For all larger values of `N` and
`K`, and with any number of erasures, the probability is lower. In most cases this is negligible,
but does reinforce the recommendation that the application inform users before proceeding with a
phrase altered by error correction.

## Backwards Compatibility

BIP-39 implementations will accept mnemonic phrases generated with error correction under this
proposal, as described above. Clients implementing this proposal are also able to support plain
BIP-39 phrases, as described above.

## Test Cases

WIP

## Implementation

* A proof-of-concept implementation of phrases with error-correcting codes: [nategraf/cip39](https://github.com/nategraf/cip39)
  * Note, the Reed-Solomon implementation used in this PoC is currently not compliant with this standard.

## Security Considerations

* As with any proposal handling the source material for key generation, mistakes in implementation
  can lead to loss of funds or leakage of key material.
* Users understanding BIP-39 security levels may misinterpret an `N` word CIP-# phrase to have a
  higher security level than it actually does. (I.e. Assuming a 24-word CIP-# phrase has 256 bits
  of entropy, when it may have between 124 and 223 bits, depending on selection of `K`)

## License
This work is licensed under the Apache License, Version 2.0.
