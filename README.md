# Deterministic Emoji Mnemonic Encoding

> A living specification for generating memorable, entropy-preserving emoji phrases from cryptographic seeds.

## Status

**Current Phase:** Draft 00 — Specification Outline

This document follows RFC-style conventions (section numbering, MUST/SHOULD/MAY terminology, security considerations) but is **self-published as a living specification**. An IETF Independent Submission may follow if community traction warrants institutional permanence.

## Quick Links

- [Draft 00 Specification](docs/draft-guan-emoji-mnemonic-00.md)
- [Reference Implementation](https://github.com/guan-tends/passgen) — `generate_emoji_phrase()` in `@guan-tends/passgen`

## Authorship

- **Primary Author:** Guan — architecture, bit-precise encoding, symbol set design, entropy calculations
- **Acknowledged Catalyst:** Freeman King — posed the challenge that exposed index-halving entropy loss, validated novel approach

## Symbol Sets

| Set Name | Size | Bits/Symbol | 12-Symbol Entropy |
|----------|------|-------------|-------------------|
| Emoji-256 | 256 | 8.0 | 96 bits |
| Emoji-512 | 512 | 9.0 | 108 bits |
| Emoji-1024 | 1024 | 10.0 | 120 bits |
| Emoji-2048 | 2048 | 11.0 | 132 bits |

## Design Principles

1. **No entropy loss** — bit-precise indexing via modular arithmetic, not rounded/ halved indices
2. **Deterministic** — same master + same parameters → same phrase, every time, on every device
3. **Visually distinct** — curated sets exclude ZWJ sequences, skin-tone variants, and flags
4. **Narrative-friendly** — category-organized symbols for memory palace construction
5. **Honest entropy** — documented bit counts, no hidden loss, no security theater

## Contributing

This is a living specification. Issues and PRs welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT — see [LICENSE](LICENSE)

---

Crafted with ❤️ by [Sage Labs](https://sagelabs.dev)
