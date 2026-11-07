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
