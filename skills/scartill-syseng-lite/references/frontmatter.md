# Frontmatter Reference

This is the **canonical schema** for YAML frontmatter across all systems-engineering artifacts produced or consumed by this skill (and by the companion `gdrive-export` skill). Every prompt and template in this kit defers to this document; where a prompt names a frontmatter field, the definition here is authoritative. If a prompt and this reference disagree, this reference wins.

## Principles

- **One schema per artifact type.** Each artifact type (seed, task, critique, result) has a fixed set of frontmatter keys. Do not invent per-document keys.
- **Lifecycle state lives in frontmatter; progress narration lives in the body.** Frontmatter carries machine-readable lifecycle and provenance; the body carries human prose, flags, and open questions.
- **Retain uncertainty.** An unknown value is left blank (not guessed). A blank is a visible discussion point, never an assumption.
- **Relative paths.** All path-valued fields use repository-relative paths, never absolute paths. The sole exception is `export-location`, which is an external Google Drive URL.
- **Timestamps are ISO-8601** with timezone offset, e.g. `2026-10-16T12:03:33+03:00`.
- **Revisions are git commit hashes** (short or full) identifying the state of the source tree at the moment the field was written.

## Shared provenance fields

These fields record how an agent-generated artifact came to exist. They appear on **task** artifacts (and on any other artifact an agent writes from an upstream source). They are omitted from hand-written artifacts.

| Key            | Type              | Meaning                                                                                   |
|----------------|-------------------|-------------------------------------------------------------------------------------------|
| `generated-by` | string            | The command that created the artifact, e.g. `"Seed to Task"`, `"Scaffold Document"`, `"Ingest Feedback"`. |
| `source`       | relative path / `inline` | The origin the artifact was derived from: outline file, seed file/dir. `inline` if pasted inline; both the path and `inline` when both were supplied. |
| `generated-at` | ISO-8601 timestamp | When the artifact was generated.                                                          |

## Per-artifact schemas

### Seed

A seed is an informal, human-written intent document. Its only frontmatter is a consumption flag that records whether the seed has been converted into downstream task(s).

```yaml
---
consumed: false        # true once the seed has been converted to task(s); false otherwise
consumed-at:           # ISO-8601 timestamp set when consumed flips to true; blank while false
---
```

- `consumed` defaults to `false` on a fresh seed.
- When `Seed to Task` consumes the seed, it sets `consumed: true` and stamps `consumed-at` with the current ISO-8601 timestamp.
- Note the spelling: **`consumed`** (the historical `comsumed` is a typo and must not be used).

### Task

A task is a template-shaped unit of work. It carries the shared provenance block, two human-adjudication flags, and a lifecycle `status`.

```yaml
---
generated-by: "Seed to Task"              # see Shared provenance fields
source: path/to/origin.md                 # relative path or "inline"
generated-at: 2026-10-16T12:03:33+03:00   # ISO-8601
inputs-ready: false                       # user flips to true once the task's inputs are supplied
completed: false                          # user flips to true once the task's work is accepted as done
status: not-started                       # lifecycle state — see vocabulary below
---
```

**Human-adjudication flags.** `inputs-ready` and `completed` are user-owned gates. They default to `false` and the user flips them to `true`; the agent never sets them on the user's behalf. They live in frontmatter (not the body) so the state is machine-readable and consistent with the other lifecycle fields.

**Status.** `status` is the single "we are here" indicator — one current value, named so the state is unambiguous across sessions and to the team.

Vocabulary (sequential): `not-started` → `in-progress` → `awaiting-user` → `done`; plus `deferred` / `blocked` as needed. For **multi-phase tasks**, expand the middle with phase-named states, e.g. `phase1-drafting` → `phase1-awaiting-selection` → `phase2-drafting` → `phase2-awaiting-decisions` → `folding` → `phase3-verifying` → `done`.

The agent advances `status` when it finishes an agent step and stops at a gate; the value names what the agent is waiting for (e.g. `awaiting-user`). A user reply at a gate is the signal to advance; the user need not edit `status`. The flags record *what the user has done* (inputs supplied, work accepted); `status` records *where the workflow is* — they are complementary, and the flags are authoritative for their respective gates.

The task's identity fields (`Task ID`, `Covers`, `Location`) and all prose live in the **body**.

### Critique

A critique is the structured, human-adjudicated analysis of a document under critique. Its frontmatter records the critique's lifecycle and the exact revision of the target it was written against, so that an applied critique can be traced to the state of the document it amended.

```yaml
---
status: new                               # new | applied
target: ../report/document.md             # relative path to the document under critique
target-revision:                          # git commit hash of the target when the critique was written; blank if uncommitted
---
```

- `status: new` on creation; set to `applied` once the accepted items have been folded into the target during consolidation.
- `target` is the repository-relative path to the document under critique.
- `target-revision` pins the target's git commit hash at critique time. Leave blank if the target is uncommitted; fill it when known.

The per-item acceptance flags, the flag legend, and item sourcing all live in the critique **body** (see `prompts/sc.ingest.feedback.md`).

### Result

A result is a consolidated output document (a report, a draft, an exported deliverable). Its frontmatter tracks where the document has been exported and the state of the source at the last export, so re-exports are idempotent and traceable. These fields are the contract between this kit and the `gdrive-export` skill.

```yaml
---
export-location:                          # Google Drive URL (folder or file) the result was exported to; blank if never exported
last-export-revision:                     # git commit hash of this result at the last successful export; blank if never exported
last-export-timestamp:                    # ISO-8601 timestamp of the last successful export; blank if never exported
---
```

- `export-location` is the **only** absolute/external value permitted in frontmatter (a Google Drive URL).
- The `gdrive-export` skill reads `export-location` to resolve the overwrite target and writes all three fields back after a successful export.
- A document whose current git revision equals `last-export-revision` is already up to date on Drive; a mismatch signals a pending re-export.
- Human-authored descriptive keys such as `title` and `version` may coexist with these fields; they are document-level metadata, not part of the export contract.

## Legacy field migration

| Legacy / stray usage            | Canonical replacement                         |
|---------------------------------|-----------------------------------------------|
| `comsumed`                      | `consumed`                                     |
| unnamed "add a timestamp" (seed)| `consumed-at`                                  |
| ad-hoc `Status:` body line only | `status` frontmatter key (body marker retained for narration) |
| body `Inputs ready: [ ]` flag   | `inputs-ready: false` frontmatter key          |
| body `Completion flag: [ ]`     | `completed: false` frontmatter key             |
