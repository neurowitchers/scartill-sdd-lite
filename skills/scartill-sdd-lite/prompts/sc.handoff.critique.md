# Hand Off Critique to Adversarial Agent

Use the Orca `/orchestration` skill to start a supervised worker running Antigravity and have
it execute the Scartill SDD Lite `Critique` command against the requested scope. The agent
must write a new report in `docs/critiques/` and use relative paths only.

Launch the worker non-interactively: pass `--agent antigravity` and the launch arguments
`--mode auto --dangerously-skip-permissions` so the worker can run its `orca orchestration`
reporting calls without raising an interactive permission prompt. Without this, the worker's
completion signal stalls on a confirmation prompt and breaks an autonomous (`--auto`) run.

The worker is a supervised Orca dispatch: its prompt carries Task and Dispatch IDs, so it is
obligated to report back. Instruct it in the task spec to do its job, write the report, and
**settle with exactly one `worker_done` signal** (`--outcome succeeded` when the report is
written, `--outcome failed` otherwise) — and to **not** use `ask` or send any free-form
messages back. One expected completion signal, nothing else.

## Command Arguments

Supplied arguments: $ARGUMENTS

Allowed arguments:
- `--auto` — apply eligible critique findings without waiting for the user's confirmation.
- `--task-level` — run the **Autoflow-only second pass** against both the saved Full Spec and
  its detailed task directory.

The flags may be combined: `Handoff Critique --task-level --auto`.

## Scope Selection

### Default: Specification-Level Pass

Without `--task-level`, hand off the normal pre-split critique. It reviews the saved Full Spec
in `docs/specs/` and is the only required critique pass in the manual flow. It evaluates the
specification's requirements, architecture, acceptance criteria, and Testing & Risk Strategy;
it must not demand the per-task file/fixture/assertion/demo-command detail that `Split Tasks`
generates later.

### `--task-level`: Autoflow-Only Second Pass

With `--task-level`, first identify:

- the saved Full Spec: `docs/specs/<spec-name>.md`; and
- the detailed task set: `docs/tasks/<spec-name>/`, including `summary.md`.

Both artifacts are required. If either is missing, stop and report the blocker; do not launch
the handoff and do not fall back to a spec-only critique. This pass runs only after mandatory
`Split Tasks`, critiques the spec and tasks together, and is **not** recommended or required
for the manual workflow.

Tell Antigravity to run `Critique` in `--task-level` mode, explicitly providing both artifact
paths. It must assess the normal product/engineering concerns plus task traceability,
implementation readiness, task-level verification evidence, orchestration, dependencies, and
parallel-work risks. Findings must name the affected spec section and/or task file.

## Handoff Workflow

1. **Locate artifacts.** Find the latest relevant saved spec. For task-level mode, validate its
   matching task directory and `summary.md` exist. If a spec has not been saved, call `Save
   Spec` before handing off.
2. **Launch the worker via Orca.** Start the Antigravity worker with `worker-start`, passing
   `--mode auto --dangerously-skip-permissions`. In the task spec, state the pass scope, spec
   path, and (for task-level mode) task-directory path; tell it to create a new critique file
   rather than overwrite prior reports, and to settle with a single `worker_done` (no `ask`,
   no free-form messages). If this is not the first critique pass, pass that fact along so the
   report is clearly distinguishable.
3. **Yield and resume, or ask.** With `--auto`, **end your turn** after launching — do not poll
   or watch. Orca wakes you when the worker settles with `worker_done`; only then locate the
   latest corresponding report in `docs/critiques/` and continue, without asking for
   confirmation. Without `--auto`, ask the user whether the critique is ready and proceed when
   they confirm.
4. **Apply findings.**
   - **Specification-level mode:** follow the critique report's remediation flow. Without
     `--auto`, obtain user approval (apply 🎯 Must-Address and 💡 Recommendations as the user
     selects). **With `--auto`, do not ask for confirmation and apply only 🎯 Must-Address
     findings**, retaining 💡 Recommendations and 🤔 Questions in the report for human review —
     the same Must-Address-only rule as the task-level pass, so the earlier pass never applies
     more than the later one.
   - **Task-level mode with `--auto`:** apply **only** 🎯 Must-Address findings. Apply fixes to
     the affected task files by default. Amend the Full Spec only if a finding changes intent,
     architecture, acceptance criteria, or the Testing & Risk Strategy. Leave 💡
     Recommendations and 🤔 Questions recorded in the critique report for human review.
   - **Task-level mode without `--auto`:** present the findings and ask the user what to apply.
5. **Close handoff.** A supervised worker settles and releases itself on `worker_done`; it
   does not need to be told to exit. After the critique has been applied or recorded, if the
   worker is still open (e.g. it was kept open), release it via Orca.

## Operating Constraints

- The default manual path remains exactly one, pre-split specification-level critique.
- `--task-level` is reserved for Autoflow's required second pass after `Split Tasks`.
- Always use the latest critique report for the requested scope; never overwrite an existing
  report.
- Do not implement production code during critique remediation unless an approved finding
  explicitly requires it. This stage corrects planning artifacts.
