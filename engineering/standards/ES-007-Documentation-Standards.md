# ES-007: Documentation Standards

## Purpose

This standard defines the required documentation practices for Northstar engineering work.

## Scope

This standard applies to all repository documentation, architectural records, engineering standards, implementation notes, and contributor guidance.

## Mandatory Rules

- Documentation MUST be accurate, current, and aligned with the implementation.
- Documentation MUST distinguish architecture from implementation guidance.
- Markdown MUST be used for repository documentation.
- Headings MUST be structured consistently.
- Code examples MUST be fenced and labeled where relevant.
- Cross references MUST be used where documents depend on one another.
- ADRs, Engineering Standards, Domain Handbook, and README files MUST remain consistent with each other.

## Recommended Practices

- Documents SHOULD use concise, precise prose.
- Tables SHOULD be used for structured comparison where appropriate.
- Diagrams SHOULD be used sparingly and only when they improve clarity.
- Documentation SHOULD be written for long-term maintainability, not short-term convenience.

## Examples

### Markdown Conventions

- Use a single H1 per document.
- Use hierarchical headings for structure.
- Use bullet lists for grouped facts.
- Use fenced code blocks for examples.

### Cross References

Engineering documents SHOULD reference the relevant ADRs, Architecture Constitution, and Domain Handbook when the topic is governed by those documents.

### README Expectations

A README SHOULD explain purpose, scope, usage, and any important constraints or assumptions.

## Common Mistakes

- Documenting implementation details that are already captured in architecture or domain documents.
- Allowing documentation to diverge from the implementation.
- Using inconsistent heading and formatting patterns.
- Treating documentation as optional work.

## Review Checklist

- [ ] The document is current and accurate.
- [ ] The structure is clear and consistent.
- [ ] Cross references are correct.
- [ ] Examples and code blocks are appropriate.
- [ ] The document does not duplicate architecture content unnecessarily.

## Related Standards

- ES-005: Code Review Checklist
- ES-006: Git Workflow

## References

- Architecture Constitution
- Domain Handbook
- ADR set
