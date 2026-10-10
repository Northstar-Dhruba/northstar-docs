# Indian Futures: Expiry Exception Operator Procedure

Status: **proposed, not approved** (2026-10-10; updated for M1.4.3.2). The
operator procedure is not an approved policy. It describes what the code does,
how an operator detects the two expiry exceptions, and how to contain them
without editing data.

**Implemented detection (M1.4.3.2):** the code now reports
`STATUS: EXPIRY EXCEPTION` with exit 7 instead of `ROLLOVER REQUIRED` or
`WAITING` (corrections S-1, S-2 and S-3, section 12). This is verified only by
local synthetic tests (section 13). It is **not approved and not deployed**: a
production build that predates M1.4.3.2 still behaves as the "before
M1.4.3.2" notes below describe. Containment, escalation and reconciliation in
this document remain **proposed**. X-4 of the
[Paper MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md)
stays **BLOCKED** (section 9).

Related: [Paper MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md)
(section 13, EXP-1 and EXP-2), [Indian Futures Deployment Runbook](Indian-Futures-Deployment-Runbook.md)
(sections 5, 8, 11, 13 and 18), [Windows SQLite Backup and Isolated Restore Drill](Indian-Futures-Windows-SQLite-Backup-and-Restore-Drill.md).

Source paths below are relative to the Northstar workspace root.

## 1. Purpose and scope

The paper portfolio operates one dated contract. For the October contract
(NIFTY / NSE / 2026-10-27, strategy `directional-mvp-v1-nifty-paper-ops`,
portfolio `nifty-paper-ops-oct26`) this procedure covers:

- **EXP-1:** a paper position in the contract survives its expiry, because the
  closing (flatten) execution was not completed;
- **EXP-2:** a paper order in the contract stays pending because no later
  session of the contract will ever be stored to fill it.

It does not cover live trading, settlement, rollover automation or any change
to finality.

## 2. Preconditions

- The operator can run the existing read-only commands of section 7 on the
  deployment machine.
- The operator knows the real calendar date in IST. Northstar reads no clock
  for eligibility or finality. From M1.4.3.2 the daily operation compares the
  Asia/Kolkata date with the expiration date, but only while the contract is
  exposed (S-2, section 12). The early warning (section 4) and the dashboard
  still depend on **the operator's own date**.
- No operator action in this procedure approves a market-data session,
  changes finality, runs a writer, or edits the database.

## 3. What the code does today

Verified in code unless stated otherwise. "Before M1.4.3.2" describes a build
without the S-1 and S-2 corrections, which may still be the deployed one.

| Topic | Behaviour | Source |
|-------|-----------|--------|
| Contract identity | One exact dated contract from configuration; the expiration must be a session of its venue | `northstar-api/src/northstar_api/settings.py`; `FuturesExpiryFlattenGuard.assess` in `northstar-application/src/northstar_application/application_services/futures_expiry_flatten_guard.py` |
| Calendar | NSE sessions from versioned reference data; weekends and holidays are not sessions; unknown years and the untimed 2026-11-08 Muhurat session fail closed | `NSEFuturesTradingSessionResolver` in `northstar-infrastructure/src/northstar_infrastructure/market_data/nse_futures_session.py` |
| K = 5 flatten | A decision at the E-6 close requires the flatten; it fills at the E-5 open; from E-5 to E no signal may reopen; a flatten closes the full position and never reverses | `FuturesExpiryFlattenGuard`; `CreateFuturesExecutionIntentUseCase._flatten` in `northstar-application/src/northstar_application/application_services/create_futures_execution_intent.py`; `NSE_EXPIRY_FLATTEN_SESSIONS = 5` in `northstar-api/src/northstar_api/runtime.py` |
| Next-session fill | An order fills at the OPEN of the first stored daily bar strictly after its decision; until then it is pending. There is no cancel, expire or settle | `SimulateFuturesPaperOrderFillUseCase` in `northstar-application/src/northstar_application/application_services/simulate_futures_paper_order_fill.py` |
| Persistence | Orders and fills are insert-only rows in `futures_paper_orders` and `futures_paper_fills`; a fill cannot be stored before its order | `northstar-infrastructure/src/northstar_infrastructure/persistence/sqlite_futures_paper_trading.py` |
| Positions | Derived from fills, never stored | `BuildFuturesPaperPortfolioUseCase` in `northstar-application/src/northstar_application/application_services/build_futures_paper_portfolio.py` |
| WAITING | Processing stops before the first session that is not approved as final. It prints `STATUS: WAITING`, exits 0 and makes no provider request. With nothing open in the contract it reads no clock. **Before M1.4.3.2** this continued indefinitely, including after the real expiry date with a position open. From M1.4.3.2 an exposed contract past its expiration date reports `EXPIRY EXCEPTION` instead | `plan_session_backlog` in `northstar-api/src/northstar_api/operations.py`; `_chronological_daily_operation` in `northstar-api/src/northstar_api/cli.py`; test `test_india8b_nse_chronological_operations.py::test_waiting_reads_no_clock_and_makes_no_provider_request` |
| ROLLOVER REQUIRED | When every session through E has been decided, the run prints `STATUS: ROLLOVER REQUIRED` and exits 3. **Before M1.4.3.2** it did not check flatness. From M1.4.3.2 it is printed only when C is flat with no pending order; otherwise the run reports `EXPIRY EXCEPTION` (S-1) | `_chronological_daily_operation` (`backlog.exhausted` branch) in `cli.py` |
| EXPIRY EXCEPTION (M1.4.3.2) | `STATUS: EXPIRY EXCEPTION`, exit 7 (`EXPIRY_EXCEPTION`), with the affected contract, its expiration date, the position (`Flat` or `OPEN LONG/SHORT n @ price`), `Pending orders: n` and one `PENDING` line per order. It is reported before any provider request and writes nothing | `_contract_exposure` and `_expiry_exception` in `cli.py`; `contract_exposure`, `india_calendar_date` and `is_past_expiry` in `operations.py`; `expiry_exception_lines` in `northstar-api/src/northstar_api/_cli_rendering.py` |
| After expiry | Acquisition resolves the contract through the current Upstox instrument master, which "lists only currently tradable contracts, so an expired contract cannot be resolved". The run then fails with exit 6 (PROVIDER). From M1.4.3.2 an exposed contract past expiry reports exit 7 before any acquisition, so exit 6 can no longer hide it | `resolve_future_in_master` in `northstar-infrastructure/src/northstar_infrastructure/market_data/upstox_instrument_master.py` |
| Read-only report | `northstar paper status` lists orders, fills, `Pending: n`, one `PENDING <contract>: <side> <n> decided <instant> ID <id>` line per pending order, positions per contract, and gross P&L. It takes no lock and records nothing | `_paper_status` in `cli.py`; `execution_summary_lines` and `portfolio_lines` in `northstar-api/src/northstar_api/_cli_rendering.py` |
| Dashboard | `GET /futures/dashboard` shows positions, pending orders, backlog stage, finality and an expiry-safety block. The window is computed from the **latest decision**, not from the real date. From M1.4.3.2: stage `expiry_exception` (S-1), `expiry.pending_orders`, and `expiry.assessed_as_of` with the label "Window assessed as of the latest decision ..., not today's date" (S-3). The dashboard reads no clock, so it does not apply S-2 | `operational_status` and `_expiry` in `northstar-api/src/northstar_api/operational_status.py` |
| Windows wrapper (M1.4.3.2) | Exit 7 is recorded in `last-run.json` as outcome `EXPIRY_EXCEPTION`, exit class `EXPIRY_EXCEPTION`, never `WAITING` | `northstar-api/deploy/india/windows/Invoke-NorthstarIndiaOperations.ps1` |
| Exception detection | **Before M1.4.3.2: none.** From M1.4.3.2: S-1 and S-2 in the daily operation, S-1 on the dashboard | as above |

### How an exception arises today

- **Under scheduled operation alone, the flatten completes** whenever the
  sessions through E-5 are approved and processed. The runner decides every
  approved session in order, so the E-6 flatten is decided and fills at E-5.
- **EXP-1 and EXP-2 arise in production when processing stops before E-5.**
  This happens when finality is not approved in time, or when the sessions are
  approved only after the contract can no longer be acquired. The position
  stays open; if E-6 was processed, its flatten order stays pending.
- **They also arise through manual `paper run` cutoffs that skip sessions.**
  If the flatten is first decided at E, it can never fill. Before M1.4.3.2 a
  later `operations daily` then reported `ROLLOVER REQUIRED` while the
  position was open; from M1.4.3.2 it reports `EXPIRY EXCEPTION`. The test
  `test_india7_nifty_incremental_operations_acceptance.py::test_skipping_the_e6_cutoff_fills_the_flatten_one_session_late`
  pins the milder case.

### October 2026 dates

E = 2026-10-27; E-6 = 2026-10-16 (flatten decision); E-5 = 2026-10-19 (flatten
fill; 2026-10-20 is a holiday). The normal flatten therefore needs the
sessions through 2026-10-19 approved and processed while the contract can
still be acquired. This document never asks for a session to be approved to
meet that date; approvals follow only the normal finality procedure
(runbook section 5).

## 4. EXP-1 detection

**Entry conditions.** For the configured contract C (expiry session E) and
portfolio P, EXP-1 exists when **both** hold:

1. `paper status` for P, at a cutoff on or after E's close, shows an open
   position in C. The position comes from fills in `futures_paper_fills`, so
   an open position means no completed closing fill was recorded.
2. Either:
   - (a) the real date (IST) is after E, so the contract has expired; or
   - (b) `operations daily` reports `ROLLOVER REQUIRED`, so every session
     through E has been decided.

**Early warning (not yet EXP-1).** The real date is on or after E-6
(2026-10-16), a position in C is open, and the latest decided session is
before E-5. The normal flatten can still complete if the sessions are approved
and processed in time.

**How it differs from normal states:**

| What the operator sees | Normal | EXP-1 |
|------------------------|--------|-------|
| `STATUS: EXPIRY EXCEPTION`, exit 7 (M1.4.3.2) | Never normal | Reported when C is not flat or has a pending order, and either no session is left through E (S-1) or the India date is after E (S-2) |
| `STATUS: WAITING`, exit 0 | The real date is on or before E, or C is flat | Before M1.4.3.2: the real date is after E **and** C is not flat, and the system reports exactly the same thing. From M1.4.3.2 this is exit 7 instead |
| `STATUS: ROLLOVER REQUIRED`, exit 3 | C is flat and `Pending: 0` | Before M1.4.3.2: C is not flat, or an order in C is pending, and the system reports exactly the same thing. From M1.4.3.2 this is exit 7 instead |
| Exit 6 with "an expired contract cannot be resolved" | Not normal for an active contract | Confirms that C can no longer be processed; EXP-1 if C is not flat. From M1.4.3.2 an exposed contract past E gets exit 7 first |

Until M1.4.3.2 is deployed and verified on the deployment machine, apply the
"before M1.4.3.2" column: check every WAITING and ROLLOVER REQUIRED result
with `paper status` and the real date.

**Completed versus intended flatten.** A flatten is completed only when a
fill for its order exists. That means `paper status` shows no position in C
and no `PENDING` line for C, and the dashboard shows `position_flat: true`.

The following are **not** proof that the flatten completed:

- a SELL or BUY flatten decision;
- the run output line `Expiry flatten: yes`;
- an order with no fill;
- the dashboard expiry window.

## 5. EXP-2 detection

**Entry conditions.** EXP-2 exists when `paper status` for P shows a
`PENDING <C>: ...` line and no later session of C will ever be stored for it.
That is the case when either:

- (a) the order was decided at E's close, so no later session exists; or
- (b) the real date is after E and the session after the order's decision was
  never processed.

The order is a row in `futures_paper_orders` with no row in
`futures_paper_fills`.

**Normal pending order (not EXP-2).** The E-6 flatten order is pending from
the 2026-10-16 close until the 2026-10-19 session is processed. That is
expected while the real date is on or before E.

EXP-2 usually comes with EXP-1: in practice the stranded order is the flatten
of a still-open position. Handle them together.

## 6. Operator containment (proposed)

Use only existing, documented controls. Each preserves every persisted fact.

1. **Do not roll over.** Leave `NORTHSTAR_FUTURES_EXPIRATION`,
   `NORTHSTAR_PORTFOLIO` and the strategy unchanged while the exception is
   open. Rolling over starts a new portfolio and leaves the exception out of
   view.
2. **Do not run any manual writer** for C: no `paper run`, `market-data sync`
   or `economics set`.
3. **Do not approve a session to resolve the exception.** Approval follows only
   the normal finality procedure (runbook section 5).
4. **Optionally, stop processing:**
   - disable the scheduled task (runbook section 18: `Disable-ScheduledTask`);
     or
   - set `NORTHSTAR_FUTURES_DAILY_BAR_FINALITY=disabled` (runbook section 5).

   With finality disabled, runs report WAITING (UNKNOWN). From M1.4.3.2 an
   exposed contract past its expiry still reports EXPIRY EXCEPTION (exit 7),
   because the check runs whatever the finality mode. Neither control changes
   any fact. Which one to use is an operator decision (D-EXP-5).
5. **Never edit the database**, never delete or insert orders, fills or
   decisions by hand, and never restore a backup over production to "undo"
   anything.

## 7. Read-only inspection commands (verified to exist; not run for this document)

All commands are from existing code or the runbook. Run them when no
scheduled run is in progress. Until M1.2 is deployed, a report taken during a
write can fail transiently; if it does, run it again.

**Last scheduled run** (runbook section 18 E):

```powershell
Get-ScheduledTaskInfo -TaskName 'Northstar India Operations' | Select-Object LastRunTime, LastTaskResult, NextRunTime
Get-Content -LiteralPath "$env:LOCALAPPDATA\Northstar\india-operations\last-run.json"
```

From M1.4.3.2, `LastTaskResult` 7 and `"outcome": "EXPIRY_EXCEPTION"` mean the
run detected an expiry exception; its log holds the `STATUS: EXPIRY EXCEPTION`
report.

**Configured contract** (only these non-secret lines of `.env` are read):

```powershell
Select-String -LiteralPath 'C:\Local Disk(E)\Codes\Northstar\northstar-api\deploy\india\.env' -Pattern '^\s*NORTHSTAR_(FUTURES_EXPIRATION|PORTFOLIO|STRATEGY|FUTURES_DAILY_BAR_FINALITY|FUTURES_FINAL_THROUGH)\s*='
```

**Paper status** (runbook section 4E). This starts a one-off `india-operations`
container that runs only the read-only report: it takes no lock, makes no
provider request, and creates no order, fill or decision. Use a cutoff after
E's close; `2026-10-27T18:30:00+00:00` is midnight IST after the October
expiry.

```powershell
cd 'C:\Local Disk(E)\Codes\Northstar\northstar-api\deploy\india'
docker compose run --rm --no-deps india-operations northstar paper status --database /data/northstar.sqlite3 --strategy directional-mvp-v1-nifty-paper-ops --portfolio nifty-paper-ops-oct26 --as-of 2026-10-27T18:30:00+00:00
```

Read `Pending:`, every `PENDING NIFTY@NSE 2026-10-27: ...` line, and the
position lines under `PORTFOLIO (all contracts)`.

**Dashboard** (runbook section 8), if the stack's `india-web` is running:
`GET /api/futures/dashboard` shows positions, pending orders, the backlog
stage and `expiry.position_flat` (from M1.4.3.2 also stage `expiry_exception`
and `expiry.pending_orders`). Do not rely on its expiry `window` after the
real expiry date: it is assessed as of the latest decision; see section 14.

**Decisions, orders, fills and positions are different facts.** Read each
from the right place:

| Fact | What it is | Read-only source | Limits |
|------|------------|------------------|--------|
| Strategy decisions | Frozen research decisions (BUY, SELL, HOLD, including `Expiry flatten: yes`). A decision is **not** an execution | `operations daily` output (`Latest frozen decision:`); dashboard `recent_decisions` | The dashboard shows the latest 10 only |
| Pending orders | Persisted orders with no persisted fill | `paper status`: `Pending: n` and one `PENDING <contract>: <side> <n> decided <instant> ID <order id>` line per order | Complete for the portfolio at the cutoff |
| Persisted fills | Simulated executions; only a fill changes a position | `paper status`: `Fills: n`, the count only | See the limitation below |
| Position | Derived from the persisted fills, never stored | `paper status` under `PORTFOLIO (all contracts)`: direction, contracts and average entry per contract, or `Flat` | Complete for the portfolio at the cutoff |

**Limitation: no read-only command shows the details of each fill.** No
existing read-only interface lists each persisted fill's identity, timestamp
and price. As a result, the identity, time and price of the last fill cannot
be established authoritatively today.
- `paper status` gives only the count.
- `/api/futures/analysis` gives counts and derived trade aggregates (opening
  and closing times, average entry and exit), not fill records.
- The dashboard's `recent_decisions` attach a fill's price and instant to the
  orders of the latest 10 decisions only, without the fill identity. They are
  context, **not** evidence of executed fills.
- `paper run` prints fill details but is a writer: never use it to inspect.

What can be established is whether C is flat and whether any order in C is
pending (`paper status`). Never query or open the database to get fill
details. A read-only fill listing would be new software and needs its own
approved milestone.

## 8. Evidence to preserve

Keep evidence on the deployment machine, in a new dated directory under
`C:\Local Disk(E)\Codes\NorthstarEvidence`, and never in a repository:

- the `paper status` output from section 7;
- `last-run.json` and the run logs of the runs that reported WAITING,
  ROLLOVER REQUIRED, EXPIRY EXCEPTION (exit 7) or exit 6;
- the configured-contract lines from section 7;
- the real date and time of each observation, and who made it.

These are text outputs; no database copy is needed. They do contain a few
provider-derived numbers (fill quotes and the mark close), and the run logs
contain more. Retaining them is subject to open gate G-1 (acceptance
specification 11.1). A database backup made with the
[restore drill](Indian-Futures-Windows-SQLite-Backup-and-Restore-Drill.md) is a
full copy of Upstox-derived data and is likewise subject to G-1.

## 9. Escalation and operator decisions

An exception is escalated to the approver at once, with the evidence. Until
approved policies exist, every resolution step is a recorded decision:

| ID | Decision |
|----|----------|
| D-EXP-1 | Approve, amend or reject the EXP-1 and EXP-2 policies (acceptance specification 13.7) |
| D-EXP-2 | Approve or reject the implemented fail-closed correction S-1 (section 12), and its deployment |
| D-EXP-3 | Approve or reject the implemented real-time expiry check S-2 (section 12): an Asia/Kolkata date read in the daily operation, only while the contract is exposed. The alternatives were procedure only or the Windows wrapper |
| D-EXP-4 | Evidence retention under G-1 |
| D-EXP-5 | Which containment control to use (section 6, step 4) |
| D-EXP-6 | The reconciliation or settlement method for a surviving position or stranded order, which does not exist yet |
| D-EXP-7 | Whether the next contract may start while an exception on the previous one is open (this also depends on the November calendar blocker, runbook section 13) |

## 10. Returning to normal operation (proposed)

Detection keeps reporting exit 7 on every run while the contract is exposed;
nothing in the code clears it. The affected contract returns to normal only
when one of these is recorded:

- **No exception.** `paper status` shows C flat and no pending order in C.
  For example, the flatten completed after normal approvals.
- **Exception resolved.** An approved reconciliation or settlement procedure
  (D-EXP-6) was carried out and verified. That procedure does not exist yet.

Rollover to the next contract follows runbook section 13 only after one of
these.

### Contract-transition gate (proposed, not approved; D-EXP-7)

S-1 and S-2 watch only the configured contract, so once `.env` names the next
contract and portfolio, nothing reports an exception left on the previous one
(section 14). Before editing `.env` for the next contract, the operator records
all of the following, with the date, time and their name:

| Gate | Check | Passes when |
|------|-------|-------------|
| TG-1 | The latest `operations daily` run on the **current** configuration (previous contract C) | Exit 3, `STATUS: ROLLOVER REQUIRED`, and `last-run.json` outcome not `EXPIRY_EXCEPTION`, from a deployed **S-1-protected** build: the deployment record names the deployed `northstar-api` build as 545b3b6 (M1.4.3.2) or a later `develop` commit. There is no need to wait for the calendar expiry date; ROLLOVER REQUIRED itself means every session of C through its expiry has been decided. If the deployed build is pre-M1.4.3.2 or cannot be identified, exit 3 says nothing about flatness: record TG-1 as **not authoritative** |
| TG-2 | `paper status` for the current portfolio (section 7), with `--as-of` no earlier than C's latest frozen decision (printed by the TG-1 run as `Latest frozen decision:`) | Required in every case, independently of TG-1: no position line for C under `PORTFOLIO (all contracts)` (zero net position) and no `PENDING <C>:` line (zero pending orders in C). Any position or pending order in another contract is a STOP (section 11) |
| TG-3 | Dashboard, if running (section 7) | Corroboration only, never sufficient alone: `position_flat: true`; with M1.4.3.2 deployed, also stage `rollover_required` and `pending_orders` 0 |
| TG-4 | Evidence (section 8) | The TG-1 to TG-3 outputs, the deployed build identity used for TG-1 (or "not established"), and the next portfolio identity, which differs from the current one |
| TG-5 | Approval | The approver acknowledges TG-1 to TG-4 before `.env` is changed, stating whether TG-1 was authoritative. Without an authoritative TG-1, the approver accepts in writing that the transition rests on TG-2 alone |

If any gate fails, do not change the configuration: follow sections 6 to 9.

`paper status` takes the portfolio as an argument rather than reading `.env`,
so after the transition the operator can still check the previous portfolio.
That check is manual; nothing runs it automatically.

**Assessment (M1.4.3.3, for the approver to decide).** For the internal paper
MVP, a procedural gate is a workable minimum **only once M1.4.3.2 is
deployed** and its deployed build is recorded. TG-1 is then backed by S-1: the
operation reports ROLLOVER REQUIRED only for a resolved contract, and TG-2
confirms it independently. Before deployment, TG-1 is not authoritative and
the gate rests on TG-2 alone. What it cannot prevent is an operator who edits `.env` despite exit
7. Software enforcement would close that gap: for example, `operations daily`
could refuse to start a new contract while another contract of the same
strategy in the same database still holds a position or pending order. That
would be a read-only check across portfolios, with no settlement. It is a
**proposal only**, not implemented. Whether the procedural gate is sufficient
is decision D-EXP-7.

## 11. STOP conditions

Stop and escalate, without changing anything, when:

- any command in section 7 fails or reports an error;
- the configured contract or portfolio differs from the one under
  investigation;
- `paper status` shows a position or pending order in a contract other than
  the configured one;
- an action would edit the database, create an order, fill or decision,
  restore over production, roll over, or approve a session to resolve the
  exception;
- any step would require copying production data outside the deployment
  machine while G-1 is unresolved.

## 12. Recovery and reconciliation boundaries

**Allowed:**
- read-only inspection (section 7);
- the containment controls (section 6);
- normal approvals through the normal finality procedure;
- recording evidence.

**Not allowed:**
- editing, deleting or inserting rows;
- fabricating a fill, closing trade or settlement price;
- marking an order cancelled or expired, because the domain has no such state;
- automatic rollover;
- restoring a backup over production;
- treating a flatten decision or a pending order as a completed close.

**Feasibility verdict (M1.4.3.1, before the corrections).** Procedure-only
enforcement is **not sufficient** to close X-4:

| Question | Answer before M1.4.3.2 | With M1.4.3.2 (local tests only) |
|----------|------------------------|----------------------------------|
| A. Can an operator reliably detect EXP-1 and EXP-2 with existing tools? | **Partly.** Deliberate inspection can (sections 4, 5 and 7), but only using the operator's calendar. Nothing in the system flags either exception. | The daily operation flags both, by S-1 and S-2 |
| B. Can processing be contained without editing the database? | **Yes**, with the controls in section 6. | Unchanged |
| C. Can evidence be preserved without a new copy of production Upstox-derived data? | **Partly.** No database copy is needed, but text outputs contain some provider-derived values (G-1). | Unchanged |
| D. Can the system continue normally despite an unresolved expired position or order? | **Yes.** WAITING exit 0 continues indefinitely after real expiry, and ROLLOVER REQUIRED can be reported while non-flat. | No: every run reports exit 7 until the exception is resolved |
| E. Are status outputs sufficient to prevent a misleading success indication? | **No.** See section 14. | The daily operation and the wrapper: yes. The dashboard: S-1 only (section 14) |

**Fail-closed software corrections (implemented in M1.4.3.2; not approved, not deployed):**

- **S-1 (fact-based, no clock).**
  - Where: before printing `ROLLOVER REQUIRED`, in the `backlog.exhausted` branch of `_chronological_daily_operation` (`northstar-api/src/northstar_api/cli.py`), and in the stage logic of `operational_status` (`northstar-api/src/northstar_api/operational_status.py`).
  - What: `_contract_exposure` reads the configured contract's position and pending orders through the existing paper snapshot (`GetFuturesPaperTradingSnapshotUseCase`) at the later of the latest decision and the latest stored bar. If C is not flat or has a pending order, the run reports `STATUS: EXPIRY EXCEPTION` with exit 7 and no rollover instruction; the dashboard reports stage `expiry_exception`.
  - Writes: nothing, and fabricates nothing. The existing idempotent replay of the latest cutoff still runs first, as before.
- **S-2 (real-time awareness).** Before WAITING or any acquisition, if C is not flat or has a pending order, the daily operation reads its clock once, takes the Asia/Kolkata calendar date, and reports `EXPIRY EXCEPTION` (exit 7) when that date is strictly later than `NORTHSTAR_FUTURES_EXPIRATION`. The same reading then routes Upstox's current-day endpoint, so a run still reads the clock at most once. Eligibility, finality and strategy decisions still read no clock. With nothing open in C, no clock is read (`test_waiting_reads_no_clock_and_makes_no_provider_request` still passes). This is option (c) of M1.4.3.1, but it fails closed rather than being advisory. Options (a) procedure only and (b) the Windows wrapper were not implemented. Approval is D-EXP-3.
- **S-3 (presentation).** The dashboard labels the expiry window "Window assessed as of the latest decision (...), not today's date" and exposes `assessed_as_of` and `pending_orders`.
- **Wrapper.** `Invoke-NorthstarIndiaOperations.ps1` maps exit 7 to outcome and exit class `EXPIRY_EXCEPTION`. No scheduler configuration changed.

## 13. Synthetic isolated verification

M1.4.3.2 implemented this plan as automated tests in
`northstar-api/tests/test_india_expiry_exception_detection.py`, which also
cover each position and pending-order combination, the expiration-date
boundary in Asia/Kolkata, and an expired instrument. They passed on the
development machine on 2026-10-10. That is a local engineering result: it is
not a production verification and does not satisfy X-4 on its own.

The tests use only the existing synthetic acceptance infrastructure in
temporary directories:
- `_expiry_market`, `Operator` and `_expiry_operator` in
  `northstar-api/tests/test_india7_nifty_incremental_operations_acceptance.py`;
- `_daily`, `_operator` and `_daily_reference` in
  `northstar-api/tests/test_india8b_nse_chronological_operations.py`.

They use a fake Upstox transport with the network refused, seeded prices and
the real NSE calendar. No production database, Docker, provider or real
finality setting is involved. The clock is injected; the test machine's clock
is never read.

| ID | Scenario | Setup | Before M1.4.3.2 | With M1.4.3.2 (tested) |
|----|----------|-------|-----------------|------------------------|
| V-1 | EXP-1, stalled finality | LONG 1 filled at the E-7 open; final-through E-7; injected date 2026-11-02 | WAITING, exit 0 | `EXPIRY EXCEPTION`, exit 7, `OPEN LONG 1`, `Pending orders: 0`; one clock read; no provider request; no facts change |
| V-2 | EXP-2 with EXP-1, manual skip | Manual `paper run` cutoffs that skip from E-7 to E, so the flatten is decided at E | ROLLOVER REQUIRED, exit 3 | `EXPIRY EXCEPTION`, exit 7, `OPEN LONG 1`, `Pending orders: 1`; no clock read; `paper status` agrees (`Pending: 1`); dashboard stage `expiry_exception` |
| V-3 | Completed flatten | Manual cycles through E | ROLLOVER REQUIRED, exit 3 | Unchanged; dashboard stage `rollover_required`, `pending_orders` 0 |
| V-4 | Normal WAITING before expiry | As V-1, injected date 2026-10-15 | WAITING, exit 0 | Unchanged (one clock read, because the contract is exposed) |
| V-5 | Normal ROLLOVER REQUIRED | Chronological operation through E, then a rerun | ROLLOVER REQUIRED, exit 3 | Unchanged; no clock read |

Also tested: a flat contract with a pending order after expiry, and an open
position without one (exit 7); no position and no pending order after expiry
(WAITING, exit 0, no clock read); 23:59 IST on the expiration date (WAITING)
against 00:00 IST the next day, while the UTC date is still the expiration date
(exit 7); an expired instrument absent from the master (exit 7 before any
request, not exit 6); the same failure before expiry (still exit 6); and the
wrapper's `EXPIRY_EXCEPTION` outcome. Every exception case asserts that
market data, decisions, orders and fills are unchanged.

### M1.4.3.3 operator exercise (synthetic, 2026-10-10)

An isolated exercise on the development machine repeated V-1 to V-5 and the
additional cases through the production CLI and runtime. It also walked the
operator steps of sections 4 to 9. Setup:

- `northstar-api` 545b3b6 (feature 71c32c1); `northstar-web` 4f6908c (feature
  5862618); `northstar-docs` 99f8517 (feature d230fd3).
- Temporary SQLite databases built by the fake-Upstox helpers above, removed
  afterwards.
- Injected clocks and a refused network. No Docker, provider, production
  database, credential, scheduler or finality setting was used.

**Scenarios.** A full SQL dump of each database was hashed before and after
every detection command, and was identical in every scenario: no decision,
order, fill or market bar was written, and no fill was fabricated or order
cancelled.

| Case | Observed |
|------|----------|
| V-1 | Exit 7 `EXPIRY EXCEPTION`, `OPEN LONG 1`, `Pending orders: 0`, one clock read, no request. A second run: exit 7 again, so nothing clears it |
| V-2 | Exit 7, `OPEN LONG 1`, `Pending orders: 1`, no clock read |
| V-3 | Exit 3 `ROLLOVER REQUIRED`, no exception |
| V-4 | Exit 0 `WAITING` before expiry |
| V-5 | Exit 3 `ROLLOVER REQUIRED` |
| Flat with a pending order | Exit 7 |
| 23:59 IST on E | Exit 0 |
| 00:00 IST on E+1 | Exit 7 |
| Nothing open after expiry | Exit 0 |
| Expired instrument | Exit 7, not 6, with no request |
| Finality `disabled` after expiry | Exit 7 |

**Operator steps.**

| Step | Result |
|------|--------|
| Recognise exit 7 | Verified with the real wrapper and a fake `docker` replaying the CLI's actual V-2 output: wrapper exit 7, `last-run.json` outcome and exit class `EXPIRY_EXCEPTION`, no secret logged. The real Task Scheduler result was **not verified** |
| Identify the contract | Verified: the report's `Affected contract:` line, and the section 7 `Select-String` pattern on a synthetic `.env` returns only the five named settings, never the token |
| Inspect position and pending orders | Verified with `paper status` run locally (the section 7 command without `docker compose`): `Pending: 1`, the `PENDING` line, `LONG 1`; no write |
| Identify the last actual fill | **Not verified.** No read-only interface lists each fill's identity, timestamp and price; the dashboard's latest-10-decision view is not fill evidence (section 7 limitation). Flatness and pending orders were established with `paper status` |
| Intended versus completed flatten | Verified: V-2's flatten decision with a pending order and an open position, against V-3's `Pending: 0` and `Flat` |
| Suspend processing | Partly verified: finality `disabled` stops acquisition and still reports exit 7. `Disable-ScheduledTask` exists but was **not run**; real suspension is **not verified** |
| Preserve evidence | Verified on synthetic text output, with no secret |
| Escalate | A human step with no tool; database state verified unchanged throughout |

**D-EXP-7.** On the same database, the dashboard configured for the next
contract shows stage `go_live_required`, and `paper status` for the next
portfolio shows `Orders: 0`, `Pending: 0`, `Flat`. Neither mentions the
unresolved October contract, confirming the gap described in section 14. The
proposed transition gate is in section 10.

These are synthetic development results. They are not a production
verification and grant no approval.

## 14. Known limitations and unresolved design decisions

- **Until M1.4.3.2 is deployed**, every limitation found in M1.4.3.1 applies
  in production:
  - WAITING continues with exit 0 after the expiry date, and the Windows task
    records `WAITING` and `LastTaskResult` 0;
  - `ROLLOVER REQUIRED` does not check flatness or pending orders;
  - after expiry, exit 6 ("an expired contract cannot be resolved") looks like
    a provider outage unless read together with the date.
- The S-2 check depends on the deployment machine's clock being correct. A
  wrong clock delays or advances the exception by the error; it never changes
  eligibility, finality or any decision.
- The dashboard reads no clock and applies only S-1. Its expiry window is
  computed from the latest decision; with final-through stuck on 2026-10-08 it
  shows `Outside flatten window` even after 2026-10-27, now explicitly labelled
  as assessed as of the latest decision.
- Unrealized P&L of a surviving position stays marked at the last stored
  close. `paper status` and the dashboard P&L carry no expiry warning; only the
  daily operation reports the exception.
- `paper run` (manual) has no expiry check; only `operations daily` does.
- **Previous-contract visibility gap (D-EXP-7, open, not solved).** S-1 and
  S-2 inspect only the currently configured contract. Reconfiguring the
  deployment to a new contract and portfolio removes the previous contract
  from ordinary daily-operation and dashboard monitoring. An unresolved
  position or pending order in the previous contract is not carried forward as
  an expiry-exception blocker. Reconfiguration therefore stays prohibited
  until the previous contract is explicitly verified as resolved (section 10),
  or an approved exception-management procedure permits the transition.
  Manual `paper run` and `paper status` provide no automatic post-expiry
  detection. This remains open for X-4 acceptance. M1.4.3.3 confirmed it on
  synthetic data (section 13). A proposed transition gate is in section 10.
- No reconciliation, settlement, cancel or expire mechanism exists. A
  surviving position or stranded order can only be recorded, not resolved.
- Decisions D-EXP-1 to D-EXP-7 (section 9) are open.
