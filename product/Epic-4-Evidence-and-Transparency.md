# Epic 4 - Evidence & Transparency

## Status

**Ready for Development**

Epic 4 extends Northstar's intelligence experience from explanation to verification. Users can inspect the factual evidence behind a recommendation, identify where that evidence came from, and see when it was observed.

This epic preserves the completed Analyze One Asset, Explain Recommendation, and Real Market Intelligence capabilities. It does not redesign Northstar Architecture v2.0 or introduce confidence, risk, execution, or a new product workflow.

---

## 1. Product Goal

Northstar should enable users to verify every recommendation.

Recommendations should never require blind trust. Each recommendation should be understandable, grounded in factual evidence, attributable to a market source, and anchored to an observation time.

---

## 2. User Story

> As an investor, I want to see the evidence behind every recommendation, so that I can independently verify Northstar's conclusions.

---

## 3. Objective

Transform `RecommendationExplanation` from:

```text
Why
```

into:

```text
Why + Evidence
```

The existing explanation remains the narrative supporting the recommendation. Epic 4 adds the factual evidence, source attribution, and observation timestamp that allow a user to assess that narrative independently.

---

## 4. Product Principles

Every recommendation must:

- explain itself;
- show factual evidence;
- identify the market source;
- identify when the evidence was observed.

Evidence is presented as information, not as a second recommendation. Northstar does not ask the user to trust an unexplained conclusion or an unattributed data point.

Confidence and risk remain separate future concepts and are not introduced by this epic.

---

## 5. User Experience

The recommendation workspace should naturally answer these questions in order:

1. What should I do?
2. Why?
3. What evidence supports it?
4. Where did the evidence come from?
5. When was it observed?

The existing Analyze One Asset flow remains the primary experience:

```text
Enter Symbol
  ↓
Recommendation
  ↓
Explanation
  ↓
Market Evidence
  ↓
Source Attribution
  ↓
Observation Time
```

The recommendation remains the headline. The explanation remains the narrative. Evidence follows as an inspectable factual layer rather than replacing either existing concept.

---

## 6. MVP Scope

### In Scope

- Factual market evidence supporting the existing recommendation explanation.
- Source attribution for the evidence.
- Observation timestamp for the evidence.
- Structured evidence associated with the relevant explanation reason.
- Rendering of evidence, source, and observation time in the existing recommendation workspace.
- Preservation of the existing Analyze One Asset API boundary and user journey.

### Out of Scope

- Confidence.
- Risk.
- Watchlist.
- Execution.
- Portfolio.
- Market scanner.
- Alerts.
- Streaming or WebSocket delivery.
- New recommendation policy.
- Automated trading decisions.
- A new evidence marketplace, provider registry, or generalized ingestion framework.

---

## 7. Business Meaning

Market evidence is factual observation data that supports a recommendation explanation. It may include values such as observed price, volume, daily high, daily low, or other observations already available from the approved market-observation workflow.

Source attribution identifies the external market-data source that supplied the evidence, such as the MVP Yahoo Finance adapter.

Observation timestamp identifies when the source reported or measured the evidence. It is not the time when the user opened the page and must not be replaced by presentation time.

Evidence supports reasoning. It does not determine the recommendation independently, override Strategy, or become a new decision policy.

---

## 8. Relationship Review

The completed product relationship becomes:

```text
Market Observations
  ↓
AssetAnalysis
  ↓
Strategy
  ↓
Recommendation
  ↓
RecommendationExplanation
  ↓
Evidence + Source + Observation Time
```

- `AssetAnalysis` remains the interpreted understanding used by Strategy.
- `Strategy` remains the sole owner of recommendation action policy.
- `Recommendation` remains the authoritative decision outcome.
- `RecommendationExplanation` remains the structured rationale supporting that outcome.
- Evidence adds verifiability to the explanation without changing any upstream decision.

The evidence model must not duplicate the full `AssetAnalysis` or take ownership of raw market-data acquisition.

---

## 9. Repository Responsibilities

### northstar-core

- Define the Domain meaning of evidence supporting a `RecommendationExplanation`.
- Model evidence structurally rather than as presentation-only text.
- Preserve source attribution and observation time as meaningful business data.
- Keep evidence associated with the explanation and recommendation it supports.
- Validate that evidence is factual, attributable, and time-bound.
- Keep Strategy policy and recommendation action unchanged.
- Add focused Domain tests proving evidence supports explanation without determining Recommendation.

### northstar-application

- Coordinate the existing observation, analysis, recommendation, and explanation flow.
- Carry evidence, source attribution, and observation time through the application result without recalculating or rewriting them.
- Preserve the existing `AnalyzeAssetResult` responsibilities while extending it only as required for the evidence contract.
- Keep Application orchestration separate from Domain interpretation and provider communication.
- Add focused application contract coverage for evidence propagation.

### northstar-api

- Extend the existing `POST /analyze` response only with evidence fields supplied by the Application result.
- Serialize structured evidence, source attribution, and observation time without flattening or inventing values.
- Preserve existing request validation and error semantics.
- Keep the router as a thin transport boundary with no recommendation, explanation, or evidence calculation.
- Add endpoint coverage for successful evidence delivery and existing failure behavior.

### northstar-web

- Reuse the existing Analyze Asset workspace.
- Keep Recommendation as the headline and Explanation as the narrative.
- Add a clearly distinguishable Market Evidence section.
- Display the factual values, source attribution, and observation timestamp returned by the API.
- Make evidence easy to scan without turning it into a charting or market-monitoring surface.
- Do not add confidence, risk, watchlist, execution, alerts, or scanner behavior.

### northstar-infrastructure

- Preserve provider-specific acquisition and parsing responsibilities.
- Supply source attribution and the original observation timestamp with the factual evidence.
- Keep Yahoo Finance isolated behind the existing Application-owned source contract.
- Do not expose provider payloads or transport details directly to API or Web layers.
- Do not introduce streaming, caching, scheduling, or multiple-provider behavior.

---

## 10. Acceptance Criteria

Epic 4 is accepted when users can:

- understand a recommendation;
- inspect its supporting factual evidence;
- identify the market source of that evidence;
- identify when the evidence was collected or observed;
- distinguish the recommendation, explanation, and evidence as separate layers.

Additional acceptance conditions:

- Existing Story 1 recommendation behavior is preserved.
- Existing Story 2 explanation behavior is preserved.
- Existing Epic 3 real-market observation behavior is preserved.
- Evidence is rendered from Application/API data and is not invented by the Web layer.
- Strategy does not consume evidence as a new recommendation-policy input.
- No confidence, risk, execution, watchlist, scanner, or alert behavior is introduced.

---

## 11. Definition of Done

Northstar recommendations are explainable, verifiable, and transparent.

Completion requires:

- A structured Domain representation for supporting evidence, source attribution, and observation time.
- Application propagation of that representation through the existing Analyze Asset workflow.
- API serialization through the existing `POST /analyze` endpoint.
- Web presentation in the existing Analyze One Asset workspace.
- Tests at the relevant Domain, Application, API, and Web boundaries.
- Existing product behavior preserved.
- Relevant repository lint, format, build, and test checks passing.

---

## 12. Self Review

- Product-first: the epic addresses the user need to verify recommendations.
- User trust increased: evidence, source, and observation time reduce blind reliance.
- Architecture unchanged: Domain owns meaning, Application coordinates, Infrastructure acquires, API transports, and Web presents.
- Existing product behavior preserved: recommendation and explanation remain intact.
- Scope controlled: only evidence and transparency are added.

---

EPIC 4

Evidence & Transparency

Status

Ready for Development

Goal

Every recommendation should be understandable and independently verifiable.

---
