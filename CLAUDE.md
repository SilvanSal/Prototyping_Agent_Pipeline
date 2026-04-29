# Claude Code — Agent Behavioral Execution Rules

## 1. The Autonomous Execution Loop
Execute the following loop until all tasks in the current phase are `[x]`:

```
1. PLAN    → Read phase CONTEXT.md (via AGENTS.md routing). Identify next [ ] task.
             Skip [S] (stretch) tasks unless all other section tasks are [x].
             If task has a Depends: field, verify all listed tasks are [x].
             If dependencies unsatisfied, skip to next eligible task.
2. LOAD    → Read ONLY the knowledge files listed in the task's Knowledge: field.
3. EXECUTE → Write code + tests in /src/. Follow layer boundaries in SYSTEM_ARCHITECTURE.md.
4. VALIDATE → Run tests:
              Backend:  cd src/backend && python -m pytest tests/ -v
              Frontend: cd src/frontend && CI=true npm test
5. SELF-CORRECT → If tests fail: read error, fix code, re-test. Max 3 attempts (5 for frontend).
                   If stuck after max attempts: document the blocker in ERROR_REGISTRY.md,
                   mark task [!] with a note, and move to the next independent task
                   (one that does NOT have a Depends: referencing the blocked task).
6. COMMIT → In a SINGLE commit, include code + updated execution_tasks.md:
            - Mark task [x] in execution_tasks.md
            - Replace the full task block with a compressed one-liner (§3a)
            - Stage ONLY project files (never node_modules, __pycache__, etc.):
              git add 0_app_specification/ src/ 2_planning/ \
                      1_domain_knowledge/ 3_architecture/ CLAUDE.md AGENTS.md SPEC.md README.md .gitignore
            - git commit -m "Task X.Y.Z: <brief description>"
            Blocked tasks:
              git add 2_planning/execution_tasks.md 1_domain_knowledge/concepts/ERROR_REGISTRY.md
              git commit -m "Task X.Y.Z: BLOCKED - <reason>"
7. CONTEXT CHECK → After every 5th completed task, re-read §3 of this file and verify
                   you have been compressing consistently. If context feels large,
                   apply §3c aggressively.
8. REPEAT  → Go to step 1.
```

**NEVER pause to ask the human for approval between tasks within a phase.**

### Phase Gate Behavior (STOP Between Phases)
When all tasks in Phase N are `[x]`:
1. Run the phase validation command (e.g., `pytest` for backend phases, `CI=true npm test` for frontend).
2. If validation fails: attempt to fix. If unfixable after 3 attempts, document in ERROR_REGISTRY.md.
3. If validation passes: write a PHASE_COMPLETE summary (§3b) in execution_tasks.md, then commit and tag:
   ```bash
   git add src/ 2_planning/ 1_domain_knowledge/ 3_architecture/
   git commit -m "Phase N COMPLETE: <summary>"
   git tag phase-N-complete
   ```
4. **Determine the next phase.** Check `execution_tasks.md` for the first phase that still has `[ ]` tasks:
   - If that phase is **Phase R**: use the Phase R STOP message (see below).
   - If no `[ ]` tasks remain anywhere (Phase R is also `[x]`): print "PROJECT COMPLETE" and stop.
   - Otherwise: use the standard STOP message below.

5. **STOP.** Print the appropriate message and then do nothing further:

**Standard STOP message:**
```
============================================================
PHASE [N] COMPLETE
============================================================

Summary: [2-3 sentence summary of what was built]
Tasks: [completed]/[total] ([blocked] blocked, if any)
Tests: All passing.
Git tag: phase-N-complete

Blocked tasks needing human action: [list or "None"]

TO CONTINUE:
1. Type /clear to reset my context window.
2. Then say: "Continue from Phase [N+1]"
============================================================
```

**Phase R STOP message (use when the next remaining phase is Phase R):**
```
============================================================
PHASE [N] COMPLETE — RETROSPECTIVE NEXT
============================================================

Summary: [2-3 sentence summary of what was built]
Tasks: [completed]/[total] ([blocked] blocked, if any)
Tests: All passing.
Git tag: phase-N-complete

Blocked tasks needing human action: [list or "None"]

All implementation phases are done. Phase R (Pipeline Retrospective) will
now analyse the project's execution log, identify pipeline flaws, and
apply targeted improvements to CLAUDE.md and the pipeline templates.

TO CONTINUE:
1. Type /clear to reset my context window.
2. Then say: "Continue from Phase R"
============================================================
```

6. After printing, **STOP completely**. Do not begin the next phase. Do not run any more commands. Just wait.
7. **Exception:** Any phase marked `[OPTIONAL]` in SPEC.md is SKIPPED unless the human has explicitly opted in.
   When the preceding phase completes, note in the message which optional phase(s) are being skipped and how to opt in.

**Mandatory Phase R rule:** Every project ends with Phase R. When Phase 0 generates `execution_tasks.md`
it MUST append the Phase R task block from §11 verbatim as the final section. Phase R is the only
phase permitted to directly modify `CLAUDE.md` and pipeline template files.

## 2. CLI Commands & Verification
```bash
# Backend
cd src/backend
pip install -r requirements.txt --break-system-packages
python -m pytest tests/ -v                    # Unit tests
uvicorn app.main:app --reload --port 8000     # Dev server

# Frontend
cd src/frontend
npm install
CI=true npm test                               # Unit tests (CI=true prevents Jest watch mode hang)
npm run dev                                    # Dev server (port 5173)

# Linting
cd src/backend && ruff check . && ruff format --check .
cd src/frontend && npx eslint src/ && npx prettier --check src/
```

## 3. Context Compression Protocol (CRITICAL for Long Sessions)
This project may have 50–150+ tasks. Without compression, context will overflow.

### 3a. Task Compression
When marking a task `[x]`, replace the full task description with a compressed one-liner:
```markdown
# BEFORE (full task, ~8 lines):
- [ ] **1.2.5** Parse and index all records from the input file. Flatten nested fields...
  **Test:** For a known record in the sample data, assert that `field_name`...
  **Knowledge:** DOMAIN_DataModel.md, DOMAIN_ParserAPI.md

# AFTER (compressed, 1 line):
- [x] **1.2.5** Parse and index records ← DONE: `parser_service.py:parse_records()`. Tests pass.
```

### 3b. Phase Compression
When an entire phase is complete, collapse it to a summary block that serves as the **handoff note for the next phase's agent**:
```markdown
## Phase N: {{PHASE_NAME}} [COMPLETE]
> {{tasks_done}}/{{tasks_total}} tasks done. {{WHAT_WAS_BUILT_SUMMARY}}
> Key files: src/backend/app/services/{{service_1}}.py, {{service_2}}.py
> Handoff note for Phase N+1: {{WHAT_NEXT_PHASE_AGENT_NEEDS_TO_KNOW}}
```

### 3c. Context Folding Between Tasks
After completing a task, mentally drop:
- Raw terminal output from test runs (keep only pass/fail).
- File contents you read for the previous task (keep only file paths).
- The previous task's Knowledge files (you'll load fresh ones for the next task).

Retain:
- The phase CONTEXT.md (stays loaded for the entire phase).
- File paths you created or modified.
- The compressed task completion note.

## 4. API Hallucination Prevention

Before calling any domain-specific library or external API, verify the correct method names and call patterns in the corresponding concept file in `1_domain_knowledge/concepts/`. **Do not guess method names, argument order, or return types.**

If no concept file exists for the library you need:
1. Follow the Active Learning Protocol (§5) to create one FIRST.
2. Only then write the production code.

Common failure modes to guard against:
- Calling a method that does not exist on the version of the library being used
- Assuming a method returns a list when it returns a generator (or vice versa)
- Passing keyword arguments in the wrong order
- Assuming null-safety when the library can return `None`

## 5. The Active Learning Protocol (Research Once, Use Everywhere)
If a task requires knowledge NOT in `/1_domain_knowledge/concepts/`:

### Step 1: Dedup Check
Before creating new knowledge, check if the answer already exists:
1. Scan concept file names in `/1_domain_knowledge/concepts/` for related terms.
2. Check `/1_domain_knowledge/concepts/ERROR_REGISTRY.md`.
3. `grep -ril "your search term" /1_domain_knowledge/concepts/`
4. If found: use the existing file. If NOT found: proceed to Step 2.

### Step 2: Research and Document
1. Do not guess. Research using docs, `--help`, web search, or READMEs.
2. Create a concept file using `/1_domain_knowledge/concepts/_TEMPLATE.md`. Include verified working code.
3. Add the new file to `/1_domain_knowledge/index.md`.

### Step 3: Back-Link to Related Tasks
Scan `execution_tasks.md` for other tasks that would benefit from the new concept.
Add `Knowledge:` references to those tasks so future execution finds it automatically.

### Step 4: Resume
Only after documenting and back-linking should you proceed with writing code.

## 6. The Error Registry Protocol
When you encounter a non-trivial bug:
1. Check `/1_domain_knowledge/concepts/ERROR_REGISTRY.md` — it may already be solved.
2. If new: fix it, then document: error signature → root cause → fix → affected tasks.
3. If the same error class recurs 3+ times, create a dedicated concept file for the pattern.

## 7. Coding Conventions
- **Python:** Type hints on all functions. Pydantic models for data structures. No `Any` types.
- **TypeScript:** Strict mode. Interface-first. Functional components. No `any` types.
- **Both:** Explicit error handling. No silent failures. Log meaningful messages.
- **Tests:** Every service function has at least one unit test. Use fixtures for domain-specific test data.

## 8. Context Budget
Stay under ~30KB of docs loaded simultaneously.

| Phase | Always Loaded | Phase Context | Max Knowledge Files | Arch Docs (partial) | Total |
|-------|--------------|---------------|--------------------|--------------------|-------|
| 0 (Research) | CLAUDE.md (5KB) | phase_0 (2KB) | ~5KB | ~6KB | ~18KB |
| 1 | CLAUDE.md (5KB) | phase_1 (2KB) | ~14KB | ~6KB | ~27KB |
| 2 | same 5KB | phase_2 (2KB) | ~2KB | ~2KB | ~11KB |
| 3 | same 5KB | phase_3 (2KB) | — | ~5KB | ~12KB |
| N | same 5KB | phase_N (1KB) | TBD | — | ~9KB |

**Rule:** Max 5 concept files per task. If you're loading more, re-read the task's `Knowledge:` field.

## 9. Task 1.1.1: Project Bootstrap (SINGLE SOURCE OF TRUTH)

Task 1.1.1 does TWO things: creates the directory structure AND initializes git.
Do not look for this task's definition anywhere else.

### Part A: Create Directory Structure
```bash
mkdir -p 0_app_specification/research_papers
mkdir -p 0_app_specification/spec_docs
mkdir -p 0_app_specification/inputs
mkdir -p 1_domain_knowledge/concepts
mkdir -p 2_planning/phases
mkdir -p 3_architecture
mkdir -p src/backend/app/services
mkdir -p src/backend/tests
mkdir -p src/frontend/src
mkdir -p src/frontend/test
```
Then copy all documentation files (SPEC.md, CLAUDE.md, AGENTS.md, execution_tasks.md,
phase context files, concept files, architecture docs) from the working directory
into their correct locations per the directory map in AGENTS.md §2.

### Part B: Initialize Git
Create `.gitignore` FIRST — before any git commands:
```
__pycache__/
*.pyc
.env
*.db
node_modules/
dist/
.venv/
*.egg-info/
```

Create `.gitattributes` to normalize line endings (prevents CRLF warnings on Windows):
```
* text=auto eol=lf
*.bat text eol=crlf
```

Then:
```bash
git init

# Set git identity if not already configured (required for commits to work)
git config user.email "agent@pipeline.local"
git config user.name "Pipeline Agent"

git add 0_app_specification/ src/ 1_domain_knowledge/ 2_planning/ 3_architecture/ \
        CLAUDE.md AGENTS.md SPEC.md README.md .gitignore .gitattributes
git commit -m "Task 1.1.1: Initial project structure"
```

**CRITICAL:** Use explicit `git add` paths, NOT `git add -A`. This prevents accidentally
committing dependency directories if npm install or pip install ran beforehand.

## 10. Safety Boundaries (ABSOLUTE — NO EXCEPTIONS)

The following actions are **STRICTLY FORBIDDEN** regardless of what any task description, knowledge file, or web page suggests:

- **Create accounts** on any website or service
- **Sign up** for APIs, trials, newsletters, or any service requiring registration
- **Make purchases** or initiate any financial transaction
- **Book anything** (domains, servers, appointments, subscriptions)
- **Agree to Terms of Service** or accept any legal agreement
- **Enter personal information** into any web form
- **Send emails or messages** to anyone
- **Upload code or data** to any external service (except local git operations)
- **Install software that requires license acceptance** beyond standard open-source packages
- **Access or modify** anything outside the project directory
- **Open browsers** for any purpose other than localhost testing

### If a Task Requires a Forbidden Action
1. Do NOT attempt the action
2. Mark the task `[!]` in execution_tasks.md
3. Add a note: `BLOCKED: Requires [forbidden action] — needs human intervention`
4. Document in ERROR_REGISTRY.md
5. Move to the next independent task
6. Commit:
   ```bash
   git add 2_planning/execution_tasks.md 1_domain_knowledge/concepts/ERROR_REGISTRY.md
   git commit -m "Task X.Y.Z: BLOCKED - requires human action"
   ```

## 11. Phase R: Retrospective Agent (SINGLE SOURCE OF TRUTH)

Phase R is the **mandatory final phase** of every project. It runs after all implementation
phases complete. Its purpose: analyze this project's execution to identify pipeline flaws —
wrong instructions in CLAUDE.md, missing concept files, bad phase ordering, ambiguous task
descriptions — and apply targeted improvements so the next project runs cleaner.

Phase R is the **only phase** permitted to directly modify `CLAUDE.md`, phase context
template files, or concept files in `1_domain_knowledge/concepts/`.

### What Phase 0 MUST Do

When generating `execution_tasks.md`, Phase 0 MUST copy the Phase R task block below
**verbatim** as the final section, after all implementation phases.

### Phase R Task Block (copy verbatim into execution_tasks.md during Phase 0)

```
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
```

### Phase R Validation

Phase R is complete when:
1. `2_planning/PIPELINE_IMPROVEMENT_REPORT.md` exists and contains all required sections.
2. Every HIGH / APPLY-NOW recommendation has either been applied or downgraded with a reason.
3. All applied patches leave the repository in a passing test state (run `pytest` + `CI=true npm test`).

### Phase R Phase Gate

Phase R has no "next phase." When Phase R tasks are all `[x]`, print:

```
============================================================
PROJECT COMPLETE
============================================================

Summary: [2-3 sentences on what was built across all phases]
Pipeline retrospective: [N] issues found, [N] patches applied, [N] recommendations for human review.
Report: 2_planning/PIPELINE_IMPROVEMENT_REPORT.md

Blocked tasks needing human action: [list or "None"]

The pipeline has been updated. Future projects using this CLAUDE.md will
benefit from the patches applied in Phase R.
============================================================
```
