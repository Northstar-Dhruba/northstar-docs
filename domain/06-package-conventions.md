# 06. Package Conventions

## Purpose

This document defines the conventions that should guide package design and code organization across Northstar.

These conventions are meant to support long-term maintainability, readability, and architectural discipline. They are not merely style preferences; they are part of the platform’s engineering constitution.

---

## 1. Package Naming

Package names should reflect business meaning, not implementation mechanics.

### Preferred

- intelligence
- research
- acquisition
- portfolios
- execution
- notifications
- configuration
- shared

### Avoid

- utils
- helpers
- misc
- service_layer
- old_stuff

Packages should be named after domains or capabilities, not after incidental technical concerns.

---

## 2. Folder Naming

Folder names should be:

- lowercase,
- descriptive,
- stable over time,
- aligned with the domain vocabulary.

Examples:

- intelligence/signals
- research/backtests
- portfolios/positions
- execution/orders
- notifications/alerts

Folders should not be created for temporary convenience or to reflect implementation shortcuts.

---

## 3. Naming Conventions

### Classes and Types

Names should be domain-oriented and expressive.

Prefer:

- Portfolio
- Position
- Indicator
- Signal
- ResearchExperiment
- NotificationAlert

Avoid:

- DataHandler
- Manager
- Thing
- Processor

### Functions and Methods

Names should describe the business action clearly.

Prefer:

- calculate_signal
- evaluate_strategy
- record_performance
- update_portfolio

Avoid vague names such as:

- do_work
- handle_it
- process_data

---

## 4. Constants

Constants should be named in uppercase when they represent fixed values or domain-level flags.

Examples:

- MAX_RISK_LIMIT
- DEFAULT_TIMEFRAME
- MARKET_OPEN_STATUS

Constants should be meaningful in the domain context, not arbitrary numeric values.

---

## 5. Enums

Enums should represent finite business states or categories.

Examples:

- OrderStatus
- SignalStrength
- MarketSessionType
- AlertPriority

Enums should be used when the domain has a closed set of meaningful states.

---

## 6. Interfaces and Contracts

Where abstractions are used, they should represent domain contracts rather than technical convenience.

Examples:

- a market data provider interface,
- a notification channel contract,
- a research evaluator contract.

These abstractions should be stable, minimal, and centered on domain intent.

---

## 7. Tests

Tests should reflect the business meaning of the code rather than implementation details.

Testing expectations:

- test domain behavior,
- test invariants,
- test important flows and event outcomes,
- keep tests readable and behavior-focused.

The domain core should be tested independently of API and UI layers.

---

## 8. Documentation

Every important domain module should have documentation that explains:

- its purpose,
- its responsibilities,
- its relationships to other domains,
- its non-responsibilities.

Documentation should be treated as part of the design, not as an afterthought.

---

## 9. Docstrings

Docstrings should be used to clarify intent and domain meaning.

They should explain:

- what the concept represents,
- why it exists,
- what assumptions it makes.

They should not simply restate the code in prose.

---

## 10. Type Hints

Type hints should be used consistently to improve clarity and reduce ambiguity.

They should reflect business types wherever possible rather than generic placeholders.

Examples:

- Money
- Percentage
- Timeframe
- Quantity
- Symbol

This improves readability and reduces the possibility of misunderstanding financial semantics.

---

## 11. Dependency Injection

Dependency injection should be used to separate domain behavior from infrastructure choices.

The principle is:

- the domain declares what it needs,
- the application layer provides the implementation.

This keeps the core independent from concrete infrastructure decisions.

---

## 12. Configuration

Configuration should be explicit and domain-aware.

Configuration should not be scattered across business logic. It should be centralized and clearly named according to its purpose.

Examples:

- market configuration,
- risk limits,
- feature flags,
- notification preferences.

---

## 13. Error Handling

Error handling should reflect business consequences.

Examples:

- invalid instrument data,
- unsupported market context,
- missing configuration,
- invalid portfolio state.

Errors should be meaningful, typed where appropriate, and understandable in the domain context.

---

## 14. Logging

Logging should be used for observability, not for replacing the domain model.

Logs should capture facts that are meaningful to operations, not every internal step.

The core should avoid relying on logging as a substitute for explicit domain behavior.

---

## 15. Imports

Imports should be structured to preserve architectural clarity.

Rules:

- prefer domain-oriented imports,
- avoid circular dependencies,
- keep imports local and intentional,
- do not import infrastructure packages into the core domain.

---

## 16. File Size and Module Size Recommendations

Modules should remain focused.

Recommended guidance:

- keep modules small enough to understand quickly,
- keep each module focused on a single responsibility,
- split modules when responsibility grows beyond a clear boundary.

A package should not become a container for unrelated concepts simply because they are convenient to reach from one place.

---

## 17. Maximum Responsibilities per Package

A package should usually own one primary responsibility or a tightly related cluster of responsibilities.

If a package begins to contain:

- unrelated concepts,
- mixed technical and domain concerns,
- multiple independent capabilities,

it should be split.

---

## 18. Guidelines for Adding New Packages

A new package should be introduced when:

- a capability has a distinct business purpose,
- a boundary is conceptually clear,
- the concept deserves its own vocabulary,
- the concern is likely to grow over time.

If a concept is merely a helper for another package and does not need its own identity, it should remain within the existing package.

---

## 19. Guidelines for Adding New Domains

A new domain should only be introduced when:

- the capability has a meaningful business identity,
- its concepts are not naturally part of an existing domain,
- it has its own responsibilities, entities, and events,
- it may evolve independently over time.

The platform should avoid creating new domains for convenience. Domains should reflect genuine business boundaries.

---

## 20. Examples of Good Package Organization

Good organization is domain-oriented and stable:

- intelligence/indicators
- intelligence/signals
- research/backtests
- portfolios/positions
- execution/orders
- notifications/alerts
- configuration/settings

This makes it easier to understand where new capabilities should go.

---

## 21. Examples of Bad Package Organization

Poor organization is often shaped by convenience or technical style:

- utils/
- services/
- common/
- modules/
- helpers/
- misc/

These structures often become dumping grounds for unrelated functionality and make the system harder to evolve.

---

## Related Documents

- [02-domain-map.md](02-domain-map.md)
- [03-ubiquitous-language.md](03-ubiquitous-language.md)
- [05-dependency-rules.md](05-dependency-rules.md)
