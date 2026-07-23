# Research: Slice 0 — Foundation

**Input**: `specs/001-slice-0-foundation/spec.md`; Technical Context in `plan.md`

**Revision note (this pass)**: this document was corrected after a
consistency review found: an under-specified WebSocket subprotocol
negotiation, a bootstrap ordering that could not actually produce a correct
CORS/Origin allowlist, floating (non-exact) Cargo version syntax, several
undocumented placeholder dependencies, an inaccurate cookie-scoping
statement, and a missing decision for `event_id`/content-hash generation.
Each correction below is marked **(Corrected)** at its section heading. All
version pins were re-verified directly against the npm registry and
crates.io API on **2026-07-22**; GitHub Action references were re-verified
against the GitHub REST API on the same date.

---

## Part 1 — Foundational Version Pins

### Exact-pin policy **(Corrected)**

Every Cargo dependency below is pinned with the `=` operator
(e.g. `axum = "=0.8.9"`), not a bare version string. In Cargo's version
syntax, a bare `"0.8.9"` is implicitly `^0.8.9` — it still permits Cargo to
resolve `0.8.10`, `0.9.0`-if-pre-1.0-caret-rules-allow-it, etc., on a fresh
`cargo update`. That is exactly the "floating or unbounded foundational
dependency version" this plan's technical baseline prohibits. The `=`
operator pins the exact version Cargo will select for that direct
dependency, full stop.

This is deliberately **layered** with a second, complementary mechanism:
`Cargo.lock` is committed to the repository and MUST NOT be deleted or
regenerated casually. `=` pins fix the *direct* dependencies this project
declares; `Cargo.lock` fixes the *entire* resolved graph, including every
transitive dependency neither this document nor `Cargo.toml` names
explicitly. Neither alone is sufficient: `=` pins without a committed lock
file would still let transitive dependencies drift; a committed lock file
without `=` pins would still let `cargo update <direct-dep>` silently move a
direct dependency forward. Both together give full reproducibility.

The same exactness intent applies to npm/pnpm dependencies, expressed the
npm way: every `package.json` entry below is written with **no** range
operator (no `^`, no `~`) — an npm version string with no prefix is already
exact. `pnpm-lock.yaml` is committed for the same transitive-graph reason as
`Cargo.lock`.

### Rust Toolchain

**Decision**: `1.97.1`, `edition = "2024"`, channel `stable`.

**Rationale**: 1.97.1 is the current stable release (2026-07-16), a
same-day-class patch over 1.97.0 (2026-07-09) that backports an LLVM fix for
a miscompilation present since 1.87. Edition 2024 requires Rust ≥1.85
(stabilized in 1.85.0); 1.97.1 is far above that floor. Pinning the exact
patch release rather than the floating `stable` channel avoids both the
known 1.97.0 miscompilation and cross-machine/CI reproducibility drift.

**Alternatives considered**: `1.97.0` — rejected, superseded by a real
miscompilation fix. A floating `stable` channel — rejected, breaks
reproducibility across contributor machines and CI runs on different days.

**Source**: [Rust 1.97.0 release notes](https://blog.rust-lang.org/2026/07/09/Rust-1.97.0/), [Rust 1.97.1 release notes](https://blog.rust-lang.org/2026/07/16/Rust-1.97.1/), [releases.rs](https://releases.rs/)

**Enforcement**: `rust-toolchain.toml` at repo root — `channel = "1.97.1"`,
`components = ["rustfmt", "clippy"]`. CI must reference this same pinned
version explicitly (never a floating `@stable` action input). This is not a
Cargo dependency, so the `=` operator does not apply; the toolchain file
itself is the exact-pin mechanism.

### Node.js

**Decision**: `24.18.0` ("Krypton"), Active LTS line `24.x`.

**Rationale**: Node 24 is the current Active LTS (supported until
2028-04-30). Node 26 (released May 2026) is still in the "Current" phase and
won't reach Active LTS until roughly October 2026 — using it now means
tracking a moving, non-LTS target. Node 22 is already Maintenance LTS
(security-fixes-only, EOL 2027-04-30) — a worse starting point for a project
beginning now.

**Alternatives considered**: Node 22 (Maintenance LTS, shorter remaining
support window) — rejected. Node 26 (Current, not yet LTS) — rejected as
premature.

**Source**: [Node.js releases](https://nodejs.org/en/about/previous-releases), [endoflife.date/nodejs](https://endoflife.date/nodejs)

**Enforcement**: `.nvmrc` (`24.18.0`) plus `package.json#engines.node` set
to `">=24.18.0 <25"`. CI's `actions/setup-node` step (or equivalent) reads
`.nvmrc` via `node-version-file: .nvmrc` — one source of truth.

### pnpm

**Decision**: `11.15.1`, pinned via Corepack.

**Rationale**: Current stable release on the npm registry, fully compatible
with Node 24 LTS. Corepack (bundled with Node ≥16.13, including Node 24) is
the standard install/pin mechanism via `package.json#packageManager`,
letting every contributor and CI runner resolve the identical pnpm binary
with no separate global install step.

**Alternatives considered**: `12.0.0-alpha.17` — rejected, pre-release.
Switching to npm/Yarn — out of scope; pnpm was already the fixed
architectural choice.

**Source**: [pnpm on npm](https://www.npmjs.com/package/pnpm), [pnpm blog](https://pnpm.io/blog)

**Enforcement**: `package.json`: `"packageManager": "pnpm@11.15.1"`, with
`corepack enable` in CI setup and contributor onboarding docs. Corepack
hard-fails on a version mismatch — the desired behavior.

### tokio

**Decision**: `tokio = "=1.53.1"` **(Corrected: exact pin)**.

**Rationale**: Current latest stable (2026-07-20). Tokio maintains both a
rolling latest and dedicated LTS lines (1.47.x LTS until Sep 2026, 1.51.x
LTS until Mar 2027); since this is a fresh project with no existing
production deployment to protect from churn, rolling latest keeps the
dependency tree current without stacking an extra LTS constraint on top of
the project's own versioning discipline.

**Alternatives considered**: Pinning the 1.51.x LTS line — defensible for
an organization protecting a long-lived stable branch; rejected here as
unnecessary for a slice with no prior production surface.

**Source**: [tokio on docs.rs](https://docs.rs/crate/tokio/latest), [tokio on crates.io](https://crates.io/crates/tokio)

**Enforcement**: Workspace `Cargo.toml`: `tokio = { version = "=1.53.1", features = [...] }` — exact pin; `Cargo.lock` committed. `tokio` MUST appear only
as a `dev-dependency` in `domain` and `application` (see Architecture
Boundary allowlists below); it is a normal dependency only in
`adapters-http`, `adapters-persistence`, and `service`.

### HTTP/WebSocket framework — axum (chosen over actix-web)

**Decision**: `axum = "=0.8.9"` with `tower-http = "=0.7.0"` **(Corrected:
exact pins)**.

**Rationale**: Axum is maintained by the Tokio team and built directly on
`tower`/`hyper`, so `tower-http` middleware (CORS, tracing, timeouts) applies
with no adapter glue — a direct fit for the CORS requirement (FR-005).
`axum::extract::ws` provides HTTP and WebSocket upgrade in the same
framework/router, matching the "one framework for both" need. It shares its
async runtime and error model natively with `tokio` and integrates cleanly
with `sqlx`'s async pool via ordinary `Arc<Pool>` state extraction — no
actor-model translation layer, unlike actix's WebSocket handling.

**Alternatives considered**: `actix-web` — more mature at very high
concurrency (10-15% higher raw throughput in benchmarks) with a longer track
record, but its actor-model architecture is heavier ceremony for a modular
monolith that needs no actor system elsewhere, and its middleware ecosystem
sits apart from `tower-http`. Rejected: raw throughput is not the bottleneck
for a local-only, single-user service; Tower/tokio ecosystem fit and direct
CORS middleware reuse matter more here.

**Source**: [axum on docs.rs](https://docs.rs/crate/axum/latest), [tower-http on docs.rs](https://docs.rs/crate/tower-http/latest)

**Enforcement**: `Cargo.toml`: `axum = { version = "=0.8.9", features = ["ws"] }`, `tower-http = { version = "=0.7.0", features = ["cors", "trace"] }`. Both are normal dependencies only in `adapters-http` and `service`.

### sqlx (+ sqlx-cli)

**Decision**: `sqlx = "=0.9.0"` (features `sqlite`, `runtime-tokio`,
`macros`, `migrate`), `sqlx-cli = "=0.9.0"` **(Corrected: exact pins)**.

**Rationale**: Current stable release (2026-05-21) with a `sqlite` driver
and compile-time/offline query checking (`sqlx::query!` + `cargo sqlx
prepare`). Native, forward-only, timestamp-ordered migrations via `sqlx
migrate add`/`sqlx migrate run` match FR-011 directly, no extra tooling.
**Note**: the project moved from `launchbadge/sqlx` to `transact-rs/sqlx` as
of the 0.9.0 announcement — update any bookmarked repo URLs accordingly.

**Alternatives considered**: Diesel (sync-first, awkward fit with an async
tokio/axum stack) — rejected. SeaORM (higher-level ORM, less direct
compile-time SQL verification) — rejected in favor of SQLx's raw
compile-checked queries, a better fit for a contracts-first, minimal-magic
architecture.

**Source**: [sqlx-cli on docs.rs](https://docs.rs/crate/sqlx-cli/latest), [sqlx 0.9.0 announcement](https://github.com/launchbadge/sqlx/discussions/4271), [transact-rs/sqlx](https://github.com/transact-rs/sqlx)

**Enforcement**: `Cargo.toml`: `sqlx = { version = "=0.9.0", default-features = false, features = ["sqlite", "runtime-tokio", "macros", "migrate"] }`; `sqlx-cli` pinned via `cargo install sqlx-cli --version 0.9.0 --locked` in CI; `migrations/` with SQLx's forward-only naming convention; `.sqlx/` offline query cache committed for CI builds without live DB access. `sqlx` is a normal dependency only in `adapters-persistence` and `service`.

**Unsafe-Rust FFI boundary note (new — correction 13; tightened — correction
5, this pass)**: SQLx's SQLite driver depends transitively on
`libsqlite3-sys`, which wraps the C SQLite library via FFI and therefore
contains `unsafe` code — this is standard for *any* SQLite binding in any
language and is outside Converge's own source tree. The Rust quality
baseline's "no unsafe Rust without separate justification and approval"
rule applies to **Converge-owned code** (the crates under `crates/` and
`apps/web`'s Rust surface, i.e. none, since the frontend has none) — it
does not, and cannot, retroactively rewrite a third-party C-interop crate.
No `unsafe` block exists or is planned in any Converge-owned crate for
Slice 0; the architecture boundary check (Testing Plan, `plan.md`) verifies
that `#![forbid(unsafe_code)]` is active in **all six** crates uniformly —
`domain`, `application`, `contracts`, `adapters-http`,
`adapters-persistence`, **and `service`** (corrected this pass: the
previous draft excluded `service` "as the composition root most likely to
need a documented, reviewed exception in a later slice," but no such
exception was ever written, requested, or justified — that carve-out
granted an unused permission rather than reflecting an actual need, exactly
the kind of unjustified exception the Rust quality baseline's "unsafe Rust
prohibited unless separately justified and approved" rule exists to
prevent by default. `service`'s own responsibilities — binding a TCP
listener, spawning the Axum server, generating a token via `rand::OsRng`,
signal handling via `tokio::signal`, and the corrected parent-liveness
watchdog (Launcher shutdown and orphan-process behavior, below) — are all
achievable through safe, standard-library-or-`tokio`-provided APIs with no
FFI of any kind; the corrected watchdog design specifically avoids
`libc::prctl`/Windows Job Object FFI bindings that an earlier design
sketch considered, precisely so this uniform `forbid(unsafe_code)` could
hold with nothing to work around). This is a documentation correction, not
a design change — no protective boundary was ever waived; the previous
`service` exception simply described a possibility that never
materialized into an actual exception, and removing it closes that gap
before a later slice could have exploited it without review.

### Rust→TypeScript contract generation — ts-rs (chosen over specta)

**Decision**: `ts-rs = "=12.0.1"` **(Corrected: exact pin)**, using its
`serde-compat` feature (enabled by default) to honor `#[serde(tag = ...)]`,
`#[serde(rename_all = ...)]`, and `#[serde(rename_all_fields = ...)]` on the
wire types below.

**Rationale**: A small, standalone derive-macro crate with no Tauri or
app-framework dependency, actively maintained, producing deterministic,
git-diffable `.ts` output per type — exactly what a CI regenerate-and-diff
drift check (FR-020) needs. Its `serde-compat` feature was specifically
re-verified this pass (`docs.rs/ts-rs/12.0.1`) to confirm `tag`,
`rename-all`, and `rename-all-fields` are all supported and produce a
TypeScript discriminated union for a tagged enum — required for the
WebSocket message contract below (correction 5).

**Alternatives considered**: `specta` — its 2.0 line, the version with the
modern export API, is still a release candidate (`2.0.0-rc.25`) after
roughly three years, a meaningful maintenance/stability red flag for a
pinned foundation dependency. (Confirmed the base `specta` crate has no
Tauri dependency on its own — Tauri integration lives in a separate
`tauri-specta` crate — so "no Tauri" alone doesn't rule it out; its RC
status does, for a dependency that needs a stable, well-understood output
format.)

**Source**: [ts-rs on docs.rs](https://docs.rs/crate/ts-rs/latest), [specta on docs.rs](https://docs.rs/specta/latest/specta/), [specta-rs/specta](https://github.com/specta-rs/specta)

**Enforcement**: `Cargo.toml`: `ts-rs = "=12.0.1"` on the crate(s) holding
contract structs; a documented generation recipe (see Command Facade in
`plan.md`) writes `.ts` files into `packages/contracts/`; CI re-runs
generation and fails (`git diff --exit-code`) on any difference from
committed output.

### Event identity and integrity — UUIDv7 + BLAKE3 **(New decision this
revision — not a pre-existing normative choice)**

No normative source (`.specify/memory/constitution.md`, `docs/PRODUCT.md`,
`docs/PRD.md`, `docs/MVP.md`, `docs/DESIGN.md`) mentions UUIDv7 or BLAKE3 —
this was verified by an explicit repository-wide search (including git
history) before writing this section; none of those terms appear anywhere.
The previous draft of this document used UUID v4 for `event_id` with no
content-hash mechanism. On review, the human maintainer directed this
revision to adopt UUIDv7 + BLAKE3 instead, recorded here as a **new,
plan-level engineering decision**, not as the correction of a
pre-existing approved choice — it does not alter any FR/SC/AC identifier
and is confined to `data-model.md`'s persistence design.

**Decision**: `uuid = "=1.24.0"` for the `event_id` **type** (parsing and
validating the producer-supplied UUIDv7 string, per FR-012 — see the
ownership correction in Enforcement below); `blake3 = "=1.8.5"` for a
`payload_hash` column added to `events` (see `data-model.md`).

**Rationale**:

- **UUIDv7 over UUIDv4** for `event_id`: UUIDv7 embeds a 48-bit millisecond
  Unix timestamp in its most significant bits, making generated IDs
  monotonically-increasing-by-creation-time (RFC 9562). This gives the
  ledger's producer-supplied idempotency key a secondary, human-inspectable
  ordering property useful for debugging/audit (e.g. `ORDER BY event_id` is
  now approximately chronological) without displacing the ledger's own
  authoritative order, which remains `events.id` (`AUTOINCREMENT`) — the
  `event_id` ordering is a convenience, never the source of truth for replay
  order (`data-model.md`'s rebuild procedure still folds by `id ASC`).
  UUIDv4's fully-random layout gave no such property.
- **BLAKE3 payload hash** as a second idempotency signal, alongside the
  `UNIQUE` constraint on `event_id`: the idempotency contract (FR-012)
  assumes that resubmitting "the same event" means the same `event_id`
  *and* the same `payload`. Without a stored hash, a resubmission carrying
  the same `event_id` but a **different** `payload` (a producer bug or a
  malicious/confused caller) would be indistinguishable, at the constraint
  level, from a legitimate idempotent retry — SQLite's `UNIQUE` constraint
  only knows the *key* collided, not whether the *content* matches. Storing
  a `payload_hash` lets the application layer tell these two cases apart and
  respond correctly to each (`data-model.md`'s Concurrency and Conflict
  Behavior section, and Testing Plan in `plan.md`, correction 10). BLAKE3
  was chosen over SHA-256 for this purely-internal integrity check because
  it is markedly faster with no cryptographic-strength requirement here (the
  hash is a duplicate-detector, not a security boundary) and has a mature,
  audited, actively maintained Rust-native crate.

**Alternatives considered**: Keeping UUIDv4 — rejected per the human
maintainer's explicit direction this revision, and because UUIDv7's ordering
property has no downside for a producer-supplied idempotency key. SHA-256
(via `sha2`) for the payload hash — rejected in favor of BLAKE3's better
throughput for a hash computed on every single insert, with no offsetting
security requirement that would justify SHA-256's slower, more
widely-standardized profile here (this hash is never used as a security
token or exposed externally — see `data-model.md`). A composite
`(event_id, payload)` `UNIQUE` constraint instead of a stored hash column —
rejected: SQLite can index a hash column cheaply and uniformly regardless of
`payload`'s size, and a stored hash is directly comparable without
re-parsing the full JSON payload on every duplicate check.

**Source**: [uuid crate on crates.io](https://crates.io/crates/uuid), [RFC 9562 (UUID v7)](https://www.rfc-editor.org/rfc/rfc9562.html), [blake3 crate on crates.io](https://crates.io/crates/blake3), [BLAKE3 spec/repo](https://github.com/BLAKE3-team/BLAKE3)

**Enforcement (corrected — final blocking-consistency pass, point 3)**:
`Cargo.toml`: `uuid = "=1.24.0"` (default features — **no** `v7` feature) and
`blake3 = "=1.8.5"` as normal dependencies of `application` only. This
corrects a genuine contradiction the previous pass introduced: this line
previously read "ID and hash generation is an application-layer concern,"
which conflicts with `data-model.md`'s "Event identity: generation,
preservation, and canonical comparison" section — **the producer alone
generates `event_id`, exclusively, and `application` never generates,
mutates, or substitutes it** (that section's own wording, unchanged since
the previous pass; only this line's contradiction of it is corrected now).
`application` depends on `uuid` solely for the `Uuid` **type** — parsing and
validating the producer-supplied `event_id` string as a well-formed UUID
when `RecordProbeEvent` receives it, and typing it through the use case and
the `EventRepository` port signature — never to call a generation function.
The `v7` feature specifically gates `Uuid::now_v7()` (the generation
method); since `application` never generates an `event_id`, it does not
enable that feature. The producer that *does* generate a UUIDv7 — in this
slice, exclusively the integration test suite (`data-model.md`) — enables
`features = ["v7"]` on its own `uuid` dev-dependency edge (`tests/
integration/`'s own `Cargo.toml`/test target), which is outside `application`'s
normal-dependency allowlist and therefore outside the architecture-boundary
check's scope by design (dev-dependencies never ship in the compiled
artifact — Testing Plan, `plan.md`). BLAKE3 **hash generation** genuinely is
an `application`-layer concern, unchanged: computing `payload_hash` is part
of the idempotency/conflict business rule `RecordProbeEvent` evaluates
(`data-model.md`'s Concurrency and Conflict Behavior), not a domain
invariant and not an adapter I/O concern.

### Token/entropy generation

**Decision**: `rand = "=0.10.2"` **(Corrected: exact pin)**, using
`rand::rngs::OsRng` (CSPRNG-backed via the OS entropy source through
`getrandom`), encoded base64url (unpadded) via `base64 = "=0.22.1"`'s
`URL_SAFE_NO_PAD` engine (its own pinned entry, "base64url encoding —
base64," above — new this pass; this section always specified the
encoding but never named the crate performing it) for the ephemeral auth
token bytes.

**Rationale**: `rand` 0.10.2 is current stable; its OS-backed RNG is the
standard, audited path for security-sensitive randomness across the Rust
ecosystem. Base64url encoding (RFC 4648 §5) is chosen over standard base64
specifically because standard base64's `+`/`/`/`=` characters are **not**
valid HTTP "token" characters (RFC 7230) and therefore not usable as-is in
an `Authorization: Bearer` value or a `Sec-WebSocket-Protocol` value (see
Part 2, Token Transport) — base64url's alphabet (`A-Z a-z 0-9 - _`) is.

**Alternatives considered**: `uuid` v4 for the *token* specifically (distinct
from the `event_id` decision above) — rejected for the same reasons as
before: a 128-bit UUID is needlessly structured (dashes, version/variant
bits reduce usable entropy to 122 bits) for a raw bearer-token; a direct
32-byte (256-bit) `rand`-generated buffer gives more entropy with a simpler
model and no wasted format bits.

**Source**: [rand on docs.rs](https://docs.rs/crate/rand/latest), [rand changelog](https://github.com/rust-random/rand/blob/master/CHANGELOG.md)

**Enforcement**: `Cargo.toml`: `rand = "=0.10.2"` in the `service` crate,
which owns token generation at startup (the only crate that needs it as a
normal dependency).

### Typed error handling

**Decision**: `thiserror = "=2.0.19"` **(Corrected: exact pin)** for all
Domain/Application error types; `anyhow` deliberately **not** added as a
workspace dependency.

**Rationale**: `thiserror`'s derive macro produces precise, matchable error
enums at zero runtime cost — required so Domain/Application errors can be
pattern-matched by callers and mapped to typed HTTP/WS error contracts, per
the Rust quality baseline's "explicit typed errors at application and
boundary layers." `anyhow` has a legitimate, narrow place only at true
binary/process edges (e.g., `main.rs` startup where an error is only ever
logged before the process exits) — never in Domain, Application, or
Adapter public function signatures.

**Alternatives considered**: Broad `anyhow` use for convenience — rejected,
erases the type information typed HTTP error responses and contract
generation need. `eyre` — same objection.

**Source**: [thiserror on docs.rs](https://docs.rs/crate/thiserror/latest), [anyhow on crates.io](https://crates.io/crates/anyhow)

**Enforcement**: `thiserror = "=2.0.19"` at workspace level, applied
per-crate; `anyhow` absent from `[workspace.dependencies]` — a binary crate
needing it at its outermost edge adds it as a direct, non-workspace
dependency scoped only to that crate, making the exception visible in
review.

### WebSocket test client — tokio-tungstenite **(New: was a placeholder)**

**Decision**: `tokio-tungstenite = "=0.30.0"`, `dev-dependency` only, in the
integration test crate/target under `tests/integration/`.

**Rationale**: A low-level WebSocket client is required to drive real
upgrade requests with custom `Sec-WebSocket-Protocol`/`Host`/`Origin`
headers (Testing Plan, `plan.md`) — something a browser-hosted client cannot
do (browsers control those headers). `tokio-tungstenite` is the
`tokio`-native async wrapper around `tungstenite`, the same low-level crate
family already implicitly assumed by the original plan; this pin makes it
concrete. It shares `tokio` as its runtime, avoiding a second async runtime
in the test binary.

**Alternatives considered**: `async-tungstenite` (runtime-agnostic wrapper
supporting both `tokio` and `async-std`) — rejected as unnecessary
generality; the project is `tokio`-only end to end. Hand-rolling a raw
handshake over `tokio::net::TcpStream` — rejected as needless
reimplementation of well-tested handshake/framing logic for test code.

**Source**: [tokio-tungstenite on crates.io](https://crates.io/crates/tokio-tungstenite)

**Enforcement**: workspace `Cargo.toml` `[dev-dependencies]` (or the
integration test crate's own `Cargo.toml`): `tokio-tungstenite = "=0.30.0"`.
Being a dev-dependency, it never appears in the compiled `service` binary
and does not participate in the Architecture Boundary allowlist check
(dev-dependencies are exempt from that check by construction — see Testing
Plan).

### HTTP integration test client — reqwest **(New this pass — was described
only as "a `reqwest`-class client," never chosen or pinned)**

**Decision**: `reqwest = "=0.13.4"`, `dev-dependency` only, in the
integration test crate/target under `tests/integration/`.

**Rationale**: The previous pass's Testing Plan described HTTP integration
tests as using "a `reqwest`-class HTTP client" — naming a category, not a
concrete, exactly-pinned dependency, exactly the placeholder pattern this
pass's correction 2 targets. `reqwest` is the de facto standard, actively
maintained high-level HTTP client in the Rust ecosystem, built on the same
`tower`/`hyper` foundation `axum` (the chosen server framework) already
uses, so no second, unrelated HTTP stack enters the dependency graph purely
for testing. It provides the ergonomic request-building and assertion
surface integration tests need (setting `Host`/`Origin`/`Authorization`
headers explicitly, reading status/headers/body) that a raw
`tokio::net::TcpStream` would require hand-rolling HTTP framing for.

**Alternatives considered**: `hyper` directly (lower-level, what `reqwest`
itself is built on) — rejected as unnecessarily low-level for test code
whose job is asserting behavior, not exercising the HTTP stack's own
framing correctness. `ureq` (a simpler, synchronous client) — rejected: the
test suite already runs under `tokio` (per the architecture boundary and
the WebSocket test client above), so an async-native client avoids mixing
a blocking client into an async test harness.

**Source**: [reqwest on crates.io](https://crates.io/crates/reqwest)

**Enforcement**: workspace `Cargo.toml` `[dev-dependencies]` (or the
integration test crate's own `Cargo.toml`): `reqwest = "=0.13.4"`. Being a
dev-dependency, it never appears in the compiled `service` binary and does
not participate in the Architecture Boundary allowlist check, matching
`tokio-tungstenite`'s treatment above.

### base64url encoding — base64 **(New this pass — the token-generation
section always assumed base64url encoding but never pinned the crate that
performs it)**

**Decision**: `base64 = "=0.22.1"`, using its `URL_SAFE_NO_PAD` engine.

**Rationale**: The Token/entropy generation section below has always
specified base64url (unpadded) encoding for the ephemeral auth token bytes
and explained *why* (RFC 4648 §5's alphabet is the one HTTP "token"
character set, RFC 7230, actually permits in an `Authorization: Bearer`
value or a `Sec-WebSocket-Protocol` value) — but never named or pinned the
crate performing that encoding, an omission this pass closes. The `base64`
crate is the canonical, actively maintained Rust implementation of RFC
4648, with a purpose-built `URL_SAFE_NO_PAD` `Engine` constant matching
this exact requirement (unpadded base64url) with no custom alphabet/padding
configuration needed.

**Alternatives considered**: Hand-rolling the encoding — rejected,
needless reimplementation of a well-specified, widely-audited encoding for
a security-sensitive value (the auth token). `data-encoding` (an
alternative, also-maintained base-N crate) — rejected in favor of `base64`
as the more widely adopted, more frequently audited choice for this exact
purpose in the Rust ecosystem, with no feature `data-encoding` offers that
this project needs.

**Source**: [base64 on crates.io](https://crates.io/crates/base64)

**Enforcement**: `Cargo.toml`: `base64 = "=0.22.1"` as a normal dependency
of `service` only — token generation and encoding happen once, at startup,
in the composition root (Project Structure, `plan.md`).

### Wire serialization — serde / serde_json **(New this pass — implied by
every `#[derive(serde::Serialize, serde::Deserialize)]` in `contracts/`
and by `ts-rs`'s own dependency on Serde, but never given their own pinned
entry)**

**Decision**: `serde = "=1.0.229"` (feature `derive`), `serde_json =
"=1.0.151"`.

**Rationale**: `serde`'s derive macros are what every wire type in
`contracts/` (`HealthResponse`, `HealthStatus`, `ApiError`, `WsMessage`)
uses to implement `Serialize`/`Deserialize`, and what `ts-rs`'s
`serde-compat` feature reads to generate matching TypeScript (already
documented above in the `ts-rs` section) — Serde is a transitive
requirement of the chosen architecture, not an independent choice, but it
was never given its own pinned entry in the previous pass despite every
Serde attribute (`rename_all`, `tag`, `rename_all_fields`) in `contracts/`
depending on the exact version's attribute support. `serde_json` is the
concrete JSON backend: `adapters-http` uses it (via `axum::Json`, which
depends on it, and directly to build `ApiError` bodies), and — corrected
this pass, point 3 — `application` uses it, not `adapters-persistence`, to
produce the canonical serialized `payload` bytes `data-model.md`'s Event
identity section hashes with BLAKE3. The previous pass named
`adapters-persistence` for this in error, contradicting `data-model.md`'s
own statement that `payload_hash` is "computed by the application layer at
insert time" and that canonical serialization happens "precisely once per
`RecordProbeEvent` invocation" (an `application` use case, not a repository
method). `application`'s use of `serde_json` here is narrow and does not
require the `derive` feature or a `#[derive(Serialize)]`-bearing
application-owned struct (which would reintroduce the wire-attribute
leakage the `contracts`-crate split specifically exists to avoid, Project
Structure): `RecordProbeEvent` builds the canonical payload as a
`serde_json::Value` directly (e.g. `serde_json::json!({"note": note})`),
calls `.to_string()` on it for both the stored `payload` column and the
BLAKE3 input, and never derives `Serialize` on any `application`- or
`domain`-owned type. This keeps `serde_json`'s use in `application` a plain
data-format/hashing utility — the same category as `blake3`/`uuid`/`time`,
none of which are transport, DB, UI, filesystem, PTY, Tauri, or
provider-specific — rather than a wire-format concern, so Constitution II's
actual constraint on `application` is unaffected. `adapters-persistence`,
correspondingly, no longer depends on `serde_json` at all: its
`EventRepository` implementation receives the already-serialized `payload`
string and the already-computed `payload_hash` string from `application`
and binds them as plain SQL parameters via `sqlx` — no JSON library is
needed at the persistence boundary. All versions re-verified current stable
directly against the crates.io API on 2026-07-23.

**Alternatives considered**: none for `serde` itself — it is the
unconditional ecosystem standard `ts-rs`, `axum`, and `sqlx` all already
depend on transitively; pinning it explicitly and exactly, rather than
leaving it to float as an unpinned transitive dependency, is what this
pass's correction closes. For JSON specifically, `simd-json` (a faster,
SIMD-accelerated parser) — rejected: no measured JSON-parsing bottleneck
exists in a local, single-user, low-payload-size service to justify its
additional unsafe-SIMD-adjacent complexity and stricter platform
requirements.

**Source**: [serde on crates.io](https://crates.io/crates/serde), [serde_json on crates.io](https://crates.io/crates/serde_json)

**Enforcement (corrected — final blocking-consistency pass, point 3)**:
`Cargo.toml`: `serde = { version = "=1.0.229", features = ["derive"] }` as a
normal dependency of `contracts` (and transitively available wherever
`contracts` types are used); `serde_json = "=1.0.151"` as a normal
dependency of `application` (canonical payload serialization for BLAKE3
hashing, above — corrected from `adapters-persistence`) and `adapters-http`
(builds `ApiError`/`HealthResponse` JSON bodies) — **not**
`adapters-persistence`, whose allowlist no longer includes `serde_json`
(Project Structure allowlists, `plan.md`).

### Timestamp implementation — time **(New this pass — `occurred_at`/
`recorded_at`/`checkedAt`/`serverTime` were always specified as ISO 8601
UTC strings, but no crate was ever chosen or pinned to produce or validate
them)**

**Decision**: `time = "=0.3.54"`, features `formatting`, `parsing`, `serde`
in `adapters-http`; feature `parsing` only in `application`.

**Rationale**: Every ISO 8601/RFC 3339 timestamp this slice produces or
consumes needs exactly one of two operations: **formatting** "now" as an
RFC 3339 string (`checkedAt` in the HTTP health response, `serverTime` in
the WebSocket `hello`/`pong` messages — both generated live, in
`adapters-http`, at request time) or **parsing/validating** a
producer-supplied RFC 3339 string (`occurred_at`, validated in
`application` before a `RecordProbeEvent` call is accepted — `data-model.md`'s
Event identity section). `recorded_at` needs neither: it is generated by a
SQLite column `DEFAULT` expression (`data-model.md`), not by Rust code, so
no crate is needed for it at all — deliberately narrowing where a
timestamp dependency is required at all, not just which crate satisfies it.
`time` was chosen over `chrono` for this narrow, UTC-only need: `time`'s
API is focused on exactly the formatting/parsing operations needed here
without pulling in a full IANA timezone database this project never
queries (Converge's local service only ever produces or consumes UTC
instants — no timezone conversion is a requirement anywhere in spec.md).
Scoping `application`'s feature set to `parsing` only (no `formatting`) is
a small, deliberate architecture-boundary signal: `application` never
computes "now" — reading the wall clock would be an untestable,
time-dependent side effect inside a layer required to compile and test in
isolation (Constitution II, FR-009) — it only ever validates a
caller-supplied string.

**Alternatives considered**: `chrono` — a legitimate, more feature-rich
alternative and arguably more ubiquitous by raw download count, but
rejected here specifically because its core value (full timezone-aware
`DateTime<Tz>` arithmetic) is unused surface area for a project that only
ever handles UTC instants; `time`'s narrower scope is a better fit, not a
security or correctness distinction between the two (both are current,
actively maintained crates with no outstanding advisory as of this
verification). Hand-formatting via `strftime`-style manual string
building — rejected, needless reimplementation of RFC 3339 formatting/
parsing edge cases (fractional seconds, the trailing `Z`) a maintained
crate already handles correctly.

**Source**: [time on crates.io](https://crates.io/crates/time)

**Enforcement**: `Cargo.toml`: `time = { version = "=0.3.54", features = ["formatting", "parsing", "serde"] }` as a normal dependency of `adapters-http`; `time = { version = "=0.3.54", features = ["parsing"] }` as a normal dependency of `application` (Project Structure allowlists, `plan.md`).

### React

**Decision**: `react` / `react-dom` `19.2.8`.

**Rationale**: Current latest stable, part of the React 19.2.x line, GA and
iterating stably since 2024/2025.

**Alternatives considered**: React 18 — offers no benefit for a project
starting now targeting TanStack Query 5/Vite 8, and forgoes 19's
`use()`/Suspense improvements Testing Library 16 is already built around.
Rejected.

**Source**: [react.dev/versions](https://react.dev/versions), [React 19.2 blog post](https://react.dev/blog/2025/10/01/react-19-2)

**Enforcement**: `apps/web/package.json`: exact pins `"react": "19.2.8"`,
`"react-dom": "19.2.8"`, matching `@types/react`/`@types/react-dom`.

### Vite

**Decision**: `vite@8.1.5` **(Corrected this pass: was pinned `8.1.0`)**.

**Rationale**: Current stable Vite 8 line, compatible with Node 24 LTS and
React 19 via `@vitejs/plugin-react`. Re-checked directly against the npm
registry on 2026-07-23 (this pass's verification date) and found that
`8.1.0` — pinned and labeled "current stable" in the previous pass — had
already been superseded by two patch releases (`8.1.5` is npm's
`dist-tags.latest`). Per this pass's explicit instruction not to call a
now-superseded version "current stable," the pin is updated to the actual
current patch; nothing about the *decision* (Vite 8, not 7) changes, only
the exact patch pinned.

**Alternatives considered**: Vite 7 — superseded; no reason to start a new
project a major version behind. Rejected. Staying on `8.1.0` with an
honest "slightly behind current, no compatibility reason" rationale — also
rejected: no compatibility conflict motivates staying behind, so there is
no honest rationale for *not* moving to the current patch (contrast
TypeScript below, where a real compatibility conflict is the documented
reason for not pinning the newest release).

**Source**: [Vite releases](https://vite.dev/releases), [Vite 8 announcement](https://vite.dev/blog/announcing-vite8), npm registry (`registry.npmjs.org/vite`), queried directly 2026-07-23.

**Enforcement**: `apps/web/package.json`: exact devDependency pin
`"vite": "8.1.5"`.

### @vitejs/plugin-react **(New this pass — was assumed but never pinned)**

**Decision**: `@vitejs/plugin-react@6.0.4`.

**Rationale**: The official Vite plugin providing React's JSX transform and
Fast Refresh — required for any Vite + React setup; the previous pass's
Vite rationale already named it in prose ("via `@vitejs/plugin-react`")
without ever giving it its own pinned entry, an omission this pass closes.
6.0.4 is current stable, verified directly against the npm registry on
2026-07-23, and tracks Vite 8's plugin API.

**Alternatives considered**: none — this is the canonical, Vite-team-
maintained plugin for React; no credible alternative exists for this exact
purpose (contrast `@vitejs/plugin-react-swc`, a legitimate but different
choice trading some correctness edge cases for build speed — not adopted
here since this slice has no build-speed problem to justify the trade).

**Source**: [@vitejs/plugin-react on npm](https://www.npmjs.com/package/@vitejs/plugin-react)

**Enforcement**: `apps/web/package.json` devDependency:
`"@vitejs/plugin-react": "6.0.4"`, registered as the first plugin in
`vite.config.ts`.

### @types/react / @types/react-dom **(New this pass — were assumed but never pinned)**

**Decision**: `@types/react@19.2.17`, `@types/react-dom@19.2.3`.

**Rationale**: TypeScript type declarations for React 19 — required since
`react`/`react-dom` themselves ship no bundled `.d.ts` files. The previous
pass's React rationale said "matching `@types/react`/`@types/react-dom`"
without pinning either; both are re-verified current stable directly
against the npm registry on 2026-07-23 and track the pinned
`react@19.2.8`/`react-dom@19.2.8` major.minor line (DefinitelyTyped
versions React types by React's own major.minor, not lockstep patch, which
is why `@types/react-dom`'s patch number differs from `react-dom`'s own).

**Alternatives considered**: none — these are the only maintained type
declarations for this React line.

**Source**: [@types/react on npm](https://www.npmjs.com/package/@types/react), [@types/react-dom on npm](https://www.npmjs.com/package/@types/react-dom)

**Enforcement**: `apps/web/package.json` devDependencies:
`"@types/react": "19.2.17"`, `"@types/react-dom": "19.2.3"`.

### TypeScript

**Decision**: `typescript@6.0.3` — **not** the newer `7.0.2`.

**Rationale**: TypeScript 7.0 reached GA on 2026-07-08 with a from-scratch
Go-native compiler, but its stable programmatic/compiler API is not
shipping until 7.1 — `typescript-eslint` explicitly closed a "support TS7"
request as not-planned for 7.0. Since this project's CI-enforced lint gate
depends on `typescript-eslint` working correctly, pinning the last fully
tool-compatible release (6.0.3) is the lower-risk choice for a foundation
being laid today. **Re-verified this pass (2026-07-23), with stronger
evidence than the previous pass cited**: `typescript-eslint@8.65.0`'s own
published `package.json#peerDependencies.typescript` range, fetched
directly from the npm registry, is `">=4.8.4 <6.1.0"` — this is not merely
"no stable API yet" but a hard, machine-enforced upper bound that
explicitly excludes `7.0.2` (and even `6.1.x`) outright; installing
`typescript@7.0.2` alongside the pinned `typescript-eslint@8.65.0` would
fail (or at minimum warn loudly under) any peer-dependency check. This is
the concrete mechanism behind the previously-cited "closed as not-planned"
GitHub issue, confirmed directly against the currently-published package
metadata rather than taken on the issue tracker's word alone. Pinning
`typescript@6.0.3` — squarely inside that range — is not an arbitrary
lag; it is the only choice that keeps the CI-enforced lint gate installable
at all under the pinned `typescript-eslint` version. This is explicitly not
an instance of calling a superseded version "current stable" (contrast the
Vite correction above, this pass's instruction): 6.0.3 is deliberately
*not* current, for a documented, machine-verifiable compatibility reason,
which is exactly the kind of exception that instruction carves out.

**Alternatives considered**: `typescript@7.0.2` (nominally "current") —
rejected for now specifically because of the `typescript-eslint`
incompatibility, which would either break the lint gate or force an
immediate compatibility-shim workaround on day one. Revisit once
`typescript-eslint` ships official TS7 support (its `peerDependencies`
range, re-checked at that time, is the concrete signal to watch for).

**Source**: [TypeScript 6.0 announcement](https://devblogs.microsoft.com/typescript/announcing-typescript-6-0/), [TypeScript 7.0 announcement](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/), npm registry `typescript-eslint` package metadata (`registry.npmjs.org/typescript-eslint`), queried directly 2026-07-23.

**Enforcement**: root and `apps/web/package.json`: exact devDependency pin
`"typescript": "6.0.3"`, referenced by `tsc --noEmit` and by
`typescript-eslint`'s parser. Tracked as a documented future upgrade (see
Part 1 Cross-Compatibility Flags below).

### TanStack Query

**Decision**: `@tanstack/react-query@5.101.4` **(Corrected this pass: was
pinned `5.101.2`)**.

**Rationale**: Current stable v5 release — the version line the technical
baseline already requires for Rust-derived server state. Re-checked
directly against the npm registry on 2026-07-23: `5.101.2` — pinned and
labeled "current stable" in the previous pass — had already been
superseded by two further patch releases (`5.101.4` is npm's
`dist-tags.latest`). Same correction class as Vite above: no compatibility
reason motivates staying behind, so the pin moves to the actual current
patch rather than keeping a now-inaccurate "current stable" label on a
version that had drifted behind the registry.

**Source**: [@tanstack/react-query versions](https://www.npmjs.com/package/@tanstack/react-query?activeTab=versions), npm registry (`registry.npmjs.org/@tanstack/react-query`), queried directly 2026-07-23.

**Enforcement**: `apps/web/package.json`: `"@tanstack/react-query": "5.101.4"`.

### Zustand

**Decision**: `zustand@5.0.14`.

**Rationale**: Current stable — the version already required for transient
UI/layout state per the technical baseline.

**Source**: [zustand on npm](https://www.npmjs.com/package/zustand)

**Enforcement**: `apps/web/package.json`: `"zustand": "5.0.14"`.

### ESLint core

**Decision**: `eslint@10.7.0`, flat config (`eslint.config.js`) as the sole
format.

**Rationale**: Flat config has been the ESLint standard since v9; by v10
(10.7.0, current stable) no legacy `.eslintrc.*` format should be
introduced, and ESLint 10.0.0 (shipped Feb 2026) removed the legacy
`eslintrc` system entirely, so flat config is not just preferred but
mandatory at this version.

**Alternatives considered**: Legacy `.eslintrc.json` config — impossible on
ESLint 10, not merely discouraged.

**Source**: [ESLint version support](https://eslint.org/version-support/), [ESLint v10.0.0 released](https://eslint.org/blog/2026/02/eslint-v10.0.0-released/)

**Enforcement**: `eslint.config.js` at repo/app root; `package.json`
devDeps: `"eslint": "10.7.0"`.

### eslint-config-prettier **(Corrected: was an undocumented placeholder)**

**Decision**: `eslint-config-prettier@10.1.8`.

**Rationale**: Imported last in `eslint.config.js` so it disables every
Prettier-conflicting stylistic rule from `typescript-eslint` and
`eslint-plugin-react-hooks`. This gives ESLint sole ownership of
correctness/logic rules and Prettier sole ownership of formatting — no
dual-ownership conflicts, directly satisfying the "ESLint and Prettier with
non-conflicting responsibilities" quality baseline. Re-verified as current
stable directly against the npm registry (`registry.npmjs.org`) on
2026-07-22.

**Alternatives considered**: `eslint-plugin-prettier` (running Prettier as a
lint rule) — rejected; slower and conflates two failure classes (style vs.
correctness) the project wants separately actionable in CI.

**Source**: [eslint-config-prettier on npm](https://www.npmjs.com/package/eslint-config-prettier), [eslint-config-prettier repo](https://github.com/prettier/eslint-config-prettier)

**Enforcement**: `apps/web/package.json` (and root, if ESLint runs
repo-wide) devDependency: `"eslint-config-prettier": "10.1.8"`, imported as
the last entry in the flat config array.

### typescript-eslint **(Corrected: was an undocumented placeholder)**

**Decision**: `typescript-eslint@8.65.0` — the unified package (it now
bundles what used to be the separate `@typescript-eslint/parser` and
`@typescript-eslint/eslint-plugin` packages behind one dependency and one
`tseslint.config(...)`/`defineConfig(...)` flat-config helper).

**Rationale**: `typescript-eslint` v8+ is required for ESLint 10
compatibility; 8.65.0 is the current stable release, re-verified directly
against the npm registry on 2026-07-22. Using the unified package (rather
than the legacy split packages) is the currently-documented installation
path and avoids two separately-versioned packages needing manual alignment.

**Alternatives considered**: The legacy split
`@typescript-eslint/parser` + `@typescript-eslint/eslint-plugin` pair —
still installable but superseded by the unified package; rejected as the
non-current path for a foundation being laid now.

**Source**: [typescript-eslint.io](https://typescript-eslint.io/packages/typescript-eslint/), [typescript-eslint on npm](https://www.npmjs.com/package/typescript-eslint)

**Enforcement**: `apps/web/package.json` devDependency:
`"typescript-eslint": "8.65.0"`, used via `tseslint.config(...)` in
`eslint.config.js`, parsing against the pinned `typescript@6.0.3`.

### eslint-plugin-react-hooks **(Corrected: was an undocumented placeholder)**

**Decision**: `eslint-plugin-react-hooks@7.1.1`.

**Rationale**: The official React-team-maintained lint plugin enforcing the
Rules of Hooks and exhaustive-deps — required so ESLint, not just code
review, catches Hooks misuse in the one frontend surface this slice
introduces. 7.1.1 is current stable, re-verified directly against the npm
registry on 2026-07-22, and supports React 19 and ESLint 10 flat config.

**Alternatives considered**: none — this is the canonical, React-team
plugin for this exact purpose; no credible alternative exists.

**Source**: [eslint-plugin-react-hooks on npm](https://www.npmjs.com/package/eslint-plugin-react-hooks), [react.dev/reference/eslint-plugin-react-hooks](https://react.dev/reference/eslint-plugin-react-hooks)

**Enforcement**: `apps/web/package.json` devDependency:
`"eslint-plugin-react-hooks": "7.1.1"`, registered as a plugin in
`eslint.config.js` with its recommended rule set enabled (not merely
installed and left unconfigured).

### Prettier

**Decision**: `prettier@3.9.6`.

**Rationale**: Current stable, owns 100% of formatting per the ESLint split
above.

**Alternatives considered**: dprint (faster, Rust-based) — rejected;
Prettier is the ecosystem-standard choice with the widest editor/plugin
support.

**Source**: [prettier on npm](https://www.npmjs.com/package/prettier)

**Enforcement**: `package.json` devDependency `"prettier": "3.9.6"`;
`prettier.config.js` at repo root; `.prettierignore` covering generated
contract files.

### Vitest / React Testing Library

**Decision**: `vitest@4.1.10`, `@testing-library/react@16.3.2`, `jsdom@29.1.1`
(new this pass — `vitest`'s DOM test environment, previously implied by
`vitest.config.ts`'s `environment: "jsdom"` but never itself a pinned
dependency), `@testing-library/dom@10.4.1` (new this pass —
`@testing-library/react`'s own required peer, previously left to float as
an unpinned transitive install).

**Rationale**: Vitest 5 exists only as a beta; `4.1.x` is the actively
patched stable production line — the correct pin for a foundation being
laid now. `@testing-library/react@16.3.2` is current stable and supports
React 19's `use()`/Suspense model. `jsdom` is not optional: Vitest has no
built-in DOM — `environment: "jsdom"` in `vitest.config.ts` (Frontend State
Matrix and accessibility tests, `plan.md` Testing Plan) requires `jsdom` to
be present as an explicit dependency, and leaving it unpinned means CI could
resolve a different `jsdom` version than a contributor's local install.
`@testing-library/dom` is `@testing-library/react`'s declared peer
dependency (the query/`fireEvent` engine `@testing-library/react` itself
wraps for React); pinning it exactly, rather than trusting whatever
`@testing-library/react@16.3.2`'s own `peerDependencies` range resolves to
at install time, keeps the query behavior contributors see locally
identical to CI's. Both versions re-verified current stable directly
against the npm registry on 2026-07-23.

**Alternatives considered**: `vitest@5.0.0-beta.x` — rejected, pre-release.
Jest — out of scope; Vite-native Vitest is the natural fit given the pinned
Vite 8 toolchain and avoids a second transform pipeline. `happy-dom` (a
faster jsdom alternative) — rejected: `jsdom` has broader API-surface
fidelity (relevant to `jest-axe`'s DOM-inspection needs) and no measured
test-suite runtime problem exists in this slice's small test count to
justify the fidelity trade-off.

**Source**: [vitest versions on npm](https://www.npmjs.com/package/vitest?activeTab=versions), [vitest releases](https://github.com/vitest-dev/vitest/blob/main/docs/releases.md), [jsdom on npm](https://www.npmjs.com/package/jsdom), [@testing-library/dom on npm](https://www.npmjs.com/package/@testing-library/dom)

**Enforcement**: `apps/web/package.json` devDependencies:
`"vitest": "4.1.10"`, `"@testing-library/react": "16.3.2"`,
`"jsdom": "29.1.1"`, `"@testing-library/dom": "10.4.1"`; `vitest.config.ts`
sets `test.environment = "jsdom"`.

### Accessibility test tooling — jest-axe **(Corrected: was an undocumented
placeholder)**

**Decision**: `jest-axe@10.0.0`, used under Vitest via its Jest-compatible
`expect.extend` API (Vitest's `expect` is designed to accept the same
custom-matcher shape Jest uses), not `vitest-axe`.

**Rationale**: `jest-axe` 10.0.0 is current stable (verified directly
against the npm registry, published roughly a year before this revision)
and actively maintained. `vitest-axe` — the nominally Vitest-native
alternative — was checked and found effectively unmaintained: its tagged
`latest` is `0.1.0`, published four years ago, with only a `1.0.0-pre`
pre-release from January 2025 since. `jest-axe`'s matcher (`toHaveNoViolations`)
has no Jest-runtime dependency itself — it operates purely on an `axe-core`
results object and registers via the test framework's own `expect.extend`,
which Vitest supports identically to Jest — so the "jest" in the name is
historical, not a functional constraint. This is the safer, more current
pin despite the naming.

**Alternatives considered**: `vitest-axe` — rejected, unmaintained/pre-release
per the version check above. Wiring `axe-core` directly with a hand-rolled
matcher — rejected as unnecessary reimplementation of what `jest-axe`
already provides.

**Source**: [jest-axe on npm](https://www.npmjs.com/package/jest-axe), [vitest-axe on npm](https://www.npmjs.com/package/vitest-axe)

**Enforcement**: `apps/web/package.json` devDependency: `"jest-axe": "10.0.0"`
(plus `axe-core` as its own transitive dependency, not pinned separately);
registered once in the Vitest setup file via
`expect.extend(await import("jest-axe").then((m) => m.toHaveNoViolations))`
or the package's documented equivalent import shape.

### WebdriverIO **(Corrected: aligned the full package family, was
partially placeholder)**

**Decision**: `webdriverio@9.30.0`, `@wdio/cli@9.30.0`,
`@wdio/local-runner@9.30.0`, `@wdio/mocha-framework@9.30.0` (Mocha as the
test-framework adapter — no BDD/Cucumber requirement exists in this slice),
`@wdio/spec-reporter@9.29.1`, using WebdriverIO's built-in automatic
browser/driver management.

**Rationale**: `docs/PRD.md` PLT-008 normatively mandates WebdriverIO for
browser E2E — not a choice being reconsidered here, only version-pinned.
Every package in the family was re-checked directly against the npm
registry on 2026-07-22 (not assumed from the previous pass): `webdriverio`,
`@wdio/cli`, `@wdio/local-runner`, and `@wdio/mocha-framework` are all
currently at `9.30.0`; `@wdio/spec-reporter` currently trails at `9.29.1` —
this is expected and accepted, since WDIO's sub-packages are published
independently and do not always release in lockstep; each is pinned to its
own current-stable version rather than forcing an artificial uniform number
across packages that do not share a release cadence. Its built-in automatic
driver management downloads and caches the matching browser driver per
configured capability automatically, working uniformly on Linux and Windows
CI runners with no separate Selenium-standalone or manual
chromedriver/geckodriver install step — directly fitting the "Linux
primary, Windows technical tests" CI matrix (PLT-005 restricted) with one
configuration. This is also why `just check` can now include `test-e2e`
unconditionally (see `plan.md` Command Facade — correction 7): there is no
environment-dependent driver-provisioning step left to make optional.

**Alternatives considered**: Manually managed `chromedriver`/`geckodriver`
binaries per OS — rejected as unneeded overhead now that WDIO's automatic
manager handles cross-platform resolution natively. Selenium Grid/standalone
— rejected, unneeded infrastructure for a local-only project's CI. Cucumber
framework adapter (`@wdio/cucumber-framework`) — rejected for this slice;
Mocha is sufficient for the one minimal browser journey and avoids
introducing Gherkin feature-file tooling with no current requirement for it.

**Source**: [@wdio/cli on npm](https://www.npmjs.com/package/@wdio/cli), [webdriverio on npm](https://www.npmjs.com/package/webdriverio), [@wdio/local-runner on npm](https://www.npmjs.com/package/@wdio/local-runner), [@wdio/mocha-framework on npm](https://www.npmjs.com/package/@wdio/mocha-framework), [@wdio/spec-reporter on npm](https://www.npmjs.com/package/@wdio/spec-reporter), [WebdriverIO capabilities docs](https://webdriver.io/docs/capabilities/)

**Enforcement**: `tests/e2e/package.json`: `"webdriverio": "9.30.0"`,
`"@wdio/cli": "9.30.0"`, `"@wdio/local-runner": "9.30.0"`,
`"@wdio/mocha-framework": "9.30.0"`, `"@wdio/spec-reporter": "9.29.1"`;
`wdio.conf.ts` sets capabilities without pinning explicit driver binaries
and configures the spec reporter.

### just (command runner)

**Decision**: `just 1.57.0`.

**Rationale**: Current stable release; already the repository's established
command facade per project instructions — this is a version pin, not a
tool reconsideration.

**Source**: [just releases](https://github.com/casey/just/releases), [just manual](https://just.systems/man/en/)

**Enforcement**: Documented contributor install method (official install
script or `cargo install just --version 1.57.0 --locked`) pinned to
`1.57.0`; CI installs the same pinned version; a `just --version` guard
documented for contributors.

### GitHub Actions **(Corrected: exact commit SHAs, were placeholders —
this pass adds the still-missing `actions/setup-node` SHA and closes two
literal `<exact-sha>` placeholders left in `plan.md`)**

**Decision**: Linux runner `ubuntu-24.04`; Windows runner `windows-2022`
(explicit labels, not `-latest`);
`actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1` (the exact
commit the `v7.0.1` tag — and, as of this writing, the floating `v7`
tag — resolves to, verified directly against the GitHub REST API on
2026-07-22, with `# v7.0.1` as an inline comment for human readability);
`actions/setup-node@820762786026740c76f36085b0efc47a31fe5020` (the exact
commit the `v7.0.0` tag — and the floating `v7` tag — resolves to, verified
directly against the GitHub REST API on 2026-07-23, with `# v7.0.0` as an
inline comment) with `node-version-file: .nvmrc` — **new this pass**: the
previous draft's Node.js section referenced "CI's `actions/setup-node` step
(or equivalent)" without ever resolving it to a pinned SHA alongside the
other two actions, an inconsistency this pass closes so all three GitHub
Actions dependencies are treated identically;
`dtolnay/rust-toolchain@2c7215f132e9ebf062739d9130488b56d53c060c` (the
latest commit on that repository's `master` branch as of 2026-07-16,
likewise verified directly against the GitHub REST API, with
`# 2026-07-16` as an inline comment), with `toolchain: "1.97.1"` passed
explicitly.

**Rationale**: GitHub actively migrates `-latest` labels underneath users —
`ubuntu-latest` has already moved to 24.04, and `windows-latest`/
`windows-2025` are being migrated to bundle Visual Studio 2026 as of June
2026, exactly the silent, unpinned drift this plan's "no floating or
unbounded foundational dependency versions" requirement exists to prevent.
`windows-2022` is GitHub's own recommended label for a stable, non-moving
VS2022-based target (maintained three more years); `windows-2025` is a
documented future upgrade once its VS2026 image stabilizes.
`actions-rs/toolchain` has been unmaintained since October 2023;
`dtolnay/rust-toolchain` is the confirmed, actively maintained community
replacement — pinned by commit SHA rather than a mutable tag to close the
supply-chain risk of a tag being silently repointed. A previous draft of
this document named `actions/checkout@v7` and `dtolnay/rust-toolchain@<sha>`
without resolving either to a concrete SHA — that placeholder is what this
correction closes; both are now resolved to a specific, dated commit.

**Alternatives considered**: `ubuntu-latest`/`windows-latest` — rejected
per the explicit no-floating-labels requirement and GitHub's active 2026
image churn. `windows-2025` — rejected for now specifically because of its
June 2026 VS2026 auto-migration, reintroducing `-latest`-style drift;
acceptable as a documented future upgrade once that image stabilizes.
`actions-rs/toolchain` — rejected, unmaintained since 2023.

**Source**: [GitHub Actions image migration changelog](https://github.blog/changelog/2026-05-14-github-actions-upcoming-image-migrations/), [dtolnay/rust-toolchain](https://github.com/dtolnay/rust-toolchain), [actions/checkout releases](https://github.com/actions/checkout/releases), [actions/setup-node releases](https://github.com/actions/setup-node/releases), GitHub REST API (`api.github.com/repos/actions/checkout/tags`, `api.github.com/repos/actions/setup-node/tags`, `api.github.com/repos/dtolnay/rust-toolchain/commits/master`), queried directly on 2026-07-22 (checkout, rust-toolchain) and 2026-07-23 (setup-node).

**Enforcement**: `.github/workflows/*.yml`: `runs-on: ubuntu-24.04` /
`runs-on: windows-2022`; `uses:
actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`; `uses:
actions/setup-node@820762786026740c76f36085b0efc47a31fe5020 # v7.0.0` with
`node-version-file: .nvmrc`; `uses:
dtolnay/rust-toolchain@2c7215f132e9ebf062739d9130488b56d53c060c # 2026-07-16`
with `toolchain: "1.97.1"`, `components: "rustfmt, clippy"`; a
Dependabot/Renovate config to keep SHA pins current under human review
rather than silently floating. `plan.md`'s CI section previously carried
two literal `<exact-sha>` placeholders instead of these real values —
corrected there this pass to match exactly what is pinned here.

### Cross-Compatibility Flags

- **Rust 1.97.1 + Edition 2024**: no known incompatibility; 1.97.1 is well
  above the ≥1.85 floor and specifically fixes a miscompilation, making it
  the safer pin over 1.97.0.
- **Rust 1.97.1 + sqlx 0.9.0 / tokio 1.53.1 / axum 0.8.9 / tower-http
  0.7.0 / uuid 1.24.0 / blake3 1.8.5 / tokio-tungstenite 0.30.0 / reqwest
  0.13.4 / base64 0.22.1 / serde 1.0.229 / serde_json 1.0.151 / time
  0.3.54** (the last five newly pinned this pass): no known incompatibility;
  all track MSRVs comfortably below 1.97.1.
- **Vite 8.1.5 + @vitejs/plugin-react 6.0.4 + React 19.2.8 + @types/react
  19.2.17 / @types/react-dom 19.2.3** (newly pinned this pass): no known
  incompatibility; `@vitejs/plugin-react` tracks Vite 8's plugin API and
  React 19's JSX runtime, and both `@types/*` packages track React 19.
- **TypeScript 6.0.3 vs 7.0.x**: the one deliberate version-selection flag,
  not a hard incompatibility — TypeScript 7.0 is GA but `typescript-eslint`
  has no stable-API support until 7.1, so 6.0.3 is pinned instead. Track
  `typescript-eslint`'s TS7 support and revisit.
- **specta 2.0**: flagged as unsuitable for pinning (still RC after ~3
  years) — a maintenance-risk exclusion, not a compatibility conflict with
  this stack; `ts-rs` was chosen instead.
- **vitest-axe**: flagged as unsuitable for pinning (stale `0.1.0`, no
  actively maintained release) — `jest-axe` was chosen instead despite its
  name, since it has no Jest-runtime coupling.

---

## Part 2 — Security Threat Model and Bootstrap/Token-Transport Design

Scope: the Rust local HTTP/WebSocket service and its handoff with the
React/Vite frontend, both launched by the one contributor command (FR-001).
Resolves the security-sensitive bootstrap/token-transport design FR-002
through FR-006 and FR-022 reference, and defines the supported threat
boundary. Grounded in live research against official docs and real
2025–2026 incident data.

### Supported Threat Boundary

Converge's local service **protects against**:

- **(A) Hostile web content reaching the local service.** Any page loaded
  in the contributor's regular browser (a malicious site, a compromised ad,
  a phishing page) attempting to read from, write to, or open a WebSocket
  to the loopback service, whether via direct fetch, DNS rebinding, or
  ambient browser credentials (cookies).
- **(B) Unauthenticated local network peers.** Any local process or network
  peer that can reach `127.0.0.1` but does not possess the current
  ephemeral token.

Converge's local service does **not** protect against, and this plan makes
no such claim:

- **(C) A different process running as the same OS user account**, with
  filesystem or process-memory access. Such a process can already inspect
  environment variables or attach a debugger to the running service — the
  same trust boundary as the OS account itself.
- **(D) A different OS user with elevated (root/Administrator) privileges**,
  or **(E) physical access to the machine.**

This boundary is consistent with Constitution Principle III's non-waivable
protections (secret leakage, data corruption, trusted-directory escape),
none of which require defending against a same-user process, and matches
the plan's explicit instruction not to claim protection from an
already-compromised OS account unless a normative source requires it — none
of the reviewed normative sources (Constitution, PRD, MVP, spec.md) do. It
is also the same line drawn by every comparable loopback-bound dev tool
surveyed below.

### Threat-by-Threat Analysis

1. **Localhost DNS rebinding / hostile local web origins.** A page at
   `evil.example` can cause the browser to resolve that name to
   `127.0.0.1` after an initial same-origin-policy check passes, then issue
   requests that land on the local service with a `Host` value the server
   would otherwise trust. Not theoretical: a December 2025 disclosure in
   the Model Context Protocol TypeScript SDK allowed malicious websites to
   reach localhost MCP servers this exact way, because those servers
   validated only "is this socket local," no `Host` check at all ([GitHub
   Blog](https://github.blog/security/application-security/dns-rebinding-attacks-explained-the-lookup-is-coming-from-inside-the-house/)).
   **Mitigation**: the frontend always addresses the service as the literal
   IPv4 loopback address `127.0.0.1:<port>` — never `localhost` or `[::1]`
   — so no DNS resolution occurs in the legitimate path at all, and FR-004
   requires the server to reject any `Host` header that is not an exact
   match. CORS (FR-005) is irrelevant to this specific vector on its own —
   it blocks the attacker page from *reading* the response, not from
   *sending* the request — so `Host` validation is what actually stops the
   rebound request from being processed. This applies identically to
   Vite's own dev-bootstrap endpoint (see Bootstrap and Token-Transport
   Design below) — it is exactly as sensitive as the Rust service's health
   path, since it is what hands the token to the frontend in the first
   place, and MUST carry the same exact-match `Host` check.

2. **Token leakage** (URLs, logs, process args, env inheritance, browser
   storage, history, referrers, crash diagnostics). The token is never
   placed in a browser-navigable URL, never set as a cookie, never passed
   as a CLI argument, and held only in an in-memory JS variable in the
   frontend — never `localStorage`/`sessionStorage`. It travels only as an
   `Authorization: Bearer` header (HTTP) or a `Sec-WebSocket-Protocol`
   value (WS), neither of which appears in default access-log line formats
   or browser history the way a query string would. FR-017 requires
   redaction of the token in any logging introduced; tracing/log
   middleware must specifically redact the `Authorization` and
   `Sec-WebSocket-Protocol` header values (not only generically named
   "credential" fields), and no panic/crash path may print them. The
   WebSocket subprotocol negotiation redesign below (correction 1)
   specifically ensures the token-bearing subprotocol value is never
   echoed back in a response header either — only the non-secret
   `converge.v1` value is.

3. **Cookies and ports.** Corrected statement: per RFC 6265, cookies are
   scoped by **domain**, never by **port**, as a general rule that applies
   everywhere a browser sets or sends cookies — this is not a localhost
   quirk or a localhost-specific exception to an otherwise port-aware
   default; browsers have never treated port as part of a cookie's scope,
   on any host. `127.0.0.1`/`localhost` are simply an easy place to observe
   this general rule in practice, because it is common to run several
   unrelated local services on different ports of the same loopback address
   ([Chrome community thread](https://support.google.com/chrome/thread/44513790/);
   long-standing [Mozilla bug](https://bugzilla.mozilla.org/show_bug.cgi?id=469287)
   discussing the same domain-not-port scoping applied to `127.0.0.1`). A
   cookie set by the service on `127.0.0.1` would be sent by the browser to
   *any* other local service on `127.0.0.1`, regardless of port, and any
   other local listener could set a colliding cookie for the same host —
   exactly as it could for any two different services sharing one domain
   name on the public internet. **Mitigation**: no cookie is used for
   authentication anywhere in this design; the token travels only as an
   explicit header/subprotocol value scoped to the exact request the
   frontend constructs, never ambiently attached by the browser.

4. **WebSocket cross-site hijacking (CSWSH).** The classic vulnerability is
   a WebSocket server authenticating via ambient credentials (cookies),
   letting a malicious page open a connection using credentials the browser
   attaches automatically. Not using cookies (#3) removes the
   ambient-credential attack surface entirely — a malicious page has no way
   to obtain the in-memory token, which lives only in the legitimate
   frontend origin's JS heap, protected by the Same-Origin Policy. FR-022's
   explicit `Origin` allowlist check is retained as defense-in-depth on top
   of removing ambient credentials.

5. **Host and Origin spoofing.** A direct browser connection sets `Host`
   and `Origin` accurately and cannot override them via page script — a
   hard browser-platform guarantee. A non-browser local HTTP/WS client
   (curl, a local script, a malicious local process) *can* set either
   header to any value, which is exactly why `Host`/`Origin` checks are a
   complete defense only against **browser-mediated** attacks (#1, #4) and
   not against an arbitrary local process — the token (FR-003/FR-022) is
   the actual control there, bounded by threat-boundary item C.
   **Mitigation**: exact-string comparison against the single expected
   value for both headers, never prefix/suffix/substring matching (which
   would be bypassable, e.g. `127.0.0.1.attacker.example`).

6. **Replay and token lifetime.** The token is a bearer credential, valid
   for the lifetime of the running service process, generated fresh in
   memory at each start, and never persisted to SQLite or any durable
   store. Restarting the service invalidates the previous token (spec.md
   User Story 1, Acceptance Scenario 4). No additional expiry timer is
   introduced in this slice — the token's validity window is exactly the
   process's own lifetime, the simplest correct answer for a per-run
   credential, consistent with `docs/MVP.md` §12's rule against inventing
   arbitrary numeric budgets.

7. **Random-port discovery.** The OS assigns the port by binding to port
   `0` and reading back the OS-selected ephemeral port
   (`TcpListener::bind("127.0.0.1:0")` then `.local_addr()?.port()` — a
   standard, portable primitive on Linux and Windows). This avoids any
   fixed, predictable port a hostile local listener could occupy in
   advance. This applies to **both** processes in this design (corrected —
   see Bootstrap and Token-Transport Design below): Vite's dev server also
   binds to `127.0.0.1` on an OS/Vite-selected port (not necessarily a
   fixed default), and the Rust service binds to its own independently
   OS-selected port. How each side learns the other's actual port is
   resolved in Bootstrap Design below.

8. **Concurrent Converge instances.** Each instance independently binds to
   port `0` and generates its own independent token; the OS guarantees
   distinct ports for concurrently held listening sockets. With the
   IPC-based bootstrap design selected below, each instance's supervising
   process passes its own port/token/Origin only to its own directly
   spawned frontend and backend child processes — there is no shared file
   or shared discovery channel between instances, so there is no
   token/port-confusion surface between two contributors (or two
   checkouts) running concurrently.

9. **Stale bootstrap artifacts.** Resolved by design choice rather than
   mitigated: because the selected bootstrap mechanism (below) never writes
   the port or token to disk, no bootstrap file can go stale, be misread by
   a new instance, or be recovered later from disk by another local
   process. A crashed run leaves nothing behind to clean up or distrust.

10. **Local users and processes outside the supported threat boundary.**
    See "Supported Threat Boundary" above (items C–E). Nothing in this
    design claims protection against another process running as the same
    OS user.

### Bootstrap and Token-Transport Design

**Prior art surveyed**:

- **Jupyter Notebook/Lab**: generates a token and logs a full `?token=...`
  URL to the terminal, exchanged once for a session cookie ([Jupyter
  security docs](https://jupyter-notebook.readthedocs.io/en/6.5.2/security.html)).
  Confirms the URL-token pattern is real-world-deployed but explicitly
  treated as a one-shot bootstrap to be exchanged away, not a durable
  transport — and it still ends in a cookie, which threat #3 shows is
  unreliable across localhost ports for this design.
- **code-server** (VS Code in-browser): a random password written to a
  config file the user copies into a login form; after authenticating, a
  hashed password is stored in a session cookie with auth-attempt
  rate-limiting ([code-server docs](https://coder.com/docs/code-server/guide)).
  Confirms "credential in a restricted-permission local file, exchanged
  once" as a legitimate, shipped pattern.
- **Ollama**: ships with **no authentication at all** by design, targeting
  `localhost`-only exposure ([Ollama auth
  docs](https://docs.ollama.com/api/authentication)). This produced real,
  current-dated harm: a joint SentinelLABS/Censys investigation found
  roughly 175,000 Ollama servers publicly reachable due to
  `OLLAMA_HOST=0.0.0.0` misconfiguration, and CVE-2026-7482 allowed
  unauthenticated process-memory disclosure through the unauthenticated API
  ([CSO Online](https://www.csoonline.com/article/4168584/)). The strongest
  available argument, from live 2026 incident data, for why this slice's
  mandatory token/Host/Origin/CORS design — not "loopback binding alone is
  safe" — is the correct baseline.
- **Browser WebSocket API header limitation**: confirmed — the JavaScript
  `WebSocket` constructor accepts only a URL and an optional subprotocol
  array; it cannot attach a custom header to the opening handshake
  ([websocket.org reference](https://websocket.org/reference/headers/)).
  This is what forces the WebSocket token-transport decision below.

**Alternatives compared**:

| Alternative | Assessment |
| --- | --- |
| Jupyter-style one-time query-token URL + session cookie | Rejected: directly violates "no token in ordinary URLs or logs" (browser history, `Referer`); the follow-up cookie reintroduces the port-sharing problem (threat #3). |
| URL fragment handoff (`#token=...`, stripped via `history.replaceState`) | Defensible — fragments are never sent to a server/proxy/`Referer`. Rejected because it requires the launcher to control the browser's initial navigation, more machinery than needed, and leaves a residual browser-history entry containing the token. |
| Discovery file on disk (owner-only permissions, atomic write, staleness detection) | A legitimate pattern (code-server's config-file password is close prior art) — rejected here as strictly inferior to the selected option: it reintroduces a disk artifact requiring staleness handling (threat #9) and POSIX-vs-Windows-ACL permission code paths a disk-free design avoids entirely. |
| Environment variable passed directly to browser JS | Infeasible — browsers do not expose the launching process's environment to page JavaScript; no portable way to inject an env var into an already-installed, user-launched browser. |
| Environment variable passed to the Vite dev-server's own Node.js process at spawn time, for **both** directions of the handoff | **Rejected as originally designed** — see the Bootstrap Ordering Correction below: this direction alone cannot work, because Vite must be running (and its port known) *before* the Rust service can be configured with the correct allowed Origin, which means Vite cannot simultaneously be receiving Rust's port/token via its *own* spawn-time environment (it is already running by then). |
| Environment variable at spawn time for supervisor→Rust (Origin), plus a private Node.js IPC channel (`child_process.fork()`, `child.send()`/`process.on('message')`) for supervisor→already-running-Vite (port/token) | **Selected.** See below. |
| Cookie-based session after initial handshake | Rejected per threats #3 and #4 (port-sharing and CSWSH ambient-credential risk). |
| Vite dev-server WebSocket/HTTP proxying of all backend traffic (frontend never talks to the Rust port directly) | Rejected — see "Direct access vs. Vite proxying" below (correction 2, third bullet). |

#### Bootstrap Ordering Correction (correction 2)

The previous draft of this design started the Rust service first, then
handed its port/token to Vite via Vite's own process environment at spawn
time. That ordering cannot actually produce a correct CORS/Origin allowlist:
FR-005 and FR-022 require the Rust service to allowlist **the frontend's
real origin**, but Vite's dev server does not necessarily bind to a fixed,
predictable port — like the Rust service, it should bind to `127.0.0.1` and
let its actual port be determined (Vite's default port-conflict behavior
tries successive ports if the preferred one is taken), so the "real origin"
is not knowable until Vite itself is listening. Starting Rust first, before
Vite's real port is known, would force either a hardcoded/assumed Vite port
(fragile — breaks the moment the default port is occupied) or an
overly-broad Origin allowlist (a security regression). The corrected
sequence resolves this by starting Vite first:

1. **Supervisor spawns the Vite dev server first**, using Node's
   `child_process.fork()` (not `spawn()`) so a private, disk-free,
   process-tree-scoped IPC channel exists between the supervisor and the
   Vite child (`child.send(...)` / `process.on("message", ...)` inside
   `vite.config.ts`'s Node-side code). The fork script configures Vite's
   `server.host` to the literal `127.0.0.1` (never `0.0.0.0`, never
   `localhost`) and lets Vite pick its own listening port.
2. Once Vite's own HTTP server is listening, the fork script reads the
   actual bound port back (Vite's programmatic API exposes this via the
   dev server's underlying `httpServer.address()` once `listen()`
   resolves) and reports it to the supervisor over the same IPC channel
   (`process.send({ type: "vite-ready", port })`).
3. **The supervisor now knows the exact frontend origin**
   (`http://127.0.0.1:<vite-port>`) and spawns the Rust service, passing
   that value as `CONVERGE_ALLOWED_ORIGIN` in the Rust child process's own
   environment at spawn time (this direction is unchanged from the
   original design and remains simple env-at-spawn, because the Rust
   process is now spawned *after* the value it needs already exists).
4. The Rust service reads `CONVERGE_ALLOWED_ORIGIN` at startup and uses it
   as the single value in both the HTTP CORS allowlist (FR-005) and the
   WebSocket Origin allowlist (FR-022) — there is exactly one allowed
   origin per run, never a wildcard or a broader match. It then binds to
   `127.0.0.1:0`, generates its ephemeral token, and reports its own
   chosen port and token back to the supervisor over its own stdout pipe
   (private to the supervisor, never a world-readable log file) —
   unchanged from the original design.
5. **The supervisor forwards the Rust port and token to the
   already-running Vite child process** over the same IPC channel opened
   in step 1 (`viteChild.send({ type: "backend-ready", port, token })`).
   Vite's dev-server middleware (registered once, at startup, before any
   request arrives) receives this message and caches the port/token in an
   in-memory variable inside the Vite Node process — still never written
   to disk. The frontend's same-origin dev-only bootstrap fetch (unchanged
   from the original design) is served from this cached value.
6. If the backend-ready message has not yet arrived when the frontend
   requests the bootstrap endpoint (a real possibility, since Rust startup
   is not instantaneous), the middleware responds with a `503 Service
   Unavailable` and the frontend's bootstrap fetch retries with a short
   backoff — this is the Initial Loading state already defined in
   spec.md's Frontend State Matrix, not a new state.

This closes the original design gap (Vite could not simultaneously be a
spawn-time-env *recipient* while being spawned *before* the value it needed
existed) using one mechanism family (process-tree-private IPC) rather than
two different disk-free mechanisms for the two handoff directions, and
still satisfies every property the original design claimed: no disk
artifact, no shared file between concurrent instances (each supervisor's
IPC channel is private to its own child processes), identical mechanism
shape on Linux and Windows (Node's `child_process` IPC is portable).

**Vite bootstrap endpoint hardening (correction 2, bullet 3)**: because this
endpoint is what hands the token to the frontend, it is exactly as
security-sensitive as the Rust service's own endpoints and MUST NOT be
treated as an implicitly-trusted implementation detail of the dev server.
Concretely:

- The middleware validates the incoming request's `Host` header against
  the exact `127.0.0.1:<vite-port>` value before responding — the same
  exact-match discipline as threat #1, applied to Vite's own listener, not
  only to Rust's.
- Vite's dev server is never configured with `server.cors = true` (an
  open/reflective CORS grant) — if any cross-origin capability is enabled
  on the dev server for other reasons, the bootstrap endpoint specifically
  is excluded from it or checked independently.
- The frontend page reaches this endpoint by same-origin `fetch()` only
  (it is served by the same Vite origin the page itself was loaded from),
  so no CORS grant is actually needed for the legitimate path — matching
  the "closed by default" posture applied everywhere else in this design.
- The bootstrap endpoint's binding to `127.0.0.1` specifically (never
  `0.0.0.0`) is what removes DNS-rebinding exposure the same way it does
  for the Rust service — combined with the exact `Host` check above, a
  rebound `evil.example → 127.0.0.1` request still fails the `Host` check
  and never reaches the token.

#### Direct access vs. Vite proxying (correction 2, bullet 4)

The original set of artifacts contained a real, unresolved contradiction:
`research.md`'s own Requirement Mapping and `contracts/websocket-proof.md`'s
Validation Approach referred to "the Vite dev-server's WebSocket proxy
path," implying the frontend connects to Vite's own WebSocket endpoint and
Vite forwards traffic to the Rust service — while FR-004/FR-005/FR-022's
explicit `Host`/`Origin`/CORS/allowlist design only makes sense if the
browser talks to the Rust service **directly**, cross-origin from Vite's
origin (CORS and an Origin allowlist are meaningless if all traffic to the
backend is same-origin-relayed through Vite; there would be nothing for
those checks to allow or deny).

**Decision: direct HTTP/WS access.** The frontend, once it holds the Rust
service's port and token (via the bootstrap endpoint above), calls
`http://127.0.0.1:<rust-port>/api/health` and opens
`ws://127.0.0.1:<rust-port>/api/ws` **directly** — not through any Vite
proxy. This is the only choice consistent with FR-005's CORS requirement
and FR-022's Origin-allowlist requirement, both of which are dead code
under same-origin proxying. Every reference to a "Vite WS proxy path" is
removed from `contracts/websocket-proof.md`, `data-model.md`, `plan.md`,
and `quickstart.md` in this revision; nothing in the current design routes
backend traffic through Vite.

**Rationale for direct access over proxying, stated explicitly**: proxying
would need to exist specifically to avoid a cross-origin browser call, but
this design already needs CORS and an Origin allowlist to exist regardless
(both are explicit Constitution/PRD/spec requirements — FR-005, FR-022),
so proxying would add a second network hop and an extra configuration
surface (keeping the proxy's forwarded headers, especially
`Sec-WebSocket-Protocol`, byte-for-byte intact) purely to make an already-
required security control unreachable. Direct access is both the simpler
implementation and the one that actually exercises FR-005/FR-022 as
written.

**Token transport per request** **(Corrected: WebSocket subprotocol
negotiation — correction 1)**:

- **HTTP**: `Authorization: Bearer <token>` (FR-003, FR-017). Rejected: a
  query parameter (logged by virtually every default HTTP access-log
  format, visible in history/`Referer`); an ad hoc custom header
  (functionally similar, but `Authorization` is the one header name generic
  logging/redaction tooling already treats as sensitive by convention).
- **WebSocket**: since the browser `WebSocket` constructor cannot set
  arbitrary headers, the token is presented via the `Sec-WebSocket-Protocol`
  field of the upgrade request. The client's `WebSocket` constructor is
  called with **two** offered subprotocol values, not one:

  ```js
  new WebSocket(url, ["converge.v1", `converge.token.${token}`]);
  ```

  - `converge.v1` is a fixed, non-secret protocol-name value, always
    offered.
  - `converge.token.<base64url-token>` carries the current run's token
    (the one browser-settable value that can carry it, via the
    constructor's `protocols` argument).
  - The server inspects **all** offered subprotocol values during the
    upgrade itself (before any `101` response), validates the
    token-bearing one against the current run's token together with
    `Host` and the explicit server-side `Origin` allowlist, and refuses
    the upgrade (a non-`101` response) on any failure — satisfying
    FR-022's "before accepting any connection" literally: no
    unauthenticated socket is ever fully established.
  - On success, the server's `Sec-WebSocket-Protocol` **response** header
    selects **only** `converge.v1` — the non-secret value the client also
    offered. Per RFC 6455 §11.3.4, a server's selected subprotocol must be
    one of the client's offered values; offering both values from the
    client side is what makes this valid negotiation, rather than the
    server being forced to either echo the secret value back (leaking it
    into the response headers a browser extension, proxy, or dev-tools
    plugin logging response headers could capture) or omit the header
    entirely (which some intermediary tooling treats as an anomalous
    handshake).
  - The token-bearing subprotocol value is **never** echoed in the
    response and **never** logged — only the boolean fact that a token
    subprotocol was present and valid/invalid is logged (unchanged
    redaction rule from threat #2, now stated explicitly for this exact
    mechanism).
  - Trade-off, unchanged from the previous draft: `Sec-WebSocket-Protocol`
    is repurposed from its intended protocol-negotiation meaning, which
    can confuse debugging tools or intermediary proxies assuming its
    values are protocol names — acceptable for a local-only, direct
    connection (see "Direct access vs. Vite proxying" above — there is no
    intermediary proxy in this design to be confused).

**Requirement mapping**: loopback-only binding and OS-selected random port
(FR-002), high-entropy ephemeral token generated fresh per run (FR-003), no
persistent token storage anywhere — not on disk, not in SQLite (SEC-005
guardrail) — explicit HTTP `Host`/`Origin` validation (FR-004), closed HTTP
CORS (FR-005), denial observability (FR-006), the authenticated health path
(FR-007), and WebSocket token/`Host`/explicit-`Origin`-allowlist validation
before connection acceptance (FR-022) are all satisfied by exact-match
server-side checks independent of which transport carries the token.

**Launcher shutdown and orphan-process behavior (new — correction 7;
mechanism replaced this pass — correction 4)**: the supervisor
(`scripts/dev.mjs`) registers handlers for `SIGINT`/`SIGTERM` (and the
equivalent Windows console-control events via Node's normal cross-platform
signal handling) for the **cooperative** shutdown path:

1. Send a termination signal to the Rust service child first (it holds no
   client connections that need a graceful drain in this slice — no
   terminal/session state exists yet).
2. Send a termination signal to the Vite child.
3. Wait, bounded by a short timeout, for both children to exit; a child
   that does not exit within the timeout is force-killed (`SIGKILL`
   equivalent) rather than left running.
4. Exit the supervisor process itself only after both children have been
   confirmed exited or force-killed — never before, so `just dev`
   returning control to the shell is a reliable signal that no Converge
   process remains.

**The previous draft's orphan-process test asserted the wrong thing
(correction 4, this pass).** It checked that "a *subsequent* `just dev`
invocation still succeeds" after `SIGKILL`-ing the supervisor — a proxy
signal, not a lifecycle guarantee: since the OS assigns a fresh port every
run, a leaked, still-running prior Rust/Vite child would not necessarily
block a new instance from starting at all, so that test could pass even in
a world where orphaned processes silently accumulate. The actual guarantee
FR-001/FR-018 need is that killing the supervisor **abruptly** — bypassing
every line of its own cleanup code, including the cooperative path above,
since `SIGKILL`/`TerminateProcess` cannot be caught or handled by the
receiving process at all — still results in **both specific child
processes terminating**, not merely in a later `just dev` happening to
work. That requires a mechanism that does not depend on the supervisor
executing any code at the moment it dies, which the cooperative path above
structurally cannot provide (it only runs when the supervisor is signaled
by a catchable signal and gets to run its handler).

**Corrected mechanism: OS-level pipe/channel teardown, not cooperative
signaling.** Both children detect the supervisor's death via a channel
whose closure is enforced by the operating system itself when the
supervisor's process handle/file descriptors are torn down at exit — for
*any* reason, including `SIGKILL` — not by the supervisor executing any
handler. This is a stronger guarantee than a catchable signal can ever
provide, and it is available in an equivalent form on both Linux and
Windows without a platform-specific native binding on either side:

1. **Vite child** (already `fork()`ed, per the corrected Bootstrap
   Ordering above, specifically to get a private IPC channel): Node's
   `child_process.fork()` IPC channel is backed by an OS pipe (POSIX) or
   named pipe (Windows) between the two processes. When the supervisor
   process exits — cooperatively or via `SIGKILL`/`TerminateProcess` — the
   OS unconditionally closes every file descriptor/handle that process
   held open, including its end of that pipe, *as part of process
   teardown itself*, with no cooperation required from the exiting
   process. The Vite child has a `process.on("disconnect", ...)` handler
   registered from startup (not only in the cooperative-shutdown path)
   that fires the instant Node observes that pipe closure, and immediately
   closes its own dev-server listener and exits. This is a standard,
   documented Node.js behavior (the `'disconnect'` event on a forked
   child), not a bespoke mechanism.
2. **Rust service child** (`spawn()`ed, not `fork()`ed — it is not a Node
   process, so it gets no automatic IPC channel): the supervisor spawns it
   with its `stdin` connected to an OS pipe whose write end only the
   supervisor holds (Node's default `stdio: ["pipe", ...]` behavior for a
   spawned child already provides this — no extra configuration beyond
   *not* setting `stdin` to `"inherit"` or `"ignore"`). `service` runs a
   background `tokio` task, started at startup, that awaits a read on
   `tokio::io::stdin()`. Exactly like the Vite case, when the supervisor's
   process exits for any reason, the OS closes its end of that pipe as
   part of unconditional process teardown; the Rust side's read then
   returns `Ok(0)` (EOF) — not an error, but the standard "the writer is
   gone" signal every OS-level pipe gives on both Linux and Windows. The
   background task treats EOF as "the supervisor is gone" and triggers the
   exact same shutdown path `service` uses for `SIGTERM` (closing the
   listener, letting in-flight work finish within a bounded timeout, then
   exiting).
3. Both mechanisms rest on the same underlying property — **OS-level
   handle/pipe teardown on process exit is unconditional and requires no
   cooperation from the exiting process** — applied through the one
   channel each child already has for another purpose (Vite's IPC channel
   from the bootstrap design; the Rust child's `stdin` pipe, which the
   supervisor holds open specifically for this purpose and never writes to
   under normal operation). No `libc::prctl(PR_SET_PDEATHSIG, ...)` (Linux-
   only) and no Windows Job Object (`JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE`,
   Windows-only) binding is used — either would need `unsafe` FFI in a
   Converge-owned crate (`service`), which correction 5's uniform
   `#![forbid(unsafe_code)]` (Constraints, `plan.md`) now rules out, and
   each would need a *different* platform-specific implementation besides.
   The pipe/IPC-closure approach is the one mechanism family that is
   already portable across Linux and Windows through Node's own
   cross-platform `child_process` abstraction and Rust's own
   cross-platform async I/O, with no native binding on either side.

This is validated by two integration-test classes (Testing Plan,
`plan.md`), the second corrected this pass:

- **Launcher shutdown test** (cooperative path): send the supervisor
  `SIGINT`/`SIGTERM`, assert both children's specific process IDs (captured
  before the signal) no longer exist within the bounded timeout.
- **Orphan-process test — corrected this pass (correction 4)**: `SIGKILL`
  the supervisor itself, bypassing every cleanup handler it registered —
  the cooperative path above never runs — and assert the **same two
  specific child PIDs are confirmed exited** within the bounded timeout,
  via a portable liveness probe (POSIX: `process.kill(pid, 0)` raising
  `ESRCH` once the process is gone; Windows: the equivalent
  process-existence check Node's `child_process`/a small helper exposes).
  This directly exercises the pipe/IPC-closure mechanism above rather than
  inferring success from a later `just dev` run succeeding — the previous
  draft's test — which is removed as a stand-in for this direct assertion
  (a subsequent `just dev` succeeding remains true as a *consequence*, but
  is no longer what the test relies on to prove the guarantee).

**Validation approach**: automated integration tests assert (a) a request
with a missing/stale/wrong token, mismatched `Host`, mismatched `Origin`, or
disallowed CORS origin is denied for HTTP, and a handshake with a
missing/stale/wrong token, mismatched `Host`, or non-allowlisted `Origin`
never receives a `101` for WebSocket; (b) restarting the service invalidates
the previous token; (c) two concurrently started instances each
authenticate only against their own token/port pair; (d) no test run,
captured log, or process output ever contains the token in a URL-shaped
string, nor the token-bearing WebSocket subprotocol value in any response
header or log line (a redaction/negative-grep check tied to FR-017); (e)
the WebSocket server selects only `converge.v1` in its response subprotocol,
never the token-bearing value, exercised as a contract test on the
handshake response itself; (f) the launcher shutdown and orphan-process
behaviors above.

### Residual Risk Statement

The design does not, and does not claim to, protect a user whose OS account
is already compromised (threat boundary items C–E). Within the same OS
account, the process environment and process memory of both the Rust
service and the Vite dev server are readable by other processes running as
that user. This is accepted as consistent with the local single-user threat
model every loopback-bound dev tool in this class operates under — Jupyter,
code-server, and Ollama all draw functionally the same line — and is not a
protection any normative source (Constitution, PRD, MVP) requires Slice 0 to
provide.
