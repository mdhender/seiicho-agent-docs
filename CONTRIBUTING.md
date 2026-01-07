# Contributing to seiicho-agent-docs

Thank you for your interest in contributing.

This repository contains the **canonical specification** for the game *Seiicho*.  It is written primarily for **coding agents** and secondarily for human readers.

Because this is a specification (not an implementation), the contribution workflow is intentionally strict.

---

## Repository Structure

The specification is divided into three documents, each with a different stability level:

- **RULEBOOK.md**  
  Player-facing concepts and definitions.  
  More flexible, but still normative.

- **SEMANTICS.md**  
  Deterministic rules, invariants, and resolution order.  
  Changes must be deliberate and justified.

- **REFERENCE.md**  
  Canonical, executable-style pseudocode.  
  This is the *most stable* document.

Contributors must respect the intent and stability of each layer.

---

## Guiding Principles

All contributions must adhere to the following principles:

- Determinism over cleverness
- Explicit rules over implied behavior
- Canonical wording over illustrative prose
- Reproducibility over flexibility

If a rule can be interpreted in more than one way, it is considered **incorrect**.

---

## What This Repository Is *Not*

Please do **not** submit contributions that focus on:

- UI or UX
- Database schemas
- Go, JavaScript, SQL, or other implementation details
- Performance optimizations
- Speculative or exploratory game design

Design exploration should occur outside this repository and be promoted here only once decisions are finalized.

---

## Contribution Workflow

### 1. Make a Focused Change

Each contribution should address **one of the following**:

- Clarifying ambiguous language
- Fixing a contradiction or omission
- Tightening determinism
- Improving agent-readability
- Correcting errors in pseudocode
- Adding missing canonical rules

Avoid bundling unrelated changes in a single commit.

---

### 2. Use Clear Commit Messages

Commit messages should be concise and descriptive, for example:

- `spec: clarify orbit index to letter mapping`
- `reference: fix planet distribution edge case`
- `semantics: define generation order explicitly`

Avoid vague messages such as “updates” or “cleanup”.

---

### 3. Respect Section Stability

If a section is marked:

- **Draft**  
  — free to revise and restructure

- **Candidate**  
  — changes should be additive or clarifying

- **Locked**  
  — changes must be bug fixes or formal clarifications only  
  — no semantic changes without explicit approval

Breaking changes to locked sections require prior discussion.

---

### 4. Propose, Don’t Assume

If you believe a rule should change (not just be clarified):

- Explain *why* the existing rule is insufficient
- Describe the proposed change in neutral, normative language
- Identify which document(s) are affected

Do not assume intent. Treat the current text as canonical unless explicitly stated otherwise.

---

## Writing Style Guidelines

Contributions should follow these style rules:

- Prefer declarative sentences (“X MUST do Y”)
- Avoid narrative or metaphorical language
- Use numbered lists for ordered rules
- Define all terms before using them
- Use consistent terminology across documents

For pseudocode in `REFERENCE.md`:

- Treat pseudocode as executable intent
- Avoid language-specific constructs
- Use assertions for invariants
- Prefer small, single-purpose functions

---

## How to Submit Changes

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Open a pull request with:
   - a summary of what changed
   - the motivation
   - which document(s) are affected
   - whether the change is clarifying or semantic

Pull requests may be rejected if they introduce ambiguity, non-determinism, or implementation-specific assumptions.

---

## Review Philosophy

Reviews focus on:

- clarity
- determinism
- internal consistency
- agent-interpretability

A contribution may be rejected even if it is technically correct,
if it makes the specification harder to implement reliably.

---

## Final Authority

The repository owner retains final authority over:

- rule changes
- document structure
- section locking
- versioning decisions

This is intentional and ensures the specification remains coherent and stable.

---

## Thank You

Clear specifications are hard to write.
Thoughtful contributions are appreciated.

If in doubt, **err on the side of explicitness**.
