<!--
Canonical import-only guardrail for a document workspace's `inputs/` directory.
When scaffolding a workspace, copy this file to `<workspace>/inputs/AGENTS.md` and
replace every `{workspace}` placeholder with the workspace directory name.
The authority for the overall workspace shape is the skill's "Workspace layout" (see SKILL.md).
-->
# AGENTS.md — `{workspace}/inputs`

**This directory is READ-ONLY. Import only.**

Everything under `{workspace}/inputs/` is a set of **imported reference copies** of documents whose source of truth lives elsewhere.

## Rules for agents

- **Do NOT edit, rewrite, reformat, or delete** any file in this directory. Treat all contents as immutable inputs.
- **Reading and quoting are allowed** — cite these files freely as evidence or source material.
- **Adding files is allowed only as a fresh import** (bringing in a new upstream copy). Record where an import came from — on the imported artifact, prefer the `source` / `source-target` provenance frontmatter from `references/frontmatter.md`. Do not alter a file after importing it; re-import instead.
- **To change the *content* of a source document, do not touch the copy here.** Produce an **amendment instruction** for the document's true owner/author's agent, referencing the relevant `inputs/` copy by path.
- If a task seems to require editing a file here, stop and treat it as a signal that the edit belongs in an amendment instruction instead.

## Why

These copies exist for traceability and for grounding analysis against a stable snapshot. Editing them would (a) desynchronise them from the real source of truth and (b) bypass the author/consortium amendment workflow.
