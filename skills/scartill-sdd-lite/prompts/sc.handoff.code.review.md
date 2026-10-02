# Hand Off Code Review to Adversarial Agent

Use the Orca orchestration skill (`orca-cli`) to launch a new terminal running Antigravity
(the command is `agy`) and ask it to apply the `Code Review` command of the Scartill SDD
Lite skill to the current PR / branch.

If the user mentions that this is not the first review pass (e.g. "pass 2"), pass that note
along to Antigravity so it produces a new review file rather than reusing a prior one.

After Antigravity is launched, ask the user whether the review is ready. When the user
confirms, find the corresponding review report in `./docs/codereviews/` (named
`pr-<NUMBER>-review.md`) and act on its findings. Be aware of multiple passes: always use
the **latest** review file.

Acting on the review means addressing the actionable findings — prioritize `CRITICAL` and
`HIGH` items, then `MEDIUM`. For each fix applied, keep the change minimal and aligned with
project conventions. Leave `LOW` style nits unless trivially safe.

After the review has been acted upon, ask Antigravity to exit and close the hand-off terminal.

## Note on the Flow

A code review targets implemented changes. Ensure there is a PR or a committed branch to
review before handing off. If there is nothing to review yet, stop and say so rather than
handing off an empty diff.

## Command Arguments

Supplied arguments: $ARGUMENTS
Allowed arguments: `--auto`

## Automatic Option

If the user supplies the `--auto` option, act on the review as soon as the adversarial
agent stops working and the review file is ready — without waiting for an explicit user
confirmation. In `--auto` mode, apply fixes for `CRITICAL` and `HIGH` findings
automatically; for anything that is ambiguous or would change intended behavior, record it
and report it rather than guessing.
