---
description: Dry-run the review scope — resolve the target, inventory files, apply frontend exclusion, decide full vs incremental, apply --max-files. Zero LLM calls.
argument-hint: <path|repo-url> [--with-frontend] [--max-files N]
---

You are running the **scope dry-run**. Invoke the `review-methodology` skill
and follow its **P0** rules. Do **zero** LLM/subagent calls — this command must
not dispatch any agent.

Arguments: $ARGUMENTS

Steps:
1. Parse the first token as the target (local path or git URL). Parse
   `--with-frontend` and `--max-files N` (N must be >= 0; reject negative with
   a usage error).
2. Resolve the target: if a local path, use it in place; if a git URL, note
   that a real run would clone into `<workdir>/.repocache/` (do NOT clone here
   — for a URL with no local checkout, report that scope preview needs a local
   path).  `<workdir>` defaults to `.reviewer`.
3. Inventory source files (use Glob).
4. Apply the **frontend-exclusion rule** from the skill (respect
   `--with-frontend`). Backend-language files are never excluded.
5. Decide scope: `full` if no `<workdir>/last-review.json`, else `incremental`
   per the skill's scope rule (changed-since-prior-SHA + reverse-import
   expansion). If git diff cannot be computed, fall back to `full`.
6. Apply the `--max-files N` cap (N=0 ⇒ "dry run, nothing would be evaluated").
7. Print a concise report ONLY (no files written):
   - target, workdir, mode (full/incremental) and the reason
   - total source files, frontend-excluded count
   - final in-scope count and the in-scope file list
   - cached file count (incremental only)
   - whether `--max-files` further capped the set

Never dispatch a subagent. Never write files. This is a cost-free preview.
