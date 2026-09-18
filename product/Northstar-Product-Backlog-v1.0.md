# Northstar Product Backlog v1.0

## Status

**Official Product Roadmap**

## Architecture

**Frozen**

## Product

**Active Development**

---

## 1. Product Vision

Northstar is an AI-powered Financial Intelligence Platform.

Its purpose is to:

- observe markets;
- interpret market conditions;
- recommend what deserves attention;
- explain why a recommendation was made; and
- prove recommendations with verifiable evidence.

Execution remains an optional downstream capability. Northstar's primary product value is useful, transparent intelligence, not order placement.

---

## 2. Completed Work

### Epic 1 - Analyze One Asset

**Status:** Completed

A user can submit one asset symbol and receive a recommendation through the completed Domain, Application, API, and Web flow.

### Epic 2 - Explain Recommendation

**Status:** Completed

Every Story 1 recommendation includes a structured explanation with rationale and supporting signals.

### Epic 3 - Real Market Intelligence

**Status:** Completed

Northstar acquires real market observations through the MVP Yahoo Finance adapter and generates `AssetAnalysis` from factual observations before recommendation.

---

## 3. Active Epic

### Epic 4 - Evidence & Transparency

**Status:** Approved - Not Started

Epic 4 extends recommendation explanation with factual market evidence, source attribution, and observation time so users can independently verify conclusions.

---

## 4. Prioritized Product Backlog

The backlog is ordered by the value required to make intelligence understandable, trustworthy, actionable, and eventually useful at scale.

### Epic 4 - Evidence & Transparency

**Priority:** P0 - Next

**Product Goal:** Make every recommendation independently verifiable by showing its factual evidence, market source, and observation time.

**User Value:** Users can inspect the evidence behind a recommendation instead of relying on an opaque conclusion.

**Estimated Size:** M

**Dependencies:** Epics 1, 2, and 3; existing `Recommendation`, `RecommendationExplanation`, `MarketObservationContext`, and `POST /analyze` flow.

### Epic 5 - Confidence Assessment

**Priority:** P1 - High

**Product Goal:** Communicate how strongly the available evidence supports a recommendation.

**User Value:** Users can distinguish a strongly supported assessment from a tentative or weakly supported one.

**Estimated Size:** L

**Dependencies:** Epics 1 through 4; stable evidence model; approved confidence business meaning and calibration rules.

### Epic 6 - Risk Assessment

**Priority:** P1 - High

**Product Goal:** Present the relevant uncertainty and downside considerations for an analyzed asset.

**User Value:** Users can weigh a recommendation against its risk context before deciding what to do.

**Estimated Size:** L

**Dependencies:** Epics 1 through 5; evidence model; confidence context; approved risk business meaning and risk rules.

### Epic 7 - Watchlist Intelligence

**Priority:** P2 - Medium

**Product Goal:** Let users retain selected assets and return to their current intelligence context.

**User Value:** Users can monitor assets that matter to them without repeatedly entering symbols.

**Estimated Size:** M

**Dependencies:** Epics 1 through 6; stable recommendation, explanation, evidence, confidence, and risk presentation; persistence and user-ownership concepts.

### Epic 8 - Opportunity Ranking

**Priority:** P2 - Medium

**Product Goal:** Rank multiple analyzed assets by current relevance and decision value.

**User Value:** Users can focus attention on the most important opportunities instead of reviewing assets in arbitrary order.

**Estimated Size:** L

**Dependencies:** Epics 1 through 7; approved ranking meaning; confidence and risk context; multiple-asset analysis capability.

### Epic 9 - Today's Opportunities

**Priority:** P2 - Medium

**Product Goal:** Provide a daily, focused view of the highest-priority opportunities without requiring manual asset-by-asset discovery.

**User Value:** Users can begin a daily review with an immediately useful intelligence shortlist.

**Estimated Size:** XL

**Dependencies:** Epics 1 through 8; opportunity ranking; reliable real market intelligence; daily observation and freshness semantics; bounded multi-asset acquisition.

### Epic 10 - Portfolio Intelligence

**Priority:** P3 - Strategic

**Product Goal:** Explain market intelligence in the context of the user's holdings and exposure.

**User Value:** Users can understand how recommendations relate to what they already own and the decisions they have already made.

**Estimated Size:** XL

**Dependencies:** Epics 1 through 9; portfolio and position concepts; user ownership; stable risk assessment; explicit separation between intelligence and portfolio state.

### Epic 11 - Research Workspace

**Priority:** P3 - Strategic

**Product Goal:** Give users a durable workspace for investigating assets, evidence, explanations, and historical intelligence.

**User Value:** Users can move from a daily recommendation to deeper, organized research without losing context.

**Estimated Size:** XL

**Dependencies:** Epics 1 through 10; stable evidence provenance; historical observation and recommendation semantics; research organization concepts.

### Epic 12 - Execution

**Priority:** P4 - Optional Downstream

**Product Goal:** Allow a user to connect an approved intelligence decision to an optional execution workflow.

**User Value:** Users can act on a decision from Northstar when they choose, without changing Northstar's role as an intelligence platform.

**Estimated Size:** XL

**Dependencies:** Epics 1 through 11; explicit user intent; broker and exchange integration; order, trade, portfolio, authentication, risk, authorization, and operational safety capabilities.

Execution remains downstream and optional. It must never replace the user's final decision or allow a recommendation to place an order automatically.

---

## 5. Product Principles

These principles are permanent product constraints:

- Every recommendation must be explainable.
- Every explanation must be supported by evidence.
- Every piece of evidence must identify its source.
- Every piece of evidence must identify when it was observed.
- Northstar owns intelligence and interpretation.
- External providers own market data and provider-specific delivery.
- Recommendations are decision support, not automatic instructions.
- The user always makes the final investment decision.
- Evidence must remain distinguishable from interpretation.
- Confidence and risk must not be implied when they are not explicitly modeled.
- The product must not present invented certainty or unsupported facts.
- Execution is optional and downstream from intelligence.

---

## 6. Engineering Delivery Principles

The established implementation workflow is:

```text
Story
  ↓
Implementation
  ↓
Tests
  ↓
API
  ↓
UI
  ↓
Demo
  ↓
Sprint Review
```

Additional rules:

- Work from the approved story and its acceptance criteria.
- Keep Domain meaning in `northstar-core`.
- Keep workflow coordination in `northstar-application`.
- Keep external communication in `northstar-infrastructure`.
- Keep transport translation in `northstar-api`.
- Keep presentation in `northstar-web`.
- Add focused tests at the boundaries affected by each story.
- Preserve completed product behavior while adding new capability.
- No architecture redesign is permitted unless a genuine new business concept emerges and is explicitly approved.
- Technical debt must be recorded rather than silently expanded.

---

## 7. Release Plan

### Alpha

**Status:** Completed

Alpha includes Epics 1 through 3:

- Analyze One Asset
- Explain Recommendation
- Real Market Intelligence

The Alpha demonstrates a complete one-asset intelligence flow grounded in real observations.

### Beta

**Target:** After Epic 6

Beta includes Evidence & Transparency, Confidence Assessment, and Risk Assessment. The user can understand a recommendation, inspect its evidence, and assess support and uncertainty before deciding.

### Release Candidate

**Target:** After Epic 9

The Release Candidate adds Watchlist Intelligence, Opportunity Ranking, and Today's Opportunities. The product supports a repeatable daily intelligence workflow beyond a single manually requested asset.

### v1.0

**Target:** After Epic 12

The v1.0 roadmap includes Portfolio Intelligence, Research Workspace, and optional downstream Execution. Execution remains subject to explicit safety, authorization, broker, and user-intent requirements.

---

## 8. Success Metrics

Northstar succeeds when:

- users trust recommendations because the reasoning and evidence are visible;
- users understand recommendations without requiring specialist interpretation of raw data;
- users can independently verify the source and observation time of supporting evidence;
- users return daily to review changing intelligence;
- users build and revisit watchlists of relevant assets;
- users focus attention on ranked opportunities rather than scanning without guidance;
- users make informed decisions using Northstar as decision support;
- users execute decisions only when they deliberately choose to do so;
- recommendations remain useful without requiring automatic execution.

Suggested measurement signals include:

- recommendation explanation and evidence inspection rate;
- time from recommendation display to evidence inspection;
- user return frequency;
- repeat analysis frequency;
- watchlist creation and revisit rate after Epic 7;
- opportunity-detail engagement after Epic 9;
- user-reported understanding and trust;
- user-reported ability to identify evidence source and observation time;
- deliberate execution conversion after Epic 12, measured separately from recommendation acceptance.

Metrics should measure understanding and informed use, not encourage blind acceptance or unnecessary trading activity.

---

## 9. Backlog Governance

This document is the single source of truth for future Northstar product planning.

A backlog change must:

- preserve the frozen architecture unless a genuine business concept requires review;
- state its user value and dependencies;
- identify scope and product impact;
- preserve completed story behavior;
- distinguish approved work from proposed work;
- avoid silently introducing confidence, risk, execution, or other future concepts into earlier epics.

---

NORTHSTAR

Product Backlog

v1.0

Status

Official Product Roadmap

Architecture

Frozen

Product

Active Development

---
