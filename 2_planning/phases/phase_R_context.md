# Phase R Context: Pipeline Retrospective

**Phase Goal:** Analyze this project's execution log to identify pipeline flaws, write a
structured improvement report, and apply high-confidence patches to CLAUDE.md and pipeline
template files — so future projects running this pipeline need fewer self-correct attempts
and fewer blocked tasks.

---

## Always Load

- `CLAUDE.md` — execution loop rules (you may modify this file in Phase R)
- `AGENTS.md` — directory map
- `2_planning/execution_tasks.md` — full task history including [!] blocked tasks

## Files to Read for Evidence (load at start of Phase R, before any task)

| File | What to look for |
|------|-----------------|
| `1_domain_knowledge/concepts/ERROR_REGISTRY.md` | Every logged error — will be classified P-* or D-* |
| `1_domain_knowledge/index.md` | Which concept files exist — gaps here are P-MK candidates |
| `2_planning/execution_tasks.md` | [!] blocked tasks, handoff notes that mention unresolved issues |

## Knowledge Files by Task

| Task | Knowledge Files |
|------|----------------|
| R.1.1 (git signals) | `PIPELINE_RETROSPECTIVE.md` |
| R.1.2 (error triage) | `PIPELINE_RETROSPECTIVE.md` |
| R.1.3 (blocked tasks) | `PIPELINE_RETROSPECTIVE.md` |
| R.2.1 (write report) | `PIPELINE_RETROSPECTIVE.md` |
| R.2.2 (apply patches) | `PIPELINE_RETROSPECTIVE.md` |

> All Phase R tasks share the same single knowledge file. Load it once at phase start and keep it in context for the entire phase.

---

## Phase-Specific Constraints

- **You are analyzing the pipeline, not the product.** Do not add features, fix bugs in
  the app, or refactor application code. Focus exclusively on CLAUDE.md, phase context
  files, and concept files in `1_domain_knowledge/concepts/`.

- **Empty ERROR_REGISTRY is valid data.** If no errors were logged, report "no P-* errors
  found" and check git stress signals for silent self-correction not captured in the registry.

- **Do not invent errors.** Only classify what is actually documented in ERROR_REGISTRY.md
  or in [!] tasks. Do not speculate about errors that might have occurred.

- **Be specific about proposed patches.** Every recommendation must quote the exact old text
  and exact new text. Vague suggestions ("improve task descriptions") are not actionable.

- **Patch application is scoped.** In R.2.2, only apply patches to pipeline infrastructure
  files — never to `src/`, `README.md`, or `SPEC.md`.

---

## Validation Criteria (Phase Gate)

Phase R is complete when ALL of the following are true:

1. `2_planning/PIPELINE_IMPROVEMENT_REPORT.md` exists with all sections from the template
   in PIPELINE_RETROSPECTIVE.md §4.
2. Every P-* error has a corresponding numbered Recommendation.
3. Every HIGH / APPLY NOW recommendation has been acted on (applied or downgraded with reason).
4. Applied patches leave tests passing: `cd src/backend && python -m pytest tests/ -v`.
5. `git log --oneline` shows at least one Phase R commit.

---

## Phase Gate Output

When validation passes, write this summary block at the top of the Phase R section in
`execution_tasks.md`:

```markdown
## Phase R: Pipeline Retrospective [COMPLETE]
> [N]/[N] tasks done.
> [N] P-* errors found, [N] patches applied, [N] items for human review.
> Report: 2_planning/PIPELINE_IMPROVEMENT_REPORT.md
> Handoff note: Pipeline updated. See Applied Patches table for changes made to CLAUDE.md.
```

Then commit and tag, and print the PROJECT COMPLETE message from CLAUDE.md §11.

```bash
git add CLAUDE.md 2_planning/ 1_domain_knowledge/ 3_architecture/
git commit -m "Phase R COMPLETE: Pipeline retrospective — [N] patches applied"
git tag phase-R-complete
```
