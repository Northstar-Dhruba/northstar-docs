# Indian Futures: Reviewed-Candle Equivalence Design (FG-9.1)

Status: **design; implemented in development as FG-9.2A to FG-9.2E, not
released and not accepted** (2026-10-11). Where the implementation differs
from the design, "Implemented behaviour" in section 6 records what was
built.
- Nothing here changes code, configuration, finality, data or any acceptance
  status.
- It proposes how to meet gate FG-9 of the
  [Revised Candle Approval Procedure](../operations/Indian-Futures-Revised-Candle-Approval-Procedure.md)
  (N-4). Until an implementation is built, verified and released, FG-9 stays
  unmet and final-through cannot advance past 2026-10-08.
- Approving this design approves no candle, grants no G-1 permission and
  authorizes no `.env` change.

**Location.** `architecture/` holds the design specifications
(`*-Design-Specification-v1.0.md`). The requested name is kept, without a
version suffix, because this is a proposal and not yet a frozen
specification. The README's Architecture row already links the folder, so
the index is unchanged.

Source paths are relative to the Northstar workspace root. Code was read at:

| Repository | Commit |
|------------|--------|
| core | e54ccdf |
| application | 9a0ab57 |
| infrastructure | aa94428 |
| api | cfa7774 |
| web | 4f6908c |
| docs | eec9828 |

No production system, database, provider or evidence file was accessed. No
candle value appears here; the only numbers in examples are synthetic.

## 1. Problem statement

The operator reviews a daily candle from private evidence (N-4), and then
advances `NORTHSTAR_FUTURES_FINAL_THROUGH`. The next scheduled run fetches
the candle from Upstox again, stores whatever it receives, and trades on it
in the same run. Nothing ties the stored candle to the reviewed one.

**Goal:** Northstar must be unable to store, or trade on, a candle for a
reviewed session unless it is exactly the version the operator approved.
Any other outcome must fail closed, before any write.

**Not a goal:**
- deciding whether a candle is final (that stays the operator's decision);
- proving the operator inspected a trustworthy candle (an identity binds a
  version; the N-4 decision record carries the judgement);
- correcting stored bars, decisions, orders or fills.

## 2. Verified current code path

**Sequence of one `operations daily` run** (`_chronological_daily_operation`
in `northstar-api/src/northstar_api/cli.py`):

| # | Step | Code | Types | Failure and exit |
|---|------|------|-------|------------------|
| 1 | Settings | `load_session_operation_settings` and `_finality_mode` in `northstar-api/src/northstar_api/settings.py` | `FuturesSessionOperationSettings(finality_mode, final_through, go_live)` | Invalid or missing value: exit 3 |
| 2 | Plan | `plan_session_backlog` in `northstar-api/src/northstar_api/operations.py`, with `build_daily_bar_finality_policy` in `northstar-api/src/northstar_api/runtime.py` | Eligible `FuturesTradingSession` tuple: the final prefix after the latest frozen decision | Exhausted with exposure: exit 7. Nothing eligible: `STATUS: WAITING`, exit 0 |
| 3 | Expiry check (S-2) | `_contract_exposure`, `is_past_expiry` | | Exposed past expiry: exit 7, before any provider request |
| 4 | Compose | `build_upstox_market_sync_runtime` (runtime.py) wires `AcquireFuturesNativeDailyHistoryUseCase` with `NSEFuturesTradingSessionResolver`, `UpstoxFuturesNativeDailyMarketDataSource` and `SQLiteFuturesHistoricalMarketDataStore` | | |
| 5 | Resolve sessions | `sessions_in_range` inside `AcquireFuturesNativeDailyHistoryUseCase.execute` (`northstar-application/src/northstar_application/application_services/acquire_futures_native_daily_history.py`) | `FuturesDailyHistoricalAcquisitionQuery(contract, first, last)` | |
| 6 | Provider request and response | `UpstoxFuturesNativeDailyMarketDataSource.fetch_daily_observations` calls `fetch_daily_candles` (`northstar-infrastructure/src/northstar_infrastructure/market_data/upstox_futures_native_daily_market_data.py`). One historical request for the range; the venue's current date, if in range, from the current-day endpoint. JSON is decoded with `parse_float=Decimal` (`upstox_http.py`) | Raw `[timestamp, open, high, low, close, volume, open_interest]` | Transport, status, shape, duplicate label, out-of-range label: `UpstoxMarketDataSourceError`, exit 6 |
| 7 | Normalization | `_labelled`, `_trading_date` (label read in Asia/Kolkata), `_observation`, `_number` (exact finite `Decimal`), `_contract_volume` (provider volume divided exactly by lot size) | `FuturesNativeDailyObservation(contract, trading_date, open, high, low, close, volume)`; open interest dropped | Non-numeric, non-finite, fractional contracts, no UTC offset: exit 6 |
| 8 | Contract and session validation | `_index_observations` (contract equals the requested one; one candle per date) | `dict[date, observation]` | `FuturesHistoricalDataContractViolationError`, exit 6 |
| 9 | Completeness | `_require_coverage` | | Missing or unexpected session: `FuturesDailySessionCoverageError`, exit 4 |
| 10 | Canonical bar | `_daily_bar` builds `FuturesOHLCVBar` stamped at the session close; `FuturesOHLCVBar` enforces price coherence | `tuple[FuturesOHLCVBar, ...]` | Incoherent: exit 6 |
| 11 | Storage | `_persist` calls `SQLiteFuturesHistoricalMarketDataStore.store`: `_prepare` refuses in-batch duplicate keys, then **one `BEGIN IMMEDIATE` transaction**; `_store_one` inserts an absent key, accepts an equal stored bar, raises on a differing one, and rolls back the whole batch | `int` (accepted count) | `FuturesHistoricalMarketDataConflictError`: exit 5, nothing stored |
| 12 | Replay | `_replay_latest_cutoff` reruns the latest frozen cutoff | | |
| 13 | Decisions, orders, fills | For each eligible session, oldest first, `_run_paper_session` runs `RunFuturesPaperTradingSessionUseCase.execute(..., available_through=session close)`: freeze a decision from **stored** bars (`FreezeFuturesForwardResearchDecisionUseCase`, which reads a `FuturesHistoricalMarketDataRepository`), then settle orders and fills (`SimulateFuturesPaperOrderFillUseCase`, fill at the next stored bar's open within `[decision, available_through]`) | | Stops at the first failing session; earlier sessions stay processed |

**Error mapping.** `_acquire` in `cli.py` maps
`FuturesTradingSessionInProgressError` and `FuturesDailySessionCoverageError`
to `DATA` (4), provider errors and `FuturesHistoricalDataContractViolationError`
to `PROVIDER` (6), and `FuturesHistoricalMarketDataConflictError` to `STATE` (5).
`ExitCode` (`cli.py`) defines 0 to 7. 8 and 9 are unused, and the Windows
wrapper reserves 10 to 13.

**Transaction boundaries:**
- The bar batch is one transaction: all bars or none.
- Decisions (`SQLiteFuturesForwardResearchRecordStore`), orders
  (`SQLiteFuturesPaperOrderStore`) and fills (`SQLiteFuturesPaperFillStore`)
  each commit in their own `BEGIN IMMEDIATE` transaction, session by
  session, **after** the bar batch has committed.
- The replay in step 12 cannot consume a newly stored bar, because its
  cutoff is the latest decision and fills are bounded by
  `available_through`.

**Dependency boundaries.** Core holds values (`FuturesContract`,
`FuturesOHLCVBar`, `QuoteValue`, `Quantity`). Application holds ports and use
cases. Infrastructure holds the Upstox adapter and the SQLite stores. The API
composes them and owns settings, the CLI and exit codes.

**Manual commands bypass finality today.** `_market_data_sync` stores any
requested range, and `_paper_run` decides at any cutoff, without consulting
finality. Both go through the same use cases as the scheduled run.

**Write-capable entry points.** The CLI parser's handlers (`cli.py`), plus
the HTTP routers:

| Entry point | Writes | Consults finality today | Under FG-9 (section 6) |
|-------------|--------|-------------------------|------------------------|
| `operations daily`, provider `upstox` (`_chronological_daily_operation`) | Bars, decisions, orders, fills | Yes | Verified acquisition (item 3) and the paper guard (item 8) |
| `operations daily`, provider `databento` or unset (`_daily_operation_cycle`) | Bars, decisions, orders, fills | **No**: decides at the latest stored bar. For NSE it fails only incidentally: no `DATABENTO_API_KEY` (exit 3), or `ExchangeCalendarFuturesTradingSessionResolver` lacks NSE (`FuturesSessionResolutionError`, exit 6). Compose pins `upstox` for `india-operations` | Explicit refusal for NSE (item 9) |
| `market-data sync --provider upstox` (`_market_data_sync`) | Bars | **No** | Verified in `operator-reviewed` (item 4) |
| `market-data sync`, provider `databento` (the default) | Bars | **No**; NSE fails incidentally as above | Explicit refusal for NSE (item 9) |
| `paper run` (`_paper_run`) | Decisions, orders, fills at any forward cutoff | **No**; only `_refuse_backward_run` | Paper guard (item 8) |
| Latest-cutoff replay (`_replay_latest_cutoff`) | Missing order or fill of the latest decision | Bounded by its own cutoff | Paper guard passes trivially (nothing after the latest decision) |
| `paper status`, `economics show`, `finality-evidence observe` and `report`, option `show` commands | None in futures candle or paper tables | -- | Unchanged |
| `economics set` | Contract economics (P&L inputs), not candles or paper facts | -- | Out of FG-9 scope |
| `options ...` sync and economics commands | Option tables only | -- | Out of FG-9 scope |
| HTTP: `/futures/*`, `/health` | None (GET) | -- | Unchanged |
| HTTP: `POST /analyze`, `POST /watchlist/refresh` (`routers/analysis.py`) | Equity analysis from Yahoo; no futures table | -- | Out of scope |
| `finality-evidence identity` (FG-9.2E, `_finality_evidence_identity`) | Nothing: reads one evidence file and the NSE calendar | -- | Prints an fdc1 manifest entry; approves nothing |
| `protection init` (FG-9.2E, `_protection_init`) | Only the protection marker, under the operations lock | -- | The explicit OD-9 marker step |
| `protection status` (FG-9.2E, `_protection_status`) | Nothing: opens the file read-only | -- | Preflight and postflight check |

Every path to paper decisions, orders and fills goes through
`_run_paper_session` (callers: `_paper_run`, `_replay_latest_cutoff`,
`_chronological_daily_operation`, `_daily_operation_cycle`), and every path
to stored futures bars goes through `_acquire`.

## 3. Safety invariants

| # | Invariant |
|---|-----------|
| I-1 | Under the reviewed mode, no bar for a session is stored unless its canonical identity equals the identity approved for that contract and session |
| I-2 | Every session in an acquisition range must have an approved identity, checked **before any provider request** |
| I-3 | Verification covers the whole batch **before** the single store transaction; any failure stores nothing and decides nothing |
| I-4 | Paper decisions, orders and fills are produced only after a fully verified batch has committed |
| I-5 | A final-through date alone never approves provider values |
| I-6 | Existing stored bars, decisions, orders and fills are never modified, re-decided or reinterpreted |
| I-7 | A verification failure is never reported as `WAITING` or as success, and exit 7 keeps its meaning and precedence |
| I-8 | A rollback to a build without verification cannot silently ingest under an approved date |
| I-9 | No provider-derived value is printed in a failure message |
| I-10 | No supported write-capable entry point (section 2) can store a bar, or create a decision, order or fill, for an NSE session that the configured mode has not authorized; under `operator-reviewed` that includes a matching approved identity |
| I-11 | No mode or manifest state falls back silently to weaker behaviour. A missing or blank mode is `disabled`, which can advance nothing; a missing or malformed manifest in `operator-reviewed` is exit 3 |

**Why I-1 also protects a previously stored bar.** The fetched bar is
verified first. The store then either inserts it, accepts an equal stored
bar, or raises a conflict. So any bar the run relies on equals the verified
fetched bar, which equals the approved identity. A bar stored earlier by a
manual sync is therefore usable only if it is the approved version;
otherwise the run stops with exit 5.

## 4. Canonical candle identity

**Fields.** Only what Northstar stores and consumes, from the canonical bar
the use case is about to persist, not from the provider schema:

| Field | Class | Why |
|-------|-------|-----|
| `open` | A: economic | Fill price (`SimulateFuturesPaperOrderFillUseCase`, `fill_quote=bar.open`) |
| `close` | A: economic | Strategy signal (`select_directional_signal`) and valuation mark (`value_futures_paper_portfolio.py`) |
| `volume_contracts` | A: economic | Strategy volume confirmation; stored after lot-size normalization |
| `high`, `low` | A: stored | Persisted, and read by `analyze_futures_replay_snapshot.py` and `historical_snapshot_evaluator.py`; a substituted value would change stored history |
| `product`, `exchange`, `expiration` | B: contract | `FuturesContract` identity, the bar's natural key |
| `trading_date` | B: session | The session label the candle is matched to by `_require_coverage` |
| `timeframe` (`1d`) | B: key | Part of the natural key |
| Provider, endpoint, `requested_at`, `received_at`, `instrument_key`, `lot_size`, raw volume, open interest, provider timestamp | C: provenance | Recorded in the N-4 decision record (by reference); **excluded** from comparison |

**Why provenance is excluded:**
- **Endpoint.** Equal canonical values from the historical endpoint are the
  same stored bar, whichever endpoint served the review. A review taken
  from the same-day endpoint whose values differ will fail to match, which
  is the safe direction.
- **Open interest.** Not stored and not consumed (the port's docstring
  section "Why there is no open interest"). E-7 reports it changing, so
  including it would fail candles whose stored values are identical.
- **Lot size and raw volume.** What is stored is `volume_contracts`; the
  normalization is the existing `_contract_volume` rule, and evidence
  records compute the same quotient (`_volume_contracts` in
  `upstox_candle_evidence.py`).
- **Session close instant.** It comes from the calendar, not the provider,
  and is derived from `trading_date`.

**Serialization (format `fdc1`).** ASCII text, LF after every line including
the last, fields in exactly this order, no spaces:

```
northstar.futures-daily-candle-identity/1
product=<product code>
exchange=<exchange code>
expiration=<YYYY-MM-DD>
trading_date=<YYYY-MM-DD>
timeframe=1d
open=<canonical decimal>
high=<canonical decimal>
low=<canonical decimal>
close=<canonical decimal>
volume_contracts=<canonical integer>
```

Synthetic example of the value lines only: `open=101.25`, `close=100`,
`volume_contracts=42`.

**Decimal representation.** Each number is the `str()` of the existing
`QuoteValue` or `Quantity`, which `canonical_decimal`
(`northstar-core/src/northstar_core/foundation/value_objects/_canonical_decimal.py`)
already makes context-free:
- plain notation, no exponent;
- trailing fractional zeros removed (`100.50` and `100.5` are one value;
  `1E+2` and `100` are one value);
- negative zero folded to `0`;
- every significant digit kept, with no rounding;
- no float ever involved (`parse_float=Decimal`; `_coerce` refuses floats).

So two spellings of one value hash identically, and different values never
do.

**Digest.** `fdc1-sha256:` followed by 64 lowercase hex characters: SHA-256
of the ASCII bytes above. The digest binds a version; it says nothing about
whether the version is trustworthy.

**Deterministic edge cases:**

| Case | Behaviour |
|------|-----------|
| Missing field | No identity can be built: the adapter already refuses a candle without seven elements or with a non-numeric value (exit 6) |
| Malformed or non-finite value | Refused by `_number` before identity (exit 6) |
| Fractional contract volume | Refused by `_contract_volume` (exit 6); the evidence record carries `volume_contracts: null`, and the identity command refuses it |
| Duplicate session in a response | Refused by `_labelled` or `_index_observations` (exit 6) |
| Duplicate identity in the manifest | Configuration error, even when the two digests are equal (same rule as `_prepare`) |
| Timezone | Identity uses `trading_date`, the label's civil date in Asia/Kolkata (`_trading_date`). A label without an offset is refused. A label resolving to another date matches another session or none (exit 4 or exit 8) |
| Uppercase hex, unknown prefix, wrong length | Configuration error (one spelling only) |

## 5. Alternative designs

| Criterion | A. Canonical digest in an operator manifest | B. Canonical values in a local manifest | C. Raw-response digest | D. Approval registry in SQLite |
|-----------|---------------------------------------------|-----------------------------------------|------------------------|--------------------------------|
| Equivalence | Exact on every stored field | Exact; can name the differing field | Brittle: open interest and provider spelling change the digest when the stored bar does not | Exact (stores A or B) |
| Time-of-check/time-of-use | Checked on the very bars about to be stored, in-process, before the one transaction | Same | Needs raw capture inside the adapter, then a second translation to the stored bar | Same as A, plus a database read |
| Fit with the pipeline | One check in `AcquireFuturesNativeDailyHistoryUseCase` | Same | Changes the port: raw data would cross into Application | New port, store and table |
| Repeated runs | Deterministic | Deterministic | Fails on unchanged stored values when open interest moves | Deterministic |
| Multiple sessions | One entry per session | One entry per session | One per session | Rows |
| Atomicity | Whole batch before one write | Same | Same | Approval writes become a second production write path |
| Deployment | One `.env` variable, read afresh by each `run --rm` | `.env` grows with market values | Adapter and port changes | Schema change, new operator write command, rollback interaction |
| Operator usability | Copy one token per session from an identity command | Copy five values per session; transcription errors | Needs raw bytes | Command against the production database |
| Provenance | In the N-4 decision record, by reference | Same | Raw bytes are better provenance | Who and when could be stored |
| G-1 | Digest is Upstox-derived (section 10) | Values in production config: strongest G-1 exposure | Raw retention: strong exposure | Values or digests in the production database |
| Security | Not secret; `.env` is already access-controlled | Same | Same | Database access |
| Testability | Pure function plus one use-case check | Same | Needs raw fixtures | Needs schema tests |
| Schema change | **No** | No | No | **Yes** |

**Rejected:**
- **C** compares what Northstar does not store, and pushes provider payloads
  across the port boundary that `FuturesNativeDailyMarketDataSource` keeps
  closed.
- **D** adds a table, a migration and a second production write path. That
  is the most change for no gain in equivalence.
- **B** is equivalent to A but places Upstox values in production
  configuration. Its only advantage, naming the differing field, can come
  from the private evidence `report` instead.

## 6. Recommended design

**Approach A, plus a new finality mode for fail-closed rollback.**

1. **Canonical identity (Application).** A pure function builds the `fdc1`
   text and digest from `(FuturesContract, trading_date, FuturesOHLCVBar)`.
   One implementation is shared by verification and by the operator's
   identity command.
2. **Approval set (Application value).** An immutable mapping
   `(expiration, trading_date) -> digest`, parsed and validated at settings
   load. It is not a port: it reads no file, network or clock.
3. **Verification (Application).** `AcquireFuturesNativeDailyHistoryUseCase`
   accepts an optional approval set. When present:
   - after `sessions_in_range` and **before** `fetch_daily_observations`, it
     requires an entry for every resolved session (I-2);
   - after `_daily_bar` builds every bar and **before** `_persist`, it
     compares every bar's digest with its entry (I-1, I-3).

   With no approval set (Databento, `disabled`), behaviour is unchanged.
4. **Finality mode `operator-reviewed` (API).** Same date policy as
   `operator-approved` (`OperatorApprovedFuturesDailyBarFinalityPolicy`),
   plus a required approval set. In a build with FG-9:
   - `operator-reviewed` composes the acquisition with the approval set,
     for both `operations daily` and `market-data sync --provider upstox`;
     every session in the requested range needs an identity, so a manual
     sync stores only approved versions;
   - `operator-approved` keeps planning and replay but **refuses any
     eligible session** (exit 3, before any provider request): a
     final-through date alone no longer authorizes acquisition (I-5). With
     the production `.env` as recorded (final-through 2026-10-08, nothing
     eligible), it still reports `WAITING`;
   - `disabled` is unchanged for `operations daily`; manual paper runs are
     bounded by item 8.

   A build without FG-9 rejects the unknown mode value in both
   `load_session_operation_settings` and `load_dashboard_settings`, so
   `operations daily` exits 3 and the API cannot start (`app.py` loads its
   settings at import). This was verified against both candidate baselines
   (section 11). It does **not** protect that build's manual commands, which
   never read the mode (section 11, rollback STOP conditions).
5. **Manifest.** `.env` variable `NORTHSTAR_FUTURES_APPROVED_CANDLES`, read
   only in `operator-reviewed` mode: comma-separated entries
   `<expiration>/<trading_date>=fdc1-sha256:<64 lowercase hex>`, no inner
   whitespace. The empty value means no approvals. The variable must be
   present in that mode.
6. **Identity command (API, operator tool).**
   `northstar finality-evidence identity --evidence <file> --line <n>`
   reads exactly one record of the private evidence file:
   - it requires the `northstar.upstox-daily-candle-observation/1` schema, a
     candle, and a non-null `volume_contracts`;
   - it builds the same `QuoteValue` and `Quantity` values and prints the
     manifest entry, the endpoint class (same-day or historical, from
     `requested_at` against `trading_date`) and the record's
     `requested_at`;
   - it prints no candle value, opens no database and makes no request.
7. **New exit code 8, `CANDLE_APPROVAL`,** for a missing or mismatched
   approval (section 9).
8. **Paper guard (API, mandatory).** One function called at the start of
   `_run_paper_session`, beside `_refuse_backward_run` and before any
   write. It applies to NSE contracts (the chronological venue) in
   **every** mode:
   - It plans with the existing `plan_session_backlog`, from the configured
     policy, the latest frozen decision for the contract and strategy, and
     `NORTHSTAR_FUTURES_GO_LIVE`, the same inputs as `operations daily`. A
     missing go-live with no decision is exit 3, as today.
   - **A cutoff after the last eligible session's close is refused (exit 5,
     "beyond approved finality").** In `disabled` nothing is eligible, so
     only the latest decision's own cutoff (the replay) is allowed.
   - In `operator-approved`, a cutoff that would process an eligible
     session is exit 3 (I-5).
   - In `operator-reviewed`, for every eligible session up to the cutoff,
     the **stored** bar must exist and match its approved identity
     (exit 8). This also covers a bar stored before the mode was switched,
     which the scheduled path catches only by re-fetching.

   It needs no new port: the bars come from `runtime.market_repository`,
   the identity from item 1. Because every caller passes through it, the
   chronological loop, the replay, manual `paper run` and the Databento
   cycle are all covered by one check.
9. **Databento refusal for NSE (API, mandatory).** `_daily_operation` and
   `_market_data_sync` refuse provider `databento` for an NSE contract
   (exit 3), replacing today's incidental failure.

**Why this is the smallest demonstrable solution:**
- one pure function and one check inside the use case that already
  verifies the whole batch before its one write;
- one guard at the single entry to paper writes, reusing the existing
  planner;
- two explicit venue refusals;
- no schema change, no port change, no adapter change;
- every property is testable with synthetic data and a disposable database.

### Implemented behaviour (FG-9.2A to FG-9.2E)

Built in development, not released and not accepted. Where it differs from
the items above, this is what the code does.

| Topic | Implemented |
|-------|-------------|
| Identity and manifest | As section 4 and item 5. Values whose `str()` is not plain notation (magnitudes below 0.000001) have no identity and are refused. The manifest is parsed for the configured product, so a lookup for another symbol or exchange finds nothing |
| OD-9 marker | Table `northstar_reviewed_candle_protection` (exchange, marking mode) inside the database file: created only when a file is first protected, one `BEGIN IMMEDIATE` transaction, rows only inserted, validated on every read (an invalid table or row is refused, never read as unprotected). It is written by `protection init`, or by the first write-capable NSE command in `operator-approved` or `operator-reviewed` mode, before anything else is written |
| Protected database | Every write-capable NSE command of this build refuses a missing, blank or `disabled` mode (exit 3), in any process that opens the file through Northstar |
| Paper guard scope | Applies in the operated modes on every database, and refuses `disabled` on a protected one. On an **unprotected** database `disabled` keeps the manual development workflow unchanged: a deliberate narrowing of item 8, whose "every mode" would have broken that workflow; the marker closes the gap for operated databases |
| Paper beyond finality | Exit 5 (OD-8). An approved session with no stored bar: exit 8 |
| Operator commands | `northstar finality-evidence identity` (requires an exact contract and session, and `--line` when several records match; prints no candle value; refuses records without a candle or a whole contract volume). `northstar protection init` (an existing Northstar database only; idempotent; never overwrites a marker; verified). `northstar protection status` (read-only; `--require-protected` exits 3 unless protected and writable by the configured mode; never prints a manifest entry) |
| Compose | `NORTHSTAR_FUTURES_APPROVED_CANDLES: ${NORTHSTAR_FUTURES_APPROVED_CANDLES:-}` in the shared settings: unset becomes empty, which approves nothing (exit 8 when there is work), so under Compose a missing variable is not exit 3. Northstar never trims the value; Compose's own `.env` parsing of an unquoted value is outside Northstar and was not verified here |
| Wrapper and web | Exit 8 is exitClass `CANDLE_APPROVAL`, outcome `FAILED`. The web type accepts `operator-reviewed` and shows it as the configured mode only |
| Limits | A file without the marker cannot show whether it lost one (a restored older backup). A build before FG-9 never reads the marker. Writes outside Northstar are not stopped. These are release and rollback procedures (Windows upgrade runbook, RB-1 to RB-5) |

## 7. Approval and execution boundaries

| # | Step | Who or what | Authorizes |
|---|------|-------------|------------|
| 1 | Review evidence (N-4 RC-1 to RC-10) | Operator, development machine, private evidence | Nothing |
| 2 | Accept one candle version: an N-4 APPROVE naming the exact evidence file and line | Operator; approver for ESCALATE | Nothing at runtime |
| 3 | Produce its identity: run the identity command on that line; record the reference | Operator | Nothing: a token in `.env` has no effect until step 4 |
| 4 | Advance finality: edit `.env` (mode `operator-reviewed`, the entries, `NORTHSTAR_FUTURES_FINAL_THROUGH`) | Operator, under a **separate explicit FG-8 authorization** | The next run may acquire sessions through the new date, **only** if each matches its entry |
| 5 | Acquire | The scheduled task (its enablement is a separate, existing approval) or a manual command | |
| 6 | Verify | Code, automatically, the whole batch | |
| 7 | Persist | Code, only after step 6 passes for every session | |
| 8 | Paper decisions, orders and fills | Code, in the same run, only from stored bars | Paper state only; no real order exists |

- **An APPROVE (step 2) never authorizes steps 4 to 8.** The `.env` change,
  scheduler enablement and the run itself are separate approvals.
- **Setting `NORTHSTAR_FUTURES_FINAL_THROUGH` (step 4) never approves
  provider values:** a session without a matching entry fails at step 6, or
  before the request at step 5.
- **Step 8 for a manual `paper run`** happens only through the paper guard
  (section 6, item 8), which re-checks the stored bars.
- **Three authorizations stay separate** (section 10):
  - the operator's approval of a version (the N-4 record);
  - the manifest's integrity (strict parsing and `.env` access control);
  - release authorization (M1.5.2 AP-1).

  None implies another, and a manifest entry is not evidence of who
  approved it.

## 8. Batch and transaction safety

**Can every affected session be verified before any write? Yes, without
restructuring:**
- `execute` already resolves all sessions first, fetches the range once,
  indexes and coverage-checks every observation, and builds every bar
  before the single `_persist` call;
- `store` commits that batch in one `BEGIN IMMEDIATE` transaction and rolls
  back on any conflict;
- decisions start only after `_acquire` returns (step 13 of section 2).

| Required behaviour | How it is met |
|--------------------|---------------|
| All reviewed candles match | Digest check of every bar before `_persist` |
| No approval missing | Entry check of every resolved session before the provider request |
| No unexpected contract or session | Existing `_index_observations` (exit 6) and `_require_coverage` (exit 4) |
| Previously stored bars | The insert-only store compares the verified bar with any stored one: equal is accepted, different is exit 5 (section 3) |
| Mismatch prevents storage and decisions | The check raises before `_persist`; `_acquire` raises before the session loop |
| No partial batch | Whole-batch check plus one transaction. A mismatch in the middle or at the end stores none of the earlier, matching sessions |
| Earlier facts unchanged | Nothing before step 11 writes; no new code writes |
| Re-run after mismatch | Same inputs give the same refusal. A changed manifest or provider response is a new run with the same checks |
| Manual paper run over an unverified stored bar | The paper guard refuses before `_run_paper_session` writes anything (exit 8) |

**What is atomic, and what is not.** Two separate guarantees:
1. **Before persistence (new, atomic).** Every fingerprint check passes
   for the whole range before the one bar transaction, or nothing is
   stored. A mismatch therefore creates no bar, decision, order or fill.
2. **After persistence (existing, per session, not atomic).** Once the
   verified batch has committed, each session's decision, order and fill
   commit in their own transactions, oldest first. If a later step fails,
   for example a decision or fill conflict (exit 5), the sessions already
   processed **stay processed**, and the stored bars of later sessions
   **stay stored**. Nothing rolls them back.

A daily run is therefore **not** one transaction, and FG-9 does not make it
one. What FG-9 adds is that everything committed in step 2 was decided on
bars that passed step 1. A rerun resumes after the latest frozen decision.
If the provider still serves the approved versions, the stored bars are
accepted as equal and the paper guard re-verifies them; otherwise the rerun
stops with exit 8 or 5.

**Option, not recommended now: process the verified prefix.** Today the
operator lowers final-through instead (an FG-8 change). Prefix processing
would add a second behaviour to test and reason about.

## 9. Failure taxonomy

| Case | Detected | Exit | Notes |
|------|----------|------|-------|
| Mode `operator-reviewed` without `NORTHSTAR_FUTURES_APPROVED_CANDLES` | Settings load | 3 | Dashboard settings also fail; nothing runs |
| Malformed manifest (grammar, case, duplicate key) | Settings load | 3 | |
| Unsupported canonical format (prefix other than `fdc1-sha256:`) | Settings load | 3 | |
| `operator-approved` with an eligible session (FG-9 build) | Before the provider request | 3 | Message points to N-4 FG-9 |
| No approved identity for an eligible session (includes approval through a date with an unreviewed intermediate session) | Use case, before the provider request | **8** | Lists the session dates only |
| Approval revoked (entry removed) before ingestion | Same as no identity | **8** | |
| Approval revoked after ingestion | Not detectable: the session is no longer eligible | -- | N-4 ESCALATE; nothing is undone |
| Provider candle differs from the approved identity | Use case, after building bars, before `_persist` | **8** | Lists dates; prints no value or fetched digest (I-9) |
| Missing provider candle | `_require_coverage` | 4 | Unchanged |
| Provider returns an extra session | `fetch_daily_candles` (outside the range: exit 6) or `_require_coverage` (not a session: exit 4) | 6 or 4 | Unchanged |
| Contract mismatch | `_index_observations` | 6 | Unchanged |
| Different expiry in the manifest | No entry for the operated contract | **8** | A wrong digest for the right key is also exit 8 |
| Incomplete price or volume field | Adapter `_labelled` and `_number` | 6 | Unchanged |
| Existing stored-bar conflict | `_store_one` | 5 | Unchanged; now also means "the approved candle differs from a stored bar": N-4 ESCALATE |
| Expiry exception | Before acquisition | 7 | Unchanged, and it still takes precedence: the approval checks run inside acquisition, after S-2 |
| Paper run cutoff after the last eligible session | Paper guard | 5 | New; same family as `_refuse_backward_run` (exit 5) |
| Paper run over an eligible session whose stored bar is missing or differs from its identity (`operator-reviewed`) | Paper guard | **8** | New |
| Paper run over an eligible session in `operator-approved` (FG-9 build) | Paper guard | 3 | New (I-5) |
| Provider `databento` for an NSE contract (`operations daily`, `market-data sync`) | Before any request | 3 | New explicit refusal; today exit 3 (no key) or 6 (calendar) by accident |

**Mode and exit matrix.** "Existing" means the behaviour in the code read
at `cfa7774`; "new" means it must be implemented.

| Situation | `disabled` | `operator-approved` | `operator-reviewed` (new mode) |
|-----------|------------|---------------------|--------------------------------|
| `operations daily`, no eligible session | `WAITING`, exit 0 (existing) | `WAITING`, exit 0 (existing) | `WAITING`, exit 0 (new; same path) |
| `operations daily`, eligible sessions | Never: every session is `UNKNOWN` (existing) | Today: acquires and trades. **New:** exit 3 before any request | Verified acquisition, then trading; `COMPLETED`, exit 0 (new) |
| Missing approval for an eligible session | -- | -- (refused as above) | Exit 8 before any request (new) |
| Malformed or missing manifest | Variable not read (new; ignored) | Variable not read (new; ignored) | Exit 3 at settings load; the API also refuses its settings (new) |
| Fetched candle mismatches | -- | -- | Exit 8 before any write (new) |
| Previously stored bar, equal | -- | -- | Accepted after verification (existing store rule) |
| Previously stored bar, different | -- | -- | Exit 5 after verification (existing store rule) |
| Exposed contract past expiry, or out of sessions | Exit 7 (existing) | Exit 7 (existing) | Exit 7 (new mode, same path, still first) |
| Manual `market-data sync --provider upstox` | Stores unverified bars (existing; such bars can never be traded, see paper guard) | Same as `disabled` | Identity required for every session in the range; else exit 8 (new) |
| Manual `market-data sync`, provider `databento`, NSE | Exit 3 (new) | Exit 3 (new) | Exit 3 (new) |
| Manual `paper run`, cutoff equal to the latest decision (replay) | Allowed (existing) | Allowed (existing) | Allowed (existing) |
| Manual `paper run`, cutoff after the latest decision | Today: allowed, unguarded. **New:** exit 5 (nothing eligible) | Today: allowed, unguarded. **New:** exit 3 over an eligible session, exit 5 beyond | Today: not a mode. **New:** exit 5 beyond final-through; exit 8 unless every eligible stored bar up to the cutoff matches |
| Mode unset or blank | `disabled` (existing, `load_session_operation_settings`) | -- | -- |

Exit 0 is reserved for `WAITING`, `COMPLETED` and `SKIPPED`. A
verification failure is never 0 (I-7). Exit 7 keeps its meaning and its
precedence over 8 in `operations daily`; manual commands never report 7, as
today.

**Why a new code:**
- 4 means the provider and the calendar disagree;
- 5 means stored evidence disagrees;
- 6 means the provider broke its contract;
- 3 means the configuration is invalid.

A valid configuration whose approved version is absent or not served is
none of these, and the operator's response differs: follow N-4, never retry.
8 is unused by `ExitCode` and outside the wrapper's 10 to 13.

**Propagation:**
- **CLI.** `_acquire` gains one `except` for the new approval errors, mapping
  them to `ExitCode.CANDLE_APPROVAL`. Without it they would surface as an
  unhandled exception rather than a classified one. The existing `_fail`
  prints `CANDLE_APPROVAL ERROR: <message>` on stderr; stdout carries no
  `STATUS:` line.
- **Windows wrapper.** `Invoke-NorthstarIndiaOperations.ps1` already passes
  any Northstar code through. A non-zero code other than 7 becomes outcome
  `FAILED`, with `reason` from the first `^[A-Z_]+ ERROR: ` line, so it is
  never `WAITING` (I-7). Add `8 = 'CANDLE_APPROVAL'` to `$ExitClasses`
  (otherwise `exitClass` is `UNKNOWN`) and document 0 to 8.
- **Scheduler.** The task records the wrapper's exit code; no change.
- **Logs.** The run log keeps the header, the session assessments and the
  error line.
- **Dashboard.** It reports no run outcome by design (`operational_status.py`
  docstring); unchanged. It must accept the new mode value: the
  `FinalityResponse.mode` `Literal` in `northstar-api/src/northstar_api/schemas/futures.py`
  and `mode` in `northstar-web/src/futures/types.ts`.

## 10. Security and G-1 restrictions

| Item | Contains | G-1 position |
|------|----------|--------------|
| Manifest entry | Expiration, session date, digest of Upstox-derived values | **Upstox-derived.** Not exempt because it is a hash: candle values come from a narrow, partly known range, so a digest may be reversible by search, and licensing terms may cover derived data regardless |
| Identity command output | The same entry, endpoint class, `requested_at` | Upstox-derived, as above |
| Evidence file | Raw values, open interest, instrument key, lot size | Upstox data; private and local (evidence guide) |
| Decision record (N-4 section 14) | Should reference the evidence line and say whether the entry was published, not copy the digest, until G-1 decides | Metadata |
| Failure messages | Session dates and contract only | Metadata (I-9) |
| Credentials | None in any new artifact; the token stays in `UPSTOX_ANALYTICS_TOKEN` | Unchanged |

**Before G-1 is resolved:**
- **Can be built and tested:** everything in section 13, with synthetic
  candles, a fake `fetch`, synthetic evidence files and disposable SQLite
  databases.
- **Cannot be done:** running the identity command on real evidence,
  placing real entries in production `.env`, or recording digests. Each
  retains or uses Upstox-derived data.

Technical feasibility does not grant that permission. A keyed digest (HMAC
with a local secret) would resist reversal, but it would not change the G-1
question, so it is not proposed.

**Manifest integrity requirements (mandatory):**

| # | Requirement | How |
|---|-------------|-----|
| M-1 | Versioned | Every digest carries the `fdc1-sha256:` prefix, and the hashed text starts with `northstar.futures-daily-candle-identity/1`. Any other prefix is exit 3; a future format needs a new prefix and code that accepts it |
| M-2 | Strictly parsed | The whole value must match the grammar exactly: comma-separated `<YYYY-MM-DD>/<YYYY-MM-DD>=fdc1-sha256:<64 lowercase hex>`, no inner whitespace, no empty element, real calendar dates. Anything else is exit 3, never a partial parse |
| M-3 | No duplicate or conflicting entries | A repeated key is exit 3, whether the digests are equal or not; there is no "last one wins" |
| M-4 | Bound to contract and session | The key binds expiration and session date; the digest also binds product, exchange, expiration, date and timeframe. An entry for another contract cannot match the operated one (exit 8) |
| M-5 | Fail closed | Missing variable or malformed value: exit 3. Missing entry, including a revoked one: exit 8. Mismatch: exit 8. Nothing ever falls back to `operator-approved` or `disabled` behaviour |
| M-6 | Not proof of identity | Anyone who can write `.env` can add an entry, and the digest of the version the provider currently serves would then pass. The manifest proves only that the bytes match. Who approved is proven by the N-4 decision record, and the right to write `.env` by the FG-8 authorization and the file's access control |
| M-7 | Change evidence | Each change to the variable is part of an FG-8 change record. The M1.5.2 runbook already records `.env`'s SHA-256, which reveals nothing |

**Three authorizations stay separate:**
1. **Operator approval of a version:** the N-4 APPROVE record, naming the
   evidence line; made by the operator, and by the approver for ESCALATE.
2. **Manifest integrity:** entries are published only under an FG-8
   authorization, are parsed strictly (M-1 to M-5), and `.env` keeps its
   access control.
3. **Release authorization:** M1.5.2 AP-1 for the build that enforces FG-9.

Holding one never implies another: a release approves no candle, an
approval writes no `.env`, and a manifest entry identifies no approver.

No production fingerprint was created for this design, and no Upstox value
was copied.

## 11. Migration and deployment implications

| Topic | Effect |
|-------|--------|
| Schema | **None.** No table, column or migration. Existing SQLite history is read exactly as now |
| Existing modes | `operations daily` in `disabled` is unchanged. `operator-approved` is unchanged while nothing is eligible, and refuses acquisition when something is (I-5). Manual `paper run` on NSE gains the paper guard in every mode (section 9 matrix) |
| Enforcement scope | Verified acquisition only in `operator-reviewed`, the only mode that can acquire under FG-9. The paper guard and the Databento refusal apply to NSE contracts in every mode |
| Startup | Settings validation fails closed (exit 3) for a missing or malformed manifest. Missing approvals fail at run time (exit 8), before any request |
| Scheduled task | No change to the task XML or its registration. The wrapper's class table and header change |
| Compose | Add `NORTHSTAR_FUTURES_APPROVED_CANDLES: ${NORTHSTAR_FUTURES_APPROVED_CANDLES:-}` to the shared `x-futures-settings` block in `northstar-api/deploy/india/compose.yaml`, so `operations daily`, `market-data sync` and the API's settings validation all see it |
| PowerShell 5.1 | The wrapper change is one hashtable entry and comment text; no new syntax |
| M1.5.2 release | The FG-9 release changes no `.env` (the runbook's rule). After upgrade with the recorded `.env`, the run still reports `WAITING`. Switching to `operator-reviewed` is a later, separately authorized FG-8 change |
| Rollback of the FG-9 release before that switch | The `.env` is unchanged; the previous build waits as now |
| Rollback of the FG-9 release after the switch | The runbook (section 10, step 4) restores the pre-upgrade `.env` copy **because it changed**. The previous build then runs `operator-approved` with the **pre-upgrade** final-through. Safe only while that date is not after the latest frozen decision's session (RB-1) |
| Rollback to a pre-FG-9 build with `operator-reviewed` still in `.env` | That build rejects the mode: `operations daily` exits 3 and the API cannot start. Nothing is acquired by the scheduled path (I-8). Its manual commands are **not** protected (RB-3) |
| Release provenance | The runbook's release manifest records commits and image hashes; the FG-9 commit must be one of them |
| Persisted trades | Untouched. No code path reads approvals for sessions already decided |

**Enablement check (runbook addition).** Before the first switch, record
the dashboard's latest market session (a date only). If it is after the
current final-through, an unreviewed bar may already be stored (N-4 RC-8).
The paper guard and the store's conflict rule keep that bar from being
traded unless it equals its approved identity, but the operator ESCALATEs
before switching.

**Rollback verification (performed read-only on the development machine).**
The deployed build is unidentified (E-4), so two candidate baselines were
checked:
- `northstar-api` `main` `76cc3e9`;
- the M1.5.2 release `develop` `cfa7774`.

Each one's `settings.py` was extracted with `git show` and executed against
synthetic environments, with no checkout, Docker or database involved:

| Baseline | `operator-approved` | `operator-reviewed` (any case) |
|----------|---------------------|--------------------------------|
| `76cc3e9` | Accepted by `load_session_operation_settings` and `load_dashboard_settings` | **Refused** by both: `DashboardSettingsError`, "NORTHSTAR_FUTURES_DAILY_BAR_FINALITY is invalid" |
| `cfa7774` | Accepted by both | **Refused** by both, same error |

At both baselines, `_market_data_sync`, `_paper_run` and `_run_paper_session`
never read the finality mode (verified with `git show` of `cli.py`). So an
older build rejects the new mode **only** on the scheduled path and in the
API.

**Rollback STOP conditions (mandatory runbook additions):**

| # | STOP when |
|---|-----------|
| RB-1 | The target build lacks FG-9, and the `.env` it will run with has a final-through after the latest frozen decision's session (the dashboard shows approved finality ahead of the latest decision). The older build would acquire and trade those sessions unverified |
| RB-2 | The target build lacks FG-9, `.env` still says `operator-reviewed`, and anyone proposes editing the mode to `operator-approved` to "fix" the refused start. That edit re-enables unverified acquisition up to final-through |
| RB-3 | Any manual `market-data sync` or `paper run` against the production database is proposed while a pre-FG-9 build is deployed, for a session after the latest frozen decision |
| RB-4 | Any FG-8 advance of final-through is proposed while a pre-FG-9 build is deployed |
| RB-5 | The deployed or rollback-target build is neither of the two verified baselines. Re-run the extraction check against that exact commit first |

**Reverting configuration can permit unverified work.** Restoring an older
`.env` (RB-1) or hand-editing the mode (RB-2) are the two ways. Both are
operator actions, so both are STOP conditions rather than code paths.

## 12. Synthetic acceptance test matrix

Every test uses synthetic candles, a fake transport or source, and a
disposable SQLite file under `tmp_path`.

**Every negative test asserts all of these** (assertions NA-1 to NA-7):

| # | Assertion | How |
|---|-----------|-----|
| NA-1 | No candle write | The full ordered contents of `futures_ohlcv` are equal before and after the command, not only its row count |
| NA-2 | No decision | `futures_forward_research_records` contents are equal before and after |
| NA-3 | No order | `futures_paper_orders` contents are equal before and after |
| NA-4 | No fill | `futures_paper_fills` contents are equal before and after; a pending order stays pending |
| NA-5 | No configuration or finality change | The synthetic environment mapping is unchanged, and no file outside `tmp_path` is written |
| NA-6 | No success or waiting report | The exit code is the listed non-zero code, the stderr line starts with that code's name and `ERROR:`, and stdout has no `STATUS: WAITING` or `STATUS: COMPLETED` |
| NA-7 | No fallback | The same input under the mode named in the test never yields the weaker mode's outcome; the asserted refusal is the one listed for that mode |

Failures detected before the provider request also assert that the fake
transport received zero calls.

| # | Case | Expected |
|---|------|----------|
| T-1 | Single session, matching | Stored; decided |
| T-2 | Same session, different close | Exit 8; nothing stored |
| T-3 | Same session, different open | Exit 8 |
| T-4 | Same session, different volume (same lot size) | Exit 8 |
| T-5 | Entry built for a different product | Exit 8 |
| T-6 | Entry keyed to a different expiry | Exit 8 (no entry for the operated contract) |
| T-7 | No entry for an eligible session | Exit 8, **zero provider requests** (fake transport call count) |
| T-8 | Provider omits a session | Exit 4; nothing stored |
| T-9 | Provider returns an extra non-session date | Exit 4; outside the range, exit 6 |
| T-10 | Three sessions, all matching | All stored in one batch; decided oldest first |
| T-11 | Three sessions, middle mismatches | Exit 8; the first is **not** stored |
| T-12 | Three sessions, last mismatches | Exit 8; none stored |
| T-13 | Identical bar already stored, entry matches | Idempotent; run proceeds |
| T-14 | Different bar already stored, provider returns the approved version | Exit 5; nothing changed |
| T-15 | Repeated run after T-1 | Nothing eligible: `WAITING`; no new facts |
| T-16 | Malformed manifest (bad grammar, uppercase hex, duplicate key, unknown prefix, missing variable) | Exit 3 at settings load; dashboard settings also refuse |
| T-17 | Entry removed after T-7 setup (revocation) | Exit 8 |
| T-18 | Final-through spans three sessions; only the first and third have entries | Exit 8 before any request; lists the second |
| T-19 | A pending order from the previous decision; next session mismatches | Exit 8; the order stays pending, no fill |
| T-20 | An open position; next session mismatches | Exit 8; position, fills and mark inputs unchanged |
| T-21 | Decimal spellings (`100.50` against `100.5`, `1E+2` against `100`, a 30-digit fraction, `-0`) | One digest per value; a one-digit change gives a different digest |
| T-22 | Labels `T00:00:00+05:30` and the equivalent UTC instant name one session; a label without an offset is exit 6; a label naming the previous IST day matches a different session | As stated |
| T-23 | `operator-approved` with an eligible session (FG-9 build) | Exit 3, zero provider requests |
| T-24 | `operator-approved` with nothing eligible | `WAITING`, exit 0, unchanged |
| T-25 | Expiry exception with missing approvals | Exit 7, not 8 |
| T-26 | Identity command on a synthetic evidence line | The printed entry equals the digest of the bar the acquisition builds from the same synthetic response; refuses `candle: null` and `volume_contracts: null`; prints no value |
| T-27 | Wrapper, fake Northstar exiting 8 with `CANDLE_APPROVAL ERROR:` | Exit 8 passed through; outcome `FAILED`; `exitClass` `CANDLE_APPROVAL`; reason set |
| T-28 | `market-data sync` in `operator-reviewed` over an unapproved session | Exit 8; nothing stored |
| T-29 | Dashboard in `operator-reviewed` mode | `FinalityResponse.mode` serializes; the web type accepts it |
| T-30 | A pre-FG-9 settings loader given `operator-reviewed` | The baseline extraction check of section 11, run against each rollback-target commit: both loaders refuse. (The existing `unknown-mode` test in `test_india8b_nse_chronological_operations.py` uses another value on the current build, so it does not prove this) |
| T-31 | `operator-reviewed`, manual `paper run` with cutoff after final-through | Exit 5; NA-1 to NA-7 |
| T-32 | `operator-reviewed`, a stored bar inserted directly into the synthetic database (simulating a pre-switch bar) that differs from its identity; manual `paper run` over it | Exit 8; NA-2 to NA-4; the stored row itself is unchanged |
| T-33 | `operator-reviewed`, eligible session with no stored bar; manual `paper run` | Exit 8; nothing decided |
| T-34 | `operator-reviewed`, every eligible stored bar matches; manual `paper run` | Proceeds; facts equal those of the scheduled run for the same inputs |
| T-35 | `disabled`, manual `paper run` after the latest decision | Exit 5; NA-1 to NA-7 |
| T-36 | `operator-approved` (FG-9 build), manual `paper run` over an eligible session | Exit 3; NA-1 to NA-7 |
| T-37 | Any mode, `paper run` at exactly the latest decision (replay) | Allowed; no new decision; an existing pending order settles only within its own cutoff (existing rule) |
| T-38 | Any mode, `operations daily` and `market-data sync` with provider `databento` for an NSE contract | Exit 3; zero transport calls; NA-1 to NA-6 |
| T-39 | `operator-reviewed`, manual `market-data sync --provider upstox` over matching sessions, then `paper run` | Stored; then decided; identical facts to T-34 |
| T-40 | Mode variable removed or blank, where `operator-reviewed` was set | Treated as `disabled`: `operations daily` is `WAITING`, manual `paper run` after the latest decision is exit 5; never `operator-approved` behaviour (NA-7) |
| T-41 | Chronological run in which the store accepts the batch and a later session's decision conflicts | Exit 5; earlier sessions stay processed and later bars stay stored, as today; asserts the documented non-atomicity rather than a rollback |
| T-42 | Rollback rehearsal on synthetic data: an FG-9 build's database and the restored pre-upgrade `.env` (`operator-approved`, final-through at the latest decision's session) | `WAITING`; with final-through one session later, the RB-1 check (dashboard: finality ahead of the latest decision) is true |

## 13. Minimal implementation plan

**Mandatory, in dependency order:**

| # | Repository and module | Responsibility | API or format impact | Errors | Tests | Docs |
|---|-----------------------|----------------|----------------------|--------|-------|------|
| 1 | application: new `application_services/futures_daily_candle_identity.py` | `fdc1` text and digest from contract, date and bar; approval-set value with its parser | New public functions and value; no port change | `InvalidFuturesCandleApprovalSetError` | T-21, T-22, parser half of T-16 | This document |
| 2 | application: `acquire_futures_native_daily_history.py` | Optional approval set; entry check before fetch; digest check before `_persist` | Constructor gains an optional keyword argument; default unchanged | `FuturesCandleApprovalMissingError`, `FuturesCandleApprovalMismatchError` (a shared base), dates only | T-1 to T-14, T-17 to T-20 at use-case level in `tests/application_services/test_acquire_futures_native_daily_history.py` | |
| 3 | api: `settings.py` | Mode `operator-reviewed`; parse `NORTHSTAR_FUTURES_APPROVED_CANDLES` in that mode; the mode reads the token | `FuturesSessionOperationSettings` gains the approval set | Exit 3 cases | T-16 | |
| 4 | api: `runtime.py` | The policy for the new mode; pass the approval set into `build_upstox_market_sync_runtime` | Builder keyword argument | | T-29 | |
| 5 | api: `cli.py` | `ExitCode.CANDLE_APPROVAL = 8`; map approval errors in `_acquire`; `operator-approved` refuses eligible sessions; `market-data sync` composes with the approval set in reviewed mode; `finality-evidence identity` | New exit code and subcommand | | T-7, T-15, T-18, T-23 to T-26, T-28 in `test_india8b_nse_chronological_operations.py` and `test_finality_evidence_cli.py` | CLI help |
| 5a | api: `cli.py` | **Paper guard** (section 6, item 8): one function called at the start of `_run_paper_session`. `_run_paper_session` gains the session settings, which `_paper_run` loads from the environment for NSE contracts (the other three callers already hold them) | Internal signature only; manual `paper run` on NSE now needs the finality variables, as `operations daily` does | Exit 3, 5, 8 per the section 9 matrix | T-31 to T-37, T-39, T-40 | `paper run` help |
| 5b | api: `cli.py` | **Databento refusal for NSE** in `_daily_operation` and `_market_data_sync` (section 6, item 9) | None | Exit 3 | T-38 | |
| 6 | api: `schemas/futures.py`; web: `src/futures/types.ts` | Accept the new mode value | Response `Literal` widened | | T-29; web type test | |
| 7 | api: `deploy/india/compose.yaml` | Pass the variable | Compose | | `test_india8e_indian_deployment.py` | |
| 8 | api: `deploy/india/windows/Invoke-NorthstarIndiaOperations.ps1` | Exit class 8; header text | `last-run.json` `exitClass` value | | T-27 in `test_india_windows_operations_scheduler.py` | |
| 9 | docs | N-4 FG-9 references the mechanism; deployment runbook section 5 (the reviewed-mode `.env` change, and that manual `paper run` now needs the finality variables); evidence guide (identity command); M1.5.2 runbook (enablement check, RB-1 to RB-5, re-running the baseline extraction check for the actual rollback target); acceptance specification row for FG-9 evidence | | | Validators; T-30, T-42 | |

**Operator approvals needed:**
- before merge: acceptance of this design;
- after release, each under its own authorization:
  - running the identity command on real evidence (G-1);
  - switching `.env` to `operator-reviewed` with entries (FG-8);
  - any final-through advance (FG-8);
  - scheduler enablement (existing).

**Optional, later:**
- a dashboard field showing which upcoming sessions have entries (dates
  only);
- verified-prefix processing;
- a per-field mismatch hint computed only on the development machine from
  evidence.

## 14. Open engineering and operator decisions

| # | Decision | Owner | Recommendation |
|---|----------|-------|----------------|
| OD-1 | Accept approach A with the `operator-reviewed` mode | Engineering and approver | Accept |
| OD-2 | Exit code 8 named `CANDLE_APPROVAL` | Engineering | Accept |
| OD-3 | Whether `operator-approved` should refuse eligible sessions in FG-9 builds, or be removed | Engineering and approver | Refuse (keeps the recorded `.env` valid) |
| OD-4 | **Resolved: mandatory.** The paper guard and the Databento refusal are part of FG-9 (plan rows 5a and 5b), not a follow-up | -- | -- |
| OD-8 | Exit 5 for "paper run beyond approved finality", the same family as the existing backward-run refusal | Engineering | Accept |
| OD-9 | Whether production `.env` and runbooks must also forbid running Northstar commands against the production database from any process that does not receive `.env` (outside `docker compose run`) | Operator and approver | Forbid. Such a process sees no mode, so it behaves as `disabled`: guarded for paper writes, but able to store unverified bars |
| OD-5 | Whether digests may be retained in `.env` and decision records | G-1 owner | Blocks production use, not implementation |
| OD-6 | Whether a review from the same-day endpoint may ever be approved, given that ingestion of a past date uses the historical endpoint | Operator (N-4 RC-2) | Approve only historical-endpoint records. A same-day record matches only if the historical endpoint later serves identical values |
| OD-7 | Whole-batch failure against verified-prefix processing | Engineering | Whole batch |

## 15. Design readiness verdict

**READY FOR ENGINEERING IMPLEMENTATION**, on the basis that:
- the code path is traced to source;
- every invariant has a concrete enforcement point in existing structures;
- every supported write-capable entry point is covered (section 2 table);
- rollback rejection of the new mode is verified against both candidate
  baselines, and its limits are STOP conditions (RB-1 to RB-5);
- no schema change, port redesign or broad restructuring is needed;
- every behaviour is testable with synthetic data before G-1.

This verdict covers implementation and synthetic verification only. It does
not cover:
- production use, which also needs G-1 (OD-5), release under the M1.5.2
  runbook, and separate FG-8 authorizations;
- October 9, which stays DEFER with final-through at 2026-10-08.

N-4 stays PENDING, and FG-9 stays unmet until the implementation is
released and verified. X-4 and D-4 stay BLOCKED; B-1, B-2 and P-3 stay
PENDING; G-1 stays unresolved.
