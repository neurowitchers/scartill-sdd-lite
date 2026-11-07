# Hand Off Code Review to Adversarial Agent

Use the Orca `/orchestration` skill to start a supervised worker running Antigravity and ask it
to apply the `Code Review` command of the Scartill SDD Lite skill to the current PR / branch.

Launch the worker non-interactively: pass `--agent antigravity` and the launch arguments
`--mode auto --dangerously-skip-permissions` so the worker can run its `orca orchestration`
reporting calls without raising an interactive permission prompt. Without this, the worker's
completion signal stalls on a confirmation prompt and breaks an autonomous (`--auto`) run.

The worker is a supervised Orca dispatch: its prompt carries Task and Dispatch IDs, so it is
obligated to report back. Instruct it in the task spec to do its job, write the report, and
**settle with exactly one `worker_done` signal** (`--outcome succeeded` when the report is
written, `--outcome failed` otherwise) — and to **not** use `ask` or send any free-form
messages back.

If the user mentions that this is not the first review pass (e.g. "pass 2"), pass that note along to Antigravity so it produces a new review file rather than reusing a prior one.

After the worker is launched, wait for completion the Orca way: with `--auto`, **end your turn**
and let Orca wake you on `worker_done` (do not poll or watch); without `--auto`, ask the user
whether the review is ready. When the review is ready, find the corresponding review report in `./docs/codereviews/` (named `pr-<NUMBER>-review.md`) and act on its findings. Be aware of multiple passes: always use
the **latest** review file.

Acting on the review means addressing the actionable findings — prioritize `CRITICAL` and `HIGH` items, then `MEDIUM`. For each fix applied, keep the change minimal and aligned with project conventions. Leave `LOW` style nits unless trivially safe.

A supervised worker settles and releases itself on `worker_done`; it does not need to be told to exit. After the review has been acted upon, release the worker via Orca only if it was kept open.

## Note on the Flow

A code review targets implemented changes. Ensure there is a PR or a committed branch to review before handing off. If there is nothing to review yet, stop and say so rather than handing off an empty diff.

## Command Arguments

Supplied arguments: $ARGUMENTS
Allowed arguments: `--auto`

## Automatic Option

If the user supplies the `--auto` option, act on the review as soon as the adversarial agent stops working and the review file is ready — without waiting for an explicit user confirmation. In `--auto` mode, apply fixes for `CRITICAL` and `HIGH` findings automatically; for anything that is ambiguous or would change intended behavior, record it and report it rather than guessing.
