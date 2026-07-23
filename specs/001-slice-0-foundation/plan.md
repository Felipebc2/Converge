# Implementation Plan: Slice 0 — Foundation

**Branch**: `001-slice-0-foundation` | **Date**: 2026-07-22 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-slice-0-foundation/spec.md`,
its quality checklist, and the normative sources listed there
(`.specify/memory/constitution.md`, `docs/PRODUCT.md`, `docs/PRD.md`,
`docs/MVP.md`, `docs/DESIGN.md`, `AGENTS.md`, `CLAUDE.md`).

**Revision note (this pass)**: this plan was corrected after a consistency
review. Fourteen corrections were applied; each is marked **(Corrected)** or
**(New)** at the relevant section, and one requested correction was
evaluated and **not applied** because it conflicts with the approved
spec.md, which this revision is not authorized to change — see "Frontend
State Matrix — Implementation Notes" below for the explicit conflict report.
Full technical detail for every correction lives in `research.md`,
`data-model.md`, and `contracts/`; this file summarizes and cross-references
rather than duplicating.

**Revision note (final consistency pass)**: a second, smaller consistency
review found and corrected nine further gaps, each marked **(this pass)** at
the relevant section: an under-formalized CORS preflight; several
still-placeholder or unpinned direct dependencies (the HTTP integration
client, the base64url and timestamp implementations, `serde`, two more
GitHub Actions SHAs, `@vitejs/plugin-react`, and the React type packages)
plus two literal `<exact-sha>` placeholders still present in this file;
Vite and TanStack Query pins that had drifted behind the registry's actual
current stable versions since the previous pass; an orphan-process test
that asserted a proxy signal (a subsequent `just dev` succeeding) rather
than the actual child-lifecycle guarantee; `#![forbid(unsafe_code)]`
inconsistently excluded from `service`; an under-specified event-identity
model (who generates `event_id`, what "canonical" means for the payload
hash, and too-narrow conflict-vs-idempotent matching); an incomplete
traceability matrix missing several PRD IDs spec.md itself already cites;
`spec.md`'s `Status` field left at `Draft` despite being treated as
approved; and a literal miscount ("two" instead of three) of the WebSocket
message shapes, also present in this file. Full technical detail again
lives in `research.md`, `data-model.md`, `contracts/`, and `traceability.md`.

**Revision note (final blocking-consistency pass)**: a third, targeted
correction closed four remaining blocking gaps, each marked **(point N,
this pass)** at the relevant section: (1) the CORS preflight still left an
*absent* `Access-Control-Request-Method` unhandled and the actual `200`
response never stated its own required `Access-Control-Allow-Origin`
header — both corrected in `contracts/http-health.md`; (2) `jsdom` and
`@testing-library/dom` were genuine, load-bearing frontend test
dependencies (Vitest's DOM environment and `@testing-library/react`'s own
peer, respectively) that had never been pinned — now `jsdom =29.1.1` /
`@testing-library/dom =10.4.1`, `research.md`; (3) a real contradiction
between `research.md` (claiming `application` generates `event_id`, and
`adapters-persistence` performs canonical serialization/BLAKE3 hashing) and
`data-model.md` (which already correctly assigned both — producer-only
generation, `application`-only serialization/hashing) is resolved in
`research.md`'s favor of `data-model.md`'s wording; `application`'s
allowlist gains `serde_json` and drops the `uuid` `v7` feature (default
features only — parsing/typing, never generation);
`adapters-persistence`'s allowlist drops `serde_json` entirely (Project
Structure, below); (4) `docs/PRODUCT.md`'s `Status` field (`Draft for
approval`) was checked against the human-approval question this pass
required answering before touching it — **no evidence of human approval
was found, so it was left unchanged and this plan stops for explicit human
direction on that specific point** (see the note at the end of this file).

**Revision note (human-approval confirmation pass)**: the human maintainer
has since confirmed `docs/PRODUCT.md` version 1.0.0 as formally approved.
`docs/PRODUCT.md`'s `Status` field now reads `Approved` (previously `Draft
for approval`) and its `Last updated` date is synchronized to the date of
this confirmation. This closes point 4 of the prior pass: Principle I's gate
(below) now reports **PASS** without qualification, and the "One unresolved
item" note at the end of this file — including the alternative under which
this plan could proceed with `docs/PRODUCT.md` still in `Draft` — no longer
applies and has been removed. No other section of this plan changes as a
result of this confirmation. Per this pass's own instructions,
`/speckit-tasks` and `/speckit-implement` were still NOT run; this plan
stops here for final human review.

## Summary

Slice 0 proves the architectural seams every later MVP slice depends on: one
command launches an authenticated local Rust service (HTTP + a minimal
WebSocket proof) alongside a minimal React/Vite frontend; the service is
loopback-only, random-port, ephemeral-token-authenticated, with `Host`/
`Origin`/CORS enforcement on HTTP and an explicit `Host`/token/Origin-
allowlist check on WebSocket; Clean Architecture boundaries between Domain,
Application, and Adapters are mechanically checked via explicit
per-crate dependency **allowlists** (corrected from a blacklist — see
Testing Plan), not just conventionally true; one representative normalized
event can be persisted idempotently — including detecting a conflicting
reuse of the same idempotency key, not only a legitimate retry — and its
derived aggregate discarded and rebuilt, transactionally, without mutating
the ledger; and formatting, linting, tests, and builds all run through one
`justfile` command facade that CI uses identically, with `just check` now
an unconditional complete gate including browser E2E (corrected — no
environment-dependent skip), so nothing passes locally that fails in CI or
vice versa.

The technical approach: an Axum-based Rust service (HTTP + WS in one
framework) over a **six-crate** Clean Architecture workspace (corrected
count — see Project Structure), SQLite via SQLx with forward-only
migrations, `ts-rs`-generated TypeScript contracts (with exact Serde
tagging/renaming so wire JSON, Rust types, and generated TypeScript match
field-for-field) with a CI drift gate, a corrected-ordering, IPC-based,
disk-free environment-variable/message bootstrap handoff between a `just
dev` supervisor, the Rust service, and the Vite dev server (`research.md`
Part 2), and React 19/TanStack Query/Zustand on the frontend with
WebdriverIO for the one minimal browser E2E journey PRD `PLT-008` mandates.
Full version pins (all now exact `=` pins for Cargo dependencies, with
`Cargo.lock` committed) and their rationale are in `research.md` Part 1; the
security design, including the corrected bootstrap ordering and WebSocket
subprotocol negotiation, is in `research.md` Part 2; the persisted schema
(including the new `payload_hash` column and transactional rebuild
semantics) is in `data-model.md`; the wire contracts (including exact Serde
tagging) are in `contracts/`; the runnable validation path is in
`quickstart.md`; the full requirement traceability matrix is in
`traceability.md`.

## Technical Context

**Language/Version**: Rust `1.97.1` (Edition 2024) for the backend service;
TypeScript `6.0.3` for the frontend (see `research.md` for why not 7.0.x
yet).

**Primary Dependencies**: Backend (all exact `=` pins — corrected, see
`research.md`'s "Exact-pin policy") — `axum =0.8.9`, `tower-http =0.7.0`,
`tokio =1.53.1`, `sqlx =0.9.0` (sqlite/runtime-tokio/macros/migrate),
`ts-rs =12.0.1`, `rand =0.10.2`, `thiserror =2.0.19`, `uuid =1.24.0`
(default features only in `application` — **corrected this pass, point 3**:
the `v7` feature, which gates *generation*, belongs only to the producer's
own dev-dependency edge, never to `application`, which only parses/types a
producer-supplied `event_id`; see Project Structure), `blake3 =1.8.5` (new,
payload-hash decision), `serde =1.0.229` (feature `derive`, `contracts`
only) and `serde_json =1.0.151` (were implied by every `#[derive
(serde::Serialize...)]` in `contracts/` but never their own pinned entry;
**reassigned this pass, point 3**, from `adapters-persistence` to
`application` — canonical payload serialization for BLAKE3 hashing is an
`application`-layer concern, matching `data-model.md`'s "Event identity"
section, which `research.md` previously contradicted),
`time =0.3.54` (features `formatting`/`parsing`/`serde` in `adapters-http`;
`parsing` only in `application` — new this pass, the timestamp
implementation `occurred_at` validation and `checkedAt`/`serverTime`
generation need), `base64 =0.22.1` (new this pass — the base64url encoding
`research.md`'s token-generation section always assumed but never pinned),
`reqwest =0.13.4` (dev-dependency, the chosen HTTP integration test client
— new this pass, was described only as "a `reqwest`-class client"),
`tokio-tungstenite =0.30.0` (dev-dependency only, new — was a placeholder).
Frontend — `react`/`react-dom 19.2.8`, `vite 8.1.5` (corrected this pass —
`8.1.0` had already been superseded by a patch release when re-checked
against the registry), `@tanstack/react-query 5.101.4` (corrected this
pass, same reason — was `5.101.2`), `zustand 5.0.14`, `@vitejs/plugin-react
6.0.4` (new this pass — Vite's React integration plugin was never pinned),
`@types/react 19.2.17` / `@types/react-dom 19.2.3` (new this pass). Tooling —
`eslint 10.7.0` (flat config) + `eslint-config-prettier 10.1.8`
(corrected — was a placeholder), `typescript-eslint 8.65.0` (corrected —
was a placeholder), `eslint-plugin-react-hooks 7.1.1` (corrected — was a
placeholder), `prettier 3.9.6`, `vitest 4.1.10` +
`@testing-library/react 16.3.2`, `jsdom 29.1.1` (new this pass, point 2 —
Vitest's DOM test environment, previously implied by `vitest.config.ts` but
never itself pinned) + `@testing-library/dom 10.4.1` (new this pass, point
2 — `@testing-library/react`'s own required peer, previously left
unpinned), `jest-axe 10.0.0` (corrected — was a
placeholder; chosen over the unmaintained `vitest-axe`),
`webdriverio 9.30.0`/`@wdio/cli 9.30.0`/`@wdio/local-runner 9.30.0`/
`@wdio/mocha-framework 9.30.0`/`@wdio/spec-reporter 9.29.1` (corrected and
aligned — was partially placeholder), `just 1.57.0`, `pnpm 11.15.1` (via
Corepack). Full rationale and enforcement mechanism per item:
`research.md` Part 1.

**Storage**: SQLite via `sqlx =0.9.0`, forward-only versioned migrations
under `migrations/`, offline query cache (`.sqlx/`) committed for
CI builds without a live database. Schema: `data-model.md`.

**Testing**: `cargo test` (unit + integration, including the architecture
boundary **allowlist** check — corrected from a blacklist check), Rust
contract tests + a regenerate-and-diff drift check, Vitest + React Testing
Library (frontend, including the Frontend State Matrix and `jest-axe`
accessibility assertions), WebdriverIO (the one minimal browser E2E
journey, per PRD `PLT-008`, now an unconditional part of `just check` —
corrected). Full matrix: Testing Plan section below.

**Target Platform**: Linux x86_64 — primary, full CI gate (`just check`
including E2E, unconditionally). Windows x86_64 — build plus applicable
technical tests (`PLT-005` restricted; no E2E-browser requirement in this
slice's Windows CI job specifically, per `docs/MVP.md` §6.4 — this is a
documented scope boundary, not the kind of silent/conditional skip
correction 7 targets, which concerned `just check` itself on a
contributor's own machine). Contributors on either OS can run `just dev`
locally — the corrected bootstrap design (env-var + IPC handoff, no disk
artifact) is identical on both, since Node's `child_process` IPC is
portable.

**Project Type**: Local web-service + browser frontend (modular monolith
served over loopback HTTP/WebSocket, not an internet-facing client/server
split, and not yet a Tauri desktop shell — `PLT-001`/`PLT-003` remain
restricted for the whole MVP).

**Performance Goals**: Qualitative "Fast" only, per `docs/MVP.md` §12 —
no numeric performance budget is invented for this slice; none is required
by any normative source.

**Constraints**: Loopback-only binding; no telemetry; **no unsafe Rust in
any Converge-owned crate, `service` included with no exception** —
clarified this revision (correction 13) and tightened this pass (correction
5): `#![forbid(unsafe_code)]` is active in all six crates under `crates/`
uniformly, not five-of-six as the previous draft stated (`service` was
previously carved out as "the composition root most likely to need a
documented exception" — no such exception was ever written, so the carve-
out granted a permission nothing used; it is removed). This does not reach
third-party C FFI inside a transitive dependency (`sqlx`'s SQLite driver
depends on `libsqlite3-sys`, which is unsafe by necessity as any SQLite
binding is — see `research.md`'s sqlx section for the exact boundary
statement) — `forbid(unsafe_code)` only constrains code written inside a
Converge-owned crate, never its dependencies' own internals. The corrected
orphan-process mechanism (Testing Plan below) was designed specifically to
need no FFI (no `libc::prctl`, no Windows Job Object bindings) precisely so
`service` could adopt this lint with nothing to work around; WCAG
2.2 AA, keyboard operation, visible focus, non-color state communication,
and reduced motion apply to the one frontend surface this slice introduces
(the health/status indicator) from day one, per spec.md FR-023.

**Scale/Scope**: Single local user, single running instance the common
case (though concurrent instances are explicitly supported per the security
design), no project/trust concept yet (Slice 1), one representative
event/aggregate pair proving the ledger mechanism — deliberately minimal
scope, proportional to a foundation slice.

## Constitution Check — Pre-Design Gate

*GATE: Must pass before Phase 0 research. Re-checked after Phase 1 design
below.*

| Principle | Gate | Status |
| --- | --- | --- |
| I. Versioned Sources of Truth | Design must not introduce requirements beyond spec.md/PRD/MVP | **PASS** — this plan adds no new functional requirement; every technical decision traces to an existing FR-/SC-/AC-/PRD-ID (see `traceability.md`). The one new engineering decision this revision (UUIDv7 + BLAKE3) is confined to persistence implementation detail within FR-012/FR-013 and does not add a new FR/SC/AC. |
| II. Modular Clean Architecture | Domain/Application must have zero transport/DB/UI/FS/PTY/Tauri/provider dependency, mechanically checked | **PASS (design)** — six-crate structure below isolates Domain/Application; mechanical check now designed as an explicit per-crate **allowlist** (Testing Plan), corrected from a blacklist. Verified again post-design. |
| III. Secure Local Runtime | Deny-by-default, no generic shell/FS access, non-waivable secret/corruption/escape protections | **PASS** — `research.md` Part 2's corrected threat model, bootstrap ordering, and WebSocket subprotocol negotiation satisfy this; no shell execution or unrestricted FS access is introduced anywhere in this design. |
| IV. Contracts as Code | Rust authoritative, deterministic TS generation, drift fails CI | **PASS (design)** — `ts-rs` generation + CI diff gate designed in Contracts section below and in `contracts/`, now with exact Serde tagging/renaming so wire/TS field names are fully determined, not merely "generated." |
| V. Durable and Reconstructable Data | Immutable events, reconstructable aggregates, tested forward-only migrations | **PASS (design)** — `data-model.md`'s triggers-enforced immutability, `payload_hash`-distinguished conflict handling, and transactional (`BEGIN IMMEDIATE`) full-rebuild-from-ledger design. |
| VI. Risk-Oriented Test-First Development | Red-Green-Refactor for domain/application/contracts/security/persistence | **PASS (design)** — Testing Plan below scopes RGR to exactly those layers, consistent with spec.md FR-016, now including the conflicting-duplicate and transactional-rebuild test cases. |
| VII. Product and UX Integrity | Graphite Signal, both themes, WCAG 2.2 AA, state matrix apply now | **PASS (design)** — Frontend State Matrix section below implements the health/status surface against spec.md's Frontend State Matrix exactly as approved; no deferral, and no unapproved deviation (see that section's explicit conflict report for the one correction not applied). |
| VIII. Privacy and Observability | No default telemetry, redaction of secrets in logs | **PASS (design)** — `research.md` Part 2 mandates redaction of `Authorization`/`Sec-WebSocket-Protocol`, now including the specific rule that the token-bearing subprotocol value is never echoed in the handshake response either; no telemetry is introduced. |
| IX. Documentation and Delivery Discipline | `justfile` facade, CI mirrors contributor commands, docs travel with the change | **PASS (design)** — Command Facade & CI section below; `just check` is now an unconditional complete gate (corrected); no competing validation path. |
| X. Human-Controlled Version Control | No agent-executed git publishing actions | **PASS** — this plan performs no git action; suggested commands are for the human maintainer at handoff. |

No violation requires an entry in Complexity Tracking.

**Pre-design gate re-verification (final consistency pass)**: none of this
pass's nine corrections change any Principle's status above. II is
strengthened, not merely preserved (`#![forbid(unsafe_code)]` now covers
all six crates uniformly, correction 5, rather than five-of-six). III is
strengthened (the CORS preflight is now fully specified, correction 1, and
the orphan-process guarantee is now a direct, mechanism-backed assertion
rather than an indirect proxy check, correction 4). I is unaffected in kind
— every new dependency pinned this pass (`reqwest`, `base64`, `serde`,
`serde_json`, `time`, `@vitejs/plugin-react`, the React type packages) is
an implementation detail of already-approved requirements (FR-002–FR-008,
FR-012, FR-019–FR-021), not a new one, and the `spec.md` `Status` field
change (correction 8) documents an approval state that already existed in
practice (this plan already presupposed it) rather than granting a new one.
No principle regresses; all ten remain **PASS**.

**Pre-design gate re-verification (final blocking-consistency pass)**:
re-checked honestly against all four corrections in this third pass. **III**
is strengthened (point 1: the CORS preflight now denies an absent
`Access-Control-Request-Method`, not only a wrong one, and the actual `200`
response now states its own required `Access-Control-Allow-Origin` header —
closing a real gap where a successful, allowed-origin response could have
been unreadable by the frontend despite passing every server-side check).
**II** is strengthened (point 3: `application`'s allowlist now precisely
matches its actual responsibilities — `serde_json` for the canonical
serialization it genuinely performs, `uuid` without an unused `v7` feature
it never exercises — resolving a real self-contradiction between
`research.md` and `data-model.md` about which layer generates `event_id`
and which layer hashes the payload, rather than leaving two normative
documents in this feature disagreeing with each other). **I is not fully
resolved by this pass** — point 4 required checking whether
`docs/PRODUCT.md` (a normative source ranked above this feature's own
artifacts per `AGENTS.md`'s Instruction Priority) has been human-approved
before touching its `Status` field; `docs/PRODUCT.md` itself still reads
`Status: Draft for approval`, and no commit, note, or other evidence of a
separate human-approval event was found. Per this pass's own explicit
instruction ("otherwise stop for explicit human approval"), `docs/
PRODUCT.md` was left unchanged, and **Principle I's gate is reported here
as open on this one point, not silently marked PASS** — this plan proceeds
on every other point but surfaces this one for the human maintainer rather
than resolving it unilaterally (see the note at the end of this file). **IV
through X**: unaffected by this pass's four corrections, remain **PASS**
unchanged.

**Pre-design gate re-verification (human-approval confirmation pass)**: the
human maintainer has confirmed `docs/PRODUCT.md` version 1.0.0 as formally
approved; its `Status` field now reads `Approved`. This closes the one open
point from the previous re-verification above. **I now reports PASS without
qualification** — every normative source this plan depends on, including
`docs/PRODUCT.md`, is confirmed approved. **II through X**: unaffected by
this confirmation, remain **PASS** as previously stated.

## Project Structure

### Documentation (this feature)

```text
specs/001-slice-0-foundation/
├── spec.md                     # Approved feature specification
├── checklists/requirements.md  # Spec quality checklist
├── plan.md                     # This file
├── research.md                 # Phase 0 output — versions + security threat model
├── data-model.md               # Phase 1 output — ledger + aggregate schema
├── quickstart.md               # Phase 1 output — runnable validation guide
├── traceability.md             # New this revision — FR/SC/AC/PRD traceability matrix (correction 12)
├── contracts/
│   ├── http-health.md          # Phase 1 output — authenticated HTTP health contract
│   └── websocket-proof.md      # Phase 1 output — authenticated WebSocket proof contract
└── tasks.md                    # NOT created by this command — /speckit-tasks only
```

### Source Code (repository root) **(Corrected: six crates, not five —
correction 6)**

`#![forbid(unsafe_code)]` is active at the crate root of **all six** crates
below, `service` included with no exception **(corrected this pass —
correction 5: the previous draft excluded `service` "as the composition
root most likely to need a documented exception," but no such exception was
ever written or justified, so the exclusion granted an unused permission;
it is removed. The corrected orphan-process mechanism below needs no FFI,
so nothing in `service` requires `unsafe` anyway.)**. This constrains code
written inside these crates only — it does not and cannot reach a
dependency's own internals (`libsqlite3-sys`'s FFI, `research.md`'s sqlx
section).

```text
crates/
├── domain/                  # Entities, value objects, invariants.
│                             # Normal-dependency ALLOWLIST: {thiserror}.
│                             # No serde, no I/O, no framework, no async.
├── application/              # Use cases (RecordProbeEvent, RebuildAggregate)
│                             # and inbound/outbound ports (traits only).
│                             # Normal-dependency ALLOWLIST: {domain (path),
│                             # thiserror, uuid (default features only —
│                             # corrected this pass, point 3: the Uuid TYPE,
│                             # to parse/validate the producer-supplied
│                             # event_id string; never the "v7" feature,
│                             # since application never GENERATES an
│                             # event_id — see "Event identity" in
│                             # data-model.md, and the corrected rationale
│                             # in research.md), blake3 (computes
│                             # payload_hash), serde_json (reassigned this
│                             # pass, point 3, from adapters-persistence —
│                             # builds the canonical payload as a
│                             # serde_json::Value and serializes it once per
│                             # RecordProbeEvent call, the bytes blake3
│                             # hashes; no #[derive(Serialize)] on any
│                             # application-owned type, so this stays a
│                             # data-format utility, not a wire-format
│                             # concern — research.md), time (feature
│                             # "parsing" only — correction 6, previous
│                             # pass: validates the producer-supplied
│                             # occurred_at string is a well-formed RFC 3339
│                             # timestamp; never reads the wall clock, so
│                             # Application stays free of "now"-dependent,
│                             # untestable behavior)}.
│                             # `tokio` appears only as a dev-dependency
│                             # (for #[tokio::test]-based unit tests), never
│                             # a normal dependency. The producer that
│                             # generates a UUIDv7 event_id (in this slice,
│                             # the integration test suite) enables uuid's
│                             # "v7" feature only on its own dev-dependency
│                             # edge, outside this allowlist's scope.
├── contracts/                 # Wire-level request/response/message DTOs only
│                             # (HealthResponse, HealthStatus, ApiError,
│                             # WsMessage). ts-rs derives live here.
│                             # Normal-dependency ALLOWLIST: {serde
│                             # (feature "derive"), ts-rs}.
│                             # No dependency on domain/application —
│                             # adapters map between contracts DTOs and
│                             # domain/application types explicitly, keeping
│                             # domain types free of wire
│                             # serialization/TS-derive attributes.
├── adapters-http/             # Axum router: HTTP health route + WS upgrade
│                             # route in the same router. Owns the
│                             # Host/Origin/CORS/token/subprotocol-negotiation
│                             # and CORS-preflight (correction 1, this pass —
│                             # contracts/http-health.md) middleware
│                             # (research.md Part 2) and maps application
│                             # results to `contracts` DTOs.
│                             # Normal-dependency ALLOWLIST: {application
│                             # (path), domain (path), contracts (path),
│                             # axum, tower-http, tokio, thiserror, serde_json
│                             # (new this pass — builds ApiError/HealthResponse
│                             # JSON via the contracts types' own Serialize
│                             # impl), time (features "formatting"/"parsing"/
│                             # "serde", new this pass — generates the live
│                             # checkedAt/serverTime response fields)}.
├── adapters-persistence/      # SQLx SQLite repositories implementing
│                             # application's outbound ports
│                             # (EventRepository, AggregateRepository).
│                             # Normal-dependency ALLOWLIST: {application
│                             # (path), domain (path), sqlx, tokio,
│                             # thiserror}. No serde_json here — corrected
│                             # this pass, point 3: canonical payload
│                             # serialization and BLAKE3 hashing are an
│                             # application-layer concern (above), not a
│                             # persistence-adapter one; this crate receives
│                             # the already-serialized payload string and
│                             # already-computed payload_hash string from
│                             # application and binds them as plain SQL
│                             # parameters — no JSON library needed at the
│                             # persistence boundary.
└── service/                   # Binary crate — composition root. Owns
                              # startup: reads CONVERGE_ALLOWED_ORIGIN from
                              # its own env (set by the supervisor — see
                              # research.md's corrected Bootstrap Ordering),
                              # binds 127.0.0.1:0, generates the token
                              # (rand::OsRng, encoded via base64 =0.22.1's
                              # URL_SAFE_NO_PAD engine — pinned this pass,
                              # was previously only described, never
                              # pinned), prints the CONVERGE_BOOTSTRAP
                              # stdout line for the `just dev` supervisor,
                              # wires adapters to ports, runs the Axum
                              # server, and owns graceful-shutdown and
                              # parent-liveness-watchdog handling
                              # (research.md's corrected Launcher shutdown
                              # / orphan-process design, correction 4 this
                              # pass). The only crate allowed to depend on
                              # every other crate; no dependency-allowlist
                              # restriction applies to `service` itself,
                              # since it is the composition root — but
                              # `#![forbid(unsafe_code)]` still applies to
                              # it uniformly with the other five crates
                              # (correction 5, this pass; see the note
                              # above this tree).

apps/web/                    # React + TypeScript + Vite frontend
├── src/
│   ├── features/health/     # The one frontend surface: health/status indicator,
│   │                        # implementing spec.md's Frontend State Matrix.
│   ├── lib/api/               # Typed HTTP/WebSocket client built on the
│   │                        # generated contracts (direct calls to the
│   │                        # Rust service's own origin — never through a
│   │                        # Vite proxy, research.md Part 2); TanStack
│   │                        # Query hooks.
│   └── state/                 # Zustand store(s) — transient UI/layout state only,
│                             # never an authoritative copy of server data (FR-024).
├── vite.config.ts             # Node-side config owning the IPC-based
│                             # dev-bootstrap middleware (research.md's
│                             # corrected Bootstrap Ordering): binds to
│                             # 127.0.0.1, receives the Rust service's
│                             # port/token from the supervisor over IPC
│                             # once it starts, and serves the same-origin,
│                             # Host-validated dev-only bootstrap response
│                             # the frontend fetches once on load.
└── (test files colocated, run by Vitest + React Testing Library + jest-axe)

packages/
└── contracts/                # Generated TypeScript output (from crates/contracts
                             # via ts-rs), a pnpm workspace package apps/web
                             # depends on as "@converge/contracts": "workspace:*".
                             # Never hand-edited — regenerated by the contracts
                             # recipe and diff-checked in CI (FR-020).

migrations/                  # Forward-only SQLx migrations: ledger + aggregate
                             # tables (including the new payload_hash column)
                             # and their immutability triggers (data-model.md).

scripts/
└── dev.mjs                   # The `just dev` supervisor (Node, cross-platform,
                             # uses child_process.fork() for the Vite child
                             # specifically so a private IPC channel exists —
                             # research.md's corrected Bootstrap Ordering):
                             # 1) forks Vite, reads back its actual bound
                             # port over IPC; 2) spawns the service binary
                             # with CONVERGE_ALLOWED_ORIGIN set to Vite's
                             # real origin; 3) captures the service's
                             # stdout bootstrap line (port + token);
                             # 4) forwards the port/token to the already-
                             # running Vite child over the same IPC
                             # channel; 5) forwards subsequent output with
                             # the bootstrap line redacted; 6) on
                             # SIGINT/SIGTERM, terminates the Rust child
                             # first, then Vite, waits bounded, force-kills
                             # any child that doesn't exit in time, and
                             # only then exits itself (research.md's
                             # Launcher shutdown design, correction 7).

tests/
├── architecture/             # Mechanical Clean Architecture ALLOWLIST check
│                            # (FR-009, FR-010, correction 11) — see Testing
│                            # Plan.
├── contract/                 # HTTP + WebSocket contract tests (including
│                            # the subprotocol-negotiation test); the
│                            # regenerate-and-diff drift check (FR-020, FR-021).
├── integration/               # Live-socket, DIRECT (non-proxied) HTTP/WS
│                            # integration tests: security denial paths
│                            # (both transports), migrations, idempotency
│                            # (sequential + concurrent, including the
│                            # conflicting-duplicate case — correction 10),
│                            # transactional aggregate rebuild, launcher
│                            # shutdown, and orphan-process behavior
│                            # (correction 7).
└── e2e/                       # WebdriverIO: the one minimal browser journey
                             # (load frontend → Ready state → WS proof completes).
```

**Structure Decision**: six Rust crates preserve inward Clean Architecture
dependencies exactly (`domain` → nothing but `thiserror`; `application` →
`domain` + `thiserror` + `uuid` + `blake3` + `serde_json` + `time`
(corrected this pass, point 3); `adapters-http`/
`adapters-persistence` → `application` + `domain` + `contracts` where
relevant; `service` → everything). `contracts` is deliberately a sibling of
`domain`/`application`, not a dependency of either — it holds wire DTOs and
`ts-rs` derive attributes so those framework concerns never touch
domain/application types, and adapters perform explicit mapping at the
boundary. This shape is what makes FR-009/FR-010's mechanical check simple:
compiling `domain`+`application` in isolation cannot succeed if either
gains a dependency outside its documented allowlist, and a
dependency-graph check (Testing Plan) backs that up as a second,
non-compilation line of defense, now checking against an explicit allowlist
per crate rather than a single shared blacklist (correction 11) — an
allowlist catches the introduction of *any* unapproved crate, not only the
specific ones a blacklist happens to name.

## Security Design (Phase 0 summary — full detail in `research.md` Part 2)

Before any bootstrap/token/discovery code is written, `research.md` Part 2
establishes: the supported threat boundary (hostile browser content and
unauthenticated local peers — in scope; a same-OS-user process, an
elevated local attacker, or physical access — explicitly out of scope, per
no normative source requiring otherwise); a 10-point threat analysis (DNS
rebinding, token leakage vectors, cookie domain/port scoping — corrected
this revision, WebSocket cross-site hijacking, Host/Origin spoofing limits,
replay/lifetime, port-discovery for **both** processes — corrected, since
Vite's port is no longer assumed fixed, concurrent instances, stale
artifacts, the same-user boundary), each grounded in live 2025–2026
sources; a compared-alternatives table for bootstrap design; and the
selected, corrected design (correction 2): the supervisor forks the Vite
dev server first (so its real port is known before the Rust service starts,
resolving the original ordering gap), passes the resulting frontend Origin
to the Rust service via its spawn-time environment, and forwards the Rust
service's port/token back to the already-running Vite process over a
private Node IPC channel — env-at-spawn for one direction, IPC for the
other, both disk-free. Token transport is `Authorization: Bearer` for HTTP,
and a two-value `Sec-WebSocket-Protocol` offer (`converge.v1` plus
`converge.token.<base64url-token>`) for WebSocket, with the server
selecting only the non-secret `converge.v1` value in its response
(corrected — correction 1), validated *during* the upgrade, before any
`101` response, never after. All traffic between the frontend and the Rust
service is **direct**, never proxied through Vite (corrected —
correction 2, bullet 4) — CORS and the Origin allowlist would be dead code
otherwise.

This plan does not restate that analysis — see `research.md` Part 2 for the
full reasoning, prior-art comparison, and validation approach.

## Contracts and Transport (Phase 1 — full contracts in `contracts/`)

- **HTTP**: `GET /api/health`, defined in `contracts/http-health.md`.
  Validation order — `Host` (400) → `Origin` (403, now required on every
  request, not merely "when present" — corrected this pass, correction 1) →
  token (401) — first failure wins, never partially trusted. `200` body:
  `{status, service, version, checkedAt}` (camelCase on the wire —
  corrected, correction 5); the `200` response itself now also always
  carries `Access-Control-Allow-Origin` (corrected — point 1, final
  blocking-consistency pass: the preflight grant and the actual response's
  own CORS header are two separate Fetch-spec checks, `contracts/
  http-health.md`'s Success Response). The `OPTIONS` CORS preflight is now
  specified as its own, separately ordered check sequence (`Host` →
  `Origin` → requested method `GET`, now denying an absent value
  identically to a wrong one — corrected, point 1, final
  blocking-consistency pass → requested header `authorization`), returning
  an exact three-header closed-CORS grant on success and no grant at all on
  any failure — carries no bearer token by the CORS specification itself,
  never checked (correction 1, previous pass; full detail in
  `contracts/http-health.md`'s CORS Preflight section).
- **WebSocket**: `GET /api/ws` (upgrade), defined in
  `contracts/websocket-proof.md`. Same `Host`(400)/`Origin`-allowlist(403)/
  token(401) ordering, checked before the upgrade completes; the client
  offers two subprotocol values (`converge.v1` and the token-bearing
  value), the server validates the token-bearing one and selects only
  `converge.v1` in response (corrected — correction 1). On success: exactly
  three message shapes exist (corrected this pass — correction 9: the
  previous draft said "two" here while listing all three), tagged by a
  `"type"` field (`{"type":"hello","serverTime":...}` /
  `{"type":"ping"}` / `{"type":"pong","serverTime":...}`) — no terminal,
  PTY, or provider streaming semantics (Out of Scope, spec.md).
- **Generation**: `crates/contracts` holds the Rust types
  (`HealthResponse`, `HealthStatus`, `ApiError`, `WsMessage`) deriving
  `ts-rs`'s `TS` trait with explicit `#[serde(rename_all = ...)]`/
  `#[serde(tag = ...)]`/`#[serde(rename_all_fields = ...)]` attributes
  (corrected — correction 5, exact Serde tagging so wire JSON, Rust field
  names, and generated TypeScript field names are related by one documented
  rule, not independently maintained). A `just contracts-generate` recipe
  writes `.ts` output into `packages/contracts/src/generated/`.
- **Drift gate**: `just contracts-check` regenerates into a temp directory
  and runs `diff` (or `git diff --exit-code` against the committed output);
  any difference is a failing exit code (FR-020). CI runs this on every
  pull request and push to the default branch.
- **Contract tests**: assert the `200`/denial wire shapes (HTTP) and the
  `hello`/`ping`/`pong` shapes plus the subprotocol-negotiation response
  (WebSocket) against the generated types, not hand-maintained duplicates
  (FR-021).

## Persistence (Phase 1 — full schema in `data-model.md`)

`events` (immutable ledger: autoincrement `id`, unique `event_id` — now a
**UUIDv7**, corrected from UUID v4 — as the idempotency key, `event_type`,
`payload`, a new `payload_hash` (BLAKE3) column used to distinguish a
legitimate idempotent resubmission from a conflicting reuse of the same
`event_id` with different content, `occurred_at`, `recorded_at`) with
`BEFORE UPDATE`/`BEFORE DELETE` triggers that `RAISE(ABORT, ...)` —
immutability is mechanically enforced at the database layer, not just by
application discipline. `aggregates` (keyed by `event_type`, fully
disposable and rebuilt inside a single `BEGIN IMMEDIATE` transaction —
corrected, correction 10 — never an incremental patch, and never partially
visible mid-rebuild). No product-domain schema is introduced — the one
representative `event_type` (`slice0.probe.recorded`) exists solely to prove
the mechanism; real provider event shapes are Slice 2 scope. Full
concurrency, conflict, and transactional-rebuild semantics: `data-model.md`.

## Command Facade (`justfile`) and CI

| Recipe | Purpose | Requirement category covered |
| --- | --- | --- |
| `just setup` | Installs pinned Rust/pnpm toolchains, runs `pnpm install`, applies migrations to a fresh local dev database. | setup/bootstrap |
| `just dev` | Runs `scripts/dev.mjs` — the one documented launch command (FR-001, AC-040), using the corrected fork-first/IPC bootstrap ordering. | development |
| `just fmt` | `cargo fmt` + `prettier --write` across the workspace. | formatting |
| `just fmt-check` | Both in check-only mode; what CI runs; never auto-fixes in CI. | formatting verification |
| `just lint` | `cargo clippy --workspace --all-targets -- -D warnings` + `eslint .` (including `typescript-eslint` and `eslint-plugin-react-hooks`, corrected pins). | linting |
| `just test-unit` | `cargo test --workspace --lib` (includes the architecture boundary allowlist check) + `vitest run --dir crates-adjacent-unit-scope` where applicable. | unit tests |
| `just test-contract` | Contract tests (including WebSocket subprotocol negotiation) + `just contracts-check` (drift gate). | integration and contract tests (contract half) |
| `just test-integration` | Live-socket, direct (non-proxied) HTTP/WS integration tests, security denial paths, migrations, idempotency (including the conflicting-duplicate case), transactional aggregate rebuild, launcher shutdown, orphan-process behavior. | integration and contract tests (integration half) |
| `just test-frontend` | Vitest + React Testing Library + `jest-axe`: Frontend State Matrix, accessibility, TanStack/Zustand separation. | frontend tests |
| `just test-e2e` | WebdriverIO minimal browser journey. | browser E2E tests |
| `just build` | Release Rust binary + Vite production bundle. | build |
| `just check` | `fmt-check` + `lint` + `test-unit` + `test-contract` + `test-integration` + `test-frontend` + `test-e2e` + `build` — **unconditionally, including E2E, with no environment-dependent skip** (corrected — correction 7: WebdriverIO's automatic browser/driver management, `research.md`, removes the previous "run `test-e2e` separately if you lack a provisioned driver" escape hatch entirely). | complete validation |
| `just contracts-generate` / `just contracts-check` | Regenerate / regenerate-and-diff TypeScript contracts. | supports FR-020 |

No recipe beyond this table is introduced. **"Integration and contract
tests"** (the category named in the plan's requirements) is deliberately
split into `test-contract` and `test-integration` for precision — both are
required to satisfy that category; neither alone does.

**CI (GitHub Actions)**: triggers `pull_request` and `push` to the default
branch (FR-015, SC-006), matching `research.md`'s pinned `ubuntu-24.04` /
`windows-2022` runners,
`actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`,
`actions/setup-node@820762786026740c76f36085b0efc47a31fe5020 # v7.0.0` with
`node-version-file: .nvmrc` (new this pass — was described only as "`actions/
setup-node` step (or equivalent)" in `research.md`, never itself SHA-pinned
alongside the other two actions), and SHA-pinned
`dtolnay/rust-toolchain@2c7215f132e9ebf062739d9130488b56d53c060c` with
`toolchain: "1.97.1"` — **the literal `<exact-sha>` placeholders previously
in this line are corrected to their real, GitHub-REST-API-verified commit
SHAs this pass** (all three actions' exact values and verification dates in
`research.md`'s GitHub Actions section).

- **Linux job**: runs `just check` in full — unconditionally, including
  `just test-e2e`, since `just check` itself now always includes it.
- **Windows job**: runs `just fmt-check`, `just lint`, `just test-unit`,
  `just test-contract`, `just test-integration`, `just build` — the
  "build plus applicable technical tests" `docs/MVP.md` §6.4's `PLT-005`
  restriction requires. This is a documented, normatively-justified CI
  matrix boundary (Windows simply has no E2E job in this slice's CI), not
  a conditional skip inside a recipe — correction 7 specifically targeted
  the latter (the previous `just test-e2e` opt-out language in
  `quickstart.md`), which has been removed; this Windows-CI-matrix scoping
  is unaffected by that correction and remains as originally designed.

CI invokes these same recipes; it does not maintain a parallel validation
path.

## Testing Plan

Red-Green-Refactor (Constitution VI) applies to: the Clean Architecture
boundary allowlist check, the ledger idempotency/conflict/rebuild logic, the
HTTP/WS authentication and subprotocol-negotiation denial logic (security
boundary), and the contract-drift check — domain logic, application use
cases, contracts, and security boundaries, exactly per spec.md FR-016's
risk-oriented scope. UI/frontend and CI-wiring tests follow risk-appropriate
coverage without mandatory test-first ordering.

| Layer | Tool / Mechanism | Requirement IDs |
| --- | --- | --- |
| Architecture boundary **(Corrected — correction 11: allowlist, not blacklist)** | `cargo test -p domain -p application` compiled in isolation, **plus** a `cargo metadata`-driven check (parsed via `serde_json`) asserting the *normal* (non-dev) dependency set of `domain` is a subset of `{thiserror}` and of `application` is a subset of `{domain, thiserror, uuid, blake3, serde_json, time}` — exactly the allowlists documented in Project Structure above (updated this pass, point 3, to include `serde_json` and to require `uuid` without the `v7` feature). An allowlist check fails on *any* unapproved crate appearing as a normal dependency, not only a specifically-named forbidden one; `tokio` may still appear as a `dev-dependency` for `#[tokio::test]`-based unit tests without violating either allowlist, since dev-dependencies never ship in the compiled artifact and are excluded from the check by construction. The same `cargo metadata` pass also asserts `#![forbid(unsafe_code)]` (or an equivalent deny-level Clippy lint) is present at the crate root of **all six** crates, `service` included with no exception (corrected this pass — correction 5) | FR-009, FR-010, SC-005 |
| Rust unit | `cargo test` | FR-011–FR-013, FR-017 |
| Contracts + drift | Rust contract tests + `just contracts-check`, including the WebSocket subprotocol-negotiation response assertion | FR-019–FR-021, SC-008 |
| HTTP integration | Live-socket tests via `reqwest =0.13.4` (dev-dependency, exact pin — corrected this pass, was described only as "a `reqwest`-class client") against a bound instance, exercising the real `Host` validation `axum::extract::ConnectInfo`/oneshot testing can't fully exercise at the TCP layer; includes the CORS preflight test (`contracts/http-health.md`'s CORS Preflight — correction 1, previous pass, extended this pass, point 1: an absent `Access-Control-Request-Method` case, and a `200`-response `Access-Control-Allow-Origin`-presence assertion) | FR-002–FR-008, SC-002 |
| WebSocket handshake | Integration tests using `tokio-tungstenite =0.30.0` (dev-dependency, exact pin — corrected, was a placeholder) driving real upgrade requests with custom `Sec-WebSocket-Protocol` (both offered values), `Host`, and `Origin` headers | FR-022, SC-009 |
| Migrations | `sqlx migrate run` against an empty DB + re-run no-op assertion | FR-011, AC-042 |
| Idempotency / conflict **(Corrected — corrections 6 and 10, this pass broadens the match/conflict criteria)** | Sequential and concurrent (N-simultaneous) resubmission of the same `event_id` with **every** producer-controlled field identical — `event_type`, canonical `payload`/`payload_hash`, and `occurred_at` (expect one row, idempotent success) — and, separately, three individual-field-mismatch cases: same `event_id` with a different `event_type` only, a different `payload` only, and a different `occurred_at` only (each expects the typed `IdempotencyConflict` error naming exactly that field via `ConflictingFields`, never a silent success or a duplicate row — `data-model.md`'s Event identity section); discard+rebuild equivalence assertion | FR-012, FR-013, SC-003, SC-004 |
| Transactional rebuild **(New — correction 10)** | A deliberately injected mid-fold failure leaves `aggregates` unchanged from its pre-rebuild state (rollback assertion); a concurrent `events` insert attempted while a rebuild transaction is open blocks and only proceeds after the rebuild resolves | FR-013, SC-004 |
| Launcher shutdown / orphan process **(Corrected — correction 4, this pass: replaces the previous indirect "a later `just dev` still succeeds" assertion with a direct child-lifecycle guarantee)** | Graceful case: send the supervisor `SIGINT`/`SIGTERM`; assert both children's specific PIDs (captured before the signal) exit within the bounded timeout. Orphan case: `SIGKILL` the supervisor itself — bypassing every cleanup handler it registered, so nothing the supervisor's own code does can be responsible for the outcome — and assert the **same two specific child PIDs** are confirmed exited (via a portable liveness probe: POSIX `process.kill(pid, 0)` raising `ESRCH`, or the Windows-equivalent process-existence check) within the bounded timeout, independent of and prior to attempting any subsequent `just dev` run. See `research.md`'s corrected mechanism (below this table's cross-reference) for exactly why this holds even under an uncatchable signal. | FR-001, FR-018, `research.md` Bootstrap design |
| Redaction | A negative-grep check over captured test/log output asserting the token string, and the token-bearing WebSocket subprotocol value, never appear in a URL-shaped, header-echoed, or plain-text log line | FR-017, `research.md` validation approach |
| Frontend state matrix | Vitest + React Testing Library, one test group per applicable state: Initial loading, Ready, Refreshing/Stale, Service unavailable/Offline, Error (Empty/Disabled intentionally absent — spec.md's Frontend State Matrix) | FR-023, SC-010 |
| TanStack/Zustand separation | Vitest + RTL asserting displayed data updates flow through query invalidation/refetch, not through Zustand | FR-024, SC-011 |
| Accessibility | `jest-axe =10.0.0` (corrected — was a placeholder) `toHaveNoViolations` assertion plus a keyboard-navigation test on the health/status surface | FR-023, SC-010 |
| Browser E2E | WebdriverIO: load the frontend, observe the Ready state, confirm the WS proof (`hello` received); now unconditionally part of `just check` (corrected) | FR-016, AC-043 |

## Frontend State Matrix — Implementation Notes

Directly implements spec.md's approved Frontend State Matrix; no state is
invented beyond it and none of the five applicable states is skipped.

**Reported, unapplied correction (transparency required by
AGENTS.md/CLAUDE.md's "report conflicts, don't silently resolve them")**:
this revision's mandatory-correction instructions requested mapping network
failures to "Offline/Unavailable" and authenticated HTTP denial responses
(400/401/403) to "Error/Connection rejected." spec.md's approved Frontend
State Matrix (out of scope for this plan-only revision) explicitly assigns
authentication failure to **Service unavailable/Offline**, and explicitly
states the Error state is "distinguished from Service unavailable, which is
a **transport or authentication failure**" — i.e., spec.md already excludes
authentication denials from Error by name. Applying the requested mapping
here would silently contradict the approved specification. Per the human
maintainer's explicit direction during this revision, **spec.md is treated
as the source of truth** and this correction was **not applied**; the
mapping below is unchanged from the previous draft and remains consistent
with spec.md. If the state-to-error mapping should actually change, that is
a product-level decision belonging in a spec.md revision, not a plan-only
correction.

- **Initial loading**: rendered while the TanStack Query health query is
  `pending` and has no prior data, **and** while the frontend's own
  dev-bootstrap fetch is still waiting on the Rust service's port/token to
  arrive over IPC (the `503` case in `research.md`'s corrected Bootstrap
  Ordering step 6) — both are the same user-observable state, since neither
  has a health result yet; text/icon-based, not color-only.
- **Ready**: rendered on a successful health response; the `checkedAt`
  value from `contracts/http-health.md` is what "Refreshing/Stale" (below)
  compares against.
- **Refreshing/Stale**: rendered when a query refetch is in flight while
  prior data exists, or when `checkedAt` exceeds the expected refresh
  interval — text/icon distinguishes it from Ready, never color alone.
- **Service unavailable/Offline**: rendered on a network-level failure
  (connection refused/timeout) **or** an auth denial (400/401/403) —
  matching spec.md's approved Frontend State Matrix exactly (see the
  reported, unapplied correction above); distinct from Error, and never
  silently continuing to show a stale Ready state.
- **Error**: rendered when the service responds but reports an unexpected
  shape or status outside the defined contract (not a transport failure and
  not an authentication denial — both of those are Service
  unavailable/Offline, per spec.md).
- **Empty / Disabled**: not implemented — spec.md's Frontend State Matrix
  documents why (no collection content; no interactive control), and this
  plan does not manufacture either state to fill out the matrix.

Server state (the health query result) lives in TanStack Query exclusively
(FR-024); Zustand, if used at all in this slice, holds only transient
layout state (e.g., whether a details panel is expanded) and is asserted by
test to never become an alternate source of the health data.

## Complexity Tracking

No entries. The six-crate Rust structure and the `contracts`/`packages`
split are the minimum needed to keep Domain and Application mechanically
free of transport/DB/UI dependencies (Constitution II) while still
generating deterministic TypeScript contracts (Constitution IV) — removing
any of the six crates would either merge a boundary that must stay
mechanically checkable or force wire-format derive attributes onto
domain/application types.

## Constitution Check — Post-Design Gate

*Re-checked after Phase 1 design (data-model.md, contracts/, quickstart.md,
traceability.md).*

| Principle | Post-design verification | Status |
| --- | --- | --- |
| I. Versioned Sources of Truth | Every design element in this plan cites an FR-/SC-/AC-/PRD-ID (`traceability.md`); no new product requirement was introduced. The one reported-but-unapplied correction (state-matrix error mapping) was resolved in spec.md's favor, not silently. | **PASS** |
| II. Modular Clean Architecture | `contracts` crate sibling design keeps `domain`/`application` free of `ts-rs`/serde-wire attributes; mechanical allowlist check designed (Testing Plan, correction 11). | **PASS** |
| III. Secure Local Runtime | `research.md` Part 2's corrected design (bootstrap ordering, direct access, subprotocol negotiation) satisfies loopback/random-port/ephemeral-token/Host/Origin/CORS/WS-allowlist requirements with no shell/unrestricted-FS surface introduced. | **PASS** |
| IV. Contracts as Code | `ts-rs` + `just contracts-check` drift gate designed with exact Serde tagging (correction 5); HTTP and WS contract tests designed (FR-021). | **PASS** |
| V. Durable and Reconstructable Data | `data-model.md`'s trigger-enforced immutability, `payload_hash` conflict detection, and transactional full-refold rebuild satisfy FR-012/FR-013 exactly as designed (correction 10). | **PASS** |
| VI. Risk-Oriented Test-First Development | Testing Plan scopes RGR to domain/application/contracts/security exactly, per FR-016, now including conflict and transactional-rebuild cases. | **PASS** |
| VII. Product and UX Integrity | Frontend State Matrix section implements all five applicable states now, exactly as spec.md approved, with the one requested-but-conflicting correction explicitly reported rather than silently applied. | **PASS** |
| VIII. Privacy and Observability | Redaction test designed (Testing Plan), now covering the WebSocket subprotocol value; no telemetry introduced anywhere in this design. | **PASS** |
| IX. Documentation and Delivery Discipline | Command Facade table is the single source of truth for both contributors and CI; `just check` is now unconditional (correction 7); no competing path designed. | **PASS** |
| X. Human-Controlled Version Control | No git-publishing action performed or designed into any recipe. | **PASS** |

**No unresolved conflict.** One requested correction (frontend state-to-
error mapping) was found to conflict with approved spec.md and was resolved
in spec.md's favor per explicit human-maintainer direction during this
revision, documented above rather than silently applied.

**Post-design gate re-verification (final consistency pass)**: re-checked
honestly against all nine corrections applied in this pass, per this pass's
own closing instruction to re-run both gates rather than assume they still
hold:

- **I**: `traceability.md` now additionally covers `PLT-003`, `PLT-006`,
  `PLT-007`, `SPEC-002`–`SPEC-006` (correction 7) — every PRD ID spec.md's
  Out of Scope section names is now traced, none silently absent. **PASS**,
  strengthened.
- **II**: `#![forbid(unsafe_code)]` now verified uniformly across all six
  crates including `service` (correction 5); the `application` allowlist
  gained `time` (parsing-only, no wall-clock read — Constraints section)
  for a genuine, narrow need, not an unbounded expansion. **PASS**,
  strengthened.
- **III**: the CORS preflight is now fully specified end to end (correction
  1: no bearer token required on `OPTIONS`, exact-match `Host`/`Origin`/
  requested-method/requested-header checks, an exact three-header grant on
  success, no grant at all on any failure) and `Origin` is now
  unconditionally required on the substantive `GET` request, closing a gap
  where an absent `Origin` was previously only implicitly, not explicitly,
  a denial condition. The orphan-process guarantee (correction 4) no longer
  rests on unsafe FFI (no `libc::prctl`, no Windows Job Object binding) —
  it uses the same OS-level pipe/IPC-channel-teardown property on both
  platforms, keeping III's "no unrestricted FS/shell access" posture intact
  while still providing a stronger, directly-verified guarantee than the
  previous indirect test. **PASS**, strengthened.
- **IV**: no change — HTTP/WS contract shapes and Serde tagging are
  unaffected by this pass's corrections. **PASS**, unchanged.
- **V**: `data-model.md`'s idempotent-vs-conflicting classification now
  compares all three producer-controlled immutable fields
  (`event_type`/`payload_hash`/`occurred_at`, correction 6) instead of
  `payload_hash` alone, closing a real gap where a mismatched `event_type`
  or `occurred_at` could have been misclassified as a legitimate retry.
  **PASS**, strengthened.
- **VI**: Testing Plan's idempotency/conflict row and the new orphan-process
  row both test strictly more precisely than the previous draft (individual
  per-field mismatch cases; a direct PID-liveness assertion instead of an
  indirect one). **PASS**, strengthened.
- **VII**: unaffected by this pass — no frontend-state-matrix change was
  requested or made this pass. **PASS**, unchanged.
- **VIII**: unaffected by this pass beyond the CORS-preflight body note
  (preflight response bodies are never read by the browser, so no new
  secret-exposure surface is introduced by returning `ApiError` on a failed
  preflight). **PASS**, unchanged.
- **IX**: `spec.md`'s `Status` field now reads `Approved` rather than
  `Draft` (correction 8), matching how this plan and its predecessor
  revision already treated it — a documentation-accuracy fix, not a new
  approval being granted here. **PASS**, unchanged in substance.
- **X**: this pass performed no git action; the `spec.md` Status edit and
  all other edits in this pass are plain file edits pending human review,
  identical in kind to every other edit in this plan. **PASS**, unchanged.

**No unresolved conflict from this pass.** All nine corrections either
strengthen an already-passing gate or leave it unchanged; none introduces a
new conflict with `spec.md` or any other normative source.

**Post-design gate re-verification (final blocking-consistency pass)**:
re-checked honestly against all four corrections in this third pass, per
this pass's own closing instruction to re-run both gates:

- **I**: **not fully resolved by this pass — reported, not silently
  passed.** Point 4 required verifying `docs/PRODUCT.md` was human-approved
  before syncing its `Status` field to `Approved`; it currently reads
  `Draft for approval`, and no evidence of a separate approval event exists
  in this repository (git history, commit messages, or any other normative
  document) as of this check. This plan does not treat `PRODUCT.md` as
  approved, does not change its `Status` field, and does not update any
  "Constitution sync report" on the strength of an assumption — doing so
  would be exactly the "silently choose one interpretation" `AGENTS.md`
  prohibits when a required approval is missing. Every other traceability
  fact in `traceability.md` is otherwise unaffected by this pass. **Gate
  status: open on this one point pending explicit human direction; every
  other aspect of Principle I remains PASS.**
- **II**: `application`'s allowlist now exactly matches its real
  responsibilities (`serde_json` added for the canonical serialization it
  performs, `uuid`'s unused `v7` feature removed), and
  `adapters-persistence`'s allowlist had `serde_json` removed since it no
  longer performs serialization — both allowlists are now internally
  consistent with `data-model.md`'s Event identity section instead of
  contradicting it, closing a real design inconsistency `research.md`
  previously introduced (claiming `application` generates `event_id` and
  `adapters-persistence` serializes/hashes the payload, both wrong). **PASS**,
  strengthened.
- **III**: the CORS preflight now denies an absent
  `Access-Control-Request-Method` identically to a wrong one (closing a gap
  where a malformed, non-preflight `OPTIONS` request had no defined
  outcome), and the actual `200` response now explicitly requires
  `Access-Control-Allow-Origin` — without it, a browser would block the
  frontend from reading a response that had already passed every
  server-side security check, a real functional/security-adjacent gap (a
  correctly-authenticated, correctly-originated request silently unusable
  from the browser) rather than a documentation nicety. **PASS**,
  strengthened.
- **IV**: no change — contract shapes and Serde tagging are unaffected by
  this pass. **PASS**, unchanged.
- **V**: no change to `data-model.md`'s schema or rebuild semantics this
  pass — the Event identity section's wording (who generates `event_id`,
  which layer hashes the payload) was already correct; only `research.md`'s
  contradiction of it is corrected. **PASS**, unchanged.
- **VI**: Testing Plan's architecture-boundary row now asserts the
  corrected `application` allowlist (including `serde_json`, excluding
  `uuid`'s `v7` feature) and the HTTP integration row now includes the two
  new CORS assertions (point 1). **PASS**, strengthened.
- **VII**: unaffected by this pass. **PASS**, unchanged.
- **VIII**: unaffected by this pass — no new secret-exposure surface;
  `jsdom`/`@testing-library/dom` are test-only dev-dependencies with no
  runtime/production exposure. **PASS**, unchanged.
- **IX**: unaffected by this pass beyond the Command Facade/testing-plan
  wording updates already reflected above; no competing validation path
  introduced. **PASS**, unchanged.
- **X**: this pass performed no git action; every edit remains a plain file
  edit pending human review. **PASS**, unchanged.

**Post-design gate re-verification (human-approval confirmation pass)**: the
human maintainer has confirmed `docs/PRODUCT.md` version 1.0.0 as formally
approved. Its `Status` field now reads `Approved` and its `Last updated`
date is synchronized to the date of this confirmation.

- **I**: **resolved — now PASS without qualification.** The prior pass's
  open point (whether `docs/PRODUCT.md` had been human-approved before its
  `Status` field could be synced) is closed by explicit human confirmation.
  `docs/PRODUCT.md`'s `Status` field has been updated accordingly. The
  Constitution's Sync Impact Report in `.specify/memory/constitution.md`
  (Follow-up TODOs note) has been synchronized to this same confirmation: its
  approval date for `docs/PRD.md`, `docs/PRODUCT.md`, and `docs/DESIGN.md`
  now reads `2026-07-23`, matching `docs/PRODUCT.md`'s updated `Last
  updated` date. **PASS**.
- **II through X**: unaffected by this confirmation; each remains at the
  status recorded in the "final blocking-consistency pass" re-verification
  above. **PASS**, unchanged.

**No unresolved item remains from this pass.** The previous pass's single
open point — Principle I's gate on `docs/PRODUCT.md`'s approval status — is
now closed by explicit human confirmation, and the alternative under which
this plan could have proceeded with `docs/PRODUCT.md` left in `Draft for
approval` no longer applies. Per this pass's own explicit instructions,
`/speckit-tasks` and `/speckit-implement` were NOT run, and no production
code was written. This plan stops here for final human review.
