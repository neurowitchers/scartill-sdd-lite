---
name: scartill-sdd-lite
description: "Lightweight Kiro-first specification driven development kit"
---

# Commands

- `Plan Spec` - expand seed(s) into a full spec via a persistent, iterative questionnaire file (file-based clarification instead of interactive terminal Q&A) (prompt: `sc.plan.spec.md`, optional arguments: a seed path and/or free-form refining notes)
- `Save Spec` - save persistent specification (prompt: `sc.save.spec.md`)
- `Split Tasks` - **required before implementation**; generate authoritative, detailed task files and orchestration guidance (prompt: `sc.split.tasks.md`)
- `Implement` - run implementation from a completed task set (prompt: `sc.implement.tasks.md`)
- `Finalize` - post-implementation actions: update docs, capture deferred items to `docs/feedback/`, and post product-manager guidance to the PR if available (prompt: `sc.finalize.md`)
- `Critique` - conduct the pre-split, specification-level critique (prompt: `sc.critique.spec.md`, spec file as a parameter)
- `Code Review` - review implementation (prompt: `sc.code.review.md`)
- `Archive` - archive older project documentation (prompt: `sc.archive.md`)
- `Brainstorm` - extended brainstorming (prompt: `sc.brainstorm.md`, problem as a parameter)
- `Seed` - convert final brainstorming results to a seed (prompt: `sc.brainstorm.to.seed.md`)
- `Gate Input` - sanitize raw input, extract clean seed specs, and generate a PM feedback report (prompt: `sc.gate.input.md`, input document path as parameter)

## Advanced Commands (Orca-dependent)

These commands require the `/orchestration` skill and an available adversarial agent.

- `Handoff Critique` - hand off a critique to an adversarial agent (Antigravity) in a new Orca terminal, then apply the resulting report (prompt: `sc.handoff.critique.md`, allowed arguments: `--auto`, `--task-level`). The default is the manual flow's specification-level pass. With `--auto`, only 🎯 Must-Address findings are applied automatically (💡 Recommendations and 🤔 Questions are preserved for human review), consistently across the specification-level and task-level passes. `--task-level` reviews the saved spec plus `docs/tasks/<spec-name>/` and is reserved for Autoflow's second pass.
- `Handoff Code Review` - hand off code review to an adversarial agent (Antigravity) in a new Orca terminal, then act on the result (prompt: `sc.handoff.code.review.md`, allowed argument: `--auto`)
- `Autoflow` - polymorphically drive the pipeline end-to-end: optionally start from a GitHub issue (issue → seed; add `--gate-input` to sanitize noisy issues via `Gate Input`), finalize brainstorms and/or derive a seed if needed, plan and handoff-critique the concise Full Spec (`--auto`), **mandatorily split tasks**, run an Autoflow-only task-level handoff critique (`--task-level --auto`), implement, handoff-code-review (`--auto`), and finalize. It commits at each stage (prompt: `sc.autoflow.md`, arguments: an optional GitHub issue reference, an optional `--gate-input` flag, and free-form user refining notes)

All prompts reside in `<skill-dir>/prompts/`.

## Frontmatter / Artifact Contract

All artifacts this kit produces or consumes (seed, spec, questionnaire, task, critique, brainstorm) carry YAML frontmatter conforming to the **canonical schema** in `references/frontmatter.md`. That document is authoritative; where a prompt names a frontmatter field, its definition there governs. Do not introduce per-document frontmatter keys outside the schema. Lifecycle state lives in frontmatter (`status`, `draft`, `consumed`, …); progress narration and in-body mirrors (e.g. a task's `- [ ]` checkbox) defer to the frontmatter value. This schema is shared with the companion `scartill-syseng-lite` skill, which ships an identical copy of `references/frontmatter.md`.

Upon activation, remember these commands, but do not run until an explicit user request.

# Guidance

## Specification Workflow: Seed vs Full Specs

This project uses a two-stage specification process. Both live under `docs/`.

### Seed Specs (`docs/seed/`)

A seed spec is a **concise, human-written intent document**. It captures:
- What the user wants (feature intent, desired behavior)
- Key examples and expected output formats
- High-level CLI interface or config shape
- Follow-up tasks (amend README, add examples, etc.)

Seed specs are informal, written in the user's voice, and may contain typos or shorthand. They do **not** include:
- Detailed implementation guidance
- Internal data models or code structure
- Explicit task breakdowns with test requirements
- Mermaid diagrams or architecture decisions
- Background on existing codebase internals

A seed spec is the **input** to the design phase.

### Full Specs (`docs/specs/`)

A Full Spec is a **concise, critique-ready design blueprint** derived from a seed spec. It includes:
- **Problem Statement**: Precise restatement of the requirement
- **Requirements**: Exhaustive, user-observable acceptance criteria
- **Background**: Relevant codebase internals (existing patterns, modules, data structures)
- **Proposed Solution**: Architecture with Mermaid diagrams, data models, interfaces, and only necessary code sketches
- **Testing & Risk Strategy**: Critical paths, test levels, edge/failure cases, test-data needs, and security/performance/operational risks with mitigations
- **Task Outline**: Numbered objectives, dependencies, and intended outcomes — not mechanical implementation recipes
- **Resolved Decisions**: Traceability to the planning questionnaire's answers or applied defaults

Full Specs support the required **pre-split** product/engineering critique. They intentionally do **not** contain exact per-task files, implementation steps, fixtures, assertions, demo commands, or parallelization instructions.

### Detailed Task Sets (`docs/tasks/<spec-name>/`)

`Split Tasks` is mandatory after a Full Spec is saved and before implementation. It expands the spec's Task Outline into the authoritative execution detail: individual task objectives, exact files and patterns, test requirements and fixtures/assertions, demo commands, dependencies, parallelization guidance, and a coverage map back to the spec's acceptance and risk strategy. Implementers execute this task set, not the Full Spec alone.

### Workflow

1. User writes a seed spec in `docs/seed/` to capture intent.
2. `Plan Spec` produces a Full Spec in `docs/specs/` through a persistent, iterative questionnaire file (`docs/plans/<spec-name>.questions.md`) rather than interactive in-terminal questioning.
3. Critique the concise Full Spec once, before splitting. This is the complete manual critique flow.
4. Run mandatory `Split Tasks` to create `docs/tasks/<spec-name>/` before implementation.
5. Implement from the detailed task set. **Autoflow only** adds a second, auto-applied task-level critique after splitting and before implementation.

When asked to implement a feature, look for the seed (intent), Full Spec (design and acceptance/risk strategy), and detailed task set (execution). If only a seed exists, offer `Plan Spec`; if a spec exists without tasks, run `Split Tasks` before implementation.
