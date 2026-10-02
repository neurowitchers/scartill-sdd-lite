# Hand Off Critique to Adversarial Agent

Use the Orca orchestration skill (`orca-cli`) to launch a new terminal running Antigravity
(the command is `agy`) and ask it to apply the `Critique` command of the Scartill SDD Lite
skill to the generated specification.

If the user mentions that this is not the first critique pass (e.g. "pass 2"), pass that note
along to Antigravity so it creates a new critique file rather than reusing a prior one.

After Antigravity is launched, ask the user whether the critique is ready. When the user
confirms, find the corresponding critique in `./docs/critiques/` and apply its suggestions.
Be aware of multiple passes: always use the **latest** critique file.

After the critique has been applied, ask Antigravity to exit and close the hand-off terminal.

## Note on the Flow

Users often forget to call `Save Spec` prior to asking for a critique. If the spec has not
been saved to `docs/specs/` yet, call `Save Spec` **before** handing off.

## Command Arguments

Supplied arguments: $ARGUMENTS
Allowed arguments: `--auto`

## Automatic Option

If the user supplies the `--auto` option, apply the critique as soon as the adversarial
agent stops working and the critique file is ready — without waiting for an explicit user
confirmation.
