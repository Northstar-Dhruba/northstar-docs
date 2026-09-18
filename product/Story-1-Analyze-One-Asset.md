# Story 1 - Analyze One Asset

## Status

**Ready for Development**

Story 1 delivers Northstar's smallest complete intelligence experience. A user enters one asset symbol, requests analysis, and receives an understandable opportunity assessment with recommendation, confidence, risk, explanation, and supporting signals.

This story is product-first and architecture-preserving. It creates no execution capability and does not redesign Northstar Architecture v2.0.

---

## 1. User Story

> As an investor, I want to enter the symbol of one asset, so that Northstar explains whether it currently represents an attractive opportunity.

---

## 2. Story Goal

Deliver the first complete user-facing intelligence experience:

```text
Open Northstar
  ↓
Enter Symbol
  ↓
Analyze
  ↓
Recommendation
  ↓
Confidence
  ↓
Risk
  ↓
Explanation
```

The feature demonstrates Northstar's primary value before any broad market scanning, watchlist management, alerting, portfolio analysis, or execution capability is introduced.

---

## 3. User Input

The user provides the smallest context necessary to identify one asset for analysis:

- **Symbol:** The asset symbol the user wants Northstar to analyze.
- **Market or venue context:** Required only when a symbol alone is insufficient to identify the market manifestation being analyzed.

The experience should request market context only when necessary. Users must not provide technical provider information, broker details, portfolio holdings, order information, indicators, confidence, risk, or a desired recommendation outcome.

Northstar uses the input to identify the relevant asset and available market context. It remains responsible for producing the analysis; the user does not need to construct it.

---

## 4. User Output

The result is intentionally small, clear, and explainable. The user sees:

- **Asset:** The asset Northstar analyzed.
- **Recommendation:** A concise current opportunity assessment.
- **Confidence:** How strongly Northstar supports the assessment.
- **Risk:** The relevant caution or uncertainty context.
- **Summary:** A short statement of why the asset deserves or does not deserve attention.
- **Detailed reasoning:** A clear explanation connecting the assessment to observed context.
- **Supporting signals:** The inputs or conditions that support the reasoning.

The result is intelligence, not an instruction to trade. It does not create an Order, Trade, Position, Portfolio change, or execution action.

---

## 5. User Journey

### Open Northstar

The user opens Northstar and sees a focused analysis entry point.

### Enter Symbol

The user enters one symbol and, only when required for unambiguous identification, chooses market or venue context.

### Analyze

The user requests analysis. Northstar retrieves the available market-observation context and coordinates the intelligence workflow.

### Review Recommendation

Northstar shows the current recommendation alongside confidence, risk, summary, and supporting signals.

### Understand Explanation

The user reads the detailed reasoning and AI explanation to understand why Northstar reached the assessment.

The user can leave with an informed view of the asset. No additional action is required for the story to succeed.

---

## 6. Repository Responsibilities

### northstar-core

Owns the approved Domain concepts required to represent market context, observations, Strategy-oriented decision behavior, and intelligence meaning. It owns all business rules, validation, and Domain semantics used by the analysis.

Story 1 must not move recommendation, confidence, risk, signal, or market-observation meaning outside the Domain merely to serve the user interface.

### northstar-application

Coordinates the Analyze One Asset use case. It accepts an application request, obtains the necessary market context through approved ports, invokes approved Domain behavior, and produces an application outcome for delivery.

Application coordinates the workflow and does not own the recommendation's business meaning or duplicate Domain decisions.

### northstar-api

Provides the public analysis boundary. It translates the user-provided symbol and optional market context into an Application request, invokes the Application use case, and translates the resulting Application outcome into a client-facing response.

API does not calculate recommendations, confidence, risk, or explanations.

### northstar-web

Provides the focused user experience: symbol entry, optional market selection when needed, analysis trigger, and the result view for recommendation, confidence, risk, summary, detailed reasoning, and supporting signals.

Web displays intelligence returned by the API. It does not call Domain code or calculate analysis results.

### northstar-infrastructure

Provides only the minimum technical support required to obtain market-data inputs for the requested asset and fulfill the approved Application access boundary. It may translate external data-provider responses into Application-facing results.

Infrastructure does not own market intelligence, recommendation logic, confidence, risk, or explanations.

---

## 7. Out of Scope

Story 1 explicitly excludes:

- Watchlist
- Alerts
- Portfolio features
- Execution
- Broker integration
- Market scanning
- Opportunity ranking
- Order management
- Trade recording
- Portfolio synchronization
- Backtesting
- Performance analytics
- Multi-broker support

This story analyzes one requested asset. It does not discover assets, rank opportunities, save user preferences, or place trades.

---

## 8. Acceptance Criteria

The story is complete when:

- A user can enter one valid asset symbol.
- The user can provide market or venue context only when required to identify the asset.
- The user can request an analysis without connecting a broker, portfolio, or watchlist.
- Northstar returns an analysis for the requested asset when sufficient supported market context is available.
- The result displays the asset, recommendation, confidence, risk, summary, detailed reasoning, and supporting signals.
- A user can explain why Northstar made the recommendation by referring to the displayed reasoning and supporting signals.
- Confidence and risk are visible and understandable without navigating to another feature.
- No execution, order, trade, portfolio, watchlist, alert, scanner, or ranking behavior is exposed.
- The complete path works through `northstar-core`, `northstar-application`, `northstar-api`, `northstar-web`, and minimum `northstar-infrastructure` market-data support.

---

## 9. Definition of Done

Story 1 is done when:

- The user journey from symbol entry to explanation is demonstrable end to end.
- The UI shows an analysis result with all required output elements.
- The API exposes only the public boundary required for one-asset analysis.
- Application coordinates the use case without owning Domain intelligence meaning.
- Core retains ownership of all business rules and analysis semantics.
- Infrastructure is limited to the market-data access required for the requested asset.
- The feature has appropriate automated coverage at Domain, Application, API, and Web boundaries.
- Relevant repository checks pass.
- Scope remains limited to Analyze One Asset.

---

## 10. Implementation Plan

### Recommended Order

1. Define the minimum Domain intelligence behavior and output needed to analyze one asset.
2. Define the Application request, outcome, and market-data access boundary for the single-asset use case.
3. Implement the Application Analyze One Asset workflow.
4. Implement minimum Infrastructure market-data support for a requested asset.
5. Expose the single analysis public API boundary.
6. Build the symbol-entry and analysis-result Web experience.
7. Add end-to-end coverage and demonstrate the complete user journey.

### northstar-core Tasks

- Define only the approved Domain functionality needed to interpret available market-observation context for one identified asset.
- Preserve Domain ownership of recommendation, confidence, risk, reasoning, and supporting-signal meaning.
- Add focused Domain contract coverage for the new intelligence behavior.

### northstar-application Tasks

- Define the narrow Application contract for Analyze One Asset.
- Define the minimum Application port required to request market context for one asset.
- Implement the use-case orchestration that invokes approved Domain behavior and returns an Application outcome.
- Add focused application tests using substitute port implementations.

### northstar-infrastructure Tasks

- Implement only the market-data access adapter required to supply the requested asset's observation context through the approved Application port.
- Add adapter tests without embedding recommendation business rules.

### northstar-api Tasks

- Implement the public analysis endpoint for one asset.
- Translate delivery input to the Application request and the Application outcome to the public response.
- Add endpoint coverage for successful analysis, unknown asset, and unavailable analysis context.

### northstar-web Tasks

- Implement the symbol-entry experience.
- Implement the analysis trigger and loading, success, and unavailable-result states.
- Implement the result view for recommendation, confidence, risk, summary, detailed reasoning, and supporting signals.
- Add user-flow coverage for entering a symbol and understanding the returned analysis.

---

## 11. Self Review

- Product-first and user-first.
- Defines a complete vertical slice rather than a framework-first milestone.
- Achievable in one sprint by limiting scope to one requested asset.
- Preserves Northstar Architecture v2.0.
- Keeps Domain business meaning, Application orchestration, Infrastructure technical access, API delivery, and Web presentation separate.
- Excludes watchlist, alerts, market scanning, ranking, portfolio, and execution work.

---

STORY 1

Analyze One Asset

Status

Ready for Development

Repositories

northstar-core

northstar-application

northstar-api

northstar-web

Goal

Deliver the first complete intelligence experience.

---
