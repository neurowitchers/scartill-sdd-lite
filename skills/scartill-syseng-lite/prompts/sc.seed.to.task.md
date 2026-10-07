# Seed to Task

## Goal

Convert an **informal seed** — rough, free-form user intent — into one or more structured task documents conforming to a **task template**. The seed is unstructured thought; the task is a traceable, template-shaped unit of work. This is a lightweight conversion: the agent drafts and writes the task(s) directly, and human adjudication happens afterward through the template's flags and explicit open-question markers rather than a pre-write gate.

## Parameters

- **Seed** (required) — the informal input. May be supplied as:
  - inline free-form text pasted by the user, or
  - a path to a seed file or directory, or
  - both.
- **Task template** (optional) — a template describing the task document shape. If omitted, use the skill default `templates/task-template.md`.

If no seed is supplied (neither inline text nor a path), ask the user before proceeding.

## Operating Constraints

- **DIRECT WRITE**: Draft the task document(s) and write them immediately. Do not stop for a pre-write confirmation gate. Human adjudication occurs after the fact via the template's `[ ]` flags, `Status`, and open-question markers.
- **RETAIN UNCERTAINTY**: Where the seed does not cover a template field, do not invent content. Leave the placeholder with an explicit open-question marker (see Uncovered Fields). Never settle an uncertainty by assumption.
- **TRACEABILITY**: Every meaningful fragment of the seed must land in a definite place in the task document(s). Nothing from the seed is silently dropped. Where a fragment seeds a field, prefer keeping the user's own wording.
- **RELATIVE PATHS**: Do not use absolute paths when referencing files in generated artifacts. Use relative paths only.
- **SINGLE OUTPUT BY DEFAULT**: Produce exactly one task document from one seed. Split into multiple tasks only when the seed plainly contains several distinct, separable units of work; when you split, confirm the split with the user before writing.

## Uncovered Fields

When a template field has no corresponding seed material, fill it with an explicit marker rather than a guess:

```text
> TODO (open question): <what is missing / what the author must decide>
```

This preserves uncertainty as a visible discussion point. Agent-inferred content is not written into uncovered fields.

## Steps

1. Gather the seed. If supplied as a path, read the file or directory; if inline, use the pasted text; if both, treat them together.
2. Read the task template. If a template parameter was supplied, read it; otherwise use the skill default `templates/task-template.md`.
3. Assess scope. By default, produce one task. If the seed plainly contains several distinct, separable units of work, propose a split to the user and wait for confirmation before writing. Otherwise, proceed directly.
4. Map seed fragments to template fields, preserving the user's wording where it fits. Assign the task a stable task ID and set its `Covers` field to a human-readable scope summary. Populate the provenance frontmatter: `generated-by: "Seed to Task"`, `source` as the seed's relative path (or `"inline"` for pasted text, or both when both are given), and `generated-at` as the current ISO-8601 timestamp.
5. For every template field with no seed material, write the open-question marker from Uncovered Fields. Leave `[ ]` flags and `Status` at their default (initial) values.
6. Ensure `tasks/` exists under the working root (ask the user for the working root if it is not evident from the seed location). Write the task document(s) there. Never overwrite an existing file — if a name would collide, create a new one with a sequence suffix.
7. Report what was written: the task file path(s), the task ID(s), and a list of the uncovered fields left as open questions so the user knows what still needs their input.

## Output

- Task document(s) → `tasks/`.

Give each generated file a descriptive name derived from the task title, with a sequence suffix on collision; never overwrite an existing file.

## Follow-up Sessions

In a follow-up session, resume from the current state. Update only the task document(s) this command owns; do not alter other files without an explicit request.
