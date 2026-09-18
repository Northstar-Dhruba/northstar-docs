# Listing Status Design Specification v1.0

## 1. Business Purpose

Listing Status represents the business lifecycle state of a Listing.

It exists to communicate the current business state and Listing lifecycle context of a Listing throughout its lifecycle.

Listing Status is a Core Domain Value Object because it:

- has explicit Core Domain business meaning,
- is immutable,
- has no independent entity identity,
- is composed by Listing,
- and is reusable within Core Domain business rules.

Listing Status is not:

- a Foundation Value Object,
- an Entity,
- or an implementation enumeration.

## 2. Business Meaning

Listing Status expresses the lifecycle state of a Listing.

It communicates whether a market-specific manifestation of an Instrument on an Exchange has been created, is active, temporarily restricted, removed from active listing, or retired from active domain use.

Listing Status does not describe:

- Instrument lifecycle,
- Exchange lifecycle,
- trading activity,
- order execution,
- Market Data,
- Portfolio state,
- or Position state.

## 3. Canonical Business Vocabulary

The approved Listing Status vocabulary is:

- Created
- Active
- Suspended
- Delisted
- Retired

This vocabulary is sufficient for the currently approved Listing lifecycle.

### Created

The Listing has been established as a recognized market-specific manifestation of an Instrument on an Exchange.

### Active

The Listing is available within its approved Listing lifecycle context for normal market participation.

### Suspended

The Listing continues to exist, but its Listing lifecycle context is temporarily restricted from normal market participation.

### Delisted

The Listing is no longer active on the Exchange, while its historical business meaning remains preserved.

### Retired

The Listing is no longer relevant to active domain workflows and is retained only as historical business context.

No additional state is required by the approved architecture.

No proposed state should be added unless a future business requirement demonstrates that the current vocabulary cannot express a distinct Listing lifecycle meaning.

No existing state should be removed because each represents a distinct approved lifecycle condition.

## 4. Business Responsibilities

Listing Status is responsible for:

- expressing Listing lifecycle meaning,
- communicating Listing lifecycle context,
- supporting downstream business rules that depend on Listing lifecycle state,
- distinguishing active, temporarily restricted, inactive, and historical Listing conditions.

Listing Status does not own:

- Instrument identity,
- Exchange identity,
- Listing identity,
- Price,
- Money,
- Market Data,
- Orders,
- Trades,
- Positions,
- Portfolio state,
- or trading activity.

## 5. Business Invariants

The following invariants must always hold:

- Every Listing has exactly one Listing Status.
- Every Listing Status represents exactly one business lifecycle state.
- Listing Status belongs to the finite approved vocabulary.
- Listing Status has stable canonical meaning.
- Listing Status is immutable as a Core Domain Value Object.
- Listing Status has no independent entity identity.
- Listing Status does not redefine Instrument, Exchange, or Listing meaning.
- Listing Status remains specific to Listing lifecycle semantics.

## 6. Relationship to Tradability

Listing Status expresses Listing lifecycle meaning.

Tradability expresses Listing market-participation capability.

These concepts are related but not interchangeable.

### Tradability Derived from Listing Status

Under this model, each Listing Status would determine Tradability completely.

Advantages:

- avoids contradictory combinations,
- centralizes lifecycle interpretation,
- simplifies downstream reasoning.

Disadvantages:

- limits the model when market participation rules become more nuanced,
- assumes every lifecycle state has one unavoidable capability outcome,
- couples capability semantics too tightly to lifecycle semantics.

### Tradability Constrained by Listing Status

Under this model, Listing Status provides the Listing lifecycle context, while Tradability expresses the market-participation capability within that context.

Advantages:

- preserves the distinct meaning of lifecycle and capability,
- prevents Tradability from contradicting Listing Status,
- supports future business rules that affect participation without redefining lifecycle,
- maintains clear DDD boundaries.

Disadvantages:

- requires explicit business rules to define valid combinations,
- may require future policy concepts for more complex market conditions.

### Tradability Independent from Listing Status

Under this model, Listing Status and Tradability could change independently.

Disadvantages:

- permits contradictory combinations,
- weakens the authority of Listing Status,
- increases ambiguity for Market Data, Orders, and Trades,
- makes downstream interpretation more difficult.

### Approved Relationship

Tradability is constrained by Listing Status.

Listing Status remains the authoritative Listing lifecycle context. Tradability expresses whether market participation is permitted within that Listing lifecycle context.

Tradability is not required to be mechanically derived from Listing Status. This preserves future extensibility while maintaining business consistency.

The exact Tradability vocabulary and detailed relationship rules are defined by the approved Tradability Design Specification.

## 7. Validation Rules

Business validation requires:

- the status must belong to the approved vocabulary,
- the status must represent one valid Listing lifecycle state,
- the status must not combine multiple lifecycle meanings,
- the status must remain specific to Listing lifecycle semantics.

Values that are unknown, ambiguous, or outside the approved vocabulary are invalid.

## 8. Normalization

Listing Status requires canonical normalization.

Normalization must ensure:

- one canonical spelling for each approved state,
- one canonical representation for each business value,
- no ambiguity caused by alternate presentation,
- and preservation of the approved business meaning.

Normalization must not create new lifecycle states or merge distinct states.

## 9. Equality

Listing Status equality is based on business value.

Two Listing Status values are equal when they represent the same canonical lifecycle state.

Different lifecycle states are not equal.

Equality does not depend on:

- the Listing that contains the status,
- presentation differences,
- object identity,
- or unrelated market concepts.

## 10. Examples

### Valid Examples

- `Created` represents a newly established Listing.
- `Active` represents a Listing available within its approved Listing lifecycle context.
- `Suspended` represents a temporarily restricted Listing lifecycle context.
- `Delisted` represents a Listing removed from active venue participation.
- `Retired` represents a Listing retained for historical context only.

The Listing Status and Tradability combination must remain consistent with approved business rules.

### Invalid Examples

- `Pending Approval`: not part of the approved vocabulary.
- `Trading`: describes activity rather than Listing lifecycle.
- `Open`: ambiguous and not an approved Listing Status.
- `Closed`: potentially ambiguous with Exchange lifecycle and therefore not an approved Listing Status.
- `Active and Suspended`: combines multiple lifecycle states.
- `USD`: represents Currency, not Listing lifecycle.

## 11. Future Dependency Impact

Listing directly depends on Listing Status to express its lifecycle state.

Tradability depends conceptually on Listing Status as its Listing lifecycle context. The exact relationship is defined by the approved Tradability Design Specification.

Market Data may use Listing Status to interpret whether observations belong to an active, suspended, delisted, or historical Listing.

Orders may use Listing Status as part of determining whether a Listing is eligible for execution workflows.

Trades may use Listing Status to preserve the historical lifecycle context associated with an execution.

Portfolio and Position concepts may consume Listing Status indirectly through Listing and related market participation records.

## 12. Boundary Review

Listing Status owns only:

- Listing lifecycle meaning.

Listing Status does not own:

- Instrument,
- Exchange,
- Listing,
- Price,
- Money,
- Market Data,
- Orders,
- Trades,
- Positions,
- Portfolio,
- or Tradability semantics.

Listing Status remains a Core Domain Value Object. It has no independent identity and does not become an Entity.

## 13. DDD Review

Listing Status satisfies the Core Domain Value Object classification because it provides:

- explicit business meaning,
- immutable value semantics,
- no independent identity,
- high cohesion around Listing lifecycle,
- low coupling to unrelated domain concepts,
- consistent Northstar ubiquitous language,
- and reusable composition within Core Domain rules.

Its boundary remains narrower than Listing and more domain-specific than Foundation Value Objects.

## Approval Summary

Listing Status is the first approved Core Domain lifecycle Value Object.

Its canonical vocabulary is:

- Created
- Active
- Suspended
- Delisted
- Retired

Listing Status is the authoritative expression of Listing lifecycle meaning.

Tradability remains a separate Core Domain Value Object representing Listing market-participation capability and constrained by Listing Status.
