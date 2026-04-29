# Execution Tasks: {{PROJECT_NAME}}

> Each task is independently testable. Tasks are numbered by Phase.Section.Task.
> **Status markers:** `[ ]` = not started, `[x]` = complete, `[!]` = blocked, `[S]` = stretch goal (skip unless all other section tasks are `[x]`).
> **Test criteria** are provided inline for every task.
> **Depends:** field lists task IDs that MUST be `[x]` before starting this task.
>
> ## Agent Instructions
> 1. Before starting a phase, read `/2_planning/phases/phase_N_context.md` for context routing.
> 2. Each task has a `Knowledge:` field listing concept files to load. Load ONLY those files.
> 3. When marking `[x]`, compress the task to a one-liner per CLAUDE.md §3a.
> 4. When an entire phase is `[x]`, compress it to a summary block per CLAUDE.md §3b — this summary is the handoff note for the next phase's agent.
> 5. Skip `[S]` (stretch) tasks unless all other tasks in the same section are `[x]`.
> 6. When a task has a `Depends:` field, verify all listed tasks are `[x]` before starting.

---

## Phase 0: Research & Specification

### 0.0 Bootstrap

- [ ] **0.0.1** Verify git identity.
  Run: `git config user.email && git config user.name`. If either is blank, set both:
  `git config user.email "agent@pipeline.local"` and `git config user.name "Pipeline Agent"`.
  **Test:** `git config user.email` returns a non-blank value.
  **Knowledge:** _(none required)_

### 0.1 Input Validation

- [ ] **0.1.1** Verify user input exists in `0_app_specification/`.
  Check `spec_docs/` for non-.gitkeep files. If only `.gitkeep` files found: print the
  ACTION REQUIRED message below and **STOP immediately**.
  ```
  ============================================================
  ACTION REQUIRED: No spec files found.
  ============================================================
  Please drop at least one .md requirements file into:
    0_app_specification/spec_docs/
  Then say "Continue from Phase 0" again.
  ============================================================
  ```
  **Test:** At least one `.md` file exists in `0_app_specification/spec_docs/`.
  **Knowledge:** _(none required)_

### 0.2 Domain Research

- [ ] **0.2.1** Read all input files.
  Read every file in `0_app_specification/spec_docs/`, `research_papers/`, and `inputs/`
  (skip .gitkeep-only directories). Extract: core domain entities, key operations, tech
  constraints, and non-functional requirements.
  **Test:** Can answer: "What does this app do? What are its main entities? What tech constraints exist?"
  **Knowledge:** _(none required)_

- [ ] **0.2.2** Identify domain terms and create concept files.
  List every domain-specific library, API, algorithm, or pattern that will appear in two or
  more implementation tasks. For each term: create a concept file in
  `1_domain_knowledge/concepts/` using `_TEMPLATE.md` as the format. Include verified working
  code patterns — do not guess. Add each file to `1_domain_knowledge/index.md`.
  **Test:** `1_domain_knowledge/index.md` lists at least one domain-specific concept file
  (beyond the permanent entries).
  **Knowledge:** _(none required)_

### 0.3 Artifact Generation

- [ ] **0.3.1** Fill in `SPEC.md`.
  Replace all `{{PLACEHOLDER}}` strings with project-specific content: project name, one-sentence
  goal, and one milestone entry per planned phase (Phase 1, Phase 2, … Phase N, Phase R).
  **Test:** `grep -c "{{" SPEC.md` returns 0.
  **Knowledge:** _(none required)_

- [ ] **0.3.2** Generate `3_architecture/SYSTEM_ARCHITECTURE.md`.
  Propose and document: tech stack (language versions, frameworks, databases), layer boundaries
  (which code lives where), and data flow (as ASCII or prose diagram).
  **Test:** File names a specific technology for every layer (frontend, backend, database, tests).
  **Knowledge:** _(none required)_

- [ ] **0.3.3** Generate `3_architecture/DECISION_LOG.md`.
  Write at least 3 Architecture Decision Records for key technical choices made in 0.3.2.
  Each ADR format: Title, Status, Context, Decision, Consequences.
  **Test:** File contains at least 3 complete ADR entries.
  **Knowledge:** _(none required)_

- [ ] **0.3.4** Generate `3_architecture/API_CONTRACTS.md`.
  For every API endpoint or service interface: specify method, path, request schema, response
  schema, and error codes. Group by resource.
  **Test:** File contains at least 2 endpoint groups with concrete request/response schemas.
  **Knowledge:** _(none required)_

- [ ] **0.3.5** Generate `2_planning/implementation_plan.md`.
  Write a technical design document: component breakdown, service interfaces, database schema,
  and any cross-cutting concerns (auth, error handling, logging).
  **Test:** File exists and references concrete files/classes that will be created in Phase 1.
  **Knowledge:** _(none required)_

- [ ] **0.3.6** Generate phase context files.
  For each planned implementation phase: create `2_planning/phases/phase_N_context.md` using
  `phase_N_context_TEMPLATE.md` as the base. Fill in all {{PLACEHOLDERS}} with phase-specific
  knowledge files, architecture sections, constraints, and validation criteria.
  **Test:** Each generated `phase_N_context.md` has no remaining `{{PLACEHOLDER}}` strings.
  **Knowledge:** _(none required)_

- [ ] **0.3.7** Generate Phase 1 tasks in this file.
  Under a new "## Phase 1" section below: write fully-specified tasks with `Test:` and
  `Knowledge:` fields. Each task must be independently testable. Add `Depends:` fields where
  execution order matters.
  **Test:** Phase 1 section has at least 5 tasks, each with `Test:` and `Knowledge:` fields.
  Every `Knowledge:` file referenced exists in `1_domain_knowledge/concepts/`.
  **Knowledge:** _(none required)_

- [ ] **0.3.8** Generate Phase 2–N task stubs and confirm Phase R is appended.
  Add stub task sections for all phases after Phase 1. Verify that the Phase R task block
  from CLAUDE.md §11 appears verbatim as the final section of this file.
  **Test:** This file has stub sections for all phases. The Phase R section with R.1.1–R.2.2
  tasks appears last.
  **Knowledge:** _(none required)_

---

> ## ↓ Phase 0 appends implementation phases here (tasks 0.3.7 and 0.3.8) ↓

---

## Phase R: Pipeline Retrospective

### R.1 Evidence Gathering

- [ ] **R.1.1** Collect git stress signals.
  Run `git log --oneline` and `git log --all --grep="BLOCKED" --oneline`. Count commits
  per task (>1 commit on the same task number = self-correct attempt). List all tasks
  with 2+ commits and all BLOCKED commits. Write findings under "## Stress Signals" in
  a new file `2_planning/PIPELINE_IMPROVEMENT_REPORT.md`.
  **Knowledge:** PIPELINE_RETROSPECTIVE.md

- [ ] **R.1.2** Triage ERROR_REGISTRY.md.
  For each error entry: classify as Pipeline-Caused (P-*) or Domain-Caused (D-*) using
  the framework in PIPELINE_RETROSPECTIVE.md §2. Write a classification table under
  "## Error Triage" in PIPELINE_IMPROVEMENT_REPORT.md.
  **Knowledge:** PIPELINE_RETROSPECTIVE.md

- [ ] **R.1.3** Audit execution_tasks.md for blocked and degraded tasks.
  Search for [!] entries. For each: identify root cause category from PIPELINE_RETROSPECTIVE.md §2.
  Also note tasks that appear in ERROR_REGISTRY.md. Write under "## Blocked Tasks" in the report.
  **Knowledge:** PIPELINE_RETROSPECTIVE.md

### R.2 Analysis and Remediation

- [ ] **R.2.1** Write PIPELINE_IMPROVEMENT_REPORT.md.
  Using evidence from R.1.1–R.1.3, produce the complete report using the template in
  PIPELINE_RETROSPECTIVE.md §4. Must include: Executive Summary, Pipeline Health score,
  Error Triage table, Stress Signals, Blocked Tasks, and numbered Recommendations
  (each with Problem / Evidence / Root cause / Proposed fix / Priority / Action).
  **Depends:** R.1.1, R.1.2, R.1.3
  **Knowledge:** PIPELINE_RETROSPECTIVE.md

- [ ] **R.2.2** Apply high-confidence patches.
  For each Recommendation rated Priority: HIGH and Action: APPLY NOW (per rules in
  PIPELINE_RETROSPECTIVE.md §5): apply the patch directly to the relevant pipeline file
  (CLAUDE.md, a phase context template, or a concept file). Record every applied change
  under "## Applied Patches" in PIPELINE_IMPROVEMENT_REPORT.md. Leave MEDIUM/LOW and
  HUMAN REVIEW items as proposals only.
  **Depends:** R.2.1
  **Knowledge:** PIPELINE_RETROSPECTIVE.md
