# Skill Harmonization Proposal: Systematic Frontmatter Across `scartill-sdd-lite` and `scartill-syseng-lite`

**Date**: 2026-10-17
**Status**: Accepted (decisions resolved — see §10); implementation in progress
**Scope**: Prompt-text and documentation changes only (no executable code)
**Trigger**: Review of whether `Plan Spec` (`sc.plan.spec.md`) is consistent with recent upstream frontmatter changes.

---

## 0. Summary of the catch

The canonical YAML frontmatter schema (`skills/scartill-syseng-lite/references/frontmatter.md`,
expanded by recent commits `70c7412` and `b523e47`) is scoped only to `scartill-syseng-lite`.
`scartill-sdd-lite` does not follow it at all: its artifacts carry **no frontmatter**. `Plan Spec`
is therefore internally consistent with its own skill (it reads the SDD-lite seed's body draft
marker and works correctly), but the repository as a whole now holds **two incompatible artifact
contracts** for the same concepts (seed, task, critique). This proposal harmonizes them.

---

## 1. Current state: the divergence mapped

The canonical schema declares a principle — *"Lifecycle state lives in frontmatter; progress
narration lives in the body"* — that `scartill-sdd-lite` does not follow.

Every lifecycle/provenance signal, side by side:

| Concept | `scartill-syseng-lite` (canonical) | `scartill-sdd-lite` (today) |
|---|---|---|
| Seed draft/consumption | Frontmatter `consumed` / `consumed-at` | Body line `NOTE TO AGENTS: This is a draft, DO NOT use.`; no consumption record |
| Task lifecycle | Frontmatter `status` + `inputs-ready` / `completed` flags | Checkbox `- [ ]` / `- [x]` in task-file body |
| Task provenance | Frontmatter `generated-by` / `source` / `generated-at` | Prose `**Source spec**:` line |
| Critique lifecycle | Frontmatter `status: new → applied` + `target` / `target-revision` | No frontmatter; "do not overwrite, create a new file" |
| Result/export | Frontmatter `export-target` / `export-location` / `last-export-*` | N/A (no export concept) |
| Timestamps | ISO-8601 with offset | Ad-hoc `<YYYY-MM-DD>` (brainstorm, finalize) |
| Revisions | git commit hash fields | Prose `**Git Reference**` (gate-input report only) |

The recent upstream commits (`70c7412`, `b523e47`) hardened the canonical schema but touched
**only** syseng-lite. SDD-lite was never brought along — hence the two divergent contracts.

---

## 2. Harmonization strategy (structural options)

Promote the canonical schema to a **repo-level contract shared by both skills**, then extend it
with the artifact types SDD-lite needs.

**Option A — Shared reference, SDD-lite defers to it.**
Keep the single canonical `references/frontmatter.md`, extend it with SDD-lite artifact types
(`spec`, `questionnaire`, and reuse `seed` / `critique` / `task`), and have
`scartill-sdd-lite/SKILL.md` add: *"All artifact frontmatter conforms to the canonical schema in
`../scartill-syseng-lite/references/frontmatter.md`."* One source of truth; lowest duplication.
- **Risk**: hard cross-skill dependency. The README states the companion is *optional*; installing
  `scartill-sdd-lite` alone would leave the reference missing.

**Option B — Copy the reference into each skill (RECOMMENDED).**
Each skill ships its own `references/frontmatter.md`, kept in sync. Preserves standalone
installability (fixes Option A's risk) at the cost of a duplicate to maintain.

**Option C — Shared schema, divergent minor conventions.**
Unify only timestamps/paths/revisions globally plus the `seed` type (since Plan Spec consumes
seeds); let each skill keep its own body-level progress markers. Minimal change, but does **not**
deliver "systematic frontmatter" — leaves the task/critique split in place.

**Recommendation: Option B** combined with the full type extension in §3. It delivers true
systematization without breaking the optional-companion promise.

---

## 3. Schema extensions needed for SDD-lite

Add these artifact types to the canonical schema (and mirror into SDD-lite's copy under Option B):

### `spec` (the `docs/specs/<spec-name>.md` Full Spec)
```yaml
---
generated-by: "Plan Spec"
source: docs/seed/<name>.md        # the primary seed; relative path
generated-at: 2026-10-17T09:50:00+03:00
status: draft                      # draft → critiqued → split → implemented → finalized
critiqued-revision:                # git hash of the spec when its critique was applied; blank until critiqued
---
```

### `questionnaire` (`docs/plans/<spec-name>.questions.md`)
Formalize the status it already names in prose (`awaiting-answers` | `resolved`):
```yaml
---
generated-by: "Plan Spec"
spec: docs/specs/<spec-name>.md
status: awaiting-answers           # awaiting-answers → resolved
round: 1                           # current questionnaire round
---
```

### `seed` (reuse canonical, add draft signal)
Reuse the canonical `consumed` / `consumed-at`, and fold the draft signal into frontmatter so it
stops living in the body:
```yaml
---
consumed: false
consumed-at:
draft: true                        # replaces the "NOTE TO AGENTS: ... DO NOT use." body line
---
```
Alternative: keep the body marker for human visibility but mirror it to `draft:`, with the
frontmatter key authoritative — exactly as `task-template.md` already mirrors `status` to a body
line.

### `critique` (reuse canonical)
Reuse `status` / `target` / `target-revision`. SDD-lite critiques gain a machine-readable
`new → applied` state instead of the "create a new file" convention.

### `task` (reuse canonical)
Reuse `status` + `inputs-ready` / `completed` + provenance. The SDD-lite checkbox
`- [ ]` / `- [x]` becomes the in-body mirror of frontmatter `status`, matching how
`task-template.md` already mirrors `status` to a body line.

---

## 4. Prompt-level changes required

Grouped by the artifact each prompt owns.

### Seed producers
- `sc.gate.input.md`: emit seed frontmatter (`consumed: false`, `draft: true`) in the Seed
  Specification Template instead of the `NOTE TO AGENTS` body line; the "remind user to remove
  draft markers" step becomes "remind user to set `draft: false`."
- `sc.brainstorm.to.seed.md`: same seed frontmatter on the generated seed.

### Spec producer / consumer
- `sc.plan.spec.md`:
  - Phase 1: read `draft:` from seed frontmatter (not the body marker); stop if `draft: true`.
  - Phase 4: write `spec` frontmatter on the Full Spec; set the seed's `consumed: true` /
    `consumed-at` once the spec is produced (mirroring `Seed to Task`).
  - Questionnaire schema: replace the prose `**Status**:` line with `questionnaire` frontmatter;
    keep an in-body mirror for readability.
  - Non-Interactive (Autoflow) section: set `status: resolved` in frontmatter.
- `sc.save.spec.md`: ensure spec frontmatter is present/stamped on save.

### Task producer
- `sc.split.tasks.md`: add `task` frontmatter to the Required Task File Schema and `status`-style
  frontmatter to the Summary Schema; keep the `- [ ]` / `- [x]` checkbox as the
  authoritative-mirrored body marker (document that frontmatter `status` is authoritative, matching
  the canonical rule).

### Critique producer
- `sc.critique.spec.md`: write `critique` frontmatter (`status: new`, `target`, `target-revision`);
  flip to `applied` after remediation. The current "create a new file, never overwrite" rule can
  remain as a secondary safeguard.

### Downstream consumers
- `sc.autoflow.md`: state detection currently sniffs body text (empty `## Recommendation`, default
  `USER_INPUT` placeholder, presence of files). Harmonization lets it read authoritative frontmatter
  instead — e.g. seed `consumed`, spec `status`, questionnaire `status` — making state detection far
  more robust. Update each Branch's state checks accordingly.
- `sc.finalize.md`: advance spec `status: finalized`; use ISO-8601 timestamps.
- `sc.archive.md` / `sc.brainstorm.md`: adopt ISO-8601 timestamps for consistency with the canonical
  "Timestamps are ISO-8601" principle.

### Brainstorm note
The brainstorm report is a working document, not a tracked artifact type. Leave its body-level
`USER_INPUT` placeholders but standardize its date fields to ISO-8601. Optionally add a `brainstorm`
type (`status: drafting → inputs-ready → recommended`) to let Autoflow's Branch-4 detection read
frontmatter instead of scanning for an empty `## Recommendation`.

---

## 5. Global conventions to unify (both skills)

Lift these from the canonical "Principles" section to apply repo-wide:
- **Timestamps**: ISO-8601 with offset everywhere (fixes `<YYYY-MM-DD>` in
  brainstorm/gate-input/finalize).
- **Relative paths**: already stated in several SDD-lite prompts; make it a single referenced rule.
- **Revisions**: git commit hashes (fixes gate-input's free-form `**Git Reference**`).
- **One schema per artifact type; no per-document keys.**

---

## 6. Documentation changes
- `README.md`: add a "Frontmatter / artifact contract" subsection pointing to the canonical
  reference; note that both skills share it.
- Both `SKILL.md` files: add the deferral line to the canonical schema (syseng-lite already has it;
  SDD-lite needs it added).

---

## 7. Migration / compatibility

**Resolved (decision 5): no migration or fallback guidance in the prompts.** To keep the skills
free of noise, prompts assume frontmatter is present and do **not** carry legacy-fallback reading
logic. Existing `docs/` artifacts created under the old convention are handled **project by
project** by the user, outside the skill definitions.

---

## 8. Sequencing (safe rollout order)

1. Extend the canonical schema with `spec`, `questionnaire`, `brainstorm` types and the global
   conventions.
2. Copy the reference into `scartill-sdd-lite/references/` (Option B).
3. Update seed producers (`gate.input`, `brainstorm.to.seed`) to emit seed frontmatter.
4. Update `plan.spec` (read `draft`, stamp spec + questionnaire + consume seed).
5. Update `split.tasks` and `critique.spec`.
6. Update `autoflow` state detection to read frontmatter.
7. Update `finalize`, timestamps in `brainstorm` / `archive`.
8. Update `README.md` and both `SKILL.md` files.

Each step is prompt-text-only (no executable code), so the whole change is low-risk and reversible.
No migration/fallback logic is added to the prompts (decision 5).

---

## 9. Open decisions for the reviewer

1. **Option A vs B vs C** — which structural approach? (Recommendation: B.)

> Option B

2. **Seed draft signal** — frontmatter-only `draft:` vs. frontmatter + mirrored body marker?

> frontmatter-only

3. **Scope of first pass** — full harmonization, or critical-path only (seed ↔ Plan Spec

> full

   consumption loop) with the rest deferred to `docs/feedback/`?
4. **Brainstorm type** — add a `brainstorm` frontmatter type, or leave the report body-only?

> add

5. **Legacy migration** — lenient fallback readers (recommended) vs. a one-time migration of existing `docs/` artifacts?

> disregard migration guidance to reduce skill's noise - will handle project by project

---

## 10. Resolved Decisions (adjudicated 2026-10-17)

| # | Decision | Resolution |
|---|----------|------------|
| 1 | Structural approach | **Option B** — each skill ships its own `references/frontmatter.md`, kept in sync. |
| 2 | Seed draft signal | **Frontmatter-only `draft:`** — remove the `NOTE TO AGENTS: ... DO NOT use.` body line entirely. |
| 3 | Scope of first pass | **Full harmonization** — all artifact types and all affected prompts in one pass. |
| 4 | Brainstorm type | **Add** a `brainstorm` frontmatter type (`status: drafting → inputs-ready → recommended`). |
| 5 | Legacy migration | **No migration/fallback guidance** in the prompts; handled per project by the user. |

**Status**: Accepted — implementation in progress.
