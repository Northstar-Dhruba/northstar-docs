# ADR-017: Repository Layering and Futures Recommendation Domain Boundary

## Status

Proposed

The product decisions recorded here (package name, model name, LONG and SHORT direction, and the daily-candle ambiguity policy) are approved. Formal architectural acceptance of this ADR is pending.

## Date

2026-10-12

## Decision Makers

Northstar Architecture Team

## Context

ADR-001 and ADR-002 (2026-08-09) organized Northstar into three repositories, `northstar-core`, `northstar-api` and `northstar-web`, with `northstar-api` described as the orchestration layer. The platform has since been built as five runtime repositories and one documentation repository. No ADR records that structure. It exists today only in each repository's `pyproject.toml`, `CONTRIBUTING.md` and import graph:

| Repository | Declared Northstar dependencies |
| --- | --- |
| `northstar-core` | none |
| `northstar-application` | `northstar-core` |
| `northstar-infrastructure` | `northstar-application`, `northstar-core` |
| `northstar-api` | `northstar-application`, `northstar-infrastructure`, `northstar-core` |
| `northstar-web` | none (TypeScript; calls `northstar-api` over HTTP) |

The Python repositories reference each other as editable workspace path sources. No source module imports outward: Core imports no Application, Infrastructure or API module; Application imports no Infrastructure or API module; Infrastructure imports no API module.

The next product capability is an actionable Futures trade recommendation: an entry, stop, target and validity for one contract, tracked through a lifecycle. Three existing concepts already use overlapping words:

- `northstar_core.strategy.FuturesRecommendation` is a directional research assessment of one contract at one decision instant. Its action is a `RecommendationAction`: BUY, SELL or HOLD.
- The Application target-position policy (`CreateFuturesExecutionIntentUseCase`) maps a frozen research BUY to a target of +N contracts and SELL to -N contracts. The paper-trading position `FuturesPosition.net_contracts` is signed: positive is long, negative is short.
- `northstar_core.paper_trading.OrderSide` (BUY, SELL) names the side of a simulated order. ADR-008 reuses it for broker demo execution.

Without a recorded boundary, a new recommendation model would either reuse one of these vocabularies with a different meaning or depend on the Paper Trading context to obtain a direction type.

## Decision

### 1. Repository layering

Northstar consists of these repositories:

| Repository | Responsibility |
| --- | --- |
| `northstar-core` | Pure domain models, value objects and domain policies. No I/O, clock, identifier generation, environment, persistence or transport. |
| `northstar-application` | Application services (use cases) and the ports they depend on. Orchestrates Core; holds no infrastructure code. |
| `northstar-infrastructure` | Adapters: persistence, market-data providers, exchange calendars and other external integrations. Implements Application ports. |
| `northstar-api` | Composition root, CLI, scheduled operations and the HTTP interface. Wires Infrastructure adapters into Application services. |
| `northstar-web` | Frontend. Consumes `northstar-api` through its supported HTTP interface. |
| `northstar-docs` | Architecture, domain, engineering-standard and operational documentation. Not a runtime dependency of any repository. |

Allowed dependency directions:

- Core must not depend on Application, Infrastructure, API or Web.
- Application may depend on Core.
- Infrastructure may depend on Application and Core.
- API composes Core, Application and Infrastructure.
- Web communicates with the platform only through supported API interfaces. It does not import any Python package.

### 2. Futures recommendation package

A new Core package, `northstar_core.recommendations.futures`, will hold the Futures trade recommendation domain. It is a sibling of the existing Strategy (`northstar_core.strategy`) and Paper Trading (`northstar_core.paper_trading`) contexts. It is not nested in either.

The approved domain model name is `FuturesTradeRecommendation`.

The approved trade-direction semantics are `LONG` and `SHORT`, represented by a `TradeDirection` defined within the recommendations package.

### 3. Core import direction

- Recommendations may consume the approved pure types of Foundation, Derivatives, Futures and Strategy.
- Strategy must not import Recommendations.
- Recommendations must not depend on Paper Trading. They do not reuse `OrderSide`, `FuturesContractCount`, `FuturesExecutionIntent` or any paper order, fill, position or portfolio type. Nor do they depend on Broker Execution, which itself depends on Paper Trading.
- No new circular dependency may be introduced. Because importing any Strategy module initializes the `northstar_core.strategy` package, a single import of Recommendations from Strategy would create a cycle.

### 4. Semantic distinctions

| Concept | Meaning | Where it exists |
| --- | --- | --- |
| `FuturesRecommendation` | The existing research assessment: a strategy's directional view of one Futures contract at one decision instant. It is not a trade setup and not an order. | `northstar_core.strategy` |
| `RecommendationAction` | The research action of an assessment: BUY, SELL or HOLD. | `northstar_core.strategy` |
| `FuturesTradeRecommendation` | The planned actionable trade setup for one Futures contract (planned contents: direction, entry, stop, target and validity), tracked through a recommendation lifecycle. | `northstar_core.recommendations.futures` (planned) |
| `TradeDirection` | Whether a trade recommendation is LONG or SHORT. | `northstar_core.recommendations.futures` (planned) |
| Target exposure | The signed position a policy intends to hold in a contract. | The Application target-position policy and the signed `FuturesPosition.net_contracts` |
| `OrderSide` | BUY or SELL: the side of an execution instruction. | `northstar_core.paper_trading`, reused by `northstar_core.broker_execution` |

A research SELL action is not inherently a SHORT recommendation. The existing paper-trading policy independently maps SELL to a negative target exposure. That mapping belongs to paper trading and is unchanged by this ADR. A future recommendation-generation policy may derive SHORT from a research SELL, but this ADR does not implement or approve such a mapping.

### 5. Daily-candle semantics

With daily candles, the order of prices within a session is unknown. The approved L-3 policy is:

- When the stop and the target of an ACTIVE recommendation are both touched within the same daily candle, the stop is applied first (conservative handling). The resulting event must report that the intraday ordering was ambiguous.
- The candle in which entry occurs cannot establish an order between entry, target and stop. No sequence may be invented for it.
- An ambiguous outcome on the entry candle must be represented explicitly, never resolved silently.
- Recommendation events are not broker fills and not paper-trading executions.

Detailed lifecycle transition tables, temporal validation rules and Core function contracts are left to the F-2 engineering specification.

## Architecture Invariants

1. **Inward dependencies only.** No repository depends on a repository outside its allowed set in Decision 1.
2. **Core stays pure.** Core reads no clock, generates no identifier, and performs no I/O.
3. **One-way Core edge.** Recommendations may import Strategy; Strategy never imports Recommendations.
4. **No execution coupling.** Recommendations never import Paper Trading or Broker Execution.
5. **Vocabularies stay separate.** Research action, trade direction, target exposure and order side are four distinct concepts; none is an alias of another.
6. **Existing semantics are unchanged.** `FuturesRecommendation`, `RecommendationAction`, the paper target-position policy and `OrderSide` keep their current meaning and behavior.
7. **Honest daily ambiguity.** A daily candle never yields an invented intraday order.

## Rationale

Recording the five-repository layering makes the current architecture reviewable against a decision rather than against conventions scattered across repositories. The dependency directions are those already in force in `pyproject.toml` and in the import graph, so the decision describes the system rather than changing it.

A separate `recommendations` package keeps the actionable trade setup apart from the research assessment it may be derived from and from the simulated execution that may later act on it. Defining `TradeDirection` inside the package lets the recommendation domain be added without moving `OrderSide` (whose relocation ADR-008 defers) and without any dependency on Paper Trading.

Keeping research SELL distinct from SHORT preserves the existing research and paper-trading semantics, and leaves the derivation of a trade direction to an explicit, separately reviewed recommendation-generation policy.

## Alternatives Considered

### Extend `FuturesRecommendation` with entry, stop and target

Rejected. It would change the meaning of an existing, persisted research type and of the frozen records that already contain it.

### Reuse `RecommendationAction` or `OrderSide` as the trade direction

Rejected. BUY, SELL and HOLD describe a research view. BUY and SELL in `OrderSide` describe an execution instruction. Neither expresses LONG or SHORT, and reusing `OrderSide` would make Recommendations depend on Paper Trading.

### Place the recommendation model in Strategy or Paper Trading

Rejected. In Strategy it would merge research assessment with trade setup. In Paper Trading it would merge the recommendation with simulated execution and couple the two lifecycles.

### Edit ADR-001 and ADR-002 in place

Rejected. They record the decision made on 2026-08-09; rewriting them would misstate the history of the architecture.

## Relationship to Earlier Documentation

ADR-001 (Repository Structure) and ADR-002 (Architecture Constitution) are historical records of the three-repository decision and are not rewritten. When this ADR is accepted:

- it supersedes ADR-001;
- it supersedes the repository-role statements of ADR-002. The architectural principles of ADR-002 (Clean Architecture, Domain Driven Design, SOLID, modular design, separation of concerns, event-oriented thinking) remain in force.

Each of those ADRs carries a reference to this ADR beneath its status. The living documents `domain/01-platform-vision.md` (section 8) and `domain/05-dependency-rules.md` are aligned with this ADR. The vocabulary of Decision 4 is defined in `architecture/Ubiquitous-Language.md`.

## Consequences

### Positive Consequences

- The repository layering in force today is recorded and reviewable.
- The recommendation domain can be added without modifying existing Core, Application, Infrastructure, API or Web code.
- Research, recommendation and simulated execution keep separate vocabularies and lifecycles.

### Negative Consequences

- Two direction vocabularies coexist in Core (`OrderSide` and `TradeDirection`); readers must use the Ubiquitous Language to distinguish them.
- A future recommendation-generation policy must state its research-action-to-direction mapping explicitly rather than inherit one.

## Scope

This ADR records repository layering and a Core package boundary.

It does not:

- implement `northstar_core.recommendations.futures` or any type in it;
- define entry, stop, target or validity rules, or any price-level policy;
- map research actions to trade directions;
- define lifecycle transition tables or Core function contracts;
- add persistence, Application services, API routes or Web views;
- change `FuturesRecommendation`, `RecommendationAction`, `OrderSide`, the paper target-position policy, FG-9 reviewed-candle protection or any SQLite schema;
- move `OrderSide` or `FuturesContractCount` out of Paper Trading.

## Future Considerations

Deferred:

- the F-2 engineering specification: lifecycle states and transitions, temporal validation, and Core function contracts;
- the recommendation-generation policy, including any research-action-to-direction mapping and price-level rules;
- recommendation persistence, publication, monitoring and API boundaries;
- relocation of `OrderSide` and `FuturesContractCount` to a shared package (already deferred by ADR-008).

## Related Documents

- ADR-001: Repository Structure
- ADR-002: Architecture Constitution
- ADR-008: Futures Broker Demo Execution Boundary
- ADR-009: Contract-Scoped Futures Economics
- Architecture Handbook: `architecture/Architecture-Handbook-v2.0.md`
- Platform Vision: `domain/01-platform-vision.md`
- Dependency Rules: `domain/05-dependency-rules.md`
- Ubiquitous Language: `architecture/Ubiquitous-Language.md`
