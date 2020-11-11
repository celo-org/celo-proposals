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

### Blake2Xs

Blake2Xs produces a variable-length digest. It is designed to allow successive
queries with an upper bound on total bytes queried. The upper bound is
explicitly commited to within the hash function IV. To support this mode of
operation without introducing state to the precompile, we allow the caller to
specify both an `xofLength` (the upper bound) and an `outputBytes` (the number
of bytes actually queried). If `outputBytes` is set to `uint32(0)`, it will be
interpreted as equal to xofLength.

**Note**: We define `outputBytes` as a big-endian 16-bit number, encoded as
exactly 2 bytes. This restricts the output to around 64 KiB.

The input to the CIP20 interface for Blake2Xs is
`configuration || key || outputBytes || preimage` where `||` is the
concatenation operator. This creates a prefix of between 34 and 66 bytes
(depending on the length of the key).

Blake2Xs modifies the Blake2s config block above to specify the XOF digest
length as follows:

```
type parameterBlock struct {
	DigestSize      byte   // 0
	KeyLength       byte   // 1
	fanout          byte   // 2
	depth           byte   // 3
	leafLength      uint32 // 4-7
	nodeOffset      uint32 // 8-11
	xofLength       uint16 // 12-13
	nodeDepth       byte   // 14
	innerLength     byte   // 15
	Salt            []byte // 16-23
	Personalization []byte // 24-31
}
```

Compared to Blake2s, the `nodeOffset` block is shortened, and a new 16-bit
`xofLength` field specifies the desired output length of the XOF.

## Gas Accounting

- TODO

## Examples

**blake2x**

- TODO

**blake2xs**

- 20200101 00000000 00000000 1400 0000 0000000000000000 0000000000000000 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 0010 00
	- Blake2s options:
		- 32-byte key length
		- XOF length set to 20 bytes (`0x1400`)
    	- key -
    `0x000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f`
	- requesting the first 16 bytes - `0x0010`
	- preimage -- `0x00`


## References

- [Blake2 documentation](https://www.blake2.net/blake2.pdf)
- [celo-blockchain integration](https://github.com/celo-org/celo-blockchain/tree/prestwich/cip-0020)

Code snippets above are reproduced from the following blake2 libraries, which
are dedicated to the public domain:
- [Blake2s lib](https://github.com/dchest/blake2s)
- [Blake2xs lib](https://github.com/dchest/blake2xs)