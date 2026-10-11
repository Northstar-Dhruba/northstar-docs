# Indian Futures: Revised Candle Approval Procedure (N-4)

Status: **written for operator review; not approved** (2026-10-11).
- This procedure decides nothing. It approves no candle, changes no
  `NORTHSTAR_FUTURES_FINAL_THROUGH`, and grants no G-1 permission.
- Requirement N-4 of the
  [Paper MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md)
  stays **PENDING** until the operator and the approver accept this procedure.
  Writing it does not satisfy N-4 on its own.

Related:
- [Deployment Runbook](Indian-Futures-Deployment-Runbook.md) (section 5:
  operator-approved finality);
- [Upstox Futures Daily-Candle Finality Evidence](Upstox-Finality-Evidence.md);
- [Paper MVP Evidence Register](Indian-Futures-Paper-MVP-Evidence-Register.md)
  (E-3, E-7);
- [Expiry Exception Operator Procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md);
- [Paper MVP Readiness Assessment](Indian-Futures-Paper-MVP-Readiness-Assessment.md).

Source paths are relative to the Northstar workspace root. Code was read at:

| Repository | Commit |
|------------|--------|
| core | e54ccdf |
| application | 9a0ab57 |
| infrastructure | aa94428 |
| api | cfa7774 |
| web | 4f6908c |
| docs | 62a905d |

No production system, database, provider or market-data file was accessed to
write this document, and no candle value is reproduced here.

## 1. Purpose and scope

This procedure gives the operator a written, evidence-based way to decide
whether a daily candle whose provider values were seen to change may be
treated as final. That is the decision `NORTHSTAR_FUTURES_FINAL_THROUGH`
records.

It covers the NIFTY / NSE futures contract the India paper stack operates
(currently 2026-10-27). The immediate case is the **2026-10-09** session.

It does not cover:
- automatic finality;
- any time-of-day rule or numeric threshold;
- correcting stored data, decisions, orders or fills;
- G-1.

## 2. Current finality architecture (verified)

| # | Fact | Source |
|---|------|--------|
| 1 | **Configuration.** `NORTHSTAR_FUTURES_DAILY_BAR_FINALITY` is `disabled` (the default when unset) or `operator-approved`, case-insensitive. `NORTHSTAR_FUTURES_FINAL_THROUGH` must be a `YYYY-MM-DD` date and is **required** in `operator-approved` mode: missing means a configuration error, never a silent fall-back. `NORTHSTAR_FUTURES_GO_LIVE` applies only before the first frozen decision | `load_session_operation_settings`, `_finality_mode`, `_trading_date` in `northstar-api/src/northstar_api/settings.py` |
| 2 | **Meaning of operator-approved.** A session is `FINAL` if its trading date is on or before final-through, and `NOT_YET_FINAL` after it. The date is a session label, never an instant or time of day; no clock is read. `disabled` makes every session `UNKNOWN` | `OperatorApprovedFuturesDailyBarFinalityPolicy` and `DisabledFuturesDailyBarFinalityPolicy` in `northstar-application/src/northstar_application/application_services/futures_daily_bar_finality_policies.py`; `build_daily_bar_finality_policy` in `northstar-api/src/northstar_api/runtime.py` |
| 3 | **Approved-through is continuous and inclusive.** There is **no per-candle approval, no per-session exclusion and no approval of a candle version**. One date approves every session up to and including it | Same policy |
| 4 | **Processing order.** `plan_session_backlog` lists the sessions after the latest frozen decision through expiry, assesses them in order, and **stops at the first non-final one**. Only the final prefix is processed. No session is ever skipped | `plan_session_backlog` and `latest_frozen_decision` in `northstar-api/src/northstar_api/operations.py` |
| 5 | **Not approved means WAITING.** With no final session to process, `operations daily` prints `STATUS: WAITING -- <reason>`, exits 0 and makes no provider request. From M1.4.3.2 an exposed contract after its expiration date reports exit 7 instead (S-2; not yet confirmed in production) | `_chronological_daily_operation` in `northstar-api/src/northstar_api/cli.py` |
| 6 | **Acquisition.** The eligible sessions are acquired as **one range, validated before one atomic write**. A missing expected session is `FuturesDailySessionCoverageError` (exit 4, nothing stored or decided). A session whose close has not yet passed is refused. Each candle's date is its label read in the venue's time zone (Asia/Kolkata), and its bar instant is the session close from the NSE calendar | `_acquire` in `cli.py`; `_trading_date` in `northstar-infrastructure/src/northstar_infrastructure/market_data/upstox_futures_native_daily_market_data.py`; test `test_india8b_nse_chronological_operations.py::test_a_missing_expected_candle_is_data_and_nothing_is_decided` |
| 7 | **Endpoint routing.** A request for the venue's current date goes to Upstox's current-day endpoint; any other date to the historical endpoint | `_chronological_daily_operation`; `upstox_candle_evidence.py` module docstring |
| 8 | **Storage is insert-only.** A bar equal to the stored one is a no-op. A **differing** bar under the same contract, timeframe and instant raises `FuturesHistoricalMarketDataConflictError` (exit 5): nothing is stored and nothing is decided. There is no UPDATE, REPLACE or DELETE, and **no reconciliation step exists** | `SQLiteFuturesHistoricalMarketDataStore` in `northstar-infrastructure/src/northstar_infrastructure/market_data/sqlite_futures_historical_market_data.py`; test `test_india8b_nse_chronological_operations.py::test_a_revised_candle_is_state_and_nothing_is_decided` |
| 9 | **No provenance on stored bars.** `futures_ohlcv` holds the contract, timeframe, instant, open, high, low, close and volume only: no provider timestamp, endpoint, acquisition time, version, hash or open interest | `northstar-infrastructure/src/northstar_infrastructure/market_data/sqlite_futures_schema.py` |
| 10 | **Decisions, orders and fills are immutable.** A decision is frozen once; a different decision for the same instant is a conflict. Orders have deterministic identities. A fill is the OPEN of the first stored bar after the decision, persisted insert-only, one per order. Positions and P&L are derived, never stored | `FreezeFuturesForwardResearchDecisionUseCase` (`freeze_futures_forward_research_decision.py`); `SimulateFuturesPaperOrderFillUseCase` (`simulate_futures_paper_order_fill.py`); acceptance R-2, O-4, F-1, C-3 |
| 11 | **What the strategy reads.** The latest 20 daily bars: closes and volumes | `select_directional_signal` in `northstar-core/src/northstar_core/strategy/_directional_signal.py`; acceptance R-1 |
| 12 | **Visibility.** The run log prints `Finality mode`, the latest frozen decision and every session assessed with its outcome and reason. The dashboard shows the mode, final-through, the next session's outcome and reason, and the latest market and decision sessions. `paper status` shows orders, fills, pending orders and positions | `_finality_line` and `_backlog_lines` in `cli.py`; `FinalityResponse` in `northstar-api/src/northstar_api/schemas/futures.py`; `operational_status` in `northstar-api/src/northstar_api/operational_status.py` |
| 13 | **Evidence tool.** `northstar finality-evidence observe` appends what Upstox returns for one contract and date to a private JSON Lines file, with request and receipt times. `report` derives every observed change per field. Neither opens a Northstar database, and neither declares anything final | `northstar-infrastructure/src/northstar_infrastructure/market_data/upstox_candle_evidence.py`; [evidence guide](Upstox-Finality-Evidence.md) |

## 3. Revised-candle detection and evidence

**A revision can become visible in exactly three ways:**
1. **Evidence observations.** Two `finality-evidence observe` records of one
   contract and date differ in a field; `report` lists it as an observed
   change.
2. **A storage conflict.** An acquisition returns a value differing from an
   already stored bar: exit 5, nothing stored or decided (section 2, item 8).
3. **Outside Northstar**, for example an exchange notice.

**What Northstar cannot show.** Northstar never re-fetches a session it has
already processed: the chronological operation acquires only sessions after
the latest decision. A revision of an already-processed session therefore
stays invisible unless (1) or a manual `market-data sync` over it is run.

**Evidence quality** (from the evidence guide):
- **Endpoint difference versus revision.** An observation of the venue's
  current date comes from the current-day endpoint, later ones from the
  historical endpoint. A difference between them may be an endpoint
  difference rather than a revision over time.
- **Classify every difference before using the word revision:**

  | Class | Meaning |
  |-------|---------|
  | Same-day endpoint observation | Made on the session's own venue date, so served by the current-day endpoint |
  | Historical endpoint observation | Made on a later venue date, so served by the historical endpoint |
  | Demonstrated change | Two observations from the **same endpoint**, of the same contract and data definition, differ. Only this shows a provider revision over time |
  | Unexplained discrepancy | Observations differ across endpoints, or the difference cannot be attributed. It is **not** a confirmed provider revision |
- **Right-censoring.** No observed change after a time does not prove no later
  change.
- **Expiry.** After the contract leaves the instrument master, it cannot be
  observed at all.

## 4. The October 9 issue

**What is recorded** (operator-reported, evidence register E-7):
- Finality is `operator-approved` through 2026-10-08.
- Three observations were made of the 2026-10-09 candle of NIFTY 2026-10-27:
  - the open, high, low and volume were the **same** in all three;
  - the **close** and the open interest were **different** in the later two
    than in the first;
  - the two later observations, about 21 minutes apart, were identical.
- E-7's own note: the first came from the current-day endpoint and the later
  two from the historical endpoint. So the change **may be an endpoint
  difference rather than a revision over time**, which is unresolved.
- **Classification (section 3):** one same-day endpoint observation and two
  historical endpoint observations. The two historical ones are identical, so
  **no change within one endpoint is demonstrated**. On the recorded evidence
  October 9 is an **unexplained discrepancy, not a confirmed provider
  revision**. "Revised" in this procedure's name does not claim otherwise.

**Why October 9 blocks everything after it:**
- **Any final-through date on or after 2026-10-09 approves October 9**
  (section 2, item 3).
- Processing cannot pass October 9 without approving it (item 4). There is
  **no mechanism to exclude it, substitute a value, or skip it**.
- So approval through 2026-10-16, 2026-10-19 or any later date includes
  October 9.

**What is not known, and why:**

| Question | Answer | Basis |
|----------|--------|-------|
| Is a 2026-10-09 bar stored in production? | **Unknown.** The chronological operation acquires only final sessions, so it has not, unless a manual `market-data sync` was run. Read-only check: the dashboard's latest market session, or a WAITING run's log (dates only) | Item 4; no production read was made |
| Is an earlier version preserved? | Only in the operator's private evidence file, if collected (E-7). The database keeps no versions, and the record would be lost if no file exists | Items 9 and 13 |
| Do revisions carry timestamps, versions, hashes or provenance? | The evidence file does (`requested_at`, `received_at`, the endpoint implied by the date). Stored bars do not | Items 9 and 13 |
| Can the versions be compared from authorized evidence? | Technically, with `finality-evidence report`. Whether the file may be retained and used is G-1 (section 12) | [Evidence guide](Upstox-Finality-Evidence.md) |
| Did any decision or fill use the original October 9 version? | **No decision can have.** The latest frozen decision is reported at the 2026-10-08 close (E-3), and every later session waits. **A fill could only use it** if an order decided at the 2026-10-08 close is pending and an October 9 bar were stored; neither is established | Items 4 and 10; E-3 |
| Is there a pending order now? | **Unknown** here. `paper status` shows `Pending: n` and each `PENDING` line (metadata: side, count, decision instant) | `execution_summary_lines` in `northstar-api/src/northstar_api/_cli_rendering.py` |
| Does the observed difference change any decision? | **Not demonstrated.** The close enters the 20-bar window, so the 2026-10-09 decision and the next 19 could depend on it. Whether they would differ depends on values this document does not hold | Item 11 |
| Could replay create duplicate orders? | No: identities are deterministic and stores are insert-only. A **different** decision for an existing instant is refused (exit 5) | Item 10 |

## 5. Operator responsibilities

The operator:
- runs the review in section 6;
- keeps the evidence under G-1's restrictions;
- records the decision with the template in section 14;
- changes `.env` only through the gate of section 9, under a separate
  explicit authorization.

The approver accepts this procedure (N-4) and decides ESCALATE cases. Neither
role may approve a candle in order to meet a date (section 11).

## 6. Review checklist

Each check is PASS, FAIL or UNKNOWN. **Any UNKNOWN makes the outcome DEFER**,
and any FAIL makes it REJECT or ESCALATE (section 8). Nothing is ever approved
by default.

| # | Check | How |
|---|-------|-----|
| RC-1 | Exact contract (product, exchange, expiration) and session date | The evidence record's contract fields and `trading_date`; `.env` `NORTHSTAR_FUTURES_EXPIRATION` |
| RC-2 | Provider and source: which endpoint served each observation | `requested_at` against the venue date (section 2, item 7) |
| RC-3 | The evidence used to compare versions is identified, is complete, and its retention is authorized | The evidence file's path and record count (metadata), and the G-1 status |
| RC-4 | The candle's completeness and validity: present; high at least open, close and low; non-negative volume; a whole number of contracts at the recorded lot size | `finality-evidence report` latest candle; canonical acquisition refuses an invalid one anyway |
| RC-5 | Timestamp, session attribution and time zone: the label's date in Asia/Kolkata equals the session; observation times relative to the close (15:40 IST since 2026-08-03) | `report` "from close" times |
| RC-6 | Contract, expiry and OHLCV interpretation: the instrument key and lot size are consistent across observations; only volume and the closes feed the strategy; open interest is not stored | `report` instrument key and lot-size lines; section 2, items 9 and 11 |
| RC-7 | Effect on already approved data: whether any session on or before the current final-through has also been seen to change | `report` over the earlier approved sessions, if observed; otherwise UNKNOWN |
| RC-8 | Effect on paper state: whether the session is already stored; any pending order whose fill would use its OPEN; which decisions would read it | Section 7 |
| RC-9 | Whether the candle has stopped changing for long enough, by the operator's judgement, after a **historical-endpoint** observation made after the last observed change | `report` last observed change and the unchanged observations after it. No threshold is defined, and none may be invented |
| RC-10 | The decision and its evidence are recorded | Section 14 |

## 7. Revision impact assessment

| Area | Immutable or derived | Effect of approving a revised session |
|------|----------------------|---------------------------------------|
| Stored bar | Immutable (insert-only) | **Not yet stored:** the value Upstox returns **at acquisition time** is stored. It may differ again from what was reviewed. **Already stored and now differing:** the run stops with exit 5 and nothing is stored |
| Strategy decisions | Immutable once frozen | Decisions **after** the session use the stored value, inside a 20-bar window. Decisions already frozen are never re-decided |
| Pending orders | Immutable | An order decided before the session fills at that session's **OPEN** once the bar is stored |
| Fills | Immutable | The fill quote is the stored OPEN. Re-simulation against a different bar is refused |
| Positions | Derived from fills | Follow the fills |
| Realized P&L | Derived from fills and economics | Follows the fills |
| Unrealized P&L and mark | Derived: the latest stored close at or before the cutoff | The mark uses the stored close of the latest session |

**If a revision ever has to change an already persisted order, fill or
decision**, that is a separate policy and engineering decision. The code has no
mechanism for it, and none may be improvised: no editing, cancellation,
settlement or synthetic fill.

## 8. Decision outcomes

These are procedure outcomes recorded in the decision record, not application
states.

| Outcome | When | Effect on finality eligibility |
|---------|------|--------------------------------|
| **APPROVE** | RC-1 to RC-10 PASS, and the approver has accepted this procedure | A procedural judgement on the **reviewed evidence only**. It is **not** authorization to ingest market data. It permits, but does not perform, a final-through change, and only through the gates of section 9, including FG-8 (runtime authorization) and FG-9 (review-to-ingestion equivalence). Until `.env` is changed under those gates, nothing changes |
| **REJECT** | The candle fails RC-4, RC-5 or RC-6, or the operator concludes it is not final | Final-through stays where it is. There is no mechanism to skip or exclude the session, so **processing stays stopped before it**: every run is WAITING. For the October contract, this leads towards the expiry exception (section 11) |
| **DEFER** | Any check is UNKNOWN: evidence unavailable, not authorized (G-1), or not yet stable | Final-through unchanged. Review again when the evidence exists |
| **ESCALATE** | Any inconsistency: an already-consumed revision (a stored bar or used OPEN differs from the reviewed value), RC-7 FAIL, an exit-5 conflict, or a possible accounting effect | Final-through unchanged. The approver decides; any data or accounting change is a separate milestone (section 7) |

## 9. Finality advancement prerequisites

Before `NORTHSTAR_FUTURES_FINAL_THROUGH` may move from its current value to a
proposed date `P`, **all** of these must hold:

| # | Gate |
|---|------|
| FG-1 | Every session after the current final-through and up to `P` has an **APPROVE** decision record (section 14), not just the last one |
| FG-2 | The 2026-10-09 session (the observed discrepancy) has an APPROVE record. Any REJECT, DEFER or ESCALATE on it blocks every `P` on or after 2026-10-09 |
| FG-3 | No known revision of an earlier approved session is unresolved (RC-7) |
| FG-4 | The effects on persisted paper state are reviewed (RC-8): the stored-bar state and pending orders are recorded as metadata |
| FG-5 | Data permission: the G-1 status allows the evidence relied on to be retained and used (section 12) |
| FG-6 | `P` is a valid NSE trading session (as `YYYY-MM-DD`) not after the contract's expiration date, and no earlier session up to `P` is unresolved in the NSE calendar |
| FG-7 | The operator states the expected runtime behaviour: the next run acquires and decides every session up to `P` in one range, oldest first, may create orders and fills, and stops with exit 5 or 4 on a conflict or a missing candle |
| FG-8 | The `.env` change is **separately and explicitly authorized** (who, when, the old and new values), and made one approval at a time as in runbook section 5 |
| FG-9 | **Review-to-ingestion equivalence.** For every session reviewed under this procedure, an approved, technically supported method ensures that the candle the run will ingest is the candle that was reviewed. **No such method exists today** (below), so FG-9 cannot be met |

**STOP on incomplete evidence.** If any gate cannot be shown, do not change
`.env`. The decision is DEFER, final-through stays unchanged, and every run
keeps reporting WAITING.

**The review-to-ingestion gap (FG-9).** Traced in code: no step checks that
the ingested candle equals the reviewed one.

| Step | What happens | Checks the reviewed version? |
|------|--------------|------------------------------|
| 1. Review evidence | `finality-evidence observe` and `report` write and read a private JSON Lines file. Nothing in the acquisition path reads it | No: it is the reference, and nothing consults it |
| 2. Procedural APPROVE | A decision record (section 14). No code reads it | No |
| 3. Final-through change | `.env`; the policy compares session **dates** only (section 2, items 2 and 3) | No |
| 4. Fetch | `_chronological_daily_operation` calls `AcquireFuturesNativeDailyHistoryUseCase.execute`, which fetches the whole eligible range from Upstox **at run time** (`UpstoxFuturesNativeDailyMarketDataSource`) | No: whatever Upstox returns then |
| 5. Validate | `_index_observations` (contract, one candle per date), `_require_coverage` (one per session), and `FuturesOHLCVBar` coherence, in `northstar-application/src/northstar_application/application_services/acquire_futures_native_daily_history.py` | No: structure only, no expected value |
| 6. Insert | `SQLiteFuturesHistoricalMarketDataStore._store_one` compares only with a bar **already stored** under the same key. With none stored, it inserts the fetched bar | No. Insert-only storage protects an existing bar, not a reviewed one |
| 7. Decide, order, fill | The same run then paper-runs each acquired session, oldest first, with no pause between insert and decision | No: the bar is consumed before any operator check |

**Consequences:**
- A run after a final-through change can store, decide on and fill against a
  candle that differs from the reviewed one, without any error.
- No hash, version pin, provider guarantee or reconciliation step exists to
  prevent it, and none may be assumed.
- **Pre-storing the session is not a control.** A manual `market-data sync`
  would store a bar fetched without review. Comparing it with the evidence
  would read production values (G-1), and it is not an approved method.
- **So N-4 cannot be completed for 2026-10-09**, even if this procedure is
  accepted: an APPROVE could be recorded, but final-through cannot move past
  2026-10-08 until FG-9 is met. Because every later date includes
  2026-10-09, this blocks all further advancement for the October contract.
- Closing the gap is an engineering decision (for example, approving a
  specific candle version and refusing ingestion of any other). It is not
  implemented, and not part of this procedure.

## 10. Replay and accounting safeguards

- **Replay changes nothing persisted.** `operations daily` replays only the
  latest frozen cutoff, idempotently. Decisions are frozen once, order
  identities are deterministic, and fills are insert-only (one per order).
  A rerun adds no facts (acceptance C-2).
- **Determinism is not a safety guarantee, nor permission to replace
  history.** If the stored bar for a session differed from the one a
  decision saw, a recomputation would differ. That is why stores refuse
  differing bars instead of replacing them. Never delete or edit a bar,
  decision, order or fill to "re-run" a session.
- **Verification after acquisition is too late.** The run that acquires a
  session also decides on it and may fill a pending order at its OPEN
  (section 9, FG-9, step 7). A comparison afterwards can only detect a
  difference, never prevent it, and it reads Upstox-derived data (G-1,
  section 12). There is **no mechanism** that pins an approval to a specific
  candle version (section 15).
- **No corrections.** No automatic correction, cancellation, settlement or
  synthetic fill. Any change to persisted paper state is a separate decision.

## 11. October flatten and expiry dependencies

Verified against `FuturesExpiryFlattenGuard`
(`northstar-application/src/northstar_application/application_services/futures_expiry_flatten_guard.py`),
`plan_session_backlog` and `SimulateFuturesPaperOrderFillUseCase`. The dates
are E = 2026-10-27, E-6 = 2026-10-16 and E-5 = 2026-10-19 (2026-10-20 is a
holiday).

1. **The flatten decision needs approval through 2026-10-16**, and therefore
   approval of 2026-10-09 to 2026-10-15 first (section 2, items 3 and 4). Only
   a decision at the E-6 close requires the flatten, and only if the contract
   is not flat then. Decisions from 2026-10-09 to 2026-10-15 may change the
   position before that.
2. **A flatten decision creates a pending order, not a flat position.** The
   order fills at the OPEN of the first stored bar after it: the 2026-10-19
   bar, which needs approval through 2026-10-19 and a processed run.
3. **Only a persisted closing fill closes the position.** A decision is not
   an order, an order is not a fill, and a flat position line alone is not
   execution evidence. Confirm all of these, recorded as metadata:
   - the run log for the 2026-10-19 session shows the flatten order
     (`Expiry flatten: yes`) with `State: FILLED`;
   - `paper status` `Fills` is one higher than the count recorded before that
     run (the count comes from the persisted fill repository);
   - `Pending: 0`, and no position line for the contract.

   Where acceptance depends on the execution (X-3), the evidence is the
   persisted fill record, read only through an authorized read-only
   procedure. A reported flat position, or a replay, does not substitute.
4. **Acquisition after expiry fails.** After 2026-10-27 the contract leaves
   the instrument master, so its sessions can no longer be acquired.
5. **If the position or a flatten order survives expiry**, it is EXP-1 or
   EXP-2: follow the
   [Expiry Exception Operator Procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md).
   With S-2 deployed, an exposed contract reports exit 7 after its expiration
   date; production is not yet confirmed to run S-2.

**A revised candle must never be approved to produce the flatten.** If the
evidence is unavailable or FG-9 is unmet, final-through stays at 2026-10-08,
and the expiry exception path applies instead.

## 12. G-1 evidence restrictions

G-1 (Upstox retention and use permission) is unresolved.

**Metadata-only evidence** may be recorded in the decision record:
- the contract and session identifiers;
- observation times and endpoints;
- record counts;
- which fields changed (names only);
- check results;
- the decision, rationale, role and timestamps;
- the current and proposed final-through dates;
- pending-order **counts**, and whether a session is stored.

**Artifacts containing Upstox-derived values:**
- candle values;
- the evidence file;
- `report` output with values;
- `paper status` prices;
- dashboard market values;
- run logs that print values.

Their copying, export or retention is **not assumed** to be permitted. The
evidence guide's rule still holds: real evidence files stay private and local,
outside every repository, and never become fixtures. Whether they may be
**relied on** for an approval is part of FG-5.

**Pending is a valid state.** While G-1 is unresolved, a review may stay at
**DEFER**. The record then names the missing authorization.

## 13. STOP and escalation conditions

**STOP and leave final-through unchanged when:**
- any RC check is UNKNOWN, or any FG gate is unmet;
- the evidence cannot be produced or retained under G-1;
- the proposed date would approve an unreviewed session;
- no approved, technically supported method ensures the ingested candle is
  the reviewed one (FG-9), which is the case today;
- a run reports exit 5 (`FuturesHistoricalMarketDataConflictError`) or
  exit 4 (`FuturesDailySessionCoverageError`);
- an approval is being considered only to reach 2026-10-16 or 2026-10-19.

**ESCALATE to the approver when:**
- a revision has already been consumed, or the stored or used value differs
  from the reviewed one;
- an earlier approved session is found revised (RC-7);
- any accounting effect is possible;
- the contract approaches expiry without a decision (expiry procedure).

Never edit the database, delete a bar, change an order or fill, or bypass the
calendar.

## 14. Decision record template

One record per session, kept with the operator's records. It contains
metadata only (section 12); market values are referenced by evidence-file
location and line count, never copied.

| Field | Value |
|-------|-------|
| Session date and contract (RC-1) | |
| Provider and endpoints of the observations (RC-2) | |
| Evidence: file location (private), record count, observation span (RC-3) | |
| G-1 status relied on (FG-5) | |
| RC-4 completeness and validity | PASS / FAIL / UNKNOWN |
| RC-5 attribution and time zone | PASS / FAIL / UNKNOWN |
| RC-6 contract and OHLCV interpretation | PASS / FAIL / UNKNOWN |
| RC-7 earlier approved sessions | PASS / FAIL / UNKNOWN |
| RC-8 stored-bar state; pending-order count; affected decisions | |
| RC-9 changed fields (names only); last observed change; later historical observations | |
| Outcome | APPROVE / REJECT / DEFER / ESCALATE |
| Rationale | |
| Decided by (role) and timestamp (UTC) | |
| Current final-through; proposed final-through | |
| Runtime change authorization (FG-8): by whom, when; or "not authorized" | |
| Review-to-ingestion method (FG-9): the approved method; or "none: FG-9 unmet" | |

## 15. Known implementation gaps

| Gap | Consequence |
|-----|-------------|
| No per-session approval or exclusion | One unresolved session blocks every later one (October 9) |
| No approval of a candle **version** | The stored value is whatever the provider returns at acquisition, which may differ from the reviewed one, and the same run decides on it. FG-9 cannot be met (section 9) |
| No provenance on stored bars | No endpoint, acquisition time, version or hash; the version used cannot be proven afterwards from the database |
| No reconciliation of a storage conflict | An exit-5 revision of a stored bar blocks processing until a separately approved mechanism exists |
| No correction of persisted decisions, orders or fills | A consumed revision cannot be undone (section 7) |
| Evidence collection on the deployment machine is not wired, and is deferred by G-1 | Only operator-run development-machine evidence exists |
| No re-check of already processed sessions | A later revision of one is invisible unless observed |
| Mark approval is not recorded | The mark is the latest stored close (reconciliation U-6) |

Each is a possible future engineering decision. None is part of this
procedure.

## 16. Validation and readiness verdict

**Static validation** (development machine, 2026-10-11):
- every cited path, function, class and test exists;
- the behaviour described matches the code: the policy comparison,
  stop-at-first-non-final, insert-only conflict, coverage error, and the
  absence of provenance columns;
- no candle value is reproduced;
- no approval is implied;
- the documentation references resolve.

**Applying this procedure to 2026-10-09 today:**
- RC-2 is unresolved: the endpoint difference is not settled (E-7). The
  evidence is an unexplained discrepancy, not a confirmed revision.
- RC-8 is not established: stored-bar state and pending orders.
- FG-5 is unmet: G-1.
- FG-9 is unmet: no method ensures the ingested candle is the reviewed one.

**The expected outcome is therefore DEFER**, with final-through staying at
2026-10-08. This is not a recommendation to approve or reject; the decision
is the operator's. Even after a later APPROVE, final-through cannot advance
past 2026-10-08 until FG-9 is met.

**Verdict: READY FOR OPERATOR REVIEW.** N-4 stays PENDING until the procedure
is accepted. Accepting it would not complete N-4 for 2026-10-09 while FG-9
is unmet. X-4 and D-4 stay BLOCKED. B-1, B-2 and P-3 stay PENDING. G-1
stays unresolved.
