Implement tasks using subagents. Relevant `summary.md`, if exists, can provide guidance for parallelization.

As each task is completed, advance its frontmatter `status` to `done` and mirror the body `- [x]` completion checkbox (see `references/frontmatter.md`, artifact type: task). Once all tasks in the set are done, advance the source spec's frontmatter `status` to `implemented` (artifact type: spec).
