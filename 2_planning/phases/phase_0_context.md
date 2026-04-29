# Phase 0 Context: Research & Specification

**Phase Goal:** Read all inputs from `0_app_specification/`, extract requirements, and generate all planning and architecture artifacts that Phase 1 needs to begin.

**This is the only phase where you READ instead of BUILD.** No source code is written in Phase 0.

---

## ⚠ Before You Start: Two Pre-Checks

**Check 1 — Git identity.** Before making any commit, verify git user is set:
```bash
git config user.email   # if blank, run: git config user.email "agent@pipeline.local"
git config user.name    # if blank, run: git config user.name "Pipeline Agent"
```
If identity is missing, `git commit` will fail with "Author identity unknown." Task 0.0.1 handles this.

**Check 2 — User input.**

Run task **0.1.1** first. If `0_app_specification/` contains only `.gitkeep` files (no user content), **print the ACTION REQUIRED message from 0.1.1 and STOP immediately.** Do not attempt requirement extraction on empty folders.

If `spec_docs/` has files but `research_papers/` and `inputs/` are empty — that is fine. Continue with what you have.

---

## Always Load

- `CLAUDE.md` — execution loop, safety rules, compression protocol
- `AGENTS.md` — directory map
- `SPEC.md` — fill in placeholders as you complete tasks
- `2_planning/execution_tasks.md` — your task list for this phase

## Documents to Read First (before any task)

```
0_app_specification/
├── README.md                       ← Drop-zone instructions (skip — for users, not agents)
├── research_papers/                ← Read ALL files here (skip if only .gitkeep)
├── spec_docs/                      ← Read ALL files here (REQUIRED — at least one file needed)
└── inputs/                         ← Read ALL files here (skip if only .gitkeep)
```

## Documents to Generate (task by task)

| Task | Output |
|------|--------|
| 0.2.2 | `1_domain_knowledge/concepts/{{TERM}}.md` (one per domain term) |
| 0.3.1 | `SPEC.md` (filled in) |
| 0.3.2 | `3_architecture/SYSTEM_ARCHITECTURE.md` (tech stack + layer diagram) |
| 0.3.3 | `3_architecture/DECISION_LOG.md` (initial ADRs) |
| 0.3.4 | `3_architecture/API_CONTRACTS.md` (endpoint groups + 2+ specs each) |
| 0.3.5 | `2_planning/implementation_plan.md` (filled in) |
| 0.3.6 | `2_planning/phases/phase_1_context.md` … `phase_N_context.md` |
| 0.3.7 | Phase 1 task list in `execution_tasks.md` |
| 0.3.8 | Phase 2–N task stubs in `execution_tasks.md` |

## Knowledge Files for This Phase

No concept files are required before Phase 0 begins — you are creating them.

Use `1_domain_knowledge/concepts/_TEMPLATE.md` as the format for every concept file you write.

## Constraints

- **Do NOT write any source code during Phase 0.** `src/` stays empty.
- **Do NOT skip a domain term** if it appears in two or more task descriptions. Create a concept file.
- **Max 5 concept files loaded at once** (CLAUDE.md §8). Create, write, and close each before moving on.
- Follow the Active Learning Protocol (CLAUDE.md §5) for all concept file creation.

## Validation Criteria (Phase Gate)

Phase 0 is complete when ALL of the following are true:

1. `SPEC.md` has no remaining `{{PLACEHOLDER}}` strings.
2. `1_domain_knowledge/index.md` lists at least one domain-specific concept file.
3. `3_architecture/SYSTEM_ARCHITECTURE.md` has a named technology for every layer.
4. `3_architecture/DECISION_LOG.md` has at least 3 ADRs.
5. `3_architecture/API_CONTRACTS.md` has at least 2 endpoint groups with concrete specs.
6. `2_planning/execution_tasks.md` has Phase 1 tasks with `Test:` and `Knowledge:` fields.
7. Every `Knowledge:` field in Phase 1 tasks points to a file that exists.

## Phase Gate Output

When validation passes, write this summary block at the top of the Phase 0 section in `execution_tasks.md`:

```markdown
## Phase 0: Research & Specification [COMPLETE]
> {{N}}/{{N}} tasks done.
> Project: {{PROJECT_NAME}} — {{ONE_SENTENCE_DESCRIPTION}}
> Stack: {{TECH_SUMMARY}}
> Phases planned: {{PHASE_COUNT}} implementation phases + Integration & Polish
> Key concept files: {{LIST_CONCEPT_FILES}}
> Handoff note for Phase 1: {{WHAT_PHASE_1_AGENT_NEEDS_TO_KNOW}}
```

Then commit, tag `phase-0-complete`, print the PHASE COMPLETE message, and **STOP**.
