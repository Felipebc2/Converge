# Traceability Matrix: Slice 0 — Foundation

**New artifact this revision (correction 12).** Cross-references every
Functional Requirement (FR-001–FR-024) and Success Criterion (SC-001–SC-011)
in `spec.md` against the Acceptance Criteria (AC-033–AC-044) and PRD
requirement IDs `spec.md` already cites, plus this plan's design artifacts.
No new FR/SC/AC identifier is introduced or renumbered here — this document
only cross-references identifiers that already exist in `spec.md` and
`docs/PRD.md`/`docs/MVP.md`.

## Functional Requirements → PRD IDs → AC → Design Artifacts

| FR | PRD ID(s) | AC | Constitution | Design artifact |
| --- | --- | --- | --- | --- |
| FR-001 | PLT-001 (restricted), PLT-004 | AC-040 | — | `research.md` Bootstrap Ordering; `quickstart.md` Launch |
| FR-002 | SEC-003 | AC-033 | — | `research.md` threat #7 |
| FR-003 | SEC-001 | AC-033 | — | `contracts/http-health.md` Request |
| FR-004 | SEC-003 | AC-034 | — | `contracts/http-health.md`, `contracts/websocket-proof.md` Validation Order |
| FR-005 | SEC-003 | AC-034 | — | `contracts/http-health.md` Request; `research.md` "Direct access vs. Vite proxying" |
| FR-006 | — | AC-034, AC-039 | — | `contracts/http-health.md`, `contracts/websocket-proof.md` Validation Order |
| FR-007 | — (MVP §8 exit gate) | AC-033 | — | `contracts/http-health.md` |
| FR-008 | SEC-001 | AC-035 | — | `plan.md` Project Structure (`apps/web/src/lib/api/`) |
| FR-009 | — | AC-041 | II | `plan.md` Testing Plan — architecture allowlist check |
| FR-010 | — | AC-041 | II | `plan.md` Testing Plan — architecture allowlist check |
| FR-011 | PLT-002 | AC-042 | — | `data-model.md` Migration Version |
| FR-012 | — (MVP §8 exit gate) | — | V | `data-model.md` Concurrency and Conflict Behavior |
| FR-013 | PLT-002 | AC-042 | V | `data-model.md` Rebuild procedure |
| FR-014 | — | AC-040, AC-044 | IX | `plan.md` Command Facade |
| FR-015 | PLT-005 (restricted), PLT-009 (restricted) | AC-044 | — | `plan.md` CI section |
| FR-016 | — | AC-043 | VI | `plan.md` Testing Plan |
| FR-017 | SEC-008 (restricted) | AC-038 | VIII | `research.md` threat #2; Bootstrap design redaction rules |
| FR-018 | — | AC-039 | — | `data-model.md` Rebuild procedure (rollback visibility) |
| FR-019 | — | AC-043 | IV | `contracts/http-health.md`, `contracts/websocket-proof.md` Wire Types |
| FR-020 | — | AC-043 | IV | `plan.md` Contracts and Transport — drift gate |
| FR-021 | — | AC-043 | IV | `contracts/*.md` Validation Approach |
| FR-022 | SEC-001, SEC-003 | AC-033, AC-034 | — | `contracts/websocket-proof.md` Handshake Validation |
| FR-023 | PLT-011 | — | VII | `plan.md` Frontend State Matrix — Implementation Notes |
| FR-024 | PLT-004 | — | II | `plan.md` Frontend State Matrix — Implementation Notes |

## Success Criteria → Functional Requirements → Validation

| SC | Related FR(s) | Validated by |
| --- | --- | --- |
| SC-001 | FR-001 | `quickstart.md` Launch |
| SC-002 | FR-003, FR-004, FR-005, FR-006 | `quickstart.md` Security Boundary; `plan.md` Testing Plan (HTTP integration) |
| SC-003 | FR-012 | `plan.md` Testing Plan (Idempotency/conflict) |
| SC-004 | FR-013 | `plan.md` Testing Plan (Transactional rebuild) |
| SC-005 | FR-009, FR-010 | `plan.md` Testing Plan (Architecture boundary) |
| SC-006 | FR-014, FR-015 | `plan.md` CI section |
| SC-007 | FR-018 | `plan.md` Command Facade (`just check`) |
| SC-008 | FR-019, FR-020 | `plan.md` Contracts and Transport — drift gate |
| SC-009 | FR-022 | `plan.md` Testing Plan (WebSocket handshake) |
| SC-010 | FR-023 | `plan.md` Testing Plan (Frontend state matrix, Accessibility) |
| SC-011 | FR-024 | `plan.md` Testing Plan (TanStack/Zustand separation) |

## Acceptance Criteria (AC-033–AC-044) → Functional Requirements

| AC | Text (docs/MVP.md, verbatim) | Satisfied by |
| --- | --- | --- |
| AC-033 | The local service accepts authenticated traffic only through loopback, a random port, and the current ephemeral token. | FR-002, FR-003, FR-007, FR-022 |
| AC-034 | Invalid `Host`, `Origin`, CORS, token, scope, and Project ID combinations are denied. | FR-004, FR-005, FR-006, FR-022 (scope/Project ID: not applicable this slice — see Non-Applicable IDs below) |
| AC-035 | The browser cannot invoke a generic shell or unrestricted filesystem operation. | FR-008 |
| AC-036 | Provider credentials are never written to SQLite, files, diagnostics, or logs and are not modified. | Not applicable this slice — vacuously satisfied (see Non-Applicable IDs below) |
| AC-037 | Original provider payloads remain disabled and normalized metadata does not expose prompts, outputs, or secrets. | Not applicable this slice — vacuously satisfied (see Non-Applicable IDs below) |
| AC-038 | Local diagnostics redact paths, tokens, credentials, and unauthorized sensitive payloads and are never sent automatically. | FR-017 |
| AC-039 | No known failure allows silent data loss, credential exposure, or escape from the trusted project boundary. | FR-006, FR-018 |
| AC-040 | The documented `just` command launches the frontend and authenticated local service together. | FR-001, FR-014 |
| AC-041 | Domain and Application compile and test without dependencies on transport, UI, database, filesystem, PTY, Tauri, or providers. | FR-009, FR-010 |
| AC-042 | SQLx migrations are forward-only, tested, and compatible with aggregate rebuild. | FR-011, FR-013 |
| AC-043 | Applicable Rust, frontend, contract, HTTP/WebSocket integration, security, and browser E2E tests pass. | FR-016, FR-019, FR-020, FR-021 |
| AC-044 | Complete MVP CI passes on Linux and build plus technical tests pass on Windows x86_64. | FR-014, FR-015 |

## PRD IDs referenced by this slice

| PRD ID | Status this slice | Where |
| --- | --- | --- |
| PLT-001 | Restricted (HTTP/WebSocket stands in for Tauri Commands/IPC for the whole MVP) | FR-001, spec.md Out of Scope |
| PLT-002 | Applies (SQLite via SQLx, forward-only migrations) | FR-011, FR-013 |
| PLT-004 | Applies (TanStack Query manages Rust-derived state; Zustand does not) | FR-001, FR-024 |
| PLT-005 | Restricted (Windows: build + applicable technical tests only, no E2E requirement in this slice's Windows CI job) | FR-015, `plan.md` CI section |
| PLT-008 | Applies, non-negotiable (WebdriverIO mandated for browser E2E) | `research.md` WebdriverIO section |
| PLT-009 | Restricted (tagged-release portion out of scope; CI-trigger portion applies) | FR-015 |
| PLT-011 | Applies (Graphite Signal / DESIGN.md compliance for the health surface) | FR-023 |
| SEC-001 | Applies (no generic shell/FS access; token-required protected paths) | FR-003, FR-008, FR-022 |
| SEC-002 | **Not applicable this slice** — see Non-Applicable IDs below | spec.md Assumptions |
| SEC-003 | Applies (loopback, random port, Host/Origin validation) | FR-002, FR-004, FR-005, FR-022 |
| SEC-004 | **Not applicable this slice** — vacuously satisfied, see below | spec.md Assumptions |
| SEC-005 | Applies as a guardrail (no persistent token storage) | `research.md` Requirement mapping |
| SEC-007 | Applies as a forward-looking guardrail (redaction principle noted; not yet exercised — trivial payload only) | `data-model.md` `events.payload` |
| SEC-008 | Restricted (redaction requirement applies; full diagnostics scope is later-slice) | FR-017 |
| ANL-008 | Applies as a forward-looking guardrail (same redaction principle as SEC-007, noted, not yet exercised) | `data-model.md` `events.payload` |

## Non-Applicable IDs — explicit justification

Per the Constitution's "report, don't silently omit" discipline, every AC
or PRD ID within the cited AC-033–AC-044 range or explicitly named in
spec.md that this slice does **not** satisfy through real behavior is
listed here with its reason, rather than being left off the matrix above
without explanation:

- **AC-036, AC-037** (provider credentials, original provider payloads):
  vacuously satisfied — no credential or provider-payload handling exists
  anywhere in this slice's code, so neither can be violated. spec.md's
  Assumptions section states this explicitly and warns it "MUST NOT be read
  as license to introduce such handling later without revisiting those
  requirements directly."
- **AC-034's "scope, and Project ID" clause**: not applicable — no
  project/trust concept or capability-scope model exists yet (Slice 1
  scope). This slice's AC-034 coverage is limited to the `Host`/`Origin`/
  CORS/token combinations FR-004/FR-005/FR-006/FR-022 actually implement.
- **SEC-002** (capabilities and scopes specific to the active project): not
  applicable — no project or trust concept exists yet; spec.md's Out of
  Scope section defers this to Slice 1.
- **SEC-004** (provider credential handling): vacuously satisfied, same
  reasoning as AC-036/AC-037 above — they cite the same underlying
  requirement.

No AC or PRD ID in this table is silently dropped: every one either maps to
a concrete FR above, or appears in this section with an explicit reason it
does not apply to Slice 0.
