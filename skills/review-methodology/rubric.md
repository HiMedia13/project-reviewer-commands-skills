# Review Rubric

The review is **qualitative**. It never evaluates whether code builds, runs, or
passes tests. It evaluates intent and fit.

## Primary deliverable — project-level tech-stack-fit

`tech_assessment` object (produced by `review-techstack`, verified by
`review-evaluator`):

```json
{
  "purpose": "1-2 sentence inference of what the project is for, from README/docs/dir structure/manifests",
  "stack": [
    {
      "tech": "name of language/framework/major library/build tool",
      "role": "what it does in this project",
      "used_well": "good | mixed | poor",
      "purpose_fit": "fit | questionable | misfit",
      "rationale": "why this judgment",
      "evidence": "concrete code/file basis"
    }
  ],
  "stack_verdict": "overall narrative summary",
  "stack_score": 0
}
```

`stack_score` is an integer 0-100. An empty `stack` is valid.

## Secondary deliverable — per-file findings (3 criteria)

Three per-file criteria: `library`, `eng`, `deadcode`. (`techstack` is the
project-level criterion above, not per-file.)

| Criterion | Question |
|---|---|
| `library` | Was each used library used as its designers intended? |
| `eng` | Over- or under-engineering relative to the problem size? |
| `deadcode` | Amount and location of unreachable / unused code? |

Per-file finding row (one row per file per criterion):

```json
{
  "file_path": "...",
  "criterion": "library | eng | deadcode",
  "findings": [
    { "severity": "low | medium | high",
      "location": "path:line",
      "evidence": "concrete basis",
      "msg": "the point being made" }
  ],
  "criterion_score": 0
}
```

After verification, `review-evaluator` adds to each row:
`"verified": true|false` and `"verify_note": "verification basis or web-search summary"`.

`criterion_score` and `stack_score` are integers **0–100, higher = better**:

- 90–100: exemplary — idiomatic, well-suited, no real concerns.
- 70–89: solid, with minor improvement points.
- 40–69: noticeable problems (medium-severity misuse / structural issues).
- 0–39: serious misuse or structural problems (high-severity findings).

A file or tech with a verified high-severity finding MUST score below 50. Do
not default to high scores — justify the score from the findings.

**Depth requirement (tech-stack).** Judge each major technology by its core
configuration correctness, not presence. Examples: LangChain/agent frameworks →
are tools/chains/agents defined with proper schemas and output parsing;
APScheduler/schedulers → job/trigger definitions, timezone, misfire/coalesce,
jobstore; ML/AI models → training-data provenance, evaluation metrics and their
meaning, train/serve separation — only when documented in the repo; if absent,
say "근거 없음", neither fabricate nor over-penalize beyond "undocumented".
Declared-only, no deep evidence → not `good`. No JSON fields are added by this.

## Severity

- `low` — minor, stylistic, low-impact.
- `medium` — meaningful design/usage concern.
- `high` — significant misuse or structural problem.

## Scoring formula (synthesis)

Only rows with `verified == true` contribute. The two deliverables are scored
on **separate axes — never average them together**:

**Primary (headline):** the project-level tech-stack-fit score is the verified
`stack_score` (0–100), reported on its own as the headline. It is NEVER mixed
into the secondary score.

**Secondary (code quality):** over the three per-file criteria ONLY —
`library`, `eng`, `deadcode`:

- Per-criterion score = mean of `criterion_score` over that criterion's
  verified rows. No verified rows for a criterion → `null`.
- Code-quality overall = mean of the present per-criterion **rounded**
  averages. All three absent → `null`.
- Round at both the per-criterion level and again at the overall level.
- `techstack` is NOT a secondary criterion and never enters this mean.
