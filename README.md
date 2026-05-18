# project-reviewer-commands-skills

A Claude Code **plugin** that ports the Project-Reviewer qualitative-review
methodology natively (no Python app). It does **not** build/run/test code — it
asks what tech was used, whether each tech was used as designed, and whether it
fits the project's purpose.

## Commands

- `/review-project <path|repo-url> [--max-files N] [--with-frontend] [--force] [--workdir DIR]`
  Full pipeline: scanner → 4 criteria in parallel → evaluator → HTML report +
  terminal summary. Spends LLM tokens.
- `/review-scope <path> [--with-frontend] [--max-files N]`
  Cost-free dry run: what would be evaluated and the full/incremental decision.
- `/review-report [--workdir DIR]`
  Cost-free: re-render the most recent result to a fresh HTML report.

## How it works

The primary deliverable is a project-level tech-stack-fit assessment; per-file
4-criteria findings (`library`, `eng`, `deadcode`) are secondary. Model
mapping: scanner + 4 criteria on `haiku` (cheap bulk), evaluator on `sonnet`
(verifies the primary deliverable). Results persist as `<workdir>/last-review.json`
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
