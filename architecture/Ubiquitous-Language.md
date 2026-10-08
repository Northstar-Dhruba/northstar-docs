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

## Options Vocabulary

Governed by ADR-010: Option Contract Identity.

### Option Product

Business Meaning:
One exchange-defined option specification, of which every individual option contract is an instance.

Purpose:
Identify the option family a contract belongs to, independently of any futures product with the same exchange code.

Represents:
A product code and the exchange that defines it, for example NIFTY@NSE, together with the underlying it is written on.

Does NOT Represent:
A futures product, an individual contract, an expiry series, a lot size, an exercise or settlement style, or a provider symbol.

Examples:
NIFTY@NSE on NIFTY50.

Relationship to Other Options Concepts:
Every Option Contract belongs to exactly one Option Product. The underlying belongs to the Option Product, not to the contract.

### Option Right

Business Meaning:
The right an option contract grants its holder: CALL, the right to buy the underlying at the strike, or PUT, the right to sell it.

Purpose:
Distinguish calls from puts within one product, expiry and strike.

Represents:
Exactly CALL or PUT.

Does NOT Represent:
Exchange or provider spellings such as CE and PE, an order direction, a position direction, or an exercise style.

Examples:
CALL, PUT.

Relationship to Other Options Concepts:
Option Right is one of the four components of Option Contract identity.

### Option Strike

Business Meaning:
The level at which an option's right is struck, fixed when the contract is listed.

Purpose:
Distinguish contracts of one product, expiry and right by where they are struck.

Represents:
A strictly positive decimal expressed in the underlying's quotation convention, for example NIFTY index points.

Does NOT Represent:
A Price, Money, an observed market quotation, an option premium, or a currency amount.

Examples:
25000, 24950.5.

Relationship to Other Options Concepts:
Option Strike is one of the four components of Option Contract identity.

### Option Contract

Business Meaning:
One individual, exactly dated option contract.

Purpose:
Provide the canonical identity of what an option holding, observation or decision refers to.

Represents:
Exactly an Option Product, an expiration date, an Option Strike and an Option Right.

Does NOT Represent:
A futures contract, a weekly or monthly classification, a lot size, an underlying, an exercise or settlement style, or a provider instrument key or trading symbol.

Examples:
NIFTY@NSE 2026-10-27 25000 CALL.

Relationship to Other Options Concepts:
Two option contracts that differ in product, expiration date, strike or right are different contracts.

### Option Premium

Business Meaning:
The observed market quotation of one option contract.

Purpose:
Express what an option contract traded or was quoted at, in premium points.

Represents:
A decimal of zero or greater in premium points. Zero is valid.

Does NOT Represent:
An Option Strike, a Price, Money, a futures quotation, or a currency amount.

Examples:
182.35, 0.

Relationship to Other Options Concepts:
Option Premium is converted into settlement currency only through an Option Point Value. Governed by ADR-011.

### Option Point Value

Business Meaning:
The settlement-currency value of a 1.0 premium-point move on one option contract.

Purpose:
Provide the one rate that converts option premium movements into settlement currency.

Represents:
A strictly positive amount and its settlement Currency, per premium point, per option contract. The contract's lot size is folded into it.

Does NOT Represent:
A lot size or multiplier on its own, a Price, Money, a futures point value, or a margin requirement.

Examples:
65 INR/premium-point/contract.

Relationship to Other Options Concepts:
Each Option Point Value belongs to exactly one Option Contract through Option Contract Economics. Governed by ADR-011.

### Option Contract Economics

Business Meaning:
The economic facts of one exact option contract.

Purpose:
State, for one Option Contract, the rate at which its premium movements are worth settlement currency.

Represents:
Exactly one Option Contract and its Option Point Value. The settlement currency is read from the point value.

Does NOT Represent:
Product-level economics, a lot size field, fees, taxes, margin, or a profit and loss calculation.

Examples:
NIFTY@NSE 2026-10-27 25000 CALL 65 INR/premium-point/contract.

Relationship to Other Options Concepts:
Option Contract Economics are keyed by the complete Option Contract. Neighbouring strikes, the other right, other expiries and other products never share them. Governed by ADR-011.

### Option Expiration Rule

Business Meaning:
An exchange rule, in force from a stated date, that names the day on which an option product's weekly and monthly contracts expire.

Purpose:
Derive expiration dates from the exchange's published rule rather than assuming a timeless weekday.

Represents:
One product, the date from which the rule applies, the weekly and monthly expiry weekdays, the Expiry Adjustment, and the primary exchange source.

Does NOT Represent:
A listing cycle, a strike scheme, a lot size, a provider instrument, or proof that any contract was listed.

Examples:
NIFTY@NSE from 2025-09-01: weekly on the Tuesday of the expiry week, monthly on the last Tuesday of the month (NSE/FAOP/68747).

Relationship to Other Options Concepts:
An Option Expiration Rule gives the Nominal Expiry Date; the Expiry Adjustment turns it into the expiration date. No rule applies to a period before its effective date. Governed by ADR-012.

### Nominal Expiry Date

Business Meaning:
The date an Option Expiration Rule names for a period, before any holiday adjustment.

Purpose:
Keep the rule's own answer visible when the actual expiration date has been adjusted.

Represents:
The Tuesday of an ISO expiry week, or the last Tuesday of an expiry month, under the current NIFTY@NSE rule.

Does NOT Represent:
The actual exchange expiration date, a trading-day judgement, or an Option Contract's identity.

Examples:
2026-11-24, the nominal monthly expiry for November 2026.

Relationship to Other Options Concepts:
The Nominal Expiry Date and the expiration date differ exactly when an Expiry Adjustment was applied. Governed by ADR-012.

### Expiry-Eligible Trading Day

Business Meaning:
A date on which an option expiration may fall under the current rule.

Purpose:
Decide where an Expiry Adjustment stops.

Represents:
A Monday to Friday date in a loaded NSE calendar year that is neither a published trading holiday nor a published special session.

Does NOT Represent:
A claim about special sessions. A date with a published special session is not treated as eligible or ineligible: reaching one makes expiration resolution fail closed, because its expiration treatment is not yet established by a primary exchange source. A date in an unloaded calendar year likewise cannot be judged and fails closed.

Examples:
2026-11-23 is an Expiry-Eligible Trading Day; 2026-11-24 (a trading holiday) is not; reaching 2025-10-21 (a holiday with a Muhurat session) stops resolution with an error.

Relationship to Other Options Concepts:
The expiration date is the first Expiry-Eligible Trading Day on or before the Nominal Expiry Date. Governed by ADR-012.

### Expiry Adjustment

Business Meaning:
Moving an expiry from its Nominal Expiry Date to the previous Expiry-Eligible Trading Day when the nominal date is not one.

Purpose:
Apply the exchange's holiday rule deterministically.

Represents:
A backward walk over weekends and trading holidays only, which fails closed on reaching a special session or an unloaded calendar year.

Does NOT Represent:
A forward move, a provider's listing decision, or a change to an Option Contract's identity.

Examples:
Nominal 2026-03-31 (Shri Mahavir Jayanti) adjusted to 2026-03-30.

Relationship to Other Options Concepts:
An adjusted expiry reports both its Nominal Expiry Date and its expiration date. Governed by ADR-012.

### Option Trading Session

Business Meaning:
One exchange trading session of an option product, named by its trading date and bounded by its normal-market open and close.

Purpose:
Decide which daily option bars exist and the instant at which each is stamped.

Represents:
A trading date and the open and close instants of the option normal market on that date, from a cited option session regime.

Does NOT Represent:
A futures session, a futures pre-open, a special session whose option timings are not established, or a provider timestamp.

Examples:
NIFTY@NSE on 2026-10-08: 09:15 to 15:40 IST.

Relationship to Other Options Concepts:
An Option Daily Bar is stamped at its Option Trading Session's close. Governed by ADR-014.

### Option Native Daily Observation

Business Meaning:
One daily candle a market-data source reports for one exact Option Contract, labelled by its trading date.

Purpose:
Carry what a source knows about a session before Northstar stamps it as a canonical bar.

Represents:
An Option Contract, a trading date, open, high, low and close Option Premiums, and a volume in option contracts.

Does NOT Represent:
An instant, a provider identifier, open interest, or a finality judgement.

Examples:
NIFTY@NSE 2026-10-27 25000 CALL on 2026-10-08: O=182.35 H=190 L=175.5 C=186.1 V=1200.

Relationship to Other Options Concepts:
Each Option Native Daily Observation becomes one Option Daily Bar when its trading date matches a resolved Option Trading Session. Governed by ADR-014.

### Option Daily Bar

Business Meaning:
The canonical daily market record of one exact Option Contract.

Purpose:
Hold the open, high, low and close premiums and the volume of one option session as immutable market data.

Represents:
An Option Contract, the close instant of its Option Trading Session, the daily timeframe, four Option Premiums and a whole-number volume of option contracts.

Does NOT Represent:
Open interest, bid or ask, a settlement price, implied volatility, Greeks, provider metadata, or a claim that the provider will not revise it.

Examples:
NIFTY@NSE 2026-10-27 25000 CALL at 2026-10-08T10:10:00Z, 1d.

Relationship to Other Options Concepts:
An Option Daily Bar is identified by its Option Contract, instant and timeframe; a neighbouring strike, the other right or another expiry never shares a bar. Governed by ADR-014.
