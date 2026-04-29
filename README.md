# Autonomous Agent Coding Pipeline

A framework for building software autonomously with Claude. Drop a spec, say "Continue from Phase 0", and the pipeline researches, plans, codes, tests, and improves itself — phase by phase, with a full retrospective at the end.

---

## How It Works

The pipeline runs as a sequence of **phases**, each handled by a fresh Claude agent. The agent reads its instructions from `CLAUDE.md` and `AGENTS.md`, executes all tasks autonomously, then stops at a phase gate and waits for you to start the next phase.

```
Phase 0  →  Phase 1  →  Phase 2  →  …  →  Phase N  →  Phase R
Research    Backend     Frontend        Polish        Retrospective
```

**Phase R** is special — it's always the final phase. It reads the error log, git history, and blocked tasks from the whole project and applies targeted improvements to the pipeline itself, so future projects run cleaner.

---

![Project Diagram](https://raw.githubusercontent.com/SilvanSal/Prototyping_Agent_Pipeline/refs/heads/main/Diagram_Visualisation.svg)

---

## Quick Start

### 1. Drop your spec

Place at least one `.md` requirements file into:

```
0_app_specification/spec_docs/    ← your spec goes here (required)
0_app_specification/research_papers/   ← PDFs, papers, prior art (optional)
0_app_specification/inputs/            ← sample data, schemas, configs (optional)
```

### 2. Start Phase 0

Open a new Claude Code session in this directory and say:

```
Continue from Phase 0
```

The agent will read your spec, generate all planning and architecture artifacts, write the full task list for all phases, and stop.

### 3. Continue phase by phase

After each phase completes, the agent prints a STOP message like:

```
TO CONTINUE:
1. Type /clear to reset my context window.
2. Then say: "Continue from Phase N"
```

Follow those instructions exactly. Each `/clear` gives the next agent a fresh context window.

### 4. Let Phase R close the loop

After the final implementation phase, the pipeline automatically queues **Phase R**. That agent:
- Classifies every error as pipeline-caused or domain-caused
- Reads git history for self-correct attempts and blocked tasks
- Writes `2_planning/PIPELINE_IMPROVEMENT_REPORT.md`
- Applies high-confidence fixes directly to `CLAUDE.md` and pipeline templates

---

## Pipeline Files

```
CLAUDE.md                          ← All agent behavioral rules (DO NOT delete)
AGENTS.md                          ← Directory map and boot sequence (DO NOT delete)
SPEC.md                            ← Project spec — Phase 0 fills this in
0_app_specification/               ← DROP ZONE: put your spec files here
1_domain_knowledge/
  concepts/                        ← Verified code patterns (Phase 0 generates, Phase R improves)
  index.md                         ← Index of all concept files
2_planning/
  execution_tasks.md               ← Master task list with status markers
  phases/
    phase_0_context.md             ← Boot context for the research agent
    phase_R_context.md             ← Boot context for the retrospective agent
    phase_N_context_TEMPLATE.md    ← Template Phase 0 uses to generate phase context files
3_architecture/                    ← Phase 0 generates SYSTEM_ARCHITECTURE, API_CONTRACTS, DECISION_LOG
src/
  backend/                         ← Python (FastAPI) — Phase 1+ writes here
  frontend/                        ← TypeScript (React) — frontend phase writes here
```

---

## Running Tests

```bash
# Backend
cd src/backend
python -m pytest tests/ -v

# Frontend
cd src/frontend
CI=true npm test
```

---

## Configuration

The agent's behaviour is controlled entirely by `CLAUDE.md`. Key sections:

| Section | Controls |
|---------|----------|
| §1 | Autonomous execution loop and phase gate behaviour |
| §3 | Context compression (critical for long projects) |
| §5 | Active Learning Protocol (how to create concept files) |
| §9 | Project bootstrap (Task 1.1.1 definition) |
| §10 | Safety boundaries (absolute — never edit) |
| §11 | Phase R definition and task block |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Agent stops asking for approval | Make sure you said "Continue from Phase N" after `/clear` — the boot sequence in AGENTS.md §1 sets the autonomous mode |
| Phase 0 prints "ACTION REQUIRED" | Drop at least one `.md` spec file into `0_app_specification/spec_docs/` |
| Tests fail after a phase | Check `1_domain_knowledge/concepts/ERROR_REGISTRY.md` — the agent documents fixes there |
| Context getting large | Say `/clear` and re-read `CLAUDE.md §3c` — the agent should be compressing aggressively |
| `git commit` fails with "Author identity unknown" | Run: `git config user.email "you@example.com" && git config user.name "Your Name"` |
