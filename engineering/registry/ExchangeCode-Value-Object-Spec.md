# ExchangeCode Value Object Design Specification

## 1. Relationship to Reference Value Object

Inherited from Symbol:

- immutability,
- value-based identity,
- canonical representation,
- deterministic equality and hashing,
- a minimal and stable public contract.

What differs:

- Symbol represents a general market or instrument identifier.
- ExchangeCode represents a trading venue identity.

Why the difference exists:

The domain needs to distinguish between what an instrument is and where that instrument is traded or observed. This distinction is essential for market context, venue-specific rules, and downstream interpretation of market events.

## 2. Business Purpose

ExchangeCode exists to represent the business identity of a trading venue.

It exists because Symbol is insufficient for the domain when the business question is not "which instrument" but "which exchange venue." This removes ambiguity between instrument identity and venue identity.

## 3. Business Rules

The following business rules are specific to ExchangeCode and differ from Symbol:

- ExchangeCode must identify a trading venue, not an instrument or product.
- ExchangeCode must be expressed in a canonical venue-code form.
- ExchangeCode must match the exact pattern: ^[A-Z0-9]{1,8}$.
- ExchangeCode must contain uppercase letters and digits only.
- ExchangeCode must not contain whitespace, punctuation, or separators.
- ExchangeCode must remain short enough to function as a practical venue identifier.
- ExchangeCode is reserved for venue identity and should not be used for general market symbols.

## 4. Business Invariants

The following invariants are unique to ExchangeCode:

- An ExchangeCode always denotes one exchange venue.
- An ExchangeCode never denotes an instrument symbol.
- An ExchangeCode always exists in its canonical uppercase form.

All other invariants are inherited from Symbol.

## 5. Validation Rules

ExchangeCode must be rejected when any of the following business conditions are true:

- the input is null,
- the input is not a string,
- the input is empty after normalization,
- the input contains whitespace,
- the input contains punctuation or separators,
- the input contains characters outside the allowed pattern ^[A-Z0-9]{1,8}$,
- the input exceeds the maximum approved length of 8 characters.

These rules are deterministic and must be enforced consistently.

## 6. Normalization Rules

The following normalization is specific to ExchangeCode:

- leading and trailing whitespace is removed,
- alphabetic characters are converted to uppercase,
- the value is preserved in its canonical venue-code form.

No additional formatting is introduced.

All other normalization behavior is inherited from Symbol.

## 7. Examples

Valid examples:

- NYSE
- XNYS
- B3
- TSE
- LSE
- nyse → normalizes to NYSE

Invalid examples:

- New York Stock Exchange — invalid because it contains whitespace.
- NYSE! — invalid because it contains punctuation.
- NASDAQ-1 — invalid because it contains a separator.
- 123456789 — invalid because it exceeds the maximum length of 8 characters.

## 8. Exception

InvalidExchangeCodeError is the domain-specific validation exception for ExchangeCode.

It belongs in the existing validation hierarchy and is raised when a supplied value violates the business rules above.

## 9. Business Motivation

ExchangeCode exists to make venue identity explicit in the domain.

It removes ambiguity between instrument identity and exchange identity, improves clarity in market context, and supports more precise interpretation of trading-related data. This is particularly valuable in domains that depend on exchange-specific behavior, venue context, and market data interpretation.

## 10. Relationship to Other Value Objects

ExchangeCode complements Symbol by separating venue identity from instrument identity.

Conceptually, this supports a clearer domain model in which exchange context and instrument context are treated as distinct business concerns, even though both remain value-based and immutable.
