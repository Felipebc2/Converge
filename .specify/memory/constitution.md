<!--
Sync Impact Report
Version change: 1.0.0 → 1.0.1
Rationale: Clarifies the Constitution's authority boundary and aligns the task
template with the already-ratified testing requirements.

Modified principles:
- Governance / Authority — scoped supersession to repository-level project
  practices, templates, and agent instructions, and added an explicit
  carve-out: this Constitution never overrides platform, legal, or security
  policies. Wording clarification only; does not change which of the ten
  Core Principles apply or their intent — PATCH.
- X. Human-Controlled Version Control — dropped "automation" from the list of
  actors barred from repository-publishing actions and added an explicit
  carve-out: CI/CD automation MAY build, test, sign, checksum, and publish
  releases, but only when triggered by an approved semantic version tag that
  the human maintainer created manually. "Releases" was removed from the
  human-exclusive-authority list accordingly, since publishing is now an
  automated consequence of a human-created tag rather than a human-invoked
  action itself; staging, commits, pushes, merges, tags, and pull requests
  remain exclusively human-invoked. Clarifies an existing non-negotiable
  (human-gated publishing) rather than relaxing it — PATCH.

Added sections:
- Core Principles I–X (Versioned Sources of Truth; Modular Clean Architecture;
  Secure Local Runtime; Contracts as Code; Durable and Reconstructable Data;
  Risk-Oriented Test-First Development; Product and UX Integrity; Privacy and
  Observability; Documentation and Delivery Discipline; Human-Controlled
  Version Control)
- Governance (Amendment Procedure, Versioning Policy, Exception Policy,
  Compliance Review)

Removed sections:
- Generic template placeholders ([SECTION_2_NAME], [SECTION_3_NAME]) — their
  content was consolidated into the ten Core Principles and Governance above,
  since the source instructions did not require separate freeform sections.

Templates requiring updates:
- ✅ .specify/templates/plan-template.md — already carries a generic
  "Constitution Check" gate; no textual change required, gate now resolves
  against this constitution's ten principles.
- ✅ .specify/templates/spec-template.md — added a "Constitution Compliance"
  subsection under Requirements to satisfy Governance's compliance-review rule.
- ✅ .specify/templates/tasks-template.md — added a "Constitution Compliance"
  checkpoint in Phase 1 (Setup); removed the "Commit after each task or
  logical group" note in Notes, which conflicted with Principle X
  (Human-Controlled Version Control), replacing it with guidance to report
  suggested Git commands for the human maintainer; replaced all
  "tests are OPTIONAL / if requested" language (Tests intro, User Story test
  headers, Dependencies, Notes) with Principle VI's actual rule — tests
  mandatory per risk, Red-Green-Refactor scoped to domain/use cases/contracts/
  security/bug fixes, other tests included in the same change without
  test-first ordering; and replaced generic Python-style path examples with
  Converge's real layout (crates/, apps/web/, packages/, migrations/, tests/).
- ✅ .specify/templates/checklist-template.md — no change required; checklist
  content is generated per-request and inherits compliance review from the
  spec/plan/tasks artifacts it references.
- ✅ .claude/skills/speckit-implement/SKILL.md and sibling speckit-*/SKILL.md
  files — reviewed; already read-only with respect to Git (only
  `git rev-parse --git-dir` for repo detection) and already load
  `.specify/memory/constitution.md` for governance constraints dynamically.
  No outdated agent-specific references found.

Follow-up TODOs: None. `docs/PRD.md`, `docs/PRODUCT.md`, and `docs/DESIGN.md`
referenced by Principle I now exist and are approved (2026-07-22), closing the
gap noted at 1.0.1 ratification. `MVP.md` (referenced by PRD.md Section 20) and
ADR records remain not yet created; their absence does not block ratification
since Principle I governs their authority once created, not their existence.
-->

# Converge Constitution

## Core Principles

### I. Versioned Sources of Truth

Implementation MUST be governed exclusively by Git-versioned documents. This
Constitution holds the highest authority over project practices. `PRD.md` and
`PRODUCT.md` govern product requirements. `DESIGN.md` governs UX, visual
language, and accessibility. Approved Architecture Decision Records (ADRs)
govern architectural decisions. Slice specifications govern their own bounded
implementation scope and MUST NOT be treated as authoritative outside it.
Notion MAY record decisions and supporting context, but it is not normative
when it conflicts with any versioned repository document. Conflicts between
normative sources MUST block implementation until resolved.

**Rationale**: A single, git-versioned hierarchy of authority prevents
implementation from drifting onto undocumented or unreviewable decisions made
in ephemeral or external tools.

### II. Modular Clean Architecture

Converge MUST remain a modular monolith built on Clean Architecture.
Dependencies MUST point inward toward the domain and application layers.
Domain code MUST remain independent from frameworks, transports, persistence,
filesystem access, and external AI providers. Provider-specific behavior MUST
be isolated behind explicit ports and adapters. Cross-module coupling MUST be
deliberate, documented, and testable — accidental or implicit coupling is a
violation regardless of how the module boundary is drawn.

**Rationale**: Inward-pointing dependencies and adapter isolation keep the
domain testable and swappable as frameworks, transports, and AI providers
change around it.

### III. Secure Local Runtime

Local execution MUST follow least privilege and deny-by-default rules.
Generic unrestricted shell execution is prohibited. Project paths MUST be
canonicalized. External symlinks MUST be blocked until explicitly authorized.
New projects MUST be untrusted by default, and scripts MUST be disabled by
default per project. Remote endpoints MUST be reached only through an
explicit allowlist, and remote content MUST NOT execute inside the WebView.
Protections against secret leakage, data corruption, and trusted-directory
escape are non-waivable and MUST NOT be relaxed by any exception.

**Rationale**: Converge runs locally with access to a user's filesystem and
credentials; deny-by-default and non-waivable protections bound the blast
radius of any single compromised project, script, or remote endpoint.

### IV. Contracts as Code

Rust contracts MUST be the authoritative source for generated TypeScript
contracts. Contract generation MUST be deterministic, and generated artifacts
MUST NOT drift from their source. HTTP, WebSocket, persistence, and event
boundaries MUST be explicit and testable. Breaking contract changes require
impact analysis and appropriate versioning before they are merged.

**Rationale**: A single authoritative source with deterministic generation
eliminates an entire class of client/server mismatch bugs and makes boundary
changes reviewable as a diff of intent, not of generated noise.

### V. Durable and Reconstructable Data

Normalized events MUST be immutable. Derived projections and aggregates MUST
be reconstructable from those events. SQLite schema changes MUST use
versioned, forward-only SQLx migrations, and migrations MUST be tested from an
empty database and through every supported upgrade path. File writes affecting
user data MUST be atomic and conflict-aware. Credentials MUST use the native
system keyring and MUST NOT be stored in SQLite.

**Rationale**: Immutable events and reconstructable projections make data loss
and corruption recoverable by replay rather than by backup restoration, and
keyring-only credential storage keeps secrets out of a file that gets synced,
backed up, or shared.

### VI. Risk-Oriented Test-First Development

Red-Green-Refactor is mandatory for domain logic, application use cases,
contracts, security boundaries, and bug fixes. UI, configuration,
infrastructure, and adapter code MUST include appropriate tests in the same
change, without requiring literal test-first ordering in every case. Critical
behavior MUST cover success, failure, boundary, recovery, and security
scenarios where applicable. No arbitrary global coverage percentage is
required, but coverage of affected critical behavior MUST NOT regress. Each
slice MAY establish additional measurable thresholds. White-box, black-box,
contract, integration, and E2E tests MUST be used according to risk.

**Rationale**: Mandating test-first only where the cost of a defect is highest
(domain, security, contracts, bugs) keeps rigor proportional to risk instead
of imposing uniform ceremony on low-risk code.

### VII. Product and UX Integrity

Implementations MUST preserve the Graphite Signal direction. Light and dark
themes MUST both be supported. User-facing functionality MUST comply with
WCAG 2.2 AA. Keyboard operation, focus visibility, contrast, reduced motion,
and loading, empty, error, offline, and disabled states MUST be considered for
every user-facing change. UI/UX changes MUST be validated against `DESIGN.md`
using Impeccable. Visual polish MUST NOT compromise usability, accessibility,
security, or performance.

**Rationale**: Naming the required state matrix (loading/empty/error/offline/
disabled) and the accessibility bar up front prevents these from being
discovered as gaps during review instead of being designed in from the start.

### VIII. Privacy and Observability

Telemetry MUST be disabled by default. Diagnostics and crash reporting MUST
require explicit opt-in. Secrets and sensitive payloads MUST be redacted in
logs and diagnostics. Original event payload retention MUST follow the
approved configurable retention policy. Logs MUST be useful for local
diagnosis without exposing credentials or unnecessary personal data.

**Rationale**: Opt-in-only telemetry and mandatory redaction keep a locally
running tool from becoming a passive data-collection surface, while still
leaving enough signal for the user to diagnose their own issues.

### IX. Documentation and Delivery Discipline

Documentation MUST be updated in the same change as the behavior or
architectural decision it describes. ADRs MUST record durable architectural
decisions and their consequences. Specifications MUST remain scoped to their
slice. Changelog entries MUST describe relevant user-facing and architectural
changes. CI MUST validate formatting, linting, tests, builds, contracts,
migrations, integration, E2E, security, and supported platforms according to
the active slice. Releases MUST be reproducible, signed where required,
checksum-protected, and triggered only from approved semantic version tags.

**Rationale**: Documentation and CI gates that travel with the change, rather
than trailing it, are what keep Principle I's sources of truth actually true.

### X. Human-Controlled Version Control

AI agents, agent integrations, and Spec Kit workflows MUST NOT execute
`git add`, `git commit`, `git push`, `git merge`, `git tag`, `git rebase`,
`git reset`, pull-request creation, or equivalent repository-publishing
actions. They MAY execute read-only Git commands such as status, diff, show,
branch inspection, and log. At the end of every implementation task, the
agent MUST provide: (1) a concise change summary, (2) the files changed,
(3) verification and test results, (4) known risks or pending work, and
(5) exact suggested Git commands for the human maintainer to review and
execute manually. CI/CD automation MAY build, test, sign, checksum, and
publish releases only when triggered by an approved semantic version tag
created manually by the human maintainer. Destructive Git operations are
prohibited. The human maintainer retains exclusive authority over staging,
commits, pushes, merges, tags, and pull requests.

**Rationale**: Keeping repository-publishing actions exclusively human-invoked
ensures every change that reaches shared history had a human review it first,
regardless of how autonomous the authoring agent was.

## Governance

**Authority**: This Constitution supersedes all repository-level project practices,
templates, and agent instructions, but never overrides platform, legal,
or security policies. Where a conflict exists between this
Constitution and any other document, the Constitution governs until the
conflicting document is amended.

**Amendment Procedure**: Amendments require a dedicated, reviewed change; an
impact report describing affected principles, templates, and documentation;
explicit human approval; and synchronized updates to every affected Spec Kit
template and dependent document in the same change.

**Versioning Policy**: This Constitution uses semantic versioning
(MAJOR.MINOR.PATCH):
- **MAJOR**: Removal or fundamental redefinition of a principle.
- **MINOR**: Addition of a new principle or material expansion of an existing
  requirement.
- **PATCH**: Wording clarifications that do not change intent.

**Exception Policy**: Exceptions to this Constitution MUST be temporary and
traceable. An exception requires human approval, a written justification, an
explicit scope, a mitigation plan, a linked issue, and an expiration or
removal condition. No exception may waive the non-waivable protections against
secret leakage, data corruption, or trusted-directory escape defined in
Principle III.

**Compliance Review**: Every specification, plan, task list, implementation,
and review MUST include an explicit Constitution compliance check before it is
considered complete.

**Version**: 1.0.1 | **Ratified**: 2026-07-21 | **Last Amended**: 2026-07-21
