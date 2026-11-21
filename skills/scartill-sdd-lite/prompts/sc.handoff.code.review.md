# Hand Off Code Review to Adversarial Agent

Hand the code review to an adversarial agent (Antigravity by default) and ask it to apply the `Code Review` command of the Scartill SDD Lite skill to the current PR / branch.

**Dispatch mode depends on `--auto`:**

- **With `--auto` (direct subagent):** do **not** use Orca and do not open a terminal. Call the adversarial agent directly as a synchronous subagent through the host's subagent mechanism (for example `invoke_sub_agent`), with a fresh context. If the host cannot run subagents, run the adversarial CLI (`agy`) synchronously in the foreground instead. The call blocks until the review is written, so there is no yielding, polling, or `worker_done` signal. If it fails or produces no review, stop and report the failure.
- **Without `--auto` (Orca):** use the Orca `/orchestration` skill to start a supervised worker running Antigravity (`--agent antigravity`, launched with `--mode auto --dangerously-skip-permissions`). Instruct it to **settle with exactly one `worker_done` signal** (`--outcome succeeded` when the report is written, `--outcome failed` otherwise), with no `ask` and no free-form messages. Then ask the user whether the review is ready.

If the user mentions that this is not the first review pass (e.g. "pass 2"), pass that note along to Antigravity so it produces a new review file rather than reusing a prior one.

When the review is ready, find the corresponding review report in `./docs/codereviews/` (named `pr-<NUMBER>-review.md`) and act on its findings. Be aware of multiple passes: always use the **latest** review file.

Acting on the review means addressing the actionable findings — prioritize `CRITICAL` and `HIGH` items, then `MEDIUM`. For each fix applied, keep the change minimal and aligned with project conventions. Leave `LOW` style nits unless trivially safe.

A direct subagent ends when it returns. An Orca worker (non-`--auto`) releases itself on `worker_done`; after the review has been acted upon, release it via Orca only if it was kept open.

## Note on the Flow

A code review targets implemented changes. Ensure there is a PR or a committed branch to review before handing off. If there is nothing to review yet, stop and say so rather than handing off an empty diff.

## Command Arguments

Supplied arguments: $ARGUMENTS
Allowed arguments: `--auto`

## Automatic Option

If the user supplies the `--auto` option, act on the review as soon as the adversarial subagent returns and the review file is ready — without waiting for an explicit user confirmation. In `--auto` mode, apply fixes for `CRITICAL` and `HIGH` findings automatically; for anything that is ambiguous or would change intended behavior, record it and report it rather than guessing.
