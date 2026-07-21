# Converge — Product Requirements Document

## 1. Metadata and Status

| Field | Value |
| --- | --- |
| Product | Converge |
| Document | Product Requirements Document |
| Version | 0.2 |
| Date | July 17, 2026 |
| Status | Stages 3A and 3B approved |
| Language | English |
| Audience | Product, Engineering, and future contributors |
| Planned license | MIT |

This document consolidates the complete validated scope for Converge. Stage 3B selected and approved the subset for the first delivery, described in Section 20 and detailed in `MVP.md`. This classification neither removes nor demotes the remaining items in the complete vision.

## 2. Executive Summary

Converge is a local-first desktop application for individual developers to observe, control, and organize development agents in one place. The product brings together Claude and Codex subscription analytics, sessions and visual terminals, Agent Loops, workflows, context files, and optional GitHub Spec Kit integration.

The product complements existing CLIs and terminals. It is not intended to replace Claude Code, Codex CLI, or the system terminal. The interface provides observability and explicit controls over predefined operations while keeping actual execution in the providers and the Rust core.

Converge uses Tauri 2, React, TypeScript, Vite, and Rust. Internal storage uses SQLite with SQLx, immutable normalized events, and reconstructable aggregates. Project YAML and Markdown files remain shareable authoritative sources.

## 3. Problem and Context

Development-agent work is fragmented across terminals, local files, sessions, and provider-specific interfaces. This fragmentation makes it difficult to:

- Understand limits, consumption, renewal, and estimated API-equivalent costs.
- Follow multiple agents and sessions simultaneously.
- Identify the phase, turn, command, or tool call of each execution.
- Coordinate independent, sequential, or collaborative executions.
- Keep context and orchestration files consistent.
- Follow the lifecycle of projects that use Spec-Driven Development.

Current tools also do not provide a suitable unified visual interface for this set of needs. Users switch among different sources and must manually reconstruct the state of their executions.

## 4. Vision and Value Proposition

### Vision

Be the local-first control center for developers who work with coding agents.

### Value Proposition

Converge centralizes consumption, quotas, agents, sessions, Agent Loops, workflows, and context files in a secure, observable, and fluid desktop interface while preserving user control and the project's authoritative sources.

### Identity

The experience MUST be modern, precise, fluid, sophisticated, and minimalist. The product identity and interface terminology remain in English.

## 5. Product Goals

The goals follow an explicit order of priority:

1. Control limits, consumption, and estimated costs.
2. Follow agents in one place.
3. Manage context and orchestration files.

Supporting goals:

- Reduce fragmentation across providers, terminals, and files.
- Make states, inferences, failures, and risks visible.
- Allow workflows to be defined visually and persisted as authoritative files.
- Keep the product useful when Spec Kit is not present.
- Create an extensible foundation without introducing dynamic plugins prematurely.

## 6. Non-Goals of the Current Scope

The following are not part of the current complete scope:

- API usage billing.
- Cloud synchronization or multi-user collaboration.
- Background or system-tray operation.
- Providers other than Claude and Codex.
- Dynamic plugins.
- Automatic installation of Claude Code or Codex CLI.
- Management or modification of provider credentials.
- A generic terminal with arbitrary input through the React interface.
- More than one active project folder at a time.
- Remote content loaded inside the WebView.
- Telemetry or crash reporting enabled by default.
- Integration with SpecifyApp tokens.

## 7. Primary User

### Individual Developer

A person who uses Claude Code, Codex CLI, or both in local projects and needs to follow consumption, sessions, agents, workflows, and files without relying on a remote solution.

Expected characteristics:

- Works with one project folder at a time.
- Switches between providers or maintains multiple sessions simultaneously.
- Values privacy, local control, and transparency.
- May use Spec Kit but does not depend on it.
- Is familiar with terminals, Markdown, YAML, and Git.

Open source, commercialization, and team use are future directions, not equivalent audiences in the current scope.

## 8. Product Principles

- **Local-first:** data and control remain primarily on the machine.
- **Safe by default:** projects begin restricted and risky actions require approval.
- **Observable:** states, sources, inferences, failures, and progress MUST be visible.
- **Explicit control:** relevant operations originate from clear user actions.
- **Progressive disclosure:** summary first, details on demand.
- **Provider-agnostic core:** common rules belong to the core, not to a provider's interface.
- **Minimal:** the interface reduces noise and highlights operational state.
- **Fast:** interactions SHOULD feel immediate. In the MVP, performance will be handled qualitatively and protected against regressions, without arbitrary numeric budgets.

## 9. Scope Classification

### Current Complete Scope

Includes every capability described in this PRD for Projects & Trust, Analytics, Agents & Sessions, Agent Loop & Workflows, Context Files, Spec Kit, Security & Privacy, and Platform & Distribution.

### MVP

The selected scope is a locally executed, web-based vertical slice: select and trust a folder, detect Claude and Codex, import and inspect Analytics, start and follow managed sessions, view the Agent Loop, and inspect context files. When `.specify/` already exists, Markdown, YAML, and text files may be edited using safe writes, without enabling the remaining Spec Kit integrations.

The runtime will be React/Vite in the browser with a local Rust service, HTTP for request/response, and WebSocket for streams. The core will follow Clean Architecture within a modular monolith and remain independent of HTTP, WebSocket, and Tauri. The operational document and complete requirements matrix are in `MVP.md`.

### Converge 2.0

- Configurable background and system-tray operation.
- Additional compiled adapters.
- Presets, bundles, and extensions.
- Traceability.
- Convergence dashboard.
- Export to GitHub Issues.
- Bug workflow.

### Converge 3.0

- Dynamic plugins.

## 10. Domain Concepts and Entities

| Entity | Definition |
| --- | --- |
| Project | Local folder chosen by the user and linked during the current execution. |
| Trust | Explicit state that enables execution capabilities within the project's canonical boundary. |
| Provider | External agent integration; initially Claude or Codex. |
| Agent | Named role that may own multiple sessions, such as Claude Reviewer or Codex Builder. |
| Session | Specific provider execution, managed by Converge or detected externally. |
| Terminal View | Read-only visual representation of a session's output. |
| Event | Normalized, immutable, and orderable record of relevant activity. |
| Agent Loop | Structure of phases, turns, commands, and tool calls for an execution. |
| Workflow | Graph or sequence of steps executed by named agents. |
| Context File | Markdown, text, or configuration file observed and editable by Converge. |
| Artifact | Result of the Spec Kit lifecycle or another workflow. |
| Aggregate | Reconstructable projection derived from the event ledger. |
| Quota Snapshot | Observed or inferred state of a provider's limit, consumption, and renewal. |

## 11. Functional Journeys

### 11.1 Link and Trust a Project

1. Converge starts without an active project.
2. The user selects a folder.
3. The core resolves the canonical path, detects external symlinks, and identifies available integrations.
4. The project remains restricted.
5. The user reviews the access boundary and explicitly grants trust.
6. Converge starts watchers, reconciliation, adapters, and reads from allowed sources.

### 11.2 Inspect Consumption and Quotas

1. Converge reads local provider events.
2. Events are normalized and aggregated.
3. Quotas are queried from allowed endpoints.
4. When the provider is unavailable, the last known value remains visible as Stale.
5. Local inferences are distinguished from provider-supplied data.
6. The user navigates by period, provider, agent, model, and Agent Loop phase.

### 11.3 Manage Sessions

1. The user creates a named agent or selects an existing one.
2. The user starts a Claude or Codex session in a managed PTY.
3. Output appears in a read-only Terminal View.
4. The session can be followed, resumed, or stopped through explicit controls.
5. External sessions are detected and, when unrecognized, appear under Unassigned Sessions.
6. Tabs can be reordered or displayed side by side.

### 11.4 Configure and Run Workflows

1. The user opens or creates a workflow.
2. The user defines steps, relationships, agents, and execution mode.
3. The visual editor saves the project's authoritative file.
4. The user reviews actions and approves risky operations.
5. Execution displays progress by phase, turn, command, and tool call.
6. Failures stop dependencies while preserving independent branches when safe.

### 11.5 Manage Context Files

1. Converge presents a tree of recognized files.
2. The user opens `CLAUDE.md`, `AGENTS.md`, files in `.claude`, `.specify`, `.converge`, or other allowed files.
3. The editor tracks external changes.
4. Saves use atomic writes.
5. Conflicts block silent overwrites and offer Compare, Reload, or Overwrite.
6. Backups enable recovery.

### 11.6 Use Spec Kit

1. Converge detects whether Spec Kit is present.
2. If absent, the user may choose to initialize it.
3. The user selects Claude, Codex, or both.
4. The shell displays the exact command and requests approval when risk exists.
5. The lifecycle board, artifacts, triggers, monitor, gates, quality panel, and analytics follow the execution.
6. If Spec Kit is not used, the rest of Converge remains functional.

## 12. Functional Requirements

### 12.1 Projects & Trust

#### PRJ-001 — Explicit Folder Selection

**Description:** Converge MUST start without an active project and request folder selection on every launch.

**Rationale:** Prevents implicit access to the last project and reinforces explicit control.

**Acceptance criteria:**

- No folder is reopened automatically.
- Only one folder may remain active at a time.
- Canceling selection leaves the product without an active project.

**Dependencies:** PRJ-002, PRJ-003.

#### PRJ-002 — Explicit Trust

**Description:** Every newly selected folder MUST begin in restricted mode and require explicit trust before running agents or workflows.

**Rationale:** Local projects may contain untrusted files or instructions.

**Acceptance criteria:**

- Allowed reads and blocked capabilities are presented clearly.
- Executions remain unavailable until confirmation.
- Revoking trust prevents new executions.

**Dependencies:** SEC-002, SEC-009.

#### PRJ-003 — Canonical Boundary

**Description:** The canonical path of the active folder MUST define the default access boundary.

**Rationale:** Prevents accidental or malicious escapes into other areas of the system.

**Acceptance criteria:**

- Paths are canonicalized before access.
- Symlinks that escape the project are blocked.
- An external target may be enabled only through explicit, target-specific authorization.

**Dependencies:** SEC-001, SEC-002.

#### PRJ-004 — Environment Detection

**Description:** Converge MUST detect Claude Code, Codex CLI, session files, context files, and the presence of Spec Kit.

**Rationale:** The initial state needs to reflect the capabilities that are actually available.

**Acceptance criteria:**

- Each integration displays Available, Missing, Unauthenticated, or Unsupported when applicable.
- Detection does not modify installations or credentials.
- Failure in one integration does not block the others.

**Dependencies:** AGT-002, SPEC-001.

#### PRJ-005 — Offline Local Access

**Description:** History, persisted aggregates, workflows, and files MUST remain available without a connection.

**Rationale:** The product is local-first, although AI execution depends on the internet.

**Acceptance criteria:**

- Local data can be inspected offline.
- Attempts to run AI indicate that a connection is required.
- Remote data retains its last value and Stale state.

**Dependencies:** ANL-006, PLT-002.

### 12.2 Analytics

#### ANL-001 — Native Claude and Codex Ingestion

**Description:** The Rust core MUST include native parsers for Claude and Codex events and sessions, adapted from relevant Codeburn behavior.

**Rationale:** Avoids a runtime dependency and enables controlled normalization.

**Acceptance criteria:**

- Redacted fixtures from both providers are processed.
- Codeburn is not required at runtime.
- MIT attribution is preserved.

**Dependencies:** PLT-002, PLT-010.

#### ANL-002 — Consumption Metrics

**Description:** The product MUST display tokens, daily, weekly, and monthly averages, percentage of limit, renewal, history, and estimated API-equivalent cost.

**Rationale:** Consumption control is the product's highest priority.

**Acceptance criteria:**

- Metrics can be filtered by period.
- Missing values are not presented as zero.
- Estimated cost is never labeled as subscription billing.

**Dependencies:** ANL-004, ANL-009.

#### ANL-003 — Analytics Dimensions

**Description:** Metrics MUST be attributable to provider, agent, session, model, phase, step, and period.

**Rationale:** The user needs to understand where consumption occurred.

**Acceptance criteria:**

- Unattributed events remain available as Unassigned or Unknown.
- Aggregations preserve the total when dimensions are applied or removed.
- Attribution can be reconstructed from the ledger.

**Dependencies:** AGT-001, LOOP-001, PLT-002.

#### ANL-004 — API-Equivalent Cost

**Description:** Converge MUST calculate a comparative estimate based on versioned API prices.

**Rationale:** Subscriptions do not provide equivalent per-session billing, but the estimate helps explain value and consumption.

**Acceptance criteria:**

- Every value is identified as Estimate.
- The calculation records the pricing snapshot version.
- Local overrides are reflected without changing distributed snapshots.

**Dependencies:** ANL-009.

#### ANL-005 — Quotas

**Description:** Converge MUST query the quota endpoints used by Codeburn and use clearly identified local inference when necessary.

**Rationale:** Actual quotas are preferable, but endpoint availability may vary.

**Acceptance criteria:**

- Provider data and local inference have distinct labels.
- Credentials are only read in memory.
- Failures use backoff and do not block the interface.

**Dependencies:** SEC-004, ANL-006.

#### ANL-006 — Updates and Freshness

**Description:** Quotas MUST update every five minutes with backoff; watchers MUST process changes immediately and reconciliation MUST occur every minute.

**Rationale:** Combines responsiveness with recovery from missed events.

**Acceptance criteria:**

- Every remote view displays the last-updated timestamp.
- Expired data receives the Stale state.
- Manual refresh respects provider limits and backoff.

**Dependencies:** CTX-005, PLT-003.

#### ANL-007 — Visual Alerts

**Description:** The product MUST apply configurable quota thresholds.

**Rationale:** The user needs to recognize exhaustion risk quickly.

**Acceptance criteria:**

- Defaults: Normal below 70%, Attention from 70%, High from 85%, and Critical from 95%.
- States do not rely on color alone.
- Threshold changes are persisted locally.

**Dependencies:** PLT-011.

#### ANL-008 — Retention

**Description:** Aggregates MUST be retained indefinitely; original payload storage MUST be disabled by default and, when enabled, use a configurable default retention of 30 days.

**Rationale:** Preserves useful history without collecting unnecessary sensitive data.

**Acceptance criteria:**

- Disabling payloads does not prevent normalized metrics.
- Expiration removes only eligible payloads.
- Aggregates remain reconstructable from available normalized events.

**Dependencies:** SEC-007, PLT-002.

#### ANL-009 — Versioned Pricing

**Description:** Converge MUST distribute a versioned pricing snapshot by model and accept local overrides.

**Rationale:** Prices change, and historical calculations need to be reproducible.

**Acceptance criteria:**

- The snapshot has a version and date.
- A local override does not overwrite the distributed file.
- Unknown models display unavailable pricing instead of a false estimate.

**Dependencies:** SEC-003.

### 12.3 Agents, Sessions & Terminal

#### AGT-001 — Named Agents

**Description:** An Agent MUST represent a named role capable of owning multiple sessions.

**Rationale:** The product's mental model is oriented around roles, not only providers.

**Acceptance criteria:**

- The user can create and rename an agent and associate sessions with it.
- An agent may contain Claude sessions, Codex sessions, or both when compatible with its workflow.
- Removing an association does not delete session history.

**Dependencies:** ANL-003.

#### AGT-002 — Common Adapter Interface

**Description:** The Rust core MUST expose a common interface implemented by the Claude and Codex adapters.

**Rationale:** Preserves shared rules and prepares for future expansion.

**Acceptance criteria:**

- Both adapters expose capabilities and limitations.
- The interface supports detection, sessions, events, quotas, and resume when available.
- An adapter failure is isolated.

**Dependencies:** PLT-001.

#### AGT-003 — PTY-Managed Sessions

**Description:** Sessions started by Converge MUST run in a managed PTY.

**Rationale:** Lifecycle control and reliable streaming require process ownership.

**Acceptance criteria:**

- Start, resume, and stop use specific Rust commands.
- Output is streamed without granting the frontend a generic shell.
- Termination updates the final session state.

**Dependencies:** SEC-001, PLT-003.

#### AGT-004 — External Session Detection

**Description:** Sessions started outside Converge MUST be detected through local provider files.

**Rationale:** History needs to include work performed in external terminals.

**Acceptance criteria:**

- New sessions are incorporated without restarting the application.
- External sessions are marked as External.
- Lack of process control is reflected in the available actions.

**Dependencies:** ANL-001, CTX-005.

#### AGT-005 — Unassigned Sessions

**Description:** External sessions without a recognized agent MUST be grouped under Unassigned Sessions.

**Rationale:** Prevents incorrect automatic attribution.

**Acceptance criteria:**

- The user can associate the session with an existing agent.
- The user can create an agent from the session.
- Previous metrics are reassigned in a reconstructable manner.

**Dependencies:** AGT-001, ANL-003.

#### AGT-006 — External Control Boundaries

**Description:** Active external sessions MUST be observable but may be stopped only when the process is controlled by Converge.

**Rationale:** Terminating processes not owned by the product is unsafe and unreliable.

**Acceptance criteria:**

- An unavailable Stop action displays the reason.
- Compatible sessions may resume in a new managed PTY when supported by the provider.
- The resumed session preserves a reference to the original session.

**Dependencies:** AGT-003.

#### AGT-007 — Read-Only Visual Terminal

**Description:** Every session MUST have a read-only Terminal View.

**Rationale:** Preserves observability without exposing an arbitrary shell to the frontend.

**Acceptance criteria:**

- The user can select and copy text.
- Arbitrary textual input is not sent to the process.
- Allowed actions appear as explicit controls.

**Dependencies:** SEC-001, PLT-003.

#### AGT-008 — Session Layout

**Description:** Session tabs MUST be rearrangeable and support side-by-side viewing.

**Rationale:** Multiple simultaneous agents require visual comparison.

**Acceptance criteria:**

- Rearranging does not alter session state.
- Temporary layout is maintained in UI state.
- Closing a view does not stop the session without an explicit action.

**Dependencies:** PLT-004.

#### AGT-009 — Concurrency

**Description:** Multiple agents and sessions MUST be able to run simultaneously within the active folder.

**Rationale:** Parallel execution is a central part of orchestration.

**Acceptance criteria:**

- Streams remain attributed to the correct sessions.
- Failure of one session does not stop independent sessions.
- Every execution respects the same canonical project boundary.

**Dependencies:** PRJ-003, LOOP-004.

#### AGT-010 — Application Shutdown

**Description:** Closing Converge MUST terminate its managed local processes.

**Rationale:** Background and system-tray operation are reserved for Converge 2.0.

**Acceptance criteria:**

- The user is warned when managed sessions are active.
- Shutdown attempts graceful termination before forcing it.
- External sessions are not terminated.

**Dependencies:** AGT-003.

### 12.4 Agent Loop & Workflows

#### LOOP-001 — Standard Phases

**Description:** The Agent Loop MUST use Understanding, Planning, Execution, Validation, and Completed as its standard phases.

**Rationale:** Provides a consistent model when no explicit workflow exists.

**Acceptance criteria:**

- Every session can be presented in these phases through events or inference.
- An inferred phase is identified as an inference.
- Explicit configurations may replace the default.

**Dependencies:** LOOP-003.

#### LOOP-002 — Progressive Disclosure

**Description:** Each phase MUST be expandable into turns, commands, and tool calls.

**Rationale:** The user needs to switch between an executive view and detailed investigation.

**Acceptance criteria:**

- The collapsed view shows state and duration.
- Expansion preserves temporal order.
- Unknown elements do not break the loop.

**Dependencies:** PLT-012.

#### LOOP-003 — Source Precedence

**Description:** The Agent Loop MUST resolve its source in this order: Spec Kit workflow YAML, `.converge/workflow.yml`, and event inference using standard phases.

**Rationale:** Explicit sources MUST take precedence over inferences.

**Acceptance criteria:**

- The active source is visible.
- A source change causes predictable reconciliation.
- Absence of Spec Kit does not prevent the loop.

**Dependencies:** SPEC-001, CTX-006.

#### LOOP-004 — Execution Modes

**Description:** Workflows MUST support independent, sequential, and collaborative execution.

**Rationale:** Different tasks require different relationships among agents.

**Acceptance criteria:**

- Dependencies are validated before execution.
- Independent branches may run in parallel.
- Collaboration records participating agents and sessions.

**Dependencies:** AGT-009.

#### LOOP-005 — Visual Editor and Authoritative Source

**Description:** The visual editor MUST save the workflow as an authoritative project file.

**Rationale:** The interface cannot create an invisible or non-shareable proprietary configuration.

**Acceptance criteria:**

- Visual changes generate valid YAML.
- External changes are detected.
- Conflicts follow the Compare, Reload, or Overwrite flow.

**Dependencies:** CTX-003, CTX-004, CTX-006.

#### LOOP-006 — Agent Assignment

**Description:** The user MUST be able to assign named agents to workflow steps.

**Rationale:** Workflows need to express responsibility, not only provider.

**Acceptance criteria:**

- Invalid steps or steps missing a required agent are flagged.
- The same role may be used in multiple steps.
- Execution records the agent actually used.

**Dependencies:** AGT-001, LOOP-005.

#### LOOP-007 — Failure Handling

**Description:** Failures MUST stop dependent steps, allow independent branches to continue, and offer Retry, Skip when allowed, and Stop workflow.

**Rationale:** Prevents silent error propagation without canceling independent work.

**Acceptance criteria:**

- An unavailable Skip action displays the reason.
- Retry creates a new traceable attempt.
- Stop workflow terminates only processes owned by the execution.

**Dependencies:** AGT-003, SEC-009.

#### LOOP-008 — Explicit Controls

**Description:** The interface MUST provide buttons to start, resume, and stop sessions and to run workflows.

**Rationale:** The visual terminal is read-only; operations occur through controlled triggers.

**Acceptance criteria:**

- Unavailable actions have a state and explanation.
- Risky actions require appropriate approval.
- Every action generates a normalized event.

**Dependencies:** AGT-003, SEC-009.

### 12.5 Context Files

#### CTX-001 — File Tree

**Description:** Converge MUST display a tree of context and orchestration files within the allowed boundary.

**Rationale:** Files are distributed across different conventions and directories.

**Acceptance criteria:**

- Supports `CLAUDE.md`, `AGENTS.md`, `.claude`, `.specify`, `.converge`, and other authorized files.
- Blocked external items are identified without reading their content.
- The tree reflects creation, modification, and removal.

**Dependencies:** PRJ-003, CTX-005.

#### CTX-002 — Markdown and Text Editor

**Description:** Allowed files MUST be openable and editable as Markdown or text.

**Rationale:** The user needs to manage context without switching tools.

**Acceptance criteria:**

- Incompatible encoding is reported without corrupting the file.
- Modified state is visible.
- Closing with unsaved changes requests a decision.

**Dependencies:** SEC-002.

#### CTX-003 — Atomic Writes and Backups

**Description:** Saves MUST use atomic writes and rotating backups in application data.

**Rationale:** Orchestration files are critical and cannot be left partially written.

**Acceptance criteria:**

- A write failure preserves the previous version.
- A backup is created before a confirmed overwrite.
- Backups are not written inside the repository without explicit configuration.

**Dependencies:** PLT-002.

#### CTX-004 — Conflicts

**Description:** A concurrent external change MUST block silent overwrite and offer Compare, Reload, and Overwrite.

**Rationale:** The editor coexists with IDEs, CLIs, and agents that modify the same files.

**Acceptance criteria:**

- Compare shows the local version and the version on disk.
- Reload discards only after confirmation when local edits exist.
- Overwrite requires confirmation and a backup.

**Dependencies:** CTX-003, CTX-005.

#### CTX-005 — Watcher and Reconciliation

**Description:** Changes MUST be observed by a filesystem watcher and reconciled every minute.

**Rationale:** Watchers may miss events across different systems.

**Acceptance criteria:**

- Common changes appear immediately.
- Reconciliation corrects missed divergences.
- Event storms are batched without blocking the interface.

**Dependencies:** PLT-003.

#### CTX-006 — Shareable Authoritative Sources

**Description:** Project YAML and Markdown files MUST remain shareable authoritative sources.

**Rationale:** Product configuration needs to accompany the project and be reviewable in Git.

**Acceptance criteria:**

- The database does not silently replace authoritative file content.
- Derived state can be reconstructed when reopening the folder.
- The visual editor displays the corresponding file.

**Dependencies:** LOOP-005, PLT-002.

### 12.6 Spec Kit

#### SPEC-001 — Optional Integration

**Description:** Converge MUST detect Spec Kit and remain fully functional in its other domains when Spec Kit is absent.

**Rationale:** Spec Kit is deeply integrated but not mandatory.

**Acceptance criteria:**

- Absence is presented as Optional, not as an error.
- Analytics, sessions, workflows, and files remain available.
- Detection does not modify the project.

**Dependencies:** PRJ-004.

#### SPEC-002 — Initialization

**Description:** The user MUST be able to initialize Spec Kit by selecting Claude, Codex, or both.

**Rationale:** Setup needs to reflect the providers actually used.

**Acceptance criteria:**

- The choice is explicit before execution.
- The exact command is displayed.
- Initialization does not include an intermediate preview of generated files.

**Dependencies:** SPEC-003, SEC-009.

#### SPEC-003 — Shell Approval

**Description:** Spec Kit commands MUST be executed by the core, display the exact command, and require approval when classified as risky.

**Rationale:** Deep integration MUST NOT reduce execution transparency.

**Acceptance criteria:**

- React does not receive generic shell access.
- The execution path remains within the authorized project unless a specific external target is approved.
- Result and exit code are recorded.

**Dependencies:** SEC-001, SEC-009.

#### SPEC-004 — Lifecycle Board and Artifacts

**Description:** Converge MUST present a lifecycle board and artifact workspace for the SDD process.

**Rationale:** Artifacts and phases are the operational center of Spec Kit.

**Acceptance criteria:**

- State is derived from project files and events.
- Artifacts can be opened in the tree and editor.
- Inferred states are distinguished from explicit states.

**Dependencies:** CTX-001, LOOP-003.

#### SPEC-005 — Monitoring and Gates

**Description:** The integration MUST include triggers, workflow monitor, gates, and a quality panel.

**Rationale:** The user needs to follow lifecycle progress and blockers.

**Acceptance criteria:**

- Gates display state, source, and available evidence.
- Triggers respect trust and risk-based approval.
- Failures follow the general workflow contract.

**Dependencies:** LOOP-007, SEC-009.

#### SPEC-006 — SDD Analytics

**Description:** Consumption MUST be attributable to Spec Kit SDD phases.

**Rationale:** Enables understanding cost and effort by phase.

**Acceptance criteria:**

- An explicit phase takes precedence over inference.
- Events without a phase remain counted.
- Totals by phase reconcile with the overall total.

**Dependencies:** ANL-003, LOOP-003.

### 12.7 Security & Privacy

#### SEC-001 — React/Rust Boundary

**Description:** The React frontend MUST NOT receive generic shell or filesystem access.

**Rationale:** The WebView MUST operate with the smallest possible privilege surface.

**Acceptance criteria:**

- Only specific Tauri Commands are exposed.
- Channels are used only for authorized streams.
- Attempts outside the capabilities are denied.

**Dependencies:** PLT-003.

#### SEC-002 — Capabilities and Scopes

**Description:** Native commands MUST be protected by capabilities and scopes specific to the active project.

**Rationale:** Project trust does not equal unrestricted system access.

**Acceptance criteria:**

- The default scope ends at the canonical path.
- External authorization is specific to the target.
- Changing the active folder invalidates scopes from the previous execution.

**Dependencies:** PRJ-002, PRJ-003.

#### SEC-003 — Network and WebView

**Description:** The network MUST be limited to required Claude, Codex, pricing, and update endpoints; the WebView MUST NOT load remote content and MUST use a restrictive CSP.

**Rationale:** Reduces exfiltration and execution of untrusted content.

**Acceptance criteria:**

- Disallowed endpoints fail visibly.
- Remote HTML is not rendered in the interface.
- Updating the allowlist requires a versioned change.

**Dependencies:** PLT-001.

#### SEC-004 — Provider Credentials

**Description:** Existing provider credentials MUST be read-only, held in memory, and never stored or modified by Converge.

**Rationale:** Providers remain responsible for authentication and credential lifecycle.

**Acceptance criteria:**

- No credential is written to SQLite or logs.
- The product does not rewrite authentication files.
- In-memory data is discarded when no longer needed.

**Dependencies:** ANL-005.

#### SEC-005 — Future Secrets

**Description:** Secrets that may belong to Converge in the future MUST use the native system keyring, never SQLite.

**Rationale:** A local database is not appropriate secret storage.

**Acceptance criteria:**

- Future implementations cannot introduce secrets persisted as plain text in the database.
- Keyring unavailability is handled as a missing capability.

**Dependencies:** PLT-002.

#### SEC-006 — Telemetry and Crash Reporting

**Description:** No telemetry MUST be collected by default; remote diagnostics and crash reporting MUST require opt-in.

**Rationale:** Local privacy is a core principle.

**Acceptance criteria:**

- A new installation starts without collection.
- Opt-in is revocable.
- Revocation prevents new transmissions.

**Dependencies:** SEC-008.

#### SEC-007 — Original Payload

**Description:** Original payload storage MUST be disabled by default and configurable per project.

**Rationale:** Payloads may contain sensitive context.

**Acceptance criteria:**

- Enabling it displays the impact and retention period.
- Default retention is 30 days.
- Disabling it stops new writes without deleting aggregates.

**Dependencies:** ANL-008.

#### SEC-008 — Redacted Diagnostics

**Description:** Local logs MUST be redacted, reviewable before export, and sent only through a manual action.

**Rationale:** Diagnostics MUST NOT expose code, credentials, or context without consent.

**Acceptance criteria:**

- Paths, tokens, and sensitive payloads are redacted according to policy.
- The user views the final file before exporting it.
- No log is sent automatically.

**Dependencies:** SEC-006.

#### SEC-009 — Risk-Based Approval

**Description:** Actions MUST be classified by risk: allowed reads are automatic, lifecycle actions use explicit buttons, and arbitrary shell access, external paths, or destructive actions require confirmation.

**Rationale:** Approval MUST be proportional to impact.

**Acceptance criteria:**

- The approval dialog states the command, target, and risk.
- Denial causes no silent partial change.
- Approval does not implicitly extend to another project or target.

**Dependencies:** PRJ-002, SEC-002.

### 12.8 Platform & Distribution

#### PLT-001 — Stack and Architecture

**Description:** Converge MUST be a desktop application with a web interface, using Tauri 2, React, TypeScript, Vite, Rust, and pnpm in a modular-monolith architecture.

**Rationale:** Combines productive web UI with secure native capabilities.

**Acceptance criteria:**

- Rust modules have explicit boundaries.
- The frontend does not duplicate critical core rules.
- Reproducible builds use pnpm.

**Dependencies:** None.

#### PLT-002 — Persistence and Ledger

**Description:** Internal state MUST use SQLite with SQLx, versioned migrations, immutable normalized events, and fully reconstructable aggregate tables.

**Rationale:** The ledger preserves auditability and allows projections to evolve.

**Acceptance criteria:**

- There is no full ORM.
- Migrations are tested.
- Aggregates can be discarded and rebuilt without changing the ledger.

**Dependencies:** PLT-001.

#### PLT-003 — IPC

**Description:** Tauri Commands MUST serve request/response, Channels MUST carry terminal and high-volume streams, and Events MUST signal lightweight state changes.

**Rationale:** Each mechanism serves a different communication profile.

**Acceptance criteria:**

- High-volume streams do not use generic Events.
- Commands validate arguments and scopes in Rust.
- Cancellation and disconnection do not leave orphaned processes.

**Dependencies:** SEC-001.

#### PLT-004 — Frontend State

**Description:** TanStack Query MUST manage data originating from Rust; Zustand MUST manage layout and temporary UI state.

**Rationale:** Separates persistent domain state from transient preferences.

**Acceptance criteria:**

- Zustand does not become the authoritative source for Rust data.
- Query invalidations follow relevant Events.
- Layout does not alter the ledger.

**Dependencies:** PLT-003.

#### PLT-005 — Systems and Architectures

**Description:** Linux is the primary focus; Linux x86_64, Linux ARM64, and Windows x86_64 MUST be supported.

**Rationale:** Reflects the primary environment without excluding Windows users.

**Acceptance criteria:**

- Tests and builds cover Linux and Windows.
- System-specific limitations are documented.
- Path and PTY fixtures cover system differences.

**Dependencies:** PLT-008.

#### PLT-006 — Packages

**Description:** Distribution MUST publish AppImage, `.deb`, and `.rpm`; an official PKGBUILD in AUR and `.pkg.tar.zst` for `pacman -U`; and NSIS `.exe` and MSI on Windows.

**Rationale:** Provides native installation on supported systems.

**Acceptance criteria:**

- AppImage remains the fallback for Arch.
- Artifacts have checksums.
- Packages install, launch, and uninstall without improperly removing user data.

**Dependencies:** PLT-005, PLT-009.

#### PLT-007 — Signed Updater

**Description:** The updater MUST verify the signature, notify the user of a new version, and request confirmation before installation.

**Rationale:** Silent automatic updating conflicts with explicit control.

**Acceptance criteria:**

- An invalid package is not installed.
- Release notes and version are displayed.
- Declining leaves the current version functional.

**Dependencies:** SEC-003, PLT-009.

#### PLT-008 — Test Strategy

**Description:** The product MUST include Rust unit and integration tests, redacted fixtures, Vitest, React Testing Library, IPC mocks, and WebdriverIO E2E on Linux and Windows.

**Rationale:** Risk spans parsing, persistence, IPC, UI, and packaging.

**Acceptance criteria:**

- Claude and Codex parsers have fixtures.
- Critical commands have authorization tests.
- Main journeys have E2E coverage proportional to the release scope.

**Dependencies:** PLT-005.

#### PLT-009 — CI/CD and Releases

**Description:** GitHub Actions MUST run linting, tests, and builds; `vX.Y.Z` tags MUST trigger tests, builds, signing, checksums, and a GitHub Release.

**Rationale:** Cross-platform releases need to be repeatable and auditable.

**Acceptance criteria:**

- A failed gate prevents publication.
- Artifacts are associated with the same version.
- Linux and Windows participate in the pipeline.

**Dependencies:** PLT-006, PLT-007, PLT-008.

#### PLT-010 — License and Attribution

**Description:** Converge MUST use the MIT license and preserve attribution for adapted MIT-licensed code, especially Codeburn.

**Rationale:** Enables open-source evolution and respects third-party obligations.

**Acceptance criteria:**

- `LICENSE` accompanies the repository and distributions.
- Notices identify relevant adaptations.
- No incompatible dependency is introduced without an explicit decision.

**Dependencies:** ANL-001.

#### PLT-011 — Accessibility

**Description:** The interface MUST target WCAG 2.2 AA, complete keyboard navigation, visible focus, and communication that does not rely on color alone.

**Rationale:** Intensive operation and critical states require consistent access.

**Acceptance criteria:**

- Main journeys can be completed with a keyboard.
- Quota states include text or an icon in addition to color.
- Interactive components expose accessible names and states.

**Dependencies:** PLT-004.

#### PLT-012 — Unknown-Format Compatibility

**Description:** Unknown provider content MUST be marked Unsupported, diagnosed locally, and isolated without interrupting all ingestion.

**Rationale:** Provider formats may change without notice.

**Acceptance criteria:**

- Recognizable events from the same file continue to be processed.
- Diagnostics do not include unauthorized sensitive content.
- Unsupported state is visible to the user.

**Dependencies:** ANL-001, SEC-008.

## 13. Persistence, Events, and Authoritative Sources

### 13.1 Authoritative Sources

- Project YAML and Markdown files are shareable authoritative sources.
- Spec Kit workflow YAML has precedence for the Agent Loop.
- `.converge/workflow.yml` is the explicit fallback.
- Event inference is the final fallback.
- SQLite stores internal state, the ledger, and projections; it does not replace authoritative files.

### 13.2 Event Model

Normalized events are immutable and MUST contain, when available:

- Internal ID.
- Provider and original ID.
- Project, Agent, and Session.
- Observed timestamp and provider timestamp.
- Normalized type.
- Model.
- Phase, step, turn, command, or tool call.
- Tokens and other metrics.
- Explicit or inferred source.
- Optional reference to the original payload.

Duplicates MUST be handled idempotently. Unknown content MUST NOT corrupt already recognized events.

### 13.3 Aggregates

Aggregates are disposable, reconstructable projections. A rebuild MUST:

- Preserve the ledger.
- Produce deterministic totals for the same rule version.
- Record the projection version.
- Avoid perceptibly blocking the interface.

In the MVP, performance will be monitored qualitatively and through observable regressions, without numeric budgets. Benchmarks may be added when a real baseline exists.

## 14. Non-Functional Requirements

### Security and Privacy

- Least privilege across React, Rust, filesystem, shell, and network.
- Provider credentials are read-only and held only in memory.
- Original payload is disabled by default.
- No telemetry by default.

### Reliability

- Atomic writes and backups for edited files.
- Immutable ledger and reconstructable aggregates.
- Watchers complemented by reconciliation.
- Backoff for remote endpoints.
- Failures isolated by adapter, session, and branch when possible.

### Accessibility

- WCAG 2.2 AA as the target.
- Main journeys operable by keyboard.
- Visible focus and states that do not rely on color alone.

### Compatibility

- Linux x86_64 and ARM64.
- Windows x86_64.
- Unknown provider changes degrade visibly and in isolation.

### Connectivity

- Running Claude and Codex requires internet access.
- History, workflows, files, and local aggregates remain available offline.
- Quotas, pricing, and remote updates use the last known value with a Stale state.

### Performance

The Fast principle remains qualitative at this stage. Quantitative targets will be defined after MVP selection and validated through benchmarks.

## 15. Dependencies and Integrations

| Dependency | Role | Behavior when absent |
| --- | --- | --- |
| Claude Code | Agent provider | Claude functionality is unavailable; other areas continue. |
| Codex CLI | Agent provider | Codex functionality is unavailable; other areas continue. |
| Internet | AI execution, quotas, pricing, and updates | Local data continues; remote actions are blocked or become Stale. |
| GitHub Spec Kit | Optional SDD lifecycle | Converge operates normally without the specific panels. |
| SQLite | Internal persistence | Failure prevents persistent operation and requires safe diagnostics/recovery. |
| Native keyring | Future secrets | The dependent capability remains unavailable; the secret does not move to SQLite. |

## 16. Success Criteria

### Functional Coverage

- The user can explicitly select and trust a folder.
- The user can understand consumption, quota, renewal, and estimated cost.
- The user can follow multiple agents and sessions in one interface.
- The user can start, resume, and stop managed sessions.
- The user can view Agent Loops and run workflows.
- The user can edit context files without silent overwrite.
- The user can use Spec Kit when present without making it mandatory.

### Reliability

- Unrecognized events do not interrupt all ingestion.
- Aggregates can be rebuilt.
- External file changes are reconciled.
- Failure of one session or adapter does not bring down independent executions.

### Security and Privacy

- The frontend has no generic shell or filesystem access.
- Projects begin restricted.
- Credentials are not persisted.
- Telemetry and original payload remain disabled by default.
- Diagnostics are redacted and exported manually.

### Perceived Value

- The user needs to switch less often among terminals, files, and provider tools to understand the state of work.
- Inferred, stale, or unavailable information is transparent.
- The interface provides enough detail without losing the consolidated view.

Quantitative product targets will not be invented at this stage. They may be defined after real usage and MVP selection.

## 17. Acceptance Criteria and Release Gates

A release may be approved only when:

- The release scope has completed acceptance criteria.
- Applicable unit, integration, frontend, and E2E tests pass.
- Linux and Windows pass the gates defined for the release.
- Migrations have been tested with representative databases and an appropriate backup.
- No known critical security issue exists without explicit mitigation.
- Capabilities, scopes, CSP, and allowlist have been reviewed.
- Installers and packages have been verified.
- Signatures and checksums have been generated and validated.
- Third-party attribution is current.
- Documentation of limitations and changes is published.

## 18. Risks and Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Change in Claude or Codex formats | Parsers stop recognizing events | Isolated adapters, versioned fixtures, Unsupported state, and safe partial ingestion. |
| Quota endpoint change or unavailability | Missing or incorrect quotas | Backoff, last-known value, Stale state, and clearly labeled local inference. |
| Inaccurate phase or consumption inference | Misleading analytics | Priority for explicit sources, Inferred label, and correction capability. |
| PTY differences between Linux and Windows | Inconsistent lifecycle or streaming | Rust abstraction, per-system E2E tests, and documented limitations. |
| Watcher misses events | Stale files or sessions | Reconciliation every minute and idempotent operations. |
| SQLite corruption | Loss of internal state | Tested migrations, backup strategy, immutable ledger, and aggregate rebuild. |
| File changed by an IDE or agent | Context overwrite | Conflict detection, Compare, Reload, confirmed Overwrite, and backup. |
| Credential or payload exposure | Privacy incident | Read-only memory, redaction, opt-in payloads, and no default telemetry. |
| Signing and packaging complexity | Delay or inconsistent release | Versioned CI/CD, platform gates, checksums, and installer validation. |
| Complete scope too large for the first delivery | Excessive time before real usage | Stage 3B selects the MVP before UI/UX and implementation. |

## 19. Future Roadmap

### Converge 2.0

- Configurable background and system-tray operation.
- Additional compiled adapters.
- Presets, bundles, and extensions.
- Traceability among requirements, artifacts, events, and execution.
- Convergence dashboard.
- Export to GitHub Issues.
- Bug workflow.

### Converge 3.0

- Dynamic plugin system.

## 20. Normative Appendix — MVP Selected in Stage 3B

### 20.1 Governance Rule

The MVP is a first-delivery subset. Deferred requirements remain part of the complete scope of this PRD. `MVP.md` is the operational document for the subset and references the stable IDs defined here; if the meaning of a requirement conflicts, this PRD takes precedence. If MVP inclusion, restriction, or implementation order conflicts, the approved classification in `MVP.md` and the Decision Log takes precedence.

### 20.2 Minimum Usable Journey

1. Start Converge with a single command.
2. Explicitly select a local folder.
3. Review and grant trust to the canonical path.
4. Detect Claude Code and Codex CLI without modifying their credentials.
5. Import the last 30 days first and process the remaining history in the background.
6. Inspect global quotas, consumption, averages, history, renewal, models, estimated API-equivalent cost, alerts, and freshness.
7. Create or select a named Agent and start a Claude or Codex Session with a task prompt sent once.
8. Follow the read-only Terminal View, Session state, and Agent Loop.
9. Stop the Session gracefully or confirm `Force Kill` after a timeout.
10. Inspect the recognized-file tree and, when `.specify/` exists, safely edit Markdown, YAML, and text files.
11. Reopen persisted data without a connection, except for capabilities that depend on providers.

### 20.3 MVP Architecture

- React, TypeScript, and Vite in the browser.
- Local Rust service restricted to `127.0.0.1`, with a random port and ephemeral token.
- HTTP for request/response and WebSocket for terminal and high-volume streams.
- Prefer frontend and API on the same origin; validate `Host` and `Origin`, with closed CORS.
- Clean Architecture within the modular monolith: Domain, Application, Adapters, and Interface.
- Domain and Application are independent of transport, framework, filesystem, and database.
- Web adapters in the MVP and later Tauri adapters over the same use cases.
- SQLite with SQLx, versioned migrations, immutable ledger, and reconstructable aggregates.
- `justfile` as the command façade, delegating to Cargo and pnpm.

### 20.4 Included Functional Scope

- **Projects & Trust:** selection, trust persisted by canonical identity, folder boundary, provider detection, and offline history. External symlinks are blocked.
- **Analytics:** Claude and Codex, tokens, averages, history, quotas, renewal, estimated cost, dimensions, models, alerts, and `Stale`. The last 30 days are ingested first; the remainder is backfilled. Original payload remains disabled.
- **Agents & Sessions:** named Agent without a fixed provider; provider and model belong to the Session. Includes managed sessions, read-only external detection, `Unassigned Sessions`, concurrency, `Start`, `Stop`, `Needs Attention`, and confirmed `Force Kill`.
- **Agent Loop:** read-only, standard phases, progressive disclosure, source, and confidence. The MVP order is `.converge/workflow.yml` followed by inference; Spec Kit sources are deferred.
- **Context Files:** tree for `CLAUDE.md`, `AGENTS.md`, `.claude/`, `.converge/`, and `.specify/`; Markdown/raw viewer. Safe editing is limited to existing Markdown, YAML, and text files inside `.specify/`.
- **Security & Privacy:** narrow boundaries, credentials only in memory, no telemetry, original payload disabled, and automatic security for the local service.
- **Platform & Quality:** Rust Core, React, HTTP/WebSocket, SQLite, white-box and black-box tests, complete CI on Linux, and continuous technical validation on Windows x86_64.

### 20.5 Explicitly Deferred Scope

- Tauri 2 desktop, Linux/Windows packaging, updater, and release signing.
- Linux ARM64 as an MVP distribution artifact.
- External-session resume and side-by-side layout.
- Continuous interactive input in the visual terminal.
- Independent, sequential, or collaborative workflows and their visual editor.
- Agent assignment to steps, Retry, Skip, and workflow controls.
- General editing of `CLAUDE.md`, `AGENTS.md`, `.claude/`, and `.converge/`.
- Spec Kit initialization, lifecycle board, artifacts, triggers, monitor, gates, quality panel, and SDD Analytics.
- Pricing, threshold, payload, and retention configuration.
- Authorization of external symlinks.

### 20.6 Implementation Slices

1. **Foundation:** Clean Architecture, modules, launcher, HTTP/WebSocket, local security, SQLite/SQLx, migrations, ledger, common client, `justfile`, and base CI.
2. **Project & Trust:** native picker, canonicalization, trust, scopes, symlinks, and provider detection.
3. **Analytics:** parsers, idempotent ingestion, backfill, aggregates, quotas, pricing, freshness, alerts, and interface.
4. **Agents & Sessions:** Agents, PTY, external sessions, read-only terminal, concurrency, and lifecycle.
5. **Agent Loop:** phase normalization, `.converge/workflow.yml`, inference, confidence, and progressive disclosure.
6. **Context Files:** tree, viewer, watcher, and safe editing limited to `.specify/`.
7. **Hardening & Acceptance:** white-box and black-box tests, security, rebuild, offline behavior, provider parity, accessibility, and documentation.

Every slice MUST end demonstrable, integrated into the accumulated journey, and covered by tests before the next slice.

### 20.7 Provider Parity

Claude and Codex MUST complete the same journey and expose the same primary capabilities. Provider-specific differences may appear in UI/UX, colors, and visual identity, provided they belong to the same design system, preserve accessibility, and do not change the meaning of states. Detailed visual direction belongs to Stage 4.

### 20.8 Success and Acceptance Criteria

- The minimum journey works in a real project with Claude and Codex.
- Both providers can run simultaneously.
- Repeated import is idempotent and does not duplicate events.
- Rebuilt aggregates produce equivalent results for the same rule version.
- Offline mode preserves history, aggregates, persisted Agent Loop, and available files.
- Sessions cover `Start`, streaming, `Needs Attention`, graceful termination, and confirmed `Force Kill`.
- Tests cover loopback, token, `Host`, `Origin`, CORS, path traversal, symlinks, credentials, and payloads.
- Applicable Rust, frontend, HTTP/WebSocket integration, and WebdriverIO tests pass.
- Complete CI passes on Linux; build and technical tests pass on Windows x86_64.
- No known failure allows silent data loss, credential exposure, or escape from the trusted boundary.
- `PRD.md`, `MVP.md`, Notion, and the Decision Log remain reconciled.

Performance remains a qualitative `Fast` requirement in the MVP. Arbitrary numeric budgets will not be used; regressions MUST be observable and protected by tests when applicable.

## 21. References

- Converge — Vision & Roadmap: https://app.notion.com/p/39f16378a61d81c280c3cbd24d51d2f1
- Converge — Decision Log: https://app.notion.com/p/39f16378a61d8168a5cdc35cf924439d
- Converge — Product Brainstorm: https://app.notion.com/p/39f16378a61d8178979ac12b7f36f0cc
- Converge — Spec Kit Integration: https://app.notion.com/p/39f16378a61d8193853dd580bebde0a4
- Converge — Requirements & Architecture: https://app.notion.com/p/39f16378a61d81e1a7d6fefe09d90cbf
- Converge — MVP Definition & Prioritization: https://app.notion.com/p/39f16378a61d81d29689f1eeb3320341
- Converge — Operational MVP: `MVP.md`
- Codeburn: https://github.com/getagentseal/codeburn
- GitHub Spec Kit: https://github.com/github/spec-kit
- Reference terminal: https://github.com/Felipebc2/SqlMap-Stsw
- Impeccable: https://impeccable.style/docs/

---

**Governance rule:** changes to scope, requirements, or priorities MUST be recorded in the Decision Log. Stages 3A and 3B are approved. Stage 4 may begin only with new explicit authorization.
