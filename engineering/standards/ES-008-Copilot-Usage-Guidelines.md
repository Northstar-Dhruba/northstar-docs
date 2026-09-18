# ES-008: Copilot Usage Guidelines

## Purpose

This standard defines how Copilot MUST be used in Northstar engineering work.

## Scope

This standard applies to all usage of Copilot or similar AI-assisted development tools in the Northstar repositories.

## Mandatory Rules

- Copilot MUST be used as an implementation aid, not as an authority on architecture or business rules.
- Copilot MUST NOT invent business rules or domain semantics.
- Copilot-generated code MUST be reviewed before merge.
- Copilot MUST NOT be used to bypass architecture review or engineering standards.
- Copilot MUST follow the approved Reference Implementations where applicable.
- AI-generated changes MUST remain consistent with the Architecture Constitution and Domain Handbook.

## Recommended Practices

- Copilot SHOULD be used to accelerate implementation once architecture and requirements are understood.
- Copilot SHOULD be used to improve clarity, consistency, and repetitive scaffolding where appropriate.
- Copilot SHOULD be paired with explicit review, tests, and documentation.
- Copilot SHOULD be used to identify possible issues, but not to replace engineering judgment.

## Examples

### Appropriate Use

- Drafting boilerplate code consistent with an approved pattern.
- Generating tests for a clearly specified behavior.
- Producing documentation from an approved implementation structure.

### Inappropriate Use

- Defining new domain rules without architectural approval.
- Replacing code review.
- Merging generated changes without review.
- Deviating from approved reference implementations without justification.

## Common Mistakes

- Treating Copilot output as authoritative design.
- Accepting generated code without review.
- Using Copilot to bypass architectural decision making.
- Allowing Copilot to introduce inconsistent or unreviewed patterns.

## Review Checklist

- [ ] The change aligns with the architecture and domain model.
- [ ] The generated code was reviewed.
- [ ] Tests were added or updated where required.
- [ ] The implementation follows the approved reference pattern.
- [ ] No new business rules were invented.

## Related Standards

- ES-002: Reference Implementations
- ES-005: Code Review Checklist
- ES-007: Documentation Standards

## References

- Architecture Constitution
- Domain Handbook
- Engineering Standards Handbook
