# Hand Off Critique to Adversarial Agent

Hand the critique to an adversarial agent (Antigravity by default) and have it execute the
Scartill SDD Lite `Critique` command against the requested scope. The agent must write a new
report in `docs/critiques/` and use relative paths only.

**Dispatch mode depends on `--auto`:**

- **With `--auto` (direct subagent):** do **not** use Orca and do not open a terminal. Call the
  adversarial agent directly as a synchronous subagent through the host's subagent mechanism
  (for example `invoke_sub_agent`), with a fresh context. If the host cannot run subagents, run
  the adversarial CLI (`agy`) synchronously in the foreground instead. The call blocks until the
  report is written, so there is no yielding, waking, polling, or `worker_done` signal.
- **Without `--auto` (Orca, interactive):** use the Orca `/orchestration` skill to start a
  supervised worker running Antigravity (`--agent antigravity`) in a separate terminal. This mode
  exists **so the user can watch the critic and interact with it** (question findings, push
  back, request changes) **before anything is applied to the spec**; do not collapse it into the
  direct-subagent path. Do not pass `--dangerously-skip-permissions`, and do not forbid `ask` or
  free-form messages. The worker should write the report and may stay open for discussion; the
  user, not a `worker_done` signal, decides when the critique is ready.

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
2. **Dispatch the adversarial agent.** In the task prompt, state the pass scope, spec path, and
   (for task-level mode) task-directory path; tell it to create a new critique file rather than
   overwrite prior reports. If this is not the first critique pass, pass that fact along so the
   report is clearly distinguishable.
   - With `--auto`: invoke it as a direct synchronous subagent (see above).
   - Without `--auto`: start the interactive Orca worker with `worker-start` (see above) and
     tell the user where to interact with it.
3. **Collect the result, or ask.** With `--auto`, the subagent call returns when the report is
   written; if it fails or returns no report, stop and report the failure. Locate the latest
   corresponding report in `docs/critiques/` and continue without asking for confirmation.
   Without `--auto`, ask the user whether the critique is ready and proceed when they confirm.
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
5. **Close handoff.** A direct subagent ends when it returns. An interactive Orca worker
   (non-`--auto`) may still be open; after the critique has been applied or recorded, release it
   via Orca.

## Operating Constraints

- The default manual path remains exactly one, pre-split specification-level critique.
- `--task-level` is reserved for Autoflow's required second pass after `Split Tasks`.
- Always use the latest critique report for the requested scope; never overwrite an existing
  report.
- Do not implement production code during critique remediation unless an approved finding
  explicitly requires it. This stage corrects planning artifacts.
