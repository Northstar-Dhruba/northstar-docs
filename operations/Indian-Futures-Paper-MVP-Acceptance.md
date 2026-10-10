# Indian Futures Paper-Trading MVP: Acceptance Specification

Status: **draft for review** (written 2026-10-10). Nothing in this document is
approved until the sign-off in section 15 is recorded.

This specification defines when the existing India NIFTY Futures paper-trading
MVP is complete. It describes behaviour that already exists in code; it adds
no trading rule. Where the code defines no behaviour, the gap is recorded as an
open policy decision (section 13), not filled in here.

Related documents:

- [Indian Futures Deployment Runbook](Indian-Futures-Deployment-Runbook.md) --
  how the stack is deployed and operated.
- [Indian Futures Paper MVP Evidence Register](Indian-Futures-Paper-MVP-Evidence-Register.md)
  -- the evidence cited below as `E-n`.
- [Upstox Finality Evidence](Upstox-Finality-Evidence.md) -- candle revision
  evidence collection.
- [ADR-008](../adr/ADR-008-Futures-Broker-Demo-Execution-Boundary.md) (broker
  demo boundary) and [ADR-009](../adr/ADR-009-Contract-Scoped-Futures-Economics.md)
  (contract-scoped economics).

Source paths below are relative to the Northstar workspace root
(`northstar-core`, `northstar-application`, `northstar-infrastructure`,
`northstar-api`).

## 0. Status and evidence definitions

| Status | Meaning |
|--------|---------|
| PASS | The criterion is met by inspected implementation **and** passing automated tests. Natural production evidence is listed where it exists but is not required for PASS unless the criterion says so. |
| PENDING | Not yet demonstrated. Includes criteria that need natural production evidence, an operational drill, a deployment, or an artifact that has only been reported verbally. |
| BLOCKED | Cannot progress until a recorded decision is made (an open policy decision or an unresolved gate). |
| DEFERRED | Deliberately outside the MVP, subject to the approval of section 14. |

| Evidence type | Meaning |
|---------------|---------|
| Automated test | A named test that exists in the repositories. All automated tests use synthetic data: seeded prices through a fake Upstox transport and the real NSE calendar. None uses real Upstox data. |
| Natural evidence | An observation from the production paper deployment on real market data. Unless the evidence register says an artifact was inspected, it is **operator-reported**. |

## 1. Objective and scope

The MVP operates one dated NIFTY futures contract in **internal paper mode** on
real Upstox daily candles:

- deterministic daily research produces BUY, SELL or HOLD;
- each decision is frozen and reproducible;
- a frozen decision becomes a paper order by a fixed target-exposure rule;
- paper orders fill at the next eligible session's OPEN;
- the portfolio reports positions and **gross** INR P&L;
- daily operation is chronological, idempotent and gated by operator-approved
  candle finality;
- a fixed pre-expiry flatten keeps the contract out of its expiry;
- a read-only dashboard and an hourly Windows scheduled task make it
  operable.

Out of scope: see section 14. In the agreed roadmap the MVP is followed
directly by broker demo integration (Phase 2) and 30 sessions of demo forward
validation (Phase 3); neither is part of this acceptance.

## 2. Supported market, instrument and strategy

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| S-1 | One exact dated contract is operated: product, exchange and expiration from configuration (October contract: NIFTY / NSE / 2026-10-27). | `northstar-api/src/northstar_api/settings.py`; Upstox resolution `northstar-infrastructure/.../market_data/upstox_instrument_master.py` `resolve_future_in_master` | `northstar-infrastructure/tests/test_upstox_futures_native_daily_market_data.py::test_the_exact_nifty_expiry_resolves_exactly_one_future` | Production operates NIFTY / NSE / 2026-10-27 (operator-reported, E-1) | PASS | none |
| S-2 | Valuation uses the contract's own economics: 65 INR per quote point per contract, never another expiry's. | `FuturesContractEconomics` (`northstar-core/.../futures/value_objects/futures_contract_economics.py`); `futures_contract_economics` table | `northstar-api/tests/test_india6_nifty_baseline_acceptance.py::test_economics_are_65_inr_for_the_exact_contract` | INR valuation observed (operator-reported, E-3) | PASS | none |
| S-3 | One built-in strategy, labelled `directional-mvp-v1-nifty-paper-ops`, one portfolio per contract (`nifty-paper-ops-oct26`), target exposure N = 1 contract. | `select_directional_signal` (`northstar-core/src/northstar_core/strategy/_directional_signal.py`); `NORTHSTAR_TARGET` in `settings.py` | `northstar-api/tests/test_india7_nifty_incremental_operations_acceptance.py::test_the_synthetic_paths_produce_the_documented_decisions` | Production identities (operator-reported, E-1) | PASS | none |

## 3. Research and frozen decisions

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| R-1 | Research produces BUY, SELL or HOLD from the latest 20 daily bars: BUY when the close rose, the 5-bar average exceeds the 20-bar average and volume is at least its 20-bar average; SELL on the mirror image; otherwise HOLD. Fewer than 20 bars decide nothing. | `select_directional_signal`; `Strategy._select_action` (`northstar-core/.../strategy/strategy.py`); `AnalyzeFuturesReplaySnapshotService` (`northstar-application/.../analyze_futures_replay_snapshot.py`) | `northstar-core/tests/strategy/test_directional_signal.py::test_rising_confirmed_trend_with_volume_is_strong_bullish` and siblings | October 7 SELL (operator-reported, E-1) | PASS | none |
| R-2 | A decision is frozen once and never rewritten: an equal retry is a no-op; a different decision for the same contract, strategy and instant is a conflict (exit 5). | `FreezeFuturesForwardResearchDecisionUseCase` (`northstar-application/.../freeze_futures_forward_research_decision.py`); insert-only `futures_forward_research_records` | `northstar-application/tests/application_services/test_freeze_futures_forward_research_decision.py::test_changed_evidence_at_the_same_instant_conflicts_rather_than_replacing` | Frozen October 8 decisions observed (operator-reported, E-3) | PASS | none |
| R-3 | No look-ahead: a decision uses only bars at or before its cutoff; later bars never change it. No clock decides. | Replay ends at `as_of` in the freeze use case | `test_freeze_futures_forward_research_decision.py::test_bars_appended_after_as_of_cannot_change_the_decision` | -- | PASS | none |
| R-4 | Missing or invalid market data decides nothing: a missing expected candle is exit 4; an incoherent candle (for example high below low) or a provider failure is exit 6; nothing from the range is stored. | `AcquireFuturesNativeDailyHistoryUseCase._require_coverage` (`northstar-application/.../acquire_futures_native_daily_history.py`); `FuturesOHLCVBar` validation | `northstar-api/tests/test_india8b_nse_chronological_operations.py::test_a_missing_expected_candle_is_data_and_nothing_is_decided`; `::test_a_provider_failure_stops_and_a_rerun_recovers` | -- | PASS | none |

## 4. Paper-order lifecycle

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| O-1 | BUY targets +N, SELL targets -N; the order is the difference between target and current position in the decided contract, in whole contracts. | `CreateFuturesExecutionIntentUseCase.execute` (`northstar-application/.../create_futures_execution_intent.py`) | `northstar-application/tests/application_services/test_create_futures_execution_intent.py::test_the_exposure_transition_matrix` | October 7 SELL from flat produced SELL 1 (operator-reported, E-1) | PASS | none |
| O-2 | HOLD, and a decision whose target is already met, create no order and keep the existing exposure. | Same use case: no-intent reasons `HOLD`, `TARGET_ALREADY_MET` | `northstar-application/tests/application_services/test_run_futures_paper_trading_decision.py::test_hold_keeps_existing_exposure` | -- | PASS | none |
| O-3 | A reversal is one order through zero (LONG 1 to SHORT 1 is SELL 2; SHORT 1 to LONG 1 is BUY 2). | Same use case | `test_create_futures_execution_intent.py::test_a_reversal_is_one_trade_through_zero`; `northstar-application/tests/application_services/test_calculate_futures_realized_pnl.py::test_a_short_reversal_realizes_only_the_old_position` | Not yet observed | PASS | none (natural observation not required) |
| O-4 | Duplicate orders are impossible: the order identity is derived deterministically from portfolio, contract, timeframe, decision instant and strategy; stores are insert-only. | `FuturesPaperExecutionIdentityService` (`northstar-application/.../futures_paper_execution_identities.py`); `northstar-infrastructure/.../persistence/sqlite_futures_paper_trading.py` | `northstar-api/tests/test_india8b_nse_chronological_operations.py::test_an_interrupted_frozen_decision_is_completed_without_duplication`; `northstar-infrastructure/tests/test_sqlite_futures_paper_trading.py::test_the_adapters_issue_insert_only_sql` | -- | PASS | none |
| O-5 | A position is taken to FLAT only by the expiry flatten (section 13); no research action targets zero. | Expiry branch `_flatten` in `create_futures_execution_intent.py` | `northstar-application/tests/application_services/test_futures_expiry_flatten_guard.py::test_a_long_is_sold_in_full_whatever_the_action` | -- | DEFERRED for signal-driven FLAT (L-5); the expiry flatten is covered by X-1 | Approve L-5 |

## 5. Next-session OPEN fills

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| F-1 | An order fills at the OPEN of the first stored daily bar strictly after its decision; the fill's instant is that bar's session close. | `SimulateFuturesPaperOrderFillUseCase` (`northstar-application/.../simulate_futures_paper_order_fill.py`) | `northstar-application/tests/application_services/test_simulate_futures_paper_order_fill.py::test_the_fill_uses_the_open_not_high_low_or_close` | October 8 fill at 22580 for the October 7 SELL (operator-reported, E-2) | PASS | Attach the inspected fill record to E-2 (section 15) |
| F-2 | Until that bar is stored the order is PENDING; a weekend or holiday simply moves the fill to the next stored session. | Same use case; `_settle_pending` in `run_futures_paper_trading_decision.py` | `test_simulate_futures_paper_order_fill.py::test_sparse_history_fills_on_the_next_actual_bar` | -- | PASS | none |

## 6. Portfolio positions and gross P&L

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| P-1 | Positions carry direction, whole-contract quantity and weighted average entry; a reversal resets the average to the fill quote. | `_transition` (`northstar-application/.../build_futures_paper_portfolio.py`) | `northstar-application/tests/application_services/test_build_futures_paper_portfolio.py` | SHORT 1 after the October 8 fill (operator-reported, E-3) | PASS | none |
| P-2 | Gross realized P&L and gross unrealized P&L (marked at the latest stored daily close at or before the cutoff) in INR at the contract's point value; an unmarked position is "unavailable", never zero. | `CalculateFuturesRealizedPnlUseCase`; `ValueFuturesPaperPortfolioUseCase`; `BuildFuturesPaperTradingValuationUseCase` | `northstar-api/tests/test_india6_nifty_baseline_acceptance.py::test_valuation_matches_an_independent_recomputation` | INR valuation observed (operator-reported, E-3) | PASS | none |
| P-3 | The live valuation of the October portfolio agrees with an independent recomputation from its recorded fill and mark. | -- | -- | Not yet recomputed | PENDING | Recompute from read-only output (section 15) |
| P-4 | Fees, taxes, slippage, cash and margin. | Explicitly excluded ("gross"): `build_futures_paper_trading_valuation.py` module docstring | -- | -- | DEFERRED (L-6) | Approve L-6 |

## 7. Operator-approved daily candle finality

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| N-1 | Finality is `disabled` or `operator-approved` only; sessions through `NORTHSTAR_FUTURES_FINAL_THROUGH` are FINAL; processing stops at the first session that is not, before any provider request. | `OperatorApprovedFuturesDailyBarFinalityPolicy` (`northstar-application/.../futures_daily_bar_finality_policies.py`); `plan_session_backlog` (`northstar-api/src/northstar_api/operations.py`) | `northstar-api/tests/test_india8b_nse_chronological_operations.py::test_processing_stops_at_the_first_unapproved_session` | Final-through 2026-10-08; October 9 unapproved (operator-reported, E-7) | PASS | none |
| N-2 | With nothing newly approved a run reports `STATUS: WAITING`, exits 0, reads no clock and makes no provider request. | `_chronological_daily_operation` (`northstar-api/src/northstar_api/cli.py`) | `test_india8b_nse_chronological_operations.py::test_waiting_reads_no_clock_and_makes_no_provider_request` | 2026-10-10 08:00 IST scheduled run returned WAITING, exit 0 (operator-reported, E-4) | PASS | Attach E-4 artifact |
| N-3 | Nothing in Northstar, the wrapper or the task writes `.env` or `FINAL_THROUGH`. | `Invoke-NorthstarIndiaOperations.ps1` reads only; CLI reads only | `northstar-api/tests/test_india_windows_operations_scheduler.py::test_the_wrapper_never_writes_env_or_finality` | -- | PASS | none |
| N-4 | The operator has a written procedure for deciding whether a session whose candle was revised is final. | Runbook section 5 says only "verify outside Northstar" | -- | October 9 revision (operator-reported, E-7) | PENDING | Write the decision procedure (no automatic rule, no threshold) |

## 8. Chronological execution, idempotency and recovery

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| C-1 | Approved sessions are acquired as one range and decided one at a time at their own close, oldest first; catching up equals operating day by day. | `plan_session_backlog`; `_chronological_daily_operation` | `test_india8b_nse_chronological_operations.py::test_a_backlog_is_one_range_then_every_cutoff_in_order`; `northstar-api/tests/test_india7_nifty_incremental_operations_acceptance.py::test_catching_up_a_range_equals_operating_day_by_day` | -- | PASS | none |
| C-2 | A rerun adds no facts; an interrupted run completes without duplication. | `_replay_latest_cutoff` (`cli.py`); insert-only stores | `test_india8b_nse_chronological_operations.py::test_an_identical_rerun_adds_no_facts`; `::test_an_interrupted_frozen_decision_is_completed_without_duplication` | -- | PASS | none |
| C-3 | Persisted facts are never updated or deleted; a provider value that differs from a stored bar is a conflict (exit 5) and nothing is decided. | INSERT-only adapters in `northstar-infrastructure/.../persistence/` and `.../market_data/sqlite_futures_historical_market_data.py` | `test_india8b_nse_chronological_operations.py::test_a_revised_candle_is_state_and_nothing_is_decided`; `northstar-infrastructure/tests/test_sqlite_futures_paper_trading.py::test_the_adapters_issue_insert_only_sql` | -- | PASS | none |
| C-4 | One writer at a time: a second scheduled run reports SKIPPED; a manual writer during a run exits 5. | `DatabaseOperationsLock` (`northstar-api/src/northstar_api/operations_lock.py`) | `northstar-api/tests/test_operations_lock.py::test_the_concurrency_boundary_for_one_database`; `test_india_windows_operations_scheduler.py::test_concurrent_invocations_never_interleave_and_one_is_skipped` | -- | PASS | none |

## 9. Contract expiry safety

Detailed behaviour, including what the code does **not** define, is in
section 13.

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| X-1 | K = 5 (code-frozen): a decision at the E-6 close requires the flatten, which fills at the E-5 open; from E-5 through E no signal may reopen or increase the contract; a flatten never reverses. | `FuturesExpiryFlattenGuard` (`northstar-application/.../futures_expiry_flatten_guard.py`); `NSE_EXPIRY_FLATTEN_SESSIONS = 5` (`northstar-api/src/northstar_api/runtime.py`) | `northstar-api/tests/test_futures_expiry_guard_runtime.py::test_the_october_contract_is_flattened_before_expiry`; `::test_the_flatten_fills_at_the_e5_open`; `test_futures_expiry_flatten_guard.py::test_a_flatten_never_reverses_through_zero` | -- | PASS | none |
| X-2 | After E is processed, every run reports `STATUS: ROLLOVER REQUIRED`, exits 3, requests nothing and writes nothing; nothing rolls automatically. | `_chronological_daily_operation` (`backlog.exhausted` branch, `cli.py`) | `northstar-api/tests/test_india8b_nse_chronological_operations.py::test_the_expiry_window_and_then_rollover_required` | -- | PASS | none |
| X-3 | The October contract's flatten is observed in production: decision at the 2026-10-16 close, fill at the 2026-10-19 open, flat through 2026-10-27. | As X-1 | As X-1 | Not yet reached | PENDING | Decide whether natural observation is required for sign-off (section 15, item 9) |
| X-4 | A position or pending order that survives into or through expiry is handled as an approved, enforced and verified exception (EXP-1, EXP-2). | **Detection implemented in M1.4.3.2, not approved, not deployed.** `operations daily` reports `STATUS: EXPIRY EXCEPTION`, exit 7, instead of `ROLLOVER REQUIRED` (contract not flat or an order pending) or WAITING (the same after the Asia/Kolkata expiration date); dashboard stage `expiry_exception` (`northstar-api/src/northstar_api/cli.py` `_chronological_daily_operation`; `operational_status.py`). Nothing is settled or reconciled. The policy (13.7) and the [operator procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md) remain proposed | `northstar-api/tests/test_india_expiry_exception_detection.py` (V-1 to V-5, boundaries; local engineering result) | -- | BLOCKED | Satisfy 13.9: decide D-EXP-1 to D-EXP-7 (approval of S-1 and S-2, a reconciliation method), deploy the detection, and verify it on the deployment machine |

## 10. Dashboard and operational visibility

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| V-1 | The read-only dashboard shows the latest recommendation, recent decisions, paper orders and fills, positions, gross P&L, finality status, backlog and expiry safety; it never writes and takes no lock. | `GET /futures/dashboard`, `GET /futures/analysis` (`northstar-api/src/northstar_api/routers/futures.py`); `operational_status` (`northstar-api/src/northstar_api/operational_status.py`) | `northstar-api/tests/test_futures_dashboard.py::test_reading_writes_nothing_and_restart_reproduces_the_response`; `northstar-api/tests/test_india8d_operational_status.py::test_reading_mutates_nothing_and_takes_no_lock` | -- | PASS | none |
| V-2 | The dashboard stays consistent while a daily write commits: portfolio and valuation always describe one persisted history. | M1.2: `GetFuturesPaperTradingSnapshotUseCase` values the history it loaded; `_load_history` reads fills before orders | `northstar-application/tests/application_services/test_get_futures_paper_trading_snapshot.py::test_a_fill_committed_after_the_fill_read_cannot_split_portfolio_and_valuation`; `northstar-api/tests/test_futures_daily_operation.py::test_dashboard_reads_during_the_daily_write_stay_consistent` (E-8) | -- | PASS (code, merged to development) | none |
| V-3 | M1.2 is deployed to the production paper stack. | -- | -- | **Not deployed** (E-8) | PENDING | Deploy M1.2 and record the deployed commits |
| V-4 | Each scheduled run leaves a log and an atomically replaced `last-run.json` with outcome, exit code and class; Task Scheduler's last result is the wrapper's exit code. | `Invoke-NorthstarIndiaOperations.ps1` | `test_india_windows_operations_scheduler.py::test_a_completed_run_is_logged_recorded_and_passes_exit_zero`; `::test_the_status_file_is_running_during_the_run_and_always_complete_json` | -- | PASS | none |
| V-5 | Automatic notifications on failure or staleness. | Not implemented (`northstar-api/README.md`) | -- | -- | DEFERRED (L-7) | Approve L-7 |

## 11. Deployment and scheduling

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| D-1 | The Indian stack is isolated from CME: its own project, services, images, volume and `.env`. | `northstar-api/deploy/india/compose.yaml` | `northstar-api/tests/test_india8e_indian_deployment.py` | -- | PASS | none |
| D-2 | The hourly Windows task (registered disabled, single instance, 55-minute limit) runs exactly `operations daily` through the wrapper, including when the task passes no deployment directory. | `northstar-api/deploy/india/windows/`; default-directory fix in commit `78e8770` | `test_india_windows_operations_scheduler.py::test_an_omitted_deployment_directory_resolves_to_the_wrappers_parent` | First automatic production run 2026-10-10 08:00 IST (operator-reported, E-4) | PENDING | Attach the E-4 artifact (log and `last-run.json`, no secrets) |
| D-3 | Isolated real-Docker acceptance of the wrapper passes without touching production. | Opt-in harness `northstar-api/tests/test_india_windows_docker_acceptance.py` | 14 opt-in tests (skipped in ordinary runs) | 14/14 passed (operator-reported, E-5); production SQLite byte-identical before and after (operator-reported, E-6) | PENDING | Attach the test output and the before/after hashes to E-5 and E-6 |
| D-4 | The open gate G-1 (11.1) has a recorded resolution by its owner. | Runbook section 0 records the gate as **unresolved** | -- | -- | BLOCKED | The gate owner records its status (11.1) |

### 11.1 Open gate G-1: data retention and use permission

Runbook section 0 records a provider / legal gate: written permission from
Upstox to retain and use its market data on a persistent server is
**unresolved** there, and the runbook says it must be cleared before the stack
runs persistently. This specification records only that the gate is open in
the documentation and that no resolution has been recorded. It draws no legal
conclusion and asserts no breach: whether the gate has been resolved outside
these documents, and what it requires, is for the gate owner to record.

G-1 is separate from the engineering requirements. It is not satisfied by any
test or document here, and nothing in this specification changes the gate.

## 12. Backup and restoration

| ID | Acceptance criterion | Implementation | Automated test evidence | Natural evidence | Status | Remaining action |
|----|----------------------|----------------|-------------------------|------------------|--------|------------------|
| B-1 | A backup on the Windows deployment machine uses SQLite's backup API while no operations writer can run, passes `integrity_check` and `foreign_key_check`, and is recorded with its SHA-256. | [Windows SQLite Backup and Isolated Restore Drill](Indian-Futures-Windows-SQLite-Backup-and-Restore-Drill.md), blocks A, P (disposable-volume lock test), B, D and E (written and statically validated, never run); runbook section 14 for Linux | -- | Earlier online backup tests during the persistence audit (operator-reported, E-9) used a different, unrecorded method; they are not this procedure | PENDING | Run blocks A, P, B, D and E under supervision (M1.4.2); PASS only when `LOCK TEST OK`, `BACKUP OK`, the SHA-256 match and `VERIFY OK` are recorded |
| B-2 | One restore from that backup into a new, isolated volume succeeds, and the restored copy passes `integrity_check` and `foreign_key_check` and is identical to the backup in schema and every table. | Same document, blocks F, H and I (written and statically validated, never run) | -- | No restore drill has been performed. E-9 is backup evidence only, not a restore. | PENDING | Run blocks F, H, I and J under supervision (M1.4.2); PASS only when `RESTORE OK` and `COMPARE OK` are recorded |

## 13. Contract expiry behaviour

Sections 13.1 to 13.6 record what the code does. Section 13.7 sets a proposed,
unapproved exception policy beside that behaviour. This section introduces no
trading rule.

### 13.1 Definitions and the October contract

- E is the contract's expiry session; sessions are counted only through the
  NSE calendar (`NSEFuturesTradingSessionResolver`), never by calendar days.
- K = 5 is frozen in code (`NSE_EXPIRY_FLATTEN_SESSIONS`); there is no
  setting for it.
- For NIFTY / NSE / 2026-10-27 (2026-10-20 is an NSE holiday): E-7 is
  2026-10-15, E-6 is 2026-10-16, E-5 is 2026-10-19, and E is 2026-10-27
  (runbook section 11).

### 13.2 Normal flattening before expiry

`FuturesExpiryFlattenGuard.assess` requires the flatten when the sessions after
the decision through E number K + 1 or fewer. Under it,
`CreateFuturesExecutionIntentUseCase._flatten` targets zero in the decided
contract whatever the research said: a long is sold in full, a short is bought
back in full, never through zero. If the portfolio is already flat, no order is
created and the reason is `EXPIRY FLATTEN WINDOW`.

For the October contract: the decision at the 2026-10-16 close creates the
flatten order; it fills at the 2026-10-19 open; every decision from E-5 through
E keeps the contract flat. The chronological runner decides every approved
session at its own close, so it never skips the E-6 cutoff.

The only tested way to fill the flatten late is a **manual** `paper run` that
skips the E-6 cutoff: the flatten is then decided at E-5 and fills at the E-4
open (`test_india7_nifty_incremental_operations_acceptance.py::test_skipping_the_e6_cutoff_fills_the_flatten_one_session_late`,
"pinned, not fixed"). The scheduled operation does not do this.

### 13.3 Pending orders awaiting a later session

An order is pending until the first stored bar after its decision exists
(F-2). The E-6 flatten order is pending from the 2026-10-16 close until the
2026-10-19 bar is acquired; that happens only after 2026-10-19 is approved
(N-1).

The code has **no** cancellation, expiry or settlement of a pending order. An
order decided at a session after which no bar of the contract is ever stored
(for example a flatten decided at E itself) stays pending indefinitely.

### 13.4 Open positions that survive into or through expiry

Under the scheduled operation a position survives into the protected window
only if the sessions through E-5 are not processed in time. The mechanisms
present today:

1. **Finality not approved.** Sessions after `FINAL_THROUGH` are never
   processed: every run reports WAITING, and the position stays as it is.
   From M1.4.3.2 (not deployed) a run after the expiration date reports
   `EXPIRY EXCEPTION` (exit 7) instead.
   This includes the present state (final through 2026-10-08, October 9
   unapproved, SHORT 1 open).
2. **Approval after the contract can no longer be acquired.** Upstox
   resolution uses the current instrument master, which lists tradable
   contracts only (`upstox_instrument_master.py`); an expired instrument key is
   answered with `UDAPI100011` (`upstox_http.py`). Acquisition after expiry is
   therefore expected to fail with exit 6, so the sessions can never be
   processed. That the master drops the contract right after expiry is
   documented provider behaviour, not observed here.

In both cases the position is not flattened, not settled and not closed:
Northstar keeps reporting it as open, with unrealized P&L at the last stored
close. The backlog is not exhausted (sessions remain unprocessed), so the
operation reports WAITING or fails with exit 6, not ROLLOVER REQUIRED. The
dashboard's expiry safety block shows `position_flat: false`.

### 13.5 Post-expiry processing

When every session through E has been processed, `plan_session_backlog`
reports the backlog exhausted. Each run then replays the latest cutoff
idempotently, prints `STATUS: ROLLOVER REQUIRED`, exits 3 ("nothing rolls
automatically"), makes no provider request and writes no fact. The dashboard
shows the backlog stage `rollover_required`. From M1.4.3.2 (implemented, not
deployed) this holds only when the contract is flat with no pending order;
otherwise the run reports `STATUS: EXPIRY EXCEPTION` with exit 7 and the
dashboard stage `expiry_exception` (13.9).

### 13.6 Manual operator intervention

- The runbook's manual rollover (section 13) starts with "Confirm the current
  contract reached the protected state and E was processed ... position flat".
  It defines no procedure for a contract that is **not** flat.
- There is no command to cancel a paper order, close or settle a paper
  position, or edit a fact. The stores are insert-only by design; editing the
  database by hand is outside every procedure.
- The operator's levers are: approving sessions (section 7), disabling
  finality or the scheduled task, and starting a new contract with a new
  portfolio identity (runbook section 13). Starting a new portfolio leaves the
  old portfolio's facts unchanged in the same database.

Automatic rollover is not part of the MVP (L-4).

### 13.7 Expiry exceptions: current behaviour and proposed policy

Sections 13.1 to 13.6 describe the code. This section separates what the code
does today from a **proposed** exception policy that has **not been approved,
implemented or verified**. Writing the policy down does not make an expiry
exception safe: until 13.9 is satisfied, a position or pending order that
survives expiry is an unhandled condition, and X-4 stays BLOCKED.

The K = 5 flatten (13.2) and ROLLOVER REQUIRED (13.5) are unchanged by
anything in this section. Automatic settlement and automatic rollover remain
outside the MVP (L-4, L-8).

**EXP-1: an open paper position survives contract expiry** (the case in
13.4).

| | Current code behaviour | Proposed policy (pending approval) |
|---|---|---|
| Recorded position and decisions | Kept unchanged; the stores are insert-only | Keep the recorded position and every historical decision unchanged |
| Fill, settlement or closing price | None is created | Do not invent a fill, a settlement, a price or a closing transaction |
| Other contracts | Nothing rolls; a new contract needs manual configuration | Do not roll the position into another contract |
| What Northstar reports | The position stays open, marked at the last stored close. Before M1.4.3.2, runs reported WAITING or failed with exit 6, and nothing identified the state as an expiry exception. From M1.4.3.2 (implemented, not deployed): `operations daily` reports `STATUS: EXPIRY EXCEPTION`, exit 7, and the dashboard stage `expiry_exception` (13.9) | Treat the state as an **operational exception**, never as a successful expiry flatten |
| Resolution | No procedure exists; the manual rollover (runbook section 13) assumes a flat position | The operator investigates, and the ledger is declared resolved only through an explicit, approved reconciliation or settlement procedure, which does not exist yet |

**EXP-2: a pending paper order has no valid later execution session** (the
case in 13.3).

| | Current code behaviour | Proposed policy (pending approval) |
|---|---|---|
| Recorded order | Kept unchanged, reported as pending | Keep the recorded pending order |
| Fill | None is created | Do not manufacture a fill |
| Cancelled or expired | The domain has no such state; nothing records one | Do not treat the order as cancelled or expired unless the domain records that state |
| Disposition | None defined | An explicit, operator-reviewed disposition policy is required |
| Audit | The order and its decision remain stored | Keep an auditable record of the exception, including how it was disposed of |

### 13.8 Implementation options considered

These are the ways the proposed policy could later be enforced. M1.4.3.2
implemented the **fail-closed exception signal** as detection only (13.9); it
is not approved and not deployed. No other option is selected or implemented.
Settlement and new order states would need an ADR.

| Option | Applies to | What it would do | Implications |
|--------|------------|------------------|--------------|
| Record only | EXP-1, EXP-2 | Record the exception in the evidence register with its last stored values | No code change. On its own this **documents** the exception but does not detect, enforce or resolve it, so it cannot close X-4. |
| Operating deadline | EXP-1 | The runbook requires sessions through E-5 to be decided and processed before expiry, with escalation if that is at risk | Reduces how often the exception occurs. It ties finality to a calendar deadline, which must never pressure the approval of a revised candle. |
| Fail-closed exception signal | EXP-1, EXP-2 | The operation reports a distinct non-zero status when the protected window is reached or passed with a non-flat position or a pending order | Makes the exception impossible to miss without changing accounting. Code and tests. |
| Terminal settlement rule | EXP-1 | An explicit, recorded close-out at an official final settlement price | A new accounting rule and data source. Outside the MVP (L-8). |
| "Expired unfilled" record | EXP-2 | A new insert-only terminal record for such orders | A new domain state. Outside the current scope unless approved. |

### 13.9 What X-4 requires

X-4 leaves BLOCKED only when all of the following are true:

1. The EXP-1 and EXP-2 policies (13.7) are approved, as written or amended.
2. The policy is **enforced**: either implemented and covered by tests, or
   demonstrably enforced by a written operator procedure that defines the
   investigation, the exception record, and the reconciliation, settlement or
   disposition steps.
3. The enforcement is verified: automated tests for any code, or a recorded
   exercise of the procedure against a copy of the database, never the live
   volume.

Approval alone, or this documentation alone, does not satisfy X-4. The proposed
[Expiry Exception Operator Procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md) concludes that
procedure-only enforcement is not sufficient: nothing in the system flags
either exception, WAITING reports success after the real expiry date, and
`ROLLOVER REQUIRED` does not check flatness. It proposes the minimum
fail-closed correction S-1 and the decision S-2 (its section 12) and the
isolated verification V-1 to V-5 (its section 13).

**M1.4.3.2 (2026-10-10): detection implemented, not approved, not deployed.**
The procedure's corrections S-1, S-2 and S-3 are implemented in
`northstar-api` (procedure section 12):

- `operations daily` reports `STATUS: EXPIRY EXCEPTION` with exit 7 instead of
  `ROLLOVER REQUIRED` when the contract is out of sessions but not flat, or has
  a pending order (S-1).
- It does the same instead of WAITING or any acquisition when the Asia/Kolkata
  date is strictly later than the expiration date (S-2).
- The dashboard reports stage `expiry_exception` and labels its expiry window
  as assessed as of the latest decision (S-3).
- The Windows wrapper records outcome `EXPIRY_EXCEPTION`.

Nothing is settled, cancelled, expired, filled or rolled over. Local synthetic
tests V-1 to V-5 and the boundary cases pass (procedure section 13). This
partly addresses 13.9 item 2 (enforcement of detection) and item 3 (automated
tests). It does not satisfy item 1 (approval), the disposition and
reconciliation part of item 2 (D-EXP-6), or deployment. X-4 stays BLOCKED.

## 14. Accepted MVP limitations (proposed scope decisions, subject to approval)

None of these capabilities is implemented. Each is a proposed MVP scope
decision; it becomes accepted only with the sign-off in section 15.

| ID | Limitation | Current behaviour | Belongs to |
|----|------------|-------------------|------------|
| L-1 | No live orders | Paper fills only; ADR-008 represents no live environment | Phase 5 (controlled live trading, after explicit approval) |
| L-2 | No broker demo integration yet | No broker adapter is wired into operations | Phase 2 |
| L-3 | No automatic candle approval | Only `disabled` and `operator-approved` exist; no time-based mode (`settings.py`) | Later, as a separate explicit decision |
| L-4 | No automatic contract rollover | ROLLOVER REQUIRED, exit 3; manual runbook section 13 | Later |
| L-5 | No signal-driven FLAT exit | Research targets +N or -N or holds; FLAT only through the expiry flatten | Later strategy work |
| L-6 | Gross P&L only | No fees, taxes, slippage, cash or margin (`build_futures_paper_trading_valuation.py`) | Phase 2 onward, with broker account data |
| L-7 | No automatic operational notifications | Exit codes, logs, `last-run.json`, Task Scheduler's last result and the dashboard only | Phase 4 (live-trading safeguards) |
| L-8 | No automatic expiry settlement | A position or order surviving expiry is neither settled nor closed (13.4, 13.7) | Later, only with an approved settlement rule and ADR |

L-4 and L-8 limit automation only. They do not remove the need to handle an
expiry exception: that is X-4, which stays BLOCKED until 13.9 is met.

## 15. Final sign-off requirements

The MVP is accepted when all of the following are recorded:

1. Every requirement in sections 2-12 is PASS, or DEFERRED with its
   limitation approved in section 14.
2. X-4 is met as defined in 13.9: the EXP-1 and EXP-2 policies are approved,
   enforced (by tested code or a written operator procedure) and verified.
   Documenting or approving the policy is not enough. Any code is a separate
   approved milestone.
3. The operator-reported evidence E-1 to E-7 is backed by attached artifacts
   (log excerpts, `last-run.json`, read-only report output, test output,
   hashes), with secrets and tokens removed, or explicitly accepted as
   operator-reported by the approver.
4. P-3: the October portfolio's valuation is independently recomputed.
5. M1.2 is deployed to production and the deployed commits are recorded
   (V-3).
6. Backup and one restore drill on a copy are completed and recorded (B-1,
   B-2).
7. The gate owner records the status of open gate G-1 (11.1, D-4).
8. The finality decision procedure for revised sessions is written (N-4).
9. The approver decides whether natural observation of the October expiry
   flatten (X-3) is required. If it is, sessions through 2026-10-19 must be
   approved and processed while the contract can still be acquired; that
   deadline must not pressure the approval of a revised candle.
10. Full automated suites pass at the deployed commits, and the counts are
    recorded in the evidence register.

| Role | Name | Decision | Date |
|------|------|----------|------|
| Approver | | | |
| Operator | | | |

## 16. Acceptance summary (2026-10-10)

| Area | PASS | PENDING | BLOCKED | DEFERRED |
|------|------|---------|---------|----------|
| 2. Market, instrument, strategy | S-1, S-2, S-3 | | | |
| 3. Research and frozen decisions | R-1, R-2, R-3, R-4 | | | |
| 4. Paper-order lifecycle | O-1, O-2, O-3, O-4 | | | O-5 |
| 5. Next-session OPEN fills | F-1, F-2 | | | |
| 6. Positions and gross P&L | P-1, P-2 | P-3 | | P-4 |
| 7. Finality | N-1, N-2, N-3 | N-4 | | |
| 8. Chronology, idempotency, recovery | C-1, C-2, C-3, C-4 | | | |
| 9. Expiry safety | X-1, X-2 | X-3 | X-4 | |
| 10. Dashboard and visibility | V-1, V-2, V-4 | V-3 | | V-5 |
| 11. Deployment and scheduling | D-1 | D-2, D-3 | D-4 | |
| 12. Backup and restoration | | B-1, B-2 | | |

Every PASS above rests on inspected code and passing automated tests. No
requirement is PASS because a policy or procedure has been written down.

## 17. Acceptance decisions outstanding

What each open item still needs. "--" means nothing of that kind is needed.

| Item | Approve | Implement | Test or verify | Evidence to attach |
|------|---------|-----------|----------------|--------------------|
| X-4 (EXP-1, EXP-2) | The 13.7 policy, as written or amended, and decisions D-EXP-1 to D-EXP-7 of the [operator procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md) | Done for detection: S-1, S-2 and S-3 implemented and tested locally (M1.4.3.2). Open: deployment, and a reconciliation method (D-EXP-6) | V-1 to V-5 passed locally; verification on the deployment machine after deployment | The verification results |
| X-3 | Whether live observation of the October flatten is required | -- | If required: the 2026-10-16 decision and 2026-10-19 fill observed | Read-only output of both |
| N-4 | The written procedure for a revised session | Write the procedure (no automatic rule, no threshold) | -- | -- |
| D-4 (G-1) | The gate owner records the gate's status | -- | -- | The recorded status |
| V-3 | -- | Deploy M1.2 to production | Dashboard reads after deployment | Deployed commits |
| P-3 | -- | -- | Independent recomputation of the October valuation | The recomputation |
| B-1, B-2 | -- | Done: the Windows procedure is written (not run) | The supervised drill, blocks A, P, B, D, E, F, H, I and J (M1.4.2) | The drill record (evidence register) |
| D-2, D-3 and E-1 to E-6 | Or the approver accepts them as operator-reported | -- | -- | Logs, `last-run.json`, test output, hashes (no secrets) |
| O-5, P-4, V-5 and L-1 to L-8 | The proposed scope limitations | -- | -- | -- |
| Sign-off item 10 | -- | -- | Full suites at the deployed commits | Recorded counts |
