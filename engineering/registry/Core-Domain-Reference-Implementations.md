# Core Domain Reference Implementation Registry

This registry records approved Core Domain Reference Entity implementations.

The registry currently contains:

- Instrument, the first approved Core Domain Reference Entity.
- Exchange, the second approved Core Domain Reference Entity.
- Listing, the third approved Core Domain Reference Entity.

The Core Domain Foundation is now complete.

Reference Core Domain Entities:

- Instrument
- Exchange
- Listing

Reference Core Domain Value Objects:

- ListingStatus
- Tradability

Future Core Domain work builds upon these approved reference implementations.

Future Core Domain entities will be added only after:

- Design Approval,
- Implementation Approval,
- Contract Test Approval.

The Foundation registry at `docs/engineering/registry/Reference-Implementations.md` contains approved Foundation Reference Value Objects.

The Foundation and Core Domain registries serve different architectural purposes and remain independent.

Core Domain entities follow:

- Design-First Engineering
- ADR-driven architecture
- Cross-Entity Architecture Review
- Reference Entity Contract Testing

This registry records only approved reference implementations. It is not a catalog of every Core Domain entity.

---

## Reference Entities

| Entity     | Status   | Version |
| ---------- | -------- | ------- |
| Instrument | Approved | v1.0    |
| Exchange   | Approved | v1.0    |
| Listing    | Approved | v1.0    |

### RCE-001 — Instrument

| Field                    | Value             |
| ------------------------ | ----------------- |
| Version                  | v1.0              |
| Reference Entity         | Instrument        |
| Status                   | Approved          |
| Domain                   | Core Domain       |
| Repository               | northstar-core    |
| Package                  | domain.instrument |
| Architecture             | ADR-005           |
| Design                   | Approved          |
| Implementation           | Approved          |
| Reference Contract Suite | Approved          |
| Approval Date            | 2026-08-16        |

**Business Purpose**

Represents the intrinsic tradable financial concept.

**Business Responsibilities**

- Intrinsic business identity
- Descriptive business meaning
- Foundation composition
- Business behavior

**Composes**

- Symbol

**Explicitly Excludes**

- Exchange
- Listing
- ExchangeCode
- Currency
- Price
- Money

**Future Dependents**

- Listing
- Market Data
- Order
- Trade
- Position
- Portfolio

**Reference Contract Test Suite**

`tests/domain/instrument/test_instrument.py`

The suite verifies the approved mutable Core Domain entity contract, Foundation composition, intrinsic validation, business behavior, and Core Domain boundary protection. Entity equality, hashing, and permanent identity semantics remain intentionally deferred until the Core Domain Entity Identity Model is approved.

---

### RCE-002 — Exchange

| Field                    | Value           |
| ------------------------ | --------------- |
| Version                  | v1.0            |
| Reference Entity         | Exchange        |
| Status                   | Approved        |
| Domain                   | Core Domain     |
| Repository               | northstar-core  |
| Package                  | domain.exchange |
| Architecture             | ADR-005         |
| Design                   | Approved        |
| Implementation           | Approved        |
| Reference Contract Suite | Approved        |
| Approval Date            | 2026-08-16      |

**Business Purpose**

Represents the trading venue.

**Business Responsibilities**

- Trading venue identity
- Stable intrinsic descriptive characteristics
- Venue lifecycle

**Composes**

- ExchangeCode

**Explicitly Excludes**

- Instrument
- Listing
- Currency
- Price
- Money
- Market participation
- Tradability

**Future Dependents**

- Listing
- Market Data
- Order
- Trade

**Reference Contract Test Suite**

`tests/domain/exchange/test_exchange.py`

The suite verifies the approved mutable Core Domain entity contract, ExchangeCode composition, intrinsic validation, business behavior, and ADR-005 boundary protection. Entity equality, hashing, permanent identity semantics, and lifecycle transition mechanics remain intentionally deferred where not defined by the approved architecture.

---

### RCE-003 — Listing

| Field                    | Value            |
| ------------------------ | ---------------- |
| Version                  | v1.0             |
| Reference Entity         | Listing          |
| Status                   | Approved         |
| Domain                   | Core Domain      |
| Repository               | northstar-core   |
| Package                  | domain.listing   |
| Architecture             | ADR-005, ADR-006 |
| Design                   | Approved         |
| Implementation           | Approved         |
| Reference Contract Suite | Approved         |
| Approval Date            | 2026-08-17       |

**Business Purpose**

Represents the market-specific manifestation of an Instrument on an Exchange.

**Business Responsibilities**

- Market-specific participation
- Composition of approved Core Domain Value Objects
- Stable market-specific descriptive characteristics

**Composes**

Foundation Value Objects:

- Currency

Core Domain Value Objects:

- ListingStatus
- Tradability

Core Domain Entities:

- Instrument
- Exchange

**Explicitly Excludes**

- ExchangeCode ownership
- Price
- Money
- Market Data
- Orders
- Trades
- Position
- Portfolio

**Future Dependents**

- Market Data
- Order
- Trade
- Position
- Portfolio

**Reference Contract Test Suite**

`tests/domain/listing/test_listing.py`

The suite verifies approved composition, mutable Listing business behavior, Core Domain Value Object usage, ExchangeCode ownership boundaries, and deferred entity identity semantics.

---

## Registry Strategy

Each architectural domain maintains its own Reference Implementations Registry.

- Foundation and Core Domain registries remain independent.
- Future domains such as Market, Portfolio, Trading, Analytics, and Risk may introduce their own Reference Implementation Registries when they establish stable reference implementations.
- Each registry records only approved reference implementations for its bounded context.
- The registry structure scales with the architecture rather than becoming a single monolithic document.
