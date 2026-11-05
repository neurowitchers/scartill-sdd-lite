---
name: scartill-session
description: "Manages git-friendly multi-session tracking across tasks. Usage: /scartill-session [new <name> | save [<id>] | resume <id> | close [<id>] | remove [<id>]]"
---

You are managing git-friendly developer session handoffs inside `.context/`.

There is **no pointer file** tracking a single "active" session. Multiple sessions may be `Active` at once (e.g. several agents or terminals working in the same worktree), so a shared "current session" marker would be misleading. Instead, "current session" means **current to this conversation**: the session this CLI conversation established by running `new <name>` or `resume <id>` earlier in the same session. That knowledge lives in the agent's working context, not on disk. If a conversation never established one, there is no current session until the user runs `new`, `resume`, or `save`.

Commands that act on an existing session take an explicit `<id>`. When `<id>` is omitted, `save`, `close`, and `remove` fall back to the conversation's current session **if one is known**; if none is known they stop and ask (listing sessions from `.context/SESSIONS.md`). If a user wants to pin a working session id for later, they can record it in `AGENTS.md` or steering (`.kiro/steering/`) — that is the user's choice, not something this skill writes.

All session artifacts live under `.context/` and are plain, diffable Markdown so they travel with the repository and review cleanly in git. Use the user's local timezone for timestamps: a full `YYYY-MM-DD HH:MM` in session frontmatter and Log entries. Filenames never carry a date — the date lives only in frontmatter. Treat `<id>` as the session filename (with or without the `.context/sessions/` prefix and `.md` suffix).

Handle the user's intent based on the arguments:

## `new <name>`

1. Generate the filename `.context/sessions/<name>.md` (slugify `<name>`: lowercase, spaces and non-alphanumerics to hyphens). The filename carries no date — the start date lives only in the frontmatter `Started` field. If the file already exists, append a short numeric suffix rather than overwriting.
2. Add an `Active` entry row to `.context/SESSIONS.md` (create the file with its table header if absent).
3. Initialise the new session file with:
   ```markdown
   # Session: <name>
   **Started**: YYYY-MM-DD HH:MM
   **Status**: Active

   ## Objective
   - [Define primary target]

   ## Environment
   - Shell: [e.g. PowerShell on Windows]
   - Build: `[exact build command]`
   - Test: `[exact test command]`
   - Running processes: [none, or e.g. dev server in background]

   ## Key Changes
   - None yet.

   ## Blockers / Open Questions
   - None.

   ## Immediate Next Step
   - [Next concrete action — write the literal command to run in backticks, e.g. `npm run dev`]

   ## Log
   - YYYY-MM-DD HH:MM: Session initialised.
   ```
4. Report the created path. This is now the conversation's current session.

Creating a new session never disturbs other `Active` sessions — several may coexist. On a filename collision, sequence-suffix rather than overwrite.

The `## Environment` section is **session-scoped** shell context — the shell in use, the exact build/test commands this session relies on, and any background processes it left running — so a resuming agent runs known-good commands instead of re-deriving them. It is *not* a substitute for project-wide guidance: stable, repo-level conventions (default shell, canonical build/test commands) belong in steering (`.kiro/steering/`) or `AGENTS.md`, which load regardless of session. Record only what is specific to or active in this session; omit the section if nothing session-specific applies.

## `save [<id>]`

`save` persists the current state. It works whether or not a session was explicitly `new`-ed — its job is to never lose accumulated work. **`save` runs only when the user explicitly invokes it.** The bootstrap-from-ongoing-context behaviour below is what `save` does *when no session is known*; it is NOT a licence to invoke `save` on your own because work seems worth checkpointing. Never self-trigger `save` (or its bootstrap) mid-task.

1. Resolve the target session:
   - If `<id>` is given, use that session file (if it does not exist, list available sessions from `.context/SESSIONS.md` and stop).
   - Otherwise use the conversation's current session (the one this conversation `new`-ed or `resume`-d).
   - If neither is available, **bootstrap** one — see step 2.
2. **Bootstrap (only when no session is known and no `<id>` was given).** Create a session from the accumulated context of the ongoing CLI session — do not refuse. Derive a concise `<name>` from what the conversation has actually been doing, then create the session exactly as `new` would (slugified `.context/sessions/<name>.md` with no date in the filename and a `SESSIONS.md` `Active` row). Populate `## Objective`, `## Environment`, `## Key Changes`, `## Blockers / Open Questions`, and `## Immediate Next Step` from the real work done so far rather than placeholders — recording any exact commands actually run or needed next as literal backticked commands — set `**Started**` to the current timestamp, and make the first `## Log` entry record that the session was captured from an in-progress CLI session (`- YYYY-MM-DD HH:MM: Session captured from ongoing CLI session.`). Tell the user which session was created; it becomes the conversation's current session. Bootstrapping is done — do not also run the reconcile below (the populated content already is the reconciliation).
3. Reconcile the resolved session file with the current state of the work: refresh `## Key Changes` (what was done since the last save), `## Environment` (if the shell, build/test commands, or running processes changed), `## Blockers / Open Questions`, and `## Immediate Next Step`, and append a timestamped `## Log` entry (`- YYYY-MM-DD HH:MM: <summary of progress>`).
4. Leave `**Status**: Active` unchanged. Do not close the session.
5. Report what was updated (or created).

## `resume <id>`

`resume` requires an explicit `<id>` — there is no pointer to fall back on.

1. Resolve the target session file from `<id>` under `.context/sessions/`. If `<id>` is missing or the file does not exist, list available sessions from `.context/SESSIONS.md` and stop.
2. If its `SESSIONS.md` row is not already `Active`, set it back to `Active`.
3. Read the session file and brief the user: Objective, latest Log entries, outstanding Blockers / Open Questions, and the Immediate Next Step — so work can continue exactly where it left off.
4. Append a `## Log` entry noting the resume (`- YYYY-MM-DD HH:MM: Session resumed.`). This is now the conversation's current session.

## `close [<id>]`

1. Resolve the target session: use `<id>` if given, otherwise the conversation's current session. If neither is available, list sessions from `.context/SESSIONS.md` and stop.
2. Perform a final reconciliation (as in `save`), then set `**Status**: Closed` in the session file and append a closing `## Log` entry (`- YYYY-MM-DD HH:MM: Session closed.`).
3. Update the matching row in `.context/SESSIONS.md` to `Closed`.
4. Report the closed session. If it was the conversation's current session, note the conversation no longer has one.

## `remove [<id>]`

Removes (deletes) a session file from the working tree.

1. Resolve the target session: use `<id>` if given, otherwise the conversation's current session. If neither is available, list sessions from `.context/SESSIONS.md` and stop (nothing to remove).
2. Deleting removes the session from the working tree; ask the user to confirm first (name the file), but note the file remains recoverable from git history since `.context/` is committed. Do not delete without confirmation.
3. On confirmation, delete the session file and remove its row from `.context/SESSIONS.md`.
4. Report the removed session. If it was the conversation's current session, note the conversation no longer has one.

> Use `remove` to discard a session entirely (e.g. one created by mistake). To retire a completed session while keeping its record, use `close` instead — `close` preserves the file and its `SESSIONS.md` row.

# Conventions

- **Git-friendly**: everything under `.context/` is plain Markdown with stable structure, so sessions diff cleanly and are safe to commit. Do not commit on the user's behalf unless asked.
- **`.context/SESSIONS.md`** is the index of all sessions and the single on-disk source of truth for what exists and each session's status. Use a Markdown table with the columns `| Session | Started | Status |`, one row per session, the `Session` cell linking to the session file. Create it with that header on first use.
- **No active-session pointer.** There is no `.current_session` file. "Current session" is scoped to the running conversation (whatever it `new`-ed or `resume`-d); it is not shared state on disk, so it can never mislead another agent or human sharing the worktree. Multiple sessions may be `Active` in `SESSIONS.md` at the same time. If a user wants to remember a working session id across conversations, they record it in `AGENTS.md` or steering themselves.
- **Never overwrite** an existing session file; on a name collision, sequence-suffix a new one.
- **Record exact, runnable commands — never paraphrase a command.** In `## Environment`, `## Immediate Next Step`, and `## Log`, write the literal command in backticks (e.g. `` `npm run dev` ``) rather than a prose description of it ("restart the dev server"), so a resuming agent runs known-good commands instead of re-deriving shell syntax. Session shell context is session-scoped; stable, repo-wide shell/build/test guidance belongs in steering (`.kiro/steering/`) or `AGENTS.md`, not in session files.

Upon activation, remember this command and its sub-actions, but do not run anything until an explicit user request. "Explicit user request" means the user invoked `/scartill-session <subcommand>` (or asked for it in plain words) in their most recent message. Ongoing work — however substantial, risky, or worth checkpointing it may seem — is NEVER implicit consent to `save`, `new`, `close`, or to append a `## Log` entry. Never create, modify, or write to any file under `.context/` on your own initiative. The "never lose accumulated work" guarantee of `save` applies only once the user has actually invoked `save`; it does not authorise auto-saving, bootstrapping, or logging beforehand.
