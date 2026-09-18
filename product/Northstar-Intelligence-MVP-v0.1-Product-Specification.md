# Northstar Intelligence MVP v0.1 Product Specification

## Status

**Product specification for the first user-facing Northstar experience.**

This document defines user value, scope, and outcomes for Northstar Intelligence MVP v0.1. It does not change Northstar Architecture v2.0 or define implementation details. Execution remains an optional downstream capability and is not required for the MVP.

---

## 1. Product Vision

Northstar is an AI-powered Financial Intelligence Platform that helps people observe markets, analyze opportunities, understand the reasoning behind them, and decide what deserves attention.

Its primary purpose is to answer a simple daily question:

> What should I pay attention to today?

Northstar is not a broker and is not an exchange. It does not need to place trades to deliver value. Its core product promise is useful, transparent intelligence: opportunities are surfaced proactively, explained clearly, and accompanied by visible confidence and risk context.

Execution may become an optional downstream capability when a user wants to act on an opportunity. It is not the first product experience and is not a requirement for MVP success.

---

## 2. Target Users

### Initial Audience

The MVP serves self-directed market participants who need a focused daily intelligence workflow:

- Swing traders looking for multi-day opportunities and changing momentum.
- Position traders monitoring trends, breakouts, reversals, and risk changes.
- Long-term equity investors seeking signals that change the case for an asset they follow.
- Crypto investors who need a concise way to monitor fast-moving assets and confidence changes.
- Equity investors who want understandable recommendations rather than raw data alone.

### Goals

Users want to:

- discover relevant opportunities without manually screening every asset;
- understand why an opportunity is worth attention;
- judge confidence and risk before investing time or capital;
- save important assets for ongoing monitoring; and
- return to a trusted daily view of changing market intelligence.

### Pain Points

Users commonly face information overload, fragmented tools, opaque signals, and insufficient context around recommendations. They may see a price move without knowing whether it reflects momentum, a reversal, a risk change, or noise. They also lack a reliable way to prioritize a watchlist each day.

### Daily Workflow

A typical user opens Northstar at the start of a market session or during a planned review. They scan the highest-ranked opportunities, open the most relevant recommendation, assess its explanation, confidence, risk, and supporting signals, then either ignore it, save the asset to a watchlist, or plan to execute later outside the MVP.

---

## 3. MVP Goal

The MVP must make Northstar useful before it supports execution.

Its goal is to provide a daily intelligence experience that proactively identifies and explains market opportunities. A successful user can quickly answer:

- What opportunities are most relevant today?
- Why are they relevant?
- How confident is Northstar?
- What risk should I consider?
- What should I keep watching?

The MVP prioritizes attention, understanding, and trust over trading workflow complexity.

---

## 4. User Journey

The canonical MVP journey is:

```text
Open Northstar
  ↓
View today's opportunities
  ↓
Select an opportunity
  ↓
Read the recommendation
  ↓
Understand reasoning and supporting signals
  ↓
Review confidence and risk
  ↓
Choose an action
  ├── Ignore
  ├── Add to Watchlist
  └── Execute Later
```

### Open Northstar

The first screen immediately presents intelligence. Users do not need to search, configure a screener, or connect a broker before seeing value.

### View Today's Opportunities

The user scans a ranked list of opportunities selected for current relevance. Ranking makes prioritization explicit and reduces information overload.

### Understand a Recommendation

The user opens a recommendation to see the asset, recommendation, supporting signals, plain-language reasoning, confidence, risk, and any available historical context.

### Take a Non-Execution Action

The user may ignore the opportunity, add it to a watchlist for ongoing monitoring, or mark it for later execution planning. The MVP does not submit orders or require broker connectivity.

---

## 5. MVP Features

### Included in v0.1

- Intelligence Dashboard
- Today's Opportunities
- Opportunity Ranking
- Recommendation Detail
- Confidence indicator
- Risk indicator
- AI Explanation
- Supporting Signals
- Watchlist
- Recent Alerts
- Market Summary
- Top Movers

### Explicitly Deferred

The following are not part of MVP v0.1:

- broker integration;
- automated execution;
- portfolio synchronization;
- order management;
- backtesting;
- risk engine;
- performance analytics; and
- multi-broker support.

These capabilities may be introduced in later releases only when they support the established intelligence-first product direction.

---

## 6. Intelligence Dashboard

The Intelligence Dashboard is the primary MVP experience. It proactively surfaces intelligence so users receive value immediately on opening Northstar.

### Today's Opportunities

A ranked set of current opportunities that deserve attention. Each opportunity should provide enough visible context for a user to decide whether to open the full recommendation.

### Opportunity Ranking

Ranking communicates relative priority across opportunities. It is not a promise of outcome and must be accompanied by confidence and risk context.

### Watchlist

The dashboard shows assets the user has chosen to monitor and their most recent recommendation or alert state.

### Recent Alerts

Recent alerts summarize changes that may require attention, including momentum breakouts, trend reversals, risk increases, confidence changes, and relevant news impact.

### Market Summary

A concise view of broad market context that helps users interpret the opportunity list.

### Top Movers

A focused view of meaningful market movement that can help users discover assets requiring further review. Movement alone is not presented as a recommendation.

---

## 7. Recommendation Detail

The Recommendation Detail page is the explanation surface for a single opportunity. It must help the user understand the recommendation rather than merely receive it.

The page includes:

- Asset: the instrument or asset being considered.
- Recommendation: the current intelligence recommendation in user-facing language.
- Confidence: the degree of confidence associated with the recommendation.
- Risk: the risk context a user should consider.
- Reasoning: the concise explanation of why the opportunity is surfaced.
- Supporting Signals: observable or derived inputs supporting the recommendation.
- AI Explanation: a clear, transparent narrative that connects reasoning to the visible signals.
- Historical Context: prior relevant observations or recommendation context when available.

A future version may include an **Execute** action. In v0.1, any such action is a future affordance only and must not require broker integration, order management, or automated execution.

---

## 8. Watchlist

The Watchlist enables a user to save assets that matter to their ongoing review.

Users can:

- save assets;
- monitor saved assets; and
- receive updated recommendations and alerts for those assets.

Northstar continuously analyzes watchlist assets within the available MVP intelligence capability. The watchlist is a user attention tool, not a portfolio, holdings ledger, or order-management system.

---

## 9. Alerts

Alerts surface meaningful changes that require user attention. They should be concise, timely, and actionable by directing the user to the relevant recommendation or asset context.

Initial alert categories include:

- Momentum breakout
- Trend reversal
- Risk increase
- Confidence change
- News impact

An alert does not require an execution action. Its primary purpose is to help a user revisit an opportunity with updated context.

---

## 10. Success Metrics

MVP success is measured by evidence that users receive, understand, and return for intelligence value.

Key indicators include:

- Daily active review: users open Northstar regularly, especially during their planned market-review window.
- Opportunity engagement: users open and review surfaced opportunities.
- Explanation engagement: users view reasoning, confidence, risk, and supporting signals.
- Watchlist adoption: users save assets and return to monitor them.
- Alert engagement: users open alerts and revisit updated recommendations.
- Trust signal: users report that recommendations are understandable and sufficiently transparent to inform their own decisions.
- Return rate: users return for updated intelligence rather than treating Northstar as a one-time screener.

The MVP should optimize for informed attention and user trust, not trade volume or execution conversion.

---

## 11. Roadmap

### v0.1: Intelligence MVP

- Recommendation Engine
- Intelligence Dashboard
- Today's Opportunities
- Watchlist
- Alerts
- Confidence, risk, and explainable reasoning

### v0.2: Intelligence Depth

- Portfolio Insights
- Recommendation History
- Improved AI reasoning
- Expanded historical context

### v0.3: Optional Execution Connection

- Broker Integration
- Manual execution
- Trade synchronization

Execution remains optional and must preserve Northstar's role as an intelligence platform.

### v1.0: Financial Intelligence Platform

A mature user experience that combines proactive market intelligence, understandable recommendations, personalized monitoring, deeper portfolio insight, and optional downstream execution capabilities.

---

## 12. Engineering Impact

The MVP uses the existing repository boundaries without changing platform architecture.

### northstar-core

Provides the Domain foundation for market observations, market context, Strategy decision concepts, and other approved business intelligence meaning. It remains the owner of business rules and Domain concepts.

### northstar-application

Coordinates recommendation and watchlist workflows through Application use cases. It does not own market, recommendation, or portfolio business meaning.

### northstar-api

Exposes recommendation, watchlist, alert, and market-summary capabilities through public delivery boundaries. It translates inbound requests and outbound application outcomes without embedding Domain rules.

### northstar-web

Delivers the Intelligence Dashboard, Recommendation Detail, Watchlist, and Alerts experience. It presents intelligence from the API and does not bypass the Application boundary.

### northstar-infrastructure

Is not required for MVP execution capability. It may support approved market-data ingestion or other technical integrations needed to populate intelligence inputs, while remaining separate from Domain and Application meaning.

---

## 13. Product Principles

1. **Intelligence before execution.** The product must deliver clear value before a user considers acting.
2. **Explain before recommending.** A recommendation without understandable reasoning is incomplete.
3. **Transparency over black-box AI.** Users must be able to inspect why Northstar surfaced an opportunity.
4. **Confidence is mandatory.** Every recommendation communicates its confidence context.
5. **Risk is mandatory.** Every recommendation communicates relevant risk context.
6. **Every recommendation is explainable.** Supporting signals and AI explanation must connect the recommendation to observable context.
7. **Proactive value.** The dashboard surfaces opportunities without requiring users to search first.
8. **User agency.** Users decide whether to ignore, watch, or act later; Northstar does not pressure execution.
9. **Architecture remains stable.** Product development uses the frozen Domain, Application, and Infrastructure boundaries rather than redefining them.

---

## 14. Self Review

- Product-first and user-centric.
- Architecture remains unchanged.
- Execution is optional and excluded from the MVP requirement.
- MVP scope focuses on visible user value: opportunities, explanations, confidence, risk, watchlists, and alerts.
- The product can deliver daily intelligence value without broker integration.
- Every included MVP feature has a clear user-facing purpose.

---

NORTHSTAR INTELLIGENCE MVP v0.1

Product Specification

Approved

Ready for Sprint 1

---
