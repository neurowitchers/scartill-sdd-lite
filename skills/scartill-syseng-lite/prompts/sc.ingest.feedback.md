# Ingest Feedback

## Goal

Turn an **input feedback document** into a structured, human-adjudicated critique of a **document under critique**, then consolidate the adjudicated result into concrete amendments, residual deviations, and follow-up brainstorming tasks. This keeps external feedback traceable: every item is extracted, assessed, decided by a human, and routed to a definite outcome.

## Parameters

Both are supplied or known at invocation; this prompt hardcodes neither.

- **Document under critique** — the report/document to be amended.
- **Feedback document** — the input feedback to analyze (a file or directory).

If either parameter is missing, ask the user before proceeding.

## Operating Constraints

- **HUMAN ADJUDICATION**: The agent proposes; the user decides. Never mark a critique item accepted or rejected on the user's behalf. Wait for the user's flags.
- **TRACEABILITY**: Every extracted feedback item must reach a definite outcome (amended, deviation, or brainstorming task). Nothing is silently dropped.
- **RELATIVE PATHS**: Do not use absolute paths when referencing files in generated artifacts. Use relative paths only.
- **ISOLATION**: Do not consult other existing critique files. Each ingestion pass is analyzed independently to avoid cross-contamination.

## Flag Legend

Include this legend at the top of the critique file. Flags are set by the user in the `Accept` block of each item.

```text
Legend for human input blocks:
- `[x]` - valid critique, accept;
- `[p]` - partially valid critique (see related comment);
- `[c]` - invalid but may need additional clarification;
- `[a]` - requires separate brainstorming;
- `[r]` - reject completely, no action needed
- `[ ]` - pending: awaiting user decision (default state).
```

`[ ]` means *not yet decided*, never "disregard".

## Critique Item Format

```text
### <item-id> — <short title>
Source: <section / line reference in the document under critique>

<the critique, phrased as a suggestion to the authors>

Agent assessment:
- Identified feedback author, if available.
- Validity: <...>.
- Recommendation: accept | reject | defer.
- Proposed change: <rewording / deletion / addition, if any>.

Accept: [ ]
> <user comment placeholder>
```

## Steps

### Phase I — Analysis

1. Read the feedback document and the document under critique.
2. Transform the feedback into structured critique items:
   - extract granular items;
   - assign each item an ID and its source location in the document under critique;
   - add the agent assessment (validity, recommendation accept/reject/defer, proposed change/additions).
3. For each item, add an `Accept: [ ]` flag and a `>` comment placeholder, following the Critique Item Format.
4. Ensure `critiques/` exists (create if necessary). Write the critique document there. If a critique file already exists, do not overwrite — create a new file with a pass/sequence suffix (e.g. `ingest-feedback-01.md`).
5. Stop and wait for the user to set flags and comments. Do not proceed to Phase II until the user confirms the critique is adjudicated.

### Phase II — Consolidation

Run only after the user confirms. Re-read the adjudicated critique file.

6. Amend the **document under critique** based on accepted feedback (`[x]`, and the valid portion of `[p]`).
7. Ensure `feedback/` exists. Consolidate residual items into a new deviations document there. Residual items are:
   - the not-accepted portion of `[p]`;
   - `[c]` items the agent still considers worth clarifying;
   - valid critique the user rejected;
   - items left `[ ]` at confirmation (carry forward as explicit discussion points; retain the uncertainty rather than dropping it).
8. For each `[a]` item, ensure `inbox/` exists and create a new brainstorming task document there, following the project's task template if one exists (`templates/task-template.md`). When the template is used, populate the provenance frontmatter: `generated-by: "Ingest Feedback"`, `source` as the feedback document's relative path, and `generated-at` as the current ISO-8601 timestamp.

## Output

- Critique document → `critiques/`.
- Deviations document → `feedback/`.
- Brainstorming tasks → `tasks/`.

Give each generated file a descriptive name with a pass/sequence suffix; never overwrite an existing file — create a new one if a name would collide.

## Follow-up Sessions

In a follow-up session, update the critique report only. Do not alter other files without an explicit request.
