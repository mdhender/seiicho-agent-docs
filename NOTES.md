# Notes

Agents must ignore this file.

----

Writing the RULEBOOK, SEMANTICS, and REFERENCE is going to be an iterative process. Help me understand the best way to coordinate with you. Should I create a separate project for this (as opposed to including it in this project which is a hodgepodge of design and coding questions)?

---

This is a *really* good meta-question — and yes, how you coordinate this will materially affect how good the final rulebook set becomes.

Short answer first, then the why and the concrete setup.

---

## Short Answer

**Yes — create a separate project / thread dedicated to RULEBOOK–SEMANTICS–REFERENCE.**

Treat it as its own artifact with:

* its own goals,
* its own iteration rhythm,
* and its own success criteria (“a coding agent can implement this without asking questions”).

Keep this current thread/project for **design + coding + experimentation**.
Use the new one for **normative specification**.

---

## Why this matters (especially with AI agents)

You are now doing something subtle and hard:

> You’re writing a *machine-consumable specification*, not documentation.

That’s a different cognitive mode than:

* debugging Go,
* choosing Next.js vs Ember,
* or designing gameplay systems.

If those concerns are mixed, three things happen:

1. **Specs drift** (rules get “softened” by discussion).
2. **Agents hallucinate intent** (because context is noisy).
3. **You lose the ability to say “this is canonical.”**

A clean separation lets you say:

> “This project is authoritative. Everything else conforms to it.”

That’s gold.

---

## Recommended Structure

### 1️⃣ Create a dedicated project / thread

Call it something explicit, e.g.:

* **Seiicho Spec**
* **Seiicho Canon**
* **Empyrean-Style Ruleset Spec**
* **Seiicho Rules Reference**

In the *first message*, set the contract clearly. Example:

> This project is for developing the canonical RULEBOOK, SEMANTICS, and REFERENCE documents for Seiicho.
> The goal is deterministic, unambiguous implementation by coding agents.
> No gameplay debate unless it affects specification clarity.

I will then stay in **spec editor mode**, not design partner mode.

---

### 2️⃣ Lock roles explicitly

When we’re in the spec project:

* **You** decide rules.
* **I**:

  * normalize language,
  * eliminate ambiguity,
  * surface edge cases,
  * enforce determinism,
  * and produce agent-safe wording.

When we’re *not* in that project, I’ll happily brainstorm, challenge assumptions, and explore alternatives.

This role clarity prevents accidental rule churn.

---

### 3️⃣ Use a predictable iteration loop

The most effective loop looks like this:

1. **You paste a section** (or diff).
2. You say one of:

   * “Normalize this”
   * “Harden this for agents”
   * “Check this for ambiguity”
   * “Convert this to REFERENCE-style pseudocode”
3. I:

   * rewrite,
   * annotate problems,
   * or propose a tightened version.
4. You accept / reject / tweak.
5. We move on.

Avoid:

* “What do you think?”
* “Is this good?”

Those invite design drift.
Instead: *“Make this deterministic / canonical / agent-proof.”*

---

### 4️⃣ Version the documents mentally (even if not in Git yet)

Think in terms of **stability levels**:

* **RULEBOOK** → allowed to evolve
* **SEMANTICS** → changes carefully
* **REFERENCE** → changes rarely, intentionally

When you say:

> “Assume REFERENCE.md v0.1”

…I will treat earlier decisions as locked unless you explicitly say otherwise.

This mirrors how RFCs and language specs work.

---

### 5️⃣ Keep implementation questions out of the spec project

This is important.

Questions like:

* “How do I store this in SQLite?”
* “Should this be a struct or interface?”
* “How do I generate this in Go?”

Belong **outside** the spec project.

In the spec project we answer only:

* *What must exist?*
* *What must always be true?*
* *What happens first, second, third?*

This discipline is why your REFERENCE.md already reads like something agents can *execute*.

---

## When to loop back to this project

Use this current “hodgepodge” project for:

* trying generator variants,
* exploring presets,
* debugging Go,
* testing ideas,
* or asking “what if?”

Once something stabilizes, **promote it** into the spec project.

That mirrors real standards work.

---

## Final Recommendation

✔ Create a **separate spec project / thread**
✔ Declare it **canonical and agent-facing**
✔ Use tight iteration language
✔ Treat REFERENCE as sacred
✔ Keep design play elsewhere

If you want, I can help you write the **opening charter message** for that new project so we start on exactly the right footing.

Just say the word.

----

Will I have many separate chat threads in that project or just one continuous thread?

---

**One continuous thread.**
Very strongly recommended.

Here’s why, and when (rarely) you’d split.

---

## Why a single continuous thread works best

### 1️⃣ Specs depend on *accumulated context*

RULEBOOK → SEMANTICS → REFERENCE is *progressively constraining*.
Each decision tightens the next layer.

A single thread lets me:

* remember which choices are **locked**,
* detect when a new change contradicts an earlier invariant,
* preserve generation order, determinism assumptions, and naming conventions.

Multiple threads force me to *re-derive intent* — which is exactly what you’re trying to avoid for coding agents.

---

### 2️⃣ You’re effectively writing a standard, not docs

Standards work (RFCs, language specs, game engines) almost always evolves in a **single canonical discussion stream** with revisions layered on top.

Your workflow already resembles:

* “draft → tighten → canonicalize”
* not “parallel brainstorming”

A continuous thread mirrors that mental model.

---

### 3️⃣ Fewer chances for silent divergence

If you split threads, this happens:

* Thread A tweaks orbit logic
* Thread B adjusts planet counts
* Thread C updates PRNG rules

…and suddenly nothing quite matches.

One thread means:

> If it’s not in this thread, it’s not canon.

That’s a powerful constraint.

---

## How to keep one thread from becoming messy

### Use *explicit section transitions*

When you change focus, say it plainly:

* “We are now editing RULEBOOK §3.2 only.”
* “Switching to SEMANTICS; no new rules.”
* “REFERENCE freeze except for bugs.”

I will respect those boundaries.

---

### Mark decisions as **locked**

When something is final, say:

> “Lock this behavior.”

I will:

* stop suggesting alternatives,
* flag later contradictions,
* treat it as normative.

This prevents accidental erosion.

---

### Treat revisions as diffs, not rewrites

Instead of:

> “Here’s a new version of the generator section…”

Prefer:

> “Change planet distribution rule as follows…”

This keeps the mental diff small and precise.

---

## When you *should* start a second thread (rare)

Only do this if:

### 🚨 The spec thread is *frozen*

For example:

* “REFERENCE.md v0.1 is frozen.”
* You want to explore a **breaking alternative**.

In that case:

* start a **design exploration thread**
* label it clearly as *non-canonical*
* promote results back *only if adopted*

Think of it as a feature branch.

---

## Practical recommendation (TL;DR)

✔ **One continuous thread** for the spec project
✔ Explicitly mark scope changes and locked decisions
✔ Use diffs and intent-driven edits
✔ Spin off side threads only for experimental design

If you want, next I can help you:

* write a **Spec Thread Charter** you paste at the top of the new project, or
* define a **“locking language”** (phrases you’ll use to freeze sections cleanly).

Just tell me which.

----

How will I share documents? Will it always be cut and paste or can I link to commits in the repository?

---

Short answer: **both**, but with a clear division of responsibility.

**Use links to commits as the source of truth**, and **paste text only when you want active editing**.

Here’s the model that works best with me *and* with human collaborators.

---

## The Best Working Pattern (Proven)

### ✅ Repository = Canonical Storage

### ✅ Chat = Editing & Review Workspace

Think of this like a code review workflow.

---

## Option A — Link to commits (recommended default)

**Use this when:**

* A section is “mostly right”
* You want review, critique, or validation
* You want to preserve authoritative history

### How to do it

You say something like:

> I’ve updated `REFERENCE.md` in commit `abc123`.
> Please review §“Planet Distribution Across Stars and Orbits” for ambiguity or agent hazards.

I will:

* reason about the content *as if it were canonical*,
* point out contradictions, missing invariants, or dangerous phrasing,
* propose **precise diffs** you can apply back to the repo.

### Why this works well

* Your repo stays the **single source of truth**
* You get real versioning, history, and rollback
* The spec thread doesn’t become a giant pastebin
* You can tag versions: `spec-v0.1`, `reference-freeze`, etc.

This mirrors how RFCs and language specs are actually developed.

---

## Option B — Paste sections (when you want active rewriting)

**Use this when:**

* A section is unstable
* You want me to *rewrite* or *normalize* it
* You’re still discovering the right structure

Typical prompts:

* “Normalize this for agents”
* “Rewrite this as canonical pseudocode”
* “Harden this section; no new rules”

In this mode, chat is the **working scratchpad**, not the archive.

Once you’re happy:

* you paste *back* into the repo,
* commit,
* and that commit becomes canonical.

---

## What NOT to do

🚫 Don’t rely on chat as the only storage
Chat has no diff history, no tags, no freeze points.

🚫 Don’t paste entire documents repeatedly
That increases error risk and makes it harder to reason about what changed.

🚫 Don’t mix speculative edits with canonical ones in the same paste
Say explicitly whether a section is *draft*, *candidate*, or *locked*.

---

## A Simple, Clean Workflow

Here’s a concrete loop you can reuse:

1. **Edit locally**

   * Update `RULEBOOK.md`, `SEMANTICS.md`, or `REFERENCE.md`
2. **Commit**

   * Message like:
     `spec: clarify orbit index ↔ letter mapping`
3. **Link in chat**

   * “Please review commit `<hash>`; focus on determinism and ambiguity.”
4. **Apply feedback**

   * Either as a follow-up commit or a patch
5. **Lock**

   * “Lock this section.”

That’s it.

---

## How I’ll treat links vs pasted text

* **Linked commit** → I assume it’s *canonical intent*
  I won’t casually suggest redesigns unless you ask.

* **Pasted text** → I assume it’s *mutable*
  I’ll rewrite aggressively if helpful.

If you ever want to override that, just say:

* “Treat this paste as canonical”
* or “This commit is exploratory”

---

## Optional but powerful: version markers

When you’re ready, you can say things like:

> REFERENCE.md is now at v0.1.
> Only bug fixes and clarifications allowed.

I will then:

* flag breaking changes,
* suggest additive-only edits,
* and help you maintain spec stability.

---

## Bottom line

✔ Use **repository commits for authority**
✔ Use **chat for precision editing and review**
✔ Paste text only when you want hands-on rewriting
✔ Link commits when you want validation and critique

If you want, next I can help you:

* define a **spec versioning scheme**, or
* draft a **CONTRIBUTING.md for the spec**, explaining this workflow for future collaborators (human or AI).

Just say the word.

----

Write the opening charter message for that new project so we start on exactly the right footing.

---

Here is a **clean, explicit opening charter message** you can paste as the *first message* of the new project/thread. It sets scope, roles, and working rules very clearly, without sounding bureaucratic.

---

# Seiicho Specification Project — Charter

## Purpose

This project exists to develop the **canonical specification** for *Seiicho*, consisting of:

* **RULEBOOK.md** — player-facing concepts and definitions
* **SEMANTICS.md** — deterministic rules, resolution order, invariants
* **REFERENCE.md** — authoritative, executable-style pseudocode

The primary goal is to enable **correct, deterministic implementation by coding agents** without requiring clarification, interpretation, or guesswork.

This is a **specification effort**, not a design playground.

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

---

If you want, next we can:

* tune this charter to sound more or less formal, or
* add a short **“How to Propose Changes”** section once the spec matures.

Just say so.

----

Here’s a **short, repo-ready section** you can drop straight into a README or CONTRIBUTING.md. It’s written to be clear, opinionated, and future-proof for *future you* (and any collaborators).

---

## Working with ChatGPT

This repository uses ChatGPT as a **specification assistant and compiler**, not as a source of truth or a file editor. To avoid ambiguity and tool limitations, follow the rules below.

### Source of Truth

* **GitHub is authoritative.**
* ChatGPT does **not** reliably read from repositories, links, or IDE tabs.
* Never assume ChatGPT has access to any file unless its contents are explicitly provided.

### Providing Context

* **Always paste file contents** when correctness matters.
* Wrap pasted text in clear guard lines:

```text
=== BEGIN FILE: SEMANTICS.md ===
...
=== END FILE ===
```

* For partial edits, paste only the relevant section and label it clearly.

### Requesting Changes

* **Never ask ChatGPT to “edit the file.”**
* Always request **replacement blocks** for specific sections.

Example:

```text
TASK:
Produce a replacement block for §3.2 “Orbit Index Mapping”.
Do not modify any other sections.
Use normative language (MUST / MUST NOT).
```

ChatGPT should respond with a self-contained block suitable for manual insertion.

### Canon and Invariants

* Treat pasted text as **canonical** unless stated otherwise.
* Explicitly mark invariants and fixed mappings.
* ChatGPT should not infer, extrapolate, or “helpfully improve” canon without being asked.

### File Uploads

* Drag-and-drop uploads may be used for:

    * Large, stable drafts
    * Review, summarization, or sanity-checking
* Do **not** rely on uploads for precise edits or canonical definitions.

### Workflow Summary

* Paste → Generate → Review → Commit
* ChatGPT proposes; humans decide.
* All final edits are applied manually in Git.

This discipline keeps the specification deterministic, reviewable, and resistant to tool or platform changes.

---

If you want, I can also:

* Tighten this further for a `CONTRIBUTING.md`
* Add a **“Normative Language”** subsection (RFC-style)
* Write a **ChatGPT Charter** specifically for Seiicho agents

Just say which direction to take it.

----

## Doc → Spec → Reference promotion pipeline

This repo treats “documentation” as a staged pipeline. Text starts as exploratory notes, gets promoted into normative specification language, and finally becomes executable-style reference material.

### Stages

**Stage 0 — Notes (DOC)**

* Purpose: capture ideas, alternatives, questions, and examples.
* Tone: informal; may include TODOs, open questions, “maybe,” and competing options.
* Allowed: ambiguity, multiple approaches, narrative explanation.
* Not allowed: declaring canon without marking it as such.

**Stage 1 — Specification (SPEC / SEMANTICS)**

* Purpose: define *canon* and invariants.
* Tone: normative, deterministic, reviewable.
* Language: MUST / MUST NOT / SHOULD / MAY.
* Structure:

    * numbered rules
    * explicit inputs/outputs
    * explicitly stated invariants
    * error/invalid-input behavior (panic/return error/etc.)
* Required: each rule is testable (or at least falsifiable) against an implementation.

**Stage 2 — Reference (REFERENCE)**

* Purpose: provide “executable-style” pseudocode and canonical mappings suitable for implementation.
* Tone: minimal prose; high precision.
* Contents:

    * pseudocode blocks only (or near-only)
    * canonical tables/mappings (index ↔ letter, enums, ranges)
    * algorithm order and RNG derivation spelled out
* Required: matches SPEC exactly; no new rules introduced here.

---

## Promotion criteria

### DOC → SPEC checklist

Promote content only when:

* The team agrees on **one** behavior (canon), not multiple options.
* Inputs, outputs, and ranges are fully defined.
* “Edge cases” are decided (invalid coordinates, overflow, empty lists, etc.).
* The rule can be tested or verified with fixtures.
* Any randomness is fully specified (seed, child RNG derivation, ordering).

**Promotion artifact:** a SPEC section with:

* a short heading
* numbered normative rules
* explicit invariants
* defined failure behavior

### SPEC → REFERENCE checklist

Promote content only when:

* The SPEC text is stable enough to implement.
* The algorithm can be expressed as deterministic pseudocode.
* All mappings (letters, indices, ranges) are fixed.
* Order-dependence is explicit (iteration order, acceptance order, sorting rules).

**Promotion artifact:** a REFERENCE section with:

* canonical pseudocode
* exact variable names / step order
* zero new behavior beyond SPEC

---

## Change control

### “Canon only changes via SPEC”

* Any behavior change MUST be made in SPEC first.
* REFERENCE is updated *after* SPEC, as a mechanical translation.
* DOC can propose changes, but does not change canon.

### Versioning and review gates

* **DOC changes**: lightweight review.
* **SPEC changes**: require explicit review and sign-off (treat like code behavior).
* **REFERENCE changes**: verify it is a faithful transcription of SPEC (diff check).

Suggested PR labels:

* `docs-notes` (DOC)
* `spec-change` (SEMANTICS / canon)
* `reference-sync` (REFERENCE mechanical update)

---

## How to use ChatGPT in the pipeline

### For DOC (Notes)

* Ask for options, tradeoffs, examples, diagrams, and naming.
* Output is allowed to be exploratory.

### For SPEC (Canon)

* Paste the relevant DOC excerpt and ask ChatGPT to:

    * rewrite as numbered MUST/MUST NOT rules
    * explicitly state invariants and invalid-input behavior
    * avoid introducing new behavior
* 
* Require replacement blocks.

### For REFERENCE (Executable pseudocode)

* Paste the SPEC section and ask ChatGPT to:

    * translate into pseudocode blocks only
    * keep algorithm order identical
    * avoid prose and avoid new rules
* Require replacement blocks.

---

## Operational workflow (repeatable)

1. **Draft in DOC**

    * Capture ideas and examples.
2. **Decide canon**

    * Resolve options; pick one.
3. **Promote to SPEC**

    * Convert to normative rules; lock invariants.
4. **Implement / test**

    * Add fixtures that encode the SPEC.
5. **Promote to REFERENCE**

    * Translate SPEC → pseudocode + mappings.
6. **Sync and verify**

    * Ensure REFERENCE matches SPEC; no drift.

---

## “Drift prevention” rule

If SPEC and REFERENCE disagree:

* SPEC wins.
* REFERENCE must be updated immediately.
* If implementation matches REFERENCE but not SPEC, treat as a bug unless the SPEC is intentionally being changed.

---

If you want, I can also produce a **ready-to-paste markdown section** tailored to your current file names (e.g., `RULEBOOK.md`, `SEMANTICS.md`, `REFERENCE.md`) and add a tiny “promotion PR template” you can copy into GitHub PR descriptions.

