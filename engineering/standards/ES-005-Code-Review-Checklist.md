# ES-005: Code Review Checklist

## Purpose

This standard defines the required review process for changes submitted to Northstar.

## Scope

This standard applies to all pull requests, merge requests, and change reviews.

## Mandatory Rules

- All changes MUST be reviewed before merge.
- Review MUST evaluate correctness, clarity, maintainability, and alignment with the Architecture Constitution.
- Review MUST confirm that the change does not introduce avoidable risk.
- Review MUST verify that tests and documentation are adequate.

## Recommended Practices

- Pull requests SHOULD be small, focused, and easy to review.
- Review comments SHOULD be specific and actionable.
- Review SHOULD favor long-term maintainability over short-term convenience.

## Examples

### Pull Request Template

```md
## Summary
- What changed?
- Why was it needed?

## Checklist
- [ ] Architecture
- [ ] DDD
- [ ] Tests
- [ ] Documentation
- [ ] Public API
- [ ] Error Handling
- [ ] Performance
- [ ] Future Maintainability
```

## Common Mistakes

- Approving changes without reviewing the impact on the domain model.
- Accepting changes that lack tests or documentation.
- Merging changes with unclear public API impact.
- Overlooking maintainability and future extensibility.

## Review Checklist

- [ ] Architecture
- [ ] DDD
- [ ] Readability
- [ ] Maintainability
- [ ] SOLID
- [ ] Error Handling
- [ ] Exceptions
- [ ] Documentation
- [ ] Type Hints
- [ ] Tests
- [ ] Public API
- [ ] Package Structure
- [ ] Naming
- [ ] Performance
- [ ] Future Maintainability

## Related Standards

- ES-003: Testing Standards
- ES-004: Package Public API
- ES-006: Git Workflow

## References

- Architecture Constitution
- Domain Handbook
- Pull request review practice
