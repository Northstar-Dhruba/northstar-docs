# 03. Ubiquitous Language

## Purpose

This document defines the official vocabulary of Northstar. Its purpose is to eliminate ambiguity across engineering, product, research, and operations.

The vocabulary in this document is the shared language for architecture discussions, domain modeling, and implementation planning.

---

## 1. Principles of the Language

The language should be:

- precise,
- business-oriented,
- consistent across domains,
- free from implementation-specific jargon.

Terms should describe meaning first and technical implementation second.

---

## 2. Core Terms

| Term | Meaning | Why it exists | Related concepts |
| --- | --- | --- | --- |
| Market | A trading or financial environment in which instruments are observed and evaluated. | Provides the context for all market activity. | Exchange, Session, Asset |
| Exchange | A venue or entity where instruments are listed or traded. | Distinguishes the source and structure of market activity. | Market, Instrument |
| Trading Session | A defined period in which market activity is available or relevant. | Enables temporal context and operational rules. | Market, Calendar |
| Asset | A financial thing that can be observed, owned, or evaluated. | Provides the primary object of analysis. | Instrument, Portfolio |
| Instrument | A specific financial product or security representation. | Makes the platform precise about what is being tracked. | Asset, Stock, Option |
| Stock | An equity instrument representing ownership in a company. | A common instrument type that requires special semantics. | Asset, Instrument |
| ETF | An exchange-traded fund representing a basket or strategy exposure. | Distinguishes pooled investment products. | Asset, Instrument |
| Option | A derivative instrument with rights and obligations tied to an underlying asset. | Requires separate modeling from simple equity instruments. | Future, Derivative |
| Future | A standardized contract for future delivery or settlement. | Requires distinct market and settlement semantics. | Option, Instrument |
| Crypto | A digital asset class that differs from traditional securities in structure and market behavior. | Represents a distinct domain of financial intelligence. | Asset, Market |
| Price | The value of an instrument at a point in time. | Serves as the most fundamental market observation. | Quote, Candle |
| Quote | A market observation containing price and volume context at a moment in time. | Distinguishes a point-in-time market view from broader series data. | Price, Candle |
| Candle | A time-bounded market summary that captures open, high, low, close, and volume. | Supports charting and time-series analysis. | Price, Timeframe |
| Indicator | A computed quantitative measure derived from market or price history. | Represents systematic interpretation of data. | Signal, Pattern |
| Pattern | A recognizable structure or recurring configuration in market behavior. | Supports interpretation beyond raw metrics. | Indicator, Signal |
| Signal | A meaningful indication or interpretation that may inform action or research. | Represents a domain-level output of intelligence work. | Insight, Strategy |
| Insight | A higher-level interpretation that explains or contextualizes financial meaning. | Goes beyond raw signal generation. | Signal, Knowledge |
| Strategy | A defined framework for reasoning, research, or decision-making. | Captures reusable logic at the domain level. | Research, Signal |
| Portfolio | A collection of holdings and exposures considered together. | Represents the investor or organizational context. | Position, Investment |
| Position | A specific holding within a portfolio. | Provides granularity for performance and exposure. | Portfolio, Trade |
| Trade | The execution outcome or completed transaction associated with an order. | Represents realized action. | Order, Position |
| Order | An intention to act in the market or system. | Represents a request before execution. | Trade, Execution |
| Knowledge | The accumulated historical intelligence and performance understanding of the platform. | Enables learning and future analytics. | Insight, Research |
| Research | A structured process of evaluation, comparison, and experimentation. | Distinguishes analytical work from operational intelligence. | Experiment, Backtest |
| Experiment | A defined analytical test or study. | Represents a reproducible investigation. | Research, Backtest |
| Backtest | A historical evaluation of a strategy or idea. | Provides evidence for future strategy development. | Research, Experiment |
| Walk Forward Analysis | A testing method that evaluates performance over rolling historical periods. | Supports robustness and overfitting analysis. | Research, Backtest |
| Risk | The exposure to loss or deviation from expectation. | A core financial concept that must be modeled explicitly. | Exposure, Position |
| Exposure | The degree of sensitivity or commitment a portfolio or position has to a market or factor. | Represents risk posture and concentration. | Risk, Portfolio |
| Confidence | The degree of certainty associated with an insight, signal, or interpretation. | Helps distinguish strong and weak evidence. | Signal, Insight |
| Money | A financial value expressed in a defined unit and currency. | Reduces ambiguity in value handling. | Price, Quantity |
| Timeframe | The interval or period used to interpret market data. | Allows consistent analysis across scales. | Candle, Indicator |
| Notification | A communication intended to inform or alert a stakeholder. | Distinguishes delivery from decision-making. | Alert |
| Alert | A specific event-driven notification of importance. | Enables focused communication. | Notification |

---

## 3. Distinctions That Must Remain Clear

### Intelligence vs Research

- Intelligence produces interpretation from available information.
- Research evaluates ideas and strategies methodically.

These are related, but they are not the same thing.

### Acquisition vs Intelligence

- Acquisition gathers information.
- Intelligence interprets information.

Acquisition should never perform calculations as a primary responsibility.

### Execution vs Notification

- Execution represents actioning and trade-related concepts.
- Notification represents communication and delivery.

Notification is not execution and should not own execution semantics.

### Knowledge vs Insight

- Insight is a specific interpretation.
- Knowledge is the accumulated body of understanding over time.

Knowledge is the durable memory of the platform.

---

## 4. Usage Guidance

When discussing the platform, engineers should prefer the domain terms above over vague or overloaded terms such as “analysis” or “data” when precision is needed.

For example:

- Use “Intelligence” when describing derived understanding.
- Use “Research” when describing structured evaluation and experimentation.
- Use “Acquisition” when describing information gathering.
- Use “Knowledge” when describing historical accumulated understanding.

---

## Related Documents

- [02-domain-map.md](02-domain-map.md)
- [04-domain-events.md](04-domain-events.md)
- [05-dependency-rules.md](05-dependency-rules.md)
