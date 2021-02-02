## Blake2s

## Simple Summary

This document is a specification of the blake2s hash function and the blake2xs
hash functions for CIP20.

## Specification

Blake2s allows several configuration options. To fully support these, the
precompile accepts a fixed-length configuration block in addition to the
preimage. To support keyed hashing, the configuration may be accompanied by an
optional, variable-width key. The size of the key is specified within the
configuration, and is bounded at 32 bytes.

## Configuration Block

The configuration block allows the caller to modify the hash function to create
domain-specific hashes. It is explicitly committed to in the initialization
vector.

For clarity, within the configuration block, all multi-byte integers use
little-endian byte order unless explicitly stated otherwise.

### Blake2s

The input to the CIP20 interface for Blake2s is
`configuration || key || preimage` where `||` is the concatenation operator.
This will produce a prefix of between 32 and 64 bytes, depending on key length.

The configuration block is specified in the blake2s documentation. It is
exactly 32 bytes as follows:

```
type parameterBlock struct {
	DigestSize      byte   // 0
	KeyLength       byte   // 1
	fanout          byte   // 2
	depth           byte   // 3
	leafLength      uint32 // 4-7
	nodeOffset      uint48 // 8-13
	nodeDepth       byte   // 14
	innerLength     byte   // 15
	Salt            []byte // 16-23
	Personalization []byte // 24-31
}
```

For example, the default configuration (blake2s-256) as specified in the Blake2
documentation is
`0x2000010100000000000000000000000000000000000000000000000000000000`. It
represents blake2s operating in sequential mode with no key, salt, or
personalization, outputting a `0x20` (32) byte digest.

## Gas Accounting

Gas for Blake2s will be set equal to SHA256 costs: `60 + (12 * words)`. The
number of words is calculated without the configuration block (equivalent to
the total length of the input minus the length of the configuration). If the
input is less than the configuration size (32 bytes), it is invalid, and the
pricing function MUST return `INVALID_CIP20_INPUT_GAS`.

```python
EVM_WORD_SIZE: int = 32
BLAKE2_CONFIG_SIZE: int = 32

def price_word_metered_hash(
	base: int,
	per_word: int,
	input: bytes
) -> int:
    length_ceiling = len(input) + EVM_WORD_SIZE - 1
    words = length_ceiling // EVM_WORD_SIZE
    return base + words * per_word

def price_blake2s(input: bytes) -> int:
	if len(input) < BLAKE2_CONFIG_SIZE:
		return INVALID_CIP20_INPUT_GAS
	return price_word_metered_hash(
		60,
		12,
		input[BLAKE2_CONFIG_SIZE:]
	)
```

## Examples

- TODO

## References

- [Blake2 documentation](https://www.blake2.net/blake2.pdf)
- [celo-blockchain integration](https://github.com/celo-org/celo-blockchain/tree/prestwich/cip-0020)

Code snippets above are reproduced from the following blake2 libraries, which
are dedicated to the public domain:

- [Blake2s lib](https://github.com/dchest/blake2s)
- [Blake2xs lib](https://github.com/dchest/blake2xs)
