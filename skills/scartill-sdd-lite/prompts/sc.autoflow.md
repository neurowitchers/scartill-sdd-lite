# Autoflow

Drive the specification-driven development pipeline end-to-end, autonomously, with no
interactive pauses. Autoflow is **polymorphic**: it inspects the current session and the
`docs/` tree to determine where in the workflow it should start, then runs forward from
there to a working, committed implementation.

This command depends on the `orca-cli` orchestration skill because it invokes
`Handoff Critique` with the `--auto` option.

## Command Arguments

Supplied arguments: $ARGUMENTS

Any supplied arguments are treated as **user refining notes** — free-form guidance that
shapes planning and critique application (e.g., scope constraints, priorities, naming,
things to avoid). Incorporate these notes into every planning and spec-refinement step.

## State Detection

Before doing anything, determine the starting state:

1. **Full spec available** — a full spec for the feature under discussion exists in
   `docs/specs/`, or a full spec was interactively planned in this session.
2. **Seed(s) only** — one or more seed specs exist in `docs/seed/` (produced in this
   session via `Brainstorm`/`Seed` or `Gate Input`), but no corresponding full spec.
3. **Neither** — no seed and no full spec relevant to the current work.

If the state is **Neither**, stop and tell the user to run `Brainstorm`, `Gate Input`, or
`Seed` first. Do not guess intent or fabricate a seed.

## Branch 2 — Seed(s) only

If only seed(s) are available (and no full spec yet):

1. Perform **non-interactive planning** to expand the seed(s) into a Full Spec, following
   the skill's "Seed vs Full Specs" guidance. Produce a complete implementation blueprint
   (problem statement, requirements, background, proposed solution, task breakdown).
   Apply the user refining notes. Do not pause for clarification; make reasonable,
   well-documented assumptions and record them in the spec.
2. Save the Full Spec to `docs/specs/` (equivalent to `Save Spec`).
3. **Commit** the new full spec with a message such as
   `spec: plan full spec for <spec-name>`.
4. Proceed to **Branch 1**.

## Branch 1 — Full spec available

If a full spec was interactively planned or already exists:

1. Ensure the spec is saved to `docs/specs/` (call `Save Spec` if it has not been saved).
2. Run `Handoff Critique --auto` (prompt: `sc.handoff.critique.md`). This hands the spec to
   an adversarial agent via Orca and applies the resulting critique automatically once ready.
3. **Commit** the critiqued spec with a message such as
   `spec: apply critique for <spec-name>`.
4. Run `Split Tasks` (prompt: `sc.split.tasks.md`) to decompose the spec into
   `docs/tasks/<spec-name>/`.
5. **Commit** the task breakdown with a message such as
   `tasks: split <spec-name> into tasks`.
6. Run `Implement` (prompt: `sc.implement.tasks.md`) to execute the tasks via subagents,
   using the `summary.md` for parallelization guidance.
7. **Commit** the implementation with a message such as
   `feat: implement <spec-name>`.
8. Run `Handoff Code Review --auto` (prompt: `sc.handoff.code.review.md`). This hands the
   implemented changes to an adversarial agent via Orca and applies the actionable
   (`CRITICAL`/`HIGH`) findings automatically once the review is ready.
9. **Commit** the review fixes with a message such as
   `fix: apply code review for <spec-name>`. (Skip this commit if the review produced no
   changes.)
10. Run `Finalize` (prompt: `sc.finalize.md`) to update `README.md` and any documentation
    affected by user-facing or configuration changes.
11. **Commit** the finalization with a message such as
    `docs: finalize <spec-name>`. (Skip this commit if there was nothing to update.)

## Operating Constraints

- Run autonomously end-to-end. Do not pause between stages for confirmation.
- Follow the project's git safety norms: stage specific files, do not push, do not force,
  and do not touch unrelated changes. Create new commits (never amend) at each commit point.
- If a stage fails, stop at that stage, report what failed and what was committed so far,
  and leave the working tree in a recoverable state. Do not skip ahead past a failed stage.
- Autoflow runs the full cycle through `Finalize`. Once complete, report every commit made
  and the final state so the user can review and push.
