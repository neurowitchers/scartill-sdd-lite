# Scaffold Document

## Goal

Turn a **document outline** into a granular, human-adjudicated task breakdown, then drive those tasks through to a consolidated first draft. The outline is decomposed into individually scoped tasks so the user can supply thoughts and inputs for each; the agent preprocesses tasks into targeted questions, works each task on confirmation, and folds the results into a single draft with a separate task-to-report mapping. This keeps the draft traceable: every outline element maps to a task, and every task maps to a documented place in the report.

## Parameters

- **Outline file** (required) — the document outline to decompose. Its bullet/section structure drives task granularity.
- **Task template** (optional) — a template describing the per-task document shape. If omitted, use the skill default `templates/task-template.md`.

If the outline file is missing, ask the user before proceeding.

## Operating Constraints

- **HUMAN ADJUDICATION**: The agent proposes; the user decides. Confirm the template and the task granularity with the user before generating tasks. Wait for the user at every gate; never advance a gate on the user's behalf.
- **TRACEABILITY**: Every outline element reaches a definite task, and every task reaches a definite place in the report. Nothing is silently dropped. The report-to-task mapping is documented separately.
- **RELATIVE PATHS**: Do not use absolute paths when referencing files in generated artifacts. Use relative paths only.
- **RETAIN UNCERTAINTY**: Unanswered questions become explicit discussion points in the report rather than being resolved by assumption.
- **MULTIPLE CHOICE FIRST**: When generating questions for the user, prefer multiple-choice questions. Avoid open questions where possible.

## Working Directories

Establish a working root for the document (ask the user if not evident from the outline location). Under it:

- Tasks → `tasks/`
- Task results → `task-results/`
- Questions with answer placeholders → `quest/`
- Report and task-to-report mapping → `report/`

Create directories as needed. Never overwrite an existing file — create a new one with a sequence suffix if a name would collide.

## Steps

### Phase I — Template and Granularity

1. Read the outline file. If a task template parameter was supplied, read it; otherwise use the skill default `templates/task-template.md`.
2. Present the task template to the user and refine it until the user is satisfied.
3. Propose the task granularity. Default: one task per bullet item. Confirm with the user before generating tasks.

### Phase II — Task Generation

4. Generate one task document per unit of granularity, following the confirmed template, into `tasks/`. Assign each a stable task ID and record its source outline location.
5. Stop and wait for the user to provide inputs to all tasks.

### Phase III — Dependency Graph

6. Once inputs are in, generate the task interdependency graph. Document it in `tasks/summary.md` with a concise summary of each task and its dependencies.

### Phase IV — Preprocessing Questions

7. Use subagents to preprocess each task and generate questions for the user, written with answer placeholders into `quest/`. Limit to multiple-choice questions; avoid open questions where possible.
8. Stop and wait for the user to confirm the answers are ready. Some questions may remain unanswered — carry these forward as explicit discussion points and retain the uncertainty in the report.

### Phase V — Task Work

9. On confirmation, use subagents to work each task, writing results into `task-results/`.

### Phase VI — Consolidation

10. Consolidate the first draft of the document into `report/`. The report must not reference tasks or their identifiers.
11. Document the mapping between tasks and the report in a separate file under `report/`.

## Output

- Task documents → `tasks/`.
- Task dependency summary → `tasks/summary.md`.
- Questions with answer placeholders → `quest/`.
- Task results → `task-results/`.
- Consolidated draft → `report/`.
- Task-to-report mapping → `report/` (separate file; the report itself carries no task references).

## Follow-up Sessions

In a follow-up session, resume from the current gate. Update only the artifacts the current phase owns; do not alter other files without an explicit request.
