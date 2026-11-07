# ADR-020: Equity Intraday V1 Recommendation Generation Architecture

## Status

Accepted

The product owner approved owner decision EI4-D6, equity-specific architecture, on 2026-11-07, which establishes this ADR. The policy decisions it relies on -- EI4-D1, EI0-D7 (revised timing structure), EI4-D2, EI0-D14, EI4-D3, EI4-D4, EI4-D7 and EI4-D8 -- were approved on the same date and are recorded in the [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md#3-approved-decisions). This ADR records the architecture those decisions require. It records no separate Architecture Review or Engineering Review sign-off.

This is a design decision. **Nothing in it is implemented** when it is recorded: the contracts it describes are planned for EI-4B to EI-4E.

## Date

2026-11-07

## Decision Makers

- Product owner: approved EI4-D6 and the related EI-4 decisions on 2026-11-07.
- Northstar Architecture Team: formal review pending.

## Context

EI-3 produces, for each strategy configuration, listing and trading date, an authoritative and immutable research timeline (EI-3C) whose first qualifying `LONG_CANDIDATE` is labelled `CANDIDATE` (EI3-D7). Each entry carries an EI-3A research decision (`eir1` fingerprint) that rests on an immutable EI-2B evidence snapshot, and each entry is chained by an `eirt1` fingerprint. A `CANDIDATE` is a research label, not a recommendation ([EI-3 Decision Register, section 4](../product/Equity-Intraday-V1-EI3-Decision-Register.md#4-limitations-and-exclusions)).

EI-4 must turn a candidate into a synthetic recommendation: an entry zone, stop, target, validity and sizing suggestion, or a recorded refusal. Several facts constrain the design:

- **Futures already has a recommendation domain**, `northstar_core.recommendations.futures` (ADR-017, ADR-018), with immutable recommendations, a generation policy, structured rejections, deterministic Application-derived identities and insert-only persistence. Its types are bound to Futures: `FuturesTradeRecommendation` names a `FuturesContract`, and its entry zone and levels use the derivatives `QuoteValue`. ADR-017 keeps that domain Futures-specific.
- **The research decision is an Application type.** `EquityIntradayResearchDecision` lives in `northstar-application`, so Core cannot import it.
- **Timing.** A research decision is made at its snapshot's `as_of` (EI3-D1), after its signal bar's finality is received, so the next bar began before the decision existed.
- **Eligibility inputs are missing.** EI-3 implemented no universe version and no series, listing-status, liquidity, price-floor, surveillance or corporate-action eligibility.
- **Real values are pending.** No model capital, risk parameter, cost rate, price increment or production cutoff is approved (EI0-D7, EI0-D10 to EI0-D13).
- **Lifecycle does not exist yet.** Which recommendations are open is EI-5's concern.

## Decision

### 1. Equity-local recommendation domain

- Equity recommendation models and generation contracts are equity-local. The planned Core package is `northstar_core.recommendations.equity`, a sibling of `northstar_core.recommendations.futures`, as proposed in the [product specification, section 8.1](../product/Equity-Intraday-V1-Product-Specification.md#81-minimum-recommendation-contract).
- Equity price levels use Foundation types in the listing's currency, INR. Derivative quote types are not used.
- Core references a research decision through an equity-local reference value -- its identities and fingerprints -- because Core cannot import the Application research decision. Application maps one to the other.
- **No common recommendation kernel** is created now. Types that look product-neutral but live in the Futures package, such as `RecommendationPolicyVersion`, are not imported; equity defines its own. Any later shared kernel needs its own ADR.

### 2. Futures patterns, not Futures types

These Futures patterns are reused as patterns. No Futures recommendation type is imported, subclassed or changed:

- deterministic identity derived by the Application, never generated in Core;
- exact arithmetic, with no binary floating point and no rounding other than documented increment alignment;
- immutable recommendation definitions;
- named, versioned generation policies, where any parameter change is a new version;
- structured, deterministic rejection reasons;
- append-only auditability of every generation attempt.

### 3. Identity and immutability

- A candidate yields zero or one recommendation per generation-policy version (EI4-D2). Only the first authoritative `CANDIDATE` of the verified EI-3C timeline is considered. `REPEAT_QUALIFYING`, `NO_SIGNAL` and `REJECTED` entries never generate one.
- The Application derives each recommendation's identity deterministically from the candidate's natural key -- strategy and configuration, listing, trading date -- the generation-policy version and a reference to the authoritative candidate entry. It excludes clock readings, random values and storage row identifiers, so a retry or restart finds what was recorded.
- A digest identifies; it does not enforce. Business uniqueness is enforced by the immutable store (EI-4D).
- Recommendation definitions and eligibility decisions are immutable once recorded. Later status, if any, is appended, never rewritten. Earlier candidates and research decisions are never rewritten by later corrections.
- Every recommendation references what it rests on:
  - its research decision, candidate entry and evidence snapshot fingerprints;
  - the strategy configuration;
  - the versioned generation, risk, cost, price-increment, timing and eligibility references;
  - its instants (section 5).

### 4. Synthetic policy versioning

- Model capital, the risk policy, the price increment, the cost schedule, entry-zone construction, timing cutoffs, the target multiple and related thresholds are explicit, versioned inputs (EI4-D4).
- Synthetic inputs carry a `synthetic-` version prefix, the convention EI3-D5 established. They are not exchange facts, broker tariffs or approved trading parameters.
- Core holds no default for any of them. A required input that is unknown fails closed.

### 5. Timing and look-ahead safeguards

- Three instants stay distinct (EI4-D1):
  - `decision_as_of`, the EI-3 research decision instant;
  - `issued_at`, the actual issuance of an operational recommendation, when one applies;
  - `eligible_from`, the earliest entry-eligibility instant.
- Historical synthetic runs label modeled issuance separately from actual issuance. An operational recommendation is never backdated.
- Entry evidence comes only from bars that open at or after the information-availability and issuance instant. A bar that opened before the decision was available never evidences a full-bar entry.
- A timing policy states the research signal-bar end, decision availability, the latest permissible entry-bar opening, any minimum remaining session time, the mandatory modeled exit boundary, its treatment of shortened or special sessions, and post-session research (EI0-D7). Its values are synthetic and versioned. No production cutoff is approved.
- EI3-D1's decision instant and EI3-D6's research window are unchanged. EI-4 never reads the EI-3 evaluator's inputs differently.

### 6. Price levels and risk arithmetic

- **Entry:** a deterministic modeled zone anchored to the original signal close, built from versioned parameters (EI0-D14). It is not a fill and not a guarantee of an executable price.
- **Stop:** for LONG, the opening-range low is the initial reference.
- **Target:** a fixed multiple of worst-entry risk (EI4-D3).
- Every level aligns to an explicitly supplied price increment, exactly. An unknown increment fails closed.
- Reward-to-risk is measured at the worst permissible entry.
- Invalid price ordering, non-positive risk and unrepresentable levels are rejected with structured reasons.

### 7. Eligibility references and open state

- Listing and universe eligibility come through an Application-owned reference port, or an equivalent boundary, supplying versioned synthetic eligibility (EI4-D7). Missing or unknown eligibility is an explicit refusal. Synthetic eligibility is never production authority, and no real liquidity, series, surveillance or corporate-action fact is invented.
- Until EI-5 provides authoritative lifecycle state, every earlier same-session recommendation counts as open for exposure and exclusivity (EI4-D8). Missing or unverifiable prior state fails closed, and an empty history is never assumed.
- An eligibility rule whose control does not exist yet fails closed, or is reported as not assessed. It is never reported as satisfied.

### 8. Dependency rules

- Repository direction is unchanged (ADR-017): Core <- Application <- Infrastructure. Application depends on Core; Infrastructure implements Application ports and depends on both. Application never imports Infrastructure.
- The equity recommendation package may consume Foundation, Domain and Strategy types. It must not import the Futures recommendation package, derivative quote types, Paper Trading or Broker Execution. Strategy must not import Recommendations.
- Futures contracts, the EI-3A evaluator, the EI-3B scheduler and the EI-3C persistence contracts are unchanged.

## Relationship to Earlier ADRs

- **ADR-017.** This ADR applies ADR-017's layering and boundary rules to equities. ADR-017 itself is unchanged.
- **ADR-018.** ADR-018 is the Futures generation policy. Its patterns -- exact alignment, worst-entry reward-to-risk, structured rejections, policy versions -- are adopted here as patterns. Its Futures-specific rules (contract selection, close-to-close movement unit, SHORT) do not apply to equities, and ADR-018 is unchanged.
- **ADR-019.** `Symbol` with `&` is what allows equity listings such as `M&M` to be referenced. It is unchanged.

No existing ADR is superseded.

## Rationale

- **Equity-local types** avoid hidden Futures assumptions -- contract cutoffs, expiry, SHORT mapping, derivative quotations -- leaking into cash equities, as the [product specification, section 15](../product/Equity-Intraday-V1-Product-Specification.md#15-material-risks) warns.
- **No shared kernel yet.** A kernel extracted from one product would encode that product's assumptions. Two working domains are a better basis for one.
- **Identity from the authoritative candidate** makes generation idempotent across retries and restarts and ties each recommendation to the exact evidence EI-3C made authoritative.
- **Opening-range low and a fixed risk multiple** use values EI-3A already records, so no new indicator is introduced.
- **Synthetic versioning** lets every rule be built and tested now without implying that any real value is approved.
- **Conservative open state** prefers refusing a valid setup to over-exposing an unknown one.

## Alternatives Considered

- **Reuse or subclass `FuturesTradeRecommendation`.** Rejected: it is bound to `FuturesContract` and derivative quotations, and ADR-017 keeps it Futures-specific.
- **Extract a common recommendation kernel now.** Rejected for this work (EI4-D6); revisit with its own ADR once two domains exist.
- **Generate from every qualifying entry.** Rejected (EI4-D2): candidate counts would depend on snapshot frequency, and later entries would let generation retry until a setup passes.
- **Entry at the next bar, or `eligible_from` at the completion of the next bar.** Rejected (EI4-D1): the next bar began before the decision was available, which is look-ahead.
- **A bar-end 14:25 / 14:30 / 15:00 schedule.** Not approved as an operational schedule (EI0-D7): under EI3-D1 it leaves no permissible entry.
- **A trigger entry, or a stop at the VWAP approximation or signal-bar low.** Not chosen for the initial synthetic model (EI0-D14, EI4-D3). They remain possible later policy versions.
- **Assume prior recommendations closed when a level might have been touched.** Rejected (EI4-D8): that is an outcome claim only EI-5 may make.

## Consequences

### Positive Consequences

- EI-4 can be built and tested end to end on synthetic data without approving any real value.
- Every candidate has one auditable disposition per policy version: a recommendation or a refusal with explicit reasons.
- Look-ahead through the post-decision bar is excluded by design.
- Futures, EI-3A, EI-3B and EI-3C are unaffected.

### Negative Consequences

- Equity and Futures recommendation code will contain parallel, similar types until a shared kernel is justified.
- Whenever finality takes any time, the earliest entry evidence is the second bar after the signal bar, which reduces modeled entry opportunities. That reduction is the honest cost of finality latency.
- The conservative open-state rule may refuse setups that a complete lifecycle would allow.
- Synthetic results describe the mechanism only, not markets.

## Scope

This ADR records an architecture. It does not:

- implement any contract, use case, store or test;
- approve a production cutoff, model capital, risk parameter, cost rate, price increment, liquidity threshold or loss limit;
- define the recommendation lifecycle (EI-5), lifecycle persistence (EI-6), interfaces (EI-7), evaluation (EI-8) or real data (EI-9);
- authorize publication, paper orders, fills, broker execution or production use.

## Future Considerations

- **EI-4B to EI-4E**, in that order: Core contracts and arithmetic; Application eligibility and generation; Infrastructure immutable persistence; synthetic end-to-end acceptance.
- **Deferred decisions:** EI4-D5 (pre-entry invalidation) and EI4-D9 (persistence sequencing), and EI0-D8, EI0-D9, EI0-D10 to EI0-D13, EI0-D15 to EI0-D21 and EI3-D10, with their recorded statuses.
- **Before real data:** every restriction in the [EI-4 Decision Register, section 7](../product/Equity-Intraday-V1-EI4-Decision-Register.md#7-production-readiness-restrictions).
- **A shared recommendation kernel**, only by a later ADR.

## Related Documents

- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md)
- [EI-3 Decision Register](../product/Equity-Intraday-V1-EI3-Decision-Register.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md)
- [EI-3C Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI3C-Synthetic-Acceptance-Record.md)
- [ADR-017: Repository Layering and Futures Recommendation Domain Boundary](ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md)
- [ADR-018: Futures V1 Research-to-Recommendation Generation Policy](ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md)
- [ADR-019: Symbol Ampersand Extension](ADR-019-Symbol-Ampersand-Extension.md)
- Engineering Standards ES-001, ES-002, ES-003, ES-004
