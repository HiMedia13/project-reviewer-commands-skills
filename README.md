# project-reviewer-commands-skills

A Claude Code **plugin** that ports the Project-Reviewer qualitative-review
methodology natively (no Python app). It does **not** build/run/test code — it
asks what tech was used, whether each tech was used as designed, and whether it
fits the project's purpose.

> **v0.5.0** — the report now includes an independent **architecture axis**: a
> new `review-architecture` agent infers the component structure and the report
> draws a self-contained clickable diagram (click a component for its internal
> composition, the inferred rationale, and its score). Architecture is scored
> on its own and never blended into the tech-stack or code-quality scores.

> **v0.4.0** — per-target isolation: every run is stored under
> `<workdir>/<slug>/` (slug derived from the repo URL/path) so reviewing
> multiple repos from one folder no longer collides. Default scope is now
> **backend + AI only**; frontend is excluded via manifest signals and
> included only with `--with-frontend`. Tech-stack rows now assess
> per-technology configuration depth, not mere presence. (v0.3.0: fixed the
> 0–100 score scale and separated the tech-stack score from the code-quality
> score; v0.2.0: Korean output; stored JSON stays English.)

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
  Full pipeline: scanner → 5 agents in parallel → evaluator → HTML report +
  terminal summary. Spends LLM tokens.
- `/review-scope <path|repo-url> [--with-frontend] [--max-files N]`
  Cost-free dry run: what would be evaluated and the full/incremental decision.
- `/review-report [target] [--workdir DIR]`
  Cost-free: re-render a stored result (the given `target`'s, else the most
  recent) to a fresh HTML report.

## Usage

Recommended first run (check cost before spending tokens):

```
/review-scope D:\path\to\repo                  # 0-cost: preview scope + slug
/review-project D:\path\to\repo --max-files 1  # low-cost: validate the pipeline
/review-project D:\path\to\repo                # full review (backend + AI only)
/review-project D:\path\to\repo --with-frontend  # include the frontend too
/review-report                                 # 0-cost: re-render newest repo
/review-report D:\path\to\repo                 # 0-cost: re-render that repo
```

Default scope is backend + AI only; frontend is detected via manifest signals
(`package.json` frontend deps / bundler config) and excluded unless
`--with-frontend` is passed. Backend / AI files (incl. `.ipynb`) are never
excluded. Flags: `--with-frontend` (also evaluate the frontend) · `--force`
(ignore cache, full re-eval) · `--max-files N` (cap evaluated files; `0` = dry
run) · `--workdir DIR` (work dir, default `.reviewer`).

Outputs (under `<workdir>/<slug>/`, where `slug` is derived from the
target repo URL/path; `<workdir>` default `.reviewer`):

- `output/report-<ts>.html` — self-contained Korean HTML report (no external
  requests), leading with the tech-stack-fit assessment and including a
  clickable architecture diagram (independent axis).
- terminal summary in Korean — tech-stack-fit score (stack_score / 100) and
  verdict first; then code-quality scores for library / engineering / dead-code
  and their mean; then the architecture score (arch_score).
  The three axes are never combined.
- `last-review.json` — this repo's latest run only (per-slug; drives
  `/review-report` and incremental re-runs); JSON keys/enums are English by
  design.

## How it works

The primary deliverable is a project-level tech-stack-fit assessment; per-file
findings (`library`, `eng`, `deadcode`) are secondary; the architecture axis is
independent (its own score, never blended). Model mapping: scanner, then
5 agents in parallel — tech-stack-fit + 3 per-file criteria on `haiku` (cheap
bulk) and `review-architecture` on `sonnet` — then evaluator on `sonnet`
(verifies the primary deliverable, including per-technology configuration depth
and the architecture axis). Results persist as
`<workdir>/<slug>/last-review.json` (latest run per repo only) and
`<workdir>/<slug>/output/report-<ts>.html`. Default workdir `.reviewer`.

## Manual verification checklist

No automated harness (this is plugin config, not application code). Verify:

1. `python -c "import json; json.load(open('.claude-plugin/plugin.json'))"` — manifest valid.
2. `/review-scope` on a small local repo → prints inventory + scope, dispatches **no** subagent.
3. `/review-project <small-repo> --max-files 1` → completes P0–P4, writes a self-contained HTML that opens with no external requests; terminal summary LEADS with the tech-stack-fit headline and also prints the code-quality and architecture axes (three separate axes, never blended).
4. `/review-project <repo> --max-files 0` → dry run, no subagents dispatched, empty report rendered.
5. `/review-report` with no prior run → clear, actionable error.
6. `/review-report` after a run → re-renders that repo (bare command picks the most recently reviewed repo's `<workdir>/<slug>/last-review.json`; `/review-report <repo>` targets a specific one), zero LLM calls.
7. Frontend exclusion is manifest-based by default (`package.json` / bundler signals); `--with-frontend` includes the frontend; backend/AI files (incl. `.ipynb`) are never excluded either way.
8. The report's 아키텍처 section renders a clickable diagram; clicking a component shows its internal composition, rationale, and score; a component whose text contains `</script>` renders as literal text (no execution); absent architecture → "아키텍처 정보 없음", report still renders.

## Spec & plan

- v0.4.0 spec: `docs/superpowers/specs/2026-05-18-v0.4.0-isolation-frontend-depth-design.md`
- v0.4.0 plan: `docs/superpowers/plans/2026-05-18-v0.4.0-isolation-frontend-depth.md`
- Original spec: `docs/superpowers/specs/2026-05-18-project-reviewer-plugin-design.md`
- Original plan: `docs/superpowers/plans/2026-05-18-project-reviewer-plugin.md`
