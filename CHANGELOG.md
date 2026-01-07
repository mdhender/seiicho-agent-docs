# Changelog

All notable changes to the Seiicho specification are documented in this file.

The format is inspired by *Keep a Changelog* and follows the versioning rules described in **VERSIONING.md**.

---

## [Unreleased]

### Added
- Initial project charter
- CONTRIBUTING workflow for agent-facing specs
- VERSIONING policy

### Changed
- N/A

### Fixed
- N/A

---

## [0.1.0] — Initial Specification Draft

### Added
- Initial RULEBOOK structure
- Deterministic cluster generation rules
- System, star, orbit, and planet identifier formats
- PRNG determinism requirements
- Canonical orbit index ↔ letter mapping
- Planet distribution rules across stars and orbits
- First draft of REFERENCE.md with executable-style pseudocode

### Notes
- This release establishes the foundational structure of the specification.
- All rules are considered **Draft** or **Candidate**.
- Breaking changes are expected in subsequent MINOR versions.

---

## Versioning Notes

- PATCH releases clarify or correct without changing behavior.
- MINOR releases may add or formalize rules.
- MAJOR releases indicate breaking changes.

See **VERSIONING.md** for full details.

---

## How to Update This Changelog

When preparing a release:
1. Move relevant entries from **[Unreleased]** into a new version section.
2. Add the release date.
3. Summarize intent, not just diffs.

Example version header:
```

## [0.2.0] — Formalize Planet Generation

```
