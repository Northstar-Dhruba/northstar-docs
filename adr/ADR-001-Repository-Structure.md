# ADR-001: Repository Structure

Status

Accepted

Date

2026-08-09

Decision Makers

Northstar Architecture Team

## Context

Northstar is intended to become a long-term Financial Intelligence Platform. Its future scope includes market data, intelligence, research, portfolio analytics, backtesting, AI-assisted insight, trading workflows, notifications, broker integrations, and dashboards.

This breadth creates a significant architectural risk if business logic, infrastructure concerns, and user experience responsibilities are not clearly separated. A repository strategy was required that would preserve modularity, limit coupling, and support long-term evolution without forcing a reorganization of the platform at a later stage.

## Decision

The platform will be organized into three repositories:

- northstar-core
- northstar-api
- northstar-web

The responsibilities of each repository are as follows:

- northstar-core contains the domain model, business rules, financial concepts, intelligence logic, research logic, and other business-critical capabilities.
- northstar-api acts as the orchestration layer. It coordinates integrations, services, and application workflows without owning domain logic.
- northstar-web is the presentation layer. It provides user interaction, visualization, and interface behavior without owning business rules.

Business logic belongs only inside northstar-core. The API repository is responsible for orchestration and adaptation. The Web repository is responsible for presentation and user experience.

## Rationale

This structure was selected because Northstar must remain extensible and maintainable over time. A clear repository boundary allows the platform to evolve in a disciplined manner, especially as new domains, integrations, and product capabilities are introduced.

The decision also reflects the architectural goal of preserving a strong domain core. Business rules are the most valuable and enduring part of the platform; they should not be diluted by infrastructure concerns or presentation concerns. Separating repositories ensures that domain logic can be developed, tested, and evolved independently of the delivery layers.

## Alternatives Considered

### Single Monolithic Repository

A single repository would have simplified initial setup but would have weakened separation of concerns. It would have increased the risk of coupling business logic to infrastructure and presentation concerns, making the platform harder to evolve.

### Monorepo with Shared Folders

A monorepo with shared folders would have allowed tighter coupling between components and would have reduced the clarity of responsibility boundaries. It would have made the platform more difficult to reason about as it matured.

### Microservices

A microservices approach was not selected because the platform is not yet defined by independent deployment units or distinct operational boundaries. At this stage, the need is for architectural discipline and clear responsibility boundaries rather than distributed service decomposition.

## Consequences

### Positive Consequences

- Clear separation of concerns across the platform.
- Stronger protection for business logic.
- Easier long-term evolution and maintenance.
- Better alignment with Clean Architecture and Domain Driven Design.

### Negative Consequences

- Initial coordination across repositories is required.
- Architectural discipline must be preserved to avoid boundary erosion.
- Cross-repository integration requires deliberate design.

## Trade-offs

The selected model favors clarity and maintainability over convenience. It introduces a modest amount of coordination overhead but provides a stronger foundation for future growth.

## Future Considerations

This repository strategy is intended to support future expansion across new markets, data providers, intelligence capabilities, research workflows, portfolio features, and AI-assisted analysis. As the platform grows, the repository boundaries will continue to protect the core domain from becoming entangled with orchestration or user interface concerns.

## Related Documents

- Architecture Constitution
- Domain Handbook
- northstar-core README
- northstar-api README
- northstar-web README
