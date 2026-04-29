# Phase N Context: {{PHASE_NAME}}

> **HOW TO USE THIS TEMPLATE**
> Phase 0 creates one copy of this file per implementation phase, named `phase_1_context.md`, `phase_2_context.md`, etc.
> Replace ALL `{{PLACEHOLDERS}}` for each phase before that phase begins.
> Keep this template file — it is required for pipeline improvements (see PIPELINE_ROADMAP.md Feature H).

---

**Phase Goal:** {{PHASE_GOAL}}
> _What is the concrete outcome of this phase? What can the system do at the end that it couldn't before?_

---

## Always Load

- `CLAUDE.md` — execution loop, safety rules, compression protocol
- `AGENTS.md` — directory map
- `2_planning/execution_tasks.md` — your task list (look for Phase {{N}} tasks)

## Architecture Docs to Load for This Phase

Load ONLY the sections relevant to this phase's tasks:

| Document | Sections to Load | Why |
|----------|-----------------|-----|
| `3_architecture/SYSTEM_ARCHITECTURE.md` | {{ARCH_SECTIONS}} | {{WHY}} |
| `3_architecture/API_CONTRACTS.md` | {{API_SECTIONS}} | {{WHY}} |
| `3_architecture/DECISION_LOG.md` | {{ADR_NUMBERS}} | {{WHY}} |

## Knowledge Files by Task Group

> _Load ONLY the files listed for your current task. Drop them from context when you move to the next task group._

| Task Group | Knowledge Files |
|------------|----------------|
| {{TASK_GROUP_1}} | `{{CONCEPT_FILE_1}}`, `{{CONCEPT_FILE_2}}` |
| {{TASK_GROUP_2}} | `{{CONCEPT_FILE_3}}` |
| {{TASK_GROUP_N}} | _(none required)_ |

## Phase-Specific Constraints

- {{CONSTRAINT_1}}
- {{CONSTRAINT_2}}
> _E.g., "All code goes in src/backend/", "Do not modify the database schema", "Frontend only — no backend changes"_

## Validation Criteria (Phase Gate)

This phase is complete when ALL of the following are true:

1. {{VALIDATION_CRITERION_1}}
2. {{VALIDATION_CRITERION_2}}
3. All backend tests pass: `cd src/backend && python -m pytest tests/ -v`
4. All frontend tests pass (if applicable): `cd src/frontend && CI=true npm test`

## Phase Gate Output

When validation passes, write this summary block at the top of the Phase {{N}} section in `execution_tasks.md`:

```markdown
## Phase {{N}}: {{PHASE_NAME}} [COMPLETE]
> {{tasks_done}}/{{tasks_total}} tasks done.
> {{WHAT_WAS_BUILT_SUMMARY}}
> Key files: {{LIST_KEY_FILES}}
> Tests: {{TEST_COUNT}} passing.
> Handoff note for Phase {{N+1}}: {{WHAT_NEXT_PHASE_AGENT_NEEDS_TO_KNOW}}
```

Then commit, tag `phase-{{N}}-complete`, print the PHASE COMPLETE message, and **STOP**.
