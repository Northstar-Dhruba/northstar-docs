# ADR-013: Option Provider Listing Reference and Historical Reproducibility

## Status

Accepted

## Date

2026-10-08

## Decision Makers

Northstar Architecture Team

## Context

An `OptionContract` is exchange identity: product, expiration date, strike and right (ADR-010). To acquire market data for one, Northstar needs the provider instrument that represents it. For Upstox that is an instrument key found in the public JSON instrument master.

The master lists only currently tradable instruments. An option contract disappears from it once it expires, and weekly NIFTY options expire every week, so the master is not historical truth. The Futures path resolves each contract against the current master on every run, which is sufficient for one monthly future at a time but cannot explain, after expiry, which instrument an option contract was mapped to.

The live public NSE master was inspected before implementation. NIFTY index options appear with `segment = NSE_FO`, `exchange = NSE`, `underlying_symbol = NIFTY` and `instrument_type = CE` or `PE`; `expiry` is an integer epoch in milliseconds for 23:59:59 IST on the expiry date, `strike_price` decodes to a Decimal, `instrument_key` carries the `NSE_FO|` prefix and `lot_size` is an integer. There is no separate option-type field and no `OPTIDX` value. Listed expiries reach several years ahead, beyond Northstar's loaded NSE calendar.

Upstox also rejects the instrument key of an expired contract on its standard historical-candle endpoint (`UDAPI100011`, already observed for futures). Knowing a key is therefore not the same as being able to fetch data with it.

## Decision

Northstar persists, while each contract is still listed, an immutable record of which provider instrument represented it, which lot the provider reported, and which exact master snapshot established that mapping.

### Identity

- `OptionContract` remains the exchange and domain identity. Provider metadata never enters Core.
- A provider mapping is identified by `(provider, OptionContract)`. The provider is part of the key, so another provider can map the same contract independently.
- Within one provider, one contract has exactly one instrument key, and one instrument key names exactly one contract.

### Upstox Master Reading

- A record is a NIFTY@NSE option candidate exactly when `segment = NSE_FO`, `exchange = NSE`, `underlying_symbol = NIFTY` and `instrument_type` is `CE` or `PE`. Every other record is skipped. `CE` maps to `OptionRight.CALL` and `PE` to `OptionRight.PUT`.
- The expiry is converted to its IST civil date, the decoded strike becomes an `OptionStrike`, the key must carry the `NSE_FO|` prefix and the lot must be a positive integer.
- A candidate that cannot be converted fails the whole snapshot. Identical duplicate records collapse. One contract with two keys or two lots, or one key for two contracts, fails the whole snapshot. There is no nearest-strike or nearest-expiry fallback, and no identity is parsed from a trading symbol.
- No expiration rule or calendar is consulted. An observed listing is valid even when its expiry lies outside Northstar's loaded calendar. Listing truth comes only from observed provider listings, never from generated strike or listing rules.
- `trading_symbol`, `exchange_token`, `tick_size`, the provider's `weekly` flag and any expiry-series classification are not persisted.

### Snapshot Provenance

- Each downloaded master is identified by the lowercase hexadecimal SHA-256 of the exact body bytes the fetch layer returned, before decompression or decoding.
- A snapshot row records the provider, the hash, the source URL, the first instant Northstar persisted that body, the number of records in the master and the number of NIFTY option candidate records before duplicates collapse.
- The fetch instant comes from a clock injected at the API composition root. It is read exactly once per sync, after the master has been received. Infrastructure reads no clock.
- The complete raw master is not stored. Archiving raw masters is deferred to a separate operational decision.

### Persistence

Two insert-only SQLite tables, in the same Northstar database file:

    option_listing_snapshots (
        provider, snapshot_sha256, source_url, first_fetched_at,
        record_count, option_record_count,
        PRIMARY KEY (provider, snapshot_sha256)
    )

    option_provider_listings (
        provider, product_code, exchange_code, expiration_date, strike, option_right,
        instrument_key, exchange_lot_size, established_snapshot_sha256, established_at,
        PRIMARY KEY (provider, product_code, exchange_code, expiration_date, strike,
                     option_right),
        UNIQUE (provider, instrument_key)
    )

- Every column is canonical TEXT, and every read re-serializes the decoded values and requires them to reproduce the stored text exactly.
- A snapshot whose body was already stored is idempotent: its first fetch instant is kept, and only the counts, which the body determines, must agree.
- A listing seen again with the same key and lot is a no-op. `established_snapshot_sha256` and `established_at` keep the observation that first established it and are never rewritten. There is no mutable last-seen field.
- A stored contract reported with a different key or lot, or a stored key reported for a different contract, is a conflict.
- One sync stores its snapshot and its whole listing batch in a single `BEGIN IMMEDIATE` transaction. Any conflict or corrupt row rolls back the entire sync.
- The tables are created by their own initializer, only from `options instruments sync` under the database operations lock, immediately before the first write. They are not part of the Futures database initializer.
- The repository is read-only. It opens the database in SQLite read-only mode, never creates a file or schema, and answers `None` when the database, the tables or the listing are absent.

### Operator Commands

- `northstar options instruments sync --database --product NIFTY --exchange NSE` downloads the public master without a token and persists the snapshot and every current NIFTY option listing.
- `northstar options instruments show` with the exact contract displays the stored mapping and its provenance. It takes no lock, reads no clock, contacts no provider and creates nothing.

## Architecture Invariants

1. **Exchange identity unchanged.** `OptionContract` is unchanged and carries no provider metadata.
2. **Provider-scoped mapping.** A mapping is identified by provider and exact contract; one key names one contract.
3. **Observed, not generated.** Listings come only from provider snapshots, never from strike or calendar rules.
4. **Fail closed.** Malformed or ambiguous provider data, and any contradiction of a stored mapping, fail the whole sync.
5. **Insert-only provenance.** Snapshots and mappings are immutable, and the observation that first established a mapping is never rewritten.
6. **Exact source identity.** Every mapping is traceable to the SHA-256 of the exact master body that established it.
7. **Reference is not data availability.** A persisted key preserves reference reproducibility; it does not guarantee that the provider will serve candles for an expired contract.
8. **No Futures change.** No Futures type, table, initializer, resolver or command changes.

## Rationale

### Why Capture While Listed

Once a contract leaves the master, nothing in the current master can say which key it had. The only way to explain an expired contract's mapping later is to have recorded it while it was listed, so syncing regularly, at least before each weekly expiry, is an operational requirement.

### Why a Hash Rather Than the Raw Master

The national master is several megabytes and is downloaded repeatedly. Its hash identifies the exact source of every mapping without storing bulk data nothing yet reads, and raw archiving can be added later without changing the tables.

### Why No Last-Seen Field

A last-seen instant would be rewritten on every sync, which would make the listing table mutable evidence. The first observation is a fact that never changes; a later presence history, if a consumer ever needs one, can be an append-only table of its own.

### Why the Lot Stays Separate from Economics

The lot Upstox reports is exchange reference evidence attributed to a provider. `OptionPointValue` is Northstar's authoritative P&L rate (ADR-011). No operation in this milestone consumes both, so neither is checked against the other here, and neither ever overwrites the other.

### Why Ingestion Ignores the Expiration Rule

Long-dated quarterly and half-yearly listings expire beyond Northstar's loaded calendar, where the expiration resolver deliberately fails closed. An observed listing is a fact about the provider's master and does not need the rule to be recorded.

## Alternatives Considered

- **Resolve against the current master on every use, as Futures does.** Rejected: expired contracts would become unexplainable.
- **Store the whole raw master in SQLite.** Rejected: bulk storage with no consumer.
- **Mutable first-seen and last-seen columns.** Rejected: mutable evidence.
- **Effective-dated mapping rows.** Rejected: one contract's mapping does not change during its life; a change is a conflict to investigate.
- **Validate listings against the expiration rule.** Rejected: long-dated listings lie outside the loaded calendar.
- **Accept `OPTIDX` or another option-type field as well.** Rejected: the live master does not use it, and an unverified second schema would be a guess.

## Consequences

### Positive Consequences

- Every observed NIFTY option contract's provider mapping, lot and source snapshot remain explainable after expiry.
- Conflicting provider metadata is caught rather than silently accepted.
- Futures behaviour, tables and commands are unchanged.

### Implementation Consequences

- Operators run `options instruments sync` regularly while contracts are listed.
- The deployed database gains the two listing tables on the first sync.

### Future Consequences

- Option market-data acquisition will resolve instrument keys from the persisted listings, not from the live master, and must acquire candles while each contract is live.
- A later milestone that consumes both the reference lot and `OptionPointValue` will fail closed when they disagree.

## Scope

This ADR establishes the option provider-listing reference.

It does not:

- acquire option candles, open interest or any market data;
- introduce option session hours, current-day routing or candle finality;
- validate the reference lot against `OptionPointValue`;
- introduce an `OptionProductSpecification` catalog or `OptionExpirySeries`;
- archive raw masters;
- change Core, Application, ADR-009, ADR-010, ADR-011, ADR-012 or any Futures component.

## Future Considerations

Deferred:

- option daily bars, option session hours and open interest;
- Upstox option candle acquisition and its current-day routing;
- raw-master archiving;
- an append-only listing-presence history, if a consumer needs one;
- cross-validation of the reference lot against `OptionPointValue`;
- the provider's separate expired-instrument data services.

## Related Documents

- ADR-010: Option Contract Identity
- ADR-011: Contract-Scoped Option Economics
- ADR-012: NSE NIFTY Option Contract Reference and Expiration Rules
