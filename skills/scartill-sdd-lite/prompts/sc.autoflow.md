# Autoflow

Drive the specification-driven development pipeline end-to-end, autonomously, with no interactive pauses. Autoflow is **polymorphic**: it inspects the current session, any supplied GitHub issue, and the `docs/` tree to determine where in the workflow it should start — from a GitHub issue, a filled-in but unprocessed brainstorm, a completed brainstorm, a seed, an answered plan questionnaire, or a full spec — then runs forward from there to a working, committed implementation.

This command depends on the `orca-cli` orchestration skill because it invokes
`Handoff Critique` with the `--auto` option.

## Command Arguments

Supplied arguments: $ARGUMENTS

Arguments are parsed as follows:

- **GitHub issue reference** — a `--issue <ref>` flag, a bare issue number (`#42` or `42`), or a GitHub issue URL (`https://github.com/<owner>/<repo>/issues/42`). When present, the referenced issue becomes the upstream entry point (see Branch 0). At most one issue reference is honored.
- **`--gate-input`** — a flag, valid only together with an issue reference. It routes the issue body through `Gate Input` (sanitizing noisy or agent-authored issues) instead of deriving a seed directly. Ignored (with a note) if no issue reference is supplied.
- **User refining notes** — everything else is treated as free-form guidance that shapes planning and critique application (e.g., scope constraints, priorities, naming, things to avoid). Incorporate these notes into every planning and spec-refinement step.

## State Detection

Before doing anything, determine the starting state. Autoflow can enter at several points in the pipeline; the states below are ordered **earliest → latest**. **If an issue reference was supplied in the arguments, start at state 0 (GitHub issue) regardless of what exists in `docs/`** — the explicit issue is the entry point. Otherwise, scan the `docs/` tree from the most-upstream artifact and **start at the earliest state whose artifact is present** — do not skip an unfinished upstream artifact just because a later one also exists. From the chosen entry point, Autoflow falls through each subsequent branch to a committed implementation.

0. **GitHub issue supplied** — the arguments include an issue reference (`--issue <ref>`, a bare `#<n>`/`<n>`, or an issue URL). The issue plays the role of a seed. Autoflow fetches the issue and either derives a seed from it directly, or — when `--gate-input` is also supplied — routes the issue body through `Gate Input` first. (See Branch 0.)
1. **Brainstorm inputs ready, not yet applied** — a brainstorm report exists in `docs/brainstorms/` whose frontmatter `status` is `inputs-ready` (the user has filled the Phase 2 `USER_INPUT` placeholders — the default `[REPLACE THIS TEXT WITH YOUR ANSWER / PREFERENCE]` text is gone — but Phases 3–4 are not yet complete). Autoflow resumes the brainstorm from Phase 3. (See `references/frontmatter.md`, artifact type: brainstorm.)
2. **Brainstorm done, no seed** — a brainstorm report in `docs/brainstorms/` has frontmatter `status: recommended` (Phases 3–4 completed, `## Recommendation` populated), but no corresponding seed exists in `docs/seed/`. Autoflow derives a seed from the brainstorm, following its recommendation.
3. **Seed(s) only** — one or more non-draft seeds (frontmatter `draft: false`) exist in `docs/seed/` (produced in this session via `Brainstorm`/`Seed` or `Gate Input` and released by the user), but no corresponding questionnaire or full spec.
4. **Questionnaire answered, no full spec** — a plan questionnaire exists in `docs/plans/<spec-name>.questions.md` with frontmatter `status: awaiting-answers` and the user has supplied at least one answer (at least one `USER_INPUT` block no longer holds the default `[REPLACE THIS TEXT …]` placeholder), but no corresponding full spec exists yet in `docs/specs/`. Autoflow resumes planning from this questionnaire, honoring the user's filled answers and defaulting only the still-blank questions. (See `references/frontmatter.md`, artifact type: questionnaire.)
5. **Full spec, not yet critiqued** — a full spec for the feature under discussion exists in `docs/specs/` with frontmatter `status: draft`, or a full spec was interactively planned in this session, and no critique has been applied yet.
6. **Full spec, already critiqued** — a full spec exists in `docs/specs/` with frontmatter `status: critiqued` (or later) **and** a critique for it has already been produced and reviewed (a critique file exists in `docs/critiques/` for this spec, or the user indicates they already ran a critique handoff *without* `--auto` and reviewed/applied it themselves). In this state the critique step is **skipped** — do not run a second critique pass.
7. **Neither** — no GitHub issue argument, brainstorm, seed, questionnaire, or full spec relevant to the current work.

If the state is **Neither**, stop and tell the user to supply a GitHub issue reference or run `Brainstorm`, `Gate Input`, `Seed`, or `Plan Spec` first. Do not guess intent or fabricate an issue, brainstorm, seed, questionnaire, or spec.

If a brainstorm report exists but its frontmatter `status` is still `drafting` (its `USER_INPUT` placeholders retain the default placeholder text), treat it as **Neither**: stop and tell the user to fill in their answers in `docs/brainstorms/` first. Do not fabricate the user's answers.

When uncertain whether an existing critique is current (e.g. the spec changed after the critique), prefer re-running the critique; but if the user explicitly states the critique is done, honor that and skip it.

## Branch 0 — GitHub issue entry point

If an issue reference was supplied in the arguments, the issue is the upstream entry point and plays the role of a seed.

1. **Resolve and fetch the issue.** Normalize the reference (a `--issue <ref>` flag, a bare `#<n>`/`<n>`, or a full issue URL) and fetch it with `gh` (consistent with the `gh` usage already in `Code Review` and `Finalize`):

   ```bash
   gh issue view <ref> --json number,title,body,url,labels
   ```

   If the fetch fails (issue not found, `gh` not authenticated, or no repo context), **stop** at this stage per the operating constraints and report the failure — do not fabricate issue content.

2. **Derive a seed.** Pick the branch by whether `--gate-input` was supplied:
   - **Without `--gate-input` (default):** write the issue directly to a seed at `docs/seed/<slug>.md`, where `<slug>` is derived from the issue title (lowercase, hyphenated). Capture the issue's intent, any stated interface/behavior, and examples in the seed body, following the seed shape in the skill's "Seed vs Full Specs" guidance. Because Autoflow runs autonomously, set the seed's frontmatter `draft: false` (as Branch 2 does) so Branch 3 can consume it without a manual release gate. Stamp the shared provenance block: `generated-by: "Autoflow"`, `source: <issue-url>`, `source-target: github-issue`, `generated-at: <ISO-8601 timestamp>`, and `consumed: false`. Apply the user refining notes.
   - **With `--gate-input`:** run `Gate Input` (prompt: `sc.gate.input.md`) on the issue body to sanitize noisy or agent-authored content — stripping code snippets, foreign/hallucinated architecture, and internal task breakdowns — producing the sanitized seed(s) in `docs/seed/` plus the feedback report in `docs/seed-feedback/`. Gate Input writes seeds as `draft: true`; override each derived seed to `draft: false` (as Branch 2 does) so Branch 3 can consume it. Set `source: <issue-url>` and `source-target: github-issue` on the generated seed(s).
3. **Commit** the new seed (and, with `--gate-input`, the feedback report) with a message such as `seed: derive seed from issue #<N> for <name>`.
4. Proceed to **Branch 3**.

If a seed, questionnaire, or full spec for this issue's feature already exists in `docs/` (e.g. a prior Autoflow run already derived it), do **not** re-create it — fall through to normal `docs/`-based state detection and resume at the earliest unfinished downstream artifact instead.

## Branch 1 — Brainstorm inputs ready, not yet applied

If a brainstorm report in `docs/brainstorms/` has its `USER_INPUT` placeholders filled but has not been carried through Phases 3–4:

1. **Resume the brainstorm** from Phase 3 (prompt: `sc.brainstorm.md`). Parse the user's answers inside the `USER_INPUT` tags, formulate the architectural approaches, apply the comparative analysis, and deliver the final recommendation and next steps — completing Phases 3 and 4 of the report. Apply the user refining notes. Do not pause for clarification; the user's placeholder answers are the input you act on.
2. Write the completed `BRAINSTORM_REPORT`-style file back to `docs/brainstorms/`, advancing its frontmatter `status` to `recommended`.
3. **Commit** the finalized brainstorm with a message such as `brainstorm: finalize analysis for <name>`.
4. Proceed to **Branch 2**.

## Branch 2 — Brainstorm done, no seed

If a brainstorm report has a populated `## Recommendation` section but no corresponding seed exists in `docs/seed/`:

1. Run `Seed` (prompt: `sc.brainstorm.to.seed.md`) to transform the brainstorm results into a seed specification, **following the brainstorm's recommended approach** unless the user refining notes specify otherwise. Capture all decisions without the implementation detail of a full spec.
2. Write the seed to `docs/seed/`. Because Autoflow runs autonomously, set the derived seed's frontmatter `draft: false` (overriding the `Seed` prompt's default of `draft: true`) so Branch 3 can consume it without a manual release gate.
3. **Commit** the new seed with a message such as `seed: derive seed from brainstorm for <name>`.
4. Proceed to **Branch 3**.

## Branch 3 — Seed(s) only

If only seed(s) are available (and no questionnaire or full spec yet):

1. Perform **non-interactive planning** to expand the seed(s) into a Full Spec, following the skill's "Seed vs Full Specs" guidance and the `Plan Spec` prompt (`sc.plan.spec.md`). Produce a concise, critique-ready blueprint: problem statement, requirements and acceptance criteria, background, proposed solution, Testing & Risk Strategy, and ordered task outlines. Apply the user refining notes. **Run Plan Spec in non-interactive mode:** write the questionnaire to `docs/plans/<spec-name>.questions.md` for traceability, then — instead of stopping at the questionnaire gate — immediately apply each question's recorded **Recommended default**, mark it `default-applied`, and record those decisions in the spec's Resolved Decisions section. Do not pause for clarification; make reasonable, well-documented assumptions.
2. Save the Full Spec to `docs/specs/` (equivalent to `Save Spec`).
3. **Commit** the new full spec with a message such as `spec: plan full spec for <spec-name>`.
4. Proceed to **Branch 5**.

## Branch 4 — Questionnaire answered, no full spec

If a plan questionnaire in `docs/plans/<spec-name>.questions.md` has frontmatter `status: awaiting-answers` and the user has supplied at least one answer (at least one `USER_INPUT` block no longer holds the default placeholder), but no full spec exists yet:

1. **Resume planning from the questionnaire** via the `Plan Spec` prompt (`sc.plan.spec.md`) in **non-interactive mode**, but do **not** regenerate the questionnaire from scratch and do **not** overwrite the user's answers. Parse the answer inside each `USER_INPUT:Qn` block: honor every filled answer, and apply the recorded **Recommended default** only to questions still left at the placeholder (marking those `default-applied`). This mirrors how Branch 1 treats a brainstorm's `inputs-ready` placeholder answers as authoritative input. Apply the user refining notes.
2. Resolve the questionnaire: once every question is either answered or defaulted, set its frontmatter `status: resolved`. Expand the seed(s) and resolved answers into a Full Spec, recording every answered decision and applied default in the spec's Resolved Decisions section (traced to its question id) so the mix of user answers and defaults remains auditable.
3. Save the Full Spec to `docs/specs/` (equivalent to `Save Spec`), consuming the seed(s) as `Plan Spec` specifies.
4. **Commit** the new full spec with a message such as `spec: plan full spec for <spec-name>`.
5. Proceed to **Branch 5**.

## Branch 5 — Full spec available

If a full spec was interactively planned or already exists:

1. Ensure the spec is saved to `docs/specs/` (call `Save Spec` if it has not been saved).
2. **Specification-level critique (skip if already critiqued).** If the spec has **not** yet been critiqued, run `Handoff Critique --auto` (prompt: `sc.handoff.critique.md`) — this hands the spec to an adversarial agent via Orca and automatically applies **only** 🎯 Must-Address findings to the spec once ready, preserving 💡 Recommendations and 🤔 Questions in the report for later human review (and for `Finalize` to capture) — then **commit** the critiqued spec with a message such as `spec: apply critique for <spec-name>`.

If the state is **Full spec, already critiqued** (the user already ran a critique handoff without `--auto` and reviewed it, or a current critique already exists in `docs/critiques/`), **skip** this step and proceed directly to Split Tasks. If the user already applied and committed the critique, there is nothing to commit here.
3. Run the **mandatory** `Split Tasks` command (prompt: `sc.split.tasks.md`) to generate the authoritative execution-detail set in `docs/tasks/<spec-name>/`, including `summary.md`. Do not proceed to implementation from the Full Spec alone.
4. **Commit** the task breakdown with a message such as `tasks: split <spec-name> into tasks`.
5. **Required Autoflow task-level critique.** Run `Handoff Critique --task-level --auto` (prompt: `sc.handoff.critique.md`) against both `docs/specs/<spec-name>.md` and `docs/tasks/<spec-name>/`. This second pass is required for Autoflow only, never for the manual flow. Apply only 🎯 Must-Address findings automatically, targeting task files by default and changing the spec only when the finding changes its intent, architecture, acceptance criteria, or Testing & Risk Strategy. Preserve 💡 Recommendations and 🤔 Questions in the critique report for human review.
6. **Commit** the task-level critique report and any applied planning-artifact fixes with a message such as `tasks: apply task-level critique for <spec-name>`. If a current task-level critique has already been auto-applied to the latest task set, do not rerun it; commit only if the existing report or artifacts remain uncommitted.
7. Run `Implement` (prompt: `sc.implement.tasks.md`) to execute the critiqued task files via subagents, using the `summary.md` for parallelization guidance.
8. **Commit** the implementation with a message such as `feat: implement <spec-name>`.
9. Run `Handoff Code Review --auto` (prompt: `sc.handoff.code.review.md`). This hands the implemented changes to an adversarial agent via Orca and applies the actionable (`CRITICAL`/`HIGH`) findings automatically once the review is ready.
10. **Commit** the review fixes with a message such as `fix: apply code review for <spec-name>`. (Skip this commit if the review produced no changes.)
11. Run `Finalize` (prompt: `sc.finalize.md`) to update `README.md`/docs, capture deferred items to `docs/feedback/`, and post product-manager guidance to the PR if available.
12. **Commit** the finalization with a message such as `docs: finalize <spec-name>`. (Skip this commit if there was nothing to update.)

## Operating Constraints

- Run autonomously end-to-end. Do not pause between stages for confirmation.
- Follow the project's git safety norms: stage specific files, do not push, do not force, and do not touch unrelated changes. Create new commits (never amend) at each commit point.
- If a stage fails, stop at that stage, report what failed and what was committed so far, and leave the working tree in a recoverable state. Do not skip ahead past a failed stage.
- Autoflow runs the full cycle through `Finalize`. Once complete, report every commit made and the final state so the user can review and push.
