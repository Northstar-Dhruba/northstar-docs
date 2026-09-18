# ES-001: Value Objects

## Purpose

This standard defines the required implementation approach for domain value objects in Northstar.

## Scope

This standard applies to all value objects introduced in the core domain and related packages.

## Mandatory Rules

- Value Objects MUST be immutable.
- Value Objects MUST be defined by their data and MUST NOT depend on object identity.
- Value Objects MUST validate input at construction time.
- Value Objects MUST reject invalid state through explicit exceptions.
- Value Objects MUST preserve a deterministic and canonical representation.
- Value Objects MUST provide equality and hashing semantics that reflect value-based semantics.
- Value Objects MUST be documented with their invariants and validation rules.

## Recommended Practices

- The default implementation SHOULD follow the dataclass pattern:

```python
@dataclass(
    frozen=True,
    slots=True,
    order=True
)
class ExampleValueObject:
    ...
```

- Dataclasses SHOULD be used when the value object is a simple immutable container with clear business meaning and straightforward validation.
- A regular class MAY be used when the domain requires behavior that cannot be expressed appropriately by a dataclass.
- Construction SHOULD validate inputs before state is accepted.
- Normalization SHOULD be applied when the domain defines a canonical form.

## Examples

### When to Create a Value Object

A value object SHOULD be created when the concept:

- is defined by its attributes rather than by persistence identity,
- is compared by value,
- must remain immutable,
- and carries business rules that are meaningful to the domain.

### When Not to Create a Value Object

A value object SHOULD NOT be created when the concept is a transient technical detail, an entity with identity, or a simple scalar that does not carry domain semantics.

### Examples of Appropriate Use

- Symbol
- Price
- Money
- Quantity
- Currency
- ExchangeCode
- Percentage
- Timeframe

### Anti-Patterns

- Primitive obsession: using strings or integers to represent domain concepts that need validation and semantics.
- Mutable state: allowing the object to change after creation.
- Hidden validation: accepting invalid values silently.
- Over-engineering: introducing a full object model when a simple immutable value object is sufficient.

## Common Mistakes

- Treating a domain concept as a primitive when it has business meaning.
- Allowing mutation after construction.
- Encoding validation rules outside the value object.
- Creating value objects with inconsistent equality or hashing behavior.

## Review Checklist

- [ ] The concept is a value object rather than an entity.
- [ ] The object is immutable.
- [ ] Validation is explicit.
- [ ] Normalization is deterministic.
- [ ] Equality and hashing are consistent.
- [ ] Documentation describes the invariants.
- [ ] Tests cover valid and invalid construction.

## Related Standards

- ES-002: Reference Implementations
- ES-003: Testing Standards
- ES-004: Package Public API

## References

- Architecture Constitution
- Domain Handbook
- Northstar Reference Value Object v1.0
