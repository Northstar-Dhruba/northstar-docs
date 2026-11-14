# EI-6B — Durable Equity Recommendation Lifecycle Persistence Design v1.0

## Status

**OWNER APPROVED — 2026-11-14 (D6B-1 to D6B-7, with three clarifications).** Version 1.0.

| Item | Status |
| --- | --- |
| EI-6B milestone (D6B-1) | **AUTHORIZED — NOT IMPLEMENTED — OWNER ACCEPTANCE PENDING** |
| O-3 (D6B-2) | **RESOLVED: option (b)** ([section 9](#9-late-evidence-after-terminal-incomplete_evidence-o-3)) |
| This design (D6B-3 to D6B-6) | **APPROVED** (owner, 2026-11-14) |
| Acceptance scenarios ([section 12](#12-acceptance)) | **PLANNED**; none executed |
| EI-6B owner acceptance | **PENDING**; requires the D6B-7 evidence ([section 13](#13-evidence-gate-d6b-7)) |
| EI-6C (monitoring) | **NOT AUTHORIZED** |
| Core change | **None authorized** |
| Operational or production authorization | **NOT GRANTED** |

- **Governance.** The owner decisions are recorded in the [EI-5 Decision Register, section 19](../product/Equity-Intraday-V1-EI5-Decision-Register.md#19-owner-authorization-of-ei-6b-2026-11-14). The architecture is [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), sections 8 to 13 (section 13 amended 2026-11-14), and EI5-D5 to EI5-D8.
- **What this is.** An addendum to the [EI-5 Issuance and Lifecycle Design v1.0](Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md). It details and pins that design's section 9 and the lifecycle part of section 10. It sits beside the [EI-6A SQLite Storage Design v1.0](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md), which it does not change.
- **Not implemented.** No module, table, trigger or test described here exists. Every scenario is **PLANNED**.
- **Binding level.** The invariants, behaviours, the event-document format and the outcome vocabulary are binding. Python and SQLite names are pinned; an implementation PR may rename one only by stating the reason in its description.

Code was read at:

| Repository | Branch | Commit |
| --- | --- | --- |
| northstar-core | `develop` | `29bd34a` (EI-5B lifecycle `9e06d1e`) |
| northstar-application | `develop` | `a17bd91` (#97) |
| northstar-infrastructure | `develop` | `63d2ab3` (#67) |
| northstar-docs | `develop` | `26a9be2` (#65) |

## 1. Scope

**In scope (EI-6B):**

- persisting the Core lifecycle events of an existing, authoritative, MODELED issuance (EI-6A);
- an Application lifecycle-store port, a content-bound append authorization and an Application-owned canonical event document;
- an authoritative lifecycle-window derivation;
- a narrow, non-synthetic Application recording use case acting on explicitly supplied **synthetic** evidence;
- an Infrastructure SQLite store in the same file as EI-4D and EI-6A, with atomic compare-and-append, rehydration, Core replay, restart and concurrency behaviour.

**Not in scope:** monitoring, snapshot selection, scheduling, polling or a monitoring cursor (EI-6C); O-2 and EI3-D10; restatement (EI-8); any new Core event kind; ACTUAL issuance, orders, fills, positions, brokers, real or live data, deployment, API or Web; any relaxation of EI4-D8.

**Trust model.** Single host, development machine only, ADR-024 section 14 unchanged.

## 2. What exists, and the names used

| Concern | Actual code (read-only) | Notes |
| --- | --- | --- |
| Core events | `EquityLifecycleEvent` (northstar-core `recommendations/equity/lifecycle/event.py`) | 13 fields; one allowed shape per kind; no identity other than `sequence`; no serialization |
| Core lifecycle | `EquityRecommendationLifecycle(window, events)` | Re-validates every event and folds the state; `advance(observation)`; `record_source_revision(...)` |
| Core window | `EquityLifecycleWindow(levels, eligible_from, latest_entry_bar_opens_at, entry_window_ends_at, modeled_exit_at)` | `entry_window_ends_at` is not in the issued record |
| EI-5C evaluator | `EvaluateSyntheticEquityRecommendationLifecycleUseCase` (the design's `EvaluateEquityRecommendationLifecycleUseCase`) | Synthetic label, keyed by generation key, in-memory only; private `_lifecycle_window(proposal, candidate)` |
| Issuance authority | `EquityRecommendationIssuanceReader`; `SQLiteEquityRecommendationIssuanceAuthority` | EI-6A, accepted |
| Pattern | Futures `FuturesTradeRecommendationLifecycleStore` | Compare-and-append pattern only; no type reused (ADR-020) |

**Core is unchanged.** Every new type below lives in the Application or Infrastructure.

## 3. Canonical lifecycle-event document (D6B-4)

### 3.1 Format

The format is `northstar.equity-recommendation-lifecycle-event/1`, owned by the Application. One document represents exactly one Core `EquityLifecycleEvent`.

Every document is a JSON object with **exactly these 14 keys**:

| Key | Value | Core field |
| --- | --- | --- |
| `format` | `"northstar.equity-recommendation-lifecycle-event/1"` | — |
| `sequence` | integer, at least 1 | `sequence` |
| `kind` | one of the 11 `EquityLifecycleEventKind` values | `kind` |
| `from_state` | an `EquityLifecycleState` value | `from_state` |
| `to_state` | an `EquityLifecycleState` value | `to_state` |
| `evaluated_at` | canonical `PointInTime` text | `evaluated_at` |
| `evidence` | an evidence object (below) | `evidence` |
| `touched` | array of `EquityLifecycleLevel` values, in Core order | `touched` |
| `ordering` | `"NOT_REQUIRED"` or `"UNKNOWN"` | `ordering` |
| `reason` | `"STOP_TOUCHED_BEFORE_ENTRY"` or `null` | `reason` |
| `deficiencies` | array of deficiency objects (below), possibly empty | `deficiencies` |
| `cited_sequence` | integer or `null` | `cited_sequence` |
| `superseded_evidence` | an evidence object or `null` | `superseded_evidence` |
| `materiality` | `"UNASSESSED"` or `null` | `materiality` |

- **Evidence object:** exactly `reference` (non-empty text), `opens_at` and `closes_at` (canonical `PointInTime` text), and `observations` (an array of non-empty text, in Core order).
- **Deficiency object:** exactly `opens_at`, `closes_at` and `kind` (`MISSING`, `UNFINALIZED`, `CONFLICTING_VALUES` or `UNRESOLVED_REVISIONS`).
- **Not in the document:** the issuance identity. Core events carry none; the binding is the row's key and foreign key (section 7) and the append authorization (section 5).
- **No invented fields.** Whether a field is null or empty is fixed by Core's shape rules for the kind; the document does not restate them.

### 3.2 Encoding and strict decoding

- **Encoding.** The existing `canonical_equity_recommendation_json`: sorted keys, no whitespace, no floats, no NaN. Instants are `PointInTime.value`.
- **Decoding**, refusing on any failure:
  1. parse JSON; require an object with exactly the 14 keys, and exact nested key sets;
  2. require `format` to be `/1`; any other format is **unsupported** and refused;
  3. require each value's JSON type, with no booleans for integers;
  4. construct the Core value objects. An unknown kind, state, level, ordering or deficiency kind is refused, and `PointInTime(text).value` must equal `text`;
  5. construct `EquityLifecycleEvent`, which enforces the kind's shape;
  6. **re-encode and require byte-identical text.**
- **Not done by decoding:** order, continuity, window and terminality across events. Those belong to Core replay (section 8).
- **Golden vectors.** PR-1 includes one golden document per kind, fixed from synthetic values.

### 3.3 No fingerprint

There is no lifecycle-event content fingerprint (D6B-4). Integrity rests on strict canonical validation, Core event construction, Core replay, SQLite constraints, parent-issuance provenance and the single-host trust model. **None of these is cryptographic tamper detection.**

### 3.4 Version boundary

EI-5C events are transient and in memory; no durable lifecycle-event dataset exists. Version 1 is the first durable event contract, so there is no migration. A future version is a new format value, and stored `/1` documents stay readable as written.

## 4. Lifecycle-store port (D6B-3)

New module: northstar-application `ports/equity_recommendation_lifecycle_store.py`.

| Name | Kind | Contract |
| --- | --- | --- |
| `EquityRecommendationLifecycleStore` | ABC port | `load_lifecycle(issuance_id: str) -> EquityRecordedLifecycle \| None`; `append(authorization: VerifiedEquityRecommendationLifecycleAppend) -> EquityRecommendationLifecycleWrite` |
| `EquityRecordedLifecycle` | Frozen record | `issuance: EquityRecordedIssuance` (re-derived parent) and `lifecycle: EquityRecommendationLifecycle` (replayed by Core); `window` is the lifecycle's |
| `EquityRecommendationLifecycleWriteOutcome` | StrEnum | `APPENDED`, `EQUIVALENT`, `CONFLICT` |
| `EquityRecommendationLifecycleWrite` | Frozen record | `outcome` and `recorded: EquityRecordedLifecycle` (the authoritative history at decision time: after the append for `APPENDED`, the equal history for `EQUIVALENT`, the stored history for `CONFLICT`) |
| `EquityRecommendationLifecycleStoreUnavailableError` | RuntimeError | Temporary: cannot open, read or lock; nothing written |
| `EquityRecommendationLifecycleCommitUnconfirmedError` | RuntimeError | COMMIT outcome unknown; never evidence of a rollback |
| `UnverifiedEquityRecommendationLifecycleAppendError` | ValueError | Missing, foreign, unissued or changed authorization; raised before any connection opens |
| `InvalidStoredEquityRecommendationLifecycleError` | ValueError | **Permanent** lifecycle corruption, never retryable. `category` is one of `EVENT` (a document does not decode strictly, or a column disagrees with its document), `HISTORY` (sequence or Core replay refuses the stored events) or `WINDOW` (the window cannot be derived from the parent's provenance) |
| `InvalidEquityRecommendationLifecycleWriteError` | ValueError | The store refused a row the checks admitted; nothing written |

**`load_lifecycle` behaviour:**

| Case | Result |
| --- | --- |
| Unknown issuance | `None` |
| Known issuance, no events | `EquityRecordedLifecycle` with an empty, `RECORDED` lifecycle |
| Existing events | Strictly decoded and replayed through Core with the derived window |
| Corrupt lifecycle | `InvalidStoredEquityRecommendationLifecycleError` |
| Corrupt parent issuance authority | The EI-6A D6-5 error `InvalidStoredEquityRecommendationIssuanceAuthorityError`, unchanged, so the two kinds of corruption stay distinct |
| Unavailable | `EquityRecommendationLifecycleStoreUnavailableError` |

An unverified raw event collection is never returned as an authoritative lifecycle. The port imports no SQLite type.

## 5. Verified append authorization (D6B-5)

New module: northstar-application `application_services/equity_recommendation_lifecycle_authorization.py`, following the EI4-D11 and EI-5D pattern.

- **`VerifiedEquityRecommendationLifecycleAppend`** is frozen, compares by identity and has a weak reference slot. Its fields:
  - `issuance: EquityRecordedIssuance`;
  - `window: EquityLifecycleWindow`;
  - `prior: tuple[EquityLifecycleEvent, ...]`;
  - `events: tuple[EquityLifecycleEvent, ...]`, non-empty;
  - `covered: str`: the canonical content captured at construction.
- **Covered content:** the issuance identity, the canonical documents of `prior` and of `events`, and the expected history position (`len(prior)`).
- **Issued only** by the private `_issue_lifecycle_append`, called only by `RecordEquityRecommendationLifecycleUseCase`. It registers the object in a module-private weak registry.
- **Public validator `require_verified_equity_recommendation_lifecycle_append(authorization)`**, which the store calls **before opening any connection**:
  - exact type and registry membership;
  - the issued record is MODELED;
  - `EquityRecommendationLifecycle(window, prior + events)` replays;
  - `events` continue `prior`'s sequence;
  - the re-derived covered content equals `covered`.
- **Effect:** mutation after validation cannot change what is written; the store writes from the validated canonical documents.
- **Architecture scans:** `append` is called only by the recording use case, and `_issue_lifecycle_append` is used only by the authorization module and that use case. There is no raw append in the production port.

## 6. Authoritative lifecycle-window derivation (clarification 2)

A new public Application function, `equity_issued_lifecycle_window(issuance: EquityIssuedRecommendation, disposition: EquityRecommendationGenerationDisposition) -> EquityLifecycleWindow`.

**What it requires:**

1. the disposition is PROPOSED and its generation key is the issuance's;
2. `equity_issued_recommendation(disposition)` equals the issuance, so the issued record and its source agree;
3. the window comes from the verified proposal's entry window and the candidate's research-window grid: `eligible_from`, `latest_entry_bar_opens_at` (which must be a grid opening), `entry_window_ends_at` (the next opening, or the research window's close), and `modeled_exit_at` (which must be a grid close);
4. the window's levels, `eligible_from`, `latest_entry_bar_opens_at` and `modeled_exit_at` equal the issued record's.

**Failures.** Any missing, inconsistent or undecodable provenance refuses, so a caller can never supply or override any window value.

**Sharing with EI-5C.** EI-5C's private `_lifecycle_window` delegates to the shared grid logic with identical behaviour and refusals, so that is the only change to EI-5C. EI-5C keeps its synthetic label and results, and its tests stay unchanged and passing.

**Who calls it.** The recording use case, with the disposition re-verified at levels A to C, and the Infrastructure store, with the disposition decoded at level A, as EI-6A loads it. Infrastructure calls this function and never computes a window itself.

## 7. Recording use case (D6B-1; clarification 3)

New module: northstar-application `application_services/record_equity_recommendation_lifecycle.py`.

```text
RecordEquityRecommendationLifecycleUseCase(
    issuances: EquityRecommendationIssuanceReader,
    generations: VerifyStoredEquityRecommendationGenerationUseCase,
    snapshots: EquityCandleEvidenceSnapshotStore,
    lifecycles: EquityRecommendationLifecycleStore,
    max_writes: int = 3,
).execute(issuance_id: str, snapshot_ids: tuple[str, ...]) -> EquityLifecycleRecordingResult
```

1. Load the issuance through the reader. If it is absent, refuse.
2. Re-verify its disposition at levels A to C by generation key.
3. Derive the window (section 6).
4. Load the stored lifecycle (`load_lifecycle`).
5. For each explicit snapshot, apply the EI-5C rules unchanged:
   - SYNTHETIC evidence class only;
   - same listing, trading date and venue reference;
   - same research window, timeframe and currency;
   - `as_of` at or after the modeled issuance and the last event;
   - within the call, strictly increasing `as_of` and a non-regressing watermark.

   Then call Core `advance` and record any `SOURCE_REVISION_RECORDED` (EI5-D8).
6. If no event resulted, return `NO_NEW_EVENTS` without writing.
7. Otherwise issue the authorization with `prior` set to the stored events, and append once.
8. Map the outcome. On commit uncertainty, reload (bounded by `max_writes`):
   - stored equals prior plus new: `EQUIVALENT`, reconciled;
   - stored equals prior: retry;
   - anything else: `CONFLICT`;
   - reload failure or bound exhausted: an unresolved error, with nothing claimed;
   - corruption on reload: the permanent error.
9. Return a result consistent with the committed history.

**Result.** `EquityLifecycleRecordingResult` holds `outcome` (`EquityLifecycleRecordingOutcome`: `APPENDED`, `EQUIVALENT`, `CONFLICT`, `NO_NEW_EVENTS`), `recorded`, `new_events`, `writes` and `reconciled`.

**Refusals.** Refusals before any write raise `EquityLifecycleRecordingRefusedError(reason, retryable)`, with at least these reasons:

| Reason | Kind |
| --- | --- |
| `ISSUANCE_NOT_FOUND` | Permanent |
| `STORED_AUTHORITY_INVALID` | Permanent |
| `STORED_LIFECYCLE_INVALID` | Permanent |
| The issuer's generation-verification reasons | As the issuer classifies them |
| `WINDOW_PROVENANCE_INVALID` | Permanent |
| The EI-5C snapshot reasons | Permanent |
| `STORE_UNAVAILABLE`, `AUTHORITY_UNAVAILABLE`, `SOURCE_EVIDENCE_UNAVAILABLE` | Retryable |

**Boundary.** It never selects snapshots, polls, schedules, advances a monitoring cursor, issues, or creates orders, fills or positions. Being non-synthetic in name grants no permission to use real data: it refuses non-SYNTHETIC evidence.

**Quiet evaluations (clarification 3).** There is no persisted quiet-evaluation cursor and no processing-offset table.

- **Nothing fabricated.** A snapshot that produces no event fabricates no event and advances nothing.
- **No exactly-once claim** is made for snapshot processing.
- **Repetition is safe.** Repeating a snapshot, or evaluating one that produces nothing, is deterministic and safe: Core re-reads quiet bars from the cursor, and revision recording is idempotent.
- **Persisted events are the authority.** State derives only from them. Every event cites its own snapshot and interval, so provenance is per event, not per processing run.
- **Deferred to EI-6C:** which snapshots are evaluated, in what order, and whether every one was processed (O-2).

## 8. SQLite lifecycle-event store (D6B-6)

### 8.1 Placement and conventions

- **Placement:** new modules `sqlite_equity_recommendation_lifecycle_schema.py` and `sqlite_equity_recommendation_lifecycle.py` in northstar-infrastructure.
- **Same file** as EI-4D and EI-6A.
- **Additive:** `IF NOT EXISTS`, no schema version, no change to any existing object.
- **Refusal:** a database without the EI-6A issuance tables is refused.
- **Foreign keys** are turned on and confirmed on every connection.
- **Instants** are stored as text plus integer microseconds, using the EI-3C and EI-6A derivation.
- **Cleanup** follows the L9 pattern.

### 8.2 Table `equity_recommendation_lifecycle_events`

| Column | Rule |
| --- | --- |
| `issuance_id` | `NOT NULL`, `GLOB 'eiris1-sha256:?*'`; **foreign key** to `equity_recommendation_issuances (issuance_id)` |
| `event_sequence` | Integer, at least 1. **PRIMARY KEY (`issuance_id`, `event_sequence`)** |
| `event_format` | `CHECK = 'northstar.equity-recommendation-lifecycle-event/1'` |
| `event_kind` | `CHECK IN` the 11 kinds |
| `from_state`, `to_state` | `CHECK IN` the 11 states; `CHECK ((event_kind = 'SOURCE_REVISION_RECORDED') = (from_state = to_state))` |
| `evaluated_at`, `evaluated_at_us` | Canonical text; the integer equals its derivation |
| `evidence_reference`, `evidence_opens_at`, `evidence_closes_at` | The event's evidence |
| `event_document` | `CHECK (json_valid(event_document))` |
| — | CHECKs tie every structured column to its document through `json_extract` (format, sequence, kind, states, `evaluated_at`, evidence reference and interval), so a column cannot disagree with its document |

**Sequence domain.** Lifecycle sequences are per issuance. The EI-6A recording-sequence ledger is **not** used and is unchanged; no issuance or declaration record is modified.

### 8.3 Database protections (defence in depth; Core stays authoritative)

| Protection | Mechanism |
| --- | --- |
| Insert-only, no replacement | Triggers refuse UPDATE, DELETE and a replacing INSERT |
| Per-issuance continuity | Trigger: the new sequence equals the issuance's maximum + 1 |
| Chronology | Trigger: `evaluated_at_us` at or after the issuance's previous event; canonical-instant trigger |
| Not before the parent issuance | Trigger: `evaluated_at_us` at or after the parent's `issued_at_us` |
| Terminal guard | Trigger: no state-changing kind after the issuance's latest state-changing event entered a terminal state |
| Provenance | Foreign key to the issuance |

**Not checked by SQL:** event shapes, the window, the cursor, interval ranges and revision citations. Those are Core replay's; SQLite does not claim to verify Core's contract and holds no authoritative lifecycle state.

### 8.4 Provenance

A foreign key is necessary, not sufficient. Every load and append re-derives the parent issuance through the accepted EI-6A boundary, decodes its disposition and derives the window (section 6). Missing, invalid or corrupt parent authority fails closed.

### 8.5 Append transaction

1. Validate the authorization **before connecting** (section 5).
2. `BEGIN IMMEDIATE`.
3. Re-derive the parent issuance; it must equal `authorization.issuance`.
4. Load, strictly decode and replay the stored events (section 8.6).
5. Compare the stored history with `authorization.prior`, by canonical documents:

   | Stored history | Outcome |
   | --- | --- |
   | Equals `prior` | Proceed |
   | Equals `prior + events` | `EQUIVALENT`; nothing inserted |
   | Anything else | `CONFLICT`; nothing inserted |

6. Replay `prior + events` through Core with the derived window.
7. Insert the whole batch.
8. Confirm that the issuance's maximum sequence is `len(prior) + len(events)`.
9. Commit once.

Failures:

- any failure before COMMIT rolls back, so no partial batch is ever authoritative;
- a COMMIT that raises is reported as `EquityRecommendationLifecycleCommitUnconfirmedError`;
- a refused row is reported as `InvalidEquityRecommendationLifecycleWriteError`;
- a lock not obtained is reported as unavailable.

### 8.6 Load and replay

1. Re-derive the parent issuance; it is `None` when unknown.
2. Derive the window.
3. Read the rows by issuance in sequence order.
4. Require contiguous sequences 1..n.
5. Strictly decode each document (section 3.2) and require its columns to agree.
6. Run `EquityRecommendationLifecycle(window, events)`.

Any failure raises the permanent error of section 4 with its category. Nothing is repaired, migrated or skipped. An unsupported format version is `EVENT`. There is no second state machine in Infrastructure.

## 9. Late evidence after terminal `INCOMPLETE_EVIDENCE` (O-3)

**Resolved: option (b)** ([EI-5 Decision Register, section 19.3](../product/Equity-Intraday-V1-EI5-Decision-Register.md#193-d6b-2--o-3-resolved-option-b)).

- **No event.** Later evidence that fills the original gap of a terminal `INCOMPLETE_EVIDENCE` produces **no lifecycle event** in EI-5 or EI-6.
- **Unchanged.** The terminal state is final. No event is deleted, corrected, superseded or rewritten, and nothing reopens automatically.
- **Where the evidence lives.** The late evidence remains in the immutable EI-2B journal and snapshots.
- **Revisions are a different case.** `SOURCE_REVISION_RECORDED` is unchanged and applies only where Core already permits it: a correction to evidence a bar-driven event cited. Core refuses a revision citing `EVIDENCE_INCOMPLETE`; that refusal is preserved and is not bypassed. A late gap-filling bar is not a source revision.
- **Product specification section 8.2.** Its audited-event requirement for late gap-filling evidence is deferred to a future EI-8 restatement design, which is **not** authorized. No reconciliation event kind and no Core change is authorized.
- **RV-04:** terminal state unchanged; no new lifecycle event for late gap-filling evidence.

## 10. Concurrency and recovery semantics (clarification 1)

| Case | Outcome |
| --- | --- |
| First valid append | `APPENDED` |
| Identical authorized retry (stored equals prior plus events) | `EQUIVALENT`; nothing inserted |
| Two concurrent writers, same prior history, identical events | First committed wins (`APPENDED`); the second is `EQUIVALENT`; no duplicates |
| Two concurrent writers, same prior history, different events | First committed wins; the second is `CONFLICT`; its events are never adopted |
| Stale writer (history advanced incompatibly) | `CONFLICT`; no merge or reorder |
| Commit uncertain | Reload: `EQUIVALENT` only after confirming the exact events are committed; otherwise retry within the bound, or the typed commit-unconfirmed or unresolved result. No blind retry |
| Corrupt stored history | Permanent `InvalidStoredEquityRecommendationLifecycleError`; never `EQUIVALENT` |
| Invalid transition, sequence gap, or state change after a terminal state | Refused before persistence: Core, the validator, and the database's defence in depth |

## 11. Terminal authority

EI5-D5 to EI5-D8 are preserved:

- every terminal state is final;
- no state-changing event follows a terminal state;
- no event is rewritten;
- no automatic reopening or unapproved restatement;
- materiality stays `UNASSESSED`;
- Core-permitted revisions are unchanged.

For terminal `INCOMPLETE_EVIDENCE`, O-3 option (b) applies. An append violating Core's terminal rules is refused before persistence.

## 12. Acceptance

**Status for every row: PLANNED.**

**Layers:**

| Layer | Meaning |
| --- | --- |
| C | Application contract with a labelled in-memory store |
| E | Synthetic end to end |
| S | Real SQLite |
| A | Architecture scan |

### 12.1 Rows of the EI-5 matrix

| ID | Requirement | Layer | Status |
| --- | --- | --- | --- |
| RR-01 | Lifecycle events reloaded after restart: the same state through Core replay | S | PLANNED |
| RR-04 (state) | Replay of the lifecycle state from a separate OS process | S | PLANNED |
| RR-05 | A stored event tampered in a test: refused on reload; preserved | S | PLANNED |
| RR-06 | Two evaluators append from the same position: one succeeds; the other is refused (`CONFLICT`), or `EQUIVALENT` when identical | C, S | PLANNED |
| RV-04 | Late evidence fills an interval cited by `INCOMPLETE_EVIDENCE`: terminal state unchanged; no new lifecycle event (O-3 (b)) | C, E | PLANNED |

### 12.2 EI-6B lifecycle-store scenarios (LS)

| ID | Requirement | Layer | Status |
| --- | --- | --- | --- |
| LS-01 | First append to an empty lifecycle: `APPENDED`; sequences 1..n stored; `load_lifecycle` returns the replayed lifecycle | C, S | PLANNED |
| LS-02 | (a) Append after existing history: `APPENDED`, contiguous. (b) A repeated snapshot, or one producing no event: `NO_NEW_EVENTS`, nothing written or fabricated, deterministic (quiet evaluations) | C, E, S | PLANNED |
| LS-03 | (a) Known issuance, no events: an empty `RECORDED` lifecycle. (b) Unknown issuance: `None`; the recording use case refuses `ISSUANCE_NOT_FOUND` | C, S | PLANNED |
| LS-04 | Parent provenance: (a) the foreign key refuses an event for a non-issued identity (raw); (b) a corrupt parent issuance raises the D6-5 error on load and append; (c) a matching identifier alone is not authority (re-derivation required) | S | PLANNED |
| LS-05 | Unauthorized append (missing, foreign, unissued, changed prior or events) refused before any connection opens; database unchanged | C, S | PLANNED |
| LS-06 | UPDATE, DELETE and replacing INSERT refused on the lifecycle table | S | PLANNED |
| LS-07 | Per-issuance continuity: a gap or duplicate refused; sequences independent across issuances; the EI-6A ledger unchanged | S | PLANNED |
| LS-08 | Chronology: integer microseconds; non-decreasing; not before the parent's `issued_at`; canonical text and integer agree | S | PLANNED |
| LS-09 | Strict canonical documents: golden vectors per kind round-trip; non-canonical text, missing or extra keys, an unknown kind and an unsupported format refused (codec C; stored S, category `EVENT`) | C, S | PLANNED |
| LS-10 | A stored history Core refuses (planted) raises `HISTORY`, unrepaired; a column disagreeing with its document is refused by CHECK | S | PLANNED |
| LS-11 | Atomic multi-event batch: a failure mid-batch persists none; the next append takes the same sequences | S | PLANNED |
| LS-12 | Commit uncertainty: acknowledgement lost then reload gives `EQUIVALENT`, reconciled; commit lost gives a bounded retry; an unresolved outcome claims nothing; corruption on reload is permanent | C, S | PLANNED |
| LS-13 | Identical retry, and concurrent identical requests in spawned processes: one `APPENDED`, one `EQUIVALENT`, no duplicates; a retry after restart is `EQUIVALENT` | C, S | PLANNED |
| LS-14 | Conflicting concurrent appends in spawned processes, and a stale writer: first wins; `CONFLICT`; events not adopted (with RR-06) | C, S | PLANNED |
| LS-15 | Terminal authority: (a) a state change after a terminal state refused before persistence (Core, validator) and by the trigger (raw); (b) a Core-permitted `SOURCE_REVISION_RECORDED` after a terminal state accepted; (c) no revision citing `EVIDENCE_INCOMPLETE` (with RV-04) | C, S | PLANNED |
| LS-16 | Additive schema (only EI-6B objects added; EI-4D and EI-6A objects and rows unchanged; idempotent; a foreign database refused) and restart-safe reconstruction in new adapter instances and a fresh interpreter (with RR-01, RR-04) | S | PLANNED |
| LS-17 | Authoritative window: (a) derived from the issued record and its verified disposition; (b) equal to EI-5C's window for the same disposition; (c) a disposition or grid inconsistent with the issued record, or undecodable, refuses (`WINDOW_PROVENANCE_INVALID`, or `WINDOW` on load); (d) no caller parameter supplies any window value; (e) Infrastructure calls the Application function and computes no window itself | C, S, A | PLANNED |

**Architecture rows (A):** `append` only in the recording use case; the private issuer only in the authorization and recording modules; the adapter exposes exactly the port methods; the lifecycle modules import no broker, execution, network, clock or randomness. These run with existing scans and do not take a scenario ID.

## 13. Evidence gate (D6B-7)

Owner acceptance of EI-6B requires:

1. local Windows synthetic and real-SQLite acceptance;
2. independently verified successful Linux `python-ci.yml` runs for the Application PR-1 revision and the Infrastructure PR-2 revision;
3. exact tested commit identities, and the Core and Application dependency commits checked out;
4. evidence that the EI-6B tests actually executed;
5. a separate owner decision after reviewing the evidence.

A merged PR alone does not satisfy this gate. EI-6A's CI evidence cannot substitute for EI-6B's tests.

**Testing strategy:**

- **During development:** focused contract, codec golden and round-trip, authorization and in-memory tests, then focused SQLite tests.
- **One final gate:** Application and Infrastructure suites, Ruff, architecture scans, real-SQLite concurrency with spawned processes, fresh-interpreter replay, and targeted mutations of the critical invariants:
  - the prior-history comparison;
  - the per-issuance sequence trigger;
  - the not-before-issuance trigger;
  - strict re-encoding;
  - the terminal guard;
  - window cross-checks.

## 14. Implementation sequence

| PR | Repository | Delivers | Prerequisite |
| --- | --- | --- | --- |
| PR-0 | northstar-docs | This addendum; D6B-1 to D6B-7; O-3; the product specification 8.2 deferral | — |
| PR-1 | northstar-application | Port and errors; canonical codec; window function; authorization; recording use case; in-memory store; C, E and A tests | PR-0 merged |
| PR-2 | northstar-infrastructure | Schema; adapter; provenance; compare-and-append; replay; S tests | PR-1 merged (Infrastructure CI checks out Application `develop`) |
| PR-3 | northstar-docs | Engineering acceptance record; matrix; Linux CI evidence; owner decision | PR-2 merged |

Core is read-only throughout. No PR imports unmerged interfaces.

## 15. Limitations and carried-forward restrictions

- **No exactly-once snapshot processing.** Quiet evaluations are not persisted (section 7).
- **No reconciliation event.** Late gap-filling evidence after terminal `INCOMPLETE_EVIDENCE` produces none (O-3 (b)); product specification section 8.2 is deferred in part.
- **O-2.** Which snapshot is evaluated decides when terminal `INCOMPLETE_EVIDENCE` is recorded. Persisted lifecycles stay synthetic evidence only until EI-6C.
- **Carried forward:** EI4-D8; EI-6A HW-03; EI-6A's adapter-enforced ledger checks; VC-04 (now more consequential, because lifecycle rows rest on evaluator-v2 issuances); N6; I08; IS-16; NB-04; EI-4 formal CI closure outstanding; single host; MODELED only.

## 16. Explicit non-authorizations

Not authorized:

- EI-6C monitoring; automatic snapshot selection; scheduling or polling;
- O-2 or EI3-D10 resolution;
- ACTUAL issuance; broker orders or fills; real-money trading;
- live or real market data; production deployment; API or Web;
- EI4-D8 relaxation;
- terminal-state reopening; historical lifecycle rewriting;
- EI-8 restatement;
- any new Core event kind or Core change.

EI-6B remains synthetic-evidence-only despite its non-synthetic Application recording boundary.

## 17. Related documents

- [EI-5 Decision Register, sections 6 and 19](../product/Equity-Intraday-V1-EI5-Decision-Register.md#19-owner-authorization-of-ei-6b-2026-11-14)
- [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design v1.0](Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md), sections 5, 6, 9, 10 and 12
- [EI-6A SQLite Storage Design v1.0](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md)
- [EI-6A Engineering Acceptance Record](../product/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md)
- [Equity Intraday V1 Product Specification, section 8.2](../product/Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states)
