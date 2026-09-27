# ADR-009: Contract-Scoped Futures Economics

## Status

Accepted

## Date

2026-09-27

## Decision Makers

Northstar Architecture Team

## Context

Futures profit and loss converts a quote-point movement into settlement currency through one rate, the `FuturesPointValue`:

    settlement-currency amount per 1.0 quote-point move per one Futures contract

The original ES reference MVP attached that rate to the exchange-defined product. `FuturesProductEconomics` held one point value per `FuturesProductReference`, the SQLite table `futures_product_economics` was keyed by `(product_code, exchange_code)`, and P&L looked economics up by `contract.product`. Every expiry of a product resolved to the same economics. For ES@CME at 50 USD per point that was sufficient, because CME does not vary the ES contract size between expiries.

Northstar is now Indian-Futures-first, with NIFTY@NSE as the reference product. NSE sizes a contract as a lot of underlying units and revises lot sizes periodically. When a revision raises the lot, the new lot applies only to newly listed contracts, while contracts already listed keep their original lot until they expire. Concurrently listed expiries of one product can therefore carry different lots. NSE bhavcopy data shows exactly this:

| Contract | Market lot |
| --- | --- |
| NIFTY November 2025 | 75 |
| NIFTY December 2025 | 75 |
| NIFTY January 2026 | 65 |

The product `NIFTY@NSE` therefore does not determine the point value. Under the product-keyed model, one stored row would be applied to every expiry, and a contract with a different lot would be valued at the wrong point value without any error: the store's conflict check only fires when a different value is written, never when a stored value is read for the wrong contract.

## Decision

P&L-authoritative Futures economics are keyed by the complete `FuturesContract`: product code, exchange code and expiration date.

### Core

- `FuturesContractEconomics(contract: FuturesContract, point_value: FuturesPointValue)` is added to `northstar_core.futures`. The contract is its identity. Product code, exchange code and expiration are read from the contract and are never stored beside it.
- `FuturesPointValue` is unchanged in type and meaning. For a lot-sized contract, the point value is the contract's lot multiplied by the quote multiplier. A NIFTY lot of 65 at 1 INR per index point per unit is `FuturesPointValue(65, INR)` for that contract.
- No separate `ContractLotSize`, `ContractMultiplier`, tick-size or margin concept is introduced. The point value remains the single P&L economics value.
- `FuturesProductEconomics` is retained unchanged for historical and reference compatibility. It is no longer the P&L authority, and nothing reads it in place of, or as a fallback for, contract economics.

### Application

- `FuturesContractEconomicsRepository.get_economics(contract: FuturesContract) -> FuturesContractEconomics | None` and `FuturesContractEconomicsStore.store(economics: tuple[FuturesContractEconomics, ...]) -> int` are the P&L-authoritative ports.
- Realized P&L, unrealized P&L, valuation and the paper-trading snapshot resolve economics by the exact contract. The P&L formulas, Decimal context, portfolio average basis, Money behaviour, mark selection and fill semantics are unchanged; only the lookup key changes.
- The product-level ports remain for the historical table only. No P&L use case accepts them.
- Missing economics identify the complete dated contract. `FuturesContractEconomicsNotFoundError.contract` carries it, and the snapshot's `missing_economics` is a `FuturesContract`. When several contracts are missing, the snapshot keeps its existing behaviour of reporting the first.

### Infrastructure

- A new insert-only SQLite table:

      futures_contract_economics (
          product_code TEXT NOT NULL,
          exchange_code TEXT NOT NULL,
          expiration_date TEXT NOT NULL,
          point_value_amount TEXT NOT NULL,
          settlement_currency TEXT NOT NULL,
          PRIMARY KEY (product_code, exchange_code, expiration_date)
      )

- For one contract, storing equal economics is an idempotent success, and storing a different amount or currency is a conflict that leaves the stored row unchanged. Different expiries of one product are different keys and coexist.
- Lookup is exact on the contract. There is no fallback to another expiration, to the latest expiration, to the same product or to product economics.
- `futures_product_economics` is not altered, migrated, dropped or read by the P&L path. Its rows are never interpreted as contract economics.

### API

- `northstar economics set` and `northstar economics show` require `--expiration YYYY-MM-DD`. No current expiry is inferred, and there is no product-level fallback.
- The dashboard P&L section reports the missing dated contract as `missing_contract`, using the existing contract schema (`product`, `exchange`, `expiration`), in place of the product string `missing_product`.
- A newly initialized database creates `futures_contract_economics` and no longer creates `futures_product_economics`. An existing database keeps its product table exactly as it was.

## Architecture Invariants

1. **Contract identity.** P&L economics are identified by the complete `FuturesContract`, never by `FuturesProductReference` alone.
2. **Exact lookup.** An economics lookup returns the economics of exactly the requested contract or nothing. One expiry's economics never value another expiry, product or exchange.
3. **Fail closed.** A visible fill or open position without its own contract economics makes the valuation unavailable, naming that contract. P&L is never valued at a borrowed or default point value.
4. **One rate.** `FuturesPointValue` remains the only P&L economics value. Lot size and quote multiplier are folded into it, not modelled beside it.
5. **Count remains contracts.** `FuturesContractCount` remains the number of Futures contracts. An exchange lot, the units one contract represents, is never modelled as a quantity of contracts.
6. **Insert-only.** Stored contract economics are immutable reference facts.
7. **Historical data untouched.** `futures_product_economics` and `FuturesProductEconomics` are preserved and never silently reinterpreted.
8. **No currency totals.** Valuations remain per contract. Currencies are never summed or converted.

## Rationale

### Why the Contract, Not an Effective Date

NSE applies a raised lot to newly listed contracts only, so the lot is observed to be fixed for a contract's life. Keying by contract therefore represents every observed NIFTY lot history exactly, with no time dimension and no resolution rule. An effective-dated product row would need a rule that maps each fill and mark instant to a date range, and would still be wrong whenever two concurrently listed expiries carry different lots.

### Why Not Lot Size and Multiplier Objects

P&L consumes one rate. Adding a lot size and a multiplier beside the point value would create a second source of truth that could disagree with it, and nothing consumes them separately yet.

### Why a New Table

A new table leaves the released CME history readable by the frozen `main` build, needs no destructive migration, and cannot be confused with product-level rows.

### Why This Is Market-Neutral

The change removes an assumption rather than adding an Indian concept. ES@CME is represented exactly as before, with one row per traded expiry, and NIFTY@NSE is represented correctly.

## Alternatives Considered

- **Keep product economics and store one row per product.** Rejected: it values any contract whose lot differs from the stored row at the wrong point value, with no error.
- **Effective-dated product economics.** Rejected: it cannot represent concurrently listed expiries with different lots, and needs an instant-resolution rule.
- **Contract economics with a product-level fallback.** Rejected: a fallback reintroduces silent borrowing of another expiry's point value, which is exactly the defect being fixed.
- **Evolve the existing product ports to accept a contract.** Rejected: a product-named API with contract semantics is misleading, and the historical product data would lose its reader.
- **Model lot size as quantity.** Rejected: it would conflate the number of Futures contracts with the units one contract represents, and change released paper-trading semantics.

## Consequences

### Positive Consequences

- Multi-expiry portfolios value correctly. Two expiries of one product with different point values coexist in one portfolio and one valuation.
- Missing economics identify the exact dated contract, so an unconfigured expiry is never mistaken for an unconfigured product.
- ES@CME and NIFTY@NSE economics coexist in one schema without collision.

### Implementation Consequences

- Operators record economics once per traded dated contract. Each monthly NIFTY contract needs its own `economics set --expiration` before its first fill is valued.
- The dashboard P&L response changes shape: `missing_product` (a string) is replaced by `missing_contract` (a contract object). The web client needs a matching compatibility change.
- Existing ES acceptance setups configure economics for the ES December 2026 contract explicitly. ES economics themselves are unchanged.
- The released production ES database is not migrated. The frozen `main` build keeps reading its product table, and a later Indian deployment starts from a fresh database.

### Future Consequences

- Provider adapters will validate the provider contract master's lot size and multiplier against the stored contract economics, and fail on disagreement rather than correcting it.

## Scope

This ADR establishes contract-scoped Futures economics and freezes their invariants.

It does not:

- introduce an NSE session calendar, a market-data provider or provider selection,
- introduce rollover, expiry flattening or settlement,
- change `FuturesPointValue`, `FuturesContractCount`, the paper target policy, the next-session OPEN fill rule or any paper-trading value,
- change broker-execution contracts,
- or migrate any existing database.

## Future Considerations

Deferred:

- effective-dated economics within one contract, for a lot change after listing;
- deriving economics automatically from a provider contract master;
- margin;
- tick-size alignment;
- fees and commissions;
- FX conversion and cross-currency totals.

## Related Documents

- ADR-002: Architecture Constitution
- ADR-004: Composed Foundation Value Objects
- ADR-008: Futures Broker Demo Execution Boundary
