# Strategy Reference Implementations Registry

This registry records the approved Reference Decision Policy Aggregate Root family in the Northstar Strategy bounded context. Each entry represents a reviewed, approved, and designated baseline for future decision-policy aggregates.

The Strategy bounded context now contains the approved Reference Aggregate Root family:

- Strategy, the Reference Decision Policy Aggregate Root.

Its supporting Value Object preserves aggregate identity without becoming an Aggregate Root itself.

These implementations establish the engineering reference pattern for mutable decision-policy aggregates. The Foundation, Core Domain, Market Data, Orders, Trades, and Portfolio registries remain independent and unchanged.

---

## Reference Family Overview

### Reference Aggregate Root

- Strategy

### Supporting Strategy Value Objects

- StrategyIdentity

## Reference Aggregate Root

The registry is aggregate-centric. Supporting Value Objects are documented as subordinate concepts and are not registered as Aggregate Roots.

| Identifier | Aggregate Root | Classification                           | Supporting Strategy Value Objects | Status   | Version |
| ---------- | -------------- | ---------------------------------------- | --------------------------------- | -------- | ------- |
| RAG-004    | Strategy       | Reference Decision Policy Aggregate Root | StrategyIdentity                  | Approved | v1.0    |

### RAG-004 — Strategy

| Field                    | Value                                    |
| ------------------------ | ---------------------------------------- |
| Version                  | v1.0                                     |
| Reference Aggregate Root | Strategy                                 |
| Classification           | Reference Decision Policy Aggregate Root |
| Status                   | Approved                                 |
| Repository               | northstar-core                           |
| Package                  | strategy                                 |
| Design                   | Approved                                 |
| Implementation           | Approved                                 |
| Reference Contract Suite | Approved                                 |

**Supporting Strategy Value Objects**

- StrategyIdentity, the fifth Aggregate Identity Value Object reference implementation.

**Approved Aggregate Composition**

```text
Strategy
    |
    +-- StrategyIdentity
```

Strategy owns decision policy, strategy consistency, and strategy lifecycle. It consumes Market Data, Portfolio context, and historical Trade outcomes, and may influence future Order creation. ParticipantReference and PointInTime remain optional in Version 1.

Strategy does not own Market Data, Portfolio, Trade, Order, Execution, Risk, Performance, or Workflow.

**Reference Contract Suite**

`tests/strategy/test_strategy.py`

The suite verifies minimal approved aggregate composition, dependency validation, identity-based equality and hashing, mutable aggregate structure, aggregate boundaries, and absence of invented policy semantics.

## Engineering Summary

Strategy provides:

- aggregate ownership,
- identity-based equality,
- mutable decision policy,
- decision consistency,
- strategy lifecycle ownership,
- and a Reference Aggregate Contract Suite.

Supporting concepts establish:

```text
StrategyIdentity
        |
        v
Aggregate Identity Value Object
```

## Architecture Summary

The approved Strategy architecture is:

```text
Strategy bounded context
    |
    +-- Strategy
           |
           +-- StrategyIdentity
```

Strategy remains the Aggregate Root and owns decision policy, consistency, and lifecycle. Market Data, Portfolio, and historical Trade outcomes remain external context consumed by Strategy. Strategy may influence future Orders without owning Order intent or lifecycle.

ParticipantReference and PointInTime remain optional in Version 1 and are not part of the minimal approved composition.

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
- StrategyIdentity

StrategyIdentity is the fifth approved Aggregate Identity Value Object reference implementation.

### Reference Aggregate Roots

```text
Mutable Aggregate Root
    |
    +-- Order
    +-- Portfolio
    +-- Strategy

Immutable Aggregate Root
    |
    +-- Trade
```

These patterns remain distinct. No generic identity, aggregate, policy, workflow, or decision framework has been introduced.

## Architecture Consistency Review

- Foundation remains unchanged.
- Core Domain remains unchanged.
- Market Data remains unchanged.
- Orders remains unchanged.
- Trades remains unchanged.
- Portfolio remains unchanged.
- Strategy remains the Reference Decision Policy Aggregate Root.
- StrategyIdentity remains the fifth Aggregate Identity Value Object.
- No new architectural layer exists.
- No implementation behavior changed.

## Registry Status

Four Reference Aggregate Roots are approved across the transactional, ownership interpretation, and decision-policy bounded contexts:

- Order, the mutable Reference Aggregate Root for participant intent.
- Trade, the immutable Reference Aggregate Root for historical execution truth.
- Portfolio, the mutable Reference Aggregate Root for current ownership interpretation.
- Strategy, the mutable Reference Aggregate Root for decision policy.

The Strategy registry records the approved Strategy family:

- Strategy
- StrategyIdentity
