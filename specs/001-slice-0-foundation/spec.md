# Feature Specification: Slice 0 — Foundation

**Feature Branch**: `001-slice-0-foundation`

**Created**: 2026-07-22

**Status**: Approved — planning artifacts (`plan.md` and Phase 0/1 outputs)
have been generated against this specification and two revision passes have
treated it as the authoritative source of truth when resolving conflicts
(2026-07-22), which presupposes human approval of its scope and requirements
per the Spec Kit Policy's "do not implement until scope and required
artifacts are approved" gate. Synchronized this pass (correction 8) — the
quality checklist (`checklists/requirements.md`) has been fully satisfied
since its creation and no unresolved `[NEEDS CLARIFICATION]` marker has
existed at any point since.

**Input**: User description: "Create the specification for Converge Slice 0 —
Foundation. Establish the smallest runnable, secure, testable foundation
required by every later MVP slice: one-command launch of the minimal frontend
and authenticated local service; a minimal authenticated health path; loopback
binding, random port, ephemeral token, Host/Origin validation, and closed CORS;
mechanically verifiable Clean Architecture boundaries; idempotent persistence
and rebuildable aggregates for a representative normalized event; formatting,
linting, tests, and builds through the repository command facade and base CI;
and visible failures that never silently weaken security or data integrity."

## User Scenarios & Testing *(mandatory)*

Slice 0 has no end-user-facing product journey yet — `docs/MVP.md` §8 defines
its outcome as "a runnable, secure skeleton that proves the architectural
seams." Its primary actor is the **Contributor** (a developer building or
reviewing later MVP slices) and, transitively, **CI**, which must reproduce
the same guarantees the Contributor observes locally. Each story below is a
standalone, independently demonstrable proof of one architectural seam; later
slices depend on all of them but do not need to exist for these to be
verified today.

### User Story 1 - One-Command Launch to an Authenticated Health Check (Priority: P1)

A Contributor checks out the repository and runs the one documented command.
The minimal frontend and the local Rust service start together; the service
binds only to loopback on a random port, issues an ephemeral token for that
run, and responds to a minimal health path only when the request carries a
valid token and matching `Host`/`Origin`. Requests missing or misusing any of
those are denied, not ignored.

**Why this priority**: Every later slice assumes a running, authenticated
local service exists. If this does not work, nothing downstream can be
demonstrated or tested. This is the load-bearing story of the slice.

**Independent Test**: From a clean checkout, run the documented command and
issue an HTTP request to the health path — first with a valid token and
matching `Host`/`Origin` (expect success), then with each individually
invalidated (expect denial). Fully testable without any other slice existing.

**Acceptance Scenarios**:

1. **Given** a clean checkout, **When** the Contributor runs the documented
   repository command, **Then** the frontend and the local service both start
   and the service is listening only on `127.0.0.1` on a port chosen at
   random for that run.
2. **Given** the service is running, **When** a request to the health path
   carries the current ephemeral token and a matching `Host`/`Origin`,
   **Then** the request succeeds and reports the service as healthy.
3. **Given** the service is running, **When** a request omits the token, uses
   a stale or wrong token, or carries a mismatched `Host`, `Origin`, or a
   disallowed CORS origin, **Then** the request is denied and the denial is
   observable (not silently dropped).
4. **Given** the service has already been started once, **When** it is
   restarted, **Then** it binds to a newly chosen random port and issues a
   new ephemeral token; the previous token no longer authenticates.

---

### User Story 2 - Mechanically Verified Clean Architecture Boundaries (Priority: P2)

A Contributor (or reviewer) runs a repeatable check that proves Domain and
Application code have no dependency on transport, UI, database, filesystem,
PTY, Tauri, or provider-specific code, and that inbound/outbound adapters
depend inward rather than the reverse.

**Why this priority**: The modular monolith's long-term maintainability
depends on this boundary holding from the very first slice; catching a
violation mechanically, rather than through code review alone, is cheaper the
earlier it exists. It does not block Story 1 from being demonstrated but is
equally foundational for every later slice.

**Independent Test**: Run the boundary check (compilation/test scoping or an
equivalent automated dependency check) against Domain and Application code in
isolation, and confirm it fails when an inward-pointing dependency is
deliberately introduced as a negative test.

**Acceptance Scenarios**:

1. **Given** the current workspace, **When** Domain and Application crates are
   built and tested in isolation, **Then** they compile and pass without
   pulling in HTTP, WebSocket, SQLx, filesystem, PTY, Tauri, or provider
   crates.
2. **Given** a deliberately introduced outward-pointing dependency (e.g.,
   Domain importing an adapter type), **When** the boundary check runs,
   **Then** it fails visibly instead of passing silently.

---

### User Story 3 - Idempotent Event Persistence and Aggregate Rebuild (Priority: P3)

A Contributor persists one representative normalized event through the
ledger, confirms that re-submitting the same event does not create a
duplicate, discards the derived aggregate, and rebuilds it from the ledger
alone, obtaining an equivalent result.

**Why this priority**: Every later slice's analytics, session, and loop data
depends on the ledger being append-only, idempotent, and the source of truth
for rebuildable aggregates. Proving this mechanism with one representative
event now removes the riskiest architectural assumption before real provider
ingestion (Slice 2) is built on top of it.

**Independent Test**: Submit a representative normalized event twice, confirm
one ledger row and one equivalent aggregate exist; then delete the aggregate
table's derived state only, rebuild it from the ledger, and confirm the
rebuilt aggregate is equivalent to the original.

**Acceptance Scenarios**:

1. **Given** an empty database with migrations applied, **When** a
   representative normalized event is persisted, **Then** exactly one
   immutable ledger row exists and a corresponding aggregate is derived from
   it.
2. **Given** that same event is submitted again unchanged, **When** it is
   persisted, **Then** no duplicate ledger row is created and the aggregate
   is unchanged.
3. **Given** the derived aggregate is discarded, **When** it is rebuilt from
   the immutable ledger, **Then** the rebuilt aggregate is equivalent to the
   original and the ledger itself is unmodified by the rebuild.

---

### User Story 4 - Command-Facade Quality Gates and Base CI (Priority: P4)

A Contributor runs formatting, linting, tests, and builds through the
repository's command facade locally, and the same gates run in base CI on
every change, on Linux fully and on Windows x86_64 for build and applicable
technical tests.

**Why this priority**: This gives every later slice a working, trustworthy
verification loop from day one. It is ordered last only because it verifies
the outcomes of Stories 1–3 rather than introducing new runtime behavior of
its own.

**Independent Test**: Run the documented format/lint/test/build recipes
locally and confirm they pass; confirm the same recipes run in the CI
workflow and that a deliberately broken check (e.g., a failing test) causes
CI to fail visibly rather than passing.

**Acceptance Scenarios**:

1. **Given** the repository at HEAD, **When** the Contributor runs the
   documented format, lint, test, and build recipes, **Then** each completes
   and reports pass/fail without requiring an undocumented command.
2. **Given** a pull request or a push to the default branch, **When** base
   CI runs, **Then** the complete gate (format, lint, test, build) passes on
   Linux and the build plus applicable technical tests pass on Windows
   x86_64.
3. **Given** a deliberately failing check (e.g., a broken test) introduced as
   a negative test, **When** CI runs, **Then** CI reports failure and does
   not mark the change as passing.

---

### User Story 5 - Minimal Authenticated WebSocket Proof (Priority: P2)

A Contributor opens a WebSocket connection to the local service using the
current ephemeral token and a matching `Host`, and confirms the connection is
accepted only when the token is valid, `Host` matches, and `Origin` is
present in an explicit server-side allowlist checked before any
application-level exchange happens. CORS is an HTTP-only mechanism — browsers
do not apply CORS preflight or CORS headers to WebSocket handshakes — so this
proof does not claim WebSocket enforces CORS; it enforces the token, `Host`,
and an explicit Origin allowlist as WebSocket's equivalent boundary. This
proof carries no terminal or provider streaming payload — it exists solely to
establish that the WebSocket boundary `docs/MVP.md` §4.1 assigns to this
slice is bound by the same authentication strength as the HTTP boundary,
through the mechanisms that actually apply to WebSocket.

**Why this priority**: `docs/MVP.md` §4.1 assigns both HTTP and WebSocket
transport to the runtime from the outset. Leaving WebSocket authentication
unproven here would let a real security gap survive into Slice 3, where
WebSocket carries live terminal streams. Proving it now, while the payload is
trivial, isolates the transport-security concern from the streaming behavior
introduced later.

**Independent Test**: Attempt a minimal WebSocket handshake with a valid
token, matching `Host`, and an allowlisted `Origin` (expect accept), then
repeat with each invalidated individually (expect refusal before any message
is exchanged). Fully testable without terminal or provider code existing.

**Acceptance Scenarios**:

1. **Given** the service is running, **When** a WebSocket handshake carries
   the current ephemeral token, a matching `Host`, and an `Origin` present in
   the explicit allowlist, **Then** the connection is accepted.
2. **Given** the service is running, **When** a WebSocket handshake omits the
   token, uses a stale or wrong token, carries a mismatched `Host`, or
   carries an `Origin` absent from the allowlist, **Then** the handshake is
   refused, the refusal is observable, and no application-level exchange
   occurs.
3. **Given** an accepted WebSocket connection carrying only a minimal proof
   payload (e.g., an echo or ping), **When** the exchange completes, **Then**
   no terminal, PTY, or provider streaming behavior is present — the
   connection demonstrates transport-level authentication only.

---

### Edge Cases

- What happens when a request reaches the health path with a valid token but
  a mismatched `Origin` (or vice versa)? The request MUST still be denied —
  every one of token, `Host`, `Origin`, and CORS must independently hold.
- What happens when a WebSocket handshake presents a valid token and matching
  `Host` but an `Origin` absent from the explicit allowlist (or vice versa)?
  It MUST be refused before any message is exchanged. CORS headers do not
  apply to the WebSocket handshake, so this enforcement is via the explicit
  Origin allowlist (FR-022), not via the CORS mechanism required for HTTP
  (FR-005).
- What happens when a Rust contract type changes but the generated
  TypeScript is not regenerated? The drift check MUST fail visibly; a stale
  generated type MUST NOT ship silently.
- What happens when the chosen random port is already in use? The service
  MUST select another port or fail with a visible, actionable error; it MUST
  NOT silently fall back to a fixed or predictable port.
- What happens when the same normalized event is submitted concurrently from
  two requests (a race, not just a sequential retry)? Exactly one ledger row
  MUST result; the idempotency guarantee MUST hold under concurrency, not
  only under sequential retries.
- What happens when an aggregate rebuild runs against a ledger containing an
  event the current rebuild logic does not yet recognize? The rebuild MUST
  fail visibly or skip the unrecognized event in a documented, non-silent
  way — it MUST NOT produce a corrupted or partially-updated aggregate.
- What happens when a forward-only migration is run twice, or run against a
  database already at a newer version? It MUST be a no-op or MUST fail
  visibly; it MUST NOT corrupt existing data.
- What happens when the frontend attempts an operation outside its typed
  HTTP/WebSocket client (e.g., a hypothetical generic shell or filesystem
  call)? No such capability MUST exist to attempt in the first place.
- What happens when a contributor runs the quality gates on Windows x86_64
  where a Linux-only tool is unavailable? The command facade MUST document
  which recipes are Linux-only versus cross-platform, and CI's Windows path
  MUST only require what is documented as supported there.

## Out of Scope

The following remain outside Slice 0 and are governed by later slices or
remain deferred per `docs/MVP.md`:

- Project selection and trust workflows (Slice 1).
- Provider (Claude/Codex) analytics ingestion (Slice 2).
- Managed Agent sessions and PTY execution (Slice 3).
- Agent Loop visualization (Slice 4).
- Context-file browsing or editing (Slice 5).
- Spec Kit product integration (`SPEC-002` through `SPEC-006`, deferred).
- Tauri 2 desktop packaging and Tauri Commands/IPC (`PLT-001`, `PLT-003`
  remain restricted to React/Vite plus HTTP/WebSocket for the MVP).
- Installers, updater, signing, checksums, or release publication
  (`PLT-006`, `PLT-007`, and the tagged-release portion of `PLT-009`).
- Terminal and provider WebSocket streaming payloads (Slice 3). Slice 0's
  WebSocket proof (User Story 5) is transport/security-only.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The repository MUST provide one documented command that
  launches the minimal frontend and the local Rust service together.
  *(PLT-001 restricted, PLT-004; AC-040)*
- **FR-002**: The local service MUST bind exclusively to the loopback
  interface on a port chosen at random for each run. *(SEC-003; AC-033)*
- **FR-003**: The local service MUST require a valid per-run ephemeral
  authentication token on every request to a protected path, including the
  health path. *(SEC-001; AC-033)*
- **FR-004**: The local service MUST validate the `Host` and `Origin` headers
  on every request and MUST reject requests carrying unrecognized values.
  *(SEC-003; AC-034)*
- **FR-005**: The local service MUST enforce closed CORS on HTTP requests,
  allowing only the expected local frontend origin. CORS is an HTTP-only
  mechanism; WebSocket's equivalent boundary is the explicit Origin
  allowlist required by FR-022, not CORS itself. *(SEC-003; AC-034)*
- **FR-006**: The local service MUST deny, individually and in combination,
  any invalid token, `Host`, `Origin`, or CORS presentation, and each denial
  MUST be observable rather than silent. *(AC-034, AC-039)*
- **FR-007**: The local service MUST expose a minimal authenticated health
  path reporting service readiness. *(AC-033; MVP §8 exit gate)*
- **FR-008**: The frontend MUST reach all backend behavior exclusively
  through a typed HTTP/WebSocket client; it MUST NOT be granted generic
  shell or unrestricted filesystem access. *(SEC-001; AC-035)*
- **FR-009**: Domain and Application code MUST compile and pass their test
  suites without depending on transport, UI, database, filesystem, PTY,
  Tauri, or provider-specific code. *(Constitution II; AC-041)*
- **FR-010**: The dependency direction between Domain, Application, inbound
  adapters (HTTP/WebSocket), outbound adapters (SQLx), and the Interface
  layer MUST be checkable by a repeatable, automatable procedure that fails
  when the inward-dependency rule is violated. *(Constitution II; AC-041)*
- **FR-011**: Persistence MUST use SQLite through SQLx with versioned,
  forward-only migrations, tested starting from an empty database. *(PLT-002;
  AC-042)*
- **FR-012**: The system MUST persist a representative normalized event
  idempotently: resubmitting the same event MUST NOT create a duplicate
  ledger entry, including under concurrent resubmission. *(Constitution V;
  MVP §8 exit gate)*
- **FR-013**: Aggregates derived from the ledger MUST be discardable and
  rebuildable from the immutable ledger alone, producing an equivalent
  result without mutating the ledger. *(Constitution V; PLT-002; AC-042; MVP
  §8 exit gate)*
- **FR-014**: The repository MUST provide a command facade (`justfile`)
  exposing formatting, linting, test, and build recipes, usable identically
  by contributors and by CI. *(Constitution IX; AC-040, AC-044)*
- **FR-015**: Base CI MUST run the complete format/lint/test/build gate on
  Linux for every pull request and every push to the default branch, and
  MUST run the build plus applicable technical tests on Windows x86_64 under
  the same triggers. *(PLT-005 restricted, PLT-009 restricted; AC-044)*
- **FR-016**: The change MUST include risk-appropriate automated tests
  covering every layer this slice introduces: the Clean Architecture
  boundary check (FR-010), generated-contract drift (FR-020), HTTP and
  WebSocket contract/integration behavior (FR-021, FR-022), the HTTP
  authentication/denial paths (token, `Host`, `Origin`, CORS) and the
  WebSocket authentication/denial paths (token, `Host`, Origin allowlist),
  SQLx migrations, idempotent ingestion, aggregate rebuild, the frontend's
  observable behavior (including its TanStack Query/Zustand
  separation and accessibility requirements), and the minimal browser
  end-to-end journey covering the authenticated health check. *(Constitution
  VI; AC-043)*
- **FR-017**: Any logging introduced in this slice MUST redact the ephemeral
  token and any other sensitive value, and MUST NOT transmit anything
  automatically. *(Constitution VIII; SEC-008 restricted; AC-038)*
- **FR-018**: A failure in authentication denial, migration, aggregate
  rebuild, or a CI gate MUST be visible to the Contributor or to CI and MUST
  NOT silently weaken security or data-integrity guarantees. *(AC-039)*
- **FR-019**: Rust types MUST be the authoritative source for the HTTP health
  contract and the minimal WebSocket contract introduced in this slice; the
  corresponding TypeScript types MUST be generated from them deterministically.
  *(Constitution IV; AC-043)*
- **FR-020**: An automated check MUST fail when generated TypeScript
  contracts drift from their Rust source (e.g., a regenerate-and-diff check
  or equivalent). *(Constitution IV; AC-043)*
- **FR-021**: The HTTP health contract and the minimal WebSocket contract
  introduced in this slice MUST each have explicit automated tests covering
  their request/response, or handshake/message, shape. *(Constitution IV;
  AC-043)*
- **FR-022**: The local service's WebSocket endpoint MUST enforce loopback
  binding, the ephemeral token, `Host` validation, and an explicit
  server-side `Origin` allowlist before accepting any connection. CORS does
  not apply to the WebSocket handshake and MUST NOT be relied upon as its
  Origin enforcement mechanism; the explicit allowlist is required instead.
  The endpoint MUST NOT carry terminal or provider streaming payloads in
  this slice. *(SEC-001, SEC-003; AC-033, AC-034)*
- **FR-023**: Any frontend surface introduced in this slice (e.g., a
  health/status indicator) MUST comply now — not deferred to a later slice —
  with `docs/DESIGN.md`, the Graphite Signal system, light and dark themes,
  keyboard operation, visible focus, non-color state communication, reduced
  motion, and WCAG 2.2 AA, across every applicable state defined in the
  Frontend State Matrix below. *(Constitution VII; PLT-011)*
- **FR-024**: If frontend health/status data originates from the Rust
  service, TanStack Query MUST manage and cache it; Zustand MUST NOT become
  an authoritative source for that data. *(PLT-004; Constitution II)*

### Key Entities *(include if feature involves data)*

- **Normalized Event**: An immutable ledger entry representing one
  occurrence, identified so that resubmission is detectable and does not
  duplicate the ledger. Slice 0 uses one representative shape only to prove
  the mechanism; real provider event shapes arrive in Slice 2.
- **Aggregate**: Derived, reconstructable state computed from the ordered
  set of Normalized Events. Discarding and recomputing an Aggregate MUST
  reproduce an equivalent result and MUST NOT alter the underlying events.
- **Ephemeral Session Token**: A credential generated fresh for each service
  run, required on every authenticated request, and invalidated when the
  service restarts. It is never persisted to SQLite or written to logs.
- **Migration Version**: A versioned, forward-only schema step applied to
  an empty or previously-migrated database, tracked so that the applied
  version is known and re-application is safe.

### Frontend State Matrix — Health/Status Surface

FR-023 requires the health/status surface introduced by this slice to comply
now with `docs/DESIGN.md`, the Graphite Signal system, both themes, keyboard
operation, visible focus, non-color state communication, reduced motion, and
WCAG 2.2 AA. This matrix defines the complete set of states applicable to
that surface; every applicable state below MUST satisfy those same
requirements wherever it is shown — a visible focus indicator when
focusable, a non-color signal (text or icon) alongside any color, no
reliance on motion alone, and correct rendering in both light and dark
themes.

| State | Applicable | Description |
| --- | --- | --- |
| Initial loading | Yes | Shown between the frontend mounting and the first health response completing; communicated with text or an icon, not a color-only spinner. |
| Ready (healthy) | Yes | The authenticated health path has responded successfully; the current status is displayed. |
| Refreshing / Stale | Yes | A previously successful status remains visible while a new check is in flight, or while the last known status is older than the expected refresh interval; distinguished from Ready by explicit text or an icon, never by color alone. |
| Service unavailable / Offline | Yes | The health request cannot reach the service (connection refused, timeout) or is denied authentication; the surface states this plainly instead of continuing to show a stale Ready state. |
| Error | Yes | The service responded but reported an unhealthy or unexpected result; distinguished from Service unavailable, which is a transport or authentication failure rather than a reported unhealthy state. |
| Empty | Not applicable | The surface has exactly one subject — the local service's own health — so there is no listable or collection content that could be empty. Intentionally omitted here; a later slice introducing a collection view (e.g., analytics, sessions) MUST define its own Empty state. |
| Disabled | Not applicable | The surface is a read-only status display with no interactive control (no button, toggle, or input) that could be enabled or disabled. Intentionally omitted here; slices that introduce interactive controls (e.g., Session Start/Stop in Slice 3) MUST define their own Disabled state. |

### Constitution Compliance *(mandatory)*

- **I. Versioned Sources of Truth**: This specification is scoped to Slice 0
  as authorized by `docs/MVP.md` §8 and does not introduce requirements
  outside `docs/PRD.md`'s stable IDs; it treats `docs/PRD.md`, `docs/MVP.md`,
  and this Constitution as authoritative and defers to them on conflict.
- **II. Modular Clean Architecture**: FR-009 and FR-010 make the inward
  dependency rule an explicit, mechanically checked requirement of this
  slice rather than a convention enforced only by review.
- **III. Secure Local Runtime**: FR-002 through FR-008 and FR-022 establish
  deny-by-default local service access — loopback, random port, ephemeral
  token, closed CORS and `Host`/`Origin` validation for HTTP, an explicit
  `Host`/Origin-allowlist boundary for WebSocket, and no generic
  shell/filesystem grant — before any project-trust or provider capability
  exists.
- **IV. Contracts as Code**: FR-019 through FR-021 make Rust the authoritative
  source for the HTTP health contract and the minimal WebSocket contract
  introduced in this slice, require deterministic TypeScript generation,
  require an automated check that fails on generated-contract drift, and
  require explicit tests for both the HTTP and WebSocket contract shapes.
- **V. Durable and Reconstructable Data**: FR-011 through FR-013 require
  forward-only tested migrations, an immutable ledger, and rebuildable
  aggregates as first-class, testable behavior of this slice.
- **VI. Risk-Oriented Test-First Development**: FR-016 requires
  Red-Green-Refactor-appropriate coverage of the security boundary and the
  persistence boundary, the two domain/security-adjacent risk areas this
  slice introduces.
- **VII. Product and UX Integrity**: FR-023 requires that any frontend
  surface introduced in this slice (e.g., a health/status indicator) comply
  now with `docs/DESIGN.md`, the Graphite Signal system, both themes,
  keyboard operation, visible focus, non-color state communication, reduced
  motion, and WCAG 2.2 AA — compliance is not deferred for UI this slice
  actually introduces. Only the full multi-screen, main-journey WCAG 2.2 AA
  acceptance audit (`AC-045`) continues to accumulate through Slice 6, per
  `docs/MVP.md` §8, as later journeys are added on top of this one.
- **VIII. Privacy and Observability**: FR-017 requires redaction of the
  ephemeral token and any sensitive value in any logging introduced, and
  confirms no telemetry or automatic transmission exists in this slice.
- **IX. Documentation and Delivery Discipline**: FR-014 and FR-015 require
  the `justfile` command facade and base CI to exist and to run the same
  gates for contributors and CI alike, in the same change as this slice's
  behavior.
- **X. Human-Controlled Version Control**: This specification does not
  authorize any agent to stage, commit, push, merge, tag, or open a pull
  request; delivery of this slice concludes with suggested Git commands for
  the human maintainer, per Constitution Principle X.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A Contributor can go from a clean checkout to a successful,
  authenticated health-check response using exactly one documented command,
  with zero undocumented manual steps.
- **SC-002**: In automated verification, 100% of requests presenting an
  invalid or mismatched token, `Host`, `Origin`, or CORS origin are denied,
  and 100% of correctly authenticated requests succeed.
- **SC-003**: Submitting the same representative event twice — sequentially
  or concurrently — results in exactly one ledger record and one equivalent
  aggregate, verified across all tested repetitions.
- **SC-004**: Discarding and rebuilding the aggregate from the ledger
  reproduces an equivalent result in 100% of tested rebuild scenarios,
  without altering the underlying ledger.
- **SC-005**: An automated boundary check confirms zero references from
  Domain or Application code to transport, UI, database, filesystem, PTY,
  Tauri, or provider-specific packages.
- **SC-006**: The complete format/lint/test/build gate passes on Linux, and
  the build plus applicable technical tests pass on Windows x86_64, for
  every pull request and every push to the default branch.
- **SC-007**: Every deliberately introduced negative-path failure (denied
  auth, boundary violation, broken migration, broken test) is reported as a
  visible failure in 100% of verification runs — none passes silently.
- **SC-008**: Regenerating TypeScript contracts from their Rust source
  produces no diff from the committed generated output, verified by an
  automated check that fails whenever drift is deliberately introduced.
- **SC-009**: Every WebSocket handshake attempt lacking a valid token, a
  matching `Host`, or an `Origin` present in the explicit allowlist is
  refused before any application-level message is exchanged, in 100% of
  tested attempts. CORS is not part of this measurement — it does not apply
  to WebSocket.
- **SC-010**: The frontend surface introduced in this slice passes the
  applicable WCAG 2.2 AA checks (keyboard operation, visible focus,
  non-color state communication, reduced motion) with zero known
  violations.
- **SC-011**: The frontend's displayed health/status data observably updates
  from the local service's latest response after a query invalidation or
  refetch, confirming TanStack Query — not client-only Zustand state — is
  its source of truth.

## Assumptions

- "Representative normalized event" means one schema-conformant sample event
  used to prove the ledger/idempotency/rebuild mechanism; it is not real
  Claude/Codex ingestion, which remains Slice 2 scope.
- Any frontend surface in this slice is minimal (e.g., a service/health
  status indicator) and is not the application's full navigational shell;
  broader screens and journeys are introduced by later slices. This minimal
  surface remains fully subject to FR-023's design-system and accessibility
  requirements now — only the multi-screen navigational journey is
  deferred, not compliance for what this slice actually ships.
- `PLT-001` and `PLT-003` remain restricted for the whole MVP per
  `docs/MVP.md` §6.4: HTTP/WebSocket serve as the interim transport in place
  of Tauri Commands/IPC, so this slice's "typed common client" is an
  HTTP/WebSocket client, not a Tauri Command boundary.
- Browser E2E coverage in `AC-043` is satisfied at the level applicable to
  this slice (the authenticated HTTP health check and the minimal WebSocket
  proof); a full end-to-end journey accumulates starting with Slice 1,
  consistent with `docs/MVP.md` §10's "every slice MUST end demonstrable,
  integrated into the accumulated journey" rule.
- Windows x86_64 involvement in this slice is limited to build success and
  applicable technical (non-E2E) tests, consistent with the `PLT-005`
  restriction recorded in `docs/MVP.md` §6.4.
- `SEC-002` (capabilities and scopes specific to the active project) is not
  a Slice 0 requirement because no project or trust concept exists yet; it
  becomes applicable starting with Slice 1 and is intentionally out of scope
  here, per the Out of Scope section above.
- Provider-credential handling (`SEC-004`, `AC-036`, `AC-037`) is vacuously
  satisfied in this slice because no credential or provider-payload handling
  exists yet; this assumption MUST NOT be read as license to introduce such
  handling later without revisiting those requirements directly.
- How the ephemeral token is transported (e.g., header vs. query parameter)
  and how the frontend discovers the run's chosen port and current token are
  security-sensitive implementation decisions left to `plan.md`. This spec
  requires only the outcomes: a valid token, matching `Host`, and (for HTTP)
  matching `Origin`/CORS or (for WebSocket) an allowlisted `Origin` — per
  FR-002 through FR-006 and FR-022 — regardless of the transport chosen. No
  clarification surfaced a user-visible behavior that would require fixing
  this at the specification level.
