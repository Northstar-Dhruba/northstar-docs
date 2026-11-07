# Equity Intraday V1 EI-3 Decision Register

## Status

**Owner acceptance record, 2026-11-05. Scope: synthetic-only EI-3A research.**

- The Northstar product owner explicitly approved the eight decisions in [section 3](#3-approved-decisions) on **2026-11-05**: EI3-D1, EI3-D2, EI3-D3, EI3-D4, EI3-D5, EI3-D8, EI0-D3 and EI0-D6.
- No other decision is approved by this record. Every decision listed in [section 5](#5-outstanding-decisions) keeps its existing status.
- Nothing here authorizes production use, real market data, a real data provider, broker execution, paper fills or a trade recommendation.
- This record has not been through Architecture Review or Engineering Review.

> **Note, 2026-11-06 (EI-3B, owner approval).** The owner **APPROVED** EI3-D6 (research-only scheduling), EI3-D7 (candidate frequency) and EI3-D9 (conflicting evidence and watermark regression) on 2026-11-06, for synthetic-only EI-3B. They are recorded in [section 10](#10-ei-3b-owner-approval-record-2026-11-06). Every approval dated 2026-11-05 is unchanged. EI0-D7 and the newly recorded EI3-D10 stay pending. Nothing here authorizes execution or production use.

> **Note, 2026-11-07 (EI-3C, owner approval).** The owner **APPROVED** EI3-D11 (timeline establishment), EI3-D12 (concurrent commit authority) and EI3-D13 (historical decision reproducibility) on 2026-11-07, for synthetic-only EI-3C. They are recorded in [section 11](#11-ei-3c-owner-approval-record-2026-11-07). Every earlier approval is unchanged. EI0-D7, EI3-D10, EI0-D8 and EI0-D21 stay pending. Nothing here authorizes execution or production use.

> **Note, 2026-11-07 (EI-3C, engineering acceptance).** EI-3C passed final synthetic acceptance on 2026-11-07 with the verdict PASS WITH DOCUMENTED LIMITATIONS, recorded in the [EI-3C Synthetic Acceptance Record](Equity-Intraday-V1-EI3C-Synthetic-Acceptance-Record.md). That record is engineering evidence, not an owner approval: no decision status changes, and nothing authorizes real data, execution or production use.

## 1. Purpose

This register records the owner decisions that govern the first Equity Intraday V1 research implementation, EI-3A, before any EI-3A code is written.

- It is the authoritative record for decisions with the `EI3-` prefix.
- It also records the approval of two decisions from the EI-0 owner decision register, EI0-D3 and EI0-D6. Their entries in the [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md), section 14, point here.
- It does not change any decision recorded in the [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md) or in [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md).
- Implementation history is not approval. That code already behaves a certain way is never recorded here as an owner decision.

## 2. Scope

| Item | Scope of this record |
| --- | --- |
| Product | Northstar Equity Intraday V1: NSE cash equities, INR |
| Milestone | EI-3A, the first research slice |
| Data | Synthetic only: invented candles, invented finality statements and explicitly synthetic reference fixtures |
| Outputs | Research decisions only: `LONG_CANDIDATE`, `NO_SIGNAL` or an explicit rejection with a reason |
| Not in scope | Entry, exit, stop-loss, target, capital, position size, brokerage, taxes, fees, slippage, recommendation lifecycle, paper fills, broker orders, real data and production use |

Every approval below is limited to this scope unless its own text says otherwise.

## 3. Approved decisions

Each decision below is **APPROVED BY OWNER, 2026-11-05**.

### EI3-D1 — Research decision instant

**Approved rule.**

- The research decision instant is the `as_of` instant of the immutable EI-2B evidence snapshot the decision is evaluated from.
- A bar's interval-end timestamp does not, by itself, establish that the candle is final or that a decision is available.
- Only evidence that is visible and valid at the snapshot's observation instant may contribute to the decision.

**Rationale.**

- Under EI-2A, a candle is final only when finality evidence from its own source, event and revision was confirmed at or after the interval end and received by the observation instant. A candle can therefore never be final at its own interval-end instant.
- The EI-2B snapshot already freezes exactly the evidence visible at its `as_of` instant and at its journal watermark.

**Sources.**

- EI-2A: `northstar-application/src/northstar_application/application_services/equity_intraday_candle_validation.py`
- EI-2B: `northstar-application/src/northstar_application/application_services/equity_candle_evidence_snapshot.py`
- Supersedes, for research decisions only, the statement in the [product specification, section 8.1](Equity-Intraday-V1-Product-Specification.md#81-minimum-recommendation-contract), that the decision instant is the completion instant of the last cited bar. See [section 6.1](#61-what-ei3-d1-does-not-decide).

### EI3-D2 — Required evidence completeness

**Approved rule.** A research evaluation is rejected when any interval it requires contains:

- a missing observation;
- an unfinalized candle;
- conflicting values;
- unresolved source revisions.

Missing candles are never interpolated. Values are never carried forward. No winning revision is selected.

**Implementation latitude.** The exact rejection-reason names are defined in EI-3A. They must preserve these four approved categories and keep them distinguishable.

**Rationale.** EI-2A already reports these four conditions per interval as `MISSING`, `UNFINALIZED`, `CONFLICTING_VALUES` and `UNRESOLVED_REVISIONS`. The [product specification](Equity-Intraday-V1-Product-Specification.md) forbids gap filling (sections 5.3 and 8.2).

### EI3-D3 — Research outcome vocabulary

**Approved rule.** A research decision has exactly one of these outcomes:

- `LONG_CANDIDATE`;
- `NO_SIGNAL`;
- an explicit rejection, with its reason.

`BUY`, `HOLD` and `SELL` are not reused as though these outcomes were executable recommendations. `LONG_CANDIDATE` is a research result. It is not a recommendation, a broker order or a paper fill.

**Rationale.** It keeps the equity research vocabulary separate from the existing daily research labels and from the Futures research-to-direction mapping ([ADR-018](../adr/ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md), F4-D1), as the [product specification](Equity-Intraday-V1-Product-Specification.md) proposes in sections 3.3 and 7.1.

### EI3-D4 — Initial research strategy

**Approved rule.** The initial EI-3A research strategy is an **opening-range breakout, with a session VWAP approximation filter and volume confirmation**.

- It is approved for synthetic research only.
- No claim of profitability or statistical significance may be made from it.
- It approves no entry, exit, stop-loss, target, sizing, brokerage or slippage rule.

**Relationship to EI0-D8.** EI0-D8 (initial strategy, with a research control) stays **PENDING**. EI3-D4 approves the candidate strategy for synthetic EI-3A research only. It does not approve the research control, the strategy for real-data research, or the strategy for recommendation generation.

**Naming.** "Session VWAP" here is an approximation computed from completed 5-minute bars. It is not the exchange's traded VWAP and must be named as an approximation ([product specification, section 7.4](Equity-Intraday-V1-Product-Specification.md#74-initial-strategy-selection-pending)).

### EI3-D5 — Synthetic calibration governance

**Approved rule.** The initial research strategy may use explicit, versioned synthetic calibration parameters. These parameters:

- must be part of the research configuration;
- must be reproducible;
- must be visibly synthetic;
- must not be presented as validated production trading rules;
- must not change silently between evaluations. A changed parameter is a new configuration version.

**Numeric examples are not approvals.** An opening-range length of three bars and a volume multiplier of 1.5 were suggested as examples. Neither value is approved, individually or together, as a production parameter or as a fixed EI-3A value.

**Rationale.** This follows the Futures precedent that policy prototypes are calibrated only with synthetic fixtures and that every material parameter belongs to a named version ([ADR-018](../adr/ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md), F4-D7).

### EI3-D8 — Volume comparison baseline

**Approved rule.**

- The breakout candle is excluded from the mean volume its own volume is compared against.
- No future bar is ever part of the baseline.
- Only eligible prior observations available under the relevant snapshot contribute.

**Rationale.** Including the breakout bar in its own baseline dampens the comparison. Including any later bar is look-ahead.

### EI0-D3 — Direction

**Approved rule.**

- Equity Intraday V1 research is LONG only.
- No short candidate is generated.
- A bearish signal is never converted, silently or otherwise, into a short trade.

**Rationale and source.** [Product specification, sections 4 and 4.1](Equity-Intraday-V1-Product-Specification.md#41-challenges-to-the-supplied-proposals): Northstar has no model of intraday short-selling product rules, square-off or borrowing, and LONG-only removes the risk of an unsettled cash-equity short.

### EI0-D6 — Candle timestamp convention

**Approved rule.**

- A five-minute candle is identified by its canonical UTC **interval-end** timestamp.
- UTC storage and IST session interpretation stay distinct: IST is used only to resolve session dates and to present times.
- A bar-start timestamp is never a canonical candle identity.

**Rationale and source.** [Product specification, section 6.4](Equity-Intraday-V1-Product-Specification.md#64-timestamp-and-timezone-semantics). The EI-1A grid and EI-2A validation already use interval ends. That implementation is not the approval; this record is.

## 4. Limitations and exclusions

- **Synthetic only.** Every approval above applies to synthetic research. None applies to real or production data.
- **No trading rules.** No entry, exit, stop-loss, target, capital, position-size, brokerage, tax, fee or slippage rule is approved.
- **No calibration values.** No numeric strategy parameter is approved (EI3-D5).
- **No edge.** No result may be presented as evidence of profitability or statistical significance.
- **No recommendations or executions.** A `LONG_CANDIDATE` is not a recommendation, order, fill, position or P&L.
- **No production finality or eligibility.** No real candle-finality source and no production listing-applicability source is approved. Research on anything but synthetic evidence remains impossible by design.

## 5. Outstanding decisions

These decisions are **not** approved by this record. Each keeps its existing status and wording.

### 5.1 EI-3 decisions still pending

| ID | Question | Status |
| --- | --- | --- |
| EI3-D6 | The signal cutoff and its relation to candle-finality latency and the entry cutoff. A decision made after a 14:25 signal bar's finality is received leaves no complete bar that opens after the decision and ends by a 14:30 entry cutoff. | PENDING |
| EI3-D7 | How many research signals one listing may produce in one session | PENDING |

EI3-D6 and EI3-D7 are first recorded here. EI3-D8 is approved above; there is no EI3-D9 or later.

> **Note, 2026-11-06 (EI-3B, owner approval).** The table and sentence above describe 2026-11-05. EI3-D6 and EI3-D7 are now **APPROVED** (owner, 2026-11-06), EI3-D9 is recorded and **APPROVED** on the same date, and EI3-D10 is recorded as **PENDING**. See [section 10](#10-ei-3b-owner-approval-record-2026-11-06).

### 5.2 EI-0 decisions still pending or proposed

Listed with the status recorded in the [product specification, section 14](Equity-Intraday-V1-Product-Specification.md#14-owner-decision-register):

- **PROPOSED:** EI0-D1 (market and currency), EI0-D2 (series), EI0-D5 (universe), EI0-D7 (signal, entry and exit schedule), EI0-D16 (data-rights gate).
- **PENDING:** EI0-D8 (initial strategy and control; see EI3-D4), EI0-D9 (intrabar ambiguity), EI0-D10 (model capital), EI0-D11 (risk parameters), EI0-D12 (cost schedule), EI0-D13 (price increment), EI0-D14 (entry representation), EI0-D15 (liquidity and price thresholds), EI0-D17 (corporate actions), EI0-D18 (surveillance exclusion), EI0-D19 (daily modeled loss limit), EI0-D20 (robustness minimums), EI0-D21 (revision materiality).

EI0-D7's proposed 14:25 / 14:30 / 15:00 IST schedule is **not** approved.

### 5.3 Session-evidence decisions still pending

CE-D8, CE-D9, CE-D10 and CE-D15 remain **PENDING** in the [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md), section 18.2.

### 5.4 Earlier approvals unchanged

- **EI0-D4**, `Symbol` ampersand support: approved by the owner on 2026-11-01 ([ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md)).
- **CE-D1 to CE-D7, CE-D11 to CE-D14 and CE-D16 to CE-D18**: approved by the owner on 2026-11-04 ([NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md), section 18.1). CE-D17 is the approved 09:15 to 15:00 IST research window.

## 6. Ambiguities recorded, not resolved

### 6.1 What EI3-D1 does not decide

EI3-D1 defines the instant of a **research decision**. It does not define these recommendation-level instants, which belong to EI-4 and EI-5 and stay undecided:

- the `issued_at` of an equity trade recommendation ([product specification, section 8.1](Equity-Intraday-V1-Product-Specification.md#81-minimum-recommendation-contract));
- its `eligible_from` ([section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states)), which the specification proposes as "the completion instant of the next bar";
- which bars may evidence an entry after a decision made later than the decision bar's interval end.

These must be decided consistently with EI3-D1, EI3-D6 and EI0-D7 before EI-4.

> **Note, 2026-11-07 (EI-4A, owner approval).** These instants are now decided by EI4-D1 and the revised EI0-D7 timing structure in the [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md#3-approved-decisions). EI3-D1 is unchanged.

### 6.2 Non-CAS close from 2026-08-03 to 2026-08-11

The [product specification, section 6.1](Equity-Intraday-V1-Product-Specification.md#61-facts-awaiting-official-validation), is corrected by a dated note. The note cannot state the non-CAS continuous close for 2026-08-03 to 2026-08-11, because that is CE-D15, which is pending. It records the gap instead of resolving it.

## 7. EI-3A implementation prerequisites and expected scope

### 7.1 Prerequisites

| Prerequisite | State |
| --- | --- |
| EI3-D1, EI3-D2, EI3-D3, EI3-D4, EI3-D5, EI3-D8, EI0-D3 and EI0-D6 approved | Met, 2026-11-05 |
| EI-2B immutable evidence snapshots and EI-2C synthetic fixtures merged | Met |
| An explicit, versioned synthetic calibration for the strategy | To be defined in EI-3A, under EI3-D5 |
| Rejection-reason names for the four EI3-D2 categories | To be defined in EI-3A, under EI3-D2 |

### 7.2 Expected EI-3A scope

The first EI-3A implementation is expected to:

- be **Application-only**, with no Core, Infrastructure, API or Web change;
- consume immutable EI-2B evidence snapshots, and nothing from the mutable generic daily OHLCV store;
- use only appropriate, finalized, acceptable five-minute candles;
- produce deterministic `LONG_CANDIDATE`, `NO_SIGNAL` or rejection decisions;
- preserve, in each decision, the snapshot identity, manifest fingerprint, reference provenance and policy versions it relied on;
- perform no broker execution;
- perform no paper fill;
- perform no entry, stop, target, capital, cost or position-size calculation;
- use no real provider and no unapproved production data.

EI-3A is not implemented by this record.

## 8. Production-readiness restrictions

Nothing in this record moves Equity Intraday V1 toward production use. Production use remains blocked at least by:

- the data-rights gate, EI0-D16, and an authorized real-data provider (EI-9);
- an approved real candle-finality source; the production finality policy approves none;
- an approved production listing-applicability source; the production source answers UNKNOWN for every listing;
- a corporate-action source (EI0-D17) and a surveillance-status source (EI0-D18);
- authoritative cost rates (EI0-D12) and price increments (EI0-D13);
- pending session-evidence decisions CE-D8, CE-D9, CE-D10 and CE-D15;
- robustness minimums (EI0-D20) and a revision materiality policy (EI0-D21).

## 9. Related documents

- [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md)
- [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md)
- [ADR-018 Futures V1 Research-to-Recommendation Generation Policy](../adr/ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md)
- [ADR-019 Symbol Ampersand Extension](../adr/ADR-019-Symbol-Ampersand-Extension.md)
- [Reference Implementation Registry](../engineering/registry/Reference-Implementations.md), RVO-001 `Symbol` v1.1

## 10. EI-3B owner approval record (2026-11-06)

**Scope: synthetic-only EI-3B, research decision scheduling.** This section records approvals given on **2026-11-06**. It does not rewrite sections 1 to 9: where they say a decision is pending, they describe the state on 2026-11-05.

### 10.1 Approved decisions

Each decision below is **APPROVED BY OWNER, 2026-11-06**, for synthetic-only EI-3B.

#### EI3-D6 — Research-only scheduling

**Approved rule.**

- EI-3B schedules research evaluations within the approved NSE cash-equity research window of 09:15 to 15:00 IST (CE-D17).
- The latest permitted signal candle is the one ending at **15:00 IST**. No signal candle ending after 15:00 IST may contribute.
- The research decision instant remains the immutable snapshot's `as_of` (EI3-D1).
- A snapshot may become evaluable after 15:00 IST, once the finality evidence for its candles arrives. A research decision made after the window is **not** an actionable intraday execution signal.
- EI-3B enforces no entry-dependent signal cutoff.
- No maximum provider finality latency is assumed.
- EI0-D7, the proposed execution schedule, stays **PROPOSED** and is deferred to later execution and risk design.

**Rationale.** A signal cutoff has meaning only together with an entry rule. A decision is made only after its signal candle's finality is received; and only bars that open at or after the decision can evidence an entry. So whether a 14:25 signal leaves any entry opportunity before 14:30 depends on the finality latency and on the entry rule, neither of which EI-3B decides.

#### EI3-D7 — Candidate frequency

**Approved rule.** For each unique combination of strategy identity, `ListingReference` and trading date:

- only the first qualifying `LONG_CANDIDATE`, ordered by the immutable snapshot's decision instant, may be labelled `CANDIDATE`;
- every later qualifying evaluation is labelled `REPEAT_QUALIFYING`, and keeps its full underlying research decision and evidence;
- `NO_SIGNAL` and `REJECTED` evaluations stay distinguishable from each other and from candidates;
- the versioned synthetic strategy configuration stays fixed for the whole session timeline. A configuration change within a timeline is rejected. It is never permission for a second candidate;
- EI-3B never withdraws an earlier candidate because of a later candle correction. Correction materiality and withdrawal stay deferred (EI0-D21).

**Rationale.** Once a breakout has occurred, most later snapshots also qualify. Labelling each of them a candidate would make the number of candidates depend on how often snapshots are issued, not on the market.

#### EI3-D9 — Conflicting evidence and watermark regression

**Approved rule.** EI-3B fails closed, and records no decision, when:

- distinct evidence manifests claim the same decision instant;
- a later decision instant carries a journal watermark lower than the previous timeline entry's;
- an out-of-order historical input would modify a decision already established at an earlier instant.

No winning snapshot is chosen by earlier issuance, higher watermark, input order, snapshot identity or any other implicit preference.

- An exactly repeated snapshot identity may be treated as an idempotent duplicate.
- Re-issued identical evidence at the same decision instant may also be treated as a duplicate, when the manifests are demonstrably identical, meaning their manifest fingerprints are equal.
- A conflicting same-instant manifest is refused explicitly.
- Existing timeline history is never overwritten.

**Rationale.** This keeps the EI-2A and EI-2B principle that no source revision, evidence set or snapshot is ever silently preferred.

### 10.2 Decisions still pending

These are **not** approved by this record:

| ID | Question | Status |
| --- | --- | --- |
| EI0-D7 | The 14:25 / 14:30 / 15:00 IST signal, entry and exit schedule | PROPOSED, deferred to later execution and risk design |
| EI3-D10 | When the latest ended candle is not yet final at the decision instant: reject the evaluation, as EI-3A does, or treat that candle as not yet required | PENDING |
| EI0-D8 | The initial strategy and its research control, beyond EI3-D4's synthetic approval | PENDING |

- EI3-D10 is first recorded here. EI-3B keeps EI-3A's strict evidence-completeness behaviour (EI3-D2). That EI-3A already rejects such evaluations is implementation, not an approval of EI3-D10.
- Under the strict behaviour, sustained finality latency of five minutes or more can prevent evaluations during the active research session: while candles keep ending, the latest ended candle is not yet final at any decision instant. After the window closes no further candle ends, so a post-window evaluation may still become possible once every required candle through 15:00 IST is final (EI3-D6). Such an evaluation is not an actionable intraday signal. This does not affect synthetic fixtures. EI3-D10 must be decided before real-data readiness.
- Every other decision listed as pending or proposed in section 5.2, and CE-D8, CE-D9, CE-D10 and CE-D15 (section 5.3), keeps its status.

### 10.3 Earlier approvals unchanged

- EI3-D1, EI3-D2, EI3-D3, EI3-D4, EI3-D5, EI3-D8, EI0-D3 and EI0-D6 stay **APPROVED** as recorded on 2026-11-05 (section 3).
- EI0-D4 (2026-11-01, [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md)) and the NSE session-evidence decisions and accepted facts approved on 2026-11-04 ([NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md), section 18) are unchanged.

### 10.4 Approved EI-3B scope

The EI-3B implementation is expected to:

- be **Application-only**, with no Core, Infrastructure, database, API or Web change;
- keep one immutable timeline per strategy identity, listing and trading date;
- reuse the EI-3A research evaluator and research decisions unchanged, over immutable EI-2B evidence snapshots;
- order evaluations deterministically by decision instant, with the EI3-D9 refusals;
- keep the synthetic configuration fixed throughout a timeline;
- label at most one `CANDIDATE` per timeline (EI3-D7);
- report duplicates and refusals as explicit outcomes;
- make every timeline entry immutable and fingerprinted;
- read no system clock;
- persist nothing;
- perform no broker execution, and model no entry, stop, target or exit.

**Timeline completeness.** The caller supplies the prior timeline. EI-3B cannot prove that a supplied timeline is complete, and the absence of a supplied timeline is never evidence that no earlier candidate exists. Authoritative persistence and completeness guarantees belong to EI-3C.

### 10.5 Restrictions unchanged

- Nothing here authorizes execution, a recommendation, a paper fill, production use, real market data or a real data provider.
- A `CANDIDATE` is a research label. It is not a recommendation, an order or a trade.
- The production-readiness restrictions in [section 8](#8-production-readiness-restrictions) all still apply.

## 11. EI-3C owner approval record (2026-11-07)

**Scope: synthetic-only EI-3C, authoritative research timeline persistence.** This section records approvals given on **2026-11-07**. It does not rewrite sections 1 to 10: each describes the state on its own date.

### 11.1 Approved decisions

Each decision below is **APPROVED BY OWNER, 2026-11-07**, for synthetic-only EI-3C.

#### EI3-D11 — Timeline establishment

**Approved rule.**

- A research timeline, together with its fixed strategy configuration, becomes authoritative when its first research timeline entry is successfully committed.
- An advancement that commits no entry persists no new timeline identity. This includes an advancement whose inputs are all refused or duplicates.

**Consequences.**

- No empty timeline identity is persisted merely because an evaluation was attempted.
- The first committed entry establishes the timeline's authoritative configuration.
- Once established, the configuration cannot change silently. An advancement under another configuration is refused (EI3-D7).
- Every later advancement continues from the stored authoritative timeline.

An uncommitted or in-memory timeline, including an EI-3B timeline that has not been committed, is not authoritative.

**Rationale.** Persisting an identity for an attempt that recorded nothing would fix a configuration on the strength of no research, and would leave an empty record standing for a timeline that holds no decision.

#### EI3-D12 — Concurrent commit authority

**Approved rule.**

- SQLite transaction serialization determines the order in which concurrent advancements of a research timeline are committed.
- Established entries are immutable.
- A writer whose loaded timeline head has become stale must reload the authoritative history and re-evaluate its submitted inputs against it.
- A later conflicting same-instant manifest, or a retroactive historical input, is refused explicitly under EI3-D9.

No advancement automatically:

- overwrites an existing entry;
- replaces a committed snapshot reference;
- selects a winner among market evidence;
- inserts an earlier decision retroactively;
- withdraws an earlier candidate;
- marks an established entry as contested.

An append-only contested-evidence annotation stays **deferred**.

**Clarification.** Commit order establishes the recorded history. It does not show that the first committed market evidence was objectively correct.

**Required guarantee.** Concurrent writers never produce two first candidates for one timeline, and never break a timeline's entry chain.

**Consequence.** Because a committed snapshot reference is never replaced, the equivalent snapshot committed first stays the entry, and a later equivalent snapshot is a duplicate of it (EI3-D9). Different valid first-commit histories may therefore carry different snapshot-specific fingerprints, while equivalent evidence still gives equivalent research.

**Rationale.** EI3-D9 refuses conflicting evidence that arrives together and never overwrites history. Under concurrency, conflicting evidence may arrive only after another writer has committed. Serialization makes that order explicit and recorded, rather than resolving it by an implicit preference.

#### EI3-D13 — Historical decision reproducibility

**Approved rule.** Every stored research decision is rehydrated and verified against its original immutable EI-2B evidence snapshot, using compatible, versioned historical evaluation behaviour. A successful load:

- verifies the original evidence snapshot;
- preserves its snapshot ID and manifest fingerprint;
- reproduces the original research decision;
- verifies the stored canonical decision document;
- verifies the decision fingerprint;
- verifies the timeline entry fingerprint;
- verifies the previous-entry chain;
- preserves the original strategy configuration and the relevant versions.

The load fails closed when compatible historical evaluation behaviour is unavailable, or when the reproduced decision differs from the stored one.

Nothing silently:

- recalculates an old decision with incompatible current logic;
- rewrites a historical decision;
- changes its strategy configuration;
- replaces its evidence references;
- migrates its fingerprint or canonical document without approval.

Any future change that alters historical decision representations or evaluation semantics needs an explicit, governed compatibility or migration decision.

**Current versions.** EI-3C keeps the existing fingerprint contracts unchanged: `eir1` research decisions (`northstar.equity-intraday-research-decision/1`), `eirt1` timeline entries (`northstar.equity-intraday-research-timeline-entry/1`), and the EI-2B `ecm1` manifest fingerprints and `ecs1` snapshot identities. As of 2026-11-07 Northstar contains no historical evaluator registry. The only compatible evaluation behaviour available is the current EI-3A evaluator for these versions; a stored decision it does not reproduce fails closed.

**Rationale.** Evidence snapshots are immutable, so re-deriving a decision from its snapshot shows that the stored decision was made from that evidence. Silently re-running different logic would rewrite research history.

### 11.2 Decisions still pending

These are **not** approved by this record:

| ID | Question | Status |
| --- | --- | --- |
| EI0-D7 | The 14:25 / 14:30 / 15:00 IST signal, entry and exit schedule | PROPOSED, deferred to later execution and risk design |
| EI3-D10 | When the latest ended candle is not yet final at the decision instant | PENDING |
| EI0-D8 | The initial strategy and its research control, beyond EI3-D4's synthetic approval | PENDING |
| EI0-D21 | Revision materiality: whether a correction may withdraw or restate a decision | PENDING |

- An append-only contested-evidence annotation is deferred (EI3-D12).
- Persisting refused or duplicate outcomes is not part of EI-3C and is not decided here.
- Every other decision listed as pending or proposed in section 5.2, and CE-D8, CE-D9, CE-D10 and CE-D15 (section 5.3), keeps its status.

### 11.3 Earlier approvals unchanged

- EI3-D1, EI3-D2, EI3-D3, EI3-D4, EI3-D5, EI3-D8, EI0-D3 and EI0-D6 stay **APPROVED** as recorded on 2026-11-05 (section 3).
- EI3-D6, EI3-D7 and EI3-D9 stay **APPROVED** as recorded on 2026-11-06 ([section 10.1](#101-approved-decisions)).
- EI0-D4 (2026-11-01, [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md)) and the NSE session-evidence decisions and accepted facts approved on 2026-11-04 ([NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md), section 18) are unchanged.
- EI3-D11 sets when EI3-D7's fixed configuration becomes authoritative, and EI3-D12 applies EI3-D9 to concurrent commits. Neither changes EI3-D7 or EI3-D9.

### 11.4 Approved EI-3C scope

The EI-3C implementation is expected to deliver Application contracts first and the Infrastructure implementation second.

**Application (`northstar-application`):**

- an Application-owned authoritative timeline persistence port and advancement use case;
- the EI-3A research evaluator, the EI-3B scheduler and timeline contracts, and the EI-2B immutable evidence snapshots, reused unchanged;
- bounded stale-head reload and re-evaluation (EI3-D12);
- exact retry and duplicate handling;
- recovery after an interrupted or unconfirmed commit;
- fingerprint-chain and historical replay verification on load (EI3-D13).

**Infrastructure (`northstar-infrastructure`):**

- an append-only SQLite research timeline store;
- a timeline identity unique per strategy identity, listing and trading date;
- a configuration fixed from the first committed entry (EI3-D11);
- atomic compare-and-append of new entries.

Throughout, EI-3C is synthetic-only research. It changes no Core model, API or Web, and adds no trading recommendation, order, fill, entry or exit logic, stop, target, sizing or transaction-cost calculation.

### 11.5 Restrictions unchanged

- Nothing here authorizes execution, a recommendation, a paper fill, production use, real market data or a real data provider.
- An authoritative `CANDIDATE` is still a research label. It is not a recommendation, an order or a trade.
- The production-readiness restrictions in [section 8](#8-production-readiness-restrictions) all still apply.
