# Trades Reference Implementations Registry

This registry records the approved Reference Aggregate Root family in the Northstar Trades bounded context. Each entry represents a reviewed, approved, and designated baseline for immutable historical transactional aggregates.

The Trades bounded context now contains the approved Reference Aggregate Root family:

- Trade, the Reference Immutable Aggregate Root.

The supporting Value Objects preserve Trade aggregate identity and participant attribution without becoming Aggregate Roots themselves.

These implementations establish the engineering reference pattern for immutable historical transactional aggregates. The Foundation, Core Domain, Market Data, and Orders registries remain independent and unchanged.

---

## Reference Family Overview

### Reference Aggregate Root

- Trade

### Supporting Trades Value Objects

- TradeIdentity

### Supporting Core Domain Value Objects

- ParticipantIdentity
- ParticipantReference

## Reference Aggregate Root

The registry is aggregate-centric. Supporting Value Objects are documented as subordinate concepts and are not registered as Aggregate Roots.

| Identifier | Aggregate Root | Classification                     | Supporting Trades Value Objects | Supporting Core Domain Concepts           | Status   | Version |
| ---------- | -------------- | ---------------------------------- | ------------------------------- | ----------------------------------------- | -------- | ------- |
| RAG-002    | Trade          | Reference Immutable Aggregate Root | TradeIdentity                   | ParticipantIdentity, ParticipantReference | Approved | v1.0    |

### RAG-002 — Trade

| Field                    | Value                              |
| ------------------------ | ---------------------------------- |
| Version                  | v1.0                               |
| Reference Aggregate Root | Trade                              |
| Classification           | Reference Immutable Aggregate Root |
| Status                   | Approved                           |
| Repository               | northstar-core                     |
| Package                  | trades                             |
| Design                   | Approved                           |
| Implementation           | Approved                           |
| Reference Contract Suite | Approved                           |

**Supporting Trades Value Objects**

- TradeIdentity, the third Aggregate Identity Value Object reference implementation.

**Supporting Core Domain Concepts**

- ParticipantIdentity, the Core Domain participant identity Value Object.
- ParticipantReference, the Reference Identity Association Value Object composed by Trade for participant attribution.

**Approved Aggregate Composition**

```text
Trade
    |
    +-- TradeIdentity
    +-- Listing
    +-- PointInTime
    +-- Quantity
    +-- Price
    +-- ParticipantReference
           |
           +-- ParticipantIdentity
```

Trade owns completed execution outcome, historical execution truth, and aggregate consistency. It does not own Order, Market Data, Portfolio, Participant, Settlement, Accounting, or Workflow. TradeStatus is intentionally absent from Version 1.0.

**Reference Contract Suite**

`tests/trades/test_trade.py`

The suite verifies approved aggregate composition, dependency validation, identity-based equality and hashing, aggregate boundaries, immutable aggregate behavior, immutable composed Value Objects, and historical identity preservation.

## Engineering Summary

The Reference Aggregate Root shares:

- aggregate ownership,
- identity-based equality,
- aggregate consistency,
- explicit composition,
- and a Reference Contract Suite.

Trade additionally establishes:

- immutable aggregate behavior,
- historical execution truth,
- and execution outcome preservation.

Supporting Value Objects establish:

```text
Aggregate Identity
        |
        v
TradeIdentity

Identity Association
        |
        v
ParticipantReference
```

ParticipantIdentity remains the Core Domain Value Object composed by ParticipantReference. Supporting Value Objects remain subordinate to their owning aggregate or association and do not become Aggregate Roots.

## Architecture Summary

The approved Trades architecture is:

```text
Trades bounded context
    |
    +-- Trade
           |
           +-- TradeIdentity
           +-- Listing
           +-- PointInTime
           +-- Quantity
           +-- Price
           +-- ParticipantReference
                  |
                  +-- ParticipantIdentity
```

Trade remains the Aggregate Root and is immutable after creation. It records historical execution truth. ParticipantReference preserves aggregate boundaries by associating Trade with Participant identity without transferring Participant ownership. TradeStatus is intentionally absent from Version 1.0.

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

TradeIdentity is the third approved Aggregate Identity Value Object reference implementation, following OrderIdentity and ParticipantIdentity.

### Identity Association Value Objects

```text
Cross-aggregate attribution
```

ParticipantReference is the approved Identity Association reference implementation.

### Reference Aggregate Roots

```text
Mutable Aggregate Root
    |
    v
Order

Immutable Aggregate Root
    |
    v
Trade
```

These patterns remain distinct. No generic status, identity, reference, or aggregate framework has been introduced.

## Architecture Consistency Review

- Foundation remains unchanged.
- Core Domain remains unchanged.
- Market Data remains unchanged.
- Orders remains unchanged.
- Trade remains the Reference Immutable Aggregate Root.
- TradeIdentity remains the third Aggregate Identity Value Object.
- ParticipantIdentity remains unchanged.
- ParticipantReference remains the Reference Identity Association Value Object.
- No new architectural layer exists.
- No implementation behavior changed.

## Registry Status

Two Reference Aggregate Roots are approved across the transactional bounded contexts:

- Order, the mutable Reference Aggregate Root.
- Trade, the immutable Reference Aggregate Root.

The Trades registry records the approved Trade family:

- Trade
- TradeIdentity
- ParticipantIdentity
- ParticipantReference
