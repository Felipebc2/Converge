---

description: "Task list for Slice 0 — Foundation implementation"
---

# Tasks: Slice 0 — Foundation

**Input**: Design documents from `specs/001-slice-0-foundation/`

**Prerequisites**: `plan.md` (required — Approved), `spec.md` (required — Approved), `research.md`, `data-model.md`, `contracts/http-health.md`, `contracts/websocket-proof.md`, `traceability.md`, `.specify/memory/constitution.md` (v1.0.1)

**Tests**: Mandatory per Constitution Principle VI, scoped by risk. Red-Green-Refactor (test written and failing before implementation) is mandatory for domain logic, application use cases, contracts, and security boundaries — every test task below marked against those layers MUST be completed, and MUST fail, before its paired implementation task. UI, configuration, infrastructure, and adapter tests are included in the same change without requiring literal test-first ordering.

**Organization**: Tasks are grouped by user story (spec.md priorities P1–P4, with US2 and US5 tied at P2) to enable independent implementation and testing of each story, per spec.md's explicit "Independent Test" for each.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no unmet dependency)
- **[Story]**: US1–US5, mapping to spec.md's five user stories. Setup, Foundational, and Polish tasks carry no story label.
- Every task states its exact file path(s) and, where applicable, the FR/SC/AC identifier(s) it satisfies (`traceability.md`).

## Repository State at Start

This slice starts from an empty implementation surface: no `crates/`, `apps/web/`, `packages/`, `migrations/`, `scripts/`, `tests/`, `justfile`, or `.github/workflows/` exist yet in the repository. Every task below creates new files; none modifies pre-existing implementation code.

## Path Conventions (from `plan.md` Project Structure)

- Rust crates: `crates/domain/`, `crates/application/`, `crates/contracts/`, `crates/adapters-http/`, `crates/adapters-persistence/`, `crates/service/`
- Frontend: `apps/web/src/features/health/`, `apps/web/src/lib/api/`, `apps/web/src/state/`
- Generated TypeScript contracts: `packages/contracts/src/generated/`
- Migrations: `migrations/`
- Dev-launch supervisor: `scripts/dev.mjs`
- Tests: `tests/architecture/`, `tests/contract/`, `tests/integration/`, `tests/e2e/`
- CI: `.github/workflows/ci.yml`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Repository/tooling scaffolding with no story-specific behavior.

- [ ] T001 Create top-level directory structure (`crates/`, `apps/web/`, `packages/contracts/`, `migrations/`, `scripts/`, `tests/architecture/`, `tests/contract/`, `tests/integration/`, `tests/e2e/`) per `plan.md` Project Structure
- [ ] T002 [P] Create root Cargo workspace manifest declaring all six crates as workspace members in `Cargo.toml`
- [ ] T003 [P] Pin Rust toolchain (`channel = "1.97.1"`, `components = ["rustfmt", "clippy"]`) in `rust-toolchain.toml`
- [ ] T004 [P] Initialize pnpm workspace: `pnpm-workspace.yaml`, root `package.json` (`"packageManager": "pnpm@11.15.1"`, `engines.node`), `.nvmrc` (`24.18.0`)
- [ ] T005 [P] Scaffold `apps/web` with Vite + React + TypeScript per `research.md` pins (`react`/`react-dom` 19.2.8, `vite` 8.1.5, `@vitejs/plugin-react` 6.0.4, `@types/react` 19.2.17, `@types/react-dom` 19.2.3, `typescript` 6.0.3) — `apps/web/package.json`, `apps/web/tsconfig.json`, `apps/web/index.html`, `apps/web/src/main.tsx`
- [ ] T006 [P] Scaffold `packages/contracts` as a pnpm workspace package that `apps/web` depends on as `"@converge/contracts": "workspace:*"` — `packages/contracts/package.json`, `packages/contracts/src/generated/.gitkeep`
- [ ] T007 [P] Configure ESLint flat config + Prettier per `research.md` pins (`eslint` 10.7.0, `eslint-config-prettier` 10.1.8, `typescript-eslint` 8.65.0, `eslint-plugin-react-hooks` 7.1.1, `prettier` 3.9.6) — `eslint.config.js`, `.prettierrc`
- [ ] T008 Create `justfile` with every recipe name from `plan.md`'s Command Facade table (`setup`, `dev`, `fmt`, `fmt-check`, `lint`, `test-unit`, `test-contract`, `test-integration`, `test-frontend`, `test-e2e`, `build`, `check`, `contracts-generate`, `contracts-check`), each wired to the underlying tool it invokes
- [ ] T009 Verify Constitution compliance (Principles I–X) for this task breakdown against `specs/001-slice-0-foundation/plan.md`'s Constitution Check — Post-Design Gate before implementation begins; record any deviation here before proceeding

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Crate-level scaffolding every user story depends on.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T010 [P] Configure `crates/domain/Cargo.toml` — normal-dependency allowlist `{thiserror}` only; no serde, I/O, framework, or async
- [ ] T011 [P] Configure `crates/application/Cargo.toml` — normal-dependency allowlist `{domain (path), thiserror, uuid (default features only), blake3, serde_json, time (feature "parsing")}`; `tokio` only as a dev-dependency
- [ ] T012 [P] Configure `crates/contracts/Cargo.toml` — normal-dependency allowlist `{serde (feature "derive"), ts-rs =12.0.1}`; no dependency on `domain`/`application`
- [ ] T013 [P] Configure `crates/adapters-http/Cargo.toml` — allowlist `{application (path), domain (path), contracts (path), axum =0.8.9 (feature "ws"), tower-http =0.7.0 (features "cors","trace"), tokio =1.53.1, thiserror, serde_json, time =0.3.54 (features "formatting","parsing","serde")}`
- [ ] T014 [P] Configure `crates/adapters-persistence/Cargo.toml` — allowlist `{application (path), domain (path), sqlx =0.9.0 (features "sqlite","runtime-tokio","macros","migrate"), tokio, thiserror}`; no `serde_json`
- [ ] T015 Configure `crates/service/Cargo.toml` — composition root, depends on all five other crates plus `rand =0.10.2` and `base64 =0.22.1`; no allowlist restriction (it is the only crate exempt from the per-crate allowlist check, per `plan.md` Project Structure)
- [ ] T016 Add `#![forbid(unsafe_code)]` at the crate root (`src/lib.rs` or `src/main.rs`) of all six crates, `service` included with no exception, in `crates/domain/src/lib.rs`, `crates/application/src/lib.rs`, `crates/contracts/src/lib.rs`, `crates/adapters-http/src/lib.rs`, `crates/adapters-persistence/src/lib.rs`, `crates/service/src/main.rs`
- [ ] T017 [P] Add `reqwest =0.13.4` and `tokio-tungstenite =0.30.0` as dev-dependencies for HTTP/WS integration testing in the workspace `Cargo.toml` `[dev-dependencies]` (or `tests/integration`'s own `Cargo.toml`)

**Checkpoint**: All six crates compile as empty skeletons with correct allowlists and `#![forbid(unsafe_code)]`. User story implementation can now begin.

---

## Phase 3: User Story 1 - One-Command Launch to an Authenticated Health Check (Priority: P1) 🎯 MVP

**Goal**: `just dev` starts the frontend and the local Rust service together; the service binds only to loopback on a random port, issues an ephemeral token, and serves an authenticated health path enforcing token/`Host`/`Origin`/CORS.

**Independent Test**: From a clean checkout, run `just dev`, then request the health path with a valid token/`Host`/`Origin` (expect success) and with each individually invalidated (expect denial), per spec.md's Acceptance Scenarios 1–4.

### Tests for User Story 1 (MANDATORY — Constitution Principle VI, security boundary) ⚠️

- [ ] T018 [P] [US1] Contract test: `HealthResponse`/`HealthStatus`/`ApiError` wire shapes with exact `camelCase` field names in `tests/contract/http_health_contract.rs` (FR-021, AC-043)
- [ ] T019 [P] [US1] Integration test: HTTP denial paths — `Host` mismatch (400), `Origin` absent/mismatched (403), missing/wrong token (401), individually and in combination, first-failure-wins order — in `tests/integration/http_auth_denial.rs` (FR-004, FR-005, FR-006, SC-002)
- [ ] T020 [P] [US1] Integration test: CORS preflight — valid grant (`204` + exact 3-header grant) and all four denial rows including an absent `Access-Control-Request-Method` — in `tests/integration/http_cors_preflight.rs` (FR-005, `contracts/http-health.md` CORS Preflight)
- [ ] T021 [P] [US1] Integration test: on restart, the service binds a new random port and issues a new token; the previous token no longer authenticates — in `tests/integration/service_restart.rs` (spec.md US1 Acceptance Scenario 4, SC-002)
- [ ] T022 [P] [US1] Integration test: two concurrently started service instances each authenticate only against their own token/port pair — in `tests/integration/concurrent_instances.rs` (`research.md` validation approach)
- [ ] T023 [P] [US1] Integration test: cooperative launcher shutdown — send the supervisor `SIGINT`/`SIGTERM`, assert both children's captured PIDs exit within the bounded timeout — in `tests/integration/launcher_shutdown.rs` (FR-001, FR-018)
- [ ] T024 [P] [US1] Integration test: orphan-process — `SIGKILL` the supervisor itself, assert the same two child PIDs are confirmed exited via a portable liveness probe (POSIX `process.kill(pid, 0)` / Windows equivalent) — in `tests/integration/orphan_process.rs` (FR-001, FR-018, `research.md` corrected mechanism)
- [ ] T025 [P] [US1] Redaction test: negative-grep over captured logs/process output — the token never appears in a URL-shaped, header-echoed, or plain-text log line — in `tests/integration/redaction.rs` (FR-017)

### Implementation for User Story 1

- [ ] T026 [US1] Define `HealthResponse`/`HealthStatus`/`ApiError` Rust types with exact Serde/`ts-rs` attributes (FR-019) in `crates/contracts/src/health.rs` (depends on T012; makes T018 pass)
- [ ] T027 [US1] Implement ephemeral token generation (`rand::OsRng`, base64url no-pad) in `crates/service/src/token.rs` (FR-003)
- [ ] T028 [US1] Implement loopback bind `127.0.0.1:0`, `CONVERGE_ALLOWED_ORIGIN` env read, and the `CONVERGE_BOOTSTRAP` stdout line in `crates/service/src/main.rs` (FR-001, FR-002)
- [ ] T029 [US1] Implement `Host`→`Origin`→token validation middleware, first-failure-wins, in `crates/adapters-http/src/middleware/auth.rs` (FR-004, FR-006; makes T019 pass)
- [ ] T030 [US1] Implement closed-CORS preflight handling (4-row check order, exact 3-header grant, `Access-Control-Allow-Origin` on the actual `200` response too) in `crates/adapters-http/src/middleware/cors.rs` (FR-005; makes T020 pass)
- [ ] T031 [US1] Implement `GET /api/health` handler in `crates/adapters-http/src/routes/health.rs` (FR-007)
- [ ] T032 [US1] Wire the Axum router (health route + middleware stack) in `crates/service/src/main.rs` (depends on T029, T030, T031)
- [ ] T033 [US1] Implement the stdin-pipe EOF watchdog (background `tokio` task; EOF triggers the same shutdown path as `SIGTERM`) in `crates/service/src/main.rs` (`research.md` orphan-process mechanism; makes T024 pass)
- [ ] T034 [US1] Implement graceful shutdown on `SIGINT`/`SIGTERM` (close listener, bounded drain, exit) in `crates/service/src/main.rs` (FR-018; makes T023 pass)
- [ ] T035 [P] [US1] Implement `scripts/dev.mjs` supervisor: fork Vite first, spawn the Rust service with `CONVERGE_ALLOWED_ORIGIN`, forward port/token over IPC, print a redacted status line, cooperative shutdown, and the Vite child's `'disconnect'`-triggered teardown (FR-001, `research.md` Bootstrap Ordering)
- [ ] T036 [P] [US1] Implement the Vite dev-bootstrap middleware — `Host`-validated, same-origin, in-memory port/token cache, `503` while pending — in `apps/web/vite.config.ts` (`research.md` Vite bootstrap endpoint hardening)
- [ ] T037 [P] [US1] Implement the typed HTTP client and TanStack Query hook for health in `apps/web/src/lib/api/health.ts` (FR-008, FR-024)
- [ ] T038 [US1] Implement the health/status frontend surface (Initial loading, Ready, Refreshing/Stale, Service unavailable/Offline, Error) in `apps/web/src/features/health/HealthStatus.tsx` (FR-023, spec.md Frontend State Matrix; depends on T037)
- [ ] T039 [P] [US1] Frontend tests: one test group per applicable Frontend State Matrix state, plus keyboard/focus/non-color-state and `jest-axe` assertions, in `apps/web/src/features/health/HealthStatus.test.tsx` (FR-023, SC-010)
- [ ] T040 [P] [US1] Frontend test: displayed health data updates via TanStack Query invalidation/refetch; Zustand never becomes an authoritative source, in `apps/web/src/features/health/HealthStatus.test.tsx` (FR-024, SC-011)
- [ ] T041 [US1] Browser E2E: load the frontend and observe the Ready state in `tests/e2e/health.e2e.ts` (WebdriverIO) (FR-016, AC-043)

**Checkpoint**: User Story 1 is fully functional and independently testable (SC-001, SC-002).

---

## Phase 4: User Story 2 - Mechanically Verified Clean Architecture Boundaries (Priority: P2)

**Goal**: A repeatable, automated check proves Domain and Application have zero transport/UI/database/filesystem/PTY/Tauri/provider dependency, and fails visibly on a deliberately introduced violation.

**Independent Test**: Run the boundary check against Domain/Application in isolation; confirm it fails when an inward-pointing-rule violation is deliberately introduced (spec.md US2 Acceptance Scenarios 1–2).

### Tests for User Story 2 (MANDATORY — Constitution Principle VI, security-adjacent architecture boundary) ⚠️

- [ ] T042 [P] [US2] Architecture boundary check: `cargo metadata`-driven assertion that `domain`'s normal dependencies ⊆ `{thiserror}` and `application`'s ⊆ `{domain, thiserror, uuid, blake3, serde_json, time}`, plus `#![forbid(unsafe_code)]` presence at all six crate roots, in `tests/architecture/allowlist_check.rs` (FR-009, FR-010, SC-005)
- [ ] T043 [P] [US2] Unit test the allowlist-check logic itself against a synthetic `cargo metadata` fixture containing a deliberately outward-pointing dependency, asserting the check reports a violation rather than passing silently, in `tests/architecture/fixtures/violation_metadata.json` and `tests/architecture/allowlist_check.rs` (spec.md US2 Acceptance Scenario 2)

### Implementation for User Story 2

- [ ] T044 [US2] Wire `cargo test -p domain -p application` (isolated-compile verification) and the allowlist check into the justfile `test-unit` recipe in `justfile` (FR-009, FR-010; depends on T042)

**Checkpoint**: User Story 2 is independently functional and testable (SC-005) — `domain`/`application` compile in isolation, and the check fails visibly on a deliberate violation.

---

## Phase 5: User Story 5 - Minimal Authenticated WebSocket Proof (Priority: P2)

**Goal**: A WebSocket handshake is accepted only with a valid token, matching `Host`, and an allowlisted `Origin`, checked before any application-level exchange; the connection carries only a minimal `hello`/`ping`/`pong` proof.

**Independent Test**: Attempt a handshake with valid token/`Host`/allowlisted `Origin` (expect accept), then repeat with each invalidated individually (expect refusal before any message is exchanged) — spec.md US5 Acceptance Scenarios 1–3.

**Note**: Shares `crates/adapters-http` and `crates/service`'s router/token infrastructure with User Story 1; the implementation tasks below depend on User Story 1's router wiring (T032) being complete, even though this story's *test* is independently executable per spec.md.

### Tests for User Story 5 (MANDATORY — Constitution Principle VI, security boundary + contract) ⚠️

- [ ] T045 [P] [US5] Contract test: `WsMessage` (`hello`/`ping`/`pong`) wire shapes and subprotocol negotiation response (only `converge.v1` echoed back) in `tests/contract/websocket_proof_contract.rs` (FR-021, SC-009)
- [ ] T046 [P] [US5] Integration test: WS handshake denial — `Host` mismatch (400), `Origin` not in the allowlist (403), no valid token subprotocol (401), individually and in combination, never returns `101` — in `tests/integration/ws_auth_denial.rs` (FR-022, SC-009)
- [ ] T047 [P] [US5] Negotiation/redaction test: the token-bearing subprotocol value never appears in the handshake response header or in any captured log line — in `tests/integration/ws_redaction.rs` (FR-017)

### Implementation for User Story 5

- [ ] T048 [US5] Define the `WsMessage` enum (`Hello`/`Ping`/`Pong`, `tag = "type"`, `camelCase` fields) in `crates/contracts/src/ws.rs` (FR-019; depends on T012; makes T045 pass)
- [ ] T049 [US5] Implement the WebSocket upgrade route `GET /api/ws` — `Host`→`Origin`-allowlist→token-subprotocol order, before any `101` response — in `crates/adapters-http/src/routes/ws.rs` (FR-022; depends on T032; makes T046 pass)
- [ ] T050 [US5] Implement subprotocol negotiation (server selects only `converge.v1` in its response) in `crates/adapters-http/src/routes/ws.rs` (`research.md` Subprotocol negotiation; makes T047 pass)
- [ ] T051 [US5] Implement the `hello`/`ping`/`pong` exchange handler (no terminal/PTY/provider streaming) in `crates/adapters-http/src/routes/ws.rs` (spec.md US5 Acceptance Scenario 3)
- [ ] T052 [P] [US5] Implement the frontend WebSocket client (`converge.v1` + `converge.token.<token>` offer) opened after the health bootstrap in `apps/web/src/lib/api/ws.ts` (FR-008)
- [ ] T053 [US5] Extend the browser E2E journey to confirm the WS proof (`hello` received) in `tests/e2e/health.e2e.ts` (FR-016; extends T041)

**Checkpoint**: User Story 5 is independently functional and testable (SC-009) — the WebSocket boundary is proven without terminal or provider code existing.

---

## Phase 6: User Story 3 - Idempotent Event Persistence and Aggregate Rebuild (Priority: P3)

**Goal**: A representative normalized event persists idempotently — resubmission never duplicates the ledger — and the derived aggregate can be discarded and rebuilt from the ledger alone without mutating it.

**Independent Test**: Submit a representative event twice, confirm one ledger row and one equivalent aggregate; discard the aggregate, rebuild it from the ledger, confirm equivalence (spec.md US3 Acceptance Scenarios 1–3).

### Tests for User Story 3 (MANDATORY — Constitution Principle VI, domain logic + application use cases + persistence boundary) ⚠️

- [ ] T054 [P] [US3] Domain unit test: Normalized Event / Aggregate invariants in `crates/domain/src/event.rs` and `crates/domain/src/aggregate.rs` (test modules)
- [ ] T055 [P] [US3] Application unit test: `RecordProbeEvent` — first submission succeeds; identical resubmission is idempotent (no duplicate); resubmission differing in any field returns `IdempotencyConflict` naming the exact `ConflictingFields` — in `crates/application/src/use_cases/record_probe_event.rs` (test module) (FR-012)
- [ ] T056 [P] [US3] Application unit test: `RebuildAggregate` — discard+rebuild equivalence, ledger left unmutated — in `crates/application/src/use_cases/rebuild_aggregate.rs` (test module) (FR-013)
- [ ] T057 [P] [US3] Migration test: fresh empty database → migrations apply → expected schema/triggers exist; re-run is a no-op; `BEFORE UPDATE`/`BEFORE DELETE` triggers `RAISE(ABORT, ...)` — in `tests/integration/migrations.rs` (FR-011, AC-042)
- [ ] T058 [P] [US3] Integration test: sequential and concurrent identical resubmission → exactly one row/aggregate; concurrent resubmission differing only in `event_type` / only in `payload` / only in `occurred_at` → each surfaces `IdempotencyConflict` — in `tests/integration/idempotency_conflict.rs` (FR-012, SC-003)
- [ ] T059 [P] [US3] Integration test: an injected mid-fold failure leaves `aggregates` unchanged (rollback); a concurrent `events` insert blocks while a rebuild transaction is open — in `tests/integration/transactional_rebuild.rs` (FR-013, SC-004)

### Implementation for User Story 3

- [ ] T060 [US3] Write migration `0001_create_events_and_aggregates.sql` (`events`/`aggregates` tables, `payload_hash` column, `BEFORE UPDATE`/`BEFORE DELETE` immutability triggers) in `migrations/0001_create_events_and_aggregates.sql` (FR-011; makes T057 pass)
- [ ] T061 [US3] Define the Normalized Event / Aggregate domain entities in `crates/domain/src/event.rs`, `crates/domain/src/aggregate.rs` (makes T054 pass)
- [ ] T062 [US3] Define `EventRepository`/`AggregateRepository` outbound ports (traits) in `crates/application/src/ports.rs`
- [ ] T063 [US3] Implement the `RecordProbeEvent` use case — producer-supplied UUIDv7 `event_id` passthrough, canonical `serde_json` payload serialization, BLAKE3 `payload_hash`, RFC 3339 `occurred_at` parse/validate, three-outcome conflict classification — in `crates/application/src/use_cases/record_probe_event.rs` (depends on T062; makes T055 pass)
- [ ] T064 [US3] Implement the `RebuildAggregate` use case — `BEGIN IMMEDIATE` → `DELETE FROM aggregates` → fold events in ledger `id` order → `INSERT` per `event_type` → `COMMIT`/`ROLLBACK` — in `crates/application/src/use_cases/rebuild_aggregate.rs` (depends on T062; makes T056 pass)
- [ ] T065 [US3] Implement the SQLx `EventRepository` (`INSERT` + `UNIQUE`-conflict re-`SELECT`/classify) in `crates/adapters-persistence/src/event_repository.rs` (depends on T060, T062; makes T058 pass)
- [ ] T066 [US3] Implement the SQLx `AggregateRepository` (transactional rebuild per T064) in `crates/adapters-persistence/src/aggregate_repository.rs` (depends on T060, T062; makes T059 pass)
- [ ] T067 [P] [US3] Pin `sqlx-cli` install (`cargo install sqlx-cli --version 0.9.0 --locked`) in the justfile `setup` recipe and commit the `.sqlx/` offline query cache — `justfile`, `crates/adapters-persistence/.sqlx/`

**Checkpoint**: User Story 3 is independently functional and testable (SC-003, SC-004) — the ledger is idempotent and the aggregate is rebuildable without mutating it.

---

## Phase 7: User Story 4 - Command-Facade Quality Gates and Base CI (Priority: P4)

**Goal**: Formatting, linting, tests, and builds run identically through the `justfile` locally and in CI on every pull request and push, on Linux fully and Windows x86_64 for build plus applicable technical tests.

**Independent Test**: Run the documented format/lint/test/build recipes locally and confirm they pass; confirm the same recipes run in CI, and a deliberately broken check fails CI visibly (spec.md US4 Acceptance Scenarios 1–3).

**Note**: This story verifies the outcomes of User Stories 1, 2, 3, and 5 rather than introducing new runtime behavior — its tasks depend on those stories' recipes (`test-unit`, `test-contract`, `test-integration`, `test-frontend`, `test-e2e`) having real content to run.

### Implementation for User Story 4

- [ ] T068 [US4] Complete every justfile recipe to its full defined behavior per `plan.md`'s Command Facade table, with `check` as the unconditional complete gate (including `test-e2e`, no environment-dependent skip) — `justfile` (FR-014; depends on T044, T067, and User Story 1/5 test/implementation tasks)
- [ ] T069 [P] [US4] Add the `.github/workflows/ci.yml` Linux job: `just check` unconditionally on `ubuntu-24.04`, triggered on `pull_request` and `push` to the default branch, using `actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`, `actions/setup-node@820762786026740c76f36085b0efc47a31fe5020 # v7.0.0` (`node-version-file: .nvmrc`), `dtolnay/rust-toolchain@2c7215f132e9ebf062739d9130488b56d53c060c # 2026-07-16` (`toolchain: "1.97.1"`, `components: "rustfmt, clippy"`) (FR-015, SC-006)
- [ ] T070 [P] [US4] Add the `.github/workflows/ci.yml` Windows job: `fmt-check`, `lint`, `test-unit`, `test-contract`, `test-integration`, `build` on `windows-2022`, same triggers, no E2E step (FR-015, PLT-005 restriction)
- [ ] T071 [P] [US4] Add a Dependabot/Renovate config to keep the GitHub Actions SHA pins current under human review in `.github/dependabot.yml` (`research.md` GitHub Actions Enforcement)
- [ ] T072 [US4] Verify a deliberately broken check (e.g., a failing test) causes `just check` and CI to fail visibly, then revert the deliberate break — documented verification, no permanent code change (spec.md US4 Acceptance Scenario 3, SC-007)

**Checkpoint**: User Story 4 is independently functional and testable (SC-006, SC-007) — contributors and CI share the identical gate, and nothing passes silently.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Final, whole-slice verification after every story is complete.

- [ ] T073 [P] Run `just contracts-check` end-to-end and confirm zero diff between regenerated and committed TypeScript in `packages/contracts/src/generated/` (FR-020, SC-008)
- [ ] T074 [P] Confirm `just fmt-check` and `just lint` pass repository-wide
- [ ] T075 Run `specs/001-slice-0-foundation/quickstart.md` validation end-to-end (Launch, Security Boundary, Persistence, Architecture Boundaries, Complete Gate) and record results
- [ ] T076 [P] Update `README.md` with the one documented launch command, only if it differs from `quickstart.md`'s `just dev`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately.
- **Foundational (Phase 2)**: Depends on Setup — BLOCKS all user stories.
- **User Stories (Phases 3–7)**: All depend on Foundational completion.
  - **US1 (Phase 3, P1)**: No dependency on any other story.
  - **US2 (Phase 4, P2)**: No dependency on any other story — touches only `tests/architecture/` and the justfile.
  - **US5 (Phase 5, P2)**: Independently *testable* per spec.md, but its implementation tasks (T049–T051) depend on US1's router wiring (T032) because both live in `crates/adapters-http`/`crates/service`.
  - **US3 (Phase 6, P3)**: No dependency on US1/US2/US5 — touches `crates/domain`, `crates/application`, `crates/adapters-persistence`, `migrations/`, none of which US1/US2/US5 touch.
  - **US4 (Phase 7, P4)**: Depends on US1, US2, US3, and US5 all being complete — it wires and verifies the gates that exercise their recipes; ordered last per spec.md ("verifies the outcomes of Stories 1–3 rather than introducing new runtime behavior").
- **Polish (Phase 8)**: Depends on every desired user story being complete.

### File-Ownership Map (for parallel safety)

| Story | Files it owns |
| --- | --- |
| US1 | `crates/contracts/src/health.rs`, `crates/service/*`, `crates/adapters-http/src/middleware/*`, `crates/adapters-http/src/routes/health.rs`, `scripts/dev.mjs`, `apps/web/vite.config.ts`, `apps/web/src/lib/api/health.ts`, `apps/web/src/features/health/*`, `tests/integration/http_*.rs`, `tests/integration/launcher_shutdown.rs`, `tests/integration/orphan_process.rs`, `tests/integration/redaction.rs`, `tests/integration/concurrent_instances.rs`, `tests/integration/service_restart.rs`, `tests/contract/http_health_contract.rs`, `tests/e2e/health.e2e.ts` |
| US2 | `tests/architecture/*` |
| US5 | `crates/contracts/src/ws.rs`, `crates/adapters-http/src/routes/ws.rs`, `apps/web/src/lib/api/ws.ts`, `tests/integration/ws_*.rs`, `tests/contract/websocket_proof_contract.rs` (extends the shared `tests/e2e/health.e2e.ts` after US1's T041) |
| US3 | `crates/domain/src/event.rs`, `crates/domain/src/aggregate.rs`, `crates/application/src/use_cases/*`, `crates/application/src/ports.rs`, `crates/adapters-persistence/src/*`, `migrations/*`, `tests/integration/migrations.rs`, `tests/integration/idempotency_conflict.rs`, `tests/integration/transactional_rebuild.rs` |
| US4 | `justfile` (completes what Setup's T008 created), `.github/workflows/ci.yml`, `.github/dependabot.yml` |

US1 and US5 share `crates/service/src/main.rs` router wiring — that is the one real coupling point; everything else above is disjoint by file.

### Within Each User Story

- Per Constitution Principle VI: tests for domain logic, application use cases, contracts, and security boundaries MUST be written and FAIL before their paired implementation task (Red-Green-Refactor); frontend/adapter/infrastructure tests are included in the same change without requiring test-first ordering.
- Contracts/wire types before the routes that use them.
- Ports/traits before the repositories that implement them.
- Router wiring before route handlers that register into it.
- Story complete (checkpoint) before its dependents (US4) begin.

### Parallel Opportunities

- All `[P]` Setup tasks (T002–T007) can run in parallel.
- All `[P]` Foundational tasks (T010–T014, T017) can run in parallel; T015 and T016 depend on all six crates existing.
- Once Foundational completes, **US1, US2, and US3 can start in parallel** (disjoint files). US5 can start in parallel for its test tasks (T045–T047) but its implementation tasks (T049–T051) must wait for US1's T032.
- Within US1: all test tasks (T018–T025) in parallel; among implementation tasks, T035, T036, T037 (frontend/supervisor) are parallel with each other and with the Rust-side tasks once contracts/routing exist.
- Within US3: all test tasks (T054–T059) in parallel; T065 and T066 both depend on T060/T062 but are mutually parallel (different files).
- US4 cannot start until US1, US2, US3, and US5 are all at their checkpoints.

---

## Parallel Example: User Story 1 Tests

```bash
Task: "Contract test for HealthResponse/HealthStatus/ApiError in tests/contract/http_health_contract.rs"
Task: "Integration test for HTTP denial paths in tests/integration/http_auth_denial.rs"
Task: "Integration test for CORS preflight in tests/integration/http_cors_preflight.rs"
Task: "Integration test for service restart in tests/integration/service_restart.rs"
Task: "Integration test for concurrent instances in tests/integration/concurrent_instances.rs"
Task: "Integration test for launcher shutdown in tests/integration/launcher_shutdown.rs"
Task: "Integration test for orphan process in tests/integration/orphan_process.rs"
Task: "Redaction test in tests/integration/redaction.rs"
```

## Parallel Example: User Story 3 Tests

```bash
Task: "Domain unit test for Event/Aggregate invariants in crates/domain/src/event.rs"
Task: "Application unit test for RecordProbeEvent in crates/application/src/use_cases/record_probe_event.rs"
Task: "Application unit test for RebuildAggregate in crates/application/src/use_cases/rebuild_aggregate.rs"
Task: "Migration test in tests/integration/migrations.rs"
Task: "Idempotency/conflict integration test in tests/integration/idempotency_conflict.rs"
Task: "Transactional rebuild integration test in tests/integration/transactional_rebuild.rs"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational (blocks all stories).
3. Complete Phase 3: User Story 1.
4. **STOP and VALIDATE**: run `tests/integration/http_auth_denial.rs`, `tests/integration/http_cors_preflight.rs`, and `tests/e2e/health.e2e.ts` against a `just dev` run; confirm SC-001/SC-002 independently.

### Incremental Delivery

1. Setup + Foundational → foundation ready.
2. Add User Story 1 → validate independently (MVP: authenticated, launchable service).
3. Add User Story 2 and User Story 3 in parallel → validate each independently (architecture boundary; ledger/aggregate mechanism).
4. Add User Story 5 → validate independently (WebSocket boundary), building on US1's router.
5. Add User Story 4 last → validate the complete local/CI gate against the now-complete recipes.
6. Complete Phase 8: Polish (contract drift, format/lint, full quickstart replay).

### Parallel Team Strategy

With multiple contributors, after Foundational completes:

- Contributor A: User Story 1 (then User Story 5, since it depends on US1's router).
- Contributor B: User Story 2.
- Contributor C: User Story 3.
- Whoever finishes first takes User Story 4 once US1, US2, US3, and US5 are all at their checkpoints.

---

## Constitution Compliance Check (Task-Breakdown Level)

Re-verified against `.specify/memory/constitution.md` v1.0.1 and `plan.md`'s Post-Design Gate (human-approval confirmation pass), which reports all ten principles at **PASS**:

| Principle | Task-breakdown verification | Status |
| --- | --- | --- |
| I. Versioned Sources of Truth | Every task cites an FR/SC/AC/design-artifact already present in `spec.md`/`plan.md`/`data-model.md`/`contracts/`; no new requirement is introduced by this breakdown. | **PASS** |
| II. Modular Clean Architecture | Phase 2 (T010–T016) and User Story 2 (T042–T044) make the six-crate allowlist and the mechanical check explicit tasks, not assumed. | **PASS** |
| III. Secure Local Runtime | User Story 1 (T027–T036) and User Story 5 (T049–T051) implement loopback/random-port/ephemeral-token/`Host`/`Origin`/CORS/allowlist exactly as `research.md` Part 2 and the contracts specify; no task introduces shell or unrestricted filesystem access. | **PASS** |
| IV. Contracts as Code | T026 and T048 make Rust the authoritative source with exact Serde/`ts-rs` attributes; T017/T073 wire and verify the drift gate. | **PASS** |
| V. Durable and Reconstructable Data | User Story 3 (T054–T067) implements immutable-ledger triggers, idempotency/conflict classification, and the transactional rebuild exactly per `data-model.md`. | **PASS** |
| VI. Risk-Oriented Test-First Development | Every domain/application/contract/security-boundary task above is paired with a test task that precedes it and is required to fail first (RGR); UI/adapter/infrastructure tasks are tested in the same-phase without forced test-first ordering. | **PASS** |
| VII. Product and UX Integrity | T038–T040 implement and test the Frontend State Matrix, both themes, keyboard/focus/non-color/reduced-motion, and `jest-axe`, exactly as FR-023 and spec.md require, with no state invented or skipped beyond the approved matrix. | **PASS** |
| VIII. Privacy and Observability | T025 (HTTP) and T047 (WebSocket) are dedicated redaction tests; no task introduces telemetry or automatic transmission. | **PASS** |
| IX. Documentation and Delivery Discipline | T008/T068 make the `justfile` the single command facade for contributors and CI; T069–T070 mirror it in CI with no competing validation path; T075 replays `quickstart.md` end-to-end. | **PASS** |
| X. Human-Controlled Version Control | No task in this breakdown stages, commits, pushes, or opens a pull request; this document is itself a plain file pending human review. | **PASS** |

**No unresolved conflict.** Per the user's explicit instruction for this run, `/speckit-implement` was **NOT** invoked. No production code, migration, configuration, or test file was created or modified while generating this task list — only `specs/001-slice-0-foundation/tasks.md` was written. This stops here for human review before any implementation task begins.

---

## Notes

- `[P]` tasks touch different files with no unmet dependency.
- `[Story]` maps each task to its user story for traceability back to `spec.md` and `traceability.md`.
- RGR (Red-Green-Refactor) tasks are cross-referenced above ("makes T0xx pass") so the paired test can be verified to fail before its implementation task starts.
- Per Constitution Principle X, no agent may stage, commit, or push after completing tasks — report a change summary, files changed, verification results, risks, and exact suggested Git commands for the human maintainer instead.
- Stop at any checkpoint to validate a story independently before continuing.
