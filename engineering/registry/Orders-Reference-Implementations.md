# Orders Reference Implementations Registry

This registry records the approved Reference Aggregate Root family in the Northstar Orders bounded context. Each entry represents a reviewed, approved, and designated baseline for future transactional aggregates.

The Orders bounded context now contains the approved Reference Aggregate Root family:

- Order, the Reference Aggregate Root.

Its supporting Value Objects preserve lifecycle meaning, aggregate identity, and participant identity association without becoming Aggregate Roots themselves.

These implementations establish the engineering reference patterns for future transactional aggregates. The Foundation, Core Domain, and Market Data registries remain independent and unchanged.

---

## Reference Family Overview

### Reference Aggregate Root

- Order

### Supporting Lifecycle Value Objects

- OrderStatus

### Supporting Aggregate Identity Value Objects

- OrderIdentity

### Supporting Core Domain Identity Value Objects

- ParticipantIdentity

### Supporting Identity Association Value Objects

- ParticipantReference

## Reference Aggregate Root

The registry is aggregate-centric. Supporting Value Objects are documented as subordinate concepts and are not registered as Aggregate Roots.

| Identifier | Aggregate Root | Classification           | Supporting Orders Value Objects | Supporting Core Domain Concepts           | Status   | Version |
| ---------- | -------------- | ------------------------ | ------------------------------- | ----------------------------------------- | -------- | ------- |
| RAG-001    | Order          | Reference Aggregate Root | OrderStatus, OrderIdentity      | ParticipantIdentity, ParticipantReference | Approved | v1.0    |

### RAG-001 — Order

| Field                    | Value                    |
| ------------------------ | ------------------------ |
| Version                  | v1.0                     |
| Reference Aggregate Root | Order                    |
| Classification           | Reference Aggregate Root |
| Status                   | Approved                 |
| Repository               | northstar-core           |
| Package                  | orders                   |
| Design                   | Approved                 |
| Implementation           | Approved                 |
| Reference Contract Suite | Approved                 |

**Supporting Orders Value Objects**

- OrderStatus, the Reference Lifecycle State Value Object.
- OrderIdentity, the Reference Aggregate Identity Value Object.

**Supporting Core Domain Concepts**

- ParticipantIdentity, the Core Domain participant identity Value Object.
- ParticipantReference, the Reference Identity Association Value Object composed by Order for participant attribution.

**Approved Aggregate Composition**

```text
Order
    |
    +-- OrderIdentity
    +-- Listing
    +-- PointInTime
    +-- Quantity
    +-- Price
    +-- ParticipantReference
    |      |
    |      +-- ParticipantIdentity
    |
    +-- OrderStatus
```

Order owns participant intent, transaction commitment, lifecycle ownership, and aggregate consistency. It does not own Participant, Market Data, Trade, Portfolio, workflow, or execution behavior.

**Reference Contract Suite**

`tests/orders/test_order.py`

The suite verifies approved aggregate composition, dependency validation, identity-based equality and hashing, aggregate boundaries, mutable aggregate structure without lifecycle behavior, and immutable composed Value Objects.

## Engineering Summary

The Reference Aggregate Root shares:

- aggregate ownership,
- identity-based equality,
- lifecycle ownership,
- aggregate consistency,
- explicit composition,
- and a Reference Contract Suite.

Supporting Value Objects establish:

```text
Lifecycle State
        |
        v
OrderStatus

Aggregate Identity
        |
        v
OrderIdentity

Identity Association
        |
        v
ParticipantReference
```

ParticipantIdentity remains the Core Domain Value Object composed by ParticipantReference. Supporting Value Objects remain subordinate to their owning aggregate or association and do not become Aggregate Roots.

## Architecture Summary

The approved Orders architecture is:

```text
Orders bounded context
    |
    +-- Order
           |
           +-- OrderIdentity
           +-- Listing
           +-- PointInTime
           +-- Quantity
           +-- Price
           +-- ParticipantReference
           |      |
           |      +-- ParticipantIdentity
           |
           +-- OrderStatus
```

Order remains the Aggregate Root. Supporting Value Objects remain subordinate. Participant remains independent, and ParticipantReference preserves aggregate boundaries by associating Order with Participant identity without transferring Participant ownership.

## Engineering Patterns

Northstar now contains the following distinct Value Object patterns:

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

OrderStatus is the first approved Orders Lifecycle State reference implementation.

### Aggregate Identity Value Objects

```text
Opaque aggregate identity
```

OrderIdentity and ParticipantIdentity are the first approved reference implementations of the Aggregate Identity family.

### Identity Association Value Objects

```text
Cross-aggregate attribution
```

ParticipantReference is the approved Identity Association reference implementation.

These remain separate engineering patterns. No generic status, identity, or reference framework has been introduced.

## Architecture Consistency Review

- Foundation remains unchanged.
- Core Domain remains unchanged.
- Market Data remains unchanged.
- Order remains the Reference Aggregate Root.
- OrderStatus remains the Reference Lifecycle State Value Object.
- OrderIdentity remains the Reference Aggregate Identity Value Object.
- ParticipantIdentity remains a Core Domain Value Object.
- ParticipantReference remains the Reference Identity Association Value Object.
- No new architectural layer exists.
- No implementation behavior changed.

## Registry Status

One Reference Aggregate Root is approved and recorded:

- Order

Supporting Value Objects are recorded as subordinate concepts:

- OrderStatus
- OrderIdentity
- ParticipantIdentity
- ParticipantReference
