# Quickstart: Slice 0 — Foundation

Validates spec.md's User Stories 1–5 end-to-end on a clean checkout. Exact
tool versions are pinned in `research.md`; this guide references them by
role, not by number, so it does not drift when `research.md` is amended.

**Revision note (this pass)**: the Launch section below now describes the
corrected supervisor sequence (Vite starts first; the Rust service starts
once Vite's real port is known — `research.md` Part 2, Bootstrap Ordering
Correction), and the Complete Gate section removes the previous
environment-dependent opt-out for browser E2E — `just check` now always
includes it (correction 7).

**Revision note (final consistency pass)**: the Security Boundary section
below now demonstrates an absent-`Origin` denial and a CORS preflight
denial explicitly (correction 1, this pass — `contracts/http-health.md`'s
CORS Preflight section); "orphan-process behavior" throughout this guide
now refers to the corrected, directly-verified child-lifecycle guarantee
(correction 4, this pass — `research.md`'s corrected mechanism), not the
previous draft's indirect "a later `just dev` still succeeds" check.

## Prerequisites

- The pinned Rust toolchain, installed via `rustup` — a `rust-toolchain.toml`
  at the repository root pins the exact version, so `rustup show` after
  cloning installs it automatically (`research.md` §Rust Toolchain).
- The pinned Node.js version, activated via the mechanism `research.md`
  §Node/pnpm records (e.g., an `.nvmrc`-aware version manager or Volta).
- pnpm via Corepack (`corepack enable`) — the version pinned in
  `package.json#packageManager` is then used automatically.
- `just`, installed per its documented method (`research.md` §CI/Tooling).

## One-Time Setup

```sh
just setup
```

Installs Rust and pnpm dependencies for the pinned toolchains above, and
applies every forward-only SQLx migration (`data-model.md`) to a fresh local
development SQLite database.

## Launch (User Story 1 & 5)

```sh
just dev
```

This is the one documented command FR-001/AC-040 require. It runs a small
supervisor (`scripts/dev.mjs`, documented in `research.md`'s corrected
Bootstrap Ordering and in `plan.md`) that, in order:

1. **Forks the Vite dev server first** (Node's `child_process.fork()`,
   which opens a private IPC channel to it), bound to `127.0.0.1` on a port
   Vite itself selects. The supervisor reads Vite's actual bound port back
   over that IPC channel — nothing is assumed or hardcoded.
2. Now knowing the frontend's real origin (`http://127.0.0.1:<vite-port>`),
   **starts the Rust local service**, passing that origin to it as
   `CONVERGE_ALLOWED_ORIGIN` in the service's own environment at spawn
   time. The service binds to `127.0.0.1` on a port the OS assigns (`:0`
   binding), generates a fresh high-entropy ephemeral token in memory, and
   reports both back to the supervisor over a private channel — never a
   file, never the terminal in a form that would be logged verbatim.
3. **Forwards the service's port and token to the already-running Vite
   process** over the same IPC channel opened in step 1. Vite's own
   dev-only bootstrap middleware caches this in memory — nothing touches
   disk at any point in this sequence.
4. Prints a redacted status line (port shown, token never shown) and the
   Vite dev server's URL.

Open the printed URL (`http://127.0.0.1:<vite-port>/`) in a browser. The
frontend's dev-only bootstrap step reads the port/token from Vite's
same-origin dev middleware (never a query string, never `localStorage`),
then calls the authenticated health path
(`contracts/http-health.md`) **directly** against the Rust service's own
origin — a genuine cross-origin request from the frontend's origin, which is
why CORS (FR-005) actually applies here; nothing is proxied through Vite
(`research.md` Part 2, "Direct access vs. Vite proxying"). The health/status
surface transitions **Initial loading → Ready** (spec.md's Frontend State
Matrix). The frontend also opens the authenticated WebSocket proof
(`contracts/websocket-proof.md`), offering both the `converge.v1` and
token-bearing subprotocol values, and receives the `hello` message with only
`converge.v1` echoed back in the handshake response.

**Expected outcome**: SC-001 — zero undocumented manual steps between clean
checkout and a successful authenticated health response.

## Validating the Security Boundary Manually (User Story 1 & 5)

With `just dev` running, from a separate terminal (replace `<port>` and
`<vite-port>` with the values `just dev` printed):

```sh
# Wrong Host -> 400 (contracts/http-health.md, check order 1)
curl -i -H "Host: example.com" http://127.0.0.1:<port>/api/health

# Correct Host, disallowed Origin -> 403 (check order 2)
curl -i -H "Origin: http://evil.example" http://127.0.0.1:<port>/api/health

# Correct Host, Origin absent entirely -> 403 (check order 2 — corrected
# this pass: Origin is REQUIRED on every request, not merely validated
# "when present"; an absent Origin is denied identically to a mismatched
# one, contracts/http-health.md)
curl -i http://127.0.0.1:<port>/api/health

# Correct Host and Origin, missing/wrong token -> 401 (check order 3)
curl -i -H "Origin: http://127.0.0.1:<vite-port>" \
     -H "Authorization: Bearer wrong-token" \
     http://127.0.0.1:<port>/api/health

# CORS preflight with a disallowed requested method -> 403, no CORS grant
# (new this pass — contracts/http-health.md's CORS Preflight section)
curl -i -X OPTIONS \
     -H "Origin: http://127.0.0.1:<vite-port>" \
     -H "Access-Control-Request-Method: POST" \
     -H "Access-Control-Request-Headers: authorization" \
     http://127.0.0.1:<port>/api/health
```

The real current token is intentionally not meant to be typed by hand in
the ordinary flow — by design, per `research.md`, it never appears
anywhere a Contributor would casually copy it from (no log line, no URL, no
file). The automated integration suite (`just test-integration`) is the
authoritative way to exercise a fully-authenticated request; this manual
section only demonstrates denial paths, not the successful case.

**Expected outcome**: SC-002 — every malformed combination above is denied,
including an absent `Origin` and a disallowed CORS preflight (both
corrected/added this pass); a correctly formed request (which the
automated suite constructs, not this manual guide) succeeds.

## Validating Persistence (User Story 3)

```sh
just test-integration
```

Exercises, against a fresh database: persisting one representative
Normalized Event, resubmitting it — sequentially and concurrently, both
with the identical payload (idempotent success) and with a different
payload under the same `event_id` (the typed conflict error, per
`data-model.md`'s Concurrency and Conflict Behavior) — discarding the
derived Aggregate, and rebuilding it transactionally from the ledger. See
`data-model.md` for the exact schema and rebuild procedure this proves.

**Expected outcome**: SC-003, SC-004 — no duplicate ledger row on an
identical resubmission, an explicit conflict on a differing one; an
equivalent Aggregate after rebuild; the ledger itself unchanged by the
rebuild, and unchanged by a rebuild that is deliberately interrupted
mid-fold (transactional rollback).

## Validating Architecture Boundaries (User Story 2)

```sh
just test-unit
```

Includes the mechanical Clean Architecture boundary check: Domain and
Application crates are compiled and tested in isolation and fail to build
if either gains a normal dependency outside its documented allowlist
(`plan.md` Project Structure — `domain`: `{thiserror}`; `application`:
`{domain, thiserror, uuid, blake3}`) (FR-009, FR-010).

**Expected outcome**: SC-005.

## Running the Complete Gate (User Story 4)

```sh
just check
```

Runs formatting verification, linting, Rust unit tests (including the
architecture boundary allowlist check), contract tests (including the
generated-TypeScript drift check and the WebSocket subprotocol-negotiation
assertion), integration tests (including launcher shutdown and
orphan-process behavior), frontend tests (including the Frontend State
Matrix and `jest-axe` accessibility assertions), **and the browser E2E
journey — unconditionally, with no opt-out** (corrected this revision:
WebdriverIO's automatic driver management means there is no
environment-dependent step to make optional), then builds — the same
recipes CI runs on every pull request and push to the default branch
(FR-015).

**Expected outcome**: SC-006, SC-007 — the gate passes on a clean change and
fails visibly on a deliberately broken one; nothing passes silently, and
nothing is silently skipped either.

## Summary: Spec Success Criteria Covered by This Guide

| Success Criterion | Validated by |
| --- | --- |
| SC-001 | `just setup` + `just dev` → health check succeeds |
| SC-002, SC-009 | Manual denial requests above + `just test-integration` |
| SC-003, SC-004 | `just test-integration` |
| SC-005 | `just test-unit` |
| SC-006, SC-007 | `just check` |
| SC-008 | `just test-contract` (contract-drift check, part of `just check`) |
| SC-010, SC-011 | `just test-frontend` (part of `just check`) |
