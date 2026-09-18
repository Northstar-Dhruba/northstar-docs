# 01. Platform Vision

## Purpose

This document defines the identity of Northstar as a financial intelligence platform and establishes the architectural posture that should guide all work across the platform.

This handbook is intended to complement the approved architecture constitution and provide the shared vocabulary, boundaries, and expectations that engineers should follow over the next decade of platform evolution.

---

## 1. Vision Statement

Northstar exists to help people understand financial markets, portfolios, strategies, and opportunities through durable, extensible, and trustworthy financial intelligence.

Northstar is not a trading terminal. It is not merely an execution tool. It is a platform for observation, interpretation, evaluation, and insight.

---

## 2. Mission

Northstar exists to:

- gather relevant market and financial information,
- transform that information into useful intelligence,
- support research and experimentation,
- help users evaluate performance and risk,
- provide a foundation for future AI-driven reasoning and decision support.

---

## 3. Platform Identity

Northstar is a Financial Intelligence Platform.

Its identity is defined by three responsibilities:

1. Observation
   - It watches markets, instruments, portfolios, and signals.

2. Interpretation
   - It turns raw information into indicators, patterns, insights, and knowledge.

3. Evaluation
   - It supports research, comparison, optimization, and performance learning.

This identity is more important than any one product feature. The platform must remain faithful to this role even as it grows into new capabilities.

---

## 4. What Northstar Is

Northstar is:

- a platform for financial intelligence,
- a multi-domain system for research and insight,
- a modular architecture for market understanding,
- a long-term foundation for analytics, portfolio intelligence, and research.

---

## 5. What Northstar Is Not

Northstar is not:

- a standalone trading application,
- a user-interface-only product,
- a database-first system,
- a place for infrastructure code to define business meaning,
- a monolith that mixes every capability into one layer.

Trading is one capability of the platform. It is not the platform’s sole purpose.

---

## 6. Core Values

The platform should be guided by the following values:

- Financial Intelligence
  - The platform exists to generate useful understanding of markets and portfolios.

- Modularity
  - Capabilities should be isolated and composable.

- Extensibility
  - New markets, data sources, and analytical methods should be easy to add.

- Testability
  - Business logic must be verifiable without infrastructure coupling.

- Long-Term Thinking
  - Design should support growth over several years rather than short-term convenience.

- Clarity of Responsibility
  - Each layer and domain should have a clear purpose.

---

## 7. Long-Term Goals (3–5 Years)

Over the next several years, Northstar should mature toward:

- multi-market support,
- broadening financial intelligence capabilities,
- AI research and assisted interpretation,
- advanced portfolio intelligence,
- options and derivatives analysis,
- crypto market awareness,
- deeper fundamental analysis,
- richer reporting and insight delivery.

These goals should be supported by a domain model that remains stable even as product capabilities evolve.

---

## 8. Repository Responsibilities

The repository boundaries are a core architectural decision.

### northstar-core

Responsible for domain logic, business rules, financial concepts, and platform intelligence. This is the authoritative home of Northstar’s business meaning.

### northstar-api

Responsible for orchestration, integration, coordination, and application services. It adapts domain capabilities to transport and infrastructure concerns.

### northstar-web

Responsible for presentation, interaction, visualization, and user experience. It consumes capabilities from the system without owning business logic.

---

## 9. Success Principles

The platform should succeed if it follows these principles:

- Business logic belongs in the Core.
- Infrastructure is replaceable.
- Everything is modular.
- Domain comes first.
- The system can evolve without rewriting the foundation.
- The architecture remains understandable to new engineers.

These principles are not optional. They are part of the platform’s operating constitution.

---

## Related Documents

- [02-domain-map.md](02-domain-map.md)
- [03-ubiquitous-language.md](03-ubiquitous-language.md)
- [05-dependency-rules.md](05-dependency-rules.md)
- [06-package-conventions.md](06-package-conventions.md)
