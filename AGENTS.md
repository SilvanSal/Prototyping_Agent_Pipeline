# Agent Routing & Directory Map

This file tells you WHERE to find things. For HOW to act, read CLAUDE.md.

## 1. Boot Sequence

You are a fresh agent with no memory of previous sessions. Follow these steps IN ORDER:

```
Step 0 → Check 0_app_specification/ for user input files.
          - If the folder has files beyond .gitkeep: Phase 0 may need to run.
          - Continue to Step 1 to determine the actual state.

Step 1 → Read CLAUDE.md

Step 2 → Read 2_planning/execution_tasks.md
          - Scan [x] tasks (compressed one-liners) to see what's done
          - Find the FIRST [ ] task — that's your target
          - Note which Phase (N) it belongs to
          - If the first [ ] task is in Phase 0: proceed to Phase 0 before anything else.

Step 3 → Read 2_planning/phases/phase_N_context.md
          ONLY load what that file lists. Nothing else.

Step 4 → Read the concept files in your task's Knowledge: field. ONLY those.

Step 5 → Execute the loop from CLAUDE.md §1.
```

**Fresh start:** If `2_planning/execution_tasks.md` does not exist, you are on a brand new project. Look for `execution_tasks.md` in the current working directory. Your first task is 1.1.1 — see CLAUDE.md §9 for its full definition.

**Resuming after a phase gate:** The human will say "Continue from Phase N". Start at Step 2. Prior phases are already compressed in execution_tasks.md with handoff notes written by the outgoing agent.

## 2. Directory Map

> These paths exist after Task 1.1.1 (defined in CLAUDE.md §9) creates the structure.

```
/SPEC.md                                    ← Master milestones and acceptance criteria
/AGENTS.md                                  ← You are here. Routing only.
/CLAUDE.md                                  ← All behavioral rules, execution loop, safety
/0_app_specification/                       ← USER INPUT DROP ZONE (read in Phase 0)
  README.md                                 ← Instructions for what to drop here
  research_papers/                          ← PDFs, papers, prior art (user provides)
  spec_docs/                                ← .md requirement specs (user provides)
  inputs/                                   ← Sample data, schemas, configs (user provides)
/1_domain_knowledge/
  index.md                                  ← Index of all concept files
  concepts/
    _TEMPLATE.md                            ← Template for creating new concept files
    ERROR_REGISTRY.md                       ← Known bugs: signature → root cause → fix
    [topic concept files].md                ← Verified code patterns per domain topic (Phase 0 generates)
/2_planning/
  execution_tasks.md                        ← Task list with status markers ([ ] [x] [!] [S])
  implementation_plan.md                    ← Technical design document (Phase 0 fills in)
  phases/
    phase_0_context.md                      ← What to do in Phase 0 (Research)
    phase_N_context_TEMPLATE.md             ← Template for Phase 1+ context files
    phase_1_context.md                      ← What to load for Phase 1 (Phase 0 generates)
    phase_2_context.md                      ← What to load for Phase 2 (Phase 0 generates)
    ...                                     ← One per phase (Phase 0 generates all)
/3_architecture/
  SYSTEM_ARCHITECTURE.md                    ← Layer boundaries, tech stack, data flow (Phase 0 fills in)
  API_CONTRACTS.md                          ← Endpoint specs, request/response schemas (Phase 0 fills in)
  DECISION_LOG.md                           ← Architecture decision records / ADRs (Phase 0 writes initial)
/src/
  backend/                                  ← Python (FastAPI) — skeleton until Phase 1
    app/services/                           ← Business logic
    tests/                                  ← pytest
  frontend/                                 ← TypeScript (React) — skeleton until frontend phase
    src/                                    ← Components, services
    test/                                   ← vitest / Jest
```

## 3. Knowledge Routing

All concept files live in `1_domain_knowledge/concepts/`. When a task's `Knowledge:` field names a file (e.g. `PIPELINE_RETROSPECTIVE.md`), prepend that path: `1_domain_knowledge/concepts/PIPELINE_RETROSPECTIVE.md`.

1. Read the concept file BEFORE writing any code.
2. Use the verified code patterns from the file.
3. If the file doesn't exist → see CLAUDE.md §5 (Active Learning Protocol).
4. If you solve a non-trivial bug → see CLAUDE.md §6 (Error Registry Protocol).
