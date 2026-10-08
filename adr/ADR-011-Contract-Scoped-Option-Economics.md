# ADR-011: Contract-Scoped Option Economics

## Status

Accepted

## Date

2026-10-08

## Decision Makers

Northstar Architecture Team

## Context

ADR-010 froze Option contract identity: an `OptionContract` is exactly `(product, expiration_date, strike, right)`, Options are a sibling of Futures, and neither family depends on the other.

The first Options lifecycle holds long-premium positions only and exits them by trading before expiry. Its profit and loss will convert a movement in option premium into settlement currency:

    (exit premium - entry premium) * point value * contracts

For NIFTY options at NSE, one contract is one lot of the index, and the premium is quoted in index points worth 1 INR per unit. The value of a 1.0 premium move on one contract is therefore the contract's lot size in INR.

NSE revises lot sizes periodically, and a revision applies only to newly listed expiries. ADR-009 records the consequence for NIFTY futures, where November and December 2025 traded at a lot of 75 while January 2026 traded at 65. NIFTY options are sized the same way. Concurrently listed option expiries of one product can therefore carry different lots, so neither the product nor the product and strike alone determine the economics of a contract.

ADR-009 already solved the same problem for Futures with contract-scoped economics. It is the precedent for this decision. Options must follow its invariants without changing any Futures type, port, table or command.

## Decision

P&L-authoritative Option economics are keyed by the complete `OptionContract`: product code, exchange code, expiration date, strike and right.

### Core

- `OptionPremium(value: Decimal)` is an observed option quotation in premium points.
  - Only a `Decimal` is accepted; floats, bools, ints, strings and other value objects are rejected.
  - The value must be finite and zero or greater. Zero is valid. `-0` and `-0.00` canonicalise to `0`.
  - It is stored in the context-independent canonical Decimal form, is ordered, carries no currency and offers no arithmetic.
  - It is a distinct type from `OptionStrike`, `QuoteValue`, `Price` and `Money`.
- `OptionPointValue(amount: Decimal, currency: Currency)` is the settlement-currency value of a 1.0 premium-point move on one option contract.
  - The amount is a canonical `Decimal` strictly greater than zero, and the currency is required.
  - It is not ordered, carries no arithmetic, and its string form is `65 INR/premium-point/contract`.
  - For a NIFTY lot of 65 at 1 INR per index point per unit, the point value is `OptionPointValue(65, INR)`.
- `OptionContractEconomics(contract: OptionContract, point_value: OptionPointValue)` is the economics of one option contract.
  - The contract is its identity. Product, exchange, expiration, strike and right are read from the contract and are never stored beside it.
  - `settlement_currency` is derived from the point value.
  - It is not ordered and performs no calculation.
- No `LotSize`, `ContractMultiplier`, option contract count, tick size, margin or P&L concept is introduced.

### Application

- `OptionContractEconomicsRepository.get_economics(contract: OptionContract) -> OptionContractEconomics | None` returns the economics of exactly the requested contract, or `None`.
- `OptionContractEconomicsStore.store(economics: tuple[OptionContractEconomics, ...]) -> int` persists immutable economics with the same semantics as the Futures store:
  - an absent contract is stored;
  - an identical re-store is an idempotent success;
  - different economics for a stored contract raise `OptionContractEconomicsConflictError` and leave the stored value unchanged;
  - a batch containing one contract twice is a conflict;
  - an empty batch returns zero;
  - a batch is stored completely or not at all, and success returns the batch length.
- `GetOptionContractEconomicsUseCase.execute(contract)` is the fail-closed query.
  - No stored economics raise `OptionContractEconomicsNotFoundError`, which carries the requested contract.
  - A repository answer that is not `OptionContractEconomics`, or that belongs to any other contract, raises `OptionContractEconomicsContractViolationError`.

### Infrastructure

- A new insert-only SQLite table in the same Northstar database file:

      option_contract_economics (
          product_code TEXT NOT NULL,
          exchange_code TEXT NOT NULL,
          expiration_date TEXT NOT NULL,
          strike TEXT NOT NULL,
          option_right TEXT NOT NULL,
          point_value_amount TEXT NOT NULL,
          settlement_currency TEXT NOT NULL,
          PRIMARY KEY (product_code, exchange_code, expiration_date,
                       strike, option_right)
      )

- Every value is canonical text. The strike and point-value amount are canonical Decimal spellings and never REAL, the right is exactly `CALL` or `PUT`, and the expiration is `YYYY-MM-DD`. Because `OptionStrike` has one canonical spelling, text equality in the key is numeric equality.
- Every read rebuilds the Core values and requires them to reproduce the stored text exactly. A corrupt or non-canonical row fails loudly.
- `option_contract_economics` is created lazily, by its own initializer `initialize_option_contract_economics_schema`, on the first Options economics configuration (write) operation. It is not added to the Futures database initializer, so a Futures-only database stays Futures-only. No Futures table is read, written or altered.
- The repository is genuinely read-only. It opens the database in SQLite read-only mode and never runs the initializer. A database file that does not exist, or one without the Options table, holds no option economics and answers `None`; a read never creates the file, the table or any other schema object.

### API

- `northstar options economics set` stores the economics of one exact option contract, and `northstar options economics show` displays them.
- Every value is parsed into its Core value before the database is opened. The right is exactly `CALL` or `PUT`; `CE`, `PE` and lowercase spellings are rejected.
- `set` is a write. It holds the database operations lock and is the only Options economics command that creates the Options table.
- `show` is read-only. It takes no operations lock and creates nothing. A missing record, a missing Options table and a database file that does not exist yet all report the economics as not configured (exit 4); an invalid database path is still a configuration error (exit 3).
- The commands use their own Options wiring. The Futures `DatabaseRuntime`, `initialize_database` and `build_database_runtime`, and the existing `economics set` and `economics show` commands, are unchanged.
- There is no HTTP endpoint and no Web change.

## Architecture Invariants

1. **Contract identity.** Option economics are identified by the complete `OptionContract`: product, exchange, expiration, strike and right.
2. **Exact lookup.** A lookup returns the economics of exactly the requested contract or nothing. A neighbouring strike, the other right, another expiry or another product never answers.
3. **Fail closed.** Missing economics are an error that names the contract. A value is never borrowed, defaulted or treated as zero.
4. **No product-level fallback.** There is no product-level option economics concept, table or port.
5. **One rate.** `OptionPointValue` is the only economics and P&L rate. Lot size and any quote multiplier are folded into it, not modelled beside it.
6. **Currency in one place.** The currency belongs only to the point value. `OptionPremium` and `OptionStrike` carry no currency.
7. **Insert-only.** Stored economics are immutable reference facts. An identical re-store is idempotent, and changed economics are a conflict.
8. **Count remains contracts.** A future option contract count will count option contracts, never units of the underlying.
9. **No currency totals.** Valuations remain per contract. Currencies are never summed or converted.
10. **No Futures genericization.** No Futures, Derivatives or Foundation type, port, table or command is changed or shared.

## P&L Definition

Option P&L is defined here but not implemented. For a long-premium position of `n` contracts entered at premium `e` and exited at premium `x`:

    realized P&L = (x - e) * point_value.amount * n    in point_value.currency

Because the first lifecycle holds long premium only, there is no direction factor, and the loss on a position cannot exceed the premium paid. The formula, its Decimal context and the option contract count are the subject of the paper-trading milestone.

## Rationale

### Why the Complete Contract

A lot revision applies per expiry, so two expiries of one product can carry different point values at the same time. Keying by the complete contract represents every observed lot history exactly, with no effective date and no resolution rule. Strike and right are part of the key because they are part of the contract's identity: an economics record that named only product and expiry would be a second, coarser identity that could answer for a contract it never described.

### Why Lot Size Is Folded into the Point Value

P&L consumes one rate. A separate lot size beside the point value would be a second source of truth that could disagree with it, and nothing consumes lot size on its own. ADR-009 made the same choice for Futures.

### Why the Futures Types Are Not Reused

`FuturesPointValue` is a value per Futures quote point, and `FuturesContractEconomics` is keyed by a `FuturesContract`. Reusing or genericizing them would change frozen contracts that paper trading, broker execution, Application and persistence depend on. Parallel Options types state plainly what they describe, which is the choice ADR-008 and ADR-010 already made.

### Why the Table Is Created Lazily

The Futures database initializer defines the exact table set of a Futures database, and released acceptance tests pin that set. Creating the Options table only on the first Options economics write keeps every existing Futures database and test unchanged, and needs no migration: an existing deployed database gains the table the first time an operator stores option economics. Reading option economics never creates it, so a read-only command never changes a database's schema. An older build ignores the table.

### Why a Dedicated Fail-Closed Query

In Futures, fail-closed resolution lives inside the valuation use cases. Options has no valuation consumer yet, so the rule would otherwise have no home in Application. A dedicated query states it once and lets the operator command and the later P&L work share it.

## Alternatives Considered

- **Reuse `FuturesPointValue` and `FuturesContractEconomics`.** Rejected: they are Futures-typed, and a shared type would let a Futures rate value an option or the reverse.
- **A generic point value or contract-economics type.** Rejected: it would change frozen Futures contracts for symmetry alone.
- **Product-level or product-and-expiry option economics.** Rejected: they cannot represent concurrent expiries with different lots, and a coarser key would silently answer for contracts it never described.
- **Store a lot size beside the point value.** Rejected: two sources of truth for one rate.
- **Add the Options table to the Futures database initializer.** Rejected: it would change the Futures database and the tests that pin it.
- **Accept `CE` and `PE` at the operator command.** Rejected: they are provider vocabulary, mapped in Infrastructure.

## Consequences

### Positive Consequences

- Every option contract carries its own exact rate, so lot revisions between expiries are represented without any time dimension.
- Missing economics identify the exact option contract.
- Futures economics, tables, commands and tests are unchanged.

### Implementation Consequences

- Operators record economics once per option contract before it is valued, with `northstar options economics set`. Each strike and right of an expiry is its own contract and needs its own record.
- The deployed SQLite database gains `option_contract_economics` the first time `northstar options economics set` runs. `northstar options economics show` never adds it.

### Future Consequences

- Option P&L will resolve economics through `GetOptionContractEconomicsUseCase` and must fail closed.
- Provider adapters will validate the provider contract master's lot size against stored option economics and fail on disagreement, as ADR-009 deferred for Futures. This validation remains deferred.

## Scope

This ADR establishes contract-scoped Option economics and freezes their invariants.

It does not:

- introduce an option contract count, positions, orders, fills or P&L calculation;
- introduce option market data, an option chain, provider resolution or contract selection;
- introduce fees, taxes, margin, Greeks, implied volatility, exercise or settlement;
- add an HTTP endpoint, a Web change or a deployment change;
- change ADR-009, ADR-010 or any Futures, Derivatives or Foundation type, port, table or command.

## Future Considerations

Deferred:

- the option contract count and option P&L, with their Decimal context;
- deriving or validating economics from a provider contract master;
- effective-dated economics for a lot change after listing;
- fees, commissions, taxes and margin;
- FX conversion and cross-currency totals.

## Related Documents

- ADR-002: Architecture Constitution
- ADR-008: Futures Broker Demo Execution Boundary
- ADR-009: Contract-Scoped Futures Economics
- ADR-010: Option Contract Identity
- Ubiquitous Language: `architecture/Ubiquitous-Language.md`
