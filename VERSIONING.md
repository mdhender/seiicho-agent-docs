# Specification Versioning

This document describes how versions are assigned and advanced for the
*Seiicho* specification contained in this repository.

The goal of versioning is to provide **clear stability guarantees** to both
human developers and coding agents implementing the specification.

---

## Version Format

The specification uses **semantic-style versioning**:

```

MAJOR.MINOR.PATCH

```

Example:
```

0.2.1

```

Version numbers apply to the **specification as a whole**, not to individual files.

---

## Version Meanings

### MAJOR

The MAJOR version increments when:

- The specification becomes incompatible with earlier versions
- Core mechanics are replaced or removed
- Determinism guarantees change
- Previously locked rules are intentionally broken

A MAJOR version bump signals:

> Existing implementations are no longer guaranteed to be correct.

---

### MINOR

The MINOR version increments when:

- New mechanics are added in a backward-compatible way
- Previously unspecified behavior is formally defined
- New sections are added to RULEBOOK, SEMANTICS, or REFERENCE
- Existing rules are extended without breaking earlier behavior

A MINOR version bump signals:

> Existing implementations may still work, but can be improved or extended.

---

### PATCH

The PATCH version increments when:

- Ambiguity is clarified
- Errors or contradictions are fixed
- Wording is tightened without changing behavior
- Pseudocode is corrected to match intended behavior
- Missing constraints or assertions are added

A PATCH version bump signals:

> No semantic changes; behavior remains identical.

---

## Pre-1.0 Versioning Policy

Until version **1.0.0**, the specification is considered **provisionally stable**.

This means:

- MAJOR version will remain `0`
- Breaking changes may occur
- Such changes will still be clearly documented and versioned

Example progression:
```

0.1.0 → 0.2.0 → 0.2.1 → 0.3.0

```

Version `1.0.0` indicates the specification is considered stable enough
for long-term engine implementations.

---

## Document Stability Levels

Each document has an expected stability level:

| Document      | Stability Expectation |
|---------------|-----------------------|
| RULEBOOK.md   | Flexible              |
| SEMANTICS.md  | Carefully managed     |
| REFERENCE.md  | Highly stable          |

Breaking changes in `REFERENCE.md` almost always require a **MINOR or MAJOR**
version increment.

---

## Section Locking and Versioning

Sections may be marked as:

- **Draft**
- **Candidate**
- **Locked**

### Draft
- Free to change
- May introduce breaking behavior
- Does not require version increment by itself

### Candidate
- Expected to stabilize soon
- Changes should be additive or clarifying
- Breaking changes require MINOR increment

### Locked
- Considered normative and authoritative
- Changes must be:
  - bug fixes, or
  - clarifications only
- Breaking changes require MAJOR increment

---

## Tags and Releases

Specification versions are identified by **git tags**:

```

spec-v0.1.0
spec-v0.2.0

```

Each tag should correspond to:

- a clean repository state
- a documented version number
- a brief release note summarizing changes

---

## Backward Compatibility Expectations

When possible:

- Later versions should document how behavior differs from earlier versions
- Migration notes may be included for significant changes
- Reference pseudocode should reflect only the current version

Backward compatibility is a goal, not a guarantee—especially pre-1.0.

---

## How to Propose Version Changes

Version changes should be explicit:

- Include the proposed new version number
- Identify which document(s) justify the change
- State whether the change is PATCH, MINOR, or MAJOR

Example commit message:
```

spec: bump to v0.3.0 (formalize planet distribution rules)

```

---

## Final Authority

The repository owner determines:

- when versions advance
- how sections are locked
- when the specification reaches 1.0.0

This ensures long-term coherence and stability.

---

## Summary

Version numbers are a **communication tool**, not just bookkeeping.

They exist to answer one question clearly:

> Can an implementation written for version X still be trusted?

If the answer changes, the version must change.
