# Engineering Standards Handbook

## Purpose

This handbook defines the engineering standards that govern how Northstar software is implemented. It establishes the mandatory and recommended practices for code structure, testing, review, packaging, documentation, and contribution workflow.

These standards MUST be followed unless an Architecture Decision Record (ADR) explicitly documents an exception.

## Scope

This handbook applies to all implementation work across the Northstar platform, including core domain logic, application services, APIs, web interfaces, tests, documentation, and repository operations.

## Relationship to Other Governance Documents

Northstar uses three related layers of governance:

- Architecture: defines the system structure, boundaries, responsibilities, and domain model.
- Engineering Standards: defines how implementation work MUST be carried out in a consistent, maintainable, and reviewable manner.
- Coding Standards: define the concrete conventions used within individual implementations, such as naming, formatting, and module organization.

The Architecture Constitution and Domain Handbook define what Northstar is. This handbook defines how contributors MUST implement that design in practice.

## Engineering Philosophy

Northstar values clarity over cleverness. Architecture MUST precede implementation. Consistency MUST take precedence over personal preference. Code SHOULD optimize for readability. Business rules MUST remain explicit. Reference Implementations are preferred over reinvention.

## Engineering Principles

All implementation work in Northstar MUST follow these principles:

- Keep APIs small and deliberate.
- Prefer explicit code over hidden behavior.
- Make illegal states unrepresentable.
- One class SHOULD have one responsibility.
- Business rules belong inside the domain.
- Frameworks are implementation details, not domain definitions.

## Applicability

Every new feature, bug fix, refactor, package addition, and documentation change MUST comply with these standards unless a documented exception exists in the ADR set.

## Standards Index

- ES-001: Value Objects
- ES-002: Reference Implementations
- ES-003: Testing Standards
- ES-004: Package Public API
- ES-005: Code Review Checklist
- ES-006: Git Workflow
- ES-007: Documentation Standards
- ES-008: Copilot Usage Guidelines

## Review Expectations

All changes MUST be reviewed for compliance with these standards. Implementation quality MUST be judged not only by correctness, but also by clarity, maintainability, consistency, and long-term extensibility.
