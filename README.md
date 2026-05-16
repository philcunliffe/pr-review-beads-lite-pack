# PR Review Workflows for Beads-Lite

Formula-driven adopt-PR workflow for Gas City. Reviews and merges contributor
PRs using a multi-model review engine (Claude + Codex + Gemini), with a human
gate between review and merge. This fork is adapted for cities that use
`gastown-beads-lite-pack` and SQLite beads-lite stores.

## What's Included

- **`mol-adopt-pr` formula** — full incoming-PR review/adoption workflow
- **`mol-pr-merge-only` formula** — fast incoming-PR merge path after human review
- **`mol-pr-ci-diagnose` formula** — read-only PR CI failure diagnosis
- **`mol-pr-from-issue` formula** — outbound issue-to-PR workflow
- **`mol-pr-iterate` formula** — iterate an outgoing PR from review/coverage feedback
- **`mol-pr-revert` formula** — mayor-invoked revert-PR workflow
- **`/review-pr` skill overlay** — upstream multi-model review prompt for Claude-backed workers

## Prerequisites

- `gh` CLI authenticated with repo access
- `gastown-beads-lite-pack` imported and configured as the city beads provider
- `bd-lite` available through `gc gastown-beads-lite bd`
- A rig worker target such as `<rig>/gastown-beads-lite.polecat`
- The companion `pr-pipeline` formulas installed when using outbound workflows
  that reference `mol-pr-start`, `mol-pr-ship`, or `mol-pr-review`

## Install

In your city's `pack.toml`:

```toml
[imports.pr-review]
source = "packs/pr-review-workflows-beads-lite-pack"

[defaults.rig.imports.pr-review]
source = "packs/pr-review-workflows-beads-lite-pack"
```

Then install formulas into the city and registered rig beads-lite stores:

```bash
gc pr-review install
```

## Usage

Sling a PR review to a beads-lite polecat:

```bash
gc sling <rig>/gastown-beads-lite.polecat mol-adopt-pr --formula \
  --var pr=https://github.com/org/repo/pull/42
```

With bare integer (uses current repo):

```bash
gc sling <rig>/gastown-beads-lite.polecat mol-adopt-pr --formula --var pr=42
```

Skip Gemini (dual-model mode — Claude + Codex only):

```bash
gc sling <rig>/gastown-beads-lite.polecat mol-adopt-pr --formula \
  --var pr=42 --var skip_gemini=true
```

Inside formula steps, upstream snippets may still show `bd ...`. In a
beads-lite rig session, translate them to:

```bash
BEADS_DIR=<rig-root>/.beads gc gastown-beads-lite bd ...
```

Do not use a global `bd`, normal `gc bd`, Dolt commands, or `bd --db`.

## Workflow

1. **Intake** — Parse PR, fetch metadata, validate scope, checkout branch
2. **Rebase check** — Auto-rebase straightforward conflicts, reject complex ones
3. **Review** — Run `/review-pr` (parallel Claude + Codex + Gemini)
4. **Human gate** — Blocks until maintainer closes the step manually:
   ```bash
   BEADS_DIR=<rig-root>/.beads gc gastown-beads-lite bd close <human-gate-step-id>
   ```
5. **Finalize** — Determine merge path, push, post synthesis comment, wait for CI, merge
6. **Complete** — Clean up refs, update root bead

## Merge Paths

| Path | Condition | Strategy |
|------|-----------|----------|
| A | No maintainer changes | Squash merge |
| B | Maintainer changes + edits enabled | Merge commit (preserves dual authorship) |
| C | Maintainer changes + edits disabled | New PR from maintainer's fork |
| D | Original PR already merged | Follow-up PR for fixups |

## Importing into pack.toml

This pack intentionally does not define standing agents. It provides formulas
for the worker sessions supplied by `gastown-beads-lite-pack`.
