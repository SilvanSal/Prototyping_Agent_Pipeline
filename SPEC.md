# Project Specification — {{PROJECT_NAME}}

## Goal

{{PROJECT_GOAL}}
> _One paragraph: what this project does and why it exists. Phase 0 fills this in._

## Autonomous Mandate

You are in fully autonomous mode within each phase. Execute the loop defined in `CLAUDE.md` until all tasks in the current phase are complete. **Do not pause for human approval between tasks within a phase.**

### Phase Gate Behavior

When all tasks in Phase N are `[x]` and validation passes:
1. Write a PHASE_COMPLETE note in execution_tasks.md, commit, and tag.
2. Print the phase-complete message defined in CLAUDE.md §1.
3. **STOP and wait.** The human will read the message, then `/clear` and start a fresh agent for the next phase.

## Milestones

> _Phase 0 fills in the milestone list below based on the project requirements._

- [ ] **Phase 0: Research & Specification** [PHASE GATE — Human review]
  Read all files in `0_app_specification/`. Extract requirements. Generate all planning and architecture artifacts.
  Acceptance: This file populated, execution_tasks.md has Phase 1+ tasks, architecture docs generated.

- [ ] **Phase 1: {{PHASE_1_NAME}}** [PHASE GATE]
  {{PHASE_1_DESCRIPTION}}
  Acceptance: {{PHASE_1_ACCEPTANCE}}

- [ ] **Phase N: Integration & Polish** [PHASE GATE]
  End-to-end workflow tests, UI polish, documentation, clean clone verification.
  Acceptance: Full workflow runs from clone. All tests pass. README accurate.

- [ ] **Phase R: Pipeline Retrospective** [PHASE GATE — Always last]
  Analyze execution log for pipeline flaws. Write PIPELINE_IMPROVEMENT_REPORT.md. Apply patches.
  Acceptance: Report written. High-priority patches applied. Tests still pass.

## How to Start

```
1. Drop your input files into 0_app_specification/ (spec_docs/, research_papers/, inputs/)
2. Read CLAUDE.md  → execution loop, compression, safety rules
3. Read AGENTS.md  → directory map and boot sequence
4. Follow the boot sequence in AGENTS.md §1. Phase 0 runs first.
```
