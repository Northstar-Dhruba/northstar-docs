# Value Object Template

## Purpose

Describe the domain concept represented by this value object.

## Business Rules

List the business rules that govern this value object.

## Validation

Document the validation rules and required constraints.

## Normalization

Describe any normalization behavior and the canonical representation.

## Invariants

List the invariants that must always hold.

## Exceptions

Document the exceptions or failure modes used for invalid input.

## Public API

Describe the constructor, accessors, and expected public behavior.

## Testing Checklist

- [ ] Valid construction
- [ ] Invalid construction
- [ ] Normalization
- [ ] Equality
- [ ] Hashing
- [ ] Ordering where applicable
- [ ] Serialization where applicable

## Review Checklist

- [ ] Aligns with the Domain Handbook
- [ ] Follows Engineering Standards
- [ ] Has explicit validation and immutability
- [ ] Includes tests and documentation

## Engineering Standards Reference

- ES-001: Value Objects
- ES-002: Reference Implementations
- ES-003: Testing Standards
- ES-004: Package Public API
