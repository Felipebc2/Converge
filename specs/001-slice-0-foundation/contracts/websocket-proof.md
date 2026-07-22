# Contract: Minimal Authenticated WebSocket Proof

**Governs**: FR-019 through FR-022; AC-033, AC-034; User Story 5; SC-009.

**Revision note (this pass)**: the subprotocol negotiation below was
under-specified in the previous draft (correction 1) — it named a
token-bearing subprotocol the client would send but never defined what, if
anything, the server selects in its response, which is not valid RFC 6455
negotiation on its own. This revision defines a concrete two-value client
offer and a non-secret server selection. It also removes the last reference
to "the Vite dev-server's WebSocket proxy path" (correction 2, bullet 4) —
this endpoint is reached by a **direct** WebSocket handshake from the
frontend's origin, never through a Vite proxy — and adds exact Rust wire
types with Serde tagging (correction 5).

**Revision note (final consistency pass)**: "Success: Minimal Proof
Exchange" below previously said "exactly two message shapes" while
enumerating three (`hello`, `ping`, `pong`) and while the Wire Types section
already correctly said "three" — a plain miscount, now corrected (correction
9, this pass).

**Rust source of truth**: as with the HTTP contract, the message shapes
below are Rust structs, TypeScript-generated deterministically (FR-019)
with CI failing on drift (FR-020).

This endpoint exists solely to prove the WebSocket boundary is bound by the
same authentication strength as HTTP, through the mechanisms that actually
apply to WebSocket (see `research.md`'s threat model — CORS does not apply
to WebSocket handshakes; the browser `WebSocket` API cannot set arbitrary
headers on the handshake). It carries no terminal, PTY, or provider
streaming payload (Out of Scope, spec.md).

## Endpoint

```text
GET /api/ws  (HTTP Upgrade to WebSocket)
```

Loopback-only, same per-run random port as HTTP (FR-002), reached by a
**direct** handshake from the frontend's origin to the service's own
origin — not proxied by Vite (`research.md` Part 2, "Direct access vs. Vite
proxying").

## Handshake Validation

Checked in this fixed order — the first failure refuses the upgrade and no
WebSocket frame is ever exchanged with an unauthenticated peer (FR-022,
SC-009); mirrors the HTTP contract's ordering discipline:

| Order | Condition | Handshake response |
| --- | --- | --- |
| 1 | `Host` mismatch | `400 Bad Request` (plain HTTP response; upgrade refused) |
| 2 | `Origin` absent from the explicit server-side allowlist | `403 Forbidden` (plain HTTP response; upgrade refused) |
| 3 | No offered `Sec-WebSocket-Protocol` value matches a valid token subprotocol (see below) | `401 Unauthorized` (plain HTTP response; upgrade refused) |

**CORS is not involved anywhere in this table.** WebSocket handshakes are
not subject to CORS preflight or CORS headers at all — `Origin` here is
enforced by an explicit server-side allowlist comparison (FR-022), not by
the CORS mechanism FR-005 requires for HTTP.

### Subprotocol negotiation and token transport **(Corrected — correction
1)**

The browser `WebSocket` constructor cannot set an `Authorization` header on
the handshake (confirmed in `research.md`). The token is instead carried as
one of **two** values the client offers via `Sec-WebSocket-Protocol` (the
constructor's `protocols` argument accepts an array):

```js
new WebSocket("ws://127.0.0.1:<port>/api/ws", [
  "converge.v1",
  `converge.token.${token}`,
]);
```

- `converge.v1` — a fixed, non-secret protocol-name value. Always offered.
  This is the value the server selects in its response (see below); it
  carries no token material and is safe to log, echo, or observe in
  dev-tools.
- `converge.token.<token>` — `<token>` is the current run's ephemeral
  token, encoded so every character is a valid HTTP/WebSocket subprotocol
  token character (no `+`, `/`, `=`, whitespace, or comma) — i.e.,
  base64url without padding. This constraint is carried back to the
  token-generation crate choice in `research.md`.

**Server-side validation, during the upgrade itself** (before any `101`
response — a deliberate rejection of the alternative "accept the socket,
then require a first authenticated message" pattern, which leaves a window
where an unauthenticated peer holds an open upgraded socket; FR-022's
"before accepting any connection" language is written to exclude that
pattern):

1. The server inspects the full list of offered `Sec-WebSocket-Protocol`
   values, locates the one matching the `converge.token.<...>` shape (if
   none does, handshake row 3 above applies), and compares its token
   material to the current run's token.
2. On success, the server's `Sec-WebSocket-Protocol` **response** header
   selects **only** `converge.v1`. Per RFC 6455 §11.3.4, the server's
   selected value must be one of the client's offered values — the client
   offering both `converge.v1` and the token-bearing value is exactly what
   makes this a valid negotiation instead of forcing the server to choose
   between echoing the secret back or omitting the response header
   entirely.
3. The token-bearing subprotocol value the client sent is **never** echoed
   back in the response, and is **never** logged (FR-017) — only the fact
   that a token subprotocol was present and valid/invalid is logged, never
   its value.

## Success: Minimal Proof Exchange

Once upgraded, exactly three message shapes exist in this slice —
**corrected this pass**: the previous draft said "two," undercounting
`hello`/`ping`/`pong` (three `WsMessage` variants, matching the Wire Types
section below exactly) — no other message type is defined, and none MUST be
added without revisiting this contract and the Out-of-Scope boundary in
spec.md:

1. **Server → Client, immediately after upgrade**:

   ```json
   { "type": "hello", "serverTime": "<ISO 8601 UTC>" }
   ```

2. **Client → Server (optional)**:

   ```json
   { "type": "ping" }
   ```

   **Server → Client (in response to `ping`)**:

   ```json
   { "type": "pong", "serverTime": "<ISO 8601 UTC>" }
   ```

Either side may close the connection normally at any point after `hello`.
There is no session state, no subscription, and no streaming loop — this is
a proof of the authenticated transport, not a feature.

## Wire Types **(New — correction 5: exact Rust types, Serde tagging, ts-rs
behavior)**

```rust
// crates/contracts/src/ws.rs

#[derive(serde::Serialize, serde::Deserialize, ts_rs::TS)]
#[serde(tag = "type", rename_all = "lowercase", rename_all_fields = "camelCase")]
#[ts(export)]
pub enum WsMessage {
    Hello { server_time: String },
    Ping,
    Pong { server_time: String },
}
```

- `#[serde(tag = "type", ...)]` makes this an internally-tagged enum: each
  variant serializes as a flat JSON object carrying a `"type"` discriminator
  field, exactly matching the three message shapes above — no separate
  wrapper object, no `content`/`data` nesting.
- `rename_all = "lowercase"` renders the variant names as the tag values:
  `Hello` → `"hello"`, `Ping` → `"ping"`, `Pong` → `"pong"`.
- `rename_all_fields = "camelCase"` (a distinct Serde attribute from
  `rename_all`, applied to the *fields inside* each variant rather than to
  the variant/tag names themselves) renders `server_time` as `serverTime`
  in the `Hello` and `Pong` variants' JSON — matching the documented wire
  shapes above exactly.
- `ts-rs 12.0.1`'s `serde-compat` feature (confirmed in `research.md`
  against `docs.rs/ts-rs/12.0.1`) supports `tag`, `rename_all`, and
  `rename_all_fields` together and generates a TypeScript discriminated
  union:

  ```ts
  export type WsMessage =
    | { type: "hello"; serverTime: string }
    | { type: "ping" }
    | { type: "pong"; serverTime: string };
  ```

  which is the type the frontend's WebSocket client narrows against on the
  `type` field — no separate hand-written `WsHello`/`WsPing`/`WsPong`
  TypeScript interfaces exist; there is exactly one generated union type,
  matching FR-019's "Rust types MUST be the authoritative source" for this
  contract precisely (a single Rust type, a single generated TypeScript
  type, no parallel hand-maintained shapes).

Denial responses on this endpoint are plain HTTP (see Handshake Validation
above) and reuse the HTTP contract's `ApiError` discriminated union
(`crates/contracts/src/health.rs`) — there is no separate WebSocket-specific
error type, since a refused handshake never becomes a WebSocket connection
in the first place; it is, structurally, still an HTTP response.

## Validation Approach

- Contract test: a handshake offering both `converge.v1` and a valid token
  subprotocol, with matching `Host` and an allowlisted `Origin`, completes
  the upgrade, receives `converge.v1` (and only `converge.v1`) as the
  response's `Sec-WebSocket-Protocol` value, and exchanges the
  `hello`/`ping`/`pong` shapes above exactly, asserted against the
  generated `WsMessage` union type (FR-021).
- Negative tests: each of the three handshake-validation rows, individually
  and in combination, asserted to never return `101` (FR-016, SC-009).
- Negotiation test: asserts the token-bearing subprotocol value never
  appears in the handshake response header or in any captured log line
  (FR-017, `research.md` validation approach).
- Drift test: shares the same regenerate-and-diff CI check as the HTTP
  contract (FR-020).
