# Futures Recommendation Generation Design Specification v1.0

Status: **design for F-4; implemented in development in `northstar-core`, not released.**

Revision: this version includes the corrections from the independent F-4 implementation review (D-1, D-2, D-3), the product-owner policy P-1, one open recommendation per product, and the final verification corrections (J-1 to J-4). The Revision Record lists them.

Governing decisions:
- **ADR-018**, Futures V1 Research-to-Recommendation Generation Policy. Proposed; product-owner approved, architecture and engineering review pending.
- **ADR-017**. Proposed; its product decisions are approved.

This specification builds on the F-2 and F-3 specifications and adds no decision beyond ADR-018.

Generated proposals are **unvalidated trading-policy prototypes**. Nothing here claims a measured trading edge. Nothing here approves production parameters, a production price increment or live publication.

## 1. Purpose

Recommendation generation turns the candidates of one F-3 selection into validated, immutable trade-setup proposals, and records why every other candidate was not turned into one.

```text
frozen research --F-3--> candidates --F-4 direction, contract arbitration, levels, risk--> proposals (+ rejections)
proposal + caller-supplied identity --to_recommendation--> FuturesTradeRecommendation (F-2) --> lifecycle
```

A **proposal** is a `FuturesTradeRecommendation` without its identity:
- It is not published.
- It is not monitored.
- It is not a recommendation until F-5 assigns an identity and publishes it.

## 2. Approved Business Decisions

| ID | Decision (ADR-018) |
| --- | --- |
| F4-D1 | BUY to LONG, SELL to SHORT. SELL is a standalone bearish view, not "reduce a long". The policy carries a `short_enabled` flag; when false, SELL candidates are rejected `SHORT_NOT_ENABLED`. V1 development enables SHORT. |
| F4-D2 | Exactly one approved, caller-supplied strategy. Candidates of other strategies are rejected. A future multi-strategy disagreement is rejected and never ranked (section 10.2). |
| F4-D3 | Per product: only validity-eligible contracts are considered. Contradictory directions are rejected as a group. Otherwise the nearest expiration is selected. There is no liquidity inference, no truncation and no fallback. |
| F4-D4 | Movement unit U is the mean absolute close-to-close change over N = 19 changes (20 closes). Multipliers and the increment are explicit policy parameters. |
| F4-D5 | Price order, positive risk and reward, worst-entry reward-to-risk at or above a configured minimum, contract life, open-recommendation duplicates and contradictions. |
| P-1 | One open recommendation per product. Across every expiry of one futures product, at most one non-terminal recommendation may exist. A new candidate is rejected `DUPLICATE_OPEN_RECOMMENDATION` when an open recommendation on the product has its direction, and `CONTRADICTS_OPEN_RECOMMENDATION` when one has the other. Terminal recommendations block nothing (section 10.4). |
| Deviation 1 | `CONFLICTING_RESEARCH` is deferred until multi-strategy generation exists. Accepted by the product owner (section 10.2). |
| Deviation 4 | The product-direction conflict veto precedes `short_enabled`. Accepted by the product owner (section 10.3). |
| F4-D6 | An immutable `FuturesRecommendationProposal`, with `to_recommendation(identity)` taking a caller-supplied identity. Core generates no identity. |
| F4-D7 | Every material parameter is under one `RecommendationPolicyVersion`. Tests use synthetic fixtures only. |

## 3. Existing Types Reused

| Type | Package | Use |
| --- | --- | --- |
| `FuturesCandidateSelection`, `FuturesRecommendationCandidate` | `recommendations.futures` (F-3) | The input. Candidates are BUY or SELL, daily, decided exactly at the selection session close. |
| `FuturesResearchDecisionReference` | `recommendations.futures` (F-2) | The natural key of a candidate's decision. It keys evidence and orders output. |
| `FuturesTradeRecommendation`, `FuturesTradeRecommendationIdentity` | `recommendations.futures` (F-2) | The conversion target, and the caller-supplied identity. |
| `TradeDirection`, `EntryZone`, `StopLoss`, `PriceTarget`, `RecommendationValidity`, `RecommendationPolicyVersion` | `recommendations.futures` (F-2) | Proposal fields, unchanged. |
| `FuturesTradeRecommendationLifecycle`, `RecommendationStatus` | `recommendations.futures` (F-2) | The open-recommendation input and its non-terminal check. |
| `FuturesMarketObservationContext` | `strategy` | The frozen market evidence for one decision. |
| `RecommendationAction` | `strategy` | Read only for the F4-D1 mapping. Never altered. |
| `StrategyIdentity`, `ExplanationReason` | `strategy` | The approved strategy, and the thesis. |
| `FuturesContract`, `FuturesProductReference` | `futures` | Contract and product identity. |
| `QuoteValue` | `derivatives` | Every price level: a signed, finite canonical `Decimal`, with no tick and no rounding. |
| `PointInTime` | `foundation.value_objects` | Every instant, compared only through `PointInTime.compare`. |

The construction rules relied on, confirmed against the source:

- `FuturesTradeRecommendation` requires:
  - for LONG, `stop < entry.low <= entry.high < target`;
  - for SHORT, `target < entry.low <= entry.high < stop`;
  - a daily source for the same contract;
  - `decision_instant <= issued_at < eligible_from`;
  - a non-empty thesis of `ExplanationReason`s.
- `RecommendationValidity` requires `eligible_from <= valid_through <= contract_cutoff`, and raises otherwise.
- `FuturesMarketObservationContext`:
  - requires at least 20 closes and volumes, of equal length;
  - requires `session_low <= latest_quote <= session_high`;
  - accepts negative and zero quotations.

  The Application builder sets `latest_quote` to the decision bar's close and `recent_closes` to the trailing 20 closes ending with it.
- `ExplanationReason(rationale, supporting_signals=())` takes a non-empty stripped rationale and stripped non-empty signal strings.
- `FuturesTradeRecommendationLifecycle.status` is derived. `RecommendationStatus.is_terminal` is false only for `WAITING_FOR_ENTRY` and `ACTIVE`.

## 4. New Types and Public API

Sub-package `northstar_core.recommendations.futures.generation`, re-exported from `northstar_core.recommendations.futures`:

| Type | Kind | Purpose |
| --- | --- | --- |
| `FuturesRecommendationGenerationPolicy` | value object | Versioned generation parameters. |
| `FuturesCandidateEvidence` | value object | The frozen market evidence for one candidate, keyed by its decision reference. |
| `FuturesRecommendationSessionWindow` | value object | Caller-resolved eligibility, intended validity end and contract cutoff for one contract. |
| `PriceMovementUnit` | value object | Exact total of absolute close-to-close changes, and the number of changes. |
| `RewardToRisk` | value object | Exact reward and risk from the worst entry. |
| `GenerationRejectionReason` | `StrEnum` | Why a candidate produced no proposal. |
| `FuturesGenerationRejection` | value object | One candidate and its single rejection reason. |
| `FuturesRecommendationProposal` | value object | A validated setup without identity; `to_recommendation(identity)`. |
| `FuturesRecommendationGeneration` | value object | Policy, issuance instant, proposals and rejections. `generate(...)` computes it. |
| `FuturesRecommendationGeneration.exclusivity_breaches` | field: `tuple[FuturesProductReference, ...]` | Immutable, distinct and sorted. Lists the futures products whose caller-supplied state holds more than one non-terminal recommendation (section 10.4). It reports what was supplied. It does not prove that the supplied open-recommendation collection is complete. |

Each value object has a matching `Invalid…Error(ValidationError)`.

The structural form follows F-2 and F-3: `@dataclass(frozen=True, slots=True)`, validation in `__post_init__`, explicit `__str__` and `__repr__`, and no `order=True`. This is the F-2 section 3 deviation from ES-002, for the same reason: there is no meaningful total order.

Private module `_price_levels` holds the arithmetic. It is not exported.

## 5. Versioned Configuration

`FuturesRecommendationGenerationPolicy` has the following fields:

| Field | Type | Rule |
| --- | --- | --- |
| `version` | `RecommendationPolicyVersion` | Required. |
| `approved_strategies` | `tuple[StrategyIdentity, ...]` | A tuple. Non-empty, no duplicates, and **exactly one** in V1. |
| `lookback_changes` | `int` | A positive integer that is not a `bool`. **Must be 19 in V1.** |
| `entry_multiplier` (kE) | `Decimal` | Finite, `>= 0`. |
| `stop_multiplier` (kS) | `Decimal` | Finite, `> 0`. |
| `target_multiplier` (kT) | `Decimal` | Finite, `> 0`. |
| `price_increment` (T) | `Decimal` | Finite, `> 0`. |
| `min_reward_to_risk` | `Decimal` | Finite, `> 0`. |
| `short_enabled` | `bool` | Strictly `bool`. |

Further rules:

- Numeric parameters must be `Decimal` instances. `float`, `int`, `str` and `bool` are refused, so nothing is converted implicitly.
- Equal Decimals, such as `1.5` and `1.50`, make equal and equally hashed policies.
- There are no defaults: every value is supplied explicitly.
- The property `strategy` returns the single approved strategy.

**Governance.** Any change to any field other than `version` requires a new `version` (ADR-018, F4-D7). Core cannot detect two policies that share a version but differ in parameters across runs. Enforcing that belongs to a future Application policy registry.

The price increment is caller-supplied. A positive increment that passes validation is **not** evidence that it matches the current NSE contract specification.

## 6. Inputs and Evidence Requirements

`FuturesRecommendationGeneration.generate(policy, selection, evidence, windows, open_recommendations, issued_at)` takes:

- `policy`: a `FuturesRecommendationGenerationPolicy`.
- `selection`: an F-3 `FuturesCandidateSelection`. Only its candidates are generated from, and nothing in it is mutated.
- `evidence`: a `tuple[FuturesCandidateEvidence, ...]`.
- `windows`: a `tuple[FuturesRecommendationSessionWindow, ...]`.
- `open_recommendations`: a `tuple[FuturesTradeRecommendationLifecycle, ...]`.
- `issued_at`: a `PointInTime`, the caller's issuance instant.

### 6.1 FuturesCandidateEvidence

`FuturesCandidateEvidence(source: FuturesResearchDecisionReference, context: FuturesMarketObservationContext)`.

Constructing it raises `InvalidFuturesCandidateEvidenceError` unless all of the following hold:

- `context.contract == source.contract`;
- `context.observed_at` is the same instant as `source.decision_instant`, so there is no look-ahead and no stale evidence;
- `context.timeframe == source.timeframe == 1d`;
- `context.recent_closes[-1] == context.latest_quote`, so the daily evidence ends at its own decision close.

The strategy is represented only on `source`. The context carries no strategy.

### 6.2 FuturesRecommendationSessionWindow

`FuturesRecommendationSessionWindow(contract, eligible_from, valid_through, contract_cutoff)`:

- `eligible_from <= valid_through`, or the constructor raises.
- `valid_through` and `contract_cutoff` are deliberately not ordered here, so that an inadequate contract life can be a policy rejection rather than an error.
- `expires_before_eligibility` is true when `eligible_from > contract_cutoff`.
- `fits_contract` is true when `valid_through <= contract_cutoff`.
- `to_validity()` returns the F-2 `RecommendationValidity`, and raises if the window does not fit.

Every instant is a trusted, caller-resolved temporal contract (F-2 section 17). Core computes no weekend, holiday, special session or last trading day.

### 6.3 Input-level validation (errors)

`generate` raises `InvalidFuturesRecommendationGenerationError`, and never returns a rejection, when:

1. any argument has the wrong type, or a collection is not a tuple of the right element type;
2. `issued_at` is earlier than the selection session close;
3. two different evidence values share one source, or two different windows share one contract (identical repeats are one value);
4. an evidence source is not a candidate of the selection, or a window's contract is not the contract of any candidate;
5. a window's `eligible_from` is not strictly after `issued_at` (F-2 timing);
6. an open recommendation was issued after `issued_at`, or two different lifecycles share one recommendation identity. This applies to every supplied lifecycle, terminal or not. A terminal lifecycle is not an error: it is validated and then ignored (section 10.4);
7. a generated level cannot be represented exactly. This only happens beyond 28 significant digits.

Malformed or contradictory input is therefore never relabelled as an ordinary rejected trade.

## 7. Research-to-Direction Mapping

| Research action | `short_enabled = true` | `short_enabled = false` |
| --- | --- | --- |
| BUY | LONG | LONG |
| SELL | SHORT | rejected `SHORT_NOT_ENABLED` |
| HOLD | unreachable: F-3 rejects it | unreachable |

The mapping is a private table in the generation module. `RecommendationAction` values are read, never changed. The SHORT proposal is a new setup. It never instructs anyone to reduce or close an existing LONG, and Paper Trading's SELL-to-negative-target mapping is untouched.

## 8. Price-Level Mathematics

### 8.1 Exactness

- All level arithmetic is exact rational arithmetic (`fractions.Fraction`) over the exact `Decimal` inputs. There is no binary floating point and no intermediate rounding.
- The only rounding is the documented alignment to the increment. Alignment uses a true floor and ceiling, correct for negative values and for values that cross zero. It never truncates toward zero.
- An aligned level is `k * T` for an integer `k`. It is converted to `Decimal` in a 28-digit context that traps inexact results, so a level that cannot be represented exactly raises an error instead of being rounded silently.
- U, and the reward-to-risk ratio, are also *displayed* to 28 significant digits with `ROUND_HALF_EVEN`, matching `_directional_signal`. That display value never decides anything.
- **Decimal contexts (D-2).**
  - Every Decimal operation that can signal runs inside `localcontext(...)` on a fresh copy of one of two private module contexts: the exact context and the display context.
  - The module contexts are configuration only. No calculation records a flag on them, results do not depend on call order, and the caller's own context is neither used nor changed.
- **Precision failures are domain errors (D-1).**
  - During generation, an unrepresentable level raises `InvalidFuturesRecommendationGenerationError`.
  - In the proposal constructor, an unrepresentable reward or risk raises `InvalidFuturesRecommendationProposalError`.
  - Both are chained from the `decimal.Inexact` or `decimal.Rounded` signal. Neither signal escapes raw, and nothing is rounded.

### 8.2 Movement unit

Take the trailing `N + 1` closes `C0 … CN` of `context.recent_closes`. N is the policy's `lookback_changes`, which is 19 in V1.

```text
S = |C1 - C0| + |C2 - C1| + ... + |CN - C(N-1)|        (exact)
U = S / N
```

- If fewer than `N + 1` closes exist, the candidate is rejected `INSUFFICIENT_HISTORY`.
  - With V1's N = 19, a valid context always has at least 20 closes, so this is unreachable through the public API.
  - It stays as a guard because the twenty-close minimum is a Strategy requirement that F-4 must not silently rely on.
- If `S = 0`, the candidate is rejected `ZERO_PRICE_MOVEMENT_UNIT`.

`PriceMovementUnit(total_absolute_change=S, changes=N)` is carried on the proposal.

### 8.3 Levels

Here `P = context.latest_quote`, the decision-session close, and T is the price increment. `floor_T(x) = floor(x / T) * T` and `ceil_T(x) = ceil(x / T) * T`.

| Level | LONG | SHORT |
| --- | --- | --- |
| entry.low | `ceil_T(P - kE*U)` | `ceil_T(P - kE*U)` |
| entry.high | `floor_T(P + kE*U)` | `floor_T(P + kE*U)` |
| stop | `floor_T(entry.low - kS*U)`, away from the zone | `ceil_T(entry.high + kS*U)`, away from the zone |
| target | `floor_T(entry.high + kT*U)`, toward the zone | `ceil_T(entry.low - kT*U)`, toward the zone |
| worst entry | `entry.high` | `entry.low` |
| risk | `worst - stop` | `stop - worst` |
| reward | `target - worst` | `worst - target` |

- The entry zone rounds inward. If the result has `entry.low > entry.high`, the candidate is rejected `DEGENERATE_ENTRY_ZONE`. A single-price zone, `low == high`, is valid.
- Because `kS > 0` and `U > 0`, the stop always lies strictly outside the zone.
- The target can collapse onto the zone boundary when `kT*U` is smaller than one increment. If `risk <= 0` or `reward <= 0`, the candidate is rejected `INVALID_PRICE_ORDER`. Otherwise the F-2 ordering holds strictly.
- No level is ever adjusted to make an order valid.

### 8.4 Mathematical validity is not exchange tradability

Every level is an exact multiple of the caller-supplied increment and satisfies the F-2 ordering. That proves mathematical consistency only. It does not prove that:
- the increment is the current exchange tick;
- the level is inside any price band;
- the setup could be filled.

## 9. Worst-Entry Reward-to-Risk

- `RewardToRisk(reward, risk)` holds the exact `Decimal` reward and risk. Both must be `> 0`.
- `meets(minimum)` returns `reward >= minimum * risk`, evaluated exactly by cross-multiplication. A ratio exactly equal to the minimum is accepted.
- If it does not meet the minimum, the candidate is rejected `INSUFFICIENT_REWARD_TO_RISK`.
- `ratio` is a display value only (section 8.1).
- **Fixed-point text (D-3).** Both strings use fixed-point notation, for example `0.0000003:0.0000002`, never `3E-7:2E-7`:
  - `str(RewardToRisk)` is `reward:risk`;
  - `str(PriceMovementUnit)` is `U over N changes`. When the displayed U is not exactly `S / N`, it reads `U (rounded to 28 significant digits) over N changes`; an exact U carries no label.

  The text is presentation. The canonical values are the `Decimal` fields.

Because the stop rounds away from the zone and the target rounds toward it, alignment never improves the ratio. The worst-entry ratio is the lowest ratio of any entry inside the zone.

The ratio is a geometric property of the levels. It is not a probability, an expected return or a forecast. No transaction cost or slippage is modelled.

## 10. Arbitration

### 10.1 Per-candidate filters

| Order | Rejection | Rule |
| --- | --- | --- |
| 1 | `STRATEGY_NOT_APPROVED` | The candidate's strategy is not the policy's approved strategy. |
| 2 | `MISSING_SESSION_WINDOW` | No window was supplied for the candidate's contract. |
| 3 | `CONTRACT_EXPIRED` | `eligible_from > contract_cutoff`. |
| 4 | `INSUFFICIENT_CONTRACT_LIFE` | `valid_through > contract_cutoff`. The window is never shortened. |

A candidate that passes all four is **eligible**.

### 10.2 Strategy agreement

V1 approves exactly one strategy, and F-3 admits at most one decision per contract, timeframe, instant and strategy. Within one session, each contract therefore has at most one eligible candidate, so no strategy disagreement can occur.

The policy for a future multi-strategy version is fixed now, but is not implemented:
- disagreement on one contract rejects the whole conflicting set;
- agreement yields one proposal;
- neither case is ever resolved by a confidence score, a name, input order or file order.

The rejection value for it (`CONFLICTING_RESEARCH`) will be added together with that version, so that V1 carries no unreachable state. The product owner accepted this deferral: same-contract conflicting research from several approved strategies is out of scope until multi-strategy generation is introduced.

### 10.3 Per-product contract arbitration

For each product, processed independently and never compared across products, over its eligible candidates:

| Order | Condition | Outcome |
| --- | --- | --- |
| 5 | Both BUY and SELL among the eligible candidates | Every eligible candidate of the product is rejected `CONTRADICTORY_PRODUCT_DIRECTION`. |
| 6 | All eligible candidates are SELL and `short_enabled` is false | Each is rejected `SHORT_NOT_ENABLED`. |
| 7 | Otherwise | The candidate with the nearest `expiration_date` is **selected**. Every other eligible candidate is rejected `NOT_NEAREST_ELIGIBLE_CONTRACT`. |

- **Ties.** Contracts of one product differ by expiration date, and each contract has at most one eligible candidate (section 10.2), so the nearest expiry is always unique.
- **Precedence of contradiction (conservative research-conflict veto).**
  - A contradiction is a fact about the product's research, so it is detected before `short_enabled`.
  - With BUY on one eligible expiry and SELL on another, every eligible candidate is rejected `CONTRADICTORY_PRODUCT_DIRECTION`, even when SHORT is disabled.
  - Disabling SHORT never turns that conflict into a LONG proposal issued against bearish research.
  - The product owner accepted this ordering.
- **No fallback.** The selected candidate must pass sections 10.4 to 10.5. If it fails, the product yields no proposal and no other expiry is tried.
- **Liquidity.** Nearest expiry is a deterministic rule, not a liquidity claim.

### 10.4 Open recommendations: one per product (P-1)

**Rule.** Across every expiry of one futures product, at most one non-terminal recommendation may exist.

**Product identity.**
- Products are compared by `FuturesProductReference` equality, which is product code plus exchange code. They are never compared by contract, symbol text or display string.
- Different expiries of one product share the rule. Different products are independent. So is the same symbol on another exchange.

**Open versus terminal.**
- An open recommendation is a lifecycle whose `RecommendationStatus.is_terminal` is false: today `WAITING_FOR_ENTRY` or `ACTIVE`.
- The other six statuses are terminal. A terminal lifecycle is validated (section 6.3) and then ignored: it blocks nothing.

The caller supplies the lifecycles relevant to the products being generated. For the selected candidate, over the open recommendations of its product:

| Order | Condition, on any expiry of the same product | Rejection |
| --- | --- | --- |
| 8 | An open recommendation has the opposite direction | `CONTRADICTS_OPEN_RECOMMENDATION` |
| 9 | An open recommendation has the same direction | `DUPLICATE_OPEN_RECOMMENDATION` |

**Cross-expiry examples:**

| Existing open recommendation | New candidate | Result |
| --- | --- | --- |
| NIFTY October LONG, `WAITING_FOR_ENTRY` | NIFTY November LONG | `DUPLICATE_OPEN_RECOMMENDATION` |
| NIFTY October LONG, `ACTIVE` | NIFTY November LONG | `DUPLICATE_OPEN_RECOMMENDATION` |
| NIFTY October LONG, `ACTIVE` | NIFTY November SHORT | `CONTRADICTS_OPEN_RECOMMENDATION` |
| NIFTY October SHORT, `ACTIVE` | NIFTY November LONG | `CONTRADICTS_OPEN_RECOMMENDATION` |
| NIFTY October SHORT, `WAITING_FOR_ENTRY` | NIFTY December SHORT | `DUPLICATE_OPEN_RECOMMENDATION` |
| NIFTY October, any terminal status | NIFTY November, valid | Proposal: not blocked |
| NIFTY October, open | BANKNIFTY October, valid | Proposal: independent product |
| none | NIFTY, valid | Unchanged F-4 behaviour |

**Supplied state that already breaks exclusivity.** Several open recommendations on one product are not repaired, deleted or chosen between. They fail closed:
- open recommendations in both directions contradict any new candidate on the product, LONG or SHORT;
- several open recommendations in one direction duplicate a same-direction candidate and contradict the other.

The check is set-based, so the order of the supplied lifecycles cannot change the outcome. Identical repeated lifecycles are one value. Two different lifecycles under one identity are an error. Each product whose supplied state holds more than one open recommendation is listed, sorted, in `FuturesRecommendationGeneration.exclusivity_breaches` for Application reconciliation. This includes a product that has no candidate in this generation.

**Relationship to same-session arbitration.** P-1 adds cross-session, cross-expiry protection. It does not replace sections 10.1 to 10.3:
- same-session contradiction still takes precedence;
- the nearest eligible contract is still the one checked, and the other eligible candidates stay `NOT_NEAREST_ELIGIBLE_CONTRACT`;
- an open-recommendation rejection never causes a fallback to another expiry.

**Scope of the rule.** Product exclusivity is a Futures V1 product policy for published recommendations. It is not an exchange rule, and it says nothing about spreads, hedges or positions that traders may hold. An open recommendation is lifecycle state. It is never a Paper Trading position, an order or a broker holding, none of which is read here.

**Trust boundary.** Core validates each supplied lifecycle, but cannot know about one that was omitted. F-5 must:
- load every non-terminal recommendation across all expiries of the product;
- reconcile lifecycle validity and expiry before generation;
- handle retries and already-published proposals idempotently;
- enforce product-level uniqueness transactionally at persistence and publication, so that concurrent requests cannot bypass it;
- keep recommendation history insert-only.

No cancellation, revision or supersession event is added to F-2.

### 10.5 Evidence, price and risk on the selected candidate

| Order | Rejection |
| --- | --- |
| 10 | `MISSING_MARKET_EVIDENCE`: no evidence for the candidate's source. |
| 11 | `INSUFFICIENT_HISTORY` |
| 12 | `ZERO_PRICE_MOVEMENT_UNIT` |
| 13 | `DEGENERATE_ENTRY_ZONE` |
| 14 | `INVALID_PRICE_ORDER` |
| 15 | `INSUFFICIENT_REWARD_TO_RISK` |

A candidate passing every check becomes a proposal.

## 11. Rejection Taxonomy and Priority

`GenerationRejectionReason` members, in the fixed priority order of sections 10.1 to 10.5:

| Priority | Reason |
| --- | --- |
| 1 | `STRATEGY_NOT_APPROVED` |
| 2 | `MISSING_SESSION_WINDOW` |
| 3 | `CONTRACT_EXPIRED` |
| 4 | `INSUFFICIENT_CONTRACT_LIFE` |
| 5 | `CONTRADICTORY_PRODUCT_DIRECTION` |
| 6 | `SHORT_NOT_ENABLED` |
| 7 | `NOT_NEAREST_ELIGIBLE_CONTRACT` |
| 8 | `CONTRADICTS_OPEN_RECOMMENDATION` |
| 9 | `DUPLICATE_OPEN_RECOMMENDATION` |
| 10 | `MISSING_MARKET_EVIDENCE` |
| 11 | `INSUFFICIENT_HISTORY` |
| 12 | `ZERO_PRICE_MOVEMENT_UNIT` |
| 13 | `DEGENERATE_ENTRY_ZONE` |
| 14 | `INVALID_PRICE_ORDER` |
| 15 | `INSUFFICIENT_REWARD_TO_RISK` |

- The first reason that applies is recorded, so each rejection has exactly one reason.
- `FuturesGenerationRejection(candidate, reason)` keeps the whole F-3 candidate. Reference, research action, strategy and analysis signals are therefore never lost.

## 12. Deterministic Evaluation Sequence

1. Validate all inputs (section 6.3), and collapse identical repeated evidence, windows and lifecycles.
2. Apply the per-candidate filters (10.1).
3. Group the eligible candidates by product, and arbitrate each product (10.3).
4. For each selected candidate, check the open recommendations of its product (10.4), then evidence, price and risk (10.5).
5. Every candidate of the selection ends as exactly one proposal or one rejection.
6. Sort proposals and rejections by product, contract expiration, timeframe, strategy and decision instant, which is the F-3 order.

The result does not depend on the order of any input tuple. Generation reads no clock, generates no identifier, performs no I/O and mutates nothing.

`FuturesRecommendationGeneration(policy, issued_at, proposals, rejections, exclusivity_breaches=())` validates its own structure:
- types;
- no source appears twice;
- canonical order;
- every proposal carries the policy's version, `issued_at` and approved strategy;
- `exclusivity_breaches` holds distinct `FuturesProductReference`s in sorted order;
- no proposal is for a product listed in `exclusivity_breaches`, compared by `FuturesProductReference` equality. Generation rejects every candidate of such a product, so a result pairing the two is refused with `InvalidFuturesRecommendationGenerationError`. This checks only the result's own consistency. It does not check completeness of the supplied open recommendations.

It does not recompute the generation, because it does not hold the evidence, windows or open recommendations it was computed from. Unlike F-3, a stored result is not self-verifying; F-5 persistence must keep the inputs if audit replay is required.

## 13. Proposal

`FuturesRecommendationProposal` has the following fields:

- `contract`, `direction`, `entry`, `stop`, `target`, `validity`, `source`, `policy_version`, `issued_at`, `thesis`: as in F-2;
- `reference_price: QuoteValue`;
- `movement_unit: PriceMovementUnit`;
- `reward_to_risk: RewardToRisk`.

The constructor raises `InvalidFuturesRecommendationProposalError` unless:

- the F-2 price order holds for the direction;
- `reward_to_risk` equals the worst-entry reward and risk recomputed from the levels;
- the source is daily and for the same contract;
- `source.decision_instant <= issued_at < validity.eligible_from`;
- the thesis is a non-empty tuple of `ExplanationReason`.

A reward or risk that cannot be represented exactly in 28 significant digits also raises `InvalidFuturesRecommendationProposalError`, chained from the decimal signal (D-1). Other validation failures keep their own messages and are never reported as precision failures.

The thesis is deterministic and factual:

1. Rationale `Research <ACTION> by <strategy> on <contract> decided at <instant>`, with the analysis's summarized signals as supporting signals.
2. Rationale `<DIRECTION> setup under policy <version>`, with supporting signals for:
   - the reference close;
   - U and N;
   - the worst entry;
   - reward and risk.

It contains no forecast or narrative.

Every number in it is written in fixed-point notation (D-3). Examples:
- `reference close 1.0000019`;
- `worst entry -0.000000075`;
- `reward 0.0000003 against risk 0.0000002`.

When the displayed U is not the exact quotient `S / N`, the text says so: `mean absolute close-to-close change 20.15789473684210526315789474 (rounded to 28 significant digits) over 19 changes`. An exact U carries no note, for example `20 over 19 changes`.

`str(proposal)` is fixed-point and unambiguous for signed values. The entry bounds are bracketed:

```text
proposal LONG NIFTY@NSE 2026-10-27 entry [-0.000000125, -0.000000075] stop -0.000000275 target 0.000000225 reward:risk 0.0000003:0.0000002 under <version>
```

The F-2 value-object strings are not changed.

`to_recommendation(identity)`:
- accepts only a `FuturesTradeRecommendationIdentity`, and raises otherwise;
- returns the F-2 `FuturesTradeRecommendation` with exactly the proposal's fields.

The proposal's own invariants guarantee that the F-2 constructor accepts it.

## 14. Worked Synthetic Examples

These examples use **synthetic, non-production calibration fixtures**:

| Parameter | Value |
| --- | --- |
| N | 19 |
| kE | 0.25 |
| kS | 1.5 |
| kT | 3 |
| T | 0.05 |
| minimum reward-to-risk | 1.5 |

They are not trading recommendations, and the increment is not a verified NSE tick.

**LONG (BUY).**
- Closes: `24000, 24020, …, 24360` (19 values, step 20), then `24383`.
- `S = 18*20 + 23 = 383`, so `U = 383/19 ≈ 20.15789473684210526315789474`. `P = 24383`.
- Entry: raw `[24377.9605…, 24388.0394…]`, aligned to `[24378, 24388]`.
- Stop: raw `24378 - 30.2368…` `= 24347.7631…`, aligned to `24347.75`.
- Target: raw `24388 + 60.4736…` `= 24448.4736…`, aligned to `24448.45`.
- Worst entry `24388`. Risk `40.25`, reward `60.45`, ratio `≈ 1.5019`. Accepted.

**SHORT (SELL).** The same closes in reverse order, ending at `P = 24000`, so `U` is identical.
- Entry: aligned `[23995, 24005]`.
- Stop: raw `24035.2368…`, aligned to `24035.25`.
- Target: raw `23934.5263…`, aligned to `23934.55`.
- Worst entry `23995`. Risk `40.25`, reward `60.45`. Accepted.

**Rejection examples**, each with the LONG fixture unless stated:

| Change | Result |
| --- | --- |
| minimum reward-to-risk `1.6` | `INSUFFICIENT_REWARD_TO_RISK` (60.45 < 64.4) |
| `short_enabled = false`, SELL | `SHORT_NOT_ENABLED` |
| all twenty closes equal | `ZERO_PRICE_MOVEMENT_UNIT` |
| `kE = 0.1`, `T = 10` | raw zone `[24380.98…, 24385.01…]` aligns to low `24390` > high `24380`: `DEGENERATE_ENTRY_ZONE` |
| `kT = 0.001` | `kT*U ≈ 0.02 < T`, so the target collapses onto `24388`: `INVALID_PRICE_ORDER` |
| October window ends after the October cutoff, November fits | October `INSUFFICIENT_CONTRACT_LIFE`, November selected |
| BUY on October, SELL on November, both eligible | both `CONTRADICTORY_PRODUCT_DIRECTION` |
| BUY on October and November, both eligible | October proposal, November `NOT_NEAREST_ELIGIBLE_CONTRACT` |

## 15. Package Exports and Dependency Rules

`generation/__init__.py` exports the types of section 4 and their errors. `recommendations/futures/__init__.py` re-exports them.

The generation sub-package may import:
- Foundation, Derivatives, Futures and Strategy;
- the F-2 value objects and lifecycle, and the F-3 selection;
- `__future__`, `dataclasses`, `enum`, `functools`, `decimal` and `fractions`.

Only selection and generation may import the research vocabulary (`RecommendationAction`, `FuturesRecommendation`). It never imports Paper Trading, Broker Execution, a clock, randomness, a provider or an outer layer. Strategy never imports it.

## 16. Trust Boundaries

Core validates the structure and mutual consistency of what it is given. It cannot prove, and F-5 or later Application services must establish before publication or performance use:

- **Market data.**
  - The provider's contractual authorization (G-1).
  - That the evidence is final, approved and complete, with no omitted sessions.
- **Calendar.** That `eligible_from`, `valid_through` and `contract_cutoff` were resolved correctly from the NSE calendar and the contract's last trading session.
- **Price increment.** That it matches the current exchange specification.
- **Open recommendations.** That the supplied view is complete across every expiry of each product, current, and protected against concurrent publication (section 10.4).
- **Versioning.** That a policy version has never been reused with different parameters.

No boolean "authorized" or "verified" flag is modelled, because nothing in Core could make it true.

## 17. Tests and Acceptance Criteria

Tests live in `northstar-core/tests/recommendations/futures/generation/`, use synthetic data only, and cover:

- **Policy:** valid construction, every invalid field, float, int and bool refusal, N = 19 pinning, the strategy set, immutability, equality, hashing.
- **Arithmetic:**
  - the exact U and the golden levels of section 14;
  - floor and ceiling for negative values and values crossing zero;
  - every alignment direction;
  - single-price and degenerate zones;
  - grid alignment of every level;
  - repeatability.
- **Risk:** worst-entry reward-to-risk for LONG and SHORT, equality with the threshold, below the threshold, target collapse.
- **Temporal:**
  - the issuance and eligibility boundaries;
  - `valid_through == contract_cutoff`;
  - insufficient life and expired contract;
  - wrong evidence instant, contract or timeframe;
  - the evidence close not matching.
- **Arbitration:**
  - nearest eligible contract;
  - excluded nearer contract;
  - contradictory expiries;
  - unapproved strategy;
  - open duplicates and contradictions;
  - independent products;
  - product exclusivity (P-1):
    - the cross-expiry matrix of section 10.4;
    - every terminal and non-terminal status, in both directions;
    - same-contract guards;
    - product identity by equality;
    - fail-closed handling of a supplied state that already breaks exclusivity, with its breach report;
    - order invariance;
    - no mutation of the supplied lifecycles;
  - every candidate disposed of;
  - permutation invariance.
- **Compatibility:**
  - BUY to LONG and SELL to SHORT;
  - short disabled;
  - F-3 candidates unchanged;
  - proposal-to-F-2 conversion with valid and invalid identities;
  - no identity generation.
- **Boundaries:** imports, exports, and no reverse dependency.
- **Numeric contracts:**
  - D-1: proposal-domain error chained from the decimal signal, and high-precision proposals within precision;
  - D-2: module contexts free of flags after inexact, display and error paths, the caller's context untouched, order independence;
  - D-3: fixed-point text for small, negative and normal-scale values, deterministic.

Acceptance requires:
- every P-1 matrix case of section 10.4 holds;
- the new tests pass;
- the full Core suite passes with no regression;
- Ruff check and format pass.

## 18. Deferred Functionality and Production Gates

**Deferred:**
- multi-strategy arbitration (section 10.2);
- invalidation before entry (F-2 N7);
- transaction cost and slippage;
- persistence, identity derivation, publication, notification and monitoring (F-5);
- performance analytics (F-6).
- rollover of an open recommendation to the next expiry, replacement or supersession of an open recommendation, hedging and multi-leg (spread) recommendations. Under P-1 none of these exists: a product's open recommendation must become terminal before another can be generated.

**Production gates**, all outstanding:
- an authoritative production price increment;
- historical calibration of N's companions `kE`, `kS`, `kT`, the minimum reward-to-risk and the validity horizon;
- G-1 data permission;
- regulatory permission to publish recommendations;
- separate execution permission;
- FG-9.4 release safety.

## Approval Summary

| Item | State |
| --- | --- |
| Product-owner decisions F4-D1 to F4-D7 | Approved (ADR-018) |
| P-1 one open recommendation per product; deviations 1 and 4 | Approved by the product owner (ADR-018) |
| Review findings D-1, D-2, D-3 | Corrected; focused independent review pending |
| ADR-018 architecture and engineering review | Pending |
| ADR-017 acceptance | Pending (unchanged by F-4) |
| Production parameters and price increment | Not approved |

## Revision Record

| Revision | Change | Origin |
| --- | --- | --- |
| v1.0 | Initial F-4 generation design. | F-4 implementation |
| v1.0, review corrections | D-1: precision failures in proposal validation raise `InvalidFuturesRecommendationProposalError`. D-2: Decimal operations run on local context copies, so no shared flags. D-3: fixed-point numeric text. | Independent F-4 implementation review |
| v1.0, P-1 | One open recommendation per product across expiries. Terminal lifecycles are accepted and ignored, where they were previously refused. Fail-closed handling and the `exclusivity_breaches` report for supplied state that already breaks exclusivity. Deviations 1 and 4 recorded as accepted. | Product-owner policy approval |
| v1.0, final corrections | J-1: a stored result refuses a proposal on a product listed in `exclusivity_breaches`. J-2: fixed-point, bracketed `str(proposal)`. J-3: `str(PriceMovementUnit)` labels a rounded display. J-4: `exclusivity_breaches` added to the section 4 public API table. | F-4 correction verification |
