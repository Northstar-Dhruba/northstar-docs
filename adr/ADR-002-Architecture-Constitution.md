# ADR-002: Architecture Constitution

Status

Accepted

Date

2026-08-09

Decision Makers

Northstar Architecture Team

## Context

Northstar required a long-term architectural vision before implementation could proceed in a disciplined manner. The platform was expected to grow into a broad financial intelligence system with multiple domains, integrations, and product capabilities. Without a clear architectural constitution, the platform would risk becoming inconsistent, over-coupled, and difficult to evolve.

A shared architectural foundation was therefore necessary to preserve clarity across the core domain, orchestration layer, and presentation layer.

## Decision

The platform will adopt the following architectural principles:

- Clean Architecture
- Domain Driven Design
- SOLID
- Modular Design
- Separation of Concerns
- Event-Oriented Thinking

The architectural role of each repository is as follows:

- northstar-core is the home of business logic and domain knowledge.
- northstar-api is the orchestration layer that coordinates application workflows and integrations.
- northstar-web is the presentation layer that delivers user experience and interaction.

Northstar is defined as a Financial Intelligence Platform. It is not a trading platform. Trading is one capability of the broader platform, not its sole identity.

## Rationale

These principles were selected because they provide a durable foundation for a platform that must remain extensible over many years. Clean Architecture and Domain Driven Design support strong domain modeling and preserve business meaning. SOLID and modular design improve maintainability and reduce architectural drift. Event-oriented thinking provides a more scalable model for reacting to changing market and platform conditions.

The decision also establishes the platform’s identity. Northstar is not primarily an execution system. It is a system for observation, interpretation, evaluation, and long-term knowledge generation.

## Alternatives Considered

### Traditional Layered Architecture

A traditional layered architecture would have been simpler to understand initially, but it would have increased the risk of business logic becoming entangled with infrastructure and interface concerns.

### Database-Centric Architecture

A database-centric approach would have emphasized persistence over domain meaning. This was rejected because it would have weakened the platform’s ability to model financial behavior and long-term intelligence effectively.

### Service-Oriented Architecture Without DDD

A service-oriented architecture without Domain Driven Design would have introduced distribution and interface complexity without establishing a coherent domain model. This would have made the platform more difficult to evolve in a meaningful way.

## Consequences

### Positive Consequences

- Stronger architectural clarity.
- Better separation between business logic and implementation concerns.
- Improved maintainability and extensibility.
- Better alignment with long-term platform goals.

### Negative Consequences

- Greater discipline is required in implementation decisions.
- Architectural boundaries must be respected consistently.
- Some short-term conveniences must be avoided in favor of long-term structure.

## Trade-offs

The selected approach emphasizes architectural durability over superficial simplicity. It requires more deliberate design work early in the lifecycle, but it provides a stronger foundation for future capabilities.

## Future Considerations

These principles are intended to support the platform as it grows into additional capabilities such as richer intelligence workflows, more research functions, portfolio analytics, broker integrations, notifications, and AI-assisted reasoning. The architecture is designed to absorb new business capabilities without eroding the core domain model.

## Related Documents

- Architecture Constitution
- Domain Handbook
- northstar-core README
- northstar-api README
- northstar-web README
