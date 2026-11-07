# Equity Intraday V1 EI-4E Synthetic Acceptance Record

## Status

**Engineering acceptance record, 2026-11-08.**

- **Verdict:** PASS WITH DOCUMENTED LIMITATIONS.
- **Status:** Engineering evidence merged; acceptance documented with limitations; owner decisions AA-1 and AA-2 remain open.
- This record is engineering acceptance evidence. It is **not** an owner approval: it approves no decision, changes no decision status and does not close the EI-4E milestone on the owner's behalf. The owner approvals that EI-4D and EI-4E implement are recorded in the [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md), sections 3 and 9.
- The two findings in [section 14](#14-open-owner-decisions), AA-1 and AA-2, are **OWNER DECISION PENDING**. The policies proposed there are proposals, not approvals, and nothing implements them.
- Accepting EI-4E is **not** real-data readiness, production readiness, recommendation issuance, trading authorization, or acceptance of any profitability or performance claim ([section 15](#15-non-authorizations)).

## 1. Acceptance identity

| Item | Value |
| --- | --- |
| Milestone | EI-4E |
| Title | Synthetic Equity Recommendation Generation — End-to-End Acceptance |
| Product | Northstar Equity Intraday V1: NSE cash equities, INR, LONG-only, synthetic only |
| Engineering verdict | PASS WITH DOCUMENTED LIMITATIONS |
| Status | Engineering evidence merged; acceptance documented with limitations; owner decisions AA-1 and AA-2 remain open |
| Evidence | Infrastructure PR #54, merged to `develop` as `5dda518` |
| Record date | 2026-11-08 |
| Kind | Engineering evidence; not an owner approval |

## 2. Scope

EI-4E is the fifth step of the approved EI4-D9 implementation sequence ([EI-4 Decision Register, section 9.4](Equity-Intraday-V1-EI4-Decision-Register.md#94-approved-implementation-sequence); [ADR-021, section 12](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md#12-implementation-sequence)): synthetic end-to-end acceptance on real SQLite, covering restart, concurrent writers, uncertain COMMIT, tampering and replay.

It accepts, as one chain on one SQLite file, the merged work of:

- **EI-2B and EI-2C**: synthetic candle ingestion, the immutable journal and snapshots;
- **EI-3A, EI-3B and EI-3C**: research evaluation and the authoritative research timeline;
- **EI-4B** (northstar-core): synthetic recommendation formation;
- **EI-4C** (northstar-application): eligibility and generation, including the EI4-D9 compatibility contracts;
- **EI-4D-A** (northstar-application): generation records, verification and record-and-reconcile orchestration, including the W1 stored-record correction;
- **EI-4D-B** (northstar-infrastructure): the immutable SQLite generation store.

EI-4E adds **tests only**. It changes no production code, schema, trigger, identity scheme or decision.

Out of scope: issuance and lifecycle (EI-5), their persistence (EI-6), API and Web (EI-7), evaluation and performance (EI-8), real market data (EI-9), forward observation (EI-10), paper or broker orders, and production use.

## 3. Governance references

| Source | What it governs here |
| --- | --- |
| [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md), section 3 | EI4-D1 (decision, modeled issuance and entry timing), EI4-D2 (first authoritative CANDIDATE), EI4-D7 (versioned synthetic eligibility), EI4-D8 (conservative prior issued-state treatment) |
| [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md#9-ei4-d9-owner-approval-record-2026-11-07), section 9 | EI4-D9-A to EI4-D9-J and clarifications A to E (approved 2026-11-07) |
| [ADR-020](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md) | Recommendation generation architecture |
| [ADR-021](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md) | Generation persistence architecture, sections 1 to 12 |
| [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md#11-ei-3c-owner-approval-record-2026-11-07), section 11 | EI3-D11 (commit authority), EI3-D12 (SQLite serialization and commit semantics), EI3-D13 (fail-closed historical compatibility) |
| [EI-3C Synthetic Acceptance Record](Equity-Intraday-V1-EI3C-Synthetic-Acceptance-Record.md) | The accepted research-persistence baseline this chain builds on |
| [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md) | Product scope and milestone sequence (section 12) |
| [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md) | The accepted venue-day reference used for the synthetic trading date |
| [ES-003 Testing Standards](../engineering/standards/ES-003-Testing-Standards.md) | Integration and acceptance test definitions |

## 4. Repository and dependency baselines

Every revision below was verified on the development machine on 2026-11-08 with a clean working tree.

| Repository | Branch | HEAD | Relevant merged change |
| --- | --- | --- | --- |
| northstar-infrastructure | `develop` | `5dda518` | EI-4E acceptance tests, PR #54 (commit `3e22f33`); before it, `bfde063` (W1 test alignment, PR #53) and `9195f0f` (EI-4D-B, PR #52) |
| northstar-application | `develop` | `bed91de` | W1 stored-record correction, PR #88; EI-4D-A, PR #87 (`75e11f1`) |
| northstar-core | `develop` | `bc081a2` | EI-4B, PR #38 |
| northstar-docs | `develop` | `04a3062` | EI4-D9 governance and ADR-021, PR #49 |

PR #54 adds exactly three files and nothing else (`git diff bfde063 5dda518`):

- `tests/equity_recommendation_acceptance_fixtures.py` (673 lines);
- `tests/test_equity_recommendation_generation_acceptance.py` (1,031 lines);
- `tests/test_equity_recommendation_generation_acceptance_faults.py` (896 lines).

Infrastructure tests resolve `northstar_application` from the Application working tree as an editable install, so they ran against `bed91de`.

## 5. Test environment

- Development machine only: Windows 11, Python 3.13, the Infrastructure virtual environment.
- Every test uses its own disposable SQLite file in a pytest temporary directory. No production database, deployment machine or production configuration is touched.
- Every acceptance test replaces socket connections with a refusal, so no NSE, Upstox, Databento, broker, provider or network call can occur.
- Results are local. **CI did not run for this evidence**, and none is claimed.

## 6. Evidence architecture

### 6.1 The source pipeline exercised

```text
EI-2C synthetic 5-minute candle ingestion (IngestSyntheticEquityCandleEventsUseCase)
  -> EI-2B immutable journal and snapshot (SQLiteEquityCandleEvidenceJournal, ...SnapshotStore)
  -> EI-3A research evaluation (EvaluateEquityIntradayResearchUseCase)
  -> EI-3C authoritative timeline and first CANDIDATE (AdvanceEquityIntradayResearchTimelineUseCase,
     SQLiteEquityIntradayResearchTimelineStore)
  -> EI-4C generation with EI-4B Core (GenerateEquityTradeRecommendationUseCase, form_equity_recommendation)
  -> EI-4D-A canonical verification and recording (RecordEquityRecommendationGenerationUseCase,
     EquityRecommendationGenerationVerifier)
  -> EI-4D-B immutable SQLite persistence (SQLiteEquityRecommendationGenerationStore)
  -> restart, rehydration and frozen historical replay (VerifyStoredEquityRecommendationGenerationUseCase,
     assess_equity_recommendation_inputs)
```

Every stage above is the merged production class or function, driven through its public interface. Nothing on this path is mocked.

### 6.2 What is synthetic

The acceptance fixtures (`tests/equity_recommendation_acceptance_fixtures.py`) are independent of the EI-3C and EI-4D-B test fixtures: they import neither. They use their own:

- listings: TCS (a breakout session), INFY (a session without one), and WIPRO (named only in synthetic issued history);
- trading date: 2026-08-19;
- prices, research calibration (opening range of 4 bars), candle source and finality policy.

The only shared reference is the accepted production NSE venue-day reference. The tests confirm that 2026-08-19 is a `NORMAL_SESSION` in it rather than assuming so.

The external sources were **synthetic test fixtures**, each labelled as such in the code:

- the policy references: generation, risk and model capital, costs, price increment and timing with its issuance latency;
- the synthetic listing eligibility;
- the issued-recommendation history. Its "empty" mode is an explicitly established synthetic history, complete through 15:30 IST at recording watermark 0. Missing, unavailable, incomplete and other-session histories are separate modes.

**Every price, volume, latency, cutoff, model capital, risk fraction, cost rate and increment is test-only and approved for nothing.** None is a production reference value, and nothing here establishes production reference-data readiness.

### 6.3 Normal-path and fault-injection evidence

- **Normal path:** `test_equity_recommendation_generation_acceptance.py`, 38 tests. No stored row is altered and no seam is replaced.
- **Fault injection:** `test_equity_recommendation_generation_acceptance_faults.py`, 34 tests. Faults are injected only through these labelled, test-only means:
  - the EI-4D-B adapter's module-level `_connect`, `_insert_disposition`, `_insert_attempt` and `_commit` functions and `_BUSY_TIMEOUT_SECONDS`, replaced with `monkeypatch` for one test;
  - `bypass()`, a separate raw connection to the disposable file that drops a trigger, or relaxes CHECK or foreign-key enforcement, before writing;
  - a forged record built from public Application types, whose candidate evidence was altered and whose documents and fingerprints are internally consistent.
- **One normal-path exception:** `test_e11_an_attempt_only_result_is_audited_without_a_disposition[research integrity failure]` deliberately wires the generator to a snapshot store over a different database. It is labelled as fault injection in the test.

None of these is a production shortcut, and no fault-injection test is counted as evidence of the normal workflow.

Test names carry the number of the EI-4E implementation requirement they evidence (`e5` to `e22`). Those numbers are not section numbers of this record.

## 7. Acceptance scenarios

Test functions are named without their module. Normal-path functions are in `test_equity_recommendation_generation_acceptance.py`; fault-injection functions are in `test_equity_recommendation_generation_acceptance_faults.py`. A count such as ×3 is the number of parametrized cases.

| ID | Scenario | Normal-path tests | Fault-injection tests |
| --- | --- | --- | --- |
| A | Synthetic candle finality and snapshot identity | `test_e5_synthetic_candles_are_journaled_and_snapshotted_through_ei2` | — |
| B | Authoritative first CANDIDATE | `test_e6_the_first_committed_candidate_is_the_authoritative_source`; `test_e6_generation_never_uses_an_uncommitted_candidate` | — |
| C | NO_SIGNAL and REJECTED research outcomes | `test_e6_the_first_committed_candidate_is_the_authoritative_source` (NO_SIGNAL entry 1); `test_e6_a_session_without_a_breakout_has_no_candidate_to_generate_from`; `test_e16_a_later_correction_leaves_the_original_candidate_and_assessment` (REJECTED) | — |
| D | PROPOSED generation | `test_e8_the_proposed_path_from_candles_to_a_verified_stored_disposition`; `test_e8_the_same_synthetic_inputs_reproduce_every_fingerprint_on_a_new_database` | — |
| E | NOT_ELIGIBLE generation | `test_e9_a_confirmed_ineligibility_is_recorded_without_a_proposal` ×3; `test_e13_a_signal_after_the_policy_cutoff_is_refused` | — |
| F | UNRESOLVED generation | `test_e10_an_unresolved_disposition_is_immutable_and_never_becomes_proposed` ×2 | — |
| G | Attempt-only outcomes | `test_e11_an_attempt_only_result_is_audited_without_a_disposition` ×8; `test_e11_identical_attempts_deduplicate_and_distinct_ones_stay_auditable`; `test_e11_an_invalid_request_is_refused_before_any_identity_or_record` | `[research integrity failure]` case of the first (miswired snapshot store) |
| H | Composite policy-set identities | `test_e12_a_b_f_one_key_per_candidate_and_approved_policy_set`; `test_e12_c_a_reused_version_label_with_changed_content_is_another_identity` | — |
| I | Idempotent equivalence | `test_e12_a_b_f_one_key_per_candidate_and_approved_policy_set`; `test_e9_a_confirmed_ineligibility_is_recorded_without_a_proposal`; `test_e16_a_later_correction_leaves_the_original_candidate_and_assessment`; `test_e11_an_attempt_only_result_is_audited_without_a_disposition` | — |
| J | Same-key, different-content conflicts | `test_e12_d_e_g_h_changed_evidence_under_one_key_is_a_conflict_never_a_replacement` ×3; `test_e10_an_unresolved_disposition_is_immutable_and_never_becomes_proposed` | — |
| K | Exact modeled issuance and no look-ahead | `test_e13_entry_bars_open_only_at_or_after_the_modeled_issuance` ×3; `test_e13_a_signal_after_the_policy_cutoff_is_refused`; `test_e5_synthetic_candles_are_journaled_and_snapshotted_through_ei2` | — |
| L | Point-in-time synthetic issued-history evidence | `test_e14_an_explicit_synthetic_empty_history_is_frozen_with_its_coverage`; `test_e14_only_records_issued_by_the_modeled_issuance_are_evidence`; `test_e14_missing_incomplete_and_contradictory_history_fail_closed`; `test_e14_recording_never_writes_history_and_never_blocks_its_own_retry` | — |
| M | SQLite atomicity and rollback | — | `test_e17_a_disposition_its_proposal_and_its_attempt_commit_in_one_transaction` (statement trace); `test_e17_a_failure_before_commit_leaves_no_partial_generation_record` ×3; `test_e17_a_real_constraint_violation_rolls_back_the_whole_write`; `test_e17_attempt_only_failures_are_reported_never_claimed_as_audited`; `test_e17_a_proposal_never_exists_without_its_disposition` |
| N | Commit-unconfirmed reconciliation | — | `test_e18_commit_did_not_occur_and_a_verified_absence_allows_a_bounded_retry`; `test_e18_a_lost_acknowledgment_is_reconciled_as_committed_by_reload`; `test_e18_another_writers_commit_found_on_reload_is_equivalent_or_a_conflict` ×2; `test_e18_an_unavailable_reload_leaves_the_commit_unresolved`; `test_e18_an_invalid_record_found_on_reload_leaves_the_commit_unresolved`; `test_e18_attempt_only_commit_uncertainty_reconciles_by_attempt_identity` ×2; `test_e18_retries_are_bounded_and_never_duplicate` |
| O | Independent-connection concurrency (threads) | — | `test_e19_two_writers_of_equivalent_content_store_it_once`; `test_e19_two_conflicting_writers_leave_one_authoritative_disposition`; `test_e19_writers_of_distinct_policy_set_identities_both_commit`; `test_e19_a_writer_that_cannot_take_the_lock_writes_nothing` (busy-timeout seam) |
| P | Windows separate-process concurrency | — | `test_e19_separate_os_processes_are_serialized_by_the_database` ×2 |
| Q | Restart and frozen historical replay | `test_e15_every_disposition_kind_replays_after_restart_from_frozen_evidence` | — |
| R | Source-correction stability | `test_e16_a_later_correction_leaves_the_original_candidate_and_assessment` | — |
| S | Database immutability and tamper rejection | `test_e9_a_confirmed_ineligibility_is_recorded_without_a_proposal` (insert-only UPDATE refusal) | `test_e20_direct_sql_cannot_change_replace_or_delete_a_generation_record`; `test_e20_a_corrupted_stored_disposition_fails_closed_and_is_never_repaired` ×5; `test_e20_a_missing_source_is_a_structural_break_and_fails_verification`; `test_e20_contradictory_candidate_evidence_is_caught_by_the_application_verifier` ×2 |
| T | W1 invalid and incompatible stored-record mapping | — | `test_e21_a_corrupt_stored_attempt_is_typed_by_the_adapter_and_mapped_by_the_recorder` ×2; `test_e21_a_corrupt_stored_disposition_leaves_the_attempt_state_unknown` |
| U | Existing EI-2B and EI-3C schema compatibility | `test_e22_recording_adds_its_schema_and_never_alters_earlier_evidence`; `test_e22_the_store_never_creates_or_adopts_a_database_without_the_evidence` | — |

The concurrency tests in O and P live in the fault-path module, but apart from one test that lowers the busy timeout, they inject no fault. They use real SQLite, real independent connections, and barriers to start the writers together.

## 8. Expected and observed results

Every observed result below is asserted by the cited tests unless it is marked *observed, not asserted*.

### 8.1 Representative PROPOSED scenario

From `test_e8_the_proposed_path_from_candles_to_a_verified_stored_disposition`, `test_e5_...`, `test_e6_...` and `test_e13_...`. **Every value is synthetic and test-only. This is not an investment recommendation.**

| Item | Value | Asserted by |
| --- | --- | --- |
| Synthetic listing | TCS (NSE) | fixtures |
| Research date | 2026-08-19 | fixtures |
| Accepted venue session | `NORMAL_SESSION` in the accepted NSE venue-day reference | `test_e5_...` |
| Research window | 09:15 to 15:00 IST, 69 five-minute intervals | `test_e5_...` |
| Synthetic opening range | Bars 1 to 4: high 252, low 249 (synthetic prices) | fixtures |
| Signal bar | Bar 5, ended 09:40 IST, synthetic close 253.5 | `test_e13_...` |
| Research decision | 09:41 IST (the decision instant of the 09:41 snapshot) | `test_e13_...` |
| First CANDIDATE | Research timeline entry 2, on the 09:41 snapshot | `test_e6_...` |
| Synthetic issuance latency | 150 seconds (timing policy `synthetic-ei4e-timing/1`) | fixtures, `test_e8_...` |
| Modeled issuance | 09:43:30 IST; issued history requested as of exactly this instant | `test_e8_...` |
| Earliest permissible entry bar | Opens 09:45 IST | `test_e8_...`, `test_e13_...` |
| Stored disposition | `PROPOSED`, one row, with its proposal document | `test_e8_...` |
| Stored attempts | 1 | `test_e8_...` |
| Other stored rows | Journal 12, snapshots 3, timelines 1, timeline entries 3 | `test_e8_...` |
| Verification | Canonical content and fingerprints; original source re-established; historical replay reproduced | `test_e8_...` |
| Actual issuance | False: `issuance_established` is False, and the proposal is not issued, not actionable and not authoritative | `test_e8_...` |
| Synthetic sizing | 493 shares; entry zone 252.25 to 253.5; stop 249; target 262.5 (model capital INR 500,000, risk fraction 0.005, exposure fraction 0.25, all synthetic) | *observed, not asserted* |

The fingerprints of this scenario were recorded with pytest `record_property`. `test_e8_the_same_synthetic_inputs_reproduce_every_fingerprint_on_a_new_database` asserts that a second fresh database reproduces all of them. The test does not pin these values as constants: they are *observed, not asserted*.

| Item | Observed value |
| --- | --- |
| Snapshot | `ecs1-sha256:38ed4bbd9405beb57e163ddfa17d0660edb4f50aa98342158356535a9143015d` |
| Manifest | `ecm1-sha256:1c2216e2ac1b8bf40b934e5203dacdfc0eea71ed93ad9eb5c7d9e29d6ca87a34` |
| Research decision | `eir1-sha256:fc2ea68a6427747deb3dedd12e2c9ec35658d349e114bd7459834657becfbfd8` |
| CANDIDATE entry | `eirt1-sha256:e097703caf1c91db76e5c589423147cee669ce88d37b9fb5a7acd114cbc9cda8` |
| Policy set | `eirs1-sha256:fdc7884181a9ee39d76669993893daaa0a03e02bcb2b59bf80861a9214cb8930` |
| Inputs | `eiri1-sha256:f8eb87ead3fa1fbee2665fa99642226e44063907c8aaafce7d25a3f536bd3798` |
| Generation key | `eirk1-sha256:96e2ffaaeafdc71eb1e3ced2f9d7c162f89fec40efae3fda2acfb847cddd623c` |
| Assessment | `eira1-sha256:9fd0204f8a1a1b21a0e0385e1e1c22b6c6d6f69c7beafd4cb24008a14848ac5b` |
| Proposal | `eirp1-sha256:70a73bda6067959d437183bd190445b0c3f1c62f56ade34aaff5b074cb13dfa7` |
| Attempt | `eirat1-sha256:c905487fead12a47034dd7be498da271d415f401f404eb5f4f0ba67c244c0f90` |

### 8.2 Results by scenario

- **A. Evidence.**
  - The 09:41 snapshot is SYNTHETIC, with `ecs1` and `ecm1` identities and the accepted venue reference.
  - Bars 1 to 5 are ACCEPTABLE and the other 64 intervals NOT_ENDED; coverage is COMPLETE_AS_OF.
  - No interval after bar 5 has an acceptable sequence, so there is no look-ahead.
  - A fresh adapter reloads it unchanged, and a `DELETE` of snapshots is refused.
- **B. First CANDIDATE.**
  - Snapshots at 09:36, 09:41 and 09:46 IST commit NO_SIGNAL, CANDIDATE and REPEAT_QUALIFYING.
  - The CANDIDATE's decision equals an independent EI-3A re-run, and rehydration reproduces the timeline document.
  - Generation uses entry 2, never the REPEAT_QUALIFYING entry 3.
  - A LONG_CANDIDATE that was only evaluated in memory, with no committed timeline, yields NO_CANDIDATE. Its attempt is recorded and no disposition is created.
- **C. NO_SIGNAL and REJECTED.**
  - Entry 1 is NO_SIGNAL. The INFY session commits only NO_SIGNAL and generates NO_CANDIDATE.
  - After a journaled revision of bar 2, a 09:51 snapshot appends a REJECTED entry with reason `UNRESOLVED_SOURCE_REVISION`. A synthetic bar 7 is journaled first, so the revision is the only cause.
- **D. PROPOSED.** As in [8.1](#81-representative-proposed-scenario).
  - Recording returns COMMITTED in one write, with `attempt_recorded` True. The issued-history source is not consulted during recording.
  - The generation key starts `eirk1-sha256:` and equals the Application's public key function.
- **E. NOT_ELIGIBLE.** Four causes, each after a valid generation identity:
  - a Core rejection under a strict synthetic policy set (`CORE_FORMATION_REJECTED`);
  - a late synthetic latency (`NO_PERMISSIBLE_ENTRY_BAR`);
  - a synthetic INELIGIBLE criterion under the original policy set (`LISTING_NOT_ELIGIBLE`);
  - a synthetic 09:35 signal cutoff (`AFTER_SIGNAL_CUTOFF`, generated but not recorded).

  The three recorded cases (`test_e9`) each have:
  - no proposal document or fingerprint, exactly one reason, and the original candidate;
  - an identical regenerated assessment, which records as EQUIVALENT_COMMITTED;
  - an `UPDATE` to PROPOSED refused as insert-only.
- **F. UNRESOLVED.**
  - A prior synthetic same-listing issuance yields `EXCLUSIVITY_POLICY_UNRESOLVED`; a prior other-listing issuance yields `EXPOSURE_POLICY_UNRESOLVED`.
  - SAME_LISTING_EXCLUSIVITY and PORTFOLIO_EXPOSURE stay NOT_ASSESSED: no threshold is approved or invented.
  - With an empty synthetic history, a later PROPOSED under the same key is CONTENT_CONFLICT. The UNRESOLVED row is unchanged and the refused attempt is audited (rows 1 and 2).
  - The Core `SIZING_SEARCH_LIMIT_EXCEEDED` route to UNRESOLVED was **not tested** end to end.
- **G. Attempt-only.**
  - Eight sources: no committed research; missing, unavailable or incomplete history; a reference outage; an unknown policy version; a research-configuration mismatch; a research-integrity failure (`RESEARCH_TIMELINE_INVALID`, by fault injection).
  - Each has no generation key and no disposition, and its attempt document has no candidate or inputs.
  - Each is recorded (COMMITTED, `attempt_recorded` True), and an identical retry is EQUIVALENT_COMMITTED with one row.
  - "Missing" and "unavailable" history are one logical attempt, because exception prose is not identity. "Incomplete" is a distinct attempt, giving two rows.
  - An invalid request raises before any identity exists, and no generation schema is created.
- **H. Policy-set identities.**
  - Same candidate and policy set: same key and same assessment fingerprint.
  - A changed set version, generation policy or timing policy gives four distinct keys.
  - The label `synthetic-ei4e-timing/1` reused with a different latency gives a different `eirs1` fingerprint, so a different key. A second disposition commits, and the first row is untouched (see [AA-1](#141-aa-1--policy-version-label-governance)).
- **I. Idempotent equivalence.** An identical record returns EQUIVALENT_COMMITTED in one write with no new rows. This holds for PROPOSED, NOT_ELIGIBLE, attempt-only records, and after a source correction.
- **J. Conflicts.** Under one key, changed eligibility evidence (universe version or verdict) and changed as-of history (a later issuance raising the watermark to 1) each produce a different assessment fingerprint and CONTENT_CONFLICT. The first committed disposition is unchanged, and the conflicting attempt is appended.
- **K. Timing.**

  | Synthetic latency | Modeled issuance | Earliest entry bar |
  | --- | --- | --- |
  | 150 seconds | 09:43:30 IST, between openings | 09:45 |
  | 4 minutes | 09:45 IST, exactly on an opening | 09:45 |
  | 65 minutes | 10:46 IST | None: NOT_ELIGIBLE (`NO_PERMISSIBLE_ENTRY_BAR`) |

  - Every grid bar opening before the eligible bar opens before the issuance.
  - Decision ≤ issuance ≤ eligible entry.
  - The issuance kind is MODELED, and the modeled exit is 14:00 IST (synthetic).
  - No fill, touch scan or intrabar ordering is modelled.
- **L. Point-in-time history.**
  - The synthetic empty history is frozen with its reference, as-of instant (09:43:30), completeness (15:30) and watermark 0.
  - Records issued after the modeled issuance are excluded, and those included are ordered by issue time, not recording order.
  - A future-only issuance still yields PROPOSED.
  - Missing, incomplete and other-session histories fail closed. A contradictory history cannot be constructed: the port type refuses it.
  - Recording never writes history and never blocks its own identical retry.
- **M. Atomicity.**
  - A statement trace of one recording shows one `BEGIN IMMEDIATE`, the disposition INSERT (the proposal is in the same row), the attempt INSERT and one `COMMIT`. There is no SAVEPOINT, ROLLBACK, RELEASE or second BEGIN.
  - A conflicting write inserts only its attempt.
  - Injected failures before any insert, after the disposition insert, or by simulated process stop leave no rows. The stop propagates.
  - A real CHECK violation on the attempt rolls back the already-inserted disposition.
  - An attempt-only write failure is NOT_COMMITTED with `attempt_recorded` False.
  - A PROPOSED row without its proposal is refused by CHECK, and no separate proposal table exists.
- **N. Commit-unconfirmed.** See [8.3](#83-conflicts-and-uncertain-commits).
- **O. Threads.**
  - Equivalent writers give {COMMITTED, EQUIVALENT_COMMITTED} with rows (1, 1).
  - Conflicting writers give one COMMITTED and one CONTENT_CONFLICT. The loser names the winner as authoritative and is audited (rows 1, 2).
  - Distinct policy sets both commit.
  - A writer that cannot take the write lock is NOT_COMMITTED and writes nothing.
- **P. Separate processes.**
  - Two processes started with the `spawn` method, each building its own adapters and Application use cases from the file path alone, report two distinct process IDs, both different from the parent's.
  - Equivalent contest: {COMMITTED, EQUIVALENT_COMMITTED}. Conflicting contest: {COMMITTED, CONTENT_CONFLICT}, with the stored row being the winner's.
  - The assertions are invariants and do not depend on which process wins.
- **Q. Restart and replay.**
  - PROPOSED, NOT_ELIGIBLE and UNRESOLVED dispositions are committed under three identities.
  - All Python objects are dropped. The file is renamed, which Windows refuses while a connection holds it open, and then copied.
  - From the copy, fresh adapters run with every synthetic reference, eligibility and history source made unavailable (generation confirms DEPENDENCY_UNAVAILABLE). For each disposition:
    - it loads equal to the original;
    - its canonical documents and fingerprints re-verify, and its evaluator version matches;
    - its attempt reloads;
    - the original EI-3C CANDIDATE is re-established and the EI-2B manifest matches;
    - the assessment replays exactly from the frozen inputs.
  - The unavailable sources' call counters do not change.
  - The levels are reported separately: document integrity, source integrity and evaluation reproduction are True; actual issuance is False.
- **R. Correction stability.** After the REJECTED entry of scenario C:
  - the CANDIDATE stays entry 2 on the 09:41 snapshot, and entries 1 to 3 are unchanged;
  - the stored disposition re-verifies on the 09:41 snapshot, not the corrected 09:51 snapshot, and its fingerprint reproduces. Its row is unchanged;
  - regeneration is EQUIVALENT_COMMITTED.

  EI0-D21 materiality is not implemented or assumed.
- **S. Immutability and tampering.** See [8.4](#84-database-guarantees-established).
- **T. W1.**
  - For a stored attempt that is invalid (non-canonical JSON) or incompatible (format version 2), the adapter raises the typed error on `load_attempt` and `record_attempt`; Invalid is never reported as Incompatible.
  - The recorder returns STORED_RECORD_INVALID or STORED_RECORD_INCOMPATIBLE, with `is_stored` False and `attempt_recorded` None. The row is unchanged and no other record is created.
  - A corrupt stored disposition gives STORED_RECORD_INVALID with `attempt_recorded` None, in one write.
- **U. Compatibility.**
  - Recording adds only the generation tables, triggers and index. Every other schema object is unchanged.
  - Journal, snapshot, timeline and timeline-entry rows are unchanged.
  - A database that already holds the generation schema reopens without a schema change.
  - The store refuses, and does not create, a missing database or one without the evidence tables.

### 8.3 Conflicts and uncertain commits

| Situation (real recorder, real SQLite) | Recording outcome | `attempt_recorded` | Rows (dispositions, attempts) |
| --- | --- | --- | --- |
| Same key, identical content | EQUIVALENT_COMMITTED | True | unchanged |
| Same key, different content | CONTENT_CONFLICT; first record authoritative | True | disposition unchanged; attempt appended |
| COMMIT did not take effect; reload finds nothing | Bounded retry, then COMMITTED in 2 writes | True | (1, 1) |
| COMMIT took effect; acknowledgment lost | EQUIVALENT_COMMITTED, reconciled | True | (1, 1) |
| Competitor committed equivalent content before the reload | EQUIVALENT_COMMITTED, reconciled | True | (1, 1) |
| Competitor committed different content before the reload | CONTENT_CONFLICT, reconciled | False (this write rolled back) | (1, 1) |
| Reload unavailable (database locked) | COMMIT_RECONCILIATION_UNRESOLVED | None | (1, 1); nothing claimed |
| Reload finds a record that does not decode | COMMIT_RECONCILIATION_UNRESOLVED | None | (1, 1); no retry |
| Attempt-only, acknowledgment lost | EQUIVALENT_COMMITTED in 1 write | True | (0, 1) |
| Attempt-only, COMMIT not made | COMMITTED in 2 writes | True | (0, 1) |
| COMMIT always fails | RETRY_LIMIT_REACHED after 3 writes | — | (0, 0) |
| Stored record under the key fails source verification | STORED_RECORD_INVALID; never adopted | True | (1, 2); stored row unchanged |

No case creates a duplicate disposition or attempt. No COMMIT failure is treated as a rollback.

### 8.4 Database guarantees established

**SQLite enforces** (shown on real SQLite):

- **Insert-only tables.** Twelve direct statements are refused with "insert-only" and all rows stay unchanged:
  - UNRESOLVED to PROPOSED, an evaluator-version change, relinking the candidate entry or the snapshot;
  - rewriting the assessment document, removing the proposal;
  - `DELETE`, `INSERT OR REPLACE` and `REPLACE` on dispositions;
  - `UPDATE`, `DELETE` and `INSERT OR REPLACE` on attempts.
- **CHECK constraints** tie the proposal to the PROPOSED kind and constrain attempt status.
- **Foreign keys** to the EI-2B snapshot and the EI-3C entry. With enforcement deliberately disabled and the snapshot deleted, `PRAGMA foreign_key_check` reports the break.
- **Serialization.** `BEGIN IMMEDIATE` serializes writers across threads and across processes on one file.
- **No silent creation.** The store never creates a database or adopts one without the evidence tables.

**The Application verifies** (shown):

- canonical documents, SHA-256 fingerprints and supported formats and evaluator version, on every load;
- re-establishment of the original CANDIDATE and snapshot;
- exact historical replay from frozen inputs;
- the typed mapping of stored-record failures.

Corruption by test-only bypass fails closed at load, at verification and at recording, and nothing is repaired:

| Corruption | Classified as |
| --- | --- |
| Invalid fingerprint | Invalid |
| Non-canonical document | Invalid |
| Empty evaluator | Invalid |
| Unsupported document format | Incompatible |
| Unsupported evaluator version | Incompatible |
| Deleted source snapshot | `EquityRecommendationSourceVerificationError`; regeneration reports RESEARCH_INTEGRITY_FAILURE |

**Not established, and not claimed:**

- SQLite does not verify SHA-256 fingerprints or the historical semantics of candidate evidence. A forged, internally consistent record with altered candidate evidence is accepted by a direct adapter call and refused only by Application verification ([AA-2](#142-aa-2--verified-writer-boundary-and-recovery)).
- SQLite `integrity_check` and `foreign_key_check` are asserted where the tests call `check_database`: `test_e5`, `test_e8`, `test_e15`, `test_e22_recording_adds_its_schema_and_never_alters_earlier_evidence`, two `test_e17` functions, the three threaded `test_e19` contests (through `_together`), `test_e19_separate_os_processes_are_serialized_by_the_database`, and two `test_e20` functions. They are not asserted in every test.

## 9. Test references

| Module | Tests | Content |
| --- | --- | --- |
| `tests/equity_recommendation_acceptance_fixtures.py` | — | Labelled synthetic fixtures, the pipeline wiring, direct inspection helpers, `bypass()` and the spawned-process worker `record_in_another_process` |
| `tests/test_equity_recommendation_generation_acceptance.py` | 38 | Normal paths, requirements `e5` to `e16` and `e22` |
| `tests/test_equity_recommendation_generation_acceptance_faults.py` | 34 | Fault paths, requirements `e17` to `e21` |

| Requirement prefix | Tests | Requirement prefix | Tests |
| --- | --- | --- | --- |
| `e5` | 1 | `e14` | 4 |
| `e6` | 3 | `e15` | 1 |
| `e8` | 2 | `e16` | 1 |
| `e9` | 3 | `e17` | 7 |
| `e10` | 2 | `e18` | 9 |
| `e11` | 10 | `e19` | 6 |
| `e12` | 5 | `e20` | 9 |
| `e13` | 4 | `e21` | 3 |
| `e22` | 2 | **Total** | **72** |

## 10. Acceptance matrix

Evidence types: **E2E**, end-to-end acceptance through the real chain; **SQL**, real SQLite persistence behaviour; **INJ**, deliberate fault injection on real SQLite; **UNIT**, Application unit tests with in-memory fakes. UNIT evidence is never counted as end-to-end proof.

| Requirement | Scenario | Expected behaviour | Observed behaviour | Test reference | Evidence type | Status | Limitation |
| --- | --- | --- | --- | --- | --- | --- | --- |
| EI-2B evidence | A | Synthetic, final, immutable snapshot with provenance | As expected | `test_e5_...` | E2E, SQL | PASS | One synthetic date and venue reference |
| EI4-D2, EI3-D11 | B | Only the committed first CANDIDATE is used | As expected; uncommitted candidate yields NO_CANDIDATE | `test_e6_...` ×2 | E2E | PASS | — |
| EI-3A, EI-3C outcomes | C | NO_SIGNAL and REJECTED entries | As expected | `test_e6_...`, `test_e16_...` | E2E | PASS | — |
| EI4-D9-A, EI4-D9-E | D | PROPOSED disposition, proposal and attempt recorded together and verified | As expected | `test_e8_...` ×2 | E2E, SQL | PASS | Synthetic references only |
| EI4-D9-A, EI4-D7 | E | NOT_ELIGIBLE without a proposal, deterministic reasons | As expected | `test_e9_...` ×3, `test_e13_a_signal_...` | E2E, SQL | PASS | — |
| EI4-D9-A, EI4-D8 | F | UNRESOLVED (exclusivity, exposure) immutable; later PROPOSED is a conflict | As expected | `test_e10_...` ×2 | E2E, SQL | PASS | No exclusivity or exposure threshold approved |
| EI4-D9-A | F | UNRESOLVED from Core `SIZING_SEARCH_LIMIT_EXCEEDED`, end to end | Not exercised | — | — | **NOT TESTED** | Route covered only by Application unit tests |
| EI4-D9-A, clarifications B and C | G | Attempt-only results audited; deduplicated; never given a disposition | As expected | `test_e11_...` ×10 | E2E (one INJ) | PASS | — |
| EI4-D9-B | H | One key per candidate and policy set; component changes give new keys | As expected | `test_e12_a_b_f_...` | E2E | PASS | — |
| EI4-D9-B | H | Reused label with changed content is not the identical set | Distinct `eirs1` and key; second disposition coexists | `test_e12_c_...` | E2E, SQL | PASS | AA-1: label uniqueness not enforced |
| EI4-D9-D | I | Identical content idempotent | As expected | `test_e12_a_b_f_...`, `test_e9_...`, `test_e16_...`, `test_e11_...` | E2E, SQL | PASS | — |
| EI4-D9-B, EI4-D9-D | J | Changed eligibility or history under one key is a conflict, never a replacement | As expected | `test_e12_d_e_g_h_...` ×3, `test_e10_...` | E2E, SQL | PASS | Commit order does not prove correctness |
| EI4-D1, EI4-D9-C | K | Derived modeled issuance; entry bar at or after it; no look-ahead | As expected | `test_e13_...` ×4, `test_e5_...` | E2E | PASS | Synthetic latencies and cutoffs only |
| EI4-D9-H, clarification D | L | As-of, complete, watermarked history; missing is not empty; never written | As expected | `test_e14_...` ×4 | E2E | PASS | No production issued-history authority |
| EI4-D9-E | M | One transaction; no partial record; failures reported | As expected | `test_e17_...` ×7 | INJ, SQL | PASS | Adapter fault-injection seams |
| EI4-D9-G | N | Uncertain COMMIT reconciled by verified reload; bounded retry | As expected | `test_e18_...` ×9 | INJ, SQL | PASS | Adapter fault-injection seams |
| EI4-D9-F, EI3-D12 | O | Serialized writers; one authoritative disposition | As expected | `test_e19_...` ×4 | SQL | PASS | One SQLite file |
| EI4-D9-F | P | Serialized across separate OS processes | As expected on Windows | `test_e19_separate_os_processes_...` ×2 | SQL | PASS | Windows only |
| EI4-D9-F | P | Separate-process concurrency on POSIX | Not run | — | — | **NOT TESTED** | Windows development machine only |
| EI4-D9-I, EI3-D13 | Q | All kinds reload, verify and replay from frozen evidence with references unavailable | As expected | `test_e15_...` | E2E, SQL | PASS | Reopened in the same OS process |
| EI4-D9-I | Q | Replay from a separate OS process after restart | Not run | — | — | **NOT TESTED** | Separate-process writing is covered by P |
| EI4-D9-I, EI3-D13 | R | Correction never relinks or rewrites | As expected | `test_e16_...` | E2E, SQL | PASS | EI0-D21 not implemented |
| EI4-D9-I, ADR-021 section 11 | S | Insert-only; corruption fails closed; never repaired | As expected | `test_e20_...` ×9, `test_e9_...` | INJ, SQL | PASS | AA-2: SQLite does not verify semantics |
| W1 (EI-4D-A) | T | Adapter raises typed errors; recorder maps them; state unknown | As expected | `test_e21_...` ×3 | INJ, SQL | PASS | — |
| ADR-021 section 11 | U | Additive schema; earlier evidence unchanged | As expected | `test_e22_...` ×2 | SQL | PASS | — |
| EI4-D9-J, EI4-D9-H | D, L | Generation records only; no issuance or history writes | As expected | `test_e8_...`, `test_e14_recording_...` | E2E, SQL | PASS | — |

**UNRESOLVED as a category passed.** Only the Core search-limit route to it is NOT TESTED.

## 11. Full-suite results

Local runs on the development machine. None is a CI artifact.

| Check | Result |
| --- | --- |
| EI-4E acceptance tests | 72 passed (38 normal-path, 34 fault-path) |
| EI-4D-B tests (`tests/test_sqlite_equity_recommendation_generation.py`) | 73 passed |
| Related EI-2B, EI-3C and equity evidence and research tests (9 modules) | 382 passed |
| Application EI-4C and EI-4D-A tests, run read-only at `bed91de` | 178 passed |
| Full Infrastructure suite | 2,808 passed, 27 skipped, 0 failed |
| Ruff check, Ruff format check, `git diff --check` (Infrastructure) | clean |
| CI | **not run**; no CI result is claimed |

The 27 Infrastructure skips are pre-existing and unrelated to EI-4E: 23 opt-in live Databento acceptance tests and 4 POSIX-only `tzset` tests. The 2,808 passes are the earlier 2,736-test baseline plus the 72 EI-4E tests.

The results above were produced during the EI-4E implementation review. For this record, at Infrastructure `5dda518` on 2026-11-08:

- the full Infrastructure suite was re-run, with the same result: 2,808 passed, 27 skipped, 0 failed;
- pytest collection of the two EI-4E modules confirmed the per-prefix counts of [section 9](#9-test-references): 72 tests, 34 of them fault-path.

The other rows were not re-run for this record.

## 12. Engineering verdict

**PASS WITH DOCUMENTED LIMITATIONS.**

The evidence establishes the complete synthetic source-to-disposition pipeline of [section 6.1](#61-the-source-pipeline-exercised) on real SQLite, through merged production code, for every disposition kind and for attempt-only results. That includes restart, replay from frozen evidence, conflicts, uncertain commits, concurrent writers and tampering.

The verdict is not a plain PASS because of:

- the routes recorded as NOT TESTED in [section 10](#10-acceptance-matrix);
- the limitations in [section 13](#13-explicit-limitations);
- two findings that need an owner decision ([section 14](#14-open-owner-decisions)).

The verdict is an engineering result only. It does not resolve AA-1 or AA-2, approve any decision, or authorize anything in [section 15](#15-non-authorizations).

## 13. Explicit limitations

These remain part of the acceptance conclusion.

1. The Core `SIZING_SEARCH_LIMIT_EXCEEDED` route to UNRESOLVED is not tested end to end. The exclusivity and exposure routes to UNRESOLVED are tested and passed.
2. Cross-process concurrency was tested on Windows only, with the `spawn` start method.
3. Historical replay was performed after reopening a database in the same OS process: objects dropped, the file moved and copied, fresh adapters.
4. One synthetic trading date (2026-08-19) and one venue reference were used.
5. Only synthetic listing, pricing, policy, eligibility and issued-history inputs were used.
6. The evaluator version is validated, but no content fingerprint binds it. It is compared explicitly, and no second evaluator was introduced.
7. No production issued-recommendation-history authority exists. Every history used is a labelled synthetic fixture.
8. The SQLite authority is limited to a single database file on one host.
9. There is no multi-host or distributed consistency guarantee.
10. Some failure tests use explicit adapter fault-injection seams (`_connect`, `_insert_disposition`, `_insert_attempt`, `_commit`, `_BUSY_TIMEOUT_SECONDS`) and test-only `bypass()` corruption.
11. The "no hidden second transaction" check relies on SQLite's statement trace. That trace repeats an INSERT as its row triggers run, and the test collapses consecutive repeats.
12. Test results are local. CI is not confirmed.

## 14. Open owner decisions

### 14.1 AA-1 — Policy-version label governance

**Status: OWNER DECISION PENDING.**

**Finding.** The same synthetic policy-version label can identify different canonical policy contents. The `eirs1` policy-set fingerprint distinguishes the contents and so produces distinct generation keys. As a result, several dispositions can coexist for one candidate under a reused label.

**Evidence.** `test_e12_c_a_reused_version_label_with_changed_content_is_another_identity`:

- the label `synthetic-ei4e-timing/1` is reused with a 240-second latency in place of 150 seconds;
- every version label in the request is identical;
- the policy-set fingerprint and the generation key differ;
- the second disposition commits, and the first row is untouched.

**What holds today.** The changed content never passes as the identical policy set, as EI4-D9-B requires. Label uniqueness is **not** enforced anywhere: not by the Application, the store or the database.

> **PROPOSAL — NOT APPROVED.** Within a policy namespace, a published version label should identify one immutable canonical policy definition, and conflicting reuse of a label should be rejected.

This record does not change the `eirk1` identity scheme or the `eirs1` fingerprint, and implements nothing for AA-1.

### 14.2 AA-2 — Verified-writer boundary and recovery

**Status: OWNER DECISION PENDING.**

**Finding.**

- The direct SQLite store validates structure and database provenance: insert-only triggers, CHECK constraints, foreign keys, and the insert-time checks that a disposition rests on the timeline's synthetic CANDIDATE.
- It does not independently perform full authoritative source verification or historical assessment replay. As ADR-021 section 10 records, SHA-256 and evidence verification are the Application's.
- A structurally valid but historically contradictory disposition can therefore be persisted through a direct adapter call. Later verification detects the contradiction and refuses the record.
- Because the record is immutable, it can occupy a generation key and prevent a valid subsequent record from being committed.

**Evidence.** `test_e20_contradictory_candidate_evidence_is_caught_by_the_application_verifier` ×2 (synthetic signal close or opening-range low altered):

1. The verified recorder refuses the forged record before writing: SOURCE_EVIDENCE_INVALID, no rows.
2. A direct `SQLiteEquityRecommendationGenerationStore.record_generation` call commits it.
3. `VerifyStoredEquityRecommendationGenerationUseCase` refuses it, naming the altered field.
4. The genuine record then receives STORED_RECORD_INVALID. It is not committed, the forged row is not overwritten, and the forged record is never adopted as authoritative.

**What holds today.**

- Contradictory stored content is never treated as authoritative.
- Nothing makes the verified recorder the only operational write path.
- No recovery procedure exists for a key occupied by an invalid record.
- SQLite itself does not verify historical semantics.

> **PROPOSAL — NOT APPROVED.** The verified Application recorder (`RecordEquityRecommendationGenerationUseCase`) should be the mandatory operational generation write boundary. Invalid committed records should fail closed and be handled through a separately governed recovery process that preserves the original evidence.

This record introduces no deletion, overwrite, mutable status transition or recovery path, and implements nothing for AA-2.

### 14.3 Decisions still pending, unchanged

This record changes none of these. Each keeps its status in the [EI-4 Decision Register, section 5 and section 9.5](Equity-Intraday-V1-EI4-Decision-Register.md#95-decisions-still-pending-or-unapproved):

- EI4-D5, pre-entry invalidation (PROPOSED, deferred);
- EI0-D11, risk parameters, including any portfolio exposure or simultaneous-open limit;
- the proposed same-listing exclusivity threshold of one open recommendation per listing (product specification, section 9.5);
- EI0-D19, daily modeled loss limit;
- EI0-D21, candle-revision materiality;
- EI3-D10, trailing unfinalized candle policy.

The tests treat these as unapproved:

- prior issuances yield UNRESOLVED, never a breach or a pass;
- the corresponding controls are reported NOT_ASSESSED;
- correction materiality and withdrawal are not implemented.

No production parameter is set or implied.

## 15. Non-authorizations

EI-4E acceptance is synthetic engineering acceptance only. It does **not** authorize:

- actual recommendation issuance;
- live-market use;
- real-stock or provider readiness;
- broker or paper orders;
- trading execution;
- open positions;
- fills;
- realized P&L;
- profitability claims;
- production trading;
- actual risk-policy approval;
- data-use rights or exchange licensing;
- EI-5 or EI-6 implementation.

Every restriction in the [EI-4 Decision Register, section 7](Equity-Intraday-V1-EI4-Decision-Register.md#7-production-readiness-restrictions) and section 9.7 still applies. A stored PROPOSED disposition is a synthetic research artifact. It is not an issued recommendation, not in the `RECORDED` lifecycle state, not actionable, and not evidence of profitability.

## 16. Milestone closure boundaries

Three states are distinct, and only the first two are established by this record:

1. **Implemented and merged.** The EI-4E engineering acceptance tests are merged to northstar-infrastructure `develop` (`5dda518`, PR #54).
2. **Engineering verdict recorded.** The EI-4E engineering verdict is recorded here as PASS WITH DOCUMENTED LIMITATIONS.
3. **Owner decisions open.** AA-1 and AA-2 are not approved and not implemented.

This record does not close the EI-4E milestone or EI-4 as a whole. That governance closure is the product owner's decision. Nothing here describes Equity Intraday V1 as production-ready.

## 17. Follow-up requirements

1. **Owner decisions on AA-1 and AA-2.** Any approved policy needs its own governed slice; for AA-2 that includes a recovery procedure, and an ADR-021 update where architecture changes.
2. **Owner disposition of the NOT TESTED routes.** Either accept them as recorded, or direct further acceptance:
   - the Core `SIZING_SEARCH_LIMIT_EXCEEDED` route;
   - separate-process concurrency on POSIX;
   - replay from a separate OS process.
3. **CI confirmation** of the Infrastructure suite, if required for closure. No CI result exists for this evidence.
4. **Review.** ADR-021 records no Architecture Review or Engineering Review sign-off; that remains pending.
5. **No implementation follows from this record.** EI-5 and EI-6 remain separate milestones, and the pending decisions in [section 14.3](#143-decisions-still-pending-unchanged) remain pending.

## 18. Related documents

- [Equity Intraday V1 EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md)
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [Equity Intraday V1 EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md)
- [Equity Intraday V1 EI-3C Synthetic Acceptance Record](Equity-Intraday-V1-EI3C-Synthetic-Acceptance-Record.md)
- [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md)
- [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md)
- [ES-003: Testing Standards](../engineering/standards/ES-003-Testing-Standards.md)
