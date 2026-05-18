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

## Secondary deliverable — per-file 4-criteria findings

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

`criterion_score` is an integer 0-100 (same scale as `stack_score`).

## Severity

- `low` — minor, stylistic, low-impact.
- `medium` — meaningful design/usage concern.
- `high` — significant misuse or structural problem.

## Scoring formula (synthesis)

Only rows with `verified == true` contribute.

- Per-criterion score = mean of `criterion_score` over verified rows for that
  criterion. If no verified rows for a criterion → `null`.
- Overall score = mean of the **rounded** per-criterion averages that are
  present. If all criteria absent → `null`.
- Round at both the per-criterion level and again at the overall level.
- Criteria set for synthesis: `library`, `eng`, `deadcode`, `techstack`
  (`techstack` per-criterion score comes from the verified `stack_score`).
