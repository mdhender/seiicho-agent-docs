# Seiicho Specification Project — Charter

## Purpose

This project exists to develop the **canonical specification** for *Seiicho*, consisting of:

* **RULEBOOK.md** — player-facing concepts and definitions
* **SEMANTICS.md** — deterministic rules, resolution order, invariants
* **REFERENCE.md** — authoritative, executable-style pseudocode

The primary goal is to enable **correct, deterministic implementation by coding agents** without requiring clarification, interpretation, or guesswork.

This is a **specification effort**, not a design playground.

The official repository is **github.com:mdhender/seiicho-agent-docs**.

---

## Scope

Included:

* Normative rules and definitions
* Deterministic generation and resolution rules
* Identifier formats
* PRNG usage and seeding rules
* Canonical pseudocode

Explicitly excluded:

* UI/UX concerns
* Database schemas
* Go/JS/SQL implementation details
* Exploratory or speculative game design (unless promoted explicitly)

Design exploration happens **outside this project** and is promoted here only once decisions are made.

---

## Roles

* **Author (Michael)**

  * Owns all design decisions
  * Decides what is canonical
  * Locks sections when ready

* **Assistant (ChatGPT)**

  * Acts as a spec editor and reviewer
  * Eliminates ambiguity and underspecification
  * Enforces determinism and consistency
  * Translates intent into agent-safe language and pseudocode
  * Flags contradictions, edge cases, and agent hazards

The assistant does **not** introduce new mechanics unless explicitly asked.

---

## Working Conventions

* This project uses **one continuous thread**.
* Changes are treated as **diffs**, not wholesale rewrites.
* Sections may be explicitly marked:

  * *Draft*
  * *Candidate*
  * *Locked*

Once locked, behavior is considered normative.

---

## Authority and Storage

* The **repository is the source of truth**.
* Chat is used for:

  * drafting,
  * normalization,
  * review,
  * and clarification.
* Sections may be shared via:

  * pasted text (for active rewriting), or
  * links to commits (for review and validation).

---

## Style Guidelines

* Prefer declarative, testable language.
* Avoid metaphor, narrative prose, or implementation hints.
* Every rule should answer:

  * *What happens?*
  * *In what order?*
  * *Under what constraints?*
* Ambiguity is treated as a bug.

---

## Success Criteria

This project is successful when:

> A coding agent can implement the game engine from RULEBOOK + SEMANTICS + REFERENCE **without asking follow-up questions**, and two independent implementations produce identical results given the same inputs.

---

## Initial Focus

Begin with:

1. Cluster generation
2. Identifiers and coordinate systems
3. PRNG and determinism rules
4. Planet and orbit assignment
