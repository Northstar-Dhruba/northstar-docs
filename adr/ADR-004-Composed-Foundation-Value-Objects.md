# ADR-004: Composed Foundation Value Objects

Status

Accepted

Date

2026-08-15

Decision Makers

Northstar Architecture Team

## Context

The original architecture proposed that Price be modeled as:

Price = Quantity + Currency

Architecture Review determined that this weakened the Financial Family by treating financial value as a measurement concept.

The business domain distinguishes between measurement and financial value. Quantity represents measurement. Price represents financial value. Those are different business concepts and should not be conflated.

## Decision

Price represents:

Monetary Amount + Currency

Price therefore becomes the first composed Foundation Value Object.

This decision preserves a clear boundary between:

- Measurement Value Objects
- Identity Value Objects
- Composed Financial Value Objects

The business architecture remains implementation-independent.

## Rationale

The approved business meaning of Price is not that it is a measured quantity in a denomination. The approved business meaning is that it is a monetary value expressed in a specific denomination.

This clarifies the Financial Family and preserves domain semantics:

- Quantity is a measurement concept.
- Currency is a denomination concept.
- Monetary Amount is a financial magnitude concept.
- Price is the composed financial value that joins magnitude and denomination.

This approach improves conceptual clarity, supports business-level reasoning, and keeps implementation decisions outside the architectural specification.

## Consequences

### Positive Consequences

- Financial value remains distinct from measurement.
- Business specifications remain implementation-independent.
- Implementation details remain deferred to implementation review.
- Money can extend the composed Financial Value Object pattern.
- Foundation architecture now clearly contains three architectural patterns:
  - Identity Value Objects
  - Measurement Value Objects
  - Composed Financial Value Objects

### Negative Consequences

- The architectural boundary between measurement and financial value must remain explicit.
- Future value objects must be checked for semantic clarity before approval.
- The model requires careful domain separation to avoid reintroducing measurement-first thinking.

## Trade-offs

The selected model favors business correctness over general numerical reuse. This reduces ambiguity and ensures that financial semantics remain specific to the Financial Family rather than being treated as a specialized measurement concept.

## Future Considerations

This ADR establishes the approved foundation for composed Financial Value Objects. Future composed value objects, including Money, should follow the same business architecture unless an ADR documents an approved deviation.

## Related Documents

- Reference Implementation Registry
- Price Value Object Design Specification
- Financial Family architecture documentation
