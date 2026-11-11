# Equity Intraday V1 EI4-D11 Mandatory Operational Generation-Write Boundary Engineering Acceptance Record

## Status

**Engineering acceptance record, 2026-11-11.**

- **Verdict:** EI4-D11 ENGINEERING ACCEPTANCE: **PASS WITH DOCUMENTED LIMITATIONS -- synthetic engineering acceptance** ([section 17](#17-engineering-verdict)).
- **What it covers:** the merged implementation of the owner-approved EI4-D11 enforcement design, slices W1, W2-I, W3 and W4-I ([section 6](#6-verified-implementation-inventory)). This record is slice W5.
- **Acceptance matrix:** all 33 scenarios are traced to evidence: **28 VERIFIED, 5 VERIFIED WITH LIMITATION, 0 NOT VERIFIED** ([section 10](#10-acceptance-evidence-matrix)).
- **CI verification: NOT INDEPENDENTLY VERIFIED FROM AVAILABLE EVIDENCE.** The results below are local executions on the development machine ([section 11.3](#113-ci-status)).
- This record is engineering acceptance evidence. It is **not** an owner approval:
  - it approves no decision and changes no decision status;
  - it does not reopen or alter the owner approvals of EI4-D11-A to EI4-D11-E (2026-11-08) or of EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8 and DQ10 with clarifications A and B (2026-11-10);
  - it approves no invalid-record recovery (EI4-D11-D).
- Accepting EI4-D11 is **not** real-data readiness, production readiness, recommendation issuance, trading authorization or a tamper-proof security system ([sections 12](#12-trust-limitations-and-excluded-threat-models) and [18](#18-non-authorizations)).

## 1. Acceptance identity

| Item | Value |
| --- | --- |
| Milestone | EI4-D11 -- Mandatory Verified Recording Boundary, enforcement slices W1 to W5 |
| Record | W5, documentation acceptance |
| Date | 2026-11-11 |
| Governance | [EI-4 Decision Register, sections 10.3 and 13](Equity-Intraday-V1-EI4-Decision-Register.md#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10); [ADR-023](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md) |
| Design | [EI4-D11 Operational Generation-Write Boundary Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md) |
| Origin | EI-4E finding AA-2 ([EI-4E record, section 14.2](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#142-aa-2--verified-writer-boundary-and-recovery)) |
| Data | Synthetic only |
| Platform | Windows (EI4D11-DQ8) |

## 2. Scope

**In scope:**

- The operational write path of synthetic equity recommendation-generation records: EI-4D dispositions with their proposals, and logical attempts.
- The Application read and authorized-write ports, the recorder-issued verified-write authorizations and their public validators.
- The SQLite generation adapter `SQLiteEquityRecommendationGenerationStore` and its pre-connection refusal.
- Removal of the legacy raw write interfaces from the port (W3) and the adapter (W4-I).
- Architecture regression scans in Application and Infrastructure.
- The existing canonical identities `eira1`, `eirp1`, `eirat1`, `eirs1` and `eirk1`, which must stay unchanged.

**Out of scope:** invalid-record recovery, any production composition root, an equity recommendation-generation API or CLI, actual issuance, the lifecycle `RECORDED` state, broker or paper orders, fills, positions, P&L, live or real market data, production trading, the futures, options and paper-trading stores, EI-5 and EI-6 ([section 14](#14-features-outside-ei4-d11)).

## 3. Governance references

| Layer | State | Where |
| --- | --- | --- |
| Owner-approved governance | EI4-D11-A to EI4-D11-E (2026-11-08) | [Decision Register, section 10.3](Equity-Intraday-V1-EI4-Decision-Register.md#103-ei4-d11----mandatory-verified-recording-boundary-and-invalid-record-recovery-governance-aa-2) |
| Owner-approved enforcement design | EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8, DQ10; clarifications A and B (2026-11-10) | [Decision Register, section 13](Equity-Intraday-V1-EI4-Decision-Register.md#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10); ADR-023 |
| Implemented engineering behaviour | W1, W2-I, W3, W4-I merged | [Section 6](#6-verified-implementation-inventory) |
| Locally validated evidence | Full Application and Infrastructure suites, at each slice and again after the final merge | [Section 11](#11-test-results) |
| CI verification | Not independently verified | [Section 11.3](#113-ci-status) |
| Remaining limitations | L1 to L10 | [Section 13](#13-remaining-engineering-limitations) |
| Non-authorizations | Unchanged | [Section 18](#18-non-authorizations) |

ADR-020, ADR-021 and ADR-022 are unchanged. The [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md) and the [EI4-D10 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md), and their verdicts, are unchanged: EI4-D11 is a later hardening milestone, not a revision of their evidence.

## 4. Repository baselines

The merged revisions were read from local Git history on 2026-11-11. Every repository was on `develop` with a clean working tree, and each local `develop` matched its last-fetched `origin/develop`. No fetch was run for this record, so the remote-tracking references are as fresh as the last fetch.

| Repository | `develop` HEAD | Role |
| --- | --- | --- |
| northstar-application | `8937f12` | W1 and W3 |
| northstar-infrastructure | `6282cae` | W2-I and W4-I |
| northstar-core | `bc081a2` | Unchanged by EI4-D11 |
| northstar-docs | `5ac3bb7` | ADR-023 and the design; this record, uncommitted |
| northstar-api | `7ad37f5` | Unchanged |
| northstar-web | `5aeb94c` | Unchanged |

Python 3.13.15 on Windows; Application and Infrastructure require `>=3.13,<3.14`. Infrastructure resolves Application and Core as editable local paths; its CI checks out Application and Core at `develop`.

## 5. Executive summary

**The defect.** EI-4E finding AA-2 showed that a direct call of the SQLite generation adapter's `record_generation` committed a forged disposition that was structurally valid but historically contradictory, while the verified recorder refused the same record. Because `eirk1` does not cover assessment content and rows are immutable, the forged row occupied the genuine generation key, and the genuine record was then refused as `STORED_RECORD_INVALID`.

**What EI4-D11 introduces:**

1. A read port and an authorized-write port in Application; read-only consumers hold only the reader.
2. Recorder-issued verified-write authorizations, bound to the exact persisted content, issued only after the EI4-D10 gate and full source, frozen-evidence and replay verification pass.
3. A separate attempt-only authorization, needing no policy evidence.
4. Application-owned public validators that the adapter calls **before any SQLite connection opens**.
5. A private snapshot of the validated canonical documents, so a caller mutating its decoded objects after validation cannot change committed evidence.
6. Removal of the raw `record_generation` and `record_attempt` from the Application port (W3) and from the adapter (W4-I).
7. Architecture regression scans in both repositories, with mutation checks showing they detect realistic regressions.

The result is a mandatory verified-recording boundary **for supported and ordinary operational writes of synthetic generation records**. It is not tamper-proof against privileged code, direct SQL or file access ([section 12](#12-trust-limitations-and-excluded-threat-models)), and no production code path writes generation records yet ([section 7.11](#711-no-production-composition-root)).

## 6. Verified implementation inventory

Source paths are relative to each repository root. Application paths are under `src/northstar_application/`; Infrastructure paths under `src/northstar_infrastructure/`.

| Slice | Repository | Merged revision (PR, merge, commit) | Capability | Source modules | Test modules | Test evidence at the slice | Limitations |
| --- | --- | --- | --- | --- | --- | --- | --- |
| W1 | application | #91, `298a3a5`, `9d6b8ad` | Reader and writer ports; `VerifiedEquityRecommendationGenerationWrite` and `VerifiedEquityRecommendationAttemptWrite`; private issuer and weak registry; public validators; recorder migrated to `write_*`; read-only consumers narrowed to the reader; temporary compatibility defaults | `application_services/equity_recommendation_generation_write_authorization.py` (new), `ports/equity_recommendation_generation_store.py`, `application_services/record_equity_recommendation_generation.py`, `application_services/verify_equity_recommendation_generation.py`, `application_services/audit_equity_recommendation_policy_bindings.py`, both package `__init__.py` | `tests/application_services/test_equity_recommendation_generation_write_authorization.py` (new) | Application suite 4,375 passed | Compatibility defaults delegated to the raw methods until W3; not enforcement on its own |
| W2-I | infrastructure | #58, `6b90597`, `5376777` | Adapter implements `write_generation` and `write_attempt`: validate first, private snapshot, then the unchanged `BEGIN IMMEDIATE` transaction; raw methods refuse with `UnverifiedEquityRecommendationGenerationWriteError` before any connection; 38 test call sites migrated; e20 split into two tests | `persistence/sqlite_equity_recommendation_generation.py` | `tests/test_sqlite_equity_recommendation_generation_write_boundary.py` (new), `tests/equity_recommendation_write_authorization_fixtures.py` (new), the EI-4D-B, W2 reliability and EI-4E faults modules and fixtures | Infrastructure suite 2,930 passed, 27 skipped | Refusing raw methods kept until W4-I |
| W3 | application | #92, `8937f12`, `9db9353` | Raw `record_*` and the W1 defaults removed from the port; `write_*` abstract; test doubles migrated; Application architecture scans | `ports/equity_recommendation_generation_store.py`, `application_services/record_equity_recommendation_generation.py` (docstring only) | `tests/application_services/test_equity_recommendation_generation_write_architecture.py` (new), plus the updated W1, EI-4D-A and EI4-D10-S1 test modules | Application suite 4,414 passed; Infrastructure 2,930 passed, 27 skipped, read-only | -- |
| W4-I | infrastructure | #59, `6282cae`, `d6591a3` | Raw `record_*` removed from the adapter; final conformance; Infrastructure architecture scans; end-to-end acceptance of the final boundary | `persistence/sqlite_equity_recommendation_generation.py` | `tests/test_equity_recommendation_generation_write_architecture.py` (new), `tests/test_equity_recommendation_generation_write_boundary_acceptance.py` (new), plus the updated EI-4D-B, W2-I and EI-4E faults modules | Infrastructure suite 2,962 passed, 27 skipped; Application 4,414 passed, read-only | -- |
| W5 | docs | This record, uncommitted | Engineering acceptance documentation | -- | -- | -- | -- |

No slice changed Core, API, Web, a schema, a migration, a canonical identity or any other SQLite adapter. The test-module names differ from the names proposed in the [design, section 15](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md#15-cross-repository-migration): Application uses `test_equity_recommendation_generation_write_authorization.py` and `test_equity_recommendation_generation_write_architecture.py` in place of the proposed `test_equity_recommendation_generation_write_boundary.py`.

## 7. Final authorized-write architecture

### 7.1 Public interfaces

The Application port module is `ports/equity_recommendation_generation_store.py`.

| Interface | Kind | Public operations |
| --- | --- | --- |
| `EquityRecommendationGenerationRecordReader` | Application port (read-only) | `load_disposition`, `load_attempt` |
| `EquityRecommendationGenerationRecordWriter` | Application port (authorized writes) | `write_generation(authorization)`, `write_attempt(authorization)` |
| `EquityRecommendationGenerationStore` | Application combined port | Exactly the four above, every one abstract and inherited; the class adds nothing |
| `SQLiteEquityRecommendationGenerationStore` | Infrastructure adapter | Exactly `load_disposition`, `load_attempt`, `write_generation`, `write_attempt` |

**Removed:** the public `record_generation(disposition, attempt)` and `record_attempt(attempt)` exist on neither the port (W3) nor the adapter (W4-I). The port has no concrete write default; an implementation that supplies only reads, or only raw writes, cannot be instantiated. A call of the absent adapter methods raises `AttributeError`; it is not a typed refusal.

The recorder's own public `RecordEquityRecommendationGenerationUseCase.record_attempt(attempt)` is the use case's attempt-only entry, not a store method. It verifies and writes through `write_attempt`.

### 7.2 Private adapter helpers

These are implementation details of `persistence/sqlite_equity_recommendation_generation.py`, not interfaces:

| Helper | Role | Reached only from |
| --- | --- | --- |
| `_generation_snapshot`, `_attempt_snapshot` | Rebuild the validated records from their immutable canonical fields | `write_generation`, `write_attempt` |
| `_record_generation` | Build the rows and run the generation transaction | `write_generation` |
| `_record_attempt` | Build the row and run the attempt-only transaction | `write_attempt` |
| `_write` | One `BEGIN IMMEDIATE` transaction, a single `_commit`, and W2's cleanup | `_record_generation`, `_record_attempt` |
| `_connect` | Open a connection with foreign keys on | `_write` and the reads |

The Infrastructure architecture scan enforces the "reached only from" column ([section 7.10](#710-architecture-regression-tests)).

### 7.3 Authorization and issuance

- **Owner.** Application owns the authorizations, in `application_services/equity_recommendation_generation_write_authorization.py`.
- **Issuance.** Only the private `_issue_generation_write` and `_issue_attempt_write` issue an authorization, by registering it in the private weak registry `_ISSUED`. Only the recorder module calls them: after the EI4-D10 gate and verification for a generation (`record_equity_recommendation_generation.py`, the call before the bounded write loop), and for the exact attempt for an attempt-only result.
- **Validation.** `require_verified_equity_recommendation_generation_write` and `require_verified_equity_recommendation_attempt_write` check the exact type, registry identity, internal consistency, the canonical re-encoding of the nested assessment document, and that the content still equals the `covered` tuple captured at construction. A failure raises `UnverifiedEquityRecommendationGenerationWriteError`.
- **Construction is not issuance.** A publicly constructed or `dataclasses.replace` copy is refused as "not issued". Infrastructure never imports the issuer or the registry.

### 7.4 Validation before SQLite

`write_generation` and `write_attempt` call the matching public validator as their first statement, with no branch around it. An invalid call therefore opens no connection, begins no transaction, takes no lock and writes nothing. The error stays the Application's typed error and is never reclassified as a storage failure.

### 7.5 Exact validated-payload snapshot binding

- An authorization binds the disposition with its proposal, the logical attempt, the verifier's actual result and the verified policy-binding check, never `eirk1` alone (clarification B).
- After validation, the adapter rebuilds the disposition and attempt from their immutable canonical strings. The rebuilt disposition decodes its own assessment document, which only the adapter holds, and every structured column comes from it.
- A caller mutating its decoded document after validation therefore cannot change what is committed.

### 7.6 Atomic persistence, idempotence and conflict

Unchanged from EI-4D-B and W2:
- one `BEGIN IMMEDIATE` transaction takes the write lock before reading;
- the disposition, its proposal (same row) and the attempt commit together, with one `COMMIT`;
- the same whole record already stored: `EQUIVALENT`; other content or another evaluator version under the key: `CONFLICT`, never replaced;
- uniqueness of the generation key and attempt identity is the database's.

### 7.7 Immutable provenance and append-only records

The schema, its insert-only triggers, `CHECK` constraints, foreign keys to the CANDIDATE timeline entry and EI-2B snapshot, and the source-matching triggers are unchanged. No row is updated or deleted, and no evidence is backfilled or migrated.

### 7.8 Attempt-only writes

An attempt-only result (an integrity failure, an outage, no candidate, an early refusal) is authorized by `VerifiedEquityRecommendationAttemptWrite` for that exact attempt. No policy evidence, assessment, proposal or disposition is required or fabricated. Attempt identity and status formats are unchanged, and no new persisted status exists.

### 7.9 Bounded commit-uncertainty reconciliation

The recorder issues **one** authorization per operation and reuses it, unchanged, for at most `max_writes` writes during the existing EI4-D9-G reconciliation. No authorization is minted during reconciliation, no nonce blocks a legitimate retry, and a retry whose content changed is refused. The adapter never retries.

### 7.10 Architecture regression tests

- **Application** (`test_equity_recommendation_generation_write_architecture.py`, 40 cases): the exact port contract; no raw method or concrete default on any generation port; incomplete implementations stay abstract; every recorder branch writes only through its one issued authorization; read-only consumers take the reader only; a syntax-tree scan of production source.
- **Infrastructure** (`test_equity_recommendation_generation_write_architecture.py`, 26 cases): the adapter's exact public surface, no alias and no constructor switch; validation as the first statement and before `_connect` at runtime; a syntax-tree scan of production source for raw methods and calls, public aliases, private-helper calls outside their entry, Infrastructure calls of `write_*`, authorization construction, issuer references, and construction of the adapter outside a recorder or read-only use-case wiring; and a scan keeping the issuer out of the tests.
- Both scans ignore comments, docstrings and strings, and do not flag similarly named operations of other stores (`record_audit`, `record_progress`, `_record_one`, the other adapters' `_write`).

### 7.11 No production composition root

**NO PRODUCTION OPERATIONAL COMPOSITION ROOT EXISTS AT EI4-D11 ACCEPTANCE.**

- Infrastructure has no entry points or CLI; the adapter is referenced only by its own module and the `persistence` package export (asserted by `test_c_the_caller_inventory_is_exactly_the_adapter_and_its_export`).
- Application does not wire the recorder anywhere.
- The API (`northstar_api/cli.py` and its routers) has no equity recommendation-generation reference; its only "equity" text is the futures paper equity curve. Web has none. This was a read-only manual review at W4-I, not a CI test.

The scans establish that current source exposes no raw generation writer. They cannot certify future operational entry points ([section 15](#15-future-integration-requirements)).

## 8. EI-4E AA-2 (e20) closure

e20 now proves two different guarantees, both in `tests/test_equity_recommendation_generation_acceptance_faults.py`, each run for two contradictions (signal close, opening-range low):

| Guarantee | Test | Evidence |
| --- | --- | --- |
| A. The unsupported direct write is closed | `test_e20_an_unauthorized_adapter_write_of_contradictory_evidence_is_refused` | The adapter has no `record_generation` or `record_attempt`; `write_generation` refuses the raw forged records and a genuine authorization tampered to carry the forged content (a labelled privileged simulation); zero `_connect` calls; the recorder refuses to authorize the forgery (`SOURCE_EVIDENCE_INVALID`, zero writes); no row |
| B. Existing invalid evidence still fails closed | `test_e20_contradictory_candidate_evidence_is_caught_by_the_application_verifier` | The forgery is planted by labelled test-only raw SQL (`plant_generation_records`); verification refuses it naming the field; the genuine record gets `STORED_RECORD_INVALID`; the key is never overwritten; no recovery |

AA-2 is closed **for supported and ordinary operational writes**. The occupied-key consequence of a privileged write remains, because recovery is not approved (L3).

## 9. Mutation detection

Temporary regressions were introduced into production source, the focused tests run, and the source restored byte-identically (SHA-256 confirmed). Every regression was detected.

| Slice | Regression | Failing tests |
| --- | --- | --- |
| W4-I | 1. Public raw `record_generation` reintroduced on the adapter | 8 |
| W4-I | 2. Public raw `record_attempt` reintroduced on the adapter | 7 |
| W4-I | 3. Validator removed from `write_generation` | 14 |
| W4-I | 4. Validator removed from `write_attempt` | 7 |
| W4-I | 5. SQLite opened before validation | 13 |
| W4-I | 6. Public alias of a private persistence helper | 3 |
| W4-I | 7. Production call bypassing the authorized writer | 1 (the production scan) |
| W3 | Raw attempt call in the recorder | 32 |
| W3 | W1 default and raw method back on the port | 87 |
| W3 | Concrete write default alone on the port | 7 |
| W3 | The read-only audit writes | 3 |
| W3 | Authorization re-issued on each retry | 3 |
| W2-I | Validator skipped; snapshot removed; raw writes allowed | 7; 1; 4 |

These counts are as reported in the W2-I, W3 and W4-I implementation reports. They were not re-run for this record.

## 10. Acceptance evidence matrix

Required layers follow the design's [acceptance matrix](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md#17-acceptance-matrix):

- **C:** Application contract tests. W1: `tests/application_services/test_equity_recommendation_generation_write_authorization.py`; W3: `test_equity_recommendation_generation_write_architecture.py`; EI-4D-A: `test_record_equity_recommendation_generation.py`; EI4-D10: `test_equity_synthetic_policy_enforcement.py`.
- **S:** Infrastructure SQLite tests. W2-I: `tests/test_sqlite_equity_recommendation_generation_write_boundary.py`; EI-4D-B: `test_sqlite_equity_recommendation_generation.py`; W2: `test_sqlite_equity_recommendation_generation_reliability.py`.
- **E:** end-to-end tests on real SQLite. W4-I: `tests/test_equity_recommendation_generation_write_boundary_acceptance.py`; EI-4E: `test_equity_recommendation_generation_acceptance.py`, `test_equity_recommendation_generation_acceptance_faults.py`; EI4-D10-S4: `test_equity_synthetic_policy_enforcement_acceptance.py`.
- **A:** architecture or source scans. Application and Infrastructure `test_equity_recommendation_generation_write_architecture.py`.

"App" marks Application tests; every other test is Infrastructure. Test names are abbreviated with `…`. Every listed test passed in the post-merge run ([section 11.2](#112-post-merge-validation-2026-11-11)).

| # | Requirement | Layers | Slices | Supporting tests | Limitation | Determination |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Valid generation through the verified recorder | C, E | W1, W4-I | C App `test_b_a_verified_disposition_is_written_through_its_issued_authorization`, `test_c_every_recorder_branch_writes_only_through_its_one_issued_authorization[committed-generation]`; E `test_w4i_1_19_the_chain_commits_exactly_what_the_recorder_authorized`, `test_the_synthetic_chain_writes_only_through_the_enforced_boundary`, EI-4E `test_e8_…` | -- | VERIFIED |
| 2 | Valid attempt-only recording | C, E | W1, W4-I | C App `test_e_a_genuine_attempt_only_result_is_authorized_without_policy_evidence`; E `test_w4i_2_a_genuine_attempt_only_result_is_recorded_without_policy_evidence` (nothing published; catalog reads forbidden) | -- | VERIFIED |
| 3 | Direct raw-store write refused; absent after W4-I | S, E | W2-I, W4-I | S `test_the_legacy_raw_writes_are_gone_and_raw_records_are_refused_before_sqlite`, `test_a_the_adapter_implements_the_merged_application_port`, `test_a_the_adapter_exposes_exactly_the_port_operations`; E `test_w4i_3_30_the_real_chain_has_no_raw_entry_and_refuses_fabrications`, e20-A (×2) | The design's W2-I typed refusal of the legacy entry is replaced, as the row anticipates, by its absence; raw records offered to `write_*` are refused with no connection | VERIFIED |
| 4 | Missing mandatory recorder dependency | C | W1 | App `test_contracts_are_validated_and_exported`, `test_b_the_recorder_requires_a_catalog` | -- | VERIFIED |
| 5 | Missing verification evidence | C, S | W1, W2-I | C App `test_d_missing_foreign_and_unissued_authorizations_are_refused_before_any_write`; S `test_missing_wrong_type_and_look_alike_authorizations_are_refused`, `test_the_legacy_raw_writes_are_gone_…` | -- | VERIFIED |
| 6 | Forged or mismatched authorization | C, S | W1, W2-I | C App `test_c_an_authorization_accepts_exactly_its_own_content`, `test_c_a_disposition_changed_after_issuance_is_refused` (×6), `test_c_changed_nested_canonical_content_is_refused_without_privilege`, `test_c_another_attempt_or_policy_check_is_refused`, `test_c_one_operation_never_authorizes_another_under_the_same_key`, `test_c_matching_identifiers_without_issuance_are_insufficient`, `test_d_an_inconsistent_authorization_cannot_even_be_constructed`; S `test_unissued_and_copied_authorizations_are_refused`, `test_a_disposition_changed_after_issuance_is_refused` (×4), `test_changed_nested_assessment_attempt_or_policy_identity_is_refused` | -- | VERIFIED |
| 7 | Unpublished policy definition | C, E | W1, W4-I | C App `test_r6_r11_an_unpublished_bundle_is_not_established_and_never_published`, `test_b_no_authorization_is_issued_when_a_mandatory_check_fails[policy catalog]`; E `test_s4_the_recorder_refuses_unpublished_policy_evidence_before_any_write` (zero writes) | -- | VERIFIED |
| 8 | Conflicting policy binding | C, E | W1, W4-I | C App `test_r5_component_labels_published_but_the_set_bound_to_others_is_a_conflict`, `test_g5_…`, `test_g6_…`; E `test_s4_scenario_33_a_permanent_conflict_is_never_retried_or_republished` (recorder gate `POLICY_REFERENCE_CONFLICT`, zero writes) | -- | VERIFIED |
| 9 | Unavailable policy catalog | C, E | W1, W4-I | C App `test_r8_a_catalog_outage_is_policy_reference_unavailable`; E `test_s4_scenario_31_a_catalog_outage_at_the_recording_gate_writes_nothing`, `test_the_recorder_writes_through_the_enforced_adapter_and_a_catalog_failure_writes_nothing` | -- | VERIFIED |
| 10 | Invalid frozen inputs | C | W1 | App `test_r_a_replay_that_does_not_reproduce_fails_closed` | -- | VERIFIED |
| 11 | Invalid source provenance | C, E | W1, W2-I | C App `test_b_no_authorization_is_issued_when_a_mandatory_check_fails[source evidence]`, `test_ai_a_disposition_whose_source_no_longer_verifies_is_never_written`; E e20-A (×2) | -- | VERIFIED |
| 12 | Invalid existing committed evidence | E | W2-I | e20-B, `test_e20_contradictory_candidate_evidence_is_caught_by_the_application_verifier` (×2) | Privileged test-only SQL simulation; no recovery of occupied invalid keys | VERIFIED WITH LIMITATION |
| 13 | Incompatible stored records | S, E | W2-I | S `test_p_an_unsupported_stored_evaluator_is_incompatible`; E `test_e21_a_corrupt_stored_attempt_is_typed_by_the_adapter_and_mapped_by_the_recorder`, `test_e21_a_corrupt_stored_disposition_leaves_the_attempt_state_unknown` | -- | VERIFIED |
| 14 | Conflicting committed content | C, E | W1, W4-I | C App `test_v_same_key_different_content_is_a_conflict_never_an_overwrite`, `test_g_a_conflicting_stored_record_is_unchanged_by_the_boundary`; E `test_w4i_14_23_later_operations_never_rewrite_committed_evidence`, `test_e19_two_conflicting_writers_leave_one_authoritative_disposition`, EI-4E `test_e12_d_e_g_h_…` | -- | VERIFIED |
| 15 | Evaluator-version conflict | C, S | W1, W2-I | C App `test_evaluator_2_equal_fingerprints_under_another_evaluator_are_not_equivalent`, `test_evaluator_6_…`; S `test_o_the_same_content_under_another_evaluator_is_a_conflict`, `test_o_a_stored_record_of_another_evaluator_is_reported_never_adopted` | -- | VERIFIED |
| 16 | Definite rollback | S | W2-I | `test_w2_c_e_a_refusal_before_commit_stays_a_definite_non_commit`, `test_v_a_writer_that_stops_before_commit_leaves_nothing`, `test_e17_a_failure_before_commit_leaves_no_partial_generation_record` | -- | VERIFIED |
| 17 | COMMIT succeeds, acknowledgment lost | S, E | W2-I | S `test_y_the_recorder_reconciles_a_lost_acknowledgment_by_reload`, `test_w2_j_the_recorder_reconciles_a_connection_lost_at_commit`; E `test_e18_a_lost_acknowledgment_is_reconciled_as_committed_by_reload`; reuse: App `test_g_retries_reuse_the_same_authorization_within_the_bound` | -- | VERIFIED |
| 18 | COMMIT uncertain, reload fails | S | W2-I | `test_y_a_failed_reload_leaves_the_outcome_unresolved`, `test_e18_an_unavailable_reload_leaves_the_commit_unresolved` | -- | VERIFIED |
| 19 | Duplicate equivalent recording | C, E | W1, W4-I | C App `test_u_an_equivalent_duplicate_is_idempotent`; E `test_w4i_1_19_…`, `test_e19_two_writers_of_equivalent_content_store_it_once` | -- | VERIFIED |
| 20 | Independent SQLite connections | S | W2-I | `test_aa_two_writers_of_identical_content_store_it_once`, `test_ab_the_first_committed_content_wins_and_the_loser_is_audited`, `test_ac_…` (×2) | -- | VERIFIED |
| 21 | Windows separate processes | E | W2-I | `test_e19_separate_os_processes_are_serialized_by_the_database` | Windows only; no POSIX or multi-host behaviour established | VERIFIED WITH LIMITATION |
| 22 | Historical replay without the current catalog | C, E | W1, W4-I | C App `test_h_historical_verification_and_audit_take_a_reader_and_issue_nothing`, `test_h_frozen_records_replay_and_verify_with_the_catalog_down`; E `test_the_synthetic_chain_writes_only_through_the_enforced_boundary`, `test_w4i_14_23_…` | -- | VERIFIED |
| 23 | No evidence backfill or migration | E | W4-I | `test_w4i_14_23_…` (rows byte-identical by rowid; schema identical), `test_s4_pre_enforcement_records_replay_and_audit_without_backfill`, `test_e22_recording_adds_its_schema_and_never_alters_earlier_evidence` | Tested against enforced code and planted historical records; historical revisions were not rerun | VERIFIED WITH LIMITATION |
| 24 | Existing EI-4D-B, EI-4E and EI4-D10 acceptance | S, E | W2-I, W4-I | EI-4D-B, W2 reliability, EI-4E acceptance and faults, S2 catalog and S4 acceptance modules; full suites ([section 11.2](#112-post-merge-validation-2026-11-11)) | -- | VERIFIED |
| 25 | No issuance or broker behaviour | A, E | W4-I | A `test_c_the_adapter_reaches_no_broker_exchange_network_or_messaging_code`; E `test_w4i_25_recording_issues_nothing_and_writes_no_history_or_orders`, `test_e14_recording_never_writes_history_and_never_blocks_its_own_retry`; App `test_al_no_issuance_or_lifecycle_state_exists_here` | -- | VERIFIED |
| 26 | Read-only consumers cannot write | A, C | W1, W3 | App `test_d_read_only_consumers_take_the_reader_and_never_write` (×2), `test_e_application_production_source_respects_the_write_boundary`, `test_h_…` | -- | VERIFIED |
| 27 | Only the recorder issues authorizations | A | W3, W4-I | App `test_only_the_recorder_issues_and_nothing_imports_infrastructure`, `test_c_the_recorder_writes_only_what_it_issued_in_the_same_operation`; `test_c_infrastructure_production_source_respects_the_write_boundary`, `test_c_tests_reach_the_issuer_only_through_the_real_recorder` | -- | VERIFIED |
| 28 | Refusal precedes any transaction | S | W2-I | Every refusal in the W2-I module (counted `_connect`, `BEGIN EXCLUSIVE` immediately available, rows unchanged); `test_b_validation_returns_before_sqlite_is_opened` (×2); also E `test_w4i_3_30_…` | -- | VERIFIED |
| 29 | Composition-root rule | A | W3, W4-I | App `test_e_application_production_source_respects_the_write_boundary`; `test_c_infrastructure_production_source_respects_the_write_boundary` (composition-root rule), `test_c_the_caller_inventory_is_exactly_the_adapter_and_its_export`; API and Web: read-only review only | No production composition root exists; future integrations cannot be certified by the current scans | VERIFIED WITH LIMITATION |
| 30 | Fabricated direct attempt-only write refused | S, E | W2-I, W4-I | C App `test_e_a_fabricated_or_changed_attempt_write_is_refused`; S `test_unissued_and_copied_authorizations_are_refused`, `test_changed_nested_assessment_attempt_or_policy_identity_is_refused`; E `test_w4i_3_30_…` (six fabrications refused, then the genuine attempt commits) | -- | VERIFIED |
| 31 | Migration ordering | C, A, S | W1, W2-I | C and A: App `test_c_every_recorder_branch_…` (×14), `test_c_the_recorder_writes_only_what_it_issued_in_the_same_operation`, the W3 scans; S: the Infrastructure suite reported green against W1 (2,914 passed, 27 skipped) and at W2-I (2,930 passed, 27 skipped) | Relies partly on previously reported W1/W2-I runs rather than rerunning historical revisions | VERIFIED WITH LIMITATION |
| 32 | Reuse only for the identical operation | C, S | W1, W2-I | C App `test_g_retries_reuse_the_same_authorization_within_the_bound` (×4), `test_g_a_retry_whose_content_changed_is_refused`, `test_c_one_operation_never_authorizes_another_under_the_same_key`, `test_c_every_recorder_branch_…`; S `test_y_a_reload_that_finds_nothing_retries_once_without_duplicates`, `test_e18_retries_are_bounded_and_never_duplicate` | -- | VERIFIED |
| 33 | Restart and durable persistence | E | W4-I | `test_the_synthetic_chain_writes_only_through_the_enforced_boundary` (generation), `test_w4i_33_attempt_only_evidence_survives_restart_and_no_authorization_persists`, `test_w4i_14_23_…`, `test_ae_ah_a_reopened_store_verifies_and_replays_what_was_committed` | -- | VERIFIED |

**Totals: 28 VERIFIED, 5 VERIFIED WITH LIMITATION, 0 NOT VERIFIED.** The count is of scenarios, not test cases; a parameterized test is counted once per scenario it supports.

### 10.1 Scenarios VERIFIED WITH LIMITATION

These limitations are part of the determination and must not be read as unrestricted verification:

- **12. Invalid committed evidence:** privileged test-only SQL simulation; no recovery of occupied invalid keys.
- **21. Windows separate-process testing:** no POSIX or multi-host behavior established.
- **23. No backfill or migration:** tested against enforced code and planted historical records; historical revisions were not rerun.
- **29. Composition-root rule:** no production composition root exists; future integrations cannot be certified by the current scans.
- **31. Migration ordering:** relies partly on previously reported W1/W2-I runs rather than rerunning historical revisions.

### 10.2 Coverage boundaries

- Application contract evidence is reported as Application evidence; it is not counted as Infrastructure SQLite evidence.
- API and Web are not checked out in Infrastructure CI; their inspection is review evidence only.
- No live-data acceptance exists for any scenario.

## 11. Test results

Focused counts are subsets of the full-suite totals. They are listed for traceability and never added to them.

### 11.1 Historically reported implementation results

| Run | Application | Infrastructure |
| --- | --- | --- |
| W1 implementation | 4,375 passed | 2,914 passed, 27 skipped (against W1) |
| W2-I implementation | 4,375 passed (read-only) | 2,930 passed, 27 skipped |
| W3 implementation | 4,414 passed | 2,930 passed, 27 skipped (read-only) |
| W4-I implementation | 4,414 passed (read-only) | 2,962 passed, 27 skipped |

At W4-I, also reported:
- Ruff check and format check passed; `git diff --check` passed.
- The architecture mutation checks detected all seven tested regressions ([section 9](#9-mutation-detection)).
- Invalid authorized writes were refused before SQLite opened, counted at the real `_connect` seam.
- No other repository was modified.
- CI was not reported as run.

The 27 skips are 23 opt-in live Databento tests and 4 POSIX-only `tzset` tests. None relates to EI4-D11.

### 11.2 Post-merge validation (2026-11-11)

For this record, the suites were run again, read-only, against the merged `develop` branches (Application `8937f12`, Infrastructure `6282cae`, Core `bc081a2`). Infrastructure imported the Application working tree at `8937f12`, whose port requires exactly `load_attempt`, `load_disposition`, `write_attempt` and `write_generation`. Python 3.13.15 on Windows.

| Run | Result |
| --- | --- |
| Application, full suite | 4,414 passed, 0 failed |
| Application, W1 authorization module | 29 passed |
| Application, W3 architecture module | 40 passed |
| Application, EI-4D-A recorder module | 86 passed |
| Application, EI4-D10 S3 enforcement module | 40 passed |
| Application, EI4-D10 S1 catalog module | 94 passed |
| Infrastructure, full suite | 2,962 passed, 27 skipped, 0 failed (the same 27 skips as in [section 11.1](#111-historically-reported-implementation-results)) |
| Infrastructure, W4-I architecture module | 26 passed |
| Infrastructure, W4-I end-to-end acceptance module | 6 passed |
| Infrastructure, W2-I write-boundary module | 14 passed |
| Infrastructure, EI-4D-B module | 73 passed |
| Infrastructure, W2 reliability module | 22 passed |
| Infrastructure, EI-4E acceptance module | 38 passed |
| Infrastructure, EI-4E faults module | 36 passed |
| Infrastructure, e19 selection (concurrency, Windows spawned processes) | 6 passed |
| Infrastructure, e20 selection | 11 passed |
| Infrastructure, EI4-D10 S2 catalog module | 66 passed |
| Infrastructure, EI4-D10 S4 acceptance module | 18 passed |

No implementation, test, schema or fixture file was changed by the run; both working trees stayed clean. Ruff and the mutation checks were not re-run for this record; their results are as reported in [section 11.1](#111-historically-reported-implementation-results).

### 11.3 CI status

**CI verification: NOT INDEPENDENTLY VERIFIED FROM AVAILABLE EVIDENCE.**

- No CI run result was available to this review: the GitHub CLI is not installed on the development machine and the repositories are private, so post-merge workflow runs could not be read. CI status is therefore **unavailable**, not failed and not passed.
- Infrastructure CI (`.github/workflows/python-ci.yml`) checks out Application and Core at `develop`, so it now builds W4-I against W3.
- The results in sections 11.1 and 11.2 are evidence of local executions only and are not CI results.

## 12. Trust limitations and excluded threat models

The boundary prevents **supported and ordinary** operational writes that bypass verification (EI4D11-DQ1). It does **not** protect against, and this record claims no protection against:

- privileged arbitrary Python, including importing the private issuer or editing the registry;
- direct SQL execution or direct SQLite file modification, including dropping triggers;
- malicious source changes;
- forged capability objects: an authorization is a Python capability object, not cryptographic proof;
- multi-host or distributed authorization;
- production trading security.

These are the ADR-022 DQ9 and ADR-023 limitations, appropriate to personal-use synthetic research only. The test-only raw-SQL setup (`plant_generation_records`, `bypass`) demonstrates them; it is labelled, not exported from production, and not a supported path.

## 13. Remaining engineering limitations

### L1 -- Synthetic-only acceptance

No real market data, provider, issuance, broker or trading acceptance exists. Every record and policy is synthetic.

### L2 -- Platform coverage

Windows separate-process and restart tests passed. POSIX separate-process behaviour and multi-host behaviour are **not established** (EI4D11-DQ8).

### L3 -- Occupied invalid keys

A key already occupied by an invalid record, including one planted before enforcement or by privileged access, stays occupied and fails closed. Recovery is not approved (EI4-D11-D).

### L4 -- No composition root

No production code path writes generation records. The boundary is evidenced by tests and scans; it is not exercised by an operational caller.

### L5 -- Future entry points

The scans cover current source. A future API, CLI or background entry point must add the composition-root scan to its own repository ([section 15](#15-future-integration-requirements)).

### L6 -- Historical revisions

Migration ordering (row 31) and no-backfill (row 23) rely partly on reported or simulated history; historical revisions were not rerun.

### L7 -- Capability, not cryptography

Within the personal-use trust model, the issuer capability is not tamper-proof against privileged code (clarification B).

### L8 -- CI

CI is not independently verified ([section 11.3](#113-ci-status)).

### L9 -- Additional SQLite cleanup findings

The cleanup-masking pattern reported outside EI-4D-B during EI4-D10 ([EI4-D10 record, L9](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md#l9----additional-sqlite-cleanup-findings)) is unchanged. EI4-D11 neither changes nor resolves it.

### L10 -- Other stores

The futures, options and paper-trading stores are not covered by EI4-D11 and are unchanged.

## 14. Features outside EI4-D11

EI4-D11 does **not** implement, and this record does not claim:

- a production composition root;
- an equity recommendation-generation CLI or API;
- broker integration or order placement;
- positions, fills or P&L processing;
- recovery or repair tooling;
- live-market execution;
- authorized real-market data integration;
- production trading readiness;
- actual recommendation issuance or the lifecycle `RECORDED` state;
- EI-5 or EI-6.

The futures and options functionality is unchanged.

## 15. Future integration requirements

Any future operational wiring of generation recording must:

1. construct `RecordEquityRecommendationGenerationUseCase` and pass the SQLite adapter **only** to it (or to a read-only use case for reads);
2. never call `write_generation` or `write_attempt` itself, and never reach the adapter's private helpers;
3. add the composition-root architecture scan (design row 29) to the repository that adds the wiring, including the API when equity wiring appears there;
4. pass architecture review before it is used.

Recovery of occupied invalid keys needs its own approved design (EI4-D11-D).

## 16. Unverified or unavailable evidence

- **CI:** unavailable; not verified ([section 11.3](#113-ci-status)).
- **Historical revisions:** not rerun (rows 23 and 31).
- **POSIX and multi-host:** not tested.
- **API and Web:** reviewed manually; no CI scan.
- **Mutation checks:** as reported at each slice; not re-run for this record.

## 17. Engineering verdict

**EI4-D11 ENGINEERING ACCEPTANCE: PASS WITH DOCUMENTED LIMITATIONS -- synthetic engineering acceptance.**

1. **Implemented and locally verified.** W1 to W4-I are merged. The Application port and the SQLite adapter expose exactly `load_disposition`, `load_attempt`, `write_generation` and `write_attempt`; the raw `record_*` interfaces are removed. Every write is validated by the Application before SQLite opens, persists exactly the authorized snapshot, atomically, and keeps first-writer immutability, attempt-only semantics and bounded reconciliation. All 33 scenarios are traced to passing evidence of their required layers, and the full suites pass locally after the final merge.
2. **Limitations and excluded threat models.** L1 to L10 and [section 12](#12-trust-limitations-and-excluded-threat-models) apply, including the five scenario limitations of [section 10.1](#101-scenarios-verified-with-limitation).
3. **Outside EI4-D11.** [Section 14](#14-features-outside-ei4-d11).
4. **Future integration requirements.** [Section 15](#15-future-integration-requirements).
5. **Unverified or unavailable evidence.** [Section 16](#16-unverified-or-unavailable-evidence), including CI.

It does **not** establish live deployment readiness, market-data consumption, brokerage activity, invalid-record recovery, or any profitability or trading-readiness claim.

## 18. Non-authorizations

This record authorizes nothing. In particular it does not authorize:

- invalid-record recovery, historical evidence repair, deletion, backfill or migration;
- changes to `eira1`, `eirp1`, `eirat1`, `eirs1`, `eirk1` or any canonical document version;
- a production composition root, API route, CLI command, scheduler or background writer;
- direct-adapter operational recording;
- actual recommendation issuance or the lifecycle `RECORDED` state;
- broker or paper orders, fills or positions;
- realized P&L, live data integrations, data-use licensing claims or production trading;
- any production policy value or default;
- extension of EI4-D11 to the futures, options or paper-trading stores;
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
- the EI4-D11 recovery mechanism (EI4-D11-D, not approved).

## 20. Related documents

- [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md), sections 10.3, 13 and 14
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)
- [EI4-D11 Operational Generation-Write Boundary Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md)
- [EI4-D10 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md)
- [Equity Intraday V1 EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md)
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
