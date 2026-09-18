# Story 3.1 - Acquire Market Observations

## Status

**Ready for Development**

Story 3.1 is the first delivery step of the Real Market Intelligence epic. It replaces the API-owned in-memory observation source with the smallest Application-owned workflow capable of acquiring factual market observations for one requested asset.

This story preserves Northstar Architecture v2.0 and all completed Story 1 and Story 2 behavior. It provides observation context only. It does not generate `AssetAnalysis`, select a recommendation, or change explanation behavior.

---

## 1. User Story

> As an investor, I want Northstar to analyze real market observations, so that recommendations are derived from actual market conditions rather than placeholder signals.

---

## 2. Product Goal

Given one symbol, Northstar obtains the factual observation context required by Story 3 to generate an `AssetAnalysis`.

```text
Requested symbol
  ↓
Acquire Market Observations
  ↓
Market Observation Context
  ↓
Story 3: Generate AssetAnalysis
```

The output of Story 3.1 is **Market Observation Context**. It is not `AssetAnalysis` and it does not contain recommendation signals such as `"strong bullish"` or `"strong bearish"`.

---

## 3. Business Purpose

Market Observation Context is the bounded factual evidence available for one requested listing at one point in time. It gives the later Domain analysis step the observed inputs needed to create an interpreted `AssetAnalysis`.

Story 3.1 separates two responsibilities:

- Acquiring factual market observations.
- Interpreting those observations into `AssetAnalysis` signals.

The Application layer coordinates the first responsibility. The Domain owns the second responsibility in Story 3.

---

## 4. Minimum Observation Context

The context must be intentionally small and sufficient only for Story 3's one-asset analysis. It contains:

- The resolved `Listing` for the requested symbol.
- The observation point in time.
- Latest price.
- Latest volume.
- A bounded recent price history sufficient for short-term trend, moving-average, and momentum interpretation.
- A bounded recent volume history sufficient to compare current volume with a recent baseline.

The bounded histories are factual inputs. They must preserve ordering and timestamp association required for later interpretation.

Story 3.1 does not define indicators, moving-average thresholds, trend labels, momentum labels, recommendation signals, or a generic market-data model. Those are interpretation concerns for Story 3 Domain behavior.

---

## 5. Application Responsibility

`northstar-application` owns the workflow boundary for acquiring observation context for one symbol.

It must:

- Define the application-level `MarketObservationContext` DTO required by Story 3.
- Define one narrow source contract, such as `MarketObservationSource`, that accepts one `Symbol` and returns one context.
- Validate only application-boundary concerns needed to coordinate the request and result.
- Make the context available to the existing Analyze One Asset flow without interpreting observations.
- Preserve the existing `AnalyzeAssetUseCase` workflow and output contracts until Story 3 performs the Domain interpretation step.

It must not:

- Compute signals.
- Produce `AssetAnalysis` from observations in this story.
- Determine BUY/HOLD/SELL.
- Construct or own Strategy policy.
- Add caching, scheduling, scanning, or multi-provider selection.

---

## 6. Domain Responsibility

The Domain continues to own the meaning of `AssetAnalysis` and all recommendation behavior.

Story 3.1 does not change:

- `AssetAnalysis`
- `Strategy.evaluate()`
- `Recommendation`
- `RecommendationExplanation`
- `ExplanationReason`

The Domain does not retrieve observations. In Story 3, it will interpret the context acquired by this story into the existing `AssetAnalysis` output.

---

## 7. Infrastructure Responsibility

Infrastructure will later implement the single Application source contract using a real market-data mechanism.

For Story 3.1, only the contract and the minimum concrete source required for one requested asset are in scope. The implementation must:

- Supply actual observed prices and volumes for one supported symbol.
- Resolve the listing needed by the returned context.
- Preserve the source observation time and bounded history ordering.
- Translate technical provider data into the Application-owned observation context.

It must not create a market-data framework, provider registry, caching layer, scheduler, scanner, or multi-provider abstraction.

---

## 8. Technical Debt Resolution

Story 3.1 removes TD-001 by replacing `InMemoryMarketObservationProvider` in `northstar-api` with the new Application-owned observation source contract and its required concrete implementation.

The dependency flow after completion is:

```text
northstar-api
  ↓
AnalyzeAssetUseCase
  ↓
Application-owned MarketObservationSource
  ↓
Minimum concrete observation source
  ↓
MarketObservationContext
```

The API must no longer construct `Listing`, `Instrument`, `Exchange`, or hardcoded market observations. It remains a thin HTTP transport layer.

---

## 9. Repository Responsibilities

### northstar-core

- No change to recommendation or explanation behavior.
- No observation-retrieval responsibility.
- No signal interpretation is implemented in Story 3.1.

### northstar-application

- Define `MarketObservationContext` as an Application DTO.
- Define the narrow `MarketObservationSource` dependency required for one symbol.
- Rework the temporary application input boundary so it can receive real observation context rather than API-supplied hardcoded signals.
- Preserve existing Analyze Asset public behavior until Story 3 introduces Domain interpretation.
- Add focused contract tests for source coordination and context propagation.

### northstar-api

- Remove the API-owned in-memory observation provider.
- Keep the existing `POST /analyze` endpoint, request validation, response shape, and error semantics.
- Delegate entirely to the configured Application workflow.
- Add focused endpoint coverage ensuring the API does not own observation construction.

### northstar-web

- No new screen, route, or workflow.
- Reuse the existing Analyze Asset workspace and `POST /analyze` integration.
- Preserve Story 1 and Story 2 behavior.

---

## 10. Scope

### In Scope

- Acquisition of real observations for one requested asset.
- An Application-owned observation context DTO and source contract.
- A minimum concrete source implementation required to replace the API in-memory provider.
- Removal of TD-001's API-owned Domain construction.
- Focused tests for the one-asset observation path.

### Out of Scope

- Market Scanner.
- Watchlist.
- Alerts.
- Confidence.
- Risk.
- Execution.
- Portfolio.
- Caching.
- Scheduling.
- Multi-provider support.
- Market-data ingestion framework.
- `AssetAnalysis` interpretation from observations.
- New API endpoints or Web routes.

---

## 11. Acceptance Criteria

Story 3.1 succeeds when:

- The Application acquires real market observations for one requested asset through an Application-owned source contract.
- The resulting Market Observation Context contains the approved minimum factual observations.
- The Application does not interpret observations or generate recommendation signals.
- Story 1 analysis, recommendation, and response behavior remain unchanged.
- Story 2 explanation behavior remains unchanged.
- `northstar-api` no longer contains `InMemoryMarketObservationProvider` or constructs hardcoded market observation context.
- TD-001 is removed.
- No scanner, watchlist, alert, confidence, risk, execution, portfolio, caching, scheduling, or multi-provider behavior is introduced.

---

## 12. Definition of Done

Story 3.1 is complete when Northstar no longer depends on hardcoded market observations for Story 1.

Completion requires:

- An Application-owned `MarketObservationContext` and source contract for one asset.
- A minimum concrete source capable of returning real observations for one supported symbol.
- API removal of the in-memory observation source and Domain object construction.
- Existing Story 1 and Story 2 behavior preserved.
- Focused automated coverage for the new observation acquisition boundary.
- Relevant repository lint, format, build, and test checks passing.

---

## 13. Self Review

- Product-first: the next feature delivers evidence that makes existing recommendations meaningful.
- Story 1 preserved: Analyze One Asset remains a one-symbol workflow.
- Story 2 preserved: Recommendation Explanation remains a downstream supporting output.
- Architecture unchanged: Application coordinates acquisition; Domain interprets; API transports; Web presents.
- No premature framework: exactly one source contract supports exactly one requested asset.

---

STORY 3.1

Acquire Market Observations

Status

Ready for Development

Goal

Replace placeholder observations with real market observations.

---
