# ADR-005: Core Domain Entity Boundaries

Status

Accepted

Date

2026-08-16

Decision Makers

Northstar Architecture Team

## Context

The initial Instrument design combined intrinsic tradable identity with market-specific identity.

Architecture Review determined that these are separate business concepts with different responsibilities and different rates of change.

When market-specific concerns are merged into Instrument, the boundary between intrinsic concept identity and market participation becomes ambiguous.

## Decision

Instrument represents intrinsic tradable identity.

Exchange represents the trading venue.

Listing represents the market-specific manifestation of an Instrument.

ExchangeCode and Currency belong to Listing.

The approved relationship is:

Instrument
│
│
▼
Listing
▲
│
Exchange

## Rationale

This boundary reflects real-world market structure:

- one tradable concept can appear across multiple venues,
- venue-specific participation can vary independently,
- and trading denomination can vary by market context.

Separating Instrument from Listing preserves one canonical intrinsic concept while allowing market-specific manifestations to evolve independently.

## Consequences

### Positive Consequences

- Instrument remains stable across markets.
- One Instrument may have one or many Listings.
- Market Data references Listing.
- Orders reference Listing.
- Trades reference Listing.
- Positions reference Listing.
- Portfolio depends on Positions.
- Future Core Domain entities inherit these boundaries.

### Negative Consequences

- Core Domain modeling must keep intrinsic identity and market participation explicitly separated.
- Listing becomes a first-class entity earlier in the roadmap.

## Trade-offs

The approved model favors long-term boundary clarity and business correctness over short-term simplification.

## Related Documents

- Instrument Design Specification v1.0
- Exchange Design Specification v1.0
- Listing Design Specification v1.0
- Core Domain Roadmap
