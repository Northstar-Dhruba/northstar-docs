# ES-003: Testing Standards

## Purpose

This standard defines the required testing discipline for Northstar.

## Scope

This standard applies to unit, integration, contract, performance, and acceptance tests across all repositories.

## Mandatory Rules

- Tests MUST be created as part of implementation work and MUST NOT be deferred.
- Tests MUST verify behavior rather than implementation detail.
- Unit tests MUST verify the behavior of a single unit of logic.
- Integration tests MUST verify interactions across package or service boundaries.
- Contract tests MUST verify that interfaces and public contracts remain valid.
- Performance tests MUST be used for critical paths where latency or resource usage is material.
- Acceptance tests SHOULD verify that business workflows are satisfied.
- Tests MUST be deterministic and repeatable.

## Recommended Practices

- Test names SHOULD describe the behavior being verified.
- Tests SHOULD be grouped by purpose and responsibility.
- Tests SHOULD use clear assertions and focused fixtures.
- Coverage SHOULD be sufficient to demonstrate correctness for critical branches, validation paths, and edge cases.
- Fixtures SHOULD be minimal and explicit.

## Examples

### Unit Tests

Unit tests verify one isolated behavior, such as valid construction, validation failure, normalization, equality, or ordering.

### Integration Tests

Integration tests verify that separate components work correctly together, including domain and infrastructure boundaries.

### Contract Tests

Contract tests verify that external-facing interfaces or package exports remain compatible with their expected behavior.

### Performance Tests

Performance tests verify that critical operations remain acceptable under expected workload.

### Acceptance Tests

Acceptance tests verify that a user or domain workflow is satisfied end to end.

### Naming Conventions

Preferred patterns include:

- test_constructs_valid_value_object
- test_rejects_invalid_input
- test_normalizes_input_to_canonical_form
- test_value_objects_compare_by_value

### Folder Structure

Tests SHOULD be organized by purpose, such as unit, integration, contract, and acceptance.

## Common Mistakes

- Writing tests after implementation without defining expected behavior first.
- Testing internal implementation rather than observable behavior.
- Using brittle fixtures that obscure the actual scenario.
- Treating coverage as a goal in itself rather than a means of validating risk.

## Review Checklist

- [ ] The behavior under test is clear.
- [ ] The test is deterministic.
- [ ] The test validates observable outcomes.
- [ ] Valid and invalid cases are covered.
- [ ] Critical branches and edge cases are tested.

## Related Standards

- ES-001: Value Objects
- ES-002: Reference Implementations
- ES-005: Code Review Checklist

## References

- Architecture Constitution
- Domain Handbook
- Pytest and repository test conventions
