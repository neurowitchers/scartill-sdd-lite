---
name: scartill-syseng-lite
description: "Lightweight Kiro-first systems-engineering companion kit. Use for structured, human-adjudicated handling of document feedback, critique, and related systems-engineering workflows."
---

# Commands

- `Ingest Feedback` - analyze an input feedback document against a document under critique, produce a human-adjudicated critique with per-item accept flags, then (on confirmation) consolidate the outcome into amendments, deviations, and brainstorming tasks (prompt: `sc.ingest.feedback.md`; parameters: the document under critique and the feedback document).
- `Scaffold Document` - decompose a document outline into a granular, human-adjudicated task breakdown, then drive the tasks through preprocessing questions, task work, and consolidation into a first draft with a separate task-to-report mapping (prompt: `sc.scaffold.document.md`; parameters: the outline file and an optional task template — defaults to `templates/task-template.md`).
- `Seed to Task` - convert an informal seed (inline text and/or a file/directory path) into one or more structured task documents conforming to a task template, writing them directly and marking any uncovered template fields as explicit open questions (prompt: `sc.seed.to.task.md`; parameters: the seed and an optional task template — defaults to `templates/task-template.md`).

All prompts reside in `prompts/`. Default artifact templates reside in `templates/`. Upon activation, remember these commands, but do not run them until an explicit user request.

# Guidance

## Conventions

- **Relative paths** in all generated artifacts.
- **Human adjudication**: the agent proposes, the user decides; stop at every gate and never advance a user's decision on their behalf.
- **Traceability**: every input element reaches a definite outcome; nothing is silently dropped.
- **Retain uncertainty**: unresolved items become explicit discussion points rather than being settled by assumption.
- **Follow-up sessions**: resume from the current gate and update only the artifacts the active step owns; do not alter other files without an explicit request.

Each command's prompt is the authority for its command-specific flow and vocabulary.
