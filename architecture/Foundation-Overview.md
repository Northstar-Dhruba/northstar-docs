# Foundation Overview

## Purpose

The Foundation package provides the stable platform for all higher-level Northstar domain modeling.

Foundation is now feature-complete and approved for platform-wide reuse.

## Business Families

- Identity
  - Symbol
  - ExchangeCode
- Measurement
  - Quantity
  - Percentage
- Financial
  - Currency
  - Price
  - Money
- Temporal
  - Timeframe

## Engineering Patterns

- Identity Pattern
- Measurement Pattern
- Composed Financial Pattern
- Closed Vocabulary Pattern

## Engineering References

- Symbol v1.0
- Quantity v1.0
- Price v1.0

Engineering references are authoritative and recorded in the Reference Implementation Registry.

## Reference-First Engineering

Northstar follows Reference-First Engineering:

- Business Families classify domain meaning.
- Engineering References define implementation inheritance.
- Approved references are reused unless an ADR authorizes deviation.

## Foundation Completion

Foundation completion means:

- all four families are approved,
- reference contracts are established,
- value object semantics are stable,
- and Core Domain development can proceed on top of Foundation.
