# ES-002: Reference Implementations

## Purpose

This standard defines the role and governance of Reference Implementations in Northstar.

## Scope

This standard applies to approved baseline implementations that establish the preferred pattern for a class of domain concepts.

## Mandatory Rules

- A Reference Implementation MUST be an approved baseline pattern for a family of related concepts.
- Symbol MUST be treated as Northstar Reference Value Object v1.0.
- Future Value Objects MUST follow the same implementation style unless a documented architectural justification establishes a material deviation.
- Changes to a Reference Implementation MUST be reviewed and approved through the established engineering review process.
- Deviations from a Reference Implementation MUST be explicit, justified, and documented.

## Recommended Practices

- Reference Implementations SHOULD remain small, focused, and easy to understand.
- Reference Implementations SHOULD emphasize explicit validation, immutability, and a minimal public API.
- Reference Implementations SHOULD be reused as the starting point for new implementations in the same domain family.
- Reference Implementations SHOULD evolve only through deliberate review rather than local adaptation.

## Examples

### What Qualifies as a Reference Implementation

A Reference Implementation is an approved pattern that other implementations are expected to follow because it embodies the preferred engineering approach for a domain concept.

### Current Reference

Symbol is the current reference value object and serves as the baseline for future value objects such as Price, Money, Quantity, Currency, ExchangeCode, Percentage, and Timeframe.

### When Deviation Is Acceptable

A deviation MAY be accepted only when the domain requires a materially different behavior and that difference is justified in an ADR or approved design document.

## Common Mistakes

- Treating a local implementation as a reference without formal review.
- Allowing repeated reinvention of the same pattern in different packages.
- Introducing deviations without architectural or standards justification.
- Changing a reference implementation in a way that weakens consistency.

## Review Checklist

- [ ] The implementation aligns with the approved reference pattern.
- [ ] Any deviation is documented and justified.
- [ ] The design remains consistent with the domain model.
- [ ] The implementation is understandable and reusable.

## Related Standards

- ES-001: Value Objects
- ES-003: Testing Standards
- ES-004: Package Public API

## References

- Architecture Constitution
- Domain Handbook
- ES-001: Value Objects

---

## Official Reference Implementations

### Northstar Reference Value Object v1.0 — Symbol

| Field | Value |
|---|---|
| Component | Symbol |
| Status | Approved |
| Date | 2026-08-10 |
| Repository | northstar-core |
| Package | foundation.value_objects |
| Registry | docs/engineering/registry/Reference-Implementations.md |

Symbol qualified as the first Reference Value Object because it satisfies the complete value object contract: immutability enforced by the language, invariants preserved at construction, deterministic normalization, explicit and layered validation, a minimal public API, and full framework independence. The accompanying contract test suite establishes the Northstar Value Object Testing Standard v1.0.

Future value objects MUST inherit this implementation style. The following characteristics MUST be preserved in every future value object:

- `@dataclass(frozen=True, slots=True, order=True)` as the default structural form.
- Normalization and validation performed exclusively in `__post_init__`.
- Module-level private helper functions rather than instance methods for internal logic.
- Domain-specific exceptions derived from `ValidationError`.
- A public API limited to domain-relevant behavior.

Deviations from this style require both an Architecture Review and an Engineering Review. A deviation that is not documented and approved MUST be rejected in code review.
