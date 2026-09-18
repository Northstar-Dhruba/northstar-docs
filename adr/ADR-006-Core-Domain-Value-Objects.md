# ADR-006: Core Domain Value Objects

## Status

Accepted

## Date

2026-08-16

## Decision Makers

Northstar Architecture Team

## Context

Northstar currently contains the following architectural layers:

Foundation

↓

Foundation Value Objects

↓

Core Domain Entities

The Foundation contains reusable primitive business concepts, including:

- Symbol
- Currency
- Quantity
- Timeframe

Foundation Value Objects provide business meaning that can be reused across domains.

Core Domain Entities compose those primitives into higher-level business concepts with identity, lifecycle, and mutable business behavior.

During Listing design and implementation, two additional business concepts were identified:

- Listing Status
- Tradability

These concepts have business meaning but do not have independent entity identity. They are specific to the Core Domain, reusable within that bounded context, and suitable for composition by Core Domain Entities.

They are neither Foundation Value Objects nor Core Domain Entities. The architecture therefore requires a distinct classification for business concepts that sit between primitive Foundation concepts and Core Domain Entities.

## Decision

Approve a new architectural layer inside the Core Domain:

Foundation Value Objects

↓

Core Domain Value Objects

↓

Core Domain Entities

Core Domain Value Objects:

- express explicit Core Domain business meaning,
- are immutable,
- have no independent entity identity,
- belong only to the Core Domain,
- may compose Foundation Value Objects where appropriate,
- and may be composed by Core Domain Entities.

Core Domain Value Objects are distinct from both Foundation Value Objects and Core Domain Entities.

## Approved Core Domain Value Objects

### Listing Status

Purpose:

Represents the business lifecycle state of a Listing.

Listing Status is approved as a Core Domain Value Object. Its canonical business meaning is Listing lifecycle meaning. Its vocabulary and detailed lifecycle semantics are defined by the approved Listing Status Design Specification.

### Tradability

Purpose:

Represents the business capability of a Listing to participate in market activity.

Tradability is approved as a Core Domain Value Object. Its canonical business meaning is Listing market-participation capability. It is not an independent Boolean concept, and its relationship to Listing Status is defined by the approved Tradability Design Specification.

The approved relationship is:

Listing Status

↓

Listing lifecycle meaning

Tradability

↓

Listing market-participation capability

## Rationale

### Why These Concepts Are Not Foundation Value Objects

Listing Status and Tradability are not Foundation Value Objects because they are:

- specific to the Core Domain,
- specific to market participation,
- specific to Listing semantics,
- and not reusable across unrelated domains as primitive business concepts.

Placing them in Foundation would broaden the Foundation beyond its approved role and would introduce Core Domain meaning into a domain-independent layer.

### Why These Concepts Are Not Entities

Listing Status and Tradability are not Core Domain Entities because they:

- have no independent identity,
- do not own an independent lifecycle,
- do not exist as autonomous business actors,
- and are composed by Listing.

Their purpose is to express business meaning within the Listing boundary, not to become independent Core Domain actors.

## Architectural Boundaries

The approved boundaries remain unchanged:

- Instrument represents intrinsic tradable identity.
- Exchange represents the trading venue.
- Listing represents the market-specific manifestation of an Instrument on an Exchange.
- Listing Status represents Listing lifecycle meaning.
- Tradability represents Listing market-participation capability.

This ADR does not move responsibilities between Instrument, Exchange, and Listing.

## Consequences

### Positive Consequences

- Stronger business modeling.
- Cleaner entity boundaries.
- Reusable Core Domain concepts.
- Reduced duplication.
- Explicit business semantics.
- Clear separation between primitive concepts and entity behavior.

### Implementation Consequences

Listing implementation may compose Listing Status and Tradability only according to their approved design specifications.

This ADR authorizes the architectural classification of both concepts but does not add implementation mechanics.

### Future Consequences

Other Core Domain Value Objects may emerge naturally during future domain design.

Any such concept should be classified according to the same architectural distinction:

- Foundation Value Object when the concept is domain-independent and primitive.
- Core Domain Value Object when the concept has reusable Core Domain meaning without independent identity.
- Core Domain Entity when the concept possesses independent business identity and lifecycle.

Each future classification remains subject to Architecture Review.

## Scope

This ADR establishes architecture only.

It does not:

- modify Instrument,
- modify Exchange,
- modify Listing responsibilities,
- modify Foundation Value Objects,
- or introduce implementation guidance.

## Related Documents

- ADR-004: Composed Foundation Value Objects
- ADR-005: Core Domain Entity Boundaries
- Core Domain Foundation
- Listing Design Specification v1.0
