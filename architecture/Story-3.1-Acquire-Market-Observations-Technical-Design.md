# Story 3.1 - Acquire Market Observations: Technical Design

## Status

**Approved - Ready for Implementation**

## 1. Purpose

Story 3.1 replaces the temporary `InMemoryMarketObservationProvider` with the smallest production-oriented path that acquires factual market observations for one requested asset.

This design supports Story 3, which will interpret those observations into the existing `AssetAnalysis`. It does not change the completed recommendation or explanation contracts.

```text
POST /analyze
  ↓
AnalyzeAssetUseCase
  ↓
Application-owned MarketObservationSource
  ↓
Yahoo Finance adapter
  ↓
MarketObservationContext
  ↓
Story 3 Domain interpretation
```

## 2. MVP Provider: Yahoo Finance

Yahoo Finance is the MVP provider because it offers broad symbol coverage and daily market data suitable for a single-asset intelligence workflow, with a low integration cost for early product validation.

This is an implementation decision for Story 3.1, not a permanent architectural dependency. The Application depends on its own narrow observation-source contract, not on Yahoo Finance APIs, response structures, or client libraries. A later provider can replace the adapter without changing the Application workflow, Domain behavior, API endpoint, or Web workspace.

The first implementation uses one bounded request per requested symbol. It does not add a provider registry, fallback strategy, multi-provider support, caching, streaming, or scheduling.

## 3. Application Contract

### MarketObservationContext

`MarketObservationContext` is an Application DTO. It carries factual, time-bound observations for one resolved listing. It does not contain interpreted signals, recommendation actions, explanation text, confidence, or risk.

The minimum fields are:

- `listing`: the resolved `Listing` for the requested symbol.
- `observed_at`: the timestamp of the latest observation.
- `latest_price`: the latest available close or regular-market price.
- `previous_close`: the immediately previous daily close.
- `latest_volume`: the latest available daily volume.
- `daily_high`: the latest available daily high.
- `daily_low`: the latest available daily low.
- `recent_closes`: an ordered, bounded sequence of recent daily closes.
- `recent_volumes`: an ordered, bounded sequence of corresponding recent daily volumes.

The initial lookback must be fixed and documented in code. A 30-session daily window is sufficient for the first implementation because it gives Story 3 enough inputs to compute a short and long moving average, basic trend, and bounded momentum without acquiring unnecessary history.

All price and volume values are raw facts. The contract must preserve chronological ordering and exact numeric values supplied by the adapter. The Application does not convert them into signal labels.

### MarketObservationSource

The Application owns one narrow dependency contract:

```python
class MarketObservationSource(Protocol):
    def get_observation_context(self, symbol: Symbol) -> MarketObservationContext:
        ...
```

The contract accepts one `Symbol` and returns one complete context, or raises one of the defined acquisition failures. It does not expose batch queries, scans, subscriptions, retries, provider selection, or persistence operations.

### Application Workflow

`AnalyzeAssetUseCase` receives the source through constructor injection. It requests a `MarketObservationContext` for the validated symbol and passes the factual context to the Story 3 Domain analysis-generation behavior.

Story 3.1 does not place interpretation in the use case. The use case must not:

- calculate moving averages;
- label trend or momentum;
- choose signals;
- select BUY, HOLD, or SELL;
- alter a Recommendation or RecommendationExplanation.

Until Story 3's Domain interpretation is implemented, this contract can be introduced behind the use case without modifying recommendation policy.

## 4. Required Yahoo Finance Data

The Yahoo Finance adapter obtains the minimum daily observations for one symbol:

- latest close or regular-market price;
- previous close;
- latest daily volume;
- latest daily high;
- latest daily low;
- 30 daily close values;
- 30 corresponding daily volume values;
- timestamps associated with the returned daily observations.

The adapter should request a single daily historical series with a bounded range sufficient to return the required 30 completed daily observations. Where available in the same provider response, regular-market quote fields may be used for current values. If a current quote is unavailable, the latest completed daily bar may supply the latest price, volume, high, low, and timestamp.

The adapter resolves a `Listing` from the supported symbol and provider instrument metadata only as far as required to populate `MarketObservationContext`. It must not expose provider payloads beyond Infrastructure or hardcode listing construction in the API.

## 5. Domain Relationship

Story 3.1 ends with factual `MarketObservationContext`.

Story 3 begins with that context and creates the existing `AssetAnalysis`:

```text
MarketObservationContext
  ↓
Domain analysis-generation behavior
  ↓
AssetAnalysis
  ↓
Strategy.evaluate()
  ↓
Recommendation
  ↓
RecommendationExplanation
```

`AssetAnalysis` remains the interpreter of market evidence. `Strategy` remains the sole owner of recommendation policy. `RecommendationExplanation` remains a downstream explanation of the resulting recommendation.

No component in the API, Application acquisition path, or Yahoo Finance adapter may produce a `"strong bullish"`, `"strong bearish"`, BUY, HOLD, or SELL value.

## 6. Infrastructure Design

The concrete Yahoo Finance adapter belongs in `northstar-infrastructure`. It implements `MarketObservationSource` and owns:

- Yahoo Finance HTTP communication;
- URL and query construction;
- request timeout configuration;
- bounded retry behavior for transient transport failures;
- provider response parsing;
- conversion from provider payloads to Application DTO values;
- provider-specific transport and parsing errors.

The adapter must use a finite timeout and at most one retry for transient connection or server failures. It must not retry invalid symbols, malformed responses, or incomplete market data. No generalized retry framework is introduced.

The adapter should use a small standard HTTP client dependency already approved for the infrastructure repository, or the Python standard library if no dependency is present. It must not add a market-data SDK, provider abstraction framework, or provider-specific types to `northstar-application`.

## 7. Failure Behavior

Failures remain explicit and bounded. The Application contract should express the following acquisition outcomes through application-level exceptions or failure values; the implementation choice must stay consistent with existing Application conventions.

### Unknown symbol

When Yahoo Finance has no usable instrument/history for a requested symbol, the adapter reports an unknown-symbol acquisition failure.

The Application propagates this without reinterpretation. The API preserves the existing `404 Unknown symbol.` behavior.

### Unavailable data

When a known symbol does not have current or historical daily data required to create a complete `MarketObservationContext`, the adapter reports unavailable-observation-context.

The Application propagates the failure. The API uses its existing application-failure behavior without exposing provider details.

### Provider unavailable

When Yahoo Finance cannot be reached, returns a transient server failure after the bounded retry, or times out, the adapter reports provider-unavailable.

The Application does not retry or select another provider. The API preserves its existing application-failure behavior and does not expose transport diagnostics to clients.

### Partial observations

A context is usable only when all required fields are present and the bounded daily series has aligned close, volume, and timestamp observations. The adapter rejects partial observations rather than inventing defaults, padding history, or deriving absent values.

Partial data is reported as unavailable-observation-context. The API does not expose raw provider payloads or partial values.

## 8. TD-001 Resolution

Story 3.1 resolves TD-001 by removing `InMemoryMarketObservationProvider` from `northstar-api`.

After implementation:

- `northstar-api` validates HTTP input, invokes the configured Application use case, and serializes its result.
- `northstar-application` owns `MarketObservationContext` and `MarketObservationSource`.
- `northstar-infrastructure` implements the Yahoo Finance adapter.
- No API module constructs `Listing`, `Instrument`, `Exchange`, or hardcoded observation values.

The public `POST /analyze` endpoint, its request shape, its successful response shape, and Story 1/2 behavior remain unchanged.

## 9. Implementation Sequence

1. Add the Application DTO and single-symbol source contract in `northstar-application`.
2. Define the minimal Domain-facing input boundary required by Story 3 without adding interpretation to the Application layer.
3. Implement the Yahoo Finance adapter in `northstar-infrastructure` with a bounded daily-history request and failure translation.
4. Wire the adapter into the Analyze Asset composition root; remove the API-owned in-memory provider.
5. Keep `POST /analyze` as the existing transport boundary.
6. Add focused adapter, Application contract, and endpoint integration tests using deterministic provider response fixtures.

## 10. Acceptance Criteria

Story 3.1 is accepted when:

- A real Yahoo Finance request acquires the required observation context for one supported symbol.
- The context includes complete, ordered daily close and volume observations plus latest price, previous close, high, and low.
- The Application acquires and propagates factual context without interpreting it.
- Unknown symbols map to the existing API `404` behavior.
- Provider-unavailable and incomplete-data failures do not expose provider internals through the API.
- `InMemoryMarketObservationProvider` is removed from `northstar-api`.
- Story 1 recommendation behavior and Story 2 explanation behavior remain unchanged.
- No scanner, cache, scheduler, streaming endpoint, WebSocket, authentication, or multi-provider behavior is introduced.

## 11. Definition of Done

Northstar acquires real observations for one asset using the MVP provider.

Completion requires:

- Application-owned `MarketObservationContext` and `MarketObservationSource` contracts.
- A Yahoo Finance adapter in Infrastructure that returns complete context for one symbol.
- API removal of hardcoded observation and Domain-object construction.
- Focused automated coverage for successful context acquisition, unknown symbol, provider unavailability, and partial observations.
- Existing Analyze Asset recommendation and explanation behavior preserved.
- Relevant repository lint, format, build, and test checks passing.

## 12. Self Review

- Product-first: the implementation grounds intelligence in actual observed conditions.
- Architecture unchanged: Application coordinates, Domain interprets, Infrastructure communicates externally, API transports, and Web presents.
- Application remains coordinator: no indicator or recommendation logic is introduced there.
- Domain remains interpreter: `AssetAnalysis` remains the outcome of Story 3 interpretation.
- Infrastructure owns external communication: Yahoo Finance specifics do not leak into Application or Domain code.
- MVP remains small: one provider, one symbol, one bounded daily data request, and no framework work.

---

STORY 3.1

Acquire Market Observations

Technical Design

Approved

Ready for Implementation

---
