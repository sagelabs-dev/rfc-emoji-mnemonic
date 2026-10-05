# Contributing to DEME

This is a living specification. Contributions welcome.

## What We're Looking For

- Security analysis of the encoding algorithm
- Symbol set curation (visually distinct, single-codepoint emoji)
- Implementation in other languages (reference is TypeScript)
- Real-world usage reports (memorability, UX, platform compatibility)

## What We're NOT Looking For

- Feature creep (DEME is intentionally minimal)
- Breaking changes to the canonical algorithm without version negotiation
- Complexity for complexity's sake

## Process

1. Open an issue describing the problem or proposal
2. If accepted, open a PR against `main` (feature branch discipline not enforced for small edits)
3. For substantial changes: branch → commit → PR → review → merge

The reference implementation in [passgen](https://github.com/sagelabs-dev/passgen) is authoritative for ambiguities.
