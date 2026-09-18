# Instrument Design Specification v1.0

## Business Purpose

Instrument represents the intrinsic tradable business concept used by the platform to refer to what is being traded, observed, or analyzed.

Instrument exists to provide a stable canonical concept that remains consistent across markets.

## Business Identity

Instrument identity is intrinsic to the tradable concept itself.

Instrument identity is not market-specific and does not include venue or listing participation attributes.

## Business Responsibilities

Instrument is responsible for:

- intrinsic tradable identity,
- canonical business reference for the tradable concept,
- descriptive business meaning required to distinguish one tradable concept from another.

Instrument is not responsible for market-specific behavior.

## Business Invariants

The following must always be true:

- Every Instrument has intrinsic business identity.
- Every Instrument remains a canonical tradable concept independent of market participation.
- Instrument meaning remains stable even when market listings change.

## Relationships

Instrument composes foundational identity meaning through Symbol as intrinsic concept identity.

Instrument does not own ExchangeCode or Currency.

Instrument is related to Listing as the underlying tradable concept that may be manifested in one or many market-specific listings.

Instrument is distinct from Exchange and Listing.

## Validation Rules

Instrument business validation requires:

- intrinsic identity must exist,
- intrinsic identity must be unambiguous,
- descriptive meaning must be coherent with the tradable concept.

Validation does not evaluate market-specific listing context.

## Future Dependency Impact

Instrument is the intrinsic reference concept used directly by:

- Listing,

and indirectly by:

- Market Data,
- Order,
- Trade,
- Position,
- Portfolio,
- Analytics,
- Risk.

Downstream dependencies consume Instrument through listing relationships rather than embedding listing semantics into Instrument.
