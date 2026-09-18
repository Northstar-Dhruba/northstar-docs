# 04. Domain Events

## Purpose

This document defines the official business events of Northstar and explains how event-oriented thinking should guide the platform.

Domain events describe meaningful facts that have occurred in the business domain. They are not implementation details, and they are not merely logs.

---

## 1. What a Domain Event Is

A domain event is a business fact that happened and matters to the system.

It should:

- be named in the past tense,
- describe a fact, not a command,
- be immutable,
- be meaningful to one or more domains.

Examples:

- MarketClosed
- PricesAcquired
- IndicatorsCalculated
- SignalsGenerated
- PortfolioUpdated
- AlertSent

---

## 2. Naming Conventions

Domain events should follow these conventions:

- Use business language over technical language.
- Use past tense verbs.
- Prefer clear, domain-specific names.
- Avoid naming events after implementation mechanisms.

Good examples:

- MarketOpened
- PricesValidated
- PatternsDetected
- ResearchCompleted
- OrderExecuted

Less suitable examples:

- DataReceived
- JobFinished
- MessagePosted

---

## 3. Event Categories

### Acquisition Events

| Event | Meaning |
| --- | --- |
| MarketOpened | A market session became active. |
| MarketClosed | A market session ended. |
| PricesAcquired | New price observations became available. |
| PricesValidated | Acquired price data passed validation. |
| NewsAcquired | New informational content was obtained. |
| ReferenceDataUpdated | Reference or metadata changed. |

### Intelligence Events

| Event | Meaning |
| --- | --- |
| IndicatorsCalculated | Indicator values were produced for a given context. |
| PatternsDetected | Recognizable patterns were identified. |
| SignalsGenerated | Intelligence outputs were created. |
| SignalsRanked | Signals were prioritized or ordered. |
| InsightsGenerated | High-level interpretations were formed. |

### Research Events

| Event | Meaning |
| --- | --- |
| BacktestCompleted | A historical strategy evaluation completed. |
| ExperimentCompleted | A research experiment ended. |
| OptimizationFinished | An optimization run completed. |
| WalkForwardCompleted | A rolling evaluation cycle finished. |

### Knowledge Events

| Event | Meaning |
| --- | --- |
| KnowledgeUpdated | The accumulated knowledge base changed. |
| PerformanceRecorded | New performance information was added to history. |
| StatisticsCalculated | Derived performance or analysis statistics were produced. |

### Investment Events

| Event | Meaning |
| --- | --- |
| PortfolioUpdated | The portfolio state changed. |
| PositionOpened | A new holding was established. |
| PositionClosed | A holding was exited. |
| RiskEvaluated | Risk posture or exposure was assessed. |

### Execution Events

| Event | Meaning |
| --- | --- |
| OrderPlaced | An order intent was created. |
| OrderExecuted | An order reached execution status. |
| TradeCompleted | A trade was finalized. |

### Notification Events

| Event | Meaning |
| --- | --- |
| AlertSent | A notification was distributed. |
| DailySummarySent | A summary report was issued. |

---

## 4. Event Producers and Consumers

### Producers

Domain events are emitted by the domain that owns the fact.

Examples:

- Acquisition emits market and reference events.
- Intelligence emits signal and insight events.
- Research emits experimentation events.
- Investment emits portfolio and position events.
- Execution emits order and trade events.
- Notification emits delivery events.

### Consumers

Consumers may be:

- other domains,
- workflow orchestrators,
- notification services,
- analytics pipelines,
- knowledge systems.

Consumers should react to events without needing to know every detail of the producer’s implementation.

---

## 5. Why Event-Oriented Thinking Matters

Event-oriented thinking is preferred because it supports:

- loose coupling,
- better scalability,
- resilient processing,
- clear domain boundaries,
- future extensibility.

A procedural design tends to create chains of tightly coupled steps. An event-driven design allows the platform to react to meaningful changes in a more modular and maintainable way.

For example:

```text
Market Closed
  ↓
Prices Acquired
  ↓
Indicators Calculated
  ↓
Signals Generated
  ↓
Research Updated
  ↓
Portfolio Updated
  ↓
Notification Sent
```

This sequence is not just a workflow. It is a chain of domain facts that can be extended over time.

---

## 6. Event Design Guidance

Events should:

- reflect business reality,
- be understandable to non-engineers when possible,
- remain stable over time,
- be used to coordinate domain activity rather than to replace domain logic.

They should not become a dumping ground for every technical action or every implementation detail.

---

## Related Documents

- [02-domain-map.md](02-domain-map.md)
- [03-ubiquitous-language.md](03-ubiquitous-language.md)
- [05-dependency-rules.md](05-dependency-rules.md)
