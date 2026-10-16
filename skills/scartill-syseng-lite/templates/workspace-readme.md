<!--
Canonical workspace README. When scaffolding, copy to `<workspace>/README.md` and
replace `{workspace}` (the directory name) and `{title}` (a human title for the document
under production). Fill the one-line purpose; leave the rest as the standard orientation.
Authority for the directory shape is the skill's "Workspace layout" (SKILL.md).
-->
# {title}

Workspace: `{workspace}/`. One-line purpose: _{what document this workspace produces, and for whom}_.

This directory is a **document workspace** managed with the `scartill-syseng-lite` kit. It holds everything needed to take the document from informal intent through to a consolidated, exportable draft, with full traceability.

## Layout

| Directory       | What lives here                                                            |
|-----------------|---------------------------------------------------------------------------|
| `inputs/`       | Imported reference copies of source documents — **read-only** (see `inputs/AGENTS.md`). |
| `seed/`         | Informal, human-written intent documents.                                 |
| `tasks/`        | Structured task documents.                                                |
| `task-results/` | Per-task output documents.                                                |
| `quest/`        | Preprocessing questions with answer placeholders.                         |
| `critiques/`    | Human-adjudicated critiques.                                              |
| `feedback/`     | Residual deviations consolidated from critiques.                          |
| `analysis/`     | Ad-hoc analysis and decision notes.                                       |
| `results/`      | The consolidated document(s) and the task-to-results mapping.             |

Not every workspace uses every directory; the shape is a superset.

## Working here

- Artifact frontmatter follows the canonical schema (the kit's `references/frontmatter.md`).
- The agent proposes; the user decides — work stops at every human-adjudication gate.
- Do not edit anything under `inputs/`; produce an amendment instruction for the source owner instead.
