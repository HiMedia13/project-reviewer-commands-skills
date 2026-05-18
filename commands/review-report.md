---
description: Re-render the most recent review result to a fresh self-contained HTML report. Zero LLM calls.
argument-hint: [--workdir DIR]
---

You are re-rendering the latest stored review. Zero LLM/subagent calls.

Arguments: `$ARGUMENTS` (optional `--workdir DIR`, default `.reviewer`).

Steps:
1. Read `<workdir>/last-review.json`. If it does not exist, STOP and tell the
   user clearly: "No prior review found in `<workdir>`. Run `/review-project`
   first." Do nothing else.
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
5. Write `<workdir>/output/report-<timestamp>.html` and print its path plus a
   one-line Korean summary: 기술 스택 점수(stack_score) + 코드 품질 종합 (두 축 분리, 합산 금지).

Never dispatch a subagent. The only file written is the HTML report.
