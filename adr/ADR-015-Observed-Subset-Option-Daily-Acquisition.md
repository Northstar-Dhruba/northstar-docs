# ADR-015: Observed-Subset Option Daily Acquisition

## Status

Accepted

Supersedes only the strict coverage assumption of ADR-014 (its Application acquisition rule and Architecture Invariant 5). Every other ADR-014 decision stands, and ADR-014 is not edited.

## Date

2026-10-08

## Decision Makers

Northstar Architecture Team

## Context

ADR-014 required strict coverage: exactly one daily candle for every resolved option trading session, and none elsewhere. It recorded that whether an illiquid but listed strike legitimately has no daily candle was not yet known, and that the rule could be revisited with provider evidence.

INDIA-OPT-4C gathered that evidence with a read-only live probe on 2026-10-08. It requested Upstox v3 historical daily candles for 24 NIFTY option contracts over 2026-08-03 to 2026-10-07: calls and puts, near-ATM and far-OTM strikes, and weekly, monthly, quarterly and long-dated expiries. The NSE option session resolver gave 46 sessions in that window.

- **Liquid contracts were contiguous.** Once a near-dated, near-ATM contract had a first candle, every later session had one.
- **Upstox omits candles.** Several contracts had a resolved session with no candle while a sibling contract had one on the same date. Examples: 2026-10-27 22600 CALL on 2026-09-18, 2026-11-23 22600 CALL on 2026-09-25, and 2027-03-30 22500 CALL on 2026-08-11 and 2026-08-17. Some listed contracts had no candle in the window at all.
- **No zero-volume candle appeared.** Across 311 candles there was no zero volume, no zero price, no duplicate date and no candle on a date that is not a session.
- **NSE confirms the gaps were listed, untraded contracts.** The public NSE F&O bhavcopy for 2026-09-18, 2026-09-25 and 2026-08-11 lists each checked gap contract with zero traded volume, a settlement price, open interest, and open, high and low of 0.00. Between 798 of 1,700 and 874 of 1,794 NIFTY option contracts had zero traded volume on each of those days.
- **Upstox matches NSE where a contract traded.** For four traded contracts, Upstox open, high, low and close equalled the bhavcopy exactly, raw volume divided by the lot of 65 equalled NSE's traded contracts, and raw open interest equalled NSE's reported open interest.
- **The master has no listing history.** The Upstox instrument master carries no historical listing-start date. The first candle of a contract is therefore not its listing date: on the 2026-10-13 expiry, the 22600 PUT's first candle was 2026-09-11 and the 22600 CALL's was 2026-09-28.

Under strict coverage, any range containing one untraded session fails and stores nothing. Since Upstox rejects an expired contract's instrument key, the traded sessions of an illiquid contract could then never be stored at all.

## Decision

### Observed-Subset Coverage

`AcquireOptionNativeDailyHistoryUseCase` applies observed-subset coverage:

1. The complete set of option trading sessions for the requested range is resolved exactly as before.
2. The source is asked once for the exact contract's observations.
3. Every returned observation must correspond to exactly one resolved session.
4. An observation for an in-range date that is not a resolved session is an `OptionDailySessionCoverageError`.
5. A duplicate observation date, an observation outside the requested range, or an observation for another contract remains a source contract violation.
6. A resolved session with no observation is allowed. It is a **session without a provider candle**.
7. Nothing is manufactured for a session without a provider candle: no zero bar, no carried-forward bar, no settlement-price bar, no synthetic close and no synthetic volume. The NSE zero-volume row, whose open, high and low are 0.00, is never turned into a bar.

A session without a provider candle is never called a no-trade session. Because listing-start history is unknown, the absence may also predate the contract's listing.

### Acquisition Result

`OptionDailyAcquisitionResult` gains `missing_trading_dates: tuple[date, ...]`:

- an exact tuple of plain dates, strictly ascending and unique, inside the query range;
- every date is a resolved session, and none is a session a stored bar represents;
- derived by the use case from the acquisition, never supplied by a caller;
- empty when every session has a candle.

The invariant is `daily_bar_count + len(missing_trading_dates) == session_count`. A range with no provider candle at all is a successful acquisition: no bar, and every resolved session reported missing.

### Storage

The use case calls the store exactly once with exactly the bars built from returned observations. A range without candles passes an empty batch. Missing sessions are reported in the result and never sent to the store.

A later acquisition of the same range may store a candle the provider exposes afterwards; insert-only storage adds the new bar beside the existing ones. A differing value under an existing key remains a conflict.

### Upstox Acquisition

- `UpstoxOptionNativeDailyMarketDataSource` implements `OptionNativeDailyMarketDataSource` from the persisted Upstox listing of ADR-013. It uses that listing's instrument key and exchange lot exactly, and never consults the live instrument master, parses a trading symbol, or selects a strike or expiry.
- A contract without a persisted listing raises `OptionProviderListingNotStoredError`. That error is outside the Upstox provider error hierarchy: Northstar lacks the mapping, and Upstox was never asked. There is no live-master fallback.
- A persisted listing whose key Upstox rejects (`UDAPI100011`, an expired contract) raises `UpstoxInvalidInstrumentKeyError`, a provider error.
- Premiums are the exact decoded provider Decimals and must be finite and non-negative. Volume is the provider's volume divided exactly by the listing's exchange lot, as a whole number of option contracts; a negative, fractional or indivisible volume fails the fetch.

### Historical-Only Manual Sync

Manual option acquisition is historical-only: it requests only Upstox's historical daily endpoint, reads no clock and never routes a date to the current-day endpoint. A venue date the historical endpoint does not serve yet appears as a session without a provider candle. Acquisition is not finality: storing a candle records what the provider returned, not that it will not be revised.

### Raw Open Interest, Persisted Atomically

- The provider's raw open interest for each returned candle remains Infrastructure-only evidence, as ADR-014 decided. It is never added to `OptionNativeDailyObservation`, `OptionOHLCVBar` or any Core or Application contract.
- The Upstox source captures it as `ProviderOptionOpenInterest` beside the observations. The capture is cleared before every fetch attempt, filled only after the whole response has been converted, and taken at most once.
- A candle whose open interest is missing, null, not a number or non-finite fails the whole fetch, and nothing from that acquisition is stored.
- `SQLiteOptionDailyAcquisitionStore` implements `OptionHistoricalMarketDataStore`. In one SQLite connection and one `BEGIN IMMEDIATE` transaction it creates the option market-data tables when absent, stores the bars and stores exactly one open-interest record per bar. Then it commits, or on any failure rolls everything back, table creation included.
- The capture must correspond exactly to the bars: one record per bar, from the configured provider, for the bar's contract and trading date, and nothing else. A bar's trading date is the IST civil date of its session-close instant. A mismatch is an internal wiring defect, never a data gap. A session without a provider candle has no bar and no open-interest record.

## Architecture Invariants

1. **Observed subset.** Every stored option daily bar comes from a returned candle for a resolved session; a resolved session may lack a candle.
2. **Never fabricated.** No bar, price or volume is ever manufactured for a session without a provider candle.
3. **Honest absence.** A missing candle is reported as a session without a provider candle, not as a no-trade session.
4. **Reported, not hidden.** Every resolved session is either represented by a bar or listed in `missing_trading_dates`.
5. **Unexplained data still fails.** A candle on a date that is not a resolved session, a duplicate, an out-of-range candle or another contract's candle fails the acquisition.
6. **Persisted listing only.** Acquisition requires the exact contract's persisted provider listing and never falls back to the live master.
7. **Historical only, never final.** Manual option acquisition uses only the historical endpoint and claims no finality.
8. **Atomic evidence.** Canonical bars and their raw provider open interest commit together or not at all.
9. **Open interest stays in Infrastructure.** No Core or Application contract carries open interest.
10. **Options only.** Futures coverage, contracts, tables and tests are unchanged.

## Rationale

### Why Not Keep Strict Coverage

Roughly half of all listed NIFTY option contracts do not trade on a given day, and the provider then publishes no candle. Strict coverage would refuse almost every range of an illiquid contract and store none of its traded sessions before the provider stops serving its key at expiry.

### Why Not Fabricate a Zero Bar

The only source for an untraded session is the exchange's zero row, whose open, high and low are 0.00 and whose close is not a traded price. A bar built from it would contradict the high-low invariants or invent prices. Absence is reported, not filled.

### Why Not Call It a No-Trade Session

The provider master does not say when a contract was listed, so a missing candle before the first candle may be pre-listing absence rather than an untraded session. Northstar only knows that the provider returned no candle.

### Why Persist Open Interest Atomically

Without atomicity, a sync could end with canonical bars stored but the open-interest evidence for the same candles missing, and that evidence becomes unrecoverable after expiry. The store already receives exactly the bars of one acquisition, so it can write the matching evidence in the same transaction without exposing open interest to Application.

## Alternatives Considered

- **Keep strict coverage.** Rejected: invalidated by the live evidence.
- **Strict by default with an operator flag.** Rejected: two policies for one fact, and the strict one is known to be wrong for options.
- **Treat a missing candle as a zero-volume bar.** Rejected: fabricates prices.
- **Fail when the whole range has no candle.** Rejected: a listed contract can go a whole range without a trade.
- **Expose open interest through the Application observation.** Rejected: ADR-014 keeps it Infrastructure-only.
- **Store open interest in a second, separate transaction.** Rejected: it can leave bars without their evidence.

## Consequences

### Positive Consequences

- Illiquid, long-dated and newly listed option contracts can be acquired while their keys are still served.
- Every session's outcome is explicit: a stored bar or a reported missing date.
- Canonical bars and raw open interest cannot diverge after a failure.

### Implementation Consequences

- `OptionDailyAcquisitionResult` has a required `missing_trading_dates` field.
- `OptionDailySessionCoverageError` is raised by acquisition only for unexpected candles; its missing-dates attribute is then empty.
- The store is called once on every successful acquisition, with an empty batch when there is no candle.
- `northstar options market-data sync` reports sessions without a provider candle and their dates, and succeeds when there are some.

### Future Consequences

- Research consumers must treat an absent bar as "no provider candle", not as "no trade".
- A later re-acquisition can fill a session whose candle the provider published late.
- Listing-start history, if a source for it is added, could distinguish pre-listing absence from untraded sessions.

## Scope

This ADR changes option daily acquisition coverage and records how Upstox option daily candles and their raw open interest are acquired and persisted.

It does not:

- edit ADR-014 or change any other ADR-014 decision;
- change Futures coverage or any Futures component;
- introduce current-day routing, finality, evidence collection or automated option operations;
- give open interest domain meaning, units or a Core type;
- introduce contract selection, option chains, implied volatility, Greeks or an HTTP or Web surface.

## Future Considerations

Deferred:

- option finality and operator-approved final-through;
- current-day option candles;
- automated option operations;
- historical listing-start reference data;
- open-interest units, revision policy and research semantics.

## Related Documents

- ADR-013: Option Provider Listing Reference and Historical Reproducibility
- ADR-014: Canonical Option Daily Market Data and Session Semantics
- Operations: `operations/Indian-Options-Market-Data.md`
- Ubiquitous Language: `architecture/Ubiquitous-Language.md`
- NSE F&O bhavcopy (UDiFF) for 2026-08-11, 2026-09-18 and 2026-09-25
