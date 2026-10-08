# ADR-010: Option Contract Identity

## Status

Accepted

## Date

2026-10-08

## Decision Makers

Northstar Architecture Team

## Context

Northstar is beginning an Indian Options track. The reference product is NIFTY Index Options at NSE. The first lifecycle is deliberately narrow:

- exact dated option contracts, CALL and PUT;
- long-premium exposure only: no short-option inventory and no naked writing;
- no multi-leg strategies;
- no implied volatility or Greeks;
- internal deterministic paper execution at a daily decision frequency, with no broker integration;
- every position exits by trading before expiry, so no expiry settlement is modelled.

Futures already has a frozen Core architecture and a running Indian NIFTY Futures paper deployment. `FuturesContract` is identified by `(FuturesProductReference, ExpirationDate)`, P&L economics are contract-scoped (ADR-009), and broker execution is a separate context (ADR-008). The shared `northstar_core.derivatives` package -- `ExpirationDate`, `QuoteValue`, `UnderlyingReference` -- was placed below Futures so that a second derivative family would never need to depend on Futures to reach a shared concept, and the Futures boundary tests already forbid Futures and Derivatives from importing `northstar_core.options`.

An option contract has two identity dimensions a futures contract does not have: the strike and the right. NSE also lists the NIFTY future and the NIFTY option under the same exchange product code, so both display as `NIFTY@NSE` while being different exchange products.

The Options work must not distort or generalize Futures contracts merely to make Options fit. An existing abstraction is reused only where its semantics are genuinely shared.

## Decision

Options are a new Core package, `northstar_core.options`, a sibling of `northstar_core.futures`. Both build on the shared derivatives layer, and neither imports the other.

### Core

- `OptionRight` is a closed `StrEnum` with exactly `CALL` and `PUT`. These are Northstar's canonical values. The NSE and provider spellings `CE` and `PE` are not members; they are mapped to `OptionRight` in Infrastructure.
- `OptionStrike(value: Decimal)` is a fixed contract term expressed in the underlying's quotation convention. A NIFTY strike of 25000 is 25000 index points.
  - Only a `Decimal` is accepted; floats, bools, ints and strings are rejected, so the caller owns conversion from a provider's spelling.
  - The value must be finite and strictly greater than zero.
  - It is stored in the context-independent canonical Decimal form shared by every Core numeric value, so `25000`, `25000.0` and `2.5E+4` are one strike, and construction never depends on the caller's decimal context.
  - It carries no currency, is ordered, and offers no arithmetic.
  - It is not `Money`, `Price` or `QuoteValue`.
- `OptionProductReference(product_code: Symbol, exchange_code: ExchangeCode)` identifies one exchange-defined option product, for example `NIFTY@NSE`. It is a distinct type from `FuturesProductReference`, although the two have the same shape and can display identically.
- `OptionProductSpecification(reference: OptionProductReference, underlying: UnderlyingReference)` records what an option product is written on, for example `NIFTY@NSE on NIFTY50`. Its identity is the reference. Product code and exchange code are derived from the reference and never stored again.
- `OptionContract(product: OptionProductReference, expiration_date: ExpirationDate, strike: OptionStrike, right: OptionRight)` is the identity of one option contract.
  - Its natural key is exactly `(product, expiration_date, strike, right)`, exposed as the derived `natural_key` property. There is no surrogate option-contract identity.
  - Its string form is `NIFTY@NSE 2026-10-27 25000 CALL`.
  - Contracts are not ordered directly. Canonical order is the natural key's order: product (product code, then exchange code), then expiration date, then strike numerically, then right.
- Each value defines its own `Invalid*Error(ValidationError)`, validates at construction, is an immutable `@dataclass(frozen=True, slots=True)`, and has a deterministic `str` and `repr`.
- `northstar_core.options` and `northstar_core.options.value_objects` each export exactly these nine names through `__all__`.

### What Is Not Identity

| Concept | Where it belongs | Status |
| --- | --- | --- |
| Underlying | `OptionProductSpecification` | Implemented |
| Weekly or monthly classification | Nowhere. Weekly and monthly series are contracts of one product with different expiration dates, and the date alone identifies them. | Not modelled |
| Lot size | Contract-scoped economics, folded into one rate as ADR-009 does for Futures. It can differ between expiries of one product. | Deferred |
| Exercise style (European) | Product specification | Deferred until expiry settlement is modelled |
| Settlement method (cash) | Product specification | Deferred until expiry settlement is modelled |
| Provider instrument key, trading symbol, exchange token | Infrastructure | Excluded from Core |

## Architecture Invariants

1. **Sibling, not specialisation.** `northstar_core.options` depends only on `northstar_core.foundation` and `northstar_core.derivatives`. It never imports Futures, paper trading, broker execution, strategy, the Generation-1 aggregates, market-listing identity, a provider, an outer layer, a clock or randomness.
2. **Four-part identity.** An option contract is identified by exactly product, expiration date, strike and right. Two contracts that differ in any one of them are different contracts.
3. **Distinct product type.** An option product is never a `FuturesProductReference`, and an option contract is never a `FuturesContract`, even where their text forms coincide.
4. **Canonical vocabulary.** Rights are `CALL` and `PUT`. Venue and provider spellings never enter Core.
5. **Strike is a term, not money.** A strike is a positive canonical Decimal in the underlying's quotation convention, with no currency and no arithmetic.
6. **Underlying on the specification.** The underlying is never stored on a contract.
7. **No Futures generalization.** No Futures, Derivatives or Foundation type is changed, renamed or genericized for Options.

## Rationale

### Why a Sibling Package

The derivatives package was designed for exactly this: concepts common to every derivative family sit below both families, so Options reuses `ExpirationDate` and `UnderlyingReference` without touching Futures. ADR-008 already chose parallel types over genericizing an existing one, and `FuturesAssetAnalysis` was added beside `AssetAnalysis` for the same reason.

### Why a Distinct Product Reference

NSE uses the code `NIFTY` for both its index future and its index option. A shared reference type would make `NIFTY@NSE` futures and `NIFTY@NSE` options one equal value, so a futures fact could be read as an options fact without any error. Distinct types make that confusion unrepresentable.

### Why the Strike Is a Dedicated Value

A `QuoteValue` is an observed market quotation that changes every bar and may be negative. A strike is set when the contract is listed and is part of its identity. `Price` and `Money` are currency-bound, and a NIFTY strike is index points, not rupees.

### Why the Strike Must Be Positive

Strikes of the reference product are positive, and a zero strike would make a call and a put on one expiry degenerate. Some venues have listed zero and negative strikes on products whose quotations went negative. Admitting them later would be a widening that keeps every strike valid today valid, whereas tightening a permissive rule later would invalidate stored data.

### Why Only Decimal Is Accepted

A strike is a contract term rather than a parsed quotation. Like `FuturesPointValue`, it accepts only a `Decimal` and leaves conversion from provider text or numbers to the caller, so no implicit coercion can change a contract's identity.

### Why Exercise and Settlement Style Are Deferred

NIFTY options are European and cash-settled, and both are product-level facts. The first lifecycle exits every position by trading before expiry, so nothing consumes either fact yet. Modelling them now would fix a shape before its consumer exists, which is the same reason Futures deferred multiplier and tick size.

## Alternatives Considered

- **Reuse `FuturesProductReference` for options.** Rejected: it makes NIFTY futures and NIFTY options one equal value.
- **A generic `DerivativeContract` or `DerivativeProductReference`.** Rejected: it would change the frozen Futures contracts that the paper-trading, broker-execution, Application and persistence layers are built on, for symmetry alone.
- **Store the underlying on the contract.** Rejected: every contract of a product shares it, and copies could disagree with the specification.
- **Model weekly and monthly as a series type.** Rejected: on NSE they are one product with different expiry dates, so a series field would be a second source of truth for what the date already says.
- **Use `CE` and `PE` as the right's values.** Rejected: they are NSE and provider spellings, and would let venue vocabulary become domain identity.
- **Represent the strike as `Money`, `Price` or `QuoteValue`.** Rejected for the reasons above.
- **Include exercise style in identity.** Rejected: one product has one style, so it never distinguishes two contracts.

## Consequences

### Positive Consequences

- Options can be added without changing any frozen Futures type, boundary test or ADR.
- An option contract cannot be confused with a futures contract, or one option product with the futures product of the same name.
- Contract identity is complete for the reference product: every NSE NIFTY option contract maps to exactly one `OptionContract`, and every weekly and monthly expiry is representable.

### Implementation Consequences

- Persistence keys, DTOs and provider resolvers for options will carry five identity columns -- product code, exchange code, expiration date, strike and right -- where Futures carries three.
- Infrastructure owns the mapping from provider records, including `CE` and `PE`, to `OptionContract`.

### Future Consequences

- **Historical option resolution risk.** Provider instrument masters cannot be assumed to resolve expired option contracts indefinitely. The current Upstox master lists only tradable contracts, and the Futures resolver fails for expired ones. Options multiply the number of dated contracts by every listed strike and both rights. The historical reference-data strategy for expired option contracts is deferred to the market-data milestone and must be decided there.
- Option premium, contract-scoped option economics and the option contract count will each need their own decision. They are expected to follow the ADR-009 invariants -- contract identity, exact lookup, fail closed, one rate, count remains contracts -- without changing Futures types.

## Scope

This ADR establishes Option contract identity in Core and freezes its invariants.

It does not:

- introduce option premium, a point value, contract economics, a contract count or profit and loss;
- introduce option market data, an option chain or contract selection;
- introduce paper trading, positions, execution or broker concepts for options;
- introduce exercise, assignment, settlement, implied volatility, Greeks, margin or multi-leg strategies;
- change any Application, Infrastructure, API or Web component;
- change ADR-008, ADR-009 or any Futures, Derivatives or Foundation type.

## Future Considerations

Deferred:

- option premium and contract-scoped option economics;
- an option contract count for paper execution;
- option market data and the historical reference-data strategy for expired contracts;
- the product-level exercise style and settlement method, once expiry settlement is modelled;
- strike interval and tick-size alignment;
- contract selection by expiry, moneyness or days to expiry.

## Related Documents

- ADR-002: Architecture Constitution
- ADR-008: Futures Broker Demo Execution Boundary
- ADR-009: Contract-Scoped Futures Economics
- Ubiquitous Language: `architecture/Ubiquitous-Language.md`
- ES-001: Value Objects
- ES-004: Package Public API
