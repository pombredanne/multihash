# multihash

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](http://ipn.io)
[![](https://img.shields.io/badge/project-multiformats-blue.svg?style=flat-square)](http://github.com/multiformats/multiformats)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](http://webchat.freenode.net/?channels=%23ipfs)

> Self identifying hashes

Multihash is a protocol for differentiating outputs from various well-established cryptographic hash functions, addressing size + encoding considerations.

It is useful to write applications that future-proof their use of hashes, and allow multiple hash functions to coexist. See https://github.com/jbenet/random-ideas/issues/1 for a longer discussion.

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Example](#example)
- [Format](#format)
- [Implementations:](#implementations)
- [Table for Multihash v1.0.0-RC (semver)](#table-for-multihash-v100-rc-semver)
  - [Other tables](#other-tables)
- [Disclaimers](#disclaimers)
- [Visual Examples](#visual-examples)
- [Maintainers](#maintainers)
- [Contribute](#contribute)
- [License](#license)

## Example

Outputs of `<encoding>.encode(multihash(<digest>, <function>))`:

```
# sha1 - 0x11 - sha1("multihash")
111488c2f11fb2ce392acb5b2986e640211c4690073e # sha1 in hex
CEKIRQXRD6ZM4OJKZNNSTBXGIAQRYRUQA47A==== # sha1 in base32
5dsgvJGnvAfiR3K6HCBc4hcokSfmjj # sha1 in base58
ERSIwvEfss45KstbKYbmQCEcRpAHPg== # sha1 in base64

# sha2-256 0x12 - sha2-256("multihash")
12209cbc07c3f991725836a3aa2a581ca2029198aa420b9d99bc0e131d9f3e2cbe47 # sha2-256 in hex
CIQJZPAHYP4ZC4SYG2R2UKSYDSRAFEMYVJBAXHMZXQHBGHM7HYWL4RY= # sha256 in base32
QmYtUc4iTCbbfVSDNKvtQqrfyezPPnFvE33wFmutw9PBBk # sha256 in base58
EiCcvAfD+ZFyWDajqipYHKICkZiqQgudmbwOEx2fPiy+Rw== # sha256 in base64
```

## Format

```
<varint hash function code><varint digest size in bytes><hash function output>
```

Binary example (only 4 bytes for simplicity):

```
fn code  dig size hash digest
-------- -------- ------------------------------------
00010001 00000100 101101100 11111000 01011100 10110101
sha1     4 bytes  4 byte sha1 digest
```

> Why have digest size as a separate number?

Because otherwise you end up with a function code really meaning "function-and-digest-size-code". Makes using custom digest sizes annoying, and is less flexible.

> Why isn't the size first?

Because aesthetically I prefer the code first. You already have to write your stream parsing code to understand that a single byte already means "a length in bytes more to skip". Reversing these doesn't buy you much.

> Why varints?

So that we have no limitation on functions or lengths. Implementation note: you do not need to implement varints until the standard multihash table has more than 127 functions.

> What kind of varints?

An Most Significant Bit unsigned varint, as defined by the [multiformats/unsigned-varint](https://github.com/multiformats/unsigned-varint).

> Don't we have to agree on a table of functions?

Yes, but we already have to agree on functions, so this is not hard. The table even leaves some room for custom function codes.

## Implementations:

- [go-multihash](//github.com/jbenet/go-multihash)
- [node-multihash](//github.com/jbenet/node-multihash)
- [clj-multihash](//github.com/greglook/clj-multihash)
- rust-multihash
  - [by @dignifiedquire](//github.com/dignifiedquire/rust-multihash)
  - [by @google](//github.com/google/rust-multihash)
- [haskell-multihash](//github.com/LukeHoersten/multihash)
- [python-multihash](//github.com/tehmaze/python-multihash)
- [elixir-multihash](//github.com/zabirauf/ex_multihash), [elixir-multihashing](//github.com/candeira/ex_multihashing)
- [swift-multihash](//github.com/NeoTeo/SwiftMultihash)
- [ruby-multihash](//github.com/neocities/ruby-multihash)
- [MultiHash.Net](//github.com/MCGPPeters/MultiHash.Net)
- [scala-multihash](//github.com/mediachain/scala-multihash)
- [php-multihash](//github.com/Fil/php-multihash)

## Table for Multihash v1.0.0-RC (semver)

The current multihash table is [here](hashtable.csv):

```
code name
0x00 identity
0x11 sha1
0x12 sha2-256
0x13 sha2-512
0x14 sha3-512
0x15 sha3-384
0x16 sha3-256
0x17 sha3-224
0x18 shake-128
0x19 shake-256
0x40 blake2b
0x41 blake2s
# 0x00-0x0f reserved for application specific functions
# 0x10-0x3f reserved for SHA standard functions
# 0x14 formerly had the name "sha3", now deprecated
```


### Other tables

Cannot find a good standard on this. Found some _different_ IANA ones:

- https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml#tls-parameters-18
- http://tools.ietf.org/html/rfc6920#section-9.4

They disagree. :(

## Disclaimers

Warning: **obviously multihash values bias the first two bytes**. Do not expect them to be uniformly distributed. The entropy size is `len(multihash) - 2`. Skip the first two bytes when using them with bloom filters, etc. Why not _ap_pend instead of _pre_pend? Because when reading a stream of hashes, you can know the length of the whole value, and allocate the right amount of memory, skip it, or discard it.

## Visual Examples

These are visual aids that help tell the story of why Multihash matters.

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.001.jpg)

#### Consider these 4 different hashes of same input

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.002.jpg)

#### Same length: 256 bits

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.003.jpg)

#### Different hash functions

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.004.jpg)

#### Idea: self-describe the values to distinguish

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.005.jpg)

#### Multihash: fn code + length prefix

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.006.jpg)

#### Multihash: a pretty good multiformat

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.007.jpg)

#### Multihash: has a bunch of implementations already

![](https://raw.githubusercontent.com/multiformats/multiformats/eb22cd807db692877a9094b5bfb4d2997fd0278a/img/multihash.008.jpg)

## Maintainers

Captain: [@jbenet](https://github.com/jbenet).

## Contribute

Contributions welcome. Please check out [the issues](https://github.com/multiformats/multihash/issues).

Check out our [contributing document](https://github.com/multiformats/multiformats/blob/master/contributing.md) for more information on how we work, and about contributing in general. Please be aware that all interactions related to multiformats are subject to the IPFS [Code of Conduct](https://github.com/ipfs/community/blob/master/code-of-conduct.md).

Small note: If editing the Readme, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License

[MIT](LICENSE)
