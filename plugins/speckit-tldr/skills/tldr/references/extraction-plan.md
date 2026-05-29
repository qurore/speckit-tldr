# Extraction profile: `plan.md`

`plan.md` is the **How**: the technical decisions the spec deliberately omits
(architecture, stack, data model, contracts, testing). For a reviewer this is
where maintainability and security risk gets baked in, so review it on its own
axis — not just "does it match the spec", but "are these good decisions". Extract
into the `plan` object of the DATA schema.

## Fields to extract

- `tldr` — 2–3 lines: the chosen approach at a glance.
- `stack[]` — concrete technology choices (`{ name, role, changedInPR }`):
  language, framework, datastore, key libraries, infra.
- `decisions[]` — the load-bearing design decisions **paired with rationale**
  (`{ decision, rationale, source, changedInPR, flags[] }`). If a decision has no
  stated rationale, set `rationale: null` and flag it.
- `constitutionCheck[]` — each item from the plan's "Constitution Check" section
  (`{ rule, status: "pass"|"fail"|"unknown", note, source }`). If the plan has no
  Constitution Check section, emit a single synthetic item with
  `status: "unknown"` and flag `constitution`.
- `structure[]` — the proposed module / directory / contract layout (short).
- `risks[]` — stated risks, trade-offs, and "alternatives considered"
  (`{ text, source, changedInPR }`).
- `phases[]` (optional) — if the plan defines implementation phases.

## Flags to raise on `plan.md`

- `mismatch` — the plan implements something the spec does not require, or omits
  something the spec requires. (Requires the spec in context; when scope is
  `both`, cross-check. When plan-only, note that cross-check was not performed.)
- `constitution` — a decision that conflicts with a `constitution.md` rule, or a
  failed/absent Constitution Check.
- `dependency-gap` — a choice that depends on an undecided/undefined component.
- `ambiguous` — a decision stated without enough specificity to implement.
- A decision with `rationale == null` → flag `ambiguous` and surface it; unexplained
  decisions are the hardest to review and the easiest to regret.
- `changed-in-pr` — touched by the diff.

## Review heuristics

- Prefer to show *decision + rationale + alternative-rejected* together; a reviewer
  can approve fast when the "why" is visible and is slowed most by silent choices.
- New external dependencies, new data stores, and new network boundaries are
  high-attention items — make sure they surface even if the prose is brief.
- If the plan introduces tech that the constitution forbids (e.g. an ORM when the
  constitution says raw SQL), that is a `constitution` flag, high severity.
