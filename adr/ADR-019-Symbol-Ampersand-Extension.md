# ADR-019: Symbol Ampersand Extension

## Status

Accepted

The product owner approved owner decision EI0-D4, Symbol ampersand support, on 2026-11-01. This ADR records that decision. It satisfies the [Foundation Freeze](../architecture/Foundation-Freeze.md) requirement that a Foundation change be backed by an approved Architecture Decision Record. It records no separate Architecture Review sign-off.

## Date

2026-11-01

## Decision Makers

- Product owner: approved EI0-D4 on 2026-11-01.
- Northstar Architecture Team.

## Context

`Symbol` is the Foundation identity value object for a canonical market symbol, and the approved reference implementation RVO-001 v1.0 in the [Reference Implementation Registry](../engineering/registry/Reference-Implementations.md).

Its validation pattern is:

```text
^[A-Z0-9._-]{1,32}$
```

Equity Intraday V1 targets NSE cash equities ([Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md), section 5.2). Some NSE equity trading symbols contain an ampersand. The product owner identified these examples:

- `M&M`
- `M&MFIN`
- `J&KBANK`

Northstar cannot represent these symbols today. Each is rejected with `Symbol contains invalid characters`, so no listing reference, historical bar, research decision or recommendation can carry them.

### Evidence

- **Official source.** The official source for NSE equity trading symbols is NSE's list of securities available for trading in the equity segment, published as `EQUITY_L.csv`.
- **Not verified here.** That list was **not** retrieved or verified while this ADR was drafted. The examples above were identified by the product owner.
- **Why that is enough for this decision.** The decision does not depend on any particular symbol being listed. It only needs NSE symbols with `&` to exist, which the product owner has approved as a premise.
- **Where listing evidence belongs.** Whether a particular symbol is listed, and in which series, is listing-eligibility evidence. It belongs to universe and reference work (EI-1C and later), not to `Symbol`.

## Decision

Extend the `Symbol` character set with the ampersand.

```text
Before:  ^[A-Z0-9._-]{1,32}$
After:   ^[A-Z0-9._&-]{1,32}$
```

This is the only change.

- **Unchanged Foundation invariants:**
  - A `Symbol` is always valid. `None`, non-string and empty input are refused with `InvalidSymbolError`.
  - Leading and trailing whitespace is trimmed, and the value is normalized to uppercase.
  - Internal whitespace is refused.
  - The value is 1 to 32 characters.
  - `Symbol` stays a frozen, slotted, ordered dataclass. Equality, hashing and ordering are by the canonical value.
  - `str()` returns the canonical value, and `repr()` keeps its format.
  - The constructor signature, exception types and messages, and public exports are unchanged.
- **No exchange rules inside `Symbol`.** `Symbol` stays exchange-neutral. It does not check that a symbol is listed on NSE, belongs to a series, or is eligible for trading. Those remain separate concerns of listing and universe reference data.

## Compatibility

- **Every previously valid symbol is still valid, with the same canonical value.**
  - The new character class is the old one plus `&`. For any input without `&`, the old and new patterns give the same result.
  - Normalization, length and whitespace rules are unchanged.
  - Existing symbols therefore keep their equality, hash and sort position.
- **Only new values are affected.**
  - No stored value can contain `&` before this change, so nothing already stored changes identity or order.
  - In the canonical value, `&` sorts before digits and letters, which affects only new symbols that contain it.
  - A lone `&` becomes a valid symbol, consistent with the existing acceptance of a lone `.`, `-` or `_`.
- **`ListingReference`** is unchanged. It composes `Symbol` and `ExchangeCode`, and renders as `M&M@NSE`.
  - `&` does not collide with the `@` separator, and nothing parses that text form back.
  - `M&M@NSE` and `M&M@BSE` remain distinct listing references.

## Relationship to UnderlyingReference

`UnderlyingReference` (`^[A-Z0-9_]{1,32}$`) is a separate derivatives value object for the underlying of a futures or option product. It is **not** changed.

- Its existing rejection of values such as `S&P500` stays in force.
- Any future change to it needs its own decision and evidence.

## Provider Identity Mapping

Provider instrument identifiers stay outside `Symbol`.

- An adapter maps a provider's identifier to a `Symbol` explicitly, and fails closed when the mapping is absent or ambiguous.
- No adapter may silently normalize a provider symbol, for example `M_M`, `M%26M` or `MANDM`, into a canonical `Symbol`, or the reverse.
- Percent-encoding for HTTP transport, as the Yahoo adapters already do with `urllib.parse.quote`, is transport encoding. It is not symbol normalization: `M&M` is sent as `M%26M` and remains `M&M` in Northstar.

## Persistence and Serialization

- **No database migration is required.**
  - Symbols are stored as `TEXT` through bound parameters, and rebuilt through `Symbol(...)` on read.
  - No schema constraint restricts their characters.
- **No serialization format changes.**
  - JSON carries `&` unchanged.
  - The Futures candle identity format uses `name=value` lines and refuses only `=`.
  - No Northstar identifier joins or splits text on `&`.

## Regression Testing Requirements

- **The `Symbol` reference suite must show that:**
  - `M&M`, `M&MFIN`, `J&KBANK` and `M&M.NS` are accepted;
  - lowercase input and leading or trailing whitespace are normalized;
  - an `&` inside a 32-character symbol is accepted, and 33 characters is still refused;
  - internal whitespace, empty input and non-string input are still refused;
  - other punctuation, such as `!`, `/` and `#`, is still refused;
  - `M&M` and `MM` are different values;
  - the string representation keeps `&`;
  - previously valid symbols keep their canonical value and equality.
- **The `ListingReference` suite must show that** `M&M@NSE` and `M&M@BSE` are distinct, and that `str()` keeps `&`.
- **These existing suites must pass unchanged:** `UnderlyingReference` (including its `S&P500` rejection), Futures, Options, the package-boundary tests, and the full Core suite.
- **Acceptance follow-up.** The S-1A synthetic NSE equity acceptance suite in northstar-infrastructure should add a synthetic `M&M@NSE` series through ingestion, SQLite and replay. This is EI-1B acceptance work outside Core.

## Consequences

### Positive Consequences

- NSE equity symbols containing `&` can be represented without a new value object type.
- Existing symbols, persisted data and integrations are unaffected.

### Negative Consequences

- `Symbol` accepts one more character for every asset class, not only NSE equities. Exchange-specific symbol rules must keep living in listing and reference data.
- The legacy web watchlist has its own copy of the old pattern, and keeps rejecting `&` until it is aligned separately.

## Reference Implementation Versioning

- **When the Core change lands,** RVO-001 becomes **v1.1** in the Reference Implementation Registry, with this ADR as its basis.
- **Until then,** the registry continues to state v1.0.

## Scope

In scope:

- the `Symbol` validation pattern;
- its documentation examples;
- its reference tests.

Out of scope:

- `UnderlyingReference`, `ExchangeCode` and every other value object;
- listing eligibility and series rules;
- the NSE cash-equity calendar (EI-1C);
- intraday ingestion (EI-2);
- provider adapters;
- web and API changes.

## Related Documents

- [Foundation Freeze](../architecture/Foundation-Freeze.md)
- [Reference Implementation Registry](../engineering/registry/Reference-Implementations.md) (RVO-001)
- [ES-001 Value Objects](../engineering/standards/ES-001-Value-Objects.md)
- [ES-002 Reference Implementations](../engineering/standards/ES-002-Reference-Implementations.md)
- [ADR-007 PointInTime Foundation Extension](ADR-007-PointInTime-Foundation-Extension.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md) (EI0-D4)
