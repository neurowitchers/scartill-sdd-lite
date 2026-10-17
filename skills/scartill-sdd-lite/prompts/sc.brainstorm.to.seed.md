Transform brainstorm resuls into a new seed specification. Capture all decisions, but avoid excessive implementation details of a full spec.

Use the recommended solution unless the user specifies otherwise.

Write the seed to `docs/seed/<feature-name>.md` with frontmatter conforming to the canonical schema (`references/frontmatter.md`, artifact type: seed, generated variant): `generated-by: "Seed"`, `source` set to the source brainstorm report's relative path, `generated-at` set to the current ISO-8601 timestamp, `draft: true`, and `consumed: false`. Tell the user the seed is marked `draft: true` and that they must set `draft: false` to release it for `Plan Spec`.
