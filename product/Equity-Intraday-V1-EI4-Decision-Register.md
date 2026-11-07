# Equity Intraday V1 EI-4 Decision Register

## Status

**Owner acceptance record, 2026-11-07. Scope: the initial synthetic-only EI-4 recommendation-generation design.**

- The Northstar product owner explicitly approved the nine decisions in [section 3](#3-approved-decisions) on **2026-11-07**: EI4-D1, EI0-D7 (revised timing structure), EI4-D2, EI0-D14, EI4-D3, EI4-D4, EI4-D6, EI4-D7 and EI4-D8.
- EI0-D7, EI0-D14 and EI4-D3 are approved **for the initial synthetic model or synthetic policy structure only**, as each entry states.
- No other decision is approved by this record. Every decision in [section 5](#5-decisions-still-pending-or-deferred) keeps its status.
- The architecture is recorded in [ADR-020](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md).
- Nothing here authorizes production use, real market data, a real data provider, a recommendation shown as actionable, an order, a paper fill, broker execution or any trading.
- This record has not been through Architecture Review or Engineering Review.
- Nothing in EI-4 is implemented by this record.

## 1. Purpose

This register records the owner decisions that govern EI-4, the step from an immutable synthetic research candidate to a synthetic recommendation, before any EI-4 code is written.

- It is the authoritative record for decisions with the `EI4-` prefix.
- It also records the owner's revised, synthetic-scoped approval of two decisions from the EI-0 owner decision register, EI0-D7 and EI0-D14. Their entries in the [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md), section 14, point here.
- It changes no decision recorded in the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md), the [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md) or any ADR.
- Implementation history is not approval.

## 2. Scope

| Item | Scope of this record |
| --- | --- |
| Product | Northstar Equity Intraday V1: NSE cash equities, INR |
| Milestone | EI-4, recommendation generation ([section 4](#4-initial-synthetic-implementation-boundaries)) |
| Data | Synthetic only: synthetic evidence, synthetic eligibility references and synthetic calibration |
| Input | The authoritative EI-3C research timeline and its immutable EI-2B evidence |
| Outputs | Synthetic recommendation definitions and eligibility decisions with explicit reasons |
| Not in scope | Recommendation lifecycle and monitoring (EI-5), lifecycle persistence (EI-6), API and Web (EI-7), evaluation and performance (EI-8), real data (EI-9), forward observation (EI-10), paper orders, fills, broker execution and production use |

Every approval below is limited to this scope unless its own text says otherwise.

## 3. Approved decisions

Each decision below is **APPROVED BY OWNER, 2026-11-07**, within the scope its entry states.

### EI4-D1 — Decision, issuance and entry timing

**Approved rule.** Three instants are distinct:

- `decision_as_of`: the original EI-3 research decision instant, the evidence snapshot's `as_of` (EI3-D1);
- `issued_at`: the actual issuance instant of a real operational recommendation, when one applies;
- `eligible_from`: the earliest modeled or operational entry-eligibility instant.

Historical synthetic simulations distinguish modeled issuance from actual issuance explicitly. An operational recommendation is never backdated so that it appears actionable before it was issued.

Entry evidence comes only from bars that open at or after the applicable information-availability and issuance instant. A candle that opened before the decision was available never establishes a full-bar hypothetical entry. No look-ahead bias is permitted.

**Rationale.** A decision is available only once its evidence is final (EI3-D1, EI3-D2), which is after its signal bar ends. The bar after the signal bar therefore began before the decision existed, and treating its range as entry evidence would use prices the decision could not have known.

### EI0-D7 — Revised timing structure

**APPROVED BY OWNER FOR SYNTHETIC POLICY STRUCTURE ONLY.**

The earlier bar-end interpretation of the proposed schedule -- last signal 14:25, last entry 14:30, planned exit 15:00 IST -- is **not** approved as an operational schedule.

**Approved structure.** A timing policy distinguishes:

- the research signal-bar end;
- the actual decision availability;
- the latest permissible entry-bar opening;
- the minimum remaining session time, where it applies;
- the mandatory modeled exit boundary;
- shortened or special sessions;
- post-session historical research.

Testing uses versioned synthetic cutoff values. **No fixed production cutoff time is approved.** EI3-D6's research window and EI3-D1's decision instant (the snapshot's `as_of`) are preserved, and the EI-3 research evaluator is unchanged.

**Rationale.** Under EI3-D1, a decision on a signal bar ending at 14:25 is made after 14:25, so the first bar that opens after it ends after a 14:30 bar-end entry cutoff. The bar-end schedule could never permit an entry once finality takes any time at all.

### EI4-D2 — Candidate eligibility

**Approved rule.**

- Only the first authoritative EI-3B `CANDIDATE` in the verified EI-3C timeline can be considered for recommendation generation under a given policy version.
- `REPEAT_QUALIFYING`, `NO_SIGNAL` and `REJECTED` entries cannot independently generate recommendations.
- A candidate yields zero or one recommendation per applicable generation-policy version.
- Eligibility failures produce deterministic, distinguishable reasons.
- No `CANDIDATE` automatically becomes an actionable recommendation.
- Existing candidate history stays immutable, and later corrections never silently rewrite an earlier candidate decision.

### EI0-D14 — Entry representation

**APPROVED BY OWNER FOR THE INITIAL SYNTHETIC MODEL.**

**Approved rule.**

- The entry is a deterministic modeled entry-price zone anchored to the original signal close.
- The zone is not a historical fill and not a guarantee of a broker-executable price.
- Its construction uses explicit, versioned parameters.
- Price levels align to an explicitly supplied valid price increment. An unknown increment fails closed.
- No production-approved zone width exists.

### EI4-D3 — Stop and target methods

**APPROVED BY OWNER FOR THE INITIAL SYNTHETIC MODEL.**

**Approved rule.**

- The initial LONG stop reference is the opening-range low.
- The target is a fixed multiple of worst-entry risk.
- Arithmetic is exact, with explicit price-increment alignment.
- Reward-to-risk is evaluated at the worst permissible entry.
- Invalid price ordering, non-positive risk and unrepresentable price levels are rejected.
- The multiple and the risk thresholds are synthetic calibration parameters. No production numeric value is approved.
- No order or fill is implied.

### EI4-D4 — Synthetic calibration

**Approved rule.** Versioned, synthetic-only inputs are allowed for:

- model capital;
- the risk policy;
- the price increment;
- the cost schedule;
- entry-zone construction;
- timing cutoffs;
- the target multiple and related thresholds.

Every synthetic policy or schedule is explicitly labelled and versioned under the established synthetic naming convention, a `synthetic-` version prefix (as in EI3-D5). Synthetic values are not official exchange facts, real broker tariffs or production-approved trading parameters. An unknown required input fails closed.

Allowing synthetic calibration approves **none** of EI0-D10 to EI0-D13 or any other decision about real values.

### EI4-D6 — Equity-specific architecture

**Approved rule.** ADR-020 is established for the EI-4 recommendation architecture: equity-local recommendation models and generation contracts, no import of Futures-specific recommendation types, reuse of established Futures patterns only, no common recommendation kernel in this work, the dependency direction Core <- Application <- Infrastructure, and no change to Futures contracts. The detail is in [ADR-020](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md).

### EI4-D7 — Synthetic eligibility references

**Approved rule.**

- An Application-owned reference port, or an equivalent clean architectural boundary, supplies versioned synthetic eligibility information.
- It supports the listing and universe eligibility decisions the initial synthetic model requires, without claiming that real exchange applicability is verified.
- Unknown or missing required eligibility returns an explicit refusal.
- Synthetic listing eligibility is never production authority.
- No real liquidity, series, surveillance or corporate-action fact is invented.

**Context.** The EI-3 synthetic research foundation does not supply recommendation eligibility. EI-3A to EI-3C implemented no universe version and no series, listing-status, liquidity, price-floor, surveillance or corporate-action eligibility.

### EI4-D8 — Conservative open-state assumption

**Approved rule.**

- Until EI-5 provides authoritative recommendation lifecycle state, every previously issued same-session recommendation is treated as open for exposure and exclusivity evaluation.
- A prior recommendation is never assumed closed because its hypothetical target or stop might have been touched.
- Missing, incomplete or unverifiable required prior-recommendation state fails closed. An empty open-recommendation history is never fabricated.
- The rule applies only to synthetic recommendation eligibility. It establishes no broker position or actual holding.

## 4. Initial synthetic implementation boundaries

The intended EI-4 sequence, recorded at the owner's direction:

| Slice | Scope |
| --- | --- |
| EI-4A | Governance: this register and [ADR-020](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md) |
| EI-4B | Equity-specific Core recommendation contracts, levels and exact risk arithmetic |
| EI-4C | Application eligibility and recommendation generation |
| EI-4D | Infrastructure immutable recommendation and eligibility-audit persistence |
| EI-4E | Synthetic end-to-end acceptance |

- The broader sequence in the [product specification, section 12](Equity-Intraday-V1-Product-Specification.md#12-milestone-dependencies) is unchanged: EI-5 recommendation lifecycle, EI-6 lifecycle persistence, EI-7 API and Web, EI-8 evaluation and performance, EI-9 real market data, EI-10 forward observation where applicable.
- Lifecycle, paper trading and broker execution are not moved into EI-4.
- EI4-D9, recommendation persistence sequencing, stays deferred ([section 5](#5-decisions-still-pending-or-deferred)). How EI-4D's persistence relates to EI-6 is to be settled by it, not assumed.
- **Unimplemented controls are never reported as satisfied.** When an EI-4 eligibility rule needs a control that does not exist -- for example the daily modeled loss limit (EI0-D19), which needs lifecycle outcomes -- the rule fails closed, or stays outside a clearly delimited early modelling slice and is reported as not assessed.

## 5. Decisions still pending or deferred

These are **not** approved by this record. Each keeps its existing status.

| ID | Question | Status |
| --- | --- | --- |
| EI4-D5 | Pre-entry invalidation: whether a stop touched before entry invalidates the setup | PROPOSED, deferred |
| EI4-D9 | Recommendation persistence sequencing relative to EI-6 | PROPOSED, deferred |
| EI0-D8 | The initial strategy and its research control, beyond EI3-D4's synthetic approval | PENDING |
| EI0-D9 | Intrabar ambiguity | PENDING |
| EI0-D10 to EI0-D13 | Model capital, risk parameters, cost schedule and price increment: actual values and market-reference approvals | PENDING |
| EI0-D15 | Real liquidity and price thresholds | PENDING |
| EI0-D16 | The market-data-rights gate | PROPOSED |
| EI0-D17 | Corporate-action handling | PENDING |
| EI0-D18 | Exchange surveillance exclusion | PENDING |
| EI0-D19 | Daily modeled loss limit | PENDING |
| EI0-D20 | Performance robustness minimums | PENDING |
| EI0-D21 | Candle-revision materiality | PENDING |
| EI3-D10 | Trailing unfinalized candle policy | PENDING |

- EI4-D5 and EI4-D9 are first recorded here, as proposals from the EI-4A design review. EI4-D1 to EI4-D4 and EI4-D6 to EI4-D8 are approved above; there is no EI4-D10 or later.
- EI0-D1, EI0-D2 and EI0-D5 stay PROPOSED, as recorded in the [product specification, section 14](Equity-Intraday-V1-Product-Specification.md#14-owner-decision-register).
- CE-D8, CE-D9, CE-D10 and CE-D15 stay PENDING in the [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md), section 18.2.

## 6. Earlier approvals unchanged

- Every decision approved in the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md) keeps its status and date: EI3-D1 to EI3-D5, EI3-D8, EI0-D3 and EI0-D6 (2026-11-05); EI3-D6, EI3-D7 and EI3-D9 (2026-11-06); EI3-D11, EI3-D12 and EI3-D13 (2026-11-07).
- The [EI-3C Synthetic Acceptance Record](Equity-Intraday-V1-EI3C-Synthetic-Acceptance-Record.md) is unchanged.
- EI0-D4 (2026-11-01, [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md)) and the NSE session-evidence decisions and facts approved on 2026-11-04 are unchanged.
- The EI-3A evaluator, the EI-3B scheduling rules and the EI-3C persistence behaviour are unchanged.

## 7. Production-readiness restrictions

Nothing in this record moves Equity Intraday V1 toward production use. Every restriction in [section 8 of the EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md#8-production-readiness-restrictions) still applies, and in particular there is:

- no authorized real-data provider and no completed data-rights approval (EI0-D16);
- no approved production finality source, and EI3-D10 is pending;
- no production listing and applicability authority;
- no corporate-action or surveillance source (EI0-D17, EI0-D18);
- no official price increment or cost schedule (EI0-D12, EI0-D13);
- no approved model capital, risk parameters, liquidity thresholds or loss limit (EI0-D10, EI0-D11, EI0-D15, EI0-D19);
- no approved production cutoff time (EI0-D7);
- no execution, paper-trading, broker or deployment authorization.

A synthetic recommendation is a research artifact. It is not a recommendation to trade, an order or a position, and no result from it is evidence of profitability.

## 8. Related documents

- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md)
- [Equity Intraday V1 EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md)
- [Equity Intraday V1 EI-3C Synthetic Acceptance Record](Equity-Intraday-V1-EI3C-Synthetic-Acceptance-Record.md)
- [ADR-017: Repository Layering and Futures Recommendation Domain Boundary](../adr/ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md)
- [ADR-018: Futures V1 Research-to-Recommendation Generation Policy](../adr/ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md)
