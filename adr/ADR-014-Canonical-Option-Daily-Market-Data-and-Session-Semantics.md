# ADR-014: Canonical Option Daily Market Data and Session Semantics

## Status

Accepted

## Date

2026-10-08

## Decision Makers

Northstar Architecture Team

## Context

ADR-013 preserves which Upstox instrument represented each exact NIFTY option contract. Before option candles can be acquired, Northstar needs the canonical shape an option daily bar takes, the instant at which it is stamped, and the trading sessions that decide which daily bars exist.

Futures already answer those questions for futures: a `FuturesOHLCVBar` carries `QuoteValue` prices and a contract-count volume, and a provider-native daily candle is stamped at the resolved session close from a cited futures session regime. Those answers are futures-specific. Option prices are premiums, not signed quotations. And the futures regime is futures hours: since 2025-12-08 futures open at 09:00 IST with a pre-open session, and NSE's derivatives pre-open applies to single-stock and index futures only. Applying it to options would stamp option bars with a session they do not have.

NSE's Equity Derivatives market timings give the normal market as 09:15 to 15:40 IST. The 15:40 close is in force from 2026-08-03 (NSE/FAOP/74467, 2026-05-29). Northstar holds no authoritative source for the hours option contracts traded before that date.

Upstox candles also carry open interest. It is valuable to options research and, because Upstox rejects an expired contract's instrument key, it may become unrecoverable once a contract expires. Its units, the instant it describes and how it is revised are not established.

## Decision

### Core

- `OptionOHLCVBar(contract, point_in_time, timeframe, open, high, low, close, volume)` is an Options-specific bar in `northstar_core.options`, alongside and not derived from `FuturesOHLCVBar`.
  - `contract` is the exact `OptionContract`.
  - `open`, `high`, `low` and `close` are `OptionPremium` values. The bar enforces that the high bounds open, close and low, and the low is bounded by open, close and high.
  - `volume` is a `Quantity` counting option contracts, and must be integral.
  - Its identity is `(contract, point_in_time, timeframe)`, exposed as `natural_key`.
  - It carries no open interest, provider or source metadata, bid or ask, settlement price, implied volatility or Greeks.
- The Options boundary test now allows `Quantity`, deliberately, for the bar's volume. No other boundary rule changes.

### Option Trading Sessions

- `OptionTradingSession(trading_date, opens_at, closes_at)`, `OptionTradingSessionResolver` and `OptionTradingSessionResolutionError` are Application contracts, separate from the futures session contracts. The resolver's one operation is `sessions_in_range(product, start_date, end_date)`.
- `NSEOptionTradingSessionResolver` resolves NIFTY@NSE option sessions from a cited regime and the NSE F&O day-level calendar reused unchanged: `HOLIDAYS`, `SPECIAL_SESSIONS` and `LOADED_YEARS`. The futures `REGIMES` and the futures resolver are not used.
- The one encoded regime is NIFTY@NSE from 2026-08-03: normal market 09:15 to 15:40 IST. No earlier regime is inferred.
- A requested date resolves as follows:
  - an unloaded calendar year fails closed, before any date is answered;
  - a date before the first regime fails closed;
  - a published special session fails closed, whatever timings the calendar lists, because those timings are not established for options;
  - a weekend or a published trading holiday is not a session;
  - every other date is a session from 09:15 to 15:40 IST, converted to UTC.

### Application

- `OptionNativeDailyObservation(contract, trading_date, open, high, low, close, volume)` is the provider-neutral daily candle a source returns. It carries a session label and no instant, no provider identifier and no open interest.
- `OptionNativeDailyMarketDataSource.fetch_daily_observations(contract, start_trading_date, end_trading_date)` acquires one exact contract over an explicit inclusive range. The caller chooses the contract; there is no selection, no default and no clock.
- `OptionHistoricalMarketDataStore`, `OptionHistoricalMarketDataRepository` and `OptionHistoricalMarketDataConflictError` follow the Futures store semantics: insert-only, idempotent equal re-store, conflict on change, all-or-nothing batches.
- `AcquireOptionNativeDailyHistoryUseCase` takes an `OptionDailyAcquisitionQuery`, resolves the range's sessions, fetches the source once, requires exactly one observation per resolved session and none elsewhere, stamps each `OptionOHLCVBar` at its session's close with the daily `1d` timeframe, and stores the range as one batch.

Strict coverage is deliberately conservative. Whether an illiquid but listed strike legitimately has no daily candle is not yet known; a missing candle therefore fails the acquisition until provider evidence shows otherwise.

Acquisition is not finality. Storing a candle says what the provider returned, not that it will not be revised.

### Infrastructure

- `option_ohlcv` stores canonical option bars, keyed by the exact contract (product, exchange, expiration, strike, right), the timeframe and the instant. Every value is canonical TEXT.
- `option_daily_provider_open_interest` preserves provider open interest as Infrastructure-only evidence, keyed by provider, exact contract and trading date. The value is the exact text of the parsed provider number, with no unit conversion and no unit claim. An identical repeat is idempotent and a changed value conflicts.
- Both tables are insert-only and created by their own initializers, only by their stores immediately before a write. Read-only repositories open SQLite in read-only mode and never create a file or schema. Neither table is part of the Futures database initializer.

## Architecture Invariants

1. **Options-specific bar.** `OptionOHLCVBar` is not derived from, and does not generalize, `FuturesOHLCVBar`.
2. **Session close is the instant.** A canonical option daily bar is stamped at its resolved option session close, never at a provider timestamp.
3. **No borrowed hours.** Option session hours come only from a cited option regime; futures hours and the futures pre-open are never used.
4. **Fail closed.** Unloaded calendar years, dates before the first regime and special sessions are errors, never guesses.
5. **Strict coverage.** One observation per resolved session and none elsewhere, until evidence justifies a change.
6. **Open interest is evidence, not domain data.** Provider open interest is preserved raw in Infrastructure and never exposed through Core or Application.
7. **Insert-only market data.** Stored bars and open interest are immutable; a differing value is a conflict.
8. **No Futures change.** No Futures type, table, initializer, resolver or test changes.

## Rationale

### Why Not Reuse the Futures Session Resolver

The futures resolver returns futures hours, including the futures-only pre-open, and is typed on futures products. Only the day-level calendar facts are shared by the whole F&O segment, and only those are reused.

### Why Fail Before 2026-08-03

Northstar has no primary source for earlier option hours. Inferring them from the futures regime would import the futures pre-open; assuming the current hours applied earlier would invent history. Failing closed keeps the gap visible until a source is added.

### Why Special Sessions Fail Closed

The calendar's special-session timings were sourced for futures, and the unresolved 2026-11-08 Muhurat has none at all. An option bar stamped from them could be stamped at the wrong instant.

### Why Preserve Open Interest Now

Once a contract expires, its candles, and so its open interest, cannot be fetched again from the standard endpoint. Recording the raw value while it is available costs one small table; deciding its units and meaning can wait for research.

## Alternatives Considered

- **Genericize `FuturesOHLCVBar`.** Rejected: it would change a frozen Futures contract and conflate premiums with signed quotations.
- **Use the futures session regime.** Rejected: futures hours include a futures-only pre-open.
- **Assume 09:15 to 15:40 before 2026-08-03.** Rejected: unsourced.
- **Key option bars by trading date only.** Rejected: it departs from the canonical bar's instant semantics.
- **Add open interest to `OptionOHLCVBar`.** Rejected: its units and revision behaviour are unknown.
- **Drop open interest.** Rejected: it may become unrecoverable.
- **Allow gaps for illiquid strikes now.** Deferred: no evidence yet.

## Consequences

### Positive Consequences

- Option daily bars have one canonical shape and one instant rule.
- Option and futures sessions cannot be confused.
- Provider open interest is retained while it can still be obtained.

### Implementation Consequences

- Option bars can be acquired only for sessions from 2026-08-03, and not across a special session.
- A range containing a session without a candle fails acquisition.

### Future Consequences

- INDIA-OPT-4C adds the Upstox option candle adapter, which populates both tables, and the operator command.
- Strict coverage may be revisited with provider evidence.
- An earlier option regime can be added once sourced.

## Scope

This ADR establishes canonical option daily market data and option session semantics.

It does not:

- acquire candles from Upstox or add a command;
- introduce current-day routing or finality;
- give open interest domain meaning, units or a Core type;
- introduce option replay snapshots, implied volatility, Greeks or bid and ask;
- change ADR-009 through ADR-013, or any Futures component.

## Future Considerations

Deferred:

- the Upstox option native-daily adapter and the options market-data command (INDIA-OPT-4C);
- earlier option session regimes;
- the option timings of special sessions;
- gap semantics for illiquid strikes;
- open-interest units, revision policy and research semantics;
- option replay snapshots.

## Related Documents

- ADR-010: Option Contract Identity
- ADR-012: NSE NIFTY Option Contract Reference and Expiration Rules
- ADR-013: Option Provider Listing Reference and Historical Reproducibility
- Ubiquitous Language: `architecture/Ubiquitous-Language.md`
- NSE Equity Derivatives market timings
- NSE/FAOP/74467 (2026-05-29): normal-market close 15:40, effective 2026-08-03
