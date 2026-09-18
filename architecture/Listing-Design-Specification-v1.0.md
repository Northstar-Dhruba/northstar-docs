# Listing Design Specification v1.0

## Business Purpose

Listing represents the market-specific manifestation of an Instrument on an Exchange.

Listing exists to bridge intrinsic tradable identity and market participation.

## Business Identity

Listing identity is market-specific and derives from the relationship between:

- Instrument,
- Exchange.

Listing identity is not equivalent to Instrument identity and not equivalent to Exchange identity.

## Business Responsibilities

Listing is responsible for:

- market-specific manifestation of an Instrument,
- participation in the Exchange context that owns ExchangeCode,
- trading Currency,
- Listing Status as the approved Core Domain Value Object for Listing lifecycle meaning,
- Tradability as the approved Core Domain Value Object for Listing market-participation capability,
- market-specific attributes.

Listing references Exchange as the canonical owner of ExchangeCode. Listing participates in the Exchange context through that relationship and does not duplicate ExchangeCode.

Listing composes the approved Listing Status and Tradability Core Domain Value Objects. Listing Status expresses Listing lifecycle meaning. Tradability expresses Listing market-participation capability and does not represent Listing lifecycle.

## Business Invariants

The following must always be true:

- Every Listing is associated with exactly one Instrument.
- Every Listing is associated with exactly one Exchange.
- Every Listing has market-specific identity.
- Exchange is the canonical owner of ExchangeCode.
- Every Listing has one trading Currency.
- Every Listing has one Listing Status.
- Every Listing has one Tradability value.
- Listing Status and Tradability remain distinct Core Domain Value Objects.
- Tradability remains consistent with Listing Status.

## Validation Rules

Listing business validation requires:

- Instrument association must exist,
- Exchange association must exist,
- Exchange must provide the canonical ExchangeCode venue identity,
- Currency must exist for market denomination context,
- Listing Status must be explicit,
- Tradability must be explicit,
- Tradability must remain consistent with Listing Status.

## Relationships

Listing composes:

- Instrument,
- Exchange.

Listing owns:

- Currency,
- Listing Status,
- Tradability,
- market-specific attributes.

Listing references Exchange as the canonical owner of ExchangeCode.

Listing is the business boundary referenced by market participation workflows.

## Lifecycle

Listing lifecycle is market-specific and conceptual:

- created,
- active,
- suspended,
- delisted,
- retired.

Lifecycle transitions describe Listing lifecycle meaning, not intrinsic meaning of the Instrument or the market-participation capability expressed by Tradability.

## Examples

Valid business interpretations:

- One Instrument has multiple Listings across different Exchanges.
- One Instrument has different market participation contexts through separate Listings.
- Listing captures denomination, Listing Status, and Tradability for a specific market context.

Invalid business interpretations:

- Treating Instrument as if it owns listing status.
- Treating Exchange as if it defines instrument identity.
- Treating Listing as intrinsic tradable concept identity.

## Future Dependency Impact

Listing is the primary dependency for:

- Market Data,
- Order,
- Trade,
- Position.

Position is the direct dependency of Portfolio. Portfolio, Analytics, and Risk consume Listing-level market context indirectly through Position and other domain concepts.

Listing Status and Tradability are Core Domain Value Objects composed by Listing. ExchangeCode remains owned by Exchange and is accessed through the Listing-to-Exchange relationship.
