## Blake2s

## Simple Summary

This document is a specification of the blake2s hash function and the blake2xs
hash functions for CIP20.

## Specification

Blake2s allows several configuration options. To fully support these, the 
precompile accepts a fixed-length configuration block in addition to the 
preimage. To support keyed hashing, the configuration there is an optional, 
variable-width key. The size of the key is specified within the configuration 
block. The input to the CIP20 interface to these functions is 
`configuration || key || preimage` where `||` is the concatenation operator.

### Configuration Block

For clarity, within the configuration block, all multi-byte integers use 
little-endian byte order.

**blake2s** 

The configuration block is specified in the blake2s documentation. It is
exactly 32 bytes as follows:

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

For example, the default configuration (blake2s-256) as specified in the Blake2 
documentation is 
`0x2000010100000000000000000000000000000000000000000000000000000000`. It 
represents blake2s operating in sequential mode with no key, salt, or 
personalization, outputting a `0x20` (32) byte digest.

**blake2xs** 

We modify the blake2s block slightly to allow longer digest sizes. Our blake2xs 
configuration block is 33 bytes as follows:

```
type parameterBlock struct {
	DigestSize      uint16 // 0
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

## Examples

**blake2x**

- TODO

**blake2xs**

- `000120010100000000000000000000000000000000000000000000000000000000 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 00`
    - 256 byte output - `0x0001`
    - 14 byte key - `0x20`
    - default tree options
    - no salt or personalization
    - key -- 
    `000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f`
    - preimage -- `00`


## References

- [Blake2 documentation](https://www.blake2.net/blake2.pdf)
- [Blake2s lib](https://github.com/dchest/blake2s)
- [Blake2xs lib](https://github.com/dchest/blake2xs)
- [celo-blockchain integration](https://github.com/celo-org/celo-blockchain/tree/prestwich/cip-0020)