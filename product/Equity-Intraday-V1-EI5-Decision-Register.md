# Equity Intraday V1 EI-5 Decision Register

## Status

**Owner acceptance record, 2026-11-12. Scope: synthetic-only, LONG-only equity recommendation issuance and lifecycle (EI-5).**

- The Northstar product owner explicitly approved the ten decisions in [section 4](#4-approved-decisions) on **2026-11-12**: EI0-D9, EI4-D5 and EI5-D1 to EI5-D8.
- Every approval is limited to synthetic research ([section 2](#2-scope)). Nothing here authorizes production use, real market data, actual issuance, an actionable recommendation, an order, a paper or live fill, a position, broker execution or any trading.
- The architecture is recorded in [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), and the engineering design and acceptance matrix in the [EI-5 Issuance and Lifecycle Design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md). ADR-024 and the design are **proposed for owner review**. They are not approved by this record.
- **Nothing in EI-5 is implemented.** No EI-5 contract, use case, state machine, table or test exists.
- **Two conflicts and three open points were found while recording these decisions.** They are listed in [section 6](#6-conflicts-and-open-points-found-during-ei-5a) as OWNER DECISION REQUIRED. None is resolved by this record, and none is resolved silently.
- This record has not been through Architecture Review or Engineering Review.

> **Note, 2026-11-12 (owner approval of the EI-5A amendments).** After reviewing this record, the owner approved the following on 2026-11-12. They are recorded in [section 12](#12-owner-approval-of-the-ei-5a-amendments-2026-11-12).
>
> - **C-1:** the issuance-identity prefix `eiris1`.
> - **C-2:** an exclusive issued-history boundary for a new, versioned EI-4C assessment. It is a required implementation prerequisite before EI-5D.
> - **O-1:** gap-through recognition for the stop and target.
> - **ADR-024 and the EI-5 design,** accepted with these amendments.
> - **Sequencing:** EI-5B may begin after the EI-5A documentation PR is merged.
>
> O-2 and O-3 stay deferred. EI-4's formal closure stays pending independent CI verification. The bullets above record the state when this register was drafted and are not rewritten. **Nothing in EI-5 is implemented.**

> **Note, 2026-11-12 (EI5-D7 exact-boundary finality clarification).** The owner later approved a clarification of EI5-D7. When a snapshot's `as_of` equals the applicable boundary and the boundary bar is `MISSING` or `UNFINALIZED`, monitoring is temporarily blocked rather than terminal. It is recorded under [EI5-D7](#ei5-d7--session-outcomes-and-insufficient-evidence) and in [section 12.8](#128-ei5-d7-exact-boundary-finality-clarification). O-2 stays deferred. EI-5B is not merged.

> **Note, 2026-11-13 (EI-4C versioned compatibility governance).** The owner approved R-1, R-2 and N1 to N10 on 2026-11-13, with N10 in its stricter form: version-2 generation records `PROPOSED` only when `complete_through == t − 1 µs`. The architecture is [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md), and the record is [section 13](#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13).
>
> - **N10 case C.** The refusal classification for an overdeclared version-2 history was approved separately on 2026-11-13: `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, status `DEPENDENCY_UNAVAILABLE`, version 2 only, keyless attempt only ([section 13.4](#134-n10-case-c--the-overdeclared-history-refusal-approved-2026-11-13)).
> - **Not implemented.** The compatibility prerequisite is **NOT STARTED**, and EI-5D is not authorized.
> - **Milestone status corrected.** Section 13.6 corrects the stale milestone status: EI-5A, EI-5B, the EI5-D7 addendum and EI-5C are merged.
> - **Unchanged.** EI-4's formal closure stays pending independent CI verification.

> **Note, 2026-11-14 (EI-4C compatibility implemented and accepted with documented limitations).** The bullets of the 2026-11-13 note record the state on that date and are not rewritten. Since then:
>
> - **N9 sequencing amendment.** The owner approved implementing slices 3a and 3b as one combined Application PR ([section 14.1](#141-n9-sequencing-amendment-combined-slices-3a--3b-owner-approved-2026-11-14)). Packaging only; N1 to N8 and N10 are unchanged.
> - **Implemented.** Slices 1, 2, 3a + 3b and 4 have merged ([section 14.2](#142-merged-implementation)).
> - **Engineering acceptance.** PASS WITH DOCUMENTED LIMITATIONS, recorded in the [EI-4C Engineering Acceptance Record](Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md).
> - **Owner acceptance, 2026-11-14.** EI-4C versioned history compatibility is **ACCEPTED WITH DOCUMENTED LIMITATIONS** ([section 14.4](#144-owner-acceptance-of-slice-5-2026-11-14)). VC-04 stays NOT VERIFIED, and rollback or downgrade to a pre-3a Application against stored evaluator-v2 records is not authorized without separate verification.
> - **Unchanged.** EI-5D is **not authorized** and needs a separate owner milestone authorization. EI-4's formal closure stays pending independent CI verification.

> **Note, 2026-11-14 (EI-5D and EI-5E authorized).** The last bullet of the note above records the state before this authorization and is not rewritten. Later on 2026-11-14 the owner **authorized EI-5D (MODELED recommendation issuance) and EI-5E (synthetic end-to-end integration)**, combined in one northstar-application implementation PR, as a synthetic, Application-only milestone, with decisions D5D-1 to D5D-5 ([section 15](#15-owner-authorization-of-ei-5d-and-ei-5e-2026-11-14)). Only MODELED issuance is authorized. No SQLite issuance authority, ACTUAL issuance, execution, deployment or API and Web expansion is authorized; EI-6 needs its own authorization. **Nothing in EI-5D or EI-5E is implemented yet.**

> **Note, 2026-11-14 (EI-5D and EI-5E implemented; engineering acceptance recorded).** The note above records the state at authorization and is not rewritten. EI-5D and EI-5E are now merged (northstar-application #96, `9a68209`), and their engineering acceptance — PASS WITH DOCUMENTED LIMITATIONS, synthetic engineering acceptance — is recorded in the [EI-5D/EI-5E Engineering Acceptance Record](Equity-Intraday-V1-EI5D-EI5E-Modeled-Issuance-and-Synthetic-Integration-Engineering-Acceptance-Record.md) ([section 16](#16-ei-5d-and-ei-5e-implementation-and-engineering-acceptance-2026-11-14)). **Owner accepted, 2026-11-14: ACCEPTED WITH DOCUMENTED LIMITATIONS** ([section 16.3](#163-owner-acceptance-of-ei-5d-and-ei-5e-2026-11-14)). The issuance authority is an in-memory test double; EI-6 is not authorized.

> **Note, 2026-11-14 (EI-6A authorized; EI-6 staged).** The note above records the state before this authorization and is not rewritten. Later on 2026-11-14 the owner approved decisions **D6-1 to D6-6** ([section 17](#17-owner-authorization-of-ei-6a-2026-11-14)):
>
> - **EI-6A — Durable SQLite Recommendation Issuance and Declaration Authority: AUTHORIZED, NOT IMPLEMENTED.** Its storage design is the [EI-6A storage design addendum](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md).
> - **Staged delivery.** EI-6 is delivered as EI-6A, EI-6B (lifecycle persistence) and EI-6C (monitoring), each with its own authorization and acceptance. **EI-6B and EI-6C: NOT AUTHORIZED.**
> - **Timing clarified.** O-3 before EI-6B; O-2 and EI3-D10 before EI-6C; none blocks EI-6A. None is resolved.
> - **Acceptance gate.** EI-6A owner acceptance requires local Windows real-SQLite acceptance **and** independently verified Linux CI. **EI-6A owner acceptance: PENDING.**

> **Note, 2026-11-14 (EI-6A implemented; local engineering acceptance recorded).** The note above records the state at authorization and is not rewritten. The D6-5 Application prerequisite (northstar-application #97, `a17bd91`) and EI-6A (northstar-infrastructure #67, `63d2ab3`) are merged. Local engineering acceptance — **PASS WITH DOCUMENTED LIMITATIONS**, synthetic, local Windows real SQLite — is recorded in the [EI-6A Engineering Acceptance Record](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md) ([section 18](#18-ei-6a-implementation-and-engineering-acceptance-2026-11-14)). **D6-6 Linux CI is not verified, so EI-6A owner acceptance stays PENDING.** EI-6B and EI-6C stay not authorized.

> **Note, 2026-11-14 (EI-6A accepted).** The note above records the state before this decision and is not rewritten.
>
> - **D6-6 satisfied.** Successful Linux `python-ci.yml` runs are recorded for northstar-application #97 (run `43599364295`) and northstar-infrastructure #67 (run `43604917993`).
> - **Owner decision.** **EI-6A — ACCEPTED WITH DOCUMENTED LIMITATIONS, 2026-11-14** ([section 18.4](#184-owner-acceptance-of-ei-6a-2026-11-14)). The matrix is 25 VERIFIED and 1 VERIFIED WITH LIMITATION (HW-03 (S)).
> - **Unchanged.** EI-6B and EI-6C stay **NOT AUTHORIZED**.

> **Note, 2026-11-14 (EI-6B authorized; O-3 resolved).** The note above records the state before this authorization and is not rewritten. Later on 2026-11-14 the owner approved **D6B-1 to D6B-7** and three clarifications ([section 19](#19-owner-authorization-of-ei-6b-2026-11-14)):
>
> - **EI-6B — Durable Equity Recommendation Lifecycle Persistence: AUTHORIZED — NOT IMPLEMENTED — OWNER ACCEPTANCE PENDING.** The design is the [EI-6B Lifecycle Persistence Design addendum](../architecture/Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md).
> - **O-3: RESOLVED, option (b).** Late gap-filling evidence after terminal `INCOMPLETE_EVIDENCE` produces no lifecycle event. Product specification section 8.2's audited-event requirement for it is deferred to a future EI-8 restatement design, which is not authorized.
> - **Unchanged.** O-2 and EI3-D10 stay unresolved and are required before EI-6C, which stays **NOT AUTHORIZED**. No Core change is authorized.

## 1. Purpose

This register records the owner decisions that govern EI-5: the issuance of a synthetic recommendation from a verified EI-4 generation record, and its intraday lifecycle.

- It is the authoritative record for decisions with the `EI5-` prefix.
- It also records the owner's approval of EI0-D9 (intrabar ambiguity), which is listed in the [product specification, section 14](Equity-Intraday-V1-Product-Specification.md#14-owner-decision-register), and of EI4-D5 (pre-entry invalidation), which was first recorded in the [EI-4 Decision Register, section 5](Equity-Intraday-V1-EI4-Decision-Register.md#5-decisions-still-pending-or-deferred). The entries there point here through dated notes.
- It changes no decision recorded in the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md), the [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md) or any ADR.
- Implementation history is not approval, and this planning record is not implementation evidence.

## 2. Scope

| Item | Scope of this record |
| --- | --- |
| Product | Northstar Equity Intraday V1: NSE cash equities, INR, LONG only |
| Milestone | EI-5, recommendation issuance and lifecycle policy ([section 7](#7-approved-engineering-sequence)) |
| Data | Synthetic only: synthetic evidence, synthetic policies and synthetic timing values |
| Input | A stored EI-4D `PROPOSED` generation disposition, re-verified, and immutable EI-2B evidence snapshots |
| Outputs | Synthetic MODELED issued recommendations, an authoritative issued history, and append-only lifecycle events |
| Not in scope | Durable issuance and lifecycle persistence and monitoring runs (EI-6), API and Web (EI-7), evaluation and performance (EI-8), real data (EI-9), forward observation (EI-10), actual issuance, paper orders, fills, broker execution and production use |

Every approval below is limited to this scope unless its own text says otherwise.

## 3. Identifier mapping

The decisions were presented to the owner as ten numbered proposals in the EI-4 closure and EI-5A governance review of 2026-11-12, and approved with that numbering. The register identifiers are:

| Review item | Register identifier | Title |
| --- | --- | --- |
| 1 | EI0-D9 | Intrabar ambiguity |
| 2 | EI4-D5 | Pre-entry invalidation |
| 3 | EI5-D1 | Issuance authority |
| 4 | EI5-D2 | `RECORDED` semantics |
| 5 | EI5-D3 | Recommendation issuance identity |
| 6 | EI5-D4 | Authoritative history and recording watermark |
| 7 | EI5-D5 | Lifecycle transitions |
| 8 | EI5-D6 | Observation and entry ordering |
| 9 | EI5-D7 | Session outcomes and insufficient evidence |
| 10 | EI5-D8 | Synthetic lifecycle timing and terminality |

EI0-D9 and EI4-D5 keep their existing identifiers. No existing decision is renumbered.

## 4. Approved decisions

Each decision below is **APPROVED BY OWNER, 2026-11-12**, within the scope of [section 2](#2-scope), exactly as worded.

### 4.1 Synthetic example conventions

Every example in this register is **synthetic**. No value in it is an approved production price, cutoff, latency or schedule. The examples share one synthetic issued recommendation:

| Item | Synthetic value |
| --- | --- |
| Listing and session | One synthetic NSE listing, on one synthetic normal session; times are IST |
| Levels (LONG) | Entry zone `[100.00, 100.50]`, stop `99.00`, target `103.00` |
| `decision_as_of` | 09:41:00 (the EI-2B snapshot instant of the research decision) |
| Modeled `issued_at` | 09:41:30 (a synthetic 30-second modeled latency, derived under EI4-D9-C) |
| `eligible_from` | 09:45:00, the opening of the first full 5-minute grid bar at or after the modeled issuance |
| `latest_entry_bar_opens_at` | 11:00:00 (a synthetic timing-policy value) |
| Entry-window end | 11:05:00, the close of the bar that opens at 11:00 |
| `modeled_exit_at` | 15:00:00, the close of the bar 14:55 to 15:00 (a synthetic timing-policy value that coincides with the end of the approved research window) |

A bar is named by its interval, for example "the bar 10:15 to 10:20". It has an opening instant and a closing instant, and its candle is timestamped at its interval end (EI0-D6).

**The approved 09:15 to 15:00 IST research window (EI3-D6, CE-D17) is a research scheduling window. It is not a production entry, exit or trading authorization**, and nothing in this register converts it into one.

### EI0-D9 — Intrabar ambiguity

**APPROVED BY OWNER, 2026-11-12.**

- The order of events inside one completed bar is not knowable from OHLC data. Unknown intrabar ordering stays `UNKNOWN`.
- An ambiguous entry produces the terminal state `INDETERMINATE_ENTRY`.
- An ambiguous exit produces the terminal state `INDETERMINATE_EXIT`.
- Neither has an invented fill or exit price.
- The Futures `MODELED_STOP_FIRST` rule is **not** used for equity lifecycle facts.
- Conservative best and worst modeling belongs to later evaluation (EI-8; [specification section 10.3](Equity-Intraday-V1-Product-Specification.md#103-indeterminate-and-no-entry-outcomes)), never to factual lifecycle events.

**Acceptance example (synthetic).** An `ACTIVE` recommendation has stop 99.00 and target 103.00. The completed bar 10:15 to 10:20 has low 98.90 and high 103.10. With no authoritative ordering evidence, the result is terminal `INDETERMINATE_EXIT`, ordering `UNKNOWN`, with no exit price. It is not `CLOSED_TARGET` and not `CLOSED_STOP`.

**Supersedes.** The PENDING status of EI0-D9 in the [specification, section 8.3](Equity-Intraday-V1-Product-Specification.md#83-intrabar-ambiguity) and [section 14](Equity-Intraday-V1-Product-Specification.md#14-owner-decision-register), by dated notes there.

### EI4-D5 — Pre-entry invalidation

**APPROVED BY OWNER, 2026-11-12.**

- A stop-only touch while `WAITING_FOR_ENTRY` invalidates the setup before entry.
- The terminal state `CANCELLED_BEFORE_ENTRY` is recorded, with the reason `STOP_TOUCHED_BEFORE_ENTRY`.
- No activation and no hypothetical fill are recorded.
- The event is not a realized winning or losing trade.

**Approved clarification.** When the entry and the stop are both touched in the same bar and their ordering is unknown, the ambiguity rule (EI0-D9) takes precedence: the result is `INDETERMINATE_ENTRY`, not `CANCELLED_BEFORE_ENTRY`. Stop-before-entry ordering is never claimed without evidence.

**Acceptance examples (synthetic).**

- A `WAITING_FOR_ENTRY` recommendation sees the bar 09:50 to 09:55 with low 98.95 and high 99.80. The stop (99.00) is touched and the entry zone `[100.00, 100.50]` is not. The result is `CANCELLED_BEFORE_ENTRY`, reason `STOP_TOUCHED_BEFORE_ENTRY`, with no `ACTIVE` event.
- The bar 09:50 to 09:55 instead has low 98.90 and high 100.20. It touches both the entry zone and the stop. The result is `INDETERMINATE_ENTRY`, ordering `UNKNOWN`.

**Supersedes.** The PROPOSED and deferred status of EI4-D5 in the [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md#5-decisions-still-pending-or-deferred), by a dated note there. `CANCELLED_BEFORE_ENTRY` is added to the lifecycle vocabulary of the [specification, section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states), by a dated note; the table there is not rewritten. EI-4C continues to report the `PRE_ENTRY_INVALIDATION` control as not assessed for generation: EI4-D5 is a lifecycle rule, and EI-4 generation records are unchanged.

### EI5-D1 — Issuance authority

**APPROVED BY OWNER, 2026-11-12.**

- Proposal generation and recommendation issuance remain separate operations.
- Issuance requires an explicitly re-verified, stored `PROPOSED` disposition, verified through the existing stored-generation verification contracts (`VerifyStoredEquityRecommendationGenerationUseCase`, levels A to C of [ADR-021, section 10](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md#10-restart-and-historical-verification-ei4-d9-i)).
- EI-5 supports **MODELED issuance only**. ACTUAL issuance remains unauthorized.
- There is at most one issuance per generation key (`eirk1`).
- There is at most one issued recommendation per listing, session and strategy, even across different synthetic policy sets.
- `NOT_ELIGIBLE` and `UNRESOLVED` dispositions cannot be issued.
- Issued recommendations are non-actionable.
- Every control that EI-4C reported as not assessed stays explicitly not assessed on the issued recommendation.
- A mandatory **verified-issuer boundary** governs writes to the issuance authority. Arbitrary direct persistence is not a supported issuance path ([ADR-024, section 4](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md#4-issuer-verification-and-authorization)).
- EI-4's disposition model (`PROPOSED`, `NOT_ELIGIBLE`, `UNRESOLVED`) and every stored generation record are unchanged.

**Acceptance examples (synthetic).**

- Issuing the verified `PROPOSED` disposition K with proposal P records one MODELED issued recommendation.
- Issuing K again returns the stored issuance as equivalent and writes nothing.
- Issuing an `UNRESOLVED` or `NOT_ELIGIBLE` disposition is refused, and nothing is written.
- A second `PROPOSED` disposition, from another policy set, for the same strategy, listing and session is refused as a duplicate issuance.

### EI5-D2 — `RECORDED` semantics

**APPROVED BY OWNER, 2026-11-12.**

- `RECORDED` begins only after an authoritative issuance is durably recorded.
- An EI-4 generation disposition is never itself `RECORDED` (EI4-D9-J).
- A valid later eligible observation permits the transition to `WAITING_FOR_ENTRY`.
- Lifecycle state is reconstructed deterministically from evidence and events, without consulting a running clock.
- The modeled issuance instant (`issued_at`, event time) and the record's persistence (its recording sequence, recording order) stay distinct. A modeled issuance is recorded later in wall-clock time than its `issued_at`, and that recording never moves `issued_at`.

**Acceptance example (synthetic).** The issuance of the example recommendation is recorded at recording sequence 7. Its state is `RECORDED`, whatever the wall-clock time. The first evaluated EI-2B snapshot whose `as_of` is at or after `eligible_from`, for example one at 09:46:10, moves it to `WAITING_FOR_ENTRY`. Until such a snapshot is evaluated, it stays `RECORDED`.

### EI5-D3 — Recommendation issuance identity

**APPROVED BY OWNER, 2026-11-12.**

- A separate, versioned **issuance identity** is introduced, named `eiri1` in the approved proposal.
- It is derived deterministically, with SHA-256 under the existing Northstar canonical encoding conventions, from:
  - a versioned, domain-separated identity namespace;
  - the established generation key (`eirk1`);
  - the proposal fingerprint (`eirp1`);
  - the issuance kind.
- It is stable across restarts and process boundaries.
- Generation identity and issuance identity stay distinct.
- No second, incompatible serialization convention is introduced. The encoding is the length-prefixed SHA-256 convention of the existing `eirk1` generation key ([ADR-024, section 5](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md#5-issuance-identity)).

> **CONFLICT FOUND, 2026-11-12 (OWNER DECISION REQUIRED).** The prefix `eiri1` is already in use. `eiri1-sha256:` is the fingerprint of the EI-4C frozen generation inputs (format `northstar.equity-recommendation-generation-inputs/1`). Stored EI-4D records carry it, the Infrastructure schema constrains it (`inputs_fingerprint ... GLOB 'eiri1-sha256:?*'`), and ADR-022 names it as unchanged. Reusing it for the issuance identity would give two different identities one prefix. The substance of EI5-D3 is unaffected. Only the prefix needs an owner decision: see [conflict C-1](#c-1--the-approved-eiri1-prefix-is-already-in-use). Until it is decided, the EI-5 documents call it "the EI5-D3 issuance identity", and EI-5D cannot begin.

> **Resolution, 2026-11-12 (C-1 OWNER APPROVED).** The issuance identity prefix is **`eiris1-sha256:`**. A search of northstar-core, northstar-application, northstar-infrastructure, northstar-api, northstar-web and northstar-docs found no existing use of `eiris1` or of its namespace. The frozen-inputs fingerprint `eiri1-sha256:` is unchanged. The exact derivation is in [section 12.1](#121-c-1--the-eiris1-issuance-identity).

**Acceptance example (synthetic).** For one generation key, proposal fingerprint and the kind `MODELED`, the issuance identity is byte-identical in two separate processes and after a restart. Recording it twice yields one issuance.

### EI5-D4 — Authoritative history and recording watermark

**APPROVED BY OWNER, 2026-11-12.**

- A monotone `recording_sequence` is assigned atomically by the issuance authority.
- Issuance records are append-only.
- Explicit, append-only **session completeness declarations** establish `complete_through`.
- A recording watermark identifies a stable historical prefix of the authority.
- A new issuance cannot be backdated into a declared-complete interval.
- The completeness and no-backdating rules stay valid after restarts and concurrent writes.
- The issuance authority enforces these rules. Callers cannot declare an arbitrary trusted watermark.
- Missing authoritative history stays unknown. An empty history is never invented (EI4-D9-H; EI4-D9 clarification D).
- The authority is one database authority. Later Infrastructure work uses `BEGIN IMMEDIATE` transactions where the approved design requires them. **No persistence is implemented in EI-5A.**

**Acceptance example (synthetic).**

1. Issuance A (`issued_at` 09:41) is recorded at sequence 1, and issuance B (10:05) at sequence 2.
2. A declaration "complete through 10:30" is recorded at sequence 3.
3. Issuance C with `issued_at` 10:10 is refused as backdated. C with `issued_at` 10:40 is accepted at sequence 4.
4. The view at watermark 3 still lists exactly A and B, complete through 10:30.

> **CONFLICT FOUND, 2026-11-12 (OWNER DECISION REQUIRED).** EI-4C reads the issued history at the proposal's own modeled issuance instant and requires it to be complete through that instant. EI5-D4 refuses any issuance at or before a declared `complete_through`. Together, they refuse the issuance of every proposal generated against the authority. See [conflict C-2](#c-2--inclusive-completeness-at-the-modeled-issuance-blocks-issuance).

> **Resolution, 2026-11-12 (C-2 OWNER APPROVED).** EI5-D4 is unchanged. A new, versioned EI-4C assessment requires authoritative history complete **strictly before** the modeled issuance instant. It is a separate Application implementation prerequisite before EI-5D, and is not implemented. Existing EI-4C behaviour and stored records are unchanged. The exact boundary and contract are in [section 12.2](#122-c-2--the-exclusive-history-boundary).

### EI5-D5 — Lifecycle transitions

**APPROVED BY OWNER, 2026-11-12.**

The equity-only state model is:

| State | Terminal | Outcome class |
| --- | --- | --- |
| `RECORDED` | No | — |
| `WAITING_FOR_ENTRY` | No | — |
| `ACTIVE` | No | — |
| `CLOSED_TARGET` | Yes | Resolved |
| `CLOSED_STOP` | Yes | Resolved |
| `EXPIRED_BEFORE_ENTRY` | Yes | Resolved, no entry |
| `CANCELLED_BEFORE_ENTRY` | Yes | Resolved, no entry (EI4-D5) |
| `INDETERMINATE_ENTRY` | Yes | Indeterminate (EI0-D9) |
| `INDETERMINATE_EXIT` | Yes | Indeterminate (EI0-D9) |
| `SESSION_CUTOFF_REACHED` | Yes | Unresolved |
| `INCOMPLETE_EVIDENCE` | Yes | Unresolved |

Permitted transitions:

- `RECORDED` → `WAITING_FOR_ENTRY`;
- `WAITING_FOR_ENTRY` → `ACTIVE`, `INDETERMINATE_ENTRY`, `CANCELLED_BEFORE_ENTRY`, `EXPIRED_BEFORE_ENTRY` or `INCOMPLETE_EVIDENCE`;
- `ACTIVE` → `CLOSED_TARGET`, `CLOSED_STOP`, `INDETERMINATE_EXIT`, `SESSION_CUTOFF_REACHED` or `INCOMPLETE_EVIDENCE`.

Every terminal state is final. All lifecycle events are append-only and deterministically replayable.

- **`INVALIDATED` is outside the initial EI-5 scope.** Its listing-applicability source and its revision-materiality policy (EI0-D21) are unresolved.
- **No Futures states or types are imported.** `EXPIRED_AFTER_ACTIVATION`, `EXPIRED_WITH_INCOMPLETE_COVERAGE` and `MODELED_STOP_FIRST` have no equity counterpart (ADR-020, section 2).
- **State transitions and evidence events are distinct.** A `SOURCE_REVISION_RECORDED` event (EI5-D8) changes no state.

**Acceptance example (synthetic).** An event that would move an `ACTIVE` recommendation back to `WAITING_FOR_ENTRY` is refused. Any state-changing event after a terminal state is refused. Replaying the same events always yields the same state.

### EI5-D6 — Observation and entry ordering

**APPROVED BY OWNER, 2026-11-12.**

- Only bars whose **opening** instant lies in the closed interval `[eligible_from, latest_entry_bar_opens_at]` can establish an entry. Both ends are included.
- Bar intervals are processed in strictly ascending, contiguous grid order.
- Decisions use eligible, immutable EI-2B evidence snapshots, each at its own `as_of`.
- There is no future evidence and no look-ahead.
- Every lifecycle event references its supporting snapshot identity (`ecs1`) and interval.
- An interval that is `MISSING`, `UNFINALIZED`, `CONFLICTING_VALUES` or `UNRESOLVED_REVISIONS` (the EI-2A interval outcomes) prevents progress past it. Deficient intervals are never skipped.
- No entry is inferred from a bar that opens before `eligible_from` or after `latest_entry_bar_opens_at`.

**Acceptance examples (synthetic).**

- The bar 09:40 to 09:45 touches the entry zone. It opens before `eligible_from` (09:45) and is never evaluated.
- The bar 11:05 to 11:10 touches the entry zone. It opens after `latest_entry_bar_opens_at` (11:00) and activates nothing.
- The bar 09:55 to 10:00 is `MISSING` in the evaluated snapshot. Monitoring does not progress past 09:55, although the bars from 10:00 are `ACCEPTABLE`.

### EI5-D7 — Session outcomes and insufficient evidence

**APPROVED BY OWNER, 2026-11-12.**

- `SESSION_CUTOFF_REACHED` occurs for an `ACTIVE` recommendation with complete eligible coverage through the modeled exit boundary and no earlier terminal event. It is unresolved and has no exit price.
- `EXPIRED_BEFORE_ENTRY` occurs when the entry window has passed without activation, with the required coverage complete.
- Missing evidence temporarily blocks monitoring.
- Terminal `INCOMPLETE_EVIDENCE` is recorded only when the applicable evaluation boundary has been reached and the required coverage is still insufficient.
- Missing intervals and other deficiencies are listed explicitly.
- Unresolved outcomes are excluded from win and loss metrics.
- No observation after the modeled exit boundary is used.
- An absence of data alone never fabricates a completed trade.

**When a temporary block becomes terminal.** Monitoring is blocked, with no event, while the next interval it needs is deficient. The block becomes terminal `INCOMPLETE_EVIDENCE` only when an evaluated EI-2B snapshot satisfies both of these:

1. its `as_of` is at or after the applicable boundary:
   - the entry-window end (the close of the bar that opens at `latest_entry_bar_opens_at`), while `WAITING_FOR_ENTRY`;
   - `modeled_exit_at`, while `ACTIVE`;
2. an interval the state still needs, at or before that boundary, is deficient in that snapshot.

Until a snapshot satisfying both is evaluated, the state does not change. No clock is read.

**Acceptance examples (synthetic).**

- An `ACTIVE` recommendation has complete `ACCEPTABLE` coverage through the bar 14:55 to 15:00, and no level was touched. The result is `SESSION_CUTOFF_REACHED`, with no exit price.
- The bar 13:15 to 13:20 is `MISSING` in a snapshot whose `as_of` is 13:40. Monitoring is blocked and no event is written. A snapshot at 15:02 still shows it `MISSING`, so the result is `INCOMPLETE_EVIDENCE`, listing `MISSING` at 13:15 to 13:20.
- A bar ending at 15:05 is never read, even when it is present and touches a level.

> **Open point.** The terminal rule depends on which post-boundary snapshot is evaluated: no maximum finality latency is approved (EI3-D6), and EI3-D10 is pending. See [open point O-2](#o-2--terminal-incomplete_evidence-depends-on-the-evaluated-snapshot).

> **Clarification, 2026-11-12 (EI5-D7 exact-boundary finality, OWNER APPROVED).** Condition 1 above ("at or after the applicable boundary") is refined for one case. Suppose an evaluated snapshot's `as_of` **equals** the applicable boundary, and the bar closing at that boundary is `MISSING` or `UNFINALIZED`. Then monitoring stays temporarily blocked: no event is recorded, and the outcome is **not** terminal `INCOMPLETE_EVIDENCE`. A later snapshot decides by the rule above. See [section 12.8](#128-ei5-d7-exact-boundary-finality-clarification).
>
> - **The bar has ended.** At `closes_at == as_of`, EI-2A reports `NOT_ENDED` only when `closes_at > as_of`, so the bar has ended.
> - **It may already be final.** EI-2A refuses a candle received, or finality confirmed, strictly *before* the close, and accepts both *at* the close. The bar may therefore already be `ACCEPTABLE`, and is then evaluated normally: expiry, cutoff, entry, cancellation or a level reach.
> - **Not yet final is not deficient.** `MISSING` or `UNFINALIZED` for the boundary bar at that exact instant means only that its close has just been reached. It is not treated as a genuine deficiency.
> - **Unchanged:**
>   - `CONFLICTING_VALUES` and `UNRESOLVED_REVISIONS` of the boundary bar;
>   - any deficient interval earlier than the boundary bar;
>   - every evaluation whose `as_of` is after the boundary.
>
>   They follow the rule above.
> - **No latency policy.** No finality delay, grace period or settlement allowance is introduced. O-2 and EI3-D10 stay deferred.
>
> **Acceptance examples (synthetic):**
>
> | Boundary | `as_of` | Evidence | Result |
> | --- | --- | --- | --- |
> | Entry window, bar 11:00 to 11:05 | 11:05 | Boundary bar `UNFINALIZED` | Temporarily blocked; no event |
> | Entry window | 11:05 | Boundary bar `ACCEPTABLE` | Evaluated normally, for example `EXPIRED_BEFORE_ENTRY` when untouched |
> | Entry window | After 11:05 | Boundary bar `MISSING` or `UNFINALIZED` | `INCOMPLETE_EVIDENCE` (existing rule) |
> | Entry window | 11:05 | Bar 10:30 to 10:35 `MISSING`, an earlier interval | `INCOMPLETE_EVIDENCE`: no exception for an earlier interval |
> | Modeled exit, bar 14:55 to 15:00 | 15:00 | Boundary bar `MISSING` | Temporarily blocked; no event |
> | Modeled exit | 15:00 | Boundary bar `ACCEPTABLE` | Stop, target, both (`INDETERMINATE_EXIT`) or `SESSION_CUTOFF_REACHED`, as its prices show |
> | Modeled exit | After 15:00 | Boundary bar still `MISSING` or `UNFINALIZED` | `INCOMPLETE_EVIDENCE` (existing rule) |
> | Modeled exit | 15:00 | Boundary bar `CONFLICTING_VALUES` | `INCOMPLETE_EVIDENCE`: a genuine deficiency |

### EI5-D8 — Synthetic lifecycle timing and terminality

**APPROVED BY OWNER, 2026-11-12.**

- The evaluation instant is the `as_of` of the snapshot being evaluated.
- No historical decision depends on a runtime clock.
- There is no overnight carry-over: only snapshots of the recommendation's own trading date are evaluated.
- No eligible observation after `modeled_exit_at` is used.
- Once terminal, the lifecycle state is immutable.
- Later corrections to cited evidence are recorded as separate, append-only revision events (`SOURCE_REVISION_RECORDED`).
- Revision materiality stays `UNASSESSED` while EI0-D21 is pending.
- Later revisions never silently rewrite earlier lifecycle outcomes.
- Every synthetic timing boundary complies with the approved 09:15 to 15:00 IST research window. **The research window is not a production entry, exit or trading authorization.**

**Acceptance example (synthetic).** The recommendation became `CLOSED_TARGET` on the bar 11:15 to 11:20. A snapshot at 16:00 shows a correction to that bar. The state stays `CLOSED_TARGET`, and a `SOURCE_REVISION_RECORDED` event is appended, citing the original event and both evidence references, with materiality `UNASSESSED`.

## 5. Status of the decisions

| Item | Status |
| --- | --- |
| EI0-D9, EI4-D5, EI5-D1 to EI5-D8 | **APPROVED BY OWNER, 2026-11-12** |
| [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md) | **PROPOSED**, for owner review |
| [EI-5 Issuance and Lifecycle Design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md) | **PROPOSED**, for owner review |
| Conflicts C-1 and C-2; open points O-1 to O-3 | **OWNER DECISION REQUIRED** ([section 6](#6-conflicts-and-open-points-found-during-ei-5a)) |
| EI-5 implementation (EI-5B to EI-5E) | **NOT STARTED** |
| EI-5 acceptance scenarios | **PLANNED**; none executed |
| EI-6 | **NOT STARTED** |

> **Note, 2026-11-12.** The table above records the status when this register was drafted. The current status, after the owner's approval of the amendments, is in [section 12.7](#127-status).

## 6. Conflicts and open points found during EI-5A

These were found by reading the current code and documents while recording the approvals. Each needs an owner decision. Each names the first slice it blocks. None blocks the approval of the decisions above.

### C-1 — The approved `eiri1` prefix is already in use

- **Evidence.** `eiri1-sha256:` is the fingerprint prefix of `northstar.equity-recommendation-generation-inputs/1`:
  - `northstar-application/src/northstar_application/application_services/equity_recommendation_generation_evidence.py`, which maps the format to the prefix;
  - the EI-4D schema in `northstar-infrastructure/src/northstar_infrastructure/persistence/sqlite_equity_recommendation_generation_schema.py` (`inputs_fingerprint ... GLOB 'eiri1-sha256:?*'`);
  - [ADR-022, section 3](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md#3-canonical-binding-identities-dq3), and the [EI-4E record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#81-representative-proposed-scenario).
- **Impact.** Two different identities under one prefix make a stored value ambiguous, and the existing `GLOB` constraint would accept an issuance identity as an inputs fingerprint.
- **Options.**
  - (a) Keep the EI5-D3 derivation and assign an unused prefix. The recommended candidate is `eiris1-sha256:` ("recommendation issuance"), following the `eira1` and `eirat1` precedent of a short distinguishing suffix.
  - (b) Rename the existing inputs prefix. Rejected: it would change stored EI-4D evidence and ADR-022's explicit non-change.
- **Recommendation.** (a). The owner chooses the prefix.
- **Blocks.** EI-5D. EI-5B and EI-5C use no issuance identity.
- **Owner decision, 2026-11-12: APPROVED, option (a), prefix `eiris1-sha256:`** ([section 12.1](#121-c-1--the-eiris1-issuance-identity)).

### C-2 — Inclusive completeness at the modeled issuance blocks issuance

- **Evidence.**
  - EI-4C reads the issued history with `as_of` equal to the proposal's own modeled issuance (`generate_equity_trade_recommendation.py`, which derives `equity_modeled_issuance(decision_as_of, latency)` and passes it to `session_history`). It needs `complete_through` at or after that instant; otherwise it records `OPEN_RECOMMENDATION_STATE_UNAVAILABLE`, and no disposition results.
  - The existing contract is inclusive: `EquitySessionRecommendationHistory` lists "every recommendation issued in the session at or before" `complete_through` (`ports/equity_recommendation_references.py`).
  - EI5-D4 refuses an issuance whose `issued_at` is at or before a declared `complete_through`.
- **Impact.** A proposal generated against the authority needs a declaration with `complete_through` at or after its own `issued_at`. That declaration then refuses its issuance. Only proposals generated against labelled synthetic fixture histories exist today, and those are not authority evidence (EI4-D9-H).
- **Options.**
  - (a) **An exclusive assessment boundary.** The EI-4C assessment reads the history complete strictly **before** the modeled issuance. This needs a versioned EI-4C compatibility change to the history request and its frozen evidence format. Existing records and formats keep their meaning. EI5-D4 is unchanged.
  - (b) **Admit an issuance exactly at `complete_through`.** This weakens EI5-D4: a declared completeness would later be contradicted. Not recommended.
  - (c) **Make `complete_through` exclusive** in the history contract. This changes the meaning of an existing Application contract and of stored history evidence. Not recommended.
- **Recommendation.** (a), with a consequence for the owner to accept: two recommendations issued at exactly the same modeled instant do not see each other in their assessments. Same-listing exclusivity is unapproved, so it stays `UNRESOLVED` or not assessed.
- **Related design point.** The EI-5 design proposes that issuance also requires the proposal's stored history evidence to name the issuance authority and to reproduce from it at the recorded watermark ([design, section 7](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md#7-application-issuance-ei-5d)). The owner should confirm that check together with C-2.
- **Blocks.** EI-5D. EI-5B and EI-5C do not depend on it.
- **Owner decision, 2026-11-12: APPROVED, option (a).** A new, versioned EI-4C assessment requires history complete strictly before the modeled issuance. Its implementation is a prerequisite before EI-5D ([section 12.2](#122-c-2--the-exclusive-history-boundary)).

### O-1 — Gap-through touches are undecided

- **Evidence.**
  - The [specification, section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states), defines a touch as "the bar's low is at or below the level and its high is at or above it". It leaves open "whether a gap-through counts as a touch at the level".
  - The Futures lifecycle treats a stop as touched when `low <= stop` and a target when `high >= target`, so a gap through either counts. It treats a gap past the entry zone as no entry.
- **Impact.** A bar wholly below the stop while `ACTIVE` is not a touch under the strict definition. Monitoring would continue, and could end in `SESSION_CUTOFF_REACHED`, although price traded below the stop. While `WAITING_FOR_ENTRY`, the same question decides whether EI4-D5 cancels the setup.
- **Options.**
  - (a) Directional reach: stop reached when `low <= stop`; target reached when `high >= target`; entry zone touched when the bar's range intersects the zone. This is the Futures *pattern*, adopted without its types.
  - (b) The strict two-sided touch for every level.
- **Recommendation.** (a). It reaches the same state as (b) whenever the level lies inside the bar's range, and never ignores a traded-through level. No exit price is implied either way.
- **Blocks.** EI-5B's stop and target predicates.
- **Owner decision, 2026-11-12: APPROVED, option (a)**, for the stop and the target only. The entry zone keeps the intersection rule ([section 12.3](#123-o-1--gap-through-recognition)).

### O-2 — Terminal `INCOMPLETE_EVIDENCE` depends on the evaluated snapshot

- **Evidence.** Under EI5-D7, the first evaluated snapshot at or after the boundary that still shows a deficiency makes `INCOMPLETE_EVIDENCE` terminal. No maximum finality latency is assumed (EI3-D6), and EI3-D10 (trailing unfinalized candles) is pending.
- **Impact.** A snapshot taken at 15:01 may show the bar 14:55 to 15:00 `UNFINALIZED` only because its finality has not arrived yet. Evaluating it makes the outcome `INCOMPLETE_EVIDENCE`, while a snapshot at 15:10 might have resolved it. The lifecycle is deterministic for a given snapshot sequence, but the sequence is chosen by the caller (EI-6 monitoring).
- **Options.**
  - (a) Keep the rule as approved, and make snapshot selection an explicit EI-6 monitoring policy.
  - (b) Add a versioned synthetic evidence-settlement allowance after each boundary, before `INCOMPLETE_EVIDENCE` may become terminal.
- **Recommendation.** (a) for EI-5B to EI-5E, which use explicit synthetic snapshots. Decide (b), together with EI3-D10, before EI-6 monitoring or any real data.
- **Blocks.** Nothing in EI-5B to EI-5E. Before EI-6 monitoring.
- **Status, 2026-11-12: DEFERRED** to the governance stage before EI-6 monitoring or any real data. No resolution is recorded.
- **Not resolved by the exact-boundary clarification (2026-11-12).** The EI5-D7 clarification ([section 12.8](#128-ei5-d7-exact-boundary-finality-clarification)) covers only `as_of` exactly at the boundary. A snapshot even one microsecond later still decides by the rule as approved, and O-2 stays **DEFERRED**.
- **Timing clarification, 2026-11-14 (D6-3, owner approved).** The text above is preserved. "Before EI-6 monitoring" means before **EI-6C** (monitoring and operational snapshot policy), and O-2 is decided together with EI3-D10. O-2 does not block EI-6A or EI-6B. O-2 itself stays **DEFERRED** ([section 17.3](#173-d6-3--o-2-o-3-and-ei3-d10-timing)).

### O-3 — Late evidence after terminal `INCOMPLETE_EVIDENCE`

- **Evidence.** The [specification, section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states), says evidence that arrives after a terminal state, such as "a late bar that fills a gap", is recorded as an explicit audited event. EI5-D8 approves revision events for corrections to cited evidence. A late arrival is not a correction.
- **Impact.** No approved event kind records it. The state is unchanged either way (EI5-D8).
- **Options.**
  - (a) Approve a separate non-state reconciliation event kind for late evidence.
  - (b) Record nothing in EI-5. A later restatement (EI-8) reads the evidence directly.
- **Recommendation.** (b) for EI-5, with (a) decided before EI-6.
- **Blocks.** Nothing in EI-5B to EI-5E. Before EI-6.
- **Status, 2026-11-12: DEFERRED** to the governance stage before EI-6. No resolution is recorded, and no late-evidence event kind exists.
- **Timing clarification, 2026-11-14 (D6-3, owner approved).** The text above is preserved. Its "before EI-6" conflicted with "O-2 and O-3 before monitoring" in [section 12.5](#125-ei-5b-sequencing-and-the-revised-sequence), step 7, and the EI-5 design, section 11. O-3 concerns the persisted lifecycle-event vocabulary, so it must be decided before **EI-6B** (durable lifecycle-event persistence). It does not block EI-6A. O-3 itself stays **DEFERRED**, and the lifecycle-event vocabulary is unchanged ([section 17.3](#173-d6-3--o-2-o-3-and-ei3-d10-timing)).
- **Owner decision, 2026-11-14 (D6B-2): RESOLVED, option (b).** The entries above are preserved. Later evidence that fills the gap of a terminal `INCOMPLETE_EVIDENCE` produces no lifecycle event in EI-5 or EI-6, and the terminal state and every event stay unchanged. The audited-event requirement of the [specification, section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states) for such evidence is deferred to a future EI-8 restatement design. RV-04: terminal state unchanged; no new lifecycle event ([section 19.3](#193-d6b-2--o-3-resolved-option-b)).

## 7. Approved engineering sequence

The owner directed this sequence on 2026-11-12. **It is a planning baseline, not evidence that any slice is implemented.**

| Slice | Repository | Scope |
| --- | --- | --- |
| EI-5A | northstar-docs | Governance: this register, ADR-024, and the EI-5 design with its acceptance matrix |
| EI-5B | northstar-core | A pure, deterministic equity lifecycle model, `northstar_core.recommendations.equity.lifecycle` |
| EI-5C | northstar-application | Lifecycle evaluation over existing EI-2B snapshots, with synthetic test doubles |
| EI-5D | northstar-application | Issuance authority and use-case contracts, with deterministic identity, completeness and watermark behaviour |
| EI-5E | northstar-application | Synthetic end-to-end contract acceptance |
| EI-6 | northstar-infrastructure (and later operational integration) | Durable issuance and lifecycle persistence, the authoritative history, monitoring, and real SQLite acceptance |

**Prerequisites per slice.**

- **EI-5B** needs owner acceptance of ADR-024 and the design, and a decision on O-1.
- **EI-5D** also needs decisions on C-1 and C-2.
- **EI-4.** EI-5 depends on EI-4 ([specification, section 12.1](Equity-Intraday-V1-Product-Specification.md#121-refined-sequence)). EI-4 is engineering-accepted, and its formal closure is pending independent CI verification ([EI-4 Decision Register, section 15](Equity-Intraday-V1-EI4-Decision-Register.md#15-ei-4-closure-dispositions-2026-11-12)). Whether EI-5B may start before formal EI-4 closure is an owner decision. This record does not decide it.

> **Note, 2026-11-12.** The owner has since decided the sequencing and the prerequisites above. The revised sequence, which adds the EI-4C versioned compatibility prerequisite before EI-5D, is in [section 12.5](#125-ei-5b-sequencing-and-the-revised-sequence). The table and prerequisites above record the state when this register was drafted.

## 8. Relationship to earlier decisions

| Decision | Relationship |
| --- | --- |
| EI4-D1 | Unchanged. Entry evidence comes only from bars that open at or after issuance; EI5-D6 applies it to lifecycle |
| EI0-D7 | Unchanged: approved for synthetic policy structure only. EI-5 uses its latest entry-bar opening and modeled exit boundary. No production cutoff is approved |
| EI3-D6 | Unchanged. The 09:15 to 15:00 IST window stays a research window |
| EI4-D8 | **Unchanged.** Every issued same-session recommendation is still treated as open for exposure and exclusivity. EI-5 lifecycle state does not relax it; relaxing it needs a separate decision |
| EI4-D9-H, EI4-D9-J | Implemented in design by EI5-D1 to EI5-D4: the issued-recommendation authority, its watermark, and the reference back to `eirk1` and `eirp1` |
| EI4-D10, EI4-D11 | Unchanged. The verified-issuer boundary follows the EI4-D11 pattern without changing the generation-write boundary |
| EI0-D21 | **Pending.** Every revision assessment stays `UNASSESSED`, and no revision produces `INVALIDATED` |
| EI3-D10 | **Pending** ([O-2](#o-2--terminal-incomplete_evidence-depends-on-the-evaluated-snapshot)) |
| ADR-020 to ADR-023 | Unchanged. ADR-024 builds on them |

## 9. Decisions still pending

These are **not** approved by this record, and each keeps its status:

- C-1, C-2 and O-1 to O-3 ([section 6](#6-conflicts-and-open-points-found-during-ei-5a));
- `INVALIDATED`: a listing-applicability or suspension source, and EI0-D21 (candle-revision materiality);
- EI0-D11, risk parameters, including any exposure or simultaneous-open limit;
- the specification's proposed one-open-recommendation-per-listing threshold ([section 9.5](Equity-Intraday-V1-Product-Specification.md#95-duplicates-and-contradictions));
- EI0-D19, the daily modeled loss limit;
- EI3-D10, the trailing unfinalized candle policy;
- the relaxation of EI4-D8 once lifecycle state exists;
- ACTUAL issuance;
- the EI4-D11 invalid-record recovery mechanism;
- every other pending decision in the [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md#5-decisions-still-pending-or-deferred), and every real-data source, venue or market-reference decision.

> **Note, 2026-11-12.** C-1, C-2 and O-1 have since been approved ([section 12](#12-owner-approval-of-the-ei-5a-amendments-2026-11-12)). O-2 and O-3 stay deferred, and every other item above keeps its status.

## 10. Production-readiness restrictions

Every restriction in the [EI-4 Decision Register, section 7](Equity-Intraday-V1-EI4-Decision-Register.md#7-production-readiness-restrictions) still applies. This record does not authorize:

- actual (non-modeled) issuance, or any recommendation shown as actionable;
- broker or paper orders, fills, positions or exits;
- realized P&L, or claims of profitable trading;
- real market data, live data integrations or exchange-data licensing;
- production cutoffs, latencies or trading schedules;
- deployment, scheduler or monitoring-job changes;
- API or Web exposure;
- implementation of EI-5B or any later slice.

An `ACTIVE` state means only that the modeled entry condition was evidenced. It is not a position.

## 11. Related documents

- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md)
- [Equity Intraday V1 EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md)
- [Equity Intraday V1 EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md)
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md)
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)
- [Equity Intraday V1 EI-4E Synthetic Acceptance Record](Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md)
- [ADR-025: Equity Intraday V1 Versioned Evaluation and History Compatibility](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) (added 2026-11-13)
- [EI-4C Versioned History Compatibility Design v1.0](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md) (added 2026-11-13)
- [EI-6A Durable Issuance Authority SQLite Storage Design v1.0](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md) (added 2026-11-14)
- [EI-6A Durable SQLite Issuance Authority Engineering Acceptance Record](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md) (added 2026-11-14)
- [EI-6B Durable Recommendation Lifecycle Persistence Design v1.0](../architecture/Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md) (added 2026-11-14)

## 12. Owner approval of the EI-5A amendments (2026-11-12)

This section records approvals given on **2026-11-12**, after the owner reviewed sections 1 to 11. Sections 1 to 11 are not rewritten; dated notes there point here. Every approval is limited to the scope of [section 2](#2-scope). **Nothing here is implemented.**

### 12.1 C-1 — The `eiris1` issuance identity

**OWNER APPROVED, 2026-11-12.** The EI5-D3 issuance identity uses the prefix `eiris1-sha256:`.

**Derivation.** The identity is `eiris1-sha256:` followed by the lowercase hexadecimal SHA-256 digest of the length-prefixed parts below, in this order:

| # | Part | Value |
| --- | --- | --- |
| 1 | Versioned, domain-separated namespace | `northstar.equity-recommendation-issuance-key/1` |
| 2 | Generation key | The full `eirk1-sha256:` text of the verified disposition |
| 3 | Proposal fingerprint | The full `eirp1-sha256:` text of the verified proposal |
| 4 | Issuance kind | The `EquityIssuanceKind` value: `MODELED` in EI-5 |

- **Encoding.** Each part is encoded exactly as the existing generation key does it (`_length_prefixed` and `equity_recommendation_generation_key` in `equity_recommendation_generation_evidence.py`): its UTF-8 byte length in decimal ASCII, a colon, then its UTF-8 bytes. The digest is taken over the concatenation. No other serialization is introduced.
- **Normalization.** Every part is already canonical text, produced by existing derivations: lowercase ASCII prefixes and lowercase hexadecimal digests. Nothing is trimmed, case-folded or re-encoded.
- **Kind binding.** The issuance kind is an identity part, so a MODELED and an ACTUAL issuance of one proposal can never share an identity. ACTUAL issuance stays refused (EI5-D1).
- **Stability.** No clock, sequence, path, process or random value is a part. The identity is therefore identical across processes and restarts.
- **No content fingerprint.** As [ADR-024, section 5](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md#5-issuance-identity) records, no separate issuance-content fingerprint is added.

**Separation of identities.**

| Identity | Prefix | What it identifies | Status |
| --- | --- | --- | --- |
| Frozen generation inputs | `eiri1-sha256:` | The canonical EI-4C inputs document | Existing; **unchanged** |
| Generation key | `eirk1-sha256:` | One candidate under one synthetic policy set | Existing; unchanged |
| Proposal fingerprint | `eirp1-sha256:` | One canonical proposal document | Existing; unchanged |
| Issuance identity | `eiris1-sha256:` | One issuance, of one kind, of one verified proposal | Approved; **not implemented** (EI-5D) |

**Namespace verification, 2026-11-12.** The working trees of northstar-core, northstar-application, northstar-infrastructure, northstar-api, northstar-web and northstar-docs were searched, at the commits of the [design baseline](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md). Neither `eiris` nor `northstar.equity-recommendation-issuance` occurs outside the EI-5A documents.

- The fingerprint prefixes defined in source are: `ecm1`, `ecp1`, `ecr1`, `ecs1`, `eipd1`, `eir1`, `eira1`, `eirat1`, `eiri1`, `eirk1`, `eirp1`, `eirs1`, `eirt1`, `fdc1` and `fhd1`.
- The existing constraint `GLOB 'eiri1-sha256:?*'` does not match an `eiris1-sha256:` value.

**Unchanged.** No application code, schema or persisted identity changes. The `eiri1-sha256:` frozen-inputs fingerprint keeps its meaning.

### 12.2 C-2 — The exclusive history boundary

**OWNER APPROVED, 2026-11-12. A new, versioned EI-4C assessment must require authoritative issued history complete strictly before the modeled issuance instant.**

EI5-D4 is unchanged: the inclusive no-backdating rule, the monotone sequence, the declarations and atomic issuance uniqueness all stay as approved.

**Exact temporal boundary.**

- **Instants.** Let `t` be the modeled issuance, derived exactly as today (EI4-D9-C). The new assessment takes the history view as of `b = t − 1 µs`.
- **Resolution assumption.** A `PointInTime` carries at most six fractional digits (northstar-core `foundation/value_objects/point_in_time.py`), so the timestamp resolution is exactly one microsecond. No instant lies strictly between `b` and `t`, and "at or before `b`" is exactly "strictly before `t`". If `PointInTime` ever gained a finer resolution, this boundary would need a new contract version.
- **Use of the existing contract.** The boundary reuses the existing inclusive history contract unchanged:
  - `session_history(exchange, date, as_of=b)`;
  - `covers(b)`, meaning `complete_through >= b`;
  - `issued_by(b)`.

  The frozen evidence therefore lists exactly the issuances with `issued_at < t`.

  *Amended 2026-11-13 (N10).* For version-2 generation, `covers(b)` is necessary but not sufficient. `PROPOSED` needs `complete_through == b` exactly; a history complete beyond `b` is refused ([section 13.3](#133-n10--exact-history-completeness)).
- **Fail closed.** These fail closed exactly as today, with no disposition and an attempt only:
  - missing history (`None`);
  - unavailable or unverifiable history;
  - history incomplete at `b`;
  - history for another session.

  Nothing assumes an empty history.

**Consequence for issuance (no conflict, but a protocol constraint).**

- The assessment needs `complete_through >= t − 1 µs`. *Amended 2026-11-13 (N10): version-2 generation records `PROPOSED` only when `complete_through == t − 1 µs` exactly ([section 13.3](#133-n10--exact-history-completeness)).*
- Issuance needs `issued_at > complete_through`, that is `complete_through <= t − 1 µs`.
- Declarations never decrease, so a proposal is issuable only while its session's greatest declared `complete_through` is exactly `t − 1 µs`.

A deterministic workflow therefore issues in ascending modeled-issuance order:

1. declare the session complete through `t − 1 µs`;
2. assess, then issue every proposal at `t`;
3. only then declare further.

A declaration beyond `t − 1 µs` made before a proposal at `t` is issued makes that proposal permanently non-issuable (`BACKDATED_ISSUANCE`): it fails closed and is never repaired. This becomes a requirement of EI-6 monitoring. Recommendations issued at exactly the same modeled instant do not see each other in their assessments. Same-listing exclusivity is unapproved, so it stays unresolved or not assessed. EI5-D1's one issuance per listing, session and strategy is still enforced at issuance.

**Versioned compatibility contract: requirements for the prerequisite implementation.**

1. **New versions.** A new evaluator version, and new frozen-inputs and assessment format versions, distinct from `northstar.equity-recommendation-evaluator/1`, `…-generation-inputs/1` and `…-assessment/1`. The new issued-history evidence states its boundary explicitly, and the new version validates `as_of = t − 1 µs`, as version 1 today requires `as_of = t` (`EquityRecommendationInputs`).
   - *Amended 2026-11-13 (N3, N4, N8).* The exact new versions are:
     - `northstar.equity-recommendation-evaluator/2`;
     - `northstar.equity-recommendation-generation-inputs/2`;
     - `northstar.equity-recommendation-assessment/2`;
     - **`northstar.equity-recommendation-generation-attempt/2`**, omitted from the original list. Without it, a version-2 refusal would share a logical-attempt identity with a version-1 refusal that assessed a different instant.
   - The proposal, policy set, generation key and Core outcome stay on `/1`.
   - The existing prefixes are kept, because the `format` in the hash preimage identifies the version.
   - The explicit boundary is the `issued_history.history_boundary` field ([section 13.2](#132-the-approved-contract)).
2. **Existing records unchanged.** Every existing version-1 record, document, fingerprint (`eiri1`, `eira1`, `eirp1`) and key (`eirk1`) keeps its meaning, "complete through the modeled issuance". None is reinterpreted, migrated, relinked or re-fingerprinted.
3. **Historical replay is not weakened.** Version-1 records stay verifiable at levels A to C by the version-1 evaluator, and new records by the new one.
4. **The generation key is unchanged.** A candidate and policy set already recorded under version 1 keep that disposition. A new-version assessment under the same `eirk1` is a content conflict (EI4-D9-D), never a replacement.
   - *Amended 2026-11-13 (N5).* Precisely: it is a conflict under EI4-D9-D, classified by the existing `EVALUATOR_VERSION_CONFLICT` outcome, **not** `CONTENT_CONFLICT`.
   - The first stored disposition is preserved and is not adopted as the competing version's record.
   - A separate version-2 key needs a genuinely different approved policy-set version.
5. **Issuance accepts only the new version.** EI-5D issues only a disposition whose assessment is of the new version, and whose history evidence names the issuance authority and reproduces exactly from the authority's view at its recorded watermark. A version-1 disposition is refused as not issuable, and is preserved unchanged.
6. **Verifiable across concurrency and restart.** After a declaration at or beyond `b`, any issuance with `issued_at < t` is refused as backdated. The frozen evidence therefore stays exactly the authority's complete prefix at every later watermark ([ADR-024, section 7](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md#7-recording-sequence-watermark-and-completeness)).

**Contract points that remain open. They are identified here and not resolved; the prerequisite's own design must settle them.**

> **Resolved 2026-11-13.** R-1 and R-2 below are **RESOLVED** by owner decisions N1 and N3 ([section 13](#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13)) and recorded in [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md). The text below records them as they were identified on 2026-11-12.

- **R-1 — Two-version verification.** *RESOLVED 2026-11-13 (N1): a closed two-version table, dispatched by the recorded label; no registry.*
  - Today the verifier replays exactly one evaluator version (`verify_equity_recommendation_generation.py` refuses any other as incompatible, "Nothing is migrated").
  - ADR-021, section 10, says there is no historical evaluator registry, and that an unsupported format fails closed.
  - Requirement 3 needs both evaluator versions to verify. That is a bounded first step toward ADR-021's "historical evaluator registry" future consideration, so it needs explicit architecture approval in the prerequisite's design.
  - The alternative, letting version-1 records become incompatible, would weaken historical replay and is not acceptable under C-2.
- **R-2 — Fingerprint prefixes and storage.** *RESOLVED 2026-11-13 (N3): every existing prefix is retained, and no schema migration is planned.*
  - Canonical documents include their `format` field, so a new-version document has a distinct fingerprint even under the existing `eiri1` and `eira1` prefixes. That keeps the Infrastructure `GLOB` constraints valid.
  - New prefixes would need an Infrastructure schema change. *(Clarified 2026-11-13: SQLite cannot alter a CHECK constraint, so that change would be a table rebuild or parallel tables.)*
  - The prerequisite's design decides which.

**Status.** The approved correction is a **required implementation prerequisite before EI-5D**. It is not an approved implementation PR: it needs its own design and approval. It is **NOT STARTED**. **Existing EI-4C behaviour has not been updated**: today's EI-4C still reads history as of the modeled issuance.

*Note, 2026-11-13.* The prerequisite's design now exists and is owner approved: [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) and the [EI-4C Versioned History Compatibility Design v1.0](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md). Its implementation is **NOT STARTED**, and existing EI-4C behaviour is still unchanged ([section 13](#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13)).

### 12.3 O-1 — Gap-through recognition

**OWNER APPROVED, 2026-11-12, option (a) of [section 6](#o-1--gap-through-touches-are-undecided), for the stop and the target.**

| Level | Approved lifecycle predicate (exact decimal comparison) | Gap-through |
| --- | --- | --- |
| Stop (LONG) | Reached when the bar's `low <= stop` | Recognized: a range wholly below the stop reaches it |
| Target (LONG) | Reached when the bar's `high >= target` | Recognized: a range wholly above the target reaches it |
| Entry zone | Touched when `low <= zone.high` and `high >= zone.low` | **Not transferred.** A range wholly above or below the zone touches nothing and activates nothing |

- **Supported observations only.** Only an `ACCEPTABLE` interval of an eligible EI-2B snapshot, within the applicable range (EI5-D6), is evaluated.
- **No execution inference.** Reaching a level is a lifecycle observation. No stop-order fill, execution price, slippage or exit price is inferred or recorded. Modeled prices at bounds belong to EI-8.
- **Ambiguity precedence is preserved.**
  - While `ACTIVE`, a bar that reaches both the stop and the target is `INDETERMINATE_EXIT`, ordering `UNKNOWN` (EI0-D9).
  - While `WAITING_FOR_ENTRY`, a bar that touches the entry zone and reaches the stop is `INDETERMINATE_ENTRY`, never `CANCELLED_BEFORE_ENTRY` (EI4-D5 clarification).
  - `UNKNOWN` ordering, `INDETERMINATE_ENTRY` and `INDETERMINATE_EXIT` are unchanged.
- **The pre-entry rule.** EI4-D5's stop-only check uses the same approved stop predicate. A waiting bar wholly below the stop, touching neither the zone nor the target, is `CANCELLED_BEFORE_ENTRY`, reason `STOP_TOUCHED_BEFORE_ENTRY`.
  - **Scope note.** The owner's statement of the rule names an active recommendation's stop or target. Option (a), as approved, defines stop reach for every lifecycle use of the stop, and this record applies it to EI4-D5 on that basis (scenario PE-04).
  - If the owner intends gap-through recognition for `ACTIVE` recommendations only, PE-04's expected result becomes "no event", and only that scenario and this bullet change.
- **No Futures types** and no stop-first rule are imported. The predicates follow the Futures *pattern* only.

**Acceptance examples (synthetic).**

| State | Bar | Result |
| --- | --- | --- |
| `ACTIVE`, stop 99.00 | Low 98.00, high 98.50 | Stop reached: `CLOSED_STOP`; no fill or exit price |
| `ACTIVE`, target 103.00 | Low 103.50, high 104.00 | Target reached: `CLOSED_TARGET`; no fill or exit price |
| `ACTIVE`, stop 99.00, target 103.00 | Low 98.90, high 103.10 | `INDETERMINATE_EXIT`, `UNKNOWN` (unchanged) |
| `WAITING_FOR_ENTRY` | Low 98.20, high 98.70 | Stop reached, zone not touched: `CANCELLED_BEFORE_ENTRY` |
| `WAITING_FOR_ENTRY` | Low 100.80, high 101.20 | Zone not touched: no activation; no event |
| `WAITING_FOR_ENTRY` | Low 98.90, high 100.20 | Zone touched and stop reached: `INDETERMINATE_ENTRY` (unchanged) |

### 12.4 ADR-024 and design acceptance

**OWNER APPROVED, 2026-11-12:** [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md) and the [EI-5 Issuance and Lifecycle Design v1.0](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md), with the amendments of sections 12.1 to 12.3 incorporated and marked "amended 2026-11-12".

| Item | Status |
| --- | --- |
| Architecture (ADR-024) and engineering design | **APPROVED** (2026-11-12, with amendments) |
| Engineering implementation | **NOT STARTED** |
| Acceptance scenarios | **PLANNED**; none executed |
| Operational or production authorization | **NOT GRANTED** |

### 12.5 EI-5B sequencing and the revised sequence

**OWNER APPROVED, 2026-11-12.** EI-5B (the Core lifecycle implementation) may begin **after the EI-5A documentation PR is merged**.

- EI-4's formal milestone closure **remains pending independent CI verification** ([EI-4 Decision Register, section 15](Equity-Intraday-V1-EI4-Decision-Register.md#15-ei-4-closure-dispositions-2026-11-12)). This approval does not waive any CI requirement.
- It does not authorize EI-5D before its compatibility prerequisite.

**The sequence.**

| # | Step | Repository | Status |
| --- | --- | --- | --- |
| 1 | EI-5A — Docs governance | northstar-docs | This documentation; **not merged** |
| 2 | EI-5B — Core lifecycle model | northstar-core | Not started; may begin after step 1 is merged |
| 3 | EI-5C — Application lifecycle evaluation | northstar-application | Not started |
| 4 | EI-4C versioned history-boundary compatibility ([section 12.2](#122-c-2--the-exclusive-history-boundary)) | northstar-application | Not started. A **required dependency** before EI-5D; needs its own approved design and PR. It is not approved as an implementation PR by this record |
| 5 | EI-5D — Application issuance | northstar-application | Not started; requires step 4 |
| 6 | EI-5E — Application synthetic acceptance | northstar-application | Not started |
| 7 | EI-6 — Infrastructure issuance and lifecycle authority, and monitoring | northstar-infrastructure | Not started; O-2 and O-3 before monitoring |

The sequence is the approved planning baseline, not implementation evidence.

> **Note, 2026-11-13.** The status column records the state on 2026-11-12.
>
> - **Steps 1 to 3 have since merged** ([section 13.6](#136-milestone-status-correction)).
> - **Step 4** is now governed by [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) and its approved slices ([section 13.5](#135-approved-implementation-sequence-n9)). It is not started.
> - **Step 5 (EI-5D)** still requires step 4 to be accepted.

> **Note, 2026-11-14 (D6-2 and D6-3, owner approved).** Steps 4 to 6 are merged and accepted ([sections 14](#14-ei-4c-compatibility-implementation-and-engineering-acceptance-2026-11-14) and [16](#16-ei-5d-and-ei-5e-implementation-and-engineering-acceptance-2026-11-14)). Step 7 is preserved as written and is now delivered in three separately authorized stages ([section 17.2](#172-d6-2--staged-ei-6-delivery)):
>
> | Stage | Scope | Repository | Prerequisite | Status |
> | --- | --- | --- | --- | --- |
> | EI-6A | Durable SQLite issuance and declaration authority | northstar-application (D6-5 contract PR), then northstar-infrastructure | — | **AUTHORIZED, NOT IMPLEMENTED** |
> | EI-6B | Durable lifecycle-event persistence | northstar-application, then northstar-infrastructure | O-3 decided; own authorization | **NOT AUTHORIZED** |
> | EI-6C | Monitoring and operational snapshot policy | To be decided at authorization | O-2 and EI3-D10 decided; own authorization | **NOT AUTHORIZED** |
>
> *2026-11-14 (later):* EI-6A is accepted ([section 18.4](#184-owner-acceptance-of-ei-6a-2026-11-14)); O-3 is resolved and EI-6B is **AUTHORIZED, NOT IMPLEMENTED** ([section 19](#19-owner-authorization-of-ei-6b-2026-11-14)). EI-6C stays not authorized.

### 12.6 Deferred and pending

- **O-2** (post-boundary snapshot selection and finality latency) and **O-3** (late evidence after terminal `INCOMPLETE_EVIDENCE`) stay **DEFERRED** to their later governance stage. No resolution is recorded.
- R-1 and R-2 ([section 12.2](#122-c-2--the-exclusive-history-boundary)) belong to the compatibility prerequisite's design. *(Resolved 2026-11-13: [section 13](#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13).)*
- Every item of [section 9](#9-decisions-still-pending) other than C-1, C-2 and O-1 keeps its status, including:
  - EI0-D21, with every revision materiality `UNASSESSED` and no `INVALIDATED`;
  - EI0-D19;
  - EI0-D11;
  - the exclusivity threshold;
  - EI3-D10;
  - ACTUAL issuance;
  - the relaxation of EI4-D8;
  - the EI4-D11 recovery mechanism.
- Every control EI-4C reports as not assessed stays not assessed.

### 12.7 Status

| Item | Status |
| --- | --- |
| EI0-D9, EI4-D5, EI5-D1 to EI5-D8 | APPROVED (2026-11-12) |
| C-1 (`eiris1`), C-2 (exclusive boundary), O-1 (gap-through) | **APPROVED** (2026-11-12) |
| ADR-024 and the EI-5 design v1.0 | **APPROVED** (2026-11-12, with amendments) |
| EI-5B sequencing | **APPROVED**: after the EI-5A documentation PR is merged |
| EI-4C versioned compatibility prerequisite | Required before EI-5D; **NOT STARTED**; own design and approval needed (R-1, R-2 open). *2026-11-13: design approved and R-1 and R-2 resolved ([section 13](#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13)); implementation still NOT STARTED* |
| O-2, O-3 | **DEFERRED** |
| EI-5B to EI-5E, EI-6 | **NOT STARTED** |
| Acceptance scenarios (119) | **PLANNED** |
| EI-4 formal closure | **PENDING** independent CI verification |
| Operational or production authorization | **NOT GRANTED** |

### 12.8 EI5-D7 exact-boundary finality clarification

**OWNER APPROVED, 2026-11-12.** The clarification is recorded, with its acceptance examples, under [EI5-D7](#ei5-d7--session-outcomes-and-insufficient-evidence). In short:

- **When it applies.** At `as_of` exactly equal to the applicable boundary, a `MISSING` or `UNFINALIZED` boundary bar blocks temporarily and is never terminal. The boundary is the close of the last entry bar while waiting, and `modeled_exit_at` while active.
- **What is unchanged.** An `ACCEPTABLE` boundary bar is evaluated normally. `CONFLICTING_VALUES`, `UNRESOLVED_REVISIONS`, earlier deficient intervals and every later evaluation keep the EI5-D7 rule.

**The EI-2A contract it relies on.** EI-2A, in `equity_intraday_candle_validation.py`:

| Rule | Comparison |
| --- | --- |
| `NOT_ENDED` | Only when `closes_at > as_of` |
| A candle received too early is refused | Only when received strictly before the close |
| Finality confirmed too early is refused | Only when confirmed strictly before the close |
| Receipt and finality must be visible | Both at or before `as_of` |

A candle received and confirmed final exactly at its close is therefore `ACCEPTABLE` at `as_of == closes_at`. A statement to the contrary in the EI3-D1 rationale is corrected by a dated erratum in the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md#ei3-d1--research-decision-instant). EI3-D1 itself is unchanged.

| Item | Status |
| --- | --- |
| EI5-D7 exact-boundary finality clarification | **APPROVED** (2026-11-12) |
| Every other EI-5 decision, C-1, C-2, O-1, R-1, R-2 | Unchanged |
| O-2, O-3, EI3-D10 | **DEFERRED** (unchanged) |
| EI-5B (Core lifecycle) | Not merged. Local engineering verification is not CI evidence |
| Acceptance scenarios (119) | **PLANNED** |
| EI-4 formal closure | **PENDING** independent CI verification |
| Operational or production authorization | **NOT GRANTED** |

## 13. Owner approval of the EI-4C versioned compatibility governance (2026-11-13)

**OWNER APPROVED, 2026-11-13.**

- The owner approved R-1 and R-2, the two contract points the C-2 correction left open ([section 12.2](#122-c-2--the-exclusive-history-boundary)), and decisions N1 to N10.
- N10 is approved in a **stricter form** than the architecture review originally recommended. It supersedes that recommendation.
- The architecture is [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md), which amends [ADR-021, section 10](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md#10-restart-and-historical-verification-ei4-d9-i) by a dated pointer.
- The engineering specification and its 52 PLANNED scenarios are in the [EI-4C Versioned History Compatibility Design v1.0](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md).

**Nothing here is implemented.** Today's EI-4C supports only evaluator version 1 and reads history as of the modeled issuance.

### 13.1 Approved decisions N1 to N10

| ID | Decision | Status |
| --- | --- | --- |
| N1 (R-1) | Exactly two governed evaluator versions, `northstar.equity-recommendation-evaluator/1` and `/2`, in one closed version table. Verification dispatches on the recorded label. Unknown labels are incompatible, and missing or mismatched labels are invalid. No registration API or plugin architecture; a third version needs another governance amendment | **APPROVED** |
| N2 | ADR-025, accepted 2026-11-13, amends ADR-021 section 10 by a dated pointer; ADR-021 is not replaced | **APPROVED** |
| N3 (R-2) | Retain the prefixes `eiri1`, `eira1`, `eirat1`, `eirk1`, `eirp1` and `eirs1`. A prefix names a fingerprint family; the document's `format`, inside the hash preimage, names the version. No schema migration is planned, subject to mixed-version SQLite acceptance | **APPROVED** |
| N4 | `northstar.equity-recommendation-generation-attempt/2`, so version-2 attempts never share an identity with version-1 attempts | **APPROVED** |
| N5 | A cross-version collision under one `eirk1` preserves the first disposition and returns the existing `EVALUATOR_VERSION_CONFLICT`. The competing record is not adopted and the conflict is not reclassified as `CONTENT_CONFLICT`. A separate version-2 key needs a different approved policy-set version | **APPROVED** |
| N6 | Mandatory Application label-format pairing at construction, decoding, verification and authorized recording. A database-level pairing trigger is **DEFERRED** | **APPROVED** |
| N7 | A keyword-only evaluator selection at the generation entry point, constrained to the closed set, defaulting to `/1`. EI-5D composition must select `/2`. Version 1 stays supported but is not issuable | **APPROVED** |
| N8 | Version-2 frozen inputs carry an explicit canonical history-boundary field, and take history as of `b = t − 1 µs` on the existing microsecond-resolution `PointInTime`. Version 1 keeps `as_of = t` | **APPROVED** |
| N9 | The staged sequence of [section 13.5](#135-approved-implementation-sequence-n9) | **APPROVED** |
| N10 | Version-2 generation may record `PROPOSED` only when `complete_through == b` ([section 13.3](#133-n10--exact-history-completeness)) | **APPROVED** (stricter form) |

### 13.2 The approved contract

| Document | Version 1 | Version 2 |
| --- | --- | --- |
| Evaluator label | `northstar.equity-recommendation-evaluator/1` | `northstar.equity-recommendation-evaluator/2` |
| Frozen inputs (`eiri1`) | `northstar.equity-recommendation-generation-inputs/1` | `northstar.equity-recommendation-generation-inputs/2` |
| Assessment (`eira1`) | `northstar.equity-recommendation-assessment/1` | `northstar.equity-recommendation-assessment/2` |
| Generation attempt (`eirat1`) | `northstar.equity-recommendation-generation-attempt/1` | `northstar.equity-recommendation-generation-attempt/2` |
| Proposal (`eirp1`) | `northstar.equity-recommendation-proposal/1` | Unchanged (`/1`) |
| Policy set (`eirs1`) | `northstar.equity-recommendation-policy-set/1` | Unchanged (`/1`) |
| Generation key (`eirk1`) | `northstar.equity-recommendation-generation-key/1` | Unchanged (`/1`) |
| Core outcome | `northstar.equity-recommendation-core-outcome/1` | Unchanged (`/1`) |
| History `as_of` | `t` | `b = t − 1 µs` |
| History boundary | Implicit, inclusive through `t` | `issued_history.history_boundary = "STRICTLY_BEFORE_MODELED_ISSUANCE"`, with `as_of = complete_through = b` |

**Fixed for every version-1 record:**
- the evaluation instant, frozen inputs, formats, fingerprints, key, evaluator label, replay behaviour and persisted identities;
- none is rehashed, converted, relabelled or made issuable.

**The version-1 freeze.** Golden vectors for `eiri1`, `eira1`, `eirp1` and `eirat1` are a **mandatory prerequisite**. No fixed reference vector pins them today. They are generated from the unchanged current code before any version-2 behaviour is introduced.

**Unchanged:**
- `eiris1` (C-1) remains the issued-recommendation identity, distinct from the `eiri1` frozen-inputs family;
- the generation-key derivation, the proposal identity and the policy-set identity.

### 13.3 N10 — Exact history completeness

Let `t = modeled_issued_at` and `b = t − 1 µs`. Version-2 generation may record `PROPOSED` only when `complete_through == b`, and not merely `>= b`.

| Case | History | Behaviour |
| --- | --- | --- |
| A | `complete_through == b` | The boundary condition is satisfied. `PROPOSED` only if every other approved check passes. Necessary, not sufficient, for issuance |
| B | `complete_through < b` | No `PROPOSED`. Existing reason `RECOMMENDATION_HISTORY_INCOMPLETE`, status `DEPENDENCY_UNAVAILABLE`. Attempt only, with no key and no disposition |
| C | `complete_through > b` | No `PROPOSED`. Fail closed. Reason `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, status `DEPENDENCY_UNAVAILABLE`, version 2 only (**approved 2026-11-13**, [section 13.4](#134-n10-case-c--the-overdeclared-history-refusal-approved-2026-11-13)). Keyless attempt only, with no disposition, so the generation key is not consumed |
| D | Missing, unavailable, unverifiable, or another session's history | No `PROPOSED`. Existing reason `OPEN_RECOMMENDATION_STATE_UNAVAILABLE`, status `DEPENDENCY_UNAVAILABLE`. Attempt only |

**Atomicity.** The generation-time boundary does not replace the issuance authority's check. The authority must re-verify history and no-backdating inside its own serialized write transaction (EI5-D4). If declarations advance after generation and before issuance, issuance fails closed. Neither issuance nor a declaration scheduler is implemented here.

### 13.4 N10 case C — The overdeclared-history refusal (approved 2026-11-13)

**OWNER APPROVED, 2026-11-13.** This was approved separately from N1 to N10, on the same date.

Let `t = modeled_issued_at` and `b = t − 1 µs`. When `complete_through > b`:

| Item | Approved value |
| --- | --- |
| Reason | `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED` |
| Status | `DEPENDENCY_UNAVAILABLE` |
| Evaluator applicability | `northstar.equity-recommendation-evaluator/2` only |
| Persistence | Record an attempt only. The attempt is keyless (`generation-attempt/2`, no generation key) |
| Disposition | No `PROPOSED` disposition, and no disposition of any kind |
| Generation key | Not consumed |
| Issuance | No authoritative issuance record is created |
| Declarations | No completeness declaration is backdated or reversed |
| Identity | Append-only and deterministic attempt-identity semantics are preserved |

**Why this classification.**
- **Accuracy.** The existing vocabulary (`equity_trade_recommendation_eligibility.py`) had no accurate reason. `RECOMMENDATION_HISTORY_INCOMPLETE` means complete only through an instant **before** the required one: it remains case B and is never used for case C.
- **Precedent.** The status follows `POLICY_REFERENCE_CONFLICT`, a permanent condition under `DEPENDENCY_UNAVAILABLE` that callers distinguish by reason, not status.
- **No schema change.** No new status is introduced, so the SQLite attempts table's status check needs no change.

**Unchanged.**
- Version-1 behaviour is unchanged.
- The recorder's outcomes are unchanged.
- The decision is **not implemented**; the reason does not yet exist in Application code (slice 3b).

Details are in the [compatibility design, section 7](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md#7-n10-exact-completeness).

### 13.5 Approved implementation sequence (N9)

| Slice | Repository | Scope | Prerequisites |
| --- | --- | --- | --- |
| 0 | northstar-docs | ADR-025, these amendments, the compatibility design and its acceptance matrix | — |
| 1 | northstar-application | Version-1 freeze: golden vectors, pinned canonical documents, conflicting future-evaluator test labels replaced. No production behaviour change | Slice 0 merged |
| 2 | northstar-infrastructure | Test hygiene: conflicting labels replaced; a version-1 persisted-row replay fixture. Production unchanged | Slice 0 merged |
| 3a | northstar-application | Two-version verification: version table, versioned formats, decoder pairing, version-aware records, verifier dispatch. No version-2 generation | Slices 1 **and** 2 merged |
| 3b | northstar-application | Version-2 generation: explicit selection, `/2` inputs at `b`, N10, `/2` assessments and attempts, typed non-`PROPOSED` outcomes. No issuance | 3a merged |
| 4 | northstar-infrastructure | Mixed-version acceptance on real SQLite. No schema migration unless an independently governed blocker is found | 3b merged |
| 5 | northstar-docs | Acceptance record, limitations, CI evidence where available | Slice 4 merged |
| 6 | northstar-application | EI-5D issuance, separately scoped and authorized | Slice 5 accepted |

**This record does not authorize EI-5D.**

**The test label `northstar.equity-recommendation-evaluator/2`.** Existing Application and Infrastructure tests use it as an unsupported "future" label, and it is now reserved for the approved version 2. Slices 1 and 2:
- replace it with a label shown to be unused and unapproved;
- keep every unknown-version refusal assertion.

The compatibility design, section 13, records the coordination needed for tests whose premise slice 3a makes impossible to construct.

### 13.6 Milestone status correction

Earlier entries describe these milestones as not merged or not started. On 2026-11-13 the local `develop` histories show them merged:

| Milestone | Repository | PR | Merge commit | Change commit |
| --- | --- | --- | --- | --- |
| EI-5A governance | northstar-docs | #57 | `0c8996c` | `a2fd857` |
| EI-5B Core lifecycle | northstar-core | #39 | `ccff26e` | `9e06d1e` |
| EI5-D7 exact-boundary addendum | northstar-docs | #58 | `498956e` | `14dbaf4` |
| EI-5C Application synthetic lifecycle evaluation | northstar-application | #93 | `75c6a35` | `f29dbc1` |

**What this correction is:**
- a status correction only, from merge history;
- not an acceptance record: no EI-5B or EI-5C acceptance record exists in Docs, and no independent CI evidence is recorded;
- EI-5C remains synthetic and non-authoritative;
- it does not close EI-4.

The earlier entries stay as recorded.

### 13.7 Unchanged

- **Decisions.** Every EI-5 decision, C-1, C-2 (with N10 adding the exact-completeness rule to version-2 generation) and O-1 are unchanged, as are the EI5-D7 clarification and the EI-5B and EI-5C lifecycle decisions.
- **Issuance requirements.** These are unchanged:
  - MODELED only; generation is not issuance;
  - version-1 proposals are non-issuable;
  - version-2 verification is required;
  - `eiris1`;
  - one issuance per generation key, and at most one per (strategy, listing, trading date);
  - no backdating; explicit declarations; stable watermarks;
  - append-only history;
  - unknown or unverifiable history fails closed.

  Application pre-checks are never authoritative.
- **EI4-D8** keeps its conservative open-state treatment. A later modeled issuance in the same session may be `EVALUATION_UNRESOLVED`. It is not relaxed.
- **Deferred.** O-2, O-3, EI3-D10, EI0-D21 and EI0-D19 stay deferred. ACTUAL issuance stays unauthorized. No production recommendation-history authority exists.

### 13.8 Status

| Item | Status |
| --- | --- |
| R-1, R-2 | **RESOLVED** (2026-11-13) |
| N1 to N10 | **APPROVED** (2026-11-13); N10 in its stricter form |
| ADR-025 | **ACCEPTED** (2026-11-13) |
| EI-4C Versioned History Compatibility Design v1.0 | Approved governance; implementation not started |
| N10 case-C refusal classification | **APPROVED** (2026-11-13): `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED` / `DEPENDENCY_UNAVAILABLE`; version 2 only; keyless attempt only; not implemented |
| SQLite label-format pairing trigger | **DEFERRED** |
| Existing SQLite schema compatibility | **EXPECTED, NOT VERIFIED** (slice 4) |
| Slices 1 to 5 | **NOT STARTED** |
| Compatibility acceptance scenarios (52) | **PLANNED** |
| EI-5 acceptance scenarios (119) | **PLANNED** |
| EI-5D | **NOT AUTHORIZED**; requires slice 5 |
| EI-4 formal closure | **PENDING** independent CI verification |
| Operational or production authorization | **NOT GRANTED** |

> **Note, 2026-11-14.** The table above records the status on 2026-11-13 and is not rewritten. The current status is in [section 14.5](#145-status).

## 14. EI-4C compatibility implementation and engineering acceptance (2026-11-14)

This section records an **owner-approved sequencing amendment** (section 14.1), the **engineering acceptance evidence** of the merged EI-4C compatibility slices, and the **owner's acceptance of slice 5** on 2026-11-14 (section 14.4). The evidence is in the [EI-4C Versioned History Compatibility Engineering Acceptance Record](Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md). The acceptance record is not an owner approval, and this section does not rewrite section 13.

### 14.1 N9 sequencing amendment: combined slices 3a + 3b (owner approved, 2026-11-14)

**OWNER APPROVED, 2026-11-14,** before the combined implementation began.

N9 (section 13.5) approved two Application slices: 3a, two-version verification, and 3b, version-2 generation, with 3b requiring 3a merged. That approval stands as the governance of record for the slices' content. The owner subsequently approved delivering both as **one Application feature: Combined Slices 3a + 3b — Closed Two-Version Verification and Version-2 Generation**.

- **What changed.** PR packaging and workflow only: one branch, one implementation commit (`713954e`), one PR (#95), instead of two PRs in sequence.
- **What stayed binding.** Every decision N1 to N8 and N10, the case-C classification, the scope of 3a and of 3b, their gates (version-1 goldens unchanged; Application suite green; the full Infrastructure suite run against the Application head) and every acceptance scenario assigned to them. No decision is reinterpreted.
- **Engineering basis for the packaging change.**
  - Both slices change the same Application modules: decoding, records, evidence, the verifier, the recorder and the generator.
  - Until 3b, version-2 verification could be exercised only on hand-resealed documents; together, it is tested on genuine version-2 records.
  - The label-format pairing that 3a introduces needed one coordinated Infrastructure test change, which a single Application PR required only once.
- **Prerequisites, as executed** (local merge history, IST):
  - slices 1 (#94, 01:08) and 2 (#64, 02:06) merged before the combined implementation commit (04:53);
  - the Infrastructure evaluator-set correction (#65, 03:42) merged before the combined Application PR (#95, 13:45). It accepts exactly `{/1}` or `{/1, /2}`, so it satisfied the design's cross-repository constraint before, not after, the Application merge;
  - the combined Application PR merged before slice 4 (#66, 14:30).
- **Unchanged.** Slices 0, 1, 2, 4, 5 and 6 and their prerequisites; slice 6 (EI-5D) still requires slice 5 accepted.

The original N9 row and section 13.5 are kept as approved on 2026-11-13.

### 14.2 Merged implementation

| Slice | Repository | PR | Change commit | Merge commit |
| --- | --- | --- | --- | --- |
| 1. Version-1 golden-vector freeze | northstar-application | #94 | `a867248` | `ba79805` |
| 2. Test hygiene and version-1 persisted replay | northstar-infrastructure | #64 | `7cfd4d8` | `efe2d7b` |
| Evaluator-set transition correction | northstar-infrastructure | #65 | `dedd36e` | `d8a911f` |
| Combined 3a + 3b | northstar-application | #95 | `713954e`, `9ccfd77` | `d908674` |
| 4. Mixed-version SQLite acceptance | northstar-infrastructure | #66 | `e1f9b83` | `f71f8f7` |
| 5. Acceptance record | northstar-docs | — | This section and the acceptance record, uncommitted | — |

Core #40 (`29bd34a`) and Application `9ccfd77` fix platform-dependent four-digit year formatting, after the owner reported a CI failure on #95. They change no EI-4C contract.

### 14.3 Engineering acceptance summary

- **Verdict:** PASS WITH DOCUMENTED LIMITATIONS — synthetic engineering acceptance.
- **Matrix (52):** 40 VERIFIED, 8 VERIFIED WITH LIMITATION, 1 NOT VERIFIED (VC-04), 3 PLANNED (VC-21 to VC-23, owned by EI-5D).
- **SQLite:** the existing schema accepts version-2 records with no migration (verified on real SQLite, Windows). The pairing trigger stays **DEFERRED** (N6).
- **I08:** an equivalent mutant under the accepted contracts, not killed.
- **CI:** not independently verified.

### 14.4 Owner acceptance of slice 5 (2026-11-14)

**OWNER APPROVED, 2026-11-14. EI-4C Versioned History Compatibility: ACCEPTED WITH DOCUMENTED LIMITATIONS.**

The owner approved the slice-5 acceptance recommendation of the [acceptance record, section 14](Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md#14-engineering-verdict-and-recommended-status). The owner accepts:

- the completed slices 0 to 5 compatibility implementation and engineering evidence;
- the 40 VERIFIED scenarios;
- the eight VERIFIED WITH LIMITATION scenarios (VC-06, VC-07, VC-12, VC-15, VC-25, HB-09, PS-04, PS-05), retaining every limitation;
- VC-04 as NOT VERIFIED (below);
- VC-21 to VC-23 as PLANNED for EI-5D;
- the deferred N6 SQLite label-format pairing trigger;
- the I08 equivalent-mutant qualification;
- Windows-only evidence where Linux verification is unavailable;
- the existing EI4-D8 restrictions and the other previously deferred decisions.

**VC-04 — explicit owner disposition.**

| Item | Owner decision, 2026-11-14 |
| --- | --- |
| Status | Remains **NOT VERIFIED** |
| Legacy readers | Compatibility of pre-3a Application implementations with stored evaluator-v2 records is **not guaranteed**. No claim of legacy-reader compatibility is approved |
| Rollback or downgrade | To a pre-3a Application version against stored evaluator-v2 records: **NOT AUTHORIZED** without separate compatibility verification |
| Disposition | Accepted as a documented limitation; no additional testing required before slice-5 acceptance |

**Boundaries.**

- **EI-4 formal CI closure** stays **PENDING** independently verified evidence ([EI-4 Decision Register, section 15.3](Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4)). EI4-C4 stays **NOT VERIFIED**; no CI run is recorded.
- **Not authorized by this acceptance:** EI-5D implementation, MODELED issuance, ACTUAL issuance, production deployment, broker execution and new historical data writes. **EI-5D requires a separate owner milestone authorization.**
- **No new decision.** The acceptance changes no architecture decision, no scenario status and no evidence. Decisions N1 to N10 and the N9 amendment of section 14.1 are unchanged.

### 14.5 Status

| Item | Status |
| --- | --- |
| R-1, R-2, N1 to N10, N10 case C | **APPROVED** (2026-11-13, unchanged) |
| N9 sequencing amendment (combined 3a + 3b) | **APPROVED** (2026-11-14) |
| Slices 1, 2, 3a + 3b, 4 | **MERGED** |
| Existing SQLite schema compatibility | **VERIFIED** (slice 4, 2026-11-14; Windows local; no migration) |
| SQLite label-format pairing trigger | **DEFERRED** |
| Compatibility acceptance scenarios (52) | 40 VERIFIED, 8 VERIFIED WITH LIMITATION, 1 NOT VERIFIED, 3 PLANNED |
| Slice 5 engineering acceptance | **PASS WITH DOCUMENTED LIMITATIONS** (2026-11-14) |
| Slice 5 owner acceptance; EI-4C versioned history compatibility | **ACCEPTED WITH DOCUMENTED LIMITATIONS** (owner, 2026-11-14; section 14.4) |
| VC-04 | **NOT VERIFIED**; accepted as a documented limitation; rollback or downgrade to a pre-3a Application against stored evaluator-v2 records **NOT AUTHORIZED** without separate verification |
| EI-5 acceptance scenarios (119) | **PLANNED** |
| EI-5D | **NOT AUTHORIZED**; slice 5 is accepted, and EI-5D requires a separate owner milestone authorization |
| MODELED and ACTUAL issuance, new historical data writes, broker execution | **NOT AUTHORIZED** |
| EI-4 formal closure | **PENDING** independent CI verification |
| Operational or production authorization | **NOT GRANTED** |

> **Note, 2026-11-14.** The EI-5D and MODELED-issuance rows above record the state before the owner's authorization later that day, and are not rewritten. The current status is in [section 15.4](#154-status).

## 15. Owner authorization of EI-5D and EI-5E (2026-11-14)

**OWNER APPROVED, 2026-11-14,** subject to the existing ADR-024 and EI-5 governance.

The owner authorizes the combined implementation of **EI-5D — MODELED Recommendation Issuance** and **EI-5E — Synthetic End-to-End Integration** in northstar-application. The authorization follows the EI-5D authorization review of 2026-11-14 and its decisions D5D-1 to D5D-5. It approves implementation scope and records interpretation; it adds no architecture decision. Every decision of sections 4, 12, 13 and 14, of [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md) and of the [EI-5 design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md) stays binding.

### 15.1 Approved decisions D5D-1 to D5D-5

| ID | Decision | Governance reference | Status |
| --- | --- | --- | --- |
| D5D-1 | EI-5D is authorized as an **Application-only, synthetic** milestone: the verified issuer, the `eiris1` issuance identity, content-bound issuance and declaration authorizations, the declaration use case, the issuance reader and writer ports and the authoritative history source, exercised against a labelled in-memory authority test double that implements the writer invariants exactly | ADR-024 sections 2 to 7 and 12; design sections 7 and 8 | **APPROVED** |
| D5D-2 | EI-5D and EI-5E are delivered in **one Application implementation PR**. Packaging only: both milestones' scope and acceptance criteria stay binding | Design section 11 | **APPROVED** |
| D5D-3 | The authoritative issued-history `reference` is the fixed, versioned text **`northstar.equity-recommendation-issuance-authority/1`**. It is how issued-history evidence "names this authority" (design section 7.1, step 4). Because version-2 frozen inputs embed it, it is identity-bearing (`eiri1`, `eira1`); a change needs a new version | Design sections 7.1 and 8.1; ADR-024 section 14 | **APPROVED** |
| D5D-4 | **BD-04** ("no declaration for the session: `ISSUED`") is a **writer-contract** scenario: the writer's no-backdating invariant alone does not refuse it. At the issuer, a request without the required authoritative completeness declaration (greatest declared `complete_through` exactly `t − 1 µs`, with reproducible history evidence) stays **ineligible** (design section 7.1 step 4; IS-16) | Design sections 7.1, 8.2 and 12.10 | **APPROVED** (interpretation) |
| D5D-5 | EI-4 formal CI closure stays a **separate outstanding obligation** and does not block this milestone. EI4-C4 stays mandatory for formal EI-4 closure | [EI-4 Decision Register, section 15](Equity-Intraday-V1-EI4-Decision-Register.md#15-ei-4-closure-dispositions-2026-11-12); design section 11 | **APPROVED** |

### 15.2 Binding requirements

The implementation must preserve:

- issuance only of a re-verified (levels A to C), evaluator-`/2`, `PROPOSED` disposition; version-1 dispositions stay verifiable and non-issuable;
- exact history completeness at `t − 1 µs`: the session's greatest declared `complete_through` exactly `b`, and frozen history evidence that names the authority and reproduces at its recorded watermark;
- no backdating, one monotone recording sequence, and the uniqueness rules per issuance identity, per generation key and per (strategy, listing, trading date);
- deterministic `eiris1` identities, with a golden vector;
- content-bound authorization, refused before any write when missing, foreign or changed;
- append-only semantics; EI-4D dispositions and attempts unchanged;
- EI4-D8's conservative open-state treatment, unchanged;
- MODELED issuance only; synthetic and non-actionable records.

### 15.3 Not authorized

- A SQLite issuance authority or any schema (EI-6);
- ACTUAL issuance, actionable recommendations, orders, broker execution, real fills, positions or P&L;
- production deployment, composition roots or monitoring jobs;
- API or Web expansion, real or live data, external publication;
- relaxation of EI4-D8 or of any previously approved restriction; resolution of O-2 or O-3.

**EI-6 requires separate authorization.** The EI-4C limitations, including VC-04's rollback restriction, remain in force.

### 15.4 Status

| Item | Status |
| --- | --- |
| EI-5D and EI-5E | **AUTHORIZED** (2026-11-14), combined in one northstar-application PR; **NOT STARTED** |
| D5D-1 to D5D-5 | **APPROVED** (2026-11-14) |
| Authoritative history reference | `northstar.equity-recommendation-issuance-authority/1` |
| MODELED issuance | Authorized **within EI-5D and EI-5E only**: synthetic, against the test-double authority |
| ACTUAL issuance; SQLite issuance authority; EI-6 | **NOT AUTHORIZED** |
| EI-5 acceptance scenarios (119) | **PLANNED** |
| EI-4 formal closure | **PENDING** independent CI verification; does not block EI-5D or EI-5E |
| Operational or production authorization | **NOT GRANTED** |

> **Note, 2026-11-14.** The first row above records the state at authorization and is not rewritten. The current status is in [section 16.2](#162-status).

## 16. EI-5D and EI-5E implementation and engineering acceptance (2026-11-14)

This section records **engineering acceptance evidence** (sections 16.1 and 16.2) and the **owner's acceptance of 2026-11-14** (section 16.3); it does not rewrite section 15. The evidence is in the [EI-5D/EI-5E Modeled Issuance and Synthetic Integration Engineering Acceptance Record](Equity-Intraday-V1-EI5D-EI5E-Modeled-Issuance-and-Synthetic-Integration-Engineering-Acceptance-Record.md). D5D-1 to D5D-5 are unchanged.

### 16.1 Implementation and acceptance summary

| Item | Value |
| --- | --- |
| Implementation | northstar-application #96, merge `9a68209`, change commit `b2c81a2`; Application only (seven production modules, two test modules) |
| Engineering verdict | **PASS WITH DOCUMENTED LIMITATIONS — synthetic engineering acceptance** |
| EI-5D scenario parts (38) | 36 VERIFIED, 2 VERIFIED WITH LIMITATION (IS-16, HW-07) |
| EI-5E scenario parts (7) | 6 VERIFIED, 1 VERIFIED WITH LIMITATION (NB-04) |
| EI-6-owned | CC-01 to CC-06, RR-01, RR-03 to RR-06, and the S parts of ID-02, ID-03, BD-01, HW-03 and HW-08: **PLANNED** |
| Authority | A labelled in-memory test authority applying the production invariants; **not durable** |
| CI | Not independently verified; the local Infrastructure run predates a private-variable rename and two test-only edits in the merged commit |

### 16.2 Status

| Item | Status |
| --- | --- |
| EI-5D and EI-5E | **IMPLEMENTED AND MERGED** (#96); engineering acceptance **PASS WITH DOCUMENTED LIMITATIONS** (2026-11-14) |
| Owner acceptance of EI-5D and EI-5E | **ACCEPTED WITH DOCUMENTED LIMITATIONS** (owner, 2026-11-14; section 16.3) |
| Accepted scenario limitations | IS-16, HW-07 (EI-5D); NB-04 (EI-5E) |
| D5D-1 to D5D-5 | **APPROVED** (2026-11-14, unchanged) |
| MODELED issuance | Implemented synthetically, against the in-memory test authority only |
| SQLite issuance authority; EI-6 | **NOT IMPLEMENTED; NOT AUTHORIZED** |
| ACTUAL issuance; broker execution | **NOT AUTHORIZED** |
| EI-4 formal closure | **PENDING** independent CI verification (separate, D5D-5) |
| Operational or production authorization | **NOT GRANTED** |

### 16.3 Owner acceptance of EI-5D and EI-5E (2026-11-14)

**OWNER APPROVED — 2026-11-14. EI-5D MODELED Recommendation Issuance and EI-5E Synthetic End-to-End Integration: ACCEPTED WITH DOCUMENTED LIMITATIONS.**

The owner approved the engineering acceptance recommendation of the [EI-5D/EI-5E Engineering Acceptance Record, section 13](Equity-Intraday-V1-EI5D-EI5E-Modeled-Issuance-and-Synthetic-Integration-Engineering-Acceptance-Record.md#13-engineering-verdict-and-recommendation).

| Item | Accepted |
| --- | --- |
| Implementation | northstar-application, PR #96, merge `9a68209` |
| EI-5D (38 parts) | 36 VERIFIED, 2 VERIFIED WITH LIMITATION |
| EI-5E (7 parts) | 6 VERIFIED, 1 VERIFIED WITH LIMITATION |

**Accepted scenario limitations:**

| Scenario | Limitation |
| --- | --- |
| IS-16 | The invalid version-2 history-boundary sub-case cannot be constructed through the approved inputs contract, so that issuer-level negative path was not directly exercised |
| HW-07 | The watermark-beyond-current refusal was verified against the in-memory authority; durable-reader enforcement remains EI-6's responsibility |
| NB-04 | No-position behaviour was established within the synthetic integration environment, without a real paper-trading store |

**Additional accepted limitations:** the issuance authority is an in-memory, non-durable test implementation; EI-5C retains its synthetic labelling; independent CI evidence is not verified; the reported Infrastructure run began before the final Application private-variable rename and two test-only changes, and no run against the exact merged Application commit was independently established; EI4-D8's exposure and exclusivity restrictions remain unchanged; every previously accepted EI-4C limitation remains applicable, including VC-04's legacy rollback restriction, the deferred N6 SQLite pairing trigger and the I08 equivalent-mutation qualification; EI-4 formal CI closure remains separately pending; O-2 and O-3 remain deferred.

**Not authorized by this acceptance:**

- EI-6 implementation;
- durable SQLite issuance or declaration persistence;
- ACTUAL issuance;
- broker or account integration;
- orders, fills or real-money trading;
- production deployment;
- API or Web expansion;
- real market-data integration;
- relaxation of EI4-D8;
- resolution of deferred governance decisions.

**EI-6 requires a separate owner authorization.** D5D-1 to D5D-5 are unchanged, and the acceptance adds no architecture or implementation requirement.

## 17. Owner authorization of EI-6A (2026-11-14)

**OWNER APPROVED, 2026-11-14:** decisions **D6-1 to D6-6**, following the EI-6 authorization review of 2026-11-14.

The owner authorizes **EI-6A — Durable SQLite Recommendation Issuance and Declaration Authority**. EI-6B and EI-6C stay separately gated and are not authorized. Every decision of sections 4 and 12 to 16, of [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md) and of the [EI-5 design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md) stays binding, including the owner acceptance of EI-5D and EI-5E (section 16.3). The storage design is the [EI-6A storage design addendum](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md).

### 17.1 Approved decisions D6-1 to D6-6

| ID | Decision | Governance reference | Status |
| --- | --- | --- | --- |
| D6-1 | **EI-6A is authorized** as an Infrastructure-owned durable SQLite issuance and declaration authority. Scope: implement the merged Application issuance reader and writer ports; persist MODELED issued recommendations and authoritative completeness declarations; share the existing EI-4D SQLite database; enforce deterministic identities and uniqueness; enforce source-provenance linkage; maintain one authority-wide monotone recording sequence; preserve append-only behaviour; enforce no-backdating transactionally; support restart-safe current and historical views; support concurrent processes and commit-uncertain recovery; refuse corrupt stored records without repair. Not an authorization of EI-6B or EI-6C | ADR-024 sections 4 to 7 and 13; EI-5 design sections 8 and 10 | **APPROVED** |
| D6-2 | **Staged EI-6 delivery**: EI-6A, EI-6B, EI-6C, each with its own authorization and acceptance (section 17.2) | ADR-024 section 13; EI-5 design section 11 | **APPROVED** |
| D6-3 | **Timing of the deferred decisions**: O-3 before EI-6B; O-2 and EI3-D10 before EI-6C; none blocks EI-6A (section 17.3) | Section 6 (O-2, O-3); section 12.5, step 7; EI-5 design section 11 | **APPROVED** (clarification) |
| D6-4 | **EI-6A storage model**: additive SQLite objects in the EI-4D database file; structured issuance columns; integer-microsecond instant columns; one shared issuance and declaration recording-sequence ledger; no new canonical issued-record document or fingerprint format; the full issued record re-derived from verified source data on load. Approved **with additional integrity requirements** for the ledger (section 17.4) | EI-5 design section 10; ADR-024 sections 5 and 7 | **APPROVED** |
| D6-5 | **Typed corrupt-record handling**: one small prerequisite northstar-application PR, merged before the Infrastructure implementation imports it (section 17.5) | ADR-024 section 12; EI-5 design section 8.3 | **APPROVED** |
| D6-6 | **Mandatory CI evidence**: final EI-6A owner acceptance requires both local Windows real-SQLite engineering acceptance and independently verified successful Linux CI against the actual EI-6A PR revisions (section 17.6) | [EI-4 Decision Register, section 15.3](Equity-Intraday-V1-EI4-Decision-Register.md#153-what-closes-ei-4) (pattern) | **APPROVED** (stronger acceptance gate) |

### 17.2 D6-2 — Staged EI-6 delivery

This is a **sequencing and packaging amendment** to ADR-024 section 13 and to the EI-5 design, sections 9 to 11. The underlying architecture is unchanged; the original text of those sections is preserved, with dated amendments.

| Stage | Scope | Status |
| --- | --- | --- |
| **EI-6A** | Durable SQLite issuance and declaration authority: the issuance and declaration tables, the recording-sequence ledger, views, concurrency, restart and commit-uncertain recovery | **AUTHORIZED, NOT IMPLEMENTED.** May proceed independently |
| **EI-6B** | Durable lifecycle-event persistence: the `EquityRecommendationLifecycleStore` port, the lifecycle-event table, compare-and-append, rehydration through Core | **NOT AUTHORIZED.** Requires O-3, its own design decisions (including any canonical lifecycle-event format) and its own authorization |
| **EI-6C** | Monitoring and operational snapshot policy | **NOT AUTHORIZED.** Requires O-2 and EI3-D10, and its own authorization |

### 17.3 D6-3 — O-2, O-3 and EI3-D10 timing

The historical wording is preserved in [O-2](#o-2--terminal-incomplete_evidence-depends-on-the-evaluated-snapshot), [O-3](#o-3--late-evidence-after-terminal-incomplete_evidence) and [section 12.5](#125-ei-5b-sequencing-and-the-revised-sequence), with dated clarifications:

| Decision | Must be decided before | Does not block |
| --- | --- | --- |
| O-3 (late evidence after terminal `INCOMPLETE_EVIDENCE`) | EI-6B | EI-6A |
| O-2 (post-boundary snapshot selection) | EI-6C | EI-6A, EI-6B |
| EI3-D10 (trailing unfinalized candles) | EI-6C | EI-6A, EI-6B |

O-2, O-3 and EI3-D10 are **not resolved** and keep their status (DEFERRED or pending). The lifecycle-event vocabulary is unchanged.

### 17.4 D6-4 — Storage model and integrity requirements

The binding design is the [EI-6A storage design addendum](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md). In summary:

- **Ledger.** One authority-wide ledger shared by issuances and declarations, allocated transactionally and committed atomically with its record. The implementation enforces, or fails closed on: duplicate sequence numbers; invalid ordering; a ledger kind that mismatches its record; a committed ledger entry without its record; a record without a valid ledger entry; and partial allocation after a transaction failure. A one-way foreign key does **not** prevent an orphan ledger entry; that requirement is met by a pre-commit check in the same transaction and by fail-closed checks on reads and writes ([addendum, section 4.3](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md#43-integrity-requirements-and-how-each-is-enforced)).
- **Issuances.** `eiris1` identity, `eirk1` generation key, `eirp1` and `eira1` fingerprints, `MODELED` kind, strategy and listing and trading date, issued instant with its integer-microsecond column, and recording sequence; a foreign key to the EI-4D disposition; uniqueness by identity, by generation key and by (strategy, listing, trading date); insert-only. The referenced disposition must be `PROPOSED`, evaluator `/2`, and consistent with the issuance's stored identities and provenance. A matching generation key alone is not sufficient authority.
- **Declarations.** Exchange and trading date, `complete_through` with its integer-microsecond column, and recording sequence; strictly monotone; equivalent retries write nothing; append-only.
- **Transactions.** Authorization validated before any connection opens; `BEGIN IMMEDIATE`; authoritative state loaded; the Application-owned writer rules applied; the next sequence allocated; ledger entry and record inserted atomically; one COMMIT; uncertain commits reconciled through the existing Application contract; first-committed-record authority preserved.
- **Views.** `load_issuance`, `load_session_issuance`, `session_view` and `session_view_at`, with the reference `northstar.equity-recommendation-issuance-authority/1`. An undeclared session returns `None`; a declared session with no visible issuance may return an empty tuple; a watermark beyond the current sequence is refused; views reproduce after restart and never repair records.
- **Unchanged.** No EI-4D table or historical row is modified.

### 17.5 D6-5 — Typed corrupt-record handling

One small prerequisite northstar-application PR, merged **before** the Infrastructure implementation imports the new contract. It:

- introduces a typed error for corrupt or non-re-deriving stored issuance records (including ledger and declaration inconsistency in the authority's stored state), named under the existing Application conventions in that PR, not frozen here;
- classifies it as a permanent failure, distinct from the retryable unavailability error;
- maps it through the existing issuer and the authoritative-history source, preserving the existing retryable versus permanent distinctions;
- preserves `STORED_ISSUANCE_INVALID` semantics where governed (EI-5 design, section 8.3).

It introduces no new issuance identity, format or write behaviour. **No Core change is authorized.**

### 17.6 D6-6 — Mandatory CI evidence

Final EI-6A owner acceptance requires **both**:

1. local Windows real-SQLite engineering acceptance; and
2. independently verified successful Linux CI, with the existing `python-ci.yml` workflows, against the actual EI-6A PR revisions — the Infrastructure EI-6A PR and the D6-5 Application PR — showing that the applicable EI-6A acceptance tests ran and passed.

The acceptance record states the Infrastructure and Application commit identities used by the local checks, and for each CI run its repository, commit, run identifier, conclusion and checked-out dependency commits ([addendum, section 13](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md#13-mandatory-ci-evidence-d6-6)).

- A passing local suite alone does not satisfy final owner acceptance.
- A merged PR alone does not prove that a particular CI run passed.
- If the CI evidence is unavailable, engineering implementation may be reported as complete, but **EI-6A owner acceptance stays PENDING**.
- The earlier EI-4 CI closure requirement is **not waived**: EI4-C4 stays mandatory for formal EI-4 closure.

### 17.7 EI-6A acceptance ownership

The EI-6A rows are listed, with their status, in the [addendum, section 12](../architecture/Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md#12-acceptance). Every row is **PLANNED**; none is VERIFIED by being documented.

| Owner | Scenarios |
| --- | --- |
| **EI-6A** | CC-01 to CC-06; RR-03; the historical-view part of RR-04; the SQLite (S) parts of ID-02, ID-03, BD-01, HW-03 and HW-08; storage scenarios SA-01 to SA-13 (verified source provenance, pre-connection refusal of unauthorized writes, append-only and no-replace protections, corruption and tampering refusal, restart-safe re-derivation, atomic recording-sequence integrity, durable view semantics). No backdating under concurrent writes is CC-03; commit uncertainty and bounded reconciliation are CC-06 and SA-09 |
| **EI-6B** | RR-01; the lifecycle-state part of RR-04; RR-05; RR-06 |
| **Unchanged** | RV-04 stays DEFERRED (O-3). Every EI-5D and EI-5E row keeps its accepted status (section 16.3) |

### 17.8 Carried-forward limitations

These stay in force and are not resolved by this authorization:

- **VC-04:** rollback or downgrade to a pre-3a Application against stored evaluator-v2 records is **NOT AUTHORIZED** without separate verification;
- the deferred **N6** SQLite label-format pairing trigger;
- the **I08** equivalent-mutant qualification;
- the conservative exposure and exclusivity restrictions of **EI4-D8**;
- the EI-5D **IS-16** limitation;
- the EI-5D **HW-07** limitation, until real-reader evidence is accepted;
- the EI-5E **NB-04** limitation;
- **EI-4 formal CI closure**, pending (EI4-C4);
- every existing historical-evidence restriction.

### 17.9 Not authorized

- EI-6B lifecycle persistence and EI-6C monitoring;
- the resolution of O-2, O-3 or EI3-D10;
- ACTUAL issuance;
- broker orders or fills, real-money trading;
- production deployment;
- API or Web exposure;
- composition roots or monitoring execution;
- relaxation of EI4-D8;
- new unapproved canonical issuance formats;
- reconstruction or modification of historical EI-4D evidence.

EI-6A remains **single-host and MODELED-only**.

### 17.10 Status

| Item | Status |
| --- | --- |
| D6-1 to D6-6 | **APPROVED** (owner, 2026-11-14) |
| EI-6A | **AUTHORIZED, NOT IMPLEMENTED** |
| EI-6A storage design addendum | **APPROVED** (D6-4, 2026-11-14) |
| D6-5 Application contract PR | **AUTHORIZED, NOT STARTED**; must merge before the Infrastructure implementation imports it |
| EI-6A acceptance scenarios | **PLANNED** |
| EI-6A owner acceptance | **PENDING** (requires D6-6 evidence) |
| EI-6B | **NOT AUTHORIZED** (O-3 first) |
| EI-6C | **NOT AUTHORIZED** (O-2 and EI3-D10 first) |
| O-2, O-3 | **DEFERRED** (timing clarified, unresolved) |
| EI3-D10 | **Pending** |
| EI-5D and EI-5E | **ACCEPTED WITH DOCUMENTED LIMITATIONS** (2026-11-14, unchanged) |
| ACTUAL issuance; broker execution | **NOT AUTHORIZED** |
| EI-4 formal closure | **PENDING** independent CI verification |
| Operational or production authorization | **NOT GRANTED** |

> **Note, 2026-11-14.** The EI-6A, D6-5 and acceptance-scenario rows above record the state at authorization and are not rewritten. The current status is in [section 18.3](#183-status).

## 18. EI-6A implementation and engineering acceptance (2026-11-14)

This section records **engineering acceptance evidence**. It is not an owner approval and does not rewrite section 17. The evidence is in the [EI-6A Durable SQLite Issuance Authority Engineering Acceptance Record](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md). D6-1 to D6-6 are unchanged.

### 18.1 Implementation and acceptance summary

| Item | Value |
| --- | --- |
| D6-5 prerequisite | northstar-application #97, merge `a17bd91`, change commit `00089fc` |
| EI-6A implementation | northstar-infrastructure #67, merge `63d2ab3`, change commit `e756d7d`; two production modules, two exports, one fixtures module, one test module |
| Local engineering verdict | **PASS WITH DOCUMENTED LIMITATIONS — synthetic, local Windows real-SQLite engineering acceptance** |
| EI-6A scenario rows (26) | 19 VERIFIED, 7 VERIFIED WITH LIMITATION (CC-01 to CC-04, RR-04 (views) and ID-02 (S): Windows-only process evidence; HW-03 (S): EI4-D8 construction limit) |
| Source verification on load | Level A on the referenced disposition, then re-derivation of the issued record; levels A to C before issuance. Matches the addendum, section 8; not a blocker |
| Local results | 43 EI-6A cases passed on merged `63d2ab3`; full Infrastructure suite 3,336 passed, 27 skipped, and 6 of 6 targeted mutations detected, on the pre-merge working tree |
| D6-6 Linux CI | **NOT VERIFIED** for #97 or #67 — evidence not available in this environment |
| EI-5D HW-07 | Durable-reader evidence now exists (SA-10); closure of that limitation is an owner decision |

### 18.2 Owner decision required

D6-6 is not satisfied, so EI-6A **cannot** be owner-accepted on this evidence. The recommended sequence is in the [acceptance record, section 13](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md#13-engineering-verdict-and-recommendation):

1. record the two Linux CI runs;
2. then decide on acceptance with the documented limitations;
3. decide separately whether the EI-5D HW-07 limitation is closed.

### 18.3 Status

| Item | Status |
| --- | --- |
| D6-1 to D6-6 | **APPROVED** (2026-11-14, unchanged) |
| D6-5 Application prerequisite | **IMPLEMENTED AND MERGED** (#97) |
| EI-6A | **IMPLEMENTED AND MERGED** (#67); local engineering acceptance **PASS WITH DOCUMENTED LIMITATIONS** |
| EI-6A acceptance scenarios (26) | 19 VERIFIED, 7 VERIFIED WITH LIMITATION (local Windows) |
| D6-6 independent Linux CI | **NOT VERIFIED** (#97 and #67) |
| EI-6A owner acceptance | **PENDING** |
| EI-6B, EI-6C | **NOT AUTHORIZED** |
| O-2, O-3; EI3-D10 | **DEFERRED**; pending |
| EI-4 formal closure | **PENDING** independent CI verification (separate) |
| ACTUAL issuance; broker execution | **NOT AUTHORIZED** |
| Operational or production authorization | **NOT GRANTED** |

> **Note, 2026-11-14.** The D6-6, scenario and owner-acceptance rows above record the state before the owner's decision and are not rewritten. The current status is in [section 18.5](#185-status).

### 18.4 Owner acceptance of EI-6A (2026-11-14)

**OWNER APPROVED — 2026-11-14. EI-6A — Durable SQLite Recommendation Issuance and Declaration Authority: ACCEPTED WITH DOCUMENTED LIMITATIONS.**

**Scope.** The durable SQLite MODELED recommendation issuance and completeness-declaration authority, exactly as authorized by D6-1 to D6-6 (section 17).

| Item | Accepted |
| --- | --- |
| Implementation | northstar-application #97 (merge `a17bd91`), the D6-5 prerequisite; northstar-infrastructure #67 (merge `63d2ab3`) |
| Local engineering acceptance | PASS WITH DOCUMENTED LIMITATIONS (Windows, real SQLite) |
| D6-6 Linux CI | **SATISFIED.** Application run `43599364295` (job `132460599656`): Linux, Python 3.13.18, 4,674 passed. Infrastructure run `43604917993` (job `132477386762`): Linux, Python 3.13.18, 3,339 passed and 24 skipped of 3,363, at `63d2ab3` with Application `a17bd91` and Core `29bd34a` |
| Workflow identity | `python-ci.yml`, the only workflow at both tested revisions; established from the repositories' workflow inventory, not from run metadata |
| Acceptance matrix (26) | 25 VERIFIED, 1 VERIFIED WITH LIMITATION (HW-03 (S)), 0 NOT VERIFIED |
| EI-5D HW-07 | The durable-reader verification gap is recognized as **addressed** through EI-6A SA-10. The EI-5D record and its owner acceptance (section 16.3) are preserved, with a dated cross-reference |

**Accepted limitations:**

- **HW-03 (S):** a later same-session issuance cannot be constructed under EI4-D8.
- **Ledger:** orphan-ledger integrity relies partly on adapter checks.
- **Issued instant:** checked against the proposal only by re-derivation on load.
- **`GENERATION_ALREADY_ISSUED`:** Application-contract evidence only.
- **Precondition:** the EI-4D generation tables are required first.
- **Load-time verification:** level A plus re-derivation; levels B and C run before issuance.
- **Test seams.**
- **Workflow identity:** as stated above.
- **Carried forward:** EI4-D8, VC-04, N6, I08, IS-16 and NB-04; EI-4 formal CI closure (separate); O-2, O-3 and EI3-D10 unresolved.

The [EI-6A Engineering Acceptance Record, sections 11.3, 12.1 and 13.1](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md#131-owner-acceptance-2026-11-14) holds the evidence.

**Not authorized by this acceptance:**

- EI-6B lifecycle persistence and EI-6C monitoring;
- the resolution of O-2, O-3 or EI3-D10;
- ACTUAL issuance; broker orders or fills; real-money trading;
- production deployment; API or Web exposure; composition roots or monitoring execution;
- a relaxation of EI4-D8; a new canonical issuance format; changes to historical EI-4D evidence.

It does not close EI-4. **EI-6B and EI-6C each need a separate owner authorization.**

### 18.5 Status

| Item | Status |
| --- | --- |
| D6-1 to D6-6 | **APPROVED** (2026-11-14, unchanged) |
| EI-6A | **ACCEPTED WITH DOCUMENTED LIMITATIONS** (owner, 2026-11-14) |
| EI-6A acceptance scenarios (26) | 25 VERIFIED, 1 VERIFIED WITH LIMITATION |
| D6-6 independent Linux CI | **SATISFIED** (runs `43599364295`, `43604917993`; workflow identity from the repository inventory) |
| EI-5D HW-07 limitation | **Addressed** through EI-6A SA-10 (owner approved); historical record preserved |
| EI-6B, EI-6C | **NOT AUTHORIZED** |
| O-2, O-3; EI3-D10 | **DEFERRED**; pending |
| EI-4 formal closure | **PENDING** (EI4-C4; separate obligation) |
| ACTUAL issuance; broker execution | **NOT AUTHORIZED** |
| Operational or production authorization | **NOT GRANTED** |

> **Note, 2026-11-14.** The EI-6B and O-3 rows above record the state before the EI-6B authorization later that day and are not rewritten. The current status is in [section 19.7](#197-status).

## 19. Owner authorization of EI-6B (2026-11-14)

**OWNER APPROVED, 2026-11-14:** decisions **D6B-1 to D6B-7**, with three clarifications, following the EI-6B authorization review of 2026-11-14.

The owner authorizes **EI-6B — Durable Equity Recommendation Lifecycle Persistence**. EI-6C stays not authorized.

- **Still binding:** every earlier decision of this register, [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), the [EI-5 design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md), EI5-D5 to EI5-D8 (the frozen Core lifecycle policy), and the owner acceptance of EI-6A (section 18.4).
- **The binding design:** the [EI-6B Lifecycle Persistence Design addendum](../architecture/Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md).

### 19.1 Approved decisions D6B-1 to D6B-7

| ID | Decision | Design | Status |
| --- | --- | --- | --- |
| D6B-1 | **EI-6B authorized.** Scope: persist the Core lifecycle events of an existing authoritative MODELED issuance, from explicitly supplied synthetic evidence; an Application lifecycle-store contract; a verified append boundary; an Application-owned canonical event format; a narrow Application recording use case; persistence in the EI-6A SQLite file; atomic compare-and-append; rehydration and replay through existing Core rules; immutable history, concurrency, restart and recovery; synthetic end-to-end persisted acceptance. Limitations: single host, MODELED only, development machine only, no real market data, no monitoring or automatic snapshot selection | Addendum sections 1, 7 | **APPROVED** |
| D6B-2 | **O-3 resolved: option (b)** (section 19.3) | Addendum section 9 | **APPROVED** |
| D6B-3 | **Lifecycle-store contract:** `EquityRecommendationLifecycleStore` with `load_lifecycle` (a verified, replayed lifecycle record, or `None` for an unknown issuance) and `append(authorization)` (`APPENDED`, `EQUIVALENT`, `CONFLICT`); typed unavailable, commit-unconfirmed, unverified-append and permanent lifecycle-corruption failures, kept distinct from the EI-6A D6-5 error | Addendum section 4 | **APPROVED** |
| D6B-4 | **Canonical event documents:** Application-owned format `northstar.equity-recommendation-lifecycle-event/1`, strictly decoded and re-encoded; **no lifecycle-event content fingerprint**; v1 is the first durable event contract, with no migration | Addendum section 3 | **APPROVED** |
| D6B-5 | **Verified append authorization:** `VerifiedEquityRecommendationLifecycleAppend`, content-bound to the issuance identity, the exact prior history, the exact new events and the expected position; issued only by the recording use case; validated before any connection opens; no raw append | Addendum section 5 | **APPROVED** |
| D6B-6 | **SQLite lifecycle store:** `equity_recommendation_lifecycle_events` in the EI-4D and EI-6A file; foreign key to `equity_recommendation_issuances`; unique (issuance, event sequence); per-issuance sequences, separate from the EI-6A ledger; insert-only; chronology and not-before-issuance guards; `BEGIN IMMEDIATE`; parent re-derivation; Core replay | Addendum section 8 | **APPROVED** |
| D6B-7 | **Evidence gate:** local Windows synthetic and real-SQLite acceptance, plus independently verified Linux CI for the Application and Infrastructure implementation PRs, with exact commits and evidence that the EI-6B tests ran; then a separate owner decision | Addendum section 13 | **APPROVED** |

### 19.2 Approved clarifications

| # | Clarification | Design |
| --- | --- | --- |
| 1 | **Concurrent identical appends.** First valid append `APPENDED`. An identical retry, or a concurrent identical request, is `EQUIVALENT` with no duplicates. A concurrent conflicting request, or a stale writer, is `CONFLICT`, and its events are never adopted. Commit uncertainty resolves by reload: `EQUIVALENT` only after confirming the exact events, with no blind retry. Corrupt history fails closed and is never `EQUIVALENT` | Addendum section 10 |
| 2 | **Authoritative window derivation.** The lifecycle window is derived from the authoritative issuance and its verified disposition and candidate grid, never from the caller. Inconsistent or undecodable provenance fails closed. Infrastructure calls the Application derivation and computes no window itself. Explicit acceptance tests (LS-17) | Addendum section 6 |
| 3 | **No persisted quiet-evaluation cursor.** A snapshot producing no event fabricates nothing and advances nothing; no exactly-once processing is claimed; repeated evaluation is deterministic and safe; no processing-offset table. Snapshot selection and monitoring progress are EI-6C's | Addendum section 7 |

### 19.3 D6B-2 — O-3 resolved: option (b)

**OWNER APPROVED, 2026-11-14.**

- **No event.** When a lifecycle has reached terminal `INCOMPLETE_EVIDENCE`, later evidence that fills the original gap causes **no new lifecycle event** during EI-5 or EI-6.
- **Unchanged.** The terminal state stays final, and the lifecycle history stays immutable. No event is deleted, corrected, superseded or rewritten, and there is no automatic reopening.
- **Where the evidence lives.** The late evidence remains in the immutable EI-2B journal and snapshots.
- **Revisions are a different case.** `SOURCE_REVISION_RECORDED` stays exactly as Core permits it; it is not expanded to cite `EVIDENCE_INCOMPLETE`. A late gap-filling bar and a permitted source revision are not equivalent.
- **Scoped exception.** The [specification, section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states) requires late gap-filling evidence to be recorded as an explicit audited event citing the original terminal event. That requirement is **deferred to a future EI-8 restatement design**. The specification is not rewritten. EI-8 is **not** authorized; it may consider a non-state reconciliation event, but no such Core event kind is authorized now.
- **RV-04:** terminal state unchanged; no new lifecycle event for late gap-filling evidence. This authorizes no monitoring or restatement.
- **Status.** O-3 is **RESOLVED**. O-2 and EI3-D10 stay **UNRESOLVED** and are required before EI-6C.

### 19.4 Implementation sequence

| PR | Repository | Delivers |
| --- | --- | --- |
| PR-0 | northstar-docs | This section; the addendum; the specification section 8.2 deferral |
| PR-1 | northstar-application | Port, errors, codec, window derivation, authorization, recording use case, in-memory and synthetic end-to-end tests |
| PR-2 | northstar-infrastructure | Additive lifecycle table and SQLite store; provenance; compare-and-append; replay; restart and concurrency acceptance |
| PR-3 | northstar-docs | Engineering acceptance record, matrix, Linux CI evidence, limitations; a separate owner decision |

One writable repository per PR. Core is read-only; no PR imports unmerged interfaces.

### 19.5 Carried-forward limitations

These stay in force:

- EI4-D8 conservative exposure and exclusivity;
- EI-6A HW-03; EI-6A's adapter-enforced ledger checks;
- VC-04's legacy rollback restriction;
- the deferred N6 trigger; the I08 qualification;
- IS-16; NB-04;
- EI-4 formal CI closure outstanding;
- the single-host trust model; MODELED-only issuance.

EI-6A's acceptance is not reopened.

### 19.6 Not authorized

- EI-6C monitoring; automatic snapshot selection; monitoring scheduling or polling;
- the resolution of O-2 or EI3-D10;
- ACTUAL issuance; broker orders or fills; real-money trading;
- live market-data integration; production deployment; API or Web endpoints;
- EI4-D8 relaxation;
- terminal-state reopening; historical lifecycle rewriting;
- EI-8 restatement implementation;
- any new Core event kind or other Core change.

EI-6B remains synthetic-evidence-only despite its non-synthetic Application recording boundary.

### 19.7 Status

| Item | Status |
| --- | --- |
| D6B-1 to D6B-7; clarifications 1 to 3 | **APPROVED** (owner, 2026-11-14) |
| EI-6B | **AUTHORIZED — NOT IMPLEMENTED — OWNER ACCEPTANCE PENDING** |
| EI-6B design addendum | **APPROVED** (2026-11-14) |
| EI-6B acceptance scenarios (RR-01, RR-04 (state), RR-05, RR-06, RV-04, LS-01 to LS-17) | **PLANNED** |
| O-3 | **RESOLVED** (option (b), 2026-11-14) |
| Specification section 8.2, late gap-filling evidence | **DEFERRED** to a future EI-8 design (scoped exception) |
| O-2; EI3-D10 | **UNRESOLVED**; required before EI-6C |
| EI-6A | **ACCEPTED WITH DOCUMENTED LIMITATIONS** (2026-11-14, unchanged) |
| EI-6C; EI-8 | **NOT AUTHORIZED** |
| Core change | **NOT AUTHORIZED** |
| EI-4 formal closure | **PENDING** (separate obligation) |
| ACTUAL issuance; broker execution | **NOT AUTHORIZED** |
| Operational or production authorization | **NOT GRANTED** |
