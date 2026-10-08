# ADR-016: Point-in-Time Option Chain Snapshot

## Status

Accepted

## Date

2026-10-08

## Decision Makers

Northstar Architecture Team

## Context

Option contract selection (INDIA-OPT-6) will need a cross-sectional view of one option expiration at one decision instant: which exact contracts existed, and what the market showed for each. Northstar already holds the two facts such a view is made from, and both are immutable:

- **Listing reference (ADR-013).** For each exact contract, the provider listing that first established it, with `established_at`, the instant Northstar first observed it in the provider's instrument master. Nothing later is recorded: there is no last-seen instant and no per-sync membership history.
- **Canonical daily bars (ADR-014, ADR-015).** `OptionOHLCVBar` values stamped at the resolved option session close, acquired with observed-subset coverage. Acquisition outcomes are reported but not stored, so a stored database cannot tell a session without a provider candle from one never acquired.

A cross-section reconstructed for a past instant can leak hindsight in two ways. A contract may be included because a later instrument master lists it, or because a bar for it was acquired later. Operators choose which contracts to acquire knowing what came after, so a later-acquired bar is evidence of that choice as much as of the market.

## Decision

### Definition

An **Option Chain Snapshot** is one product, one expiration and one resolved option session close (`as_of`), together with every exact contract of that expiration Northstar knew was listed by `as_of`. Each contract carries the canonical daily bar stamped exactly at `as_of` when one is stored.

Listed, observed and selected stay distinct:

- **listed:** the contract is an entry, because its listing was known by `as_of`;
- **observed:** the entry carries a daily bar;
- **selected:** never decided here. OPT-5 implements no selection, ranking, filtering or scoring.

### Core

- `OptionChainEntry(contract, daily_bar)` holds one exact `OptionContract` and its `OptionOHLCVBar`, or `None`. A present bar is for that exact contract.
- `OptionChainSnapshot(product, expiration_date, as_of, entries)` has these invariants:
  - `entries` is a non-empty tuple;
  - every entry has the snapshot's product and expiration;
  - contracts are unique, ordered by strike ascending, then CALL before PUT, an order the snapshot states explicitly;
  - every present bar is the `1d` bar stamped exactly at `as_of`.
- The snapshot carries no trading date or timezone, underlying value, open interest, days to expiry, moneyness, implied volatility, Greeks, bid or ask, provider metadata, listing timestamp, score or selection status.
- `chain` is released from the Options deferred-concept denylist.

### Listing Known by Cutoff

A contract is a member of the chain at `as_of` exactly when all of these hold:

1. its stored provider listing has `established_at <= as_of`, compared as `PointInTime` instants, never as text, because stored instants may carry fractional seconds;
2. its expiration date is on or after the trading date, so a contract's own expiration day has a chain;
3. its product and expiration are the snapshot's.

Consequences of the rule:

- A listing first observed after `as_of` never appears in an earlier chain, including later-added strikes and contracts first seen in a later master.
- **A bar never establishes membership.** A bar stamped exactly at `as_of` for a contract whose listing was first observed after `as_of` is excluded. For example, a listing first established on 2026-10-08 with a backfilled bar for 2026-10-07 does not enter the 2026-10-07 chain.
- No chain exists for a session before Northstar's first applicable listing observation, however many bars are stored for it. Today's or any later master is never applied backward.

### Continuity Assumption

Once Northstar has observed a contract's listing, the contract remains eligible for reconstructed chain membership through its expiration, unless Northstar holds explicit contrary evidence. This is a **Northstar reconstruction assumption** forced by the absence of listing-membership history. It is **not** an NSE guarantee that the contract remained continuously listed. Append-only listing-membership observations, able to represent early withdrawal or delisting, are deferred.

### Exact Daily-Bar Matching

The chain uses only `1d` bars stamped exactly at `as_of`. No earlier, later or latest-available bar ever stands in for a missing one.

An entry's `daily_bar = None` means only that **no canonical daily bar is stored** for that contract at `as_of`. It does not mean that nothing traded, that the provider returned no candle, that acquisition was attempted, or that the contract is illiquid. A chain never uses the term "Session Without a Provider Candle": acquisition attempts are not persisted, so the absence may equally be a candle never acquired or not yet acquired. Persisted acquisition-attempt evidence is deferred.

### Market Time, Not Availability

`as_of` is the resolved session close, the instant a bar stamped at that close describes. A snapshot at `as_of` is a market-time reconstruction. It does **not** claim that Northstar possessed or finalized the bar at `as_of`: historical bars are acquired later. Forward selection and decision work must add an availability or finality gate before a bar is used operationally. OPT-5 implements no such gate.

### Application

- `OptionListedContractQuery(product, expiration_date, known_by)` and `OptionListedContractRepository.listed_contracts(query)` return exact contracts only, unique, in canonical order. They expose no provider, instrument key, lot, snapshot hash or `established_at`.
- `OptionChainDailyBarQuery(product, expiration_date, as_of)` and `OptionChainDailyBarRepository.daily_bars_at(query)` return the expiration's `1d` bars stamped exactly at `as_of`, at most one per contract, in canonical order.
- `BuildOptionChainSnapshotUseCase` runs these steps:
  1. take an `OptionChainSnapshotQuery(product, expiration_date, trading_date)`;
  2. resolve exactly that date's option session and take its close as `as_of`;
  3. read the listings known by `as_of` and the bars at `as_of`;
  4. join them with the listing set on the left;
  5. return the snapshot.
  
  It reads no network or clock and writes nothing.
- Errors:
  - an expiration before the trading date is an invalid query;
  - a weekend or holiday is `OptionChainSessionNotFoundError`;
  - an empty known listing set is `OptionChainListingNotKnownError`;
  - a session-reference failure propagates as `OptionTradingSessionResolutionError`;
  - a port contract violation is `OptionChainContractViolationError`.

### Infrastructure

- `SQLiteOptionListedContractRepository(path, provider)` reads the stored provider listings of one product and expiration, filters them by `established_at` with `PointInTime` comparison, and returns contracts in canonical order. The current composition uses provider `upstox`.
- `SQLiteOptionChainDailyBarRepository(path)` reads one expiration's `1d` bars at one exact instant in a single query. Exact equality on stored canonical instant text is safe for equality. No ordering by instant text is done.
- Both repositories are read-only: SQLite read-only mode, no DDL, no lock, no network, no instrument master, no clock. A missing file or table holds nothing. Rows are decoded by the existing canonical decoders, so corruption fails loudly.

### Build on Demand

There is no chain table and no write path. A snapshot is built from the immutable listing reference and the immutable canonical bars whenever it is requested. The `OptionChainSnapshot` value is immutable, but **the projection is not a frozen research artifact**:

- a rebuild of the chain at T always has the same members, because membership is protected by `established_at <= T`;
- an entry may change from `daily_bar = None` to the bar at T if that bar is acquired later.

OPT-6 will freeze the selection and decision evidence a forward decision actually used.

### Command

`northstar options chain show --database --product NIFTY --exchange NSE --expiration --trading-date` renders one chain, one strike per row, CALL and PUT cells showing the close and volume, `no daily bar`, or `no known listing`.

- It is read-only: no token, no network, no clock, no lock, no file or schema creation.
- Exit codes:
  - 2 for invalid input, an expired expiration, or a weekend or holiday;
  - 3 for an unusable database path;
  - 4 for no listing known by the close, or a date the option session reference cannot resolve;
  - 5 for corrupt stored listings or bars;
  - 1 for an internal contract violation.
- There is no provider exit, because no provider is contacted.

## Architecture Invariants

1. **Listed is not observed is not selected.** Membership comes from listing knowledge; observation is an optional bar; nothing selects.
2. **Knowledge by the cutoff.** A contract is a member only if Northstar first observed its listing at or before `as_of`, compared as instants.
3. **No hindsight membership.** A stored bar never makes a contract a member.
4. **Exact instant.** Only the `1d` bar stamped exactly at `as_of` is attached; nothing is substituted.
5. **Honest absence.** `daily_bar = None` claims only that no canonical daily bar is stored.
6. **One expiration.** A snapshot covers one product and one expiration.
7. **Market time only.** `as_of` claims nothing about when Northstar possessed or finalized a bar.
8. **Read-only projection.** Building a chain writes nothing and creates nothing.
9. **No Futures change.** No Futures type, table, test or command changes.

## Rationale

### Why Listing Knowledge, Not Bars, Decides Membership

The listing reference is the only point-in-time evidence of what Northstar could have known at `as_of`. Which contracts have bars depends on later acquisition choices. Admitting them would let hindsight select the chain.

### Why One Expiration Per Snapshot

Strike comparison is cross-sectional within one expiry. Choosing the expiry is a separate OPT-6 decision, and keeping the two apart keeps each one explicit.

### Why Build on Demand

Both inputs are already immutable and insert-only. A chain table would duplicate them, and freezing a chain that no decision has used would add nothing to reproducibility. Frozen evidence belongs with the decision that uses it.

## Alternatives Considered

- **Let stored bars establish membership.** Rejected: hindsight leakage.
- **Reconstruct membership from the current instrument master.** Rejected: applies today's listing backward and invents later strikes.
- **Multi-expiry snapshot.** Rejected for now: it merges expiry choice into the chain.
- **Omit contracts without a bar.** Rejected: it conflates listed with observed.
- **Substitute the latest earlier bar.** Rejected: not point-in-time at `as_of`.
- **Persist snapshots.** Deferred to the decisions that use them.
- **Add `OptionExpirySeries`.** Rejected: no consumer. Monthly expiries remain derivable from ADR-012's resolver.
- **Expose open interest.** Rejected: provider open interest stays Infrastructure-only evidence with unresolved units (ADR-014).
- **Store an underlying value per snapshot or entry.** Rejected: OPT-6 combines the chain with a separate underlying observation at the same instant.

## Consequences

### Positive Consequences

- OPT-6 receives a deterministic, provider-neutral cross-section with listing and observation kept apart.
- Historical chains cannot leak later listings or hindsight acquisition choices.

### Implementation Consequences

- Chains exist only for sessions after Northstar's first applicable instruments sync. Instruments must be synced before the session close a chain is wanted for.
- The listing set is Northstar's knowledge, not a complete exchange listing: contracts listed after the last sync before `as_of` are absent.

### Future Consequences

- **Underlying value.** Northstar has no canonical NIFTY 50 index market-data path. NIFTY futures are not substituted silently: the futures-index basis makes them a different market reference. OPT-6 must resolve the underlying observation.
- **Availability and finality.** A forward decision must gate bars on availability or finality.
- **Further evidence.** Listing-membership history and persisted acquisition attempts could later sharpen absence semantics.

## Scope

This ADR establishes the point-in-time option chain snapshot.

It does not:

- select an expiry, strike, right or contract, or implement ATM, ITM or OTM classification, DTE policy or liquidity filters;
- acquire underlying index data;
- add option finality, availability gates or decision records;
- give open interest domain meaning;
- add implied volatility, Greeks or bid and ask;
- add an HTTP or Web surface;
- modify ADR-013, ADR-014 or ADR-015, or any Futures component.

## Future Considerations

Deferred:

- contract selection (INDIA-OPT-6);
- the underlying observation for moneyness;
- availability and finality gates for forward use;
- append-only listing-membership observations;
- persisted acquisition-attempt evidence;
- multi-expiry views, if a consumer needs one.

## Related Documents

- ADR-010: Option Contract Identity
- ADR-012: NSE NIFTY Option Contract Reference and Expiration Rules
- ADR-013: Option Provider Listing Reference and Historical Reproducibility
- ADR-014: Canonical Option Daily Market Data and Session Semantics
- ADR-015: Observed-Subset Option Daily Acquisition
- Operations: `operations/Indian-Options-Market-Data.md`
- Ubiquitous Language: `architecture/Ubiquitous-Language.md`
