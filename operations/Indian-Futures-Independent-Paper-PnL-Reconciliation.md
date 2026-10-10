# Indian Futures: Independent Paper P&L Reconciliation

Status: **audit (M1.4.4.1) and synthetic verification (M1.4.4.2)**,
2026-10-10.
- **Audited accounting rules:** sections 2, 3 and 5 state verified
  implementation facts, each with its source.
- **Implemented verifier:** section 4's calculation is implemented as an
  independent verifier and executed on synthetic data only. Results are in
  section 15.
- **Proposed policy:** sections 6 to 14 remain proposed and **not approved**.
  This includes the provisional precision threshold (U-3) and every
  unresolved decision U-1 to U-8.
- **Not reconciled:** nothing here marks production P&L reconciliation
  complete. Requirement P-3 of the
  [Paper MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md)
  stays PENDING.

Related:
- [Paper MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md)
  (sections 6, 11.1 and 14);
- [Paper MVP Evidence Register](Indian-Futures-Paper-MVP-Evidence-Register.md);
- [Expiry Exception Operator Procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md)
  (section 7: read-only facts).

Source paths are relative to the Northstar workspace root. Code was read at:

| Repository | Commit |
|------------|--------|
| `northstar-core` | e54ccdf |
| `northstar-application` | 9a0ab57 |
| `northstar-infrastructure` | aa94428 |
| `northstar-api` | 545b3b6 |
| `northstar-docs` | 1d9bd0c |

## 1. Purpose and scope

This document defines how Northstar's futures paper accounting can be checked
by a calculation that shares none of its accounting code. It covers:

- positions, average entry, gross realized and gross unrealized P&L for
  futures paper portfolios;
- single and multiple contracts, LONG and SHORT, partial closes, reversals,
  pending orders and expiry.

It does not cover:

- live trading;
- fees, net P&L, settlement or cash, none of which is modelled (section 6);
- any production reconciliation, which G-1 currently prevents (section 13).

## 2. Authoritative accounting definitions (verified)

| # | Rule | Definition in code | Source |
|---|------|--------------------|--------|
| 1 | Order creation | A frozen decision names a **target exposure**: BUY means +N, SELL means -N, HOLD means no order. The order is the difference between the target and the current net exposure, so it may cross zero. In the pre-expiry flatten window the target is 0 whatever the action. | `CreateFuturesExecutionIntentUseCase` (and its `_flatten`) in `northstar-application/src/northstar_application/application_services/create_futures_execution_intent.py` |
| 2 | Order execution | An order decided at D fills at the **OPEN of the first stored daily bar strictly after D**, once that bar is stored; until then it is pending. There is no cancel, expire or partial fill. | `SimulateFuturesPaperOrderFillUseCase` in `.../application_services/simulate_futures_paper_order_fill.py` |
| 3 | Fill price and quantity | `fill_quote` is that bar's open. `filled_at` is the bar's own instant (its completion boundary). The quantity always equals the order's count: one full fill per order. | Same module; `FuturesPaperFill` in `northstar-core/src/northstar_core/paper_trading/value_objects/futures_paper_fill.py` |
| 4 | Units and lot size | Quantities are whole contracts (`FuturesContractCount`, a positive integer). There is **no separate lot size or multiplier**. One fact, the per-contract **point value**, converts quote points to currency. For NIFTY it is the lot (65) times 1 INR, i.e. 65 INR per point per contract. It is constant for a contract's life. | `FuturesContractEconomics` and `FuturesPointValue` in `northstar-core/src/northstar_core/futures/value_objects/` |
| 5 | Sign convention | BUY is `+count`, SELL is `-count`. A positive net is LONG, a negative net is SHORT, and flat is the **absence** of a position. | `_transition` in `.../application_services/build_futures_paper_portfolio.py` |
| 6 | Cost basis | A **contract-weighted average entry** per contract. It is not FIFO and has no lot tracking. | Same `_transition` |
| 7 | Opening, adding and closing | Starting flat, the position is the fill's delta at the fill quote. Adding in the same direction re-averages the basis. A partial close keeps the average. A full close removes the position. | Same `_transition` |
| 8 | Reversal | **Supported.** A fill larger than the open position closes it in full and opens the remainder at the fill quote; the old basis is discarded. | Same `_transition` |
| 9 | Realized P&L | Per closing fill, `(q - a) * sign(p) * closed` quote points. These are summed per contract and converted once by multiplying by the point value. Opening or adding realizes nothing; a reversal realizes only the closed part. | `CalculateFuturesRealizedPnlUseCase` in `.../application_services/calculate_futures_realized_pnl.py` |
| 10 | Unrealized P&L | `(m - a) * n * point_value` for each open position, with n the signed net contracts. | `ValueFuturesPaperPortfolioUseCase` in `.../application_services/value_futures_paper_portfolio.py` |
| 11 | Total P&L | **No portfolio total exists.** The valuation "deliberately has no portfolio total": it reports realized and unrealized per contract. The only total is per contract and per equity-curve point in the analysis: `total_pnl = realized_pnl + unrealized_pnl`. | `BuildFuturesPaperTradingValuationUseCase` in `.../application_services/build_futures_paper_trading_valuation.py` (module docstring); `CalculateFuturesPaperPerformanceUseCase` in `.../application_services/calculate_futures_paper_performance.py` |
| 12 | Gross and net | Gross only. Commissions, fees, slippage, financing and taxes are all excluded. | `build_futures_paper_trading_valuation.py` docstring; acceptance L-6 and P-4 |
| 13 | Fees, brokerage, taxes, slippage | **Not modelled.** | As 12 |
| 14 | Valuation mark | The **latest stored 1d bar's close** at or before the cutoff, with its instant reported. A stale mark is still used. With no such bar, the unrealized P&L is unavailable (`None`), never zero. | `ValueFuturesPaperPortfolioUseCase._last_close` |
| 15 | Snapshot cutoff | One cutoff governs everything. A fill is visible if `filled_at <= cutoff`; an order if `decided_at <= cutoff`; the mark is the latest bar `<= cutoff`. Instants are compared with `PointInTime.compare`, never as text. | `BuildFuturesPaperPortfolioUseCase.execute`; `GetFuturesPaperTradingSnapshotUseCase` (`_visible`) in `.../application_services/get_futures_paper_trading_snapshot.py` |
| 16 | Multiple contracts and rollover | Each dated contract has its own position, basis, economics and P&L row. Contracts never pool, and currencies are never summed. Nothing rolls automatically: rollover is a manual configuration change to a new portfolio (runbook section 13). | `build_futures_paper_portfolio.py`, `build_futures_paper_trading_valuation.py` docstrings |

**Precision (verified).**
- Every average, realized and unrealized calculation runs under an explicit
  `Context(prec=28, rounding=ROUND_HALF_EVEN)` (`_BASIS_CONTEXT` and
  `_PNL_CONTEXT` in the three modules above).
- The average entry is therefore rounded to 28 significant digits when it does
  not terminate. Realized P&L deliberately reuses that rounded basis.
- `Money` and `QuoteValue` canonicalize without rounding (`_canonical_decimal`
  in `northstar-core/src/northstar_core/foundation/value_objects/_canonical_decimal.py`).
- No amount is rounded to paise or to any other display precision.

**History validity (verified).** The fold refuses, and does not repair:
- a fill from another portfolio or strategy;
- a duplicate fill identity, or two fills for one order;
- fills not in canonical order: `filled_at` ascending by `PointInTime.compare`,
  then order identity (`_validate_fill_history`).

The fill repository returns exactly that order (`_compare_fills` in
`northstar-infrastructure/src/northstar_infrastructure/persistence/sqlite_futures_paper_trading.py`).

## 3. Persisted facts and lineage (verified)

| Table | Columns | Role |
|-------|---------|------|
| `futures_paper_orders` | `order_identity`, `portfolio_identity`, `product_code`, `exchange_code`, `expiration_date`, `side`, `contracts`, `strategy_identity`, `decided_at` | Insert-only; the source of a fill's contract, side, count and owner |
| `futures_paper_fills` | `fill_identity`, `order_identity` (UNIQUE), `fill_quote`, `filled_at` | Insert-only; one fill per order |
| `futures_ohlcv` | contract key, `timeframe`, `point_in_time`, `open_value` to `close_value`, `volume` | Insert-only; a revised bar is a conflict, never an update |
| `futures_contract_economics` | contract key, `point_value_amount`, `settlement_currency` | Insert-only frozen reference; different economics are a conflict |
| `futures_forward_research_records` | Frozen decisions | Decisions only; never executions |

Schemas: `northstar-infrastructure/src/northstar_infrastructure/persistence/sqlite_futures_paper_trading_schema.py`,
`.../persistence/sqlite_futures_contract_economics_schema.py` and
`.../market_data/sqlite_futures_schema.py`.
- Numbers are stored as exact decimal **text**.
- `contracts` is TEXT because the count is unbounded.

**Derived, never stored:** positions, average entry, realized, unrealized and
total P&L, order status (pending or filled) and trades.

**Answers to the Task 2 questions:**

| Question | Answer |
|----------|--------|
| A. Fields needed | Fill: identity, order identity, quote and instant. Its parent order: portfolio, strategy, contract key, side, count and decision instant. The contract's point value and currency. The stored daily bars, for marks and the execution-rule cross-check |
| B. Every fill reconstructable? | Yes: fill row joined to its order |
| C. Order, time, quantity and price sufficient? | Yes. The order must be canonical (parsed instant, then order identity) |
| D. Lot or multiplier source? | `futures_contract_economics.point_value_amount`, entered by the operator with `economics set`; there is no separate lot size. Northstar does not check that the value matches the exchange's lot: it is operator-entered reference data |
| E. Mark price and its date identifiable? | Yes: the latest stored bar at or before the cutoff, reported as `mark_quote` and `mark_instant`. Section 5 covers the ambiguity of finality |
| F. Realized and unrealized reconcilable independently? | Yes, from B to E |
| G. Depends on anything not persisted? | Only the cutoff (an input) and the fixed calculation context. The finality configuration does **not** affect the valuation (section 5) |

**Reporting interfaces (derived results only):**

| Interface | What it reports | Source |
|-----------|-----------------|--------|
| `northstar paper status` | `Orders`, `Fills` and `Pending` counts, `PENDING` lines, positions (direction, contracts, average entry), and per contract the realized P&L, mark and unrealized P&L | `_paper_status` in `northstar-api/src/northstar_api/cli.py`; `execution_summary_lines`, `portfolio_lines` and `pnl_lines` in `northstar-api/src/northstar_api/_cli_rendering.py` |
| `GET /api/futures/dashboard` | `pnl.rows` (realized, mark quote and instant, unrealized, as exact strings), `portfolio.positions`, `recent_decisions` | `_pnl`, `_portfolio` and `_recent` in `northstar-api/src/northstar_api/routers/futures.py` |
| `GET /api/futures/analysis` | `portfolio.canonical_realized_pnl` and `unrealized_pnl`, completed trades, open exposure, equity curve (per-point `total_pnl`) | `northstar-api/src/northstar_api/schemas/futures_analysis.py` |

None of these is a source of truth for reconciliation: they are what is
checked.

## 4. Independent calculation specification (proposed)

**Inputs** are read only from a synthetic SQLite database opened read-only
(`mode=ro`): the four tables of section 3 and one fixed cutoff C.

**Independence.** The verifier must not import:
- `northstar_application`, nor in particular `build_futures_paper_portfolio`,
  `calculate_futures_realized_pnl`, `value_futures_paper_portfolio`,
  `build_futures_paper_trading_valuation` or
  `calculate_futures_paper_performance`;
- Northstar's repositories, which decode the same rows.

It may use the Python standard library only. An automated import-boundary test
enforces this (section 10).

**Arithmetic.** Use exact rationals (`fractions.Fraction`) parsed from the
stored decimal text, with no decimal context. Instants are parsed to aware UTC
datetimes and never compared as text.

**Visibility at C.**
- A fill is visible if `filled_at <= C`.
- Process visible fills in canonical order: `filled_at`, then
  `order_identity`.
- A pending order is one with no fill row at all; one whose fill is after C
  counts as pending at C. It never contributes to a position.

Per contract K, with point value `v`, visible fills `i = 1..k`, signed delta
`d_i` (BUY `+count`, SELL `-count`) and quote `q_i`. Start with `n = 0`,
`a` undefined, and `R = 0` quote points. For each fill:

1. **Flat or same direction** (`n == 0`, or `sign(n) == sign(d)`):
   `a := (|n| * a + |d| * q) / (|n| + |d|)` (just `q` when flat), then
   `n := n + d`.
2. **Opposite direction:** add `c = min(|d|, |n|)` and
   `R := R + (q - a) * sign(n) * c`. Then let `n' = n + d`:
   - `n' == 0` makes the position flat (`a` undefined);
   - `sign(n') == sign(n)` keeps `a`;
   - otherwise it is a reversal and `a := q`.

   Finally `n := n'`.

**Results for K:**

| Quantity | Formula |
|----------|---------|
| Signed net quantity | `n`: LONG if positive, SHORT if negative, flat if zero |
| Open cost basis | `a`, only when `n != 0` |
| Realized gross P&L | `R * v`, in K's currency |
| Mark | The stored bar of K with the greatest `point_in_time <= C`, taking its close `m` and instant |
| Unrealized gross P&L | `0` if `n == 0`; **unavailable** if `n != 0` and there is no mark; otherwise `(m - a) * n * v` |
| Total gross P&L for K | realized plus unrealized, unavailable whenever unrealized is unavailable. This matches the analysis `total_pnl` |
| Fees and net P&L | Not modelled: report "not applicable (gross only)" and never a number (section 6) |
| Aggregate portfolio P&L | **Not defined by Northstar** (decision U-1). The verifier may report a same-currency sum labelled "verifier aggregate, no application counterpart", never compared |

The formulas cover LONG and SHORT through `sign(n)` and the signed `n`. They
are linear through zero, so zero and negative quotes need no special case.

**Cross-checks** (report findings; never adjust):
- each fill's quote equals the open of the first stored bar of its contract
  strictly after the order's `decided_at`;
- every fill has its parent order, and no order has two fills;
- every visible fill's portfolio and strategy match;
- every filled contract has economics.

## 5. Chronology and valuation-mark rules (verified, with ambiguities)

**Rules:**
- Decisions are taken at session closes. An order's `decided_at` is its
  decision instant.
- A fill can only be created by a later paper run, once the next bar is stored
  and visible at that run's cutoff.
- A historical replay at an earlier cutoff is refused by
  `_refuse_backward_run` (`cli.py`).
- Reading at an earlier `--as-of` is read-only and sees only facts at or before
  it.

**A fixed synthetic cutoff is safe:**
- bars after C are excluded by the query window and the instant comparison;
- pending orders are never treated as fills;
- the verifier reads no finality setting and no provider;
- it opens the database read-only, so persisted decisions cannot change. A dump
  hash before and after proves this (section 10).

**Ambiguities to keep in view:**

- **"Latest approved candle" is not a persisted fact.** Finality is
  configuration (`NORTHSTAR_FUTURES_FINAL_THROUGH`), not a per-bar record. The
  mark is the latest **stored** bar at or before C, whether or not its session
  was approved: a manual `market-data sync` can store a bar the chronological
  operation has not processed (U-6).
- **The mark can be newer than the latest decision.** The dashboard's default
  cutoff is the latest stored bar (`futures_dashboard` in `routers/futures.py`),
  which can be after the latest decision. An order whose fill bar is already
  stored is still **pending** until a paper run creates the fill, so the
  valuation can use a newer mark while that order shows as unfilled. Reconcile
  from persisted fills only; never re-simulate a fill.
- **Each output has its own cutoff.**
  - `paper status`: `--as-of` is required, and C is exactly that.
  - Dashboard and analysis: the latest stored bar, or the given `as_of`.
  - `paper run`: the session close.

  The comparison must use the application output taken at the same C.
- **Expiry.**
  - A position surviving expiry keeps being marked at the last stored close,
    whose instant shows it is stale.
  - Nothing settles it (acceptance L-8, D-EXP-6).
  - Fills cannot occur after the last stored bar.

## 6. Gross and net P&L treatment

**Verified:** Northstar reports gross simulated P&L only. Fees, brokerage,
taxes, slippage, financing, margin and cash are not modelled (acceptance P-4,
deferred under L-6).

**Proposed:** the verifier reports gross figures and marks net as "not
applicable". It must not invent a fee schedule. Net reconciliation becomes
possible only after an approved fee model exists (U-4).

## 7. Synthetic scenario matrix (proposed)

The scenarios are named PR-n to avoid the acceptance requirement IDs P-1 to P-4
(section 6 of the specification). PR-1 to PR-10 correspond to scenarios P-1 to
P-10 of the M1.4.4.1 request.

All use NIFTY with economics of 65 INR per point. PR-12's second contract uses
75 INR, which proves economics are per contract. The values were verified as
described in section 10.

| ID | Synthetic fills (in order) | Mark | Position | Basis | Realized (INR) | Unrealized (INR) | Total (INR) |
|----|----------------------------|------|----------|-------|----------------|------------------|-------------|
| PR-1 | BUY 1 @ 25000 | 25100 | LONG 1 | 25000 | 0 | 6500 | 6500 |
| PR-2 | SELL 1 @ 25000 | 24900 | SHORT 1 | 25000 | 0 | 6500 | 6500 |
| PR-3 | BUY 1 @ 25000 | 24900 | LONG 1 | 25000 | 0 | -6500 | -6500 |
| PR-4 | SELL 1 @ 25000 | 25100 | SHORT 1 | 25000 | 0 | -6500 | -6500 |
| PR-5 | BUY 1 @ 25000; SELL 1 @ 25200 | any | flat | -- | 13000 | 0 | 13000 |
| PR-6 | SELL 1 @ 25000; BUY 1 @ 25200 | any | flat | -- | -13000 | 0 | -13000 |
| PR-7 | BUY 2 @ 25000; SELL 1 @ 25100 | 25050 | LONG 1 | 25000 | 6500 | 3250 | 9750 |
| PR-8 | BUY 1 @ 25000; BUY 2 @ 25300; SELL 1 @ 25500 | 25100 | LONG 2 | 25200 | 19500 | -13000 | 6500 |
| PR-9 | Order BUY 1 with no fill | -- | flat | -- | no row | no row | no row; `Pending: 1` |
| PR-10 | None | -- | flat | -- | no row | no row | no row; `Pending: 0` |
| PR-11 | BUY 1 @ 25000; SELL 2 @ 25200 (reversal) | 25100 | SHORT 1 | 25200 | 13000 | 6500 | 19500 |
| PR-12 | Oct BUY 1 @ 25000; Nov SELL 1 @ 25300 | Oct 25100; Nov 25200 | Oct LONG 1; Nov SHORT 1 | 25000; 25300 | 0; 0 | 6500; 7500 (at 75 INR) | Per contract only (U-1) |
| PR-13 | BUY 1 @ 25000; BUY 2 @ 25001 | 25001 | LONG 3 | 75002/3 exactly | 0 | exact 65; the application gives 64.99999999999999999999935 | Exact 65 (section 8) |
| PR-14 | Fees and slippage | -- | -- | -- | -- | -- | Not modelled: "not applicable", never compared |
| PR-15 | Open position after expiry, no further bars | last stored close | unchanged | unchanged | unchanged | marked at the stale close, with its instant reported | Not settled. The daily operation reports exit 7 ([procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md)) |
| PR-16 | Open position, missing economics or no mark | -- | as filled | as filled | -- | unavailable | The application reports P&L unavailable (CLI exit 4; dashboard `missing_contract`) |

**Missing source facts.** In every scenario, a missing fact makes the result
**unavailable**, never zero (section 9).

**No mark.** A filled contract always has a stored bar at its fill instant, and
bars are insert-only, so "open with no mark" cannot arise from consistent
persisted data. If it is seen, it is a data defect.

**Reaching the scenarios.**
- Through the production writers with a fixed target N:
  - PR-1 to PR-6 and PR-11 (a SELL against LONG N is a 2N reversal);
  - PR-9 and PR-10;
  - PR-15 (the expiry fixtures of M1.4.3.2).
- PR-7, PR-8, PR-12 and PR-13 need either a changed `--target`, which the CLI
  allows with a warning, or orders and fills inserted directly into a
  **synthetic** database through the existing insert-only stores (section 10).

## 8. Comparison and tolerances (proposed, not approved)

**Compare:**
- per contract: net contracts, average entry, realized, mark quote and instant,
  and unrealized;
- the counts of fills and pending orders.

Compare as exact decimals parsed from the application's output strings.

**No tolerance is defined anywhere in Northstar** (U-3).

**Proposed rule:**
1. **Exact equality** whenever every average entry terminates: PR-1 to PR-12.
2. Where an average repeats (PR-13), the application's 28-significant-digit
   basis makes it differ from the exact result in trailing digits. Accept
   `|application - exact| <= 1e-15 INR` per amount, and **only** in such
   cases.

   The observed difference in PR-13 is 6.5e-22 INR. The bound is about 10^6
   above the expected rounding error and about 10^13 below one paisa.
3. Every other difference is a finding. It is never rounded away.

**Display.** The application reports unrounded amounts, so PR-13 prints
`64.99999999999999999999935`. Whether reports should round to paise is open
(U-2), and the verifier compares unrounded values.

## 9. Missing-data and fail-closed behaviour

| Condition | Application (verified) | Verifier (proposed) |
|-----------|------------------------|---------------------|
| Contract economics missing for a filled contract | Valuation unavailable: `FuturesContractEconomicsNotFoundError`, P&L unavailable, `paper status` exit 4 | Stop for that portfolio; report "economics missing" |
| No mark at or before C for an open position | Unrealized `None`, "no synced daily close observable by cutoff" | Unavailable, never 0; flag it as a data defect (section 7) |
| Fill without its order, or a second fill for one order | Refused by the store and by the fold | Fail: integrity finding |
| Foreign portfolio or strategy in the history | Refused (`_validate_fill_history`, `_load_history`) | Fail |
| Fill quote differs from the execution rule | Not re-checked after storage (back-filled bars are accepted debt: `simulate_futures_paper_order_fill.py`) | Report the finding; reconcile from the stored fill |
| Pending order | Excluded from positions; listed as pending | The same; never treated as filled |
| Expired, unresolved position | Marked at the stale close; exit 7 in the daily operation | Report with the stale mark instant; no settlement |

## 10. Reproducible isolated verification design (implemented in M1.4.4.2)

This design is implemented as described below, with two deviations:
- the verifier lives at `northstar-api/tests/independent_pnl_verifier.py`, not
  under `tests/reconciliation/`, following the flat layout of `tests/`;
- the application side is a separate adapter,
  `northstar-api/tests/pnl_reporting_adapter.py`.

The original proposal follows.

**Environment.** Development machine only:
- temporary SQLite databases;
- the existing fake-Upstox helpers (INDIA-7/8B tests), injected clocks and a
  refused network;
- no Docker, provider, production database or credential.

**Verifier.** A test-side module, for example
`northstar-api/tests/reconciliation/independent_pnl.py`. It:
- uses the standard library only;
- opens the database read-only and implements section 4;
- is guarded by an import-boundary test asserting that it imports no
  `northstar_application` or `northstar_api` module.

**Fixtures:**
- end-to-end scenarios through the production CLI (section 7);
- accounting-only scenarios by inserting orders and fills into a synthetic
  database through the existing insert-only stores, with bars and economics
  for the marks.

**Comparison.** Against the application's own outputs at the same cutoff:
- `paper status` text;
- the dashboard's `pnl.rows` JSON;
- the analysis `portfolio` section.

**Preservation.** A SHA-256 of the full SQL dump before and after each
verifier run must be identical.

**Validation already done for this audit** (2026-10-10, development machine,
in memory, no database). An exact-fraction model of section 4 reproduced every
hand-computed value in section 7. The application's own use cases gave the
same results:
- exactly for PR-1 to PR-12;
- within 6.5e-22 INR for PR-13.

Missing economics failed closed, and an unmarked position was reported
unavailable. This validates the specification only; it is **not** the M1.4.4.2
verifier.

**Precedent.** `_independent_pnl` in
`northstar-api/tests/test_india7_nifty_incremental_operations_acceptance.py`
is an existing independent fold. It is limited to one contract, a fixed
65 INR, the ambient Decimal context and fills read through Northstar
repositories. It is used by
`northstar-api/tests/test_india6_nifty_baseline_acceptance.py::test_valuation_matches_an_independent_recomputation`.
M1.4.4.2 would generalize it.

## 11. Evidence requirements (proposed)

For each run, record:
- the commits of all repositories;
- the verifier's own hash;
- the scenario list, cutoffs and, per scenario, the expected, application and
  verifier values with the difference;
- the database hashes before and after;
- every finding.

Keep this synthetic evidence outside the repositories with a SHA-256 manifest,
as done for M1.4.3.3 (evidence register E-11). Production evidence is out of
scope (section 13).

## 12. Unresolved accounting decisions

| ID | Decision | Current state |
|----|----------|---------------|
| U-1 | Portfolio-level total P&L across contracts | Not defined; the valuation has no total by design |
| U-2 | Reporting precision (for example, whole paise) | None; amounts are unrounded |
| U-3 | Reconciliation tolerance | None; 1e-15 INR is proposed (section 8) |
| U-4 | Net P&L: fees, brokerage, taxes, slippage | Not modelled (L-6, deferred) |
| U-5 | Valuation or settlement of a position surviving expiry | None (L-8; D-EXP-6 of the expiry procedure) |
| U-6 | Whether a mark must come from an approved (final) session | Not enforced; the mark is the latest stored bar |
| U-7 | Maximum mark staleness | None; a stale mark is used and its instant reported |
| U-8 | Economics changing within a contract's life | Assumed constant; effective dating deferred |

Nothing in this document decides any of them.

## 13. Production verification limitations

- **P-3** (the October portfolio's recomputation) would have to read the
  production fill quote, mark and bars. All of these are Upstox-derived.
  Reading or copying the production SQLite database, or retaining its output
  as evidence, processes and retains that data while **G-1 is unresolved**.
  This audit therefore performs no production reconciliation, and P-3 stays
  **PENDING**.
- No read-only operator command lists each fill's identity, timestamp and price
  (expiry procedure, section 7). A production reconciliation without database
  access would need such an interface.
- M1.4.3.2 is not deployed in production.
- A synthetic reconciliation proves the accounting rules. It says nothing about
  production profitability.

## 14. Milestone acceptance criteria (proposed)

**M1.4.4.1 (this audit) is complete when:**
- this document exists, with its sources checked;
- the scenario values are validated;
- the acceptance specification references it.

**M1.4.4.2 would be met when:**
- the verifier exists under the import-boundary test;
- PR-1 to PR-16 run on synthetic databases with the comparison rule of
  section 8;
- the database hashes are unchanged;
- the evidence of section 11 is recorded.

All four were observed on synthetic data on 2026-10-10 (section 15). That is
an engineering result, not an approval.

**P-3** additionally needs G-1 resolved and an approved production method.

None of these criteria is approved, and X-4, G-1, B-1 and B-2 are unaffected.

**Findings by class:**

| Class | Findings |
|-------|----------|
| BLOCKING | Production reconciliation (P-3) is blocked by G-1 (section 13). Nothing blocks the synthetic verifier |
| NON-BLOCKING | Basis rounding at 28 digits (section 8). A pending order can coexist with a newer mark (section 5). Stale marks after expiry. No-mark is unreachable for consistent data. Fill re-simulation debt (section 9) |
| DOCUMENTATION GAP | U-1 portfolio total, U-2 precision, U-3 tolerance and U-6 mark finality, all now recorded here. Found in M1.4.4.2: the analysis reports a signed `open_exposure.net_contracts` while the dashboard reports an unsigned count plus direction; the analysis covers only its selected contract and reports "contract economics not configured" for an empty database |
| FUTURE ENHANCEMENT | A read-only fill listing; a fee and net model (U-4); effective-dated economics (U-8); a same-currency portfolio aggregate (U-1) |

## 15. Synthetic verification results (M1.4.4.2, 2026-10-10)

Executed on the development machine, on disposable synthetic SQLite databases
only.
- The network was refused and no clock was read.
- No production database, provider, credential, Docker, scheduler or finality
  setting was used.
- Code was read at the commits listed at the top of this document. The new
  test files were uncommitted.

**Implementation:**

| Part | File | Role |
|------|------|------|
| Independent verifier | `northstar-api/tests/independent_pnl_verifier.py` | Standard library only. Opens SQLite `mode=ro` with `query_only`. Parses instants to aware UTC. Orders fills by (instant, order identity). Folds with a total-entry-cost formulation in `fractions.Fraction`. Fails explicitly on malformed or inconsistent facts and on missing economics. Reports a missing mark as unavailable, and every mark's source, instant and approval (`UNKNOWN`) |
| Northstar adapter | `northstar-api/tests/pnl_reporting_adapter.py` | Reads `paper status`, `economics show`, `GET /futures/dashboard` and `GET /futures/analysis` at the same cutoff, and computes nothing |
| Tests and scenarios | `northstar-api/tests/test_independent_pnl_reconciliation.py` | PR-1 to PR-16, end-to-end, chronology, defects, preservation, independence |

**Independence.** Three tests enforce it, and all pass:
- the verifier's imports are standard library only, checked from its syntax
  tree;
- importing it in a fresh interpreter loads no `northstar*` module;
- its source contains none of Northstar's fold identifiers.

**Comparison outcomes.** Each field on each surface is classified as:
- EXACT;
- PROVISIONAL_PRECISION (at most 1e-15 INR, and only for a non-terminating
  exact average);
- MISMATCH;
- UNAVAILABLE;
- NOT_APPLICABLE (a flat contract has no average or mark);
- NOT_EXPOSED (the surface does not report it; never a pass).

There were no MISMATCH outcomes:

| Scenario | Database | Outcome |
|----------|----------|---------|
| PR-1 to PR-8, PR-11, PR-12 | Stores | Every compared field EXACT on the CLI, dashboard, analysis and `economics show` |
| PR-9 | Stores, and end to end at bar 22 | No contract row, one pending BUY on every surface; the analysis reports the selected contract flat with zero P&L |
| PR-10 | Stores (empty) | No rows, `Pending: 0`. The analysis is unavailable: "contract economics not configured" |
| PR-13 | Stores | Average entry exact 75002/3, reported 25000.66666666666666666666667 (difference 3.33e-24 points). Unrealized and total exact 65 INR, reported 64.99999999999999999999935 (difference 6.5e-22 INR). Seven PROVISIONAL_PRECISION outcomes, all other fields EXACT |
| PR-14 | Stores | Gross only: no fee, net, slippage or tax field on any surface; the verifier reports net P&L as not applicable |
| PR-15 | Production CLI, fake Upstox | LONG 1 valued after expiry at the stale E-7 close of 2026-10-15 (`approval UNKNOWN`), exact on every surface |
| PR-16a | Stores, no economics | The verifier raises `MissingEconomics`. `paper status` exits 4 with the position but no P&L; the dashboard P&L is unavailable with `missing_contract` |
| PR-16b | Stores, bar removed | Unrealized and total UNAVAILABLE on both sides, never zero. Findings: no mark, and no bar for the execution rule |
| End to end, bar 26 | Production CLI, fake Upstox | BUY 1 at 25103, then SELL 2 at 24003. SHORT 1 at 24003; realized -71500 INR; unrealized 195 INR, all EXACT |
| C-1 | Stores | Equal fill instants follow the order identity: LONG 1 at 25100, realized 6500, EXACT. The other order would give 25050 and 3250 |
| C-2 | Stores | A fill, order and mark after the cutoff are all invisible. The order decided at the cutoff and filled after it counts as pending |

**Fields not exposed** (reported as NOT_EXPOSED, never passed):
- the CLI and dashboard report no per-contract total;
- the dashboard and analysis report no fill count;
- the analysis reports no pending orders;
- no surface reports a portfolio total (U-1).

**Precision assessment.** The 1e-15 INR threshold is applied only where the
verifier proves the exact average entry has no finite decimal expansion. It
passed PR-13's 6.5e-22 INR rounding. Five deliberately corrupted expectations
were all classified MISMATCH, threshold or not:
- point value 66 instead of 65;
- quantity 3 instead of 2;
- mark 25150 instead of 25100;
- cutoff day 15 instead of day 19;
- 2e-15 INR on a repeating basis.

The threshold remains provisional (U-3). Nothing was rounded to paise (U-2).

**Missing and inconsistent facts.** Each of these raised an explicit error on a
disposable copy and left it unchanged:
- a fill not after its decision;
- a fill without its order;
- a malformed quote;
- an instant without an offset;
- a non-positive point value;
- a foreign strategy;
- a missing database, which was not created.

Re-inserting fills in reverse storage order did not change the result.

**Database preservation.** For every verification:
- the file's SHA-256, the SHA-256 of its logical dump and its `-wal`, `-shm`
  and `-journal` sidecars were identical before and after;
- the logical dump was also unchanged after Northstar's own reads.

**Evidence.** Synthetic only, outside every repository, in
`C:\Code\Private\Northstar\NorthstarEvidence\M1.4.4.2-synthetic-2026-10-10`.
- Contents: 8 files: the sources, environment, scenarios, results, test output
  and limitations.
- `MANIFEST.sha256` has SHA-256
  `0f1b2c144acb83edc81909d2d6c14de8a48fe57eb1ee553ad732425282021e7c`.
- The files were scanned for credentials, account and computer names, user
  paths, production paths and the production figures recorded in the
  evidence register: none found.
- The manifest was verified with `sha256sum -c`.

**Still open:**
- U-1 to U-8 (section 12): not decided by these results.
- Production reconciliation (P-3) is still blocked by G-1 (section 13).
- These results say nothing about production profitability.
