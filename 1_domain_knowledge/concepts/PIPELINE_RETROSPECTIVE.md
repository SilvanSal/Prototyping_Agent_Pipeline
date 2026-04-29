# Pipeline Retrospective — Analysis Framework

> Used exclusively by the Phase R agent. Load this file for all Phase R tasks.

---

## §1 Purpose

Guide the retrospective agent in:
1. Classifying every error and blocked task as pipeline-caused or domain-caused.
2. Extracting stress signals from git history.
3. Writing a structured `PIPELINE_IMPROVEMENT_REPORT.md`.
4. Deciding which improvements to apply directly vs. flag for human review.

---

## §2 Error Classification Framework

For each entry in ERROR_REGISTRY.md and each [!] task, ask: **"Could a better pipeline have prevented this?"**

| Code | Category | Pipeline flaw | Example |
|------|----------|---------------|---------|
| **P-MK** | Missing Knowledge file | A library was used without a concept file, causing hallucinated method names or wrong patterns | `requests.get()` called with wrong arg order because no HTTP_Client.md existed |
| **P-BA** | Bad Instruction | An explicit rule in CLAUDE.md caused the agent to behave incorrectly or wastefully | §3a said to compress tasks but didn't say WHEN, causing inconsistent compression |
| **P-PO** | Phase Ordering | A task depended on an artifact from a later phase | Frontend task needed backend schema defined in a later Phase 1 section |
| **P-MD** | Missing `Depends:` field | Task ran before its dependency completed because the Depends: line was missing | Task 2.3 built on 2.2 output but had no `Depends: 2.2` — ran out of order |
| **P-AT** | Ambiguous Task description | Task description was vague; agent built wrong thing and had to redo it | "Add auth" with no spec of which auth method |
| **P-CG** | Context Gap | `phase_N_context.md` failed to list a knowledge file or architecture doc the agent needed | Agent re-derived the DB schema from scratch because API_CONTRACTS.md wasn't listed |
| **P-KB** | Knowledge file wrong/stale | Concept file existed but contained incorrect or outdated patterns | Concept file showed SQLAlchemy 1.x patterns; project uses 2.x |
| **D-IC** | Inherent Complexity | Error was unavoidable given domain difficulty; no pipeline change would have helped | OAuth flow required multi-step state machine not predictable from specs |
| **D-EX** | External/Environment quirk | Error caused by OS, library version, or environment behaviour not predictable from docs | `datetime.utcnow()` deprecation warning from Python 3.12 |

**Decision rule:**
- P-* errors → generate a Recommendation in the report.
- D-* errors → note in report as "expected complexity" with no pipeline change.

When in doubt, ask: "If I wrote one more sentence in CLAUDE.md or created one more concept file, would this error have been avoided?" If yes → P-*. If no → D-*.

---

## §3 Git Stress Signal Analysis

Run these commands and record output in the report:

```bash
# Full commit log (one line each)
git log --oneline

# Commits that mention BLOCKED
git log --all --grep="BLOCKED" --oneline

# Count commits per task ID — sort by frequency descending
# Tasks with count > 1 had at least one self-correct attempt
git log --oneline | grep -oP "(Task|Phase) [\d]+\.[\d]+\.[\d]+" | sort | uniq -c | sort -rn
```

Interpretation guide:

| Commits on a task | Meaning | Signal strength |
|-------------------|---------|-----------------|
| 1 | Clean execution | Normal |
| 2 | One self-correct attempt | Low — watch for patterns |
| 3 | Two self-correct attempts | Medium — near the retry limit |
| 4+ | Hit retry ceiling or required human fix | **HIGH — investigate** |

Also look at the overall pattern:
- Many tasks with 2 commits in the same phase → that phase's context file is missing something.
- BLOCKED commits concentrated in one task type → that task category needs a concept file.
- Self-correct spikes on test tasks → test setup (conftest, fixtures) may need a concept file.

---

## §4 PIPELINE_IMPROVEMENT_REPORT.md Template

Copy this template exactly when creating the report. Replace all `[...]` placeholders.

```markdown
# Pipeline Improvement Report — [Project Name] — [Date YYYY-MM-DD]

## Executive Summary

[2–3 sentences: what was built, overall pipeline health, most significant issues found.]

**Pipeline Health:** [Healthy ✓ / Needs Improvement ⚠ / Critical Issues ✗]

| Metric | Count |
|--------|-------|
| Pipeline-caused (P-*) errors | N |
| Domain-caused (D-*) errors | N |
| Blocked tasks [!] | N |
| Tasks with 3+ commits (self-correct ceiling hit) | N |
| High-priority recommendations | N |
| Patches applied in this run | N |

---

## Stress Signals

[Git log summary. Paste the output of the three git commands from §3.
List tasks with 2+ commits and explain what self-correction was needed.]

### Tasks with multiple commits
| Task ID | Commit count | What went wrong |
|---------|-------------|-----------------|
| ... | ... | ... |

### BLOCKED commits
[List or "None"]

---

## Error Triage

[One row per error in ERROR_REGISTRY.md plus any [!] task not already in the registry.]

| Error / Blocked Task | Classification | Root cause summary | Recommendation # |
|----------------------|---------------|-------------------|-----------------|
| ... | P-MK | Missing concept file for library X | R-001 |
| ... | D-EX | Python 3.12 deprecation warning | — |

---

## Blocked Tasks

[One entry per [!] task in execution_tasks.md.]

| Task | Blocking reason | Category | Recommendation # |
|------|----------------|----------|-----------------|
| ... | Requires external account signup | Safety boundary — correct behaviour | — |

---

## Recommendations

[One recommendation per P-* root cause. Number sequentially R-001, R-002, ...]

### R-001: [Short title]

**Problem:** [What went wrong in concrete terms]
**Evidence:** [Specific commit, error message, or task reference]
**Root cause:** [P-MK / P-BA / P-PO / P-MD / P-AT / P-CG / P-KB]
**Proposed fix:**
> File: `[path/to/file.md]`
> Old text: `[exact old text, or "N/A — new addition"]`
> New text: `[exact new text to replace it with]`

**Priority:** HIGH / MEDIUM / LOW
**Action:** APPLY NOW / HUMAN REVIEW

*(APPLY NOW criteria: clear, low-risk, no restructuring of phase ordering or safety rules.)*
*(HUMAN REVIEW criteria: restructures phases, changes retry limits, touches §10, or requires judgment.)*

---

### R-002: [Short title]
...

---

## Applied Patches

[Record every change actually made to pipeline files during R.2.2.
Leave empty if no patches were applied.]

| File | Change description | Recommendation # |
|------|-------------------|-----------------|
| `CLAUDE.md` | Added `Depends: X.Y.Z` to task ... | R-001 |

---

## Recommendations for Human Review

[Summarise all HUMAN REVIEW items so the human can find them quickly.]

| # | Title | Priority | Why human review needed |
|---|-------|----------|------------------------|
| R-003 | ... | HIGH | Restructures phase ordering — risk of dependency cascade |
```

---

## §5 Patch Application Rules

### Apply directly (mark Action: APPLY NOW) when the fix is:
- Adding a missing `Depends:` field to a task that clearly requires it
- Adding a new concept file for a library the project actually used
- Adding a missing knowledge file reference to a `phase_N_context.md`
- Correcting a factual error in CLAUDE.md (wrong command, wrong path, typo in code block)
- Updating a stale concept file with the correct library version's patterns

### Do NOT apply directly (mark Action: HUMAN REVIEW) when the fix:
- Restructures phase ordering (risk of breaking downstream dependency chains)
- Changes the self-correct attempt limit (§1 step 5)
- Removes or weakens a safety boundary rule (§10)
- Changes how git commits or tags are structured (§1 step 6)
- Requires judgment about overall project architecture
- Touches the Phase Gate Behavior rules in §1
- Is uncertain — if you are not 100% confident the patch is correct, flag it

### After applying a patch:
1. Re-run tests to confirm the patch didn't break anything:
   ```bash
   cd src/backend && python -m pytest tests/ -v
   cd src/frontend && CI=true npm test
   ```
2. Record the change in the "Applied Patches" table of PIPELINE_IMPROVEMENT_REPORT.md.
3. Commit:
   ```bash
   git add CLAUDE.md 2_planning/ 1_domain_knowledge/
   git commit -m "Task R.2.2: Apply pipeline patch — [short description]"
   ```
