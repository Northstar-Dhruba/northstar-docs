# Equity Intraday V1 EI-3 Decision Register

## Status

**Owner acceptance record, 2026-11-05. Scope: synthetic-only EI-3A research.**

- The Northstar product owner explicitly approved the eight decisions in [section 3](#3-approved-decisions) on **2026-11-05**: EI3-D1, EI3-D2, EI3-D3, EI3-D4, EI3-D5, EI3-D8, EI0-D3 and EI0-D6.
- No other decision is approved by this record. Every decision listed in [section 5](#5-outstanding-decisions) keeps its existing status.
- Nothing here authorizes production use, real market data, a real data provider, broker execution, paper fills or a trade recommendation.
- This record has not been through Architecture Review or Engineering Review.

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
