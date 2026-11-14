# Equity Intraday V1 EI-4C Versioned History Compatibility Engineering Acceptance Record

## Status

**Engineering acceptance record, 2026-11-14.** This is slice 5 of the [EI-4C Versioned History Compatibility Design v1.0, section 13](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md#13-implementation-sequence).

- **Verdict:** EI-4C VERSIONED HISTORY COMPATIBILITY ENGINEERING ACCEPTANCE: **PASS WITH DOCUMENTED LIMITATIONS — synthetic engineering acceptance** ([section 14](#14-engineering-verdict-and-recommended-status)).
- **Owner acceptance: ACCEPTED WITH DOCUMENTED LIMITATIONS, 2026-11-14.** The owner accepted slice 5 and the EI-4C versioned history compatibility prerequisite on 2026-11-14, with VC-04 accepted as NOT VERIFIED and every limitation retained ([section 14.1](#141-owner-acceptance-2026-11-14)).
- **What it covers:** the merged slices 1, 2, the Infrastructure evaluator-set correction, the combined slices 3a + 3b, and slice 4 ([section 4](#4-repository-baselines-and-merge-sequence)).
- **Acceptance matrix:** all 52 scenarios are traced to evidence: **40 VERIFIED, 8 VERIFIED WITH LIMITATION, 1 NOT VERIFIED, 3 PLANNED** (EI-5D-owned) ([section 10](#10-acceptance-evidence-matrix)).
- **CI verification: NOT INDEPENDENTLY VERIFIED FROM AVAILABLE EVIDENCE.** Every result below is a local execution on the Windows development machine ([section 12.2](#122-ci-status)).
- Sections 1 to 13 are engineering acceptance evidence, and are **not** themselves an owner approval. The owner's acceptance of 2026-11-14 is recorded in [section 14.1](#141-owner-acceptance-2026-11-14) and in the [EI-5 Decision Register, section 14.4](Equity-Intraday-V1-EI5-Decision-Register.md#144-owner-acceptance-of-slice-5-2026-11-14). Otherwise:
  - the record approves no other decision; the owner approvals of R-1, R-2, N1 to N10 and the N10 case-C classification (2026-11-13) are unchanged;
  - the N9 sequencing amendment it reports was owner approved separately and is recorded in the [EI-5 Decision Register, section 14.1](Equity-Intraday-V1-EI5-Decision-Register.md#141-n9-sequencing-amendment-combined-slices-3a--3b-owner-approved-2026-11-14);
  - it does **not** authorize EI-5D, issuance, real data or any production use ([section 15](#15-non-authorizations)).

## 1. Acceptance identity

| Item | Value |
| --- | --- |
| Milestone | EI-4C versioned history compatibility (the EI-5D prerequisite), slices 1 to 5 |
| Record | Slice 5, documentation acceptance |
| Date | 2026-11-14 |
| Governance | [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md); [EI-5 Decision Register, sections 13 and 14](Equity-Intraday-V1-EI5-Decision-Register.md#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13) |
| Design | [EI-4C Versioned History Compatibility Design v1.0](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md) |
| Data | Synthetic only |
| Platform | Windows development machine; local executions only |

## 2. Scope

**In scope:** the closed two-evaluator table and verification dispatch (N1); the `/2` inputs, assessment and attempt formats under the retained prefixes (N3, N4); same-key cross-version conflicts (N5); label-format pairing (N6); keyword-only evaluator selection (N7); the strict-before-issuance history boundary (N8); exact completeness and its cases A to D (N10); version-1 preservation; and real SQLite compatibility with both versions.

**Out of scope:** EI-5D issuance, `eiris1` records, issuance authority, declarations, monitoring (EI-6), lifecycle persistence, the deferred SQLite pairing trigger (N6), any change to Core `PointInTime` semantics, API, Web, and every production or real-data use.

## 3. Governance references

| Layer | State | Where |
| --- | --- | --- |
| Owner-approved governance | R-1, R-2, N1 to N10, the N10 case-C classification (2026-11-13) | [EI-5 Decision Register, section 13](Equity-Intraday-V1-EI5-Decision-Register.md#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13); [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) |
| Owner-approved sequencing amendment | Slices 3a and 3b combined into one Application PR (2026-11-14) | [EI-5 Decision Register, section 14.1](Equity-Intraday-V1-EI5-Decision-Register.md#141-n9-sequencing-amendment-combined-slices-3a--3b-owner-approved-2026-11-14) |
| Implemented engineering behaviour | All implementation slices merged | [Section 4](#4-repository-baselines-and-merge-sequence) |
| Locally validated evidence | Reported suite results per slice | [Section 12.1](#121-historically-reported-local-results) |
| CI verification | Not independently verified | [Section 12.2](#122-ci-status) |
| Remaining limitations | L1 to L9 | [Section 13](#13-remaining-limitations) |

ADR-021 (amended by ADR-025 section 1 through a dated pointer), ADR-022, ADR-023 and ADR-024 are unchanged. The EI-4E, EI4-D10 and EI4-D11 acceptance records and their verdicts are unchanged.

## 4. Repository baselines and merge sequence

The merged revisions were read from local Git history on 2026-11-14. Every repository was on `develop` with a clean working tree. No fetch was run, so remote-tracking references are as fresh as the last fetch.

| Repository | `develop` HEAD | Role |
| --- | --- | --- |
| northstar-application | `d908674` (#95) | Slice 1; combined slices 3a + 3b |
| northstar-infrastructure | `f71f8f7` (#66) | Slice 2; evaluator-set correction; slice 4 |
| northstar-core | `29bd34a` (#40) | Year-formatting portability fix only; no EI-4C contract change |
| northstar-docs | `8b15d42` (#59) | Slice 0; this record, uncommitted |
| northstar-api | `7ad37f5` | Unchanged |
| northstar-web | `5aeb94c` | Unchanged |

**Merge sequence** (committer times, IST):

| # | Slice | Repository | PR | Change commit | Merge commit | Merged |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 0. Governance (ADR-025, design, registers) | northstar-docs | #59 | `7d6d1a2` | `8b15d42` | 2026-11-13 02:20 |
| 2 | 1. Version-1 golden-vector freeze | northstar-application | #94 | `a867248` | `ba79805` | 2026-11-14 01:08 |
| 3 | 2. Infrastructure test hygiene and v1 persisted replay | northstar-infrastructure | #64 | `7cfd4d8` | `efe2d7b` | 2026-11-14 02:06 |
| 4 | Infrastructure evaluator-set transition correction | northstar-infrastructure | #65 | `dedd36e` | `d8a911f` | 2026-11-14 03:42 |
| 5 | Core four-digit-year portability fix (not an EI-4C slice) | northstar-core | #40 | `6be6852` | `29bd34a` | 2026-11-14 12:21 |
| 6 | Combined 3a + 3b. Closed two-version verification and version-2 generation | northstar-application | #95 | `713954e`, `9ccfd77` | `d908674` | 2026-11-14 13:45 |
| 7 | 4. Mixed-version SQLite acceptance | northstar-infrastructure | #66 | `e1f9b83` | `f71f8f7` | 2026-11-14 14:30 |

- **Slices 1 and 2 merged before the combined implementation.** The combined change commit `713954e` is dated 04:53, after both.
- **The evaluator-set correction (#65) merged before the combined Application PR (#95).** It accepts exactly `{/1}` or `{/1, /2}`, so the Infrastructure suite passes against either Application state (design section 13's cross-repository constraint).
- **Core #40 and Application `9ccfd77`** replace platform-dependent `strftime("%Y")` year formatting with fixed four-digit fields (`PointInTime` canonical form and `_exact_point`). Output for years 1000 to 9999 is unchanged; they fix years below 1000 on Linux. They change no EI-4C contract.
- **The combined implementation merged before slice 4.**
- **Change scope.** Slice 1, slice 2, #65 and slice 4 are test-only. The combined slice changes 14 Application files. Slice 4 adds one Infrastructure test module and no production code.

## 5. Sequencing amendment (N9)

N9 approved separate slices 3a (verification) and 3b (generation). On 2026-11-14 the owner approved implementing them as **one Application feature: Combined Slices 3a + 3b — Closed Two-Version Verification and Version-2 Generation**. The record is in the [EI-5 Decision Register, section 14.1](Equity-Intraday-V1-EI5-Decision-Register.md#141-n9-sequencing-amendment-combined-slices-3a--3b-owner-approved-2026-11-14). It changed PR packaging and workflow only; every architecture decision, scope item, gate and acceptance scenario of slices 3a and 3b stayed binding, and the prerequisites were met in the order of section 4.

## 6. Executive summary

- **Version 1 is preserved.** Golden documents, fingerprints and persisted rows are unchanged since they were frozen, and they verify at levels A to C under evaluator `/1` with its inclusive history at `t` ([section 7](#7-version-1-preservation)).
- **Version 2 exists, closed and paired.** Exactly two evaluators; `/2` documents at `b = t − 1 µs` with exact completeness; cases A to D with the approved case-C refusal; label-format pairing at construction, decoding, verification and authorization ([section 8](#8-version-2-requirements)).
- **SQLite needs no migration.** The existing schema and adapter store, reload, replay and refuse both versions on real SQLite ([section 9](#9-sqlite-compatibility)).
- **Cross-version keys conflict, never merge.** One `eirk1` holds one disposition, of one version; the competitor gets `EVALUATOR_VERSION_CONFLICT` and its attempt is audited, in both orders, under concurrency and after an unconfirmed commit.
- **I08** is classified as an equivalent mutant under the accepted contracts, not killed ([section 11](#11-i08-mutation-disposition)).

## 7. Version-1 preservation

| Requirement | Evidence |
| --- | --- |
| V1 golden documents unchanged | The Application fixtures `tests/application_services/golden/equity_recommendation_v1/` (11 files) have one commit in their history, `a867248` (slice 1). The Infrastructure fixtures `tests/golden/equity_recommendation_v1_persisted/` have one, `7cfd4d8` (slice 2) |
| V1 fingerprints unchanged | App `test_equity_recommendation_v1_golden_vectors.py`: `test_gv01_eiri1_…`, `test_gv02_eira1_…`, `test_gv03_eirp1_…`, `test_gv04_keyed_…`, `test_gv04_keyless_…`, three-way checks (text, recomputed fingerprint, pinned literal), passing at `d908674` |
| Historical rows reload byte-identically | Infra `test_sqlite_equity_recommendation_v1_persisted_replay.py`: `test_path_b_the_reference_loads_unchanged_through_fresh_adapters`, `test_path_b_a_fresh_process_reloads_and_verifies_the_reference`; slice 4 `test_ps04_vc15_…` (file SHA-256 unchanged after a fresh-process read) |
| V1 verifiable at levels A to C | App `test_gv06_level_a_…`, `test_gv06_levels_b_and_c_…`, `test_gv06_a_fresh_process_reproduces_every_golden_identity`; App `test_record_equity_recommendation_generation.py::test_evaluator_7_replay_requires_the_evaluator_the_record_names`; Infra `test_path_b_levels_a_to_c_verify_the_reference` |
| V1 inclusive-history semantics unchanged | App `test_gv01_version1_history_is_observed_inclusively_at_the_modeled_issuance`, `test_gv01_records_issued_exactly_at_t_are_frozen_into_version1_history`, `test_gv04_v1_history_complete_exactly_at_t_is_proposed_not_incomplete`; App `test_equity_recommendation_v2_generation.py::test_an_explicit_version_1_selection_is_the_default_byte_for_byte`, `test_version_1_never_reports_the_version_2_reason` |
| V2 never reinterprets or migrates v1 | First records are kept on cross-version conflict (App `test_evaluator_2_…`; Infra `test_vc09_…`); no repair on read (Infra `test_ps04_vc15_…`); a known label over the other version's documents never constructs (`test_evaluator_7_…`) |
| `eirk1` rules unchanged | App `test_gv05_existing_eirs1_and_eirk1_goldens_are_unchanged`, `test_gv05_generation_keys_re_derive_from_the_frozen_inputs_goldens`; the version-2 key equals the version-1 golden key (`test_case_a_version_2_proposes_on_history_complete_exactly_through_b`) |

**Governed extensions, not changes.** GV-05 and the Infrastructure contract assertion now admit the approved `/2` evaluator and the three `/2` prefix-table entries. Every `/1` pin is unchanged, and no golden fixture was regenerated.

## 8. Version-2 requirements

Application paths are in northstar-application `tests/application_services/`; "V2" is `test_equity_recommendation_v2_generation.py` and "MIX" is northstar-infrastructure `tests/test_sqlite_equity_recommendation_mixed_version.py`.

| Requirement | Implementation | Evidence |
| --- | --- | --- |
| Exactly two evaluators; closed table | `EQUITY_RECOMMENDATION_EVALUATORS` (read-only mapping) in `equity_recommendation_generation_evidence.py` | V2 `test_the_closed_table_holds_exactly_the_two_governed_evaluators` |
| Closed dispatch | `require_supported_…` and `require_paired_equity_recommendation_evaluator` in `equity_recommendation_generation_decoding.py`; verifier dispatch in `verify_equity_recommendation_generation.py` | V2 `test_levels_b_and_c_a_version_2_record_verifies_under_version_2`; `test_evaluator_7_…` |
| Canonical `/2` inputs, assessment, attempt | Format constants and the version table | V2 `test_case_a_version_2_proposes_…`, `test_level_a_a_version_2_record_decodes_re_encodes_and_pairs` (pinned `eiri1`, `eira1`, `eirat1`) |
| `/1` proposal, key and policy formats | Unchanged | V2 `test_case_a_…` (proposal `/1`, key and `eirp1` equal the version-1 goldens) |
| Retained prefixes | Three `/2` entries in `EQUITY_RECOMMENDATION_DOCUMENT_PREFIXES` | V2 `test_the_closed_table_…`; App `test_gv05_version1_labels_formats_prefixes_and_key_namespace_are_pinned` |
| Mandatory pairing | Construction, decoding, verifier, recorder | `test_evaluator_7_…`; V2 `test_a_resealed_version_2_record_with_version_1_semantics_is_invalid`, `test_version_1_inputs_never_state_a_boundary_and_never_nest_under_version_2`, `test_the_recorder_refuses_an_attempt_of_another_evaluator` |
| Selection | Keyword-only `evaluator_version`, default `/1` | V2 `test_selection_is_keyword_only_closed_and_defaults_to_version_1` |
| History read at `b = t − 1 µs` | `equity_history_boundary` (integer microseconds) | V2 `test_the_boundary_is_the_last_representable_instant_before_t` (7 rows), `test_case_a_…` (history asked as of `b`); MIX `test_a_version2_proposed_record_persists_…` |
| `complete_through == b` | `_history_before` in `generate_equity_trade_recommendation.py` | V2 `test_case_a_…`, `test_case_a_the_strict_boundary_includes_b_and_never_t` |
| Case B | `RECOMMENDATION_HISTORY_INCOMPLETE`, `DEPENDENCY_UNAVAILABLE`, keyless | V2 `test_case_b_…`; MIX `test_ps02_…[case B …]` |
| Case C | See below | V2 `test_case_c_history_declared_beyond_b_is_the_approved_refusal` (3 instants), `test_case_c_is_recorded_as_one_idempotent_keyless_attempt_and_consumes_no_key`; MIX `test_case_c_is_dependency_unavailable_attempt_only_and_leaves_the_key_unconsumed`, `test_ps02_…[case C …]` |
| Case D | `OPEN_RECOMMENDATION_STATE_UNAVAILABLE`, keyless | V2 `test_case_d_…` (missing, unavailable, another session); MIX `test_ps02_…[case D missing]` |
| Deterministic attempt identity | `generation-attempt/2` | V2 `test_keyless_attempts_are_distinct_across_versions_and_stable_within_one`, `test_repeated_version_2_generation_is_identical`, `test_a_fresh_process_reproduces_the_version_2_identities` |
| Levels A to C | Dispatched verifier | V2 `test_level_a_…`, `test_levels_b_and_c_…`; MIX `test_ps04_vc15_…` |
| Same-key conflict; first record kept | Recorder `EVALUATOR_VERSION_CONFLICT`; store `CONFLICT` | `test_evaluator_2_…` (both orders); MIX `test_vc09_…` (both orders), `test_ps05_…`, `test_ps06_…` |

**Case C, as approved and as verified.** History declared complete beyond `b` gives reason `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, status `DEPENDENCY_UNAVAILABLE`, evaluator `northstar.equity-recommendation-evaluator/2` only. It persists a keyless `generation-attempt/2` only: no disposition of any kind, no proposal, and the generation key is not consumed (a later case-A record commits under it). An identical re-run is the same logical attempt (`EQUIVALENT_COMMITTED`). Version 1 never produces the reason, and a version-1 audit or attempt stating it is refused. No issuance record exists to create, and no generation path writes or reverses a completeness declaration.

## 9. SQLite compatibility

**Finding: VERIFIED on real SQLite, 2026-11-14, with no migration** (slice 4, MIX; Windows local runs).

| Property | Evidence |
| --- | --- |
| No migration; nothing altered | `test_ps08_ps01_the_current_schema_accepts_version2_rows_with_no_migration`: `sqlite_master`, `PRAGMA user_version` and both tables' `table_info` identical before and after version-2 writes; no version or migration object |
| Format columns compatible | `/2` assessment and attempt formats and `/1` proposal format stored and checked against `$.format` (same test; `test_a_version2_proposed_record_persists_…`) |
| Prefixes valid | `eirk1`, `eira1`, `eirp1`, `eiri1`, `eirs1`, `eirat1` GLOB checks satisfied by version-2 rows (same tests) |
| Foreign-key provenance | Candidate entry, decision, snapshot and manifest provenance asserted; `test_foreign_keys_remain_enforced_for_version2_rows` (source trigger, then the foreign key with the trigger dropped; adapter connection `PRAGMA foreign_keys = 1`) |
| Insert-only | `test_ps03_update_delete_and_replacement_are_refused_for_version2_rows` (11 statements, including relabelling to `/1`); `test_vc10_…` (no-replace trigger, then `UNIQUE` with the trigger dropped) |
| `BEGIN IMMEDIATE` atomicity | `test_version2_writes_are_one_transaction_and_identical_retries_are_idempotent` (traced: `BEGIN IMMEDIATE`, two inserts, `COMMIT`; a retry writes nothing; a conflict appends only the attempt) |
| Authorized writes | Every success path through the verified recorder and `write_generation`/`write_attempt`; `test_ps07_vc16_…` (wrong-version writes refused before any connection opens) |
| Historical v1 rows unchanged | GV-07 module unchanged in substance; `test_ps04_vc15_…` |
| Coexistence and replay | `test_vc13_vc14_…`; `test_ps04_vc15_…` (fresh process, levels A to C, each under its own label) |
| First-record authority | `test_vc09_…` (both orders); `test_same_version2_key_with_other_content_is_a_content_conflict` (within one version) |
| Concurrency and commit uncertainty | `test_ps05_…` (spawned processes); `test_ps06_…` (both directions: never equivalent) |
| Failure classification | `test_a_corrupted_stored_version2_disposition_is_classified_and_never_repaired` (fingerprint, non-canonical, relabelled to `/1` → Invalid; `/999`, `assessment/9` → Incompatible; never repaired); `test_missing_provenance_of_a_version2_record_fails_source_verification` |

**SQLite does not enforce label-format pairing.** A row with a mismatched label can be written by privileged raw SQL; the adapter refuses it on load as Invalid (tested). The database-level pairing trigger stays **DEFERRED** under N6.

## 10. Acceptance evidence matrix

**Statuses** follow the existing acceptance records: VERIFIED; VERIFIED WITH LIMITATION; NOT VERIFIED; PLANNED (not yet executable: owned by a later slice). A merged PR alone is never counted as evidence.

**Paths.** GV: northstar-application `tests/application_services/test_equity_recommendation_v1_golden_vectors.py`. V2: `…/test_equity_recommendation_v2_generation.py`. REC: `…/test_record_equity_recommendation_generation.py`. AUTH: `…/test_equity_recommendation_generation_write_authorization.py`. GEN: `…/test_generate_equity_trade_recommendation.py`. P7: northstar-infrastructure `tests/test_sqlite_equity_recommendation_v1_persisted_replay.py`. MIX: `tests/test_sqlite_equity_recommendation_mixed_version.py`. SQL: `tests/test_sqlite_equity_recommendation_generation.py`. FLT: `tests/test_equity_recommendation_generation_acceptance_faults.py`. ACC: `tests/test_equity_recommendation_generation_acceptance.py`.

**The issuance-eligibility column.** No issuance code exists. Where a VC row states issuance behaviour, it is carried by the EI-5D scenarios (VC-21 to VC-23 and the EI-5 design's IS rows), not by slices 1 to 4, and the rows below are assessed on their decoding, verification, generation and recording columns.

| ID | Scenario | Evidence | Limitation | Status |
| --- | --- | --- | --- | --- |
| VC-01 | Valid version-1 record | GV `test_gv06_level_a_…`, `test_gv06_levels_b_and_c_…`; V2 `test_an_explicit_version_1_selection_is_the_default_byte_for_byte`; REC `test_evaluator_1_…`; P7 `test_path_b_levels_a_to_c_verify_the_reference` | — | VERIFIED |
| VC-02 | Valid version-2 record | V2 `test_case_a_…`, `test_level_a_…`, `test_levels_b_and_c_…`, `test_a_version_2_record_is_recorded_through_the_authorized_path_idempotently`; MIX `test_a_version2_proposed_record_persists_…` | — | VERIFIED |
| VC-03 | Version-1 record through the new verifier | GV `test_gv06_…` (goldens frozen from pre-3a code, verified by the merged dispatcher); REC `test_evaluator_7_…` | — | VERIFIED |
| VC-04 | Version-2 record met by legacy-only code | None executed. Analogous current-code evidence only: REC `test_evaluator_3_…` (unknown label), V2 `test_unknown_versions_within_a_family_are_incompatible` | No test ran pre-3a (v1-only) code against a genuine version-2 record | NOT VERIFIED |
| VC-05 | Unknown evaluator label | REC `test_evaluator_3_…` (`STORED_RECORD_INCOMPATIBLE`); V2 `test_selection_is_keyword_only_…` (selection refused), `test_unknown_versions_…`; SQL `test_p_an_unsupported_stored_evaluator_is_incompatible`; FLT `test_e20_…[unsupported evaluator]`; MIX `test_a_corrupted_…[unknown evaluator]` | — | VERIFIED |
| VC-06 | Missing or empty label | REC `test_evaluator_4_…`; V2 `test_unknown_versions_…` (empty); SQL and FLT `[empty evaluator]` (planted row loads as Invalid) | The SQLite `NOT NULL`/`length > 0` CHECK is shown only indirectly: tests must disable CHECKs to plant an empty label; its refusal is not asserted directly | VERIFIED WITH LIMITATION |
| VC-07 | Label and formats mismatched | REC `test_evaluator_7_…` (both directions, "do not pair"); V2 `test_a_resealed_…[assessment /2 embedding inputs /1]`, `test_version_1_inputs_never_…` (nested and relabelled), `test_the_recorder_refuses_an_attempt_of_another_evaluator` | Design section 8's last row (proposal, policy set or Core outcome other than `/1`): only `proposal/9` is refused (GV `test_neg01_…`); no test places a non-`/1` policy-set or Core-outcome format inside a version-2 record | VERIFIED WITH LIMITATION |
| VC-08 | Correct version, wrong fingerprint | GV `test_neg14_a_tampered_fingerprint_is_refused`; MIX `test_a_corrupted_…[tampered fingerprint]`; FLT `test_e20_…[invalid fingerprint]` | — | VERIFIED |
| VC-09 | Same generation key across versions | REC `test_evaluator_2_…[/1, /2]`; MIX `test_vc09_…[A v1 first, B v2 first]`, `test_ps05_…`, `test_i08_…` | — | VERIFIED |
| VC-10 | Same `eirp1` under different assessments | REC `_both_versions` (same key and proposal); MIX `test_vc10_the_shared_proposal_identity_cannot_add_a_second_row` | `eiris1` is EI-5D's | VERIFIED |
| VC-11 | Version-1 retry | REC `test_evaluator_1_…` (`EQUIVALENT_COMMITTED`); ACC `test_e10_an_unresolved_disposition_is_immutable_and_never_becomes_proposed` (changed history evidence → `CONTENT_CONFLICT`) | — | VERIFIED |
| VC-12 | Version-2 retry | V2 `test_repeated_version_2_generation_is_identical`, `test_a_version_2_record_is_recorded_…_idempotently`, `test_case_c_is_recorded_as_one_idempotent_…`; MIX `test_same_version2_key_with_other_content_is_a_content_conflict` | The specific "competing issuance at `t` changes only the watermark" variant is not exercised; the version-2 `CONTENT_CONFLICT` path is shown with other changed history evidence | VERIFIED WITH LIMITATION |
| VC-13 | V1 and v2 attempts for equivalent requests | V2 `test_keyless_attempts_are_distinct_…`, `test_case_a_every_version_2_identity_differs_…`; MIX `test_vc13_vc14_…` | — | VERIFIED |
| VC-14 | Mixed-version persistence | MIX `test_vc13_vc14_…`, `test_ps04_vc15_…` | — | VERIFIED |
| VC-15 | Mixed-version restart and replay | MIX `test_ps04_vc15_…` (separate interpreter; levels A to C per label; byte-identical; file unchanged) | Windows local execution only; no POSIX or CI result | VERIFIED WITH LIMITATION |
| VC-16 | Wrong-version authorized write | REC `test_evaluator_7_…` (cannot construct); AUTH `test_c_a_disposition_changed_after_issuance_is_refused[evaluator_version]`; MIX `test_ps07_vc16_…` (relabelled, swapped attempt, swapped content, swapped keyless attempt; no connection opened) | — | VERIFIED |
| VC-17 | Case A | V2 `test_case_a_…`, `test_case_a_the_strict_boundary_…` (other failures still decide), `test_levels_b_and_c_…`; MIX `test_ps08_ps01_…` (v2 NOT_ELIGIBLE and UNRESOLVED at `b`) | — | VERIFIED |
| VC-18 | Case B | V2 `test_case_b_…`; MIX `test_ps02_…[case B]` | — | VERIFIED |
| VC-19 | Case C | V2 `test_case_c_…` (×2 tests); MIX `test_case_c_…`, `test_ps02_…[case C]` | Issuance column: no issuance code exists | VERIFIED |
| VC-20 | Case D | V2 `test_case_d_…` (×3); MIX `test_ps02_…[case D]` | — | VERIFIED |
| VC-21 | Unverifiable watermark | Owned by EI-5D (layer I, slice 6) | — | PLANNED |
| VC-22 | Issuing a version-1 proposal | Owned by EI-5D | — | PLANNED |
| VC-23 | Competing issuance proposals | Owned by EI-5D | — | PLANNED |
| VC-24 | Unknown or unapproved document format | V2 `test_unknown_versions_…` (`assessment/3`, `attempt/9`); REC `test_d_aj_…`; GEN `test_compat_j_a_missing_or_unsupported_format_is_refused` (Invalid); MIX `test_a_corrupted_…[unknown format]` | — | VERIFIED |
| VC-25 | Replay after a later Core change | GV goldens pin `eira1`/`eirp1` including the embedded Core outcome; V2 `test_a_version_2_replay_that_does_not_reproduce_fails_closed` (`REPLAY_MISMATCH`) | No actual Core implementation change was exercised; detection is shown by pinned identities and by altered formation output | VERIFIED WITH LIMITATION |
| GV-01 | `eiri1` golden | GV `test_gv01_…` (5 tests) | — | VERIFIED |
| GV-02 | `eira1` goldens | GV `test_gv02_…` | — | VERIFIED |
| GV-03 | `eirp1` golden | GV `test_gv03_…` | — | VERIFIED |
| GV-04 | `eirat1` keyed and keyless | GV `test_gv04_…` | — | VERIFIED |
| GV-05 | `eirk1`, `eirs1`, label and `/1` formats | GV `test_gv05_…` (3 tests) | — | VERIFIED |
| GV-06 | Goldens replay A to C | GV `test_gv06_…` (7 tests, including a fresh process) | — | VERIFIED |
| GV-07 | Persisted v1 row on real SQLite | P7 (14 tests: paths A and B, fresh process, tamper by layer) | — | VERIFIED |
| TH-01 | Application future-label hygiene | REC `_FUTURE = …/999`, `test_evaluator_8_…`; GV `test_neg_the_unsupported_test_label_…` | — | VERIFIED |
| TH-02 | Infrastructure future-label hygiene | SQL `test_o_…`, `test_o_a_…`, `test_p_…`; FLT `_UNSUPPORTED_EVALUATOR`; P7 `test_the_version1_contract_…` | — | VERIFIED |
| TH-03 | Replacement label unused and ungoverned | REC `test_evaluator_8_…` (searches Application sources); P7 asserts `/999` outside `{/1, /2}`; read-only search on 2026-11-14 found `evaluator/999` in no production source of Core, Application or Infrastructure | — | VERIFIED |
| HB-01 | `t` without fraction | V2 `test_the_boundary_…` | — | VERIFIED |
| HB-02 | `t` ending `.000001Z` | V2 `test_the_boundary_…` | — | VERIFIED |
| HB-03 | Second and minute rollover | V2 `test_the_boundary_…` | — | VERIFIED |
| HB-04 | Hour rollover | V2 `test_the_boundary_…` | — | VERIFIED |
| HB-05 | UTC midnight | V2 `test_the_boundary_…` | — | VERIFIED |
| HB-06 | IST offset; IST midnight | V2 `test_the_boundary_…` (IST rows) | — | VERIFIED |
| HB-07 | Canonical round trip | V2 `test_level_a_…` | — | VERIFIED |
| HB-08 | Deterministic replay, fresh process | V2 `test_repeated_version_2_generation_is_identical`, `test_a_fresh_process_reproduces_the_version_2_identities` | — | VERIFIED |
| HB-09 | Inputs `/2` and `/1` refusals | V2 `test_a_resealed_…` (as of `t`; complete beyond `b`; boundary missing; boundary changed); GV `test_neg06_a_changed_history_instant_is_refused_for_version1` | Only the "beyond `b`" direction of `complete_through ≠ b` is exercised as an inputs document; "before `b`" is exercised at generation (case B) | VERIFIED WITH LIMITATION |
| PS-01 | V2 three kinds insert | MIX `test_ps08_ps01_…` | — | VERIFIED |
| PS-02 | V2 attempts insert; v2 attempt under a v1 disposition | MIX `test_ps02_…` (×3), `test_vc09_…[A v1 first]` | — | VERIFIED |
| PS-03 | Append-only v2 rows | MIX `test_ps03_…` | — | VERIFIED |
| PS-04 | Mixed file after restart, fresh process | MIX `test_ps04_vc15_…` | Windows local execution only | VERIFIED WITH LIMITATION |
| PS-05 | Concurrent v1 and v2 processes | MIX `test_ps05_…` (`spawn`) | Windows only; POSIX not established (as EI4-C2) | VERIFIED WITH LIMITATION |
| PS-06 | Unconfirmed commit reconciling to the other version | MIX `test_ps06_…` (both directions) | — | VERIFIED |
| PS-07 | EI4-D11 boundary and scans unchanged; refusal before connection | MIX `test_ps07_vc16_…`; the architecture scan modules (Application `test_equity_recommendation_generation_write_architecture.py`, last changed `9db9353`; Infrastructure, last changed `d6591a3`) predate EI-4C and pass in the full suites | — | VERIFIED |
| PS-08 | No table altered, no migration | MIX `test_ps08_ps01_…` | — | VERIFIED |

**Totals: 40 VERIFIED, 8 VERIFIED WITH LIMITATION, 1 NOT VERIFIED, 3 PLANNED = 52.** By group: VC 16 / 5 / 1 / 3; GV 7 / 0 / 0 / 0; TH 3 / 0 / 0 / 0; HB 8 / 1 / 0 / 0; PS 6 / 2 / 0 / 0. A scenario is counted once, whatever the number of test cases.

### 10.1 Scenarios VERIFIED WITH LIMITATION

VC-06, VC-07, VC-12, VC-15, VC-25, HB-09, PS-04 and PS-05. Each limitation is a coverage boundary, not a known defect. VC-15, PS-04 and PS-05 are platform limitations; the others are narrow variants not exercised by a dedicated test.

### 10.2 Scenario NOT VERIFIED

**VC-04 — a version-2 record met by legacy-only (pre-3a) code.** The expected outcome is `Incompatible` with `STORED_RECORD_INCOMPATIBLE`. By inspection, the pre-3a decoder required the exact `/1` formats and a `/1`-only supported set, and the current code's handling of unknown labels and unknown family versions is tested. But no test executed pre-3a code (for example Application `ba79805`) against a genuine version-2 record.

**Owner disposition, 2026-11-14: accepted as a documented limitation; VC-04 remains NOT VERIFIED.** Compatibility of pre-3a Application implementations with stored evaluator-v2 records is **not guaranteed**, and no claim of legacy-reader compatibility is approved. Rollback or downgrade to a pre-3a Application version against stored evaluator-v2 records is **NOT AUTHORIZED** without separate compatibility verification. No additional testing was required before slice-5 acceptance.

### 10.3 PLANNED scenarios

VC-21, VC-22 and VC-23 restate EI-5D obligations (IS-16, BD-02, BD-03) and are executable only in slice 6. They do not block this slice.

## 11. I08 mutation disposition

**I08 was not killed. It is classified as an equivalent mutant under the accepted contracts.**

- **Origin (slice 2).** The mutant replaced the adapter's equivalence test `stored == disposition` with equality of `assessment_fingerprint` alone. Without a genuine version-2 record, its effect was unobservable, and slice 2 recorded it as a survivor.
- **Slice 4 targeted mutations** of the same line, against MIX:

| Mutant | Result |
| --- | --- |
| Equivalence by generation key only | Detected (7 tests fail) |
| Equivalence by proposal identity only | Detected (6 tests fail) |
| Equivalence by assessment fingerprint only (I08) | Survived (all pass) |

- **Why I08 is equivalent.** Construction of `EquityRecommendationGenerationDisposition` (northstar-application `equity_recommendation_generation_records.py`) requires the label to pair with the assessment's format, the generation key to be the assessment's, the kind to follow the assessment's status, and the proposal document to be the one the assessment holds. Every constructible disposition is therefore determined by its assessment document, and the assessment fingerprint is SHA-256 over that canonical document, `format` included. For constructible records, equal `eira1` implies equal records, so the mutant and the original decide identically on every input the adapter can receive.
- **Proof boundary.** The equivalence holds only while the Application's pairing and construction invariants hold, and under SHA-256 collision resistance. It is not enforced by SQLite (N6 trigger deferred). A row that violates pairing, written by privileged SQL, is refused on load as Invalid (MIX `test_a_corrupted_…[relabelled to /1]`), so it never reaches the equivalence decision.
- **Negative evidence kept.** MIX `test_i08_shared_identities_never_make_records_of_two_evaluators_equivalent`: the shared key, proposal identity and proposal document decide nothing; a label swap does not construct in either direction (Invalid, not Incompatible); and the store returns `CONFLICT` and the recorder `EVALUATOR_VERSION_CONFLICT` in both orders.

## 12. Test results

### 12.1 Historically reported local results

These are results reported by the implementing slices on the Windows development machine. They are historical, not rerun for this record, and not CI results.

| Point | Result |
| --- | --- |
| Combined 3a + 3b, before the year fixes, against the corrected Infrastructure | Application 4,587 passed; Core 5,113 passed; Infrastructure 3,267 passed, 27 skipped |
| Core #40 | Core 5,122 passed; the Application boundary test passed against the local Core |
| Application `9ccfd77` | Application 4,588 passed; v1 goldens and v2 module 88 passed |
| Slice 4 | New module 26 passed; Infrastructure 3,293 passed, 27 skipped; Application focused 88 passed |

### 12.2 CI status

**NOT INDEPENDENTLY VERIFIED FROM AVAILABLE EVIDENCE.**

- No CI run, artifact or check result was available in this environment: no GitHub CLI is installed, and no CI artifact exists locally. No run identifier, URL or conclusion is recorded here.
- The one CI fact reported to this record is a **failure**: the owner reported that the combined Application PR (#95) failed CI on `test_no_boundary_precedes_the_lowest_representable_instant` before Core #40 and Application `9ccfd77`. No subsequent CI result has been inspected.
- A merged PR is not CI evidence.
- **Evidence needed** (as [EI-4 Decision Register, section 15.3](Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4)): successful `python-ci.yml` runs (`ruff check`, `ruff format --check`, `pytest` on `ubuntu-latest`) for northstar-core at `29bd34a`, northstar-application at `d908674` and northstar-infrastructure at `f71f8f7`, or accepted successors, each recorded with its run identifier, conclusion and, for Application and Infrastructure, the dependency commits checked out. Linux runs of MIX would also add POSIX evidence for PS-04, PS-05 and VC-15.

## 13. Remaining limitations

- **L1 — SQLite pairing trigger.** Deferred under N6; SQLite does not enforce label-format pairing.
- **L2 — I08.** An equivalent mutant, not killed; it relies on the Application's construction invariants ([section 11](#11-i08-mutation-disposition)).
- **L3 — Platform.** PS-04, PS-05 and VC-15 are Windows-only local evidence; no Linux CI result is recorded (as EI4-C2).
- **L4 — Scenario gaps.** VC-04 NOT VERIFIED; eight scenarios VERIFIED WITH LIMITATION ([section 10.1](#101-scenarios-verified-with-limitation)). Owner accepted 2026-11-14, with every limitation retained. Because of VC-04, rollback or downgrade to a pre-3a Application against stored evaluator-v2 records is not authorized without separate compatibility verification ([section 10.2](#102-scenario-not-verified)).
- **L5 — EI4-D8.** The conservative open-state treatment is unchanged: a later modeled issuance in the same session may resolve as `EVALUATION_UNRESOLVED`.
- **L6 — Deferred decisions.** O-2, O-3, EI3-D10, EI0-D21 and EI0-D19 stay deferred.
- **L7 — CI and EI-4 closure.** EI4-C4 is NOT VERIFIED; EI-4 formal closure stays pending.
- **L8 — Synthetic only.** Every record is synthetic; no production recommendation-history authority exists.
- **L9 — Compatibility is not issuance.** This prerequisite makes version-2 records exist and verify. It authorizes no issuance; EI-5D, its authority and its atomic history and no-backdating checks are unimplemented.

The design's limitations 1 to 9 and 11 to 14 ([section 14](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md#14-deferred-work-and-limitations)) remain as recorded. Its limitation 10 (the `/2` test label) was resolved by slices 1 and 2.

## 14. Engineering verdict and recommended status

**EI-4C VERSIONED HISTORY COMPATIBILITY ENGINEERING ACCEPTANCE: PASS WITH DOCUMENTED LIMITATIONS — synthetic engineering acceptance.**

- Every implementation slice merged in an order that met its prerequisites (section 4).
- Version-1 preservation, version-2 behaviour and SQLite compatibility are evidenced (sections 7 to 9).
- No critical invariant is blocked. One scenario (VC-04) is NOT VERIFIED and needed an owner disposition; three are EI-5D-owned.

**Recommended program status, as recorded before the owner decision: ACCEPTED WITH DOCUMENTED LIMITATIONS, subject to owner acceptance of this slice and an owner disposition of VC-04.** The owner decided on 2026-11-14 (section 14.1).

### 14.1 Owner acceptance (2026-11-14)

**OWNER APPROVED, 2026-11-14. EI-4C Versioned History Compatibility: ACCEPTED WITH DOCUMENTED LIMITATIONS.**

The owner accepts:

- the completed slices 0 to 5 compatibility implementation and engineering evidence;
- the 40 VERIFIED scenarios;
- the eight VERIFIED WITH LIMITATION scenarios, retaining every limitation (section 10.1);
- VC-04 as NOT VERIFIED, as a documented limitation (section 10.2): pre-3a Application compatibility with stored evaluator-v2 records is not guaranteed, no legacy-reader compatibility is claimed, and rollback or downgrade to a pre-3a Application against stored evaluator-v2 records is **not authorized** without separate compatibility verification;
- VC-21 to VC-23 as PLANNED for EI-5D;
- the deferred N6 SQLite pairing trigger;
- the I08 equivalent-mutant qualification (section 11);
- Windows-only evidence where Linux verification is unavailable;
- the existing EI4-D8 restrictions and the other previously deferred decisions.

**Still pending or not authorized:**

- **EI-4 formal CI closure** stays **PENDING** independently verified evidence. EI4-C4 is **NOT VERIFIED** (section 12.2).
- **This acceptance does not authorize** EI-5D implementation, MODELED issuance, ACTUAL issuance, production deployment, broker execution or new historical data writes. EI-5D requires a separate owner milestone authorization.

The acceptance changes no scenario status, evidence or test result in this record. The owner record is the [EI-5 Decision Register, section 14.4](Equity-Intraday-V1-EI5-Decision-Register.md#144-owner-acceptance-of-slice-5-2026-11-14).

## 15. Non-authorizations

Neither this record nor the owner acceptance of section 14.1 authorizes EI-5D implementation or EI-6, MODELED or ACTUAL issuance, production deployment, new historical data writes, rollback or downgrade to a pre-3a Application against stored evaluator-v2 records, `eiris1` records, declarations, an issuance authority, real or live data, broker or demo execution, API or Web exposure, a production composition root, the SQLite pairing trigger, a third evaluator, or any relaxation of EI4-D8. It does not close EI-4. ACTUAL issuance stays unauthorized.

## 16. Related documents

- [ADR-025: Equity Intraday V1 Versioned Evaluation and History Compatibility](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md)
- [EI-4C Versioned History Compatibility Design v1.0](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md)
- [EI-5 Decision Register](Equity-Intraday-V1-EI5-Decision-Register.md), sections 13 and 14
- [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md), sections 15 and 16
- [EI4-D11 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Engineering-Acceptance-Record.md)
- [EI-5 Issuance and Lifecycle Design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [ADR-021](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md), [ADR-023](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md), [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
