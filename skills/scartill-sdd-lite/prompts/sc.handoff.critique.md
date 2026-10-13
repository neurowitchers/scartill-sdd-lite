# Hand Off Critique to Adversarial Agent

Use the Orca `/orchestration` skill to launch a new terminal running Antigravity (the command
is `agy`) and have it execute the Scartill SDD Lite `Critique` command against the requested
scope. The agent must write a new report in `docs/critiques/` and use relative paths only.

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
2. **Launch Antigravity via Orca.** In the new terminal, state the pass scope, spec path, and
   (for task-level mode) task-directory path. Ask it to create a new critique file rather
   than overwrite prior reports. If this is not the first critique pass, pass that fact along
   so the report is clearly distinguishable.
3. **Wait or auto-apply.** Without `--auto`, ask the user whether the critique is ready. With
   `--auto`, wait for the adversarial agent to finish, locate the latest corresponding report
   in `docs/critiques/`, and continue without asking for confirmation.
4. **Apply findings.**
   - **Specification-level mode:** follow the critique report's remediation flow; obtain user
     approval unless `--auto` was supplied.
   - **Task-level mode with `--auto`:** apply **only** 🎯 Must-Address findings. Apply fixes to
     the affected task files by default. Amend the Full Spec only if a finding changes intent,
     architecture, acceptance criteria, or the Testing & Risk Strategy. Leave 💡
     Recommendations and 🤔 Questions recorded in the critique report for human review.
   - **Task-level mode without `--auto`:** present the findings and ask the user what to apply.
5. **Close handoff.** After the critique has been applied or recorded, ask Antigravity to exit
   and close the handoff terminal.

## Operating Constraints

- The default manual path remains exactly one, pre-split specification-level critique.
- `--task-level` is reserved for Autoflow's required second pass after `Split Tasks`.
- Always use the latest critique report for the requested scope; never overwrite an existing
  report.
- Do not implement production code during critique remediation unless an approved finding
  explicitly requires it. This stage corrects planning artifacts.
