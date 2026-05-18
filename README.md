# project-reviewer-commands-skills

A Claude Code **plugin** that ports the Project-Reviewer qualitative-review
methodology natively (no Python app). It does **not** build/run/test code — it
asks what tech was used, whether each tech was used as designed, and whether it
fits the project's purpose.

> **v0.3.0** — fixed the 0–100 score scale and separated the tech-stack score
> (headline) from the code-quality score (3 criteria) so they are no longer
> averaged together; stricter evidence-based tech-stack judgment; report table
> no longer overflows. (v0.2.0: Korean output; stored JSON stays English.)

## Installation

This repo is a self-hosted Claude Code plugin marketplace (not a public
registry — only people who know the repo can install it; visibility follows
the GitHub repo's visibility).

In a Claude Code session:

```
/plugin marketplace add HiMedia13/project-reviewer-commands-skills
/plugin install project-reviewer
/reload-plugins
```

Local checkout instead of GitHub:

```
/plugin marketplace add D:\dev\project-reviewer-commands-skills
/plugin install project-reviewer
/reload-plugins
```

**Updating to a newer version** (the installed copy does not auto-update):

```
/plugin marketplace update project-reviewer-marketplace
/plugin update project-reviewer
/reload-plugins
```

## Commands

- `/review-project <path|repo-url> [--max-files N] [--with-frontend] [--force] [--workdir DIR]`
  Full pipeline: scanner → 4 criteria in parallel → evaluator → HTML report +
  terminal summary. Spends LLM tokens.
- `/review-scope <path|repo-url> [--with-frontend] [--max-files N]`
  Cost-free dry run: what would be evaluated and the full/incremental decision.
- `/review-report [--workdir DIR]`
  Cost-free: re-render the most recent result to a fresh HTML report.

## Usage

Recommended first run (check cost before spending tokens):

```
/review-scope D:\path\to\repo                  # 0-cost: preview what gets evaluated
/review-project D:\path\to\repo --max-files 1  # low-cost: validate the pipeline
/review-project D:\path\to\repo                # full qualitative review
/review-report                                 # 0-cost: re-render last result
```

Flags: `--with-frontend` (include UI files; backend files are never excluded
either way) · `--force` (ignore cache, full re-eval) · `--max-files N`
(cap evaluated files; `0` = dry run) · `--workdir DIR` (work dir, default
`.reviewer`).

Outputs (under the target's `<workdir>`, default `.reviewer`):

- `output/report-<ts>.html` — self-contained Korean HTML report (no external
  requests), leading with the tech-stack-fit assessment.
- terminal summary in Korean — tech-stack-fit score (stack_score / 100) and
  verdict first; then code-quality scores for library / engineering / dead-code
  and their mean. The two axes are never combined.
- `last-review.json` — latest run only (drives `/review-report` and
  incremental re-runs); JSON keys/enums are English by design.

## How it works

The primary deliverable is a project-level tech-stack-fit assessment; per-file
findings (`library`, `eng`, `deadcode`) are secondary. Model
mapping: scanner + 4 agents in parallel (tech-stack-fit + 3 per-file
criteria) on `haiku` (cheap bulk), evaluator on `sonnet` (verifies the
primary deliverable). Results persist as `<workdir>/last-review.json`
(latest run only) and `<workdir>/output/report-<ts>.html`. Default workdir
`.reviewer`.

## Manual verification checklist

No automated harness (this is plugin config, not application code). Verify:

1. `python -c "import json; json.load(open('.claude-plugin/plugin.json'))"` — manifest valid.
2. `/review-scope` on a small local repo → prints inventory + scope, dispatches **no** subagent.
3. `/review-project <small-repo> --max-files 1` → completes P0–P4, writes a self-contained HTML that opens with no external requests; terminal summary LEADS with the tech-stack-fit headline.
4. `/review-project <repo> --max-files 0` → dry run, no subagents dispatched, empty report rendered.
5. `/review-report` with no prior run → clear, actionable error.
6. `/review-report` after a successful run → identical HTML re-rendered, zero LLM calls.
7. `--with-frontend` flips frontend files into scope; backend files never excluded either way.

## Spec & plan

- Spec: `docs/superpowers/specs/2026-05-18-project-reviewer-plugin-design.md`
- Plan: `docs/superpowers/plans/2026-05-18-project-reviewer-plugin.md`
