---
name: scartill-session
description: "Manages git-friendly multi-session tracking across tasks. Usage: /scartill-session [new <name> | save | resume <id> | close | remove]"
---

You are managing git-friendly developer session handoffs inside `.context/`.

Read `.context/.current_session` if it exists to resolve the active session file path. If a command needs an active session and `.current_session` is missing (or points at a non-existent file), say so and ask the user to run `new <name>` or `resume <id>` first — never silently create one.

All session artifacts live under `.context/` and are plain, diffable Markdown so they travel with the repository and review cleanly in git. Use the user's local timezone for timestamps: a full `YYYY-MM-DD HH:MM` in session frontmatter and Log entries. Filenames never carry a date — the date lives only in frontmatter. Treat `<id>` as the session filename (with or without the `.context/sessions/` prefix and `.md` suffix).

Handle the user's intent based on the arguments:

## `new <name>`

1. Generate the filename `.context/sessions/<name>.md` (slugify `<name>`: lowercase, spaces and non-alphanumerics to hyphens). The filename carries no date — the start date lives only in the frontmatter `Started` field. If the file already exists, append a short numeric suffix rather than overwriting.
2. Write the resolved path into `.context/.current_session` (overwriting any previous pointer).
3. Add an `Active` entry row to `.context/SESSIONS.md` (create the file with its table header if absent).
4. Initialise the new session file with:
   ```markdown
   # Session: <name>
   **Started**: YYYY-MM-DD HH:MM
   **Status**: Active

   ## Objective
   - [Define primary target]

   ## Key Changes
   - None yet.

   ## Blockers / Open Questions
   - None.

   ## Immediate Next Step
   - [Next concrete action]

   ## Log
   - YYYY-MM-DD HH:MM: Session initialised.
   ```
5. Report the created path and confirm it is now the active session.

## `save`

1. Resolve the active session from `.context/.current_session`.
2. Reconcile the session file with the current state of the work: refresh `## Key Changes` (what was done since the last save), `## Blockers / Open Questions`, and `## Immediate Next Step`, and append a timestamped `## Log` entry (`- YYYY-MM-DD HH:MM: <summary of progress>`).
3. Leave `**Status**: Active` unchanged. Do not close the session.
4. Report what was updated.

## `resume <id>`

1. Resolve the target session file from `<id>` under `.context/sessions/`. If it does not exist, list available sessions from `.context/SESSIONS.md` and stop.
2. Point `.context/.current_session` at it and, if its `SESSIONS.md` row is not already `Active`, set it back to `Active`.
3. Read the session file and brief the user: Objective, latest Log entries, outstanding Blockers / Open Questions, and the Immediate Next Step — so work can continue exactly where it left off.
4. Append a `## Log` entry noting the resume (`- YYYY-MM-DD HH:MM: Session resumed.`).

## `close`

1. Resolve the active session from `.context/.current_session`.
2. Perform a final reconciliation (as in `save`), then set `**Status**: Closed` in the session file and append a closing `## Log` entry (`- YYYY-MM-DD HH:MM: Session closed.`).
3. Update the matching row in `.context/SESSIONS.md` to `Closed`.
4. Clear `.context/.current_session` (remove the file or blank it) so no session is active.
5. Report the closed session and that there is now no active session.

## `remove`

Removes (deletes) the **active** session — the one in `.context/.current_session`. Takes no arguments.

1. Resolve the active session from `.context/.current_session`. If none is active, say so and stop (nothing to remove).
2. Deleting removes the session from the working tree; ask the user to confirm first (name the file), but note the file remains recoverable from git history since `.context/` is committed. Do not delete without confirmation.
3. On confirmation, delete the session file, remove its row from `.context/SESSIONS.md`, and clear `.context/.current_session` so no session is active.
4. Report the removed session and that there is now no active session.

> Use `remove` to discard a session entirely (e.g. one created by mistake). To retire a completed session while keeping its record, use `close` instead — `close` preserves the file and its `SESSIONS.md` row.

# Conventions

- **Git-friendly**: everything under `.context/` is plain Markdown with stable structure, so sessions diff cleanly and are safe to commit. Do not commit on the user's behalf unless asked.
- **`.context/SESSIONS.md`** is the index of all sessions. Use a Markdown table with the columns `| Session | Started | Status |`, one row per session, the `Session` cell linking to the session file. Create it with that header on first use.
- **`.context/.current_session`** holds exactly one line: the relative path to the active session file. It is the single source of truth for which session is active.
- **Never overwrite** an existing session file; on a name collision, sequence-suffix a new one.
- **One active session at a time.** `new` and `resume` make their target the active session; `close` leaves none active.

Upon activation, remember this command and its sub-actions, but do not run anything until an explicit user request.
