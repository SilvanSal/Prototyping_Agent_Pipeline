# 0_app_specification — User Input Drop Zone

This folder is the **entry point** for the agentic pipeline. Drop your project-specific files here before running Phase 0.

The Phase 0 agent reads everything in this folder and uses it to:
- Populate `SPEC.md` with your project's requirements
- Generate domain concept files in `1_domain_knowledge/concepts/`
- Propose a technology stack in `3_architecture/SYSTEM_ARCHITECTURE.md`
- Write the Phase 1–N task lists in `2_planning/execution_tasks.md`

---

## Subfolders

### `research_papers/`
Drop academic papers, technical articles, or prior-art documents here.

**Accepted formats:** PDF, Markdown (.md), plain text (.txt)

**Examples:**
- Research papers describing the domain problem
- Technical reports from stakeholders
- Documentation for libraries or APIs you want to use
- Competing or related products to reference

### `spec_docs/`
Drop your requirement documents, design specs, and wireframes here.

**Accepted formats:** Markdown (.md), plain text (.txt)

**Examples:**
- User stories or feature lists
- Wireframe descriptions or mockup notes
- Data model sketches
- Non-functional requirements (performance, security, accessibility)
- A list of "must have / nice to have / out of scope" items

### `inputs/`
Drop sample data, schemas, and configuration files here.

**Accepted formats:** JSON, YAML, CSV, XML, SQL, or any structured format

**Examples:**
- Sample input data files the application will process
- Database schemas or ERD descriptions
- Example API responses from external services you will integrate with
- Configuration templates

---

## Tips for Good Results

1. **Be specific about deliverables.** Write "a web app where users can upload X and get Y" rather than "a tool for working with X."
2. **Include constraints.** If you must use a specific language, database, or framework, say so in a spec doc.
3. **Include sample data.** Even one realistic example file helps the agent understand your data format.
4. **One spec doc is enough to start.** You can add more files and re-run Phase 0 later.

---

## After Dropping Your Files

1. Start a new Claude Code session in this directory and say: `Continue from Phase 0`
2. The agent will follow the boot sequence in `AGENTS.md`, read your spec files, and run Phase 0.
3. Phase 0 ends with a **Phase Gate** — you review the generated artifacts before continuing.
4. After your review, `/clear` the context window and say: `Continue from Phase 1`

---

## What If I Have No Files Yet?

Create a single file called `spec_docs/my_project.md` and answer these questions:

```markdown
# My Project

## What problem does this solve?
(answer here)

## Who uses it?
(answer here)

## What is the core deliverable?
(answer here)

## What inputs does the application take?
(answer here)

## What outputs does it produce?
(answer here)

## Technology constraints (optional):
- Must use: (e.g., Python, React, PostgreSQL)
- Must avoid: (e.g., Java, AWS)

## Out of scope:
- (list things you explicitly do NOT want built)
```
