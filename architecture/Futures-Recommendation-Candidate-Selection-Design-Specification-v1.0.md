# Futures Recommendation Candidate Selection Design Specification v1.0

Status: **design for F-3; implemented in development in `northstar-core`, not released.** Governed by ADR-017 (Proposed; its product decisions are approved). This specification builds on the Futures Trade Recommendation Design Specification v1.0 (F-2) and adds no architectural decision beyond ADR-017. Section 9 records the product-owner decisions it depends on.

## 1. Purpose

Candidate selection decides which frozen futures research decisions may be considered for an actionable trade recommendation in one selection session, and records why every other decision was not.

It is the step between research and recommendation generation:

```
frozen research decisions --F-3 selection--> candidates (+ rejections) --F-4 generation--> FuturesTradeRecommendation
```

A **candidate** is a frozen research decision accepted for consideration:
- It is not a recommendation.
- It has no direction, entry, stop, target or validity.
- It carries its research action unchanged. F-3 never maps BUY or SELL to LONG or SHORT (ADR-017); that mapping, and any arbitration between candidates, belongs to F-4.

## 2. Existing Types Reused

| Type | Package | Use |
| --- | --- | --- |
| `FuturesRecommendation` | `northstar_core.strategy` | The frozen research assessment: action, analysis, strategy and decision instant. |
| `RecommendationAction` | `northstar_core.strategy` | The research action (BUY, SELL, HOLD), preserved unchanged. |
| `StrategyIdentity` | `northstar_core.strategy` | Approved-strategy scope and provenance. |
| `FuturesResearchDecisionReference` | `northstar_core.recommendations.futures` (F-2) | Natural key of the frozen decision: contract, timeframe, decision instant, strategy. |
| `FuturesProductReference`, `FuturesContract` | `northstar_core.futures` | Approved-product scope and contract identity. |
| `PointInTime`, `Timeframe` | `northstar_core.foundation.value_objects` | Selection session and timeframe. Instants are compared only through `PointInTime.compare`. |

ADR-017 permits Recommendations to consume these Strategy types. Only the selection sub-package imports `FuturesRecommendation` and `RecommendationAction`; the trade-setup and lifecycle modules still do not.

## 3. New Types and Public API

Sub-package `northstar_core.recommendations.futures.selection`, re-exported from `northstar_core.recommendations.futures`:

| Type | Kind | Purpose |
| --- | --- | --- |
| `FuturesResearchDecision` | value object | One frozen research decision: its reference and its assessment, required to agree. |
| `FuturesCandidateSelectionCriteria` | value object | The selection session close, approved products and approved strategies. |
| `CandidateRejectionReason` | `StrEnum` | Why a decision was not selected. |
| `FuturesRecommendationCandidate` | value object | A decision accepted for consideration (BUY or SELL only). |
| `FuturesCandidateRejection` | value object | A decision not selected, with exactly one reason. |
| `FuturesCandidateSelection` | value object | Criteria, candidates and rejections; `select(criteria, decisions)` computes it. |

Each value object has a matching `Invalid…Error(ValidationError)`. Structural form follows F-2: `@dataclass(frozen=True, slots=True)`, validation in `__post_init__`, explicit `__str__` and `__repr__`, and no `order=True` (no meaningful total order; F-2 section 3).

## 4. Input: FuturesResearchDecision

`FuturesResearchDecision(reference: FuturesResearchDecisionReference, assessment: FuturesRecommendation)`.

The reference and the assessment must describe the same decision:
- `reference.contract == assessment.contract`;
- `reference.decision_instant` is the same instant as `assessment.point_in_time`;
- `reference.strategy_identity == assessment.strategy_identity`.

A mismatch is an invalid input and raises `InvalidFuturesResearchDecisionError`. It is never treated as a rejection.

The timeframe lives on the reference, because the research assessment does not carry one. The decision must already be frozen. Application builds it from a stored `FuturesForwardResearchRecord`:
- `reference` from the record's natural key;
- `assessment` from `record.result.recommendation`.

Core cannot verify that a decision was frozen.

## 5. Criteria

`FuturesCandidateSelectionCriteria(session_close, products, strategies)`:

- **`session_close`** is the completion instant of the selection session: the latest *final* session for which candidates are selected.
  - Application resolves it from the trading-session calendar and the finality policy. Under FG-9 it must not be later than the approved final-through session.
  - Core cannot verify that it is final; it is a trusted caller-supplied temporal contract.
- **`products`** is the non-empty set of approved `FuturesProductReference`s, for example `NIFTY@NSE`.
- **`strategies`** is the non-empty set of approved `StrategyIdentity`s whose research may become recommendations.

Both sets are supplied as tuples, must contain no duplicates, and are stored in canonical sorted order, so equal criteria are equal values whatever order they were given in.

## 6. Selection Rules

Each distinct decision is either a candidate or a rejection with exactly one reason. The rules are checked in this fixed priority, and the first that applies decides:

| Priority | Reason | Rule | Justification |
| --- | --- | --- | --- |
| 1 | `AFTER_SELECTION_SESSION` | `decision_instant > session_close` | Look-ahead: the decision was made on data not available at the selection session. |
| 2 | `STALE_DECISION` | `decision_instant < session_close` | Freshness: only the decision made exactly at the latest final session qualifies. An older decision is stale or superseded (product-owner decision D2). |
| 3 | `UNSUPPORTED_TIMEFRAME` | `timeframe != 1d` | F-2 defines daily-candle semantics only. |
| 4 | `UNAPPROVED_PRODUCT` | contract's product not in `products` | Scope. |
| 5 | `UNAPPROVED_STRATEGY` | strategy not in `strategies` | Only approved, versioned strategies may produce recommendations. |
| 6 | `NO_DIRECTIONAL_VIEW` | research action is `HOLD` | HOLD states no direction, so it cannot become a trade setup under any mapping. |

Otherwise the decision is a candidate (BUY or SELL, unchanged).

**What these rules imply** (product-owner decisions D1, D3 and D4):
- Every contract of an approved product may yield a candidate. There is no front-month selection, near-expiry exclusion or rollover in F-3. F-4 rejects a setup whose validity window cannot fit before the contract cutoff.
- Each approved strategy yields its own candidate for the same contract and session. F-3 does not arbitrate between strategies or between contracts.

## 7. Integrity, Determinism and Reproducibility

- **Duplicates.**
  - Identical decisions supplied more than once are one decision.
  - Two different decisions with the same reference (natural key) are contradictory frozen evidence: selection raises `InvalidFuturesCandidateSelectionError` and never picks one.
- **Order independence.** The result does not depend on input order. Candidates and rejections are each sorted by product, contract expiration, timeframe, strategy and then decision instant (chronologically, via `PointInTime.compare`).
- **Purity.** Selection is pure: it reads no clock, generates no identifier, performs no I/O and mutates nothing.
- **Reconstruction.** Constructing a `FuturesCandidateSelection` directly from its parts recomputes the selection from the decisions it lists and refuses any mismatch. A stored selection therefore cannot hold a forged candidate or rejection.
- **Provenance.** Each candidate and rejection keeps the whole `FuturesResearchDecision`: reference, research action, strategy and analysis signals. F-4 can cite the research thesis and policy.

## 8. Errors

| Error | Raised when |
| --- | --- |
| `InvalidFuturesResearchDecisionError` | Wrong types, or the reference and assessment disagree. |
| `InvalidFuturesCandidateSelectionCriteriaError` | Wrong types, empty or duplicated products or strategies. |
| `InvalidFuturesRecommendationCandidateError` | A candidate built from a HOLD decision, or of the wrong type. |
| `InvalidFuturesCandidateRejectionError` | Wrong types. |
| `InvalidFuturesCandidateSelectionError` | Contradictory duplicate decisions, or a stored selection that is not what the criteria produce. |

## 9. Product-Owner Decisions

| ID | Decision | Approved option |
| --- | --- | --- |
| D1 | Research actions eligible as candidates | BUY and SELL, action kept unchanged; HOLD rejected; no LONG/SHORT mapping in F-3 (decided in F-4) |
| D2 | Freshness | Latest final session only |
| D3 | Contracts | Any researched contract of an approved product; contract life checked by F-4 |
| D4 | Several approved strategies on one contract | One candidate each; arbitration in F-4 |

## 10. Package Boundaries

The selection sub-package follows the F-2 boundaries. It may import:
- Foundation, Derivatives, Futures, Strategy and Recommendations;
- `__future__`, `dataclasses`, `enum` and `functools`.

It never imports:
- Paper Trading or Broker Execution;
- a clock, randomness, provider or outer layer.

Strategy never imports it.

## 11. Non-Goals and Later Dependencies

**Not in F-3:**
- the research-action-to-direction mapping;
- price levels;
- validity windows;
- risk sizing;
- persistence;
- publication;
- notification;
- performance;
- execution.

**Later dependencies:**
- **F-4:**
  - approve and apply the BUY/SELL to LONG/SHORT mapping (D1), recorded as an ADR-017 amendment or a new ADR;
  - arbitrate between candidates (D4);
  - check that a validity window fits before the contract cutoff (D3);
  - build `FuturesTradeRecommendation` values.
- **F-5 (Application):**
  - convert stored `FuturesForwardResearchRecord`s to `FuturesResearchDecision`s;
  - resolve `session_close` as the latest final session (FG-9);
  - supply the approved products and strategies from configuration;
  - persist selections insert-only if they must be audited.
