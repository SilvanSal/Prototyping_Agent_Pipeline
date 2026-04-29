# Pipeline Roadmap — Three Missing Capabilities

Three capabilities are specified but not yet implemented. Each section is self-contained;
an agent can pick up any section independently. Complete them in order O → H → E, as the
orchestrator (O) depends on the handoff system (H), and evals (E) plug into both.

**How to continue:** Read CLAUDE.md and AGENTS.md for behavioral rules, then return here
and find the first unchecked task. Implement it, check the box, commit, and continue.

---

## Feature O: Automated Orchestrator

**What it does:** Replaces the human "Continue from Phase N" step. A master orchestrator
script reads the pipeline state, builds a focused prompt for the current phase's sub-agent,
spawns it via the Claude API, monitors for phase gate completion, and automatically advances
to the next phase — until Phase R finishes and the project is complete.

**Target state:** `python orchestrator/run.py` drives the entire pipeline from Phase 0 to
Phase R without human input between phases.

---

- [ ] **O.1** Add a machine-readable sentinel to `CLAUDE.md` §1 Phase Gate Behavior.

  In the STOP message printed at every phase gate, add this exact line at the end:
  ```
  PIPELINE_SIGNAL: PHASE_COMPLETE phase={N}
  ```
  This is how the orchestrator detects that a phase finished without parsing natural language.

  **File to edit:** `CLAUDE.md` — both STOP message templates in §1 Phase Gate Behavior.
  **Done when:** Both STOP message templates in CLAUDE.md contain the `PIPELINE_SIGNAL:` line.

---

- [ ] **O.2** Create `orchestrator/` directory and add `orchestrator/phase_prompt_template.md`.

  This template is what the orchestrator passes as the system prompt to each phase sub-agent.
  It must contain these exact sections with `{{VARIABLE}}` placeholders:
  ```
  ## Your Role
  You are a software development agent executing Phase {{PHASE_NUMBER}} of a multi-phase
  pipeline. Execute all tasks in your phase autonomously, then STOP.

  ## Behavioral Rules
  {{CLAUDE_MD_SECTIONS_1_2_3_4_5_6_7}}

  ## Phase Goal and Context
  {{PHASE_CONTEXT_FILE_CONTENT}}

  ## Your Task List
  {{CURRENT_PHASE_TASKS}}

  ## Stopping Condition
  When all tasks in your phase are [x] and validation passes, write the PHASE_COMPLETE
  summary, commit, tag, print the STOP message, and do nothing further.
  ```

  Note: `CLAUDE_MD_SECTIONS_1_2_3_4_5_6_7` injects §1–§7 of CLAUDE.md only — not §8–§11
  (context budget tables and bootstrap details the sub-agent doesn't need).

  **Files to create:** `orchestrator/phase_prompt_template.md`, `orchestrator/.gitkeep`
  **Done when:** Template exists with all five `{{VARIABLE}}` blocks documented.

---

- [ ] **O.3** Create `orchestrator/pipeline_state.py`.

  A utility module with two functions:
  - `get_current_phase(repo_path) -> str | None` — reads git tags to find the highest
    `phase-N-complete` tag. Returns the *next* phase identifier (e.g. if `phase-1-complete`
    exists but not `phase-2-complete`, returns `"2"`). Returns `"0"` if no tags exist.
    Returns `None` if `phase-R-complete` exists (pipeline done).
  - `get_phase_tasks(repo_path, phase_id) -> str` — reads `2_planning/execution_tasks.md`
    and returns only the lines belonging to the given phase (from its `## Phase N` header
    to the next `## Phase` header). Strips already-compressed `[x]` lines to save tokens.

  **File to create:** `orchestrator/pipeline_state.py`
  **Done when:** Both functions exist with type hints. `get_current_phase` correctly advances
  phase on each call when the corresponding git tag exists.

---

- [ ] **O.4** Create `orchestrator/run.py` — the master orchestrator.
  **Depends on:** O.1, O.2, O.3

  Logic:
  ```python
  while True:
      phase = get_current_phase(repo_path)
      if phase is None:
          print("Pipeline complete.")
          break

      # Build sub-agent prompt
      prompt = build_prompt(phase)  # fills O.2 template with real content

      # Spawn sub-agent via Anthropic API with these tools:
      # - text_editor (read/write files)
      # - bash (git, pytest, npm test)
      response = run_claude_agent(prompt, tools=[...])

      # Verify the sentinel appears in output
      if "PIPELINE_SIGNAL: PHASE_COMPLETE" not in response:
          log_error(f"Phase {phase} did not emit PHASE_COMPLETE sentinel")
          break

      # Loop — get_current_phase will now return the next phase
  ```

  The agent spawned for each phase must have these tools available:
  - Read/write access to the full project directory
  - `bash` for running git, pytest, npm, and ruff
  - No internet access (safety boundary from CLAUDE.md §10)

  **File to create:** `orchestrator/run.py`
  **File to create:** `orchestrator/requirements.txt` (anthropic>=0.40.0, pyyaml)
  **Done when:** `python orchestrator/run.py` detects current phase state, constructs a
  valid prompt, and spawns a Claude agent that can read/write files and run tests.
  Full end-to-end test: run against a fresh project with a test spec and confirm it
  reaches Phase 1 COMPLETE without human input.

---

- [ ] **O.5** Update `README.md` Quick Start.

  Replace Step 2 ("Open a new Claude Code session and say: Continue from Phase 0") with:
  ```
  ## 2. Run the orchestrator
  cd orchestrator
  pip install -r requirements.txt
  export ANTHROPIC_API_KEY=your_key_here
  python run.py
  ```
  Keep the manual "Continue from Phase N" instructions as a fallback under
  "### Manual mode (without the orchestrator)".

  **File to edit:** `README.md`
  **Done when:** README describes both automated and manual start paths.

---

## Feature H: Structured Inter-Agent Handoff Documents

**What it does:** After each task and each phase, the agent writes a structured handoff
file to `2_planning/handoffs/`. The next task/phase agent reads it before starting.
This replaces the current system of compressed one-liners as the sole inter-agent record.

**Target state:** Every completed task leaves a `task_X_Y_Z.md` handoff. Every completed
phase leaves a `phase_N.md` handoff. The next agent reads the relevant file at boot.

---

- [ ] **H.1** Create `2_planning/handoffs/` directory and `2_planning/handoffs/HANDOFF_FORMAT.md`.

  HANDOFF_FORMAT.md defines two templates:

  **Task-level handoff** (`task_X_Y_Z.md`):
  ```markdown
  # Task X.Y.Z Handoff
  **Status:** COMPLETE | BLOCKED
  **Files created/modified:** [list with line ranges for key functions]
  **Test result:** N passed, 0 failed
  **Key decisions made:** [anything not in the spec that the next agent needs to know]
  **Open questions / known issues:** [or "None"]
  ```

  **Phase-level handoff** (`phase_N.md`):
  ```markdown
  # Phase N Handoff
  **Status:** COMPLETE | PARTIAL (N/M tasks done)
  **What was built:** [2-3 sentences]
  **All files created/modified:** [full list]
  **Blocked tasks:** [list with reasons, or "None"]
  **Decisions that affect later phases:** [anything the next phase agent must know]
  **Eval results:** PASS | FAIL | NOT RUN
  **Recommended first task for next phase:** [task ID and why]
  ```

  **Files to create:** `2_planning/handoffs/.gitkeep`, `2_planning/handoffs/HANDOFF_FORMAT.md`
  **Done when:** Both templates are defined in the format file.

---

- [ ] **H.2** Update `CLAUDE.md` §1 step 6 (COMMIT) to write a task handoff before committing.

  Add after the existing COMMIT instructions:
  ```
  Before committing, write 2_planning/handoffs/task_X_Y_Z.md using the task-level
  template from 2_planning/handoffs/HANDOFF_FORMAT.md. Include this file in the commit:
    git add 2_planning/handoffs/task_X_Y_Z.md
  ```

  **File to edit:** `CLAUDE.md` §1 step 6
  **Done when:** The COMMIT step explicitly mentions writing and staging the handoff file.

---

- [ ] **H.3** Update `CLAUDE.md` §1 step 1 (PLAN) to read the predecessor's task handoff.

  Add at the start of the PLAN step:
  ```
  Before identifying the next task: if a handoff file exists for the most recently
  completed task (2_planning/handoffs/task_X_Y_Z.md), read it. Pay attention to
  "Open questions / known issues" — these may affect your current task.
  ```

  **File to edit:** `CLAUDE.md` §1 step 1
  **Done when:** PLAN step instructs the agent to read the predecessor handoff file.

---

- [ ] **H.4** Update `CLAUDE.md` §1 Phase Gate Behavior to write a phase-level handoff.

  In step 3 (after validation passes, before the STOP message), add:
  ```
  Write 2_planning/handoffs/phase_N.md using the phase-level template from
  2_planning/handoffs/HANDOFF_FORMAT.md. Stage and include it in the phase commit.
  ```

  **File to edit:** `CLAUDE.md` §1 Phase Gate Behavior step 3
  **Done when:** Phase Gate instructions include writing phase_N.md handoff.

---

- [ ] **H.5** Update `phase_N_context_TEMPLATE.md` to reference the handoff from the prior phase.

  Add to the "Always Load" section:
  ```
  - `2_planning/handoffs/phase_{N-1}.md` — handoff from the previous phase (if it exists)
  ```
  Phase 0 should propagate this reference when it generates each phase_N_context.md.

  **File to edit:** `2_planning/phases/phase_N_context_TEMPLATE.md`
  **Done when:** Template lists the prior-phase handoff file in its Always Load section.

---

- [ ] **H.6** Update `AGENTS.md` boot sequence Step 2.

  Add: "Also read `2_planning/handoffs/phase_{N-1}.md` if it exists — this is the
  structured summary written by the previous phase's agent."

  **File to edit:** `AGENTS.md` §1 Boot Sequence Step 2
  **Done when:** AGENTS.md boot sequence references the handoff directory.

---

## Feature E: Eval Framework

**What it does:** After unit tests pass, a suite of evals checks output quality beyond
pass/fail: API contract conformance, type safety, lint cleanliness, and test coverage.
Results are written to `evals/eval_results.md` and read by Phase R. Failing an eval
triggers the same SELF-CORRECT loop as a failing test.

**Target state:** `python evals/run_evals.py` runs after every phase gate. Phase R reads
`evals/eval_results.md` as a third evidence source alongside ERROR_REGISTRY.md and git log.

---

- [ ] **E.1** Create `evals/` directory structure.

  ```
  evals/
    run_evals.py          ← master runner (writes eval_results.md)
    eval_config.yaml      ← thresholds and which evals are active
    eval_results.md       ← output file (committed, read by Phase R)
    backend/
      contract_eval.py    ← API contract conformance
      coverage_eval.py    ← pytest coverage threshold
      type_eval.py        ← mypy
      lint_eval.py        ← ruff
    frontend/
      coverage_eval.py    ← vitest coverage threshold
      type_eval.py        ← tsc --noEmit
      lint_eval.py        ← eslint
  ```

  **Files to create:** `evals/.gitkeep` + all files listed above (stubs are fine for now)
  **Done when:** Directory and all files exist, even if eval logic is not yet implemented.

---

- [ ] **E.2** Implement `evals/eval_config.yaml`.

  ```yaml
  backend:
    active: true
    coverage_threshold: 80        # percent
    max_lint_errors: 0
    mypy_strict: false

  frontend:
    active: true
    coverage_threshold: 70
    max_lint_errors: 0
    tsc_strict: true
  ```

  **Done when:** File is valid YAML and all keys are documented with comments.

---

- [ ] **E.3** Implement `evals/backend/contract_eval.py`.
  **Depends on:** E.1

  Logic:
  1. Parse `3_architecture/API_CONTRACTS.md` to extract all specified endpoints
     (method + path, e.g. `POST /api/links`).
  2. Start the FastAPI app in test mode and fetch `/openapi.json`.
  3. Compare: every endpoint in API_CONTRACTS.md must appear in openapi.json.
  4. Return PASS if all endpoints present, FAIL with diff if any are missing.

  If `3_architecture/API_CONTRACTS.md` does not exist (Phase 0 not yet run), return SKIP.

  **Done when:** Function returns structured result `{status: PASS|FAIL|SKIP, details: str}`.

---

- [ ] **E.4** Implement `evals/backend/coverage_eval.py`, `type_eval.py`, `lint_eval.py`.
  **Depends on:** E.1, E.2

  - `coverage_eval.py`: runs `pytest --cov=app --cov-report=json -q`, reads
    `.coverage_report.json`, checks line coverage ≥ `eval_config.yaml` threshold.
  - `type_eval.py`: runs `mypy app/` from `src/backend/`. Captures returncode.
  - `lint_eval.py`: runs `ruff check app/` from `src/backend/`. Parses output for
    error count, fails if count > `max_lint_errors`.

  Each returns `{status: PASS|FAIL|SKIP, details: str}`.
  SKIP if `src/backend/app/` does not exist (backend phase not yet run).

  **Done when:** All three functions return structured results and have a `__main__` block
  for standalone testing.

---

- [ ] **E.5** Implement `evals/frontend/type_eval.py`, `lint_eval.py`, `coverage_eval.py`.
  **Depends on:** E.1, E.2

  - `type_eval.py`: runs `npx tsc --noEmit` from `src/frontend/`. Captures output.
  - `lint_eval.py`: runs `npx eslint src/` from `src/frontend/`. Parses error count.
  - `coverage_eval.py`: runs `CI=true npm test -- --coverage --coverageReporters=json`
    from `src/frontend/`. Reads `coverage/coverage-summary.json`. Checks line coverage.

  Each returns `{status: PASS|FAIL|SKIP, details: str}`.
  SKIP if `src/frontend/src/` does not exist.

  **Done when:** All three functions return structured results.

---

- [ ] **E.6** Implement `evals/run_evals.py` — master eval runner.
  **Depends on:** E.2, E.3, E.4, E.5

  Logic:
  1. Load `eval_config.yaml`.
  2. Run all non-SKIP evals, collect results.
  3. Write `evals/eval_results.md`:
     ```markdown
     # Eval Results — Phase N — {datetime}

     | Eval | Status | Details |
     |------|--------|---------|
     | backend/contract | PASS | 5/5 endpoints present |
     | backend/coverage | FAIL | 72% < 80% threshold |
     ...

     **Overall: PASS | FAIL**
     ```
  4. Exit code 0 if all active evals pass, 1 if any fail.

  **Done when:** `python evals/run_evals.py` produces `eval_results.md` and returns the
  correct exit code. Test against both a passing and a failing state.

---

- [ ] **E.7** Wire evals into `CLAUDE.md` §1.

  **Step 4 (VALIDATE)** — append after the existing test commands:
  ```
  Evals (run after tests pass):
    python evals/run_evals.py
  If evals fail: treat as a test failure. Apply SELF-CORRECT (max 3 attempts).
  Log each eval failure to ERROR_REGISTRY.md with the eval name and details.
  ```

  **Phase Gate Behavior step 1** — replace "Run the phase validation command" with:
  ```
  Run tests AND evals:
    Backend: cd src/backend && python -m pytest tests/ -v
    Frontend: cd src/frontend && CI=true npm test
    Evals: python evals/run_evals.py   (from project root)
  All three must pass before proceeding.
  ```

  **File to edit:** `CLAUDE.md` §1 steps 4 and Phase Gate Behavior step 1
  **Done when:** Both locations explicitly include `python evals/run_evals.py`.

---

- [ ] **E.8** Update Phase R to include evals evidence.

  In `2_planning/execution_tasks.md` task R.1.1: append to its description:
  "Also read `evals/eval_results.md` if it exists. Log recurring eval failures under
  'Stress Signals' — a pattern of coverage failures across phases is a P-AT or P-CG
  signal indicating tasks are under-specified."

  In `1_domain_knowledge/concepts/PIPELINE_RETROSPECTIVE.md` §3 (Git Stress Signals):
  add a new subsection "Eval History Analysis" explaining how to read eval_results.md
  and map eval failure patterns to the P-* taxonomy.

  In `2_planning/phases/phase_R_context.md` "Files to Read for Evidence" table:
  add `evals/eval_results.md` row.

  **Files to edit:** `execution_tasks.md` (R.1.1), `PIPELINE_RETROSPECTIVE.md`,
  `phase_R_context.md`
  **Done when:** All three files reference `evals/eval_results.md`.

---

## Completion Checklist

- [ ] All O tasks done → `python orchestrator/run.py` drives phases without human input
- [ ] All H tasks done → every completed task and phase has a handoff file in `2_planning/handoffs/`
- [ ] All E tasks done → `python evals/run_evals.py` runs after every phase gate and feeds Phase R
- [ ] End-to-end smoke test: drop a simple spec, run `python orchestrator/run.py`, confirm
      the pipeline reaches Phase R COMPLETE with handoff files and eval_results.md present
