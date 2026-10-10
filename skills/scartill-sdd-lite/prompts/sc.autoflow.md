# Autoflow

Drive the specification-driven development pipeline end-to-end, autonomously, with no
interactive pauses. Autoflow is **polymorphic**: it inspects the current session and the
`docs/` tree to determine where in the workflow it should start — from a filled-in but
unprocessed brainstorm, a completed brainstorm, a seed, or a full spec — then runs forward
from there to a working, committed implementation.

This command depends on the `orca-cli` orchestration skill because it invokes
`Handoff Critique` with the `--auto` option.

## Command Arguments

Supplied arguments: $ARGUMENTS

Any supplied arguments are treated as **user refining notes** — free-form guidance that
shapes planning and critique application (e.g., scope constraints, priorities, naming,
things to avoid). Incorporate these notes into every planning and spec-refinement step.

## State Detection

Before doing anything, determine the starting state. Autoflow can enter at several points
in the pipeline; the states below are ordered **earliest → latest**. Scan the `docs/` tree
from the most-upstream artifact and **start at the earliest state whose artifact is
present** — do not skip an unfinished upstream artifact just because a later one also
exists. From the chosen entry point, Autoflow falls through each subsequent branch to a
committed implementation.

1. **Brainstorm inputs ready, not yet applied** — a brainstorm report exists in
   `docs/brainstorms/` whose Phase 2 `USER_INPUT` placeholders have been **filled** by the
   user (the default `[REPLACE THIS TEXT WITH YOUR ANSWER / PREFERENCE]` text is gone), but
   the report has **not** yet been carried through Phases 3–4 (its `## Recommendation`
   section is still the empty template). Autoflow resumes the brainstorm from Phase 3.
2. **Brainstorm done, no seed** — a brainstorm report in `docs/brainstorms/` has a populated
   `## Recommendation` section (Phases 3–4 completed), but no corresponding seed exists in
   `docs/seed/`. Autoflow derives a seed from the brainstorm, following its recommendation.
3. **Seed(s) only** — one or more seed specs exist in `docs/seed/` (produced in this
   session via `Brainstorm`/`Seed` or `Gate Input`), but no corresponding full spec.
4. **Full spec, not yet critiqued** — a full spec for the feature under discussion exists in
   `docs/specs/`, or a full spec was interactively planned in this session, and no critique
   has been applied yet.
5. **Full spec, already critiqued** — a full spec exists in `docs/specs/` **and** a critique
   for it has already been produced and reviewed (a critique file exists in
   `docs/critiques/` for this spec, or the user indicates they already ran a critique
   handoff *without* `--auto` and reviewed/applied it themselves). In this state the
   critique step is **skipped** — do not run a second critique pass.
6. **Neither** — no brainstorm, seed, or full spec relevant to the current work.

If the state is **Neither**, stop and tell the user to run `Brainstorm`, `Gate Input`, or
`Seed` first. Do not guess intent or fabricate a brainstorm, seed, or spec.

If a brainstorm report exists but its `USER_INPUT` placeholders are **still unfilled** (they
retain the default placeholder text), treat it as **Neither**: stop and tell the user to
fill in their answers in `docs/brainstorms/` first. Do not fabricate the user's answers.

When uncertain whether an existing critique is current (e.g. the spec changed after the
critique), prefer re-running the critique; but if the user explicitly states the critique is
done, honor that and skip it.

## Branch 4 — Brainstorm inputs ready, not yet applied

If a brainstorm report in `docs/brainstorms/` has its `USER_INPUT` placeholders filled but
has not been carried through Phases 3–4:

1. **Resume the brainstorm** from Phase 3 (prompt: `sc.brainstorm.md`). Parse the user's
   answers inside the `USER_INPUT` tags, formulate the architectural approaches, apply the
   comparative analysis, and deliver the final recommendation and next steps — completing
   Phases 3 and 4 of the report. Apply the user refining notes. Do not pause for
   clarification; the user's placeholder answers are the input you act on.
2. Write the completed `BRAINSTORM_REPORT`-style file back to `docs/brainstorms/`.
3. **Commit** the finalized brainstorm with a message such as
   `brainstorm: finalize analysis for <name>`.
4. Proceed to **Branch 3**.

## Branch 3 — Brainstorm done, no seed

If a brainstorm report has a populated `## Recommendation` section but no corresponding seed
exists in `docs/seed/`:

1. Run `Seed` (prompt: `sc.brainstorm.to.seed.md`) to transform the brainstorm results into
   a seed specification, **following the brainstorm's recommended approach** unless the user
   refining notes specify otherwise. Capture all decisions without the implementation detail
   of a full spec.
2. Write the seed to `docs/seed/`.
3. **Commit** the new seed with a message such as
   `seed: derive seed from brainstorm for <name>`.
4. Proceed to **Branch 2**.

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
