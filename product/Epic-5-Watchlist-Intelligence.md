# Epic 5 - Watchlist Intelligence

## Status

**Ready for Development**

Epic 5 turns Northstar's one-asset intelligence workflow into personalized intelligence. Users can maintain a focused list of assets they choose, run the existing analysis pipeline across those assets, and recognize when a recommendation changes over time.

This epic does not redesign Northstar Architecture v2.0. It reuses the completed Analyze One Asset, Explain Recommendation, Real Market Intelligence, and Evidence & Transparency capabilities.

---

## 1. Product Goal

Enable users to monitor a personalized list of assets using the existing Northstar intelligence pipeline.

Northstar should analyze user-selected assets rather than the entire market. The watchlist is a user-directed collection, not a market scanner or ranked opportunity universe.

---

## 2. User Story

> As an investor, I want to maintain a watchlist of assets, so that Northstar can automatically analyze them and highlight meaningful changes.

---

## 3. Product Value

A watchlist gives users continuity. Instead of repeatedly entering symbols one at a time, they can retain the assets relevant to their decisions and return to current intelligence for those assets.

The value is personalized attention:

- the user chooses what matters;
- Northstar reuses the existing intelligence pipeline;
- each watched asset retains recommendation and explanation context;
- meaningful recommendation changes are visible over time.

The watchlist does not determine recommendation policy, rank the whole market, or authorize execution.

---

## 4. Scope Decomposition

Each story is independently demonstrable and delivers a complete, narrow product behavior.

### Story 5.1 - Manage Watchlist

**User story:** As an investor, I want to add and remove assets from my watchlist, so that I can control which assets Northstar monitors.

**Goal:** Provide a basic user-owned watchlist for one user and supported asset symbols.

**Demonstrable outcome:** A user can add a symbol, see it in the watchlist, prevent duplicate entries, and remove it.

**In scope:**

- add one supported asset;
- view the user's watched assets;
- remove a watched asset;
- preserve watchlist membership between views;
- handle duplicate and unknown symbols clearly.

**Out of scope:** recommendation calculation, ranking, scanning, alerts, portfolio linkage, and execution.

**Dependencies:** Completed Analyze One Asset asset identity and the minimum user ownership/persistence boundary required to retain a list.

### Story 5.2 - Analyze Watchlist

**User story:** As an investor, I want Northstar to analyze every asset on my watchlist, so that I can review current intelligence without entering each symbol manually.

**Goal:** Apply the existing Analyze One Asset workflow to the user's selected assets.

**Demonstrable outcome:** A user can request watchlist analysis and see each watched asset's current Recommendation, RecommendationExplanation, and available evidence.

**In scope:**

- request analysis for the current user's watchlist;
- invoke the existing single-asset analysis workflow per watched asset;
- display each result independently;
- preserve source and observation-time context;
- communicate unavailable results without hiding successful asset results.

**Out of scope:** market-wide discovery, opportunity ranking, automatic trading, portfolio analysis, scheduling, and caching.

**Dependencies:** Story 5.1; completed Epics 1 through 4; bounded multi-item orchestration using the existing single-asset use case.

### Story 5.3 - Highlight Recommendation Changes

**User story:** As an investor, I want to see when a watched asset's recommendation changes, so that I can focus on meaningful developments.

**Goal:** Compare a current watched-asset recommendation with its previously recorded recommendation and make a change visible.

**Demonstrable outcome:** A user can identify that an asset changed from BUY to HOLD, HOLD to SELL, or another valid action transition, with the relevant observation context.

**In scope:**

- retain the prior recommendation state for a watched asset;
- compare the current and previous recommendation actions;
- show changed versus unchanged status;
- show the current observation time and evidence context;
- handle the first observation as having no prior recommendation.

**Out of scope:** predicting future changes, ranking changes, alert delivery, confidence, risk, execution, and market scanning.

**Dependencies:** Stories 5.1 and 5.2; recommendation history semantics; existing Recommendation, explanation, and evidence contracts.

---

## 5. User Journey

```text
Analyze One Asset
  ↓
Add Asset to Watchlist
  ↓
Return to Watchlist
  ↓
Analyze Watched Assets
  ↓
Review Recommendation + Explanation + Evidence
  ↓
See Recommendation Changes
```

### Maintain Watchlist

The user adds an asset from the existing analysis experience or watchlist control, views the saved symbol, and removes it when it is no longer relevant.

### Analyze Watchlist

The user requests an updated view for their selected assets. Northstar applies the existing single-asset workflow to each watched symbol. Each asset remains a distinct analysis result with its own recommendation, explanation, evidence, source, and observation time.

### Highlight Changes

When a current recommendation differs from the prior recorded recommendation, the watchlist makes that transition clear. A first observation is shown as new rather than incorrectly described as a change.

---

## 6. Responsibility Boundaries

### Domain

The Domain owns watchlist business meaning where a genuine business concept is required, including membership identity and recommendation-change semantics. It continues to own Recommendation and RecommendationExplanation meaning.

The Domain does not scan the market, choose watchlist membership, rank opportunities, or execute trades.

### Application

The Application coordinates user watchlist workflows and invokes the existing Analyze One Asset use case for each selected symbol. It owns orchestration, not recommendation policy.

### Infrastructure

Infrastructure persists watchlist membership and relevant recommendation history through narrow Application contracts. It continues to acquire market observations through the existing source boundary. It does not introduce a generalized market-data or notification framework.

### API

The API exposes only the endpoints required by Stories 5.1 through 5.3, validates transport input, delegates to Application, and serializes results. API code does not calculate recommendation changes or inspect market signals.

### Web

The Web application adds the minimum watchlist views and controls needed to maintain the list, review its analyses, and distinguish changed recommendations. It does not create a scanner, dashboard for the whole market, or execution surface.

---

## 7. Product Principles

- The user controls watchlist membership.
- Watchlist analysis reuses the existing Analyze One Asset workflow.
- Every watched recommendation remains explainable.
- Every explanation remains supported by evidence.
- Evidence retains its source and observation time.
- A watchlist is personalized monitoring, not market-wide discovery.
- Recommendation changes are observations of state transitions, not predictions.
- The user always makes the final investment decision.
- No watchlist result automatically creates an order or execution action.
- Unavailable data must not be represented as a recommendation.

---

## 8. Repository Responsibilities

### northstar-core

- Define only the Domain concepts needed for watchlist membership and recommendation-change meaning.
- Preserve Recommendation, RecommendationExplanation, and evidence semantics.
- Model a recommendation transition as changed, unchanged, or not previously observed where required.
- Add focused tests for membership and transition invariants.
- Do not add scanner, ranking, confidence, risk, execution, or alert policy.

### northstar-application

- Coordinate add, view, and remove watchlist workflows.
- Coordinate analysis of selected symbols by reusing the existing AnalyzeAssetUseCase.
- Preserve each asset's Recommendation, RecommendationExplanation, evidence, source, and observation time.
- Coordinate comparison of current and previous recommendation states without duplicating Strategy policy.
- Define narrow persistence ports only where required by the stories.
- Add application contract coverage for partial failure and deterministic orchestration.

### northstar-infrastructure

- Implement the minimum watchlist persistence adapter.
- Implement the minimum recommendation-history persistence needed to identify changes.
- Preserve existing market-observation acquisition behavior.
- Avoid caching, scheduling, streaming, notification frameworks, and multi-provider support.

### northstar-api

- Expose only the transport endpoints required by the three watchlist stories.
- Validate symbols and watchlist requests.
- Delegate to Application and serialize structured results.
- Keep recommendation, explanation, evidence, and change semantics out of the router.
- Preserve existing single-asset API behavior.

### northstar-web

- Add the smallest controls for add, view, and remove.
- Provide a focused watchlist view using existing result presentation patterns.
- Show current recommendation, explanation, evidence, and recommendation-change status.
- Keep the single-asset Analyze Asset workspace intact.
- Do not add a market scanner, ranking surface, charting system, or execution workflow.

---

## 9. Out of Scope

- Today's Opportunities.
- Confidence.
- Risk.
- Execution.
- Portfolio.
- Market Scanner.
- Opportunity Ranking.
- Alerts and notifications.
- Scheduling.
- Caching.
- Streaming or WebSockets.
- Multi-provider market data.
- Automatic trading decisions.

---

## 10. Acceptance Criteria

Epic 5 succeeds when:

- users can add supported assets to a personal watchlist;
- users can view and remove watched assets;
- users can analyze every watched asset through the existing Analyze One Asset workflow;
- each watched result preserves Recommendation, RecommendationExplanation, and factual evidence;
- unavailable data is communicated without inventing a result or preventing available results from being shown;
- users can clearly see when a watched asset's recommendation changed over time;
- first-time observations are not falsely represented as changes;
- the user remains the final decision-maker;
- no market-wide scanner, ranking, execution, confidence, or risk behavior is introduced.

---

## 11. Definition of Done

Epic 5 is complete when users can maintain a watchlist, analyze every watched asset, and clearly see recommendation changes over time.

Completion requires:

- Story 5.1 independently demonstrable with persistent membership behavior.
- Story 5.2 independently demonstrable with reused single-asset analysis and partial-failure handling.
- Story 5.3 independently demonstrable with accurate recommendation transition display.
- Existing recommendation, explanation, evidence, and single-asset behavior preserved.
- Focused Domain, Application, API, Infrastructure, and Web coverage at affected boundaries.
- Relevant repository checks passing.

---

## 12. Recommended Delivery Order

1. Story 5.1 - Manage Watchlist.
2. Story 5.2 - Analyze Watchlist.
3. Story 5.3 - Highlight Recommendation Changes.

Each story should be reviewed independently before the next begins. This keeps the personalized workflow demonstrable at every step and prevents watchlist scope from becoming an early scanner or ranking initiative.

---

EPIC 5

Watchlist Intelligence

Status

Ready for Development

Goal

Turn one-asset intelligence into daily personalized intelligence.

---
