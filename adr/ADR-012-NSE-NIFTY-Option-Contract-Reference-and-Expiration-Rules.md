# ADR-012: NSE NIFTY Option Contract Reference and Expiration Rules

## Status

Accepted

## Date

2026-10-08

## Decision Makers

Northstar Architecture Team

## Context

ADR-010 froze Option contract identity as `(product, expiration_date, strike, right)`, with no weekly or monthly classification in it. ADR-011 froze contract-scoped Option economics. Neither says how Northstar knows which date a NIFTY option expiry falls on.

Futures never needed that answer. The operator configures each futures expiration explicitly, and the Futures pre-expiry guard reads it from the contract. Options list many expiries at once -- weekly and monthly -- so selecting and validating option contracts later will need the exchange's expiration rule, applied deterministically to the exchange's trading calendar.

Exchange rules are versioned facts. NSE moved NIFTY expiries from Thursday to Tuesday in 2025, so no weekday may be treated as a timeless truth.

### Primary Source

NSE/FAOP/68747, dated 2025-06-25, "Revision in Expiry Day of Index and Stock Derivatives Contracts - Update", states for NIFTY:

- the weekly expiry day changes from Thursday to Tuesday;
- the monthly, quarterly and half-yearly expiry day changes from the last Thursday to the last Tuesday of the expiry month;
- newly generated contracts with an expiry on or after 2025-09-01 use the revised Tuesday expiry.

### Current Exchange Context

The official NSE NIFTY 50 F&O contract specification currently describes:

- four weekly expiry contracts;
- three consecutive monthly expiry contracts;
- additional quarterly and half-yearly contracts;
- a 50-point strike interval for weekly and monthly contracts, with 35 in-the-money, 1 at-the-money and 35 out-of-the-money strikes.

These are recorded here as context only. They are listing and strike rules, and this ADR does not implement them.

## Decision

Northstar resolves NIFTY@NSE option expiration dates from an effective-dated, cited expiration rule applied to the existing NSE F&O trading-day reference data. Resolution answers which date the rule gives; it never claims that any contract was listed.

### Scope

- NIFTY@NSE only. Every other product and venue fails closed.
- No Core change. `OptionContract`, `OptionProductReference`, `OptionStrike`, `OptionRight` and `OptionContractEconomics` are unchanged.
- Weekly and monthly expiries are resolved by two explicit operations. There is no `OptionExpirySeries` type yet.

### Application

- `OptionExpirationResolver` is an Application port with two operations:
  - `weekly_expiration(product, iso_year, iso_week)`;
  - `monthly_expiration(product, year, month)`.
- Both return `ResolvedOptionExpiration(product, nominal_date, expiration_date, rule_source)`, calendar-derived Application data. `is_adjusted` is derived: it is true exactly when the nominal date differs from the expiration date.
- Every failure raises `OptionExpirationResolutionError`. Invalid coordinates, an unsupported product, a period without a rule epoch, an unloaded calendar year and an encountered special session all fail explicitly.
- There is no wall clock, no default date, no provider and no network.

### Infrastructure

- `nse_option_expiry_reference` holds the expiration rule epochs as immutable, cited rows, validated at import. The MVP encodes exactly one epoch:

      NIFTY@NSE, effective from 2025-09-01
      weekly:  Tuesday of the ISO expiry week
      monthly: last Tuesday of the expiry month
      adjustment: previous expiry-eligible trading day
      source: NSE/FAOP/68747, 2025-06-25

- `NSEOptionExpirationResolver` implements the port. It reuses three day-level NSE F&O facts from the existing calendar reference unchanged: `HOLIDAYS`, `SPECIAL_SESSIONS` and `LOADED_YEARS`. They are constructor-injected, with the existing module as the production default.
- The Futures session hours (`REGIMES`), `NSEFuturesTradingSessionResolver`, `FuturesTradingSessionResolver`, `FuturesTradingSession` and `FuturesExpiryFlattenGuard` are not reused. Option session hours are not verified and an expiration date needs none.

### Resolution

1. Validate the coordinates exactly.
2. Compute the nominal expiry date: the Tuesday of the ISO week, or the last Tuesday of the month.
3. Select the rule epoch in force for the period: the epoch in force on the ISO week's Monday for weekly resolution, and on the month's first day for monthly resolution. A period whose last day falls under a later epoch straddles two epochs and fails closed. A period before the first epoch fails closed.
4. Starting at the nominal date, walk backward one calendar day at a time:
   - a date in an unloaded calendar year fails closed;
   - a date that is a published special session fails closed;
   - a Saturday, a Sunday or a published trading holiday is passed over;
   - the first remaining date is the expiration date.

The walk has a defensive bound of a few weeks. The bound is an implementation safety limit against corrupt reference data, not an NSE rule.

### Nominal Date and Expiration Date

The nominal expiry date is what the rule names before any adjustment. The expiration date is the actual exchange expiration after the holiday adjustment. Both are kept, so an adjusted expiry is visible rather than silently replacing the rule's date.

### Special Sessions Fail Closed

NSE publishes special live sessions on weekends and holidays, such as Muhurat and Union Budget sessions. NSE/FAOP/70320 confirms a special F&O Muhurat session on 2025-10-21, which is also a trading holiday and a nominal NIFTY weekly expiry date. Northstar has no sufficiently explicit primary exchange reference for how an expiry falling on such a date is treated.

Resolution therefore fails closed whenever the walk reaches a special-session date. It neither treats the session as an expiration date nor steps silently through it. The assumption that special sessions are never expiry days is deliberately not encoded as an exchange fact.

A special session the walk never reaches has no effect. The unresolved 2026-11-08 Muhurat session does not affect the resolution of 2026-11-10 to 2026-11-09, because the walk stops on 2026-11-09.

## Architecture Invariants

1. **Rules are dated facts.** Every rule epoch has an effective date and a primary source. No rule applies before its epoch.
2. **Fail closed.** Missing calendar coverage, a missing epoch, an unsupported product and an encountered special session are errors, never guesses.
3. **Nominal versus actual.** The nominal expiry date and the adjusted expiration date are distinct, and both are reported.
4. **Calendar reuse at day level only.** Holidays, special sessions and loaded years are reused unchanged. Futures session hours and Futures resolvers are not.
5. **Resolution is not listing.** Resolving an expiration does not establish strike availability, contract listing, provider instrument existence, lot size or option-chain membership. No `OptionContract` is generated from a rule.
6. **Identity unchanged.** Weekly or monthly classification is not part of `OptionContract` identity. A weekly and a monthly expiry that resolve to one date name one contract.
7. **No Futures genericization.** No Futures type, port, reference module or test changes.

## Rationale

### Why an Application Port with an Infrastructure Adapter

The Futures session calendar is an Application port backed by a cited Infrastructure reference module. Expiration rules are exchange facts of the same kind, with the same provenance and versioning needs, so they follow the same shape.

### Why Two Operations Instead of a Series Type

Contract selection will ask for the weekly or the monthly expiry of a period, and two explicit operations answer that. Whether a dated expiry is the monthly one can be derived by resolving that month. Quarterly and half-yearly contracts expire on the monthly rule's date; they differ in which months are listed, which is a listing-cycle fact. A series type is introduced only when a consumer needs one.

### Why Strikes Are Not Generated

A 50-point grid is necessary but not sufficient for a listed strike. The listed range is centred on a moving at-the-money strike and is extended as the index moves, and long-dated contracts use other intervals. Generating contracts from a grid would invent contracts that never existed. Listing truth comes from observed exchange or provider reference data.

## Alternatives Considered

- **Reuse `NSEFuturesTradingSessionResolver`.** Rejected: it is typed on Futures products, returns futures session hours and treats special sessions as sessions.
- **Treat special sessions as never being expiry days.** Rejected for now: it is not established by a primary exchange source.
- **Encode the historical Thursday regime.** Deferred: it needs its own cited sources and no consumer needs it yet.
- **Introduce `OptionExpirySeries` now.** Rejected: no consumer needs it yet.
- **Generate contracts from the strike scheme.** Rejected: it would create fictional contracts.
- **Move the NSE calendar to a neutral module.** Deferred: it is a refactor of frozen Futures code for naming alone.

## Consequences

### Positive Consequences

- Option expiration dates are derived deterministically from cited, dated rules and the existing NSE calendar.
- Holiday adjustments are explicit and auditable through the nominal date.
- No Core, Futures or API component changes.

### Implementation Consequences

- Extending coverage needs both a loaded calendar year and an applicable rule epoch.
- A nominal date on or walking into a special session cannot be resolved until its expiration treatment is sourced.

### Future Consequences

- Option listings, provider instrument mapping and exchange lot sizes will be persisted as reference data in the market-data milestone. `OptionPointValue` remains the P&L authority; a reference lot size will be checked against it, never substituted for it.
- Contract selection will introduce expiry-series vocabulary if it needs one.

## Scope

This ADR establishes NIFTY@NSE option expiration rules and their resolution.

It does not:

- change Core, `OptionContract` identity or any Futures component;
- implement listing cycles, strike rules, strike generation or option chains;
- persist provider instrument keys, trading symbols, listings or lot sizes;
- validate lot sizes against `OptionPointValue`;
- implement contract selection, days-to-expiry policy, an expiry guard, market data, paper trading or P&L;
- change any API, CLI, Web or deployment component.

## Future Considerations

Deferred:

- the explicit expiration treatment of special sessions, once sourced;
- the historical Thursday regime;
- listing-cycle facts: weekly, monthly, quarterly and half-yearly availability;
- strike scheme facts and listing truth, through provider instrument reference data;
- `OptionExpirySeries`, when contract selection needs it;
- option session hours and an Options pre-expiry guard;
- a neutral NSE F&O calendar module, as a separate approved refactor.

## Related Documents

- ADR-009: Contract-Scoped Futures Economics
- ADR-010: Option Contract Identity
- ADR-011: Contract-Scoped Option Economics
- Ubiquitous Language: `architecture/Ubiquitous-Language.md`
- NSE/FAOP/68747 (2025-06-25): Revision in Expiry Day of Index and Stock Derivatives Contracts - Update
- NSE/FAOP/70320 (2025-09-22): Muhurat trading session, 2025-10-21
