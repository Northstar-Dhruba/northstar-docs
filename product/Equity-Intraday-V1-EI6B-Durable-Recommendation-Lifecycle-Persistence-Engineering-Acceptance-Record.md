# Equity Intraday V1 — EI-6B Durable Recommendation Lifecycle Persistence Engineering Acceptance Record

## Status

**Engineering acceptance record, 2026-11-15.** This is Docs PR-3 of the EI-6B plan authorized in the [EI-5 Decision Register, section 19](Equity-Intraday-V1-EI5-Decision-Register.md#19-owner-authorization-of-ei-6b-2026-11-14).

**EI-6B — ACCEPTED WITH DOCUMENTED LIMITATIONS, 2026-11-15 (owner).** See [section 17.1](#171-owner-acceptance-2026-11-15).

> **Note, 2026-11-15 (D6B-7 evidence and owner acceptance).** The headline and bullets below record the state at engineering acceptance (Docs PR-3, northstar-docs #67) and are not rewritten. Since then:
>
> - **Linux CI recorded.** Successful Linux runs exist for northstar-application #98 (run `43623021952`) and northstar-infrastructure #68 (run `43643943540`) ([section 15.3](#153-verified-linux-ci-evidence-2026-11-15)). **D6B-7: SATISFIED**, with the workflow-identity qualification stated there.
> - **Skip counts reconciled.** Windows 3,412 passed and 27 skipped; Linux 3,415 passed and 24 skipped; 3,439 collected on both ([section 15.4](#154-windows-and-linux-skip-reconciliation)).
> - **Reclassified.** The five rows limited only by Windows-only separate-process evidence are VERIFIED. The matrix is now **20 VERIFIED, 2 VERIFIED WITH LIMITATION (RR-05, LS-17), 0 NOT VERIFIED** ([section 13.4](#134-reclassification-after-linux-ci-2026-11-15)).
> - **Limitations reconciled** in [section 16.3](#163-limitation-reconciliation-2026-11-15).
> - **Owner decision.** The owner accepted EI-6B with documented limitations on 2026-11-15 ([section 17.1](#171-owner-acceptance-2026-11-15)).
> - **Unchanged.** EI-6C and EI-8 stay NOT AUTHORIZED; O-2 and EI3-D10 stay unresolved.

**EI-6B — ENGINEERING IMPLEMENTATION MERGED; LOCAL ACCEPTANCE RECORDED; D6B-7 LINUX CI EVIDENCE PENDING; FINAL OWNER ACCEPTANCE PENDING.**

- **Verdict:** EI-6B LOCAL ENGINEERING ACCEPTANCE: **PASS WITH DOCUMENTED LIMITATIONS — synthetic, local Windows, Application contract and real-SQLite engineering acceptance** ([section 17](#17-engineering-verdict-and-milestone-status)).
- **What it covers:** EI-6B — Durable Equity Recommendation Lifecycle Persistence, merged as northstar-application #98 and northstar-infrastructure #68 ([section 4](#4-repository-baselines)).
- **Acceptance matrix:** 22 governed rows — **15 VERIFIED, 7 VERIFIED WITH LIMITATION**, none partially verified, not tested or blocked ([section 13](#13-acceptance-evidence-matrix)). Every row rests on local Windows execution only.
- **Commit-uncertainty retry review:** **COMPLIANT WITH APPROVED POLICY** ([section 10](#10-commit-uncertainty-safety-review)). No acceptance requirement is blocked by it.
- **D6B-7 independent Linux CI: PENDING for both PRs** — no run evidence was supplied or is accessible in this environment ([section 15.2](#152-d6b-7-independent-linux-ci)). This is a hard owner-acceptance gate.
- **Owner acceptance: PENDING.** EI-6B cannot be owner-accepted until both Linux runs are independently verified, and then only by a separate owner decision.
- **Not closed.** Sections 1 to 18 are engineering evidence, **not** an owner approval. D6B-1 to D6B-7, O-3 option (b) and every earlier EI-5, ADR-024 and ADR-025 decision are unchanged. Nothing here authorizes EI-6C, EI-8, ACTUAL issuance or any production use ([section 18](#18-non-authorizations)).

## 1. Acceptance identity

| Item | Value |
| --- | --- |
| Milestone | EI-6B — Durable Equity Recommendation Lifecycle Persistence |
| Record | Docs PR-3, engineering acceptance and evidence reconciliation |
| Date | 2026-11-15 |
| Governance | [EI-5 Decision Register, section 19](Equity-Intraday-V1-EI5-Decision-Register.md#19-owner-authorization-of-ei-6b-2026-11-14) (D6B-1 to D6B-7, clarifications 1 to 3); [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), sections 8 to 14 (section 13 amended 2026-11-14) |
| Design | [EI-6B Durable Recommendation Lifecycle Persistence Design v1.0](../architecture/Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md); [EI-5 Issuance and Lifecycle Design v1.0](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md), sections 9, 10 and 12 |
| Data | Synthetic evidence only; MODELED issuances only |
| Authority | The SQLite lifecycle-event table, in the same file as the EI-4D generation records and the EI-6A issuance authority; single host |
| Platform | Windows development machine; local executions only |

## 2. Scope

**In scope:**

- **Application (#98):** the lifecycle-store port and errors; the canonical event codec `/1`; the authoritative window derivation; the content-bound append authorization and the shared append rule; `RecordEquityRecommendationLifecycleUseCase`; a labelled in-memory test store; the C, E and A tests.
- **Infrastructure (#68):** the additive lifecycle-event schema; `SQLiteEquityRecommendationLifecycleStore`; parent provenance; compare-and-append; rehydration and Core replay; restart, fresh-process and spawned-process acceptance; the S tests.

**Out of scope:** EI-6C (monitoring, snapshot selection, a monitoring cursor), O-2, EI3-D10, EI-8 restatement, composition roots, ACTUAL issuance, orders, fills, positions, brokers, API, Web, real data, any Core change, and any change to EI-4D records or the EI-6A authority.

## 3. Governance references

| Layer | State | Where |
| --- | --- | --- |
| Owner authorization | D6B-1 to D6B-7 and clarifications 1 to 3 (2026-11-14); EI-6C not authorized | [EI-5 Decision Register, section 19](Equity-Intraday-V1-EI5-Decision-Register.md#19-owner-authorization-of-ei-6b-2026-11-14) |
| O-3 | Resolved, option (b); specification section 8.2 deferred to EI-8 | [EI-5 Decision Register, section 19.3](Equity-Intraday-V1-EI5-Decision-Register.md#193-d6b-2--o-3-resolved-option-b) |
| Binding design | The EI-6B addendum: format `/1` (section 3), port (4), authorization (5), window (6), use case (7), SQLite store (8), O-3 (9), concurrency (10), 22 scenario rows (12), D6B-7 (13), exclusions (16) | [EI-6B design addendum](../architecture/Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md) |
| Parent authority | EI-6A, accepted with documented limitations (2026-11-14); not reopened | [EI-6A Engineering Acceptance Record](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md) |
| Implemented behaviour | northstar-application #98; northstar-infrastructure #68 | [Sections 5 to 12](#5-implementation-inventory) |
| Local evidence | Local Windows runs | [Section 15.1](#151-local-results) |
| Independent CI | Pending | [Section 15.2](#152-d6b-7-independent-linux-ci) |
| Limitations | E1 to E9 (EI-6B), carried-forward C1 to C10 | [Section 16](#16-limitations-and-verification-gaps) |

## 4. Repository baselines

Read from local Git on 2026-11-15. Every repository was on `develop` with a clean working tree; no fetch was run.

| Repository | `develop` HEAD | Role |
| --- | --- | --- |
| northstar-application | `b7df16b` (#98; merge `b7df16b8f3348672627b4f692660d9fa844e03ef`, parents `a17bd91`, `82f872d`; change commit `82f872d34a5fc3266046118d8f504e0a43935bcc`) | EI-6B contracts, codec, window, authorization, recording use case and C, E, A tests |
| northstar-infrastructure | `dd6a434` (#68; merge `dd6a434368bfa7124f33381c38bc52c65ab4efde`, parents `63d2ab3`, `ac66907`; change commit `ac66907a71ffd220afb5b8bc55a9292d2605f396`) | EI-6B schema, adapter and S tests |
| northstar-docs | `20601e7` (#66; EI-6B authorization change commit `2d193e8`) | EI-6B authorization and addendum; this record, uncommitted |
| northstar-core | `29bd34a` (#40) | Unchanged |
| northstar-api | `7ad37f5` | Unchanged |
| northstar-web | `5aeb94c` | Unchanged |

**Change scope:**

- **`82f872d`:** 9 files, 2,682 insertions, 17 deletions — 6 new files (one port module, four Application service modules, one test module) and 3 modified (two package export modules, and the EI-5C evaluator's window delegation).
- **`ac66907`:** 5 files, 2,717 insertions, 0 deletions — two new production modules, one export change, one new fixtures module and one new test module. No existing table, trigger, test or module is modified.

Infrastructure resolves the Application and Core as editable path dependencies (`../northstar-application`, `../northstar-core`), so local Infrastructure runs used the Application at `b7df16b` and Core at `29bd34a`.

## 5. Implementation inventory

### 5.1 Application (#98)

Paths are under `northstar-application/src/northstar_application/`.

| Component | File | Responsibility |
| --- | --- | --- |
| Port | `ports/equity_recommendation_lifecycle_store.py` | `EquityRecommendationLifecycleStore` (`load_lifecycle`, `append`), `EquityRecordedLifecycle`, `EquityRecommendationLifecycleWrite`, the outcome enum and five typed errors (lines 69–201) |
| Codec | `application_services/equity_recommendation_lifecycle_events.py` | `equity_lifecycle_event_document(s)`, strict `decode_equity_lifecycle_event_document` (line 250), the shared rehydration `equity_recorded_lifecycle` (line 285) |
| Window | `application_services/equity_recommendation_lifecycle_window.py` | `equity_lifecycle_window` (grid, shared with EI-5C, line 59) and `equity_issued_lifecycle_window` (line 80) |
| Authorization | `application_services/equity_recommendation_lifecycle_authorization.py` | `VerifiedEquityRecommendationLifecycleAppend` (line 149), private `_issue_lifecycle_append` (183), `require_verified_equity_recommendation_lifecycle_append` (195), `equity_lifecycle_append_decision` (225) |
| Use case | `application_services/record_equity_recommendation_lifecycle.py` | `RecordEquityRecommendationLifecycleUseCase` (line 226); bounded reconciliation `_write` (400–444) |
| EI-5C | `application_services/evaluate_synthetic_equity_recommendation_lifecycle.py` | `_lifecycle_window` delegates to the shared grid function; same refusal and message |
| Tests | `tests/application_services/test_record_equity_recommendation_lifecycle.py` | 32 test functions, 52 cases |

**Not in the Application:** any SQLite persistence. The Application's only store is the labelled in-memory test store inside its test module.

### 5.2 Infrastructure (#68)

Paths are under `northstar-infrastructure/`.

| Component | File | Responsibility |
| --- | --- | --- |
| Schema | `src/northstar_infrastructure/persistence/sqlite_equity_recommendation_lifecycle_schema.py` | The table, its CHECKs and 8 triggers; `initialize_equity_recommendation_lifecycle_schema` (line 216), which refuses a database without the EI-6A issuance tables |
| Adapter | `src/northstar_infrastructure/persistence/sqlite_equity_recommendation_lifecycle.py` | `SQLiteEquityRecommendationLifecycleStore` (line 365): `load_lifecycle`, authorized `append`; `_parent` (218), `_window` (249), `_stored_lifecycle` (259), `_write` (451) |
| Exports | `src/northstar_infrastructure/persistence/__init__.py` | `SQLiteEquityRecommendationLifecycleStore`, `initialize_equity_recommendation_lifecycle_schema` |
| Fixtures | `tests/equity_recommendation_lifecycle_fixtures.py` | The real chain over one file; the test-only `AppendCapture`, `KeepingLifecycleStore` and `BarrierLifecycleStore`; spawned workers |
| Tests | `tests/test_sqlite_equity_recommendation_lifecycle.py` | 46 test functions, 76 cases (the PR-2 report said 47 functions; 46 is the count in the merged file) |

**Ownership.** The adapter calls the Application's validator, codec, rehydration, window derivation and append rule unchanged, and Core replays every history. The adapter owns durability, transactions, locking, rehydration from rows and the database guards. It contains no state machine and computes no window, transition or outcome of its own (`test_ar_the_lifecycle_modules_reuse_the_application_and_reach_nothing_else`, `test_ls17_d_e_…`).

## 6. Canonical format and contracts

| Requirement (addendum section 3) | Evidence |
| --- | --- |
| Exactly 14 keys; exact evidence and deficiency key sets; no issuance identity; no fingerprint | Codec source; A `test_ls09_every_core_event_kind_has_a_frozen_canonical_document`; S `test_e2e_the_real_sqlite_vertical_slice` (every stored document has exactly the 14 keys) |
| Golden vector per kind | A `test_ls09_every_core_event_kind_…` (11 kinds, frozen digests, one document pinned byte for byte); `test_ls09_the_golden_documents_reproduce_in_a_fresh_interpreter` |
| Strict decoding, byte-identical re-encoding, nothing normalized | A `test_ls09_strict_decoding_refuses_…` (21 refusal cases); S `test_ls09_…` (12 stored corruptions) |
| Stored bytes are exactly the Application encoding | S `test_ls01_ls02_a_…`: each row equals `lifecycle_event_row(issuance_id, event, document)` and each document equals `equity_lifecycle_event_documents` of the replayed events |

**Kinds produced through the real SQLite chain:** `ENTRY_WINDOW_OPENED`, `ENTRY_ACTIVATED`, `TARGET_REACHED`, `ENTRY_CANCELLED`, `EVIDENCE_INCOMPLETE` and `SOURCE_REVISION_RECORDED`. The other five kinds are covered by the Application golden vectors and strict-decoding tests, not by persisted real-chain rows.

## 7. Storage, provenance and authorized append

### 7.1 Schema as implemented

**Table `equity_recommendation_lifecycle_events`:** `issuance_id` (shape `eiris1-sha256:?*`, **foreign key** to `equity_recommendation_issuances (issuance_id)`); `event_sequence` (integer ≥ 1; **primary key (`issuance_id`, `event_sequence`)**); `event_format` (= `/1`); `event_kind`, `from_state`, `to_state` (the 11 kinds and 11 states; the state changes exactly when the kind is not `SOURCE_REVISION_RECORDED`); `evaluated_at`, `evaluated_at_us`; `evidence_reference`, `evidence_opens_at`, `evidence_closes_at`; `event_document` (`json_valid`). A guarded CHECK ties the document's format, sequence, kind, states, evaluated instant and evidence reference and interval to the columns.

**Triggers (8):** `no_update`, `no_delete`, `no_replace`, `next_sequence` (issuance maximum + 1), `canonical_instant` (three canonical instants; the integer equals its integer-only derivation, reused from EI-6A), `chronology` (not before the issuance's latest event), `not_before_issuance` (not before the parent's `issued_at_us`), `terminal` (no state-changing kind after the latest state-changing event entered a terminal state; revisions pass to Core).

**Not checked by SQL**, as the addendum section 8.3 states: event shapes beyond the columns, the window, the cursor, interval ranges and revision citations. **The database does not enforce Core's lifecycle rules;** Core replay does, on every load and append. SQLite stores no authoritative lifecycle state.

### 7.2 Storage and provenance evidence

| Requirement | Evidence | Result |
| --- | --- | --- |
| Additive, same file, idempotent, no version | S `test_ls16_the_schema_is_additive_…` | Only the table and 8 triggers added; every existing object and the EI-4D and EI-6A rows unchanged; idempotent; `user_version` unchanged; EI-6A still declares and loads afterwards |
| A database without EI-6A, a foreign database, a missing file | Same test | Refused as unavailable; nothing created; a missing file is never created |
| Foreign key | S `test_ls04_a_…` | An event for a non-issued identity refused (`FOREIGN KEY`) |
| EI-6A parent re-derivation on load and append | Adapter `_parent` (lines 218–246): the EI-6A whole-ledger check, `_stored_issuance` (re-derivation from the EI-4D disposition with every column compared), EI-4D `_stored_disposition` (level A) | S `test_ls04_b_…` (two tests): a tampered issuance raises D6-5 `ISSUANCE`, an orphan ledger entry D6-5 `LEDGER`, on load and append; the recorder refuses `STORED_AUTHORITY_INVALID`; nothing written |
| A matching identifier alone is not authority | S `test_ls04_c_…` | The same identity recorded at sequence 3 in another file refuses an authorization for sequence 2; a file without the issuance refuses it; both write nothing |
| Window from authoritative provenance | Adapter `_window` (249) calls `equity_issued_lifecycle_window` | S `test_ls17_a_b_…` (section 13, LS-17) |
| No authoritative stored state | Load replays every row through Core; no state column exists | S `test_ls01_ls02_a_…`, `test_ls16_rr01_rr04_…` |

**Level of source verification on load.** As for EI-6A (its limitation L7), the store decodes the referenced disposition at level A and re-derives the issuance; levels A to C run in the recording use case before any authorization is issued.

### 7.3 Append authorization

| Requirement (addendum section 5) | Evidence |
| --- | --- |
| Application-owned, issued only by the recording use case, private weak registry | Authorization module; A `test_ar_only_the_recording_use_case_depends_on_the_store_or_issues_authorizations` |
| Content binding: issuance identity, full issued record, recording sequence, window, prior and new documents, position | `_check` (lines 99–146); A `test_ls05_unauthorized_or_changed_appends_…` |
| Validation before any connection | `append`'s first statement (line 411); S `test_ar_append_validates_first_and_is_the_only_insert` |
| Mutation and forgery refusal, zero connections | S `test_ls05_…` (15 cases): none, plain object, raw dictionary, raw canonical document, raw Core events, built directly, copied, `dataclasses.replace`, and changed prior events, new events, nested evidence, parent, window, sequence position and covered documents. A spy on `_connect` records **no call** and the file hash is unchanged; a genuine authorization then connects once |
| No raw production append | Only `append` uses the insert statement; the only public function is the pure `lifecycle_event_row`; no production module constructs the adapter (S `test_ar_…`, 3 tests) |

## 8. Atomicity, sequence integrity and chronology

| Requirement | Evidence | Result |
| --- | --- | --- |
| `BEGIN IMMEDIATE` | Adapter `_write` (line 460) | One write transaction; checks, decision, inserts and confirmation under one lock |
| Atomic batch: a row refused mid-batch | S `test_ls11_a_row_refused_mid_batch_…` | A TEST-ONLY trigger refuses row 2 of a 3-event batch: nothing persists; the next append takes 1..3 |
| Failure after inserts, before COMMIT | S `test_ls11_a_failure_after_the_inserts_…` (2) | The 3 rows are visible inside the transaction, none afterwards (`RuntimeError`; SQLite error mapped to unavailable) |
| Process ends before COMMIT | S `test_ls11_a_process_that_ends_before_commit_leaves_nothing` | A spawned process exits at the commit seam (exit code 17): no rows, clean `integrity_check`, next append 1..3 |
| Bad later row in a raw batch | S `test_ls11_raw_batches_…` | Gap, duplicate and decreasing batches refused whole; the same inconsistent batch cannot be authorized (Core) |
| Per-issuance continuity | S `test_ls07_sequences_…` | Duplicate, gap and sequence 0 refused; a second issuance (INFY) has its own sequence 1; the EI-6A ledger is unchanged by lifecycle appends |
| Integer-microsecond chronology | S `test_ls08_…` | One microsecond before the parent's `issued_at` refused, exactly at it admitted; one microsecond before the latest event refused, equal admitted; `…:30.5Z` after `…:30Z` admitted where text order would refuse (both guards); integer disagreeing with text and five non-canonical texts refused |
| Insert-only | S `test_ls06_…` | Two UPDATEs and two DELETEs refused; five replacing inserts (`INSERT`, `INSERT OR REPLACE`, `REPLACE`, `INSERT OR IGNORE`, upsert) refused, and refused again by `no_replace` alone with the overlapping triggers dropped in a rolled-back transaction; rows unchanged |
| Document/column consistency | S `test_ls10_a_column_disagreeing_…` | Seven disagreeing or malformed values refused by CHECK; four missing values refused (NOT NULL, CHECK or the canonical-instant trigger) |

All raw probes run in transactions that are rolled back, except labelled TEST-ONLY plants.

## 9. Concurrency and idempotence

| Case | Evidence | Result | Platform |
| --- | --- | --- | --- |
| First append | S `test_ls01_ls02_a_…`; A `test_ls01_…` | `APPENDED`; sequences 1..n | In-process |
| Identical retry | S `test_ls13_an_identical_retry_…`; A `test_ls13_…` | `EQUIVALENT` twice, from two new adapter instances; nothing inserted | In-process |
| Identical concurrent writers | S `test_ls13_rr06_two_processes_…` | One `APPENDED`, one `EQUIVALENT`; identical documents; no duplicate; a later process sees `NO_NEW_EVENTS` | Spawned, Windows |
| Conflicting concurrent writers | S `test_ls14_rr06_two_processes_…` | One `APPENDED`, one `CONFLICT`; only the winner's documents persist; the stored state is the winner's | Spawned, Windows |
| Stale authorization | S `test_ls14_a_stale_writer_…`; A `test_ls14_rr06_…` | `CONFLICT`; never adopted, also after the history advanced further | In-process |
| Same-length divergent history | S `test_ls14_a_stale_writer_…`; A `test_ls14_a_prior_history_of_the_same_length_…` | `CONFLICT`, not decided by count | In-process |
| Separate issuances | S `test_ls07_two_processes_…` | Both `APPENDED` at sequence 1; no cross-contamination | Spawned, Windows |
| Lock not obtained | S `test_a_write_lock_not_obtained_…` | Typed unavailable; recorder `STORE_UNAVAILABLE`, retryable; reads continue | In-process |
| Fresh-process replay | S `test_ls16_rr01_rr04_…`; `test_e2e_…` | Identical identity, sequence, issued content, state, sequences, references, documents and window | Fresh interpreter, Windows |

**Synchronization.** Spawned workers build the real recorder from the file path alone; a test-only barrier inside `append` guarantees both hold authorizations over the same prior history. The winner is the first committed, not assumed by the test.

**Platform.** All separate-process evidence is Windows only (`spawn`). Linux evidence is pending (section 15.2).

## 10. Commit-uncertainty safety review

**Classification: COMPLIANT WITH APPROVED POLICY.** No blind retry occurs: a second write is made only after an authoritative reload proves the batch is not committed, and the retry itself is idempotent.

### 10.1 The approved policy

- Addendum section 7, step 8: on commit uncertainty, reload (bounded by `max_writes`): stored equals prior plus new → `EQUIVALENT`, reconciled; **stored equals prior → retry**; anything else → `CONFLICT`; reload failure or bound exhausted → unresolved, nothing claimed; corruption → permanent.
- Addendum section 10 and clarification 1: "`EQUIVALENT` only after confirming the exact events are committed; otherwise retry within the bound, or the typed commit-unconfirmed or unresolved result. No blind retry."

### 10.2 What the code does

**Adapter** (`sqlite_equity_recommendation_lifecycle.py`, `_write`, lines 451–487): when COMMIT raises, the connection is rolled back (line 479) and closed (line 484) **before** `EquityRecommendationLifecycleCommitUnconfirmedError` reaches the caller. Once closed, the uncertain transaction is resolved: it either committed and is durable, or did not commit and can no longer commit.

**Use case** (`record_equity_recommendation_lifecycle.py`, `_write`, lines 400–444), on `CommitUnconfirmed`:

1. Reload through a new connection (`load_lifecycle`, line 417). Unavailable → `EquityLifecycleRecordingUnresolvedError`; corruption → the permanent error, unchanged.
2. Stored documents **exactly equal to the authorized prior** (line 430) → write again (line 431), within `max_writes`.
3. Stored begins with exactly prior plus the new events → `EQUIVALENT`, reconciled (lines 432–440).
4. Anything else → `CONFLICT`, reconciled.
5. Bound exhausted → unresolved (line 442).

### 10.3 Why case 3 writes again, and why it is safe

- **Authoritative evidence of non-commit.** The retry happens only when a fresh read of the durable store shows the stored history is exactly the prior history. A missing acknowledgement alone never triggers it.
- **Idempotent retry.** The retry re-submits the **same** content-bound authorization through the adapter's compare-and-append under `BEGIN IMMEDIATE`. Had the batch somehow committed, the store would return `EQUIVALENT` (stored equals prior plus events), never a duplicate. A concurrent writer in between yields `CONFLICT`. Identical-retry idempotence is evidenced by S `test_ls13_an_identical_retry_…` and the spawned identical race.
- **Bounded.** At most `max_writes` (default 3) writes.

### 10.4 Evidence

| Case | Application (C) | Infrastructure (S) | Result |
| --- | --- | --- | --- |
| Acknowledgement lost after commit | `test_ls12_an_acknowledgment_lost_after_commit_…` | `test_ls12_a_lost_acknowledgement_…` | `EQUIVALENT`, reconciled, 1 write, exactly one row |
| Later writer extends the committed prefix | `test_ls12_a_reload_showing_our_events_then_later_ones_…` | `test_ls12_a_later_writer_…` | `EQUIVALENT` by prefix; the store's own rule stays exact (the same authorization then gives `CONFLICT`) |
| Commit lost once, before effect | `test_ls12_a_commit_lost_before_effect_…` | `test_ls12_a_commit_lost_before_effect_…` | The reload shows exactly the prior; the second write is `APPENDED` (writes = 2); one row |
| Commit always lost | Same | `test_ls12_unresolved_…` | Unresolved after 3 writes, each reload showing the prior; nothing stored |
| Reload unavailable | `test_ls12_an_unreadable_reload_…` | `test_ls12_unresolved_…` | Unresolved; nothing claimed |
| Conflicting committed history | — | `test_ls12_a_conflicting_history_found_on_reload_is_conflict` | `CONFLICT`, reconciled |
| Corruption on reload | `test_ls12_an_unreadable_reload_or_corruption_on_reload_…` | `test_ls12_corruption_met_on_reload_is_permanent` | Permanent `EVENT` error; nothing claimed |

### 10.5 Qualifications

- **Single host.** The non-commit proof relies on SQLite resolving a transaction when its connection is rolled back and closed, under ADR-024 section 14's single-host model.
- **Close failure.** `_discard` suppresses a close error. If a connection then stayed open, its transaction stays uncommitted (nothing calls COMMIT on it again), and its lock makes the retry fail as unavailable rather than duplicate.
- **Test seams.** The cases use the module-level `_commit` and `_connect` seams (limitation E6).

## 11. Canonical rehydration and corruption

**Load path** (`_stored_lifecycle`, lines 259–288): sequences exactly `1..n` (`HISTORY`); each document strictly decoded (`EVENT`); its columns compared with types (`EVENT`); `equity_recorded_lifecycle` replays through Core (`HISTORY`). Nothing is normalized, repaired, skipped or rebuilt from columns.

Every Infrastructure corruption test asserts, through one helper: load raises the permanent error with its category and `retryable` False; append raises it too, never `EQUIVALENT` or `CONFLICT`; the recorder refuses `STORED_LIFECYCLE_INVALID` (not retryable); and the stored rows are byte-identical afterwards.

| Category | Real-SQLite planted cases | Evidence |
| --- | --- | --- |
| `EVENT` | Whitespace, unsorted keys, extra key, missing key, duplicate key, boolean for integer, non-canonical instant, microsecond column disagreeing; and, refused by CHECK first and then planted with CHECKs off: unsupported format, unknown kind, float sequence, column disagreeing with its document | S `test_ls09_…` (12) |
| `HISTORY` | Sequence gap; repeated `ENTRY_WINDOW_OPENED`; transition from the wrong state; decreasing instants; state change after a terminal state | S `test_ls10_a_stored_history_…` (5); `test_rr05_a_tampered_…` (a shifted evidence interval) |
| `WINDOW` | **Test seam only** (section 16, E2) | S `test_ls17_c_…` |
| EI-6A parent (D6-5) | Tampered issuance (`ISSUANCE`); orphan ledger entry (`LEDGER`) | S `test_ls04_b_…` (2) |

Corruption is never returned as `None`, an empty lifecycle, a retryable outage, `EQUIVALENT` or `CONFLICT`.

### 11.1 RR-05 qualification

**Governing requirement** (EI-5 matrix and addendum 12.1): "A stored event tampered in a test: refused on reload; preserved."

- **Demonstrated:** tampering that makes a document malformed or non-canonical, makes it disagree with its columns, or makes the history Core-invalid is refused on reload with a permanent typed error, and the tampered rows are preserved.
- **Not detected:** a privileged rewrite that keeps the document canonical, updates the matching columns and still replays through Core. `test_rr05_d6b4_limitation_a_canonical_core_valid_rewrite_is_not_detected` documents this: a rewritten evidence reference loads.
- **Why.** D6B-4 approved no content fingerprint; integrity rests on strict validation, Core replay, SQLite constraints, parent provenance and the single-host trust model. **None of this is cryptographic tamper detection, and none is claimed.**
- **Classification:** VERIFIED WITH LIMITATION.

### 11.2 WINDOW test seam, and a correction

The stored `WINDOW` category is proved only by monkeypatching the adapter's reference to `equity_issued_lifecycle_window` to refuse (`test_ls17_c_…`). The seam proves the mapping, the refusal of load and append, and that nothing is repaired. It is **not** a planted real-SQLite corruption.

**Correction.** The PR-2 report and that test's docstring say a stored disposition passing EI-4D level-A decoding and EI-6A re-derivation "always yields a window, so no planted row reaches `WINDOW`". Review of the merged code shows this is **overstated**:

- Every check in `equity_issued_lifecycle_window` except the grid membership check (window module, lines 64–68) and the derived-window cross-check is already guaranteed by `_stored_issuance`. The cross-check holds by construction, because the issued record and the window both come from the same proposal.
- The grid check is **not** part of level-A decoding (`equity_recommendation_generation_decoding.py` has no grid check; generation checks it, `generate_equity_trade_recommendation.py` lines 1008–1020).
- So a forged disposition with an off-grid entry window, with recomputed fingerprints, plus matching raw EI-6A ledger and issuance rows, would probably reach `WINDOW` on load. No governed write path produces such a record (the EI-4D recorder verifies levels A to C before writing), and the tests do not construct one.

The fail-closed behaviour is evidenced by the seam and by code review. The real-SQLite triggering of `WINDOW` is **not demonstrated**. No implementation change is made here; the docstring overstatement is recorded as limitation E2.

## 12. Terminal authority and O-3

| Requirement | Evidence | Result |
| --- | --- | --- |
| `CLOSED_TARGET` final; no state change | S `test_ls15_a_…`; A `test_ls15_a_…` | Later snapshot `NO_NEW_EVENTS`; constructing an authorization refused ("terminal"); a raw insert refused by the `terminal` trigger; rows unchanged |
| Core-permitted revision after terminal | S `test_ls15_b_…`; A `test_ls15_b_…` | A corrected bar 8 records `SOURCE_REVISION_RECORDED` (sequence 4, citing 3, `UNASSESSED`); state stays `CLOSED_TARGET` after reload; recorded once |
| `CANCELLED_BEFORE_ENTRY` final | S `test_ls15_terminal_states_reached_on_sqlite_are_final_after_restart` | A later zone touch records nothing; reload from a new adapter is terminal; rows unchanged |
| Deterministic terminal replay | S `test_ls16_rr01_rr04_…` | `CLOSED_TARGET`, identical in new adapters and a fresh interpreter |
| **RV-04 / O-3 (b)** | S `test_ls15_c_rv04_…`; A `test_ls15_c_rv04_…` | Missing bars at 10:51 record `ENTRY_WINDOW_OPENED` and `EVIDENCE_INCOMPLETE` (equal instants). Bars 7 to 19 arriving later, then a snapshot at 10:52: `NO_NEW_EVENTS`; state unchanged; rows byte-identical; no table, cursor or event kind added. A planted revision citing `EVIDENCE_INCOMPLETE` is admitted by SQL and refused by Core on reload (`HISTORY`) |
| Quiet evaluations | S `test_ls02_b_…`; A `test_ls02_b_…` | Repeated quiet snapshots: `NO_NEW_EVENTS`, zero rows, no sequence allocated, no cursor or offset table |

No restatement event is fabricated and no terminal state reopens. Specification section 8.2's audited-event requirement stays deferred to EI-8, which is not authorized.

## 13. Acceptance evidence matrix

**Statuses:**

| Status | Meaning |
| --- | --- |
| VERIFIED | Executed and passed locally (Windows); no qualification beyond pending Linux CI |
| VERIFIED WITH LIMITATION | Executed and passed locally, with a stated qualification: Windows-only separate-process evidence (WL), or a documented scope limit |
| PARTIALLY VERIFIED / NOT TESTED / BLOCKED | Not used: no row falls there |

- **Paths:** A is `northstar-application/tests/application_services/test_record_equity_recommendation_lifecycle.py`; S is `northstar-infrastructure/tests/test_sqlite_equity_recommendation_lifecycle.py`.
- **Local execution:** A 52 of 52 passed on merged `b7df16b`; S 76 of 76 passed on merged `dd6a434` (section 15.1).
- **Linux CI: PENDING for every row** (section 15.2). As for EI-6A, Windows-only separate-process rows stay VERIFIED WITH LIMITATION until Linux evidence is recorded.

### 13.1 Rows of the EI-5 matrix

| ID | Requirement | Application (A) | Infrastructure (S) | Qualification | Status |
| --- | --- | --- | --- | --- | --- |
| RR-01 | Lifecycle events reloaded after restart: the same state through Core replay | — | `test_ls16_rr01_rr04_…`; `test_e2e_…` | New adapter instances; file hash unchanged by loads | VERIFIED |
| RR-04 (state) | Replay of the lifecycle state from a separate OS process | — | `test_ls16_rr01_rr04_…`; `test_e2e_…` (fresh interpreter) | WL | VERIFIED WITH LIMITATION |
| RR-05 | A stored event tampered in a test: refused on reload; preserved | `test_ls10_rr05_…` | `test_rr05_a_tampered_…`; LS-09 and LS-10 plants; `test_rr05_d6b4_limitation_…` | Detectable tampering only; canonical Core-valid rewrite undetected (D6B-4; section 11.1) | VERIFIED WITH LIMITATION |
| RR-06 | Two evaluators from the same position: one succeeds; the other `CONFLICT`, or `EQUIVALENT` when identical | `test_ls13_…`; `test_ls14_rr06_…` | `test_ls13_rr06_…`; `test_ls14_rr06_…` | WL | VERIFIED WITH LIMITATION |
| RV-04 | Late gap-filling evidence after `INCOMPLETE_EVIDENCE`: terminal unchanged; no new event | `test_ls15_c_rv04_…`; `test_e2e_…` | `test_ls15_c_rv04_…` | — | VERIFIED |

### 13.2 EI-6B lifecycle-store scenarios

| ID | Requirement and subcases | Application (A) | Infrastructure (S) | Qualification | Status |
| --- | --- | --- | --- | --- | --- |
| LS-01 | First append: `APPENDED`; 1..n stored; replayed load | `test_ls01_…` | `test_ls01_ls02_a_…` | — | VERIFIED |
| LS-02 | (a) append after history, contiguous; (b) repeated or quiet snapshot: `NO_NEW_EVENTS`, nothing written | (a) `test_ls02_a_…`; (b) `test_ls02_b_…` | (a) `test_ls01_ls02_a_…`; (b) `test_ls02_b_…` | — | VERIFIED |
| LS-03 | (a) known, no events: empty `RECORDED`; (b) unknown: `None`, recorder `ISSUANCE_NOT_FOUND` | `test_ls03_a_…`; `test_ls03_b_…` | `test_ls03_a_…`; `test_ls03_b_…` | — | VERIFIED |
| LS-04 | (a) foreign key refuses a non-issued identity; (b) corrupt parent: D6-5 on load and append; (c) identifier alone is not authority | (b)(c) `test_ls04_b_c_…` | (a) `test_ls04_a_…`; (b) `test_ls04_b_corrupt_parent_…`, `test_ls04_b_an_inconsistent_ei6a_ledger_…`; (c) `test_ls04_c_…` | — | VERIFIED |
| LS-05 | Unauthorized append refused before any connection; database unchanged | `test_ls05_unauthorized_…`; `test_ls05_an_authorization_refuses_…` | `test_ls05_…` (15) | — | VERIFIED |
| LS-06 | UPDATE, DELETE, replacing INSERT refused | — | `test_ls06_…` | — | VERIFIED |
| LS-07 | Gap or duplicate refused; independent per issuance; EI-6A ledger unchanged | — | `test_ls07_sequences_…`; `test_ls07_two_processes_…` | The spawned part is additional evidence (WL); the requirement is met in-process | VERIFIED |
| LS-08 | Integer microseconds; non-decreasing; not before `issued_at`; text and integer agree | — | `test_ls08_…`; `test_ls15_c_rv04_…` (equal instants stored) | — | VERIFIED |
| LS-09 | Golden vectors round-trip; non-canonical, missing or extra keys, unknown kind, unsupported format refused (C; stored S `EVENT`) | `test_ls09_…` (3 functions, 23 cases) | `test_ls09_…` (12) | — | VERIFIED |
| LS-10 | Planted Core-refused history raises `HISTORY`, unrepaired; column disagreeing with document refused by CHECK | `test_ls10_rr05_…` | `test_ls10_a_stored_history_…` (5); `test_ls10_a_column_disagreeing_…` | — | VERIFIED |
| LS-11 | Failure mid-batch persists none; next append takes the same sequences | — | Four `test_ls11_…` tests | The process-exit test is additional (WL) | VERIFIED |
| LS-12 | Ack lost → `EQUIVALENT`; commit lost → bounded retry; unresolved claims nothing; corruption on reload permanent | Five `test_ls12_…` tests | Six `test_ls12_…` tests | Retry reviewed: compliant (section 10) | VERIFIED |
| LS-13 | Identical retry and concurrent identical requests in spawned processes: one `APPENDED`, one `EQUIVALENT`, no duplicates; a retry after restart is `EQUIVALENT` | `test_ls13_…` | `test_ls13_an_identical_retry_…`; `test_ls13_rr06_…` | WL; "after restart" shown with new adapter instances in one process, because authorizations are process-local (E4) | VERIFIED WITH LIMITATION |
| LS-14 | Conflicting concurrent appends in spawned processes, and a stale writer: first wins; `CONFLICT`; not adopted | `test_ls14_rr06_…`; `test_ls14_a_prior_history_…` | `test_ls14_a_stale_writer_…`; `test_ls14_rr06_…` | WL | VERIFIED WITH LIMITATION |
| LS-15 | (a) state change after terminal refused (Core, validator, trigger); (b) permitted revision after terminal accepted; (c) no revision citing `EVIDENCE_INCOMPLETE` | (a)(b)(c) `test_ls15_a_…`, `test_ls15_b_…`, `test_ls15_c_rv04_…` | (a) `test_ls15_a_…`; (b) `test_ls15_b_…`; (c) `test_ls15_c_rv04_…`; `test_ls15_terminal_states_…` | — | VERIFIED |
| LS-16 | Additive schema, idempotent, foreign database refused; restart-safe reconstruction in new adapters and a fresh interpreter | — | `test_ls16_the_schema_…`; `test_ls16_rr01_rr04_…` | WL (fresh interpreter) | VERIFIED WITH LIMITATION |
| LS-17 | (a) derived from the issued record and verified disposition; (b) equals EI-5C's; (c) inconsistent or undecodable provenance refuses (`WINDOW_PROVENANCE_INVALID`, or `WINDOW` on load); (d) no caller window input; (e) Infrastructure computes no window | (a)(b) `test_ls17_a_b_…`; (c) `test_ls17_c_inconsistent_…`; (c)(d) `test_ls17_c_d_…` | (a)(b) `test_ls17_a_b_…`; (c) `test_ls17_c_…` (seam); (d)(e) `test_ls17_d_e_…` | (c) on load: `WINDOW` proved by a test seam only (E2, section 11.2); on load, undecodable or inconsistent dispositions fail closed earlier with D6-5 (`test_ls04_b_…`) | VERIFIED WITH LIMITATION |

### 13.3 Totals

| Group | Rows | VERIFIED | VERIFIED WITH LIMITATION | PARTIALLY VERIFIED | NOT TESTED | BLOCKED |
| --- | --- | --- | --- | --- | --- | --- |
| EI-5 matrix rows (13.1) | 5 | 2 (RR-01, RV-04) | 3 (RR-04 (state), RR-05, RR-06) | 0 | 0 | 0 |
| LS scenarios (13.2) | 17 | 13 | 4 (LS-13, LS-14, LS-16, LS-17) | 0 | 0 | 0 |
| **Total** | **22** | **15** | **7** | **0** | **0** | **0** |

- **By reason:** five rows are qualified by Windows-only separate-process evidence (RR-04 (state), RR-06, LS-13, LS-14, LS-16); RR-05 by D6B-4; LS-17 by the `WINDOW` seam. LS-13 also carries E4.
- **Which may change with Linux CI.** Like EI-6A's section 9.4, the five WL rows may be reclassified once a Linux run shows the S module executed without skips. RR-05 and LS-17 keep their limitations, and LS-13 keeps E4.
- **Historical separation.** These are new EI-6B rows. The accepted EI-5D/EI-5E and EI-6A totals are unchanged.

**Also tested, outside the matrix:** A `test_the_append_rule_refuses_another_parent_or_window_as_an_invalid_write`, `test_snapshot_rules_are_ei5cs_…`, `test_a_missing_disposition_…`, `test_e2e_…` and two architecture tests; S `test_a_write_lock_not_obtained_…`, `test_e2e_the_real_sqlite_vertical_slice` (empty lifecycle, opened, activated, `EQUIVALENT` retry, stale `CONFLICT`, reopen, target, terminal, fresh process, no order, fill, position, cursor or monitor object) and three architecture tests.

### 13.4 Reclassification after Linux CI (2026-11-15)

Sections 13.1 to 13.3 record the classification at engineering acceptance and are not rewritten.

**Why exactly five rows change.** Their only limitation was Windows-only separate-process evidence (WL). Their S tests are in `tests/test_sqlite_equity_recommendation_lifecycle.py`, which has no platform conditional, skip or xfail marker, and spawns processes with the `spawn` start method on every platform. The Infrastructure Linux run (section 15.3) executed that module with successful progress and no skip or failure displayed in its section. All 76 cases, including the spawned-process and fresh-interpreter ones, therefore ran and passed on Linux as well as on Windows. The GitHub log shows the module's progress, not each test name.

| ID | Tests (unchanged mapping) | Before | After | Remark |
| --- | --- | --- | --- | --- |
| RR-04 (state) | S `test_ls16_rr01_rr04_…`; `test_e2e_…` | VERIFIED WITH LIMITATION | **VERIFIED** | Fresh interpreter, now on Linux too |
| RR-06 | A `test_ls13_…`, `test_ls14_rr06_…`; S `test_ls13_rr06_…`, `test_ls14_rr06_…` | VERIFIED WITH LIMITATION | **VERIFIED** | Spawned identical and conflicting races, now on Linux too |
| LS-13 | A `test_ls13_…`; S `test_ls13_an_identical_retry_…`, `test_ls13_rr06_…` | VERIFIED WITH LIMITATION | **VERIFIED** (owner) | E4 is retained as an accepted architectural boundary, not a scenario limitation: authorizations are process-local and are never portable between processes; the exact `EQUIVALENT` retry after restart is shown with new adapter instances, and a fresh process re-evaluates from the durable history (`NO_NEW_EVENTS`) |
| LS-14 | A `test_ls14_rr06_…`, `test_ls14_a_prior_history_…`; S `test_ls14_a_stale_writer_…`, `test_ls14_rr06_…` | VERIFIED WITH LIMITATION | **VERIFIED** | Spawned conflicting race, now on Linux too |
| LS-16 | S `test_ls16_the_schema_…`; `test_ls16_rr01_rr04_…` | VERIFIED WITH LIMITATION | **VERIFIED** | Fresh interpreter, now on Linux too |
| RR-05 | Section 13.1 | VERIFIED WITH LIMITATION | **Unchanged: VERIFIED WITH LIMITATION** | D6B-4 boundary (section 11.1) |
| LS-17 | Section 13.2 | VERIFIED WITH LIMITATION | **Unchanged: VERIFIED WITH LIMITATION** | `WINDOW` on load by test seam only (section 11.2) |

**Final totals:**

| Group | Rows | VERIFIED | VERIFIED WITH LIMITATION | NOT VERIFIED |
| --- | --- | --- | --- | --- |
| EI-5 matrix rows | 5 | 4 (RR-01, RR-04 (state), RR-06, RV-04) | 1 (RR-05) | 0 |
| LS scenarios | 17 | 16 | 1 (LS-17) | 0 |
| **Total** | **22** | **20** | **2** | **0** |

Arithmetic: 15 VERIFIED + 5 reclassified = 20; 7 VERIFIED WITH LIMITATION − 5 = 2.

## 14. Targeted mutations

Run on the pre-merge working trees (same files and line counts as the change commits). Sources were restored byte for byte after each run.

### 14.1 Application (#98): 6 of 6 detected

Decoder accepting non-canonical text; append rule comparing only history length; `EQUIVALENT` classified as `CONFLICT`; validator skipping Core replay; window accepting a non-re-deriving issued record; validator ignoring post-issue mutation. The first run detected 4: the length-only and window mutants survived and the two missing tests were added (`test_ls14_a_prior_history_of_the_same_length_…`; the narrowed-record cases of `test_ls17_c_…`) before merge.

### 14.2 Infrastructure (#68): 10 of 11 detected, 1 equivalent survivor

| # | Mutant | Result | Caught by |
| --- | --- | --- | --- |
| 1 | Authorization gate removed | Detected | `test_ls05_…` |
| 2 | History compared by length | Detected | `test_ls14_a_stale_writer_…` |
| 3 | `next_sequence` trigger disabled | Detected | `test_ls07_sequences_…` |
| 4 | `not_before_issuance` trigger disabled | Detected | `test_ls08_…` |
| 5 | Load-time column check removed | Detected | `test_ls09_…[column disagrees with document]` |
| 6 | Document/column CHECK removed | Detected | `test_ls09_…` |
| 7 | UPDATE allowed | Detected | `test_ls06_…` |
| 8 | `chronology` trigger disabled | Detected | `test_ls08_…` |
| 9 | `terminal` trigger disabled | Detected | `test_ls15_a_…` (also tripped an over-narrow message match in LS-06 on the first run) |
| 10 | `no_replace` trigger disabled | Detected **after LS-06 was strengthened** | `test_ls06_…` |
| 11 | Adapter contiguity check removed | **Survived — equivalent** | — |

- **Mutant 10.** It survived the first run because `next_sequence` refuses every row `no_replace` refuses. LS-06 now also probes `no_replace` with the overlapping triggers dropped inside a rolled-back transaction, which detects it.
- **Mutant 11 is redundant under the tested path.** Core's `_replay` refuses any event whose decoded sequence is not its position, and the adapter's column comparison and the CHECK tie the column sequence to the document sequence. So a non-contiguous history still fails closed and permanently, as `HISTORY` or `EVENT`.
- **Residual risk of 11:** for a planted row whose column and document sequences disagree, the category reported could be `EVENT` instead of `HISTORY`. The failure stays permanent and non-retryable.

**Result: 10 detected, 1 surviving equivalent.** This is not 11 of 11.

## 15. Test results and CI

### 15.1 Local results

All Windows 11; Python 3.13.15; SQLite 3.53.1; dependencies resolved by `uv`.

| Point | Revision | Result |
| --- | --- | --- |
| EI-6B Application module (52) and EI-5C evaluator module (84) | **Merged `b7df16b`**, Core `29bd34a`; run 2026-11-15 for this record, writing no bytecode or cache | **136 passed** |
| EI-6B Infrastructure module (76) | **Merged `dd6a434`**, Application `b7df16b`, Core `29bd34a`; run 2026-11-15 for this record, writing no bytecode or cache | **76 passed**, none skipped; spawned-process and fresh-interpreter tests executed |
| Full Application suite | Pre-merge working tree of `82f872d` | 4,726 passed (reported) |
| EI-5C and EI-5E focused regressions | Pre-merge working tree | 89 passed (reported) |
| Full Infrastructure suite | Pre-merge working tree of `ac66907` (same files and line counts) | 3,412 passed, 27 skipped, 0 failed (reported): 23 opt-in live Databento tests and 4 `tzset is POSIX-only` |
| Ruff check and format; architecture tests; `git diff --check` | Pre-merge working trees | Clean (reported) |
| Targeted mutations | Pre-merge working trees | Section 14 |

**Not run against the merged commits:** the full Application and Infrastructure suites. Only the focused modules ran on `b7df16b` and `dd6a434` (limitation E7).

### 15.2 D6B-7 independent Linux CI

**PENDING — evidence not available in this environment.** No GitHub CLI or API access is available, no CI artifact exists locally, and no credential was used. No run identifier, URL, conclusion or dependency SHA is recorded. **A merged PR is not CI evidence.**

| Required evidence | northstar-application #98 | northstar-infrastructure #68 |
| --- | --- | --- |
| Workflow | `python-ci.yml` (job `test`, `ubuntu-latest`) — the only workflow file in each repository; identity to be confirmed from run metadata or stated as inferred from the inventory | Same |
| Run URL, run ID, job ID | **PENDING** | **PENDING** |
| Linux runner; test-job conclusion | **PENDING** | **PENDING** |
| Tested revision | **PENDING** (expected `82f872d` head or `b7df16b` merge) | **PENDING** (expected `ac66907` head or `dd6a434` merge) |
| Core checkout | **PENDING** | **PENDING** |
| Application checkout | — | **PENDING**; must contain `b7df16b` |
| EI-6B tests ran | **PENDING**: the 52 cases of `test_record_equity_recommendation_lifecycle.py` | **PENDING**: the 76 cases of `test_sqlite_equity_recommendation_lifecycle.py`, with no `s` or `F` in its section |
| Spawned-process and fresh-process tests not skipped | — | **PENDING** (the module has no skip, platform or xfail marker) |
| The 4 `tzset` tests ran on Linux | — | **PENDING** (`tests/test_nse_futures_session.py`) |
| Ruff check and format | **PENDING** | **PENDING** |

**Evidence-collection step.** A person with repository access records, for each run: the run URL and identifiers, event and conclusion; the head SHA tested; the commits checked out in the "Checkout northstar-core" and "Checkout northstar-application" steps (the workflow takes `develop` at run time); and, from "Run Tests", the EI-6B module's progress line and the summary line. A run on `develop` at or after `dd6a434` whose Application checkout contains `b7df16b` satisfies the Infrastructure part.

**D6B-7: NOT SATISFIED.**

### 15.3 Verified Linux CI evidence (2026-11-15)

Section 15.2 records the state before this evidence and is not rewritten.

**Source of the evidence.** The run and job identifiers, URLs, revisions and results were supplied and screenshot-verified by the owner on 2026-11-15. They were reconciled for this record against local Git and the repositories' workflow definitions. They were not fetched from this environment, which has no GitHub CLI or API access; no additional run metadata was viewed, and no CI run was repeated.

| Item | northstar-application #98 | northstar-infrastructure #68 |
| --- | --- | --- |
| Run ID | `43623021952` | `43643943540` |
| Job ID | `132532666955` | `132597422331` |
| URL | <https://github.com/Northstar-Dhruba/northstar-application/actions/runs/43623021952/job/132532666955> | <https://github.com/Northstar-Dhruba/northstar-infrastructure/actions/runs/43643943540/job/132597422331> |
| Tested revision | `b7df16b8f3348672627b4f692660d9fa844e03ef` (the #98 merge; local `develop` HEAD) | `dd6a434368bfa7124f33381c38bc52c65ab4efde` (the #68 merge; local `develop` HEAD) |
| Application dependency | — | `b7df16b8f3348672627b4f692660d9fa844e03ef` (contains #98) |
| Core dependency | `29bd34ac97810e6b471ed6c2ba4618ebaa57d9c6` | `29bd34ac97810e6b471ed6c2ba4618ebaa57d9c6` |
| Platform; Python | Linux; 3.13.18 | Linux; 3.13.18 |
| Test job | SUCCESS | SUCCESS |
| Pytest | **4,726 collected, 4,726 passed**, no failures or skips; 42.33 s | **3,439 collected; 3,415 passed, 24 skipped**, no failures; 216.67 s |
| EI-6B module | `tests/application_services/test_record_equity_recommendation_lifecycle.py` (52 cases) in the pytest output | `tests/test_sqlite_equity_recommendation_lifecycle.py` (76 cases) shown with successful progress and no displayed skip or failure |

**Consistency checks:**

- **Revisions.** All three SHAs exist locally and equal the `develop` HEADs of their repositories. The #98 and #68 merge subjects name those PRs.
- **Dependencies.** Each workflow checks out its dependencies at `develop`; the recorded commits are those `develop` HEADs. The Infrastructure run's Application checkout is the #98 merge.
- **Counts.** Application's 4,726 equals the local full-suite count (section 15.1). Infrastructure's 3,439 collected equals the local Windows total (3,412 + 27); section 15.4 reconciles the skips.
- **Modules.** The 52-case and 76-case EI-6B modules hold no skip or platform marker. Their execution is recorded at module level; individual scenario names are not claimed to appear in the GitHub log.

**Workflow identity.** At both tested revisions, `.github/workflows/python-ci.yml` is the **only** workflow file. It is named "Python CI" and has one job, `test`, which runs on `ubuntu-latest`, installs Python 3.13 with `uv`, checks out its dependencies at `develop`, and runs `ruff check`, `ruff format --check` and `pytest`. The evidence (a Linux test job, Python 3.13.18, these dependency commits) matches it, so the runs are `python-ci.yml` runs.

- **Qualification.** This identity is established from the repositories' workflow inventory, not from run metadata displaying the filename, which the supplied screenshots did not show and this environment cannot access.
- **Ruff.** The supplied evidence shows the pytest step; the Ruff steps are covered by the job's SUCCESS, not shown individually.

**D6B-7: SATISFIED, 2026-11-15** (owner accepted the reviewed evidence):

- local Windows synthetic and real-SQLite acceptance: sections 13 and 15.1;
- independently verified successful Linux CI for the Application and Infrastructure implementation PRs, at their actual merged revisions, with their dependency commits: this section;
- evidence that the EI-6B tests executed, including the spawned-process and fresh-process cases: this section and section 13.4;
- reconciled Linux and Windows skip totals: section 15.4.

The workflow-identity qualification is the only residual one. EI-4 formal CI closure (EI4-C4) stays a separate obligation; these runs do not close it unless they meet the [EI-4 Decision Register, section 15.3](Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4).

### 15.4 Windows and Linux skip reconciliation

| Environment | Passed | Skipped | Collected |
| --- | --- | --- | --- |
| Windows local (section 15.1) | 3,412 | 27 | 3,439 |
| Linux CI (section 15.3) | 3,415 | 24 | 3,439 |

**Source-supported reconciliation.** The Linux skip identities are not listed individually in the supplied CI evidence. The explanation below is reconciled from the test sources at `dd6a434` and the workflow definition, not a verbatim Linux skip-reason report. The Windows skips are from the local `-rs` report.

| Skip condition | Source | Windows | Linux |
| --- | --- | --- | --- |
| Opt-in live Databento acceptance | `tests/test_live_databento_futures_acceptance.py`: module-level `skipif(not os.environ.get("NORTHSTAR_LIVE_DATABENTO"))` | 23 skipped | 23 skipped (the flag is not set in the workflow) |
| POSIX timezone | `tests/test_nse_futures_session.py`: `test_the_host_timezone_does_not_move_a_session`, `skipif(not hasattr(time_module, "tzset"))`, 4 zones (America/Chicago, Asia/Kolkata, Pacific/Kiritimati, UTC) | 4 skipped (no `time.tzset`) | Eligible and counted as passed; the module showed successful progress |
| Docs approval-record cross-check | `tests/test_nse_equity_session_reference.py`: `test_production_evidence_matches_the_owner_approval_record_in_the_register`, `skipif(not _REGISTER.is_file())`, reading `../northstar-docs/research/NSE-Cash-Equity-Session-Evidence-Register.md` | Runs (Docs is checked out beside) | 1 skipped: the workflow checks out Infrastructure, Core and Application only; the earlier Linux log showed a skipped case in this module |
| **Total** | | **27** | **24** |

Linux therefore passes three more cases than Windows (4 − 1). No other skip, platform or xfail condition exists in the Infrastructure tests. The Docs cross-check skip is unrelated to EI-6B and is not a failed or skipped EI-6B test.

## 16. Limitations and verification gaps

### 16.1 Remaining EI-6B limitations

| ID | Limitation | Classification |
| --- | --- | --- |
| E1 | **RR-05 / D6B-4:** a canonical, column-consistent, Core-valid privileged rewrite is not detected; no fingerprint, no cryptographic tamper detection | Accepted architectural boundary (D6B-4) |
| E2 | **`WINDOW` on load proved by a test seam only;** the test docstring and PR-2 report overstate it as unreachable (section 11.2) | Remaining EI-6B verification gap |
| E3 | **No persisted quiet-evaluation cursor;** across calls only the last recorded event bounds chronology; no exactly-once processing | Accepted architectural boundary (clarification 3) |
| E4 | **Process-local authorizations:** an exact `EQUIVALENT` retry after restart is shown with new adapter instances; a fresh process re-evaluating gets `NO_NEW_EVENTS` | Accepted architectural boundary (D6B-5 registry) |
| E5 | **Private helper coupling:** the adapter reuses EI-6A `_stored_issuance`, `_consistent_ledger`, `_SELECT_ISSUANCES`, `_SELECT_DISPOSITION`, EI-4D `_stored_disposition` and the EI-6A schema's `_canonical` and `_instant_us`; a refactor of those modules must keep them | Remaining EI-6B limitation (maintainability; follows EI-6A's precedent) |
| E6 | **Test seams:** `_commit`, `_connect`, `_confirm_appended`, `_BUSY_TIMEOUT_SECONDS`, the window seam; TEST-ONLY raw SQL and planted triggers on disposable files | Remaining EI-6B limitation |
| E7 | **Local evidence:** the full suites and mutations ran on pre-merge working trees; only the focused modules ran on the merged commits | Verification gap; addressed by D6B-7 Linux runs at the merged revisions |
| E8 | **EI-6A prerequisite:** the lifecycle store refuses a database without the EI-6A tables and never initializes or repairs EI-6A objects; lifecycle reads check the whole EI-6A ledger | Accepted architectural boundary (addendum 8.1, 8.4) |
| E9 | **Redundant guards:** the adapter contiguity check duplicates Core (equivalent mutant); `no_replace` overlaps `next_sequence` (now probed alone) | Accepted (defence in depth) |
| — | Windows-only separate-process evidence (five rows) | Verification gap; awaits D6B-7 |

### 16.2 Carried forward, unchanged

| ID | Limitation | Classification |
| --- | --- | --- |
| C1 | EI4-D8 conservative exposure: the real-chain tests use TCS and INFY issued at the same instant | Carried forward |
| C2 | VC-04: rollback or downgrade against stored evaluator-v2 records not authorized; lifecycle rows now rest on evaluator-v2 issuances | Carried forward (more consequential) |
| C3 | N6: database label-format pairing trigger deferred | Carried forward |
| C4 | I08: equivalent-mutant qualification | Carried forward |
| C5 | IS-16 (EI-5D) | Carried forward |
| C6 | NB-04 (EI-5E) | Carried forward |
| C7 | EI-6A HW-03 and adapter-enforced ledger checks | Carried forward |
| C8 | EI-4 formal CI closure (EI4-C4) pending | Carried forward; separate obligation |
| C9 | O-2 and EI3-D10 unresolved; EI-6C not authorized | Carried forward; unrelated to EI-6B acceptance |
| C10 | Single host; MODELED only; synthetic evidence only | Carried forward |

**No new blocker** is raised. The acceptance blocker is D6B-7 (section 15.2).

### 16.3 Limitation reconciliation (2026-11-15)

Sections 16.1 and 16.2 record the state at engineering acceptance and are not rewritten.

**Resolved or addressed:**

| Item | Disposition | Evidence |
| --- | --- | --- |
| D6B-7 Linux CI | **RESOLVED**; workflow identity qualified as stated | Section 15.3 |
| Windows-only separate-process evidence (five rows) | **RESOLVED**: RR-04 (state), RR-06, LS-13, LS-14 and LS-16 also ran and passed on Linux | Run `43643943540`; section 13.4 |
| E7 — No full suite on the merged commits | **ADDRESSED**: the full Application suite passed on Linux at merged `b7df16b` and the full Infrastructure suite at merged `dd6a434`. No full local Windows run on the merged commits is recorded; none is now required | Runs `43623021952`, `43643943540` |

**Remaining (accepted by the owner):**

| Item | Limitation |
| --- | --- |
| E1 | No lifecycle-event content fingerprint (D6B-4); RR-05 does not detect a canonical, column-consistent, Core-valid privileged rewrite; no cryptographic tamper detection |
| E2 | The stored `WINDOW` integrity path is proved by a disclosed test seam; a planted real-SQLite record reaching it was not constructed. A possible forged-disposition route (recomputed fingerprints with matching raw EI-6A rows) remains untested and is not claimed impossible (LS-17) |
| E3 | No persisted quiet-evaluation cursor; no exactly-once snapshot processing |
| E4 | Process-local append authorizations: never portable between processes; a fresh process reconstructs the durable history and re-evaluates from it |
| E5 | Reuse of EI-4D and EI-6A module-private helpers |
| E6 | Some evidence relies on test seams and TEST-ONLY raw SQL on disposable files |
| E8 | The EI-6A tables must exist before EI-6B storage initialization; the lifecycle store never creates or repairs EI-6A objects |
| E9 | Redundant defence-in-depth guards (the adapter contiguity check duplicates Core: the equivalent mutant of section 14.2) |
| — | D6B-7 workflow identity: established from the workflow inventory, not from run metadata (section 15.3) |

**Carried forward, unchanged:** C1 to C10 of section 16.2 — EI4-D8; VC-04; N6; I08; IS-16; NB-04; EI-6A HW-03 and its adapter-enforced ledger checks; EI-4 formal CI closure (EI4-C4), separately outstanding; O-2 and EI3-D10 unresolved, EI-6C not authorized; single host, MODELED-only issuance, synthetic-evidence-only lifecycle recording.

**Commit uncertainty (section 10), accepted under the single-host SQLite model.** No blind retry follows a missing acknowledgement: the failed transaction is rolled back and its connection closed; the authoritative history is reloaded; a further write is made only when the stored history is exactly the authorized prior; the same content-bound compare-and-append prevents duplicate writes; unresolved uncertainty stays a typed failure. No implementation was changed.

## 17. Engineering verdict and milestone status

**EI-6B LOCAL ENGINEERING ACCEPTANCE: PASS WITH DOCUMENTED LIMITATIONS — synthetic, local Windows, Application contract and real-SQLite engineering acceptance.**

| Milestone | Status | Evidence |
| --- | --- | --- |
| 1. Engineering contracts merged | **DONE** | #98 (`b7df16b`), #68 (`dd6a434`) |
| 2. Local Windows test acceptance | **DONE** | Sections 13 and 15.1: 22 rows, 15 VERIFIED, 7 VERIFIED WITH LIMITATION |
| 3. Independent Linux CI verified (D6B-7) | **PENDING** | Section 15.2 |
| 4. Acceptance limitations reconciled | **PENDING** (after Linux CI) | Section 16 |
| 5. Final owner decision | **PENDING** | — |
| 6. Documentation closure | **PENDING** | — |

**Recommended owner decision:** do **not** accept EI-6B yet.

1. Collect and record the two Linux CI runs (section 15.2).
2. Reclassify the Windows-only process rows if those runs show the modules executed without skips.
3. Then decide on acceptance with the documented limitations of section 16, including E2.

### 17.1 Owner acceptance (2026-11-15)

**OWNER APPROVED — 2026-11-15.**

**EI-6B — Durable Equity Recommendation Lifecycle Persistence: ACCEPTED WITH DOCUMENTED LIMITATIONS.**

**Scope accepted:** durable lifecycle events for authoritative MODELED recommendations; controlled synthetic evidence; the Application-owned lifecycle-store contract; versioned canonical event documents (`northstar.equity-recommendation-lifecycle-event/1`); content-bound verified append authorization; same-file SQLite lifecycle storage; atomic compare-and-append; deterministic Core replay; restart and cross-process persistence verification; O-3 option (b); the single-host trust model — exactly as authorized by D6B-1 to D6B-7.

| Stage | Record |
| --- | --- |
| 1. Implementation merged | northstar-application #98 (`b7df16b`); northstar-infrastructure #68 (`dd6a434`) |
| 2. Local Windows engineering acceptance | PASS WITH DOCUMENTED LIMITATIONS (sections 13 and 15.1; Docs PR-3, northstar-docs #67) |
| 3. Independent Linux CI | Runs `43623021952` and `43643943540`, SUCCESS (section 15.3) |
| 4. D6B-7 | **SATISFIED**, 2026-11-15, with the workflow-identity qualification |
| 5. Final owner acceptance | **ACCEPTED WITH DOCUMENTED LIMITATIONS**, 2026-11-15 |
| 6. Remaining documented limitations | Scenario level: RR-05 and LS-17. Architectural and carried forward: section 16.3 |
| Acceptance matrix (22) | 20 VERIFIED, 2 VERIFIED WITH LIMITATION (RR-05, LS-17), 0 NOT VERIFIED (section 13.4) |
| Mutations | Application 6 of 6 detected; Infrastructure 10 of 11 detected, 1 equivalent survivor (the adapter contiguity check; Core replay refuses the same histories) |

**Not authorized by this acceptance:** EI-6C monitoring; automatic snapshot selection; the resolution of O-2 or EI3-D10; scheduled lifecycle evaluation; market-data polling; EI-8 restatement; ACTUAL issuance; broker execution; orders, fills or positions; real-money trading; production deployment; API or Web integrations; a relaxation of EI4-D8; terminal-state reopening; historical lifecycle rewriting. It does not close EI-4 and does not reopen EI-6A or EI-5. **EI-6C needs its own architecture review and owner authorization.**

The owner record is the [EI-5 Decision Register, section 21](Equity-Intraday-V1-EI5-Decision-Register.md#21-owner-acceptance-of-ei-6b-2026-11-15).

## 18. Non-authorizations

This record authorizes nothing: not EI-6C monitoring, automatic snapshot selection, scheduling, polling or a monitoring cursor; the resolution of O-2 or EI3-D10; EI-8 restatement or any new Core event kind; terminal-state reopening or historical lifecycle rewriting; ACTUAL issuance; broker orders or fills; real-money trading; real or live market data; production deployment; API or Web exposure; composition roots; a relaxation of EI4-D8. It does not close EI-4 and does not reopen EI-6A. EI-6B remains single-host, MODELED-only and synthetic-evidence-only.

Neither this record nor the owner acceptance of section 17.1 authorizes anything further. O-3 stays resolved as option (b); specification section 8.2's audited-event requirement for late gap-filling evidence stays deferred to a future EI-8 design, which is not authorized.

## 19. Related documents

- [EI-5 Decision Register](Equity-Intraday-V1-EI5-Decision-Register.md), sections 19 and 20
- [EI-6B Durable Recommendation Lifecycle Persistence Design v1.0](../architecture/Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design v1.0](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [EI-6A Durable Issuance Authority Engineering Acceptance Record](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md)
- [EI-6A SQLite Storage Design v1.0](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md)
- [EI-5D/EI-5E Engineering Acceptance Record](Equity-Intraday-V1-EI5D-EI5E-Modeled-Issuance-and-Synthetic-Integration-Engineering-Acceptance-Record.md)
- [Equity Intraday V1 Product Specification, section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states)
