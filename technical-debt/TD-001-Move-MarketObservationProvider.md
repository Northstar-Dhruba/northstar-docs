# TD-001: Move InMemoryMarketObservationProvider from northstar-api to northstar-application

## Status

Open

## Priority

Medium

## Target Sprint

Sprint 2

## Title

Move InMemoryMarketObservationProvider from northstar-api to northstar-application

## Background

During the first vertical slice for Story 1, the project required a minimal end-to-end implementation to validate the product experience quickly. The API layer was needed to expose an initial HTTP endpoint for analyzing one asset, while the existing application workflow and domain model were already implemented and validated.

To enable this first working slice without delaying delivery, the `InMemoryMarketObservationProvider` was introduced directly in `northstar-api`. It was intentionally created as an in-process adapter that could produce the required `AssetAnalysisInput` values for a known symbol and delegate to the existing `AnalyzeAssetUseCase`.

This compromise allowed the team to complete the Story 1 user journey end to end without introducing additional architectural scaffolding that was not yet required for the sprint goal.

## Problem

The current placement is not ideal from an architectural perspective.

The API layer is now responsible for Domain concepts that should remain outside the transport boundary. In its current form, the provider is constructing application-facing Domain objects such as:

- `Listing`
- `Instrument`
- `Exchange`
- `Currency`
- `PointInTime`
- `AssetAnalysisInput`

This type of logic belongs behind the Application boundary. By placing the provider in the API layer, the API becomes aware of Domain composition and object construction semantics. That increases coupling, leaks Domain concerns into transport code, and reduces the clarity of the boundary between HTTP delivery and business orchestration.

The current implementation also makes the API look more like an application adapter than a thin transport layer. The API should accept inbound request data, delegate to the application layer, and translate the result for HTTP consumers without constructing or owning Domain market context.

## Desired State

The architecture should follow the following boundary:

`northstar-api`

↓

`AnalyzeAssetUseCase`

↓

`MarketObservationProvider` (Application contract)

↓

Infrastructure implementation

In the desired state:

- `northstar-api` remains a thin transport boundary.
- `northstar-application` owns the `MarketObservationProvider` contract and orchestration boundary.
- Infrastructure is responsible for concrete market observation retrieval or adaptation.
- The API does not construct Domain objects or directly satisfy application dependencies.
- Existing Story 1 behavior remains unchanged while the architectural separation is restored.

## Rationale

This debt was intentionally accepted to maximize Sprint 1 delivery and to avoid premature abstraction while the Story 1 vertical slice was still being validated.

The team prioritized:

- shipping the first user-facing experience quickly,
- validating the end-to-end flow from UI to API to application to domain,
- preserving the frozen architecture while keeping the implementation lean,
- avoiding broad refactoring before the first business value was demonstrated.

This was a deliberate tradeoff: the architecture is not ideal in the short term, but it enabled the MVP to reach the approved Story 1 outcome without blocking user value delivery.

## Impact

### Current impact

The current impact is moderate and contained to the Story 1 slice.

- The API is slightly more coupled than desired.
- Domain construction is happening in a transport layer.
- The application boundary is less clean than the approved architecture suggests.
- The project still functions end to end for the shipped feature.

### Future impact

The debt will become more relevant as the platform grows beyond the MVP.

If left unresolved, the following risks increase:

- additional API code will continue to own Domain assembly logic,
- infrastructure and application responsibilities become harder to separate,
- Story 2 and later work will need to untangle the current dependency direction,
- developer onboarding will be slower because the architectural boundary is less clear.

### Risk

The risk is limited in the current story because the implementation is narrow and validated. However, the risk increases when the platform introduces real infrastructure adapters, additional market data sources, or more complex application workflows.

### Maintainability

The current implementation is maintainable for the Story 1 scope, but it weakens the maintainability of the platform as a system because the API layer is doing work that belongs to application/infrastructure boundaries.

### Developer experience

Developer experience is currently acceptable for the MVP, but the debt introduces friction:

- engineers must understand that API code contains Domain object construction,
- boundary expectations are less obvious,
- future refactors will require a more careful move of responsibilities.

## Priority

Medium

Reason:

- This debt does not block Story 1.
- It does not prevent the first user journey from working.
- It should be addressed before Story 2 or before major infrastructure integration begins to prevent boundary drift and unnecessary coupling.

## Acceptance Criteria

This technical debt is resolved when all of the following are true:

- The API no longer constructs Domain objects directly.
- `MarketObservationProvider` resides in `northstar-application`.
- The API delegates entirely to the Application layer.
- Existing Story 1 behavior remains unchanged.
- No new API behavior is introduced as part of the refactor.

## Notes

This technical debt is an intentional engineering compromise accepted to support early user value and vertical-slice validation. It is explicit, bounded, and should be closed before the project moves into the next stage of platform maturity.
