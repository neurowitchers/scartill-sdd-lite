# Finalize

Post-implementation wrap-up. Finalize has two responsibilities: update documentation, and
capture deferred work for future iterations.

## 1. Documentation Updates

If the changes affect user interfaces or project configuration, make the relevant updates
to `README.md` and any documentation it references.

## 2. Capture Deferred Items (Feedback)

Regardless of how the work was driven (manual, `--auto`, Autoflow, or any other mode), some
things deliberately do not make it into the implementation. This is especially common in
**manual** sessions, where the human consciously sets items aside while staying focused on
the core change. Finalize must capture those so they are not lost.

Create a new document under `docs/feedback/` (create the directory if needed). **Never
overwrite** an existing feedback file — name each one by date and spec, e.g.
`docs/feedback/<YYYY-MM-DD>-<spec-name>.md`, so feedback accumulates across iterations.

### What counts as a deferred item

Gather candidates from the whole session, including:

- Critique findings that were acknowledged but **not applied** (open `💡 Recommendation`
  and `🤔 Question` items, or any `🎯 Must-Address` consciously deferred).
- Code review findings that were **not fixed** (typically `MEDIUM`/`LOW`, or anything
  parked for a follow-up).
- Explicit "do this later", "out of scope", "v2", or "nice to have" notes raised during the
  session — by the user or surfaced during implementation.
- `TODO`/`FIXME` markers introduced by the implementation.
- Seed-spec follow-up tasks (e.g. amend README, add examples) that were not completed.
- Known limitations, shortcuts, or technical debt knowingly accepted.

### Document structure

```markdown
# Deferred Items — <spec-name>

**Date**: <YYYY-MM-DD>
**Source spec**: docs/specs/<spec-name>.md
**Mode**: <manual | auto | autoflow>

## Summary
<1–2 sentences on what shipped and what was intentionally left out>

## Deferred Items

| ID | Origin | Priority | Item | Rationale / Next Step |
|----|--------|----------|------|-----------------------|
| D1 | Critique 💡 | Medium | <what was deferred> | <why deferred, suggested follow-up> |
| D2 | Code Review MEDIUM | Low | <what was deferred> | <why deferred, suggested follow-up> |
| D3 | User note | High | <what was deferred> | <why deferred, suggested follow-up> |

## Suggested Follow-up
<Which items are candidates for the next iteration — a future Brainstorm/Seed/Spec>
```

Use relative paths only — do not use absolute paths in the feedback document.

If there are genuinely **no** deferred items, do not create an empty file; instead state
that nothing was deferred this session.
