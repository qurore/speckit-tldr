---
name: tldr
description: >-
  Generate a review-oriented TLDR of GitHub Spec Kit artifacts (spec.md and/or
  plan.md) as a self-contained HTML dashboard plus a Markdown version, to make
  reviewing those files in a pull request faster. Use this skill whenever the
  user wants to review, summarize, "TLDR", skim, or get an overview of a Spec
  Kit spec.md or plan.md, whenever they are reviewing a PR that touches files
  under specs/<feature>/, or whenever they mention spec-driven development (SDD)
  review, even if they do not explicitly say "TLDR". This skill produces an
  on-ramp to reading the document (with pointers back into it), NOT a replacement
  for reading it.
argument-hint: "[spec|plan|both|<path-to-md>]"
allowed-tools: Read, Write, Glob, Bash(git diff:*), Bash(git log:*), Bash(git merge-base:*)
---

# Spec Kit TLDR

Produce a fast, review-oriented overview of Spec Kit `spec.md` / `plan.md` files.
The goal is to cut the time a PR **reviewer** spends getting oriented, while
deliberately surfacing the spots that deserve close reading rather than hiding
them. A TLDR here is an *entry point* into the document, not a substitute for it.

## Core principles (read first)

1. **Regenerate from the canonical source.** Always extract from the actual files
   in the repo / PR. Never trust a pre-existing summary an author may have written;
   a summary's omissions become the reviewer's blind spots.
2. **Surface risk, don't smooth it over.** Ambiguities, missing acceptance
   criteria, and unanswered questions go to the *top*, not buried. The summary's
   job is to point at what to scrutinize.
3. **Point back into the document.** Every extracted item records its source file
   and (when available) heading/line, so the reviewer can jump to the full text.
4. **Be diff-aware.** Reviewers review the *change*, not the whole document. Mark
   what this PR added/changed and let the reviewer filter to just that.

## Step 1 — Resolve scope and target files

Parse the invocation argument (`$ARGUMENTS`), and also honor natural language in
the user's message. Resolution order:

- A path argument (e.g. `specs/042-foo/plan.md`) → operate on exactly that file;
  detect spec vs plan from the filename.
- `spec` / `plan` → restrict to that artifact type. `both` (or no argument) →
  process both `spec.md` and `plan.md` for the target feature.
- Natural language overrides/refines the argument: "just the plan", "spec only",
  "tldr the auth feature spec" → scope accordingly.

To find the feature directory when no path is given:
- If the user names a feature, glob `specs/*<feature>*/`.
- Otherwise infer from the current PR/branch: the changed `specs/<feature>/*.md`
  files (see Step 2). If still ambiguous, list the candidates under `specs/` and
  ask which feature.

Spec Kit convention: each feature lives in `specs/<NNN-feature-name>/` with
`spec.md`, `plan.md`, and often `research.md`, `data-model.md`, `contracts/`,
`tasks.md`. Only `spec.md` and `plan.md` are in scope for this skill.

## Step 2 — Determine the diff (PR-review mode)

This skill defaults to reviewer mode. Establish what changed so items can be
marked `changedInPR`:

```bash
# Base for comparison: the merge-base with the default branch (adjust if needed).
git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD main
# Per-file changed line ranges:
git diff --unified=0 <base>...HEAD -- specs/<feature>/spec.md
git diff --unified=0 <base>...HEAD -- specs/<feature>/plan.md
```

Map changed line ranges back to the headings/items you extract so each item gets
`changedInPR: true|false`. If this is not a PR (no meaningful diff, or the user
asked for a plain overview), set every item's `changedInPR` to `false` and skip
the change-filter affordance — the rest of the flow is unchanged.

## Step 3 — Extract

Read the file(s), then extract into the **DATA schema** described in
`assets/tldr.template.html` (a single JSON object the template renders). The
*what to extract* and the *flag taxonomy* differ by artifact type:

- For `spec.md`, follow `references/extraction-spec.md`.
- For `plan.md`, follow `references/extraction-plan.md`.

Both reference files define exactly which fields to pull and which flags to raise.
Read the relevant one(s) before extracting. Do not invent content that is not in
the source; if a required section is absent, that absence is itself a flag
(e.g. a requirement with no acceptance criteria).

## Step 4 — Render outputs

Produce **both** formats (unless the user asked for only one):

1. **HTML** — copy `assets/tldr.template.html`, replace the
   `/* === DATA PLACEHOLDER === */` block with your populated `DATA` object, and
   write the result. The template is self-contained (no network/build needed) and
   handles all rendering, the flag-summary bar, and the "changed in PR" filter.
2. **Markdown** — follow `assets/tldr.template.md`. This version renders inline in
   a GitHub PR comment or file view (GitHub strips styled HTML, so Markdown is the
   PR-native surface). Keep it scannable; lead with open questions / flags.

### Output location and naming

Write to a `tldr/` directory next to the source (create if missing), or to the
path the user specifies:

```
specs/<feature>/tldr/spec.tldr.html
specs/<feature>/tldr/spec.tldr.md
specs/<feature>/tldr/plan.tldr.html
specs/<feature>/tldr/plan.tldr.md
```

When scope is `both`, you may also emit a combined `feature.tldr.html` that
includes both sections (the template supports a `spec` section, a `plan` section,
or both). After writing, tell the user the file paths and give a one-line summary
of the most important flags found.

## Flag taxonomy (shared)

Each extracted item may carry zero or more flags. Keep the meanings consistent;
the template colors them and counts them in the summary bar.

| flag | meaning |
| --- | --- |
| `ambiguous` | wording allows multiple readings; not precise enough to implement |
| `untestable` | no measurable / verifiable acceptance criterion |
| `missing-criteria` | a requirement with no acceptance criteria at all |
| `unanswered` | an open question / `[NEEDS CLARIFICATION]` marker |
| `mismatch` | spec↔plan disagreement (plan does something the spec doesn't ask for, or vice versa) |
| `uncovered` | (when `tasks.md` is present) a requirement with no covering task |
| `constitution` | conflicts with a rule in `constitution.md` / Constitution Check |
| `dependency-gap` | depends on something undefined / not yet decided |
| `changed-in-pr` | added or modified by the PR under review |

`unanswered`, `missing-criteria`, `mismatch`, and `constitution` are high-severity
and must appear at the top of the output regardless of where they occur in the
source.

## What NOT to do

- Do not paraphrase so heavily that the reviewer could pass review on the TLDR
  alone. Extract structure and flags; link back to the source for the prose.
- Do not drop edge cases, constraints, or caveats just because they are verbose —
  those are exactly where review risk lives.
- Do not summarize files other than `spec.md` / `plan.md`.
