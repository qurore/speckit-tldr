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

## Install

```shell
# In Claude Code:
/plugin marketplace add qurore/spec-kit-tldr
/plugin install speckit-tldr@speckit-tldr
```

> The GitHub repository is `spec-kit-tldr`, while the marketplace and the plugin
> it provides are both named `speckit-tldr` — so you add `qurore/spec-kit-tldr`
> but install `speckit-tldr@speckit-tldr`.

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

## Repo layout

```
spec-kit-tldr/
├── .claude-plugin/marketplace.json     # marketplace catalog
├── plugins/speckit-tldr/
│   ├── .claude-plugin/plugin.json      # plugin manifest
│   └── skills/tldr/
│       ├── SKILL.md                    # skill logic
│       ├── references/                 # spec/plan extraction profiles + flag taxonomy
│       │   ├── extraction-spec.md
│       │   └── extraction-plan.md
│       └── assets/                     # output templates
│           ├── tldr.template.html
│           └── tldr.template.md
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

> Plugin versions: `plugin.json` pins `version` (`0.1.0`). Bump it on each release,
> or remove it to let every commit count as a new version.

## License

MIT — see [LICENSE](./LICENSE).
