# Domain Knowledge Index

This directory contains verified code patterns, concept definitions, and domain knowledge for this project.

## How to Use

1. Before writing code for a task, check whether a concept file exists for the relevant domain term.
2. If it exists: read it, use the verified patterns.
3. If it does NOT exist: follow the **Active Learning Protocol** in CLAUDE.md §5 to create one.
4. After creating a new concept file, add it to this index.

## Index

| File | Topic | Phase Introduced |
|------|-------|-----------------|
| [`_TEMPLATE.md`](_TEMPLATE.md) | Template for creating new concept files | — |
| [`ERROR_REGISTRY.md`](ERROR_REGISTRY.md) | Known bugs: signature → root cause → fix | — |
| [`PIPELINE_RETROSPECTIVE.md`](PIPELINE_RETROSPECTIVE.md) | Phase R analysis framework: error classification, git stress signals, report template, patch rules | Phase R |

> _Phase 0 adds project-specific concept files here. Each entry: filename, topic (one phrase), and which phase introduced it._
