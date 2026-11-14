# Equity Intraday V1 — EI-6A Durable SQLite Issuance Authority Engineering Acceptance Record

## Status

**Engineering acceptance record, 2026-11-14.** This is Docs PR-3 of the EI-6A plan authorized in the [EI-5 Decision Register, section 17](Equity-Intraday-V1-EI5-Decision-Register.md#17-owner-authorization-of-ei-6a-2026-11-14).

**EI-6A — ACCEPTED WITH DOCUMENTED LIMITATIONS, 2026-11-14 (owner).** See [section 13.1](#131-owner-acceptance-2026-11-14).

> **Note, 2026-11-14 (D6-6 evidence and owner acceptance).** The bullets below record the state at engineering acceptance and are not rewritten. Since then:
>
> - **Linux CI recorded.** Successful Linux runs exist for northstar-application #97 (run `43599364295`) and northstar-infrastructure #67 (run `43604917993`) ([section 11.3](#113-verified-linux-ci-evidence-2026-11-14)). **D6-6: SATISFIED**, with the workflow-identity qualification stated there.
> - **Reclassified.** The six Windows-only separate-process rows are VERIFIED. The matrix is now **25 VERIFIED, 1 VERIFIED WITH LIMITATION (HW-03 (S)), 0 NOT VERIFIED** ([section 9.4](#94-reclassification-after-linux-ci-2026-11-14)).
> - **Limitations reconciled** in [section 12.1](#121-limitation-reconciliation-2026-11-14).
> - **Owner decision.** The owner accepted EI-6A with documented limitations on 2026-11-14 ([section 13.1](#131-owner-acceptance-2026-11-14)).
> - **Unchanged.** EI-6B and EI-6C stay NOT AUTHORIZED.

- **Verdict:** EI-6A LOCAL ENGINEERING ACCEPTANCE: **PASS WITH DOCUMENTED LIMITATIONS — synthetic, local Windows real-SQLite engineering acceptance** ([section 13](#13-engineering-verdict-and-recommendation)).
- **What it covers:** EI-6A — Durable SQLite Recommendation Issuance and Declaration Authority, merged as northstar-infrastructure #67 on top of the D6-5 Application prerequisite, northstar-application #97 ([section 4](#4-repository-baselines)).
- **Acceptance matrix:** 26 EI-6A scenario rows — **19 VERIFIED, 7 VERIFIED WITH LIMITATION**, none NOT VERIFIED ([section 9](#9-acceptance-evidence-matrix)). EI-6B rows stay out of scope.
- **D6-6 independent Linux CI: NOT VERIFIED — evidence not available in this environment** ([section 11.2](#112-d6-6-independent-linux-ci)). This is a hard owner-acceptance gate.
- **Owner acceptance: PENDING.** Under D6-6, EI-6A cannot be owner-accepted until both required Linux CI runs are independently verified, and then only by a separate owner decision.
- Sections 1 to 13 are engineering evidence, **not** an owner approval. D6-1 to D6-6 and every earlier EI-5, ADR-024 and ADR-025 decision are unchanged. Nothing here authorizes EI-6B, EI-6C, ACTUAL issuance or any production use ([section 14](#14-non-authorizations)).

## 1. Acceptance identity

| Item | Value |
| --- | --- |
| Milestone | EI-6A — Durable SQLite Recommendation Issuance and Declaration Authority |
| Record | Docs PR-3, engineering acceptance |
| Date | 2026-11-14 |
| Governance | [EI-5 Decision Register, section 17](Equity-Intraday-V1-EI5-Decision-Register.md#17-owner-authorization-of-ei-6a-2026-11-14) (D6-1 to D6-6); [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), sections 4 to 7, 13 (amended 2026-11-14) and 14 |
| Design | [EI-6A SQLite Storage Design v1.0](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md); [EI-5 Issuance and Lifecycle Design v1.0](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md), sections 8, 10 and 12 |
| Data | Synthetic only; MODELED only |
| Authority | The SQLite issuance authority, in the same file as the EI-4D generation records; single host |
| Platform | Windows development machine; local executions only |

## 2. Scope

**In scope:** the Infrastructure adapter `SQLiteEquityRecommendationIssuanceAuthority`, implementing the merged Application `EquityRecommendationIssuanceReader` and `EquityRecommendationIssuanceWriter`; its additive schema (the recording-sequence ledger, the issuance table and the declaration table); the D6-5 typed corruption handling; and the EI-6A acceptance scenarios of the addendum, section 12.

**Out of scope:** EI-6B (the lifecycle-event table, `EquityRecommendationLifecycleStore`, compare-and-append; RR-01, the state part of RR-04, RR-05, RR-06), EI-6C (monitoring, snapshot selection), composition roots, ACTUAL issuance, orders, fills, positions, brokers, API, Web, real data, and any change to Core, EI-4D records or Application contracts.

## 3. Governance references

| Layer | State | Where |
| --- | --- | --- |
| Owner authorization | D6-1 to D6-6 (2026-11-14); EI-6A authorized; EI-6B and EI-6C not authorized | [EI-5 Decision Register, section 17](Equity-Intraday-V1-EI5-Decision-Register.md#17-owner-authorization-of-ei-6a-2026-11-14) |
| Storage design | The EI-6A addendum (D6-4) | [EI-6A SQLite Storage Design v1.0](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md) |
| Prerequisite | D6-5 typed stored-authority corruption error, merged | northstar-application #97 ([section 4](#4-repository-baselines)) |
| Implemented behaviour | northstar-infrastructure #67 | [Sections 5 to 8](#5-implementation-inventory) |
| Local evidence | Local Windows runs | [Section 11.1](#111-local-results) |
| Independent CI | Not verified | [Section 11.2](#112-d6-6-independent-linux-ci) |
| Remaining limitations | L1 to L17 | [Section 12](#12-remaining-limitations-and-blockers) |

## 4. Repository baselines

Read from local Git on 2026-11-14. Every repository was on `develop` with a clean working tree; no fetch was run.

| Repository | `develop` HEAD | Role |
| --- | --- | --- |
| northstar-infrastructure | `63d2ab3` (#67; change commit `e756d7d`, parents `f71f8f7`, `e756d7d`) | EI-6A adapter, schema and acceptance tests |
| northstar-application | `a17bd91` (#97; change commit `00089fc`, parents `9a68209`, `00089fc`) | D6-5 prerequisite: `InvalidStoredEquityRecommendationIssuanceAuthorityError` and its mappings |
| northstar-docs | `5268532` (#63) | EI-6A authorization and addendum; this record, uncommitted |
| northstar-core | `29bd34a` (#40) | Unchanged |
| northstar-api | `7ad37f5` | Unchanged |
| northstar-web | `5aeb94c` | Unchanged |

**Change scope of `e756d7d`:** 5 files, 2,560 insertions, 0 deletions — two new production modules, two package exports, one new fixtures module and one new test module. No existing test, schema object or EI-4D module is modified.

## 5. Implementation inventory

Paths are under northstar-infrastructure.

| Component | File | Responsibility |
| --- | --- | --- |
| Schema | `src/northstar_infrastructure/persistence/sqlite_equity_recommendation_issuance_schema.py` | The three tables, the session index and 16 triggers; `initialize_equity_recommendation_issuance_schema`, which refuses a database without the EI-4D generation tables |
| Adapter | `src/northstar_infrastructure/persistence/sqlite_equity_recommendation_issuance.py` | `SQLiteEquityRecommendationIssuanceAuthority`: the four reads and two authorized writes; the ledger checks; rehydration; failure mapping |
| Exports | `src/northstar_infrastructure/persistence/__init__.py` | `SQLiteEquityRecommendationIssuanceAuthority`, `initialize_equity_recommendation_issuance_schema` |
| Fixtures | `tests/equity_recommendation_issuance_fixtures.py` | The real chain over one file; spawned-process workers |
| Tests | `tests/test_sqlite_equity_recommendation_issuance.py` | 36 test functions, 43 cases |

**Ownership.** The Application's pure rules (`equity_issuance_write_decision`, `equity_declaration_write_decision`, `equity_issuance_session_view`), the record derivation (`equity_issued_recommendation`), the identity and the authorization validators are called unchanged. The adapter owns durability, transactions, locking, sequence allocation, rehydration and the database guards.

## 6. Storage and provenance as implemented

### 6.1 Objects

| Object | Key constraints |
| --- | --- |
| `equity_recommendation_issuance_ledger` | `recording_sequence` INTEGER PRIMARY KEY, at least 1; `record_kind` in (ISSUANCE, DECLARATION); UNIQUE (sequence, kind) |
| `equity_recommendation_issuances` | `issuance_id` primary key with `GLOB 'eiris1-sha256:?*'`; `generation_key` UNIQUE, foreign key to the EI-4D disposition; `eirp1` and `eira1` shapes; `issuance_kind` = MODELED; UNIQUE (strategy, symbol, exchange, date); `issued_at` and `issued_at_us`; UNIQUE sequence; constant kind with a composite foreign key to the ledger |
| `equity_recommendation_issuance_declarations` | Sequence primary key; constant kind with a composite foreign key to the ledger; session; `complete_through` and `complete_through_us`; UNIQUE (exchange, date, microseconds) |
| `equity_recommendation_issuances_by_session` | Index on (exchange, date) |

Additive: no existing object is altered, no trigger is added to an EI-4D table, and no schema version is kept.

### 6.2 What the database enforces, and what the adapter enforces

| Invariant | Database (constraint or trigger) | Adapter |
| --- | --- | --- |
| Identity, generation-key, session and sequence uniqueness | UNIQUE constraints; no-replace triggers | The Application rule decides first |
| Provenance | Foreign key to the disposition; trigger: PROPOSED, evaluator `/2`, same fingerprints, strategy, listing and date | Authorization validated before connecting; full re-derivation on load |
| No backdating | Trigger on integer microseconds: `issued_at_us` must exceed the session's greatest declared | The Application rule, inside `BEGIN IMMEDIATE` |
| Strict declaration monotonicity | Trigger on integer microseconds | The Application rule |
| Canonical instants | Trigger: canonical UTC text, and the integer equals its exact derivation (the EI-3C rule) | Integer computed without floats |
| Append-only | No UPDATE, DELETE or replacing INSERT on all three tables | Never issues an update or delete |
| `issued_at` equals the proposal's | **Not checked in SQL** | Re-derivation on load compares it (L4) |
| Ledger requirements | See section 7 | See section 7 |
| First committed record authoritative | Uniqueness | Check and insert under one write lock |

### 6.3 Writes

`write_issuance` and `write_declaration` first call the Application's public validator; a missing, foreign, unissued or changed authorization raises `UnverifiedEquityRecommendationIssuanceWriteError` before any connection opens. Then one `BEGIN IMMEDIATE`:

1. the whole-ledger check;
2. the relevant stored rows loaded and re-derived;
3. the Application rule decides;
4. only `ISSUED` or `DECLARED` allocates `max + 1` and inserts the ledger entry, then the record;
5. the pre-commit check;
6. one COMMIT.

Every other outcome inserts nothing and rolls back.

**Failure mapping:**

| Failure | Raised as |
| --- | --- |
| Open, read or lock failure; not the generation database | `EquityRecommendationIssuanceUnavailableError` |
| A constraint or trigger refusing a row the rules admitted | `InvalidEquityRecommendationIssuanceWriteError`; rolled back |
| COMMIT raising | `EquityRecommendationIssuanceCommitUnconfirmedError` |
| Corrupt stored state | Section 8 |

## 7. Sequence-ledger integrity

| # | Requirement (addendum 4.3) | Database | Adapter | Evidence |
| --- | --- | --- | --- | --- |
| 1 | No duplicate sequence | Primary key; UNIQUE; no-replace trigger | — | `test_sa06_…` (duplicate refused) |
| 2 | Valid ordering | `max + 1` trigger | Allocates `max + 1` under the write lock | `test_sa06_…` (out of order, and the next-sequence trigger on its own) |
| 3 | Kind matches the record | Composite foreign key; constant-kind CHECK | — | `test_sa06_…` (declaration under an ISSUANCE entry; a record claiming another kind) |
| 4 | No committed orphan entry accepted | **Not a database constraint.** A trigger refuses the next allocation while the previous sequence lacks its record | Pre-commit check: the new sequence is the maximum with exactly one record. Before every read and write: dense 1..N, each entry with its own record, each record with its entry | `test_sa06_…` (previous-recorded trigger); `test_sa07_…[orphan ledger entry]`; `test_corruption_met_reconciling_an_unconfirmed_commit_is_never_adopted` |
| 5 | No record without a valid entry | Composite foreign key (connections with foreign keys on) | The same whole-ledger check covers rows written with foreign keys off | `test_sa06_…` (no ledger entry); `test_sa07_…[record without its ledger entry]`, `[ledger kind mismatch]`, `[gap in the sequence]` |
| 6 | No partial allocation | One transaction | Guarded rollback; COMMIT once | `test_sa08_…[unexpected failure]`, `[SQLite failure]`: the allocation is visible inside the transaction, nothing persists, and the next write takes the same number |

**Planted orphan.** A raw, committed ledger row with no record makes every read (`session_view`, `session_view_at`, `load_issuance`, `load_session_issuance`, the history source) raise the `LEDGER` integrity error. The declarer raises it too, the issuer refuses with `STORED_AUTHORITY_INVALID`, and version-2 generation over the authority raises it. The planted state is unchanged afterwards. The database does not by itself prevent a committed orphan; the adapter refuses to accept one.

## 8. Corruption, provenance and source verification

### 8.1 D6-5 categories as implemented

| Category | Raised when | Evidence |
| --- | --- | --- |
| `ISSUANCE` (with `issuance_id`) | The referenced disposition is missing or does not decode; is not PROPOSED or not evaluator `/2`; the record does not re-derive; or a stored column (including `issued_at` or `issued_at_us`) disagrees with the re-derived record | `test_sa04_…[another issued instant]`, `[integer one microsecond off]`; `test_sa05_…` |
| `DECLARATION` | A declaration's text, date, instant or integer is invalid or inconsistent | `test_sa07_a_corrupt_declaration_…` |
| `LEDGER` | Section 7, requirements 4 and 5, and an invalid kind or sequence | `test_sa07_an_inconsistent_ledger_…` (4 cases) |

**The two refusals:**

- **`STORED_ISSUANCE_INVALID`** (writer outcome): the adapter reports it when the row under the *offered* identity fails re-derivation. It writes nothing, and the row is preserved. Evidence: `test_sa04_…`, where a genuine authorization is written again directly after tampering.
- **`STORED_AUTHORITY_INVALID`** (issuer refusal, permanent): through the issuer, the same corruption is met first in the history reads. Evidence: `test_sa04_…`, `test_sa07_…`.

Nothing is repaired: every tampering test asserts the stored state is unchanged afterwards.

### 8.2 Provenance and authorization evidence

| Case | Evidence |
| --- | --- |
| Missing disposition; NOT_ELIGIBLE disposition; evaluator `/1` disposition; another assessment or proposal fingerprint; another strategy, listing or date | `test_sa01_…`, each refused by the provenance trigger. The foreign key alone, with the trigger dropped, also refuses a missing disposition |
| Identity shape; MODELED only | `test_id03_s_…` |
| A verified authorization whose disposition is in another file | `test_a_verified_issuance_whose_disposition_is_not_in_this_file_…` → `InvalidEquityRecommendationIssuanceWriteError`; nothing written |
| Unauthorized write (none, foreign, wrong type, changed content), refused before any connection opens | `test_sa02_…`: the connection seam is not called and the file is byte-identical; the genuine captures then write normally |

### 8.3 Source verification versus canonical rehydration (the open review question)

**Finding: the load path performs ADR-021 level A on the referenced disposition, then re-derives and compares the issued record. It does not re-run levels B and C. That matches the governing contracts, so it is not an acceptance blocker.**

**What the load path calls** (`_stored_issuance`):

1. The referenced disposition row is decoded by the EI-4D decoder (`_stored_disposition`). Constructing `EquityRecommendationGenerationDisposition` verifies the evaluator label, verifies each document against its fingerprint (`decode_equity_recommendation_assessment`, `verified_equity_recommendation_document`), checks the evaluator label-format pairing (N6, at the Application layer) and the record's internal consistency. Every EI-4D column must equal what the record produces. This is level A (stored-document integrity).
2. The disposition must be PROPOSED, evaluator `/2`.
3. `equity_issued_recommendation(disposition)` derives the record and recomputes the `eiris1` identity.
4. Every stored issuance column must equal the derived record's.

**What it does not do on load:**

- **Level B:** no rehydration of the EI-3C candidate and EI-2B snapshot under EI3-D13. The EI-4D foreign keys and insert triggers only prove that those rows exist and matched when the disposition was recorded.
- **Level C:** no replay of the assessment.

**What governance requires:**

| Point | Required by | Implemented |
| --- | --- | --- |
| Levels A to C before issuance | ADR-024 section 4, item 2 | Yes, by the merged issuer (EI-5D) |
| On load, re-derive the issued record | EI-5 design section 10 ("Re-derive the issued record from the verified proposal"); addendum section 8, steps 1 to 6, which name exactly the EI-4D decoding and `equity_issued_recommendation` | Yes, step for step |
| Authority from the verified issuer plus re-derivation on every load | Addendum section 5.2 | Yes |

No governing document requires levels B and C at every authoritative read. A consumer that needs them for a stored issuance's source can run `VerifyStoredEquityRecommendationGenerationUseCase` on its generation key.

**If the owner wants levels B and C on every read,** that is a new requirement, not a defect. The smallest scope would be:

- an addendum amendment;
- an Application service composing the reader with the stored-generation verifier;
- a cost: a full replay per issuance per view.

It is recorded as limitation L7.

ADR-024 section 6 says a failure to read or verify raises `EquityRecommendationHistoryUnavailableError`. For corrupt stored state, D6-5 (2026-11-14) refines that: the typed integrity error propagates, permanent, as implemented in #97.

## 9. Acceptance evidence matrix

**Statuses:** VERIFIED; VERIFIED WITH LIMITATION. Every row was executed on real SQLite, on Windows, locally (section 11.1).

- **Paths:** S is `tests/test_sqlite_equity_recommendation_issuance.py`.
- **Process limitation:** processes are spawned on every platform. As in EI4-C2 and EI-4C PS-04 and PS-05, separate-process rows are VERIFIED WITH LIMITATION until Linux evidence is recorded.

### 9.1 EI-6A rows of the EI-5 matrix (layer S)

| ID | Requirement | Implementation | Evidence | Actual behaviour | Limitation | Status |
| --- | --- | --- | --- | --- | --- | --- |
| CC-01 | Two processes issue the same disposition | `BEGIN IMMEDIATE`; identity rule; uniqueness | S `test_cc01_…` | One `ISSUED`, one `EQUIVALENT`, both at sequence 2; one row; a third process's later retry is `EQUIVALENT` at 2 | Windows spawn only (L2) | VERIFIED WITH LIMITATION |
| CC-02 | Different policy sets, one strategy, listing and session | Session rule; UNIQUE (strategy, listing, date) | S `test_cc02_…`; `test_another_policy_set_…` | One `ISSUED`, one `SESSION_STRATEGY_ALREADY_ISSUED`; the stored row is the winner's (first committed stays authoritative) | Windows spawn only (L2) | VERIFIED WITH LIMITATION |
| CC-03 | Issuance concurrent with a covering declaration | No-backdating rule and trigger, under the write lock | S `test_cc03_an_issuance_racing_…`; `test_cc03_both_orderings_deterministically` (2) | Either `ISSUED` with a lower sequence and listed, or `BACKDATED_ISSUANCE` with no row; both orderings shown deterministically | One ordering per spawned run; Windows spawn only (L2) | VERIFIED WITH LIMITATION |
| CC-04 | Concurrent declarations | Monotone rule and trigger | S `test_cc04_…` | The higher one `DECLARED`; the lower one `DECLARED` before it or `COMPLETENESS_REGRESSION`; instants strictly increasing; ledger dense | Windows spawn only (L2) | VERIFIED WITH LIMITATION |
| CC-05 | Write lock not acquired | Busy timeout; `BEGIN IMMEDIATE` | S `test_cc05_…` | Issuer: retryable `AUTHORITY_UNAVAILABLE`; declarer: typed unavailable; nothing written; reads continue | Timeout shortened by the test seam | VERIFIED |
| CC-06 | COMMIT raises after taking effect | Commit-unconfirmed error; Application reload | S `test_cc06_…` | Issuance `EQUIVALENT`, `reconciled`, one write, sequence 2; declaration `DECLARATION_EQUIVALENT`, `reconciled` | — | VERIFIED |
| RR-03 | Views identical at every watermark after restart | Views from committed rows only | S `test_rr03_rr04_id02_…` | New adapter instances reproduce every watermark and the current view; the file is unchanged by reads | — | VERIFIED |
| RR-04 (views) | Replay of views from a separate OS process | As RR-03 | S `test_rr03_rr04_id02_…` | A fresh interpreter, from the file alone, reproduces every view | Windows only (L2); the state part is EI-6B | VERIFIED WITH LIMITATION |
| ID-02 (S) | Identity identical across processes and restart, on SQLite | Re-derivation on load | S `test_rr03_rr04_id02_…`; `test_cc01_…` | The fresh process loads the same identity, sequence and canonical content; racing processes agree | Windows only (L2) | VERIFIED WITH LIMITATION |
| ID-03 (S) | `GLOB 'eiris1-sha256:?*'` enforced | CHECK constraint | S `test_id03_s_…` | `eiri1-…`, an empty digest and junk refused; ACTUAL refused | — | VERIFIED |
| BD-01 (S) | 09:43:30 against a declaration through 10:30 (the BD-01 shape) | Rule and trigger | S `test_bd01_s_…`; boundary `test_bd02_s_…`; integer case `test_sa13_s_…` | Writer `BACKDATED_ISSUANCE`; a raw insert refused by the trigger, also at exactly `t` and where text order would admit | — | VERIFIED |
| HW-03 (S) | The EI5-D4 example keeps its view after later recording | Views at watermarks | S `test_hw03_s_…` | A (TCS) and B (INFY) issued at the same `t` (sequences 2, 3); a declaration (4) lists both; after a later declaration (5) the view at 4 is unchanged | A later issuance C cannot be constructed (L1) | VERIFIED WITH LIMITATION |
| HW-08 (S) | Authority unreadable | Unavailable mapping | S `test_hw08_s_…` | Non-database file and absent file: `EquityRecommendationHistoryUnavailableError`; no file created | — | VERIFIED |

### 9.2 EI-6A storage scenarios (SA)

| ID | Requirement | Evidence | Actual behaviour | Status |
| --- | --- | --- | --- | --- |
| SA-01 | Provenance enforced by the database | S `test_sa01_…` | Eight invalid provenance cases refused; the foreign key alone refuses a missing disposition | VERIFIED |
| SA-02 | Unauthorized writes refused before connecting | S `test_sa02_…` | Seven refusals; no connection opened; file byte-identical | VERIFIED |
| SA-03 | Append-only, no replacement, all three tables | S `test_sa03_…` | Nine UPDATE, DELETE and replacing INSERT statements refused; rows unchanged | VERIFIED |
| SA-04 | A tampered issuance | S `test_sa04_…` (2) | Every read raises `ISSUANCE` with the identity; writer `STORED_ISSUANCE_INVALID`; issuer `STORED_AUTHORITY_INVALID`; preserved | VERIFIED |
| SA-05 | An undecodable referenced disposition | S `test_sa05_…` | `ISSUANCE` on load and in views; nothing repaired | VERIFIED |
| SA-06 | Invalid ledger states refused by the database | S `test_sa06_…` | Duplicate, out-of-order, unapproved kind, mismatched kind, unlinked record and missing-previous-record cases refused | VERIFIED |
| SA-07 | A planted inconsistency fails closed | S `test_sa07_an_inconsistent_ledger_…` (4); `test_sa07_a_corrupt_declaration_…` | `LEDGER` or `DECLARATION` on every read and write; issuer `STORED_AUTHORITY_INVALID`; generation raises; unchanged | VERIFIED |
| SA-08 | Failure after allocation, before COMMIT | S `test_sa08_…` (2) | Nothing committed; next write takes sequence 2; no gap | VERIFIED |
| SA-09 | COMMIT without effect; reload failing | S `test_sa09_…`; `test_corruption_met_reconciling_…` | Bounded: `EquityRecommendationIssuanceUnresolvedError`, nothing claimed; corruption met while reconciling raises `LEDGER`, never adopted | VERIFIED |
| SA-10 | View semantics on SQLite | S `test_sa10_…` | Undeclared `None`; declared with nothing visible `()`; watermark beyond refused (durable HW-07 evidence); invalid watermarks refused; reads change nothing | VERIFIED |
| SA-11 | Declarations: equal and lower | S `test_sa11_…` | `DECLARATION_EQUIVALENT` and `COMPLETENESS_REGRESSION` write nothing; equal and lower raw inserts refused | VERIFIED |
| SA-12 | Additive initialization | S `test_sa12_…` | Only the 20 EI-6A objects (3 tables, 1 index, 16 triggers) added; existing objects, EI-4D rows and `user_version` unchanged; idempotent; a foreign or absent database refused, never created | VERIFIED |
| SA-13 | Integer-microsecond ordering | S `test_sa13_chronology_…`; `test_sa13_s_backdating_…` | `…:00Z` after `…:00.5Z` is a regression and refused by the trigger; a covering `…:30.5Z` declaration refuses `t`; exact integers, including before 1970 and year 0001; non-canonical text and integer mismatch refused | VERIFIED |

### 9.3 Totals

| Group | Rows | VERIFIED | VERIFIED WITH LIMITATION |
| --- | --- | --- | --- |
| EI-5 matrix rows, S parts (section 9.1) | 13 | 6 | 7 (CC-01, CC-02, CC-03, CC-04, RR-04 (views), ID-02 (S), HW-03 (S)) |
| Storage scenarios (section 9.2) | 13 | 13 | 0 |
| **Total** | **26** | **19** | **7** |

**Also tested, outside the matrix:**

- `test_an_issuance_is_durable_once_…`, `test_an_identical_retry_stays_equivalent_…` (identity before backdating), and the three architecture tests (`test_ar_…`, 4 cases): the adapter exposes exactly the port operations, each write entry validates first, and no production module constructs the adapter.
- `GENERATION_ALREADY_ISSUED` is covered at the Application contract level only (L5).

**Not here:** RR-01, the state part of RR-04, RR-05 and RR-06 (EI-6B, not authorized).

### 9.4 Reclassification after Linux CI (2026-11-14)

Sections 9.1 to 9.3 record the classification at engineering acceptance and are not rewritten.

**Why six rows change.** Their only limitation was Windows-only separate-process evidence (L2). Their tests are in `tests/test_sqlite_equity_recommendation_issuance.py`, which has no platform conditional, skip or xfail marker, and spawns processes with the `spawn` start method on every platform. The Infrastructure Linux run (section 11.3) executed that module with successful progress and no skip or failure in its section. All 43 cases, including the separate-process ones, therefore ran and passed on Linux as well as on Windows. The GitHub log shows the module's progress, not each test name.

| ID | Test | Before | After |
| --- | --- | --- | --- |
| CC-01 | `test_cc01_…` | VERIFIED WITH LIMITATION | VERIFIED |
| CC-02 | `test_cc02_…` | VERIFIED WITH LIMITATION | VERIFIED |
| CC-03 | `test_cc03_an_issuance_racing_…`; `test_cc03_both_orderings_deterministically` | VERIFIED WITH LIMITATION | VERIFIED (each spawned run still exercises one ordering; both are shown deterministically) |
| CC-04 | `test_cc04_…` | VERIFIED WITH LIMITATION | VERIFIED |
| RR-04 (views) | `test_rr03_rr04_id02_…` | VERIFIED WITH LIMITATION | VERIFIED (the state part stays EI-6B) |
| ID-02 (S) | `test_rr03_rr04_id02_…`; `test_cc01_…` | VERIFIED WITH LIMITATION | VERIFIED |
| HW-03 (S) | `test_hw03_s_…` | VERIFIED WITH LIMITATION | **Unchanged: VERIFIED WITH LIMITATION** (L1; section 10) |

**Final totals:**

| Group | Rows | VERIFIED | VERIFIED WITH LIMITATION | NOT VERIFIED |
| --- | --- | --- | --- | --- |
| EI-5 matrix rows, S parts | 13 | 12 | 1 (HW-03 (S)) | 0 |
| Storage scenarios SA-01 to SA-13 | 13 | 13 | 0 | 0 |
| **Total** | **26** | **25** | **1** | **0** |

## 10. HW-03 and the EI-5D HW-07 limitation

- **HW-03 (S).** The EI5-D4 example has a later issuance C. Once A is issued, EI4-D8 treats it as open, and every later same-session proposal is refused (`EXPOSURE_POLICY_UNRESOLVED`), so no C can be built through the real chain. The evidence instead uses two listings issued at the same modeled instant (neither in the other's assessment) and a later declaration. It shows the view at a watermark is unchanged by later recording. EI4-D8 is not relaxed.
- **HW-07.** `test_sa10_view_semantics_on_sqlite` shows the durable SQLite reader refusing `session_view_at` beyond its current position, before and after a declaration. In engineering terms this is the durable-reader evidence the EI-5D HW-07 limitation awaited.
  - **Not rewritten:** the EI-5D/EI-5E acceptance record and its owner decision are unchanged.
  - **Owner decision:** whether HW-07's limitation is closed is the owner's to decide.
  - **Platform:** the evidence is local Windows.

## 11. Test results and CI

### 11.1 Local results

All Windows. Python and the dependencies are as resolved by `uv` in the Infrastructure project.

| Point | Revision | Result |
| --- | --- | --- |
| EI-6A module (43 cases) | **Merged `63d2ab3`**, with Application `a17bd91` and Core `29bd34a`; run 2026-11-14 for this record, writing no bytecode or cache | **43 passed** |
| EI-6A module | The pre-merge working tree committed as `e756d7d` (same files and line counts) | 43 passed (reported) |
| Full Infrastructure suite | The pre-merge working tree | 3,336 passed, 27 skipped (reported) |
| Targeted mutations | The pre-merge working tree | 6 of 6 detected (reported): allocation `max + 2`; the next-sequence trigger disabled; the writer ignoring declarations; strict backdating trigger; the row under the offered identity not loaded; the provenance trigger ignoring the evaluator |
| Ruff check and format; `git diff --check` | The pre-merge working tree | Clean (reported) |

**Limitation.** No full Infrastructure run against the exact merged commit is recorded. Only the focused 43-case module was run on `63d2ab3` (L8).

### 11.2 D6-6 independent Linux CI

**NOT VERIFIED — evidence not available in this environment.** No GitHub CLI is installed, no CI artifact exists locally, and no credential was used or changed. No run identifier, URL or conclusion is recorded. A merged PR is not CI evidence.

| Required evidence | Status |
| --- | --- |
| **A.** `python-ci.yml` (job `test`, `ubuntu-latest`) for northstar-application #97, at its tested revision (head `00089fc` or merge `a17bd91`), with the Core commit it checked out | **MISSING** |
| **B.** `python-ci.yml` for northstar-infrastructure #67, at its tested revision (head `e756d7d` or merge `63d2ab3`), with the Application commit it checked out (must contain `a17bd91`) and the Core commit | **MISSING** |
| B: the 43 EI-6A cases ran | **MISSING** |
| B: the spawned-process cases ran on Linux, not skipped | **MISSING** (the module has no skip markers) |

**Evidence-collection step.** A person with repository access records, for each run:

- the run URL or identifier, the event and the conclusion;
- the head SHA tested;
- from the "Checkout northstar-core" and "Checkout northstar-application" step logs, the commits checked out (the workflow takes `develop` at run time);
- from the "Run Tests" log, the `tests/test_sqlite_equity_recommendation_issuance.py` progress line with 43 passing results and no `s` or `F`, and the summary line.

A run on `develop` at or after `63d2ab3` whose Application checkout contains `a17bd91` satisfies B.

**Reconciliation attempt, 2026-11-14.** A final D6-6 reconciliation was requested. No run identifier, URL, log or artifact for #97 or #67 was supplied or found in the workspace (including `NorthstarEvidence`, which holds only unrelated October milestone evidence). Every item in the table above therefore stays **MISSING**, and no limitation or status in this record changes.

### 11.3 Verified Linux CI evidence (2026-11-14)

Section 11.2 records the state before this evidence and is not rewritten.

**Source of the evidence.** The run and job identifiers, URLs, revisions and results were supplied and screenshot-verified by the owner on 2026-11-14. They were reconciled for this record against local Git and the repositories' workflow definitions. They were not fetched from this environment, which has no GitHub CLI or API access.

| Item | northstar-application #97 | northstar-infrastructure #67 |
| --- | --- | --- |
| Run ID | `43599364295` | `43604917993` |
| Job ID | `132460599656` | `132477386762` |
| URL | <https://github.com/Northstar-Dhruba/northstar-application/actions/runs/43599364295/job/132460599656> | <https://github.com/Northstar-Dhruba/northstar-infrastructure/actions/runs/43604917993/job/132477386762> |
| Tested revision | `a17bd91e308c298db8cce2896c0366279e4437cd` (the #97 merge; local `develop` HEAD) | `63d2ab3ac931c0bbb3b1a124fa17d9737030fe17` (the #67 merge; local `develop` HEAD) |
| Application dependency | — | `a17bd91e308c298db8cce2896c0366279e4437cd` (contains D6-5) |
| Core dependency | `29bd34ac97810e6b471ed6c2ba4618ebaa57d9c6` | `29bd34ac97810e6b471ed6c2ba4618ebaa57d9c6` |
| Platform; Python | Linux; 3.13.18 | Linux; 3.13.18 |
| Test job | SUCCESS | SUCCESS |
| Pytest | **4,674 passed**, no failures or skips reported; 47.30 s | **3,363 collected; 3,339 passed, 24 skipped**, no failures; 182.21 s |
| EI-6A module | Application issuance, generation and architecture tests in the suite | `tests/test_sqlite_equity_recommendation_issuance.py` shown with successful progress, no skip or failure in its section |

**Consistency checks:**

- **Revisions.** All four SHAs exist locally and equal the `develop` HEADs of their repositories. The #97 and #67 merge subjects name those PRs.
- **Dependencies.** Each workflow checks out its dependencies at `develop`; the recorded commits are those `develop` HEADs.
- **Counts.** Application's 4,674 equals the local full-suite count reported for the D6-5 change. Infrastructure's 3,363 collected equals the local Windows total (3,336 passed + 27 skipped); the Linux run skips 3 fewer.
- **Module.** The EI-6A module holds 43 cases with no skip or platform marker. Individual scenario names are not claimed to appear in the GitHub log.

**Workflow identity.** At both tested revisions, `.github/workflows/python-ci.yml` is the **only** workflow file. It is named "Python CI" and has one job, `test`, which runs on `ubuntu-latest`, installs Python 3.13 with `uv`, checks out its dependencies at `develop`, runs `ruff check`, `ruff format --check` and `pytest`. The evidence (a Linux test job, Python 3.13.18, these dependency commits) matches it, so the runs are `python-ci.yml` runs.

- **Qualification.** This identity is established from the repositories' workflow inventory, not from run metadata displaying the filename, which this environment cannot access.
- **Not shown.** The supplied evidence shows the pytest step; it does not show the Ruff steps individually. The job's SUCCESS covers them.

**D6-6: SATISFIED.**

- Local Windows real-SQLite acceptance: section 11.1.
- Independently verified successful Linux CI for the D6-5 Application prerequisite and for the EI-6A Infrastructure implementation, at their actual merged revisions: this section.
- The workflow-identity qualification is the only residual one.

EI-4 formal CI closure (EI4-C4) stays a separate obligation; these runs do not close it unless they meet the [EI-4 Decision Register, section 15.3](Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4).

## 12. Remaining limitations and blockers

**Acceptance blocker:**

- **B1 — D6-6 CI.** Neither required Linux run is evidenced (section 11.2). Owner acceptance stays PENDING until both are.

**Belonging to EI-6A:**

- **L1 — HW-03 (S).** A later issuance cannot be constructed under EI4-D8; the evidence uses two same-instant listings and a later declaration (section 10).
- **L2 — Windows-only process evidence.** CC-01 to CC-04, RR-04 (views) and ID-02 (S) ran on Windows only; POSIX spawn behaviour awaits B1.
- **L3 — Ledger requirement 4** relies on the adapter: the pre-commit check and the fail-closed whole-ledger checks. The database refuses the next allocation after an orphan but does not prevent a committed orphan.
- **L4 — Issued instant.** SQL does not compare `issued_at` with the stored proposal; re-derivation on load does.
- **L5 — `GENERATION_ALREADY_ISSUED`** cannot be produced through the real chain on SQLite (one generation key derives one identity); it is covered at the Application contract level.
- **L6 — Ordering precondition.** The EI-4D generation tables must exist before the first EI-6A operation (addendum section 2); the tests establish them through the EI-4D store.
- **L7 — Load-time verification** is level A plus re-derivation, not levels B and C, as governed (section 8.3).
- **L8 — Local evidence.** The full suite and mutations ran on the pre-merge working tree; only the 43-case module ran on the merged commit.
- **L9 — Test seams.** CC-05, CC-06, SA-08 and SA-09 use module-level seams (`_BUSY_TIMEOUT_SECONDS`, `_commit`, `_connect`, `_confirm_recorded`); tampering uses labelled raw SQL on disposable files.

**Carried forward (unchanged):**

- **L10 — EI4-D8:** conservative exposure and exclusivity.
- **L11 — VC-04:** rollback or downgrade to a pre-3a Application against stored evaluator-v2 records is not authorized. More consequential now that issuances reference evaluator-v2 dispositions.
- **L12 — N6:** the database-level label-format pairing trigger stays deferred. EI-6A's `/2` check on the referenced disposition is separate from it.
- **L13 — I08:** the equivalent-mutant qualification.
- **L14 — IS-16 and NB-04** (EI-5D, EI-5E).
- **L15 — HW-07:** still recorded as a limitation in the EI-5D record; durable evidence is now available (section 10).
- **L16 — EI-4 closure:** formal CI closure (EI4-C4) pending, separately.
- **L17 — Deferred decisions:** O-2, O-3 and EI3-D10 deferred; EI-6B and EI-6C not authorized; ACTUAL issuance unauthorized.

### 12.1 Limitation reconciliation (2026-11-14)

The list above records the state at engineering acceptance and is not rewritten.

**Resolved or addressed (historical limitations):**

| Item | Disposition | Evidence |
| --- | --- | --- |
| B1 — D6-6 CI | **RESOLVED.** Both Linux runs are recorded; workflow identity is qualified as stated | Section 11.3 |
| L2 — Windows-only process evidence | **RESOLVED.** CC-01 to CC-04, RR-04 (views) and ID-02 (S) also ran and passed on Linux | Infrastructure run `43604917993`; section 9.4 |
| L8 — No full run on the merged commit | **ADDRESSED.** The full Infrastructure suite passed on Linux at merged `63d2ab3`, and the full Application suite at merged `a17bd91`. No full local Windows run on the merged commit is recorded; none is now required | Runs `43599364295`, `43604917993` |
| L15 — EI-5D HW-07 | **ADDRESSED (owner approved, 2026-11-14).** SA-10 is real-SQLite evidence that `session_view_at` beyond the authority's current position is refused. The EI-5D acceptance record and its original limitation are preserved, with a dated cross-reference | Section 10; `test_sa10_view_semantics_on_sqlite` |

**Remaining (accepted):**

| Item | Limitation |
| --- | --- |
| L1 | HW-03 (S): a later same-session issuance cannot be constructed under EI4-D8; the evidence uses two same-instant listings and a later declaration |
| L3 | Orphan-ledger integrity relies partly on the adapter (pre-commit and whole-ledger checks); the database alone does not prevent a committed orphan |
| L4 | The issued instant is checked against the proposal only by re-derivation on load |
| L5 | `GENERATION_ALREADY_ISSUED` is evidenced at the Application contract level only |
| L6 | The EI-4D generation tables must exist before the first EI-6A operation |
| L7 | Load performs the governed level-A validation of the referenced disposition and re-derivation; levels B and C run before issuance |
| L9 | CC-05, CC-06, SA-08 and SA-09 use test seams; tampering uses labelled raw SQL |
| — | D6-6 workflow identity: established from the workflow inventory, not from run metadata (section 11.3) |

**Carried forward, unchanged:**

| Item | Limitation |
| --- | --- |
| L10 | EI4-D8 conservative exposure and exclusivity |
| L11 | VC-04: rollback or downgrade to a pre-3a Application against stored evaluator-v2 records not authorized |
| L12 | N6: database-level label-format pairing trigger deferred |
| L13 | I08: the equivalent-mutant qualification |
| L14 | IS-16 (EI-5D) and NB-04 (EI-5E) |
| L16 | EI-4 formal CI closure (EI4-C4) pending, as a separate obligation |
| L17 | O-2, O-3 and EI3-D10 unresolved; EI-6B and EI-6C not authorized; ACTUAL issuance unauthorized |

## 13. Engineering verdict and recommendation

**EI-6A LOCAL ENGINEERING ACCEPTANCE: PASS WITH DOCUMENTED LIMITATIONS — synthetic, local Windows real-SQLite engineering acceptance.**

- The implementation is merged (#67 on #97). Every EI-6A scenario row is evidenced on real SQLite; 7 carry a documented limitation; none is NOT VERIFIED.
- The source-verification question is resolved: the load path matches the governed contract, and no verification step is missing (section 8.3).
- **D6-6 is not satisfied** (B1).

**Recommended owner decision:** do **not** accept EI-6A yet.

1. Collect and record the two Linux CI runs (section 11.2).
2. Then decide on acceptance with documented limitations L1 to L17.
3. Decide separately whether the EI-5D HW-07 limitation is closed by the durable evidence of section 10.

The recommendation above records the state before the owner decided. The owner decided on 2026-11-14 (section 13.1).

### 13.1 Owner acceptance (2026-11-14)

**OWNER APPROVED — 2026-11-14.**

**EI-6A — Durable SQLite Recommendation Issuance and Declaration Authority: ACCEPTED WITH DOCUMENTED LIMITATIONS.**

**Scope.** The durable SQLite MODELED recommendation issuance and completeness-declaration authority, exactly as authorized by D6-1 to D6-6.

| Stage | Record |
| --- | --- |
| Implementation merged | northstar-application #97 (`a17bd91`), the D6-5 prerequisite; northstar-infrastructure #67 (`63d2ab3`) |
| Local Windows engineering acceptance | PASS WITH DOCUMENTED LIMITATIONS (sections 9 and 11.1) |
| Independent Linux CI (D6-6) | SATISFIED: runs `43599364295` and `43604917993` (section 11.3), with the workflow-identity qualification |
| Acceptance matrix (26) | 25 VERIFIED, 1 VERIFIED WITH LIMITATION (HW-03 (S)), 0 NOT VERIFIED (section 9.4) |
| Owner acceptance | **ACCEPTED WITH DOCUMENTED LIMITATIONS**, 2026-11-14 |
| EI-5D HW-07 | Durable-reader verification gap recognized as addressed through SA-10 (owner approved); the EI-5D record is preserved with a dated cross-reference |

**Accepted limitations:** the remaining and carried-forward items of section 12.1.

**Not authorized by this acceptance:** EI-6B lifecycle persistence; EI-6C monitoring; the resolution of O-2, O-3 or EI3-D10; ACTUAL issuance; broker orders or fills; real-money trading; production deployment; API or Web exposure; composition roots or monitoring execution; a relaxation of EI4-D8; a new canonical issuance format; changes to historical EI-4D evidence. It does not close EI-4. **EI-6B and EI-6C each need a separate owner authorization.**

The owner record is the [EI-5 Decision Register, section 18.4](Equity-Intraday-V1-EI5-Decision-Register.md#184-owner-acceptance-of-ei-6a-2026-11-14).

## 14. Non-authorizations

Neither this record nor the owner acceptance of section 13.1 authorizes anything further: not EI-6B lifecycle persistence, EI-6C monitoring, the resolution of O-2, O-3 or EI3-D10, ACTUAL issuance, broker orders or fills, real-money trading, production deployment, API or Web exposure, composition roots or monitoring execution, a relaxation of EI4-D8, a new canonical issuance format, or any change to historical EI-4D evidence. It does not close EI-4. EI-6A remains single-host and MODELED-only.

## 15. Related documents

- [EI-5 Decision Register](Equity-Intraday-V1-EI5-Decision-Register.md), sections 16 and 17
- [EI-6A Durable Issuance Authority SQLite Storage Design v1.0](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design v1.0](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [EI-5D/EI-5E Engineering Acceptance Record](Equity-Intraday-V1-EI5D-EI5E-Modeled-Issuance-and-Synthetic-Integration-Engineering-Acceptance-Record.md)
- [EI-4C Versioned History Compatibility Engineering Acceptance Record](Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md), section 10
- [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md), section 15
