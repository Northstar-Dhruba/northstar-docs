# ADR-007: PointInTime Foundation Extension

## Status

Proposed

## Date

2026-08-17

## Decision Makers

Northstar Architecture Team

## Context

The Foundation Temporal family currently contains Timeframe.

Timeframe represents the identity of a temporal interval. It answers the business question:

> What interval?

Examples include:

- 1m
- 5m
- 1h
- 1d

Business workflows also require a reusable concept representing a specific temporal location. It answers the business question:

> When?

Quote implementation and the Temporal Point-in-Time Business Analysis identified this requirement.

Timeframe and PointInTime are complementary:

- Timeframe represents interval identity.
- PointInTime represents a specific temporal location.

Neither replaces the other.

Quote, Tick, Trade, Order, Portfolio history, Analytics, and Risk all naturally require PointInTime meaning.

## Decision

Approve PointInTime as a Foundation Value Object.

PointInTime:

- represents a specific temporal location,
- is immutable,
- has value semantics,
- has no independent identity,
- is reusable across bounded contexts,
- complements Timeframe,
- and belongs to the Foundation.

## Rationale

### Why PointInTime Is Not a Core Domain Value Object

PointInTime is not specific to one Core Domain concept. It is reusable across Market Data, Orders, Trades, Portfolio history, Analytics, Risk, and future bounded contexts.

Placing PointInTime in the Core Domain would make a reusable temporal primitive depend on one domain layer and would encourage duplication elsewhere.

### Why PointInTime Is Not a Core Domain Entity

PointInTime has no independent business identity or lifecycle.

Its business meaning is defined by temporal value, not by an autonomous business role.

### Why PointInTime Is Not a Market Data Concept

Market Data uses PointInTime but does not define it.

PointInTime applies beyond Market Data to workflow, historical, analytical, and risk concepts.

### Why PointInTime Is Not an Implementation Detail

A technical representation of time is an implementation concern. PointInTime is the business concept that expresses when a relevant fact, observation, action, or state occurred.

This business meaning is shared across bounded contexts and requires a canonical domain term.

## Foundation Impact

The Temporal family now contains:

- Timeframe
- PointInTime

The approved meanings are:

```text
Timeframe
    |
    v
Interval identity

PointInTime
    |
    v
Specific temporal location
```

No existing Foundation Value Object changes.

No Core Domain concepts change.

No Market Data concepts change.

The Foundation is extended, not redesigned.

## Dependency Impact

Future dependents include:

- Market Observation
- Quote
- OHLC Bar
- Tick
- Trade
- Order
- Portfolio history
- Analytics
- Risk

Higher layers compose PointInTime. They do not redefine it.

## Naming

PointInTime is approved as the canonical ubiquitous-language term.

### Rejected Terms

#### Timestamp

Rejected because it primarily describes a technical representation rather than business meaning.

#### Instant

Rejected because it is concise but ambiguous and may be confused with technical runtime or messaging terminology.

#### ObservationTime

Rejected because it is limited to Market Data observations and does not apply to Orders, Trades, Portfolio history, Analytics, or Risk.

#### MarketInstant

Rejected because it limits the concept to Market Data despite broader temporal reuse.

## Consequences

### Positive Consequences

- Reusable temporal primitive.
- Consistent temporal vocabulary.
- Prevention of duplicated temporal concepts across bounded contexts.
- Cleaner Market Data model.
- Cleaner workflow modeling.
- Clear complement to Timeframe.

### Implementation Consequences

Quote implementation may resume after PointInTime is implemented under the approved Foundation Value Object process.

This ADR does not define PointInTime attributes, validation, normalization, or implementation.

## Scope

This ADR establishes architecture only.

It does not:

- design PointInTime,
- define attributes,
- define validation,
- define normalization,
- define implementation,
- modify Timeframe,
- modify existing Foundation Value Objects,
- modify Core Domain concepts,
- or modify Market Data concepts.

## Related Documents

- Foundation Overview
- ADR-004: Composed Foundation Value Objects
- ADR-005: Core Domain Entity Boundaries
- ADR-006: Core Domain Value Objects
- Market Observation Design Specification v1.0
- Quote Design Specification v1.0
