# Foundation Reference Implementation Registry

This document is the official record of approved Foundation Reference Value Object implementations in the Northstar platform. Every entry represents a reviewed, approved, and designated baseline that future Foundation Value Objects in the same category MUST follow unless an ADR documents an explicit architectural deviation.

The Temporal Foundation family contains:

- Timeframe, the approved interval identity concept that answers "What interval?"
- PointInTime, the approved reusable temporal primitive that answers "When?"

Future Foundation temporal concepts MUST follow these approved reference implementations unless a future ADR explicitly changes the architecture.

---

## Reference Value Objects

| Component    | Status   | Version |
| ------------ | -------- | ------- |
| Symbol       | Approved | v1.0    |
| ExchangeCode | Approved | v1.0    |
| Quantity     | Approved | v1.0    |
| Percentage   | Approved | v1.0    |
| Currency     | Approved | v1.0    |
| Price        | Approved | v1.0    |
| Money        | Approved | v1.0    |
| Timeframe    | Approved | v1.0    |
| PointInTime  | Approved | v1.0    |

## Foundation Value Object Family

| Family      | Component    | Status   |
| ----------- | ------------ | -------- |
| Identity    | Symbol       | Approved |
| Identity    | ExchangeCode | Approved |
| Measurement | Quantity     | Approved |
| Measurement | Percentage   | Approved |
| Financial   | Currency     | Approved |
| Financial   | Price        | Approved |
| Financial   | Money        | Approved |
| Temporal    | Timeframe    | Approved |
| Temporal    | PointInTime  | Approved |

## Engineering References vs Business Families

Business Families answer: "What business concept does this Value Object belong to?"

Engineering References answer: "Which approved implementation pattern should this Value Object inherit?"

These concepts are intentionally independent.

This distinction is part of the approved Reference-First Engineering model.

### Engineering Reference Guidance

Future Foundation Value Objects MUST inherit their engineering implementation pattern from the Engineering Reference designated during Architecture Review and recorded in the Reference Implementation Registry.

Engineering References are implementation relationships.

Business Families are domain classification relationships.

These are intentionally different concepts.

Business Family membership does NOT automatically determine the Engineering Reference.

The Reference Implementation Registry is the single source of truth for engineering inheritance.

Future deviations from the designated Engineering Reference MUST be approved through both Architecture Review and Engineering Review before implementation.

**Examples**

- Identity Family
  - Business Family: Identity
  - Engineering Reference: Symbol
- Measurement Family
  - Business Family: Measurement
  - Engineering Reference: Quantity
- Financial Family
  - Business Family: Financial
  - Engineering Reference: Defined per approved Financial Value Object and documented in the Reference Implementation Registry.

Do not imply that every Financial Value Object inherits from the same Engineering Reference.

The Registry remains authoritative.

### RVO-001 — Symbol

| Field         | Value                    |
| ------------- | ------------------------ |
| Version       | v1.0                     |
| Component     | Symbol                   |
| Category      | Foundation Value Object  |
| Status        | Approved                 |
| Repository    | northstar-core           |
| Package       | foundation.value_objects |
| Approval Date | 2026-08-10               |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Engineering Standards**

| Standard | Title                     |
| -------- | ------------------------- |
| ES-001   | Value Objects             |
| ES-002   | Reference Implementations |

**Reference Design**

Approved — Symbol defines the structural baseline for all Foundation value objects.

**Reference Implementation**

Approved — The implementation conforms to ES-001. It uses `@dataclass(frozen=True, slots=True, order=True)`, performs normalization and validation exclusively in `__post_init__`, uses module-level private helpers, exposes a minimal public API, and is fully framework-independent.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_symbol.py` establishes the Northstar Value Object Testing Standard v1.0. The suite is organized into eight sections: Construction, Validation, Normalization, Equality, Hashing, Ordering, Representation, and Edge Cases.

---

### RVO-002 — ExchangeCode

| Field         | Value                    |
| ------------- | ------------------------ |
| Version       | v1.0                     |
| Component     | ExchangeCode             |
| Category      | Foundation Value Object  |
| Status        | Approved                 |
| Repository    | northstar-core           |
| Package       | foundation.value_objects |
| Approval Date | 2026-08-11               |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Engineering Standards**

| Standard | Title                     |
| -------- | ------------------------- |
| ES-001   | Value Objects             |
| ES-002   | Reference Implementations |
| ES-009   | Engineering Standards     |
| ES-010   | Reference Value Objects   |

**Reference Design**

Approved — ExchangeCode defines the approved business-specific extension of the Northstar value object pattern for trading venue identifiers.

**Reference Implementation**

Approved — The implementation conforms to the approved reference pattern and is aligned with the Symbol-based reference structure while applying only the approved business-rule differences.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_exchange_code.py` verifies the ExchangeCode contract using the same structure and review philosophy as the Symbol reference suite.

**Reference Chain**

Symbol v1.0
↓
ExchangeCode v1.0

**Business Purpose**

Represents the canonical identifier of an exchange or trading venue.

---

### RVO-003 — Quantity

| Field                 | Value                    |
| --------------------- | ------------------------ |
| Version               | v1.0                     |
| Component             | Quantity                 |
| Category              | Foundation Value Object  |
| Business Family       | Measurement              |
| Family Root           | Yes                      |
| Status                | Approved                 |
| Repository            | northstar-core           |
| Package               | foundation.value_objects |
| Engineering Reference | Quantity v1.0            |
| Approval Date         | 2026-08-11               |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Reference Design**

Approved — Quantity defines the approved Measurement Family behavioral foundation while inheriting the designated Engineering Reference pattern recorded in the Registry.

**Reference Implementation**

Approved — The implementation conforms to the approved Engineering Reference pattern and applies only the approved Measurement business-rule differences.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_quantity.py` verifies the inherited value object contract plus approved behavioral extensions.

**Reference Chain**

Quantity v1.0

**Business Purpose**

Represents an immutable measurable quantity.

**Business Responsibilities**

- Numeric magnitude
- Decimal normalization
- Arithmetic behavior
- Numeric comparison
- Canonical representation

**Future Dependents**

- Percentage
- Price
- Money

---

### RVO-004 — Percentage

| Field                 | Value                    |
| --------------------- | ------------------------ |
| Version               | v1.0                     |
| Component             | Percentage               |
| Category              | Foundation Value Object  |
| Business Family       | Measurement              |
| Family Root           | No                       |
| Status                | Approved                 |
| Repository            | northstar-core           |
| Package               | foundation.value_objects |
| Engineering Reference | Quantity v1.0            |
| Approval Date         | 2026-08-12               |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Reference Design**

Approved — Percentage defines the approved measurement-family extension of the Quantitative value object pattern for normalized percentage values.

**Reference Implementation**

Approved — The implementation conforms to the approved quantity-based engineering pattern and applies only the approved percentage-specific business rules.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_percentage.py` verifies the inherited value object contract plus approved percentage semantics.

**Reference Chain**

Symbol v1.0
↓
Quantity v1.0
↓
Percentage v1.0

**Business Purpose**

Represents a normalized percentage value within the approved Measurement family.

**Business Responsibilities**

- Signed percentage magnitude
- Decimal normalization
- Arithmetic behavior
- Numeric comparison
- Canonical representation

**Future Dependents**

- Rate
- Discount
- Allocation

---

### RVO-005 — Currency

| Field                 | Value                    |
| --------------------- | ------------------------ |
| Version               | v1.0                     |
| Component             | Currency                 |
| Category              | Foundation Value Object  |
| Business Family       | Financial                |
| Family Root           | Yes                      |
| Status                | Approved                 |
| Repository            | northstar-core           |
| Package               | foundation.value_objects |
| Engineering Reference | Symbol v1.0              |
| Approval Date         | 2026-08-15               |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Reference Design**

Approved — Currency defines the approved Financial Family root while inheriting the Symbol engineering reference pattern.

**Reference Implementation**

Approved — The implementation conforms to the approved reference pattern and applies only the approved financial business-rule differences.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_currency.py` verifies the inherited value object contract with approved financial semantics.

**Reference Chain**

Symbol v1.0
↓
Currency v1.0

**Business Purpose**

Represents a reusable financial denomination code.

**Business Responsibilities**

- Structural currency validation
- Canonical representation
- Financial denomination identity
- Fiat and digital asset support
- Registry-independent validation

**Future Dependents**

- Price
- Money

---

### RVO-006 — Price

| Field               | Value                            |
| ------------------- | -------------------------------- |
| Version             | v1.0                             |
| Component           | Price                            |
| Category            | Foundation Value Object          |
| Business Family     | Financial                        |
| Family Root         | No                               |
| Status              | Approved                         |
| Repository          | northstar-core                   |
| Package             | foundation.value_objects         |
| Engineering Pattern | Composed Foundation Value Object |
| Approval Date       | 2026-08-15                       |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Reference Design**

Approved — Price is the first approved composed Foundation Value Object. It represents a monetary value expressed in a specific Currency and does not inherit its business semantics from Quantity.

**Reference Implementation**

Approved — The implementation conforms to the approved composed Financial Value Object pattern and applies only the approved financial business-rule differences.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_price.py` verifies the approved Price contract using the Northstar composed value object review pattern.

**Reference Chain**

Currency v1.0
↓
Price v1.0

**Business Purpose**

Represents a monetary value expressed in a specific Currency.

**Business Responsibilities**

- Monetary value
- Financial denomination
- Currency-aware arithmetic
- Currency-aware ordering
- Canonical monetary representation

**Future Dependents**

- Money

---

### RVO-007 — Money

| Field                 | Value                            |
| --------------------- | -------------------------------- |
| Version               | v1.0                             |
| Component             | Money                            |
| Category              | Foundation Value Object          |
| Business Family       | Financial                        |
| Family Root           | No                               |
| Status                | Approved                         |
| Repository            | northstar-core                   |
| Package               | foundation.value_objects         |
| Engineering Pattern   | Composed Foundation Value Object |
| Engineering Reference | Price v1.0                       |
| Approval Date         | 2026-08-15                       |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Reference Design**

Approved — Money is the final approved Financial Family member and extends the approved composed Foundation Value Object pattern introduced by Price.

**Reference Implementation**

Approved — The implementation conforms to the approved composed Financial Value Object pattern and applies only the approved financial business-rule differences.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_money.py` verifies the approved Money contract using the same composed value object review pattern as Price.

**Reference Chain**

Currency v1.0
↓
Price v1.0
↓
Money v1.0

**Business Purpose**

Represents monetary value within a financial context.

**Business Responsibilities**

- Financial state
- Monetary value
- Currency-aware arithmetic
- Currency-aware ordering
- Canonical monetary representation
- Positive, zero, and negative financial values

**Future Dependents**

None

---

### RVO-008 — Timeframe

| Field                 | Value                    |
| --------------------- | ------------------------ |
| Version               | v1.0                     |
| Component             | Timeframe                |
| Category              | Foundation Value Object  |
| Business Family       | Temporal                 |
| Family Root           | Yes                      |
| Status                | Approved                 |
| Repository            | northstar-core           |
| Package               | foundation.value_objects |
| Engineering Reference | Symbol v1.0              |
| Approval Date         | 2026-08-15               |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Reference Design**

Approved — Timeframe defines the approved Temporal Family root while inheriting the Symbol engineering reference pattern.

**Reference Implementation**

Approved — The implementation conforms to the approved reference pattern and applies only the approved temporal business-rule differences.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_timeframe.py` verifies the inherited value object contract with approved closed-vocabulary temporal semantics.

**Reference Chain**

Symbol v1.0
↓
Timeframe v1.0

**Business Purpose**

Represents a standardized temporal interval label.

**Business Responsibilities**

- Canonical temporal interval identity
- Closed vocabulary validation
- Case-sensitive semantics
- Stable temporal reference
- Approved financial interval vocabulary

**Future Dependents**

- Market Data
- Charting
- Analytics
- Reporting
- Strategy Evaluation

---

### RVO-009 — PointInTime

| Field                 | Value                    |
| --------------------- | ------------------------ |
| Version               | v1.0                     |
| Component             | PointInTime              |
| Category              | Foundation Value Object  |
| Business Family       | Temporal                 |
| Family Root           | No                       |
| Status                | Approved                 |
| Repository            | northstar-core           |
| Package               | foundation.value_objects |
| Engineering Reference | Symbol v1.0              |
| Approval Date         | 2026-08-17               |

**Review Status**

| Review                | Outcome  |
| --------------------- | -------- |
| Architecture Review   | Approved |
| Implementation Review | Approved |
| Contract Test Review  | Approved |
| Reference Review      | Approved |

**Reference Design**

Approved — PointInTime defines the approved reusable temporal primitive for one specific temporal location while complementing Timeframe interval identity.

**Reference Implementation**

Approved — The implementation conforms to the approved Foundation Value Object pattern and applies canonical UTC normalization, deterministic value equality, and hashing without introducing ordering behavior.

**Reference Contract Test Suite**

Approved — `tests/foundation/value_objects/test_point_in_time.py` verifies canonical UTC normalization, validation, value semantics, hashing, representation, and immutability.

**Reference Chain**

Symbol v1.0
↓
PointInTime v1.0

**Business Purpose**

Represents one specific temporal location having business meaning.

**Business Responsibilities**

- Specific temporal business meaning
- Reusable temporal primitive
- Immutable value semantics
- Reusable across bounded contexts

**Relationship**

Complements Timeframe.

Timeframe
↓
Interval identity

PointInTime
↓
Specific temporal location

**Explicitly Excludes**

- Interval identity
- Duration
- Chronology
- Workflow state
- Implementation representation

**Future Dependents**

- Market Observation
- Quote
- Tick
- Trade
- Order
- Portfolio History
- Analytics
- Risk

---

### Composed Foundation Value Objects

Price is now an approved architecture, implementation, and contract test reference for future composed Foundation Value Objects.

Money is the final approved composed Financial Value Object and extends the approved composed pattern introduced by Price.

Price

- Approved Architecture
- Approved Implementation
- Approved Contract Test Suite
- Reference Implementation for composed Foundation Value Objects

Money

- Approved Architecture
- Approved Implementation
- Approved Contract Test Suite
- Reference specialization of the composed Foundation Value Object pattern.

The composed Financial Value Object architecture is now complete.

---

### Foundation Family Summary

Identity

✓ Symbol

✓ ExchangeCode

Status

COMPLETE

Measurement

✓ Quantity

✓ Percentage

Status

COMPLETE

Financial

✓ Currency

✓ Price

✓ Money

Status

COMPLETE

Temporal

✓ Timeframe

✓ PointInTime

Status

COMPLETE

Foundation

Status

COMPLETE

---

### Financial Family Progress

The Financial Family implementation is complete and officially approved in the Northstar foundation model.

Current approved members

- Currency
- Price
- Money

Family Root

- Currency

Status

- COMPLETE

Price remains the reference implementation for composed Foundation Value Objects.

Money extends the approved composed pattern and completes the Financial Family.

---

### Measurement Family Completion

The Measurement Family is now complete and officially approved in the Northstar foundation model.

- Quantity v1.0 is the approved family root.
- Percentage v1.0 is the approved second member of the Measurement family.
- Identity is complete with Symbol and ExchangeCode.
- Financial is complete with Currency, Price, and Money.
- Temporal is complete with Timeframe as the approved interval identity root and PointInTime as the approved reusable temporal primitive.

---

### Foundation Completion

The Foundation package is now complete.

Approved Foundation Business Families

- Identity
- Measurement
- Financial
- Temporal

Approved Engineering Patterns

- Identity Pattern
- Measurement Pattern
- Composed Financial Pattern
- Closed Vocabulary Pattern

Reference-First Engineering is now fully established for the Foundation layer.

---

### Registry Review

✓ Identity Family remains complete.

✓ Measurement Family remains complete.

✓ Financial Family is now complete.

✓ Temporal Family is now complete.

✓ PointInTime complements Timeframe as the Reference Temporal Foundation Value Object.

✓ Foundation package is now complete.

✓ Currency remains the Financial Family Root.

✓ Price remains the reference implementation for composed Foundation Value Objects.

✓ Money extends the approved composed pattern.

✓ Engineering References remain authoritative.

✓ Business Families remain correctly classified.

✓ Registry terminology remains consistent.

---

### Why Symbol Became the Reference Value Object

Symbol satisfied every qualifying criterion for designation as a Reference Implementation:

- Immutability is enforced at the language level through `frozen=True`.
- All invariants are expressed in the type and preserved at construction. An invalid `Symbol` cannot be constructed.
- Normalization occurs deterministically before validation, producing a canonical form.
- Validation is explicit, layered, and produces precise domain error messages through a typed exception hierarchy.
- The public API is limited to what the domain requires and nothing more.
- The implementation is independent of all frameworks and infrastructure.
- The contract test suite verifies the full value object behavioral contract, not implementation detail.

---

### Guidance for Future Value Objects

Future value objects — including `Price`, `Money`, `Quantity`, `Currency`, `ExchangeCode`, `Percentage`, `Timeframe`, and `PointInTime` — MUST follow the same implementation style as Symbol unless an ADR explicitly documents and justifies a material deviation.

Deviations MUST be reviewed through both the Architecture Review process and the Engineering Review process before implementation proceeds.

---

### Foundation Architecture Freeze

The Foundation Architecture is now considered stable.

Future architectural changes MUST occur only through approved Architecture Decision Records, Architecture Review, and Engineering Review.

Routine feature implementation MUST NOT modify Foundation Architecture without an approved ADR.

This freeze applies to Business Families, Engineering References, Foundation Value Object responsibilities, and all registry-authoritative architectural guidance.
