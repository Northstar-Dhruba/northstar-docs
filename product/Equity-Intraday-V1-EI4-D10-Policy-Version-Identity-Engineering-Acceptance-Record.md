# Equity Intraday V1 EI4-D10 Immutable Policy-Version Identity Enforcement Engineering Acceptance Record

## Status

**Engineering acceptance record, 2026-11-10.**

- **Verdict:** EI4-D10 ENGINEERING ACCEPTANCE: **PASS WITH DOCUMENTED LIMITATIONS** ([section 17](#17-engineering-verdict)).
- **What it covers:** the merged implementation of the owner-approved EI4-D10 enforcement design, slices S1 to S4, with the separate W2 reliability fix that preceded S3 ([section 6](#6-verified-implementation-inventory)).
- **CI verification: NOT INDEPENDENTLY VERIFIED FROM AVAILABLE EVIDENCE.** The results below are local executions on the development machine ([section 14](#14-test-results)).
- This record is engineering acceptance evidence. It is **not** an owner approval:
  - it approves no decision and changes no decision status;
  - it does not reopen or alter the owner approvals of EI4-D10 and EI4-D10-DQ1 to EI4-D10-DQ9 (2026-11-08);
  - it does not complete EI4-D11, whose mandatory verified-recording boundary is still not implemented ([section 10.2](#102-ei4-d11-is-separate)).
- Accepting EI4-D10 is **not** real-data readiness, production readiness, recommendation issuance, trading authorization or a security system ([section 18](#18-non-authorizations)).

## 1. Acceptance identity

| Item | Value |
| --- | --- |
| Milestone | EI4-D10 -- Immutable Policy-Version Identity, slices S1 to S5 |
| Record | S5, documentation acceptance |
| Date | 2026-11-10 |
| Governance | [EI-4 Decision Register, sections 10.2 and 11](Equity-Intraday-V1-EI4-Decision-Register.md#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08); [ADR-022](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md) |
| Design | [EI4-D10 Policy-Version Identity Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md) |
| Origin | EI-4E finding AA-1 ([EI-4E record, section 14.1](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#141-aa-1--policy-version-label-governance)) |
| Data | Synthetic only |

## 2. Scope

**In scope:**

- The approved namespace `northstar.equity-intraday-v1.synthetic`.
- The six policy types `GENERATION_POLICY`, `RISK_POLICY`, `COST_SCHEDULE`, `PRICE_INCREMENT`, `TIMING_POLICY` and `POLICY_SET`.
- The immutable identity `(namespace, policy_type, published_version_label)`.
- The component format `northstar.equity-synthetic-policy-definition/1` with fingerprint prefix `eipd1-sha256:`.
- The existing `eirs1` (composite set) and `eirk1` (generation) identities, which must stay unchanged.
- Publication, the same-file SQLite catalog, generation validation, the recording gate and historical compatibility.

**Out of scope:** EI4-D11 enforcement, invalid-record recovery, actual issuance, the lifecycle `RECORDED` state, broker or paper orders, fills, positions, P&L, live data, production policies, EI-5 and EI-6.

## 3. Governance references

| Layer | State | Where |
| --- | --- | --- |
| Owner-approved architecture | EI4-D10 (2026-11-08); EI4-D10-DQ1 to DQ9 as recommended (2026-11-08) | Decision Register sections [10.2](Equity-Intraday-V1-EI4-Decision-Register.md#102-ei4-d10----immutable-policy-version-identity-aa-1) and [11](Equity-Intraday-V1-EI4-Decision-Register.md#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08); ADR-022 |
| Implemented engineering behaviour | S1, S2, S3, S4 merged; W2 merged separately | [Section 6](#6-verified-implementation-inventory) |
| Locally validated evidence | Full Application and Infrastructure suites, at implementation and again after merge | [Section 14](#14-test-results) |
| CI verification | Not independently verified | [Section 14.3](#143-ci-status) |
| Remaining limitations | L1 to L9 | [Section 16](#16-remaining-engineering-limitations) |
| Non-authorizations | Unchanged | [Section 18](#18-non-authorizations) |

ADR-020 and ADR-021 are unchanged. The [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md) and its verdict are unchanged: EI4-D10 is a later hardening milestone, not a revision of EI-4E's evidence.

## 4. Repository baselines

The merged revisions were read from local Git history. Every repository was on `develop` with a clean working tree.

| Repository | `develop` HEAD | Role |
| --- | --- | --- |
| northstar-application | `cd2301b` | S1 and S3 |
| northstar-infrastructure | `97700b3` | S2, W2 and S4 |
| northstar-core | `bc081a2` | Unchanged by EI4-D10 |
| northstar-docs | `420483f` | This record, uncommitted |
| northstar-api | `7ad37f5` | Unchanged |
| northstar-web | `5aeb94c` | Unchanged |

Python 3.13.15 on Windows; Application and Infrastructure require `>=3.13,<3.14`. Infrastructure resolves Application and Core as editable local paths; its CI checks out Application and Core at `develop`.

## 5. Executive summary

**The defect.** Before enforcement, a synthetic policy reference could reuse a version label that already named a published definition, with different canonical content. The label and the definition could diverge: EI-4E showed `timing/1` naming two latencies, producing a second generation key and a second disposition for the same candidate (AA-1).

**What EI4-D10 introduces:**

1. Immutable canonical policy-definition bindings, fingerprinted `eipd1`, with the policy set bound by the existing `eirs1`.
2. Explicit publication through one Application use case; nothing else publishes.
3. A durable catalog in the same SQLite file as the evidence and generation records.
4. Database-enforced uniqueness, foreign keys, insert-only rows and atomic six-binding publication.
5. Catalog validation during EI-4C generation.
6. An independent catalog check of the frozen policy evidence before the EI-4D-A recorder writes a disposition.
7. A permanent-conflict classification, `POLICY_REFERENCE_CONFLICT`, distinct from outages, eligibility refusals and content conflicts.
8. Historical compatibility with no backfill and no fabricated publication evidence.

The result is enforceable policy-version integrity **for the supported synthetic Application workflow**. It is not a production identity system, a trading system or a security boundary against privileged file access ([section 16](#16-remaining-engineering-limitations), L6 and L7).

## 6. Verified implementation inventory

Source paths are relative to each repository root. Application paths are under `src/northstar_application/`; Infrastructure paths under `src/northstar_infrastructure/`.

| Slice | Repository | Merged revision (PR, merge, commit) | Capability | Source modules | Test modules | Test evidence | Limitations |
| --- | --- | --- | --- | --- | --- | --- | --- |
| S1 | application | #89, `ed98ecc`, `c92c9f6` | Binding contracts, `eipd1`, catalog port, explicit publication with bounded reconciliation, catalog-backed resolver, read-only historical audit | `application_services/equity_synthetic_policy_bindings.py`, `ports/equity_synthetic_policy_catalog_store.py`, `application_services/publish_equity_synthetic_policies.py`, `application_services/catalog_equity_synthetic_recommendation_references.py`, `application_services/audit_equity_recommendation_policy_bindings.py` | `tests/application_services/test_equity_synthetic_policy_catalog.py` | 94 cases (51 functions), passing | Contract level, in-memory test double; the publication-history port has no implementation |
| S2 | infrastructure | #55, `f7b1ab8`, `24f86ea` | Same-file SQLite catalog: additive schema, keys, foreign keys, 12 triggers, `BEGIN IMMEDIATE` publication, monotone publication sequence, typed failures | `persistence/sqlite_equity_synthetic_policy_catalog_schema.py`, `persistence/sqlite_equity_synthetic_policy_catalog.py` | `tests/test_sqlite_equity_synthetic_policy_catalog.py`, `tests/equity_synthetic_policy_catalog_fixtures.py` | 66 cases (37 functions), passing | Windows processes only; publication history not implemented |
| W2 | infrastructure | #56, `0fa10b8`, `290f8fc` | EI-4D-B cleanup never replaces a typed failure: a closed or failed connection keeps `CommitUnconfirmed` and definite non-commit. **A separate reliability fix to the EI-4D-B generation store, not part of the policy catalog** | `persistence/sqlite_equity_recommendation_generation.py` | `tests/test_sqlite_equity_recommendation_generation_reliability.py` | 22 cases (11 functions), passing | Same pattern remains in other adapters (L9) |
| S3 | application | #90, `cd2301b`, `d55f761` | Shared policy-set verifier; required catalog in generator and recorder; generation validation; recording gate; `POLICY_REFERENCE_CONFLICT` and `POLICY_SET_UNAVAILABLE` reasons; three recording outcomes | `application_services/verify_equity_synthetic_policy_bindings.py`, `application_services/generate_equity_trade_recommendation.py`, `application_services/record_equity_recommendation_generation.py`, `application_services/equity_trade_recommendation_eligibility.py` | `tests/application_services/test_equity_synthetic_policy_enforcement.py` (new), plus the updated EI-4C, EI-4D-A and S1 test modules | 40 new cases (37 functions); Application suite 4,345 passed | Invalid and incompatible share the generation reason (L4) |
| S4 | infrastructure | #57, `97700b3`, `d1adce3` | Fixtures publish their policies through S1 into the real SQLite catalog; generator and recorder take that catalog; EI-4E AA-1 expectation corrected; end-to-end acceptance | `tests/equity_recommendation_acceptance_fixtures.py`, `tests/equity_recommendation_generation_fixtures.py` (test code only; no production change) | `tests/test_equity_synthetic_policy_enforcement_acceptance.py` (new), `tests/test_equity_recommendation_generation_acceptance.py`, `tests/test_sqlite_equity_synthetic_policy_catalog.py` | 18 new cases (17 functions); Infrastructure suite 2,914 passed, 27 skipped | Outages end to end by lock contention only (L5); legacy records simulated |

## 7. Canonical identity acceptance

| Requirement | Application canonical verification (S1, S3) | SQLite enforcement (S2) |
| --- | --- | --- |
| Approved namespace | Publication and resolution accept only the approved namespace: `test_the_record_type_states_its_namespace_but_publication_is_approved_only` | Namespace CHECK constraint: `test_the_database_checks_every_binding_column` |
| Six policy types | Closed enum of six: `test_identity_is_namespace_type_and_label_never_global` | `policy_type` CHECK; set columns fixed to the five component types: `test_the_set_columns_follow_the_approved_component_types` |
| Uniqueness per namespace, type and label | Same identity, other content is a conflict: `test_same_identity_equal_content_is_equivalent_and_other_content_conflicts` | Primary keys refuse duplicates: `test_the_database_refuses_duplicate_identities`; one label under two types is two bindings: `test_one_label_under_two_types_is_two_bindings` |
| Equal content republishes idempotently | `test_first_publication_then_an_identical_one_is_idempotent` | `test_first_publication_commits_and_an_identical_one_is_equivalent` |
| Shared equivalent components | `test_a_new_set_may_reuse_published_components`, `test_different_labels_may_hold_identical_content` | `test_bundles_share_identical_components` |
| A changed definition needs a new label | `test_one_price_increment_label_binds_one_amount` | End to end: `test_e12_c_a_reused_version_label_with_changed_content_is_a_permanent_conflict` ([section 11](#11-ei-4e-aa-1-closure)) |
| Deterministic `eipd1` | Pinned digests: `test_eipd1_bindings_are_deterministic_canonical_and_pinned` | Stored rows reload exactly: `test_published_bindings_are_stored_canonically_and_reload_exactly` |
| Decimal canonicalization preserved | `test_canonical_decimals_give_one_binding` | Non-canonical stored values fail closed: `test_a_corrupted_stored_binding_fails_closed_and_is_never_repaired` |
| `eirs1` and `eirk1` unchanged | Golden values: `test_existing_eirs1_and_eirk1_are_unchanged`; `test_g1_g2_a_published_bundle_generates_with_unchanged_identities` | End to end: `test_s4_unchanged_identities_reproduce_under_explicit_publication`; EI-4E identity tests pass unchanged |

The Application rebuilds and re-verifies every binding from canonical text (`from_stored`). The SQLite layer enforces structure, uniqueness and immutability; it computes no SHA-256, as with EI-4D-B (ADR-021, section 10).

## 8. Atomic SQLite publication acceptance

| Guarantee | Evidence (`tests/test_sqlite_equity_synthetic_policy_catalog.py` unless stated) |
| --- | --- |
| Same-file authority, additive schema | `test_a_fresh_evidence_database_acquires_the_catalog_schema`, `test_initialization_is_idempotent_and_never_rebuilds`, `test_an_existing_evidence_and_generation_database_is_extended_additively` |
| Never creates or adopts a second database | `test_the_store_never_creates_a_database_or_a_second_authority` |
| Real foreign keys | `test_a_set_needs_its_publication_and_its_five_component_bindings` |
| Insert-only | `test_the_database_refuses_to_change_or_delete_any_binding` |
| Atomic six-binding publication; no partial bundle | `test_a_conflicting_component_writes_none_of_the_bundle`, `test_a_conflicting_set_writes_none_of_the_bundle`, `test_a_failure_before_commit_rolls_back_the_whole_bundle`, `test_a_real_constraint_violation_rolls_back_the_whole_bundle` |
| `BEGIN IMMEDIATE` serialization, independent connections | `test_identical_concurrent_publications_store_one_binding_each`, `test_conflicting_concurrent_publications_commit_exactly_one`, `test_concurrent_bundles_sharing_components_both_commit`, `test_a_writer_that_cannot_take_the_lock_writes_nothing`, `test_a_waiting_writer_and_a_reader_see_only_committed_bindings` |
| Windows spawned processes | `test_separate_processes_are_serialized_by_the_database` (identical and conflicting) |
| Unavailability before any transaction | `test_a_failure_before_any_transaction_is_unavailability` |
| COMMIT uncertainty, never a rollback claim | `test_a_commit_failure_is_unconfirmed_never_a_rollback_claim` |
| Bounded Application reconciliation on SQLite | `test_the_application_reconciles_an_unconfirmed_commit_on_real_sqlite`, `test_reconciliation_finds_a_competitors_conflicting_binding`, `test_reconciliation_that_cannot_read_the_catalog_is_unresolved`, `test_publication_writes_stay_within_the_application_bound` |
| Restart durability | `test_bindings_survive_restart_and_a_file_copy`; end to end `test_s4_the_published_chain_generates_records_restarts_and_replays` |
| Reads never publish | `test_reads_and_resolution_never_publish` |

No POSIX separate-process acceptance and no multi-host correctness is claimed ([L2](#l2----platform-coverage)).

### 8.1 Publication-history boundary

- The catalog's `publication_sequence` orders publications **within the catalog only**.
- No EI-4D disposition or attempt records a catalog position, so the sequence is not a shared historical ordering authority for older records.
- `EquitySyntheticPolicyPublicationHistory` remains **unimplemented**, as `test_no_publication_history_is_implemented_from_the_catalog_sequence` asserts.
- No historically verified publication claim can be made without such evidence ([section 15](#15-historical-evidence-and-audit-limitations)).

## 9. Generation enforcement acceptance

When all five components resolve through the existing reference source, `GenerateEquityTradeRecommendationUseCase` builds the resolved set and checks it with `verify_equity_synthetic_policy_set`. Generation continues only if:

- each of the five component bindings, and the `POLICY_SET` binding, is published under exactly its identity;
- each is equal in every field to the offered binding: namespace, type, label, canonical definition and `eipd1` (for the set, its reference document, five component labels and `eirs1`);
- the published bindings rebuild one consistent bundle, which re-derives `eirs1`. A valid-looking `eirs1` alone is never accepted (`test_a_set_alone_with_a_valid_eirs1_is_not_established`).

Each identity is read once, nothing is published, and nothing retries. The catalog is a required constructor argument with no default (`test_g_the_generator_requires_a_catalog`). A failure happens while gathering, so it has no inputs and no generation key: it is **attempt-only**, and no disposition is formed.

| Case | Status | Reason | Evidence (C; E) |
| --- | --- | --- | --- |
| Other content under a published label | `DEPENDENCY_UNAVAILABLE` | `POLICY_REFERENCE_CONFLICT` | `test_g5_…`; `test_s4_other_content_under_a_published_component_label_is_a_permanent_conflict` |
| A set label naming other components | `DEPENDENCY_UNAVAILABLE` | `POLICY_REFERENCE_CONFLICT` | `test_g6_…`; `test_s4_a_set_label_rebound_to_other_components_is_a_permanent_conflict` |
| The catalog answers another identity | `DEPENDENCY_UNAVAILABLE` | `POLICY_REFERENCE_CONFLICT` | `test_g7_a_catalog_answer_for_another_identity_is_a_conflict` |
| Component not published | `DEPENDENCY_UNAVAILABLE` | That component's existing `*_UNAVAILABLE` | `test_g3_…`; `test_s4_an_unpublished_component_or_set_fails_closed_and_is_never_published` |
| Set not published | `DEPENDENCY_UNAVAILABLE` | `POLICY_SET_UNAVAILABLE` | `test_g4_…`; same S4 test |
| Empty catalog | `DEPENDENCY_UNAVAILABLE` | All five `*_UNAVAILABLE` and `POLICY_SET_UNAVAILABLE` | `test_g13_an_empty_catalog_is_never_an_empty_authorized_registry`; `test_s4_an_empty_catalog_is_never_an_empty_authorized_registry` |
| Catalog outage | `DEPENDENCY_UNAVAILABLE` | The same existing `*_UNAVAILABLE` reasons | `test_g10_g11_…`; `test_s4_scenario_30_a_catalog_outage_during_generation_fails_closed` |
| Eligibility refusal | `NOT_ELIGIBLE` | `LISTING_NOT_ELIGIBLE` | `test_g14_…`; `test_s4_an_eligibility_refusal_under_published_policies_stays_not_eligible` |
| No candidate | `NO_CANDIDATE` | Catalog not read | `test_g15_…`; `test_s4_no_candidate_never_consults_the_catalog` |

Conflicts are read once and never retried (`test_g12_g13_a_conflict_is_read_once_never_retried_and_never_published`). Research, timing and Core formation are unchanged (`test_g16_research_and_formation_are_unchanged_by_the_check`).

### 9.1 Corrupt and unsupported bindings, as merged

Verified in the merged source (`generate_equity_trade_recommendation.py`, `_published`; `record_equity_recommendation_generation.py`, `_GATE`):

- **At generation**, a binding that does not verify (`INVALID`) and one in an unsupported format (`INCOMPATIBLE`) both give the reason **`POLICY_REFERENCE_CONFLICT`**, as ADR-022 section 8, case D, specifies. The subtype is visible only in the `SYNTHETIC_REFERENCES` check detail, which starts `INVALID:` or `INCOMPATIBLE:`. There is no separate generation reason code.
- **At recording**, the outcome is `POLICY_REFERENCE_CONFLICT`, and the result's `policy_verification` keeps the typed verdict, `INVALID` or `INCOMPATIBLE` (`EquitySyntheticPolicyBindingVerdict`).
- **At load**, S1 and S2 raise the typed `InvalidEquitySyntheticPolicyBindingError` or `IncompatibleEquitySyntheticPolicyBindingError`, and the S1 publication use case reports `STORED_BINDING_INVALID` or `STORED_BINDING_INCOMPATIBLE`.
- Evidence: `test_g8_g9_corrupt_and_unsupported_bindings_fail_closed_and_stay_distinct`, `test_r9_r10_invalid_and_incompatible_bindings_fail_closed_and_stay_distinct`, and end to end `test_s4_corrupt_and_unsupported_catalog_bindings_fail_closed`. Corrupt rows are never repaired.

This meets the DQ6 clarification ([Decision Register, section 11.2](Equity-Intraday-V1-EI4-Decision-Register.md#112-ei4-d10-dq6-clarification-a-permanent-conflict)) through typed errors, verdicts and diagnostics, but not as a generation reason code ([L4](#l4----failure-subtype-visibility)).

## 10. Recorder pre-write enforcement acceptance

`RecordEquityRecommendationGenerationUseCase.record` checks the disposition's frozen policy set against the catalog **before** source verification and before any store call. Because the gate sits in `record()` itself, direct `record()` callers pass through it too. A refusal writes nothing, reports `attempt_recorded=False` and carries the verification in `policy_verification`.

| Outcome | When | Evidence (C; E) |
| --- | --- | --- |
| `POLICY_REFERENCE_CONFLICT` | A label is published with other content, or its binding is invalid or unsupported | `test_r3_r4_r7_a_forged_policy_definition_is_refused_before_any_write`, `test_r5_component_labels_published_but_the_set_bound_to_others_is_a_conflict`; `test_s4_the_recorder_refuses_a_forged_disposition_before_any_write` |
| `POLICY_REFERENCE_NOT_ESTABLISHED` | A label is not published | `test_r6_r11_an_unpublished_bundle_is_not_established_and_never_published`, `test_b_a_published_definition_under_another_label_is_refused`; `test_s4_the_recorder_refuses_unpublished_policy_evidence_before_any_write` |
| `POLICY_REFERENCE_UNAVAILABLE` | The catalog cannot be read | `test_r8_a_catalog_outage_is_policy_reference_unavailable`; `test_s4_scenario_31_a_catalog_outage_at_the_recording_gate_writes_nothing` |

**Also verified:**
- A valid `eirs1` without published components is refused: `test_b_a_valid_eirs1_without_published_components_is_refused`.
- Zero writes and no claimed attempt after a failed gate: `test_r12_r13_a_failed_gate_writes_nothing_and_claims_no_attempt`.
- A missing catalog is a constructor error: `test_b_the_recorder_requires_a_catalog`.
- Existing behaviour after the gate passes is unchanged:
  - content conflicts: `test_r15_a_content_conflict_under_published_policies_is_unchanged`;
  - W1 invalid and incompatible stored-record mappings: `test_r16_w1_stored_record_mappings_are_unchanged_after_the_gate`;
  - bounded COMMIT reconciliation, with the gate run once: `test_r17_commit_uncertainty_stays_bounded_and_the_gate_runs_once`.
- Equivalent records stay immutable and are never certified: `test_r14_an_equivalent_record_stays_immutable_and_is_never_certified`.

### 10.1 Attempt-only results

An attempt-only audit carries no policy content, so the gate does not apply. It is recorded through the existing path, and `attempt_recorded` reflects the real write: `test_r18_attempt_only_results_record_as_before_without_the_gate`, and end to end `test_s4_a_policy_failure_is_an_attempt_only_record_with_a_machine_readable_reason`.

- The stored row's status is `DEPENDENCY_UNAVAILABLE`, and its JSON reasons include `POLICY_REFERENCE_CONFLICT`.
- A duplicate is `EQUIVALENT_COMMITTED`.
- The attempts table's status CHECK still lists exactly the six EI4-D9 statuses: no new persisted status exists.

### 10.2 EI4-D11 is separate

The gate enforces the invariant on the **verified Application recording path**. It does not make a direct write through the EI-4D-B adapter, or through SQLite, impossible. EI4-D11's mandatory operational write boundary is approved as governance but **not implemented or accepted** ([Decision Register, section 10.3](Equity-Intraday-V1-EI4-Decision-Register.md#103-ei4-d11----mandatory-verified-recording-boundary-and-invalid-record-recovery-governance-aa-2)). S3 and S4 do not complete it.

## 11. EI-4E AA-1 closure

**The original failure.** EI-4E's `test_e12_c_a_reused_version_label_with_changed_content_is_another_identity` showed that `timing/1` could be answered with a 240 s latency instead of 150 s. That produced a second `eirs1`, a second `eirk1` and a second COMMITTED disposition for the same candidate.

**After S4**, the same test is named `test_e12_c_a_reused_version_label_with_changed_content_is_a_permanent_conflict` (in `tests/test_equity_recommendation_generation_acceptance.py`, on real SQLite). It keeps the attempt itself as setup, and observes the rejected path:

1. The changed `timing/1` content is offered for explicit publication through S1.
2. Publication reports `POLICY_REFERENCE_CONFLICT`.
3. `writes == 1`: one transaction, with no retry.
4. Every catalog row is unchanged; the original binding stays.
5. Generation reports `DEPENDENCY_UNAVAILABLE` with exactly `(POLICY_REFERENCE_CONFLICT,)`, with no generation key and no inputs.
6. No new disposition: the disposition rows are byte-identical.
7. Recording stores only the attempt-only audit, under the existing status scheme: counts are 1 disposition and 2 attempts.
8. The original disposition remains immutable.

**The legitimate path**, observed separately in the same test:

9. The new content is published under new labels (`timing/2` in set `set/3`), and generation proposes under a new, different generation key.

Further end-to-end evidence:
- `test_s4_scenario_33_a_permanent_conflict_is_never_retried_or_republished`;
- `test_s4_other_content_under_a_published_component_label_is_a_permanent_conflict`.

**AA-1 is closed for the verified Application workflow on the same-file catalog**, within limitations L6 and L7.

## 12. Full synthetic end-to-end acceptance

`test_s4_the_published_chain_generates_records_restarts_and_replays` (in `tests/test_equity_synthetic_policy_enforcement_acceptance.py`) runs, on one disposable SQLite file:

EI-2C ingestion → EI-2B snapshot → EI-3A evaluation → EI-3C first CANDIDATE → explicit S1 publication into the S2 catalog → EI-4C generation with catalog verification → EI-4D-A recording gate → EI-4D-B immutable persistence → SQLite restart → frozen historical replay.

- **Real SQLite, same file.** The fixtures construct `SQLiteEquitySyntheticPolicyCatalogStore` on the evidence database and publish 8 synthetic policy sets through `PublishEquitySyntheticPoliciesUseCase` before generating.
- **Reads never publish.** Generation reads 6 bindings and recording reads 6, with 0 publications (counting subclasses of the real adapter).
- **Deterministic identities.** After restart, generation reproduces the same `eirk1` and assessment fingerprint.
- **Durable after restart.** The file is renamed (which Windows refuses while any connection is open). Fresh adapters reload every binding exactly, and re-recording is `EQUIVALENT_COMMITTED`.
- **Replay without the catalog.** With every catalog read made to raise, the stored record still verifies, and the frozen assessment reproduces.

Everything is synthetic: synthetic candles, listings (TCS, INFY, WIPRO as labelled fixtures), eligibility and issued history. No real NSE data, provider, broker or market order is involved.

## 13. Acceptance evidence matrix

Required layers follow the design's [acceptance matrix](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md#13-acceptance-matrix):

- **C:** Application contract tests (S1: `test_equity_synthetic_policy_catalog.py`; S3: `test_equity_synthetic_policy_enforcement.py`).
- **S:** Infrastructure SQLite tests (S2: `test_sqlite_equity_synthetic_policy_catalog.py`).
- **E:** end-to-end tests on real SQLite (S4: `test_equity_synthetic_policy_enforcement_acceptance.py`; EI-4E: `test_equity_recommendation_generation_acceptance*.py`).

Test names are abbreviated with `…` where the full name appears in an earlier section. Every observed result is a pass in the post-merge run ([section 14.2](#142-post-merge-validation-2026-11-10)).

| # | Required behaviour | Layers | Slices | Supporting tests | Limitations | Determination |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | First publication; rows verified on reload | C, S | S1, S2 | C `test_first_publication_then_an_identical_one_is_idempotent`; S `test_first_publication_commits_and_an_identical_one_is_equivalent`, `test_published_bindings_are_stored_canonically_and_reload_exactly` | — | VERIFIED |
| 2 | Identical re-publication is EQUIVALENT; no row | C, S | S1, S2 | As row 1 | — | VERIFIED |
| 3 | Same identity, other content: CONFLICT; nothing written | C, S, E | S1, S2, S4 | C `test_same_identity_equal_content_is_equivalent_and_other_content_conflicts`, `test_a_component_conflict_is_permanent_never_retried_and_writes_nothing`; S `test_a_conflicting_component_writes_none_of_the_bundle`, `test_a_conflicting_set_writes_none_of_the_bundle`; E `test_e12_c_…_permanent_conflict`, `test_s4_other_content_under_…`, `test_s4_a_set_label_rebound_…` | — | VERIFIED |
| 4 | Another namespace refused | C, S | S1, S2 | C `test_the_record_type_states_its_namespace_but_publication_is_approved_only`; S `test_the_database_checks_every_binding_column` (namespace) | — | VERIFIED |
| 5 | One label under two types: two bindings | C, S | S1, S2 | C `test_identity_is_namespace_type_and_label_never_global`; S `test_one_label_under_two_types_is_two_bindings` | — | VERIFIED |
| 6 | Different label, different content: new binding | C, S | S1, S2 | C `test_a_new_set_may_reuse_published_components`, `test_different_labels_may_hold_identical_content`; S `test_bundles_share_identical_components` | — | VERIFIED |
| 7 | Decimal `1.50` and `1.5`: one binding | C | S1 | C `test_canonical_decimals_give_one_binding` | Contract level only, as the matrix requires | VERIFIED |
| 8 | Whitespace, case or prefix variants | C, S | S1, S2 | C `test_invalid_labels_are_refused_and_never_rewritten`, `test_labels_are_case_sensitive`; S `test_the_database_checks_every_binding_column` (label cases) | — | VERIFIED |
| 9 | Unpublished label at generation; attempt-only | C, E | S3, S4 | C `test_g3_…`, `test_g4_…`, `test_g13_an_empty_catalog_…`; E `test_s4_an_unpublished_component_or_set_fails_closed_and_is_never_published`, `test_s4_an_empty_catalog_…` | — | VERIFIED |
| 10 | Unpublished label at recording: NOT_ESTABLISHED; nothing written | C, E | S3, S4 | C `test_r6_r11_…`, `test_b_a_published_definition_under_another_label_is_refused`; E `test_s4_the_recorder_refuses_unpublished_policy_evidence_before_any_write`, `test_s4_pre_enforcement_records_replay_and_audit_without_backfill` | — | VERIFIED |
| 11 | Conflicting content at recording (AA-1) | C, E | S3, S4 | C `test_r3_r4_r7_…`, `test_r5_…`; E `test_e12_c_…`, `test_s4_the_recorder_refuses_a_forged_disposition_before_any_write` | — | VERIFIED |
| 12 | Unsupported binding version: Incompatible at load; conflict at generation | C, S | S1, S2, S3 | C `test_a_stored_binding_that_does_not_verify_fails_closed` (unsupported version), `test_g8_g9_…`, `test_r9_r10_…`; S `test_a_corrupted_stored_binding_fails_closed_and_is_never_repaired` (unsupported format); also E `test_s4_corrupt_and_unsupported_catalog_bindings_fail_closed` | Subtype is not a generation reason (L4) | VERIFIED |
| 13 | Corrupted stored binding: invalid; row unchanged | S, E | S2, S4 | S `test_a_corrupted_stored_binding_fails_closed_and_is_never_repaired`, `test_an_inconsistent_stored_set_is_invalid_evidence`; E `test_s4_corrupt_and_unsupported_catalog_bindings_fail_closed` (fingerprint mismatch) | Corruption by the test-only `bypass()` | VERIFIED |
| 14 | Restart: bindings load and verify | S, E | S2, S4 | S `test_bindings_survive_restart_and_a_file_copy`; E `test_s4_the_published_chain_generates_records_restarts_and_replays` | — | VERIFIED |
| 15 | Independent-connection concurrency | S | S2 | S `test_identical_concurrent_publications_store_one_binding_each`, `test_conflicting_concurrent_publications_commit_exactly_one`, `test_concurrent_bundles_sharing_components_both_commit`, `test_a_writer_that_cannot_take_the_lock_writes_nothing`, `test_a_waiting_writer_and_a_reader_see_only_committed_bindings`; also EI-4E `test_e19_two_writers_of_equivalent_content_store_it_once`, `test_e19_two_conflicting_writers_leave_one_authoritative_disposition` (recording through the gate) | Single host | VERIFIED |
| 16 | Separate spawned processes on Windows | S | S2 | S `test_separate_processes_are_serialized_by_the_database`; also EI-4E `test_e19_separate_os_processes_are_serialized_by_the_database` (spawned workers record through the gate) | Windows only; POSIX not tested (L2) | VERIFIED (Windows) |
| 17 | Lost COMMIT acknowledgment reconciled | C, S | S1, S2 | C `test_a_lost_acknowledgment_reconciles_as_equivalent_by_reload`; S `test_a_commit_failure_is_unconfirmed_never_a_rollback_claim`, `test_the_application_reconciles_an_unconfirmed_commit_on_real_sqlite` | — | VERIFIED |
| 18 | COMMIT not made: bounded retry, at most 3 writes | C, S | S1, S2 | C `test_a_verified_absent_set_is_written_again_within_the_bound`, `test_publication_writes_never_exceed_the_bound`; S `test_publication_writes_stay_within_the_application_bound` | — | VERIFIED |
| 19 | Reload unavailable after an uncertain COMMIT | C, S | S1, S2 | C `test_a_failed_reload_leaves_the_commit_unresolved`; S `test_reconciliation_that_cannot_read_the_catalog_is_unresolved` | — | VERIFIED |
| 20 | Partial bundle: nothing written | S | S2 | S `test_a_failure_before_commit_rolls_back_the_whole_bundle`, `test_a_real_constraint_violation_rolls_back_the_whole_bundle`, `test_a_set_needs_its_publication_and_its_five_component_bindings` | Pre-commit failures by test seams and a real CHECK violation | VERIFIED |
| 21 | Historical replay with the catalog unavailable | C, E | S1, S3, S4 | C `test_historical_replay_stays_independent_of_the_catalog`, `test_h_frozen_records_replay_and_verify_with_the_catalog_down`; E `test_s4_the_published_chain_…` (every catalog read raises), `test_s4_pre_enforcement_records_replay_and_audit_without_backfill` | — | VERIFIED |
| 22 | Pre-enforcement records: matching, unbound, reused label | C, E | S1, S4 | C `test_an_unpublished_policy_is_binding_not_established`, `test_a_matching_binding_today_is_never_historical_proof`, `test_the_catalog_binding_other_content_is_a_historical_conflict`, `test_one_label_with_two_contents_across_records_is_a_historical_conflict`; E `test_s4_pre_enforcement_records_replay_and_audit_without_backfill` | Legacy records simulated with a test-only catalog in another file ([section 15](#15-historical-evidence-and-audit-limitations)) | VERIFIED (simulated) |
| 23 | Existing EI-4E and EI-4D-B suites pass, with only the listed S4 changes | S, E | S2, S4 | S `test_an_existing_evidence_and_generation_database_is_extended_additively`; E: EI-4E acceptance (38) and faults (34), EI-4D-B (73) pass. Expectation changes: `test_e12_c_…` (renamed, now the conflict), `test_e12_a_b_f_…` (distinct set labels), `test_e13_a_signal_after_the_policy_cutoff_is_refused` (publishes its new set) | — | VERIFIED |
| 24 | `eirs1`, `eirk1` and formats unchanged | C | S1, S3 | C `test_existing_eirs1_and_eirk1_are_unchanged`, `test_eipd1_bindings_are_deterministic_canonical_and_pinned`, `test_g1_g2_…`; also E `test_s4_unchanged_identities_reproduce_under_explicit_publication` | — | VERIFIED |
| 25 | Existing dispositions byte-identical | S, E | S2, S4 | S `test_an_existing_evidence_and_generation_database_is_extended_additively`; E `test_s4_pre_enforcement_records_replay_and_audit_without_backfill`, `test_e12_c_…` | — | VERIFIED |
| 26 | No issuance or history writes | E | S4 | E `test_e14_recording_never_writes_history_and_never_blocks_its_own_retry` (EI-4E, now recording through the gate); also `test_am_recording_never_writes_or_fabricates_issued_history` (EI-4D-B) | Covers recording; no issuance path exists | VERIFIED |
| 27 | Valid policy, no candidate: catalog not consulted | C | S3 | C `test_g15_no_candidate_never_consults_the_catalog`; also E `test_s4_no_candidate_never_consults_the_catalog` | — | VERIFIED |
| 28 | Eligibility refusal stays NOT_ELIGIBLE | C, E | S3, S4 | C `test_g14_…`; E `test_s4_an_eligibility_refusal_under_published_policies_stays_not_eligible` | — | VERIFIED |
| 29 | `integrity_check` and `foreign_key_check` clean | S | S2 | S `check_database` in 8 S2 functions: `test_a_fresh_evidence_database_acquires_the_catalog_schema`, `test_published_bindings_are_stored_canonically_and_reload_exactly`, `test_reads_and_resolution_never_publish`, `test_a_waiting_writer_and_a_reader_see_only_committed_bindings`, `test_separate_processes_are_serialized_by_the_database`, `test_a_failure_before_commit_rolls_back_the_whole_bundle`, `test_bindings_survive_restart_and_a_file_copy`, `test_an_existing_evidence_and_generation_database_is_extended_additively`; and 2 S4 functions | Asserted in 8 of 37 S2 functions, not in each S test as the matrix proposed | VERIFIED WITH LIMITATION |
| 30 | Catalog outage during generation | C, E | S3, S4 | C `test_g10_g11_a_catalog_outage_and_a_conflict_have_distinct_reasons`; S1 `test_an_unavailable_catalog_is_the_existing_outage_error`; E `test_s4_scenario_30_a_catalog_outage_during_generation_fails_closed` | E outage is SQLite lock contention (L5) | VERIFIED |
| 31 | Catalog outage at the recording gate | C, E | S3, S4 | C `test_r8_a_catalog_outage_is_policy_reference_unavailable`, `test_r12_r13_…`; E `test_s4_scenario_31_a_catalog_outage_at_the_recording_gate_writes_nothing` | As row 30 | VERIFIED |
| 32 | Permanent conflict versus outage, by value | C, E | S3, S4 | C `test_g10_g11_…`; E `test_s4_scenario_32_a_permanent_conflict_and_an_outage_are_told_apart_by_value` | Outage and unpublished share generation reasons (by design, ADR-022 section 8) | VERIFIED |
| 33 | Permanent conflict never retried or republished | C, S, E | S1, S2, S3, S4 | C `test_a_component_conflict_is_permanent_never_retried_and_writes_nothing`, `test_g12_g13_…`; S `test_a_conflicting_component_writes_none_of_the_bundle`, `test_conflicting_concurrent_publications_commit_exactly_one`; E `test_s4_scenario_33_a_permanent_conflict_is_never_retried_or_republished`, `test_e12_c_…` | — | VERIFIED |

**Summary.** 31 scenarios are VERIFIED. Scenario 16 is VERIFIED on Windows only, and scenario 22 with simulated legacy records. Scenario 29 is VERIFIED WITH LIMITATION: integrity checks are asserted in a subset of S tests. No scenario lacks its required layer of evidence.

### 13.1 Scenarios 30 to 33

- **Scenarios 30 and 31.** The end-to-end outages are **real SQLite lock contention**: another connection holds `BEGIN EXCLUSIVE` while the catalog is read, with the adapter's busy timeout shortened to 0.05 s through its module seam.
  - For scenario 30, the lock is taken after the five components resolve, so only the catalog meets it.
  - For scenario 31, the gate refuses before the generation store is ever opened.
- **Missing catalog authority.** A missing database file, or one without the evidence tables, was tested at the adapter layer: `test_the_store_never_creates_a_database_or_a_second_authority` and `test_a_failure_before_any_transaction_is_unavailability`. It was not tested through the full chain.
- **Scenario 32.** The conflict and the outage share the status `DEPENDENCY_UNAVAILABLE` but differ in reason and attempt identity at generation, and in outcome at recording. Neither is ever `NOT_ELIGIBLE`, `EVALUATION_UNRESOLVED`, `CONTENT_CONFLICT` or `NOT_COMMITTED`.
- **Scenario 33.**
  - Publication refuses after one transaction.
  - Generation reads each identity once and publishes nothing.
  - The gate reads 6 bindings and writes nothing.
  - The binding rows are byte-identical afterwards.

These scenarios test the EI4-D10 gate inside the EI-4D-A recorder. They do not test EI4-D11's requirement that every operational write pass through that recorder.

### 13.2 Coverage boundaries

- Decimal canonicalization is covered by Application contract tests only (row 7).
- POSIX separate-process concurrency and multi-host behaviour are not established (row 16).
- Legacy pre-enforcement records are simulated in synthetic tests (row 22).
- No live-data acceptance exists for any scenario.

## 14. Test results

Focused counts are subsets of the full-suite totals. They are listed for traceability and never added to them.

### 14.1 Historically reported implementation results

| Run | Result | Includes |
| --- | --- | --- |
| Application, at S3 implementation | 4,345 passed, 0 failed | 40 new S3 cases |
| Infrastructure, at S4 implementation | 2,914 passed, 27 skipped, 0 failed | 18 new S4 cases |
| Infrastructure, S3 applied before S4 | 2,737 passed, 27 skipped, 158 failed, 1 error | All 159 were the expected constructor change (missing `policy_catalog`; positional `max_writes`). All passed after S4 |
| S1 contract tests, at S1 review | 94 passed | — |
| S2 SQLite catalog tests | 66 passed | — |
| W2 reliability tests | 22 passed | — |

The 27 skips are 23 opt-in live Databento tests and 4 POSIX-only `tzset` tests. None relates to EI4-D10.

### 14.2 Post-merge validation (2026-11-10)

For this record, the suites were run again, read-only, against the merged `develop` branches (Application `cd2301b`, Infrastructure `97700b3`). Infrastructure imports Application `develop`, which contains S3.

| Run | Result |
| --- | --- |
| Application, full suite | 4,345 passed, 0 failed |
| Application, S1 contract module | 94 passed |
| Application, S3 enforcement module | 40 passed |
| Application, EI-4C generator module | 92 passed |
| Application, EI-4D-A recorder module | 86 passed |
| Infrastructure, full suite | 2,914 passed, 27 skipped, 0 failed (the same 27 skips as in [section 14.1](#141-historically-reported-implementation-results)) |
| Infrastructure, S2 catalog module | 66 passed |
| Infrastructure, W2 reliability module | 22 passed |
| Infrastructure, S4 acceptance module | 18 passed |
| Infrastructure, EI-4E acceptance module | 38 passed |
| Infrastructure, EI-4E faults module | 34 passed |
| Infrastructure, EI-4D-B module | 73 passed |

No implementation, test, schema or fixture file was changed by the run; both working trees stayed clean.

### 14.3 CI status

**CI verification: NOT INDEPENDENTLY VERIFIED FROM AVAILABLE EVIDENCE.**

- No CI run result was available to this review.
- Infrastructure CI (`.github/workflows/python-ci.yml`) checks out Application and Core at `develop`, so it now builds S4 against S3.
- The local results above are evidence of local executions only.

## 15. Historical evidence and audit limitations

- **No automatic backfill** and **no fabricated publication events**. No disposition, attempt, `eirs1`, `eirk1` or canonical document is rewritten.
- **Replay is catalog-free.** Replay of frozen inputs never reads the catalog, and the read-only historical binding audit is separate from it.
- **Audit classifications.**
  - A binding that matches today is **not** historical publication proof: the audit reports `BINDING_NOT_ESTABLISHED` where publication-time evidence is missing.
  - It reports `HISTORICAL_POLICY_VERSION_CONFLICT` where contradictory label reuse is demonstrable.
  - `HISTORICALLY_VERIFIED_BINDING` is **not reachable** from real stored evidence, because no publication-history implementation exists ([section 8.1](#81-publication-history-boundary)).
  - Even a record that passed the S3 gate audits as `BINDING_NOT_ESTABLISHED`: `test_h_a_gated_recording_is_still_not_historical_publication_proof`.
- **Re-recording fails closed.** Re-recording a record whose bindings are not established, or conflict, writes nothing: `test_r14_…` and `test_s4_pre_enforcement_records_replay_and_audit_without_backfill`.

**Test-only pre-enforcement simulation.** S4 builds legacy-like records through the ordinary gated chain against `pre_enforcement_catalog` or `scratch_catalog`: a real SQLite catalog in **another disposable file**, labelled test-only in `tests/equity_recommendation_acceptance_fixtures.py`. This is a controlled simulation of a time before any catalog existed. It is not an authorized second catalog authority (DQ4 is unchanged), and it is not historical publication evidence.

## 16. Remaining engineering limitations

### L1 -- Synthetic-only acceptance

No real market data, real stock or provider, actual issuance, broker or trading acceptance exists. Every policy is synthetic and approved for nothing.

### L2 -- Platform coverage

Windows separate-process tests passed: S2 publication and EI-4E e19 recording. POSIX separate-process behaviour and multi-host behaviour are **not established**.

### L3 -- Historical publication proof

No shared ordering authority connects older EI-4D records to catalog publication history. No automatic retrospective compliance claim is permitted ([section 15](#15-historical-evidence-and-audit-limitations)).

### L4 -- Failure-subtype visibility

Invalid versus incompatible bindings are:
- typed at load (S1, S2);
- typed in the recording verification (`policy_verification`);
- visible in the generation check detail.

They are **not** independent generation reason codes. Both are `POLICY_REFERENCE_CONFLICT`, per ADR-022 section 8, case D ([section 9.1](#91-corrupt-and-unsupported-bindings-as-merged)).

### L5 -- Catalog outage coverage

End-to-end outages were exercised through SQLite lock contention. A missing database file or missing evidence tables have adapter-level evidence only ([section 13.1](#131-scenarios-30-to-33)).

### L6 -- File-level trust

SQLite constraints and triggers protect ordinary supported writes. Privileged local file or database access can bypass application-level controls: for example, by inserting a binding directly or dropping a trigger, as the test-only `bypass()` shows. This is the owner-approved DQ9 limitation, for personal-use synthetic research only ([ADR-022, section 10](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md#10-publication-trust-boundary-dq9)).

### L7 -- EI4-D11 is separate

Direct privileged writes through the EI-4D-B generation adapter remain possible. The mandatory operational write boundary is not established ([section 10.2](#102-ei4-d11-is-separate)).

### L8 -- Invalid-record recovery

Recovery remains unapproved and unimplemented. Invalid evidence stays fail-closed and preserved.

### L9 -- Additional SQLite cleanup findings

The W2 review reported the same cleanup-masking pattern outside EI-4D-B. On `develop` (`97700b3`) these remain, read from source:
- `persistence/sqlite_equity_intraday_research_timeline.py`: `_rollback` reads `in_transaction` outside its exception guard.
- `persistence/sqlite_equity_candle_evidence.py`: `_write` calls `in_transaction` and `rollback()` unguarded in its exception handlers.
- `persistence/sqlite_futures_trade_recommendations.py`: the same pattern.
- `persistence/sqlite_equity_synthetic_policy_catalog.py`: guards its rollback, but closes in a plain `finally`, a latent close-masking risk.

These are **separate reliability follow-ups**. EI4-D10 neither changes nor resolves them.

## 17. Engineering verdict

**EI4-D10 ENGINEERING ACCEPTANCE: PASS WITH DOCUMENTED LIMITATIONS**

The required evidence is present: S1 to S4 merged, with every one of the 33 scenarios traced to evidence of its required layer, and the full suites passing locally. The supported synthetic research-generation-recording path now enforces immutable policy-version bindings through:
- the verified Application workflow: explicit publication, generation validation and the recording gate;
- the same-file SQLite catalog.

Limitations L1 to L9 apply.

It does **not** establish:

- EI4-D11 mandatory operational write-boundary enforcement;
- invalid-record recovery;
- historical publication compliance for legacy evidence;
- production-grade or tamper-proof publication;
- live-market readiness or broker trading readiness;
- actual recommendation issuance, positions, fills or P&L;
- EI-5 or EI-6 authorization.

## 18. Non-authorizations

This record authorizes nothing. In particular it does not authorize:

- historical evidence mutation, backfill or retrospective approval;
- changes to `eirs1`, `eirk1` or any canonical document version;
- invalid-record recovery;
- direct-adapter operational recording;
- actual recommendation issuance or the lifecycle `RECORDED` state;
- broker or paper orders, fills or positions;
- realized P&L, live data integrations, data-use licensing claims or production trading;
- any production policy value or default;
- EI-5 or EI-6.

Every restriction of the [EI-4 Decision Register, section 7](Equity-Intraday-V1-EI4-Decision-Register.md#7-production-readiness-restrictions) still applies.

## 19. Pending governance decisions

These are unchanged and **not** resolved by this record:

- EI4-D5;
- EI0-D11;
- the same-listing exclusivity threshold;
- EI0-D19;
- EI0-D21;
- EI3-D10;
- the EI4-D11 recovery mechanism (not approved);
- EI4-D11's operational-boundary implementation (governance approved; implementation pending).

## 20. Related documents

- [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md), sections 10 to 12
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md)
- [EI4-D10 Policy-Version Identity Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md)
- [Equity Intraday V1 EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
