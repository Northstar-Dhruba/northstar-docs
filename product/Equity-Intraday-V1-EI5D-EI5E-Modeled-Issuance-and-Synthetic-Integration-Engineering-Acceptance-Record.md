# Equity Intraday V1 — EI-5D/EI-5E Modeled Issuance and Synthetic Integration Engineering Acceptance Record

## Status

**Engineering acceptance record, 2026-11-14.** This is Docs PR-2 of the EI-5D and EI-5E plan authorized in the [EI-5 Decision Register, section 15](Equity-Intraday-V1-EI5-Decision-Register.md#15-owner-authorization-of-ei-5d-and-ei-5e-2026-11-14).

- **Verdict:** EI-5D/EI-5E ENGINEERING ACCEPTANCE: **PASS WITH DOCUMENTED LIMITATIONS — synthetic engineering acceptance** ([section 13](#13-engineering-verdict-and-recommendation)).
- **Owner acceptance: ACCEPTED WITH DOCUMENTED LIMITATIONS, 2026-11-14** (northstar-application #96, merge `9a68209`). Every scenario status and limitation below is retained ([section 13.1](#131-owner-acceptance-2026-11-14)).
- **What it covers:** the combined EI-5D (MODELED recommendation issuance) and EI-5E (synthetic end-to-end integration) implementation, merged as northstar-application #96 ([section 4](#4-repository-baselines)).
- **Acceptance matrix:**
  - **EI-5D parts:** 38 scenarios — **36 VERIFIED, 2 VERIFIED WITH LIMITATION**;
  - **EI-5E parts:** 7 — **6 VERIFIED, 1 VERIFIED WITH LIMITATION**;
  - **EI-6-owned:** 11 scenarios and 5 S-layer parts stay **PLANNED** ([section 9](#9-acceptance-evidence-matrix)).
- **CI verification: NOT INDEPENDENTLY VERIFIED FROM AVAILABLE EVIDENCE** ([section 11](#11-test-results-and-ci)).
- Sections 1 to 13 are engineering evidence, **not** themselves an owner approval; the owner's acceptance of 2026-11-14 is recorded in [section 13.1](#131-owner-acceptance-2026-11-14) and the [EI-5 Decision Register, section 16.3](Equity-Intraday-V1-EI5-Decision-Register.md#163-owner-acceptance-of-ei-5d-and-ei-5e-2026-11-14). D5D-1 to D5D-5 (2026-11-14) and every earlier EI-5 and ADR-024 decision are unchanged. It does **not** authorize EI-6, a SQLite issuance authority, ACTUAL issuance or any production use ([section 14](#14-non-authorizations)).

## 1. Acceptance identity

| Item | Value |
| --- | --- |
| Milestone | EI-5D MODELED Recommendation Issuance + EI-5E Synthetic End-to-End Integration (one Application PR, D5D-2) |
| Record | Docs PR-2, engineering acceptance |
| Date | 2026-11-14 |
| Governance | [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), sections 2 to 7 and 12; [EI-5 Decision Register, section 15](Equity-Intraday-V1-EI5-Decision-Register.md#15-owner-authorization-of-ei-5d-and-ei-5e-2026-11-14) |
| Design | [EI-5 Issuance and Lifecycle Design v1.0](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md), sections 7, 8 and 12 |
| Data | Synthetic only |
| Authority | A labelled in-memory **test** authority; no durable authority exists |
| Platform | Windows development machine; local executions only |

## 2. Scope

**In scope:** the Application issuance contracts (reader and writer ports, outcomes, typed errors), the `EquityIssuedRecommendation` record and its `eiris1` identity, the content-bound issuance and declaration authorizations, the issuer and declaration use cases, the authoritative history source, the authority invariants as pure functions, and the synthetic end-to-end chain from a verified version-2 proposal through issuance, issued history and the EI-5C lifecycle.

**Out of scope:** the SQLite issuance authority and every real-SQLite (S-layer) scenario (EI-6), lifecycle persistence, monitoring, composition roots, ACTUAL issuance, actionable output, orders, fills, positions, broker or account integration, API, Web, real data, and any change to Core, Infrastructure, EI-5B or EI-5C.

## 3. Governance references

| Layer | State | Where |
| --- | --- | --- |
| Owner-approved policy | EI5-D1 to EI5-D8, C-1, C-2, O-1 (2026-11-12) | [EI-5 Decision Register, sections 4 and 12](Equity-Intraday-V1-EI5-Decision-Register.md#12-owner-approval-of-the-ei-5a-amendments-2026-11-12) |
| Owner-accepted prerequisite | EI-4C versioned compatibility, accepted with documented limitations (2026-11-14) | [EI-5 Decision Register, section 14](Equity-Intraday-V1-EI5-Decision-Register.md#14-ei-4c-compatibility-implementation-and-engineering-acceptance-2026-11-14) |
| Owner authorization | D5D-1 to D5D-5 (2026-11-14) | [EI-5 Decision Register, section 15](Equity-Intraday-V1-EI5-Decision-Register.md#15-owner-authorization-of-ei-5d-and-ei-5e-2026-11-14) |
| Implemented behaviour | northstar-application #96 | [Section 5](#5-implementation-inventory) |
| Local evidence | Reported suite results | [Section 11](#11-test-results-and-ci) |
| Remaining limitations | L1 to L10 | [Section 12](#12-remaining-limitations) |

## 4. Repository baselines

Read from local Git on 2026-11-14. Every repository was on `develop` with a clean working tree; no fetch was run.

| Repository | `develop` HEAD | Role |
| --- | --- | --- |
| northstar-application | `9a68209` (#96; change commit `b2c81a2`, parents `d908674`, `b2c81a2`) | EI-5D and EI-5E |
| northstar-docs | `5c8f453` (#61) | Authorization; this record, uncommitted |
| northstar-core | `29bd34a` (#40) | Unchanged; EI-5B lifecycle last changed `9e06d1e` (#39) |
| northstar-infrastructure | `f71f8f7` (#66) | Unchanged; references none of the EI-5D names |
| northstar-api | `7ad37f5` | Unchanged |
| northstar-web | `5aeb94c` | Unchanged |

**Change scope of `b2c81a2`:** 13 files, 3,291 insertions, 6 deletions — seven new Application production modules, additive exports in the two package `__init__` files, two new test modules, and narrow updates to two existing tests (section 10). EI-5C's `evaluate_synthetic_equity_recommendation_lifecycle.py` is unchanged since `f29dbc1` (#93).

## 5. Implementation inventory

Paths are under northstar-application `src/northstar_application/`.

| Component | Module | Responsibility |
| --- | --- | --- |
| Ports, outcomes, errors | `ports/equity_recommendation_issuance.py` | `EquityRecommendationIssuanceReader` (`load_issuance`, `load_session_issuance`, `session_view`, `session_view_at`); `EquityRecommendationIssuanceWriter` (`write_issuance`, `write_declaration`, authorizations only); the six issuance and three declaration outcomes; unavailable, commit-unconfirmed, watermark-beyond and unverified-write errors |
| Records and identity | `application_services/equity_recommendation_issuance_records.py` | `EquityIssuedRecommendation`, `equity_issued_recommendation`, `EquityRecordedIssuance`, the declaration records, `equity_recommendation_issuance_identity`, the authority reference constant |
| Invariants | `application_services/equity_recommendation_issuance_rules.py` | Pure functions: the session view at a watermark, `CT(W)`, backdating, the issuance write decision, the declaration write decision |
| Authorizations | `application_services/equity_recommendation_issuance_authorization.py` | `VerifiedEquityRecommendationIssuance`, `VerifiedEquityIssuanceCompletenessDeclaration`, private issuers and registry, public validators, the venue-day check |
| Issuer | `application_services/issue_equity_trade_recommendation.py` | `IssueEquityTradeRecommendationUseCase`, typed refusals, the result, commit reconciliation |
| Declarer | `application_services/declare_equity_session_issuance_completeness.py` | `DeclareEquitySessionIssuanceCompletenessUseCase` |
| History source | `application_services/authoritative_equity_session_recommendation_history.py` | `AuthoritativeEquitySessionRecommendationHistorySource` over the reader only |

Tests: northstar-application `tests/application_services/test_issue_equity_trade_recommendation.py` (55 test functions, 63 cases) and `test_equity_recommendation_issuance_end_to_end.py` (5).

## 6. Issuance behaviour as implemented

### 6.1 The issuer's order of checks

`IssueEquityTradeRecommendationUseCase.execute(generation_key, issuance_kind=MODELED)` has no other parameter:

1. **MODELED only.** `ACTUAL` raises `ACTUAL_ISSUANCE_NOT_AUTHORIZED` before any read.
2. **Re-verification** through `VerifyStoredEquityRecommendationGenerationUseCase` (levels A to C). No disposition, invalid, incompatible, source-invalid and replay-mismatch records are permanent refusals; an unavailable generation store or source evidence is retryable. Nothing is written; the stored record is preserved.
3. **`PROPOSED` only.**
4. **Evaluator `/2` only.** A version-1 disposition is refused (`EVALUATOR_NOT_ISSUABLE`) and preserved.
5. **Authoritative history.** The frozen evidence must name `northstar.equity-recommendation-issuance-authority/1` (D5D-3), and `session_view_at(recorded watermark)` must reproduce it exactly (`EquityIssuedHistoryEvidence.as_of_view` at `b`).
6. **Completeness pre-check.** The current view must exist and be declared at least through `b = t − 1 µs`; otherwise `AUTHORITATIVE_COMPLETENESS_NOT_DECLARED` (D5D-4). A missing declaration is never an empty history.
7. **Derive, authorize, write once**, then map the writer's outcome.

Refusals before a write raise `EquityRecommendationIssuanceRefusedError` with `reason` and `retryable`.

### 6.2 Where exact completeness and no-backdating are enforced

- **The issuer** requires a declaration at least through `b` and reproducible evidence at its watermark. It does **not** itself refuse a current declaration beyond `b`.
- **The writer** applies the no-backdating rule (`issued_at <= max declared complete_through` is refused) in its own serialized write. Because `PointInTime` has microsecond resolution, no instant lies strictly between `b` and `t`: a declaration beyond `b` is at or after `t`, so a **new** issuance cannot commit once the history has advanced beyond `b` — it is `BACKDATED_ISSUANCE`.
- **The writer matches the stored identity before backdating.** A previously committed identical issuance therefore returns `EQUIVALENT` after later declarations.
- **In EI-5D this writer is the in-memory test authority** applying the production rules; the durable, transactional enforcement is EI-6's.

### 6.3 Outcomes and commit uncertainty

- **Outcomes:** `ISSUED`, `EQUIVALENT`, `GENERATION_ALREADY_ISSUED`, `SESSION_STRATEGY_ALREADY_ISSUED`, `BACKDATED_ISSUANCE`, `STORED_ISSUANCE_INVALID`. A writer answer whose recorded content contradicts the offered record is never adopted (`STORED_ISSUANCE_INVALID`).
- **Commit uncertainty:** reload by identity with bounded retries. Equal record: `EQUIVALENT`, reconciled; none: retry; different: `STORED_ISSUANCE_INVALID`; unreadable or exhausted: `EquityRecommendationIssuanceUnresolvedError` (nothing claimed).
- **Declarations:** `DECLARED`, `DECLARATION_EQUIVALENT`, `COMPLETENESS_REGRESSION`; venue day, NORMAL_SESSION and local-calendar-day checks; commit uncertainty reconciled by the current view.

## 7. Identity, authority reference and authorization

- **Identity:** `eiris1-sha256:` + SHA-256 over the length-prefixed `northstar.equity-recommendation-issuance-key/1`, the `eirk1` key, the `eirp1` fingerprint and `MODELED`, using the `eirk1` encoder. Golden vectors computed independently (plain `hashlib`): `eiris1-sha256:545abd237c87b33a33afb633f3b326006be736859eacb884e0dbd36a6d04d655` (synthetic parts) and `eiris1-sha256:3954fcc79f33df12abefbb9f3d504815436ff708134bd4ab2adc5cd0ea9dc9cb` (the EI-4C fixture's key and proposal).
- **Authority reference:** the constant `northstar.equity-recommendation-issuance-authority/1`, frozen into every authoritative version-2 inputs document, checked by the issuer and by the history source.
- **Authorization, by layer:**

| Layer | What enforces it | Status |
| --- | --- | --- |
| Application | Issuer-only and declarer-only construction (private registry); exact content binding; validators re-derive the record from a verified v2 PROPOSED disposition; architecture scans | Implemented and tested |
| In-memory test authority | Validates authorizations first; applies the production rules to append-only lists | Test double only — not durable |
| SQLite (EI-6) | Transactions, constraints, foreign keys, pre-connection refusal | **Not implemented** |

## 8. Synthetic lifecycle integration (EI-5E)

`test_equity_recommendation_issuance_end_to_end.py` runs the real components in memory:

1. the authority declared complete through `b` (sequence 1);
2. version-2 generation against the authoritative history source;
3. recording through the verified EI-4D recorder;
4. re-verification at levels A to C;
5. issuance `ISSUED` at sequence 2;
6. history: not listed while complete only through `b`; listed at watermark 3 after a further declaration; the view at watermark 2 unchanged;
7. the EI-5C evaluation, under the issued record's generation key, with a window equal to the issued levels and timing;
8. the Core lifecycle starting `RECORDED`;
9. `WAITING_FOR_ENTRY`, then `ACTIVE` under EI-5B rules;
10. EI4-D8: after the issued recommendation's lifecycle closes at its stop, a later proposal (10:06 IST) is `EVALUATION_UNRESOLVED` (`EXCLUSIVITY_POLICY_UNRESOLVED`) and is not issuable.

**Not established:** a durable or authoritative issued lifecycle. EI-5C keeps its synthetic label (`issuance_established` and `is_authoritative` are always False); the link to the issued record is the generation key, checked for consistency, and no lifecycle event is persisted.

## 9. Acceptance evidence matrix

**Statuses:** VERIFIED; VERIFIED WITH LIMITATION; PLANNED (owned by EI-6). **Paths:** C is `test_issue_equity_trade_recommendation.py`, E is `test_equity_recommendation_issuance_end_to_end.py`. Rows the design gives several layers are assessed for their EI-5D or EI-5E part; S-layer parts are listed in section 9.3.

### 9.1 EI-5D (layers C and A)

| ID | Requirement | Evidence | Limitation | Status |
| --- | --- | --- | --- | --- |
| IS-01 | Verified PROPOSED: `ISSUED`, MODELED; state `RECORDED` | C `test_is01_…` (MODELED, sequence 2, record derived from the disposition); `RECORDED` shown at E (section 9.2) | — | VERIFIED |
| IS-02 | Issue again: `EQUIVALENT`, nothing written | C `test_is02_…` (also after a later declaration) | — | VERIFIED |
| IS-03 | Another policy set, same strategy, listing, session | C `test_is03_…` (`SESSION_STRATEGY_ALREADY_ISSUED`; first stays) | — | VERIFIED |
| IS-04 | Another strategy, same listing and session | C `test_is04_…` (second strategy's real timeline; `ISSUED` at 3) | — | VERIFIED |
| IS-05 | NOT_ELIGIBLE refused, no write | C `test_is05_…` | — | VERIFIED |
| IS-06 | UNRESOLVED refused, no write | C `test_is06_…` | — | VERIFIED |
| IS-07 | Unknown key refused, no write | C `test_is07_…` | — | VERIFIED |
| IS-08 | Invalid at level A: permanent refusal, preserved, no write | C `test_is08_…` (undecodable, incompatible; and unavailable as retryable) via the EI-4D-A store double's typed failures | — | VERIFIED |
| IS-09 | Source unavailable: retryable, no write | C `test_is09_…` (and a removed source as permanent) | — | VERIFIED |
| IS-10 | Replay mismatch at level C: permanent | C `test_is10_…` | — | VERIFIED |
| IS-11 | `ACTUAL` refused before any read | C `test_is11_…` (no store or authority read) | — | VERIFIED |
| IS-12 | Non-actionable; `not_assessed` identical | C `test_is12_…` | — | VERIFIED |
| IS-13 | Timing equals the proposal's; no caller parameter | C `test_is13_…` (instants; signature) | — | VERIFIED |
| IS-14 | Writer call without verified authorization refused | C `test_is14_…`; A `test_ar_only_the_two_use_cases_call_the_writer_or_issue_authorizations` | — | VERIFIED |
| IS-15 | Swapped-content authorization refused | C `test_is15_…` (copy; tampered record) | — | VERIFIED |
| IS-16 | Version-1; fixture history; not as of `t − 1 µs`; not reproducing; declared beyond `t − 1 µs` | C `test_is16_a_version1_…`, `test_is16_v2_history_from_a_fixture_…`, `test_is16_v2_history_that_does_not_reproduce_…` (×2), `test_is16_a_session_declared_beyond_b_…` (writer `BACKDATED_ISSUANCE`) | "Not taken as of `t − 1 µs`" is not exercised at the issuer: such a version-2 inputs document cannot be constructed (EI-4C HB-09, itself VERIFIED WITH LIMITATION) | VERIFIED WITH LIMITATION |
| IS-17 | EI-4D records byte-identical; no disposition `RECORDED` | C `test_is17_…` | — | VERIFIED |
| ID-01 | Golden vector | C `test_id01_…` (two vectors, independently computed) | — | VERIFIED |
| ID-02 | Identical in spawned processes and after restart (C part) | C `test_id02_…` (two fresh interpreters, different hash seeds) | S part: section 9.3 | VERIFIED |
| ID-03 | Prefix `eiris1-sha256:`; GLOB behaviour (C part) | C `test_id03_…` (`fnmatch` with GLOB semantics) | S part: section 9.3 | VERIFIED |
| ID-04 | Each part changes the identity | C `test_id04_…` | — | VERIFIED |
| ID-05 | Shifted part boundary changes the identity | C `test_id05_…` | — | VERIFIED |
| BD-01 | 10:10 against 10:30: backdated (C part) | C `test_bd01_bd02_bd03_…`, `test_bd01_at_the_issuer_…` | S part: section 9.3 | VERIFIED |
| BD-02 | `issued_at` equal to `complete_through` | C `test_bd01_bd02_bd03_…`; `test_is16_a_session_declared_beyond_b_…` (declared exactly through `t`) | — | VERIFIED |
| BD-03 | 10:40 against 10:30; `t` against exactly `t − 1 µs` | C `test_bd01_bd02_bd03_…`, `test_bd03_at_the_issuer_…` | — | VERIFIED |
| BD-04 | No declaration (writer contract, D5D-4) | C `test_bd04_…`; issuer refusal `test_d5d4_…` | — | VERIFIED |
| HW-01 | No declaration: `None`, never empty | C `test_hw01_…` | — | VERIFIED |
| HW-02 | Declaration, no issuance: empty tuple | C `test_hw02_…` | — | VERIFIED |
| HW-03 | EI5-D4 example stable (C part) | C `test_hw03_…` (production view function over constructed records) | S part: section 9.3 | VERIFIED |
| HW-04 | View content and order | C `test_hw04_…` | — | VERIFIED |
| HW-05 | Regression; equal declaration | C `test_hw05_…` | — | VERIFIED |
| HW-06 | No sequence or watermark parameter | C `test_hw06_…` | — | VERIFIED |
| HW-07 | `session_view_at` beyond the watermark refused | C `test_hw07_…`; issuer mapping in `test_is16_v2_history_that_does_not_reproduce_…[watermark beyond]` | The refusal itself is a reader-port obligation implemented only by the test authority; production enforcement is EI-6's reader | VERIFIED WITH LIMITATION |
| HW-08 | Authority unreadable (C part) | C `test_hw08_an_unreadable_…`, `test_hw08_a_view_not_naming_…`; issuer `test_an_unavailable_authority_…` | S part: section 9.3 | VERIFIED |
| HW-09 | Every view passes the history contract | C `test_hw09_…` | — | VERIFIED |
| NB-01 | No order, fill, quantity, account, P&L or exit price (C, A parts) | C `test_nb01_…` | K part is EI-5B's | VERIFIED |
| NB-03 | Issuance packages import no execution module (A part) | A `test_nb03_ar_…` | — | VERIFIED |
| AR-03 | Futures and Options suites unchanged and passing | Full Application and Infrastructure suites (section 11) | Local only | VERIFIED |

**Totals (EI-5D): 38 — 36 VERIFIED, 2 VERIFIED WITH LIMITATION (IS-16, HW-07).**

### 9.2 EI-5E (layer E)

| ID | Requirement | Evidence | Limitation | Status |
| --- | --- | --- | --- | --- |
| IS-01 (E) | Issued end to end; state `RECORDED` | E `test_e2e_a_verified_v2_proposal_…` | — | VERIFIED |
| IS-02 (E) | Retry `EQUIVALENT` end to end | E `test_e2e_replay_…` (after a later declaration) | — | VERIFIED |
| IS-03 (E) | Second policy set refused end to end | E `test_e2e_a_second_policy_set_…` | — | VERIFIED |
| IS-17 (E) | EI-4D records unchanged end to end | E `test_e2e_replay_…` | — | VERIFIED |
| HW-10 | A lifecycle-closed issuance stays open for EI-4C | E `test_e2e_hw10_…` | — | VERIFIED |
| NB-04 | `ACTIVE` creates no position or paper record | E `test_e2e_an_active_recommendation_…` | Shown within the synthetic world (authority and generation store unchanged; no paper store is wired), plus the issuance-module import scan | VERIFIED WITH LIMITATION |
| RR-02 (E) | Same snapshots from `RECORDED`: identical events | E `test_e2e_replay_…` | — | VERIFIED |

**Totals (EI-5E): 7 — 6 VERIFIED, 1 VERIFIED WITH LIMITATION (NB-04).**

### 9.3 EI-6-owned (not complete)

| Item | Status |
| --- | --- |
| CC-01 to CC-06 | PLANNED (EI-6) |
| RR-01, RR-03, RR-04, RR-05, RR-06 | PLANNED (EI-6) |
| S parts of ID-02, ID-03, BD-01, HW-03, HW-08 | PLANNED (EI-6) |

**Supplementary evidence (governance-backed, not separate matrix rows):** commit reconciliation (`test_an_unconfirmed_commit_reconciles_by_reload`, `test_unsettled_commit_uncertainty_claims_nothing`), `STORED_ISSUANCE_INVALID` and `GENERATION_ALREADY_ISSUED` (`test_a_different_stored_record_…`, `test_another_issuance_of_the_generation_key_…`), contradictory writer answers, declaration venue-day and commit uncertainty, and the targeted mutation check (section 11).

## 10. Changes to existing tests

Three existing assertions conflicted with the governed change and were updated in #96:

- **The EI4-D11 boundary scan** forbids the name `_ISSUED` outside the generation recorder; the issuance registry is named `_ISSUED_ISSUANCE_WRITES`. The scan itself is unchanged.
- **Two tests** (`test_compat_w_no_application_source_fabricates_an_issued_history`, `test_ak_recording_never_writes_or_fabricates_issued_history`) pinned the pre-EI-5D rule that no Application class implements `EquitySessionRecommendationHistorySource`. They now allow exactly `AuthoritativeEquitySessionRecommendationHistorySource` (ADR-024, sections 6 and 12), which takes only the authority's reader, and still assert that recording never writes or fabricates history.

## 11. Test results and CI

### 11.1 Historically reported local results

| Point | Result |
| --- | --- |
| New EI-5D and EI-5E tests | 68 passed |
| Focused set (new modules, v2 generation, v1 goldens, EI-5C lifecycle, write architecture) | 280 passed |
| Full Application suite | 4,656 passed, 0 skipped |
| Full Infrastructure suite against the modified Application | 3,293 passed, 27 skipped |
| Targeted mutations | 4 of 4 detected: the issuer's evaluator-`/2` gate, its completeness gate (D5D-4), strict instead of inclusive no-backdating, and backdating before identity |
| Ruff check and format; `git diff --check` | Clean |

**Timing limitation (unresolved).** The Infrastructure run began before three final edits that are in the merged commit: the rename of a private registry variable to `_ISSUED_ISSUANCE_WRITES`, and two test-only changes. The full Application suite was re-run after them; one later import-order fix in a test was re-run individually. Infrastructure source and tests reference none of the EI-5D names (read-only search, 2026-11-14), so the rename cannot change Infrastructure behaviour — but **no run against the exact merged commit is recorded**.

### 11.2 CI status

**NOT INDEPENDENTLY VERIFIED FROM AVAILABLE EVIDENCE.** No CI run, artifact or check result was available in this environment (no GitHub CLI; no local artifacts). No run identifier, URL or conclusion is recorded. A merged PR is not CI evidence. The evidence needed is a successful `python-ci.yml` run for northstar-application at `9a68209` (with its Core dependency commit) and for northstar-infrastructure against it, recorded as in the [EI-4 Decision Register, section 15.3](Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4).

## 12. Remaining limitations

**Belonging to EI-5D/EI-5E:**

- **L1 — Not durable.** The authority is an in-memory test double; no issued history persists.
- **L2 — EI-5C synthetic.** The lifecycle is evaluated under EI-5C's synthetic label; no issued lifecycle is persisted.
- **L3 — Scenario limits.** IS-16, HW-07 and NB-04 are VERIFIED WITH LIMITATION (section 9).
- **L4 — Evidence.** Local Windows runs only; CI not verified; the Infrastructure timing limitation of section 11.1.

**Carried forward (unchanged):**

- **L5 — EI4-D8.** Conservative open-state treatment: after the first issuance in a session, later proposals in that session resolve as `EVALUATION_UNRESOLVED`.
- **L6 — VC-04.** NOT VERIFIED; rollback or downgrade to a pre-3a Application against stored evaluator-v2 records is not authorized.
- **L7 — N6.** The SQLite label-format pairing trigger stays deferred.
- **L8 — I08.** An equivalent mutant under the accepted contracts.
- **L9 — EI-4 closure.** Formal CI closure (EI4-C4) stays pending, separately (D5D-5).
- **L10 — Deferred and unauthorized.** O-2 and O-3 deferred; ACTUAL issuance and broker execution unauthorized.

**Deferred to EI-6 (obligations, not EI-5D/EI-5E defects):** durable issuance and declaration tables in the same SQLite authority with a foreign key to the EI-4D disposition; `BEGIN IMMEDIATE` enforcement; constraints and insert-only triggers; pre-connection authorization refusal; concurrent writers; restart-safe views; the scenarios of section 9.3.

## 13. Engineering verdict and recommendation

**EI-5D/EI-5E ENGINEERING ACCEPTANCE: PASS WITH DOCUMENTED LIMITATIONS — synthetic engineering acceptance.**

- The implementation is merged, Application-only, and limited to synthetic in-memory acceptance.
- Every EI-5D and EI-5E scenario part is evidenced; three carry a documented limitation; none is NOT VERIFIED.
- EI-6-owned scenarios stay PLANNED.

**Recommended owner decision, as recorded before the owner decided:** accept EI-5D and EI-5E **with documented limitations** (section 12), retaining the in-memory-authority limitation and the open CI evidence, without authorizing EI-6. The owner decided on 2026-11-14 (section 13.1).

### 13.1 Owner acceptance (2026-11-14)

**OWNER APPROVED — 2026-11-14.**

**EI-5D MODELED Recommendation Issuance and EI-5E Synthetic End-to-End Integration: ACCEPTED WITH DOCUMENTED LIMITATIONS.**

| Item | Accepted |
| --- | --- |
| Implementation | northstar-application, PR #96, merge `9a68209` |
| EI-5D (38 parts) | 36 VERIFIED, 2 VERIFIED WITH LIMITATION |
| EI-5E (7 parts) | 6 VERIFIED, 1 VERIFIED WITH LIMITATION |

**Accepted scenario limitations:**

- **IS-16:** the invalid version-2 history-boundary sub-case cannot be constructed through the approved inputs contract, so that issuer-level negative path was not directly exercised.
- **HW-07:** the watermark-beyond-current refusal was verified against the in-memory authority; durable-reader enforcement remains EI-6's responsibility.
- **NB-04:** no-position behaviour was established within the synthetic integration environment, without a real paper-trading store.

**Additional accepted limitations:**

- the issuance authority is an in-memory, non-durable test implementation;
- EI-5C retains its synthetic labelling;
- independent CI evidence is not verified;
- the reported Infrastructure run began before the final Application private-variable rename and two test-only changes; no run against the exact merged Application commit was independently established;
- EI4-D8's exposure and exclusivity restrictions remain unchanged;
- every previously accepted EI-4C limitation remains applicable, including VC-04's legacy rollback restriction, the deferred N6 SQLite pairing trigger and the I08 equivalent-mutation qualification;
- EI-4 formal CI closure remains separately **pending**;
- O-2 and O-3 remain deferred.

**Not authorized by this acceptance:** EI-6 implementation; durable SQLite issuance or declaration persistence; ACTUAL issuance; broker or account integration; orders, fills or real-money trading; production deployment; API or Web expansion; real market-data integration; relaxation of EI4-D8; resolution of deferred governance decisions. **EI-6 requires a separate owner authorization.**

The acceptance changes no scenario status, evidence or test result in this record. The owner record is the [EI-5 Decision Register, section 16.3](Equity-Intraday-V1-EI5-Decision-Register.md#163-owner-acceptance-of-ei-5d-and-ei-5e-2026-11-14).

## 14. Non-authorizations

Neither this record nor the owner acceptance of section 13.1 authorizes anything further: not EI-6, a SQLite issuance authority, lifecycle persistence, monitoring, composition roots, ACTUAL issuance, actionable recommendations, orders, fills, positions, broker execution, API or Web, real data, production deployment, the N6 trigger, or any relaxation of EI4-D8 or resolution of O-2 and O-3. It does not close EI-4.

## 15. Related documents

- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design v1.0](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [EI-5 Decision Register](Equity-Intraday-V1-EI5-Decision-Register.md), sections 14, 15 and 16
- [EI-4C Versioned History Compatibility Engineering Acceptance Record](Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md)
- [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md); [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md), section 15
