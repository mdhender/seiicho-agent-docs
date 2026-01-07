# Seiicho Agent Documentation

This repository contains the **canonical specification** for the game **Seiicho**.

It is written primarily for **coding agents** and secondarily for human developers.  The goal is deterministic, unambiguous implementation of the game engine without requiring interpretation, inference, or follow-up questions.

If two independent implementations follow this specification, they should produce **identical results** given the same inputs.

---

## What This Repository Is

This repository defines *what the game is*, not how to implement it.

It contains:
- authoritative rules and definitions,
- deterministic semantics and resolution order,
- canonical pseudocode suitable for direct transcription by coding agents.

The specification is divided into three documents:

| Document         | Purpose                                               |
|------------------|-------------------------------------------------------|
| **RULEBOOK.md**  | Player-facing concepts and domain definitions         |
| **SEMANTICS.md** | Deterministic rules, invariants, and resolution order |
| **REFERENCE.md** | Executable-style pseudocode (authoritative)           |

Together, these form the complete Seiicho specification.

---

## What This Repository Is Not

This repository does **not** contain:

- engine code
- UI or UX design
- database schemas
- Go, JavaScript, or SQL implementations
- performance optimizations
- speculative or exploratory game design

Those belong in separate repositories or design discussions and may be promoted here only once decisions are finalized.

---

## Audience

This repository is intended for:

- coding agents (primary audience),
- engine developers implementing Seiicho,
- maintainers validating deterministic behavior,
- future contributors reviewing canonical rules.

The writing style intentionally favors explicitness and determinism over narrative flow.

---

## Determinism and Reproducibility

A core design goal of Seiicho is **deterministic replay**.

Accordingly:
- all randomness is explicitly modeled,
- PRNG usage is specified and constrained,
- generation and resolution order are normative,
- ambiguous behavior is treated as a specification bug.

---

## Versioning and Stability

The specification uses semantic-style versioning.  See **VERSIONING.md** for details.

In brief:
- PATCH versions clarify without changing behavior,
- MINOR versions add compatible rules,
- MAJOR versions introduce breaking changes.

Until version `1.0.0`, the specification is considered provisionally stable.

---

## Contributing

Contributions are welcome, but this repository follows a **strict specification workflow**.

Please read **CONTRIBUTING.md** before opening an issue or pull request.

In particular:
- ambiguity is considered a defect,
- determinism is mandatory,
- `REFERENCE.md` is highly stable,
- design exploration happens elsewhere.

---

## License and Attribution

Seiicho is inspired by *Empyrean Cluster*, which is owned by Jay Columbo and used with permission.

This repository contains **original specification text** and does not include proprietary materials.

---

## Status

This repository is under active development.

Expect:
- iterative refinement,
- tightening of rules,
- gradual promotion of sections to “Locked” status,
- and eventual stabilization toward version 1.0.0.

---

## Summary

If you are implementing Seiicho and find yourself asking:

> “What does the game intend here?”

then the specification is incomplete.

Please help improve it.
