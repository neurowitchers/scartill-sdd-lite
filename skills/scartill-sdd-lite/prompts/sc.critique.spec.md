## Goal

Challenge the specification and implementation plan through two distinct expert lenses BEFORE committing to implementation. The **Product Lens** evaluates whether the right problem is being solved in the right way for users. The **Engineering Lens** evaluates whether the technical approach is sound, scalable, and free of hidden risks. This dual review prevents costly mid-implementation pivots and catches strategic and technical blind spots early.

## Operating Constraints

**CONSTRUCTIVE CHALLENGE**: The goal is to strengthen the spec and plan, not to block progress. Every critique item must include a constructive suggestion for improvement.

1. **Product Lens Review** (CEO/Product Lead Perspective):

   Adopt the mindset of an experienced product leader who cares deeply about user value, market fit, and business impact. Evaluate:

   This is the **pre-split, specification-level** pass. Assess user journeys, acceptance
   criteria, declared edge/failure behaviour, and risks that the Full Spec must contain; do
   not reject the spec merely because individual task files, exact fixtures, or demo commands
   do not exist yet. In manual flow, this is the only required critique pass.

   #### 1a. Problem Validation
   - Is the problem statement clear and well-defined?
   - Is this solving a real user pain point, or is it a solution looking for a problem?
   - What evidence supports the need for this feature? (user research, data, customer requests)
   - Is the scope appropriate — not too broad (trying to do everything) or too narrow (missing the core value)?

   #### 1b. User Value Assessment
   - Does every user story deliver tangible user value?
   - Are the acceptance criteria written from the user's perspective (outcomes, not implementation)?
   - Is the user journey complete — or are there gaps where users would get stuck?
   - What's the simplest version that would deliver 80% of the value? (MVP analysis)
   - Are there unnecessary features that add complexity without proportional value?

   #### 1c. Alternative Approaches
   - Could a simpler solution achieve the same outcome?
   - Are there existing tools, libraries, or services that could replace custom implementation?
   - What would a competitor's approach look like?
   - What would happen if this feature were NOT built? What's the cost of inaction?

   #### 1d. Edge Cases & User Experience
   - Do the acceptance criteria and Testing & Risk Strategy identify user-visible error,
     empty, loading, migration, and recovery states appropriate to the feature?
   - Are the stated user journeys complete, including interactions with existing functionality?
   - Are accessibility, discoverability, onboarding, and backwards-compatibility implications
     explicitly addressed at the specification level?
   - Are material unresolved edge cases recorded as risks, decisions, or questions rather than
     being hidden in a future implementation task?

   #### 1e. Success Measurement
   - Are the success criteria measurable and time-bound?
   - How will you know if this feature is successful after launch?
   - What metrics should be tracked?
   - What would trigger a rollback decision?

2. **Engineering Lens Review** (Staff Engineer Perspective):

   Adopt the mindset of a senior staff engineer who has seen projects fail due to hidden technical risks. Evaluate:

   #### 2a. Architecture Soundness
   - Does the architecture follow established patterns for this type of system?
   - Are boundaries and interfaces well-defined (separation of concerns)?
   - Is the architecture testable at each layer?
   - Are there circular dependencies or tight coupling risks?
   - Does the architecture support future evolution without major refactoring?

   #### 2b. Failure Mode Analysis
   - What are the most likely failure modes? (network failures, data corruption, resource exhaustion)
   - How does the system degrade gracefully under each failure mode?
   - What happens under peak load? Is there a scaling bottleneck?
   - What are the blast radius implications — can a failure in this feature affect other parts of the system?
   - Are retry, timeout, and circuit-breaker strategies defined?

   #### 2c. Security & Privacy Review
   - What is the threat model? What attack vectors does this feature introduce?
   - Are trust boundaries clearly defined (user input, API responses, third-party data)?
   - Is sensitive data handled appropriately (encryption, access control, retention)?
   - Are there compliance implications (GDPR, SOC2, HIPAA)?
   - Is the principle of least privilege followed?

   #### 2d. Performance & Scalability
   - Are there potential bottlenecks in the data flow?
   - What are the expected data volumes? Will the design handle 10x growth?
   - Are caching strategies appropriate and cache invalidation well-defined?
   - Are database queries optimized (indexing, pagination, query complexity)?
   - Are there resource-intensive operations that should be async or batched?

   #### 2e. Testing Strategy
   - Does the Full Spec's **Testing & Risk Strategy** define proportionate unit, integration,
     and end-to-end coverage for the declared acceptance criteria and critical paths?
   - Are high-risk failure modes, realistic test-data needs, and testability concerns identified
     at the architecture/contract level?
   - Is the coverage ambition appropriate for the feature's risk level, including explicit
     verification evidence or success criteria?
   - Do not require exact test file paths, fixture definitions, assertions, or demo commands in
     this pass; `Split Tasks` generates those mandatory execution details after this critique.

   #### 2f. Operational Readiness
   - Is observability planned (logging, metrics, tracing)?
   - Are alerting thresholds defined?
   - Is there a rollback strategy?
   - Are database migrations reversible?
   - Is the deployment strategy clear (blue-green, canary, feature flags)?

   #### 2g. Dependencies & Integration Risks
   - Are third-party dependencies well-understood (stability, licensing, maintenance)?
   - Are integration points with existing systems well-defined?
   - What happens if an external service is unavailable?
   - Are API versioning and backward compatibility considered?

## Task-Level Critique Mode (Autoflow Only)

When the caller supplies `--task-level`, conduct a **second critique pass** after mandatory
`Split Tasks`. This mode is intended for `Autoflow`; do not recommend or require it in the
manual workflow.

Read both the saved Full Spec and `docs/tasks/<spec-name>/`, including `summary.md`. Retain
all Product and Engineering checks above, then additionally assess:

- **Task traceability:** every acceptance criterion and critical Testing & Risk Strategy item
  maps to a task and concrete verification evidence.
- **Implementation readiness:** task file ownership, proposed paths/patterns, interfaces,
  dependencies, and sequencing are internally consistent and grounded in the codebase.
- **Verification readiness:** task-level test fixtures, assertions, manual checks, demo
  commands, expected results, and integration checkpoints cover critical and user-visible
  behaviours.
- **Orchestration risks:** safe parallel groups, shared-file conflicts, handoffs, migration or
  rollout ordering, and final integration work are explicit and viable.

Classify and report findings with the normal severity scheme. In this mode, associate each
finding with the affected spec section and/or task file. For `--auto`, apply **only** 🎯
Must-Address findings automatically; preserve 💡 Recommendations and 🤔 Questions in the
critique report for later human review. Task-file fixes are the default target; amend the Full
Spec only when the finding changes its intent, architecture, acceptance criteria, or Testing
& Risk Strategy.

3. **Cross-Lens Synthesis**:
   Identify items where both lenses converge (these are highest priority):
   - Product simplification that also reduces engineering risk
   - Engineering constraints that affect user experience
   - Scope adjustments that improve both value delivery and technical feasibility

4. **Severity Classification**:
   Classify each finding:

   - 🎯 **Must-Address**: Blocks proceeding to implementation. Critical product gap, security vulnerability, or architecture flaw.
   - 💡 **Recommendation**: Strongly suggested improvement that would significantly improve quality, value, or risk profile. Should be addressed but won't block progress.
   - 🤔 **Question**: Ambiguity or assumption that needs stakeholder input. Cannot be resolved by the development team alone.

5. **Generate Critique Report**:
   Ensure the directory `docs/critiques/` exists (create it if necessary), then create the critique report. If critique report already exists, do not overwrite, create a new file. The report carries frontmatter per the canonical schema (`references/frontmatter.md`, artifact type: critique): `status: new`, `target` set to the critiqued spec's relative path (plus the task directory in `--task-level` mode), and `target-revision` set to the target's current git commit hash (blank if uncommitted). The report must include:

   - **Executive Summary**: Overall assessment and readiness to proceed
   - **Product Lens Findings**: Organized by subcategory
   - **Engineering Lens Findings**: Organized by subcategory
   - **Cross-Lens Insights**: Items where both perspectives converge
   - **Findings Summary Table**: All items with ID, lens, severity, summary, suggestion

   **Findings Table Format**:
   | ID | Lens | Severity | Category | Finding | Suggestion |
   |----|------|----------|----------|---------|------------|
   | P1 | Product | 🎯 | Problem Validation | No evidence of user need | Conduct 5 user interviews or reference support tickets |
   | E1 | Engineering | 💡 | Failure Modes | No retry strategy for API calls | Add exponential backoff with circuit breaker |
   | X1 | Both | 🎯 | Scope × Risk | Feature X adds complexity with unclear value | Defer to v2; reduces both scope and technical risk |

6. **Provide Verdict**:
   Based on findings, provide one of:
   - ⚠️ **PROCEED WITH UPDATES**: Must-address items found but are resolvable.
   - 🛑 **RETHINK**: Fundamental product or architecture concerns. Recommend revisiting the spec.

7. **Offer Remediation**:
   - In the default specification-level pass **without `--auto`** (the manual flow), for each
     🎯 Must-Address item and 💡 Recommendation, provide a specific suggested edit to the spec
     and ask: "Would you like me to apply these changes? (all / select / none)". If approved,
     apply changes to the relevant planning artifacts.
   - In the specification-level pass **with `--auto`** (e.g. Autoflow's first pass), do **not**
     ask for confirmation: apply **only** 🎯 Must-Address findings to the spec. Retain 💡
     Recommendations and 🤔 Questions in the report for human review — do not auto-apply them.
     This mirrors the task-level `--auto` rule so the earlier, less-reviewed pass never applies
     more than the later one.
   - In `--task-level --auto` mode, do **not** ask for confirmation: apply only 🎯
     Must-Address findings, normally to the affected task files; retain 💡 Recommendations and
     🤔 Questions in the report. Update the spec only for a change to its intent, architecture,
     acceptance criteria, or Testing & Risk Strategy.
   - After applying changes, set the critique report's frontmatter `status: applied`. When the applied critique targets a Full Spec, advance that spec's frontmatter `status` to `critiqued` and stamp `critiqued-revision` with the spec's git commit hash (see `references/frontmatter.md`). Then recommend re-running the relevant critique scope to verify.

8. Additional Instructions

Do not use absolute paths for referencing files in the report. Use relative paths only.

After generating the report, update only the critique report in a follow-up session unless the user explicitly requests remediation or the invocation supplies `--auto` (specification-level or `--task-level`), whose Must-Address-only remediation rules above apply.

