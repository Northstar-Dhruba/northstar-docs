# Core Domain Foundation

## 1. Core Domain Overview

The Core Domain models tradable financial concepts and their participation in markets.

It builds on the completed Foundation package, which provides primitive business concepts such as identity, measurement, financial denomination, monetary value, and temporal intervals.

The Core Domain does not redefine Foundation concepts. Instead, it composes approved Foundation Value Objects into richer business entities with distinct responsibilities and lifecycles.

## 2. The Three Foundational Entities

### Instrument

Instrument represents the intrinsic tradable financial concept.

It answers the question:

> What is being traded?

### Exchange

Exchange represents the trading venue.

It answers the question:

> Where is it traded?

### Listing

Listing represents the market-specific manifestation of an Instrument on an Exchange.

It answers the question:

> How does this Instrument participate on this Exchange?

Together, Instrument, Exchange, and Listing form the conceptual foundation of the Core Domain. Instrument provides intrinsic meaning, Exchange provides venue meaning, and Listing connects those concepts to market participation.

## 3. Conceptual Relationship

The approved relationship is:

```text
Instrument        Exchange
      \           /
       \         /
        ▼       ▼
         Listing
```

Instrument remains independent of market-specific participation.

Exchange remains independent of any individual Instrument.

Listing is the business bridge between intrinsic identity and market participation.

## 4. Foundation Composition

The Core Domain reuses the completed Foundation through approved Value Objects:

```text
Instrument
    │
    ▼
  Symbol

Exchange
    │
    ▼
ExchangeCode

Listing
    │
    ▼
 Currency
```

Foundation Value Objects remain the authoritative source for primitive business concepts.

Core Domain entities compose those concepts and never redefine their business meaning.

## 5. Business Boundaries

### Instrument

- Intrinsic identity
- Descriptive meaning

### Exchange

- Trading venue
- Venue meaning
- Venue lifecycle

### Listing

- Market participation
- Listing status
- Tradability
- Trading Currency
- Market-specific characteristics

These responsibilities are intentionally separated to preserve high cohesion, low coupling, and long-term maintainability.

## 6. Core Domain Dependency Model

The approved downstream dependency model is:

```text
Listing
   │
   ▼
Market Data
   │
   ▼
Order
   │
   ▼
Trade
   │
   ▼
Position
   │
   ▼
Portfolio
   │
   ▼
Analytics / Risk
```

Downstream concepts depend on Listing rather than directly on Instrument or Exchange because market workflows require the combined context of an intrinsic tradable concept and its venue-specific manifestation.

Instrument and Exchange remain stable business references. Listing supplies the market context needed by observations, execution, exposure, portfolio, analytics, and risk concepts.

## 7. Architectural Principles

The Core Domain follows these conceptual principles:

- Design-First Engineering
- Architecture before implementation
- Stable entity boundaries
- Composition over duplication
- Foundation reuse
- Explicit business responsibilities
- Ubiquitous Language
- Cross-Entity Architecture Review

These principles preserve business clarity as the Core Domain expands.

## 8. Current Status

Foundation

Status

Complete

Core Domain Foundation

Status

Complete

Reference Core Domain Entities

- Instrument
- Exchange
- Listing

Future work now focuses on building business workflows on top of these approved entities.

## 9. Future Evolution

The next architectural layer includes:

- Market Data
- Order
- Trade
- Position
- Portfolio
- Analytics
- Risk

These concepts build upon the approved Core Domain Foundation rather than redefining Instrument, Exchange, Listing, or the Foundation Value Objects they compose.
