---

description: "Task list for Slice 0 — Foundation implementation"
---

# Tasks: Slice 0 — Foundation

**Input**: Design documents from `specs/001-slice-0-foundation/`

**Prerequisites**: `plan.md` (required — Approved), `spec.md` (required — Approved), `research.md`, `data-model.md`, `contracts/http-health.md`, `contracts/websocket-proof.md`, `traceability.md`, `.specify/memory/constitution.md` (v1.0.1)

**Correction pass note (first revision)**: an earlier revision of this file corrected nine defects found in the original generation — a concrete Cargo test-harness strategy, dependency-pin synchronization, missing tooling tasks, false `[P]` markers, a Clean Architecture fix for `RebuildAggregate`, human-controlled Git wording, a platform-specific orphan-process mechanism, a full traceability matrix, and a re-verified Constitution table.

**Correction pass note (second revision, this pass)**: a follow-up audit found ten further defects, all corrected here, none touching any normative document: (1) two remaining invalid `[P]`/ownership conflicts (`T005`/`T008` on `apps/web/package.json`; the tasks now numbered `T066`/`T067` on `rebuild_aggregate.rs`), plus a full re-validation of every `[P]` marker in the document, which also found and fixed a false *absence* of `[P]` on `T077`/`T078` (genuinely parallel, different files); (2) explicit cross-story dependency edges added where a task silently relied on another story's file-append without declaring it (`T056`→`T029`, `T063`→`T048`, plus `packages/contracts/src/index.ts`'s full three-story chain); (3) the WebSocket vertical slice completed — two tasks were missing entirely: registering `/api/ws` into the running router (new `T060`) and a frontend consumer that actually opens the connection the E2E test observes (new `T062`); (4) the orphan-process test's invented Windows `tasklist`/empty-output parsing removed and replaced with the exact mechanism `research.md` names, with the residual language ambiguity honestly flagged rather than silently resolved; (5) `AggregateRepository::rebuild` corrected from a synchronous to a native-`async fn`-in-trait port contract, matching `adapters-persistence`'s inherently async SQLx implementation, with no new dependency; (6) `converge-tests`' `Cargo.toml` given the `adapters-persistence`/`sqlx` dependencies its own migration/persistence/transaction tests actually need, and a genuine `futures-util` dependency gap for the WebSocket tests reported rather than silently invented; (7) the Vitest setup corrected to drop the unpinned `@testing-library/jest-dom` and to configure `jest-axe` via ESM `import`, not `require`; (8) the final quickstart-validation task rewritten so it only checks that the human-performed CI-break runbook exists and is correct, never executes it itself; (9) the traceability matrix corrected to stop over-attributing requirements to tasks that only support nearby infrastructure; (10) the Constitution table re-verified against all of the above, including the two items honestly left open. Renumbering after `T059` was unavoidable (two new tasks); every cross-reference in this file was updated to match. Total task count: **87** (was 85 — see "Flagged Items" and the Notes section for the exact accounting).

**Tests**: Mandatory per Constitution Principle VI, scoped by risk. Red-Green-Refactor (test written and failing before implementation) is mandatory for domain logic, application use cases, contracts, and security boundaries — every test task below marked against those layers MUST be completed, and MUST fail, before its paired implementation task. UI, configuration, infrastructure, and adapter tests are included in the same change without requiring literal test-first ordering.

**Organization**: Tasks are grouped by user story (spec.md priorities P1–P4, with US2 and US5 tied at P2) to enable independent implementation and testing of each story, per spec.md's explicit "Independent Test" for each.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no unmet dependency)
- **[Story]**: US1–US5, mapping to spec.md's five user stories. Setup, Foundational, and Polish tasks carry no story label.
- Every task states its exact file path(s) and, where applicable, the FR/SC/AC/PLT/SEC identifier(s) it satisfies (`traceability.md`). A full task-level matrix is at the end of this file.

## Repository State at Start

This slice starts from an empty implementation surface: no `crates/`, `apps/web/`, `packages/`, `migrations/`, `scripts/`, `tests/`, `justfile`, or `.github/workflows/` exist yet in the repository. Every task below creates new files; none modifies pre-existing implementation code.

## Test Harness Package (human-approved, first revision)

`tests/architecture/`, `tests/contract/`, and `tests/integration/` hold plain `.rs` files under a top-level directory that is not itself a Cargo package — Cargo cannot compile or run them without one. **Human decision, first correction pass**: a single, dedicated, non-production Cargo package, `tests/Cargo.toml` (package name `converge-tests`), is added as a **seventh workspace member**, alongside — never replacing — the six approved production crates under `crates/`. This package:

- Carries **no** Clean Architecture normal-dependency allowlist (FR-009/FR-010's mechanical check, Testing Plan, only ever applies to `domain` and `application`; a black-box test harness is architecturally inert by definition and is explicitly excluded from that check's scope in the task that wires it, T049).
- Owns every `.rs` file under `tests/architecture/`, `tests/contract/`, `tests/integration/` as an explicit `[[test]]` target (`path = "..."`) — not Cargo's automatic `tests/*.rs` discovery, which only scans a package's immediate `tests/` directory, not subdirectories.
- Is created once, fully, in Foundational (T018) with every target pre-declared against a trivially-passing stub body, so no later story ever re-edits `tests/Cargo.toml` itself — each story only replaces the body of its own already-declared, uniquely-named stub file(s), which keeps every later story's test-writing task genuinely single-file and parallel-safe.
- `tests/e2e/` is a separate, ordinary **pnpm** package (`@converge/e2e`, WebdriverIO), not part of this Cargo package — it is TypeScript, not Rust.
- **Corrected this pass**: `converge-tests` also depends on `adapters-persistence` (path) and `sqlx.workspace = true` (T018) — its own migration/persistence/transaction integration tests need the real adapter and a real SQLite connection, which the first revision omitted.

## Flagged Items for Human Confirmation (this pass)

Per this pass's explicit instruction to report, not silently invent, an unresolved normative dependency or mechanism decision, two items are recorded here rather than resolved unilaterally:

1. **Orphan-process/launcher-shutdown PID capture and Windows liveness probe** (T025, T026, T038). `research.md`/`plan.md` describe the liveness probe primarily in Node.js's own API terms (`process.kill(pid, 0)`) and never specify how the two child PIDs (Vite, Rust service) reach an external test process. This revision assumes: (a) `scripts/dev.mjs` (T038) prints an additional, non-secret PID status line for test observability, and (b) the Rust integration test (T026) shells out to a one-line Node helper for the Windows liveness check specifically, keeping both tests inside the `converge-tests` Cargo package rather than moving them to a Node-hosted test. This is a reasonable, low-risk, dependency-free assumption, not a silent invention of new protocol — but it was not explicitly specified by either normative document, and a human maintainer should confirm it (or redirect these two tests to a Node-hosted implementation) before T025/T026/T038 are implemented.
2. **`futures-util` gap for WebSocket tests** (T018, T052–T054). Driving `tokio-tungstenite`'s `WebSocketStream` ergonomically (`.next()`/`.send()`) needs `futures_util::{StreamExt, SinkExt}` in scope. `research.md` pins `tokio-tungstenite` but never pins `futures-util` (or the `futures` umbrella crate), and this pass does not add it, per the explicit instruction not to invent or silently pin a new package. Recommend amending `research.md` to add an exact `futures-util` pin (matching `tokio-tungstenite`'s dependency line) or confirming an alternative mechanism, before T052–T054 are implemented.

Every other correction this pass was resolved directly — see the revision note above and the Notes section at the end of this file.

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
- [ ] T002 [P] Create root Cargo workspace manifest in `Cargo.toml`: `[workspace] members = ["crates/domain", "crates/application", "crates/contracts", "crates/adapters-http", "crates/adapters-persistence", "crates/service", "tests"]` (seven members — six production crates plus the test harness, see "Test Harness Package" above) and a `[workspace.dependencies]` table pinning every Rust dependency this slice uses, exactly per `research.md`/`plan.md` Technical Context: `axum = "=0.8.9"` (feature `"ws"` added per-crate), `tower-http = "=0.7.0"` (features `"cors"`, `"trace"` added per-crate), `tokio = "=1.53.1"` (features added per-crate), `sqlx = "=0.9.0"` (features `"sqlite"`, `"runtime-tokio"`, `"macros"`, `"migrate"` added per-crate), `ts-rs = "=12.0.1"`, `rand = "=0.10.2"`, `thiserror = "=2.0.19"`, `uuid = "=1.24.0"` (default features at the workspace level; the `"v7"` feature is added only on the test-harness package's own dependency edge, T018 — `application` never enables it, per `data-model.md`'s Event identity section), `blake3 = "=1.8.5"`, `serde = "=1.0.229"` (feature `"derive"` added per-crate), `serde_json = "=1.0.151"`, `time = "=0.3.54"` (features added per-crate), `base64 = "=0.22.1"`, `reqwest = "=0.13.4"` (dev/test-only), `tokio-tungstenite = "=0.30.0"` (dev/test-only) — this task pins dependencies only; it does not itself implement any behavioral requirement (Constitution I)
- [ ] T003 [P] Pin Rust toolchain (`channel = "1.97.1"`, `components = ["rustfmt", "clippy"]`) in `rust-toolchain.toml`
- [ ] T004 [P] Initialize pnpm workspace: `pnpm-workspace.yaml` (`packages: ["apps/web", "packages/contracts", "tests/e2e"]`), root `package.json` (`"packageManager": "pnpm@11.15.1"`, `engines.node`), `.nvmrc` (`24.18.0`)
- [ ] T005 [P] Scaffold `apps/web` with Vite + React + TypeScript per `research.md` pins (`react`/`react-dom` 19.2.8, `vite` 8.1.5, `@vitejs/plugin-react` 6.0.4, `@types/react` 19.2.17, `@types/react-dom` 19.2.3, `typescript` 6.0.3) plus its production state-management dependencies (`@tanstack/react-query` 5.101.4, `zustand` 5.0.14) and its workspace dependency on the generated contracts package (`"@converge/contracts": "workspace:*"`) — `apps/web/package.json` (sole owner of this file's `@converge/contracts` dependency line — see File-Ownership Map), `apps/web/tsconfig.json`, `apps/web/index.html`, `apps/web/src/main.tsx` (empty entry point; populated by T041/T043/T045/T062)
- [ ] T006 [P] Scaffold `packages/contracts` as a pnpm workspace package (`@converge/contracts`) — `packages/contracts/package.json` (no dependency on/edit of `apps/web/package.json` — see File-Ownership Map), `packages/contracts/src/generated/.gitkeep`, `packages/contracts/src/index.ts` (hand-written barrel; re-exports everything from `./generated`; never itself regenerated — FR-020; first of three sequential appenders to this file, see File-Ownership Map)
- [ ] T007 [P] Configure ESLint flat config + Prettier per `research.md` pins (`eslint` 10.7.0, `eslint-config-prettier` 10.1.8, `typescript-eslint` 8.65.0, `eslint-plugin-react-hooks` 7.1.1, `prettier` 3.9.6) — `eslint.config.js`, `.prettierrc`
- [ ] T008 Configure Vitest + React Testing Library + jsdom + jest-axe per `research.md` pins (`vitest` 4.1.10, `@testing-library/react` 16.3.2, `@testing-library/dom` 10.4.1, `jsdom` 29.1.1, `jest-axe` 10.0.0) — **corrected this pass: `@testing-library/jest-dom` removed (no exact pin exists for it anywhere in `research.md`; do not add it), and `jest-axe` is wired via ESM `import`, not `require`** — `apps/web/package.json` devDependencies (depends on T005 — same file, sequential append-only edit, not parallel; see File-Ownership Map), `apps/web/vitest.config.ts` (`environment: "jsdom"`, `setupFiles: "./vitest.setup.ts"`), `apps/web/vitest.setup.ts` (`import { toHaveNoViolations } from "jest-axe"; expect.extend(toHaveNoViolations);` — no `@testing-library/jest-dom` import; frontend assertions elsewhere in this slice use plain Testing-Library/DOM queries and Vitest's built-in `expect`, not jest-dom-specific matchers)
- [ ] T009 [P] Scaffold the WebdriverIO E2E harness as its own pnpm package (`@converge/e2e`) per `research.md` pins (`webdriverio` 9.30.0, `@wdio/cli` 9.30.0, `@wdio/local-runner` 9.30.0, `@wdio/mocha-framework` 9.30.0, `@wdio/spec-reporter` 9.29.1) — `tests/e2e/package.json`, `tests/e2e/wdio.conf.ts` (automatic browser/driver management per `research.md`, no manual driver provisioning — supports `just check`'s unconditional `test-e2e` step)
- [ ] T010 Create `justfile` with every recipe name from `plan.md`'s Command Facade table (`setup`, `dev`, `fmt`, `fmt-check`, `lint`, `test-unit`, `test-contract`, `test-integration`, `test-frontend`, `test-e2e`, `build`, `check`, `contracts-generate`, `contracts-check`) as skeletons wired to their underlying tool, with `test-unit`/`test-contract`/`test-integration` skeletons already targeting the `converge-tests` package by exact `--test <name>` flags (real bodies filled by T051, T079, T080 — `justfile` is a shared file across those three tasks; edits MUST be applied in that sequence regardless of which stories otherwise run in parallel, see Dependencies & Execution Order)
- [ ] T011 Verify Constitution compliance (Principles I–X) for this task breakdown against `specs/001-slice-0-foundation/plan.md`'s Constitution Check — Post-Design Gate before implementation begins; record any deviation here before proceeding

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Crate-level scaffolding every user story depends on.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T012 [P] Configure `crates/domain/Cargo.toml` — normal-dependency allowlist `{thiserror.workspace = true}` only; no serde, I/O, framework, or async
- [ ] T013 [P] Configure `crates/application/Cargo.toml` — normal-dependency allowlist `{domain (path), thiserror.workspace = true, uuid.workspace = true (default features only — never `"v7"`), blake3.workspace = true, serde_json.workspace = true, time.workspace = true (feature `"parsing"`)}`; `tokio.workspace = true` only as a dev-dependency (for `#[tokio::test]`-based unit tests). Note: `application`'s outbound ports use Rust's native `async fn` in traits (T073) — this requires no `tokio` normal dependency to declare or call, only to execute; `application` itself never runs an executor
- [ ] T014 [P] Configure `crates/contracts/Cargo.toml` — normal-dependency allowlist `{serde.workspace = true (feature "derive"), ts-rs.workspace = true}`; no dependency on `domain`/`application`
- [ ] T015 [P] Configure `crates/adapters-http/Cargo.toml` — allowlist `{application (path), domain (path), contracts (path), axum.workspace = true (feature "ws"), tower-http.workspace = true (features "cors","trace"), tokio.workspace = true, thiserror.workspace = true, serde_json.workspace = true, time.workspace = true (features "formatting","parsing","serde")}`
- [ ] T016 [P] Configure `crates/adapters-persistence/Cargo.toml` — allowlist `{application (path), domain (path), sqlx.workspace = true (features "sqlite","runtime-tokio","macros","migrate"), tokio.workspace = true, thiserror.workspace = true}`; no `serde_json`
- [ ] T017 Configure `crates/service/Cargo.toml` — composition root, depends on all five other crates plus `rand.workspace = true`, `base64.workspace = true`, **and its own direct `tokio.workspace = true` dependency** (features `"rt-multi-thread"`, `"macros"`, `"signal"`, `"process"`, `"io-std"` — the async runtime `service`'s own `main.rs` needs for the Axum server, the `SIGINT`/`SIGTERM` handlers, and the stdin-EOF watchdog; missing from the previous draft of this task list). No allowlist restriction applies to `service` itself (the only crate exempt from the per-crate allowlist check, per `plan.md` Project Structure)
- [ ] T018 [P] Create `tests/Cargo.toml` — the test-harness package (`converge-tests`, workspace member #7, no Clean Architecture allowlist restriction; see "Test Harness Package" above). Normal dependencies: `domain`/`application`/`contracts`/**`adapters-persistence`** (path, for white-box assertions where needed — **`adapters-persistence` added this pass**: T068/T069/T070's schema, trigger, repository, and real-transaction tests exercise the actual persistence adapter and a live SQLite database, which the previous revision omitted entirely from this package's dependencies), `reqwest.workspace = true`, `tokio-tungstenite.workspace = true`, `tokio.workspace = true` (full features), `serde_json.workspace = true`, `time.workspace = true` (feature `"parsing"`), **`sqlx = { workspace = true, features = ["sqlite", "runtime-tokio", "macros", "migrate"] }`** (added this pass — same exact features as `adapters-persistence`'s own pin, T016; needed to run migrations and drive real transactions against a test database), and `uuid = { workspace = true, features = ["v7"] }` — the `"v7"` generation feature belongs on this package's dependency edge specifically, since `data-model.md`'s Event identity section names "the integration test suite" as the producer that generates `event_id`. **Flagged, not resolved this pass** (see "Flagged Items for Human Confirmation" above): T052–T054's WebSocket tests drive `tokio-tungstenite`'s `Stream`/`Sink` API, which needs `futures_util::{StreamExt, SinkExt}` — `futures-util` has no exact pin in `research.md` and is deliberately **not** added here. Declare one `[[test]]` target per eventual test file, each with a trivially-passing stub body created in this same task: `allowlist_check` (`architecture/allowlist_check.rs`), `http_health_contract` (`contract/http_health_contract.rs`), `websocket_proof_contract` (`contract/websocket_proof_contract.rs`), `http_auth_denial`, `http_cors_preflight`, `service_restart`, `concurrent_instances`, `launcher_shutdown`, `orphan_process`, `redaction`, `ws_auth_denial`, `ws_redaction`, `migrations`, `idempotency_conflict`, `transactional_rebuild` (all under `integration/`). `tests/architecture/fixtures/violation_metadata.json` is a data fixture, not a compiled target — created empty here, populated by T050.
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
- [ ] T025 [P] [US1] Integration test: cooperative launcher shutdown — parse both children's PIDs from T038's status line, send the supervisor `SIGINT`/`SIGTERM`, assert both captured PIDs exit within the bounded timeout — in `tests/integration/launcher_shutdown.rs` (FR-001, FR-018)
- [ ] T026 [P] [US1] Integration test: orphan-process — parse both children's PIDs from T038's status line, then terminate the supervisor abruptly via Rust `std::process::Child::kill()` (sends `SIGKILL` on POSIX; calls `TerminateProcess` on Windows — std's own portable, uncatchable primitive, no new dependency needed), then confirm the two captured child PIDs are exited via a portable liveness probe: POSIX, shell out to `kill -0 <pid>` via `std::process::Command` (nonzero/`ESRCH` exit means not running); Windows, shell out to a one-line Node helper invoking Node's own `process.kill(pid, 0)` (`research.md`'s explicitly named mechanism) via `std::process::Command::new("node").arg("-e")` — **replacing the previous draft's invented `tasklist /FI ... /NH` empty-output parsing, which had no basis in either normative document (correction 4, this pass)**. Both PIDs, captured before the kill signal, MUST be independently confirmed exited — a later successful `just dev` run is explicitly NOT accepted as evidence — in `tests/integration/orphan_process.rs` (FR-001, FR-018; see "Flagged Items for Human Confirmation" above for the residual mechanism-language ambiguity between `research.md`'s Node-flavored wording and this Rust-hosted implementation)
- [ ] T027 [P] [US1] Redaction test: negative-grep over captured logs/process output — the token never appears in a URL-shaped, header-echoed, or plain-text log line — in `tests/integration/redaction.rs` (FR-017, SEC-008)

### Implementation for User Story 1

- [ ] T028 [US1] Define `HealthResponse`/`HealthStatus`/`ApiError` Rust types with exact Serde/`ts-rs` attributes (FR-019) in `crates/contracts/src/health.rs` (depends on T014, T019; makes T020 pass)
- [ ] T029 [US1] Run `just contracts-generate` to produce `packages/contracts/src/generated/health.ts` from `crates/contracts/src/health.rs` and add its re-export to `packages/contracts/src/index.ts` (T006's hand-written barrel — second of three sequential appenders, after T006 and before T056) — the previous task list referenced generated frontend types before any task ever produced them; this closes that gap (depends on T028; FR-019, FR-020)
- [ ] T030 [US1] Implement ephemeral token generation (`rand::OsRng`, base64url no-pad via `base64 =0.22.1`) in `crates/service/src/token.rs` (FR-003)
- [ ] T031 [US1] Implement loopback bind `127.0.0.1:0`, `CONVERGE_ALLOWED_ORIGIN` env read, and the `CONVERGE_BOOTSTRAP` stdout line in `crates/service/src/main.rs` (FR-001, FR-002, SEC-003)
- [ ] T032 [US1] Implement `Host`→`Origin`→token validation middleware, first-failure-wins, in `crates/adapters-http/src/middleware/auth.rs` (FR-004, FR-006, SEC-003; makes T021 pass)
- [ ] T033 [US1] Implement closed-CORS preflight handling (4-row check order, exact 3-header grant, `Access-Control-Allow-Origin` on the actual `200` response too) in `crates/adapters-http/src/middleware/cors.rs` (FR-005, SEC-003; makes T022 pass)
- [ ] T034 [US1] Implement `GET /api/health` handler in `crates/adapters-http/src/routes/health.rs` (FR-007)
- [ ] T035 [US1] Wire the Axum router (health route + middleware stack) in `crates/service/src/main.rs` (depends on T032, T033, T034)
- [ ] T036 [US1] Implement the stdin-pipe EOF watchdog (background `tokio` task; EOF triggers the same shutdown path as `SIGTERM`) in `crates/service/src/main.rs` (`research.md` orphan-process mechanism; makes T026 pass)
- [ ] T037 [US1] Implement graceful shutdown on `SIGINT`/`SIGTERM` (close listener, bounded drain, exit) in `crates/service/src/main.rs` (FR-018; makes T025 pass)
- [ ] T038 [P] [US1] Implement `scripts/dev.mjs` supervisor: fork Vite first, spawn the Rust service with `CONVERGE_ALLOWED_ORIGIN`, forward port/token over IPC, print a redacted status line — **plus a separate, non-secret line reporting both spawned children's OS PIDs (Vite, Rust service), added this pass so T025/T026's integration tests have a documented way to capture the exact PIDs they must assert on (a low-risk implementation-detail assumption, not specified verbatim by `research.md`/`plan.md` — see "Flagged Items for Human Confirmation" above)** — cooperative shutdown, and the Vite child's `'disconnect'`-triggered teardown (FR-001, `research.md` Bootstrap Ordering)
- [ ] T039 [P] [US1] Implement the Vite dev-bootstrap middleware — `Host`-validated, same-origin, in-memory port/token cache, `503` while pending — in `apps/web/vite.config.ts` (`research.md` Vite bootstrap endpoint hardening)
- [ ] T040 [P] [US1] Implement Graphite Signal design tokens — primitives → semantic → component CSS Custom Properties (`--cv-{category}-{role}-{state?}` naming, `DESIGN.md` §3–§4.3) for both Light and Dark themes, plus typed TypeScript accessors — in `apps/web/src/styles/tokens.css` and `apps/web/src/styles/tokens.ts` (FR-023, PLT-011, Constitution VII)
- [ ] T041 [US1] Implement theme selection (system-default, explicit Light/Dark override, persisted preference, no prolonged flash on switch, per `DESIGN.md` §14) in `apps/web/src/state/themeStore.ts` (Zustand — transient UI/layout state only, FR-024) and wire theme initialization in `apps/web/src/main.tsx` (depends on T040; shares `main.tsx` with T043/T045/T062 — sequential, not parallel; see File-Ownership Map)
- [ ] T042 [P] [US1] Implement `prefers-reduced-motion` handling — remove displacement/parallax/sequenced motion, use a short crossfade or instant change instead, per `DESIGN.md` §10 — in `apps/web/src/styles/motion.css` (FR-023, PLT-011)
- [ ] T043 [US1] Implement the QueryClient bootstrap — `QueryClient` instance and `QueryClientProvider` mounting — in `apps/web/src/lib/api/queryClient.ts` and `apps/web/src/main.tsx` (depends on T041 — same file, sequential; FR-024)
- [ ] T044 [US1] Implement the typed HTTP client and TanStack Query hook for health, against the generated `HealthResponse` type, in `apps/web/src/lib/api/health.ts` (depends on T029, T043; FR-008, FR-024)
- [ ] T045 [US1] Implement the health/status frontend surface (Initial loading, Ready, Refreshing/Stale, Service unavailable/Offline, Error), consuming only `--cv-*` tokens (T040) and the reduced-motion rules (T042), in `apps/web/src/features/health/HealthStatus.tsx`, mounted in `apps/web/src/main.tsx` (depends on T040, T042, T044, and T043 — same file, sequential; FR-023, spec.md Frontend State Matrix)
- [ ] T046 [US1] Frontend tests: one test group per applicable Frontend State Matrix state, plus keyboard/focus/non-color-state and `jest-axe` assertions (plain Testing-Library/DOM assertions, not jest-dom matchers — T008), in `apps/web/src/features/health/HealthStatus.test.tsx` (depends on T045; FR-023, SC-010)
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

- [ ] T051 [US2] Wire `cargo test --workspace --lib` (isolated unit-test compile of every crate, `domain`/`application` included) and `cargo test -p converge-tests --test allowlist_check` into the `justfile` `test-unit` recipe (FR-009, FR-010; depends on T049; `justfile` is shared with T010/T079/T080 — this edit applies second in that sequence)

**Checkpoint**: User Story 2 is independently functional and testable (SC-005) — `domain`/`application` compile in isolation, and the check fails visibly on a deliberate violation.

---

## Phase 5: User Story 5 - Minimal Authenticated WebSocket Proof (Priority: P2)

**Goal**: A WebSocket handshake is accepted only with a valid token, matching `Host`, and an allowlisted `Origin`, checked before any application-level exchange; the connection carries only a minimal `hello`/`ping`/`pong` proof.

**Independent Test**: Attempt a handshake with valid token/`Host`/allowlisted `Origin` (expect accept), then repeat with each invalidated individually (expect refusal before any message is exchanged) — spec.md US5 Acceptance Scenarios 1–3.

**Note**: Shares `crates/adapters-http` and `crates/service`'s router/token infrastructure with User Story 1; the implementation tasks below depend on User Story 1's router wiring (T035) being complete, even though this story's *test* is independently executable per spec.md.

**Corrected this pass**: the previous revision defined the WebSocket route/handler (T057–T059) and a frontend client (T060, renumbered T061) but never registered the route into the running router and never had any task actually open the connection from the app — so `just dev` could never produce the `hello` exchange the E2E test (T063) is supposed to observe. Two tasks close that gap: T060 (router registration) and T062 (frontend consumer/bootstrap).

### Tests for User Story 5 (MANDATORY — Constitution Principle VI, security boundary + contract) ⚠️

- [ ] T052 [P] [US5] Contract test: `WsMessage` (`hello`/`ping`/`pong`) wire shapes and subprotocol negotiation response (only `converge.v1` echoed back) in `tests/contract/websocket_proof_contract.rs` (`converge-tests`, replacing its T018 stub) (FR-021, SC-009)
- [ ] T053 [P] [US5] Integration test: WS handshake denial — `Host` mismatch (400), `Origin` not in the allowlist (403), no valid token subprotocol (401), individually and in combination, never returns `101` — in `tests/integration/ws_auth_denial.rs` (FR-022, SEC-001, SEC-003, SC-009)
- [ ] T054 [P] [US5] Negotiation/redaction test: the token-bearing subprotocol value never appears in the handshake response header or in any captured log line — in `tests/integration/ws_redaction.rs` (FR-017)

### Implementation for User Story 5

- [ ] T055 [US5] Define the `WsMessage` enum (`Hello`/`Ping`/`Pong`, `tag = "type"`, `camelCase` fields) in `crates/contracts/src/ws.rs` (depends on T014, T019; makes T052 pass)
- [ ] T056 [US5] Run `just contracts-generate` to produce `packages/contracts/src/generated/ws.ts` from `crates/contracts/src/ws.rs` and add its re-export to `packages/contracts/src/index.ts` (depends on T055, **and on T029 — both append to the shared `packages/contracts/src/index.ts` barrel; T029 MUST land first, corrected this pass, see File-Ownership Map**; FR-019, FR-020)
- [ ] T057 [US5] Implement the WebSocket upgrade route `GET /api/ws` — `Host`→`Origin`-allowlist→token-subprotocol order, before any `101` response — in `crates/adapters-http/src/routes/ws.rs` (FR-022; depends on T035; makes T053 pass)
- [ ] T058 [US5] Implement subprotocol negotiation (server selects only `converge.v1` in its response) in `crates/adapters-http/src/routes/ws.rs` (`research.md` Subprotocol negotiation; makes T054 pass)
- [ ] T059 [US5] Implement the `hello`/`ping`/`pong` exchange handler (no terminal/PTY/provider streaming) in `crates/adapters-http/src/routes/ws.rs` (spec.md US5 Acceptance Scenario 3)
- [ ] T060 [US5] Register the `/api/ws` upgrade route into the Axum router in `crates/service/src/main.rs` — **new this pass**: the previous revision defined the route/handler (T057–T059) but never wired it into the actual running router, so no live WS connection existed for the frontend or E2E test to reach — in `crates/service/src/main.rs` (depends on T057, T058, T059, and T037 — same file as T031/T035/T036/T037, sequential, see File-Ownership Map; FR-022, SC-009)
- [ ] T061 [P] [US5] Implement the frontend WebSocket client (`converge.v1` + `converge.token.<token>` offer), against the generated `WsMessage` type, opened after the health bootstrap, in `apps/web/src/lib/api/ws.ts` (depends on T056; FR-008)
- [ ] T062 [US5] Implement the frontend WebSocket-proof bootstrap: after the health surface reaches Ready (T045), open the authenticated connection via T061's client, and on receiving `hello`, record a plain, non-secret, test-observable readiness signal (e.g., a stable `data-testid` element/attribute) — **new this pass**: the previous revision created the WS client (T061) but no task ever consumed it, so `just dev` never actually established the connection `quickstart.md`'s Launch section already describes the frontend performing — in `apps/web/src/lib/api/wsBootstrap.ts` and `apps/web/src/main.tsx` (depends on T045, T060, T061 — `main.tsx` shared with T041/T043/T045, sequential, see File-Ownership Map; FR-008, FR-016)
- [ ] T063 [US5] Extend the browser E2E journey to confirm the WS proof (`hello` received, observed via T062's test-observable signal) — appended to `tests/e2e/health.e2e.ts` (depends on T048, T062 — same file as T048, sequential, not parallel) (FR-016)

**Checkpoint**: User Story 5 is independently functional and testable (SC-009) — the WebSocket boundary is proven without terminal or provider code existing, and the app actually establishes the connection the E2E test observes.

---

## Phase 6: User Story 3 - Idempotent Event Persistence and Aggregate Rebuild (Priority: P3)

**Goal**: A representative normalized event persists idempotently — resubmission never duplicates the ledger — and the derived aggregate can be discarded and rebuilt from the ledger alone without mutating it.

**Independent Test**: Submit a representative event twice, confirm one ledger row and one equivalent aggregate; discard the aggregate, rebuild it from the ledger, confirm equivalence (spec.md US3 Acceptance Scenarios 1–3).

**Clean Architecture note (corrected, first pass; async-corrected, this pass)**: `RebuildAggregate` (application) MUST contain no SQL, transaction keywords, or `sqlx` types. It orchestrates exclusively through an explicit, transaction-owning `AggregateRepository::rebuild` port method (T073) that accepts a pure fold function (T074) as a parameter. `BEGIN IMMEDIATE`, `DELETE`, `SELECT`, `INSERT`, `COMMIT`/`ROLLBACK`, and every `sqlx` type are owned exclusively by `adapters-persistence` (T078), which calls the injected pure function *inside* its own open transaction. **Corrected this pass**: the port method is now a native `async fn` in a trait (stable since Rust 1.75, present in Edition 2024, no `async-trait` crate or other unapproved async-helper dependency) rather than the synchronous signature the first revision used — a synchronous trait method could not have been genuinely implemented by `adapters-persistence`'s inherently async SQLx pool (`runtime-tokio`) without blocking or an undocumented workaround. Call sites use static/generic dispatch (`impl AggregateRepository` / a generic type parameter), never `dyn AggregateRepository`, since native async-fn-in-trait is not `dyn`-compatible without additional boxing this slice does not need.

### Tests for User Story 3 (MANDATORY — Constitution Principle VI, domain logic + application use cases + persistence boundary) ⚠️

- [ ] T064 [P] [US3] Domain unit test: Normalized Event / Aggregate invariants in `crates/domain/src/event.rs` and `crates/domain/src/aggregate.rs` (test modules)
- [ ] T065 [P] [US3] Application unit test: `RecordProbeEvent` — first submission succeeds; identical resubmission is idempotent (no duplicate); resubmission differing in any field returns `IdempotencyConflict` naming the exact `ConflictingFields` — in `crates/application/src/use_cases/record_probe_event.rs` (test module) (FR-012)
- [ ] T066 [P] [US3] Application unit test: the **pure** `fold_events` function — given an in-memory slice of `Event` values (no I/O, no mock repository needed), asserts correct per-`event_type` projection counts and `last_event_id` — in `crates/application/src/use_cases/rebuild_aggregate.rs` (test module) (FR-013)
- [ ] T067 [US3] Application unit test (`#[tokio::test]`): `RebuildAggregate` orchestration — using a fake/in-memory `AggregateRepository` port implementation whose `rebuild` method is itself a trivial `async fn` returning immediately, asserts the use case calls `repository.rebuild(fold_events)` exactly once, awaits it, and returns the port's result verbatim, and asserts (by construction — the use case has no SQL-capable dependency available to it) that it cannot itself issue any transaction or SQL statement — appended to the same `crates/application/src/use_cases/rebuild_aggregate.rs` (test module; same file as T066 — not parallel, corrected this pass: both were previously marked `[P]` despite sharing a file) (FR-013)
- [ ] T068 [P] [US3] Migration test: fresh empty database → migrations apply → expected schema/triggers exist; re-run is a no-op; `BEFORE UPDATE`/`BEFORE DELETE` triggers `RAISE(ABORT, ...)` — in `tests/integration/migrations.rs` (`converge-tests`, replacing its T018 stub) (FR-011, AC-042)
- [ ] T069 [P] [US3] Integration test: sequential and concurrent identical resubmission → exactly one row/aggregate; concurrent resubmission differing only in `event_type` / only in `payload` / only in `occurred_at` → each surfaces `IdempotencyConflict` — in `tests/integration/idempotency_conflict.rs` (FR-012, SC-003)
- [ ] T070 [P] [US3] Integration test: an injected mid-fold failure leaves `aggregates` unchanged (rollback), asserted against the real `adapters-persistence` transaction (this is the one test in this story that exercises the actual `BEGIN IMMEDIATE` behavior, confirming it — not `application` — owns it); a concurrent `events` insert blocks while a rebuild transaction is open — in `tests/integration/transactional_rebuild.rs` (FR-013, SC-004)

### Implementation for User Story 3

- [ ] T071 [US3] Write migration `0001_create_events_and_aggregates.sql` (`events`/`aggregates` tables, `payload_hash` column, `BEFORE UPDATE`/`BEFORE DELETE` immutability triggers) in `migrations/0001_create_events_and_aggregates.sql` (FR-011; makes T068 pass)
- [ ] T072 [US3] Define the Normalized Event / Aggregate domain entities in `crates/domain/src/event.rs`, `crates/domain/src/aggregate.rs` (makes T064 pass)
- [ ] T073 [US3] Define `EventRepository`/`AggregateRepository` outbound ports (traits) in `crates/application/src/ports.rs`, making the transaction-owning contract explicit and asynchronous — **corrected this pass**: the previous draft's `AggregateRepository::rebuild` was a synchronous trait method, incompatible with `adapters-persistence`'s inherently async SQLx implementation. Uses Rust's native `async fn` in traits (stable since Rust 1.75, present in Edition 2024 per `plan.md`'s Technical Context — no `async-trait` crate or other unapproved async-helper dependency): `pub trait AggregateRepository { async fn rebuild(&self, fold: impl Fn(&[Event]) -> Vec<AggregateProjection> + Send) -> Result<(), RepositoryError>; }` — the port's single method is the *only* place a rebuild can be triggered from `application`, and its signature accepts the pure fold logic as a parameter rather than exposing any SQL-shaped operation (`SELECT`/`DELETE`/`INSERT`) as separate port methods `application` could sequence itself. Call sites use static/generic dispatch, never `dyn AggregateRepository`, since native async-fn-in-trait is not `dyn`-compatible without boxing this slice does not need
- [ ] T074 [US3] Implement the **pure** `fold_events(events: &[Event]) -> Vec<AggregateProjection>` function — no I/O, no SQL, no port dependency; remains synchronous (only the port method and use case that call it are async, per T073's corrected signature) — in `crates/application/src/use_cases/rebuild_aggregate.rs` (makes T066 pass)
- [ ] T075 [US3] Implement the `RecordProbeEvent` use case — producer-supplied UUIDv7 `event_id` passthrough, canonical `serde_json` payload serialization, BLAKE3 `payload_hash`, RFC 3339 `occurred_at` parse/validate, three-outcome conflict classification — in `crates/application/src/use_cases/record_probe_event.rs` (depends on T073; makes T065 pass)
- [ ] T076 [US3] Implement the `RebuildAggregate` use case as an `async fn` that calls `repository.rebuild(fold_events).await` and returns its result verbatim; contains no SQL, no transaction keyword, no `sqlx` type — in `crates/application/src/use_cases/rebuild_aggregate.rs` (depends on T073, T074; makes T067 pass)
- [ ] T077 [P] [US3] Implement the SQLx `EventRepository` (`INSERT` + `UNIQUE`-conflict re-`SELECT`/classify, `async fn` methods using SQLx's async pool) in `crates/adapters-persistence/src/event_repository.rs` (depends on T071, T073; makes T069 pass; mutually parallel with T078 — different files, corrected this pass: the prose already claimed this parallelism but the checklist markers were missing it)
- [ ] T078 [P] [US3] Implement the SQLx `AggregateRepository::rebuild` as an `async fn` (matching T073's corrected async port signature) — owns `BEGIN IMMEDIATE` → `DELETE FROM aggregates` → `SELECT * FROM events ORDER BY id ASC` → calls the caller-supplied `fold` closure on the fetched events → `INSERT` per resulting projection → `COMMIT`/`ROLLBACK` — exclusively, using SQLx's async pool — in `crates/adapters-persistence/src/aggregate_repository.rs` (depends on T071, T073; makes T070 pass; mutually parallel with T077 — different files)
- [ ] T079 [US3] Pin `sqlx-cli` install (`cargo install sqlx-cli --version 0.9.0 --locked`) in the `justfile` `setup` recipe, and **generate/update** (not commit) the `.sqlx/` offline query cache via `cargo sqlx prepare` against a running local dev database — staging and committing `crates/adapters-persistence/.sqlx/` is left entirely to the human maintainer (Constitution X) — `justfile`, `crates/adapters-persistence/.sqlx/` (generated artifact, not staged/committed by this task; `justfile` is shared with T010/T051/T080 — this edit applies third in that sequence)

**Checkpoint**: User Story 3 is independently functional and testable (SC-003, SC-004) — the ledger is idempotent and the aggregate is rebuildable without mutating it, and `application` contains no SQL anywhere in the rebuild path.

---

## Phase 7: User Story 4 - Command-Facade Quality Gates and Base CI (Priority: P4)

**Goal**: Formatting, linting, tests, and builds run identically through the `justfile` locally and in CI on every pull request and push, on Linux fully and Windows x86_64 for build plus applicable technical tests.

**Independent Test**: Run the documented format/lint/test/build recipes locally and confirm they pass; confirm the same recipes run in CI, and a deliberately broken check fails CI visibly (spec.md US4 Acceptance Scenarios 1–3).

**Note**: This story verifies the outcomes of User Stories 1, 2, 3, and 5 rather than introducing new runtime behavior — its tasks depend on those stories' recipes (`test-unit`, `test-contract`, `test-integration`, `test-frontend`, `test-e2e`) having real content to run.

### Implementation for User Story 4

- [ ] T080 [US4] Complete every `justfile` recipe to its full defined behavior per `plan.md`'s Command Facade table, with `check` as the unconditional complete gate (including `test-e2e`, no environment-dependent skip); `test-contract` runs `cargo test -p converge-tests --test http_health_contract --test websocket_proof_contract` plus `just contracts-check`; `test-integration` runs `cargo test -p converge-tests` filtered to every `integration/*` target listed in T018; `test-frontend` runs `pnpm --filter @converge/web test`; `test-e2e` runs `pnpm --filter @converge/e2e wdio run wdio.conf.ts` — `justfile` (FR-014; depends on T051, T079, and every User Story 1/3/5 test/implementation task; this is the fourth and final sequential edit to `justfile`)
- [ ] T081 [P] [US4] Add `.github/workflows/ci.yml` with **both** CI jobs in this one file: a **Linux job** running `just check` unconditionally on `ubuntu-24.04`, and a **Windows job** running `just fmt-check`, `just lint`, `just test-unit`, `just test-contract`, `just test-integration`, `just build` on `windows-2022` (no E2E step, per `PLT-005`) — both triggered on `pull_request` and `push` to the default branch, using `actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`, `actions/setup-node@820762786026740c76f36085b0efc47a31fe5020 # v7.0.0` (`node-version-file: .nvmrc`), `dtolnay/rust-toolchain@2c7215f132e9ebf062739d9130488b56d53c060c # 2026-07-16` (`toolchain: "1.97.1"`, `components: "rustfmt, clippy"`) (FR-015, PLT-005, PLT-009, SC-006)
- [ ] T082 [P] [US4] Add a Dependabot/Renovate config to keep the GitHub Actions SHA pins current under human review in `.github/dependabot.yml` (`research.md` GitHub Actions Enforcement)
- [ ] T083 [US4] Document the manual, human-performed CI-break verification runbook — a human maintainer pushes a throwaway branch/PR with one deliberately failing check, confirms `just check` and CI both fail visibly, then closes/deletes the throwaway branch — appended to `specs/001-slice-0-foundation/quickstart.md` under a new "Manual CI-Break Verification (human-performed)" heading. **This task is documentation only; the agent completing it MUST NOT execute `git push`, open a pull request, commit a deliberate break, or revert Git history** (Constitution X, AGENTS.md Git and Release Boundaries) (spec.md US4 Acceptance Scenario 3, SC-007)

**Checkpoint**: User Story 4 is independently functional and testable (SC-006, SC-007) — contributors and CI share the identical gate, and nothing passes silently.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Final, whole-slice verification after every story is complete.

- [ ] T084 [P] Run `just contracts-check` end-to-end and confirm zero diff between regenerated and committed TypeScript in `packages/contracts/src/generated/` (FR-020, SC-008)
- [ ] T085 [P] Confirm `just fmt-check` and `just lint` pass repository-wide (FR-014)
- [ ] T086 Run `specs/001-slice-0-foundation/quickstart.md` validation end-to-end (Launch, Security Boundary, Persistence, Architecture Boundaries, Complete Gate) and, separately, confirm T083's manual CI-break runbook document exists and accurately describes the human-performed procedure. **Corrected this pass**: this task validates only that the runbook is present and correct — the agent performing it MUST NOT execute the intentionally failing Git/CI workflow (no push, no PR, no revert); any result of that human-performed verification may be recorded in this task's report only when supplied by a human maintainer, never inferred, simulated, or fabricated by the agent. Any claim about local staged/committed/pushed state in this task's report MUST cite the current `git status` output as evidence, not be asserted from memory (FR-014, FR-016, FR-018; SC-006, SC-007)
- [ ] T087 [P] Update `README.md` with the one documented launch command, only if it differs from `quickstart.md`'s `just dev` (FR-014)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately.
- **Foundational (Phase 2)**: Depends on Setup — BLOCKS all user stories.
- **User Stories (Phases 3–7)**: All depend on Foundational completion.
  - **US1 (Phase 3, P1)**: No dependency on any other story.
  - **US2 (Phase 4, P2)**: No dependency on any other story — touches only `tests/architecture/` and the `justfile` (second sequential edit).
  - **US5 (Phase 5, P2)**: Independently *testable* per spec.md, but its implementation tasks (T057–T060, T062) depend on US1's router wiring (T035, T037) because they live in `crates/adapters-http`/`crates/service`, and T062 depends on US1's `HealthStatus` mount (T045) because it shares `apps/web/src/main.tsx`.
  - **US3 (Phase 6, P3)**: No dependency on US1/US2/US5 — touches `crates/domain`, `crates/application`, `crates/adapters-persistence`, `migrations/`, and the `justfile` (third sequential edit).
  - **US4 (Phase 7, P4)**: Depends on US1, US2, US3, and US5 all being complete — it wires and verifies the gates that exercise their recipes (`justfile`'s fourth and final sequential edit); ordered last per spec.md ("verifies the outcomes of Stories 1–3 rather than introducing new runtime behavior").
- **Polish (Phase 8)**: Depends on every desired user story being complete.

### File-Ownership Map (for parallel safety)

| Story | Files it owns |
| --- | --- |
| US1 | `crates/contracts/src/health.rs`, `packages/contracts/src/generated/health.ts` + its line in `src/index.ts`, `crates/service/*` (router/token/main.rs — shared with US5's T060, see below), `crates/adapters-http/src/middleware/*`, `crates/adapters-http/src/routes/health.rs`, `scripts/dev.mjs`, `apps/web/vite.config.ts`, `apps/web/src/styles/*`, `apps/web/src/state/themeStore.ts`, `apps/web/src/lib/api/queryClient.ts`, `apps/web/src/lib/api/health.ts`, `apps/web/src/features/health/*`, `apps/web/src/main.tsx` (sequential internal edits T041→T043→T045, then US5's T062 — see below), `tests/integration/http_*.rs`, `tests/integration/launcher_shutdown.rs`, `tests/integration/orphan_process.rs`, `tests/integration/redaction.rs`, `tests/integration/concurrent_instances.rs`, `tests/integration/service_restart.rs`, `tests/contract/http_health_contract.rs`, `tests/e2e/health.e2e.ts` (extended by US5's T063) |
| US2 | `tests/architecture/*` |
| US5 | `crates/contracts/src/ws.rs`, `packages/contracts/src/generated/ws.ts` + its line in `src/index.ts` (after US1's T029 — see below), `crates/adapters-http/src/routes/ws.rs`, `crates/service/src/main.rs` (T060, after US1's T037 — see below), `apps/web/src/lib/api/ws.ts`, `apps/web/src/lib/api/wsBootstrap.ts`, `apps/web/src/main.tsx` (T062, after US1's T045 — see below), `tests/integration/ws_*.rs`, `tests/contract/websocket_proof_contract.rs`, `tests/e2e/health.e2e.ts` (T063, after US1's T048) |
| US3 | `crates/domain/src/event.rs`, `crates/domain/src/aggregate.rs`, `crates/application/src/use_cases/*`, `crates/application/src/ports.rs`, `crates/adapters-persistence/src/*`, `migrations/*`, `tests/integration/migrations.rs`, `tests/integration/idempotency_conflict.rs`, `tests/integration/transactional_rebuild.rs` |
| US4 | `.github/workflows/ci.yml` (single task, T081 — not split across two falsely-parallel tasks), `.github/dependabot.yml`, `specs/001-slice-0-foundation/quickstart.md` (T083 addition) |

**Cross-story shared files (not disjoint — explicit sequencing required, not parallel):**

- `justfile`: created once in Setup (T010, skeleton), then edited sequentially by US2 (T051), US3 (T079), and US4 (T080) in that exact order, regardless of which stories otherwise run in parallel.
- `tests/Cargo.toml`: created once, fully, in Foundational (T018) with every `[[test]]` target pre-declared; no story ever edits this file again — each story only replaces the body of its own already-declared stub `.rs` file(s), which restores per-story parallel safety on the individual test files.
- `apps/web/package.json`: T005 is the sole owner of its `@converge/contracts` workspace dependency line; T008 only appends devDependencies to the same file (append-only, no conflicting edit — the two are sequenced, T008 depends on T005, and only T005 carries `[P]`).
- `packages/contracts/src/index.ts`: created by T006 (Setup, hand-written barrel), then appended sequentially by US1's T029 and US5's T056, in that exact order — **corrected this pass**: T056 previously declared a dependency only on T055 (same story), not on T029, even though both append to this same file.
- `apps/web/src/main.tsx`: edited sequentially within US1 by T041 (theme init) → T043 (QueryClient provider) → T045 (mount `HealthStatus`), then by US5's T062 (WS bootstrap) — **corrected this pass**: T062 is new and closes the gap where nothing consumed the WS client; none of the four is marked `[P]` against the others.
- `crates/service/src/main.rs`: edited sequentially within US1 by T031 → T035 → T036 → T037, then by US5's T060 (register `/api/ws`) — **corrected this pass**: T060 is new and closes the gap where the WS route was never wired into the running router.

### Within Each User Story

- Per Constitution Principle VI: tests for domain logic, application use cases, contracts, and security boundaries MUST be written and FAIL before their paired implementation task (Red-Green-Refactor); frontend/adapter/infrastructure tests are included in the same change without requiring test-first ordering.
- Contracts/wire types before the routes that use them; `contracts-generate` before any frontend code imports the generated type.
- Ports/traits before the repositories that implement them and before the use cases that call them.
- Router wiring before route handlers that register into it, and before any frontend consumer that depends on the route being live.
- Story complete (checkpoint) before its dependents (US4) begin.

### Parallel Opportunities

- All `[P]` Setup tasks (T002–T007, T009) can run in parallel; T008 depends on T005 (same file); T010/T011 depend on the rest of Setup existing.
- All `[P]` Foundational tasks (T012–T018) can run in parallel; T019 depends on all seven Rust packages (six crates + test harness) existing.
- Once Foundational completes, **US1, US2, and US3 can start in parallel** (disjoint files, `justfile`'s sequencing aside). US5 can start in parallel for its test tasks (T052–T054) but its implementation tasks (T057–T060, T062) must wait for US1's T035/T037/T045.
- Within US1: all test tasks (T020–T027) in parallel; among implementation tasks, T038, T039, T040, T042 are mutually parallel and parallel with the Rust-side tasks; T041/T043/T045 are sequential (shared `main.tsx`).
- Within US5: T052–T054 in parallel; T061 (`ws.ts`) is parallel with the Rust-side tasks (T057–T060); T060 and T062 are each sequential within their own shared file (`main.rs`, `main.tsx` respectively), not with each other.
- Within US3: all test tasks except T067 (T064, T065, T066, T068, T069, T070) in parallel; T067 is sequential after T066 (shared file); T077 and T078 both depend on T071/T073 but are mutually parallel (different files — corrected this pass to carry `[P]`, matching what the prose already claimed).
- US4 cannot start until US1, US2, US3, and US5 are all at their checkpoints, and `justfile`'s prior three sequential edits (T010, T051, T079) are done.

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
Task: "Migration test in tests/integration/migrations.rs"
Task: "Idempotency/conflict integration test in tests/integration/idempotency_conflict.rs"
Task: "Transactional rebuild integration test in tests/integration/transactional_rebuild.rs"
```

Note: the `RebuildAggregate` orchestration test (T067) is deliberately **not** in this parallel batch — it shares `rebuild_aggregate.rs` with the pure `fold_events` test (T066) above and must run after it.

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
4. Add User Story 5 → validate independently (WebSocket boundary), building on US1's router (T035, T037) and health mount (T045); confirm the router-registration (T060) and frontend-consumer (T062) tasks both landed, or the E2E WS assertion (T063) has nothing to observe.
5. Add User Story 4 last → validate the complete local/CI gate against the now-complete recipes.
6. Complete Phase 8: Polish (contract drift, format/lint, full quickstart replay).

### Parallel Team Strategy

With multiple contributors, after Foundational completes:

- Contributor A: User Story 1 (then User Story 5, since it depends on US1's router and health mount).
- Contributor B: User Story 2.
- Contributor C: User Story 3.
- `justfile` edits (T051, T079, T080) MUST be coordinated sequentially between B, C, and whoever takes US4, even though the rest of their stories proceed independently.
- Whoever finishes first takes User Story 4 once US1, US2, US3, and US5 are all at their checkpoints.

---

## Task-to-Requirement Traceability Matrix

Full task-level cross-reference to `traceability.md`'s FR/SC/AC/PLT/SEC identifiers. IDs marked **out of scope** in `traceability.md` (`AC-036`, `AC-037`, `SEC-002`, `SEC-004`, `PLT-003`, `PLT-006`, `PLT-007`, `SPEC-002`–`SPEC-006`) are intentionally addressed by **no task below** — see `traceability.md`'s "Non-Applicable IDs" section for the explicit justification.

**Corrected this pass**: several rows in the previous revision assigned a requirement to a task merely because the task supported nearby infrastructure, rather than because the task concretely provides the stated coverage. Every row below was re-derived directly against `traceability.md`'s own FR→AC/PLT and SC→FR mappings; rows that no longer hold up are trimmed to `—`, and grouped rows (e.g. `T083–T085` in the previous revision) are split so each task's own contribution is shown, not assumed shared.

| Task(s) | FR | SC | AC | PLT / SEC | Constitution |
| --- | --- | --- | --- | --- | --- |
| T001, T003–T009, T011 | — | — | — | — | I, IX |
| T002 | — | — | — | — | I *(pins dependencies; does not itself implement any FR's behavior — corrected this pass, was FR-002–FR-008, FR-012, FR-019–FR-022)* |
| T010, T051, T079, T080 | FR-014 | SC-006, SC-007 | AC-040, AC-044 | PLT-005 (restricted), PLT-009 (restricted) | IX |
| T012–T017 | FR-009, FR-010 | SC-005 | AC-041 | — | II |
| T018, T049, T050 | FR-009, FR-010 | SC-005 | AC-041 | — | II, VI |
| T019 | — | — | — | — | II, III *(unsafe-code prohibition + module skeletons; not the dependency-allowlist mechanism itself — corrected this pass, was FR-009, FR-010, SC-005, AC-041)* |
| T020 | FR-021 | — | AC-043 | — | IV |
| T028 | FR-019 | — | AC-043 | — | IV |
| T029 | FR-019, FR-020 | SC-008 | AC-043 | — | IV *(the concrete drift-generation task — corrected this pass: FR-020/SC-008 no longer spread across T020/T028 as well)* |
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
| T035 | FR-007 | SC-001, SC-002 | AC-040 | — | II *(composition-point wiring; the specific Host/Origin/CORS/token behaviors it wires are already credited to T031–T033 individually — corrected this pass, was FR-001–FR-007 collectively, plus III)* |
| T038 | FR-001 | SC-001 | AC-040 | PLT-001 (restricted) | III *(corrected this pass: PLT-004 removed — the supervisor script has no TanStack/Zustand state-management concern)* |
| T039 | FR-001 | SC-001 | AC-040 | — | III |
| T040, T042 | FR-023 | SC-010 | — | PLT-011 | VII |
| T041 | FR-024 | SC-011 | — | PLT-004 | VII |
| T043, T044 | FR-008, FR-024 | SC-011 | AC-035 | PLT-004 | II |
| T045 | FR-023 | SC-010 | — | PLT-011 | VII |
| T046 | FR-023 | SC-010 | — | PLT-011 | VII |
| T047 | FR-024 | SC-011 | — | PLT-004 | II |
| T048 | FR-016 | — | AC-043 | PLT-008 | VI |
| T052 | FR-021 | SC-009 | AC-043 | — | IV |
| T055 | FR-019 | — | AC-043 | — | IV |
| T056 | FR-019, FR-020 | SC-008 | AC-043 | — | IV *(corrected this pass, split from T052/T055 for the same reason as T020/T028/T029)* |
| T053, T057 | FR-022 | SC-009 | AC-033, AC-034 | SEC-001, SEC-003 | III |
| T054, T058 | FR-017, FR-022 | SC-009 | AC-038 | SEC-008 | VIII |
| T059 | — | SC-009 | — | — | VI *(implements the proof exchange itself; FR-016/AC-043's actual test coverage is credited to T048/T063 — corrected this pass, was FR-016, AC-043)* |
| T060 | FR-022 | SC-009 | AC-033, AC-034 | SEC-001, SEC-003 | II, III *(new this pass — router registration is the composition point that makes the WS security boundary reachable at all)* |
| T061 | FR-008 | — | AC-035 | — | II |
| T062 | FR-008, FR-016 | — | AC-043 | PLT-008 | VI *(new this pass — the frontend consumer that makes the E2E WS observation possible)* |
| T063, T048 | FR-016 | — | AC-043 | PLT-008 | VI |
| T064, T072 | — | — | — | — | V *(domain entity definition/invariants only; no single FR/AC/PLT cites entity definition as its satisfying artifact — corrected this pass, was FR-011, FR-012, FR-013 collectively)* |
| T065, T075 | FR-012 | SC-003 | — | — | V |
| T066, T074 | FR-013 | SC-004 | — | — | II, V *(pure Application folding — corrected this pass: AC-042/PLT-002 removed, since those identifiers concern migration/persistence/SQLx behavior this code deliberately does not touch)* |
| T067, T076 | FR-013 | SC-004 | — | — | II, V *(same correction as above — orchestration test/impl, still no SQL)* |
| T068, T071 | FR-011 | — | AC-042 | PLT-002 | V |
| T069, T077 | FR-012 | SC-003 | AC-042 | PLT-002 | V |
| T070, T078 | FR-013 | SC-004 | AC-042 | PLT-002 | II, V |
| T081 | FR-015 | SC-006 | AC-044 | PLT-005 (restricted), PLT-009 (restricted) | IX |
| T082 | FR-015 | — | — | PLT-009 (restricted) | IX |
| T083 | FR-018 | SC-007 | — | — | IX, X |
| T084 | FR-019, FR-020 | SC-008 | AC-043 | — | IV |
| T085 | FR-014 | — | AC-040 | — | IX *(fmt/lint gate execution only — corrected this pass, split out of the previous `T083–T085` grouped row)* |
| T086 | FR-014, FR-016, FR-018 | SC-006, SC-007 | AC-040, AC-043, AC-044 | — | VI, IX, X *(the comprehensive end-to-end replay plus the human-runbook existence check)* |
| T087 | FR-014 | — | AC-040 | — | IX *(README only, no independent test evidence beyond matching `quickstart.md`)* |

---

## Constitution Compliance Check (Task-Breakdown Level)

Re-verified against `.specify/memory/constitution.md` v1.0.1 and `plan.md`'s Post-Design Gate, and re-checked specifically against this pass's ten corrections:

| Principle | Task-breakdown verification | Status |
| --- | --- | --- |
| I. Versioned Sources of Truth | Every task cites an FR/SC/AC/PLT/SEC/design-artifact already present in `spec.md`/`plan.md`/`data-model.md`/`contracts/`/`traceability.md` (full matrix above, now precisely rather than loosely attributed). No new requirement is introduced by this correction pass. Two genuine open points (PID-capture mechanism, `futures-util` gap) are reported explicitly rather than silently resolved — see "Flagged Items for Human Confirmation." | **PASS** |
| II. Modular Clean Architecture | Phase 2 (T012–T019) and User Story 2 (T049–T051) make the six-crate allowlist and the mechanical check explicit tasks; the seventh, test-only workspace member (`tests/`) is explicitly excluded from that allowlist check by design. `RebuildAggregate` contains no SQL (T073, T074, T076, T078), and its port contract is now **correctly asynchronous** (T073) — **strengthened this pass**: the first revision's synchronous port signature could not have been genuinely implemented by `adapters-persistence`'s async SQLx pool, a real correctness gap this pass closes without any new dependency (native `async fn` in traits). | **PASS**, strengthened |
| III. Secure Local Runtime | User Story 1 (T030–T039) and User Story 5 (T057–T060) implement loopback/random-port/ephemeral-token/`Host`/`Origin`/CORS/allowlist exactly as `research.md` Part 2 and the contracts specify. The orphan-process test (T026) drops the previous draft's invented Windows `tasklist`/empty-output parsing and instead names the mechanism `research.md` actually specifies (`Child::kill()` → `SIGKILL`/`TerminateProcess`; liveness via POSIX `kill -0` / a Node one-liner calling `process.kill(pid, 0)`) — **strengthened this pass**, though the exact PID-capture wiring (T038) is honestly flagged as an assumption, not fully specified by either normative document. No task introduces shell or unrestricted filesystem access. | **PASS**, strengthened, one item flagged |
| IV. Contracts as Code | T028/T055 make Rust the authoritative source with exact Serde/`ts-rs` attributes; T029/T056 actually run `contracts-generate` before any frontend task imports the generated type, now correctly the only tasks credited with FR-020/SC-008 (corrected this pass — the previous revision spread the drift-gate requirement onto the wire-type-definition and contract-test tasks too). T084 verifies the drift gate. **One open item**: T052–T054's WebSocket tests need `futures-util` to drive `tokio-tungstenite` ergonomically, and that crate is not pinned anywhere in `research.md` — reported, not silently added (see "Flagged Items for Human Confirmation"); this does not block T029/T056/T084, only the WS test-writing tasks. | **PASS** for contract generation/drift; **one dependency gap flagged**, not yet resolved |
| V. Durable and Reconstructable Data | User Story 3 (T064–T079) implements immutable-ledger triggers, idempotency/conflict classification, and the transactional rebuild exactly per `data-model.md`, with the rebuild's transaction ownership confined to `adapters-persistence` alone (T073, T078). Traceability corrected this pass so `AC-042`/`PLT-002` (migration/persistence/SQLx identifiers) are credited only to the tasks that actually touch migrations or a real transaction (T068/T071, T069/T077, T070/T078), not to the pure Application folding tasks (T066/T074, T067/T076). | **PASS**, strengthened (traceability precision) |
| VI. Risk-Oriented Test-First Development | Every domain/application/contract/security-boundary task above is paired with a test task that precedes it and is required to fail first (RGR); the pure `fold_events` function and the `RebuildAggregate` orchestration have separate, independently RGR-paired tests (T066/T074, T067/T076), with T067 now correctly marked non-parallel against T066 (same file — corrected this pass). UI/adapter/infrastructure tasks are tested in the same phase without forced test-first ordering. | **PASS** |
| VII. Product and UX Integrity | T040–T042 make Graphite Signal tokens, theme selection, and reduced-motion handling explicit tasks with exact file paths. T045–T046 implement and test the Frontend State Matrix, both themes, keyboard/focus/non-color/reduced-motion, and `jest-axe` (now correctly configured via ESM import, with no unpinned `jest-dom` dependency — corrected this pass), exactly as FR-023 and spec.md require. | **PASS** |
| VIII. Privacy and Observability | T027 (HTTP) and T054 (WebSocket) are dedicated redaction tests; T038's new PID status line is explicitly documented as non-secret (process IDs, not the auth token) so it does not reopen the redaction requirement; no task introduces telemetry or automatic transmission. | **PASS** |
| IX. Documentation and Delivery Discipline | T010/T080 make the `justfile` the single command facade for contributors and CI, with its four sequential edit-owners renumbered but still explicit (T010→T051→T079→T080); T081 keeps the Linux/Windows CI jobs merged into one correctly-scoped task; T083 documents the human-performed CI-break runbook, and T086 (corrected this pass) validates only that the runbook exists and is correct without executing it — removing the last place this task list could have been read as instructing an agent to perform a Git-publishing action. | **PASS**, strengthened |
| X. Human-Controlled Version Control | No task in this breakdown stages, commits, pushes, or opens a pull request. T079's `.sqlx/` cache task generates the cache and leaves staging/committing to the human maintainer. T083's CI-break verification is a documentation-only, human-performed runbook. T086 (corrected this pass) explicitly forbids the validating agent from executing that workflow itself and requires any reported Git state to cite actual `git status` output, not be asserted from memory. This document is itself a plain file pending human review. | **PASS**, strengthened |

**No unresolved conflict, and two honestly unresolved dependency/mechanism gaps** (see "Flagged Items for Human Confirmation" above — the orphan-process PID-capture assumption and the `futures-util` gap for WebSocket tests). Per the user's explicit instruction for this correction pass, `/speckit-implement` was **NOT** invoked. No production code, migration, configuration, or test file was created or modified while generating or correcting this task list — only `specs/001-slice-0-foundation/tasks.md` was written. This stops here for human review before any implementation task begins.

---

## Notes

- `[P]` tasks touch different files with no unmet dependency. This pass re-validated every `[P]` marker in the document: it found and fixed two remaining invalid-parallel cases the previous revision missed (`T005`/`T008` on `apps/web/package.json`; the tasks now numbered `T066`/`T067` on `rebuild_aggregate.rs`), and one missing-parallel case (`T077`/`T078`, genuinely different files, now both correctly carry `[P]`).
- `[Story]` maps each task to its user story for traceability back to `spec.md` and `traceability.md`.
- RGR (Red-Green-Refactor) tasks are cross-referenced above ("makes T0xx pass") so the paired test can be verified to fail before its implementation task starts.
- Per Constitution Principle X, no agent may stage, commit, push, revert Git history, or open a pull request after completing tasks — report a change summary, files changed, verification results, risks, and exact suggested Git commands for the human maintainer instead. T083 and T086 in particular MUST NOT be executed as live Git operations by an agent.
- Task count changed from **85 to 87** this pass: two tasks were added (T060, register `/api/ws` into the router; T062, the frontend WS-proof consumer) to complete the WebSocket vertical slice (correction 3). Every task from the old `T060` onward was renumbered by +2 to keep the sequence contiguous; every cross-reference in this file (Dependencies, File-Ownership Map, Parallel Examples, Implementation Strategy, Traceability Matrix, Constitution table) was updated to match the new numbering.
- Two items are deliberately left open rather than silently resolved — see "Flagged Items for Human Confirmation" near the top of this file.
- Stop at any checkpoint to validate a story independently before continuing.
