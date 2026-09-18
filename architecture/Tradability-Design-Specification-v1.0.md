# Tradability Design Specification v1.0

## 1. Business Purpose

Tradability represents the business capability of a Listing to participate in market activity according to approved business rules.

It exists to express whether a market-specific manifestation of an Instrument on an Exchange is currently permitted to participate in relevant market workflows.

Tradability is a Core Domain Value Object because it:

- expresses explicit Core Domain business meaning,
- is immutable,
- has no independent identity,
- is composed by Listing,
- and is reusable by Core Domain business rules.

Tradability is not:

- a Foundation Value Object,
- an Entity,
- a raw Boolean,
- or an implementation enumeration.

## 2. Business Meaning

Tradability expresses the current market-participation capability of a Listing.

It answers the business question:

> Is this Listing permitted to participate in market activity under the applicable business rules?

Tradability is distinct from:

- Instrument lifecycle,
- Exchange lifecycle,
- Listing lifecycle,
- Price,
- Money,
- Market Data,
- Order execution,
- Position state,
- Portfolio state.

Tradability describes capability, not activity or execution outcome.

## 3. Relationship to Listing Status

Listing Status expresses Listing lifecycle meaning.

Tradability expresses Listing market-participation capability.

These concepts are related but not interchangeable.

### Tradability Derived Directly from Listing Status

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

Under this model, Tradability could vary independently of Listing Status.

Advantages:

- maximum flexibility,
- separates capability decisions from lifecycle meaning.

Disadvantages:

- permits contradictory combinations,
- weakens the authority of Listing Status,
- increases ambiguity for Market Data, Orders, and Trades,
- makes downstream interpretation more difficult.

### Approved Relationship

Tradability is constrained by Listing Status.

Listing Status remains the authoritative Listing lifecycle context. Tradability expresses whether market participation is permitted within that Listing lifecycle context.

Tradability is not required to be mechanically derived from Listing Status. This preserves future extensibility while maintaining business consistency.

## 4. Canonical Business Vocabulary

Tradability has two canonical business values:

- Permitted
- Not Permitted

### Permitted

The Listing is capable of participating in market activity under the applicable business rules.

### Not Permitted

The Listing is not currently capable of participating in market activity under the applicable business rules.

These values express business capability rather than a technical Boolean condition.

Additional Tradability values require a separate Architecture Review if future business rules require distinctions such as conditional, restricted, or partially permitted participation.

## 5. Business Responsibilities

Tradability is responsible for:

- expressing Listing market-participation capability,
- communicating whether market participation is currently permitted,
- supporting downstream business rules,
- providing a consistent business value for Listing participation decisions.

Tradability does not own:

- Instrument identity,
- Exchange identity,
- Listing identity,
- Listing Status,
- Price,
- Money,
- Market Data,
- Orders,
- Trades,
- Positions,
- Portfolio state,
- or execution activity.

## 6. Business Invariants

The following invariants must always hold:

- Every Listing has exactly one Tradability value.
- Every Tradability value represents one business capability state.
- Tradability belongs to the finite approved vocabulary.
- Tradability is immutable as a Core Domain Value Object.
- Tradability has no independent identity.
- Tradability remains specific to Listing market participation.
- Tradability remains consistent with Listing Status.
- Tradability does not redefine Listing lifecycle meaning.
- Tradability does not represent completed market activity or execution outcome.

## 7. Validation Rules

Business validation requires:

- the value must belong to the approved Tradability vocabulary,
- the value must represent one market-participation capability,
- the value must remain specific to Listing,
- the value must remain consistent with Listing Status,
- and the value must not imply activity or execution has occurred.

A Tradability value is invalid when it is:

- unknown,
- ambiguous,
- outside the approved vocabulary,
- unrelated to market-participation capability,
- or inconsistent with the Listing lifecycle context.

## 8. Normalization

Tradability requires canonical normalization.

Normalization must ensure:

- one canonical spelling for each approved value,
- one canonical representation for each business capability,
- no ambiguity caused by alternate presentation,
- and preservation of the distinction between `Permitted` and `Not Permitted`.

Normalization must not create new capability states or merge distinct future states.

## 9. Equality

Tradability equality is based on business value.

Two Tradability values are equal when they represent the same canonical market-participation capability.

Examples:

- `Permitted` equals `Permitted`.
- `Not Permitted` equals `Not Permitted`.
- `Permitted` does not equal `Not Permitted`.

Equality does not depend on:

- the Listing containing the value,
- object identity,
- presentation formatting,
- Instrument,
- Exchange,
- or Listing Status identity.

## 10. Examples

### Valid Examples

- `Permitted`: an Active Listing is allowed to participate in market activity.
- `Not Permitted`: a Suspended Listing is not allowed to participate in market activity.
- `Not Permitted`: a Delisted Listing is no longer eligible for current market participation.
- `Not Permitted`: a Retired Listing remains historical and is not eligible for active participation.

The Listing Status and Tradability combination must remain consistent with approved business rules.

### Invalid Examples

- `Unknown`: not part of the approved vocabulary.
- `Maybe`: ambiguous capability meaning.
- `Trading`: describes activity rather than capability.
- `Executed`: describes an outcome rather than Listing capability.
- `Open`: ambiguous and not an approved Tradability value.
- A Tradability value that permits participation for a Listing whose approved lifecycle context prohibits participation.

## 11. Future Dependency Impact

Listing directly depends on Tradability to express its market-participation capability.

Market Data may use Tradability to interpret whether observations relate to a Listing that is currently permitted to participate.

Orders depend on Tradability when determining whether execution intent may target a Listing.

Trades may preserve the Tradability context associated with the Listing at the time of execution.

Position may consume Tradability indirectly through Listing and trade-related market context.

Portfolio consumes Tradability indirectly through Positions and related domain concepts.

Tradability does not execute orders, produce Market Data, record Trades, or manage Portfolio state.

## 12. Boundary Review

Tradability owns only:

- business capability for market participation.

Tradability does not own:

- Instrument,
- Exchange,
- Listing,
- Listing Status,
- Price,
- Money,
- Market Data,
- Orders,
- Trades,
- Positions,
- or Portfolio.

Tradability remains a Core Domain Value Object. It has no independent identity and does not become an Entity.

## 13. DDD Review

Tradability satisfies the Core Domain Value Object classification because it provides:

- explicit business meaning,
- immutable value semantics,
- no independent identity,
- high cohesion around Listing participation capability,
- low coupling to unrelated domain concepts,
- consistent ubiquitous language,
- and reusable composition within Core Domain rules.

Its relationship to Listing Status is explicit:

- Listing Status provides Listing lifecycle context.
- Tradability provides market-participation capability.
- Listing Status constrains Tradability.
- Neither concept replaces the other.

## Approval Summary

Tradability is an approved Core Domain Value Object representing Listing market-participation capability.

Its canonical values are:

- Permitted
- Not Permitted

Tradability is constrained by Listing Status and must remain consistent with Listing lifecycle meaning.
