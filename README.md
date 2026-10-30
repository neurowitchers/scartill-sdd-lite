# Lite SDD Kit by Scartill

A lightweight, Kiro-first specification-driven development (SDD) kit.

This kit provides a structured, prompt-driven workflow for specification-driven development. It helps orchestrate development sessions through stages of brainstorming, spec writing, task decomposition, implementation, critique, code review, and finalization.

This repository ships **two companion skills** under `skills/`:

- **`scartill-sdd-lite`** — the specification-driven development kit documented below.
- **`scartill-syseng-lite`** — a lightweight systems-engineering companion for structured, human-adjudicated handling of document feedback, critique, and document scaffolding. See [Companion Skill: scartill-syseng-lite](#companion-skill-scartill-syseng-lite).

## Workflow Overview

The kit uses a multi-stage specification workflow to separate intent from execution details:

1. **Brainstorming**: Structured technical exploration of a problem, producing a research report with options and trade-offs.
2. **Seed Specs (`docs/seed/`)**: Concise, human-written documents capturing core user intent, high-level CLI/config shape, and expected behavior.
3. **Full Specs (`docs/specs/`)**: Concise, critique-ready design blueprints derived from seed specs: requirements and acceptance criteria, background, architecture, testing/risk strategy, and ordered task outlines. The **Plan Spec** command drives this expansion through a persistent, iterative questionnaire file (`docs/plans/`) rather than interactive in-terminal questioning.
4. **Task Splitting**: A mandatory post-critique stage that expands task outlines into authoritative, implementation-ready task files (`docs/tasks/`) with exact guidance, verification requirements, demo commands, and orchestration details.
5. **Implementation**: Executing the tasks using subagents.
6. **Critique & Review**: Challenging specifications and reviewing resulting pull requests.
7. **Finalization**: Updating documentation and capturing key session information.

### Frontmatter / Artifact Contract

Every artifact the kit produces or consumes (seed, spec, questionnaire, task, critique, brainstorm) carries YAML frontmatter conforming to a **canonical schema** at `skills/scartill-sdd-lite/references/frontmatter.md`. Lifecycle state lives in frontmatter (`status`, `draft`, `consumed`, …) and is machine-readable across sessions and by `Autoflow`'s state detection; in-body markers such as a task's `- [ ]` checkbox mirror the frontmatter value, which is authoritative. A seed is gated by `draft: true` until the user releases it with `draft: false`. The same schema is shared with the companion `scartill-syseng-lite` skill, which ships an identical copy of the reference.

---

## Commands

The workflow is driven by prompts located in `skills/scartill-sdd-lite/prompts/`. Available commands:

| Command | Prompt | Description |
|---------|--------|-------------|
| **Gate Input** | `sc.gate.input.md` | Sanitizes raw input, extracts clean seed specs, and generates a PM feedback report. Takes an input document path as parameter. |
| **Brainstorm** | `sc.brainstorm.md` | Conducts a rigorous, structured technical brainstorming session. Takes the problem to consider as parameter. |
| **Seed** | `sc.brainstorm.to.seed.md` | Converts final brainstorming results into a seed specification. |
| **Plan Spec** | `sc.plan.spec.md` | Expands seed(s) into a full spec via a persistent, iterative questionnaire file (`docs/plans/<spec-name>.questions.md`), gathering clarifications through file edits instead of interactive in-terminal questioning. Optional parameters: a seed path and/or free-form refining notes. |
| **Save Spec** | `sc.save.spec.md` | Saves a persistent specification to `docs/specs` before implementation. |
| **Split Tasks** | `sc.split.tasks.md` | **Required before implementation.** Expands the concise spec's task outlines into authoritative task files under `docs/tasks/<spec-name>/`, including implementation guidance, verification requirements, demo commands, and orchestration details. |
| **Implement** | `sc.implement.tasks.md` | Orchestrates task implementation using subagents. |
| **Critique** | `sc.critique.spec.md` | Challenges the specification from Product and Engineering perspectives. Takes the file to critique as parameter. |
| **Code Review** | `sc.code.review.md` | Reviews pull requests/branches for correctness, type safety, security, and performance. |
| **Finalize** | `sc.finalize.md` | Post-implementation tasks: updates `README.md`/docs, captures deferred items for future iterations in `docs/feedback/`, and posts product-manager guidance (shipped vs. deferred) to the PR if available. |
| **Archive** | `sc.archive.md` | Archives documents from `docs/` older than a week to `docs/archive`, preserving structure. |

### Advanced Commands (Orca-dependent)

These commands require the [`orca-cli`](https://www.onorca.dev/) orchestration skill and an available adversarial agent.

| Command | Prompt | Description |
|---------|--------|-------------|
| **Handoff Critique** | `sc.handoff.critique.md` | Uses Orca orchestration to launch an adversarial agent (Antigravity, via `agy`) in a new terminal to run the Critique command against the spec, then applies the resulting critique from `docs/critiques/`. Supports `--auto` to apply only 🎯 Must-Address findings automatically once the critique is ready (preserving Recommendations and Questions for human review). `--task-level` additionally critiques the saved spec plus `docs/tasks/<spec-name>/`; this second pass is reserved for Autoflow. |
| **Handoff Code Review** | `sc.handoff.code.review.md` | Uses Orca orchestration to launch an adversarial agent (Antigravity, via `agy`) in a new terminal to run the Code Review command against the current PR/branch, then acts on the resulting review from `docs/codereviews/`. Supports `--auto` to apply CRITICAL/HIGH fixes automatically once the review is ready. |
| **Autoflow** | `sc.autoflow.md` | Polymorphically drives the full pipeline end-to-end and autonomously. Detects the starting state and begins at the earliest available artifact: a supplied GitHub issue (issue → seed; with `--gate-input` the issue is sanitized through Gate Input first), a filled-in brainstorm awaiting Phase 3–4 analysis (which it finalizes), a completed brainstorm with no seed (from which it derives a seed following the recommendation), seed(s) only (from which it plans a Full Spec), an answered plan questionnaire in `docs/plans/` (from which it resumes planning, honoring the user's supplied answers and defaulting only the still-blank questions), or an existing Full Spec. From there it runs specification-level Handoff Critique (`--auto`), mandatory Split Tasks, an Autoflow-only task-level Handoff Critique (`--task-level --auto`, applying only Must-Address findings), Implement, Handoff Code Review (`--auto`), and Finalize, committing at each stage. Accepts an optional GitHub issue reference, an optional `--gate-input` flag, and free-form user refining notes as arguments. |

---

## File Structure

```
skills/scartill-sdd-lite/
├── SKILL.md              # Skill definition with command list and workflow guidance
├── prompts/              # Prompt templates for each workflow stage
└── references/           # Canonical frontmatter / artifact schema (shared, synced with the companion skill)

skills/scartill-syseng-lite/
├── SKILL.md              # Companion systems-engineering skill definition
├── prompts/              # Prompt templates (Ingest Feedback, Scaffold Document)
└── templates/            # Default artifact templates (task template)

docs/                     # Specification artifacts (created per-project)
├── seed/                 # Initial human-written seed specifications
├── plans/                # Persistent planning questionnaires (Plan Spec: <spec-name>.questions.md)
├── specs/                # Full implementation specifications
├── tasks/                # Detailed authoritative task files (per-spec subdirectories)
├── brainstorms/          # Brainstorming research reports
├── critiques/            # Reports generated by spec critiques
├── codereviews/          # Reports generated by code reviews
├── feedback/             # Deferred items captured at finalization for future iterations
└── archive/              # Archived documentation
```

---

## Installation

Copy or symlink the skill directories under `skills/` (`scartill-sdd-lite` and, optionally, its companion `scartill-syseng-lite`) into your Kiro skills folder (`~/.kiro/skills/`). The commands will be available in any Kiro CLI session.

---

## Typical Workflow

```
Brainstorm → Seed → Plan Spec → Save Spec → Critique → Split Tasks → Implement → Finalize
```

Or starting from external input:

```
Gate Input → Seed → Plan Spec → Save Spec → Critique → Split Tasks → Implement → Code Review → Finalize
```

These diagrams show the **manual flow**: one specification-level critique before mandatory
Split Tasks. Autoflow adds one automated step after Split Tasks — `Handoff Critique
--task-level --auto` — before implementation; that second pass is not required or recommended
for the manual workflow.

---

## Companion Skill: scartill-syseng-lite

*The companion skill is work in progress*

A lightweight, Kiro-first systems-engineering companion kit for structured, human-adjudicated handling of document feedback, critique, and document scaffolding. It lives at `skills/scartill-syseng-lite/` and is designed to sit alongside the SDD kit: the SDD kit drives code specifications, while the systems-engineering kit drives narrative documents and their feedback loops.

### Commands

The commands are driven by prompts in `skills/scartill-syseng-lite/prompts/`.

| Command | Prompt | Description |
|---------|--------|-------------|
| **Ingest Feedback** | `sc.ingest.feedback.md` | Analyze an input feedback document against a document under critique, produce a human-adjudicated critique with per-item accept flags, then (on confirmation) consolidate the outcome into amendments, deviations, and brainstorming tasks. Parameters: the document under critique and the feedback document. |
| **Scaffold Document** | `sc.scaffold.document.md` | Decompose a document outline into a granular, human-adjudicated task breakdown, then drive the tasks through preprocessing questions, task work, and consolidation into a first draft with a separate task-to-report mapping. Parameters: the outline file and an optional task template (defaults to `templates/task-template.md`). |
| **Seed to Task** | `sc.seed.to.task.md` | Convert an informal seed (inline text and/or a file/directory path) into one or more structured task documents conforming to a task template. Writes directly and marks any uncovered template fields as explicit open questions. Parameters: the seed and an optional task template (defaults to `templates/task-template.md`). |

### Conventions

- **Relative paths** in all generated artifacts.
- **Human adjudication**: the agent proposes, the user decides; it stops at every gate and never advances a user's decision on their behalf.
- **Traceability**: every input element reaches a definite outcome; nothing is silently dropped.
- **Retain uncertainty**: unresolved items become explicit discussion points rather than being settled by assumption.
- **Follow-up sessions**: resume from the current gate and update only the artifacts the active step owns.

### File Structure

```
skills/scartill-syseng-lite/
├── SKILL.md              # Companion systems-engineering skill definition
├── prompts/              # Ingest Feedback, Scaffold Document, Seed to Task
└── templates/            # Default task template
```
