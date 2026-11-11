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

> **Note, 2026-11-07 (EI4-D9 owner approval).** The product owner later approved EI4-D9 on 2026-11-07, as ten separately identified subdecisions, EI4-D9-A to EI4-D9-J, recorded in [section 9](#9-ei4-d9-owner-approval-record-2026-11-07). The persistence architecture is recorded in [ADR-021](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md). The bullets above describe the record before that approval and are not rewritten. EI4-D5 stays deferred.

> **Note, 2026-11-08 (EI-4E, engineering acceptance).** EI-4E synthetic end-to-end acceptance is recorded with the engineering verdict PASS WITH DOCUMENTED LIMITATIONS in the [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md). That record is engineering evidence, not an owner approval: no decision status changes, the milestone is not closed by it, and nothing authorizes real data, issuance, execution or production use. Two findings from it, AA-1 (policy-version label governance) and AA-2 (verified-writer boundary and recovery), are OWNER DECISION PENDING.

> **Note, 2026-11-08 (EI4-D10 and EI4-D11 owner approval).** The product owner later approved both EI-4E findings on 2026-11-08: AA-1 as **EI4-D10** (Immutable Policy-Version Identity) and AA-2 as **EI4-D11** (Mandatory Verified Recording Boundary and Invalid-Record Recovery Governance), recorded in [section 10](#10-ei4-d10-and-ei4-d11-owner-approval-record-2026-11-08). The note above records their status before that approval and is not rewritten. Neither decision is implemented, and no invalid-record recovery mechanism is approved.

> **Note, 2026-11-08 (EI4-D10 enforcement design owner approval).** The product owner later approved, on 2026-11-08, all nine engineering design decisions of the EI4-D10 enforcement design, EI4-D10-DQ1 to EI4-D10-DQ9, as recommended. They are recorded in [section 11](#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08), and the enforcement architecture in [ADR-022](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md). EI4-D10 enforcement is not yet implemented, and its acceptance is not yet established. EI4-D11 is unchanged.

> **Note, 2026-11-10 (EI4-D10, engineering acceptance).** EI4-D10 enforcement is implemented (slices S1 to S4, merged) and its engineering acceptance is recorded with the verdict PASS WITH DOCUMENTED LIMITATIONS in the [EI4-D10 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md), summarized in [section 12](#12-ei4-d10-engineering-acceptance-record-2026-11-10). That record is engineering evidence, not an owner approval: no decision status changes. CI is not independently verified. EI4-D11's operational boundary is still not implemented, and no invalid-record recovery is approved. The note above records the status before implementation and is not rewritten.

> **Note, 2026-11-10 (EI4-D11 enforcement design owner approval).** The product owner approved, on 2026-11-10, six engineering decisions of the EI4-D11 enforcement design: EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8 and DQ10, with two mandatory clarifications. They are recorded in [section 13](#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10), and the architecture in [ADR-023](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md). EI4-D11 enforcement is **not yet implemented**, its engineering acceptance is **not yet established**, and no invalid-record recovery is approved.

> **Note, 2026-11-11 (EI4-D11, engineering acceptance).** EI4-D11 enforcement is implemented (slices W1 to W4-I, merged) and its engineering acceptance is recorded with the verdict PASS WITH DOCUMENTED LIMITATIONS (synthetic engineering acceptance) in the [EI4-D11 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Engineering-Acceptance-Record.md), summarized in [section 14](#14-ei4-d11-engineering-acceptance-record-2026-11-11). That record is engineering evidence, not an owner approval: no decision status changes, and invalid-record recovery is still not approved.

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

> **Note, 2026-11-07 (EI4-D9 owner approval).** EI4-D9 now settles the EI-4D row above and its relation to EI-6 ([section 9](#9-ei4-d9-owner-approval-record-2026-11-07)). EI-4D persists immutable **generation records** -- dispositions, synthetic proposals and logical generation attempts -- not issued recommendations or lifecycle records, which stay with EI-5 and EI-6 (EI4-D9-J). An EI-4C compatibility slice in Application now precedes EI-4D, and EI-4D is delivered as an Application slice (EI-4D-A) followed by an Infrastructure slice (EI-4D-B) ([section 9.4](#94-approved-implementation-sequence)). The table above is not rewritten.

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

> **Note, 2026-11-07 (EI4-D9 owner approval).** The EI4-D9 row above records its status before approval and is kept as historical evidence. It is **superseded**: EI4-D9 is now **APPROVED BY OWNER, 2026-11-07**, as subdecisions EI4-D9-A to EI4-D9-J in [section 9](#9-ei4-d9-owner-approval-record-2026-11-07). Every other row of the table keeps its status; EI4-D5 stays PROPOSED and deferred. There is still no EI4-D10 or later.

> **Note, 2026-11-08 (EI4-D10 and EI4-D11 owner approval).** The statements above that there is no EI4-D10 or later describe the register before 2026-11-08 and are kept as historical evidence. EI4-D10 and EI4-D11 now exist: they record the owner's approval of EI-4E findings AA-1 and AA-2, in [section 10](#10-ei4-d10-and-ei4-d11-owner-approval-record-2026-11-08). Neither was ever a row of the table above. Every row of the table keeps its status.
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
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md) (added 2026-11-07, [section 9](#9-ei4-d9-owner-approval-record-2026-11-07))
- [Equity Intraday V1 EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md) (added 2026-11-08; engineering evidence, not an approval)
- [EI4-D10 Policy-Version Identity Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md) (added 2026-11-08, [section 11](#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08))
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md) (added 2026-11-08, [section 11](#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08))
- [Equity Intraday V1 EI4-D10 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md) (added 2026-11-10, [section 12](#12-ei4-d10-engineering-acceptance-record-2026-11-10); engineering evidence, not an approval)
- [EI4-D11 Mandatory Operational Generation-Write Boundary Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md) (added 2026-11-10, [section 13](#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10))
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md) (added 2026-11-10, [section 13](#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10))

## 9. EI4-D9 owner approval record (2026-11-07)

**Decision: EI4-D9 -- Recommendation Generation Persistence Identity, Sequencing and Historical Verification.**
**Scope: synthetic-only, LONG-only NSE equity intraday recommendation generation.**

This section records an approval given on **2026-11-07**. It does not rewrite sections 1 to 8, which describe the record before it; where they say EI4-D9 is proposed or deferred, the dated notes there point here. The architecture is recorded in [ADR-021](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md). Nothing in this section is implemented by this record.

### 9.1 Approved subdecisions

The product owner explicitly approved the ten subdecisions below. Each is **APPROVED BY OWNER, 2026-11-07**, within the scope above, and each applies exactly as worded.

#### EI4-D9-A -- Durable generation records

**APPROVED BY OWNER, 2026-11-07.**

- EI-4D persists one immutable **generation disposition** per authorized candidate and policy identity (EI4-D9-B).
- A disposition has exactly one kind: `PROPOSED`, `NOT_ELIGIBLE` or `UNRESOLVED`.
- A `PROPOSED` disposition may contain a synthetic recommendation proposal. A `NOT_ELIGIBLE` or `UNRESOLVED` disposition never contains a successful proposal.
- A separate, append-only log records distinct logical generation attempts.
- Integrity failures, dependency outages and attempts that found no candidate may be represented in the attempt log. They never acquire a disposition -- successful or unresolved -- under a candidate and policy key that does not exist.
- These dispositions are generation records. They are not recommendation issuance and not lifecycle states.

#### EI4-D9-B -- Composite synthetic policy-set identity

**APPROVED BY OWNER, 2026-11-07.** This is option B-2 of the EI4-D9 design review.

- A composite, versioned **synthetic policy-set identity** covers the generation policy, the risk policy with its model-capital configuration, the cost schedule, the price-increment reference and the timing policy.
- The stable recommendation-generation key binds that policy-set identity, the authoritative first `CANDIDATE`, and its strategy, research configuration, listing and trading date.
- This explicitly **refines EI4-D2**: the "applicable generation-policy version" under which a candidate yields zero or one recommendation is the synthetic policy-set identity.
- A materially different approved policy set may have a different identity.
- Different listing-eligibility evidence or issued-history evidence under the same policy-set identity does not authorize replacing a disposition or creating another. Such changes are content, subject to the conflict rules of EI4-D9-D.
- A future Application compatibility change introduces a clearly versioned identity scheme. The existing EI-4C proposal-key namespace, which binds only the generation-policy version, is not silently reinterpreted.
- A new synthetic policy-set identity never authorizes several actually issued recommendations or any broker order.

#### EI4-D9-C -- Canonical content and modeled issuance

**APPROVED BY OWNER, 2026-11-07.**

- Assessment and proposal documents are persisted as format-versioned, canonical, structured documents.
- A deterministic SHA-256 **content fingerprint** is computed from the canonical structured data.
- Free-text diagnostics, exception prose and human-readable descriptions are excluded from the identity-bearing fingerprint. They may be preserved separately, but never affect deterministic identity or evaluation reproducibility.
- The content binding captures every relevant approved synthetic input, source reference, piece of timing evidence and policy or reference version.
- For synthetic historical modeling, **modeled issuance is derived deterministically** from an explicitly versioned synthetic latency policy and the verified source decision availability. It is not an arbitrary caller-supplied timestamp.
- Research `decision_as_of`, modeled issuance, actual operational issuance and the earliest eligible entry stay distinct (EI4-D1). Actual operational issuance stays outside EI-4D.
- No backdating and no look-ahead are permitted.
- No numeric latency and no production cutoff are approved by this decision.

#### EI4-D9-D -- Duplicates and conflicts

**APPROVED BY OWNER, 2026-11-07.** For one generation identity:

- identical verified canonical content is idempotent;
- different canonical content is an explicit conflict. It never overwrites or silently replaces the first committed record;
- the conflicting logical attempt is recorded when storage is available;
- a new disposition is permitted only under a genuinely distinct approved policy identity.

There is no last-write-wins, and commit order is never treated as proof that evidence is objectively correct. The rule applies equally to `PROPOSED`, `NOT_ELIGIBLE` and `UNRESOLVED` dispositions: an `UNRESOLVED` disposition never silently becomes `PROPOSED` under the same identity.

#### EI4-D9-E -- Atomic transactions

**APPROVED BY OWNER, 2026-11-07.**

- The disposition, its proposal document where applicable and the corresponding logical attempt are committed atomically, in one SQLite transaction.
- A failure during a multi-record write never leaves a partially authoritative generation result.
- The transaction supports idempotent duplicate reconciliation and explicit conflicts.
- A write is never reported as successful without committed evidence.

#### EI4-D9-F -- Concurrent writers

**APPROVED BY OWNER, 2026-11-07.**

- SQLite transaction serialization and database uniqueness constraints govern concurrent writers, never in-memory locks alone.
- The first successfully committed disposition establishes the authoritative record for its key.
- A competing write of equal content is idempotent; a competing write of different content is refused as a conflict.
- The guarantees are scoped to a single SQLite database authority. No multi-host or distributed exactly-once guarantee is approved.

#### EI4-D9-G -- Commit-unconfirmed recovery

**APPROVED BY OWNER, 2026-11-07.** An error during COMMIT is an uncertain outcome until a verified reload of the generation identity resolves it:

- the same verified fingerprint is present: committed, or an equivalent was committed;
- a different verified fingerprint is present: an explicit conflict;
- the record is absent after a successful authoritative reload: no matching commit is established, and a controlled, bounded retry may proceed;
- the reload is unavailable or inconsistent: the commit outcome stays unresolved.

A COMMIT exception is never assumed to mean rollback, and no exactly-once issuance is claimed.

#### EI4-D9-H -- Point-in-time issued history

**APPROVED BY OWNER, 2026-11-07.**

- EI-4D stores generation dispositions and attempts, and never writes them into the authoritative issued-recommendation history.
- A stored `PROPOSED` disposition is not an issued recommendation, and neither is a stored eligibility audit.
- An independent **issued-recommendation authority** belongs to later issuance and lifecycle work. It must eventually provide verifiable point-in-time history, with adequate recording provenance and a recording watermark.
- A recommendation issued later never retroactively changes an earlier modeled assessment.
- Until that authority exists:
  - EI-4D never fabricates issuance history;
  - missing history is unknown, never empty;
  - explicitly labelled synthetic test fixtures may establish a complete history for testing, and they are not production authority.
- EI4-D8's conservative open-state treatment is preserved.

#### EI4-D9-I -- Restart and historical verification

**APPROVED BY OWNER, 2026-11-07.** The complete evidence needed for historical verification is persisted. On reload, the following are verified:

1. canonical stored-document integrity;
2. content fingerprints;
3. source referential integrity;
4. the authoritative EI-3C `CANDIDATE` history;
5. the original EI-2B snapshot and manifest;
6. the required synthetic policy and reference documents;
7. the exact venue-day evidence used for timing;
8. the original point-in-time issued-history evidence, with its completeness and recording watermark;
9. compatibility with the historical evaluation format;
10. exact reproduction of the original EI-4C eligibility assessment and EI-4B Core result.

Missing, corrupted, inconsistent or unsupported historical evidence fails closed. A record is never relinked to a newer corrected snapshot, and historical decisions are never silently migrated or reinterpreted (as in EI3-D13). SQLite may enforce structural constraints and foreign keys; cryptographic fingerprint verification stays an explicit Application responsibility unless a verified implementation shows otherwise.

#### EI4-D9-J -- EI-4D versus EI-5 and EI-6 responsibilities

**APPROVED BY OWNER, 2026-11-07.**

- **EI-4D** owns immutable generation records: eligibility and generation dispositions, generated synthetic proposal definitions, and auditable logical generation attempts.
- **EI-5** owns recommendation issuance and lifecycle policy.
- **EI-6** owns durable issuance and lifecycle records.
- A later issued recommendation refers back to the exact immutable EI-4D proposal identity and content fingerprint.
- A proposal stored by EI-4D is not automatically in the `RECORDED` lifecycle state.
- The product specification's EI-6 persistence wording is clarified by dated notes ([section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states) and [section 12.1](Equity-Intraday-V1-Product-Specification.md#121-refined-sequence)); its earlier text is not deleted.
- EI-4D does not implement EI-5 or EI-6.

### 9.2 Approved clarifications

Approved by the owner on 2026-11-07 with EI4-D9:

- **A. Unresolved dispositions are immutable.** A committed `UNRESOLVED` disposition can never be upgraded to `PROPOSED` under the same generation identity. A different disposition needs a separate approved policy identity.
- **B. Logical attempts are deduplicated.** The attempt log records distinct, deterministic logical attempts. Repeated identical invocations may deduplicate; there is no promise of a separate persistent row for every identical call.
- **C. Audit-storage failure is reported.** When a database failure prevents an attempt from being recorded, the failure is reported, and the attempt is never claimed as durably audited.
- **D. No fabricated empty history.** No production-style adapter may assert an empty issuance history merely because EI-5 and EI-6 are not implemented. Only explicitly established synthetic test histories may be used for acceptance testing.
- **E. Architecture record.** The persistence architecture is the separate [ADR-021](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md). ADR-020 is not replaced.

### 9.3 Required EI-4C compatibility work (pending)

The approved design needs an EI-4C Application compatibility slice **before** EI-4D. None of it is implemented by this record:

- the composite policy-set identity, under a clearly versioned identity scheme (EI4-D9-B);
- versioned canonical structured assessment and proposal documents (EI4-D9-C);
- content fingerprints that exclude diagnostic prose (EI4-D9-C);
- modeled issuance derived deterministically from a versioned synthetic latency (EI4-D9-C);
- the complete venue-day timing evidence preserved (EI4-D9-I);
- verifiable point-in-time issued-history evidence, with coverage and recording provenance, preserved (EI4-D9-H, EI4-D9-I);
- explicit persistence-oriented disposition and attempt contracts where needed (EI4-D9-A).

### 9.4 Approved implementation sequence

1. EI4-D9 governance approval and ADR-021 (this record).
2. EI-4C compatibility adjustments in Application ([section 9.3](#93-required-ei-4c-compatibility-work-pending)).
3. EI-4D-A: Application persistence ports, immutable records and record-and-reconcile orchestration.
4. EI-4D-B: Infrastructure SQLite schema, adapter and concurrency protection.
5. EI-4E: synthetic end-to-end acceptance on real SQLite.

EI-5 and EI-6 stay separate milestones.

### 9.5 Decisions still pending or unapproved

These are **not** approved by this record, and each keeps its existing status:

- EI4-D5, pre-entry invalidation (PROPOSED, deferred);
- EI0-D11, risk parameters, including any portfolio exposure or simultaneous-open limit;
- the product specification's proposed one-open-recommendation-per-listing threshold (section 9.5 of the specification);
- EI0-D19, the daily modeled loss limit;
- EI0-D21, candle-revision materiality;
- EI3-D10, the trailing unfinalized candle policy;
- every other decision in [section 5](#5-decisions-still-pending-or-deferred) other than EI4-D9, and every real-data source, venue or market-reference decision.

### 9.6 Earlier approvals unchanged

Every approval in [section 3](#3-approved-decisions) and [section 6](#6-earlier-approvals-unchanged) keeps its status and date. EI4-D9-B refines how EI4-D2's policy version is identified; it does not change EI4-D2's rule of zero or one recommendation per candidate under that identity.

### 9.7 Restrictions unchanged

Every restriction in [section 7](#7-production-readiness-restrictions) still applies. EI4-D9 authorizes no real-data use, actual recommendation issuance, trading, broker order, fill or position.

## 10. EI4-D10 and EI4-D11 owner approval record (2026-11-08)

**Decisions:**

- **EI4-D10 -- Immutable Policy-Version Identity** (EI-4E finding AA-1);
- **EI4-D11 -- Mandatory Verified Recording Boundary and Invalid-Record Recovery Governance** (EI-4E finding AA-2).

**Scope: synthetic-only, LONG-only NSE equity intraday recommendation generation.**

This section records an approval given on **2026-11-08**. It does not rewrite sections 1 to 9. Where they, or the [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md), describe AA-1 and AA-2 as pending, or say there is no EI4-D10 or later, dated notes there point here. **Nothing in this section is implemented by this record.**

The owner's instruction, given on 2026-11-08, was:

> "Northstar owner governance decisions: AA-1: approve. AA-2: approve. Proceed only with the explicitly approved decisions; prepare the next governance documentation task before implementation."

The approval applies to the AA-1 and AA-2 proposals reviewed after EI-4E acceptance ([EI-4E Synthetic Acceptance Record, section 14](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#14-open-owner-decisions)), in the wording below. It extends to no other unresolved policy, to operational issuance, or to any recovery implementation.

### 10.1 Identifier mapping

| Register identifier | EI-4E finding | Title |
| --- | --- | --- |
| EI4-D10 | AA-1 | Immutable Policy-Version Identity |
| EI4-D11 | AA-2 | Mandatory Verified Recording Boundary and Invalid-Record Recovery Governance |
| EI4-D11-A | AA-2.1 | Mandatory verified recording boundary |
| EI4-D11-B | AA-2.2 | Fail-closed invalid-record handling |
| EI4-D11-C | AA-2.3 | Evidence preservation |
| EI4-D11-D | AA-2.4 | Separately governed recovery |
| EI4-D11-E | AA-2.5 | Enforcement and regression tests |

The identifiers continue this register's `EI4-` numbering. No existing decision is renumbered.

### 10.2 EI4-D10 -- Immutable Policy-Version Identity (AA-1)

**OWNER APPROVED, 2026-11-08.**

- Within a defined policy namespace and policy type, a published policy-version label must identify exactly one immutable canonical policy definition.
- Once a label has been published, its canonical policy content cannot change under that label.
- Reusing the same label with different canonical policy content must fail closed as a policy-reference integrity conflict.
- A legitimate change to a policy definition requires a new version label.

**Existing identity architecture preserved.** Different canonical policy content continues to have different content fingerprints. EI4-D10 concerns conflicting reuse of an existing published version label; it does not remove content-based identity, and EI4-D9-B is unchanged. EI4-D10 does not authorize changing:

- the `eirs1` policy-set fingerprint format;
- the `eirk1` generation key format;
- existing canonical document versions;
- previously committed generation dispositions;
- historical policy-definition evidence.

**Enforcement requirements for a future implementation.** A future implementation must:

1. define and respect the actual policy namespace and policy type;
2. bind each published label to one immutable canonical definition;
3. reject conflicting reuse before it can become accepted policy reference evidence;
4. fail closed on incompatible historical or current references;
5. prevent conflicting concurrent registration from establishing contradictory label bindings;
6. preserve historical evidence and existing immutable generation records.

The namespace schema, the registry storage mechanism and the concurrency mechanism are implementation design questions. This record decides none of them.

**Status.**

| Item | Status |
| --- | --- |
| Decision | **APPROVED** |
| Engineering enforcement | **NOT YET IMPLEMENTED.** Today no Application component, store or database constraint enforces label uniqueness |
| Acceptance evidence | EI-4E demonstrated the underlying label-reuse issue (`test_e12_c_a_reused_version_label_with_changed_content_is_another_identity`; [EI-4E record, section 14.1](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#141-aa-1--policy-version-label-governance)). It did not test any enforcement |

### 10.3 EI4-D11 -- Mandatory Verified Recording Boundary and Invalid-Record Recovery Governance (AA-2)

**OWNER APPROVED, 2026-11-08,** as five subdecisions, each applying exactly as worded.

#### EI4-D11-A -- Mandatory verified recording boundary (AA-2.1)

- The EI-4D-A verified Application recorder (`RecordEquityRecommendationGenerationUseCase`) is the required operational path for committing synthetic equity recommendation-generation dispositions.
- The raw EI-4D-B SQLite adapter (`SQLiteEquityRecommendationGenerationStore`) is an Infrastructure persistence mechanism. It is not an independently authorized operational generation interface.

#### EI4-D11-B -- Fail-closed invalid-record handling (AA-2.2)

A stored generation record that fails any of these:

- canonical document verification;
- authoritative EI-3C source verification;
- original EI-2B snapshot verification;
- frozen historical assessment replay

must not be:

- accepted as equivalent;
- silently repaired;
- silently migrated;
- replaced;
- treated as actually issued;
- used as authoritative recommendation evidence.

The existing precise error and incompatibility distinctions are preserved, including:

- invalid versus incompatible stored records;
- source evidence that is invalid versus unavailable;
- replay mismatch;
- unresolved commit reconciliation.

#### EI4-D11-C -- Evidence preservation (AA-2.3)

The original invalid committed record, and the relevant source evidence, are preserved for investigation. They are not silently deleted, overwritten or relinked, and their immutable identity is not changed.

#### EI4-D11-D -- Separately governed recovery (AA-2.4)

Any recovery of an invalid committed record requires:

- a separately approved recovery design;
- explicit authorization;
- an auditable evidence trail;
- preservation of the original invalid record;
- defined authority and operational boundaries.

**The owner has not approved any specific recovery procedure.** The recovery mechanism remains a separate future governance decision. EI4-D11 does **not** authorize:

- deleting immutable rows;
- updating immutable rows;
- replacing an occupied generation key;
- rewriting canonical fingerprints;
- introducing a privileged production repair bypass;
- changing historical issuance status.

#### EI4-D11-E -- Enforcement and regression tests (AA-2.5)

- A future engineering slice must establish enforceable architectural restrictions, and appropriate regression tests, around the required operational recording boundary.
- A documentation-only convention is not technical enforcement, and this record is not presented as one.
- SQLite does not verify every historical semantic invariant. It enforces structure, insert-only history, foreign keys and the insert-time CANDIDATE checks of ADR-021 section 11. Fingerprint, source and replay verification remain the Application's (EI4-D9-I; [ADR-021, section 10](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md#10-restart-and-historical-verification-ei4-d9-i)).

#### EI4-D11 status

| Item | Status |
| --- | --- |
| Decision | **APPROVED** (EI4-D11-A to EI4-D11-E) |
| Verified-recorder enforcement | **NOT YET IMPLEMENTED OR INDEPENDENTLY ACCEPTED.** Today a direct adapter call can still commit a record that later fails verification ([EI-4E record, section 14.2](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#142-aa-2--verified-writer-boundary-and-recovery)) |
| Recovery mechanism | **NOT APPROVED** |
| Existing invalid evidence | Must remain fail-closed and preserved |
| Acceptance evidence | EI-4E observed fail-closed handling on the paths it tested (`test_e20_*` and `test_e21_*`), and the occupied-key consequence. That observation is not acceptance of EI4-D11's enforcement |

### 10.4 Not authorized by this record

This record approves two governance decisions only. It does not authorize:

- actual recommendation issuance, or the lifecycle `RECORDED` state;
- broker or paper orders, positions or fills;
- realized P&L, or claims of profitable trading;
- live market-data integrations, or exchange-data licensing;
- production trading;
- changes to existing fingerprints, or to the `eirs1` or `eirk1` formats;
- deletion or mutation of historical data or of immutable generation records;
- any invalid-record recovery process;
- EI-5 or EI-6 implementation.

Every restriction in [section 7](#7-production-readiness-restrictions) and [section 9.7](#97-restrictions-unchanged) still applies.

### 10.5 Decisions still pending or unapproved

These are **not** approved by this record, and each keeps its existing status:

- EI4-D5, pre-entry invalidation (PROPOSED, deferred);
- EI0-D11, risk parameters, including any portfolio exposure or simultaneous-open limit;
- the product specification's proposed one-open-recommendation-per-listing threshold (section 9.5 of the specification);
- EI0-D19, the daily modeled loss limit;
- EI0-D21, candle-revision materiality;
- EI3-D10, the trailing unfinalized candle policy;
- any specific invalid-record recovery design or procedure (EI4-D11-D);
- every other decision listed as pending in [section 5](#5-decisions-still-pending-or-deferred) and [section 9.5](#95-decisions-still-pending-or-unapproved), and every real-data source, venue or market-reference decision.

No production parameter is approved or implied.

### 10.6 Earlier approvals and records unchanged

- Every approval in sections 3, 6 and 9 keeps its text, status and date, including EI4-D9-A to EI4-D9-J and clarifications A to E. EI4-D10 adds a rule about version labels to EI4-D9-B's policy-set identity without changing it.
- [ADR-020](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md) and [ADR-021](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md) are unchanged. Neither conflicts with EI4-D10 or EI4-D11:
  - ADR-020 section 2 adopts "named, versioned generation policies, where any parameter change is a new version", and section 4 makes every policy input explicit and versioned. EI4-D10 makes conflicting label reuse fail closed; it contradicts neither.
  - ADR-020 section 3 says business uniqueness is enforced by the immutable store (EI-4D). That remains true: EI4-D11 governs which path may write to the store, not where uniqueness is enforced.
  - ADR-021 sections 10 and 11 already assign fingerprint, source and replay verification to the Application, and only structure and foreign keys to SQLite. Neither ADR names the raw adapter as an operational generation interface.
- The [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md) keeps its engineering verdict, PASS WITH DOCUMENTED LIMITATIONS, and its evidence:
  - 72 EI-4E acceptance tests;
  - 2,808 Infrastructure tests passed, 27 skipped, 0 failed;
  - CI not confirmed.

  It records this approval in a dated post-acceptance update. Owner approval adds no test coverage, and the EI-4E coverage gaps recorded there stay open.

### 10.7 Prerequisites before implementation

1. **Next governance documentation task.** Before any enforcement implementation begins, the owner directed that the next governance documentation task be prepared. It is not part of this record.
2. **EI4-D10 design.** A design for the policy namespace and policy type, the immutable label binding, conflict rejection, concurrent registration and historical compatibility, consistent with EI4-D9-B and without changing `eirs1` or `eirk1`.
3. **EI4-D11 boundary design.** A design for enforcing the verified-recorder boundary of EI4-D11-A and EI4-D11-E, with regression tests and independent acceptance.
4. **Recovery.** Any invalid-record recovery mechanism needs its own separately approved design and authorization (EI4-D11-D). None exists.

> **Note, 2026-11-08 (EI4-D10 enforcement design owner approval).** Prerequisite 2 is met. The EI4-D10 design was prepared and its nine design decisions were approved by the owner on 2026-11-08 ([section 11](#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08); [ADR-022](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md)). Prerequisites 3 and 4 are unchanged: no EI4-D11 boundary design and no recovery design is approved.

> **Note, 2026-11-10 (EI4-D11 enforcement design owner approval).** Prerequisite 3 is met. The EI4-D11 boundary design was prepared, and its engineering decisions were approved by the owner on 2026-11-10 ([section 13](#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10); [ADR-023](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)). Its implementation and acceptance are not yet established. Prerequisite 4 is unchanged: no recovery design is approved.

## 11. EI4-D10 enforcement design owner approval record (2026-11-08)

**Decisions: EI4-D10-DQ1 to EI4-D10-DQ9, the engineering design decisions of the [EI4-D10 Policy-Version Identity Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md).**
**Scope: EI4-D10 enforcement for synthetic-only, LONG-only NSE equity intraday recommendation generation.**

This section records an approval given on **2026-11-08**. It does not rewrite sections 1 to 10. The owner approved all nine design decisions **as recommended in the design's section 15**, including the clarifications below for EI4-D10-DQ6 and EI4-D10-DQ9. The enforcement architecture is recorded in [ADR-022](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md). **Nothing in this section is implemented by this record.**

### 11.1 Approved design decisions

| ID | Question | Approved decision | Status |
| --- | --- | --- | --- |
| EI4-D10-DQ1 | Policy namespace | One closed namespace, `northstar.equity-intraday-v1.synthetic`, part of every binding identity. It covers the current synthetic equity intraday policy catalog only, never Futures, Options, live equity execution, commodities or other verticals | **OWNER APPROVED, 2026-11-08** |
| EI4-D10-DQ2 | Policy types | Exactly `GENERATION_POLICY`, `RISK_POLICY`, `COST_SCHEDULE`, `PRICE_INCREMENT`, `TIMING_POLICY` and `POLICY_SET`. Uniqueness per (namespace, policy type, published label), never global across types. Listing eligibility and the EI-3 research configuration stay with their existing mechanisms | **OWNER APPROVED, 2026-11-08** |
| EI4-D10-DQ3 | Price-increment granularity | One immutable canonical amount per published label. A changed amount needs a new label. No production NSE tick size is introduced or inferred | **OWNER APPROVED, 2026-11-08** |
| EI4-D10-DQ4 | Catalog location | A durable immutable catalog in the same SQLite file as the EI-2B evidence, the EI-3C timelines and the EI-4D records, behind an Application-owned port and an Infrastructure-owned SQLite adapter. No separate file, second authority, process-local authority or direct Application SQL | **OWNER APPROVED, 2026-11-08** |
| EI4-D10-DQ5 | Enforcement points | Layered: explicit publication; atomic catalog registration; catalog-backed resolution; EI-4C validation while gathering inputs; a pre-write binding gate in the EI-4D-A verified recorder; catalog-free historical replay with a separate read-only audit | **OWNER APPROVED, 2026-11-08** |
| EI4-D10-DQ6 | Failure classification | Existing top-level `DEPENDENCY_UNAVAILABLE` with the distinct machine-readable reason `POLICY_REFERENCE_CONFLICT`, under the clarification in 11.2 | **OWNER APPROVED, 2026-11-08** |
| EI4-D10-DQ7 | Historical compatibility | No automatic backfill and no fabricated publication history. A read-only historical binding audit; no rewrite of evidence, records, fingerprints or keys | **OWNER APPROVED, 2026-11-08** |
| EI4-D10-DQ8 | Architecture record | A new ADR: [ADR-022](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md) | **OWNER APPROVED, 2026-11-08** |
| EI4-D10-DQ9 | Publication trust | Reliance on the existing SQLite database-file trust boundary, as a deliberate and limited assumption, under the clarification in 11.3 | **OWNER APPROVED, 2026-11-08** |

### 11.2 EI4-D10-DQ6 clarification: a permanent conflict

- **Permanent, not transient.** A published label bound to different canonical content is a **permanent integrity conflict** for that policy identity, never transient catalog downtime. It fails closed, is never retried automatically, and never leads to silent republication with changed content.
- **Temporary outage.** A temporary catalog outage is dependency unavailability. It may be retried only under a separately approved operational retry policy; none exists. It never creates a binding.
- **Unpublished label.** An unknown or unpublished label fails closed. No default is invented, and absence is not publication evidence.
- **Corrupt or unsupported binding.** These keep the invalid versus incompatible distinction, fail closed, preserve evidence and are never migrated.
- **Other conflicts stay distinct.** Generation-content conflicts (`CONTENT_CONFLICT`) and eligibility rejections stay distinct from a policy-reference conflict.
- **No schema change.** No new persisted attempt status is created. The immutable attempts table and the approved status semantics are not changed.
- **Propagation.** The machine-readable reason must propagate through the relevant outcome and audit contracts. The future Application and Infrastructure slices must validate this.

### 11.3 EI4-D10-DQ9 clarification: a limited trust assumption

- **Scope.** Reliance on the SQLite file trust boundary applies to current personal-use synthetic research only.
- **What it protects.** SQLite constraints protect ordinary supported writes.
- **What it does not protect against.** Sufficiently privileged local file or database access can bypass application-level publication authority.
- **What it does not establish:**
  - tamper-proof publication;
  - protection against an operator with file write access;
  - cryptographic proof of the publisher;
  - multi-host authorization;
  - a production identity or permission system;
  - protection against every direct database-write bypass.
- **Real trading.** It is not acceptable for real trading without separate review.
- **EI4-D11.** It does not weaken EI4-D11's mandatory verified recording requirement.

### 11.4 Status

| Item | Status |
| --- | --- |
| EI4-D10 governance policy | APPROVED (2026-11-08, [section 10.2](#102-ei4-d10----immutable-policy-version-identity-aa-1)) |
| EI4-D10 technical enforcement design | **APPROVED** (2026-11-08): EI4-D10-DQ1 to EI4-D10-DQ9 and ADR-022 |
| EI4-D10 engineering implementation | **NOT YET IMPLEMENTED** |
| EI4-D10 implementation acceptance | **NOT YET ESTABLISHED.** The design's acceptance tests are proposed and not executed |

### 11.5 Approved implementation sequence

1. **S1. Application contracts** (northstar-application): binding contracts, catalog port, publication use case, catalog-backed resolver, historical binding audit, typed errors. No SQLite.
2. **S2. Infrastructure catalog** (northstar-infrastructure): the same-file SQLite catalog, additive immutable schema, atomic publication, concurrency, commit uncertainty. No generation-gate changes.
3. **S3. Application enforcement** (northstar-application): generation validation and the recorder pre-write gate.
4. **S4. Infrastructure end-to-end acceptance** (northstar-infrastructure): fixtures publish their synthetic policies; EI-4E integration is updated; conflicts, restart and concurrency are accepted.
5. **S5. Documentation acceptance** (northstar-docs).

S3 and S4 change generator and recorder construction and must be coordinated. Neither is represented as independently accepted full enforcement before integration. No slice is implemented or started by this record, and EI-4E's historical evidence is not altered.

### 11.6 Unchanged

- **EI4-D11** and its recovery restriction are unchanged ([section 10.3](#103-ei4-d11----mandatory-verified-recording-boundary-and-invalid-record-recovery-governance-aa-2)).
  - EI4-D10 enforcement does not complete EI4-D11.
  - Until EI4-D11's boundary is implemented and accepted, the EI4-D10 recording gate can be bypassed by direct Infrastructure adapter use.
  - No invalid-record recovery design is approved.
- **Identity and records.** `eirs1`, `eirk1`, every existing canonical document version, the evaluator version and every stored record are unchanged. ADR-020 and ADR-021 are unchanged.
- **EI-4E.** The [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md) and its verdict, PASS WITH DOCUMENTED LIMITATIONS, are unchanged.
- **Pending decisions.** These are not approved by this record, and keep their status:
  - EI4-D5;
  - EI0-D11;
  - the same-listing exclusivity threshold;
  - EI0-D19;
  - EI0-D21;
  - EI3-D10;
  - any invalid-record recovery design;
  - every other pending decision in [section 5](#5-decisions-still-pending-or-deferred), [section 9.5](#95-decisions-still-pending-or-unapproved) and [section 10.5](#105-decisions-still-pending-or-unapproved).

  No production policy parameter is introduced.
- **Non-authorizations.** This record does not authorize:
  - historical evidence mutation;
  - changes to `eirs1` or `eirk1`;
  - invalid-record recovery;
  - direct-adapter operational recording;
  - actual recommendation issuance or the lifecycle `RECORDED` state;
  - broker or paper orders, fills or positions;
  - realized P&L, live data integrations, data-use licensing claims or production trading;
  - EI-5 or EI-6.

  Every restriction in [section 7](#7-production-readiness-restrictions) still applies.

## 12. EI4-D10 engineering acceptance record (2026-11-10)

This section records engineering acceptance evidence dated **2026-11-10**. It is not an owner approval, and it does not rewrite sections 1 to 11. The owner approvals of EI4-D10 ([section 10.2](#102-ei4-d10----immutable-policy-version-identity-aa-1)) and of EI4-D10-DQ1 to EI4-D10-DQ9 ([section 11](#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08)), both dated 2026-11-08, are unchanged. The evidence is in the [EI4-D10 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md).

### 12.1 Status

The status table in [section 11.4](#114-status) records the state before implementation and is not rewritten. The current state is:

| Item | Status |
| --- | --- |
| EI4-D10 governance policy | APPROVED (2026-11-08, unchanged) |
| EI4-D10 technical enforcement design | APPROVED (2026-11-08, unchanged): EI4-D10-DQ1 to EI4-D10-DQ9 and ADR-022 |
| EI4-D10 engineering implementation | **IMPLEMENTED** (S1 to S4 merged; S5 is the acceptance record) |
| EI4-D10 engineering acceptance | **PASS WITH DOCUMENTED LIMITATIONS** (2026-11-10). CI is not independently verified |
| EI4-D11 operational write boundary | **NOT YET IMPLEMENTED OR INDEPENDENTLY ACCEPTED** (unchanged, [section 10.3](#103-ei4-d11----mandatory-verified-recording-boundary-and-invalid-record-recovery-governance-aa-2)) |
| Invalid-record recovery | **NOT APPROVED** (unchanged) |

### 12.2 Implementation slices

| Slice | Repository | Merge (PR) |
| --- | --- | --- |
| S1. Application contracts | northstar-application | `ed98ecc` (#89) |
| S2. Infrastructure catalog | northstar-infrastructure | `f7b1ab8` (#55) |
| W2. EI-4D-B cleanup reliability fix (separate; not part of the catalog) | northstar-infrastructure | `0fa10b8` (#56) |
| S3. Application enforcement | northstar-application | `cd2301b` (#90) |
| S4. Infrastructure end-to-end acceptance | northstar-infrastructure | `97700b3` (#57) |
| S5. Documentation acceptance | northstar-docs | This section and the acceptance record |

### 12.3 What the acceptance does not change

- **Limitations.** The acceptance holds within the limitations of its record: synthetic only; Windows process tests only; no historical publication proof; the invalid and incompatible subtype not a generation reason code; end-to-end outages by lock contention only; the DQ9 file-level trust; EI4-D11 not enforced; recovery not approved.
- **EI4-D11.** EI4-D10 enforcement does not complete EI4-D11. A direct Infrastructure adapter write can still bypass the recording gate.
- **EI-4E.** The [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md) and its verdict are unchanged. Its AA-1 finding is closed for the verified Application workflow by EI4-D10, as the acceptance record shows.
- **Pending decisions.** These keep their status:
  - EI4-D5;
  - EI0-D11;
  - the same-listing exclusivity threshold;
  - EI0-D19;
  - EI0-D21;
  - EI3-D10;
  - the EI4-D11 recovery mechanism;
  - every other pending decision in sections [5](#5-decisions-still-pending-or-deferred), [9.5](#95-decisions-still-pending-or-unapproved) and [10.5](#105-decisions-still-pending-or-unapproved).
- **Non-authorizations.** The non-authorizations of [section 11.6](#116-unchanged) and the restrictions of [section 7](#7-production-readiness-restrictions) still apply.

## 13. EI4-D11 enforcement design owner approval record (2026-11-10)

**Decisions: EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8 and DQ10, engineering decisions of the [EI4-D11 Mandatory Operational Generation-Write Boundary Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md).**
**Scope: the operational write path for synthetic equity recommendation-generation records (EI-4D dispositions, proposals and logical attempts).**

This section records an approval given on **2026-11-10**. It does not rewrite sections 1 to 12.
- The governance decision itself, EI4-D11-A to EI4-D11-E ([section 10.3](#103-ei4-d11----mandatory-verified-recording-boundary-and-invalid-record-recovery-governance-aa-2)), was approved on 2026-11-08 and is unchanged.
- This section records the **engineering enforcement design** approved for it, with two mandatory clarifications.
- The architecture is recorded in [ADR-023](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md).
- **Nothing in this section is implemented by this record.**

### 13.1 Approved engineering decisions

| ID | Question | Approved decision | Status |
| --- | --- | --- | --- |
| EI4D11-DQ1 | Operational trust boundary | Covers supported operational Application workflows, composition roots, public Application read and write interfaces, supported Infrastructure persistence interfaces, accidental or ordinary raw-adapter use, and mandatory verification before operational writes. No protection is claimed against privileged Python, direct SQL or file modification, malicious code, cryptographic forgery by privileged code, multi-host authorization or production trading security. Appropriate to personal-use synthetic research only; the ADR-022 DQ9 limitation is unchanged | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ2 | Enforcement mechanism | Option D, layered: read/write port separation; the verified recorder as the operational write boundary; recorder-issued verified-write authorization; Infrastructure validation before opening SQLite; CI architecture and dependency checks; existing SQLite immutability as defence in depth | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ3 | Attempt-only writes | Inside the boundary, under a separate attempt authorization bound to the exact attempt; no fabricated policy evidence, assessment, proposal or disposition; attempt identity and status formats unchanged | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ5 | Legacy raw write methods | Removed from the supported port and adapter after a phased migration. Temporary compatibility only during migration. Read-only historical access stays. Test-only corruption injection is separate from the production APIs | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ8 | Acceptance platform | Windows: local SQLite, independent connections, spawned processes, restart and durability, failed authorization before any connection, immutable-record guarantees. POSIX separate-process and multi-host behaviour are explicit limitations | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ10 | Architecture record | [ADR-023](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md) | **OWNER APPROVED, 2026-11-10** |

EI4D11-DQ4 (authorization representation), DQ6 (test migration), DQ7 (PR sequence) and DQ9 (investigation through existing read-only surfaces) are engineering choices. They need no separate approval, and are constrained by the clarifications below.

### 13.2 Mandatory clarification A: migration order

- **The order is W1 → W2-I → W3 → W4-I → W5.**
- **W1 (Application)** migrates the verified recorder to the authorized-write interface **before W2-I (Infrastructure)** refuses legacy raw writes.
- **W1's temporary defaults** delegate the new methods to the legacy ones. They are a compatibility mechanism, not enforcement, and W1 alone does not complete EI4-D11.
- **Compatibility is tested, not assumed.** Each slice is tested against the other repository's current `develop`. Cross-repository merges are not atomic, and required CI checks are never bypassed.
- **Gaps are reported, not worked around.** A compatibility gap found in source is reported for engineering review before implementation, never closed by weakening enforcement.

### 13.3 Mandatory clarification B: exact write-content binding

- **Exact binding.** An authorization binds to the exact verified write contents: the disposition with its proposal, the logical attempt, the verified source and frozen-evidence results, and the verified policy-binding check. It never binds by `eirk1` alone, which does not cover assessment content.
- **Existing identities only.** It uses existing canonical documents and identities; no new fingerprint scheme is approved.
- **Adapter refusal.** The adapter refuses contents that differ from those the authorization covers, before any SQLite connection opens.
- **Reuse.** Reuse is permitted only for the identical operation during existing bounded EI4-D9-G reconciliation. There is no blanket one-time nonce, and no second authorization for contradictory contents under the same key.
- **Limitation.** Within the personal-use trust model, the issuer capability is not tamper-proof against privileged code.

### 13.4 Status

| Item | Status |
| --- | --- |
| EI4-D11 governance (EI4-D11-A to EI4-D11-E) | APPROVED (2026-11-08, unchanged) |
| EI4-D11 engineering enforcement design | **APPROVED** (2026-11-10): EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8, DQ10, clarifications A and B, and ADR-023 |
| EI4-D11 implementation | **NOT YET IMPLEMENTED** |
| EI4-D11 engineering acceptance | **NOT YET ESTABLISHED.** The design's 33 acceptance scenarios are proposed and not executed |
| Invalid-record recovery | **NOT APPROVED** (EI4-D11-D, unchanged) |

The [EI4-D11 status table in section 10.3](#ei4-d11-status) records the status before this approval and is not rewritten.

### 13.5 Unchanged

- **EI4-D10.** The EI4-D10 architecture (ADR-022) and its engineering acceptance ([section 12](#12-ei4-d10-engineering-acceptance-record-2026-11-10)) are unchanged. ADR-021 is unchanged.
- **EI-4E.** The [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md) and its verdict are unchanged.
- **Other stores.** EI4-D11 is not extended to the futures, options or paper-trading stores.
- **Pending decisions.** These keep their status:
  - EI4-D5;
  - EI0-D11;
  - the same-listing exclusivity threshold;
  - EI0-D19;
  - EI0-D21;
  - EI3-D10;
  - the EI4-D11 recovery mechanism;
  - every other pending decision in sections [5](#5-decisions-still-pending-or-deferred), [9.5](#95-decisions-still-pending-or-unapproved) and [10.5](#105-decisions-still-pending-or-unapproved).
- **Non-authorizations.** This record does not authorize:
  - implementation of any slice;
  - invalid-record recovery, or overwrite, repair or deletion of immutable evidence;
  - new `eirs1`, `eirk1` or `eirat1` schemes;
  - changes to historical canonical evidence;
  - new production operational entry points or database authorities;
  - broker integration;
  - recommendation issuance or the lifecycle `RECORDED` state;
  - paper or live trading;
  - positions or fills;
  - real-market readiness;
  - EI-5 or EI-6.

  Every restriction of [section 7](#7-production-readiness-restrictions) still applies.

## 14. EI4-D11 engineering acceptance record (2026-11-11)

This section records engineering acceptance evidence dated **2026-11-11**. It is not an owner approval, and it does not rewrite sections 1 to 13. The owner approvals of EI4-D11-A to EI4-D11-E ([section 10.3](#103-ei4-d11----mandatory-verified-recording-boundary-and-invalid-record-recovery-governance-aa-2), 2026-11-08) and of the enforcement design ([section 13](#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10), 2026-11-10) are unchanged. The evidence is in the [EI4-D11 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Engineering-Acceptance-Record.md).

### 14.1 Status

The status table in [section 13.4](#134-status) records the state before implementation and is not rewritten. The current state is:

| Item | Status |
| --- | --- |
| EI4-D11 governance (EI4-D11-A to EI4-D11-E) | APPROVED (2026-11-08, unchanged) |
| EI4-D11 engineering enforcement design | APPROVED (2026-11-10, unchanged): EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8, DQ10, clarifications A and B, and ADR-023 |
| EI4-D11 engineering implementation | **IMPLEMENTED** (W1 to W4-I merged; W5 is the acceptance record) |
| EI4-D11 engineering acceptance | **PASS WITH DOCUMENTED LIMITATIONS -- synthetic engineering acceptance** (2026-11-11). 28 scenarios VERIFIED, 5 VERIFIED WITH LIMITATION, 0 NOT VERIFIED. CI is not independently verified |
| Production operational composition root | **NONE EXISTS** |
| Invalid-record recovery | **NOT APPROVED** (EI4-D11-D, unchanged) |

### 14.2 Implementation slices

| Slice | Repository | Merge (PR) |
| --- | --- | --- |
| W1. Application authorization contracts and recorder migration | northstar-application | `298a3a5` (#91) |
| W2-I. Infrastructure authorized-write enforcement | northstar-infrastructure | `6b90597` (#58) |
| W3. Application legacy write-interface removal | northstar-application | `8937f12` (#92) |
| W4-I. Infrastructure final adapter interface and acceptance | northstar-infrastructure | `6282cae` (#59) |
| W5. Documentation acceptance | northstar-docs | This section and the acceptance record |

### 14.3 What the acceptance does not change

- **Limitations.** The acceptance holds within the limitations of its record. Five scenarios are VERIFIED WITH LIMITATION:
  - 12, invalid committed evidence: privileged test-only SQL simulation; no recovery of occupied invalid keys;
  - 21, Windows separate-process testing: no POSIX or multi-host behavior established;
  - 23, no backfill or migration: tested against enforced code and planted historical records; historical revisions were not rerun;
  - 29, composition-root rule: no production composition root exists; future integrations cannot be certified by the current scans;
  - 31, migration ordering: relies partly on previously reported W1/W2-I runs rather than rerunning historical revisions.
- **Trust model.** No protection is claimed against privileged Python, direct database or file modification, source tampering or forged capability objects (EI4D11-DQ1; ADR-022 DQ9).
- **Earlier records.** The [EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md) and the [EI4-D10 Engineering Acceptance Record](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md), with their verdicts, are unchanged. EI-4E finding AA-2 is closed for supported and ordinary operational writes, as the acceptance record shows; an occupied invalid key stays occupied.
- **Other stores.** The futures, options and paper-trading stores are unchanged.
- **Pending decisions.** These keep their status:
  - EI4-D5;
  - EI0-D11;
  - the same-listing exclusivity threshold;
  - EI0-D19;
  - EI0-D21;
  - EI3-D10;
  - the EI4-D11 recovery mechanism;
  - every other pending decision in sections [5](#5-decisions-still-pending-or-deferred), [9.5](#95-decisions-still-pending-or-unapproved) and [10.5](#105-decisions-still-pending-or-unapproved).
- **Non-authorizations.** The non-authorizations of [section 13.5](#135-unchanged) and the restrictions of [section 7](#7-production-readiness-restrictions) still apply. Nothing here authorizes a production composition root, live deployment, market-data consumption, brokerage activity, recovery, EI-5 or EI-6.
