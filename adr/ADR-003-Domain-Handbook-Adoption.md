# ADR-003: Domain Handbook Adoption

Status

Accepted

Date

2026-08-09

Decision Makers

Northstar Architecture Team

## Context

The platform required a common business vocabulary and a shared understanding of domain boundaries before implementation could proceed consistently. Without a formal domain handbook, teams would have risked using different meanings for the same concepts, mixing responsibilities across layers, and introducing avoidable architectural ambiguity.

A shared vocabulary and a documented domain map were therefore necessary to ensure that implementation work remained aligned with the approved architecture.

## Decision

The platform will adopt the Domain Handbook as the authoritative reference for domain understanding and architectural coordination.

The handbook includes the following components:

- Platform Vision
- Domain Map
- Ubiquitous Language
- Domain Events
- Dependency Rules
- Package Conventions

The handbook formalizes the following bounded contexts:

- Foundation
- Acquisition
- Market
- Assets
- Intelligence
- Research
- Knowledge
- Investment
- Execution
- Notification
- Configuration

The handbook also establishes the shared vocabulary used to describe financial concepts, domain events, and architectural dependencies.

## Rationale

This decision was selected because architecture is strongest when the team shares a common language. A domain handbook ensures that business meaning is not lost to implementation shortcuts or inconsistent terminology. It also provides a durable framework for future implementation work by documenting the intended boundaries of each capability.

The decision to adopt Ubiquitous Language was essential because financial systems require precise meaning. Domain Events were adopted to formalize how the platform reacts to meaningful business changes. Dependency Rules were documented to prevent architectural drift and preserve the intended direction of dependencies.

## Alternatives Considered

### Ad-hoc Development

Ad-hoc development would have been faster initially, but it would have introduced inconsistent terminology and unclear ownership. This would have increased the risk of rework and architectural confusion over time.

### Technical-Layer-First Organization

A technical-layer-first organization would have emphasized implementation structure over business meaning. This was rejected because it would have weakened the domain model and made future evolution more difficult.

### Database-First Design

A database-first design would have prioritized persistence concerns above business semantics. This was rejected because it would have created an architecture centered on storage rather than on financial meaning and domain behavior.

## Consequences

### Positive Consequences

- Shared vocabulary across disciplines and teams.
- Clearer domain boundaries.
- Stronger alignment between architecture and implementation.
- Better long-term maintainability and knowledge transfer.

### Negative Consequences

- Documentation must be maintained as the platform evolves.
- Teams must invest in understanding the handbook rather than relying on informal conventions.
- Architectural decisions require alignment with the documented model.

## Trade-offs

The selected approach favors clarity and consistency over informal flexibility. It requires a greater upfront investment in documentation and discipline, but it creates a stronger foundation for sustainable growth.

## Future Considerations

The Domain Handbook is intended to become the long-term reference for engineers, architects, and domain stakeholders as the platform evolves. It will support future implementation work across intelligence, research, portfolio analysis, execution, notification, and configuration without losing architectural coherence.

## Related Documents

- Architecture Constitution
- Domain Handbook
- Platform Vision
- Domain Map
- Ubiquitous Language
- Domain Events
- Dependency Rules
- Package Conventions
