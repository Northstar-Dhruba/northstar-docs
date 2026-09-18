# 05. Dependency Rules

## Purpose

This document defines the architectural dependency rules for Northstar. These rules ensure that the platform remains clean, modular, and aligned with the approved domain model.

The fundamental rule is simple: business meaning must remain in the core, and infrastructure must not define or distort that meaning.

---

## 1. Dependency Rule

The core domain must remain the source of truth for business logic.

### Rule

- northstar-core may define business concepts and domain rules.
- northstar-api may orchestrate and adapt those concepts.
- northstar-web may present and interact with them.
- Infrastructure concerns must depend on the domain, not the reverse.

### Reason

This rule protects the platform from architectural drift. If business logic is allowed to spread into infrastructure or presentation layers, the system becomes harder to test, evolve, and reason about.

---

## 2. Core Rules

### Rule 1: Core never imports API

The domain core must not depend on the API layer for its implementation.

**Reason**
- The core should remain independent of delivery mechanisms.

**Example**
- Domain concepts can be used by an API layer without the domain needing to know about API frameworks.

**Forbidden Example**
- Domain modules importing API-specific transport or routing structures.

---

### Rule 2: Core never imports Web

The core must not depend on UI or presentation concerns.

**Reason**
- Presentation concerns should not shape the meaning of domain behavior.

**Forbidden Example**
- Domain rules depending on React components, view models, or UI state structures.

---

### Rule 3: Web never imports Core directly

The presentation layer should not depend directly on the core domain in a way that bypasses the application boundary.

**Reason**
- The web layer should consume capabilities through a stable application boundary rather than coupling itself directly to domain internals.

**Preferred Direction**
- Web → API → Core

---

### Rule 4: API imports Core

The orchestration layer should depend on the core domain.

**Reason**
- The API layer exists to coordinate and adapt domain capabilities for external use.

**Preferred Direction**
- API → Core

---

### Rule 5: Infrastructure depends on Core

Infrastructure concerns such as persistence, messaging, scheduling, and integrations should depend on the domain model.

**Reason**
- Infrastructure must serve the domain, not define it.

---

### Rule 6: Core has zero infrastructure knowledge

The core should not contain knowledge of databases, HTTP, UI frameworks, or infrastructure technologies.

**Reason**
- Infrastructure choices should be replaceable without rewriting the domain model.

**Forbidden Example**
- Strategy logic referencing database access patterns.
- Indicator models depending on HTTP clients.

---

## 3. Domain-Specific Rules

### Strategies never know about databases

Strategy concepts should remain focused on business meaning and reasoning. They should not become coupled to persistence or storage concerns.

### Indicators never know about HTTP

Indicators and analytical concepts should remain domain-focused. They should not depend on transport or network protocols.

### Notifications never calculate business rules

Notification capabilities should communicate results, not define business logic. They should not decide what constitutes a signal or a risk condition.

### Research never replaces the domain model

Research may evaluate ideas, but it should not become the place where the platform’s core meaning is defined.

---

## 4. Why These Rules Matter

These rules exist to protect the platform from common architectural failures:

- hidden coupling,
- logic leaking into infrastructure,
- poor testability,
- hard-to-replace integrations,
- weakening of the domain model over time.

If the rules are followed consistently, the platform will remain adaptable and understandable even as it grows.

---

## 5. Enforcement Expectations

Every new capability should be reviewed against these questions:

- Does this belong in the domain core?
- Does it depend on infrastructure concerns that should be externalized?
- Does it introduce a forbidden dependency?
- Does it preserve the direction of architecture?

If the answer is unclear, the feature should be treated as an architectural concern and not implemented casually.

---

## Related Documents

- [01-platform-vision.md](01-platform-vision.md)
- [02-domain-map.md](02-domain-map.md)
- [04-domain-events.md](04-domain-events.md)
- [06-package-conventions.md](06-package-conventions.md)
