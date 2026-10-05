# Deterministic Emoji Mnemonic Encoding (DEME)

## Guan
Internet-Draft: draft-guan-emoji-mnemonic-00
Intended status: Informational
Expires: November 13, 2026

---

## Abstract

This document specifies the Deterministic Emoji Mnemonic Encoding (DEME), a method for generating memorable, entropy-preserving sequences of emoji symbols from a cryptographic seed. DEME produces reproducible mnemonic phrases suitable for human memorization while preserving the full entropy of the underlying secret. The encoding is deterministic, stateless, and requires no persistent storage beyond the seed itself.

## 1. Introduction

Human-friendly representations of cryptographic material are essential for systems where users must memorize or transcribe secrets. BIP-39 (Bitcoin Improvement Proposal 39) addresses this need with word-based mnemonics [BIP39]. However, no standard exists for emoji-based mnemonics, despite emoji's:

- Global recognizability across languages
- Visual distinctiveness reducing transcription errors
- Narrative potential supporting memory palace techniques [YATES]

Existing emoji mnemonic implementations suffer from entropy loss due to flawed indexing strategies (e.g., arithmetic halving). DEME eliminates this loss through bit-precise modular indexing.

### 1.1 Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

### 1.2 Design Goals

1. **Entropy preservation** — no information loss during encoding
2. **Determinism** — identical inputs always produce identical output
3. **Statelessness** — no storage required beyond the seed
4. **Visual distinctiveness** — symbols selected for easy differentiation
5. **Cross-platform compatibility** — single Unicode codepoint symbols only

## 2. Terminology

- **Seed** — The high-entropy secret from which the mnemonic is derived
- **Symbol Set** — A curated, ordered list of emoji characters
- **Encoding Function** — The deterministic algorithm mapping seed → symbol sequence
- **Entropy Density** — Effective bits of entropy per symbol (log2 of symbol set size)

## 3. Symbol Set Definitions

DEME defines four canonical symbol set sizes. Implementations MUST support at least Emoji-256 and Emoji-2048.

### 3.1 Symbol Set Requirements

All symbols in a DEME symbol set MUST:

1. Consist of exactly one Unicode codepoint (no ZWJ sequences)
2. Have no skin-tone modifier variants (Fitzpatrick modifiers excluded)
3. Be displayable on major platforms (iOS, Android, Windows, macOS, Linux)
4. Be visually distinct from other symbols in the same set

Symbols in a DEME symbol set MUST NOT include:

1. Regional indicator symbols (flag components)
2. Keycap base characters (U+0023, U+002A, U+0030–U+0039)
3. Combining marks or variation selectors
4. Symbols likely to be rendered identically on common platforms

### 3.2 Canonical Symbol Sets

| Identifier | Size | Entropy/Symbol | Log2 Bits |
|------------|------|----------------|-----------|
| Emoji-256  | 256  | 8.0 bits       | 8         |
| Emoji-512  | 512  | 9.0 bits       | 9         |
| Emoji-1024 | 1024 | 10.0 bits      | 10        |
| Emoji-2048 | 2048 | 11.0 bits      | 11        |

The total entropy of a DEME phrase equals:

    H_total = count × log2(set_size)

Where `count` is the number of symbols in the phrase.

### 3.3 Category Organization

To support narrative memory techniques, symbol sets SHOULD be organized into categories with contiguous index ranges:

- Nature (sky, water, land elements)
- Creatures (animals, insects, birds)
- Objects (tools, food, vehicles)
- Places (buildings, landmarks)
- Abstract (shapes, signs, concepts)

This organization is optional for encoding correctness but RECOMMENDED for human memorability.

## 4. Encoding Algorithm

The DEME encoding function maps a seed and parameters to an ordered sequence of emoji symbols.

### 4.1 Inputs

- `seed` (REQUIRED) — High-entropy secret, arbitrary length
- `set_identifier` (REQUIRED) — One of: "emoji-256", "emoji-512", "emoji-1024", "emoji-2048"
- `count` (REQUIRED) — Number of symbols to generate (positive integer)
- `salt` (OPTIONAL) — Additional entropy source, arbitrary string. Default: empty string

### 4.2 Hash Generation

    input = salt || "\0" || seed || "\0" || set_identifier || "\0" || "emoji"
    digest = SHA3-512(input)

Where:
- `||` denotes concatenation
- `\0` is a single null byte (U+0000) acting as domain separator

The input encoding MUST be UTF-8.

### 4.3 Symbol Extraction

Given `digest` (64 bytes = 512 bits) and `set_size`:

    required_bits = count × log2(set_size)

If `required_bits > 512`, additional hash iterations are performed:

    digest_i = SHA3-512(digest_{i-1} || "\0" || i)

Where `i` increments from 1 until sufficient bits are collected.

Bits are read from the hash output as a continuous big-endian bit stream.

For each symbol position `k` (0-indexed):

    bit_offset = k × log2(set_size)
    chunk = bits[bit_offset : bit_offset + log2(set_size)]
    index = integer_value(chunk) mod set_size
    symbol = SymbolSet[index]

The `mod` operation ensures uniform distribution across the symbol set.

### 4.4 Example

    seed = "my-secret-master-key"
    set_identifier = "emoji-2048"
    count = 12
    salt = ""

    input = "\0my-secret-master-key\0emoji-2048\0emoji"
    digest = SHA3-512(input)
    
    # For each of 12 positions, extract 11 bits
    # index = 11-bit value mod 2048
    # Look up index in Emoji-2048 symbol set

Result (illustrative): 🦊 🌊 🏔️ 📚 🔥 🌙 ⚔️ 🍃 🐋 🔮 🏛️  🌸

## 5. Deterministic Generation Guarantee

For all valid inputs, DEME guarantees:

    encode(seed, set, count, salt) == encode(seed, set, count, salt)

Across all conforming implementations, on all platforms, at all times.

This property enables "brain wallet" usage: a user who remembers their seed can regenerate their exact emoji mnemonic on any device without stored state.

## 6. Entropy Analysis

### 6.1 Effective Entropy

| Set + Count | Total Bits | Crack Time (10^12 guesses/sec) |
|-------------|------------|-------------------------------|
| Emoji-256 × 12  | 96  | ~1.5 days          |
| Emoji-512 × 12  | 108 | ~1.7 years         |
| Emoji-1024 × 12 | 120 | ~6.9 centuries     |
| Emoji-2048 × 12 | 132 | ~28,000 centuries  |
| Emoji-2048 × 24 | 264 | thermodynamically infeasible |

### 6.2 Comparison with BIP-39

BIP-39 with a 2048-word list provides 11 bits per word. DEME Emoji-2048 provides equivalent entropy density with the benefit of global visual recognizability.

| System | 12 units | 24 units |
|--------|----------|----------|
| BIP-39 (2048 words) | 132 bits | 264 bits |
| DEME Emoji-2048 | 132 bits | 264 bits |

## 7. Security Considerations

### 7.1 Seed Entropy

DEME is a representation scheme, not an entropy source. The security of a DEME mnemonic depends entirely on the entropy of the seed. A low-entropy seed produces a memorable but easily guessable mnemonic.

Implementations MUST warn users when seed entropy falls below 80 bits.

### 7.2 Salt Usage

An optional salt adds defense against precomputation attacks (rainbow tables) at the cost of requiring two memorized secrets (seed + salt) for regeneration. This tradeoff is application-dependent.

### 7.3 Symbol Rendering

Different platforms render emoji differently. A phrase validated on one device may display differently on another. Implementations SHOULD provide hex dump or codepoint listings alongside rendered emoji for verification.

### 7.4 Transcription Risk

Emoji are not human-transcribable in the same way as alphanumeric characters. DEME is optimized for memory and digital copy-paste, not manual transcription.

## 8. IANA Considerations

This document makes no requests of IANA. Future versions may register symbol set identifiers.

## 9. References

### 9.1 Normative References

- [RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
- [SHA3] NIST FIPS 202, "SHA-3 Standard: Permutation-Based Hash and Extendable-Output Functions", August 2015.
- [UNICODE] The Unicode Consortium, "The Unicode Standard, Version 15.0", September 2022.

### 9.2 Informative References

- [BIP39] Palatinus, M. and Rusnak, P., "BIP-39: Mnemonic code for generating deterministic keys", Bitcoin Improvement Proposal, 2013.
- [RFC1321] Rivest, R., "The MD5 Message-Digest Algorithm", April 1992.
- [YATES] Yates, Frances A., "The Art of Memory", 1966. (narrative memory techniques)

## Author's Address

Guan
Email: just.guan@proton.me
GitHub: https://github.com/sagelabs-dev

With acknowledgment to Freeman King, who posed the challenge that this specification answers.

---

## Appendix A: Reference Implementation

Reference implementation available in TypeScript:
https://github.com/sagelabs-dev/passgen/blob/main/passgen.js (function generateEmojiPhrase)

The reference implementation MUST be considered authoritative for any ambiguity in this specification.
