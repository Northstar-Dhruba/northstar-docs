# Quantity Value Object Design Specification

## 1. Relationship to Symbol

Quantity inherits the approved reference-first engineering pattern from Symbol.

Inherited from Symbol:

- immutable value-object semantics,
- value-based equality,
- canonical normalized representation,
- deterministic ordering,
- explicit validation during construction,
- a minimal public contract,
- the same documentation and review philosophy.

What differs:

- Symbol represents a general normalized identifier.
- Quantity represents a measurable numeric amount.

This difference exists because the domain needs a value object that can express magnitude and arithmetic semantics, not just identity.

## 2. Why Quantity Becomes the Numeric Family Root

Quantity is the appropriate Numeric Family root because it represents the most fundamental business concept in the numeric family: a measurable amount with explicit value semantics.

It is the natural foundation for other numeric concepts because:

- it captures magnitude directly,
- it supports arithmetic behavior,
- it can be composed into more specialized business concepts,
- it provides a common semantic anchor for percentage, price, and money.

Quantity is not a general-purpose identifier and therefore does not belong to the Identity family. It belongs to the Numeric family because its business meaning is rooted in amount and measurement.

## 3. Business Purpose

Quantity represents a measurable amount of something in the domain.

It exists to make numeric business values explicit, safe, and semantically meaningful. It removes ambiguity between a raw number and a domain quantity that carries business intent.

The existing Symbol value object is insufficient because it cannot represent magnitude, arithmetic, or numeric constraints.

## 4. Business Rules

Quantity is governed by the following business rules:

- It represents a measurable amount.
- It must be expressed as a canonical numeric value.
- It must be valid at construction.
- It must preserve a deterministic representation.
- It must support business-consistent comparison and arithmetic.
- It must not be used for identity, classification, or symbolic labeling.

## 5. Business Invariants

The following invariants are unique to Quantity:

- A Quantity always represents a numeric amount.
- A Quantity always exists in a canonical numeric form.
- A Quantity never represents a symbolic identifier.
- A Quantity always preserves its business meaning as magnitude rather than as formatting.

All other invariants are inherited from Symbol.

## 6. Validation Rules

Quantity must reject values that violate numeric business intent.

The value must be rejected when:

- the input is null,
- the input is not a numeric value,
- the input is empty or not representable as a valid quantity,
- the input is not finite,
- the input is outside the approved business range for quantity values.

The business rules must be deterministic and explicit.

## 7. Normalization Rules

Quantity normalization is business-specific and must ensure a canonical numeric form.

Normalization rules should include:

- removing insignificant formatting differences,
- preserving a canonical representation of the numeric value,
- avoiding ambiguous presentation such as inconsistent decimal formatting.

No additional formatting beyond canonical numeric representation should be introduced.

## 8. Numeric Representation

Decimal should be used for Quantity.

Reason:

- business quantities often require exact decimal semantics,
- financial and operational domains are sensitive to rounding error,
- decimal arithmetic preserves correctness where binary floating-point would introduce subtle inaccuracies.

Using Decimal avoids ambiguous numeric behavior and better supports future business concepts such as price, money, and percentage.

## 9. Arithmetic

Quantity must support arithmetic because it is the numeric family root.

Supported operations:

- addition,
- subtraction,
- multiplication,
- division.

Business rules for arithmetic:

- addition and subtraction should preserve the numeric semantics of quantity,
- multiplication and division should yield a Quantity when the operation remains within the domain meaning of quantity,
- division by zero must be rejected,
- subtraction resulting in a negative quantity should be allowed only if the domain explicitly permits negative quantities; otherwise it should be rejected.

For v1.0, the safer default is to allow negative quantities only when the business context explicitly requires them. If the domain does not define that need, the value object should reject negative results.

## 10. Comparison

Quantity must support value-based comparison.

Comparisons should be based on the canonical numeric value rather than formatting or representation details.

This ensures consistency with the inherited value-object semantics from Symbol.

## 11. Examples

Valid examples:

- 1
- 10.5
- 0.25
- 1000
- 3.14159

Invalid examples:

- null
- empty input
- NaN
- infinity
- an unrepresentable non-numeric value
- a value outside the approved business range
- a division-by-zero operation

## 12. Future Dependency Impact

Quantity is the numeric foundation for the following conceptual dependencies:

- Percentage depends on Quantity as the base unit for proportional measurement.
- Price depends on Quantity as the basis for amount and scaling.
- Money depends on Quantity as the numeric magnitude portion of a monetary amount.

These relationships should remain conceptual and directional: Quantity underpins the numeric semantics of more specialized value objects without changing the core meaning of Quantity itself.
