---

description: "Task list for Slice 0 — Foundation implementation"
---

# Tasks: Slice 0 — Foundation

**Input**: Design documents from `specs/001-slice-0-foundation/`

**Prerequisites**: `plan.md` (required — Approved), `spec.md` (required — Approved), `research.md`, `data-model.md`, `contracts/http-health.md`, `contracts/websocket-proof.md`, `traceability.md`, `.specify/memory/constitution.md` (v1.0.1)

**Correction pass note (first revision)**: an earlier revision corrected nine defects found in the original generation — a concrete Cargo test-harness strategy, dependency-pin synchronization, missing tooling tasks, false `[P]` markers, a Clean Architecture fix for `RebuildAggregate`, human-controlled Git wording, a platform-specific orphan-process mechanism, a full traceability matrix, and a re-verified Constitution table.

**Correction pass note (second revision)**: a follow-up audit found ten further defects — remaining `[P]`/ownership conflicts, missing cross-story dependency edges, an incomplete WebSocket vertical slice (router registration and a frontend consumer were both missing), an invented orphan-process mechanism, a synchronous port contract incompatible with async SQLx, missing `converge-tests` dependencies, an unpinned `jest-dom`/`require`-based `jest-axe` setup, Git-boundary-crossing wording in the final validation task, inflated traceability, and a Constitution table that needed re-verification. Task count moved from 85 to 87.

**Correction pass note (third revision, this pass)**: a third audit found seven further defects, all corrected here, none touching any normative document: (1) the pure `fold_events` contract was infallible (`Fn(&[Event]) -> Vec<AggregateProjection>`), which cannot express the mid-fold-failure-and-rollback behavior `T070` already required — corrected to a typed, fallible contract (`Fn(&[Event]) -> Result<Vec<AggregateProjection>, FoldError>`), synchronized across `T073`/`T074`/`T076`/`T078` and their tests; (2) `T008`'s Vitest setup called `expect.extend(...)` without an explicit `expect` import — corrected to the ESM form `import { expect } from "vitest";`; (3) the `futures-util` gap for `T052`–`T054` is now treated as the **hard, unresolved normative blocker** it is, not a soft footnote — those three test tasks are marked **PENDING**, not executable; `T057`–`T059` (the US5 route-handler implementation tasks they gate) remain individually implementable, since neither their files nor their declared dependencies are themselves blocked, but by Constitution VI's test-first discipline they cannot honestly complete a Red-Green-Refactor cycle against `T053`/`T054` until those tests compile — **corrected in the fourth revision below**, since this sentence originally read as if the implementation tasks themselves were marked `PENDING`, which they are not and never were at the individual task level; (4) the previous revision's assumption that `scripts/dev.mjs` prints a dedicated PID status line was itself an invented transport mechanism the second revision was supposed to avoid — removed; the PID-capture-and-liveness-probe mechanism is now an explicit, unresolved human decision gate with **no** assumed transport, keeping only what `research.md` concretely supports (`Child::kill()` → `SIGKILL`/`TerminateProcess`); (5) the traceability matrix was re-audited task by task — `FR-009`/`FR-010` are no longer collectively assigned to `T012`–`T017`; `T018` is no longer grouped with `T049`/`T050` as requirement evidence; `T010`/`T051`/`T079`/`T080` no longer collectively inherit CI-level `SC`/`AC`/`PLT` identifiers that `T081` alone concretely provides; `FR-008` no longer sits on `T043` (mounts `QueryClient`, reaches no backend behavior itself); `T062` (implementation) and `T063` (validation) are no longer credited with the same FR; `SC-007` no longer sits on `T086`(now `T087`, documentation-only); (6) a new human-maintainer-only checkpoint task records `SC-007`'s actual execution evidence, distinct from documenting the runbook — task count moves from 87 to **88**; (7) the Constitution table now uses `PASS`/`PENDING` as actual statuses (not "strengthened" as a stand-in), with Principles III, IV, VI, IX, and X marked `PENDING` and their exact blockers cited.

**Correction pass note (fourth revision, this pass)**: a fourth audit, run after the third revision's 88-task version was otherwise approved, found six further defects, all corrected here without touching any normative document and without adding, removing, or renumbering any task: (1) Constitution Principle VIII cited `T054` (WebSocket redaction) as `PASS`-level evidence even though `T054` remains `PENDING` on the unresolved `futures-util` pin — corrected to `PENDING`, with `T027`'s HTTP redaction evidence preserved separately as valid, executable evidence for its own scope; (2) `T046` labelled itself as covering both themes, reduced motion, keyboard/focus, and non-color state communication without concretely requiring any of them — corrected to state each check explicitly: rendering under both Light and Dark themes, a simulated `prefers-reduced-motion: reduce` query asserting the removal of non-essential movement/displacement/sequenced animation while state is still communicated immediately, keyboard navigation and visible focus, non-color state communication, and the existing `jest-axe` assertion — using only already-approved frontend dependencies, no visual-regression tool added; (3) the traceability matrix was re-audited again: `SC-005` removed from `T012`/`T013` (manifest configuration only — `SC-005`'s automated-boundary-check evidence stays exclusively on `T049`/`T050`); `SC-008` removed from `T029`/`T056` (contract generation only — `SC-008`'s zero-drift evidence stays exclusively on `T085`); `SC-009` removed from `T052` (a wire-shape/negotiation contract test, not a handshake-rejection test) and from `T059` (implements the post-connection message exchange, not a rejection path); `FR-022`/`SC-009` removed from `T054` (redaction only, not handshake-rejection behavior); `SC-007` removed from `T036`/`T037` (shutdown/watchdog implementation, not CI-failure-visibility evidence) and, by the identical reasoning, from `T025` (cooperative-shutdown test, which tests FR-001/FR-018, not the CI-gate-visibility criterion) and `SC-001` removed from `T026` (orphan-process test, which does not test the one-command-launch criterion); (4) `SC-007`'s evidence is now explicitly split into two distinct forms — automated category coverage, cited narratively rather than by adding the `SC-007` identifier to further rows (to avoid re-introducing the inflation this pass corrects elsewhere), and `T084`'s human checkpoint, which remains the sole task the `SC-007` identifier is credited to — see the new note directly above the traceability matrix; (5) the third revision's own note inaccurately implied `T057`–`T059` themselves were marked `PENDING` — corrected in place: only `T052`–`T054` are `PENDING`; `T057`–`T059` remain individually implementable, and only their Constitution VI Red-Green-Refactor *pairing* with `T053`/`T054` is blocked; (6) the Constitution table is re-verified against every fix above — Principle VII now cites `T046`'s concretely expanded scope for a fully-supported `PASS`; Principle VIII moves to `PENDING`; Principles III, IV, VI, IX, and X remain `PENDING` for their previously-cited, still-unresolved blockers. Task count is unchanged at **88**.

**Correction pass note (fifth revision, this pass)**: a fifth audit found six further defects, all corrected here without touching any normative document and without adding, removing, or renumbering any task: (1) `T008`'s Vitest/`jest-axe` setup imported `expect` correctly but had no TypeScript path making `toHaveNoViolations` recognized by Vitest's assertion types, and `@types/jest-axe` has no approved pin in `research.md` — resolved, not left `PENDING`, by expanding `T008` with a local Vitest module-augmentation declaration at `apps/web/src/types/jest-axe.d.ts`, using only the already-approved, directly-depended-on `vitest@4.1.10` package's own exported `Assertion`/`AsymmetricMatchersContaining` interfaces — no `@types/jest-axe` pin invented, no Jest-namespace/global dependency introduced, no new package added; Constitution Principle VII returns to `PASS` on this basis; (2) the traceability matrix still had inconsistent `SC-009` mappings — `T060` (route registration only) and the combined `T057, T058` row (which credited `T058`'s successful-subprotocol-negotiation implementation with the same rejected-handshake evidence as `T057`) both removed `SC-009`; `T057` and `T058` are now separate rows, keeping rejected-handshake (`T057`, paired with `T053`), successful-negotiation (`T058`, evidenced narratively by `T052`), redaction (`T054`), and post-connection-message (`T059`) evidence distinct; (3) further inflated traceability identifiers were removed after re-auditing every row: `FR-020` off `T029`/`T056` (contract generation only — `FR-020`'s automated drift-failure check and `SC-008`'s zero-drift evidence both stay exclusively on `T085`); `FR-009` off `T049` (satisfied by `T012`/`T013`'s manifests, not by the check) and `SC-005` off `T050` (a synthetic-fixture unit test of the check's own logic, not a confirmation of the real codebase's zero-reference state — split into two individual rows); `AC-042`/`PLT-002` off `T069`/`T077` (idempotent-persistence evidence, not migration/aggregate-rebuild evidence per `traceability.md`'s own `AC-042`/`PLT-002` scope); `AC-040` off `T086` (runs `fmt-check`/`lint` only, unrelated to the one-command-launch criterion); `SC-010` off `T040`/`T042`/`T045` and `SC-011` off `T041`/`T043`/`T044` (measured success criteria now credited exclusively to their validation tasks, `T046`/`T047`, not to the implementation tasks that merely build the related UI); (4) `SC-007` previously had automated planned coverage for only three of its four required categories, with the "broken migration" category explicitly disclosed as unautomated — closed this pass by expanding `T068` with a deliberate migration-checksum-mismatch case (mutating an already-applied migration file and asserting `sqlx::migrate::Migrator` returns a typed `MigrateError` rather than succeeding silently or panicking, using SQLx's own built-in integrity check, no new dependency); all four categories (denied authentication, architectural-boundary violation, broken migration, broken test/gate-visibility via `T084`) now have concrete, executable planned coverage; (5) Constitution Principle IX is corrected to `PASS` on this basis — its planning-gate bar is that the plan concretely and executably covers all four `SC-007` categories, not that every human-performed step (`T084`'s execution) has already happened; `T084` itself is unchanged, still explicitly human-operated, still unchecked, still not fabricated; (6) Constitution Principle X is corrected to `PASS`, decoupling agent Git-write discipline (a planning-time property this document has honored throughout every revision) from `T084`'s still-pending future execution evidence, which the planning gate must not require. Principles III, IV, VI, and VIII remain `PENDING`, unaffected by any fix in this pass, on their previously-cited and still-unresolved blockers (`futures-util`, the PID-capture/liveness-probe mechanism). Task count is unchanged at **88**.

**Tests**: Mandatory per Constitution Principle VI, scoped by risk. Red-Green-Refactor (test written and failing before implementation) is mandatory for domain logic, application use cases, contracts, and security boundaries — every test task below marked against those layers MUST be completed, and MUST fail, before its paired implementation task. UI, configuration, infrastructure, and adapter tests are included in the same change without requiring literal test-first ordering.

**Organization**: Tasks are grouped by user story (spec.md priorities P1–P4, with US2 and US5 tied at P2) to enable independent implementation and testing of each story, per spec.md's explicit "Independent Test" for each.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no unmet dependency)
- **[Story]**: US1–US5, mapping to spec.md's five user stories. Setup, Foundational, and Polish tasks carry no story label.
- Every task states its exact file path(s) and, where applicable, the FR/SC/AC/PLT/SEC identifier(s) it satisfies (`traceability.md`). A full task-level matrix is at the end of this file.
- **PENDING** tasks (marked explicitly below) are not yet executable: either a normative dependency is unresolved (`futures-util`) or a mechanism is an open human decision (PID capture/liveness probe). They are not implemented, not skipped silently, and not counted as passing.

## Repository State at Start

This slice starts from an empty implementation surface: no `crates/`, `apps/web/`, `packages/`, `migrations/`, `scripts/`, `tests/`, `justfile`, or `.github/workflows/` exist yet in the repository. Every task below creates new files; none modifies pre-existing implementation code.

## Test Harness Package (human-approved, first revision)

`tests/architecture/`, `tests/contract/`, and `tests/integration/` hold plain `.rs` files under a top-level directory that is not itself a Cargo package — Cargo cannot compile or run them without one. **Human decision, first correction pass**: a single, dedicated, non-production Cargo package, `tests/Cargo.toml` (package name `converge-tests`), is added as a **seventh workspace member**, alongside — never replacing — the six approved production crates under `crates/`. This package:

- Carries **no** Clean Architecture normal-dependency allowlist (FR-009/FR-010's mechanical check, Testing Plan, only ever applies to `domain` and `application`; a black-box test harness is architecturally inert by definition and is explicitly excluded from that check's scope in the task that wires it, T049).
- Owns every `.rs` file under `tests/architecture/`, `tests/contract/`, `tests/integration/` as an explicit `[[test]]` target (`path = "..."`) — not Cargo's automatic `tests/*.rs` discovery, which only scans a package's immediate `tests/` directory, not subdirectories.
- Is created once, fully, in Foundational (T018) with every target pre-declared against a trivially-passing stub body, so no later story ever re-edits `tests/Cargo.toml` itself — each story only replaces the body of its own already-declared, uniquely-named stub file(s), which keeps every later story's test-writing task genuinely single-file and parallel-safe.
- `tests/e2e/` is a separate, ordinary **pnpm** package (`@converge/e2e`, WebdriverIO), not part of this Cargo package — it is TypeScript, not Rust.
- Also depends on `adapters-persistence` (path) and `sqlx.workspace = true` (T018) — its own migration/persistence/transaction integration tests need the real adapter and a real SQLite connection.

## Pending / Blocked Items (this pass) — read before executing any task below

Two matters remain genuinely unresolved and are **not** silently invented around. Every task they affect is marked `PENDING` in its own description and in the traceability matrix, not presented as ready or passing.

1. **`futures-util` dependency gap — blocks T052, T053, T054** (US5 contract/integration tests). Driving `tokio-tungstenite`'s `WebSocketStream` ergonomically (`.next()`/`.send()`) requires the extension traits `futures_util::{StreamExt, SinkExt}` in scope. This is not optional ergonomics: `WebSocketStream` implements the `Stream`/`Sink` traits from the `futures` family, and Rust does not let a crate call trait methods from a crate it has not itself declared as a direct dependency — `futures-util` cannot be reached as an undocumented transitive dependency of `tokio-tungstenite`, confirmed by inspecting what `tokio-tungstenite`'s own public API actually requires. `research.md` pins `tokio-tungstenite` but never pins `futures-util` (or the `futures` umbrella crate), and no already-approved direct dependency provides an equivalent API. **This task list does not add `futures-util`.** T052–T054 are marked `PENDING` below; they cannot compile as specified until a human maintainer either approves an exact `futures-util` pin in `research.md` or confirms an alternative mechanism. Under Constitution VI's test-first discipline, US5's route-handler implementation tasks that these tests are meant to gate (T057, T058, T059) cannot complete their RGR cycle while T053/T054 cannot compile — those tasks remain implementable in isolation but the story's checkpoint cannot be honestly declared complete until this is resolved.
2. **PID-capture and liveness-probe mechanism for the abrupt-supervisor test — blocks nothing in T038, keeps T025/T026 mechanism-incomplete**. `research.md`/`plan.md` describe the desired guarantee (both children terminate when the supervisor is killed abruptly) and name the *production* mechanism precisely (OS-level pipe/IPC teardown; `SIGKILL`/`TerminateProcess` cannot be caught), but neither document specifies how an external **test** learns the two children's OS PIDs, nor a concrete cross-platform liveness-probe implementation. The previous revision assumed `scripts/dev.mjs` would print a dedicated PID status line for test observability — that assumption is **removed this pass**: it is itself an invented transport mechanism (a bespoke stdout protocol), which is exactly what this pass was asked not to do. **No PID file, environment variable, stdout protocol, or test-only endpoint is assumed or implemented anywhere in this revision.** `T038` no longer describes any such mechanism. `T025`/`T026` keep only what is concretely approved — terminating via `std::process::Child::kill()` (`SIGKILL` on POSIX, `TerminateProcess` on Windows) — and mark the PID-capture/liveness-probe mechanism itself as an open human decision. Both tasks still require that both captured child PIDs are independently confirmed exited, and that a later successful `just dev` run is explicitly **not** accepted as evidence — that acceptance criterion does not depend on which mechanism is eventually chosen.

Recommendation (not a normative decision made here): a human maintainer should decide (a) whether `futures-util` gets an exact pin in `research.md`, and (b) how the two child PIDs are meant to reach the test — e.g. by amending `research.md`/`plan.md` with an explicit protocol, or by redirecting these two tests to a Node-hosted implementation that can call `child.pid` directly. Neither decision is made in this file.

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
- [ ] T008 Configure Vitest + React Testing Library + jsdom + jest-axe per `research.md` pins (`vitest` 4.1.10, `@testing-library/react` 16.3.2, `@testing-library/dom` 10.4.1, `jsdom` 29.1.1, `jest-axe` 10.0.0 — all five re-validated against `research.md`'s exact pins this pass) — **corrected this pass: `expect.extend(...)` had no explicit `expect` in scope** — `apps/web/package.json` devDependencies (depends on T005 — same file, sequential append-only edit, not parallel; see File-Ownership Map), `apps/web/vitest.config.ts` (`environment: "jsdom"`, `setupFiles: "./vitest.setup.ts"`), `apps/web/vitest.setup.ts`:
  ```ts
  import { expect } from "vitest";
  import { toHaveNoViolations } from "jest-axe";
  expect.extend(toHaveNoViolations);
  ```
  No implicit/global `expect` is enabled to paper over the missing import, and no `@testing-library/jest-dom` import is added (no exact pin exists for it anywhere in `research.md`). Frontend assertions elsewhere in this slice use plain Testing-Library/DOM queries and Vitest's built-in `expect`, not jest-dom-specific matchers.

  **`toHaveNoViolations` type declaration (new this pass — resolves the TypeScript blocker without any new dependency)**: `jest-axe@10.0.0` ships no bundled `.d.ts`, and `research.md` pins no `@types/jest-axe` — no such pin is invented here. Vitest's own documented module-augmentation mechanism is sufficient by itself, using only the already-approved, directly-depended-on `vitest@4.1.10` package's own exported interfaces — add `apps/web/src/types/jest-axe.d.ts`:
  ```ts
  import "vitest";

  interface JestAxeMatchers<R = unknown> {
    toHaveNoViolations(): R;
  }

  declare module "vitest" {
    interface Assertion<T = unknown> extends JestAxeMatchers<T> {}
    interface AsymmetricMatchersContaining extends JestAxeMatchers {}
  }
  ```
  This declares exactly `jest-axe`'s actual runtime matcher shape — a zero-argument matcher registered through `expect.extend`, per `jest-axe`'s own documented API — no broader or mismatched type is fabricated. It augments Vitest's own `Assertion`/`AsymmetricMatchersContaining` interfaces directly, not a Jest namespace (no `declare global { namespace jest { ... } }`), so it does not depend on Jest globals or `@types/jest`. The file lives under `apps/web/src/`, already inside `apps/web/tsconfig.json`'s default `"include": ["src"]` scope from T005's scaffold — no `tsconfig.json` edit is required; if a later task narrows that `include` pattern, it MUST keep `src/types/**/*.d.ts` covered. This task's Done condition includes the frontend type-check (`tsc --noEmit`, part of `just lint`/`just test-frontend`) resolving `toHaveNoViolations()` on `expect(...)` with no error, using only already-approved direct dependencies — not merely that the declaration file exists.
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
- [ ] T017 Configure `crates/service/Cargo.toml` — composition root, depends on all five other crates plus `rand.workspace = true`, `base64.workspace = true`, and its own direct `tokio.workspace = true` dependency (features `"rt-multi-thread"`, `"macros"`, `"signal"`, `"process"`, `"io-std"` — the async runtime `service`'s own `main.rs` needs for the Axum server, the `SIGINT`/`SIGTERM` handlers, and the stdin-EOF watchdog). No allowlist restriction applies to `service` itself (the only crate exempt from the per-crate allowlist check, per `plan.md` Project Structure)
- [ ] T018 [P] Create `tests/Cargo.toml` — the test-harness package (`converge-tests`, workspace member #7, no Clean Architecture allowlist restriction; see "Test Harness Package" above). Normal dependencies: `domain`/`application`/`contracts`/`adapters-persistence` (path, for white-box assertions where needed — T068/T069/T070's schema, trigger, repository, and real-transaction tests exercise the actual persistence adapter and a live SQLite database), `reqwest.workspace = true`, `tokio-tungstenite.workspace = true`, `tokio.workspace = true` (full features), `serde_json.workspace = true`, `time.workspace = true` (feature `"parsing"`), `sqlx = { workspace = true, features = ["sqlite", "runtime-tokio", "macros", "migrate"] }` (same exact features as `adapters-persistence`'s own pin, T016), and `uuid = { workspace = true, features = ["v7"] }` (the "v7" generation feature belongs on this package's dependency edge specifically, since `data-model.md`'s Event identity section names "the integration test suite" as the producer that generates `event_id`). **Does not include `futures-util`** — this is a deliberate, reported omission, not an oversight: see "Pending / Blocked Items" above. T052–T054's `[[test]]` targets are declared as usual (stub bodies), but implementing real bodies for them is `PENDING`, not part of this task. Declare one `[[test]]` target per eventual test file, each with a trivially-passing stub body created in this same task: `allowlist_check` (`architecture/allowlist_check.rs`), `http_health_contract` (`contract/http_health_contract.rs`), `websocket_proof_contract` (`contract/websocket_proof_contract.rs`), `http_auth_denial`, `http_cors_preflight`, `service_restart`, `concurrent_instances`, `launcher_shutdown`, `orphan_process`, `redaction`, `ws_auth_denial`, `ws_redaction`, `migrations`, `idempotency_conflict`, `transactional_rebuild` (all under `integration/`). `tests/architecture/fixtures/violation_metadata.json` is a data fixture, not a compiled target — created empty here, populated by T050.
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
- [ ] T025 [P] [US1] **PENDING (mechanism incomplete — see "Pending / Blocked Items" above)**: Integration test: cooperative launcher shutdown — send the supervisor `SIGINT`/`SIGTERM`, assert both children's PIDs exit within the bounded timeout. The mechanism for this test to obtain the two children's PIDs is an open human decision, not assumed or invented here (no PID file, env var, stdout protocol, or test-only endpoint) — in `tests/integration/launcher_shutdown.rs` (FR-001, FR-018)
- [ ] T026 [P] [US1] **PENDING (mechanism incomplete — see "Pending / Blocked Items" above)**: Integration test: orphan-process — terminate the supervisor abruptly via Rust `std::process::Child::kill()` (sends `SIGKILL` on POSIX; calls `TerminateProcess` on Windows — std's own portable, uncatchable primitive; this part is concrete and approved, `research.md`'s corrected mechanism), then confirm the two child processes (Vite, Rust service) are exited. Neither the PID-capture mechanism nor the exact liveness-probe implementation on either platform is specified concretely enough in `research.md`/`plan.md` to implement without inventing new protocol — this is an open human decision, not resolved here. Regardless of mechanism, both PIDs captured before the kill signal MUST be independently confirmed exited — a later successful `just dev` run is explicitly NOT accepted as evidence — in `tests/integration/orphan_process.rs` (FR-001, FR-018)
- [ ] T027 [P] [US1] Redaction test: negative-grep over captured logs/process output — the token never appears in a URL-shaped, header-echoed, or plain-text log line — in `tests/integration/redaction.rs` (FR-017, SEC-008)

### Implementation for User Story 1

- [ ] T028 [US1] Define `HealthResponse`/`HealthStatus`/`ApiError` Rust types with exact Serde/`ts-rs` attributes (FR-019) in `crates/contracts/src/health.rs` (depends on T014, T019; makes T020 pass)
- [ ] T029 [US1] Run `just contracts-generate` to produce `packages/contracts/src/generated/health.ts` from `crates/contracts/src/health.rs` and add its re-export to `packages/contracts/src/index.ts` (T006's hand-written barrel — second of three sequential appenders, after T006 and before T056) (depends on T028; FR-019, FR-020)
- [ ] T030 [US1] Implement ephemeral token generation (`rand::OsRng`, base64url no-pad via `base64 =0.22.1`) in `crates/service/src/token.rs` (FR-003)
- [ ] T031 [US1] Implement loopback bind `127.0.0.1:0`, `CONVERGE_ALLOWED_ORIGIN` env read, and the `CONVERGE_BOOTSTRAP` stdout line in `crates/service/src/main.rs` (FR-001, FR-002, SEC-003)
- [ ] T032 [US1] Implement `Host`→`Origin`→token validation middleware, first-failure-wins, in `crates/adapters-http/src/middleware/auth.rs` (FR-004, FR-006, SEC-003; makes T021 pass)
- [ ] T033 [US1] Implement closed-CORS preflight handling (4-row check order, exact 3-header grant, `Access-Control-Allow-Origin` on the actual `200` response too) in `crates/adapters-http/src/middleware/cors.rs` (FR-005, SEC-003; makes T022 pass)
- [ ] T034 [US1] Implement `GET /api/health` handler in `crates/adapters-http/src/routes/health.rs` (FR-007)
- [ ] T035 [US1] Wire the Axum router (health route + middleware stack) in `crates/service/src/main.rs` (depends on T032, T033, T034)
- [ ] T036 [US1] Implement the stdin-pipe EOF watchdog (background `tokio` task; EOF triggers the same shutdown path as `SIGTERM`) in `crates/service/src/main.rs` (`research.md` orphan-process mechanism)
- [ ] T037 [US1] Implement graceful shutdown on `SIGINT`/`SIGTERM` (close listener, bounded drain, exit) in `crates/service/src/main.rs` (FR-018)
- [ ] T038 [P] [US1] Implement `scripts/dev.mjs` supervisor: fork Vite first, spawn the Rust service with `CONVERGE_ALLOWED_ORIGIN`, forward port/token over IPC, print a redacted status line, cooperative shutdown, and the Vite child's `'disconnect'`-triggered teardown (FR-001, `research.md` Bootstrap Ordering). **Corrected this pass**: this task no longer prints any additional PID-status line — the previous revision's assumption that it would was itself an invented transport mechanism; see "Pending / Blocked Items" above. This task's scope is exactly the supervisor behavior `research.md`/`plan.md` already describe, nothing more.
- [ ] T039 [P] [US1] Implement the Vite dev-bootstrap middleware — `Host`-validated, same-origin, in-memory port/token cache, `503` while pending — in `apps/web/vite.config.ts` (`research.md` Vite bootstrap endpoint hardening)
- [ ] T040 [P] [US1] Implement Graphite Signal design tokens — primitives → semantic → component CSS Custom Properties (`--cv-{category}-{role}-{state?}` naming, `DESIGN.md` §3–§4.3) for both Light and Dark themes, plus typed TypeScript accessors — in `apps/web/src/styles/tokens.css` and `apps/web/src/styles/tokens.ts` (FR-023, PLT-011, Constitution VII)
- [ ] T041 [US1] Implement theme selection (system-default, explicit Light/Dark override, persisted preference, no prolonged flash on switch, per `DESIGN.md` §14) in `apps/web/src/state/themeStore.ts` (Zustand — transient UI/layout state only, FR-024) and wire theme initialization in `apps/web/src/main.tsx` (depends on T040; shares `main.tsx` with T043/T045/T062 — sequential, not parallel; see File-Ownership Map)
- [ ] T042 [P] [US1] Implement `prefers-reduced-motion` handling — remove displacement/parallax/sequenced motion, use a short crossfade or instant change instead, per `DESIGN.md` §10 — in `apps/web/src/styles/motion.css` (FR-023, PLT-011)
- [ ] T043 [US1] Implement the QueryClient bootstrap — `QueryClient` instance and `QueryClientProvider` mounting — in `apps/web/src/lib/api/queryClient.ts` and `apps/web/src/main.tsx` (depends on T041 — same file, sequential; FR-024). **Note (traceability, corrected this pass)**: this task alone does not reach any backend behavior (FR-008) — it only instantiates and mounts the client; FR-008 is credited to T044, which actually performs a typed request.
- [ ] T044 [US1] Implement the typed HTTP client and TanStack Query hook for health, against the generated `HealthResponse` type, in `apps/web/src/lib/api/health.ts` (depends on T029, T043; FR-008, FR-024)
- [ ] T045 [US1] Implement the health/status frontend surface (Initial loading, Ready, Refreshing/Stale, Service unavailable/Offline, Error), consuming only `--cv-*` tokens (T040) and the reduced-motion rules (T042), in `apps/web/src/features/health/HealthStatus.tsx`, mounted in `apps/web/src/main.tsx` (depends on T040, T042, T044, and T043 — same file, sequential; FR-023, spec.md Frontend State Matrix)
- [ ] T046 [US1] Frontend tests, in `apps/web/src/features/health/HealthStatus.test.tsx` (depends on T045; FR-023, SC-010, PLT-011 — **corrected this pass so the task concretely requires, not merely labels, every behavior Principle VII cites**), covering explicitly:
  - One test group per applicable Frontend State Matrix state, each rendered/asserted under **both** Light and Dark themes (T040's `--cv-*` tokens) — not just one theme.
  - A reduced-motion test group that simulates `prefers-reduced-motion: reduce` (a `window.matchMedia` mock supplied directly in the test — jsdom does not implement `matchMedia` natively, so no new package is needed; this mirrors T042's own CSS media-query approach) and asserts: T042's motion rules take effect (no displacement/parallax/sequenced-animation class or transition applies), and the relevant state is still communicated **immediately**, without depending on any animation having completed or started.
  - Keyboard-navigation and visible-focus assertions (tab order reaches every interactive element; focus is visibly indicated via `--cv-*` tokens, never suppressed).
  - A non-color state-communication assertion — each state is distinguishable by text/icon/`aria-*` content alone, independent of any color-only cue.
  - The existing `jest-axe` accessibility assertion (`toHaveNoViolations`, T008's setup and its local Vitest module-augmentation declaration, corrected this pass, making the matcher concretely type-checkable; plain Testing-Library/DOM queries elsewhere, not jest-dom matchers).

  Uses only already-approved frontend dependencies (Vitest, Testing Library, jsdom, jest-axe); no visual-regression tool or other unapproved package is introduced.
- [ ] T047 [US1] Frontend test: displayed health data updates via TanStack Query invalidation/refetch; Zustand never becomes an authoritative source — appended to `apps/web/src/features/health/HealthStatus.test.tsx` (same file as T046 — not parallel) (FR-024, SC-011)
- [ ] T048 [US1] Browser E2E: load the frontend and observe the Ready state in `tests/e2e/health.e2e.ts` (`@converge/e2e`, WebdriverIO) (FR-016, AC-043)

**Checkpoint**: User Story 1 is functional and independently testable for SC-001/SC-002 (health-check path). **T025/T026 remain `PENDING`** — the launcher-shutdown/orphan-process guarantee cannot be declared complete until the human decision in "Pending / Blocked Items" is resolved.

---

## Phase 4: User Story 2 - Mechanically Verified Clean Architecture Boundaries (Priority: P2)

**Goal**: A repeatable, automated check proves Domain and Application have zero transport/UI/database/filesystem/PTY/Tauri/provider dependency, and fails visibly on a deliberately introduced violation.

**Independent Test**: Run the boundary check against Domain/Application in isolation; confirm it fails when an inward-pointing-rule violation is deliberately introduced (spec.md US2 Acceptance Scenarios 1–2).

### Tests for User Story 2 (MANDATORY — Constitution Principle VI, security-adjacent architecture boundary) ⚠️

- [ ] T049 [P] [US2] Architecture boundary check: shells out to `cargo metadata --format-version 1`, parses the JSON via `serde_json`, and asserts `domain`'s normal dependencies ⊆ `{thiserror}` and `application`'s ⊆ `{domain, thiserror, uuid, blake3, serde_json, time}` — checked only against these two production crates, never against the `converge-tests` harness package itself, which is architecturally inert by design (see "Test Harness Package" above) — plus `#![forbid(unsafe_code)]` presence at all six production crate roots, in `tests/architecture/allowlist_check.rs` (`converge-tests`, replacing its T018 stub) (FR-009, FR-010, SC-005)
- [ ] T050 [US2] Unit test the allowlist-check logic itself against a synthetic `cargo metadata` fixture containing a deliberately outward-pointing dependency, asserting the check reports a violation rather than passing silently — appended to the same `tests/architecture/allowlist_check.rs` (same file as T049 — not parallel) and `tests/architecture/fixtures/violation_metadata.json` (populates T018's empty fixture) (spec.md US2 Acceptance Scenario 2)

### Implementation for User Story 2

- [ ] T051 [US2] Wire `cargo test --workspace --lib` (isolated unit-test compile of every crate, `domain`/`application` included) and `cargo test -p converge-tests --test allowlist_check` into the `justfile` `test-unit` recipe (depends on T049; `justfile` is shared with T010/T079/T080 — this edit applies second in that sequence; FR-014, partial — wires one recipe, see traceability matrix)

**Checkpoint**: User Story 2 is independently functional and testable (SC-005) — `domain`/`application` compile in isolation, and the check fails visibly on a deliberate violation.

---

## Phase 5: User Story 5 - Minimal Authenticated WebSocket Proof (Priority: P2)

**Goal**: A WebSocket handshake is accepted only with a valid token, matching `Host`, and an allowlisted `Origin`, checked before any application-level exchange; the connection carries only a minimal `hello`/`ping`/`pong` proof.

**Independent Test**: Attempt a handshake with valid token/`Host`/allowlisted `Origin` (expect accept), then repeat with each invalidated individually (expect refusal before any message is exchanged) — spec.md US5 Acceptance Scenarios 1–3.

**Note**: Shares `crates/adapters-http` and `crates/service`'s router/token infrastructure with User Story 1; the implementation tasks below depend on User Story 1's router wiring (T035) being complete, even though this story's *test* is independently executable per spec.md.

**⚠️ Story-level blocker (this pass)**: T052–T054 are `PENDING` — see "Pending / Blocked Items" above (`futures-util` gap). Per Constitution VI, `T057`/`T058`/`T059` cannot honestly complete an RGR (test-fails-then-passes) cycle while their paired tests (`T053`/`T054`) cannot compile. This story's checkpoint below cannot be declared complete until the `futures-util` decision is resolved, even though `T055`–`T063` remain individually well-specified and are not blocked at the file/dependency level themselves.

### Tests for User Story 5 (MANDATORY — Constitution Principle VI, security boundary + contract) ⚠️

- [ ] T052 [P] [US5] **PENDING (blocked on unresolved `futures-util` dependency — see "Pending / Blocked Items" above)**: Contract test: `WsMessage` (`hello`/`ping`/`pong`) wire shapes and subprotocol negotiation response (only `converge.v1` echoed back) in `tests/contract/websocket_proof_contract.rs` (`converge-tests`, replacing its T018 stub) (FR-021, SC-009)
- [ ] T053 [P] [US5] **PENDING (blocked on unresolved `futures-util` dependency)**: Integration test: WS handshake denial — `Host` mismatch (400), `Origin` not in the allowlist (403), no valid token subprotocol (401), individually and in combination, never returns `101` — in `tests/integration/ws_auth_denial.rs` (FR-022, SEC-001, SEC-003, SC-009)
- [ ] T054 [P] [US5] **PENDING (blocked on unresolved `futures-util` dependency)**: Negotiation/redaction test: the token-bearing subprotocol value never appears in the handshake response header or in any captured log line — in `tests/integration/ws_redaction.rs` (FR-017)

### Implementation for User Story 5

- [ ] T055 [US5] Define the `WsMessage` enum (`Hello`/`Ping`/`Pong`, `tag = "type"`, `camelCase` fields) in `crates/contracts/src/ws.rs` (depends on T014, T019; makes T052 pass once T052 is unblocked)
- [ ] T056 [US5] Run `just contracts-generate` to produce `packages/contracts/src/generated/ws.ts` from `crates/contracts/src/ws.rs` and add its re-export to `packages/contracts/src/index.ts` (depends on T055, and on T029 — both append to the shared `packages/contracts/src/index.ts` barrel; T029 MUST land first, see File-Ownership Map; FR-019, FR-020)
- [ ] T057 [US5] Implement the WebSocket upgrade route `GET /api/ws` — `Host`→`Origin`-allowlist→token-subprotocol order, before any `101` response — in `crates/adapters-http/src/routes/ws.rs` (FR-022; depends on T035; RGR pairing with T053 is `PENDING` until T053 compiles)
- [ ] T058 [US5] Implement subprotocol negotiation (server selects only `converge.v1` in its response) in `crates/adapters-http/src/routes/ws.rs` (`research.md` Subprotocol negotiation; RGR pairing with T054 is `PENDING` until T054 compiles)
- [ ] T059 [US5] Implement the `hello`/`ping`/`pong` exchange handler (no terminal/PTY/provider streaming) in `crates/adapters-http/src/routes/ws.rs` (spec.md US5 Acceptance Scenario 3)
- [ ] T060 [US5] Register the `/api/ws` upgrade route into the Axum router in `crates/service/src/main.rs` — in `crates/service/src/main.rs` (depends on T057, T058, T059, and T037 — same file as T031/T035/T036/T037, sequential, see File-Ownership Map; FR-022 — router registration only; `SC-009`'s rejected-handshake evidence stays exclusively on T053/T057, removed from this task this pass)
- [ ] T061 [P] [US5] Implement the frontend WebSocket client (`converge.v1` + `converge.token.<token>` offer), against the generated `WsMessage` type, opened after the health bootstrap, in `apps/web/src/lib/api/ws.ts` (depends on T056; FR-008)
- [ ] T062 [US5] Implement the frontend WebSocket-proof bootstrap: after the health surface reaches Ready (T045), open the authenticated connection via T061's client, and on receiving `hello`, record a plain, non-secret, test-observable readiness signal (e.g., a stable `data-testid` element/attribute) — in `apps/web/src/lib/api/wsBootstrap.ts` and `apps/web/src/main.tsx` (depends on T045, T060, T061 — `main.tsx` shared with T041/T043/T045, sequential, see File-Ownership Map; FR-008 — the implementation that reaches the backend behavior; distinct from T063's behavioral validation, corrected this pass)
- [ ] T063 [US5] Extend the browser E2E journey to confirm the WS proof (`hello` received, observed via T062's test-observable signal) — appended to `tests/e2e/health.e2e.ts` (depends on T048, T062 — same file as T048, sequential, not parallel) (FR-016 — the concrete behavioral validation itself, corrected this pass to no longer be shared with T062's implementation)

**Checkpoint**: User Story 5's WebSocket boundary and vertical slice (route registration, frontend consumer) are fully specified, but the story **cannot be declared independently testable and passing** while T052–T054 remain `PENDING`.

---

## Phase 6: User Story 3 - Idempotent Event Persistence and Aggregate Rebuild (Priority: P3)

**Goal**: A representative normalized event persists idempotently — resubmission never duplicates the ledger — and the derived aggregate can be discarded and rebuilt from the ledger alone without mutating it.

**Independent Test**: Submit a representative event twice, confirm one ledger row and one equivalent aggregate; discard the aggregate, rebuild it from the ledger, confirm equivalence (spec.md US3 Acceptance Scenarios 1–3).

**Clean Architecture note (corrected across passes)**: `RebuildAggregate` (application) MUST contain no SQL, transaction keywords, or `sqlx` types. It orchestrates exclusively through an explicit, transaction-owning `AggregateRepository::rebuild` port method (T073) that accepts a pure fold function (T074) as a parameter. `BEGIN IMMEDIATE`, `DELETE`, `SELECT`, `INSERT`, `COMMIT`/`ROLLBACK`, and every `sqlx` type are owned exclusively by `adapters-persistence` (T078), which calls the injected pure function *inside* its own open transaction. The port method is a native `async fn` in a trait (stable since Rust 1.75, present in Edition 2024, no `async-trait` crate or other unapproved async-helper dependency); call sites use static/generic dispatch (`impl AggregateRepository` / a generic type parameter), never `dyn AggregateRepository`, since native async-fn-in-trait is not `dyn`-compatible without additional boxing this slice does not need.

**Corrected this pass — fallible fold**: the pure fold contract was previously infallible (`Fn(&[Event]) -> Vec<AggregateProjection>`), which cannot express `T070`'s already-required "an injected mid-fold failure leaves `aggregates` unchanged (rollback)" behavior without either panicking (an error channel this slice's Rust quality baseline does not sanction) or silently producing a wrong result. The contract is now typed and fallible: `Fn(&[Event]) -> Result<Vec<AggregateProjection>, FoldError>`. `FoldError` and a `RepositoryError::Fold(FoldError)` variant are defined alongside the port (T073, `thiserror`-derived, no new dependency). `adapters-persistence` (T078) receives this `Result` from the closure, and on `Err`, issues `ROLLBACK` and propagates the typed error — it never inspects or reinterprets the error's payload, only relays it. `application` still contains no SQL and no `sqlx` type; static dispatch (no `dyn`) is preserved, matching the async-fn-in-trait constraint above.

### Tests for User Story 3 (MANDATORY — Constitution Principle VI, domain logic + application use cases + persistence boundary) ⚠️

- [ ] T064 [P] [US3] Domain unit test: Normalized Event / Aggregate invariants in `crates/domain/src/event.rs` and `crates/domain/src/aggregate.rs` (test modules)
- [ ] T065 [P] [US3] Application unit test: `RecordProbeEvent` — first submission succeeds; identical resubmission is idempotent (no duplicate); resubmission differing in any field returns `IdempotencyConflict` naming the exact `ConflictingFields` — in `crates/application/src/use_cases/record_probe_event.rs` (test module) (FR-012)
- [ ] T066 [P] [US3] Application unit test: the **pure**, now-fallible `fold_events` function — given an in-memory slice of `Event` values (no I/O, no mock repository needed), asserts correct per-`event_type` projection counts and `last_event_id` on the `Ok` path, and asserts a malformed/unrecognized event input yields `Err(FoldError::...)` — never a panic — on the failure path — in `crates/application/src/use_cases/rebuild_aggregate.rs` (test module) (FR-013)
- [ ] T067 [US3] Application unit test (`#[tokio::test]`): `RebuildAggregate` orchestration — using a fake/in-memory `AggregateRepository` port implementation whose `rebuild` method is itself a trivial `async fn`, asserts (a) on the success path, the use case calls `repository.rebuild(fold_events)` exactly once, awaits it, and returns the port's result verbatim, and (b) on a fake `rebuild` that returns `Err(RepositoryError::Fold(...))` (simulating an adapter that already rolled back after a fold failure), the use case propagates that error verbatim rather than swallowing or panicking on it; asserts (by construction — the use case has no SQL-capable dependency available to it) that it cannot itself issue any transaction or SQL statement — appended to the same `crates/application/src/use_cases/rebuild_aggregate.rs` (test module; same file as T066 — not parallel) (FR-013)
- [ ] T068 [P] [US3] Migration test, **expanded this pass with a negative case for SC-007's "broken migration" category**: (a) fresh empty database → migrations apply → expected schema/triggers exist; re-run is a no-op; `BEFORE UPDATE`/`BEFORE DELETE` triggers `RAISE(ABORT, ...)`; (b) **new**: after migrations are applied, mutate the on-disk content of an already-applied migration file (simulating an edited/incompatible migration) and assert that re-running `sqlx::migrate::Migrator` against the same database returns a typed `MigrateError` (SQLx's own built-in applied-migration checksum verification, `sqlx@0.9.0`, no new dependency) rather than silently succeeding or panicking — in `tests/integration/migrations.rs` (`converge-tests`, replacing its T018 stub) (FR-011, FR-018, AC-042)
- [ ] T069 [P] [US3] Integration test: sequential and concurrent identical resubmission → exactly one row/aggregate; concurrent resubmission differing only in `event_type` / only in `payload` / only in `occurred_at` → each surfaces `IdempotencyConflict` — in `tests/integration/idempotency_conflict.rs` (FR-012, SC-003)
- [ ] T070 [P] [US3] Integration test: an injected fold that deterministically returns `Err(FoldError::...)` (not a panic) leaves `aggregates` unchanged (rollback), asserted against the real `adapters-persistence` transaction and its real `ROLLBACK` (this is the one test in this story that exercises the actual `BEGIN IMMEDIATE`/`ROLLBACK` behavior, confirming it — not `application` — owns it); a concurrent `events` insert blocks while a rebuild transaction is open — in `tests/integration/transactional_rebuild.rs` (FR-013, SC-004)

### Implementation for User Story 3

- [ ] T071 [US3] Write migration `0001_create_events_and_aggregates.sql` (`events`/`aggregates` tables, `payload_hash` column, `BEFORE UPDATE`/`BEFORE DELETE` immutability triggers) in `migrations/0001_create_events_and_aggregates.sql` (FR-011; makes T068 pass)
- [ ] T072 [US3] Define the Normalized Event / Aggregate domain entities in `crates/domain/src/event.rs`, `crates/domain/src/aggregate.rs` (makes T064 pass)
- [ ] T073 [US3] Define `EventRepository`/`AggregateRepository` outbound ports (traits), `FoldError`, and `RepositoryError` in `crates/application/src/ports.rs`, making the transaction-owning contract explicit, asynchronous, and fallible — **corrected this pass**: the fold contract is now typed and fallible, matching `T070`'s already-required rollback behavior. Uses Rust's native `async fn` in traits (stable since Rust 1.75, present in Edition 2024 — no `async-trait` crate or other unapproved async-helper dependency):
  ```rust
  #[derive(Debug, thiserror::Error)]
  pub enum FoldError {
      #[error("malformed event encountered while folding: {0}")]
      MalformedEvent(String),
  }

  #[derive(Debug, thiserror::Error)]
  pub enum RepositoryError {
      // ... existing variants ...
      #[error("aggregate fold failed: {0}")]
      Fold(#[from] FoldError),
  }

  pub trait AggregateRepository {
      async fn rebuild(
          &self,
          fold: impl Fn(&[Event]) -> Result<Vec<AggregateProjection>, FoldError> + Send,
      ) -> Result<(), RepositoryError>;
  }
  ```
  The port's single method is the *only* place a rebuild can be triggered from `application`, and its signature accepts the pure, fallible fold logic as a parameter rather than exposing any SQL-shaped operation (`SELECT`/`DELETE`/`INSERT`) as separate port methods `application` could sequence itself. Call sites use static/generic dispatch, never `dyn AggregateRepository`, since native async-fn-in-trait is not `dyn`-compatible without boxing this slice does not need. No new dependency is introduced for either the `async fn` or the `Result`-based fold contract.
- [ ] T074 [US3] Implement the **pure** `fold_events(events: &[Event]) -> Result<Vec<AggregateProjection>, FoldError>` function — no I/O, no SQL, no port dependency, no panics as an error channel; remains synchronous (only the port method and use case that call it are async, per T073's signature) — in `crates/application/src/use_cases/rebuild_aggregate.rs` (depends on T073's `FoldError`; makes T066 pass)
- [ ] T075 [US3] Implement the `RecordProbeEvent` use case — producer-supplied UUIDv7 `event_id` passthrough, canonical `serde_json` payload serialization, BLAKE3 `payload_hash`, RFC 3339 `occurred_at` parse/validate, three-outcome conflict classification — in `crates/application/src/use_cases/record_probe_event.rs` (depends on T073; makes T065 pass)
- [ ] T076 [US3] Implement the `RebuildAggregate` use case as an `async fn` that calls `repository.rebuild(fold_events).await` and returns its result verbatim (any `FoldError` the port surfaces arrives already wrapped in `RepositoryError` per T073's contract, so this use case needs no fold-error-specific branch — it simply propagates); contains no SQL, no transaction keyword, no `sqlx` type — in `crates/application/src/use_cases/rebuild_aggregate.rs` (depends on T073, T074; makes T067 pass)
- [ ] T077 [P] [US3] Implement the SQLx `EventRepository` (`INSERT` + `UNIQUE`-conflict re-`SELECT`/classify, `async fn` methods using SQLx's async pool) in `crates/adapters-persistence/src/event_repository.rs` (depends on T071, T073; makes T069 pass; mutually parallel with T078 — different files)
- [ ] T078 [P] [US3] Implement the SQLx `AggregateRepository::rebuild` as an `async fn` (matching T073's async, fallible port signature) — owns `BEGIN IMMEDIATE` → `DELETE FROM aggregates` → `SELECT * FROM events ORDER BY id ASC` → calls the caller-supplied `fold` closure on the fetched events → on `Ok(projections)`, `INSERT` each projection then `COMMIT`; on `Err(fold_error)`, issues `ROLLBACK` and returns `Err(RepositoryError::Fold(fold_error))` without inserting anything — exclusively, using SQLx's async pool — in `crates/adapters-persistence/src/aggregate_repository.rs` (depends on T071, T073; makes T070 pass; mutually parallel with T077 — different files)
- [ ] T079 [US3] Pin `sqlx-cli` install (`cargo install sqlx-cli --version 0.9.0 --locked`) in the `justfile` `setup` recipe, and **generate/update** (not commit) the `.sqlx/` offline query cache via `cargo sqlx prepare` against a running local dev database — staging and committing `crates/adapters-persistence/.sqlx/` is left entirely to the human maintainer (Constitution X) — `justfile`, `crates/adapters-persistence/.sqlx/` (generated artifact, not staged/committed by this task; `justfile` is shared with T010/T051/T080 — this edit applies third in that sequence; FR-014, partial — wires `sqlx-cli`/cache generation into `setup`)

**Checkpoint**: User Story 3 is independently functional and testable (SC-003, SC-004) — the ledger is idempotent, the aggregate is rebuildable without mutating it, `application` contains no SQL anywhere in the rebuild path, and the fold contract is now concretely fallible with a real, tested rollback path.

---

## Phase 7: User Story 4 - Command-Facade Quality Gates and Base CI (Priority: P4)

**Goal**: Formatting, linting, tests, and builds run identically through the `justfile` locally and in CI on every pull request and push, on Linux fully and Windows x86_64 for build plus applicable technical tests.

**Independent Test**: Run the documented format/lint/test/build recipes locally and confirm they pass; confirm the same recipes run in CI, and a deliberately broken check fails CI visibly (spec.md US4 Acceptance Scenarios 1–3).

**Note**: This story verifies the outcomes of User Stories 1, 2, 3, and 5 rather than introducing new runtime behavior — its tasks depend on those stories' recipes (`test-unit`, `test-contract`, `test-integration`, `test-frontend`, `test-e2e`) having real content to run.

### Implementation for User Story 4

- [ ] T080 [US4] Complete every `justfile` recipe to its full defined behavior per `plan.md`'s Command Facade table, with `check` as the unconditional complete gate (including `test-e2e`, no environment-dependent skip); `test-contract` runs `cargo test -p converge-tests --test http_health_contract --test websocket_proof_contract` plus `just contracts-check`; `test-integration` runs `cargo test -p converge-tests` filtered to every `integration/*` target listed in T018; `test-frontend` runs `pnpm --filter @converge/web test`; `test-e2e` runs `pnpm --filter @converge/e2e wdio run wdio.conf.ts` — `justfile` (depends on T051, T079, and every User Story 1/3/5 test/implementation task; this is the fourth and final sequential edit to `justfile`; FR-014 — completes the local recipe surface; the CI-level SC/AC/PLT identifiers this recipe surface supports are credited to T081, which alone actually runs it in CI, not to this task — corrected this pass)
- [ ] T081 [P] [US4] Add `.github/workflows/ci.yml` with **both** CI jobs in this one file: a **Linux job** running `just check` unconditionally on `ubuntu-24.04`, and a **Windows job** running `just fmt-check`, `just lint`, `just test-unit`, `just test-contract`, `just test-integration`, `just build` on `windows-2022` (no E2E step, per `PLT-005`) — both triggered on `pull_request` and `push` to the default branch, using `actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`, `actions/setup-node@820762786026740c76f36085b0efc47a31fe5020 # v7.0.0` (`node-version-file: .nvmrc`), `dtolnay/rust-toolchain@2c7215f132e9ebf062739d9130488b56d53c060c # 2026-07-16` (`toolchain: "1.97.1"`, `components: "rustfmt, clippy"`) (FR-015, PLT-005, PLT-009, SC-006)
- [ ] T082 [P] [US4] Add a Dependabot/Renovate config to keep the GitHub Actions SHA pins current under human review in `.github/dependabot.yml` (`research.md` GitHub Actions Enforcement)
- [ ] T083 [US4] Document the manual, human-performed CI-break verification runbook — a human maintainer pushes a throwaway branch/PR with one deliberately failing check, confirms `just check` and CI both fail visibly, then closes/deletes the throwaway branch — appended to `specs/001-slice-0-foundation/quickstart.md` under a new "Manual CI-Break Verification (human-performed)" heading. **This task is documentation only; the agent completing it MUST NOT execute `git push`, open a pull request, commit a deliberate break, or revert Git history** (Constitution X, AGENTS.md Git and Release Boundaries). Documenting the runbook is not, by itself, evidence that `SC-007` holds — see T084 (corrected this pass, split from the previous single-task treatment of this acceptance scenario)
- [ ] T084 [US4] **Human-maintainer checkpoint — not implementable or executable by an agent (new this pass)**. The human maintainer actually performs T083's documented runbook: pushes a throwaway branch/PR with one deliberately failing check, observes both `just check` locally and the CI workflow (T081) fail visibly, then closes/deletes the throwaway branch — and records the evidence (which check was broken, confirmation that both local and CI runs reported failure, and that the throwaway branch/PR was closed) in `specs/001-slice-0-foundation/quickstart.md`, appended immediately below T083's runbook heading, under a new "CI-Break Verification Evidence (human-supplied)" subsection, dated and attributed to the maintainer who ran it. **`SC-007` is `PENDING` until this evidence entry exists** — it is not satisfied merely by T083's runbook being documented. The agent completing this task entry MUST NOT execute the intentionally failing Git/CI workflow itself, MUST NOT create or push a throwaway branch, and MUST NOT run `git add`, `commit`, `push`, `merge`, `tag`, `rebase`, or `reset` — this checkbox stays unchecked until a human maintainer supplies the evidence directly (Constitution X) (`quickstart.md`; FR-018, SC-007)

**Checkpoint**: User Story 4's local/CI gate is fully wired and specified (SC-006, T081). **SC-007 remains `PENDING`** until T084's human-supplied evidence exists — the story is not considered fully verified until then.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Final, whole-slice verification after every story is complete.

- [ ] T085 [P] Run `just contracts-check` end-to-end and confirm zero diff between regenerated and committed TypeScript in `packages/contracts/src/generated/` (FR-020, SC-008)
- [ ] T086 [P] Confirm `just fmt-check` and `just lint` pass repository-wide (FR-014)
- [ ] T087 Run `specs/001-slice-0-foundation/quickstart.md` validation end-to-end (Launch, Security Boundary, Persistence, Architecture Boundaries, Complete Gate) and, separately, confirm both (a) T083's manual CI-break runbook document exists and is correct, and (b) T084's human-supplied evidence entry exists. **This task validates only that both exist and are correctly written** — the agent performing it MUST NOT execute the intentionally failing Git/CI workflow itself (no push, no PR, no revert); `SC-007` may be reported as satisfied in this task's output only if T084's evidence entry is actually present, never inferred, simulated, or fabricated by the agent. Any claim about local staged/committed/pushed state in this task's report MUST cite the current `git status` output as evidence, not be asserted from memory (FR-014, FR-016, FR-018; SC-006)
- [ ] T088 [P] Update `README.md` with the one documented launch command, only if it differs from `quickstart.md`'s `just dev` (FR-014)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately.
- **Foundational (Phase 2)**: Depends on Setup — BLOCKS all user stories.
- **User Stories (Phases 3–7)**: All depend on Foundational completion.
  - **US1 (Phase 3, P1)**: No dependency on any other story. T025/T026 are `PENDING` (see "Pending / Blocked Items").
  - **US2 (Phase 4, P2)**: No dependency on any other story — touches only `tests/architecture/` and the `justfile` (second sequential edit).
  - **US5 (Phase 5, P2)**: Independently *testable* per spec.md, but its implementation tasks (T057–T060, T062) depend on US1's router wiring (T035, T037) because they live in `crates/adapters-http`/`crates/service`, and T062 depends on US1's `HealthStatus` mount (T045) because it shares `apps/web/src/main.tsx`. T052–T054 are `PENDING` (`futures-util` gap), which transitively blocks the story's checkpoint per Constitution VI.
  - **US3 (Phase 6, P3)**: No dependency on US1/US2/US5 — touches `crates/domain`, `crates/application`, `crates/adapters-persistence`, `migrations/`, and the `justfile` (third sequential edit).
  - **US4 (Phase 7, P4)**: Depends on US1, US2, US3, and US5 all being complete — it wires and verifies the gates that exercise their recipes (`justfile`'s fourth and final sequential edit); ordered last per spec.md. T084 additionally depends on T083 (same file, sequential) and cannot be closed by an agent.
- **Polish (Phase 8)**: Depends on every desired user story being complete.

### File-Ownership Map (for parallel safety)

| Story | Files it owns |
| --- | --- |
| US1 | `crates/contracts/src/health.rs`, `packages/contracts/src/generated/health.ts` + its line in `src/index.ts`, `crates/service/*` (router/token/main.rs — shared with US5's T060, see below), `crates/adapters-http/src/middleware/*`, `crates/adapters-http/src/routes/health.rs`, `scripts/dev.mjs`, `apps/web/vite.config.ts`, `apps/web/src/styles/*`, `apps/web/src/state/themeStore.ts`, `apps/web/src/lib/api/queryClient.ts`, `apps/web/src/lib/api/health.ts`, `apps/web/src/features/health/*`, `apps/web/src/main.tsx` (sequential internal edits T041→T043→T045, then US5's T062 — see below), `tests/integration/http_*.rs`, `tests/integration/launcher_shutdown.rs`, `tests/integration/orphan_process.rs`, `tests/integration/redaction.rs`, `tests/integration/concurrent_instances.rs`, `tests/integration/service_restart.rs`, `tests/contract/http_health_contract.rs`, `tests/e2e/health.e2e.ts` (extended by US5's T063) |
| US2 | `tests/architecture/*` |
| US5 | `crates/contracts/src/ws.rs`, `packages/contracts/src/generated/ws.ts` + its line in `src/index.ts` (after US1's T029 — see below), `crates/adapters-http/src/routes/ws.rs`, `crates/service/src/main.rs` (T060, after US1's T037 — see below), `apps/web/src/lib/api/ws.ts`, `apps/web/src/lib/api/wsBootstrap.ts`, `apps/web/src/main.tsx` (T062, after US1's T045 — see below), `tests/integration/ws_*.rs` (T052-T054's `PENDING` status), `tests/contract/websocket_proof_contract.rs` (`PENDING`), `tests/e2e/health.e2e.ts` (T063, after US1's T048) |
| US3 | `crates/domain/src/event.rs`, `crates/domain/src/aggregate.rs`, `crates/application/src/use_cases/*`, `crates/application/src/ports.rs`, `crates/adapters-persistence/src/*`, `migrations/*`, `tests/integration/migrations.rs`, `tests/integration/idempotency_conflict.rs`, `tests/integration/transactional_rebuild.rs` |
| US4 | `.github/workflows/ci.yml` (single task, T081 — not split across two falsely-parallel tasks), `.github/dependabot.yml`, `specs/001-slice-0-foundation/quickstart.md` (T083, then T084 — sequential, same file) |

**Cross-story shared files (not disjoint — explicit sequencing required, not parallel):**

- `justfile`: created once in Setup (T010, skeleton), then edited sequentially by US2 (T051), US3 (T079), and US4 (T080) in that exact order, regardless of which stories otherwise run in parallel.
- `tests/Cargo.toml`: created once, fully, in Foundational (T018) with every `[[test]]` target pre-declared; no story ever edits this file again — each story only replaces the body of its own already-declared stub `.rs` file(s).
- `apps/web/package.json`: T005 is the sole owner of its `@converge/contracts` workspace dependency line; T008 only appends devDependencies to the same file (append-only, no conflicting edit — the two are sequenced, T008 depends on T005, and only T005 carries `[P]`).
- `packages/contracts/src/index.ts`: created by T006 (Setup, hand-written barrel), then appended sequentially by US1's T029 and US5's T056, in that exact order.
- `apps/web/src/main.tsx`: edited sequentially within US1 by T041 (theme init) → T043 (QueryClient provider) → T045 (mount `HealthStatus`), then by US5's T062 (WS bootstrap); none of the four is marked `[P]` against the others.
- `crates/service/src/main.rs`: edited sequentially within US1 by T031 → T035 → T036 → T037, then by US5's T060 (register `/api/ws`).
- `specs/001-slice-0-foundation/quickstart.md`: appended sequentially by US4's T083 (runbook) then T084 (human-supplied evidence), then read (not edited) by T087.

### Within Each User Story

- Per Constitution Principle VI: tests for domain logic, application use cases, contracts, and security boundaries MUST be written and FAIL before their paired implementation task (Red-Green-Refactor); frontend/adapter/infrastructure tests are included in the same change without requiring test-first ordering. Where a paired test is `PENDING` (T025, T026, T052–T054), the paired implementation task cannot honestly be marked RGR-complete either, even if it is otherwise implementable.
- Contracts/wire types before the routes that use them; `contracts-generate` before any frontend code imports the generated type.
- Ports/traits before the repositories that implement them and before the use cases that call them.
- Router wiring before route handlers that register into it, and before any frontend consumer that depends on the route being live.
- Story complete (checkpoint) before its dependents (US4) begin.

### Parallel Opportunities

- All `[P]` Setup tasks (T002–T007, T009) can run in parallel; T008 depends on T005 (same file); T010/T011 depend on the rest of Setup existing.
- All `[P]` Foundational tasks (T012–T018) can run in parallel; T019 depends on all seven Rust packages (six crates + test harness) existing.
- Once Foundational completes, **US1, US2, and US3 can start in parallel** (disjoint files, `justfile`'s sequencing aside). US5 can start in parallel for its test tasks (T052–T054, though `PENDING`) but its implementation tasks (T057–T060, T062) must wait for US1's T035/T037/T045.
- Within US1: all test tasks (T020–T027) in parallel; among implementation tasks, T038, T039, T040, T042 are mutually parallel and parallel with the Rust-side tasks; T041/T043/T045 are sequential (shared `main.tsx`).
- Within US5: T052–T054 in parallel with each other (though all `PENDING`); T061 (`ws.ts`) is parallel with the Rust-side tasks (T057–T060); T060 and T062 are each sequential within their own shared file (`main.rs`, `main.tsx` respectively), not with each other.
- Within US3: all test tasks except T067 (T064, T065, T066, T068, T069, T070) in parallel; T067 is sequential after T066 (shared file); T077 and T078 both depend on T071/T073 but are mutually parallel (different files).
- US4 cannot start until US1, US2, US3, and US5 are all at their checkpoints, and `justfile`'s prior three sequential edits (T010, T051, T079) are done. T084 cannot close until a human maintainer supplies its evidence.

---

## Parallel Example: User Story 1 Tests

```bash
Task: "Contract test for HealthResponse/HealthStatus/ApiError in tests/contract/http_health_contract.rs"
Task: "Integration test for HTTP denial paths in tests/integration/http_auth_denial.rs"
Task: "Integration test for CORS preflight in tests/integration/http_cors_preflight.rs"
Task: "Integration test for service restart in tests/integration/service_restart.rs"
Task: "Integration test for concurrent instances in tests/integration/concurrent_instances.rs"
Task: "Integration test for launcher shutdown in tests/integration/launcher_shutdown.rs (PENDING — see Pending / Blocked Items)"
Task: "Integration test for orphan process in tests/integration/orphan_process.rs (PENDING — see Pending / Blocked Items)"
Task: "Redaction test in tests/integration/redaction.rs"
```

## Parallel Example: User Story 3 Tests

```bash
Task: "Domain unit test for Event/Aggregate invariants in crates/domain/src/event.rs"
Task: "Application unit test for RecordProbeEvent in crates/application/src/use_cases/record_probe_event.rs"
Task: "Application unit test for pure, fallible fold_events in crates/application/src/use_cases/rebuild_aggregate.rs"
Task: "Migration test in tests/integration/migrations.rs"
Task: "Idempotency/conflict integration test in tests/integration/idempotency_conflict.rs"
Task: "Transactional rebuild integration test (fold failure -> rollback) in tests/integration/transactional_rebuild.rs"
```

Note: the `RebuildAggregate` orchestration test (T067) is deliberately **not** in this parallel batch — it shares `rebuild_aggregate.rs` with the pure `fold_events` test (T066) above and must run after it.

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational (blocks all stories).
3. Complete Phase 3: User Story 1, noting T025/T026 remain `PENDING`.
4. **STOP and VALIDATE**: run `tests/integration/http_auth_denial.rs`, `tests/integration/http_cors_preflight.rs`, and `tests/e2e/health.e2e.ts` against a `just dev` run; confirm SC-001/SC-002 independently.

### Incremental Delivery

1. Setup + Foundational → foundation ready.
2. Add User Story 1 → validate independently (MVP: authenticated, launchable service); T025/T026 remain open pending the PID-mechanism decision.
3. Add User Story 2 and User Story 3 in parallel → validate each independently (architecture boundary; ledger/aggregate mechanism, now with a concretely fallible, rollback-tested fold). Coordinate `justfile` edits sequentially even though the rest of each story is parallel.
4. Add User Story 5 → its route/router/frontend tasks (T055–T063) can be implemented, but the story cannot be declared checkpoint-complete until the `futures-util` decision resolves T052–T054.
5. Add User Story 4 last → validate the complete local/CI gate against the now-complete recipes; T084 remains open until a human maintainer supplies SC-007's evidence.
6. Complete Phase 8: Polish (contract drift, format/lint, full quickstart replay); T087 reports SC-007 as satisfied only if T084's evidence exists.

### Parallel Team Strategy

With multiple contributors, after Foundational completes:

- Contributor A: User Story 1 (then User Story 5, since it depends on US1's router and health mount).
- Contributor B: User Story 2.
- Contributor C: User Story 3.
- `justfile` edits (T051, T079, T080) MUST be coordinated sequentially between B, C, and whoever takes US4, even though the rest of their stories proceed independently.
- Whoever finishes first takes User Story 4 once US1, US2, US3, and US5 are all at their checkpoints (noting US5's checkpoint is conditional on the `futures-util` decision).

---

## Task-to-Requirement Traceability Matrix

Full task-level cross-reference to `traceability.md`'s FR/SC/AC/PLT/SEC identifiers. IDs marked **out of scope** in `traceability.md` (`AC-036`, `AC-037`, `SEC-002`, `SEC-004`, `PLT-003`, `PLT-006`, `PLT-007`, `SPEC-002`–`SPEC-006`) are intentionally addressed by **no task below**.

**Re-audited this pass**: every row was re-derived task by task against `traceability.md`'s own FR→AC/PLT and SC→FR mappings. Rows that assigned a requirement merely because a task supports nearby infrastructure, or that assumed every task in a range independently provides the same coverage, are corrected. Each row is tagged with its evidence type: **[impl]** production code, **[test]** automated test, **[infra]** scaffolding/tooling with no behavioral claim, **[doc]** documentation, **[human]** requires human-supplied evidence.

**`SC-007` now has concrete, executable planned coverage for all four required categories (this pass — fifth revision)** — spec.md's SC-007 covers visibility of failures in denied authentication, architectural-boundary violation, broken migration, and broken test. The fourth revision correctly split this into "automated category coverage" vs. "human checkpoint" but left the migration category with no automated negative case at all, which is corrected here. As before, `SC-007` is credited as a traceability-matrix identifier **only to `T084`** (not added to any further row — doing so would re-introduce the exact inflation this document otherwise corrects), and the other three categories are cited narratively:

1. **Denied authentication** — `T021`/`T032` (HTTP auth-denial detection, executable now) and `T053`/`T057` (WS auth-denial detection; `PENDING` only on the separate, already-disclosed `futures-util` decision, not a gap in planned coverage itself).
2. **Architectural-boundary violation** — `T049`/`T050`, with `T050`'s synthetic-violation fixture concretely proving the check reports a violation rather than passing silently.
3. **Broken migration** — **new this pass**: `T068`, expanded to include a deliberate migration-checksum-mismatch case (mutating an already-applied migration file's content and asserting `sqlx::migrate::Migrator` returns a typed `MigrateError` rather than succeeding silently or panicking), using SQLx's own built-in applied-migration integrity check — no new dependency. This closes the gap the fourth revision explicitly disclosed as unautomated.
4. **Broken test / gate-visibility (`T084`)** — the sole task the `SC-007` identifier is credited to below. It is the human-maintainer checkpoint proving the complete local (`just check`) and CI (`T081`'s workflow) gates make an intentionally introduced failure visible end-to-end. **Its own execution remains a later, human-only, post-implementation checkpoint** — the checkbox stays unchecked and no evidence is fabricated here — but, corrected this pass, that non-execution does not by itself block the Constitution's planning-gate evaluation of `SC-007` (see Principle IX below): the planning gate asks whether all four categories have concrete, well-specified, executable-when-run planned coverage, which they now do, not whether every human-performed step has already happened.

| Task(s) | Type | FR | SC | AC | PLT / SEC | Constitution |
| --- | --- | --- | --- | --- | --- | --- |
| T001, T003, T004, T006, T007, T009, T011 | [infra] | — | — | — | — | I, IX |
| T002 | [infra] | — | — | — | — | I *(pins dependencies only)* |
| T005 | [infra] | — | — | — | — | I |
| T008 | [infra] | — | — | — | — | VI, VII *(test tooling; enables T046's `jest-axe` assertions)* |
| T010 | [infra] | — | — | — | — | IX *(skeleton only, no recipe has real behavior yet)* |
| T012, T013 | [impl] | FR-009 | — | AC-041 | — | II *(the two manifests FR-009 actually names — Domain and Application; manifest configuration only, no automated check runs here — `SC-005`'s automated-boundary-check evidence stays exclusively on T049/T050, corrected this pass)* |
| T014, T015, T016, T017 | [infra] | — | — | — | — | II *(the other four crates' manifests; structural, not FR-009's own subject)* |
| T018 | [infra] | — | — | — | — | II *(creates the harness package; not requirement evidence by itself — corrected this pass, no longer grouped with T049/T050)* |
| T019 | [infra] | — | — | — | — | II, III *(unsafe-code prohibition + module skeletons)* |
| T020 | [test] | FR-021 | — | AC-043 | — | IV |
| T028 | [impl] | FR-019 | — | AC-043 | — | IV |
| T029 | [impl] | FR-019 | — | AC-043 | — | IV *(runs `contracts-generate`; `FR-020`'s automated drift-failure check and `SC-008`'s zero-drift evidence both stay exclusively on T085 — `FR-020` removed this pass, `SC-008` removed the prior pass)* |
| T021, T032 | [test]/[impl] | FR-004, FR-006 | SC-002 | AC-034 | SEC-003 | III |
| T022, T033 | [test]/[impl] | FR-005 | SC-002 | AC-034 | SEC-003 | III |
| T023 | [test] | FR-003 | SC-002 | AC-033 | SEC-001 | III |
| T024 | [test] | FR-001, FR-002 | SC-001 | AC-033 | SEC-003 | III |
| T025 | [test], **PENDING** | FR-001, FR-018 | — | AC-039 | — | III (blocked — see Pending / Blocked Items) *(cooperative-shutdown test; neither the one-command-launch criterion `SC-001` nor the CI-failure-visibility criterion `SC-007` — removed this pass)* |
| T026 | [test], **PENDING** | FR-001, FR-018 | — | AC-039 | — | III (blocked — see Pending / Blocked Items) *(orphan-process test; not `SC-001` — removed this pass, corrected in the same audit that removed `SC-007` from T036/T037)* |
| T036, T037 | [impl] | FR-001, FR-018 | — | AC-039 | — | III *(shutdown/watchdog implementation; not `SC-007` evidence — lifecycle implementation does not itself validate deliberate CI-failure visibility, removed this pass)* |
| T027 | [test] | FR-017 | — | AC-038 | SEC-008 | VIII |
| T030 | [impl] | FR-003 | — | AC-033 | SEC-001 | III |
| T031 | [impl] | FR-001, FR-002 | SC-001 | AC-033, AC-040 | PLT-001 (restricted), PLT-004 | III |
| T034 | [impl] | FR-007 | — | AC-033 | — | — |
| T035 | [impl] | FR-007 | SC-001, SC-002 | AC-040 | — | II |
| T038 | [impl] | FR-001 | SC-001 | AC-040 | PLT-001 (restricted) | III |
| T039 | [impl] | FR-001 | SC-001 | AC-040 | — | III |
| T040, T042 | [impl] | FR-023 | — | — | PLT-011 | VII *(builds the tokens/motion CSS this behavior consumes; `SC-010`'s measured WCAG checks are credited exclusively to T046, removed from here this pass — building the related UI is not itself validating it)* |
| T041 | [impl] | FR-024 | — | — | PLT-004 | VII *(implements theme selection/persistence; `SC-011` measures TanStack-Query-vs-Zustand data freshness, unrelated to theme state and not validated by this task — removed this pass)* |
| T043 | [impl] | FR-024 | — | — | PLT-004 | II *(mounts `QueryClient`; does not itself reach backend behavior — `FR-008` removed the prior pass — nor validate `SC-011`'s observable-update behavior, credited exclusively to T047 — removed this pass)* |
| T044 | [impl] | FR-008, FR-024 | — | AC-035 | PLT-004 | II *(the task that concretely reaches the backend through a typed client; `SC-011`'s measured refetch-update behavior is credited exclusively to T047, removed from here this pass)* |
| T045 | [impl] | FR-023 | — | — | PLT-011 | VII *(implements the Frontend State Matrix; `SC-010`'s measured WCAG checks are credited exclusively to T046, removed this pass)* |
| T046 | [test] | FR-023 | SC-010 | — | PLT-011 | VII *(corrected this pass to concretely require both themes, simulated reduced motion, keyboard/focus, non-color state, and `jest-axe` — not merely labeled)* |
| T047 | [test] | FR-024 | SC-011 | — | PLT-004 | II |
| T048 | [test] | FR-016 | — | AC-043 | PLT-008 | VI |
| T049 | [test] | FR-010 | SC-005 | AC-041 | — | II, VI *(the actual check, run against the real crates — `FR-009` itself is satisfied by T012/T013's manifests, not by this check, removed this pass; `SC-005`'s "confirms zero references" claim concretely lives here)* |
| T050 | [test] | FR-010 | — | AC-041 | — | II, VI *(unit-tests the check's own violation-detection logic against a synthetic fixture — validates FR-010's "fails when violated" behavior specifically; does not itself confirm the real codebase's zero-reference state, so `SC-005` is not credited here, corrected this pass)* |
| T051 | [infra] | FR-014 | — | — | — | IX *(wires one recipe; the full CI-level SC/AC/PLT surface belongs to T081)* |
| T052 | [test], **PENDING** | FR-021 | — | AC-043 | — | IV, VI (blocked — see Pending / Blocked Items) *(wire-shape/negotiation contract test, not a handshake-rejection test — `SC-009` removed this pass, stays exclusively on T053)* |
| T053 | [test], **PENDING** | FR-022 | SC-009 | AC-033, AC-034 | SEC-001, SEC-003 | III, IV, VI (blocked) |
| T054 | [test], **PENDING** | FR-017 | — | AC-038 | SEC-008 | IV, VI, VIII (blocked) *(negotiation/redaction test only, not handshake-rejection behavior — `FR-022`/`SC-009` removed this pass)* |
| T055 | [impl] | FR-019 | — | AC-043 | — | IV |
| T056 | [impl] | FR-019 | — | AC-043 | — | IV *(runs `contracts-generate`; `FR-020` and `SC-008`'s zero-drift evidence both stay exclusively on T085 — `FR-020` removed this pass)* |
| T057 | [impl] | FR-022 | SC-009 | AC-033, AC-034 | SEC-001, SEC-003 | III, VI *(the `Host`→`Origin`-allowlist→token-subprotocol enforcement order itself — the concrete implementation `SC-009` measures; RGR pairing with T053 is `PENDING` until T053 compiles)* |
| T058 | [impl] | — | — | — | — | VI *(implements successful subprotocol selection — server echoes back only `converge.v1` — a positive-negotiation behavior, not `SC-009`'s rejection-path criterion; evidenced narratively by T052's wire-shape contract test, `FR-021`, not a separate FR/SC citation for this task; split from T057 this pass to keep rejection and successful-negotiation evidence distinct)* |
| T059 | [impl] | — | — | — | — | VI *(implements the post-connection `hello`/`ping`/`pong` message exchange only, after a successful handshake — not `SC-009`'s rejection-path criterion, removed this pass)* |
| T060 | [impl] | FR-022 | — | AC-033, AC-034 | SEC-001, SEC-003 | II, III *(router registration only — `SC-009`'s rejected-handshake evidence stays exclusively on T053/T057, removed this pass)* |
| T061 | [impl] | FR-008 | — | AC-035 | — | II |
| T062 | [impl] | FR-008 | — | AC-043 | PLT-008 | VI *(implementation only — distinct from T063's validation, corrected this pass)* |
| T063 | [test] | FR-016 | — | AC-043 | PLT-008 | VI |
| T064, T072 | [test]/[impl] | — | — | — | — | V *(domain entity definition/invariants only; no FR/AC/PLT cites entity definition as its satisfying artifact)* |
| T065, T075 | [test]/[impl] | FR-012 | SC-003 | — | — | V |
| T066, T074 | [test]/[impl] | FR-013 | SC-004 | — | — | II, V *(pure, now-fallible Application folding; no SQL/migration identifier — those concern T068/T071 and T070/T078)* |
| T067, T076 | [test]/[impl] | FR-013 | SC-004 | — | — | II, V |
| T068, T071 | [test]/[impl] | FR-011, FR-018 | — | AC-042 | PLT-002 | V *(T068 expanded this pass with a negative migration-checksum-mismatch case — see the `SC-007` note above the matrix, "broken migration" category)* |
| T069, T077 | [test]/[impl] | FR-012 | SC-003 | — | — | V *(idempotent persistence only — `AC-042`/`PLT-002` concern migrations/aggregate rebuild specifically, per `traceability.md`; they are not this row's evidence, removed this pass)* |
| T070, T078 | [test]/[impl] | FR-013 | SC-004 | AC-042 | PLT-002 | II, V *(the concrete rollback-on-fold-failure evidence)* |
| T079 | [infra] | FR-014 | — | — | — | IX |
| T080 | [infra] | FR-014 | — | — | — | IX *(local recipe completion; CI-level identifiers belong to T081)* |
| T081 | [impl] | FR-015 | SC-006 | AC-044 | PLT-005 (restricted), PLT-009 (restricted) | IX |
| T082 | [infra] | FR-015 | — | — | PLT-009 (restricted) | IX |
| T083 | [doc] | FR-018 | — | — | — | IX, X *(documents the runbook; does not by itself satisfy SC-007)* |
| T084 | [human], **PENDING until evidence supplied** | FR-018 | SC-007 | — | — | IX, X *(the only task SC-007 is credited to — corrected this pass)* |
| T085 | [test] | FR-019, FR-020 | SC-008 | AC-043 | — | IV |
| T086 | [test] | FR-014 | — | — | — | IX *(runs `just fmt-check`/`just lint` only; `AC-040` concerns the one-command launch, unrelated to formatting/linting — removed this pass)* |
| T087 | [test]/[human-dependent] | FR-014, FR-016, FR-018 | SC-006 | AC-040, AC-043, AC-044 | — | VI, IX, X *(SC-007 reportable here only if T084's evidence exists — not claimed independently)* |
| T088 | [doc] | FR-014 | — | AC-040 | — | IX |

---

## Constitution Compliance Check (Task-Breakdown Level)

Re-verified against `.specify/memory/constitution.md` v1.0.1 and `plan.md`'s Post-Design Gate. **Status values are `PASS` or `PENDING` only** — "strengthened" is descriptive text within a status, never a substitute for one, per this pass's explicit correction.

| Principle | Status | Task-breakdown verification |
| --- | --- | --- |
| I. Versioned Sources of Truth | **PASS** | Every task cites a concrete FR/SC/AC/PLT/SEC/design-artifact already present in the normative documents (matrix above). No new requirement is introduced. Both open blockers are reported explicitly (see "Pending / Blocked Items"), which is itself compliant behavior under this principle, not a deficiency of it. |
| II. Modular Clean Architecture | **PASS** | T012/T013 concretely satisfy FR-009 for Domain/Application specifically (not T014–T017, corrected this pass); T049/T050 concretely satisfy FR-010's "repeatable, automatable procedure." `RebuildAggregate` (T073, T074, T076, T078) contains no SQL, uses a correctly asynchronous **and now fallible** port contract, with static (non-`dyn`) dispatch — no new dependency introduced for either property. |
| III. Secure Local Runtime | **PENDING** | User Story 1 (T030–T039) and User Story 5 (T057–T060) implement loopback/random-port/ephemeral-token/`Host`/`Origin`/CORS/allowlist exactly as specified. **Blocker**: T025/T026's PID-capture-and-liveness-probe mechanism is an open human decision (see "Pending / Blocked Items") — the previous revision's assumed stdout-PID-line mechanism is removed as an invention this pass was asked not to repeat. This principle cannot be marked `PASS` until that decision is made and the two tests are concretely specified. |
| IV. Contracts as Code | **PENDING** | T028/T055 make Rust the authoritative source with exact Serde/`ts-rs` attributes; T029/T056 run `contracts-generate` before any frontend import; T085 verifies the drift gate — this sub-scope is executable and would independently be `PASS`. **Blocker**: T052–T054 (the WebSocket contract/integration tests) cannot compile without `futures-util`, which has no approved pin — this is a hard blocker on this principle's WebSocket-contract sub-scope, not a soft footnote, so the principle overall is `PENDING` until a human maintainer resolves it. |
| V. Durable and Reconstructable Data | **PASS** | User Story 3 (T064–T079) implements immutable-ledger triggers, idempotency/conflict classification, and a transactional rebuild whose transaction ownership is confined to `adapters-persistence` alone (T073, T078). The fold contract is now concretely fallible and its rollback path is directly tested (T070/T078), closing a real correctness gap the previous revision left implicit. |
| VI. Risk-Oriented Test-First Development | **PENDING** | The fold/rollback contract is now executable and RGR-paired (T066/T074, T067/T076) — that part of this principle is satisfied. **Blocker**: T052–T054 cannot compile, so T057–T059 cannot honestly complete an RGR cycle against them (Constitution VI requires the test to exist and fail before the implementation) — this principle remains `PENDING` for User Story 5 specifically until the `futures-util` decision resolves. |
| VII. Product and UX Integrity | **PASS** | T040–T042 make Graphite Signal tokens, theme selection, and reduced-motion CSS handling explicit tasks with exact file paths; T045 implements the Frontend State Matrix consuming only those tokens/rules (these implementation tasks are no longer credited with `SC-010` itself, corrected this pass — see traceability matrix). T046 tests, concretely: every applicable Frontend State Matrix state rendered under both Light and Dark themes; a simulated `prefers-reduced-motion: reduce` query asserting T042's motion rules suppress non-essential movement while state is still communicated immediately; keyboard navigation and visible focus; non-color state communication; and the `jest-axe` assertion — `expect` correctly imported (T008). **Blocker resolved this pass**: T008 now adds a local Vitest module-augmentation declaration (`apps/web/src/types/jest-axe.d.ts`) making `toHaveNoViolations` concretely type-checkable, using only Vitest's own already-approved, directly-depended-on `Assertion`/`AsymmetricMatchersContaining` interfaces — no `@types/jest-axe` pin invented, no `jest-dom`/Jest-namespace dependency introduced, no new package added. This principle is `PASS` only because that declaration now exists and is type-checkable with approved direct dependencies alone. |
| VIII. Privacy and Observability | **PENDING** | T027 (HTTP redaction) is executable, concrete evidence for its own scope — no telemetry or automatic transmission is introduced anywhere in this task list, and removing the invented PID-status-line assumption (T038) also removes a privacy question that never needed to exist. **Blocker (corrected this pass)**: T054 (WebSocket redaction) cannot be cited as passing or executable evidence while it remains `PENDING` — it cannot compile without an approved `futures-util` pin (see "Pending / Blocked Items"). A pending, non-compilable test is not used as passing evidence for this principle. This principle returns to `PASS` only once `futures-util` is normatively approved and T054 becomes executable. |
| IX. Documentation and Delivery Discipline | **PASS** | T010/T080 make the `justfile` the single command facade, each credited only with the specific recipe surface it concretely completes; T081 is the CI-level evidence for `SC-006`/`AC-044`/`PLT-005`/`PLT-009`; T083 documents the CI-break runbook. **`SC-007`'s four required categories now each have concrete, executable planned coverage (corrected this pass — closes the previous revision's disclosed migration gap)**: (1) denied authentication — T021/T032 (HTTP, executable) and T053/T057 (WS, `PENDING` only on the separate `futures-util` decision); (2) architectural-boundary violation — T049/T050, with T050's synthetic-violation fixture; (3) broken migration — T068, expanded this pass with a deliberate migration-checksum-mismatch case using SQLx's built-in integrity check; (4) broken test/gate-visibility — T084, the human-maintainer checkpoint (see the note above the traceability matrix). This principle's planning-gate bar is that the plan concretely and executably covers all four categories — it now does. **T084's own execution remains a later, human-only, post-implementation checkpoint, not a prerequisite for this planning-gate `PASS`** — corrected this pass: the planning gate evaluates plan compliance, not whether every future human-performed step has already happened; T084's checkbox stays unchecked and its evidence unfabricated regardless. |
| X. Human-Controlled Version Control | **PASS** | This principle concerns whether the plan itself keeps Git staging, committing, pushing, merging, tagging, rebasing, resetting, and PR creation exclusively under human control. It does: no task in this breakdown stages, commits, pushes, or opens a pull request, and T083, T084, and T087 each explicitly forbid the agent from executing the intentionally-failing Git/CI workflow, creating or pushing a throwaway branch, or running any Git write/publishing command (Constitution X; AGENTS.md Git and Release Boundaries) — a rule this document itself has followed throughout every revision. **Corrected this pass**: this principle is evaluated at the planning gate — whether the plan complies with the Git-discipline rule — not whether T084's future human-performed evidence already exists; requiring the latter would demand an artifact that can only exist after implementation, which the planning gate must not do. T084 remains a genuine future human checkpoint (tracked under Principle IX and at the task level), not a blocker for this principle's own `PASS`. |

**Two unresolved normative/mechanism decisions still block Principles III, IV, VI, and VIII specifically** (see "Pending / Blocked Items" above): the `futures-util` pin for WebSocket tests, and the PID-capture/liveness-probe mechanism for the abrupt-supervisor test. Neither is fabricated here. **Separately, T084's human-performed CI-break evidence remains a genuine future checkpoint**, tracked at the task level (its checkbox stays unchecked) — but, corrected this pass, its non-existence does not block Principles IX or X's planning-gate `PASS`, since the planning gate evaluates whether the plan concretely and executably covers the requirement, not whether every future human-performed step has already happened (see Principles IX/X above). Per the user's explicit instruction for this correction pass, `/speckit-implement` was **NOT** invoked. No production code, migration, configuration, or test file was created or modified while generating or correcting this task list — only `specs/001-slice-0-foundation/tasks.md` was written. **This document still does not claim implementation-readiness**: Principles III, IV, VI, and VIII remain genuinely `PENDING` on the two normative/mechanism decisions above, and the plan stops here for human review on those two points specifically.

---

## Notes

- `[P]` tasks touch different files with no unmet dependency; `PENDING` tasks may still carry `[P]` if their file-ownership is genuinely independent — `[P]` describes parallel-safety, not executability.
- `[Story]` maps each task to its user story for traceability back to `spec.md` and `traceability.md`.
- RGR (Red-Green-Refactor) tasks are cross-referenced above ("makes T0xx pass") so the paired test can be verified to fail before its implementation task starts; where the paired test is `PENDING`, no RGR claim is made.
- Per Constitution Principle X, no agent may stage, commit, push, revert Git history, or open a pull request after completing tasks. T083 and T084 in particular MUST NOT be executed as live Git operations by an agent — T084 specifically records only evidence a human maintainer supplies.
- Task count changed from **87 to 88** in the third revision: one task was added (T084, the human-maintainer checkpoint recording `SC-007`'s actual execution evidence, distinct from T083's documentation-only runbook). Every task from the old `T084` onward was renumbered by +1; every cross-reference in this file was updated to match. **Neither the fourth nor the fifth (this) revision adds, removes, or renumbers any task — the count stays at 88.** The fifth revision resolves the `jest-axe` TypeScript blocker by expanding T008 with a local Vitest module-augmentation declaration (no new dependency, no new task), expands T068 with a negative migration-checksum-mismatch case to close `SC-007`'s previously-disclosed migration-category gap (no new task), splits the previously-combined `T057, T058` traceability row to keep rejected-handshake, successful-negotiation, and post-connection-message evidence distinct, removes several further inflated identifiers (`FR-020` from T029/T056; `FR-009` from T049; `SC-005` from T050; `SC-009` from T060; `SC-010` from T040/T042/T045; `SC-011` from T041/T043/T044; `AC-042`/`PLT-002` from T069/T077; `AC-040` from T086), and re-runs Constitution Principles VII, IX, and X — VII now `PASS` (the `jest-axe` type blocker resolved), IX now `PASS` (all four `SC-007` categories have concrete planned coverage), and X now `PASS` (the plan's own Git-write prohibition is a planning-time property, correctly decoupled this pass from T084's still-pending future execution).
- Two items remain deliberately open, with no invented mechanism or dependency — see "Pending / Blocked Items" near the top of this file. Neither is presented as resolved anywhere else in this document.
- Stop at any checkpoint to validate a story independently before continuing.
