# Story 3 - Generate Asset Analysis

## Status

**Ready for Development**

Story 3 replaces the hardcoded MVP `AssetAnalysis` input used by the Analyze One Asset workflow with an `AssetAnalysis` generated from actual market observations for the requested asset.

This story preserves Northstar Architecture v2.0 and the completed Story 1 and Story 2 behavior. It improves the quality of the input to Strategy. It does not change recommendation policy, explanation behavior, or the existing user workflow.

---

## 1. User Story

> As an investor, I want Northstar to derive its analysis from actual market observations, so that recommendations are based on real evidence rather than placeholder signals.

---

## 2. Goal

Replace the MVP placeholder `AssetAnalysis` generation with a real AssetAnalysis generation workflow.

```text
Market observations
  ↓
AssetAnalysis
  ↓
Strategy
  ↓
Recommendation
  ↓
RecommendationExplanation
```

The output of Story 3 remains the existing `AssetAnalysis`. `Recommendation` remains downstream and unchanged. Story 3 improves the evidence and interpreted signals entering the existing decision path.

---

## 3. Business Purpose

Generating an `AssetAnalysis` means converting current, asset-specific market observations into an interpreted understanding of one listed asset at one point in time.

The generated `AssetAnalysis` provides the summarized signals that the existing `Strategy` evaluates. It does not select a recommendation, produce a BUY/HOLD/SELL action, calculate confidence or risk, or create an explanation independently.

`AssetAnalysis` remains the boundary between observed market conditions and Strategy decision policy:

```text
Observed market conditions
  ↓
Interpreted signals
  ↓
AssetAnalysis
```

The resulting `AssetAnalysis` must be traceable to the observations supplied for the requested asset, deterministic for the same input observations, and valid according to the existing Domain model.

---

## 4. Required Market Observations

Story 3 requires only the observations needed to generate the current MVP signal vocabulary for one requested asset. It does not establish a reusable market-data platform or ingestion framework.

The minimum observation set is:

- Listing identity and venue context for the requested symbol.
- Observation timestamp.
- Latest tradable price.
- Recent historical prices sufficient to determine the short-term directional trend.
- Recent volume sufficient to compare current activity with a recent baseline.
- Short and long moving-average values, or the bounded price series needed to derive them.
- A bounded momentum measure, or the price series needed to derive it.

These observations allow Northstar to determine whether the current asset evidence supports the existing Strategy signal vocabulary. The implementation must remain limited to the evidence required by the current Strategy behavior.

---

## 5. Output

The sole Domain output added by Story 3 is the existing `AssetAnalysis` value:

- `listing`
- `point_in_time`
- `summarized_signals`

No new recommendation concept is introduced. Story 3 does not add confidence, risk, ranking, scanning, portfolio context, or execution intent.

The generated signals must remain concise, deterministic, and compatible with the existing `Strategy.evaluate()` behavior. They are interpreted analysis inputs, not raw market-data records.

---

## 6. Responsibility Boundaries

### Market observations

Market observations are factual, time-bound inputs for one requested listing. They describe available price and volume evidence without selecting a recommendation.

### AssetAnalysis

`AssetAnalysis` owns the interpreted summary of the relevant observed conditions. It does not own observation retrieval, market-data persistence, or a recommendation action.

### Strategy

`Strategy` continues to own recommendation policy. It consumes the generated `AssetAnalysis` and remains solely responsible for BUY/HOLD/SELL selection.

### Recommendation and RecommendationExplanation

The completed Story 1 Recommendation and Story 2 RecommendationExplanation remain downstream outputs. Their contracts and responsibilities do not change in Story 3.

---

## 7. Repository Responsibilities

### northstar-core

- Preserve the existing `AssetAnalysis`, `Strategy`, `Recommendation`, and `RecommendationExplanation` responsibilities.
- Define only the bounded Domain behavior required to interpret the approved market observations into the existing summarized-signal vocabulary.
- Keep signal interpretation separate from recommendation policy.
- Add focused tests for deterministic analysis generation from fixed market observations.
- Do not introduce confidence, risk, market-data persistence, scanning, or recommendation changes.

### northstar-application

- Move the market-observation access boundary out of `northstar-api` in accordance with TD-001.
- Define or place the narrow Application-owned analysis source contract required to request observations for one symbol.
- Update the existing `AnalyzeAssetUseCase` to obtain observations through that contract and construct the existing `AssetAnalysis` using the approved Domain behavior.
- Preserve the existing `AnalyzeAssetResult`, Recommendation, and RecommendationExplanation outputs.
- Add focused application contract tests using deterministic observation-source doubles.

### northstar-api

- Remove API-owned construction of Listings, Instruments, Exchanges, and placeholder signals.
- Continue to validate the request, invoke the existing `AnalyzeAssetUseCase`, and serialize the existing application result.
- Preserve the existing `POST /analyze` path, response shape, and error semantics except where real unavailable observation context requires the already-approved application failure translation.
- Add focused endpoint coverage for a real analysis result and unavailable observation context.

### northstar-web

- Reuse the existing Analyze Asset workspace and `POST /analyze` integration.
- Continue to display only the recommendation and structured explanation returned by the API.
- Replace no UI behavior unless a visible unavailable-analysis state is needed for the existing endpoint response.
- Do not add charts, scanner views, watchlists, or market-data presentation surfaces.

---

## 8. Technical Debt Resolution

Story 3 resolves the delivery concern recorded in TD-001 by replacing `InMemoryMarketObservationProvider` in `northstar-api` with a proper Application-owned analysis source and its minimum concrete implementation.

This is not a full market-data ingestion initiative. The implementation must provide only what the Analyze One Asset workflow needs:

```text
northstar-api
  ↓
AnalyzeAssetUseCase
  ↓
Application-owned analysis source contract
  ↓
Minimum market-observation implementation
  ↓
AssetAnalysis generation
```

The API must not construct Domain objects or hardcode `"strong bullish"` signals after Story 3 is complete.

---

## 9. Scope

### In Scope

- Real market observations for one requested, supported asset.
- Generation of the existing `AssetAnalysis` from those observations.
- Minimum Application-owned source boundary and concrete support needed by Analyze One Asset.
- Preservation of the existing recommendation and explanation flow.
- Focused Domain, Application, API, and Web coverage required for the one-asset journey.

### Out of Scope

- Market scanner.
- Watchlist.
- Alerts.
- Confidence.
- Risk.
- Execution.
- Portfolio.
- Broker integration.
- Opportunity ranking.
- Full market-data ingestion framework.
- New endpoints or screens.

---

## 10. Acceptance Criteria

Story 3 is accepted when:

- `AssetAnalysis` is generated from actual market observations for one requested asset.
- Generated summarized signals are deterministic for the same market observations.
- `Strategy.evaluate()` continues to work unchanged with the generated `AssetAnalysis`.
- `Recommendation` behavior remains unchanged.
- `RecommendationExplanation` behavior remains unchanged.
- The existing `POST /analyze` endpoint and Analyze Asset workspace continue to support the completed Story 1 and Story 2 journey.
- `northstar-api` no longer constructs Domain market context or hardcodes analysis signals.
- No scanner, watchlist, alert, confidence, risk, execution, portfolio, or broker behavior is introduced.

---

## 11. Definition of Done

Story 3 is complete when Northstar no longer depends on hardcoded `"strong bullish"` signals to produce recommendations.

Completion requires:

- A real observation-to-`AssetAnalysis` generation path for one requested asset.
- An Application-owned source boundary replacing the API-owned in-memory provider.
- Existing Recommendation and RecommendationExplanation behavior preserved.
- Automated coverage demonstrating deterministic analysis generation and the complete existing one-asset flow.
- Relevant repository lint, format, build, and test checks passing.

---

## 12. Self Review

- Product-first: recommendations become grounded in current evidence for the requested asset.
- Story 1 unchanged: Analyze One Asset remains one request, one analysis, and one recommendation.
- Story 2 unchanged: explanation continues to support the resulting recommendation.
- Architecture unchanged: Domain interprets, Application coordinates, API delivers, and Web presents.
- AssetAnalysis becomes real: hardcoded placeholder signals are replaced by generated interpretation from observations.
- Recommendation remains downstream: no recommendation policy moves into analysis generation.

---

STORY 3

Generate Asset Analysis

Status

Ready for Development

Goal

Produce real AssetAnalysis from market observations.

---
