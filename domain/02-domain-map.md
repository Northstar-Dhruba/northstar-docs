# 02. Domain Map

## Purpose

This document defines the bounded contexts of Northstar and explains how the platform should be organized as a set of cooperating domains.

The domain map is the architectural blueprint for how business capabilities are grouped and how responsibilities are separated.

---

## 1. Overview

Northstar is composed of several business domains that operate together while preserving clear boundaries.

The intended model is not a single large domain. It is a network of focused domains that interact through agreed contracts and events.

```text
Foundation
  └── shared concepts, identity, time, value, reference structures

Acquisition
  └── gathers market and external information

Market
  └── defines exchanges, sessions, and market context

Assets
  └── defines instruments and financial products

Intelligence
  └── produces indicators, patterns, signals, and insight

Research
  └── evaluates strategies and experiments

Knowledge
  └── accumulates historical intelligence and performance

Investment
  └── models portfolios, positions, exposure, and performance

Execution
  └── models orders, trades, broker interaction, and execution policy

Notification
  └── delivers alerts, reports, and communication

Configuration
  └── governs calendars, sessions, markets, flags, and settings
```

---

## 2. Domain Descriptions

### Foundation

**Purpose**
- Provide the shared conceptual foundation used across the platform.

**Responsibilities**
- common financial value concepts,
- identity and reference structures,
- time and calendar concepts,
- shared domain conventions.

**Non-responsibilities**
- direct market acquisition,
- strategy execution,
- UI behavior.

**Entities**
- reference identifiers,
- value objects,
- common metadata.

**Events**
- reference structures updated.

**Future Modules**
- shared pricing conventions,
- identity services,
- market metadata helpers.

---

### Acquisition

**Purpose**
- Acquire information from external and internal sources.

**Responsibilities**
- ingest market data,
- collect news and reference information,
- gather corporate actions and economic events,
- provide raw information to other domains.

**Non-responsibilities**
- calculating indicators,
- generating trading signals,
- making investment decisions,
- executing orders.

**Entities**
- source feed,
- data collection context,
- acquisition record.

**Events**
- PricesAcquired,
- NewsAcquired,
- ReferenceDataUpdated.

**Future Modules**
- Yahoo Finance integration,
- Polygon integration,
- NSE integration,
- SEC and corporate actions ingestion,
- economic calendar ingestion.

---

### Market

**Purpose**
- Define the market environment in which the platform operates.

**Responsibilities**
- exchange definitions,
- trading sessions,
- market calendars,
- market status,
- market context.

**Non-responsibilities**
- portfolio management,
- signal generation,
- execution dispatch.

**Entities**
- market,
- exchange,
- trading session.

**Events**
- MarketOpened,
- MarketClosed,
- SessionChanged.

**Future Modules**
- market holiday rules,
- session-specific calendars,
- multi-region market context.

---

### Assets

**Purpose**
- Define the financial instruments and products the platform reasons about.

**Responsibilities**
- instrument identity,
- asset classification,
- product-specific traits,
- instrument metadata.

**Non-responsibilities**
- strategy execution,
- notification,
- UI rendering.

**Entities**
- asset,
- instrument,
- stock,
- ETF,
- option,
- future,
- crypto.

**Events**
- InstrumentRegistered,
- InstrumentUpdated.

**Future Modules**
- derivatives taxonomy,
- crypto-specific metadata,
- corporate action handling.

---

### Intelligence

**Purpose**
- Produce financial intelligence from acquired information.

**Responsibilities**
- indicators,
- pattern detection,
- signal generation,
- market scoring,
- ranking,
- relative strength,
- AI-assisted insight,
- sentiment interpretation.

**Non-responsibilities**
- direct data acquisition,
- execution of orders,
- delivery of alerts as a primary responsibility.

**Entities**
- indicator,
- pattern,
- signal,
- insight,
- score.

**Events**
- IndicatorsCalculated,
- PatternsDetected,
- SignalsGenerated,
- SignalsRanked,
- InsightsGenerated.

**Future Modules**
- factor models,
- multi-signal ensembles,
- AI reasoning pipelines,
- event-based scoring.

---

### Research

**Purpose**
- Evaluate strategies and ideas in a controlled analytical environment.

**Responsibilities**
- backtesting,
- walk forward analysis,
- optimization,
- Monte Carlo simulation,
- performance evaluation,
- experimentation,
- strategy comparison.

**Non-responsibilities**
- live execution,
- publishing alerts,
- generating production signals as a direct consequence of research.

**Entities**
- experiment,
- backtest,
- optimization run,
- performance report.

**Events**
- BacktestCompleted,
- ExperimentCompleted,
- OptimizationFinished.

**Future Modules**
- scenario analysis,
- parameter sweeps,
- portfolio backtests,
- robust performance analytics.

---

### Knowledge

**Purpose**
- Preserve and organize accumulated historical understanding.

**Responsibilities**
- historical prices,
- historical indicators,
- historical signals,
- trade outcomes,
- performance summaries,
- insight accumulation.

**Non-responsibilities**
- real-time execution,
- creating new data from thin air,
- owning user-facing presentation.

**Entities**
- knowledge base,
- historical performance record,
- insight memory.

**Events**
- KnowledgeUpdated,
- PerformanceRecorded.

**Future Modules**
- semantic knowledge indexing,
- AI training and retrieval context,
- long-term analytical summarization.

---

### Investment

**Purpose**
- Represent the investment context of the platform.

**Responsibilities**
- portfolios,
- positions,
- exposure,
- performance context,
- allocation awareness,
- risk posture.

**Non-responsibilities**
- direct market acquisition,
- delivery of notifications,
- fundamental strategy formulation.

**Entities**
- portfolio,
- position,
- allocation,
- risk view.

**Events**
- PortfolioUpdated,
- PositionOpened,
- PositionClosed,
- RiskEvaluated.

**Future Modules**
- portfolio scenario analysis,
- benchmark comparison,
- allocation optimization,
- risk contribution analysis.

---

### Execution

**Purpose**
- Represent the actioning side of the platform.

**Responsibilities**
- orders,
- trades,
- fill outcomes,
- execution policy,
- broker integration concepts.

**Non-responsibilities**
- intelligence generation,
- research evaluation,
- notification delivery.

**Entities**
- order,
- trade,
- execution policy,
- broker integration context.

**Events**
- OrderPlaced,
- OrderExecuted,
- TradeCompleted.

**Future Modules**
- routing policies,
- execution cost models,
- broker-specific adapters.

---

### Notification

**Purpose**
- Deliver information and insight to users and systems.

**Responsibilities**
- alerts,
- daily reports,
- summaries,
- channel-specific communication.

**Non-responsibilities**
- generating trading logic,
- making decisions,
- performing calculations.

**Entities**
- notification,
- alert,
- report,
- delivery channel.

**Events**
- AlertSent,
- DailySummarySent.

**Future Modules**
- email delivery,
- Telegram and WhatsApp integration,
- push notifications.

---

### Configuration

**Purpose**
- Define the rule-based operating context of the platform.

**Responsibilities**
- market calendars,
- trading sessions,
- supported markets,
- feature flags,
- risk limits,
- platform settings.

**Non-responsibilities**
- creating intelligence,
- executing orders,
- user interaction.

**Entities**
- configuration setting,
- feature flag,
- market calendar,
- risk limit.

**Events**
- ConfigurationChanged.

**Future Modules**
- environment-specific settings,
- user profile preferences,
- governance rules.

---

## 3. How Domains Communicate

Domains communicate through well-defined business events and shared domain concepts.

Typical flow:

- Acquisition supplies information.
- Market and Assets provide context.
- Intelligence derives signals and insights.
- Research evaluates ideas and strategies.
- Investment reflects portfolio and performance realities.
- Notification communicates relevant outcomes.
- Configuration shapes the operating boundaries.

---

## 4. Dependency Direction

Allowed direction:

- Acquisition → Intelligence
- Market → Intelligence
- Assets → Intelligence
- Intelligence → Investment
- Research → Knowledge
- Investment → Notification
- Configuration → all domains

Forbidden direction:

- Core logic should not depend on UI concerns.
- Core logic should not depend on transport or presentation frameworks.
- Notification should not define business rules.
- Execution should not own analytical reasoning.

---

## 5. Future Expansion

The domain map should remain stable even as new capabilities are added. New domains should only be introduced when a capability has a distinct business purpose and a meaningful boundary.

---

## Related Documents

- [01-platform-vision.md](01-platform-vision.md)
- [03-ubiquitous-language.md](03-ubiquitous-language.md)
- [04-domain-events.md](04-domain-events.md)
- [05-dependency-rules.md](05-dependency-rules.md)
