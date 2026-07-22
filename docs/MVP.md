# Converge — MVP Definition & Delivery Plan

## 1. Metadata and Status

| Field | Value |
| --- | --- |
| Product | Converge |
| Document | MVP Definition & Delivery Plan |
| Version | 0.1 |
| Approved on | July 17, 2026 |
| Status | Stage 3B — approved |
| Product requirements | `docs/PRD.md` |
| Product direction | `docs/PRODUCT.md` |
| Design requirements | `docs/DESIGN.md` |
| Language | English |

## 2. Purpose and Governance

This document defines the approved first usable delivery of Converge. It does
not redefine the complete product scope in `docs/PRD.md` and does not remove
deferred capabilities from the product direction.

- `docs/PRD.md` governs the meaning and acceptance criteria of requirements.
- This document governs MVP inclusion, restrictions, delivery order, and gates.
- Approved Spec Kit artifacts govern the implementation of each delivery slice.
- The Project Constitution governs repository-wide non-negotiable practices.
- Decision D-164 records approval of the Stage 3B MVP selection.

If normative sources conflict, implementation MUST stop until the conflict is
resolved and the affected documents are reconciled.

## 3. First-Delivery Outcome

The MVP is a locally executed, web-based vertical slice for one active project
folder. It enables an individual developer to:

1. Start Converge with a single documented command.
2. Select a local folder and explicitly trust its canonical path.
3. Detect Claude Code and Codex CLI without modifying provider credentials.
4. Import and inspect provider analytics, quotas, history, and estimates.
5. Create a named Agent and start managed Claude and Codex Sessions.
6. Follow read-only terminal streams and Session lifecycle state.
7. Inspect an Agent Loop with source and confidence information.
8. Browse recognized context files and safely edit eligible `.specify/` files.
9. Reopen persisted history, aggregates, loops, and available files offline.

The MVP complements provider CLIs and the system terminal. It does not replace
them and does not expose arbitrary terminal input through the web interface.

## 4. Runtime and Architecture

### 4.1 Runtime

- React, TypeScript, and Vite run in the browser.
- TanStack Query manages state originating from Rust.
- Zustand manages temporary interface and layout state.
- pnpm manages frontend dependencies and scripts.
- A local Rust service binds only to `127.0.0.1` on a random port.
- Every runtime uses an ephemeral authentication token.
- HTTP handles request/response operations.
- WebSocket handles terminal and other high-volume streams.
- Frontend and API SHOULD use the same origin when possible.
- `Host` and `Origin` are validated and CORS remains closed.
- The browser receives only a Project ID and explicitly authorized data.

### 4.2 Clean Architecture

Converge uses Clean Architecture inside a modular monolith:

- **Domain:** entities, value objects, invariants, and policies.
- **Application:** use cases and inbound/outbound ports.
- **Inbound adapters:** HTTP and WebSocket interfaces.
- **Outbound adapters:** Claude, Codex, PTY/process, filesystem, SQLx, quotas,
  and pricing.
- **Interface:** React through a typed common client interface.

Dependencies point inward. Domain and Application MUST NOT depend on React,
HTTP, WebSocket, Tauri, SQLx, filesystem, PTY, or provider implementations.

### 4.3 Persistence and Events

- SQLite with SQLx stores internal state.
- Migrations are versioned and forward-only.
- Normalized events form an immutable ledger.
- Aggregate tables are fully reconstructable.
- Ingestion is idempotent.
- Original provider payload storage remains disabled throughout the MVP.
- Project YAML and Markdown files remain portable authoritative sources where
  defined by the product documents.
- `justfile` is the command facade for Cargo and pnpm workflows.

## 5. Included Functional Scope

### 5.1 Projects and Trust

- Converge starts without reopening a project automatically.
- The user explicitly selects one folder per execution.
- A newly selected project starts restricted and requires explicit trust.
- Trust is bound to canonical project identity.
- The canonical folder path is the default access boundary.
- External symlinks are blocked; authorizing them is deferred.
- Claude, Codex, provider session files, context files, and Spec Kit presence are
  detected without modifying installations or credentials.
- Persisted local information remains inspectable offline.

### 5.2 Analytics

- Native Rust parsers support Claude and Codex with functional parity.
- Codeburn behavior may be adapted with MIT attribution, but Codeburn is not a
  runtime dependency.
- The last 30 days are imported first; older history is backfilled without
  blocking the interface.
- Tokens, averages, history, quotas, renewal, and API-equivalent estimated cost
  are available.
- Provider, model, Session, period, and time dimensions remain available.
- Agent and phase are used when attributable; otherwise data remains `Unknown`
  or `Unassigned` without changing totals.
- Provider-reported, inferred, estimated, unavailable, and `Stale` values remain
  visibly distinct.
- Quotas refresh every five minutes with backoff.
- Filesystem changes are processed immediately when observed and reconciled
  every minute.
- Pricing uses a fixed versioned snapshot for reproducible estimates.
- Quota thresholds use approved defaults; configuration is deferred.
- Aggregates are retained indefinitely and remain available offline.

### 5.3 Agents and Sessions

- An Agent is a named role and does not have a fixed provider or model.
- Provider and model belong to each Session.
- One Agent may own Claude Sessions, Codex Sessions, or both.
- Starting a Session requests Agent, provider, optional model, and a task prompt.
- The task prompt is sent once at Session start.
- Sessions started by Converge run in managed PTYs.
- Claude and Codex Sessions may run simultaneously.
- Terminal Views remain read-only and allow text selection and copying.
- A Session that needs further interaction enters `Needs Attention`.
- `Stop` attempts graceful termination before offering a confirmed `Force Kill`.
- External Sessions are detected and remain read-only.
- Unrecognized external Sessions appear under `Unassigned Sessions`.
- Closing Converge warns about and terminates only managed local processes.

### 5.4 Agent Loop

- The MVP Agent Loop is read-only.
- Standard phases are `Understanding`, `Planning`, `Execution`, `Validation`,
  and `Completed`.
- `.converge/workflow.yml` is the explicit source when present.
- Without an explicit source, phases are inferred from normalized events.
- The active source and confidence are visible.
- Phases expand into turns, commands, and tool calls in temporal order.
- Only normalized metadata is persisted, including name, normalized action or
  command, timestamp, duration, and status.
- Complete arguments, raw outputs, prompts, and secrets are not persisted.

### 5.5 Context Files

- The recognized tree includes `CLAUDE.md`, `AGENTS.md`, `.claude/`,
  `.converge/`, and `.specify/` within the trusted boundary.
- The viewer provides rendered Markdown, raw text, path, size, and modification
  time when applicable.
- `CLAUDE.md`, `AGENTS.md`, `.claude/`, and `.converge/` are read-only in the MVP.
- Existing Markdown, YAML, and text files inside `.specify/` may be edited.
- Writes are atomic and detect external conflicts.
- A confirmed overwrite creates a rotating backup outside the repository.
- Filesystem watching and periodic reconciliation keep the tree current.
- Exposing existing `.specify/` files does not enable the Spec Kit product domain.

### 5.6 Security and Privacy

- The frontend has no generic shell or filesystem access.
- Rust exposes narrow typed operations with validated arguments and scopes.
- Credentials are read-only, kept in memory, redacted, and never persisted.
- The local service enforces loopback binding, random port, ephemeral token,
  `Host`/`Origin` validation, and closed CORS.
- Remote content is not executed in the product interface.
- No telemetry or remote crash reporting is enabled.
- Local diagnostics are redacted and never transmitted automatically.
- External paths and destructive lifecycle actions are blocked or require the
  approved explicit confirmation.

## 6. Requirement Classification

The 65 stable requirement IDs remain defined by `docs/PRD.md`.

| Classification | Meaning |
| --- | --- |
| Included | Implemented and accepted in the MVP. |
| Restricted | Included only with the explicit restriction stated here. |
| Deferred | Remains in complete product scope but is not implemented in the MVP. |
| Guardrail | Governs the MVP but has no standalone user-facing capability yet. |

### 6.1 Projects and Analytics

| Requirement | Classification | MVP restriction or note |
| --- | --- | --- |
| PRJ-001 | Included | One explicitly selected folder per execution. |
| PRJ-002 | Included | Trust required before managed execution. |
| PRJ-003 | Restricted | Canonical boundary enforced; external authorization deferred. |
| PRJ-004 | Restricted | Detect Spec Kit presence without enabling its domain. |
| PRJ-005 | Included | Provider-dependent execution and remote quotas remain unavailable offline. |
| ANL-001 | Included | Native redacted-fixture-tested Claude and Codex ingestion. |
| ANL-002 | Included | Cost is always labeled API-equivalent estimate. |
| ANL-003 | Restricted | Agent and phase may be `Unknown`; workflow-step analytics are deferred. |
| ANL-004 | Restricted | Fixed distributed pricing snapshot; local override UI deferred. |
| ANL-005 | Included | Provider data preferred; inference explicitly labeled. |
| ANL-006 | Included | Five-minute quota refresh, backoff, watcher, and reconciliation. |
| ANL-007 | Restricted | Approved fixed thresholds; threshold configuration deferred. |
| ANL-008 | Restricted | Aggregates retained; original payload always disabled. |
| ANL-009 | Restricted | Versioned snapshot included; local override management deferred. |

### 6.2 Agents, Sessions, and Agent Loop

| Requirement | Classification | MVP restriction or note |
| --- | --- | --- |
| AGT-001 | Included | Agent is a named role; provider and model belong to Session. |
| AGT-002 | Included | Common Claude/Codex adapter interface. |
| AGT-003 | Restricted | Managed start/stop and one-time prompt; arbitrary input and external resume deferred. |
| AGT-004 | Included | External Sessions are detected and read-only. |
| AGT-005 | Included | Unassigned association remains reconstructable. |
| AGT-006 | Restricted | External Sessions observable; external resume deferred. |
| AGT-007 | Included | Terminal View is strictly read-only. |
| AGT-008 | Deferred | Rearrangeable tabs and side-by-side layout. |
| AGT-009 | Included | Concurrent Claude and Codex Sessions in the active project. |
| AGT-010 | Included | Only Converge-managed processes are terminated. |
| LOOP-001 | Included | Standard phases with explicit inference labeling. |
| LOOP-002 | Included | Read-only progressive disclosure. |
| LOOP-003 | Restricted | `.converge/workflow.yml`, then inference; Spec Kit source deferred. |
| LOOP-004 | Deferred | Independent, sequential, and collaborative workflow execution. |
| LOOP-005 | Deferred | Visual workflow editor. |
| LOOP-006 | Deferred | Assigning Agents to workflow steps. |
| LOOP-007 | Deferred | Workflow Retry, Skip, branch continuation, and Stop workflow. |
| LOOP-008 | Restricted | Explicit Session Start/Stop controls only; workflow controls and resume deferred. |

### 6.3 Context Files and Spec Kit

| Requirement | Classification | MVP restriction or note |
| --- | --- | --- |
| CTX-001 | Included | Recognized files only within the canonical boundary. |
| CTX-002 | Restricted | Viewer for recognized files; editing only eligible existing `.specify/` files. |
| CTX-003 | Restricted | Applies to eligible `.specify/` edits. |
| CTX-004 | Restricted | Applies to eligible `.specify/` edits. |
| CTX-005 | Included | Watcher plus one-minute reconciliation. |
| CTX-006 | Included | Project YAML and Markdown remain authoritative. |
| SPEC-001 | Restricted | Presence detection and existing files only; Converge works without Spec Kit. |
| SPEC-002 | Deferred | Spec Kit initialization. |
| SPEC-003 | Deferred | Spec Kit command execution and shell approval. |
| SPEC-004 | Deferred | Lifecycle board and artifact workspace. |
| SPEC-005 | Deferred | Triggers, monitor, gates, and quality panel. |
| SPEC-006 | Deferred | SDD phase analytics. |

### 6.4 Security, Platform, and Distribution

| Requirement | Classification | MVP restriction or note |
| --- | --- | --- |
| SEC-001 | Included | Implemented through narrow HTTP/WebSocket operations rather than Tauri Commands. |
| SEC-002 | Restricted | Project scopes enforced; external target authorization deferred. |
| SEC-003 | Included | Required endpoints only; no remote interface content. |
| SEC-004 | Included | Provider credentials remain read-only and in memory. |
| SEC-005 | Guardrail | The MVP introduces no Converge-owned persisted secrets. |
| SEC-006 | Restricted | No telemetry or remote crash reporting; opt-in reporting UI deferred. |
| SEC-007 | Restricted | Original payload storage cannot be enabled in the MVP. |
| SEC-008 | Restricted | Redacted local diagnostics only; nothing is sent automatically. |
| SEC-009 | Restricted | Approved reads and lifecycle controls only; arbitrary shell and external paths remain unavailable. |
| PLT-001 | Restricted | React/Vite plus local Rust service; Tauri 2 desktop shell deferred. |
| PLT-002 | Included | SQLite, SQLx, migrations, ledger, and rebuildable aggregates. |
| PLT-003 | Restricted | HTTP request/response and WebSocket streams replace Tauri IPC in the MVP. |
| PLT-004 | Included | TanStack Query and Zustand responsibilities remain separated. |
| PLT-005 | Restricted | Linux x86_64 is primary; Windows x86_64 receives continuous technical validation; Linux ARM64 distribution is deferred. |
| PLT-006 | Deferred | Desktop packages and AUR artifacts. |
| PLT-007 | Deferred | Signed updater. |
| PLT-008 | Included | Risk-appropriate Rust, frontend, integration, security, and browser E2E tests. |
| PLT-009 | Restricted | Base CI included; tagged release, signing, checksums, and publication deferred. |
| PLT-010 | Included | MIT license and required third-party attribution. |
| PLT-011 | Included | WCAG 2.2 AA target and approved accessibility criteria. |
| PLT-012 | Included | Unsupported provider formats are isolated and visible. |

## 7. Explicitly Deferred Scope

The following capabilities remain in the complete product scope but are outside
the MVP:

- Tauri 2 desktop shell.
- Linux and Windows packaging matrix, including AppImage, `.deb`, `.rpm`, AUR,
  `.pkg.tar.zst`, NSIS, and MSI.
- Linux ARM64 distribution artifact.
- Signed updater, release signing, checksums, and automated release publication.
- External Session resume.
- Rearrangeable Session tabs and side-by-side layout.
- Continuous interactive terminal input.
- Independent, sequential, and collaborative workflows.
- Visual workflow editor, Agent assignment, Retry, Skip, and workflow controls.
- General editing of `CLAUDE.md`, `AGENTS.md`, `.claude/`, and `.converge/`.
- Spec Kit initialization, command execution, lifecycle board, artifacts, triggers,
  monitor, gates, quality panel, and SDD Analytics.
- Pricing override, alert-threshold, original-payload, and retention configuration.
- Authorization of symlinks or targets outside the canonical project boundary.
- Telemetry and crash-reporting opt-in flows.

Deferred does not mean removed. Moving any deferred item into the MVP requires
an approved product decision and synchronized updates to normative artifacts.

## 8. Delivery Slices and Gates

### Slice 0 — Foundation

**Outcome:** a runnable, secure skeleton that proves the architectural seams.

- Workspace and module boundaries.
- Clean Architecture dependency rules.
- React/Vite frontend and local Rust launcher.
- HTTP/WebSocket transport and typed common client.
- Loopback, random-port, ephemeral-token, `Host`/`Origin`, and CORS protections.
- SQLite/SQLx, forward-only migrations, immutable ledger, and aggregate rebuild.
- `justfile`, formatting, linting, tests, and base GitHub Actions workflows.

**Exit gate:** the application launches through the documented command, exposes a
minimal authenticated health path, persists and rebuilds a representative
normalized event without changing the ledger, and passes the approved Foundation
test matrix on required CI environments.

### Slice 1 — Project and Trust

**Outcome:** the user can select, inspect, trust, revoke, and safely bound one
local project.

- Folder selection and canonicalization.
- Restricted and trusted states.
- Project-scoped capabilities.
- External symlink blocking.
- Claude, Codex, context-file, and Spec Kit presence detection.

**Exit gate:** a real project can be selected and trusted without permitting a
read or execution outside its canonical boundary.

### Slice 2 — Analytics

**Outcome:** the user can inspect reconciled Claude and Codex consumption and
quota information with honest provenance and freshness.

- Native parsers and redacted fixtures.
- Idempotent ingestion and 30-day-first backfill.
- Immutable events and rebuildable aggregates.
- Metrics, dimensions, quotas, pricing, alerts, and `Stale` state.
- Analytics interface and offline persisted views.

**Exit gate:** repeated imports do not duplicate events, aggregate rebuilds are
equivalent, both providers are represented, and the interface remains usable
during backfill and quota failures.

### Slice 3 — Agents and Sessions

**Outcome:** the user can run and observe concurrent managed Claude and Codex
Sessions through explicit lifecycle controls.

- Named Agents and provider-neutral ports.
- Managed PTY lifecycle.
- Read-only terminal streaming.
- External Session detection and `Unassigned Sessions`.
- `Start`, `Needs Attention`, graceful `Stop`, and confirmed `Force Kill`.

**Exit gate:** Claude and Codex can run simultaneously, streams stay attributed,
external Sessions remain read-only, and shutdown leaves no managed orphan process.

### Slice 4 — Agent Loop

**Outcome:** the user can inspect a read-only, progressively disclosed execution
loop with visible source and confidence.

- Standard phase normalization.
- `.converge/workflow.yml` source.
- Event-based inference.
- Turns, commands, and tool calls in temporal order.
- Normalized metadata persistence without raw payloads.

**Exit gate:** representative Claude and Codex Sessions produce inspectable loops,
source precedence is deterministic, and unsupported elements do not break the view.

### Slice 5 — Context Files

**Outcome:** the user can inspect recognized project context and safely edit only
eligible existing `.specify/` files.

- Recognized file tree and metadata.
- Markdown and raw-text viewer.
- Watcher and reconciliation.
- Atomic writes, conflicts, and rotating backups.
- Explicit read-only and editable boundaries.

**Exit gate:** external edits reconcile, encoding/path errors do not corrupt files,
and no supported save silently overwrites a concurrent change.

### Slice 6 — Hardening and Acceptance

**Outcome:** the accumulated MVP journey meets its release-quality gates.

- White-box and black-box coverage.
- Security and trusted-boundary validation.
- Ledger rebuild and idempotency validation.
- Offline behavior and provider parity.
- WCAG 2.2 AA acceptance for the main journey.
- Documentation and operational checks.

**Exit gate:** every checklist item in Section 9 passes or has an explicitly
approved, documented exception. No later slice may be considered complete before
its preceding slice is demonstrable, integrated, tested, and approved.

## 9. Acceptance Checklist

### 9.1 Project and Trust

- [ ] **AC-001:** Converge starts without automatically reopening a project.
- [ ] **AC-002:** Selecting one folder leaves the product restricted until trust is explicitly granted.
- [ ] **AC-003:** Trust and revocation are bound to the canonical project identity.
- [ ] **AC-004:** Path traversal and external symlinks cannot escape the trusted boundary.
- [ ] **AC-005:** Claude, Codex, context files, and Spec Kit presence are detected without installation or credential changes.

### 9.2 Analytics

- [ ] **AC-006:** Redacted Claude and Codex fixtures produce normalized events with provider parity.
- [ ] **AC-007:** Re-importing the same source is idempotent and does not duplicate ledger events.
- [ ] **AC-008:** Aggregate rebuilds produce equivalent results for the same ledger and rule version.
- [ ] **AC-009:** The last 30 days become usable before background backfill finishes.
- [ ] **AC-010:** Tokens, averages, history, quotas, renewal, and estimated API-equivalent cost are inspectable.
- [ ] **AC-011:** Provider, model, Session, and time dimensions reconcile with overall totals; unavailable Agent or phase values remain `Unknown`.
- [ ] **AC-012:** Reported, inferred, estimated, unavailable, and `Stale` values are distinguishable and never silently treated as zero.
- [ ] **AC-013:** Quota failures use backoff, preserve the last known value, and do not block the interface.

### 9.3 Agents and Sessions

- [ ] **AC-014:** A named Agent may own Claude and Codex Sessions without storing provider or model on the Agent.
- [ ] **AC-015:** Starting a managed Session sends the approved task prompt once and exposes no generic shell input.
- [ ] **AC-016:** Claude and Codex can run simultaneously and keep streams attributed to the correct Session.
- [ ] **AC-017:** Terminal Views are read-only while still supporting selection and copying.
- [ ] **AC-018:** Requests for further interaction produce a visible `Needs Attention` state.
- [ ] **AC-019:** `Stop` attempts graceful termination and offers `Force Kill` only after timeout and confirmation.
- [ ] **AC-020:** External Sessions are detected, marked External, and never stopped when Converge does not own the process.
- [ ] **AC-021:** Closing Converge warns about active managed Sessions and leaves no managed orphan process.

### 9.4 Agent Loop

- [ ] **AC-022:** Every representative Session can be shown through the five standard phases.
- [ ] **AC-023:** `.converge/workflow.yml` takes precedence over event inference and the active source is visible.
- [ ] **AC-024:** Inferred phases display their confidence and are never presented as provider-reported facts.
- [ ] **AC-025:** Expanded phases preserve the temporal order of turns, commands, and tool calls.
- [ ] **AC-026:** Persisted loop data contains normalized metadata but no raw output, complete arguments, prompts, or secrets.

### 9.5 Context Files

- [ ] **AC-027:** The recognized tree reflects creation, modification, and removal within the trusted boundary.
- [ ] **AC-028:** Recognized Markdown and text files open in rendered and raw views without corrupting unsupported encodings.
- [ ] **AC-029:** Files outside eligible existing `.specify/` Markdown, YAML, and text remain read-only.
- [ ] **AC-030:** Eligible saves are atomic and preserve the previous version on failure.
- [ ] **AC-031:** Concurrent external changes block silent overwrite and offer Compare, Reload, and confirmed Overwrite.
- [ ] **AC-032:** A confirmed overwrite creates a rotating backup outside the repository before replacement.

### 9.6 Security and Privacy

- [ ] **AC-033:** The local service accepts authenticated traffic only through loopback, a random port, and the current ephemeral token.
- [ ] **AC-034:** Invalid `Host`, `Origin`, CORS, token, scope, and Project ID combinations are denied.
- [ ] **AC-035:** The browser cannot invoke a generic shell or unrestricted filesystem operation.
- [ ] **AC-036:** Provider credentials are never written to SQLite, files, diagnostics, or logs and are not modified.
- [ ] **AC-037:** Original provider payloads remain disabled and normalized metadata does not expose prompts, outputs, or secrets.
- [ ] **AC-038:** Local diagnostics redact paths, tokens, credentials, and unauthorized sensitive payloads and are never sent automatically.
- [ ] **AC-039:** No known failure allows silent data loss, credential exposure, or escape from the trusted project boundary.

### 9.7 Platform, Quality, and Release Gate

- [ ] **AC-040:** The documented `just` command launches the frontend and authenticated local service together.
- [ ] **AC-041:** Domain and Application compile and test without dependencies on transport, UI, database, filesystem, PTY, Tauri, or providers.
- [ ] **AC-042:** SQLx migrations are forward-only, tested, and compatible with aggregate rebuild.
- [ ] **AC-043:** Applicable Rust, frontend, contract, HTTP/WebSocket integration, security, and browser E2E tests pass.
- [ ] **AC-044:** Complete MVP CI passes on Linux and build plus technical tests pass on Windows x86_64.
- [ ] **AC-045:** The main journey satisfies keyboard access, visible focus, non-color state communication, reduced motion, and WCAG 2.2 AA requirements.
- [ ] **AC-046:** Offline mode preserves history, aggregates, persisted Agent Loops, and locally available context files while clearly disabling provider-dependent actions.
- [ ] **AC-047:** `PRODUCT.md`, `PRD.md`, `MVP.md`, `DESIGN.md`, approved Spec Kit artifacts, and the Decision Log remain reconciled.

## 10. Testing and Quality Strategy

Testing is mandatory according to risk and the Project Constitution.

- Use Red-Green-Refactor for Domain logic, Application use cases, contracts,
  security boundaries, and bug fixes.
- Use Rust unit and integration tests with redacted Claude and Codex fixtures.
- Use Vitest and React Testing Library for frontend behavior.
- Test HTTP and WebSocket contracts and integrations.
- Test SQLx migrations, idempotent ingestion, and aggregate rebuilds.
- Test authorization, loopback, tokens, origins, paths, symlinks, credentials,
  redaction, and payload boundaries.
- Cover the accumulated browser journey with the approved E2E stack.
- Run complete lint, test, and build gates on Linux.
- Continuously run supported build and technical tests on Windows x86_64.

Every slice MUST end demonstrable, integrated into the accumulated journey, and
covered by appropriate white-box and black-box tests before the next slice.

## 11. Provider Parity

Claude and Codex MUST complete the same MVP journey and expose the same primary
capabilities. Provider-specific differences may appear in labels, colors, or
visual identity only when they remain within the Graphite Signal design system,
preserve accessibility, and do not change the meaning of states.

An unavailable provider capability MUST be exposed honestly; it MUST NOT be
silently emulated in a way that changes product semantics.

## 12. Performance and Offline Behavior

- `Fast` remains a qualitative MVP requirement.
- Arbitrary numeric performance budgets MUST NOT be invented before a reliable
  baseline exists.
- Import, backfill, reconciliation, and terminal streams MUST NOT block primary
  interface interaction.
- Observable regressions MUST be investigated and protected by tests when
  applicable.
- Local history, aggregates, persisted loops, and available files remain usable
  offline.
- Provider execution and remote quota retrieval clearly require connectivity.

## 13. Constitution Compliance

This delivery plan explicitly preserves:

- Versioned sources of truth and stable requirement IDs.
- Modular Clean Architecture with inward dependencies.
- Local-first operation and explicit human control.
- Narrow trust, security, privacy, and permission boundaries.
- Immutable normalized events and reconstructable aggregates.
- Risk-appropriate testing and demonstrable vertical slices.
- Accessibility as an acceptance criterion.
- Human-controlled Git and release actions.

**Compliance result:** compliant with Project Constitution v1.0.1. Each generated
`spec.md`, `plan.md`, and `tasks.md` MUST contain its own explicit Constitution
compliance check before implementation begins.

## 14. References

- Product requirements: `docs/PRD.md`
- Product direction: `docs/PRODUCT.md`
- Design system: `docs/DESIGN.md`
- Project Constitution: `.specify/memory/constitution.md`
- MVP Definition & Prioritization: https://app.notion.com/p/39f16378a61d81d29689f1eeb3320341
- Decision Log: https://app.notion.com/p/39f16378a61d8168a5cdc35cf924439d
- Codeburn: https://github.com/getagentseal/codeburn
- GitHub Spec Kit: https://github.com/github/spec-kit

---

Changes to MVP inclusion, restrictions, slice order, or acceptance gates require
an explicit approved decision and synchronized updates to all affected normative
documents before implementation.
