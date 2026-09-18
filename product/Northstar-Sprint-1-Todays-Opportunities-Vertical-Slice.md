# Northstar Sprint 1 - Today's Opportunities Vertical Slice

## Status

**Ready for Development**

Sprint 1 delivers Northstar's first complete user-facing vertical slice: a user opens Northstar, immediately sees today's ranked opportunities, understands a selected recommendation, and saves an asset to a watchlist.

This sprint implements product value across the established platform boundaries. It does not redesign Northstar Architecture v2.0 and does not introduce execution capability.

---

## 1. Sprint Goal

A user can open Northstar and immediately see **Today's Opportunities**.

By the end of the sprint, a new user can move from a ranked opportunity to an understandable recommendation, review its confidence and risk, read a clear AI explanation, and add the asset to a basic watchlist.

The sprint optimizes for a complete intelligence experience, not framework completeness or trading workflow.

---

## 2. Primary User Story

> As an investor, I want Northstar to proactively identify today's best opportunities, so that I know where to focus my attention without manually scanning hundreds of assets.

### Acceptance Criteria

- Opening Northstar presents a Today's Opportunities dashboard without requiring a search, broker connection, or portfolio connection.
- Opportunities are ranked and presented as concise recommendation cards.
- Each card shows an asset, recommendation, confidence, risk, and a short reason.
- A user can select an opportunity and view its complete recommendation detail.
- Recommendation detail displays recommendation, confidence, risk, reasoning, supporting signals, and AI explanation.
- A user can add an asset from the dashboard or recommendation detail to a basic watchlist.
- The watchlist displays saved assets and their current recommendation context.
- The feature works end to end through Domain, Application, API, and Web boundaries.
- No broker integration, order management, or trade execution is required to complete the user journey.

---

## 3. User Journey

```text
Open Northstar
  ↓
Today's Opportunities Dashboard
  ↓
Select an Opportunity
  ↓
Recommendation Details
  ↓
Confidence
  ↓
Risk
  ↓
AI Explanation
  ↓
Add to Watchlist
```

### Open Northstar

The user lands directly on the intelligence dashboard. Value is visible before any setup action.

### Today's Opportunities Dashboard

The user sees an ordered list of opportunities requiring attention today. The ranking provides focus; confidence, risk, and a concise reason provide the context necessary to decide whether to investigate further.

### Recommendation Details

The user opens an opportunity to understand its recommendation, supporting signals, confidence, risk, and AI explanation. The detail page is the trust-building surface of the slice.

### Add to Watchlist

The user saves an asset for continued attention. The initial watchlist supports saving and viewing assets with their latest available recommendation context.

Broker execution is intentionally absent from every stage of this journey.

---

## 4. Sprint Features

### In Scope

- Today's Opportunities Dashboard
- Ranked Recommendation Cards
- Recommendation Detail view
- Confidence score display
- Risk score display
- Concise recommendation reason
- Supporting signals display
- AI explanation display
- Basic Watchlist: add and view saved assets
- API endpoints required to serve opportunities, details, and watchlist actions
- Minimum market-data ingestion support needed to provide a demonstrable opportunity set

### Out of Scope

- Broker integration
- Execution
- Portfolio synchronization
- Order management
- Trade recording
- Backtesting
- Notifications and alert delivery
- Performance analytics
- Multi-broker support
- Automated execution
- Portfolio insights
- Recommendation history beyond the current demonstrable recommendation context

Recent-alert functionality, advanced market summaries, and top-mover discovery remain MVP capabilities for later sprint planning. Sprint 1 keeps the slice deliberately focused on opportunities, explanation, and watchlist action.

---

## 5. Repository Responsibilities

### northstar-core

Provide the approved Domain functionality required to represent the market-observation inputs and the business intelligence concepts needed for a recommendation. Core retains ownership of market context, observations, Strategy decision concepts, and all business rules.

Sprint 1 must not move recommendation meaning, confidence meaning, risk meaning, or signal meaning into Application, API, or Web merely for delivery convenience.

### northstar-application

Coordinate the recommendation workflow and basic watchlist workflow as Application use cases. Application retrieves approved Domain context through ports, invokes approved Domain behavior, and returns application outcomes for the API boundary.

Application does not own market observations, Strategy policy, or other Domain meaning. Execution coordination is not part of this sprint.

### northstar-api

Expose the public recommendation and watchlist boundary. API translates external requests into Application requests and Application outcomes into external responses. It does not calculate recommendations or duplicate Domain rules.

### northstar-web

Deliver the Intelligence Dashboard, Recommendation Card, Recommendation Detail, and Watchlist interaction. Web presents API-provided intelligence and lets users save assets without directly accessing Domain or Infrastructure concerns.

### northstar-infrastructure

Provide only the minimum technical support required to ingest or supply market-data inputs for a demonstrable opportunity set. It may implement the required Application ports and data-source adapters, but it does not contain recommendation business rules.

No broker, exchange, execution, FIX, REST trading client, WebSocket trading client, persistence platform, or messaging platform work is required beyond the minimum support needed by the Sprint 1 demonstration.

---

## 6. API Contract

Sprint 1 requires a narrow public API for the user journey.

### `GET /recommendations`

Returns today's ranked opportunities in a presentation-neutral public representation. Each opportunity communicates the asset reference, recommendation, confidence, risk, concise reason, and information needed to select the detail view.

### `GET /recommendations/{symbol}`

Returns the selected recommendation detail. It communicates the asset, recommendation, confidence, risk, reasoning, supporting signals, AI explanation, and available historical context.

### `GET /watchlist`

Returns the user's saved watchlist assets and their current available recommendation context.

### Watchlist Add Action

Provides a public action that adds a selected asset to the user's basic watchlist. The action communicates the resulting Application outcome rather than exposing Domain or persistence models.

These endpoints are conceptual contracts. API transport formats, authentication details, persistence mechanisms, and implementation choices are outside this sprint plan.

---

## 7. User Interface

### Dashboard

The dashboard prioritizes scanning and decision support. It presents:

- **Today's Opportunities** as the first visible content.
- **Recommendation Cards** ordered by opportunity ranking.
- **Asset** identity and current recommendation.
- **Confidence** shown consistently for every opportunity.
- **Risk** shown consistently for every opportunity.
- **Reason** as a short explanation of why the opportunity is surfaced.
- **Watchlist shortcut** for immediate save action.

The initial UI is intentionally simple. It should make the opportunity, confidence, risk, and explanation easy to compare without forcing a user into dense charts, configuration, or execution flows.

### Recommendation Detail

The detail view expands a selected card into a clear explanation surface. It presents recommendation, confidence, risk, reasoning, supporting signals, AI explanation, and available historical context. The user can add the asset to the watchlist from this view.

### Watchlist

The watchlist is a compact saved-assets view. It lets a user revisit assets they chose to monitor and see the latest available recommendation context.

---

## 8. Data Flow

```text
Market Data
  ↓
Recommendation Engine
  ↓
Application
  ↓
API
  ↓
Dashboard
```

### Market Data

Market Data provides the observation context needed for an opportunity. It remains an observational Domain responsibility and does not become a user-facing recommendation by itself.

### Recommendation Engine

The Recommendation Engine applies the approved intelligence and Strategy-oriented business capability to available market context, producing recommendation information with confidence, risk, reasoning, and supporting signals. Its design must respect Domain ownership and is not defined by this sprint plan.

### Application

Application coordinates the recommendation and watchlist use cases. It retrieves the necessary information through ports, invokes approved Domain behavior, and produces Application outcomes without owning recommendation business meaning.

### API

API exposes the required public endpoints and translates between delivery-specific representations and Application contracts.

### Dashboard

Web renders opportunities and details into an understandable, actionable experience. The user sees a ranked opportunity, understands the explanation, and can save the asset to the watchlist.

---

## 9. Definition of Done

Sprint 1 is complete when:

- A user can open Northstar and see Today's Opportunities immediately.
- Recommendation Cards display asset, recommendation, confidence, risk, and reason.
- A user can view a complete recommendation detail.
- Confidence and risk are clearly displayed.
- AI explanation and supporting signals are displayed.
- A user can add an asset to a basic watchlist.
- A user can view saved watchlist assets with current available recommendation context.
- The complete flow works from market-data input through Application, API, and Dashboard.
- The feature can be demonstrated without broker integration, execution, portfolio synchronization, or order management.
- Repository boundaries remain consistent with Northstar Architecture v2.0.

---

## 10. Risks

### Market Data Quality

Incomplete, stale, or inconsistent input can reduce the relevance of surfaced opportunities. Sprint work must make the input source and freshness visible enough to assess a demonstration's credibility.

### Recommendation Quality

Early ranking and recommendation logic may not yet be sufficiently selective or useful. The sprint should use a constrained, demonstrable opportunity set and gather feedback on relevance rather than claiming predictive certainty.

### AI Explanation Quality

Explanations may be vague, unsupported, or disconnected from displayed signals. The user experience must connect every explanation to visible reasoning and supporting signals.

### Performance

Recommendation retrieval and the dashboard must feel immediate enough to preserve the product promise of proactive daily intelligence. Scope should favor a responsive end-to-end flow over broad asset coverage.

### Scope Expansion

Execution, portfolio synchronization, alert delivery, analytics, and advanced visualization can easily expand the sprint. They remain out of scope until the vertical slice is demonstrably complete.

---

## 11. Sprint Deliverables

- Recommendation Engine MVP sufficient to produce a demonstrable ranked opportunity set.
- Recommendation workflow in `northstar-application`.
- Recommendation and watchlist public API boundary in `northstar-api`.
- Today's Opportunities Intelligence Dashboard in `northstar-web`.
- Recommendation Card and Recommendation Detail experiences.
- Basic Watchlist add and view behavior.
- Minimum market-data ingestion support in `northstar-infrastructure`.
- Sprint-level product and engineering documentation needed to demonstrate and maintain the slice.

---

## 12. Success Metrics

Sprint 1 succeeds when a new user can:

1. Open Northstar and identify today's opportunities without manual searching.
2. Select an opportunity and understand the recommendation, confidence, risk, and reasoning.
3. Explain why Northstar surfaced the recommendation using the visible supporting signals and AI explanation.
4. Save an asset to a watchlist for later review.

The primary qualitative test is simple: a user can explain why Northstar made a recommendation, not merely repeat that one exists.

---

## 13. Self Review

- Product-first and user-first.
- A complete vertical slice from market-data input to Dashboard is defined.
- Architecture remains unchanged.
- Every participating repository has a clear responsibility.
- Execution, broker integration, Order management, Trade recording, and portfolio synchronization remain out of scope.
- The sprint focuses on a realistic, demonstrable intelligence experience.

---

NORTHSTAR

Sprint 1

Today's Opportunities

Status

Ready for Development

Estimated Repositories

- northstar-core
- northstar-application
- northstar-api
- northstar-web
- northstar-infrastructure (market data support only)

Goal

Deliver the first end-to-end intelligence experience.

---
