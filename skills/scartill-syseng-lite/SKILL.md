---
name: scartill-syseng-lite
description: "Lightweight Kiro-first systems-engineering companion kit. Use for structured, human-adjudicated handling of document feedback, critique, and related systems-engineering workflows."
---

# Commands

- `Ingest Feedback` - analyze an input feedback document against a document under critique, produce a human-adjudicated critique with per-item accept flags, then (on confirmation) consolidate the outcome into amendments, deviations, and brainstorming tasks (prompt: `sc.ingest.feedback.md`; parameters: the document under critique and the feedback document).
- `Scaffold Document` - decompose a document outline into a granular, human-adjudicated task breakdown, then drive the tasks through preprocessing questions, task work, and consolidation into a first draft with a separate task-to-results mapping (prompt: `sc.scaffold.document.md`; parameters: the outline file and an optional task template — defaults to `templates/task-template.md`).
- `Seed to Task` - convert an informal seed (inline text and/or a file/directory path) into one or more structured task documents conforming to a task template, writing them directly and marking any uncovered template fields as explicit open questions (prompt: `sc.seed.to.task.md`; parameters: the seed and an optional task template — defaults to `templates/task-template.md`).

All prompts reside in `prompts/`. Default artifact templates reside in `templates/`. Shared references reside in `references/`. Upon activation, remember these commands, but do not run them until an explicit user request.

# Guidance

## Workspace layout

All three commands operate inside a **document workspace** — one directory per document under production (e.g. `fleet-opscon/`, `proposal-science/`). They share one canonical directory shape; this section is the single authority for it. A command creates a directory on first use (lazily), but the names, roles, and the import-only guardrail are fixed here. `Scaffold Document` additionally scaffolds the skeleton up front (its Phase 0).

| Directory       | Role                                                                                     | Guardrail |
|-----------------|------------------------------------------------------------------------------------------|-----------|
| `inputs/`       | Imported reference copies of source documents whose truth lives elsewhere.               | **Read-only / import-only** — carries `inputs/AGENTS.md` (see below). |
| `seed/`         | Informal, human-written intent documents (seeds).                                        | —         |
| `tasks/`        | Structured task documents.                                                               | —         |
| `task-results/` | Per-task output documents (the `Scaffold Document` work products).                       | —         |
| `quest/`        | Preprocessing questions with answer placeholders (`Scaffold Document`).                   | —         |
| `critiques/`    | Human-adjudicated critiques (`Ingest Feedback`).                                          | —         |
| `feedback/`     | Residual deviations consolidated from an adjudicated critique.                            | —         |
| `analysis/`     | Ad-hoc analysis / decision notes.                                                        | —         |
| `results/`      | Consolidated output document(s) and the task-to-results mapping.                         | —         |
| `templates/`    | Optional project-local template overrides; absent means the skill defaults are used.     | —         |

Fixed conventions:

- **Names are canonical.** Use `inputs` (plural) and `results` (not `report` — not every document is a report). Do not invent singular/alternate spellings.
- **Workspace root files.** A scaffolded workspace carries an initial `README.md` at its root — copy `templates/workspace-readme.md` to `<workspace>/README.md`, replacing `{workspace}` with the directory name and `{title}` with a human title for the document, and fill the one-line purpose. It orients a human reader to the layout and to the document under production.
- **`inputs/` is import-only.** Whenever an `inputs/` directory is created, drop the import-only guardrail into it: copy `templates/inputs-agents.md` to `<workspace>/inputs/AGENTS.md`, replacing every `{workspace}` placeholder with the workspace directory name. Imported artifacts record their origin via the `source` / `source-target` provenance frontmatter (`references/frontmatter.md`).
- **Create lazily, never overwrite.** A command creates only the directories it needs, when it needs them; it never overwrites an existing file (sequence-suffix a new one on collision). The workspace `README.md` and `inputs/AGENTS.md` are written once at scaffolding time and never overwritten thereafter.
- **Not every workspace uses every directory.** The shape is a superset; a given document uses the subset its workflow touches.

## Conventions

- **Frontmatter schema**: all artifact frontmatter (seed, spec, questionnaire, task, critique, brainstorm, result) conforms to the canonical schema in `references/frontmatter.md`. That document is authoritative; where a prompt names a frontmatter field, its definition there governs. Do not introduce per-document frontmatter keys outside the schema. This schema is shared with the companion `scartill-sdd-lite` skill, which ships an identical copy of `references/frontmatter.md`.
- **Relative paths** in all generated artifacts.
- **Human adjudication**: the agent proposes, the user decides; stop at every gate and never advance a user's decision on their behalf.
- **Traceability**: every input element reaches a definite outcome; nothing is silently dropped.
- **Retain uncertainty**: unresolved items become explicit discussion points rather than being settled by assumption.
- **Follow-up sessions**: resume from the current gate and update only the artifacts the active step owns; do not alter other files without an explicit request.

Each command's prompt is the authority for its command-specific flow and vocabulary.
