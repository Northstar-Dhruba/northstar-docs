# Portfolio Reference Implementations Registry

This registry records the approved Reference Ownership Interpretation Aggregate Root family in the Northstar Portfolio bounded context. Each entry represents a reviewed, approved, and designated baseline for future ownership interpretation aggregates.

The Portfolio bounded context now contains the approved Reference Aggregate Root family:

- Portfolio, the Reference Ownership Interpretation Aggregate Root.

Its supporting Value Objects and subordinate Entity preserve aggregate identity, participant attribution, temporal snapshot context, and Listing-specific ownership interpretation without becoming Aggregate Roots themselves.

These implementations establish the engineering reference pattern for mutable ownership interpretation aggregates. The Foundation, Core Domain, Market Data, Orders, and Trades registries remain independent and unchanged.

---

## Reference Family Overview

### Reference Aggregate Root

- Portfolio

### Supporting Portfolio Value Objects

- PortfolioIdentity

### Supporting Portfolio Entity

- Position

### Supporting Core Domain Value Objects

- ParticipantIdentity
- ParticipantReference

## Reference Aggregate Root

The registry is aggregate-centric. Supporting Value Objects and subordinate Entities are documented as subordinate concepts and are not registered as Aggregate Roots.

| Identifier | Aggregate Root | Classification                                    | Supporting Portfolio Value Objects | Supporting Portfolio Entity | Supporting Core Domain Concepts           | Status   | Version |
| ---------- | -------------- | ------------------------------------------------- | ---------------------------------- | --------------------------- | ----------------------------------------- | -------- | ------- |
| RAG-003    | Portfolio      | Reference Ownership Interpretation Aggregate Root | PortfolioIdentity                  | Position                    | ParticipantIdentity, ParticipantReference | Approved | v1.0    |

### RAG-003 — Portfolio

| Field                    | Value                                             |
| ------------------------ | ------------------------------------------------- |
| Version                  | v1.0                                              |
| Reference Aggregate Root | Portfolio                                         |
| Classification           | Reference Ownership Interpretation Aggregate Root |
| Status                   | Approved                                          |
| Repository               | northstar-core                                    |
| Package                  | portfolio                                         |
| Design                   | Approved                                          |
| Implementation           | Approved                                          |
| Reference Contract Suite | Approved                                          |

**Supporting Portfolio Value Objects**

- PortfolioIdentity, the fourth Aggregate Identity Value Object reference implementation.

**Supporting Portfolio Entity**

- Position, the Reference Subordinate Entity for Listing-specific ownership interpretation.

**Supporting Core Domain Concepts**

- ParticipantIdentity, the Core Domain participant identity Value Object.
- ParticipantReference, the Reference Identity Association Value Object composed by Portfolio for participant attribution.

**Approved Aggregate Composition**

```text
Portfolio
    |
    +-- PortfolioIdentity
    +-- ParticipantReference
    +-- PointInTime
    +-- Position(s)
           |
           +-- Listing
```

Portfolio owns current ownership interpretation, aggregate consistency, and the Position collection. It permits zero or more Positions and protects the invariant of at most one Position for each Listing. Position inherits participant and temporal context from Portfolio. Portfolio does not own Trade, Order, Market Data, Risk, Performance, Settlement, or Workflow.

**Reference Contract Suite**

`tests/portfolio/test_portfolio.py`

The suite verifies approved aggregate composition, dependency validation, identity-based equality and hashing, Position collection consistency, mutable aggregate structure, aggregate boundaries, and subordinate Position behavior.

## Engineering Summary

Portfolio provides:

- aggregate ownership,
- identity-based equality,
- mutable ownership interpretation,
- Position collection ownership,
- one Position per Listing invariant,
- and a Reference Aggregate Contract Suite.

Supporting concepts establish:

```text
PortfolioIdentity
        |
        v
Aggregate Identity Value Object

Position
        |
        v
Reference Subordinate Entity
```

ParticipantIdentity and ParticipantReference remain Core Domain supporting concepts. Position remains subordinate to Portfolio and does not own ParticipantReference, PointInTime, Trade, or Portfolio.

## Architecture Summary

The approved Portfolio architecture is:

```text
Portfolio bounded context
    |
    +-- Portfolio
           |
           +-- PortfolioIdentity
           +-- ParticipantReference
           +-- PointInTime
           +-- Position(s)
                  |
                  +-- Listing
```

Portfolio remains the Aggregate Root. Portfolio owns Position. Position inherits participant and temporal context from Portfolio. Trade remains immutable execution truth, and Market Data remains external valuation context.

## Engineering Patterns

Northstar contains the following distinct engineering patterns:

### Foundation Value Objects

```text
Business primitives
```

### Observation State Value Objects

```text
Market meaning
```

### Lifecycle State Value Objects

```text
Aggregate condition
```

### Aggregate Identity Value Objects

```text
Opaque aggregate identity
```

The approved Aggregate Identity references are:

- OrderIdentity
- ParticipantIdentity
- TradeIdentity
- PortfolioIdentity

PortfolioIdentity is the fourth approved Aggregate Identity Value Object reference implementation.

### Identity Association Value Objects

```text
Cross-aggregate attribution
```

ParticipantReference remains the approved Identity Association reference implementation.

### Reference Aggregate Roots

```text
Mutable Aggregate Root
    |
    +-- Order
    +-- Portfolio

Immutable Aggregate Root
    |
    +-- Trade
```

### Reference Subordinate Entities

```text
Position
```

These patterns remain distinct. No generic identity, aggregate, status, or ownership framework has been introduced.

## Architecture Consistency Review

- Foundation remains unchanged.
- Core Domain remains unchanged.
- Market Data remains unchanged.
- Orders remains unchanged.
- Trades remains unchanged.
- Portfolio remains the Reference Ownership Interpretation Aggregate Root.
- PortfolioIdentity remains the fourth Aggregate Identity Value Object.
- Position remains the Reference Subordinate Entity.
- ParticipantIdentity remains unchanged.
- ParticipantReference remains the Reference Identity Association Value Object.
- No new architectural layer exists.
- No implementation behavior changed.

## Registry Status

Three Reference Aggregate Roots are approved across the transactional and ownership interpretation bounded contexts:

- Order, the mutable Reference Aggregate Root for participant intent.
- Trade, the immutable Reference Aggregate Root for historical execution truth.
- Portfolio, the mutable Reference Aggregate Root for current ownership interpretation.

The Portfolio registry records the approved Portfolio family:

- Portfolio
- PortfolioIdentity
- Position
- ParticipantIdentity
- ParticipantReference
