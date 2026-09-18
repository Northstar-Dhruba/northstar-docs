# Price Value Object Design Specification

## 1. Relationship to Quantity

Price is not a measurement and is not composed from Quantity.

This is the approved architectural decision.

Price represents a monetary amount expressed in a specific Currency.

Quantity represents measurement.

These are different business concepts.

Price therefore does not inherit its business meaning from Quantity. The business architecture is intentionally separated:

- Quantity answers the question: "How much in a measurable sense?"
- Price answers the question: "What is the monetary value in a denomination?"

This distinction is part of the approved Financial Family semantics.

## 2. Relationship to Currency

Price is a financial value object that depends on Currency for denomination.

Currency provides the business meaning of the unit in which the price is expressed.

The business relationship is:

- Monetary Amount provides financial magnitude.
- Currency provides denomination.
- Price combines those two concepts into a single financial value.

This means:

- Price is not just a raw numeric statement.
- Price is a monetary value with explicit denomination context.
- Currency does not define the magnitude.
- Monetary Amount does not define the denomination.
- Price provides the integrated business meaning that combines both.

## 3. Business Purpose

Price represents the monetary value of an item or concept expressed in a specific Currency.

Its purpose is to make financial valuation explicit without conflating it with general measurement.

Price exists to express:

- a financial value,
- in a denomination,
- with clear business meaning,
- without requiring conversion or registry membership.

Price is not a measurement object and is not a general-purpose quantity object.

## 4. Business Rules

Price is governed by the following business rules:

- Price represents a monetary amount.
- Price is expressed in a specific Currency.
- Price is a financial value, not a measurement value.
- Price may be zero.
- Price must not be negative.
- Price arithmetic requires matching currencies.
- Price ordering requires matching currencies.
- Price does not perform currency conversion.
- Price does not infer registry membership.
- Price does not infer legal tender status.
- Price does not infer exchange support.

## 5. Business Invariants

The following invariants define Price:

- A Price always represents a monetary amount.
- A Price always carries a specific Currency.
- A Price is always denomination-aware.
- A Price may be zero but never negative.
- A Price is not a Quantity.
- A Price is not a measurement concept.
- A Price is not interchangeable with a generic amount unless the denomination is also the same.
- A Price remains a value object with explicit financial meaning.

## 6. Validation Rules

Price validation is based on the business meaning of the composed concepts.

Price must be rejected when:

- the monetary amount is invalid,
- the currency is invalid,
- the monetary amount is negative,
- the price is expressed without a valid Currency,
- price comparison or arithmetic is attempted across currencies,
- the business value lacks the required denomination context.

Price validation does not include:

- conversion logic,
- registry validation,
- exchange support validation,
- legal tender validation,
- price-source validation.

Those responsibilities remain outside the Foundation Value Object layer.

## 7. Normalization Rules

Price normalization is governed by the canonical normalization behavior of its two business components:

- Monetary Amount normalization preserves the canonical business magnitude.
- Currency normalization preserves the canonical denomination representation.
- Price preserves the normalized business value and normalized denomination together.

The combined representation remains canonical and deterministic.

Price does not create a new normalization regime beyond the approved business semantics of its constituent concepts.

## 8. Equality

Price equality is value-based.

Two Price values are equal when:

- their Monetary Amount values are equal,
- their Currency values are equal.

This means:

- 100 USD equals 100 USD
- 100 USD does not equal 100 EUR
- 100 USD does not equal 50 USD

Equality requires both magnitude and denomination to match.

## 9. Ordering

Price ordering applies only when the prices share the same Currency.

Ordering rules:

- Price values can be ordered only within the same denomination.
- Cross-currency ordering is not valid business behavior.
- Ordering is based on the financial magnitude of the Monetary Amount within the shared Currency.

This preserves the business meaning that prices are not directly comparable across different denominations without an approved conversion process.

## 10. Arithmetic

Price arithmetic is defined only within a matching Currency.

Approved arithmetic behavior:

- addition requires matching Currency values,
- subtraction requires matching Currency values,
- other financial operations remain subject to approved business rules defined by higher-level domain services,
- conversion is not performed implicitly,
- mismatched currencies are not combined.

Price is therefore a denomination-aware financial value, not a generalized measurable quantity.

## 11. Examples

Valid examples:

- 100 USD
- 25.50 EUR
- 0 BTC
- 1.25 GBP

Invalid examples:

- -10 USD
- 100 EUR with a mismatched Currency context
- arbitrarily comparing 100 USD to 100 EUR without conversion rules
- any price lacking a valid Currency

## 12. Future Dependency

Price is the foundation for Money.

Money extends Price by adding broader financial meaning beyond a simple monetary value expression.

The relationship is conceptual and directional:

- Price represents a financial value in a denomination.
- Money extends that concept into richer financial semantics, such as account, settlement, or value-bearing responsibility.

This keeps the design architecture clear:

- Quantity is measurement.
- Currency is denomination.
- Price is monetary value.
- Money is a richer financial expression built on Price.

## Design Review Summary

Price is approved as a business concept representing a monetary amount in a specific Currency.

This design explicitly rejects the earlier Quantity-based composition model.

The approved business architecture is therefore:

- Price is not a measurement.
- Price is not composed from Quantity.
- Price represents financial value.
- Currency provides denomination.
- Monetary Amount provides financial magnitude.

This remains implementation-independent and suitable for the next phase of implementation review.
