---

description: "Task list for Slice 0 — Foundation implementation"
---

# Tasks: Slice 0 — Foundation

**Input**: Design documents from `specs/001-slice-0-foundation/`

**Prerequisites**: `plan.md` (required — Approved), `spec.md` (required — Approved), `research.md`, `data-model.md`, `contracts/http-health.md`, `contracts/websocket-proof.md`, `traceability.md`, `.specify/memory/constitution.md` (v1.0.1)

**Correction pass note (this revision)**: this file corrects nine defects found in the prior generation, all confined to this task list — no normative document (`spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `traceability.md`) was changed. The corrections are: (1) a concrete, human-approved Cargo strategy for `tests/architecture`, `tests/contract`, `tests/integration` (see "Test Harness Package" below); (2) every exact pin from `research.md`/`plan.md` synchronized, including previously-missing TanStack Query, Zustand, Vitest, Testing Library, jsdom, jest-axe, WebdriverIO, and `service`'s own direct `tokio` dependency; (3) explicit tasks for QueryClient bootstrap, Vitest/WebdriverIO configuration, contract generation, Graphite Signal tokens/themes/reduced motion, and Rust module skeletons; (4) false `[P]` markers removed where tasks share a file, and file-ownership conflicts resolved (`justfile`, `apps/web/package.json`, `apps/web/src/main.tsx`, `.github/workflows/ci.yml`); (5) `RebuildAggregate` corrected to contain no SQL — it orchestrates through an explicit transaction-owning port and a pure fold function; (6) "commit the `.sqlx` cache" and the CI-break verification reworded so no task implies an agent-executed Git action; (7) the orphan-process test now names the exact POSIX/Windows termination and liveness mechanisms; (8) a full task-to-requirement traceability matrix restored; (9) the Constitution compliance table re-verified against every correction above.

**Tests**: Mandatory per Constitution Principle VI, scoped by risk. Red-Green-Refactor (test written and failing before implementation) is mandatory for domain logic, application use cases, contracts, and security boundaries — every test task below marked against those layers MUST be completed, and MUST fail, before its paired implementation task. UI, configuration, infrastructure, and adapter tests are included in the same change without requiring literal test-first ordering.

**Organization**: Tasks are grouped by user story (spec.md priorities P1–P4, with US2 and US5 tied at P2) to enable independent implementation and testing of each story, per spec.md's explicit "Independent Test" for each.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no unmet dependency)
- **[Story]**: US1–US5, mapping to spec.md's five user stories. Setup, Foundational, and Polish tasks carry no story label.
- Every task states its exact file path(s) and, where applicable, the FR/SC/AC/PLT/SEC identifier(s) it satisfies (`traceability.md`). A full task-level matrix is at the end of this file.

## Repository State at Start

This slice starts from an empty implementation surface: no `crates/`, `apps/web/`, `packages/`, `migrations/`, `scripts/`, `tests/`, `justfile`, or `.github/workflows/` exist yet in the repository. Every task below creates new files; none modifies pre-existing implementation code.

## Test Harness Package (human-approved this revision)

`tests/architecture/`, `tests/contract/`, and `tests/integration/` hold plain `.rs` files under a top-level directory that is not itself a Cargo package — Cargo cannot compile or run them without one. **Human decision for this correction pass**: a single, dedicated, non-production Cargo package, `tests/Cargo.toml` (package name `converge-tests`), is added as a **seventh workspace member**, alongside — never replacing — the six approved production crates under `crates/`. This package:

- Carries **no** Clean Architecture normal-dependency allowlist (FR-009/FR-010's mechanical check, Testing Plan, only ever applies to `domain` and `application`; a black-box test harness is architecturally inert by definition and is explicitly excluded from that check's scope in the task that wires it, T049).
- Owns every `.rs` file under `tests/architecture/`, `tests/contract/`, `tests/integration/` as an explicit `[[test]]` target (`path = "..."`) — not Cargo's automatic `tests/*.rs` discovery, which only scans a package's immediate `tests/` directory, not subdirectories.
- Is created once, fully, in Foundational (T018) with every target pre-declared against a trivially-passing stub body, so no later story ever re-edits `tests/Cargo.toml` itself — each story only replaces the body of its own already-declared, uniquely-named stub file(s), which keeps every later story's test-writing task genuinely single-file and parallel-safe.
- `tests/e2e/` is a separate, ordinary **pnpm** package (`@converge/e2e`, WebdriverIO), not part of this Cargo package — it is TypeScript, not Rust.

## Path Conventions (from `plan.md` Project Structure)

- Rust crates (six, production, Clean-Architecture-allowlisted): `crates/domain/`, `crates/application/`, `crates/contracts/`, `crates/adapters-http/`, `crates/adapters-persistence/`, `crates/service/`
- Rust test harness (seventh workspace member, non-production, no allowlist): `tests/Cargo.toml`, package `converge-tests`
- Frontend: `apps/web/src/features/health/`, `apps/web/src/lib/api/`, `apps/web/src/state/`, `apps/web/src/styles/`
- Generated TypeScript contracts: `packages/contracts/src/generated/` (regenerated, never hand-edited); `packages/contracts/src/index.ts` (hand-written barrel, never regenerated)
- Migrations: `migrations/`
- Dev-launch supervisor: `scripts/dev.mjs`
- Tests: `tests/architecture/`, `tests/contract/`, `tests/integration/` (all three: `converge-tests` Cargo package), `tests/e2e/` (`@converge/e2e` pnpm package, WebdriverIO)
- CI: `.github/workflows/ci.yml`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Repository/tooling scaffolding with no story-specific behavior.

- [ ] T001 Create top-level directory structure (`crates/{domain,application,contracts,adapters-http,adapters-persistence,service}/src`, `apps/web/src/{features/health,lib/api,state,styles}`, `packages/contracts/src/generated`, `migrations/`, `scripts/`, `tests/{architecture,contract,integration}` as the future `tests/Cargo.toml` package root, `tests/e2e` as the future `@converge/e2e` pnpm package root) per `plan.md` Project Structure
- [ ] T002 [P] Create root Cargo workspace manifest in `Cargo.toml`: `[workspace] members = ["crates/domain", "crates/application", "crates/contracts", "crates/adapters-http", "crates/adapters-persistence", "crates/service", "tests"]` (seven members — six production crates plus the test harness, see "Test Harness Package" above) and a `[workspace.dependencies]` table pinning every Rust dependency this slice uses, exactly per `research.md`/`plan.md` Technical Context: `axum = "=0.8.9"` (feature `"ws"` added per-crate), `tower-http = "=0.7.0"` (features `"cors"`, `"trace"` added per-crate), `tokio = "=1.53.1"` (features added per-crate), `sqlx = "=0.9.0"` (features `"sqlite"`, `"runtime-tokio"`, `"macros"`, `"migrate"` added per-crate), `ts-rs = "=12.0.1"`, `rand = "=0.10.2"`, `thiserror = "=2.0.19"`, `uuid = "=1.24.0"` (default features at the workspace level; the `"v7"` feature is added only on the test-harness package's own dependency edge, T018 — `application` never enables it, per `data-model.md`'s Event identity section), `blake3 = "=1.8.5"`, `serde = "=1.0.229"` (feature `"derive"` added per-crate), `serde_json = "=1.0.151"`, `time = "=0.3.54"` (features added per-crate), `base64 = "=0.22.1"`, `reqwest = "=0.13.4"` (dev/test-only), `tokio-tungstenite = "=0.30.0"` (dev/test-only) (FR-002–FR-008, FR-012, FR-019–FR-022; `research.md` "Exact-pin policy")
- [ ] T003 [P] Pin Rust toolchain (`channel = "1.97.1"`, `components = ["rustfmt", "clippy"]`) in `rust-toolchain.toml`
- [ ] T004 [P] Initialize pnpm workspace: `pnpm-workspace.yaml` (`packages: ["apps/web", "packages/contracts", "tests/e2e"]`), root `package.json` (`"packageManager": "pnpm@11.15.1"`, `engines.node`), `.nvmrc` (`24.18.0`)
- [ ] T005 [P] Scaffold `apps/web` with Vite + React + TypeScript per `research.md` pins (`react`/`react-dom` 19.2.8, `vite` 8.1.5, `@vitejs/plugin-react` 6.0.4, `@types/react` 19.2.17, `@types/react-dom` 19.2.3, `typescript` 6.0.3) plus its production state-management dependencies (`@tanstack/react-query` 5.101.4, `zustand` 5.0.14) and its workspace dependency on the generated contracts package (`"@converge/contracts": "workspace:*"`) — `apps/web/package.json` (sole owner of this file's `@converge/contracts` dependency line — see File-Ownership Map), `apps/web/tsconfig.json`, `apps/web/index.html`, `apps/web/src/main.tsx` (empty entry point; populated by T041/T043/T045)
- [ ] T006 [P] Scaffold `packages/contracts` as a pnpm workspace package (`@converge/contracts`) — `packages/contracts/package.json` (no dependency on/edit of `apps/web/package.json` — see File-Ownership Map), `packages/contracts/src/generated/.gitkeep`, `packages/contracts/src/index.ts` (hand-written barrel; re-exports everything from `./generated`; never itself regenerated — FR-020)
- [ ] T007 [P] Configure ESLint flat config + Prettier per `research.md` pins (`eslint` 10.7.0, `eslint-config-prettier` 10.1.8, `typescript-eslint` 8.65.0, `eslint-plugin-react-hooks` 7.1.1, `prettier` 3.9.6) — `eslint.config.js`, `.prettierrc`
- [ ] T008 [P] Configure Vitest + React Testing Library + jsdom + jest-axe per `research.md` pins (`vitest` 4.1.10, `@testing-library/react` 16.3.2, `@testing-library/dom` 10.4.1, `jsdom` 29.1.1, `jest-axe` 10.0.0) — `apps/web/package.json` devDependencies (same file as T005; append-only, no conflicting edit), `apps/web/vitest.config.ts` (`environment: "jsdom"`, `setupFiles: "./vitest.setup.ts"`), `apps/web/vitest.setup.ts` (`expect.extend(require("jest-axe").toHaveNoViolations)` and `@testing-library/jest-dom` matchers)
- [ ] T009 [P] Scaffold the WebdriverIO E2E harness as its own pnpm package (`@converge/e2e`) per `research.md` pins (`webdriverio` 9.30.0, `@wdio/cli` 9.30.0, `@wdio/local-runner` 9.30.0, `@wdio/mocha-framework` 9.30.0, `@wdio/spec-reporter` 9.29.1) — `tests/e2e/package.json`, `tests/e2e/wdio.conf.ts` (automatic browser/driver management per `research.md`, no manual driver provisioning — supports `just check`'s unconditional `test-e2e` step)
- [ ] T010 Create `justfile` with every recipe name from `plan.md`'s Command Facade table (`setup`, `dev`, `fmt`, `fmt-check`, `lint`, `test-unit`, `test-contract`, `test-integration`, `test-frontend`, `test-e2e`, `build`, `check`, `contracts-generate`, `contracts-check`) as skeletons wired to their underlying tool, with `test-unit`/`test-contract`/`test-integration` skeletons already targeting the `converge-tests` package by exact `--test <name>` flags (real bodies filled by T051, T077, T078 — `justfile` is a shared file across those three tasks; edits MUST be applied in that sequence regardless of which stories otherwise run in parallel, see Dependencies & Execution Order)
- [ ] T011 Verify Constitution compliance (Principles I–X) for this task breakdown against `specs/001-slice-0-foundation/plan.md`'s Constitution Check — Post-Design Gate before implementation begins; record any deviation here before proceeding

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Crate-level scaffolding every user story depends on.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T012 [P] Configure `crates/domain/Cargo.toml` — normal-dependency allowlist `{thiserror.workspace = true}` only; no serde, I/O, framework, or async
- [ ] T013 [P] Configure `crates/application/Cargo.toml` — normal-dependency allowlist `{domain (path), thiserror.workspace = true, uuid.workspace = true (default features only — never `"v7"`), blake3.workspace = true, serde_json.workspace = true, time.workspace = true (feature `"parsing"`)}`; `tokio.workspace = true` only as a dev-dependency (for `#[tokio::test]`-based unit tests)
- [ ] T014 [P] Configure `crates/contracts/Cargo.toml` — normal-dependency allowlist `{serde.workspace = true (feature "derive"), ts-rs.workspace = true}`; no dependency on `domain`/`application`
- [ ] T015 [P] Configure `crates/adapters-http/Cargo.toml` — allowlist `{application (path), domain (path), contracts (path), axum.workspace = true (feature "ws"), tower-http.workspace = true (features "cors","trace"), tokio.workspace = true, thiserror.workspace = true, serde_json.workspace = true, time.workspace = true (features "formatting","parsing","serde")}`
- [ ] T016 [P] Configure `crates/adapters-persistence/Cargo.toml` — allowlist `{application (path), domain (path), sqlx.workspace = true (features "sqlite","runtime-tokio","macros","migrate"), tokio.workspace = true, thiserror.workspace = true}`; no `serde_json`
- [ ] T017 Configure `crates/service/Cargo.toml` — composition root, depends on all five other crates plus `rand.workspace = true`, `base64.workspace = true`, **and its own direct `tokio.workspace = true` dependency** (features `"rt-multi-thread"`, `"macros"`, `"signal"`, `"process"`, `"io-std"` — the async runtime `service`'s own `main.rs` needs for the Axum server, the `SIGINT`/`SIGTERM` handlers, and the stdin-EOF watchdog; missing from the previous draft of this task list). No allowlist restriction applies to `service` itself (the only crate exempt from the per-crate allowlist check, per `plan.md` Project Structure)
- [ ] T018 [P] Create `tests/Cargo.toml` — the test-harness package (`converge-tests`, workspace member #7, no Clean Architecture allowlist restriction; see "Test Harness Package" above). Normal dependencies: `domain`/`application`/`contracts` (path, for white-box assertions where needed), `reqwest.workspace = true`, `tokio-tungstenite.workspace = true`, `tokio.workspace = true` (full features), `serde_json.workspace = true`, `time.workspace = true` (feature `"parsing"`), and `uuid = { workspace = true, features = ["v7"] }` — **the `"v7"` generation feature belongs on this package's dependency edge specifically**, since `data-model.md`'s Event identity section names "the integration test suite" as the producer that generates `event_id`. Declare one `[[test]]` target per eventual test file, each with a trivially-passing stub body created in this same task: `allowlist_check` (`architecture/allowlist_check.rs`), `http_health_contract` (`contract/http_health_contract.rs`), `websocket_proof_contract` (`contract/websocket_proof_contract.rs`), `http_auth_denial`, `http_cors_preflight`, `service_restart`, `concurrent_instances`, `launcher_shutdown`, `orphan_process`, `redaction`, `ws_auth_denial`, `ws_redaction`, `migrations`, `idempotency_conflict`, `transactional_rebuild` (all under `integration/`). `tests/architecture/fixtures/violation_metadata.json` is a data fixture, not a compiled target — created empty here, populated by T050.
- [ ] T019 Add `#![forbid(unsafe_code)]` at the crate root of all six production crates, `service` included with no exception, and create each crate's module skeleton so it compiles as an empty but structurally complete unit: `crates/domain/src/lib.rs` (`pub mod event; pub mod aggregate;`), `crates/application/src/lib.rs` (`pub mod ports; pub mod use_cases;`, with `crates/application/src/use_cases/mod.rs` declaring `pub mod record_probe_event; pub mod rebuild_aggregate;`), `crates/contracts/src/lib.rs` (`pub mod health; pub mod ws;`), `crates/adapters-http/src/lib.rs` (`pub mod middleware; pub mod routes;`, with `mod.rs` files declaring `pub mod auth; pub mod cors;` under `middleware/` and `pub mod health; pub mod ws;` under `routes/`), `crates/adapters-persistence/src/lib.rs` (`pub mod event_repository; pub mod aggregate_repository;`), `crates/service/src/main.rs` (`mod token;`) — every submodule file referenced here is created as an empty stub in this task; later tasks fill in real content, never re-declare the module

**Checkpoint**: All six production crates plus the test-harness package (`tests/`, package `converge-tests`) compile; the six crates' allowlists and `#![forbid(unsafe_code)]` are structurally in place; every planned test target exists as a trivially-passing stub. User story implementation can now begin.

---

## Phase 3: User Story 1 - One-Command Launch to an Authenticated Health Check (Priority: P1) 🎯 MVP

**Goal**: `just dev` starts the frontend and the local Rust service together; the service binds only to loopback on a random port, issues an ephemeral token, and serves an authenticated health path enforcing token/`Host`/`Origin`/CORS.

**Independent Test**: From a clean checkout, run `just dev`, then request the health path with a valid token/`Host`/`Origin` (expect success) and with each individually invalidated (expect denial), per spec.md's Acceptance Scenarios 1–4.

### Tests for User Story 1 (MANDATORY — Constitution Principle VI, security boundary) ⚠️

- [ ] T020 [P] [US1] Contract test: `HealthResponse`/`HealthStatus`/`ApiError` wire shapes with exact `camelCase` field names in `tests/contract/http_health_contract.rs` (`converge-tests`, replacing its T018 stub) (FR-021, AC-043)
- [ ] T021 [P] [US1] Integration test: `Host` mismatch (400), `Origin` absent/mismatched (403), missing/wrong token (401), individually and in combination, first-failure-wins order — in `tests/integration/http_auth_denial.rs` (FR-004, FR-005, FR-006, SEC-003, SC-002)
- [ ] T022 [P] [US1] Integration test: CORS preflight — valid grant (`204` + exact 3-header grant) and all four denial rows including an absent `Access-Control-Request-Method` — in `tests/integration/http_cors_preflight.rs` (FR-005, SEC-003, `contracts/http-health.md` CORS Preflight)
- [ ] T023 [P] [US1] Integration test: on restart, the service binds a new random port and issues a new token; the previous token no longer authenticates — in `tests/integration/service_restart.rs` (spec.md US1 Acceptance Scenario 4, SC-002)
- [ ] T024 [P] [US1] Integration test: two concurrently started service instances each authenticate only against their own token/port pair — in `tests/integration/concurrent_instances.rs` (`research.md` validation approach)
- [ ] T025 [P] [US1] Integration test: cooperative launcher shutdown — send the supervisor `SIGINT`/`SIGTERM`, assert both children's captured PIDs exit within the bounded timeout — in `tests/integration/launcher_shutdown.rs` (FR-001, FR-018)
- [ ] T026 [P] [US1] Integration test: orphan-process, exact platform mechanism — **terminate the supervisor abruptly via Rust `std::process::Child::kill()`** (sends `SIGKILL` on POSIX; calls `TerminateProcess` on Windows — std's own portable, uncatchable primitive, no new dependency needed), then confirm the two previously-captured child PIDs (Vite, Rust service) are exited via a portable liveness probe implemented by shelling out to the platform's native tool: POSIX `kill -0 <pid>` (nonzero/`ESRCH` exit means not running) via `std::process::Command`; Windows `tasklist /FI "PID eq <pid>" /NH` via `std::process::Command`, treating empty output as not running — in `tests/integration/orphan_process.rs` (FR-001, FR-018, `research.md` corrected mechanism)
- [ ] T027 [P] [US1] Redaction test: negative-grep over captured logs/process output — the token never appears in a URL-shaped, header-echoed, or plain-text log line — in `tests/integration/redaction.rs` (FR-017, SEC-008)

### Implementation for User Story 1

- [ ] T028 [US1] Define `HealthResponse`/`HealthStatus`/`ApiError` Rust types with exact Serde/`ts-rs` attributes (FR-019) in `crates/contracts/src/health.rs` (depends on T014, T019; makes T020 pass)
- [ ] T029 [US1] Run `just contracts-generate` to produce `packages/contracts/src/generated/health.ts` from `crates/contracts/src/health.rs` and add its re-export to `packages/contracts/src/index.ts` (T006's hand-written barrel) — the previous task list referenced generated frontend types before any task ever produced them; this closes that gap (depends on T028; FR-019, FR-020)
- [ ] T030 [US1] Implement ephemeral token generation (`rand::OsRng`, base64url no-pad via `base64 =0.22.1`) in `crates/service/src/token.rs` (FR-003)
- [ ] T031 [US1] Implement loopback bind `127.0.0.1:0`, `CONVERGE_ALLOWED_ORIGIN` env read, and the `CONVERGE_BOOTSTRAP` stdout line in `crates/service/src/main.rs` (FR-001, FR-002, SEC-003)
- [ ] T032 [US1] Implement `Host`→`Origin`→token validation middleware, first-failure-wins, in `crates/adapters-http/src/middleware/auth.rs` (FR-004, FR-006, SEC-003; makes T021 pass)
- [ ] T033 [US1] Implement closed-CORS preflight handling (4-row check order, exact 3-header grant, `Access-Control-Allow-Origin` on the actual `200` response too) in `crates/adapters-http/src/middleware/cors.rs` (FR-005, SEC-003; makes T022 pass)
- [ ] T034 [US1] Implement `GET /api/health` handler in `crates/adapters-http/src/routes/health.rs` (FR-007)
- [ ] T035 [US1] Wire the Axum router (health route + middleware stack) in `crates/service/src/main.rs` (depends on T032, T033, T034)
- [ ] T036 [US1] Implement the stdin-pipe EOF watchdog (background `tokio` task; EOF triggers the same shutdown path as `SIGTERM`) in `crates/service/src/main.rs` (`research.md` orphan-process mechanism; makes T026 pass)
- [ ] T037 [US1] Implement graceful shutdown on `SIGINT`/`SIGTERM` (close listener, bounded drain, exit) in `crates/service/src/main.rs` (FR-018; makes T025 pass)
- [ ] T038 [P] [US1] Implement `scripts/dev.mjs` supervisor: fork Vite first, spawn the Rust service with `CONVERGE_ALLOWED_ORIGIN`, forward port/token over IPC, print a redacted status line, cooperative shutdown, and the Vite child's `'disconnect'`-triggered teardown (FR-001, `research.md` Bootstrap Ordering)
- [ ] T039 [P] [US1] Implement the Vite dev-bootstrap middleware — `Host`-validated, same-origin, in-memory port/token cache, `503` while pending — in `apps/web/vite.config.ts` (`research.md` Vite bootstrap endpoint hardening)
- [ ] T040 [P] [US1] Implement Graphite Signal design tokens — primitives → semantic → component CSS Custom Properties (`--cv-{category}-{role}-{state?}` naming, `DESIGN.md` §3–§4.3) for both Light and Dark themes, plus typed TypeScript accessors — in `apps/web/src/styles/tokens.css` and `apps/web/src/styles/tokens.ts` (FR-023, PLT-011, Constitution VII)
- [ ] T041 [US1] Implement theme selection (system-default, explicit Light/Dark override, persisted preference, no prolonged flash on switch, per `DESIGN.md` §14) in `apps/web/src/state/themeStore.ts` (Zustand — transient UI/layout state only, FR-024) and wire theme initialization in `apps/web/src/main.tsx` (depends on T040; shares `main.tsx` with T043/T045 — sequential, not parallel; see File-Ownership Map)
- [ ] T042 [P] [US1] Implement `prefers-reduced-motion` handling — remove displacement/parallax/sequenced motion, use a short crossfade or instant change instead, per `DESIGN.md` §10 — in `apps/web/src/styles/motion.css` (FR-023, PLT-011)
- [ ] T043 [US1] Implement the QueryClient bootstrap — `QueryClient` instance and `QueryClientProvider` mounting — in `apps/web/src/lib/api/queryClient.ts` and `apps/web/src/main.tsx` (depends on T041 — same file, sequential; FR-024)
- [ ] T044 [US1] Implement the typed HTTP client and TanStack Query hook for health, against the generated `HealthResponse` type, in `apps/web/src/lib/api/health.ts` (depends on T029, T043; FR-008, FR-024)
- [ ] T045 [US1] Implement the health/status frontend surface (Initial loading, Ready, Refreshing/Stale, Service unavailable/Offline, Error), consuming only `--cv-*` tokens (T040) and the reduced-motion rules (T042), in `apps/web/src/features/health/HealthStatus.tsx`, mounted in `apps/web/src/main.tsx` (depends on T040, T042, T044, and T043 — same file, sequential; FR-023, spec.md Frontend State Matrix)
- [ ] T046 [US1] Frontend tests: one test group per applicable Frontend State Matrix state, plus keyboard/focus/non-color-state and `jest-axe` assertions, in `apps/web/src/features/health/HealthStatus.test.tsx` (depends on T045; FR-023, SC-010)
- [ ] T047 [US1] Frontend test: displayed health data updates via TanStack Query invalidation/refetch; Zustand never becomes an authoritative source — appended to `apps/web/src/features/health/HealthStatus.test.tsx` (same file as T046 — not parallel) (FR-024, SC-011)
- [ ] T048 [US1] Browser E2E: load the frontend and observe the Ready state in `tests/e2e/health.e2e.ts` (`@converge/e2e`, WebdriverIO) (FR-016, AC-043)

**Checkpoint**: User Story 1 is fully functional and independently testable (SC-001, SC-002).

---

## Phase 4: User Story 2 - Mechanically Verified Clean Architecture Boundaries (Priority: P2)

**Goal**: A repeatable, automated check proves Domain and Application have zero transport/UI/database/filesystem/PTY/Tauri/provider dependency, and fails visibly on a deliberately introduced violation.

**Independent Test**: Run the boundary check against Domain/Application in isolation; confirm it fails when an inward-pointing-rule violation is deliberately introduced (spec.md US2 Acceptance Scenarios 1–2).

### Tests for User Story 2 (MANDATORY — Constitution Principle VI, security-adjacent architecture boundary) ⚠️

- [ ] T049 [P] [US2] Architecture boundary check: shells out to `cargo metadata --format-version 1`, parses the JSON via `serde_json`, and asserts `domain`'s normal dependencies ⊆ `{thiserror}` and `application`'s ⊆ `{domain, thiserror, uuid, blake3, serde_json, time}` — checked only against these two production crates, never against the `converge-tests` harness package itself, which is architecturally inert by design (see "Test Harness Package" above) — plus `#![forbid(unsafe_code)]` presence at all six production crate roots, in `tests/architecture/allowlist_check.rs` (`converge-tests`, replacing its T018 stub) (FR-009, FR-010, SC-005)
- [ ] T050 [US2] Unit test the allowlist-check logic itself against a synthetic `cargo metadata` fixture containing a deliberately outward-pointing dependency, asserting the check reports a violation rather than passing silently — appended to the same `tests/architecture/allowlist_check.rs` (same file as T049 — not parallel) and `tests/architecture/fixtures/violation_metadata.json` (populates T018's empty fixture) (spec.md US2 Acceptance Scenario 2)

### Implementation for User Story 2

- [ ] T051 [US2] Wire `cargo test --workspace --lib` (isolated unit-test compile of every crate, `domain`/`application` included) and `cargo test -p converge-tests --test allowlist_check` into the `justfile` `test-unit` recipe (FR-009, FR-010; depends on T049; `justfile` is shared with T010/T077/T078 — this edit applies second in that sequence)

**Checkpoint**: User Story 2 is independently functional and testable (SC-005) — `domain`/`application` compile in isolation, and the check fails visibly on a deliberate violation.

---

## Phase 5: User Story 5 - Minimal Authenticated WebSocket Proof (Priority: P2)

**Goal**: A WebSocket handshake is accepted only with a valid token, matching `Host`, and an allowlisted `Origin`, checked before any application-level exchange; the connection carries only a minimal `hello`/`ping`/`pong` proof.

**Independent Test**: Attempt a handshake with valid token/`Host`/allowlisted `Origin` (expect accept), then repeat with each invalidated individually (expect refusal before any message is exchanged) — spec.md US5 Acceptance Scenarios 1–3.

**Note**: Shares `crates/adapters-http` and `crates/service`'s router/token infrastructure with User Story 1; the implementation tasks below depend on User Story 1's router wiring (T035) being complete, even though this story's *test* is independently executable per spec.md.

### Tests for User Story 5 (MANDATORY — Constitution Principle VI, security boundary + contract) ⚠️

- [ ] T052 [P] [US5] Contract test: `WsMessage` (`hello`/`ping`/`pong`) wire shapes and subprotocol negotiation response (only `converge.v1` echoed back) in `tests/contract/websocket_proof_contract.rs` (`converge-tests`, replacing its T018 stub) (FR-021, SC-009)
- [ ] T053 [P] [US5] Integration test: WS handshake denial — `Host` mismatch (400), `Origin` not in the allowlist (403), no valid token subprotocol (401), individually and in combination, never returns `101` — in `tests/integration/ws_auth_denial.rs` (FR-022, SEC-001, SEC-003, SC-009)
- [ ] T054 [P] [US5] Negotiation/redaction test: the token-bearing subprotocol value never appears in the handshake response header or in any captured log line — in `tests/integration/ws_redaction.rs` (FR-017)

### Implementation for User Story 5

- [ ] T055 [US5] Define the `WsMessage` enum (`Hello`/`Ping`/`Pong`, `tag = "type"`, `camelCase` fields) in `crates/contracts/src/ws.rs` (depends on T014, T019; makes T052 pass)
- [ ] T056 [US5] Run `just contracts-generate` to produce `packages/contracts/src/generated/ws.ts` from `crates/contracts/src/ws.rs` and add its re-export to `packages/contracts/src/index.ts` (depends on T055; FR-019, FR-020)
- [ ] T057 [US5] Implement the WebSocket upgrade route `GET /api/ws` — `Host`→`Origin`-allowlist→token-subprotocol order, before any `101` response — in `crates/adapters-http/src/routes/ws.rs` (FR-022; depends on T035; makes T053 pass)
- [ ] T058 [US5] Implement subprotocol negotiation (server selects only `converge.v1` in its response) in `crates/adapters-http/src/routes/ws.rs` (`research.md` Subprotocol negotiation; makes T054 pass)
- [ ] T059 [US5] Implement the `hello`/`ping`/`pong` exchange handler (no terminal/PTY/provider streaming) in `crates/adapters-http/src/routes/ws.rs` (spec.md US5 Acceptance Scenario 3)
- [ ] T060 [P] [US5] Implement the frontend WebSocket client (`converge.v1` + `converge.token.<token>` offer), against the generated `WsMessage` type, opened after the health bootstrap, in `apps/web/src/lib/api/ws.ts` (depends on T056; FR-008)
- [ ] T061 [US5] Extend the browser E2E journey to confirm the WS proof (`hello` received) — appended to `tests/e2e/health.e2e.ts` (same file as T048 — not parallel) (FR-016)

**Checkpoint**: User Story 5 is independently functional and testable (SC-009) — the WebSocket boundary is proven without terminal or provider code existing.

---

## Phase 6: User Story 3 - Idempotent Event Persistence and Aggregate Rebuild (Priority: P3)

**Goal**: A representative normalized event persists idempotently — resubmission never duplicates the ledger — and the derived aggregate can be discarded and rebuilt from the ledger alone without mutating it.

**Independent Test**: Submit a representative event twice, confirm one ledger row and one equivalent aggregate; discard the aggregate, rebuild it from the ledger, confirm equivalence (spec.md US3 Acceptance Scenarios 1–3).

**Clean Architecture note (corrected this pass)**: `RebuildAggregate` (application) MUST contain no SQL, transaction keywords, or `sqlx` types. It orchestrates exclusively through an explicit, transaction-owning `AggregateRepository::rebuild` port method (T071) that accepts a pure fold function (T072) as a parameter. `BEGIN IMMEDIATE`, `DELETE`, `SELECT`, `INSERT`, `COMMIT`/`ROLLBACK`, and every `sqlx` type are owned exclusively by `adapters-persistence` (T076), which calls the injected pure function *inside* its own open transaction. This is the concrete mechanism, not merely an assertion — the previous draft's `RebuildAggregate` task literally named `BEGIN IMMEDIATE`/`DELETE`/`INSERT`/`COMMIT` inside `crates/application`, which this corrects.

### Tests for User Story 3 (MANDATORY — Constitution Principle VI, domain logic + application use cases + persistence boundary) ⚠️

- [ ] T062 [P] [US3] Domain unit test: Normalized Event / Aggregate invariants in `crates/domain/src/event.rs` and `crates/domain/src/aggregate.rs` (test modules)
- [ ] T063 [P] [US3] Application unit test: `RecordProbeEvent` — first submission succeeds; identical resubmission is idempotent (no duplicate); resubmission differing in any field returns `IdempotencyConflict` naming the exact `ConflictingFields` — in `crates/application/src/use_cases/record_probe_event.rs` (test module) (FR-012)
- [ ] T064 [P] [US3] Application unit test: the **pure** `fold_events` function — given an in-memory slice of `Event` values (no I/O, no mock repository needed), asserts correct per-`event_type` projection counts and `last_event_id` — in `crates/application/src/use_cases/rebuild_aggregate.rs` (test module) (FR-013)
- [ ] T065 [P] [US3] Application unit test: `RebuildAggregate` orchestration — using a fake/in-memory `AggregateRepository` port implementation, asserts the use case calls `repository.rebuild(fold_events)` exactly once and returns the port's result verbatim, and asserts (by construction — the use case has no SQL-capable dependency available to it) that it cannot itself issue any transaction or SQL statement — in `crates/application/src/use_cases/rebuild_aggregate.rs` (test module) (FR-013)
- [ ] T066 [P] [US3] Migration test: fresh empty database → migrations apply → expected schema/triggers exist; re-run is a no-op; `BEFORE UPDATE`/`BEFORE DELETE` triggers `RAISE(ABORT, ...)` — in `tests/integration/migrations.rs` (`converge-tests`, replacing its T018 stub) (FR-011, AC-042)
- [ ] T067 [P] [US3] Integration test: sequential and concurrent identical resubmission → exactly one row/aggregate; concurrent resubmission differing only in `event_type` / only in `payload` / only in `occurred_at` → each surfaces `IdempotencyConflict` — in `tests/integration/idempotency_conflict.rs` (FR-012, SC-003)
- [ ] T068 [P] [US3] Integration test: an injected mid-fold failure leaves `aggregates` unchanged (rollback), asserted against the real `adapters-persistence` transaction (this is the one test in this story that exercises the actual `BEGIN IMMEDIATE` behavior, confirming it — not `application` — owns it); a concurrent `events` insert blocks while a rebuild transaction is open — in `tests/integration/transactional_rebuild.rs` (FR-013, SC-004)

### Implementation for User Story 3

- [ ] T069 [US3] Write migration `0001_create_events_and_aggregates.sql` (`events`/`aggregates` tables, `payload_hash` column, `BEFORE UPDATE`/`BEFORE DELETE` immutability triggers) in `migrations/0001_create_events_and_aggregates.sql` (FR-011; makes T066 pass)
- [ ] T070 [US3] Define the Normalized Event / Aggregate domain entities in `crates/domain/src/event.rs`, `crates/domain/src/aggregate.rs` (makes T062 pass)
- [ ] T071 [US3] Define `EventRepository`/`AggregateRepository` outbound ports (traits) in `crates/application/src/ports.rs`, making the transaction-owning contract explicit: `AggregateRepository::rebuild(&self, fold: impl Fn(&[Event]) -> Vec<AggregateProjection>) -> Result<(), RepositoryError>` — the port's single method is the *only* place a rebuild can be triggered from `application`, and its signature accepts the pure fold logic as a parameter rather than exposing any SQL-shaped operation (`SELECT`/`DELETE`/`INSERT`) as separate port methods `application` could sequence itself
- [ ] T072 [US3] Implement the **pure** `fold_events(events: &[Event]) -> Vec<AggregateProjection>` function — no I/O, no SQL, no port dependency — in `crates/application/src/use_cases/rebuild_aggregate.rs` (makes T064 pass)
- [ ] T073 [US3] Implement the `RecordProbeEvent` use case — producer-supplied UUIDv7 `event_id` passthrough, canonical `serde_json` payload serialization, BLAKE3 `payload_hash`, RFC 3339 `occurred_at` parse/validate, three-outcome conflict classification — in `crates/application/src/use_cases/record_probe_event.rs` (depends on T071; makes T063 pass)
- [ ] T074 [US3] Implement the `RebuildAggregate` use case — calls `repository.rebuild(fold_events)` and returns its result; contains no SQL, no transaction keyword, no `sqlx` type — in `crates/application/src/use_cases/rebuild_aggregate.rs` (depends on T071, T072; makes T065 pass)
- [ ] T075 [US3] Implement the SQLx `EventRepository` (`INSERT` + `UNIQUE`-conflict re-`SELECT`/classify) in `crates/adapters-persistence/src/event_repository.rs` (depends on T069, T071; makes T067 pass)
- [ ] T076 [US3] Implement the SQLx `AggregateRepository::rebuild` — owns `BEGIN IMMEDIATE` → `DELETE FROM aggregates` → `SELECT * FROM events ORDER BY id ASC` → calls the caller-supplied `fold` closure on the fetched events → `INSERT` per resulting projection → `COMMIT`/`ROLLBACK` — exclusively, in `crates/adapters-persistence/src/aggregate_repository.rs` (depends on T069, T071; makes T068 pass)
- [ ] T077 [US3] Pin `sqlx-cli` install (`cargo install sqlx-cli --version 0.9.0 --locked`) in the `justfile` `setup` recipe, and **generate/update** (not commit) the `.sqlx/` offline query cache via `cargo sqlx prepare` against a running local dev database — staging and committing `crates/adapters-persistence/.sqlx/` is left entirely to the human maintainer (Constitution X; corrected — the previous draft said "commit the cache," which no task may do) — `justfile`, `crates/adapters-persistence/.sqlx/` (generated artifact, not staged/committed by this task; `justfile` is shared with T010/T051/T078 — this edit applies third in that sequence)

**Checkpoint**: User Story 3 is independently functional and testable (SC-003, SC-004) — the ledger is idempotent and the aggregate is rebuildable without mutating it, and `application` contains no SQL anywhere in the rebuild path.

---

## Phase 7: User Story 4 - Command-Facade Quality Gates and Base CI (Priority: P4)

**Goal**: Formatting, linting, tests, and builds run identically through the `justfile` locally and in CI on every pull request and push, on Linux fully and Windows x86_64 for build plus applicable technical tests.

**Independent Test**: Run the documented format/lint/test/build recipes locally and confirm they pass; confirm the same recipes run in CI, and a deliberately broken check fails CI visibly (spec.md US4 Acceptance Scenarios 1–3).

**Note**: This story verifies the outcomes of User Stories 1, 2, 3, and 5 rather than introducing new runtime behavior — its tasks depend on those stories' recipes (`test-unit`, `test-contract`, `test-integration`, `test-frontend`, `test-e2e`) having real content to run.

### Implementation for User Story 4

- [ ] T078 [US4] Complete every `justfile` recipe to its full defined behavior per `plan.md`'s Command Facade table, with `check` as the unconditional complete gate (including `test-e2e`, no environment-dependent skip); `test-contract` runs `cargo test -p converge-tests --test http_health_contract --test websocket_proof_contract` plus `just contracts-check`; `test-integration` runs `cargo test -p converge-tests` filtered to every `integration/*` target listed in T018; `test-frontend` runs `pnpm --filter @converge/web test`; `test-e2e` runs `pnpm --filter @converge/e2e wdio run wdio.conf.ts` — `justfile` (FR-014; depends on T051, T077, and every User Story 1/3/5 test/implementation task; this is the fourth and final sequential edit to `justfile`)
- [ ] T079 [P] [US4] Add `.github/workflows/ci.yml` with **both** CI jobs in this one file (previously two separate parallel tasks incorrectly claimed `[P]` on the same file — corrected): a **Linux job** running `just check` unconditionally on `ubuntu-24.04`, and a **Windows job** running `just fmt-check`, `just lint`, `just test-unit`, `just test-contract`, `just test-integration`, `just build` on `windows-2022` (no E2E step, per `PLT-005`) — both triggered on `pull_request` and `push` to the default branch, using `actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`, `actions/setup-node@820762786026740c76f36085b0efc47a31fe5020 # v7.0.0` (`node-version-file: .nvmrc`), `dtolnay/rust-toolchain@2c7215f132e9ebf062739d9130488b56d53c060c # 2026-07-16` (`toolchain: "1.97.1"`, `components: "rustfmt, clippy"`) (FR-015, PLT-005, PLT-009, SC-006)
- [ ] T080 [P] [US4] Add a Dependabot/Renovate config to keep the GitHub Actions SHA pins current under human review in `.github/dependabot.yml` (`research.md` GitHub Actions Enforcement)
- [ ] T081 [US4] Document the manual, human-performed CI-break verification runbook — a human maintainer pushes a throwaway branch/PR with one deliberately failing check, confirms `just check` and CI both fail visibly, then closes/deletes the throwaway branch — appended to `specs/001-slice-0-foundation/quickstart.md` under a new "Manual CI-Break Verification (human-performed)" heading. **This task is documentation only; the agent completing it MUST NOT execute `git push`, open a pull request, commit a deliberate break, or revert Git history** (Constitution X, AGENTS.md Git and Release Boundaries) — corrected from the previous draft, which described performing and reverting the break itself (spec.md US4 Acceptance Scenario 3, SC-007)

**Checkpoint**: User Story 4 is independently functional and testable (SC-006, SC-007) — contributors and CI share the identical gate, and nothing passes silently.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Final, whole-slice verification after every story is complete.

- [ ] T082 [P] Run `just contracts-check` end-to-end and confirm zero diff between regenerated and committed TypeScript in `packages/contracts/src/generated/` (FR-020, SC-008)
- [ ] T083 [P] Confirm `just fmt-check` and `just lint` pass repository-wide
- [ ] T084 Run `specs/001-slice-0-foundation/quickstart.md` validation end-to-end (Launch, Security Boundary, Persistence, Architecture Boundaries, Complete Gate, including T081's manual CI-break runbook) and record results
- [ ] T085 [P] Update `README.md` with the one documented launch command, only if it differs from `quickstart.md`'s `just dev`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately.
- **Foundational (Phase 2)**: Depends on Setup — BLOCKS all user stories.
- **User Stories (Phases 3–7)**: All depend on Foundational completion.
  - **US1 (Phase 3, P1)**: No dependency on any other story.
  - **US2 (Phase 4, P2)**: No dependency on any other story — touches only `tests/architecture/` and the `justfile` (second sequential edit).
  - **US5 (Phase 5, P2)**: Independently *testable* per spec.md, but its implementation tasks (T057–T059) depend on US1's router wiring (T035) because both live in `crates/adapters-http`/`crates/service`.
  - **US3 (Phase 6, P3)**: No dependency on US1/US2/US5 — touches `crates/domain`, `crates/application`, `crates/adapters-persistence`, `migrations/`, and the `justfile` (third sequential edit).
  - **US4 (Phase 7, P4)**: Depends on US1, US2, US3, and US5 all being complete — it wires and verifies the gates that exercise their recipes (`justfile`'s fourth and final sequential edit); ordered last per spec.md ("verifies the outcomes of Stories 1–3 rather than introducing new runtime behavior").
- **Polish (Phase 8)**: Depends on every desired user story being complete.

### File-Ownership Map (for parallel safety)

| Story | Files it owns |
| --- | --- |
| US1 | `crates/contracts/src/health.rs`, `packages/contracts/src/generated/health.ts` + its line in `src/index.ts`, `crates/service/*`, `crates/adapters-http/src/middleware/*`, `crates/adapters-http/src/routes/health.rs`, `scripts/dev.mjs`, `apps/web/vite.config.ts`, `apps/web/src/styles/*`, `apps/web/src/state/themeStore.ts`, `apps/web/src/lib/api/queryClient.ts`, `apps/web/src/lib/api/health.ts`, `apps/web/src/features/health/*`, `apps/web/src/main.tsx` (sequential internal edits T041→T043→T045 — see below), `tests/integration/http_*.rs`, `tests/integration/launcher_shutdown.rs`, `tests/integration/orphan_process.rs`, `tests/integration/redaction.rs`, `tests/integration/concurrent_instances.rs`, `tests/integration/service_restart.rs`, `tests/contract/http_health_contract.rs`, `tests/e2e/health.e2e.ts` |
| US2 | `tests/architecture/*` |
| US5 | `crates/contracts/src/ws.rs`, `packages/contracts/src/generated/ws.ts` + its line in `src/index.ts`, `crates/adapters-http/src/routes/ws.rs`, `apps/web/src/lib/api/ws.ts`, `tests/integration/ws_*.rs`, `tests/contract/websocket_proof_contract.rs` (extends the shared `tests/e2e/health.e2e.ts` after US1's T048) |
| US3 | `crates/domain/src/event.rs`, `crates/domain/src/aggregate.rs`, `crates/application/src/use_cases/*`, `crates/application/src/ports.rs`, `crates/adapters-persistence/src/*`, `migrations/*`, `tests/integration/migrations.rs`, `tests/integration/idempotency_conflict.rs`, `tests/integration/transactional_rebuild.rs` |
| US4 | `.github/workflows/ci.yml` (single task, T079 — no longer split across two falsely-parallel tasks), `.github/dependabot.yml`, `specs/001-slice-0-foundation/quickstart.md` (T081 addition) |

**Cross-story shared files (not disjoint — explicit sequencing required, not parallel):**

- `justfile`: created once in Setup (T010, skeleton), then edited sequentially by US2 (T051), US3 (T077), and US4 (T078) in that exact order, regardless of which stories otherwise run in parallel. This is the one cross-story file-ownership exception to the table above.
- `tests/Cargo.toml`: created once, fully, in Foundational (T018) with every `[[test]]` target pre-declared; no story ever edits this file again — each story only replaces the body of its own already-declared stub `.rs` file(s), which restores per-story parallel safety on the individual test files.
- `apps/web/package.json`: T005 is the sole owner of its `@converge/contracts` workspace dependency line; T008 only appends devDependencies to the same file (append-only, no conflicting edit — the two are sequenced, not marked `[P]` against each other).
- `apps/web/src/main.tsx`: edited sequentially within US1 by T041 (theme init) → T043 (QueryClient provider) → T045 (mount `HealthStatus`); none of the three is marked `[P]` against the others.
- `crates/service/src/main.rs`: US1 and US5 share this file's router wiring — the one real cross-story coupling point beyond `justfile` and `tests/Cargo.toml`.

### Within Each User Story

- Per Constitution Principle VI: tests for domain logic, application use cases, contracts, and security boundaries MUST be written and FAIL before their paired implementation task (Red-Green-Refactor); frontend/adapter/infrastructure tests are included in the same change without requiring test-first ordering.
- Contracts/wire types before the routes that use them; `contracts-generate` before any frontend code imports the generated type.
- Ports/traits before the repositories that implement them and before the use cases that call them.
- Router wiring before route handlers that register into it.
- Story complete (checkpoint) before its dependents (US4) begin.

### Parallel Opportunities

- All `[P]` Setup tasks (T002–T009) can run in parallel; T010/T011 depend on the rest of Setup existing.
- All `[P]` Foundational tasks (T012–T018) can run in parallel; T019 depends on all seven Rust packages (six crates + test harness) existing.
- Once Foundational completes, **US1, US2, and US3 can start in parallel** (disjoint files, `justfile`'s sequencing aside). US5 can start in parallel for its test tasks (T052–T054) but its implementation tasks (T057–T059) must wait for US1's T035.
- Within US1: all test tasks (T020–T027) in parallel; among implementation tasks, T038, T039, T040, T042 are mutually parallel and parallel with the Rust-side tasks; T041/T043/T045 are sequential (shared `main.tsx`).
- Within US3: all test tasks (T062–T068) in parallel; T075 and T076 both depend on T069/T071 but are mutually parallel (different files).
- US4 cannot start until US1, US2, US3, and US5 are all at their checkpoints, and `justfile`'s prior three sequential edits (T010, T051, T077) are done.

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
Task: "Application unit test for pure fold_events in crates/application/src/use_cases/rebuild_aggregate.rs"
Task: "Application unit test for RebuildAggregate orchestration in crates/application/src/use_cases/rebuild_aggregate.rs"
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
3. Add User Story 2 and User Story 3 in parallel → validate each independently (architecture boundary; ledger/aggregate mechanism). Coordinate `justfile` edits sequentially even though the rest of each story is parallel.
4. Add User Story 5 → validate independently (WebSocket boundary), building on US1's router (T035).
5. Add User Story 4 last → validate the complete local/CI gate against the now-complete recipes.
6. Complete Phase 8: Polish (contract drift, format/lint, full quickstart replay).

### Parallel Team Strategy

With multiple contributors, after Foundational completes:

- Contributor A: User Story 1 (then User Story 5, since it depends on US1's router).
- Contributor B: User Story 2.
- Contributor C: User Story 3.
- `justfile` edits (T051, T077, T078) MUST be coordinated sequentially between B, C, and whoever takes US4, even though the rest of their stories proceed independently.
- Whoever finishes first takes User Story 4 once US1, US2, US3, and US5 are all at their checkpoints.

---

## Task-to-Requirement Traceability Matrix

Full task-level cross-reference to `traceability.md`'s FR/SC/AC/PLT/SEC identifiers, restoring what the previous draft only partially inlined. IDs marked **out of scope** in `traceability.md` (`AC-036`, `AC-037`, `SEC-002`, `SEC-004`, `PLT-003`, `PLT-006`, `PLT-007`, `SPEC-002`–`SPEC-006`) are intentionally addressed by **no task below** — see `traceability.md`'s "Non-Applicable IDs" section for the explicit justification; they are not silently dropped, they are silently *absent from this slice by design*.

| Task(s) | FR | SC | AC | PLT / SEC | Constitution |
| --- | --- | --- | --- | --- | --- |
| T001–T009, T011 | — | — | — | — | I, IX |
| T002 | FR-002–FR-008, FR-012, FR-019–FR-022 | — | — | — | I, IV |
| T010, T051, T077, T078 | FR-014 | SC-006, SC-007 | AC-040, AC-044 | PLT-005, PLT-009 | IX |
| T012–T017 | FR-009, FR-010 | SC-005 | AC-041 | — | II |
| T018, T049, T050 | FR-009, FR-010 | SC-005 | AC-041 | — | II, VI |
| T019 | FR-009, FR-010 | SC-005 | AC-041 | — | II, III |
| T020, T028, T029 | FR-019, FR-020, FR-021 | SC-008 | AC-043 | — | IV |
| T021, T032 | FR-004, FR-006 | SC-002 | AC-034 | SEC-003 | III |
| T022, T033 | FR-005 | SC-002 | AC-034 | SEC-003 | III |
| T023 | FR-003 | SC-002 | AC-033 | SEC-001 | III |
| T024 | FR-001, FR-002 | SC-001 | AC-033 | SEC-003 | III |
| T025, T037 | FR-018 | SC-007 | AC-039 | — | III |
| T026, T036 | FR-001, FR-018 | SC-001 | AC-039 | — | III |
| T027 | FR-017 | — | AC-038 | SEC-008 | VIII |
| T030 | FR-003 | — | AC-033 | SEC-001 | III |
| T031 | FR-001, FR-002 | SC-001 | AC-033, AC-040 | PLT-001 (restricted), PLT-004 | III |
| T034 | FR-007 | — | AC-033 | — | — |
| T035 | FR-001–FR-007 | SC-001, SC-002 | AC-033, AC-034, AC-040 | — | II, III |
| T038 | FR-001 | SC-001 | AC-040 | PLT-001 (restricted), PLT-004 | III |
| T039 | FR-001 | SC-001 | AC-040 | — | III |
| T040, T042 | FR-023 | SC-010 | — | PLT-011 | VII |
| T041 | FR-024 | SC-011 | — | PLT-004 | VII |
| T043, T044 | FR-008, FR-024 | SC-011 | AC-035 | PLT-004 | II |
| T045 | FR-023 | SC-010 | — | PLT-011 | VII |
| T046 | FR-023 | SC-010 | — | PLT-011 | VII |
| T047 | FR-024 | SC-011 | — | PLT-004 | II |
| T048, T061 | FR-016 | — | AC-043 | PLT-008 | VI |
| T052, T055, T056 | FR-019, FR-020, FR-021 | SC-008, SC-009 | AC-043 | — | IV |
| T053, T057 | FR-022 | SC-009 | AC-033, AC-034 | SEC-001, SEC-003 | III |
| T054, T058 | FR-017, FR-022 | SC-009 | AC-038 | SEC-008 | VIII |
| T059 | FR-016 | SC-009 | AC-043 | — | VI |
| T060 | FR-008 | — | AC-035 | — | II |
| T062, T070 | FR-011, FR-012, FR-013 | — | — | PLT-002 | V |
| T063, T073 | FR-012 | SC-003 | — | — | V |
| T064, T065, T072, T074 | FR-013 | SC-004 | AC-042 | PLT-002 | II, V |
| T066, T069 | FR-011 | — | AC-042 | PLT-002 | V |
| T067, T075 | FR-012 | SC-003 | AC-042 | PLT-002 | V |
| T068, T076 | FR-013 | SC-004 | AC-042 | PLT-002 | V |
| T079 | FR-015 | SC-006 | AC-044 | PLT-005 (restricted), PLT-009 (restricted) | IX |
| T080 | FR-015 | — | — | PLT-009 (restricted) | IX |
| T081 | FR-018 | SC-007 | — | — | IX, X |
| T082 | FR-019, FR-020 | SC-008 | AC-043 | — | IV |
| T083–T085 | FR-014, FR-016 | SC-006, SC-007 | AC-040, AC-043, AC-044 | — | IX |

---

## Constitution Compliance Check (Task-Breakdown Level)

Re-verified against `.specify/memory/constitution.md` v1.0.1 and `plan.md`'s Post-Design Gate (human-approval confirmation pass), and re-checked specifically against this pass's nine corrections:

| Principle | Task-breakdown verification | Status |
| --- | --- | --- |
| I. Versioned Sources of Truth | Every task cites an FR/SC/AC/PLT/SEC/design-artifact already present in `spec.md`/`plan.md`/`data-model.md`/`contracts/`/`traceability.md` (full matrix above); no new requirement is introduced by this correction pass. The test-harness package decision (T018) was made explicitly by the human maintainer this pass, not silently assumed. | **PASS** |
| II. Modular Clean Architecture | Phase 2 (T012–T019) and User Story 2 (T049–T051) make the six-crate allowlist and the mechanical check explicit tasks; the seventh, test-only workspace member (`tests/`) is explicitly excluded from that allowlist check by design (T049), not silently swept into "six crates." `RebuildAggregate` is now corrected to contain no SQL (T071, T072, T074, T076) — **strengthened this pass**, closing a real Clean Architecture violation the previous draft's T064 introduced. | **PASS**, strengthened |
| III. Secure Local Runtime | User Story 1 (T030–T039) and User Story 5 (T057–T059) implement loopback/random-port/ephemeral-token/`Host`/`Origin`/CORS/allowlist exactly as `research.md` Part 2 and the contracts specify. The orphan-process test (T026) now names the exact platform-specific termination (`Child::kill()` → `SIGKILL`/`TerminateProcess`) and liveness-probe (`kill -0` / `tasklist`) mechanisms — **strengthened this pass**, closing a previously vague "Windows equivalent" reference. No task introduces shell or unrestricted filesystem access. | **PASS**, strengthened |
| IV. Contracts as Code | T028/T055 make Rust the authoritative source with exact Serde/`ts-rs` attributes; T029/T056 (new this pass) actually run `contracts-generate` before any frontend task imports the generated type, closing a real gap where generated TypeScript was referenced (T044/T060) before any task ever produced it. T082 verifies the drift gate. | **PASS**, strengthened |
| V. Durable and Reconstructable Data | User Story 3 (T062–T077) implements immutable-ledger triggers, idempotency/conflict classification, and the transactional rebuild exactly per `data-model.md` — now with the rebuild's transaction ownership made explicit and confined to `adapters-persistence` alone (T071, T076), correcting a real violation the previous draft introduced. | **PASS**, strengthened |
| VI. Risk-Oriented Test-First Development | Every domain/application/contract/security-boundary task above is paired with a test task that precedes it and is required to fail first (RGR); the pure `fold_events` function and the `RebuildAggregate` orchestration now have separate, independently RGR-paired tests (T064/T072, T065/T074) reflecting their separated responsibilities. UI/adapter/infrastructure tasks are tested in the same phase without forced test-first ordering. | **PASS** |
| VII. Product and UX Integrity | T040–T042 make Graphite Signal tokens, theme selection, and reduced-motion handling explicit tasks with exact file paths for the first time this pass, closing a real gap where FR-023/`DESIGN.md` compliance was asserted without any concrete implementation task. T045–T046 implement and test the Frontend State Matrix, both themes, keyboard/focus/non-color/reduced-motion, and `jest-axe`, exactly as FR-023 and spec.md require. | **PASS**, strengthened |
| VIII. Privacy and Observability | T027 (HTTP) and T054 (WebSocket) are dedicated redaction tests; no task introduces telemetry or automatic transmission. | **PASS** |
| IX. Documentation and Delivery Discipline | T010/T078 make the `justfile` the single command facade for contributors and CI, with its four sequential edit-owners now explicit (T010→T051→T077→T078) rather than an implicit shared-file conflict; T079 merges the previously falsely-parallel Linux/Windows CI tasks into one correctly-scoped task; T081 replaces "revert the deliberate break" with a human-performed runbook, removing the only place this task list previously implied an agent-executed Git action short of an actual `add`/`commit`. T084 replays `quickstart.md` end-to-end. | **PASS**, strengthened |
| X. Human-Controlled Version Control | No task in this breakdown stages, commits, pushes, or opens a pull request. T077's `.sqlx/` cache task now explicitly generates the cache and leaves staging/committing to the human maintainer (corrected from "commit the cache," which this pass found and fixed). T081's CI-break verification is now explicitly a documentation-only, human-performed runbook, not an agent-executed push/revert. This document is itself a plain file pending human review. | **PASS**, strengthened |

**No unresolved conflict.** Per the user's explicit instruction for this correction pass, `/speckit-implement` was **NOT** invoked. No production code, migration, configuration, or test file was created or modified while generating or correcting this task list — only `specs/001-slice-0-foundation/tasks.md` was written. The one decision requiring human approval (the test-harness package's shape) was resolved by explicit human confirmation before this file was rewritten (see "Test Harness Package" above). This stops here for human review before any implementation task begins.

---

## Notes

- `[P]` tasks touch different files with no unmet dependency. Every previously-incorrect `[P]` marker on a shared-file pair (T039/T040, T042/T043, T069/T070 in the prior draft's numbering) has been corrected in this revision — see File-Ownership Map for the cross-story shared files that require explicit sequencing instead.
- `[Story]` maps each task to its user story for traceability back to `spec.md` and `traceability.md`.
- RGR (Red-Green-Refactor) tasks are cross-referenced above ("makes T0xx pass") so the paired test can be verified to fail before its implementation task starts.
- Per Constitution Principle X, no agent may stage, commit, push, revert Git history, or open a pull request after completing tasks — report a change summary, files changed, verification results, risks, and exact suggested Git commands for the human maintainer instead. T081 in particular MUST NOT be executed as a live Git operation by an agent — it is a documentation task describing a human-performed procedure.
- Stop at any checkpoint to validate a story independently before continuing.
