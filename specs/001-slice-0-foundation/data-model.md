# Data Model: Slice 0 — Foundation

**Input**: `specs/001-slice-0-foundation/spec.md` §Key Entities, FR-011 through FR-013

**Governs**: the immutable normalized-event ledger and the rebuildable
aggregate proven by User Story 3 (FR-012, FR-013, AC-042, MVP §8 exit gate).

**Revision note (this pass)**: `event_id` generation moved from UUID v4 to
UUIDv7, a `payload_hash` column (BLAKE3) was added, concurrent-insert and
conflicting-duplicate `event_id` behavior are now specified explicitly, the
aggregate rebuild procedure is now specified as a single transaction, and
the foreign-key omission rationale below was corrected — the original
wording described an impossible cascade direction. See `research.md`'s new
"Event identity and integrity — UUIDv7 + BLAKE3" section for the full
rationale of the identity/hash decision.

**Revision note (final consistency pass)**: event identity was
under-specified — who generates `event_id` and how it survives a retry was
implicit, the "canonical representation" `payload_hash` is computed over was
undocumented (a real gap: two differently-serialized-but-equivalent JSON
payloads could hash differently and wrongly appear as a conflict), and the
idempotent-vs-conflicting classification compared `payload_hash` alone,
when a retry that changed `event_type` or `occurred_at` while coincidentally
preserving `payload_hash` would have been misclassified as a legitimate
idempotent resubmission. All three are corrected in "Event identity:
generation, preservation, and canonical comparison" and "Concurrency and
Conflict Behavior" below (correction 6, this pass).

This slice does not introduce any product-domain schema. The entities below
exist solely to prove the ledger/idempotency/rebuild mechanism with one
representative, technically-named event type. Real provider event shapes
(Claude/Codex sessions, analytics, quotas) belong to Slice 2 and are
explicitly out of scope here — see spec.md's Out of Scope section and FR-012.

## Entity: `events` (the immutable ledger)

Maps to spec.md's **Normalized Event** key entity.

| Column | Type | Constraints | Notes |
| --- | --- | --- | --- |
| `id` | `INTEGER` | `PRIMARY KEY AUTOINCREMENT` | Internal, monotonic ledger sequence. Defines the ledger's total temporal order; never reused. This — not `event_id` — remains the sole authoritative replay order for the rebuild procedure below. |
| `event_id` | `TEXT` | `NOT NULL`, `UNIQUE` | Producer-supplied idempotency key, a **UUIDv7** (RFC 9562) — see `research.md`. Generated once by the producer and preserved unchanged across any retry of the same logical occurrence — see "Event identity: generation, preservation, and canonical comparison" below for exactly who generates it and when. Resubmitting an event with the same `event_id` MUST NOT create a second row — this is the idempotency boundary FR-012 requires, including under concurrent resubmission (see Concurrency and Conflict Behavior below). UUIDv7 gives `event_id` a secondary, approximately-chronological ordering property (useful for debugging/audit) without displacing `id` as the ledger's authoritative order. |
| `event_type` | `TEXT` | `NOT NULL` | Discriminator. This slice defines exactly one representative value: `slice0.probe.recorded`. New event types are Slice 2+ scope. One of the three producer-controlled immutable fields a resubmission is compared against (below). |
| `payload` | `TEXT` (JSON) | `NOT NULL` | Normalized fields only. No raw provider payload, no prompts, no secrets — consistent with the redaction principle `SEC-007`/`ANL-008` will fully apply once provider ingestion exists; this slice keeps the payload shape trivial and inert (e.g., `{"note": "..."}`) to avoid pre-empting Slice 2's real schema. Always produced by exactly one canonical serialization call — see "Event identity" below. |
| `payload_hash` | `TEXT` | `NOT NULL` | Lowercase hex-encoded BLAKE3 digest (32 bytes → 64 hex characters) of the canonical serialized `payload` bytes (see "Event identity" below for what "canonical" means here), computed by the application layer at insert time. One of the three producer-controlled immutable fields a resubmission is compared against — see Concurrency and Conflict Behavior below. Never used as a security token or exposed outside this table. |
| `occurred_at` | `TEXT` (ISO 8601 UTC, RFC 3339) | `NOT NULL` | When the represented occurrence happened, as reported by the producer. Parsed and validated (not merely stored as an opaque string) by `application` using the `time` crate's parsing feature (`research.md`) before insertion, so a malformed timestamp is rejected at the use-case boundary rather than persisted. One of the three producer-controlled immutable fields a resubmission is compared against (below). |
| `recorded_at` | `TEXT` (ISO 8601 UTC, RFC 3339) | `NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%fZ', 'now'))` | When this row was appended to the ledger. Computed by SQLite itself via the column's `DEFAULT` expression at `INSERT` time — never supplied by application code and never updated afterward — so no Rust timestamp dependency is needed for this column specifically (contrast `occurred_at` above and the HTTP/WS `checkedAt`/`serverTime` fields in `contracts/`, which are generated in `adapters-http` using the `time` crate). |

**Invariants**:

- **Append-only.** No application code path issues `UPDATE` or `DELETE`
  against this table. As defense in depth beyond application discipline, the
  migration also creates `BEFORE UPDATE` and `BEFORE DELETE` triggers on
  `events` that raise `RAISE(ABORT, 'events ledger is immutable')`, so a
  mutation attempt fails at the database layer even if application code
  regresses. This makes immutability mechanically enforced, not just
  conventionally true, matching Constitution V.
- **No update path exists**, so there is no risk of the ledger silently
  drifting from what was originally recorded — a discarded/rebuilt aggregate
  (below) is the only thing ever recomputed.

### Event identity: generation, preservation, and canonical comparison **(New — correction 6, this pass)**

**Who generates `event_id`, and when.** `event_id` is exclusively
producer-generated (FR-012 already assumes this; this section makes it
explicit). "Producer" here means whatever caller invokes the application
layer's `RecordProbeEvent` use case — in this slice, that is exclusively
the integration test suite exercising the ledger mechanism, since no
HTTP/WebSocket contract exposes event ingestion yet (`contracts/` defines
only the health and WebSocket-proof endpoints). Real external producers
(Claude/Codex adapters) are Slice 2+ scope. Concretely:

- The producer generates a fresh UUIDv7 **once**, at the moment it first
  attempts to record a given logical occurrence (a single `RecordProbeEvent`
  call as the producer understands it).
- If that attempt fails to confirm success (a network error, a timeout, a
  process restart before the caller observed the result) and the producer
  retries what it considers **the same** logical occurrence, it MUST reuse
  the exact `event_id` value it generated the first time — never mint a new
  one for what it believes is a retry of the same attempt. This is what
  makes the retry observable as a retry at all: the ledger has no other way
  to recognize "this is the same occurrence again" than the `event_id` the
  producer chooses to repeat.
- `application`'s `RecordProbeEvent` use case and `adapters-persistence`'s
  repository implementation never generate, mutate, or substitute
  `event_id` — both accept whatever value the caller supplies and pass it
  through unchanged to the `INSERT`. Generation is a producer responsibility
  by design, not an accident of internal retry plumbing that could silently
  mint a fresh id and defeat the idempotency guarantee FR-012 requires.

**Canonical representation for duplicate comparison.** `payload` is never
accepted as raw, pre-serialized, producer-supplied JSON text at the
application boundary — no HTTP/WebSocket ingestion endpoint exists in this
slice to accept one, and none should be added without revisiting this
document. The one representative payload shape is a single Rust struct
(e.g. `struct ProbePayload { note: String }`), and `payload_hash` is always
computed over the bytes of exactly one canonical serialization path:
`serde_json::to_string(&payload_struct)`, called precisely once per
`RecordProbeEvent` invocation. `serde_json`'s struct serialization emits
fields in declaration order — a fixed property of the Rust struct
definition, not something a caller can vary — so re-invoking the use case
with logically identical input necessarily produces byte-identical
serialized output. This sidesteps the general "canonical JSON" problem
(key-ordering, whitespace, numeric-formatting normalization across
independently-produced JSON documents) entirely, because there is exactly
one code path that ever produces the bytes `payload_hash` is computed
from — canonical by construction, not by convention or by trusting the
producer to format consistently. A future slice that accepts raw,
externally-produced JSON as `payload` MUST introduce an explicit
canonicalization step before hashing (e.g. RFC 8785 JSON Canonicalization
Scheme) and revisit this section — the current design's simplicity depends
on `payload` always originating from one internal serialization call.

### Concurrency and Conflict Behavior **(Corrected — correction 6 and
correction 10, this pass: matching now spans `event_type`, `payload_hash`,
and `occurred_at` together, not `payload_hash` alone)**

`event_id` carries a `UNIQUE` constraint. Three distinct outcomes are
defined for an `INSERT` attempt, and the application MUST distinguish all
three rather than collapsing them into a single success/failure signal. A
resubmission is judged against **all three** producer-controlled immutable
fields — `event_type`, the canonical `payload_hash` (above), and
`occurred_at` — not `payload_hash` in isolation: a retry that reused
`event_id` but changed `event_type` or `occurred_at`, even while
`payload_hash` happened to coincide (e.g. two structurally different
payloads that both hash-collide, or more realistically two callers sharing
an `event_id` by mistake with different `occurred_at` values), MUST be
classified as a conflict, not silently accepted as idempotent:

1. **No existing row with this `event_id`.** The `INSERT` succeeds
   normally. This is the first-submission case.
2. **An existing row with this `event_id` AND identical `event_type`,
   `payload_hash`, AND `occurred_at`.** This is a legitimate idempotent
   resubmission (FR-012) — every producer-controlled immutable field
   matches exactly. The application catches the `UNIQUE`-constraint failure
   the second concurrent/sequential `INSERT` raises, re-`SELECT`s the
   existing row by `event_id`, confirms all three fields match, and returns
   the **same successful outcome** the caller would have seen on the
   original insert — no error surfaces, no duplicate row is created.
3. **An existing row with this `event_id` AND a difference in `event_type`,
   `payload_hash`, or `occurred_at` (any one or more).** This is an
   idempotency-key-reuse conflict — the caller reused an `event_id` for a
   genuinely different occurrence, which is a producer bug or a
   confused/malicious caller, not a legitimate retry. The application MUST
   surface this as a distinct, typed error:

   ```rust
   pub struct ConflictingFields {
       pub event_type: bool,
       pub payload_hash: bool,
       pub occurred_at: bool,
   }

   pub enum EventRepositoryError {
       IdempotencyConflict {
           event_id: String,
           conflicting: ConflictingFields,
       },
       // ...other variants
   }
   ```

   naming exactly which of the three fields differed (per the Rust quality
   baseline's "explicit typed errors at application and boundary layers,"
   and FR-018's "no known failure allows silent data loss" — a Contributor
   or CI reading this error can tell at a glance whether the mismatch was
   in `event_type`, the payload, `occurred_at`, or several at once, without
   re-fetching both rows to diff them by hand). It MUST NOT be silently
   treated as success (that would let a resubmission silently discard
   different content the caller believed it was recording) and MUST NOT
   silently create a second row (that would violate the `UNIQUE` constraint
   the idempotency guarantee depends on). No HTTP/WebSocket contract in this
   slice exposes event ingestion directly (see `contracts/` — only the
   health and WebSocket-proof endpoints exist), so this error is currently
   only observable through the `application` layer's own test suite and the
   integration tests described below; a future slice that exposes an
   ingestion endpoint MUST map this typed error to an explicit HTTP status
   (not reuse `200`) when it does.

**Concurrent-insert mechanics**: two concurrent `INSERT` attempts with the
same `event_id` race on the `UNIQUE` constraint at the SQLite layer —
exactly one succeeds at the database level; the other receives a
constraint-violation error (SQLite result code `SQLITE_CONSTRAINT_UNIQUE`,
surfaced by `sqlx` as `sqlx::Error::Database` with that code). The
"losing" side of the race does not treat this as a bare failure — it
re-`SELECT`s the row that won the race and applies outcome 2 or 3 above
based on whether `event_type`/`payload_hash`/`occurred_at` all match. This
means the *outcome* of a race is identical to the outcome of a sequential
resubmission; only which of the two concurrent callers happens to perform
the physical `INSERT` is non-deterministic, which is invisible to both
callers because both observe the same final row and the same classified
outcome. This is validated by FR-016's concurrency test requirement,
extended in this revision to cover outcome 3 under each of the three
individual field mismatches (a concurrent resubmission differing only in
`event_type`, only in `payload`, and only in `occurred_at`, each exercised
separately), not only a single "any conflict" case as the previous draft
tested.

## Entity: `aggregates` (disposable, rebuildable projection)

Maps to spec.md's **Aggregate** key entity.

| Column | Type | Constraints | Notes |
| --- | --- | --- | --- |
| `event_type` | `TEXT` | `PRIMARY KEY` | Natural key: one projection row per distinct `event_type` seen in the ledger. This slice's ledger only ever contains `slice0.probe.recorded`, so exactly one row exists in practice — but keying by `event_type` rather than a slice-specific literal keeps the mechanism itself generic and avoids baking a throwaway Slice 0 name into the schema shape. |
| `event_count` | `INTEGER` | `NOT NULL` | Count of ledger rows folded into this projection. |
| `last_event_id` | `TEXT` | `NOT NULL` | `event_id` of the most recent ledger row folded in (by ledger `id` order), for traceability back to the ledger. |
| `rebuilt_at` | `TEXT` (ISO 8601 UTC) | `NOT NULL` | When this projection row was last (re)computed. Changes on every rebuild even if the folded values don't. |

### Rebuild procedure — transactional semantics **(Corrected — correction
10)**

FR-013, AC-042, MVP §8 exit gate. The entire procedure runs inside **one**
SQLite transaction, opened with `BEGIN IMMEDIATE` (not the default
`BEGIN DEFERRED`):

1. `BEGIN IMMEDIATE TRANSACTION` — acquires SQLite's write lock at the
   start of the rebuild, before any statement executes. Using `IMMEDIATE`
   rather than `DEFERRED` here specifically avoids the classic SQLite
   failure mode where a transaction starts as a reader, later tries to
   upgrade to a writer, and loses that race to a different writer that
   started after it — `IMMEDIATE` acquires the write lock up front instead,
   so the rebuild cannot fail midway through purely due to a lock-upgrade
   conflict.
2. `DELETE FROM aggregates` — discards every current projection row, inside
   the transaction (not yet visible to other connections until commit).
3. `SELECT * FROM events ORDER BY id ASC` and fold every row into
   per-`event_type` projection state purely in application memory, in
   ledger insertion order (not client-supplied `occurred_at`, which is not
   guaranteed monotonic or unique across producers). Because the
   transaction holds SQLite's write lock for its entire duration, no
   concurrent `INSERT` into `events` can interleave with this read — a
   concurrent event-ingestion attempt blocks until the rebuild transaction
   commits or rolls back, then proceeds normally against the
   now-consistent post-rebuild state. This is what "transactional" means
   here concretely: the rebuild always observes a single consistent
   snapshot of `events`, never a partial one.
4. `INSERT INTO aggregates (...) VALUES (...)` once per distinct
   `event_type` folded, with the current timestamp — still inside the same
   transaction.
5. `COMMIT`. If any step 2–4 fails for any reason, the transaction is
   `ROLLBACK`ed instead: the previous `aggregates` contents remain exactly
   as they were before the rebuild attempt (SQLite's transaction atomicity
   guarantees this without any extra application-level snapshot/restore
   logic) — a failed rebuild is visible (FR-018) and never leaves
   `aggregates` empty or half-populated.

Because step 3 reads only from the immutable `events` table and steps 2/4
touch only `aggregates`, a rebuild can never mutate the ledger — the
property Success Criterion SC-004 measures — regardless of whether the
transaction commits or rolls back. Running the rebuild twice in a row
without new events yields an equivalent `event_count`/`last_event_id` (only
`rebuilt_at` changes), which is what "equivalent result" means for this
entity.

This is validated by a new **transactional-rebuild test** (Testing Plan,
`plan.md`): a deliberately injected failure partway through the fold step
(e.g. a poisoned in-memory fold triggering an application-level abort before
step 4 completes) MUST leave `aggregates` unchanged from its pre-rebuild
state, and a concurrent `events` insert attempted while a rebuild
transaction is open MUST observe blocking (not an interleaved partial read)
and succeed only after the rebuild transaction resolves.

## Entity: Migration Version

Maps to spec.md's **Migration Version** key entity. This is not a
hand-designed table — it is SQLx's own built-in migrations-tracking table
(created and managed by `sqlx migrate run`), which records each applied
migration's version number, description, checksum, and applied timestamp.
Documented here only to confirm the concept has a concrete implementation:

- Migrations are plain, versioned SQL files under `migrations/` (e.g.,
  `0001_create_events_and_aggregates.sql`), applied in filename order and
  never edited after being merged — forward-only per Constitution V and
  FR-011.
- SQLx refuses to apply a migration file whose checksum no longer matches
  what was previously recorded as applied, which is the mechanism that
  makes "run twice, or run against a database already at a newer version"
  (spec.md Edge Cases) a safe no-op or a visible failure rather than silent
  corruption.
- Tested from an empty database (FR-011): the test suite creates a fresh
  SQLite file, runs all migrations, and asserts the expected schema and
  triggers exist.

## Entity: Ephemeral Session Token — explicitly NOT part of this data model

Maps to spec.md's **Ephemeral Session Token** key entity, included here only
to state plainly what is excluded: the token is generated fresh in memory
when the local service starts, held only in the running process's
application state, and is never written to any SQLite table, migration, or
row. No schema exists for it, and none should ever be added without
revisiting Constitution VIII and FR-017 directly. Its generation and
transport are a runtime/security design decision recorded in `research.md`
and `plan.md`, not a data-model concern.

## Relationships

`aggregates.last_event_id` references `events.event_id` informationally
(traceability for debugging/audit), not as a SQL foreign key.

**Corrected rationale (correction 10)**: the previous draft claimed a
foreign key here "would let a `DELETE`/rebuild ordering bug on `aggregates`
cascade into the ledger" — this direction is not how foreign keys work and
was inaccurate. A foreign key from `aggregates.last_event_id` (the
*referencing*/child column) to `events.event_id` (the *referenced*/parent
column) can only ever cascade an action performed **on `events`** (the
parent) **onto `aggregates`** (the child) — for example, an `ON DELETE
CASCADE` clause would cascade a deletion of an `events` row into a deletion
of the referencing `aggregates` row. It is structurally impossible for a
`DELETE` issued against `aggregates` (the child) to cascade anything onto
`events` (the parent) — child-to-parent cascades do not exist in standard
foreign-key semantics, regardless of which `ON DELETE`/`ON UPDATE` clause is
chosen.

The actual reasons the foreign key is omitted are narrower and different
from the original claim:

1. **It would be structurally moot.** The `BEFORE DELETE` trigger on
   `events` already makes deleting an `events` row impossible at the
   database layer (see Invariants above), so an `ON DELETE` cascade clause
   on this foreign key could never actually fire in the first place — there
   is nothing for it to protect against that the trigger does not already
   prevent.
2. **It would add transactional ceremony for zero protective benefit.**
   Because `aggregates` is fully wiped and rebuilt on every run (the
   `DELETE FROM aggregates` step in the rebuild transaction above), a naive
   foreign key would need every `last_event_id` to continuously reference an
   existing `events.event_id` at all times, including the brief window
   between the `DELETE` and the corresponding `INSERT` within the same
   transaction. SQLite supports deferring foreign-key checks to
   transaction-commit time (`PRAGMA defer_foreign_keys = 1`, scoped to the
   current transaction) specifically for this kind of case, but adding that
   configuration exists only to accommodate a constraint that (per point 1)
   can never be violated anyway — pure ceremony, not protection.

The foreign key is omitted because it would be inert, not because omitting
it prevents some cascade risk — that specific characterization was wrong
and is corrected here.

## Validation Rules Summary

| Rule | Enforced by | Requirement |
| --- | --- | --- |
| `event_id` uniqueness (idempotency) | `UNIQUE` constraint on `events.event_id` | FR-012 |
| Idempotent-resubmission vs. conflicting-reuse distinction | `event_type` + canonical `payload_hash` (BLAKE3) + `occurred_at` compared together on `UNIQUE`-constraint conflict — all three must match for idempotent success | FR-012 (corrections 6, 10) |
| Producer-supplied `occurred_at` is well-formed | RFC 3339 parse/validate at the `application` boundary (`time` crate) before insert | FR-012 |
| Ledger immutability | `BEFORE UPDATE`/`BEFORE DELETE` triggers on `events` | FR-013, Constitution V |
| Aggregate rebuild never mutates ledger, and is atomic | Rebuild procedure runs in one `BEGIN IMMEDIATE` transaction, only ever reads `events`, only ever writes `aggregates` | FR-013, SC-004 |
| Forward-only, tested migrations | SQLx migration runner + checksum verification + empty-database test | FR-011, AC-042 |
| No token persistence | No schema/table exists for the token; enforced by absence | FR-017, Constitution VIII |
