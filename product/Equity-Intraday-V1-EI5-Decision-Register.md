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

### O-3 — Late evidence after terminal `INCOMPLETE_EVIDENCE`

- **Evidence.** The [specification, section 8.2](Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states), says evidence that arrives after a terminal state, such as "a late bar that fills a gap", is recorded as an explicit audited event. EI5-D8 approves revision events for corrections to cited evidence. A late arrival is not a correction.
- **Impact.** No approved event kind records it. The state is unchanged either way (EI5-D8).
- **Options.**
  - (a) Approve a separate non-state reconciliation event kind for late evidence.
  - (b) Record nothing in EI-5. A later restatement (EI-8) reads the evidence directly.
- **Recommendation.** (b) for EI-5, with (a) decided before EI-6.
- **Blocks.** Nothing in EI-5B to EI-5E. Before EI-6.
- **Status, 2026-11-12: DEFERRED** to the governance stage before EI-6. No resolution is recorded, and no late-evidence event kind exists.

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
- **Fail closed.** These fail closed exactly as today, with no disposition and an attempt only:
  - missing history (`None`);
  - unavailable or unverifiable history;
  - history incomplete at `b`;
  - history for another session.

  Nothing assumes an empty history.

**Consequence for issuance (no conflict, but a protocol constraint).**

- The assessment needs `complete_through >= t − 1 µs`.
- Issuance needs `issued_at > complete_through`, that is `complete_through <= t − 1 µs`.
- Declarations never decrease, so a proposal is issuable only while its session's greatest declared `complete_through` is exactly `t − 1 µs`.

A deterministic workflow therefore issues in ascending modeled-issuance order:

1. declare the session complete through `t − 1 µs`;
2. assess, then issue every proposal at `t`;
3. only then declare further.

A declaration beyond `t − 1 µs` made before a proposal at `t` is issued makes that proposal permanently non-issuable (`BACKDATED_ISSUANCE`): it fails closed and is never repaired. This becomes a requirement of EI-6 monitoring. Recommendations issued at exactly the same modeled instant do not see each other in their assessments. Same-listing exclusivity is unapproved, so it stays unresolved or not assessed. EI5-D1's one issuance per listing, session and strategy is still enforced at issuance.

**Versioned compatibility contract: requirements for the prerequisite implementation.**

1. **New versions.** A new evaluator version, and new frozen-inputs and assessment format versions, distinct from `northstar.equity-recommendation-evaluator/1`, `…-generation-inputs/1` and `…-assessment/1`. The new issued-history evidence states its boundary explicitly, and the new version validates `as_of = t − 1 µs`, as version 1 today requires `as_of = t` (`EquityRecommendationInputs`).
2. **Existing records unchanged.** Every existing version-1 record, document, fingerprint (`eiri1`, `eira1`, `eirp1`) and key (`eirk1`) keeps its meaning, "complete through the modeled issuance". None is reinterpreted, migrated, relinked or re-fingerprinted.
3. **Historical replay is not weakened.** Version-1 records stay verifiable at levels A to C by the version-1 evaluator, and new records by the new one.
4. **The generation key is unchanged.** A candidate and policy set already recorded under version 1 keep that disposition. A new-version assessment under the same `eirk1` is a content conflict (EI4-D9-D), never a replacement.
5. **Issuance accepts only the new version.** EI-5D issues only a disposition whose assessment is of the new version, and whose history evidence names the issuance authority and reproduces exactly from the authority's view at its recorded watermark. A version-1 disposition is refused as not issuable, and is preserved unchanged.
6. **Verifiable across concurrency and restart.** After a declaration at or beyond `b`, any issuance with `issued_at < t` is refused as backdated. The frozen evidence therefore stays exactly the authority's complete prefix at every later watermark ([ADR-024, section 7](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md#7-recording-sequence-watermark-and-completeness)).

**Contract points that remain open. They are identified here and not resolved; the prerequisite's own design must settle them.**

- **R-1 — Two-version verification.**
  - Today the verifier replays exactly one evaluator version (`verify_equity_recommendation_generation.py` refuses any other as incompatible, "Nothing is migrated").
  - ADR-021, section 10, says there is no historical evaluator registry, and that an unsupported format fails closed.
  - Requirement 3 needs both evaluator versions to verify. That is a bounded first step toward ADR-021's "historical evaluator registry" future consideration, so it needs explicit architecture approval in the prerequisite's design.
  - The alternative, letting version-1 records become incompatible, would weaken historical replay and is not acceptable under C-2.
- **R-2 — Fingerprint prefixes and storage.**
  - Canonical documents include their `format` field, so a new-version document has a distinct fingerprint even under the existing `eiri1` and `eira1` prefixes. That keeps the Infrastructure `GLOB` constraints valid.
  - New prefixes would need an Infrastructure schema change.
  - The prerequisite's design decides which.

**Status.** The approved correction is a **required implementation prerequisite before EI-5D**. It is not an approved implementation PR: it needs its own design and approval. It is **NOT STARTED**. **Existing EI-4C behaviour has not been updated**: today's EI-4C still reads history as of the modeled issuance.

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

### 12.6 Deferred and pending

- **O-2** (post-boundary snapshot selection and finality latency) and **O-3** (late evidence after terminal `INCOMPLETE_EVIDENCE`) stay **DEFERRED** to their later governance stage. No resolution is recorded.
- R-1 and R-2 ([section 12.2](#122-c-2--the-exclusive-history-boundary)) belong to the compatibility prerequisite's design.
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
| EI-4C versioned compatibility prerequisite | Required before EI-5D; **NOT STARTED**; own design and approval needed (R-1, R-2 open) |
| O-2, O-3 | **DEFERRED** |
| EI-5B to EI-5E, EI-6 | **NOT STARTED** |
| Acceptance scenarios (119) | **PLANNED** |
| EI-4 formal closure | **PENDING** independent CI verification |
| Operational or production authorization | **NOT GRANTED** |
