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

1. **Full spec, already critiqued** — a full spec exists in `docs/specs/` **and** a critique
   for it has already been produced and reviewed (a critique file exists in
   `docs/critiques/` for this spec, or the user indicates they already ran a critique
   handoff *without* `--auto` and reviewed/applied it themselves). In this state the
   critique step is **skipped** — do not run a second critique pass.
2. **Full spec, not yet critiqued** — a full spec for the feature under discussion exists in
   `docs/specs/`, or a full spec was interactively planned in this session, and no critique
   has been applied yet.
3. **Seed(s) only** — one or more seed specs exist in `docs/seed/` (produced in this
   session via `Brainstorm`/`Seed` or `Gate Input`), but no corresponding full spec.
4. **Neither** — no seed and no full spec relevant to the current work.

If the state is **Neither**, stop and tell the user to run `Brainstorm`, `Gate Input`, or
`Seed` first. Do not guess intent or fabricate a seed.

When uncertain whether an existing critique is current (e.g. the spec changed after the
critique), prefer re-running the critique; but if the user explicitly states the critique is
done, honor that and skip it.

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
2. **Critique (skip if already critiqued).** If the spec has **not** yet been critiqued, run
   `Handoff Critique --auto` (prompt: `sc.handoff.critique.md`) — this hands the spec to an
   adversarial agent via Orca and applies the resulting critique automatically once ready —
   then **commit** the critiqued spec with a message such as
   `spec: apply critique for <spec-name>`.

   If the state is **Full spec, already critiqued** (the user already ran a critique handoff
   without `--auto` and reviewed it, or a current critique already exists in
   `docs/critiques/`), **skip** this step and proceed directly to Split Tasks. If the user
   already applied and committed the critique, there is nothing to commit here.
3. Run `Split Tasks` (prompt: `sc.split.tasks.md`) to decompose the spec into
   `docs/tasks/<spec-name>/`.
4. **Commit** the task breakdown with a message such as
   `tasks: split <spec-name> into tasks`.
5. Run `Implement` (prompt: `sc.implement.tasks.md`) to execute the tasks via subagents,
   using the `summary.md` for parallelization guidance.
6. **Commit** the implementation with a message such as
   `feat: implement <spec-name>`.
7. Run `Handoff Code Review --auto` (prompt: `sc.handoff.code.review.md`). This hands the
   implemented changes to an adversarial agent via Orca and applies the actionable
   (`CRITICAL`/`HIGH`) findings automatically once the review is ready.
8. **Commit** the review fixes with a message such as
   `fix: apply code review for <spec-name>`. (Skip this commit if the review produced no
   changes.)
9. Run `Finalize` (prompt: `sc.finalize.md`) to update `README.md`/docs, capture deferred
   items to `docs/feedback/`, and post product-manager guidance to the PR if available.
10. **Commit** the finalization with a message such as
    `docs: finalize <spec-name>`. (Skip this commit if there was nothing to update.)

## Operating Constraints

- Run autonomously end-to-end. Do not pause between stages for confirmation.
- Follow the project's git safety norms: stage specific files, do not push, do not force,
  and do not touch unrelated changes. Create new commits (never amend) at each commit point.
- If a stage fails, stop at that stage, report what failed and what was committed so far,
  and leave the working tree in a recoverable state. Do not skip ahead past a failed stage.
- Autoflow runs the full cycle through `Finalize`. Once complete, report every commit made
  and the final state so the user can review and push.
