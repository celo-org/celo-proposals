---
cip: <to be assigned>
title: Error correction for BIP-39 mnemonic phrases
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
Proposed in this CIP are two methods for providing error correction to mnemonic phrases in the
context of account recovery:

* In context where a new phrase is being generated, a backward-compatible extension to BIP-39 is
  proposed to add error correction information for guaranteed correction of a fixed number of error
  words.
* Additionally, a heuristic-based approach is discussed, including an edit-distance based
  instantiation and theoretical framework for extension to include improved rules.

## Abstract

<!-- TODO(victor) Possibly a bit long for an abstract -->
[BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic phrases are widely
used as the seed for account generation in the crypto ecosystem, and a pen-and-paper backup is the
most commonly recommended way to store these phrases to enable account recovery.

In the case that the user has a copy of their phrase with errors, it is possible to recovery the
original phrase as long as the errors are relatively small. This CIP introduces a proposed standard
and framework for correcting errors based on the "noisy channel" model which is commonly used in
[spelling correction programs](https://norvig.com/spell-correct.html) to generate suggested
correction. An instantiation of this standard is described using Levenshtein distance, and assuming
a number of independent random errors are introduced to the mnemonic phrase.

Another approach is to add error correcting codes to the mnemonic phrase, which would allow for
correction of a limited number of arbitrary errors. This CIP proposes a fully-compatible extension
to BIP-39 in which some of the random words are replaced with "error correcting" words to produce an
instance of [Reed-Solomon](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction)
coding. It is constructed to be interchangeable in applications that support BIP-39, whether or not
they support this proposal.

Note that neither of these ideas are entirely new, and have been discussed at various points in the
crypto community. This CIP aims to provide a standard and implementation to put these ideas into
production.

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
time. This CIP aims to provide a standard way to remove these errors automatically.

When information is lost from the phrase (e.g. some of the words are entirely missing), or when the
heuristic does not do appropriately model a particular error, the heuristic based approach may fail.
The introduction of error correcting codes can allow for recovering phases with a number of
arbitrary errors, covering many cases that heuristic based approaches cannot.

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

<!-- TODO(victor): Should the requirement for K to be a multiple of 3 be relaxed? -->
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
2. If re-encoded using the `RS(N, K)` codec described above, the same phrase is constructed,

Also note that although `11 * K` bits of entropy are used as input, the resulting phrase has `11 * K - N/3`
bits of entropy, as the requirement of a valid BIP-39 checksum reduces the space of valid phrases by
a factor of `2^{N/3}`.

#### Error correction

<!-- TODO: Include some thoughts on how to handle different choices of K -->

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

#### Acceptance of BIP-39 phrases by applications

### Heuristic-based correction

WIP

Heuristic based approaches can correct a errors which are well-modeled by the "noisy channel". In
cases where the error is not well modeled, or reduce the information content of the phrase to a
significant extent, it may not be possible to recovery the original phrase.

## Rationale
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.

## Backwards Compatibility

BIP-39 implementations will accept mnemonic phrases generated with error correction under this
proposal, as described above. Compatibility of clients implementing this proposal with existing
BIP-39 phrases is described above.

Heuristic-based error correction described above does not introduce any concerns with backwards
compatibility. Implementations of this model also need not be deterministic, or otherwise return
results consistent with each other.

## Test Cases

WIP

## Implementation

WIP

* A proof-of-concept implementation of phrases with error-correcting codes: [nategraf/cip39](https://github.com/nategraf/cip39)
* An implementation of heuristic-based mnemonic phrase correction is implemented in [celo-org/celo-monorepo/pull/8034](https://github.com/celo-org/celo-monorepo/pull/8034)

## Security Considerations

WIP

## License
This work is licensed under the Apache License, Version 2.0.
