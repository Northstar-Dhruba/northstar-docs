# Ubiquitous Language

## Purpose

This document defines the authoritative shared business vocabulary for the Northstar platform.

Each approved business concept has one canonical meaning.
All future domain models should use these terms consistently.

## Domain Language Principles

- Northstar follows Domain-Driven Design.
- Business terminology is authoritative.
- Implementation follows business language.
- Business concepts should not have multiple names.
- Each approved business concept should have one canonical definition.
- New terminology should be introduced only after Architecture Review.

## Foundation Vocabulary

### Identity

#### Symbol

Business Meaning:
Canonical identifier used to uniquely identify a business concept.

Purpose:
Provide a stable and unambiguous identity label.

Represents:
A canonical business identifier.

Does NOT Represent:
A measurement, a financial amount, or a time interval.

Examples:
AAPL, NIFTY50, BTCUSDT.

Relationship to Other Foundation Concepts:
Symbol is an Identity concept. It is distinct from ExchangeCode (venue identity), Quantity and Percentage (measurement), Currency/Price/Money (financial), and Timeframe (temporal interval identity).

#### ExchangeCode

Business Meaning:
Canonical identifier representing a trading venue.

Purpose:
Provide a stable and unambiguous venue identity.

Represents:
A trading venue identifier.

Does NOT Represent:
An instrument identifier, a financial value, or a time interval.

Examples:
NYSE, NASDAQ, NSE.

Relationship to Other Foundation Concepts:
ExchangeCode is an Identity concept specific to venue identity. It complements Symbol, which identifies business concepts such as instruments.

### Measurement

#### Quantity

Business Meaning:
A measurable amount.

Purpose:
Express magnitude for measurable business concepts.

Represents:
A measurable amount value.

Does NOT Represent:
Identity, denomination, monetary value, or temporal interval identity.

Examples:
1, 10.5, 250.

Relationship to Other Foundation Concepts:
Quantity is a Measurement concept. It differs from Percentage, which expresses relative measurement.

#### Percentage

Business Meaning:
A relative measurement expressed in percentage units.

Purpose:
Express proportional or relative magnitude.

Represents:
A percentage-based relative value.

Does NOT Represent:
Absolute quantity, identity, monetary denomination, or temporal interval identity.

Examples:
5%, 12.5%, -2%.

Relationship to Other Foundation Concepts:
Percentage is a Measurement concept related to Quantity but used for relative, not absolute, measurement.

### Financial

#### Currency

Business Meaning:
Financial denomination.

Purpose:
Identify the denomination context for financial values.

Represents:
A financial denomination identifier.

Does NOT Represent:
A monetary amount, valuation, or account state.

Examples:
USD, EUR, INR.

Relationship to Other Foundation Concepts:
Currency provides denomination context for Price and Money.

#### Price

Business Meaning:
Monetary valuation expressed in a specific Currency.

Purpose:
Express valuation in a denomination-aware form.

Represents:
A monetary valuation tied to a Currency.

Does NOT Represent:
A general measurement, an account state, or a temporal interval identity.

Examples:
150 USD, 72.5 EUR.

Relationship to Other Foundation Concepts:
Price depends on Currency for denomination and remains distinct from Quantity and Percentage.

#### Money

Business Meaning:
Monetary value within a financial context.

Purpose:
Express financial value used in contexts such as balances, obligations, and reserves.

Represents:
A denomination-aware financial value in context.

Does NOT Represent:
A pure valuation quote, a general measurement, or temporal interval identity.

Examples:
100 USD, -50 USD, 0 EUR.

Relationship to Other Foundation Concepts:
Money shares denomination semantics with Currency and valuation semantics with Price, but carries broader financial-context meaning.

### Temporal

#### Timeframe

Business Meaning:
Standardized temporal interval identifier.

Purpose:
Provide a canonical interval label used across temporal business contexts.

Represents:
A standardized interval identity.

Does NOT Represent:
A timestamp, date, timezone, calendar object, or scheduling object.

Examples:
1m, 5m, 1h, 1d, 1w, 1M, 1Y.

Relationship to Other Foundation Concepts:
Timeframe is a Temporal concept and is distinct from Identity, Measurement, and Financial concepts.

## Core Domain Vocabulary

### Instrument

Business Meaning:
Intrinsic tradable financial concept.

Purpose:
Provide the canonical business concept for what is traded, observed, or analyzed.

Represents:
Intrinsic tradable identity and descriptive business meaning.

Does NOT Represent:
An Exchange, Listing, price, money, portfolio, position, order, or trade.

Relationship to Other Core Domain Concepts:
Instrument may be manifested through one or many Listings and remains independent of market-specific participation.

### Exchange

Business Meaning:
Trading venue.

Purpose:
Provide canonical venue meaning for market participation.

Represents:
Venue identity and venue business meaning.

Does NOT Represent:
An Instrument or Listing.

Relationship to Other Core Domain Concepts:
Exchange participates in the Core Domain through Listings, which connect venue context to an Instrument.

### Listing

Business Meaning:
Market-specific manifestation of an Instrument on an Exchange.

Purpose:
Bridge intrinsic tradable identity and market-specific participation.

Represents:
ExchangeCode, Currency, listing status, tradability, and market-specific attributes in the context of one Instrument and one Exchange.

Does NOT Represent:
The intrinsic Instrument or the Exchange independently.

Relationship to Other Core Domain Concepts:
Listing connects one Instrument to one Exchange and provides the market context used by Market Data, Orders, Trades, and Positions.

### Portfolio

Definition pending Core Domain Design.

### Position

Definition pending Core Domain Design.

### Order

Definition pending Core Domain Design.

### Trade

Definition pending Core Domain Design.

### Market Data

Definition pending Core Domain Design.

### Risk

Definition pending Core Domain Design.

### Analytics

Definition pending Core Domain Design.
