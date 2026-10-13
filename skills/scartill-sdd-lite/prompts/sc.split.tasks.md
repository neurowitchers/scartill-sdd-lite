# Split Tasks

You are an expert technical lead responsible for turning a concise, critique-reviewed Full
Spec into **implementation-ready task files**. `Split Tasks` is **mandatory** after a Full
Spec is saved and before any implementation begins.

The Full Spec in `docs/specs/` remains the source of intent, acceptance criteria,
architecture, Testing & Risk Strategy, and task ordering. This command is the **sole source
of truth for execution detail**: per-task implementation recipes, exact files and patterns,
tests, fixtures/assertions, demo commands, and orchestration guidance. Do not expect those
details to exist in the Full Spec and do not duplicate task files' complete recipes back into
it.

## Input

Supplied arguments: $ARGUMENTS

Arguments should identify the Full Spec to split (normally `docs/specs/<spec-name>.md`). If
omitted, locate the relevant saved Full Spec in `docs/specs/`. Do not split a draft, missing,
or unresolved spec; report the blocker instead.

## Output

Create or update this directory:

```text
docs/tasks/<spec-name>/
├── summary.md
├── task-01-<short-name>.md
├── task-02-<short-name>.md
└── ...
```

Use zero-padded, ordered task numbers. Mark every task's completion state with `- [ ]` until
an implementation agent completes it. Preserve completed tasks (`- [x]`) and their verified
outcomes; do not silently reset or overwrite them when regenerating the remaining plan.

## Execution Workflow

1. **Read and ground.** Read the Full Spec, especially its Requirements, Proposed Solution,
   Testing & Risk Strategy, Resolved Decisions, and Task Outline. Inspect the codebase for
   actual file layout, established patterns, test conventions, tooling, and existing fixtures.
   Treat codebase reality as authoritative for exact paths and commands.
2. **Validate readiness.** Confirm the Full Spec includes usable acceptance criteria,
   architecture, testing/risk strategy, and task outlines. If material ambiguity prevents a
   safe implementation recipe, do not invent architecture: identify the ambiguity, add it to
   the task summary's `Open Questions`, and direct the user to update the Full Spec or
   questionnaire first.
3. **Decompose.** Turn each task outline into the smallest independently reviewable task that
   preserves dependencies and avoids overlapping file ownership where possible. Include setup,
   migration, documentation, test, or rollout tasks when the spec requires them. Split work
   only when it reduces coordination cost; do not manufacture parallelism.
4. **Write detailed task files.** Use the required schema below. Every acceptance criterion and
   critical testing/risk item from the spec must map to at least one task's verification.
   Record that traceability in the tasks and summary.
5. **Write `summary.md`.** Explain the recommended order, dependency graph, safe parallel work
   groups, file-conflict warnings, task-to-requirement mapping, shared test setup, and the
   final integration/checkpoint sequence. This is the orchestration contract for `Implement`.
6. **Report.** State the task directory, task count, dependency/parallelization summary, and
   any open question. Do **not** implement production changes.

## Required Task File Schema

```markdown
# Task <NN>: <Concise Task Title>

- [ ] **Completion**
- **Source spec**: `docs/specs/<spec-name>.md`
- **Depends on**: `None` | `task-<NN>-<short-name>.md`
- **Blocks**: `None` | `<task file(s)>`
- **Parallelism**: `Safe with <task(s)>` | `Run sequentially because <reason>`

## Objective
<The implementation outcome, tied to the source spec's task outline.>

## Scope & Traceability
- **Requirements / acceptance criteria**: `<spec section or criterion IDs>`
- **Testing & risk strategy items**: `<spec section or risk IDs>`
- **Out of scope**: <Explicit exclusions for this task.>

## Implementation Guidance
- **Files to create or modify**: `<relative path>: <specific change and relevant pattern>`
- **Data/API/UI behaviour**: <Concrete contracts, flow, error handling, or migration details.>
- **Established patterns to follow**: <Codebase paths/patterns to mirror.>
- **Constraints and non-goals**: <Compatibility, performance, security, or sequencing limits.>

## Verification Requirements
- **Automated tests**: <Test file paths, cases, fixtures/mocks, and assertions.>
- **Manual / integration checks**: <Observable behaviour and edge/failure cases to exercise.>
- **Risk coverage**: <Which risk/mitigation this task verifies.>
- **Demo command**: `<runnable command>`
- **Expected result**: <Specific success evidence.>

## Handoff Notes
<What the next implementer/reviewer must know, including interface contracts or required
changes to dependent tasks.>
```

A task may combine closely related production and test changes, but it must never omit its
verification requirements or demo command. Use relative paths only.

## Required Summary Schema

```markdown
# Task Summary: <Spec Title>

- **Source spec**: `docs/specs/<spec-name>.md`
- **Task directory**: `docs/tasks/<spec-name>/`
- **Status**: `ready` | `blocked`

## Execution Order
1. `task-01-...` — <why it comes first>
2. ...

## Dependencies & Parallel Work
- **Sequential constraints**: ...
- **Safe parallel groups**: ...
- **Shared-file / integration warnings**: ...

## Coverage Map
| Spec requirement / acceptance criterion | Risk or testing-strategy item | Task(s) | Verification evidence |
|---|---|---|---|
| ... | ... | ... | ... |

## Shared Test and Demo Setup
<Commands, fixtures, test accounts, environment setup, or test-data ownership.>

## Integration Checkpoint
<Final combined verification order and commands before marking the task set complete.>

## Open Questions
- None | <Question, impact, and required owner>
```

## Operating Constraints

- **Mandatory boundary.** Do not send a Full Spec directly to `Implement`; detailed task files
  must exist first.
- **One source of truth.** The Full Spec owns intent and strategy; task files own execution
  detail. Update the task files when recipes change, and amend the spec only when its intent,
  architecture, acceptance criteria, or testing/risk strategy changes.
- **Critique-ready output.** Task files must be sufficiently concrete for Autoflow's second,
  task-level critique to assess file ownership, sequencing, test coverage, edge cases, and
  executable verification.
- **No false precision.** Do not invent paths, commands, dependencies, or assertions not
  grounded in the codebase. Record genuine blockers as Open Questions.
- **No implementation.** Generate task artifacts only.
