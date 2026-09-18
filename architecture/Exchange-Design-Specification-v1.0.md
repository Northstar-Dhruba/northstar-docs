# Exchange Design Specification v1.0

## Business Purpose

Exchange represents the trading venue where market participation occurs.

Exchange exists to provide canonical venue meaning independent of any single Instrument or Listing.

## Business Identity

Exchange identity is venue identity.

ExchangeCode represents canonical venue identity within the Foundation vocabulary and is used by Listing to bind market participation to a specific venue.

## Business Responsibilities

Exchange is responsible for:

- venue identity,
- stable intrinsic descriptive characteristics of the trading venue,
- serving as a market anchor for listings.

Exchange owns only stable intrinsic business characteristics of the trading venue. Operational configuration remains outside Exchange, including:

- trading hours,
- market sessions,
- maintenance windows,
- fee schedules,
- holiday schedules,
- supported order types,
- settlement rules.

Exchange is not responsible for intrinsic instrument identity.

Exchange is not a listing.

## Business Invariants

The following must always be true:

- Every Exchange has canonical venue identity.
- Exchange meaning remains independent of any single Instrument.
- Exchange participates in the domain through Listings.

## Validation Rules

Exchange business validation requires:

- venue identity must exist,
- venue identity must be unambiguous,
- venue meaning must remain distinct from instrument meaning.

## Lifecycle

Exchange lifecycle is conceptual and belongs to the trading venue:

- created,
- active,
- suspended,
- closed,
- retired.

Lifecycle changes describe the business availability and historical status of the venue. They do not define the lifecycle of an Instrument or Listing.

## Relationships

Exchange is related to Listing as the venue side of market-specific manifestation.

Exchange is distinct from Instrument.

Listing composes Exchange with Instrument to represent market participation.

## Examples

Valid business interpretations:

- An Exchange represents a trading venue regardless of which instruments are currently listed.
- Multiple instruments can be associated to the same Exchange through separate Listings.

Invalid business interpretations:

- Treating Exchange as a tradable concept.
- Treating Exchange as a listing record.

## Future Dependency Impact

Exchange is a direct dependency for:

- Listing,

and an indirect dependency for:

- Market Data,
- Order,
- Trade,
- Position,
- Portfolio,
- Analytics,
- Risk.

These indirect dependencies flow through listing-based market participation boundaries.
