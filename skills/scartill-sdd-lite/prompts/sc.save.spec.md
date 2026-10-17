Save the spec in `docs/specs`, but don't implement it yet. Switch to execution agent if needed.

Ensure the saved spec carries frontmatter per the canonical schema (`references/frontmatter.md`, artifact type: spec): `generated-by`, `source` (the primary seed's relative path), `generated-at` (ISO-8601), `status` (`draft` on first save), and `critiqued-revision:` (blank until critiqued). If the spec already has this frontmatter, preserve its `status` and `critiqued-revision`; do not reset them.
