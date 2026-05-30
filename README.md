# speckit-tldr

A [Claude Code](https://code.claude.com/docs/) plugin that turns
[GitHub Spec Kit](https://github.com/github/spec-kit) `spec.md` and `plan.md`
files into review-oriented **TLDRs** — a self-contained HTML dashboard plus a
PR-native Markdown version — so a reviewer can get oriented and spot the risky
parts before reading the full document.

It is built for the **PR reviewer**: it regenerates from the canonical files,
surfaces ambiguities / open questions / spec↔plan mismatches at the top, and
marks what changed in the PR. The TLDR is an *entry point* into the document, not
a replacement for reading it — every item links back to its source location.

## What it does

- **`spec.md`** → intent, requirements (with acceptance criteria), decisions,
  and open questions, flagged for ambiguity / untestability / missing criteria.
- **`plan.md`** → stack, design decisions *paired with rationale*, Constitution
  Check results, structure, and risks, flagged for mismatch / constitution
  violations / unexplained decisions.
- **Diff-aware**: items touched by the current PR are marked, with a "show only
  changed" filter in the HTML.
- **Cleanup**: a companion `tldr-delete` skill removes the generated TLDRs again
  so these local review artifacts never get committed into the PR.

## Install

Run these in Claude Code.

### From the Claude community marketplace

```shell
/plugin marketplace add anthropics/claude-plugins-community
/plugin install speckit-tldr@claude-community
```

> The community-marketplace entry becomes installable once the submission is
> approved and synced; until then, use the direct method below.

### Directly from this repo

```shell
/plugin marketplace add qurore/speckit-tldr
/plugin install speckit-tldr@speckit-tldr
```

## Use

```shell
/speckit-tldr:tldr            # both spec.md and plan.md for the current feature
/speckit-tldr:tldr plan       # plan only
/speckit-tldr:tldr spec       # spec only
/speckit-tldr:tldr specs/042-export/plan.md   # a specific file
```

It also triggers from natural language ("tldr the auth feature spec",
"summarize this plan.md for review"). Output is written to
`specs/<feature>/tldr/` as `*.tldr.html` and `*.tldr.md`.

## Clean up before a PR

The TLDRs are a local reviewing aid, not something the PR should carry. Remove
them again with:

```shell
/speckit-tldr:tldr-delete         # remove the current feature's TLDRs
/speckit-tldr:tldr-delete plan    # only the plan TLDR
/speckit-tldr:tldr-delete all     # every *.tldr.* under specs/ in the repo
```

It lists exactly what it will remove and asks before deleting, removes already
committed files with `git rm` (so the deletion lands in the PR), can offer to add
the patterns to `.gitignore`, and only ever touches `*.tldr.html` / `*.tldr.md` —
never `spec.md` / `plan.md`.

## Repo layout

```
speckit-tldr/
├── .claude-plugin/marketplace.json         # marketplace catalog
├── plugins/speckit-tldr/
│   ├── .claude-plugin/plugin.json          # plugin manifest
│   └── skills/
│       ├── tldr/                           # generate the TLDR
│       │   ├── SKILL.md
│       │   ├── references/                 # spec/plan extraction profiles + flag taxonomy
│       │   │   ├── extraction-spec.md
│       │   │   └── extraction-plan.md
│       │   └── assets/                     # output templates
│       │       ├── tldr.template.html
│       │       └── tldr.template.md
│       └── tldr-delete/                    # remove generated TLDRs before a PR
│           └── SKILL.md
├── LICENSE
└── README.md
```

## Development

Test against a local clone before publishing changes:

```shell
claude plugin validate .                      # validate marketplace.json
claude plugin validate ./plugins/speckit-tldr # validate plugin.json + skill frontmatter

# In Claude Code, from the repo root:
/plugin marketplace add .
/plugin install speckit-tldr@speckit-tldr
/speckit-tldr:tldr            # run against a real specs/<feature>/ in some repo
```

> Plugin versions: `plugin.json` pins `version` (`0.2.0`). Bump it on each release,
> or remove it to let every commit count as a new version.

## License

MIT — see [LICENSE](./LICENSE).
