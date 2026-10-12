# ADR-018: Futures V1 Research-to-Recommendation Generation Policy

## Status

Proposed

The product owner approved the F-4 policy choices recorded in this ADR (F4-D1 to F4-D7) for development. After the independent F-4 implementation review, the product owner also approved:
- Decision 8, one open recommendation per product (P-1);
- the omission of `CONFLICTING_RESEARCH` from single-strategy V1 (deviation 1);
- the conservative conflict veto before `short_enabled` (deviation 4).

Architecture Review and Engineering Review have not yet taken place, and this ADR records no reviewer sign-off. ADR-017, which this ADR builds on, is itself still Proposed; nothing here changes its status.

## Date

2026-10-12

## Decision Makers

- Product owner: approved the F-4 policy decisions F4-D1 to F4-D7, P-1, and deviations 1 and 4.
- Northstar Architecture Team: formal review pending.

## Context

ADR-017 created the `northstar_core.recommendations.futures` package and approved LONG and SHORT as trade directions. It deliberately left open:

- the mapping from research actions (BUY, SELL, HOLD) to trade directions;
- the price-level policy for entry, stop and target.

F-2 implemented the immutable `FuturesTradeRecommendation` and its lifecycle. F-3 implemented deterministic candidate selection: BUY and SELL decisions of approved products and strategies from the latest final session, with the research action kept unchanged.

F-4 turns those candidates into validated trade-setup proposals. That requires product decisions that no earlier document made.

The capabilities actually available in Core constrain those decisions:

- The frozen research evidence (`FuturesMarketObservationContext`) holds the decision-session close, the previous close, the decision bar's high and low, and at least twenty closes and volumes.
- It holds no high or low history. Average true range therefore cannot be computed without inventing observations.
- No tick size or price-increment model exists. `FuturesProductSpecification` defers it.
- Core has no trading-session calendar and no last-trading-day model. Validity instants are caller-supplied (F-2).
- Core never generates identifiers. Recommendation identity is caller-supplied (F-2), and its derivation is deferred to F-5.
- Research outcomes are measured as market movement only. No directional edge has been measured for BUY or SELL.

## Decision

### 1. Research action to trade direction (F4-D1)

- An explicit mapping inside the F-4 generation package: BUY maps to LONG and SELL maps to SHORT.
- HOLD cannot reach F-4, because F-3 rejects it.
- A futures research SELL is a standalone bearish assessment of one contract: a falling close, a short average below the long average, and confirmed volume. It is not an instruction to reduce or close an existing LONG. A SHORT recommendation is a new, independent trade setup.
- The generation policy carries a `short_enabled` flag. When it is false, SELL candidates are rejected with `SHORT_NOT_ENABLED`. The initial V1 development policy enables SHORT.
- Any change to the mapping behaviour, including `short_enabled`, requires a new `RecommendationPolicyVersion`.

The research vocabulary is unchanged. `FuturesRecommendation`, `RecommendationAction`, the Paper Trading target-exposure policy and `OrderSide` keep their existing meaning. The mapping lives only in F-4, not in Strategy or F-3.

### 2. Strategy arbitration (F4-D2)

- The Futures V1 generation policy approves exactly one caller-supplied `StrategyIdentity`. No strategy name is hard-coded in Core.
- A candidate from any other strategy is rejected with `STRATEGY_NOT_APPROVED`.
- For a future policy that approves several strategies: if approved strategies disagree on one contract and session, the conflicting set is rejected. No confidence score, ranking, name ordering or input order may resolve a disagreement. That rule is specified but not implemented in V1. With one approved strategy and F-3's one-decision-per-reference rule, a disagreement cannot arise.
- Accepted deviation 1: the `CONFLICTING_RESEARCH` rejection is not part of V1. Same-contract conflicting research from several approved strategies is deferred until multi-strategy generation is introduced. No unreachable code is added for it.

### 3. Contract selection (F4-D3)

For each approved product and session:

- Only contracts whose caller-supplied validity window fits before the contract cutoff are eligible. The window is never shortened to fit.
- If eligible contracts carry contradictory BUY and SELL research, every eligible candidate of that product is rejected with `CONTRADICTORY_PRODUCT_DIRECTION`.
- Accepted deviation 4, a conservative research-conflict veto: this contradiction is evaluated before `short_enabled`. Disabling SHORT therefore never turns contradictory research across expiries into a LONG proposal. The BUY candidate is vetoed together with the SELL candidate.
- Otherwise the eligible contract with the nearest expiration date is selected. Every other eligible candidate is rejected with `NOT_NEAREST_ELIGIBLE_CONTRACT`.
- Within one product, contracts are identified by their expiration date. Two candidate contracts therefore cannot share a rank, and no tie rule is needed.
- "Nearest expiry" is a deterministic selection rule. It is not a claim that the nearest expiry is the most liquid contract. No open-interest or liquidity data is used, because none exists in Core.
- The selected contract must still pass every price, data and risk check. If it fails, no other expiry is tried: there is no fallback.

### 4. Price generation (F4-D4)

The price-movement unit is the mean absolute close-to-close change:

```text
U = (|C1 - C0| + |C2 - C1| + ... + |CN - C(N-1)|) / N
```

- It is computed over the trailing N+1 closes of the frozen evidence. V1 fixes N = 19, using twenty closes.
- U is not average true range and not realized volatility, and it is never named either.
- Reference price `P` is the decision-session close (`latest_quote`).
- Entry zone: `[ceil(P - kE*U), floor(P + kE*U)]`, aligned inward to the price increment, for both directions.
- LONG stop and target:
  - stop `floor(entry.low - kS*U)`, rounded away from the zone;
  - target `floor(entry.high + kT*U)`, rounded toward the zone.
- SHORT is the mirror image:
  - stop `ceil(entry.high + kS*U)`;
  - target `ceil(entry.low - kT*U)`.
- `kE`, `kS`, `kT` and the price increment are explicit, validated policy parameters. Core holds no default.
- Every level is an exact multiple of the caller-supplied price increment.
- Arithmetic is exact: no binary floating point, and no rounding other than the documented alignment.

### 5. Risk validation (F4-D5)

- Reward-to-risk is measured from the worst entry inside the aligned zone, after alignment:
  - LONG: `entry.high`;
  - SHORT: `entry.low`.
- A setup is accepted only when:
  - risk and reward are both strictly positive;
  - `reward >= min_reward_to_risk * risk`, compared exactly.
- A setup whose aligned stop or target collapses onto the entry zone is rejected (`INVALID_PRICE_ORDER`).
- A setup whose zone collapses is rejected (`DEGENERATE_ENTRY_ZONE`).
- Contract life:
  - `CONTRACT_EXPIRED` when the first eligible observation is after the contract cutoff;
  - `INSUFFICIENT_CONTRACT_LIFE` when the intended validity window ends after the cutoff.
- Open recommendations: product-level exclusivity, Decision 8.
- Reward-to-risk is a geometric property of the levels. It is not a probability of success or an expected return. No transaction cost or slippage is modelled.

### 6. Output and identity (F4-D6)

- F-4 produces an immutable `FuturesRecommendationProposal`. It contains every field of a `FuturesTradeRecommendation` except identity, plus the price and risk evidence it was generated from.
- `proposal.to_recommendation(identity)` takes a caller-supplied `FuturesTradeRecommendationIdentity` and constructs the F-2 recommendation unchanged.
- Core generates no identity. Deriving identity deterministically is an F-5 Application responsibility.

### 7. Calibration and governance (F4-D7)

- Every material parameter is part of the policy, and a policy is named by one `RecommendationPolicyVersion`. The parameters are N, `kE`, `kS`, `kT`, the price increment, the minimum reward-to-risk, `short_enabled` and the approved strategy.
- Changing any parameter requires a new version. Core cannot police a registry of past versions; that is a future Application persistence responsibility.
- F-4 is calibrated only with synthetic, non-production fixtures. No production multiplier, threshold or price increment is approved by this ADR. Generated proposals are unvalidated trading-policy prototypes, and nothing claims a measured trading edge.

### 8. One open recommendation per product (P-1)

- **The rule.** Across all expiration contracts of one futures product, at most one non-terminal recommendation may exist at a time.
- **Product identity.** Products are compared by `FuturesProductReference` equality, never by contract or display text. Different products, including the same symbol on another exchange, are independent.
- **Same direction is a duplicate.** A new candidate whose product has an open recommendation in the same direction, on any expiry, is rejected `DUPLICATE_OPEN_RECOMMENDATION`.
  - Example: open NIFTY October LONG plus new NIFTY November LONG.
- **Opposite direction is a contradiction.** One whose product has an open recommendation in the opposite direction is rejected `CONTRADICTS_OPEN_RECOMMENDATION`.
  - Example: open NIFTY October SHORT plus new NIFTY November LONG.
- **Terminal recommendations block nothing.**
  - Openness is the existing F-2 lifecycle's `is_terminal` semantics: `WAITING_FOR_ENTRY` and `ACTIVE` are open.
  - A terminal lifecycle supplied to generation is validated and then ignored.
- **Scope of enforcement.** The rule is enforced across sessions and across expiries. It adds to same-session arbitration and does not replace it. Same-session contradiction still takes precedence, the nearest eligible contract is still the one checked, and there is still no fallback to another expiry.
- **Supplied state that already breaks the rule fails closed.** It is not repaired:
  - open recommendations in both directions contradict any new candidate on the product;
  - several in one direction duplicate a same-direction candidate and contradict the other.
  The outcome never depends on input order. Each such product is reported for Application reconciliation.
- **Recommendations are not positions.** An open recommendation is recommendation-lifecycle state. It is not an executed trade, a Paper Trading position, an order or a broker holding, and none of those is read.
- **Trust boundary.** Core sees only the lifecycles the caller supplies and cannot detect an omitted one. F-5 must:
  - load every non-terminal recommendation across all expiries of the product;
  - reconcile lifecycle expiry before generation;
  - handle retries and already-published proposals idempotently;
  - enforce product-level uniqueness transactionally at persistence and publication, so that concurrent requests cannot bypass it;
  - keep recommendation history insert-only.
- **This is a Futures V1 product policy for published recommendations.** It is not an exchange rule, and it does not prohibit spreads or hedges that traders may hold. Rollover of an open recommendation to the next expiry, replacement or supersession, hedging and multi-leg recommendations are deferred: under this rule a product's open recommendation must become terminal before another can be generated.

Neither this rule nor the conflict veto establishes measured profitability or any liquidity property. Both are conservative publication rules.

## Relationship to ADR-017

This ADR complements ADR-017. It does not supersede it:

- ADR-017 Decision 4 states that a future generation policy may derive SHORT from a research SELL, subject to separate review. This ADR is that policy.
- Repository layering, the package boundary, import direction, the four distinct vocabularies, the daily-candle (L-3) semantics and the Paper Trading target-exposure mapping remain exactly as ADR-017 records them.
- ADR-017's statement that Recommendations may build on Strategy is what permits the generation package to read `RecommendationAction` and `FuturesMarketObservationContext`. The trade setup and lifecycle modules still never see research vocabulary.

No Accepted ADR conflicts with this policy.

## Rationale

- **Symmetric mapping.** The research signal is exactly symmetric, so treating SELL differently from BUY would have no basis in the research. Futures shorts are native instruments, with no borrow.
- **The `short_enabled` flag** lets the product owner fall back to LONG-only without a code change, under a new policy version.
- **One strategy** makes arbitration trivial in V1 while still fixing the conservative conflict rule for later.
- **Nearest eligible expiry** is deterministic and uses only facts Core holds.
- **Close-to-close unit.** It is the only movement measure the frozen evidence supports without inventing observations.
- **Conservative rounding directions.** Alignment never shrinks risk or enlarges reward. The worst-entry reward-to-risk is therefore a lower bound over the zone.
- **Proposal type.** It keeps Core free of identity generation while guaranteeing that every proposal converts to a valid F-2 recommendation.
- **One open recommendation per product.** Expiries of one product track the same underlying, so two open recommendations on one product would duplicate or contradict each other. That is true whichever expiry each is on.

## Alternatives Considered

- **LONG-only V1.** Kept available through `short_enabled = false`, but not the default: it would discard symmetric research without evidence.
- **SELL to SHORT only with extra bearish confirmation.** Rejected: it would add research logic to generation.
- **Process every strategy independently.** Rejected: identical strategies would produce duplicate setups, and opposite views would be published side by side.
- **Average true range.** Rejected: the frozen evidence holds no high and low history.
- **Percentage-of-price levels.** Rejected: not adaptive, and undefined for non-positive quotations.
- **Truncating validity to the contract cutoff.** Rejected: a positional thesis needs its full window.
- **Falling back to the next expiry after a price or risk failure.** Rejected: the fallback would change which contract the research is applied to, after the fact.
- **Generating identity in Core.** Rejected: F-2 forbids it, and derivation belongs to F-5.
- **Open-recommendation checks per contract only.** Rejected under P-1: a LONG on one expiry could be published beside an open SHORT on another expiry of the same product.

## Consequences

### Positive Consequences

- Real, deterministic, fully testable generation behaviour exists in Core.
- Every candidate has exactly one auditable disposition: a proposal or a rejection with one reason.
- The F-2 and F-3 public behaviour and the research vocabulary are unchanged.

### Negative Consequences

- Proposals are not proven tradable on the exchange. They are aligned only to a caller-supplied increment.
- Parameters are uncalibrated. No proposal may be published as a live recommendation.
- U ignores intraday range and gaps, so it understates the daily range. A small `kS` will make many setups end `INDETERMINATE_ENTRY` under the L-3 policy.
- Duplicate and contradiction protection is only as complete as the caller's view of open recommendations across every expiry. F-5 must make that view complete and enforce uniqueness transactionally.
- While a product has an open recommendation, fresh research on it produces no proposal. Rollover and replacement policies do not yet exist.

## Scope

This ADR records a trading policy. It does not:

- change F-2 or F-3 types or behaviour;
- add persistence, publication, notification, monitoring, Application services, API routes or Web views;
- define identity derivation, session-calendar resolution or data approval;
- approve production parameters, a production price increment, live publication or any execution.

## Future Considerations

The following are outstanding before any proposal may be published or treated as performance evidence:

- **Production price increment.** It must be sourced from current, authoritative NSE contract specifications and verified by the product owner.
- **Calibration.** Multipliers, minimum reward-to-risk and validity length need historical evaluation, including the rate of `INDETERMINATE_ENTRY` outcomes.
- **F-5.**
  - Identity derivation.
  - Calendar resolution of `eligible_from`, `valid_through` and `contract_cutoff`.
  - Complete, approved and final evidence.
  - A complete, current open-recommendation view across every expiry, and transactional product-level uniqueness at publication.
  - Version-registry enforcement.
- **G-1.** Written market-data permission from the data provider.
- **Permissions.** Separate regulatory permission for recommendation publication, and separate permission for any trading execution.
- **FG-9.4.** Release-safety work.
- **Deferred.**
  - Multi-strategy arbitration, including `CONFLICTING_RESEARCH`.
  - Invalidation before entry (F-2 N7).
  - Rollover, replacement and supersession of open recommendations.
  - Hedging and multi-leg recommendations.
  - Transaction cost and slippage modelling.

## Related Documents

- ADR-017: Repository Layering and Futures Recommendation Domain Boundary
- ADR-009: Contract-Scoped Futures Economics
- Futures Trade Recommendation Design Specification v1.0 (F-2): `architecture/Futures-Trade-Recommendation-Design-Specification-v1.0.md`
- Futures Recommendation Candidate Selection Design Specification v1.0 (F-3): `architecture/Futures-Recommendation-Candidate-Selection-Design-Specification-v1.0.md`
- Futures Recommendation Generation Design Specification v1.0 (F-4): `architecture/Futures-Recommendation-Generation-Design-Specification-v1.0.md`
- Engineering Standards ES-001, ES-002, ES-003, ES-004
