# Equity Intraday V1 — EI-6C Lifecycle Monitoring and Snapshot Selection Engineering Acceptance Record

## Status

**Engineering acceptance record and owner acceptance, 2026-11-15.** This is Docs PR-3 of the EI-6C plan governed by the [EI-5 Decision Register, section 22](Equity-Intraday-V1-EI5-Decision-Register.md#22-owner-authorization-of-ei-6c-governance-2026-11-15) and recorded in [section 23](Equity-Intraday-V1-EI5-Decision-Register.md#23-ei-6c-implementation-and-acceptance-2026-11-15).

**EI-6C — ACCEPTED WITH DOCUMENTED LIMITATIONS, 2026-11-15 (owner; EI6C-FINAL-ACCEPTANCE).** See [section 14.1](#141-final-owner-decision-2026-11-15).

- **What it covers:** EI-6C — Equity Recommendation Lifecycle Monitoring and Snapshot Selection, merged as northstar-application #99 and northstar-infrastructure #69 under the governance of northstar-docs #69 ([section 4](#4-repository-baselines)). Synthetic-only and MODELED-only.
- **Acceptance matrix (owner-accepted):** 40 governed rows — **36 VERIFIED, 4 VERIFIED WITH LIMITATION** (MS-02, MS-20, MS-21, MS-36), 0 NOT TESTED, 0 BLOCKED, unchanged from the engineering proposal ([section 8](#8-acceptance-evidence-matrix)).
- **Linux CI:** the four relevant jobs (PR and post-merge, both implementation PRs) completed **SUCCESS**, every step including Ruff, the format check and pytest. Their pytest summaries, from **owner-supplied GitHub Actions log excerpts**, show Application **4,799 passed of 4,799** and Infrastructure **3,532 passed, 24 skipped of 3,556**, 0 failed in every job, on Python 3.13.18 ([section 7.2](#72-linux-ci-evidence)).
- **Acceptance gate EI6C-CI: CLOSED — PASS** (2026-11-15). The Linux skip-reason breakdown is derived from the test declarations, and the checked-out dependency commits are not verified ([section 7.3](#73-acceptance-gate-ei6c-ci)).
- **Limitations:** the four scenario limitations and L1 to L13 are accepted, not corrected ([section 11](#11-limitations)). The follow-ups F1 to F10 stay unauthorized ([section 12](#12-follow-up-register)).
- **Not authorized.** Neither this record nor the owner acceptance authorizes real market data, ACTUAL issuance, new issuance functionality, broker execution, scheduling, API or Web integration, deployment or any trading ([section 15](#15-non-authorizations)).

## 1. Acceptance identity

| Item | Value |
| --- | --- |
| Milestone | EI-6C — Equity Recommendation Lifecycle Monitoring and Snapshot Selection |
| Record | Docs PR-3, engineering acceptance, CI-gate closure and owner acceptance |
| Date | 2026-11-15 |
| Governance | [EI-5 Decision Register, section 22](Equity-Intraday-V1-EI5-Decision-Register.md#22-owner-authorization-of-ei-6c-governance-2026-11-15) (EI6C-P1 to EI6C-P12) and [section 23](Equity-Intraday-V1-EI5-Decision-Register.md#23-ei-6c-implementation-and-acceptance-2026-11-15) (implementation authorizations, clarifications and EI6C-FINAL-ACCEPTANCE); [EI-3 Decision Register, section 12](Equity-Intraday-V1-EI3-Decision-Register.md#12-ei3-d10-owner-decision-2026-11-15) (EI3-D10); [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), section 13 |
| Design | [EI-6C Lifecycle Monitoring and Snapshot Selection Design v1.0](../architecture/Equity-Intraday-V1-EI6C-Lifecycle-Monitoring-and-Snapshot-Selection-Design-v1.0.md) |
| Data | Synthetic evidence only; MODELED issuances only |
| Authority | The existing EI-2B snapshot table (read only), the EI-6A issuance authority and the EI-6B lifecycle store, in one SQLite file; single host |
| Platform | Local Windows executions; Linux CI job conclusions and owner-supplied pytest summaries (section 7.2) |

## 2. Scope

**In scope:**

- **Application (#99):** the read-only snapshot catalog port and errors; the additive read-only lifecycle reader protocol (V1); pure snapshot selection; `MonitorEquityRecommendationLifecyclesUseCase` with its scope modes, request, outcomes, refusals, diagnostics and report; the fixed policy identity and catalog limit; the C and A tests.
- **Infrastructure (#69):** `SQLiteEquityCandleEvidenceSnapshotCatalog` over the existing EI-2B table; real-SQLite composition of the merged monitor with the EI-6A authority, EI-4D generation records, the EI-6B recorder and store; the S and P tests.

**Out of scope:** snapshot issuance, research, generation, declaration or issuance orchestration; scheduling or polling; real or live market data; ACTUAL issuance; orders, fills, positions or brokers; API and Web; EI-8 restatement; any Core change; any change to the EI-2B, EI-4D, EI-6A or EI-6B persistence contracts; any schema change.

## 3. Governing owner decisions

No decision below is created by this record. Each is stated with its source and date. The final owner acceptance, EI6C-FINAL-ACCEPTANCE, is recorded in [section 14.1](#141-final-owner-decision-2026-11-15).

### 3.1 Governance (approved 2026-11-15)

| Decision | Summary | Source |
| --- | --- | --- |
| EI6C-P1 to EI6C-P12 | Scope L1; O-2 option A (Δ = 0); EI3-D10 option (a); read-only catalog with dual-timestamp eligibility; ascending one-snapshot-per-call recording; no checkpoint (K1); three scope modes; conflict and revision rules; per-issuance isolation; EI-6B concurrency only; typed outcomes; non-durable policy provenance | [EI-5 Decision Register, section 22.1](Equity-Intraday-V1-EI5-Decision-Register.md#221-approved-decisions-ei6c-p1-to-ei6c-p12) |
| O-3 | Option (b), unchanged since 2026-11-14 | [Section 22.2](Equity-Intraday-V1-EI5-Decision-Register.md#222-o-2-and-ei3-d10-resolved) |
| 40 scenarios | MS-01 to MS-40, PLANNED | [Design, section 12](../architecture/Equity-Intraday-V1-EI6C-Lifecycle-Monitoring-and-Snapshot-Selection-Design-v1.0.md#12-acceptance) |

### 3.2 Implementation authorizations and clarifications

| ID | Decision | Status and date |
| --- | --- | --- |
| Application PR-1 | Lifecycle monitoring, snapshot selection and Application contracts | **APPROVED**, 2026-11-15 (owner implementation authorization) |
| Infrastructure PR-2 | Read-only SQLite snapshot catalog, monitoring integration and process-level reliability | **APPROVED**, 2026-11-15 (owner implementation authorization, after PR-1 merged) |
| V1 | Additive read-only lifecycle reader protocol (`EquityRecommendationLifecycleReader`, `load_lifecycle` only), so the monitor never names the EI-6B store and EI-6B's accepted architecture test stays unchanged | **APPROVED**, 2026-11-15 (recorded in the PR-2 authorization) |
| V2 | Non-durable late-historical diagnostic `LATE_HISTORICAL_SNAPSHOT_IGNORED`, emitted only for snapshots older than the anchor whose **declared** `issued_at` is after the anchor snapshot's | **APPROVED WITH QUALIFICATION**, 2026-11-15 (recorded in the PR-2 authorization); best-effort, based on declared issuance chronology |
| V3 | Fixed catalog limit of **300 eligible snapshots** per issuance per run (`EQUITY_LIFECYCLE_MONITORING_CATALOG_LIMIT`); overflow is a typed refusal, never truncation | **APPROVED**, 2026-11-15 (recorded in the PR-2 authorization) |
| I2-1 | Measured near-limit performance (section 9) accepted for synthetic v1 | **APPROVED WITH LIMITATION** — recorded from the owner's Docs PR-3 instruction, 2026-11-15 |
| I2-2 | Retain both the catalog's own EI-2B verification and the monitor's re-verification of every candidate | **APPROVED** — recorded from the owner's Docs PR-3 instruction, 2026-11-15 |
| I2-3 | The inherited misclassification of corrupt anchor evidence as retryable unavailability is accepted; no unauthorized contract change is made | **APPROVED WITH LIMITATION** — recorded from the owner's Docs PR-3 instruction, 2026-11-15 |

V3 fixes the number that the design's section 6.4 left open ("the numerical limit is not yet fixed"); the design text is not rewritten.

## 4. Repository baselines

Read from local Git on 2026-11-15; every repository on `develop` with a clean working tree; no fetch was run. The Core, Application, Infrastructure, Docs and API HEADs also equal the `develop` HEADs reported by the GitHub REST API on 2026-11-15. northstar-web is private to unauthenticated access, so its remote HEAD was not confirmed.

| Repository | `develop` HEAD | Role |
| --- | --- | --- |
| northstar-application | `617b538` (#99; merge `617b5386da62fdda381c5edff521cfd019799a19`, parents `b7df16b`, `e13beb0`; change commit `e13beb04a89ebccf9d77901695efd5863a03822f`) | EI-6C contracts, selection, monitor, C and A tests |
| northstar-infrastructure | `9d9c960` (#69; merge `9d9c9607b1fee93ceeb1060d21fcd7b52ee0008c`, parents `dd6a434`, `889559d`; change commit `889559d01259f20aa6efbbdcc936615e14c64381`) | SQLite catalog adapter, S and P tests |
| northstar-docs | `372e0ff` (#69; EI-6C governance, change commit `0b287e7`) | Governance; this record, uncommitted |
| northstar-core | `29bd34a` (#40) | Unchanged throughout EI-6C |
| northstar-api | `7ad37f5` | Unchanged |
| northstar-web | `5aeb94c` (local) | Unchanged |

**Change scope (both purely additive):**

- **`e13beb0`:** 7 files, 2,326 insertions, 0 deletions — 4 new source modules, 1 new test module, 2 modified package export modules.
- **`889559d`:** 6 files, 2,748 insertions, 0 deletions — 1 new production module, 1 export change, 1 new fixtures module, 3 new test modules. No existing table, trigger, index, module or test is modified.

Infrastructure resolves the Application and Core as editable path dependencies, so local Infrastructure runs used the Application at `617b538` and Core at `29bd34a`.

## 5. Application implementation (#99)

Paths are under `northstar-application/`.

| Component | File | Responsibility |
| --- | --- | --- |
| Catalog port | `src/northstar_application/ports/equity_candle_evidence_snapshot_catalog.py` | `EquityCandleEvidenceSnapshotCatalog.list_snapshots(scope, monitoring_instant, limit)` (line 181); `…CatalogScope`; `…CatalogEntry`; `…CatalogUnavailableError`, `…CatalogLimitExceededError`, `InvalidEquityCandleEvidenceSnapshotCatalogError` (`INTEGRITY`, `SCOPE`, `METADATA`, `INCOMPLETE`) |
| Reader (V1) | `src/northstar_application/ports/equity_recommendation_lifecycle_reader.py` | `EquityRecommendationLifecycleReader` (line 22): runtime-checkable protocol with `load_lifecycle` only |
| Selection | `src/northstar_application/application_services/equity_lifecycle_monitoring_selection.py` | `EQUITY_LIFECYCLE_MONITORING_POLICY = "northstar.equity-lifecycle-monitoring-policy/1"` (line 57); `EQUITY_LIFECYCLE_MONITORING_CATALOG_LIMIT = 300` (line 60); `select_equity_lifecycle_snapshots` (line 140): eligibility, parsed ordering, same-instant groups, anchor, late-history diagnostic, watermark regression |
| Monitor | `src/northstar_application/application_services/monitor_equity_recommendation_lifecycles.py` | `MonitorEquityRecommendationLifecyclesUseCase` (line 346); scopes EXPLICIT, STRATEGY_LISTING, DECLARED_SESSION; 12 outcomes; EI-6C refusals; one recorder call per selected snapshot |
| Exports | `application_services/__init__.py`, `ports/__init__.py` | Exports only |
| Tests | `tests/application_services/test_monitor_equity_recommendation_lifecycles.py` | 49 test functions, 73 cases |

**Behaviour.** The monitor resolves the scope; loads each issuance and its stored disposition; derives the catalog scope (listing, trading date, `5m`, research-window version, SYNTHETIC); authenticates the anchor (the last event's cited snapshot, through `get_snapshot`); reads the catalog with the governed limit; re-verifies every entry against its authenticated snapshot; selects; and calls `RecordEquityRecommendationLifecycleUseCase.execute(issuance_id, (snapshot_id,))` once per selected snapshot. Every lifecycle write goes through the EI-6B recorder and store. No clock, scheduler, durable checkpoint or second state machine exists.

## 6. Infrastructure implementation (#69)

Paths are under `northstar-infrastructure/`.

| Component | File | Responsibility |
| --- | --- | --- |
| Adapter | `src/northstar_infrastructure/persistence/sqlite_equity_candle_evidence_snapshot_catalog.py` | `SQLiteEquityCandleEvidenceSnapshotCatalog` (line 350); `_listed` (191), `_verified` (230), `_failure` (314), `_connect` (327) |
| Export | `src/northstar_infrastructure/persistence/__init__.py` | `SQLiteEquityCandleEvidenceSnapshotCatalog` |
| Fixtures | `tests/equity_lifecycle_monitoring_fixtures.py` | Real composition of the merged monitor from the database path; TEST-ONLY `SpyRecorder`, `FirstAppendBarrierStore`; a real foreign-currency snapshot; spawned workers |
| Tests | `tests/test_sqlite_equity_candle_evidence_snapshot_catalog.py` | 51 functions, 85 cases (catalog, S and A) |
| | `tests/test_equity_lifecycle_monitoring_sqlite_acceptance.py` | 27 functions, 27 cases (monitor over real SQLite, S) |
| | `tests/test_equity_lifecycle_monitoring_sqlite_processes.py` | 5 functions, 5 cases (spawned and fresh processes, P) |

**Adapter design:**

- **Read only.** Connections use `mode=ro` (a missing file is not created) and `PRAGMA query_only = ON`, which is confirmed. The EI-2B connection path, which initializes its schema, is never used.
- **One listing transaction** (`BEGIN` … `ROLLBACK`): the evidence tables must exist; every row of the scope is read through the existing index `equity_candle_evidence_snapshots_scope` with six bound equality predicates; every row is checked (canonical instants, `issued_at >= as_of`, watermark, identity, fingerprint), whatever its eligibility; eligibility is decided on parsed instants; the limit counts eligible rows; overflow raises the typed limit error. No SQL `LIMIT` is used.
- **Verification.** Each eligible row is verified by the EI-2B verified decoding `_stored_snapshot`, reused unchanged, in its own short read transaction, and must restate the claim read in the listing view.
- **Answer.** Entries are built from the verified snapshots, ordered by parsed `as_of` then `snapshot_id`; same-instant rows are all returned.
- **Failures.** Unavailable (retryable): missing file or tables, lock or busy beyond five seconds, other SQLite errors. Invalid (permanent): `METADATA`, `INTEGRITY`, `INCOMPLETE`, including SQLite `SQLITE_CORRUPT` and `SQLITE_NOTADB`. Cleanup never replaces the reported failure.

## 7. Test results and CI

### 7.1 Local Windows results

All Windows 11; Python 3.13.15; SQLite 3.53.1; dependencies resolved by `uv`; no bytecode or cache written.

| Point | Revision | Result |
| --- | --- | --- |
| EI-6C Application module | Merged `617b538` (Core `29bd34a`) | **73 passed** |
| Full Application suite | Merged `617b538` | **4,799 passed**, 0 skipped |
| EI-6C Infrastructure modules | Pre-merge working tree identical to `889559d` (same six files and line counts), Application `617b538`, Core `29bd34a` | **117 passed** (85 + 27 + 5); the process module also passed three further consecutive runs (5 of 5 each) |
| Full Infrastructure suite | Same tree | **3,529 passed, 27 skipped**, 0 failed; 3,556 collected |
| Infrastructure baseline before PR-2 | `dd6a434` | 3,412 passed, 27 skipped |
| Ruff check and format; `git diff --check` | Both PR trees | Clean |
| Targeted mutations | Pre-merge trees | Application 12 of 12 caught; Infrastructure 16 of 19 caught, 3 equivalent survivors (section 7.5) |

**Windows skips (27), from the local `-rs` report:** 23 opt-in live Databento tests (`tests/test_live_databento_futures_acceptance.py`, module-level `skipif` on `NORTHSTAR_LIVE_DATABENTO`) and 4 parametrized cases of `tests/test_nse_futures_session.py::test_the_host_timezone_does_not_move_a_session` (`tzset is POSIX-only`). PR-2 added no skip.

**Not run against the merged Infrastructure commit:** the Infrastructure suites ran on the pre-merge working tree, whose files and line counts equal `889559d`.

### 7.2 Linux CI evidence

**Sources.**

- **Job metadata.** Run, job, check-run and annotation metadata were read on 2026-11-15 from the GitHub REST API without authentication. Job logs returned HTTP 403, and the job pages show "Sign in to view logs". No credential was used, and no run was repeated.
- **Pytest summaries.** The collected, passed, failed and skipped counts, the durations and the Python, pytest and pluggy versions are **owner-supplied GitHub Actions log excerpts**, provided on 2026-11-15 from the four job logs. They were not read by engineering directly. The `SKIPPED` lines and the dependency checkout steps were not supplied.

| Item | Application PR | Application post-merge | Infrastructure PR | Infrastructure post-merge |
| --- | --- | --- | --- | --- |
| Run / job | [`43670875275`](https://github.com/Northstar-Dhruba/northstar-application/actions/runs/43670875275/job/132680399843) / `132680399843` | [`43681457742`](https://github.com/Northstar-Dhruba/northstar-application/actions/runs/43681457742/job/132712777880) / `132712777880` | [`43690799893`](https://github.com/Northstar-Dhruba/northstar-infrastructure/actions/runs/43690799893/job/132740839995) / `132740839995` | [`43691231828`](https://github.com/Northstar-Dhruba/northstar-infrastructure/actions/runs/43691231828/job/132742128997) / `132742128997` |
| Event | `pull_request` | `push` (`develop`) | `pull_request` | `push` (`develop`) |
| Tested SHA | `e13beb0` (the #99 change commit) | `617b538` (the #99 merge; `develop` HEAD) | `889559d` (the #69 change commit) | `9d9c960` (the #69 merge; `develop` HEAD) |
| Workflow; job | Python CI (`python-ci.yml`); `test` | same | same | same |
| Runner label | `ubuntu-latest` | `ubuntu-latest` | `ubuntu-latest` | `ubuntu-latest` |
| Time (UTC) | 03:07:05–03:08:03 | 05:31:45–05:33:04 | 07:37:53–07:42:50 | 07:43:49–07:48:43 |
| Job conclusion | **SUCCESS** | **SUCCESS** | **SUCCESS** | **SUCCESS** |
| Steps | All 14 SUCCESS, including Run Ruff, Run Ruff Formatter Check, Run Tests | All SUCCESS | All 16 SUCCESS, including the three above | All SUCCESS |
| Python; pytest; pluggy | 3.13.18; 9.1.1; 1.6.0 (owner-supplied; installed by `uv python install 3.13`) | same | same | same |
| Dependency checkouts | Core `develop`; **SHA NOT VERIFIED** (inferred `29bd34a`) | same | Core and Application `develop`; **SHAs NOT VERIFIED** (inferred `29bd34a`, `617b538`) | same |
| Pytest collected | **4,799** | **4,799** | **3,556** | **3,556** |
| Passed / failed / skipped | **4,799 / 0 / 0** | **4,799 / 0 / 0** | **3,532 / 0 / 24** | **3,532 / 0 / 24** |
| Pytest duration | 48.34 s | 66.88 s | 287.07 s | 277.82 s |
| Skip reasons | None (0 skipped) | None (0 skipped) | Count verified; breakdown derived, not observed (below) | same |
| Annotations | not read | not read | Warning: `actions/checkout@v4` and `astral-sh/setup-uv@v5` target the deprecated Node.js 20 and ran on Node.js 24. Notice: `ubuntu-latest` migrates to Ubuntu 26 from 2026-10-19 | not read |

**What is established.**

- **Revisions.** Each tested SHA exists locally, and the post-merge SHAs equal the `develop` HEADs. Both the PR runs and the post-merge runs succeeded on the merged implementation.
- **Workflow.** `python-ci.yml` is the only workflow in each repository. It runs `ruff check`, `ruff format --check` and plain `pytest` (no pytest configuration exists in either repository).
- **The EI-6C modules cannot be skipped or deselected:** none of the four EI-6C test modules, the fixtures module or any `conftest.py` contains a skip, `xfail`, `importorskip` or `pytestmark`, and pytest exits non-zero on any failure or collection error.
- **The counts equal the local collections.** Application 4,799 and Infrastructure 3,556 collected on Linux, the same totals as the local Windows collections, which contain the 73 and 117 EI-6C cases.
- **Durations fit the jobs.** Each pytest duration lies within its job's wall-clock window above.
- **Dependency inference.** PR-1 merged at 05:31Z and the API reports no later Application commit; Core `develop` has been `29bd34a` since 2026-11-14. So the PR-2 runs checked out the merged PR-1, by timing. The checkout steps were not supplied, so this stays unconfirmed.

**Skip reconciliation (derived, not observed).** Only three skip sources exist in the Infrastructure tests:

- the module-level `skipif` of `tests/test_live_databento_futures_acceptance.py` (23 cases, opt-in `NORTHSTAR_LIVE_DATABENTO`);
- `tests/test_nse_equity_session_reference.py:267`, which skips when northstar-docs is not checked out beside it, as on CI (1 case);
- `tests/test_nse_futures_session.py:563` (`tzset is POSIX-only`), which runs on Linux (4 cases).

The derived Linux skips are 23 + 1 = 24, the verified count. Against Windows: 3,529 + 4 − 1 = 3,532 passed and 27 − 4 + 1 = 24 skipped, of the same 3,556. This matches EI-6B's recorded Linux reconciliation. The expectations recorded before the summaries were supplied (Application 4,799 with no skips; Infrastructure 3,532 passed and 24 skipped of 3,556) equal the supplied figures. **The breakdown is derived from the test declarations; the Linux `SKIPPED` lines were not supplied, and no direct observation of them is claimed.**

### 7.3 Acceptance gate EI6C-CI

**CLOSED — PASS, 2026-11-15.** The design's verification strategy requires independent Linux CI for each implementation PR before owner acceptance, with the P rows shown to execute on Linux. Job SUCCESS was verified from the REST API; the test-execution figures come from the owner-supplied log excerpts of section 7.2. The gate was OPEN when this record was first prepared, because the logs could not be read without authentication.

| Closing condition | Result | Basis |
| --- | --- | --- |
| 1. The Application runs show the EI-6C module's 73 cases passing with no failure | **Met** | Both jobs: 4,799 collected, 4,799 passed, 0 failed, 0 skipped. Every collected test passed, and the collection equals the local one containing the 73 cases. Per-module lines were not supplied |
| 2. The Infrastructure runs show the three EI-6C modules (117 cases) passing with no skip or failure | **Met** | Both jobs: 3,556 collected, 3,532 passed, 0 failed, 24 skipped. No skip source exists in the EI-6C modules, so none of the 24 skips falls in them; the collection equals the local one containing the 117 cases, including the five spawned-process and fresh-process (P) tests. Per-module lines were not supplied |
| 3. The Infrastructure skips reconcile with the declarations | **Met for the count** | 24 verified; the 23 + 1 breakdown is derived from the declarations, not observed |
| 4. The Infrastructure Application checkout contains `617b538` | **Not verified; accepted as a provenance limitation** | The checkout steps were not supplied; by timing the PR-2 runs checked out the merged PR-1 (L12) |

The classifications of section 8 stand unchanged and were accepted by the owner ([section 14.1](#141-final-owner-decision-2026-11-15)).

### 7.4 Concurrency and process evidence on Windows

See [section 10](#10-concurrency-and-process-reliability).

### 7.5 Targeted mutations

- **Application (PR-1), 12 of 12 caught:** each timestamp guard removed; lexical and reversed ordering; same-instant conflict ignored, in a group and at the anchor; regression allowed; prefix dropped; thinning; an unexpected error reported as `NO_NEW_EVENTS`; batching; a direct-write attempt.
- **Infrastructure (PR-2), 16 of 19 caught:** the `issued_at` guard; the `issued_at >= as_of` invariant; lexical ordering; twins deduplicated; a conflicting group reduced to one; SQL `LIMIT 300`; overflow not refused; the 301st row ignored; unverified metadata trusted; malformed rows suppressed; corrupt evidence reported as unavailable; `SQLITE_CORRUPT` reported as unavailable; a vanished row reported as unavailable; write access allowed; the table-presence check skipped; a negative watermark accepted.
- **Three equivalent survivors:**
  1. **`as_of <= T` removed:** implied by `issued_at <= T` together with the enforced `issued_at >= as_of` invariant, whose own mutant is caught.
  2. **Listing transaction removed:** the listing is a single data statement, which SQLite keeps on one view for its whole iteration.
  3. **Claimed watermark not restated:** the claim's watermark affects nothing that is returned, and `_stored_snapshot` already compares every stored column with the document.

Sources were restored byte for byte.

## 8. Acceptance evidence matrix

**Owner-accepted classification, 2026-11-15** (EI6C-FINAL-ACCEPTANCE, [section 14.1](#141-final-owner-decision-2026-11-15)). It is the engineering proposal unchanged, confirmed after gate EI6C-CI closed ([section 7.3](#73-acceptance-gate-ei6c-ci)). The planned-governance baseline, all 40 rows PLANNED, stays in the [design's section 12](../architecture/Equity-Intraday-V1-EI6C-Lifecycle-Monitoring-and-Snapshot-Selection-Design-v1.0.md#12-acceptance) and the [EI-5 Decision Register, section 22.6](Equity-Intraday-V1-EI5-Decision-Register.md#226-acceptance-scenarios).

| Status | Meaning |
| --- | --- |
| VERIFIED | Executed and passed locally and in the Linux CI jobs (section 7.3); no qualification |
| VERIFIED WITH LIMITATION | As VERIFIED, with a stated scope limitation |
| NOT TESTED / BLOCKED | Not used: no row falls there |

**Paths:**

- **A:** `northstar-application/tests/application_services/test_monitor_equity_recommendation_lifecycles.py`
- **CAT:** `northstar-infrastructure/tests/test_sqlite_equity_candle_evidence_snapshot_catalog.py`
- **INT:** `northstar-infrastructure/tests/test_equity_lifecycle_monitoring_sqlite_acceptance.py`
- **PROC:** `northstar-infrastructure/tests/test_equity_lifecycle_monitoring_sqlite_processes.py`

**Linux, for every row:** the A tests ran in the Application jobs `132680399843` and `132712777880` (4,799 of 4,799 passed in each); CAT, INT and PROC ran in the Infrastructure jobs `132740839995` and `132742128997` (3,532 passed, 24 skipped, 0 failed of 3,556 in each). That each row's tests ran on Linux is derived from these totals (section 7.3); per-test Linux lines were not supplied.

| ID | Requirement | Application (A) | Infrastructure (CAT, INT, PROC) | Accepted | Limitation |
| --- | --- | --- | --- | --- | --- |
| MS-01 | No eligible snapshot; nothing written | `test_ms01_no_eligible_snapshot_writes_nothing` | INT `test_ms01_no_eligible_candidate_records_nothing`; CAT `test_ms01_no_eligible_snapshot_is_an_empty_answer_and_the_file_is_untouched` | VERIFIED | — |
| MS-02 | Dual-timestamp eligibility on parsed instants; equality eligible | `test_ms02_ms40_eligibility_requires_both_instants_at_or_before_t`; `test_ms02_ms29_instants_compare_parsed_never_as_text` | CAT `test_ms02_ms35_ms40_both_guards_on_parsed_instants_and_equality_is_eligible`; `test_ms02_a_monitoring_instant_given_with_an_offset_is_the_same_instant`; `test_ms02_limitation_a_declared_issued_at_is_not_proof_of_availability` | **VERIFIED WITH LIMITATION** | Declared `issued_at`; no independent creation-time proof |
| MS-03 | Same-instant equivalence and conflicts | `test_ms03_equivalent_manifests_resolve_to_the_lowest_snapshot_id`; `test_ms03_a_same_instant_conflict_stops_after_the_earlier_prefix` | CAT `test_ms03_equivalent_snapshots_at_one_instant_are_all_listed`; `test_ms03_conflicting_manifests_at_one_instant_are_all_listed`; INT `test_ms36_ms37_ms28_late_evidence_after_an_event_never_rewrites_history` | VERIFIED | — |
| MS-04 | Conditional run-frequency equivalence | `test_ms04_conditional_equivalence_holds_for_stable_ordered_prefixes` | INT `test_ms04_split_runs_equal_one_later_run_when_prefixes_are_stable` | VERIFIED | Conditional by definition |
| MS-05 | Identical rerun writes nothing | `test_ms05_ms06_reruns_and_an_earlier_instant_write_nothing` | INT `test_ms05_ms06_a_rerun_and_a_lower_instant_write_nothing` | VERIFIED | — |
| MS-06 | Smaller-T run writes nothing | `test_ms05_ms06_reruns_and_an_earlier_instant_write_nothing` | INT `test_ms05_ms06_a_rerun_and_a_lower_instant_write_nothing` | VERIFIED | — |
| MS-07 | Quiet evaluation; no checkpoint | `test_ms07_a_quiet_evaluation_writes_nothing_and_keeps_no_cursor` | INT `test_ms07_quiet_evaluations_leave_no_row_cursor_or_checkpoint` | VERIFIED | — |
| MS-08 | Event path to a terminal state | `test_ms08_ms38_progression_to_terminal_one_snapshot_per_recorder_call` | INT `test_ms08_ms38_monitoring_reaches_a_terminal_state_one_snapshot_per_recorder_call` | VERIFIED | — |
| MS-09 | O-2 option A; EI5-D7 at both boundaries | `test_ms09_ms10_exit_boundary_no_allowance_and_the_exact_boundary_rule`; `test_ms09_entry_boundary_without_activation` | INT `test_ms09_ms10_ms12_the_entry_window_boundary_blocks_exactly_then_is_terminal`; `test_ms09_the_modeled_exit_boundary_blocks_exactly_then_is_terminal` | VERIFIED | — |
| MS-10 | No artificial deferral; O-3 | `test_ms09_ms10_exit_boundary_no_allowance_and_the_exact_boundary_rule`; `test_ms15_ms10_late_gap_filling_evidence_after_incomplete_evidence_records_nothing` | INT `test_ms09_ms10_ms12_the_entry_window_boundary_blocks_exactly_then_is_terminal`; `test_ms15_ms10_late_gap_filling_after_incomplete_evidence_records_nothing` | VERIFIED | — |
| MS-11 | EI3-D10 option (a); EI-3C and EI-4 unchanged | The full Application suite, including the EI-3A, EI-3C and EI-4 modules, unchanged (4,799 passed) | Layer C only; no S part | VERIFIED | — |
| MS-12 | Deficient bar before the boundary blocks without an event | `test_ms09_ms10_exit_boundary_no_allowance_and_the_exact_boundary_rule` | INT `test_ms09_ms10_ms12_the_entry_window_boundary_blocks_exactly_then_is_terminal` | VERIFIED | — |
| MS-13 | Later-instant revision recorded once; no rewrite | `test_ms13_a_later_instant_correction_is_recorded_once_as_a_revision`; `test_ms28_ms13_a_later_same_instant_manifest_never_rewrites_recorded_history` | INT `test_ms13_ms14_a_later_instant_revision_after_a_terminal_state_is_recorded_once`; `test_ms36_ms37_ms28_late_evidence_after_an_event_never_rewrites_history` | VERIFIED | — |
| MS-14 | Terminal: Core-permitted revisions only | `test_ms14_a_terminal_lifecycle_records_only_core_permitted_revisions` | INT `test_ms14_a_terminal_lifecycle_records_no_state_change`; `test_ms13_ms14_a_later_instant_revision_after_a_terminal_state_is_recorded_once` | VERIFIED | — |
| MS-15 | O-3 option (b) through monitoring | `test_ms15_ms10_late_gap_filling_evidence_after_incomplete_evidence_records_nothing` | INT `test_ms15_ms10_late_gap_filling_after_incomplete_evidence_records_nothing` | VERIFIED | — |
| MS-16 | Concurrent monitors; one history; no required `EQUIVALENT` | `test_ms16_an_identical_competing_append_is_equivalent` | PROC `test_ms16_two_monitors_over_the_same_evidence_commit_one_history` | VERIFIED | — |
| MS-17 | Divergent monitors; first commit wins | `test_ms17_a_divergent_competing_append_is_a_conflict_and_the_winner_stays` | PROC `test_ms17_divergent_monitors_with_different_instants_and_views_first_commit_wins` | VERIFIED | — |
| MS-18 | Process ending before COMMIT leaves nothing | — | PROC `test_ms18_a_monitor_process_ending_before_commit_leaves_nothing_and_a_later_run_completes` | VERIFIED | — |
| MS-19 | Commit uncertainty through the recorder | `test_store_unavailable_and_unresolved_commits_stay_typed_with_no_blind_retry`; `test_a_lost_acknowledgement_is_reconciled_by_the_recorder` | INT `test_ms19_a_lost_acknowledgement_reconciles_to_equivalent_exactly_once`; `test_ms19_unresolved_commit_uncertainty_is_reported_never_retried_by_the_monitor`; `test_h2_a_held_write_lock_is_a_typed_retryable_refusal_and_nothing_is_written` | VERIFIED | — |
| MS-20 | Corrupt issuance or lifecycle authority typed | `test_ms20_corrupt_lifecycle_history_is_a_permanent_typed_refusal`; `test_corrupt_or_unavailable_issuance_authority_is_typed` | INT `test_ms20_ms23_corrupt_lifecycle_authority_is_typed_for_that_issuance_only` | **VERIFIED WITH LIMITATION** | The SQLite monitor composition exercises corrupt *lifecycle* authority only; corrupt issuance authority is evidenced at layer C and by EI-6A's accepted adapter tests |
| MS-21 | Scope never crossed; venue mismatch is a typed refusal | `test_ms21_ms25_ms35_a_catalog_answer_out_of_scope_or_ineligible_is_refused`; `test_ms21_ms38_a_real_snapshot_refusal_keeps_the_committed_prefix`; `test_snapshots_outside_the_scope_are_never_candidates` | CAT `test_ms21_ms25_each_scope_column_is_filtered_in_sql`; `test_ms21_the_same_clone_within_the_scope_is_listed_and_fails_verification`; `test_ms21_another_listing_in_the_same_file_is_never_listed`; `test_ms21_ms25_a_scope_no_snapshot_matches_lists_nothing`; INT `test_ms21_ms38_a_later_ei5c_refusal_keeps_the_committed_prefix` | **VERIFIED WITH LIMITATION** | SQLite venue mismatch not exercised directly; a real currency mismatch (`SNAPSHOT_WINDOW_MISMATCH`) proves the refusal path |
| MS-22 | Fresh process reproduces selection and outcomes | `test_ms22_identical_inputs_give_identical_selection_and_outcomes` | CAT `test_ms22_repeated_calls_and_new_instances_give_identical_answers`; PROC `test_ms22_a_fresh_process_reproduces_the_selection_and_outcomes_from_the_file` | VERIFIED | Past quiet runs and policy identity not reconstructed, as the requirement states |
| MS-23 | Issuances isolated | `test_ms32_explicit_ids_are_normalized_and_an_unknown_one_is_isolated` | INT `test_ms20_ms23_corrupt_lifecycle_authority_is_typed_for_that_issuance_only`; `test_g4_catalog_corruption_of_one_listing_never_suppresses_another` | VERIFIED | — |
| MS-24 | Architecture | `test_ms24_the_monitoring_modules_read_no_clock_and_reach_no_infrastructure`; `test_ms24_the_monitor_never_writes_lifecycles_itself`; `test_ms24_selection_is_pure` | CAT `test_ms24_the_adapter_implements_the_application_port_and_nothing_else`; `test_ms24_the_catalog_module_is_read_only_by_construction`; `test_ms24_construction_opens_nothing_and_starts_nothing`; `test_j_the_catalog_connection_cannot_write_or_change_the_schema`; INT `test_ms24_monitoring_writes_lifecycle_rows_only_through_the_ei6b_append` | VERIFIED | — |
| MS-25 | Non-SYNTHETIC snapshots never selected | `test_ms21_ms25_ms35_a_catalog_answer_out_of_scope_or_ineligible_is_refused` | CAT `test_ms21_ms25_each_scope_column_is_filtered_in_sql`; `test_ms21_ms25_a_scope_no_snapshot_matches_lists_nothing`; INT `test_ms25_a_non_synthetic_row_is_never_selected` | VERIFIED | — |
| MS-26 | No regression in the EI-5, EI-6A and EI-6B suites | Full Application suite (4,799 passed on Windows and in both Linux jobs) | Full Infrastructure suite (3,529 passed, 27 skipped on Windows; 3,532 passed, 24 skipped in both Linux jobs); PROC module | VERIFIED | — |
| MS-27 | Watermark regression | `test_ms27_a_watermark_regression_stops_selection` | INT `test_ms27_watermark_regression_against_selected_evidence_and_the_anchor` | VERIFIED | — |
| MS-28 | Changing-catalog path dependence | `test_ms28_ms13_a_later_same_instant_manifest_never_rewrites_recorded_history` | INT `test_ms36_ms37_ms28_late_evidence_after_an_event_never_rewrites_history`; `test_ms34_ms36_a_fixed_catalog_with_unstable_eligibility_is_path_dependent` | VERIFIED | Accepted path dependence |
| MS-29 | Parsed-instant ordering | `test_ms02_ms29_instants_compare_parsed_never_as_text` | CAT `test_ms29_fractional_seconds_are_ordered_chronologically_not_as_text`; `test_ms30_ms29_the_limit_holds_across_mixed_textual_instants` | VERIFIED | — |
| MS-30 | Typed overflow; limit fixed and verified | `test_ms30_candidate_overflow_is_a_typed_refusal_and_nothing_is_recorded`; `test_ms30_a_catalog_returning_more_than_the_limit_is_refused` | CAT `test_ms30_up_to_the_limit_every_eligible_snapshot_is_listed` (0, 1, 299, 300); `test_ms30_one_eligible_snapshot_beyond_the_limit_is_refused_never_truncated`; `test_ms30_the_limit_counts_eligible_rows_not_physical_rows`; `test_ms30_ms29_the_limit_holds_across_mixed_textual_instants`; `test_ms30_overflow_on_the_final_same_instant_group_returns_nothing`; `test_ms30_the_limit_must_be_a_positive_integer`; INT `test_ms30_300_eligible_proceed_and_301_refuse_with_no_recorder_call` | VERIFIED | Near-limit cost (section 9; I2-1) |
| MS-31 | One fixed policy | `test_ms31_one_fixed_policy_with_no_selectable_allowance_or_limit` | INT `test_ms31_the_report_names_the_one_fixed_policy` | VERIFIED | Not durable (EI6C-P12) |
| MS-32 | Three scope modes; an undeclared issuance | `test_ms32_explicit_ids_are_normalized_and_an_unknown_one_is_isolated`; `test_ms32_strategy_listing_finds_an_issuance_not_yet_declared`; `test_ms32_a_declared_session_lists_an_issuance_once_a_declaration_covers_it` | INT `test_ms32_the_three_scope_modes_against_the_persisted_authority` | VERIFIED | DECLARED_SESSION incomplete by design (EI6C-P7) |
| MS-33 | Earlier EI-6B history stays authoritative | `test_ms33_a_history_recorded_before_monitoring_stays_authoritative` | INT `test_ms33_history_recorded_directly_through_ei6b_stays_authoritative` | VERIFIED | — |
| MS-34 | Fixed catalog, unstable eligibility | `test_ms34_ms36_a_fixed_catalog_with_unstable_eligibility_is_path_dependent` | INT `test_ms34_ms36_a_fixed_catalog_with_unstable_eligibility_is_path_dependent` | VERIFIED | Accepted path dependence |
| MS-35 | `issued_at <= T` guard required by regression coverage | `test_ms21_ms25_ms35_a_catalog_answer_out_of_scope_or_ineligible_is_refused`, with the PR-1 `issued_at` mutant caught | CAT `test_ms02_ms35_ms40_both_guards_on_parsed_instants_and_equality_is_eligible`, with the PR-2 `issued_at` mutant caught | VERIFIED | — |
| MS-36 | Late historical snapshot diagnosed, not evaluated | `test_ms36_ms37_ms28_the_anchor_bounds_history_and_is_never_reinterpreted`; `test_ms34_ms36_a_fixed_catalog_with_unstable_eligibility_is_path_dependent` | INT `test_ms34_ms36_a_fixed_catalog_with_unstable_eligibility_is_path_dependent`; `test_ms36_ms37_ms28_late_evidence_after_an_event_never_rewrites_history` | **VERIFIED WITH LIMITATION** | Best-effort, non-durable diagnostic based on declared issuance chronology (V2) |
| MS-37 | Lower-ID equivalent causes no rewrite | `test_ms37_a_later_equivalent_snapshot_never_rewrites_the_cited_reference` | INT `test_ms36_ms37_ms28_late_evidence_after_an_event_never_rewrites_history` (a twin with a lower identity) | VERIFIED | — |
| MS-38 | Prefix preserved after an EI-5C refusal | `test_ms21_ms38_a_real_snapshot_refusal_keeps_the_committed_prefix`; `test_ms08_ms38_progression_to_terminal_one_snapshot_per_recorder_call` | INT `test_ms21_ms38_a_later_ei5c_refusal_keeps_the_committed_prefix`; `test_ms08_ms38_monitoring_reaches_a_terminal_state_one_snapshot_per_recorder_call` | VERIFIED | — |
| MS-39 | Unexpected failure fails closed; prefix kept | `test_ms39_an_unexpected_failure_fails_the_issuance_closed_and_keeps_the_prefix`; `test_ms39_a_process_signal_is_never_swallowed` | INT `test_ms39_an_unexpected_failure_fails_the_issuance_closed_and_keeps_the_prefix` | VERIFIED | The SQLite case injects the failure; real Core and append-verification errors are exercised at layer C |
| MS-40 | Future-declared snapshot excluded; equality selected | `test_ms40_a_snapshot_declaring_issuance_after_t_is_excluded_until_t_reaches_it` | INT `test_ms40_a_future_declared_snapshot_is_selected_only_from_its_declared_issuance`; CAT `test_ms02_ms35_ms40_both_guards_on_parsed_instants_and_equality_is_eligible` | VERIFIED | — |

| Count | Value |
| --- | --- |
| Scenario rows | 40 (MS-01 to MS-40, no gaps or duplicates) |
| VERIFIED | 36 |
| VERIFIED WITH LIMITATION | 4 (MS-02, MS-20, MS-21, MS-36) |
| NOT TESTED; BLOCKED | 0; 0 |

The rows' limitations noted as "accepted path dependence", "conditional by definition" or "not durable" restate the governed requirement itself. They do not qualify the classification.

## 9. Performance evidence

These are local synthetic measurements, not provider or production throughput figures.

- **Environment:** Windows 11 build 26300; Intel family 6 model 154; Python 3.13.15; SQLite 3.53.1.
- **Fixture:** the EI-6B TCS world with quiet final bars through 15:00 and real EI-2B snapshots.
- **Harness:** a session benchmark script, not committed to any repository.
- **Repetitions:** 3 for the three smaller cases, 2 for the larger ones.
- **Values:** seconds; medians, with total ranges in brackets.

| Case: eligible catalog entries (selected snapshots = recorder calls) | Listing | Catalog verification | Monitor re-verification | Recorder | Total, first run | Quiet rerun (calls) |
| --- | --- | --- | --- | --- | --- | --- |
| 6 (4) | 0.003 | 0.20 | 0.19 | 0.63 | 1.08 [1.06–1.10] | 0.83 [0.78–1.00] (3) |
| 66 (64), one per 5-minute bar | 0.003 | 2.85 | 3.13 | 9.87 | 15.89 [15.62–19.86] | 13.85 [12.92–21.74] (49) |
| 123 (121), four per interval | 0.008 | 8.53 | 9.47 | 31.77 | 50.00 [47.89–54.16] | 36.28 [29.48–42.06] (67) |
| 290 (288) | 0.012 | 25.59 | 26.29 | 82.16 | 134.12 [127.80–140.44] | 115.46 [114.15–116.76] (222) |
| **300 (298)** | 0.014 | 24.61 | 25.69 | 107.45 | **157.84 [146.79–168.90]** | **127.91 [117.52–138.30] (232)** |
| 301 | 0.014 | — | — | — | 0.08 [0.08–0.08]: typed `CATALOG_LIMIT_EXCEEDED`, no recorder call | 0.08 |

How the counts relate:

- **Eligible versus selected.** Each case's eligible count includes the research snapshots at 09:36 and 09:41. They precede the 09:43:30 issuance, so the selection discards them.
- **Committed events.** In the 290 and 300 cases each first run reached `EXPIRED_BEFORE_ENTRY`. That implies two committed events, `ENTRY_WINDOW_OPENED` and `ENTRY_WINDOW_EXPIRED`, inferred from the final state and the quiet-call arithmetic; event counts were not recorded by the harness.
- **Quiet reruns** re-evaluate every snapshot after the last event: 298 − 66 = 232, and 288 − 66 = 222.

**Observations:**

- EI-6B recorder calls dominate, at about 68% of the time and about 0.36 s per call near the limit.
- The catalog's verification and the monitor's re-verification each take about 16%. Both are retained by I2-2.
- The listing transaction held its shared lock for at most 17 ms.
- A quiet rerun costs about 80% of a first run, because there is no checkpoint (K1).

**Disposition (I2-1).** The fixed limit of 300 stays as owner-approved (V3). The measured near-limit cost is accepted for synthetic v1. No thinning, batching or new checkpoint is authorized.

## 10. Concurrency and process reliability

All run on Windows and, per the Linux totals of section 7.3, in the four Linux CI jobs.

| Property | Evidence |
| --- | --- |
| One consistent listing view | CAT `test_e1_no_write_can_commit_inside_the_listing_view` (a writer is refused while the view holds its shared lock); `test_e1_a_snapshot_committed_after_the_listing_view_is_not_in_that_answer`; `test_e3_a_candidate_set_growing_past_the_limit_concurrently` (300, then refused, never 300 of 301) |
| No dirty or partial reads | CAT `test_i6_an_uncommitted_or_rolled_back_snapshot_row_is_never_listed`; PROC `test_e3_i6_readers_in_other_processes_see_only_whole_committed_states` |
| A listed row that vanishes | CAT `test_e2_a_listed_row_that_vanishes_before_verification_is_incomplete` |
| Journal mode | CAT `test_e1_the_database_runs_the_rollback_journal_the_guarantees_were_verified_under` (`delete`) |
| Concurrent monitors | PROC `test_ms16_…`, `test_ms17_…` (spawn start method; barriers; bounded timeouts; no sleep as an oracle) |
| Termination before COMMIT | PROC `test_ms18_…` (exit code 17 shows the COMMIT seam was reached) |
| Fresh interpreter and restart | PROC `test_ms22_…`; INT `test_ms21_ms38_…` (restart from the file alone) |
| Commit uncertainty and lock contention | INT `test_ms19_…` ×2; `test_h2_a_held_write_lock_is_a_typed_retryable_refusal_and_nothing_is_written` |
| SQLite failures | CAT: missing file (never created), missing tables, locked database, read-only file, damaged page and non-database file (`INCOMPLETE`), read failure, interrupt, cleanup failure, close failure |

**Guarantee across connections.** The set, count and claims of an answer come from one read transaction. The monitor's later `get_snapshot` calls use their own connections and re-read the same append-only rows; a changed row fails closed. No single end-to-end view across connections is claimed.

## 11. Limitations

### 11.1 Scenario level

| Scenario | Limitation |
| --- | --- |
| MS-02 | `issued_at` is declared by the issuing caller; there is no independent creation-time proof |
| MS-20 | The SQLite monitor composition does not exercise corrupt issuance authority; the Application tests and EI-6A's accepted adapter tests do |
| MS-21 | SQLite venue mismatch is not exercised directly (only the production venue-day reference source exists); a real currency mismatch proves the refusal path |
| MS-36 | Best-effort, non-durable late-historical diagnostic based on declared issuance chronology (V2) |

### 11.2 Architectural and operational (accepted, not resolved)

The owner accepted these on 2026-11-15 (section 14.1) without treating any of them as corrected.

| # | Limitation | Classification |
| --- | --- | --- |
| L1 | **Caller-declared issuance timestamps,** not trusted creation time; backdating is undetectable | Accepted (EI6C-P4) |
| L2 | **Monitoring path dependence:** history depends on which snapshots existed at each run (MS-28, MS-34) | Accepted (design section 10) |
| L3 | **Non-durable selection-policy provenance** | Accepted (EI6C-P12) |
| L4 | **Expensive quiet reruns:** about 128 s at the limit; every snapshot after the last event is re-evaluated | Accepted (K1; I2-1) |
| L5 | **Scope and timestamp metadata trust boundary.** Completeness assumes trustworthy stored scope and instant columns. EI-2B `issue` writes them from the snapshot itself. An out-of-contract INSERT can mis-scope a new row without dropping a trigger, because no insert guard ties the columns to the document. Altering an existing row needs triggers removed. A hidden row is not discovered unless it is the anchor | Accepted trust boundary |
| L6 | **Corrupt anchor misclassified.** EI-2B `get_snapshot` raises `EquityCandleEvidenceStorageError`, a subclass of `EquityCandleEvidenceUnavailableError`, for corrupt evidence. The monitor's `_authenticated` maps it to retryable `SNAPSHOT_STORE_UNAVAILABLE`. It fails closed before the catalog is read or any recorder call is made, so nothing is written. Untested on SQLite | Accepted (I2-3) |
| L7 | **No single end-to-end view** across separate SQLite connections (section 10) | Accepted |
| L8 | **Recovery of a hot rollback journal** after a crashed writer is untested; a `mode=ro` reader cannot perform it and would report unavailability until a read-write connection does | Verification gap |
| L9 | **Single-host SQLite authority** | Accepted |
| L10 | **Incomplete DECLARED_SESSION visibility** for issuances not yet covered by a declaration | Accepted (EI6C-P7) |
| L11 | **Existing EI-6B limitations** E1 to E9 and the carried-forward C1 to C10 ([EI-6B record, section 16](Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Engineering-Acceptance-Record.md#16-limitations-and-verification-gaps)) | Inherited, unchanged |
| L12 | **CI drift and dependency provenance:** `uv python install 3.13` (patch unpinned; 3.13.18 in all four jobs); dependencies checked out at `develop`, not pinned SHAs, and the checked-out Core and Application commits of the four jobs are not verified; `actions/checkout@v4` and `setup-uv@v5` on deprecated Node.js 20; `ubuntu-latest` moving to Ubuntu 26 | Operational risk; provenance limitation of gate EI6C-CI |
| L13 | **No real-market readiness or provider finality guarantee;** O-2 option A may produce a premature synthetic terminal `INCOMPLETE_EVIDENCE` under modeled delayed finality | Accepted (EI6C-P2) |

**Further observations (not limitations):**

- `RecordEquityRecommendationLifecycleUseCase._load` does not catch `get_snapshot` errors. A corrupt snapshot reaching the recorder becomes `ISSUANCE_EVALUATION_FAILED`, which still fails closed.
- `EquityLifecycleMonitoringScope` is not re-exported from the Application package.
- A rerun after a terminal state reports `NO_ELIGIBLE_SNAPSHOT`, the merged PR-1 behaviour.

## 12. Follow-up register

**Acceptance-blocking evidence:** none. Gate EI6C-CI closed on 2026-11-15 (section 7.3).

**Accepted, deferred reliability work.** None is authorized, including by the owner acceptance of section 14.1; each needs its own owner decision.

| # | Follow-up | Boundary |
| --- | --- | --- |
| F1 | Separate corruption from unavailability in snapshot loading | A subtype of `EquityCandleEvidenceUnavailableError` in the Application port, raised only in EI-2B `get_snapshot`'s decode and evidence branch; the monitor's `_authenticated` maps it to a permanent reason. EI-2B error contract change; needs governance |
| F2 | Corrupt-anchor SQLite test | Infrastructure tests |
| F3 | Full MS-20 corrupt-issuance integration test through the monitor on SQLite | Infrastructure tests |
| F4 | Hot rollback-journal recovery test | Infrastructure tests |
| F5 | Venue-mismatch SQLite test once a second applicable venue-day reference source exists | Infrastructure tests |
| F6 | Optional schema-enforced consistency of scope and instant columns with the snapshot document | EI-2B schema change; needs governance |
| F7 | Evidence-based verification performance work (for example, avoiding the duplicated candidate verification, currently retained by I2-2) | Needs governance |
| F8 | CI tooling and dependency pinning: Node.js 24 actions, runner image, Python patch, dependency SHAs, and recording the checked-out dependency commits | Repository CI configuration |
| F9 | DECLARED_SESSION completeness | Needs governance (no list-all interface is authorized) |
| F10 | Carried-forward EI-6B reliability items | EI-6B record, section 16 |

## 13. EI4-C4

EI-4 formal CI closure (**EI4-C4**) stays **PENDING** as a separate, outstanding obligation; the owner acceptance of EI-6C does not close it. EI-6C's CI runs do not close it unless they meet the [EI-4 Decision Register, section 15.3](Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4).

## 14. Engineering verdict and milestone status

**EI-6C ENGINEERING ACCEPTANCE: PASS WITH DOCUMENTED LIMITATIONS — synthetic, local Windows Application-contract, real-SQLite and separate-process acceptance; Linux CI job SUCCESS and pytest summaries recorded (gate EI6C-CI CLOSED — PASS).**

| Milestone | Status | Evidence |
| --- | --- | --- |
| 1. Governance approved | **DONE** | EI-5 Decision Register, section 22 |
| 2. Implementation merged | **DONE** | #99 (`617b538`), #69 (`9d9c960`) |
| 3. Local Windows acceptance | **DONE** | Sections 7.1 and 8 |
| 4. Linux CI job conclusions | **SUCCESS** (4 of 4) | Section 7.2 |
| 5. Linux test-execution figures (gate EI6C-CI) | **CLOSED — PASS** (2026-11-15) | Sections 7.2 and 7.3 |
| 6. Limitations reconciled | **DONE** | Section 11 |
| 7. Final owner decision | **APPROVED: ACCEPTED WITH DOCUMENTED LIMITATIONS** (2026-11-15) | Section 14.1 |
| 8. Documentation closure | **DONE** in this Docs PR-3 package | This record; EI-5 Decision Register, section 23 |

The engineering recommendation — record the four Linux pytest summaries, confirm that they reconcile, then accept EI-6C with documented limitations — was followed and adopted by the owner on 2026-11-15.

### 14.1 Final owner decision (2026-11-15)

**OWNER APPROVED — 2026-11-15.**

**EI-6C — ACCEPTED WITH DOCUMENTED LIMITATIONS.**

| Item | Value |
| --- | --- |
| Decision ID | **EI6C-FINAL-ACCEPTANCE** |
| Date | 2026-11-15 |
| Status | **APPROVED** |
| Verdict | **ACCEPTED WITH DOCUMENTED LIMITATIONS** |
| Source | The owner's Docs PR-3 finalization instruction, 2026-11-15 |
| Scope accepted | Synthetic-only, MODELED-only EI-6C lifecycle monitoring and snapshot selection: the read-only snapshot catalog port and SQLite adapter; dual-timestamp eligibility; parsed-instant selection; the monitor's three scope modes, outcomes and refusals; one snapshot per EI-6B recorder call; the fixed policy identity and the catalog limit of 300; single-host SQLite |
| Implementation | EI-6C Docs governance northstar-docs #69 (`372e0ff`); Application northstar-application #99 (`617b538`); Infrastructure northstar-infrastructure #69 (`9d9c960`); all merged |
| Gate EI6C-CI | **CLOSED — PASS** (section 7.3), on owner-supplied GitHub Actions log excerpts for jobs `132680399843`, `132712777880`, `132740839995` and `132742128997` |
| Acceptance matrix (40) | **36 VERIFIED, 4 VERIFIED WITH LIMITATION, 0 NOT TESTED, 0 BLOCKED** (section 8) |
| Accepted scenario limitations | **MS-02:** the declared issuance timestamp does not independently prove creation time. **MS-20:** corrupt issuance authority lacks end-to-end SQLite monitoring-composition evidence. **MS-21:** venue mismatch was not directly exercised in real-SQLite monitoring integration. **MS-36:** the late-history diagnostic is best-effort and non-durable |
| Other accepted limitations | L1 to L13 (section 11.2) and the synthetic performance evidence (section 9: about 158 s first run and 128 s quiet rerun at 300 eligible; typed refusal at 301), accepted without being treated as corrected. The limit stays 300; no batching, thinning or new checkpoint is approved |
| Not authorized | The follow-ups F1 to F10 (section 12) and everything in section 15 |
| Separate obligation | EI4-C4 stays outstanding (section 13) |

## 15. Non-authorizations

Neither this record nor the owner acceptance of section 14.1 authorizes anything beyond the accepted scope. EI-6C is synthetic-only and monitors existing authoritative MODELED issuances. The existing EI-6A synthetic MODELED issuance authority is not revoked. Nothing here authorizes, or may be read as authorizing:

- new issuance functionality, or research, generation, declaration, issuance or snapshot-creation orchestration;
- ACTUAL issuance, actionable recommendations, broker integration or execution, orders, fills, positions or real-money trading;
- real or live market data, real provider data or integration, or provider-based finality allowances;
- an automated monitoring scheduler, background polling or automated monitoring runs;
- API or Web integration, composition roots, production deployment or operational use;
- EI-7, EI-8 restatement, EI-9 or any later milestone;
- terminal-state reopening, terminal or historical lifecycle rewriting, Core changes, schema changes, or changes to the EI-2B, EI-6A or EI-6B contracts;
- the deferred follow-ups F1 to F10 (section 12), each of which needs its own owner decision;
- any relaxation of EI4-D8;
- multi-host durability.

It does not close EI-4 (EI4-C4 stays outstanding) and does not reopen EI-6A or EI-6B. EI-6C remains single-host, MODELED-only and synthetic-evidence-only.

## 16. Related documents

- [EI-5 Decision Register](Equity-Intraday-V1-EI5-Decision-Register.md), sections 22 and 23
- [EI-6C Lifecycle Monitoring and Snapshot Selection Design v1.0](../architecture/Equity-Intraday-V1-EI6C-Lifecycle-Monitoring-and-Snapshot-Selection-Design-v1.0.md)
- [EI-3 Decision Register, section 12](Equity-Intraday-V1-EI3-Decision-Register.md#12-ei3-d10-owner-decision-2026-11-15)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design v1.0](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [EI-6B Durable Recommendation Lifecycle Persistence Engineering Acceptance Record](Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Engineering-Acceptance-Record.md)
- [EI-6B Durable Recommendation Lifecycle Persistence Design v1.0](../architecture/Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md)
- [EI-6A Durable Issuance Authority Engineering Acceptance Record](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md)
- [EI-4 Decision Register, section 15.3](Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4)
