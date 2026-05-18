---
description: Re-render a stored review result to a fresh self-contained HTML report. Optional target selects the repo; otherwise the most recent. Zero LLM calls.
argument-hint: [target] [--workdir DIR]
---

You are re-rendering a stored review (the given `target`'s, else the most recent). Zero LLM/subagent calls.

Arguments: `$ARGUMENTS` (optional first positional `target` = the repo URL or
local path that was reviewed; optional `--workdir DIR`, default `.reviewer`).

Steps:
1. Select the stored review under `<workdir>`:
   - If `target` is given, compute `slug` per the review-methodology Per-target
     isolation rule and read `<workdir>/<slug>/last-review.json`.
   - Else pick the most recently modified `<workdir>/*/last-review.json`; print
     which repo was chosen (its stored `repo` field + the slug folder).
   - Legacy fallback: if no `<workdir>/*/last-review.json` exists but a flat
     `<workdir>/last-review.json` does, use it and note it is a legacy
     pre-v0.4.0 file.
   - If none is found, STOP and tell the user clearly: "No prior review found
     in `<workdir>`. Run `/review-project` first." Do nothing else.
2. Load `skills/review-methodology/report-template.html` from this plugin.
3. Apply the scoring formula from `review-methodology` / `rubric.md` to the
   stored verified rows (only `verified == true` rows contribute).
4. HTML-escape every LLM-produced string (purpose, verdict, rationale,
   evidence, msg, file paths). Replace the `{{...}}` tokens in the template:
   - `{{REPO}} {{COMMIT}} {{MODE}}` from the stored metadata; `{{GENERATED_AT}}`
     is the current re-render timestamp (now), not the original run time.
   - `{{PURPOSE}} {{STACK_VERDICT}} {{STACK_SCORE}}` from `tech_assessment`.
   - `{{STACK_ROWS}}`: one `<tr>` per stack entry (escaped cells).
   - `{{CRITERIA_ROWS}}`: one `<tr>` per per-file criterion — `library`,
     `eng`, `deadcode` ONLY (NOT `techstack`) — with its synthesized score
     (`—` when null).
   - `{{OVERALL_SCORE}}`: secondary code-quality overall = mean of the present
     rounded `library`/`eng`/`deadcode` per-criterion scores (`N/A` when null).
     `{{STACK_SCORE}}` is the separate primary headline (verified
     `stack_score`); never blend the two.
   - `{{FINDINGS_BLOCKS}}`: one `<details>` per file/criterion row; each
     finding as a line with a `sev-<severity>` class.
   Apply the Localization display mapping from the review-methodology skill (enum/criterion/mode → Korean) for displayed cells; the stored last-review.json is unchanged (English tokens).
5. Write the report next to the selected `last-review.json`
   (`<workdir>/<slug>/output/report-<timestamp>.html`, or
   `<workdir>/output/...` when a legacy flat file was used) and print its
   path plus a one-line Korean summary: 기술 스택 점수(stack_score) + 코드 품질
   종합 (두 축 분리, 합산 금지).

Never dispatch a subagent. The only file written is the HTML report.
