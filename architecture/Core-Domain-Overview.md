# Core Domain Overview

## Core Domain Overview

The Core Domain models tradable financial concepts and their market participation.

It establishes the business concepts that connect intrinsic tradable meaning to market-specific behavior used by downstream workflows.

## Approved Core Domain Entities

### Instrument

Instrument represents intrinsic tradable identity.

It provides the canonical business concept for what is being traded, observed, and analyzed, independent of any specific market venue.

### Exchange

Exchange represents the trading venue.

It provides canonical venue meaning independent of any specific instrument manifestation.

### Listing

Listing represents market-specific participation.

It is the market-specific manifestation of an Instrument on an Exchange and is the bridge between intrinsic identity and market context.

## Approved Dependency Graph

Instrument Exchange
\ /
\ /
▼ ▼
Listing
│
▼
Market Data
│
▼
Order
│
▼
Trade
│
▼
Position
│
▼
Portfolio
│
▼
Analytics / Risk

## Architectural Interpretation

Instrument and Exchange are parallel Core Domain roots.

Listing is the approved bridge between intrinsic business identity and market participation.

Downstream market-context workflows reference Listing where venue and denomination semantics matter.
