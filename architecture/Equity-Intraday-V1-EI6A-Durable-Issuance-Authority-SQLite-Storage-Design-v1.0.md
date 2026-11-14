# EI-6A — Durable SQLite Recommendation Issuance and Declaration Authority: Storage Design v1.0

## Status

**OWNER APPROVED — 2026-11-14 (D6-4, with additional integrity requirements).** Version 1.0.

| Item | Status |
| --- | --- |
| EI-6A milestone (D6-1) | **AUTHORIZED, NOT IMPLEMENTED** |
| This storage design (D6-4) | **APPROVED** (owner, 2026-11-14) |
| Typed corrupt-record Application contract (D6-5) | **AUTHORIZED** as one prerequisite northstar-application PR; **NOT STARTED** |
| EI-6A acceptance scenarios ([section 12](#12-acceptance)) | **PLANNED**; none executed |
| EI-6A owner acceptance | **PENDING**; requires local Windows real-SQLite acceptance **and** independently verified Linux CI (D6-6, [section 13](#13-mandatory-ci-evidence-d6-6)) |
| EI-6B (lifecycle persistence), EI-6C (monitoring) | **NOT AUTHORIZED** |
| Operational or production authorization | **NOT GRANTED** |

> **Status update, 2026-11-14: implemented; local engineering acceptance recorded.** The table above records the state at approval and is not rewritten.
>
> - **Merged.** The D6-5 Application contract (northstar-application #97, `a17bd91`) and this design (northstar-infrastructure #67, `63d2ab3`) are merged.
> - **Acceptance.** Local Windows real-SQLite engineering acceptance is PASS WITH DOCUMENTED LIMITATIONS: 26 rows, 19 VERIFIED and 7 VERIFIED WITH LIMITATION ([EI-6A Engineering Acceptance Record](../product/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md)). The section 12 statuses keep their plan.
> - **Pending.** D6-6 Linux CI is not verified, so **owner acceptance is PENDING**.

> **Status update, 2026-11-14: accepted.** The update above records the state before this decision and is not rewritten. D6-6 is satisfied (Linux runs `43599364295` and `43604917993`), and **EI-6A is ACCEPTED WITH DOCUMENTED LIMITATIONS (owner, 2026-11-14)**: 25 VERIFIED and 1 VERIFIED WITH LIMITATION ([EI-6A Engineering Acceptance Record, section 13.1](../product/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md#131-owner-acceptance-2026-11-14)).

- **Governance.** The owner decisions D6-1 to D6-6 are recorded in the [EI-5 Decision Register, section 17](../product/Equity-Intraday-V1-EI5-Decision-Register.md#17-owner-authorization-of-ei-6a-2026-11-14). The architecture is [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md) sections 4 to 7, 13 and 14, with its dated staged-delivery amendment of 2026-11-14.
- **What this document is.** An addendum to the [EI-5 Issuance and Lifecycle Design v1.0](Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md). It details and tightens that design's section 10 for the issuance and declaration authority only. It changes no invariant of the EI-5 design, section 8.2, and no Application port, record, identity or format.
- **Not implemented.** No table, trigger, adapter or test described here exists. Every scenario is **PLANNED**.
- **Binding level.** The invariants, constraints and behaviours below are binding on the EI-6A implementation. SQLite object names are proposals; the implementation PR may rename an object only, and must say so in its description.

Code was read at:

| Repository | Branch | Commit |
| --- | --- | --- |
| northstar-core | `develop` | `29bd34a` (#40) |
| northstar-application | `develop` | `9a68209` (EI-5D/EI-5E, #96) |
| northstar-infrastructure | `develop` | `f71f8f7` (EI-4C slice 4, #66) |
| northstar-docs | `develop` | `825d3ef` (EI-5D/EI-5E acceptance, #62) |

## 1. Scope

**In scope (EI-6A):** an Infrastructure SQLite adapter implementing the merged Application ports `EquityRecommendationIssuanceReader` and `EquityRecommendationIssuanceWriter` (northstar-application `ports/equity_recommendation_issuance.py`):

- durable MODELED issued recommendations;
- durable session completeness declarations;
- one authority-wide, transactionally allocated recording sequence;
- current and historical session views;
- restart, separate-process and concurrency behaviour, and commit-uncertain recovery;
- refusal of corrupt stored records, without repair.

**Not in scope:**

- the lifecycle-event table, `EquityRecommendationLifecycleStore` and compare-and-append (EI-6B);
- monitoring, snapshot selection, composition roots and scheduling (EI-6C or later);
- ACTUAL issuance, orders, fills, positions, brokers, real data, API or Web;
- any change to EI-4D tables, rows, identities or formats.

## 2. Placement and schema conventions

The EI-6A objects follow the EI-4D conventions of `sqlite_equity_recommendation_generation_schema.py`:

- **One database authority.** The objects live in the **same SQLite file** as the EI-2B evidence, the EI-3C timelines and the EI-4D generation records (ADR-024 section 13; ADR-021).
- **Additive.** Initialization creates only the EI-6A tables, indexes and triggers, with `CREATE … IF NOT EXISTS`. It never alters, rebuilds or migrates an existing table, row, index or trigger, and adds no trigger to an EI-4D table. There is no schema-version table.
- **Refuses a foreign database.** Initialization and every connection refuse a database that does not already hold the EI-4D generation tables (and therefore the EI-2B and EI-3C evidence they require). It never creates an empty copy of the records it references.
- **Foreign keys.** Every connection turns `PRAGMA foreign_keys` on and confirms that it is on before any read or write, as the EI-4D store's connection does.
- **Instants.** Every instant is stored twice: as canonical `PointInTime` text and as integer microseconds since 1970-01-01T00:00:00Z. Canonical text does not sort chronologically (`…:00.5Z` sorts before `…:00Z`), so **every chronological comparison in SQL uses the integer column**. A trigger requires the text to be canonical and the integer to equal its derivation, using the canonical-instant check and integer-microsecond derivation of the EI-3C timeline schema (`decision_instant_us`).
- **Immutability.** Triggers refuse every UPDATE and DELETE, and every INSERT that would replace a row, on all three tables. As in EI-4D, a replacing insert is refused before it can delete.

## 3. Objects

| Object (proposed name) | Purpose |
| --- | --- |
| `equity_recommendation_issuance_ledger` | The authority-wide recording sequence, shared by issuances and declarations ([section 4](#4-the-recording-sequence-ledger)) |
| `equity_recommendation_issuances` | MODELED issued recommendations ([section 5](#5-the-issuance-table)) |
| `equity_recommendation_issuance_declarations` | Session completeness declarations ([section 6](#6-the-completeness-declaration-table)) |
| Indexes | On the session columns of the issuance and declaration tables |

No issued-record document, no new canonical format and no new fingerprint is stored (D6-4). The full issued record is re-derived on load ([section 8](#8-loading-and-re-derivation)).

## 4. The recording-sequence ledger

### 4.1 Columns

| Column | Rule |
| --- | --- |
| `recording_sequence` | `INTEGER PRIMARY KEY`, at least 1. Assigned only by the writer inside its transaction; never supplied by a caller (EI5-D4, HW-06). No `AUTOINCREMENT`, so no `sqlite_sequence` counter exists |
| `record_kind` | `'ISSUANCE'` or `'DECLARATION'` |
| — | `UNIQUE (recording_sequence, record_kind)`, the parent key of the record tables' composite foreign keys |

### 4.2 Allocation

Inside the write transaction the next sequence is `COALESCE(MAX(recording_sequence), 0) + 1`. An insert trigger refuses any other value, so the committed sequence is **dense**: exactly `1 … N`, with `N` the authority's current position (its watermark).

A sequence is **assigned only when its transaction commits**. If the transaction rolls back, nothing was assigned, and the next committed write takes the same number. That is not reuse: no committed record ever held it (ADR-024 section 7: "unique and never reused").

### 4.3 Integrity requirements and how each is enforced

A foreign key from a record to the ledger proves that the ledger entry exists. It does **not** prove that every ledger entry has its record: SQLite cannot express a polymorphic reverse reference with a simple constraint. Requirement 4 is therefore enforced by the transaction and by fail-closed checks, not by a foreign key.

| # | Requirement | Database enforcement | Transaction and read enforcement |
| --- | --- | --- | --- |
| 1 | No duplicate sequence numbers | Ledger primary key; `UNIQUE` sequence on each record table; one ledger row per sequence means an issuance and a declaration can never share one | — |
| 2 | No invalid ordering | Ledger insert trigger: the new sequence must be the current maximum + 1 | The writer allocates only as in section 4.2 |
| 3 | Ledger kind matches the record | Each record table stores a constant kind column (`CHECK` equal to `'ISSUANCE'` or `'DECLARATION'`) and a composite foreign key `(recording_sequence, record_kind)` to the ledger | — |
| 4 | No committed ledger entry without its record | **Not expressible as a simple constraint.** Defence in depth: a ledger insert trigger refuses a new sequence while the previous sequence lacks its record, so an orphan blocks every later write | **Required.** Before COMMIT, the writer confirms inside the same transaction that the new sequence has exactly one record of its kind and that the ledger maximum is that sequence; otherwise it rolls back. Every read that establishes a watermark, and every write transaction, first checks the whole ledger: row count equals maximum, and every entry has exactly one record of its kind. Any inconsistency is refused as corrupt authority state ([section 9](#9-corruption-handling-d6-5)) |
| 5 | No record without a valid ledger entry | `NOT NULL` sequence and the composite foreign key of requirement 3, on connections with foreign keys on | The same whole-ledger check refuses a record whose sequence has no ledger entry of its kind, which covers rows written with foreign keys off |
| 6 | No partial allocation after a failure | The ledger entry and its record are inserted in **one** transaction; SQLite rolls both back together | Any failure before COMMIT rolls back through the guarded rollback, close and discard pattern of the L9 fixes. COMMIT is issued once. A COMMIT whose outcome is unknown is reported as commit-unconfirmed and resolved by reload ([section 7.3](#73-commit-uncertainty)), never by assuming either outcome |

## 5. The issuance table

### 5.1 Columns

| Column | Rule |
| --- | --- |
| `issuance_id` | Primary key. `GLOB 'eiris1-sha256:?*'` (C-1; ID-03) |
| `generation_key` | `NOT NULL UNIQUE`, `GLOB 'eirk1-sha256:?*'`; **foreign key** to `equity_recommendation_generation_dispositions (generation_key)` |
| `proposal_fingerprint` | `NOT NULL`, `GLOB 'eirp1-sha256:?*'` |
| `assessment_fingerprint` | `NOT NULL`, `GLOB 'eira1-sha256:?*'` |
| `issuance_kind` | `CHECK (issuance_kind = 'MODELED')` |
| `strategy`, `symbol`, `exchange_code`, `trading_date` | `NOT NULL`; the listing is `symbol` and `exchange_code`; `trading_date` in `YYYY-MM-DD` form. `UNIQUE (strategy, symbol, exchange_code, trading_date)` (EI5-D1) |
| `issued_at` | Canonical `PointInTime` text |
| `issued_at_us` | Integer microseconds, equal to the derivation from `issued_at` (section 2) |
| `recording_sequence` | `NOT NULL UNIQUE` |
| `record_kind` | `CHECK (record_kind = 'ISSUANCE')`; with `recording_sequence`, a composite foreign key to the ledger (section 4.3) |

The uniqueness rules of ADR-024 section 7 — per issuance identity, per generation key, and per (strategy, listing, trading date) — are each a database constraint.

### 5.2 Source-provenance trigger

A matching generation key alone is **not** sufficient authority. A `BEFORE INSERT` trigger refuses the row unless the disposition it references:

- is `PROPOSED`;
- has `evaluator_version` exactly `northstar.equity-recommendation-evaluator/2`;
- has the issuance's `proposal_fingerprint` and `assessment_fingerprint`;
- has the issuance's `strategy`, `symbol`, `exchange_code` and `trading_date`.

This is defence in depth only. Authority comes from the verified Application issuer, which re-verifies the disposition (levels A to C) before it issues a content-bound authorization (ADR-024 section 4), and from the re-derivation on every load (section 8). The database cannot compute the SHA-256 identity; the Application re-derives it.

This trigger checks the referenced row's evaluator version. It is **not** the deferred N6 label-format pairing trigger on the disposition table, which stays **DEFERRED**.

### 5.3 No-backdating trigger

A `BEFORE INSERT` trigger refuses an issuance whose `issued_at_us` is at or below the greatest `complete_through_us` declared for its (`exchange_code`, `trading_date`) (EI5-D4; BD-01 S). The writer refuses the same case first, as `BACKDATED_ISSUANCE` (section 7.1); the trigger guards against a writer defect.

## 6. The completeness declaration table

| Column | Rule |
| --- | --- |
| `recording_sequence` | Primary key |
| `record_kind` | `CHECK (record_kind = 'DECLARATION')`; with `recording_sequence`, a composite foreign key to the ledger |
| `exchange_code`, `trading_date` | `NOT NULL`; the session |
| `complete_through` | Canonical `PointInTime` text |
| `complete_through_us` | Integer microseconds, equal to its derivation |
| — | `UNIQUE (exchange_code, trading_date, complete_through_us)` |

- **Strictly monotone.** A `BEFORE INSERT` trigger refuses a declaration whose `complete_through_us` is not strictly greater than the session's current greatest.
- **Equivalent retries write nothing.** An equal declaration is `DECLARATION_EQUIVALENT` and a lower one `COMPLETENESS_REGRESSION`; both are decided by the writer, and neither inserts a declaration or a ledger entry.
- **Venue day.** Whether `complete_through` lies within the session's venue day is validated by the Application declaration use case (EI-5 design, section 7.5), not by the database.
- **Append-only**, as section 2.

## 7. Write transactions

### 7.1 Issuance and declaration writes

`write_issuance(authorization)` and `write_declaration(authorization)` each:

1. **Validate the authorization before any connection opens**, with the Application's public validator (`require_verified_equity_recommendation_issuance` or `require_verified_equity_issuance_completeness_declaration`). A missing, foreign, unissued or changed authorization raises `UnverifiedEquityRecommendationIssuanceWriteError`, and the database is not opened (ADR-023 pattern).
2. **Start `BEGIN IMMEDIATE`**, taking the write lock before reading.
3. **Load the authoritative state** under that lock:
   - the whole-ledger consistency check of section 4.3;
   - for an issuance: the stored issuances with the offered identity, generation key or (strategy, listing, trading date), each re-derived (section 8), and the session's declarations;
   - for a declaration: the session's declarations.
4. **Apply the Application-owned rules**, `equity_issuance_write_decision` or `equity_declaration_write_decision` (northstar-application `equity_recommendation_issuance_rules.py`). The adapter does not restate or reorder them: identity first, then generation key, then session, then backdating.
5. **Allocate** the next recording sequence (section 4.2), only for `ISSUED` or `DECLARED`.
6. **Insert** the ledger entry and its record, in that order, in the same transaction.
7. **Confirm and commit once**: the pre-commit check of section 4.3, requirement 4, then a single COMMIT.
8. **Report** the port's write result, with the stored record it concerns. Every other outcome inserts no ledger entry and no record.

**First-committed-record authority.** Because the check and the insert happen under one write lock, the first committed record for an identity, generation key or session stays authoritative. A later conflicting write is reported as a conflict and never replaces it.

### 7.2 Failure mapping

| Failure | Port behaviour |
| --- | --- |
| Database cannot be opened or read, is not the evidence database, or foreign keys cannot be confirmed | `EquityRecommendationIssuanceUnavailableError`; nothing written |
| Write lock not obtained within the bounded timeout (CC-05) | `EquityRecommendationIssuanceUnavailableError`; nothing written |
| A constraint or trigger refuses a row the rules admitted | Rolled back; nothing written; reported as a typed failure, never as success |
| Corrupt stored state | [Section 9](#9-corruption-handling-d6-5); rolled back; nothing written |
| COMMIT raises | `EquityRecommendationIssuanceCommitUnconfirmedError` (section 7.3) |

### 7.3 Commit uncertainty

A COMMIT that raises is reported as `EquityRecommendationIssuanceCommitUnconfirmedError`. It is never evidence of a rollback. The Application use cases reconcile it with their existing, bounded protocol: an authoritative reload by issuance identity (issuer) or of the session's declarations (declarer), at most `max_writes` writes, and an unresolved error when the reload cannot settle it (EI4-D9-G pattern; CC-06). The adapter adds no retry of its own.

### 7.4 Concurrency

Serialization comes from SQLite's write lock and the constraints, never from in-memory locks. The authority is one SQLite file on one host; no multi-host claim is made (ADR-024 section 14).

## 8. Loading and re-derivation

Every stored issuance is returned only after it is re-derived:

1. Load the issuance row and the disposition its `generation_key` references.
2. Decode the disposition through the existing EI-4D stored-record decoding, which constructs the Application disposition record and so decodes and verifies its documents and fingerprints.
3. Require it to be `PROPOSED` with evaluator `/2`.
4. Derive the issued record with the Application's `equity_issued_recommendation(disposition)`, which also re-computes the `eiris1` identity.
5. Require every stored structured column — identity, generation key, both fingerprints, kind, strategy, listing, trading date and `issued_at` — to equal the derived record's.
6. Return `EquityRecordedIssuance(recommendation, recording_sequence)`.

Any failure in steps 1 to 5 is corruption ([section 9](#9-corruption-handling-d6-5)). Nothing is repaired, migrated, skipped or rewritten.

A stored declaration is returned as `EquityRecordedCompletenessDeclaration` only when its text and integer instants agree and its ledger entry is consistent.

## 9. Corruption handling (D6-5)

**Adapter behaviour.**

| Where the corruption is found | Behaviour |
| --- | --- |
| A stored row under the **offered issuance identity** that does not re-derive, or re-derives to other content | Writer outcome `STORED_ISSUANCE_INVALID` (EI-5 design, section 8.3): nothing written; the stored row is preserved and never treated as equivalent |
| Any other stored issuance or declaration, or a ledger inconsistency, met during a write | The D6-5 typed permanent error; rolled back; nothing written |
| Any corrupt row or ledger inconsistency met during a read (`load_issuance`, `load_session_issuance`, `session_view`, `session_view_at`) | The D6-5 typed permanent error |

**Application contract (the D6-5 prerequisite PR).** One small northstar-application PR, merged **before** the Infrastructure adapter imports it:

- adds one typed error to the issuance port module for corrupt or non-re-deriving stored issuance-authority state, covering a stored issuance that does not re-derive and a ledger or declaration inconsistency, following the existing Application naming conventions (the class name is chosen and reviewed in that PR, not frozen here);
- classifies it as a **permanent** failure, never as the retryable `EquityRecommendationIssuanceUnavailableError` outage;
- maps it through the issuer: a permanent refusal before any write; and on a commit-uncertainty reload under the request's own identity, `STORED_ISSUANCE_INVALID`, as that outcome is governed today;
- maps it through the declarer's reconciliation and through `AuthoritativeEquitySessionRecommendationHistorySource`, so the failure stays distinguishable from a temporary outage within the existing history-port conventions;
- preserves every existing retryable and permanent distinction, and the existing outcomes.

It introduces no new identity, format, outcome semantics or write behaviour. No Core change is authorized.

## 10. Reads and historical views

| Method | SQLite behaviour |
| --- | --- |
| `load_issuance(issuance_id)` | The re-derived recorded issuance (section 8), or `None` |
| `load_session_issuance(strategy, listing, trading_date)` | The same, by the session uniqueness key, or `None` |
| `session_view(exchange, trading_date)` | The view at the current position `N` (the ledger maximum; 0 when empty) |
| `session_view_at(exchange, trading_date, watermark)` | The view at `watermark`. A watermark beyond `N` raises `EquityIssuanceWatermarkBeyondAuthorityError` |

Every view is built by the Application's pure `equity_issuance_session_view` from records loaded in one read transaction, never by SQL ordering alone:

- **Reference.** Every view carries `northstar.equity-recommendation-issuance-authority/1` (D5D-3).
- **Undeclared.** No declaration for the session at or below the watermark returns `None`: unknown, never an authoritative empty history (HW-01).
- **Declared, nothing visible.** A declaration with no visible issuance returns an empty recommendation tuple (HW-02).
- **Content.** Issuances recorded at or below the watermark with `issued_at` at or before `CT(W)`, in (`issued_at`, key) order (EI-5 design, section 8.2, rule 5).
- **Deterministic.** Views depend only on committed rows, so they are identical after a restart and from another process (RR-03, RR-04 views).
- **Read-only.** Reading never writes, repairs, compacts or rewrites a record.

## 11. Reliability

- **Cleanup.** The guarded close, rollback and discard pattern of the L9 fixes on every exit path.
- **Test seams.** The EI-4D conventions for tests that plant rows or bypass triggers are reused. Seams are test-only and not reachable from a supported path.

## 12. Acceptance

**Status for every row: PLANNED.** A row is VERIFIED only when executed and recorded in an EI-6A acceptance record. Documentation is not evidence.

### 12.1 EI-6A rows of the EI-5 matrix

| ID | Scenario (EI-5 design, section 12) | Layer | Status |
| --- | --- | --- | --- |
| CC-01 | Two processes issue the same disposition: one row; the other `EQUIVALENT` | S | PLANNED |
| CC-02 | Two processes issue different policy sets for one strategy, listing and session: exactly one `ISSUED` | S | PLANNED |
| CC-03 | An issuance concurrent with a declaration covering its `issued_at`: listed, or refused as backdated; never silently outside the view | S | PLANNED |
| CC-04 | Concurrent declarations: monotone; regression refused | S | PLANNED |
| CC-05 | Write lock not acquired: nothing written; typed unavailable | S | PLANNED |
| CC-06 | COMMIT raises after taking effect: reload by identity resolves it as committed | S | PLANNED |
| RR-03 | Issuances and declarations after restart: identical views at every watermark | S | PLANNED |
| RR-04 (views) | Replay of views from a separate OS process: identical views. The lifecycle-state part is EI-6B | S | PLANNED |
| ID-02 (S) | Identity identical across two spawned processes and a restart, on real SQLite | S | PLANNED |
| ID-03 (S) | `GLOB 'eiris1-sha256:?*'` enforced; an `eiri1-sha256:` identity refused | S | PLANNED |
| BD-01 (S) | `issued_at` 10:10, declared through 10:30: `BACKDATED_ISSUANCE` from the writer, and refused by the trigger when the writer is bypassed | S | PLANNED |
| HW-03 (S) | The A, B, declaration, C example of EI5-D4: the view at watermark 3 lists exactly A and B, before and after C, durably | S | PLANNED |
| HW-08 (S) | Authority unreadable: `EquityRecommendationHistoryUnavailableError` through the history source | S | PLANNED |

### 12.2 EI-6A storage scenarios (SA)

Derived from ADR-024 section 13, the EI-5 design section 10 acceptance list, and D6-1 to D6-6.

| ID | Scenario | Expected | Layer | Status |
| --- | --- | --- | --- | --- |
| SA-01 | Insert, bypassing the writer, an issuance whose disposition is missing, not `PROPOSED`, evaluator `/1`, or differs in a fingerprint or session column | Refused by the foreign key or the provenance trigger | S | PLANNED |
| SA-02 | Write with a missing, foreign, unissued or changed authorization | `UnverifiedEquityRecommendationIssuanceWriteError`; no connection opened; database unchanged | S | PLANNED |
| SA-03 | UPDATE, DELETE and replacing INSERT on each of the three tables | Refused; rows unchanged | S | PLANNED |
| SA-04 | A stored issuance column tampered in a test | Reads raise the D6-5 typed error; a write of that identity gives `STORED_ISSUANCE_INVALID`; the row is preserved and never repaired | S | PLANNED |
| SA-05 | The referenced disposition made undecodable in a test | Loading the issuance raises the D6-5 typed error; nothing repaired | S | PLANNED |
| SA-06 | Ledger: a duplicate sequence, a non-next sequence, a kind mismatch, and a record without its ledger entry | Each refused by the database | S | PLANNED |
| SA-07 | Ledger: an orphan entry planted in a test | Every watermark-establishing read and every write fails closed with the D6-5 typed error | S | PLANNED |
| SA-08 | A failure injected after the ledger insert and before COMMIT | Nothing committed; the next write takes the same sequence; no gap | S | PLANNED |
| SA-09 | COMMIT raises without taking effect; and the reload also fails | Bounded retries, then the use case's unresolved error; nothing claimed | S | PLANNED |
| SA-10 | Views on SQLite: undeclared; declared with no visible issuance; watermark beyond the position | `None`; empty tuple; `EquityIssuanceWatermarkBeyondAuthorityError` (durable evidence for the EI-5D HW-07 limitation) | S | PLANNED |
| SA-11 | Equal and lower declarations; a lower declaration inserted bypassing the writer | `DECLARATION_EQUIVALENT` and `COMPLETENESS_REGRESSION` with no ledger entry; the bypassed insert refused | S | PLANNED |
| SA-12 | Initialization on an EI-4D database; again; and on a database without the EI-4D tables | Only EI-6A objects created, EI-4D tables and rows unchanged; idempotent; refused | S | PLANNED |
| SA-13 | Integer-microsecond ordering, for example `…:00.5Z` against `…:01Z`, in the backdating and monotonicity triggers | Chronological, not textual, comparison | S | PLANNED |

### 12.3 Not EI-6A

RR-01, the lifecycle-state part of RR-04, RR-05 and RR-06 belong to **EI-6B**. RV-04 stays DEFERRED (O-3).

### 12.4 Required evidence

- Tests on real SQLite files, with spawned OS processes for CC-01 to CC-04, ID-02 and RR-04 (views).
- The full Infrastructure suite; the full Application suite for the D6-5 PR; Ruff check and format; the architecture scans; `git diff --check`.
- A targeted mutation check of at least: sequence allocation, the backdating comparison (writer and trigger), identity-before-backdating ordering, and the provenance trigger.

## 13. Mandatory CI evidence (D6-6)

Final EI-6A owner acceptance requires **both**:

1. **Local Windows real-SQLite engineering acceptance**, with the Infrastructure and Application commit identities the checks ran against.
2. **Independently verified successful Linux CI** with the existing `python-ci.yml` workflows (`ruff check`, `ruff format --check`, `pytest` on `ubuntu-latest`):
   - northstar-infrastructure, at the actual EI-6A Infrastructure PR revision;
   - northstar-application, at the actual D6-5 prerequisite PR revision.

For each run the acceptance record states:

- the repository and commit;
- the run identifier and conclusion;
- the Core and Application commits the run checked out (the workflows check out their dependencies' `develop` at run time, so the Infrastructure run must show an Application commit that contains the merged D6-5 PR);
- that the EI-6A acceptance tests ran and passed, not skipped.

A passing local suite alone, or a merged PR alone, does not satisfy this gate. If the CI evidence is unavailable, engineering completion may be reported, but **EI-6A owner acceptance stays PENDING**.

This gate does not waive or satisfy EI4-C4. Formal EI-4 closure keeps its own requirements ([EI-4 Decision Register, section 15.3](../product/Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4)).

## 14. Limitations and carried-forward restrictions

- **Trust model.** Personal-use, single-host SQLite. No protection against direct SQL, file modification or privileged code (ADR-024 section 14). The database guards are defence in depth.
- **Requirement 4** of section 4.3 is enforced by the transaction and fail-closed checks; it is not a database constraint.
- **VC-04.** Rollback or downgrade to a pre-3a Application against stored evaluator-v2 records stays **NOT AUTHORIZED**. EI-6A rows reference evaluator-v2 dispositions, which makes this restriction more consequential.
- **N6.** The database-level label-format pairing trigger stays **DEFERRED**.
- **I08.** The equivalent-mutant qualification stays.
- **EI4-D8.** Every issued same-session recommendation is still treated as open; at most one issuance per (strategy, listing, trading date) remains the effective rule.
- **EI-5D and EI-5E limitations.** IS-16 and NB-04 stay. HW-07 stays a limitation until durable-reader evidence (SA-10) is accepted.
- **EI-4.** Formal CI closure stays pending (EI4-C4).
- **Historical evidence.** No historical EI-4D evidence is reconstructed or modified.

## 15. Explicit non-authorizations

This design does not authorize EI-6B, EI-6C, the resolution of O-2, O-3 or EI3-D10, ACTUAL issuance, orders, fills, real-money trading, production deployment, API or Web exposure, composition roots or monitoring execution, any relaxation of EI4-D8, any new canonical issuance format, or any change to historical EI-4D evidence. EI-6A is single-host and MODELED-only.

## 16. Related documents

- [EI-5 Decision Register, section 17](../product/Equity-Intraday-V1-EI5-Decision-Register.md#17-owner-authorization-of-ei-6a-2026-11-14)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design v1.0](Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [EI-5D/EI-5E Engineering Acceptance Record](../product/Equity-Intraday-V1-EI5D-EI5E-Modeled-Issuance-and-Synthetic-Integration-Engineering-Acceptance-Record.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)
- [ADR-025: Equity Intraday V1 Versioned Evaluation and History Compatibility](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md)
- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md)
