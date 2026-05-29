# Extraction profile: `spec.md`

`spec.md` is the technology-agnostic **What / Why**. Errors here propagate to plan,
tasks, and code, so this is the highest-leverage thing to review. Extract the
following into the `spec` object of the DATA schema.

## Fields to extract

- `tldr` — 2–3 lines: what is being built and why. Pull from the spec's intent /
  user-scenario / overview section. No implementation detail.
- `requirements[]` — one entry per functional/non-functional requirement:
  - `id` — the spec's own ID if present (e.g. `FR-001`), else assign `R1, R2, …`.
  - `text` — the requirement, lightly condensed (do not lose conditions/quantifiers).
  - `acceptance` — its acceptance criteria, or `null` if none stated.
  - `source` — `{ file, heading, lines }` so the reviewer can jump to it.
  - `changedInPR` — from the diff (Step 2).
  - `flags[]` — see below.
- `decisions[]` — explicit product/scope decisions and constraints
  (`{ text, source, changedInPR }`). Include out-of-scope statements.
- `openQuestions[]` — every `[NEEDS CLARIFICATION]`, TODO, "TBD", or open question
  (`{ text, severity: "high"|"med", source, changedInPR }`). These render first.
- `actors[]` (optional) — user roles / personas the spec names.

## Flags to raise on `spec.md`

- `ambiguous` — vague quantifiers ("fast", "many", "appropriate", "etc."),
  undefined terms, or requirements that admit multiple implementations.
- `untestable` — a requirement whose satisfaction can't be objectively checked.
- `missing-criteria` — a requirement with `acceptance == null`.
- `unanswered` — any open question / `[NEEDS CLARIFICATION]` marker.
- `dependency-gap` — references a concept, system, or term the spec never defines.
- `changed-in-pr` — touched by the diff.

## Review heuristics (apply while reading, not exhaustively)

- A requirement that mixes *what* with *how* (names a library, schema, endpoint)
  is a smell — note it; it usually belongs in `plan.md`.
- Numbers without units, thresholds without comparison operators, and lists ending
  in "etc." are classic ambiguity sources.
- Every actor introduced should have at least one requirement; orphan actors and
  orphan requirements are both worth flagging in `notes`.
