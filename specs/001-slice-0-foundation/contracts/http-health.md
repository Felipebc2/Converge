# Contract: Authenticated HTTP Health Path

**Governs**: FR-003, FR-004, FR-005, FR-006, FR-007, FR-019, FR-020, FR-021;
AC-033, AC-034; User Story 1.

**Revision note (this pass)**: the Rust wire types below are now specified
exactly, including their Serde attributes, so the generated TypeScript is
fully deterministic and the JSON casing shown here is the actual wire
format (correction 5) — the previous draft showed `checked_at` (Rust-style
snake_case) in the JSON example while leaving the Serde attribute that
would produce it unstated. This revision also removes the last reference to
Vite proxying — this endpoint is reached by **direct** cross-origin
HTTP request from the frontend's origin, never through a Vite proxy (see
`research.md` Part 2, "Direct access vs. Vite proxying").

**Revision note (final consistency pass)**: the CORS preflight (`OPTIONS`)
was previously collapsed into a single Request-table row alongside the
substantive `GET` request; the two are governed by different rules (no
bearer token on `OPTIONS`, different header set, a different response
shape) and are now specified separately (correction 1, this pass). The
`Origin` header is now stated as unconditionally required on `GET
/api/health` — the previous "when present" wording incorrectly implied
`Origin` was optional on the actual request; FR-004 requires validating
`Origin` "on every request," which an absent header cannot satisfy.

**Revision note (final blocking-consistency pass, point 1)**: two gaps
remained after the pass above: an absent (not merely wrong)
`Access-Control-Request-Method` on a preflight was left unhandled (CORS
Preflight, row 3 — now an explicit denial), and the actual `200` response
never stated its own required `Access-Control-Allow-Origin` header,
conflating "the preflight granted CORS" with "the real response is
readable by page script" — two genuinely separate Fetch-spec checks (Success
Response, below). Both are corrected here with dedicated contract/
integration test assertions (Validation Approach).

**Rust source of truth**: the request/response types below are Rust structs
(exact module path: `crates/contracts`, per `plan.md` Project Structure).
Generated TypeScript types MUST be produced deterministically from them
(FR-019) and a regenerate-and-diff check MUST fail CI on drift (FR-020).

## Endpoint

```text
GET /api/health
```

Loopback-only, per-run random port (FR-002), reached by a **direct** HTTP
request from the frontend's origin (`http://127.0.0.1:<vite-port>`) to the
service's own origin (`http://127.0.0.1:<rust-port>`) — a genuinely
cross-origin browser request, which is exactly why FR-005's CORS
enforcement applies here and is not dead code. No other verb is defined on
this path in this slice.

## Request

| Element | Requirement |
| --- | --- |
| `Host` header | MUST equal the loopback address and the service's bound port for this run (e.g., `127.0.0.1:<port>`). Any other value is denied. (FR-004) |
| `Origin` header — **REQUIRED on every request, corrected this pass** | MUST be present and MUST equal the single expected local frontend origin for this run (`CONVERGE_ALLOWED_ORIGIN`, `research.md` Part 2 Bootstrap Ordering) exactly. An absent `Origin` is denied identically to a mismatched one — FR-004 requires validating `Origin` "on every request," which an optional/skip-when-absent check would not satisfy. (FR-004, FR-005) |
| `Authorization` header | `Authorization: Bearer <ephemeral-token>`. MUST equal the current run's token exactly. Never present, and never required, on the `OPTIONS` preflight — see CORS Preflight below. (FR-003) |

Token transport mechanism (how the frontend obtains the token to place in
this header) is a security-sensitive bootstrap decision recorded in
`research.md`, not part of this contract — this contract only fixes the
*wire shape* (`Authorization: Bearer`), consistent with "no token in
ordinary URLs or logs."

## CORS Preflight (`OPTIONS`) **(New — correction 1, this pass)**

Browsers issue a CORS preflight `OPTIONS` request before any cross-origin
request that carries a non-"simple" header — `Authorization` always
triggers one ([Fetch §CORS-preflight
fetch](https://fetch.spec.whatwg.org/#cors-preflight-fetch)). Since the
frontend always sends `Authorization` on the real request, every legitimate
`GET /api/health` call is preceded by exactly one `OPTIONS` preflight to
this same path.

**`OPTIONS` carries no bearer token, by the CORS specification itself, not
by this service's choice.** The Fetch/CORS mechanism never attaches the
*following* request's own headers (including `Authorization`) to the
preflight — it only declares intent via `Access-Control-Request-Method`
and `Access-Control-Request-Headers`. The service MUST NOT require, and
MUST NOT treat the absence of, an `Authorization` header on `OPTIONS` as a
denial condition; only the checks below apply to a preflight.

Checked in this fixed order — the first failure wins and grants no CORS
headers at all, so a browser blocks the subsequent real request itself
before it ever reaches the server (the actual mechanism CORS enforcement
provides):

| Order | Condition | Response |
| --- | --- | --- |
| 1 | `Host` mismatch | `400 Bad Request`, `{"error": "bad_request"}`, no CORS grant |
| 2 | `Origin` absent, or present and not the single expected value | `403 Forbidden`, `{"error": "forbidden"}`, no CORS grant |
| 3 | `Access-Control-Request-Method` absent, or present and not exactly `GET` **(corrected this pass — an absent value is now an explicit denial, not merely "present and wrong")** | `403 Forbidden`, `{"error": "forbidden"}`, no CORS grant |
| 4 | `Access-Control-Request-Headers` does not include `authorization` (case-insensitive) | `403 Forbidden`, `{"error": "forbidden"}`, no CORS grant |
| — | All four checks pass | `204 No Content` with the exact grant below |

Valid-preflight response headers — an exact, closed grant, never a wildcard
and never a reflected value:

```text
Access-Control-Allow-Origin: <the single expected frontend origin>
Access-Control-Allow-Methods: GET
Access-Control-Allow-Headers: authorization
```

No `Access-Control-Allow-Credentials` header is ever sent — this design
uses no cookies (`research.md` threat #3), so there is no ambient
credential for that header to authorize. `Access-Control-Max-Age` MAY be
set to reduce duplicate preflights; its presence or absence changes no
security property above, only preflight frequency.

**Row 3 corrected — final blocking-consistency pass, point 1**: a genuine
CORS preflight always carries `Access-Control-Request-Method`, since that
header is what the Fetch specification uses to construct a preflight
request in the first place ([Fetch §CORS-preflight
fetch](https://fetch.spec.whatwg.org/#cors-preflight-fetch)) — a browser
never sends an `OPTIONS` request to this path without it. An `OPTIONS`
request missing that header is therefore not a browser-issued preflight at
all; the previous draft only denied a *present-but-wrong* value (e.g.
`POST`), leaving an *absent* value unhandled — a gap this correction closes
by denying both identically.

An invalid preflight (any of rows 1-4) receives **no** CORS grant — no
`Access-Control-Allow-Origin` header at all — which is what actually stops
the browser from sending the real request; the `400`/`403` status and
`ApiError` body exist for server-side observability and contract testing
(FR-006), since browsers discard the preflight response body and never
expose it to page script.

## Wire Types **(New — correction 5: exact Rust types, Serde tagging, ts-rs
behavior)**

```rust
// crates/contracts/src/health.rs

#[derive(serde::Serialize, serde::Deserialize, ts_rs::TS)]
#[serde(rename_all = "camelCase")]
#[ts(export)]
pub struct HealthResponse {
    pub status: HealthStatus,
    pub service: String,
    pub version: String,
    pub checked_at: String, // ISO 8601 UTC; -> "checkedAt" on the wire
}

#[derive(serde::Serialize, serde::Deserialize, ts_rs::TS)]
#[serde(rename_all = "lowercase")]
#[ts(export)]
pub enum HealthStatus {
    Healthy,
}

#[derive(serde::Serialize, serde::Deserialize, ts_rs::TS)]
#[serde(tag = "error", rename_all = "snake_case")]
#[ts(export)]
pub enum ApiError {
    BadRequest,
    Forbidden,
    Unauthorized,
}
```

- `#[serde(rename_all = "camelCase")]` on `HealthResponse` is what makes
  `checked_at` (the idiomatic Rust field name) serialize as `checkedAt` on
  the wire and in the generated TypeScript — `ts-rs 12.0.1`'s
  `serde-compat` feature (default-on, re-verified in `research.md`) reads
  this attribute and emits the TypeScript field as `checkedAt` too, so the
  Rust field name and the wire/TypeScript name are related by one
  documented, mechanical rule rather than two independently-maintained
  spellings.
- `status` is typed as the single-variant `HealthStatus` enum, not a bare
  `String`, specifically so a later slice extending it with a genuine
  degraded state (see below) is a compile-time-checked, ts-rs-regenerated
  change rather than an untyped string literal contributors could typo.
- `ApiError` is a tag-only enum (`#[serde(tag = "error", rename_all =
  "snake_case")]`, no variant carries data) — ts-rs generates a TypeScript
  discriminated union keyed by the `error` field, matching the three denial
  bodies in the table below exactly:
  `{"error":"bad_request"}` / `{"error":"forbidden"}` /
  `{"error":"unauthorized"}`.

## Success Response

`200 OK`, `Content-Type: application/json`,
`Access-Control-Allow-Origin: <the single expected frontend origin>`
**(header requirement made explicit this pass, point 1)**

```json
{
  "status": "healthy",
  "service": "converge-local",
  "version": "<compile-time semantic version of the running service>",
  "checkedAt": "<ISO 8601 UTC timestamp>"
}
```

**`Access-Control-Allow-Origin` on the actual `200` response, not only on
the preflight — corrected this pass.** A preflight granting CORS only
authorizes the browser to *send* the real request; the real request's own
response still needs its own `Access-Control-Allow-Origin` header, or the
browser blocks page script from reading the response body even though the
preflight succeeded and the request itself reached the server ([Fetch §HTTP
network or cache fetch](https://fetch.spec.whatwg.org/#http-network-or-cache-fetch),
the actual-response CORS check is a separate step from the preflight
check). The previous draft's Success Response section specified only
`Content-Type`, leaving this header unstated — every successful,
allowed-origin `GET /api/health` response now unconditionally carries the
exact, single, non-wildcard `Access-Control-Allow-Origin` value (never a
wildcard, never a reflected value, matching the preflight grant's own "exact
grant" rule above). No `Access-Control-Allow-Methods`/`-Headers` are
required on the actual response — those are preflight-only headers; the
actual response's own CORS gate is `Access-Control-Allow-Origin` alone
([Fetch §CORS protocol](https://fetch.spec.whatwg.org/#http-responses)).

- `status` is currently always the literal `"healthy"` when this response is
  returned at all — there is no degraded/partial state in Slice 0, since no
  dependency this slice introduces (SQLite) has a meaningful degraded mode
  short of being fully reachable or not. A later slice MAY extend
  `HealthStatus` with a new variant if it introduces a dependency with a
  genuine degraded state; because `HealthStatus` is a typed enum (above),
  that extension is a Rust-side, ts-rs-regenerated change, not a
  free-floating string.
- `version` and `checkedAt` exist so the frontend's Ready/Refreshing-Stale
  distinction (Frontend State Matrix) has something to compare against
  between polls, without requiring a second endpoint.

## Validation Order and Denial Responses

Applies to the substantive `GET /api/health` request only — the `OPTIONS`
preflight has its own, separately ordered checks (CORS Preflight above).
Checks run in this fixed order; the first failure wins and determines the
response — later checks are never reached once an earlier one fails, so a
request is never partially trusted:

| Order | Condition | Status | Body |
| --- | --- | --- | --- |
| 1 | `Host` mismatch | `400 Bad Request` | `{"error": "bad_request"}` |
| 2 | `Origin` absent, or present and not allowlisted **(corrected: absence is now an explicit denial condition, not merely "when present")** | `403 Forbidden` | `{"error": "forbidden"}` |
| 3 | Missing or invalid `Authorization` token | `401 Unauthorized` | `{"error": "unauthorized"}` |

Rationale: `Host` is checked first because a `Host` mismatch means the
request was not even addressed to this service instance (the DNS-rebinding
case research.md's threat model covers) — treated as a malformed target,
`400`. `Origin` is checked next as the browser-boundary/CORS decision
(`403`, "this origin is not permitted"). `Authorization` is checked last as
the actual authentication decision (`401`, "who are you") — checking it last
means a request that already failed the structural `Host`/`Origin` checks
never reaches token comparison, so no timing signal about token validity is
observable from an already-out-of-policy request. Every case is always a
denial (FR-006), never a partial/soft-allow, and no response body ever
echoes the invalid token, `Host`, or `Origin` value back (avoids reflecting
attacker-controlled input and avoids incidentally logging/echoing anything
sensitive). This three-way split gives tests (FR-016) a precise way to
assert *which* check failed, rather than a single undifferentiated denial.

Every denial MUST be observable server-side (FR-006, FR-018) — the service
logs the denial reason (not the token value) so a Contributor or CI can see
why a request was rejected; see `research.md` for the redaction rule that
keeps the token itself out of that log line.

## Validation Approach

- Contract test: request with a valid token/`Host`/`Origin` asserts the
  `200` shape above field-by-field, including exact `camelCase` field names
  (FR-021), **and asserts the exact `Access-Control-Allow-Origin` value is
  present on that `200` response (new this pass, point 1)** — not only on
  the preflight.
- Negative tests: each of the three failure rows above, tested individually
  and in combination per spec.md's edge cases (FR-016), asserting the exact
  `ApiError` discriminant returned.
- Preflight contract test **(New — correction 1, previous pass; extended
  this pass, point 1)**: a valid `OPTIONS` preflight (matching
  `Host`/`Origin`, `Access-Control-Request-Method: GET`,
  `Access-Control-Request-Headers: authorization`) asserts the exact
  three-header closed-CORS grant above and `204`; each of the four
  preflight failure rows, tested individually, asserts no
  `Access-Control-Allow-Origin` header is present in the response — **now
  including a dedicated case for row 3 with `Access-Control-Request-Method`
  omitted entirely (absent, not merely wrong), asserting the same `403`/
  no-grant outcome as an explicitly-wrong value** (new this pass, closing
  the gap row 3's correction above describes); a preflight test also
  asserts no `Authorization` header is required or inspected on `OPTIONS`.
- Drift test: CI regenerates the TypeScript client type from the Rust
  source type (including `HealthStatus` and `ApiError`) and diffs it
  against the committed generated file; any difference fails the build
  (FR-020).
