# EI-5 — Equity Recommendation Issuance and Lifecycle Design v1.0

Status at proposal (historical): **PROPOSED — FOR OWNER REVIEW** (2026-11-12). Version 1.0.

> **Status update, 2026-11-12: OWNER APPROVED WITH AMENDMENTS — 2026-11-12.** The owner approved this design and [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), with three approved amendments ([EI-5 Decision Register, section 12](../product/Equity-Intraday-V1-EI5-Decision-Register.md#12-owner-approval-of-the-ei-5a-amendments-2026-11-12)):
>
> - **C-1:** the `eiris1` issuance identity;
> - **C-2:** the exclusive history boundary, through a versioned EI-4C prerequisite before EI-5D;
> - **O-1:** gap-through stop and target recognition.
>
> The owner also approved that EI-5B may begin after the EI-5A documentation PR is merged.
>
> - **Amendments.** They are incorporated in sections 4, 5.4, 7, 8, 10, 11, 12, 13 and 14, each marked "amended 2026-11-12". The owner-approval record is in [section 17](#17-owner-approval-2026-11-12).
> - **Status.** Architecture APPROVED. Engineering implementation NOT STARTED. Every acceptance scenario PLANNED. Operational or production authorization NOT GRANTED.
> - **Deferred.** O-2 and O-3 stay deferred.
> - **Historical text.** Where the text below says the design is proposed, or that C-1, C-2 or O-1 needs a decision, that is the status at proposal.

> **Status update, 2026-11-12: EI5-D7 exact-boundary finality clarification (owner approved).**
>
> - **The rule.** At `as_of` exactly equal to the applicable boundary, a `MISSING` or `UNFINALIZED` boundary bar blocks temporarily and is never terminal. An `ACCEPTABLE` boundary bar is evaluated normally.
> - **Where it is incorporated.** Sections 4, 5.3, 5.5, 6.3, 6.4 and 12.5 (TB-03, TB-04), each marked "amended 2026-11-12 (exact boundary)". The record is in the [EI-5 Decision Register, section 12.8](../product/Equity-Intraday-V1-EI5-Decision-Register.md#128-ei5-d7-exact-boundary-finality-clarification).
> - **Unchanged.** O-2 stays deferred, and the matrix keeps 119 PLANNED scenarios.

- **Governance basis.** EI0-D9, EI4-D5 and EI5-D1 to EI5-D8 were **OWNER APPROVED, 2026-11-12** ([EI-5 Decision Register](../product/Equity-Intraday-V1-EI5-Decision-Register.md)).
- **The approved decisions and this design are different things.** The decisions are approved; this design is a proposal. Every type, port, use case, table and test below is proposed, and none exists.
- **Architecture record.** [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), also proposed.
- **Open items.** Conflicts C-1 and C-2 and open points O-1 to O-3 need owner decisions ([EI-5 Decision Register, section 6](../product/Equity-Intraday-V1-EI5-Decision-Register.md#6-conflicts-and-open-points-found-during-ei-5a)). Where the design depends on one, it says so.
- **Not implemented and not accepted.** Every acceptance scenario is **PLANNED**.

**Location.** `architecture/` holds design specifications. The README's Architecture row already links the folder, so no index changes. Version 1.0 is the first version; a revised design is a new version, and this file is not rewritten.

Source paths are relative to each repository root. Code was read at:

| Repository | Branch | Commit |
| --- | --- | --- |
| northstar-core | `develop` | `bc081a2` (EI-4B #38) |
| northstar-application | `develop` | `8937f12` (EI4-D11 W3 #92) |
| northstar-infrastructure | `develop` | `c105e94` (L9-P #63) |
| northstar-api | `develop` | `7ad37f5` |
| northstar-web | `develop` | `5aeb94c` |
| northstar-docs | `develop` | `89ac48f` (EI4-D11 W5 #56) |

## 1. Status and scope

| Item | Status |
| --- | --- |
| EI-5 policy decisions | APPROVED, 2026-11-12 |
| This design and ADR-024 | **PROPOSED — NOT OWNER APPROVED** |
| C-1, C-2, O-1 to O-3 | **OWNER DECISION REQUIRED** |
| EI-5B to EI-5E, EI-6 | **NOT STARTED** |
| Acceptance | **PLANNED**; no scenario executed |

> **Note, 2026-11-12.** The table records the status at proposal. The current status:
>
> - EI-5 policy decisions, C-1, C-2 and O-1: APPROVED.
> - This design and ADR-024: **APPROVED with amendments**.
> - The EI-4C versioned compatibility prerequisite: NOT STARTED.
> - O-2 and O-3: **DEFERRED**.
> - EI-5B to EI-5E and EI-6: **NOT STARTED**. EI-5B may begin after the EI-5A documentation PR is merged.
> - Acceptance: **PLANNED**.

**Scope.** Synthetic, LONG-only, MODELED issuance of equity recommendations from verified EI-4D proposals; the authoritative issued history; and the intraday lifecycle evaluated from EI-2B snapshots.

**Not in scope:**

- ACTUAL issuance;
- `INVALIDATED`;
- monitoring jobs and scheduling;
- API and Web;
- evaluation and performance (EI-8);
- real data;
- broker, paper or live execution;
- Futures, Options and paper-trading stores, which are unchanged.

## 2. Governing decisions

| ID | Summary | Design sections |
| --- | --- | --- |
| EI0-D9 | `UNKNOWN` ordering; `INDETERMINATE_ENTRY` and `INDETERMINATE_EXIT` terminal; no stop-first | 5.4, 5.5 |
| EI4-D5 | Stop-only touch while waiting gives `CANCELLED_BEFORE_ENTRY`; ambiguity takes precedence | 5.4 |
| EI5-D1 | Separate, re-verified, MODELED-only issuance; uniqueness; non-actionable; verified-issuer boundary | 7 |
| EI5-D2 | `RECORDED` begins at durable issuance; no clock | 5.3, 7 |
| EI5-D3 | Versioned issuance identity (prefix: C-1) | 7.4 |
| EI5-D4 | Monotone sequence, declarations, watermark, no backdating, unknown history | 8 |
| EI5-D5 | State model, transitions, terminality; `INVALIDATED` out of scope | 5 |
| EI5-D6 | Entry window, interval order, snapshot citation, deficiency blocking | 4, 6 |
| EI5-D7 | Cutoff, expiry, temporary block, terminal `INCOMPLETE_EVIDENCE` | 5.5, 6.4 |
| EI5-D8 | Snapshot-instant evaluation, immutable terminal states, revision events | 6.5 |

## 3. Current implemented architecture

### 3.1 What exists and is reused

| Concern | Existing component | Location |
| --- | --- | --- |
| Levels | `EquityRecommendationLevels` (`entry_zone`, `stop`, `target`), `EquityEntryZone` (`low`, `high`) | northstar-core `recommendations/equity/levels.py` |
| Timing | `EquityRecommendationTiming` (`decision_as_of`, `issuance_kind`, `issued_at`, `eligible_from`), `EquityIssuanceKind` (`ACTUAL`, `MODELED`) | northstar-core `recommendations/equity/timing.py` |
| Proposal envelope | `EquityModeledEntryWindow` (`timing`, `latest_entry_bar_opens_at`, `modeled_exit_at`, `timing_policy_version`); `EquityTradeRecommendationProposal` with its `not_assessed` controls | northstar-application `application_services/equity_trade_recommendation_eligibility.py` |
| Research-window fit | EI-4C refuses a timing policy whose latest entry-bar opening is not a grid opening, or whose modeled exit is not a grid close, of the candidate's research window (`TIMING_POLICY_NOT_APPLICABLE`) | `generate_equity_trade_recommendation.py` |
| Stored-generation verification | `VerifyStoredEquityRecommendationGenerationUseCase.execute(generation_key)`, levels A to C, with distinct invalid, incompatible, unavailable and replay-mismatch errors | `verify_equity_recommendation_generation.py` |
| Generation identity and encoding | `eirk1` (length-prefixed SHA-256), canonical JSON fingerprints | `equity_recommendation_generation_evidence.py` |
| Issued-history read contract | `EquitySessionRecommendationHistory`, `EquityPriorSessionRecommendation`, `EquitySessionRecommendationHistorySource`, `EquityRecommendationHistoryUnavailableError` | `ports/equity_recommendation_references.py` |
| Evidence snapshots | `EquityCandleEvidenceSnapshot` (`snapshot_id` `ecs1`, `as_of`, `watermark`, `intervals` with `opens_at`, `closes_at`, `outcome`, `acceptable_sequences`); `EquityCandleEvidenceSnapshotStore.get_snapshot` | `equity_candle_evidence_snapshot.py`; `ports/equity_candle_evidence_journal.py` |
| Interval outcomes | `EquityCandleIntervalOutcome`: `NOT_ENDED`, `MISSING`, `ACCEPTABLE`, `UNFINALIZED`, `CONFLICTING_VALUES`, `UNRESOLVED_REVISIONS` | `equity_intraday_candle_validation.py` |
| Verified-write pattern | Recorder-issued, content-bound write authorizations (EI4-D11) | `equity_recommendation_generation_write_authorization.py` |

### 3.2 What does not exist

- No equity lifecycle model in Core: no state, event or transition.
- No issuance use case, issued-record type, issuance identity or issuance authority.
- No production `EquitySessionRecommendationHistorySource`; only labelled synthetic fixtures.
- No snapshot query by listing and session: the snapshot store answers by `ecs1` only.
- No lifecycle store and no monitoring run.

## 4. Definitions

All instants are `PointInTime` values, compared as times. The synthetic values in the [EI-5 Decision Register, section 4.1](../product/Equity-Intraday-V1-EI5-Decision-Register.md#41-synthetic-example-conventions) illustrate them.

| Term | Definition |
| --- | --- |
| Interval | One grid interval of the snapshot's research window: `[opens_at, closes_at)`. Its candle is timestamped at `closes_at` (EI0-D6) |
| Entry window | The intervals whose `opens_at` satisfies `eligible_from <= opens_at <= latest_entry_bar_opens_at`. Both ends included |
| Entry-window end `E` | The `closes_at` of the interval that opens at `latest_entry_bar_opens_at` |
| Exit range | The intervals after the activation interval whose `closes_at <= modeled_exit_at`. The interval closing exactly at `modeled_exit_at` is included; an interval closing after it is never read |
| Applicable boundary | `E` while `RECORDED` or `WAITING_FOR_ENTRY`; `modeled_exit_at` while `ACTIVE` |
| Cursor | The next interval a recommendation's lifecycle needs. It starts at the interval that opens at `eligible_from` |
| Deficient interval | An interval whose outcome in the evaluated snapshot is `MISSING`, `UNFINALIZED`, `CONFLICTING_VALUES` or `UNRESOLVED_REVISIONS` |
| Exact-boundary finality (amended 2026-11-12 (exact boundary)) | The bar closing at the applicable boundary is `MISSING` or `UNFINALIZED` in a snapshot whose `as_of` equals that boundary. EI-2A treats an interval as ended when `closes_at <= as_of` (`NOT_ENDED` only when `closes_at > as_of`), and accepts a candle received and confirmed final exactly at its close, so such a bar may already be `ACCEPTABLE`. When it is `MISSING` or `UNFINALIZED`, it is not yet final only because its close has just been reached. It is a temporary block, never terminal |
| Evaluation instant | The evaluated snapshot's `as_of` (EI5-D8) |

**Level predicates** (amended 2026-11-12; O-1 approved). These use exact decimal comparison, on the low and high of the interval's acceptable candle:

- **Entry zone touched:** `low <= zone.high` and `high >= zone.low`. The bar's range intersects the zone. Gap-through is **not** transferred to entry: a range wholly above or below the zone touches nothing.
- **Stop reached:** `low <= stop`. A range wholly below the stop reaches it (gap-through).
- **Target reached:** `high >= target`. A range wholly above the target reaches it (gap-through).

In this design, "touched" for the stop or target means reached under these predicates. Reaching a level is a lifecycle observation only: no stop-order fill, execution price or exit price is inferred. The strict two-sided definition (`low <= level <= high`) was option (b) of O-1 and is not used.

## 5. Core: `northstar_core.recommendations.equity.lifecycle` (EI-5B)

### 5.1 Purity

- **Imports.** Only Core foundation and market-data value objects, and `northstar_core.recommendations.equity`. Never the Application, Infrastructure or `northstar_core.recommendations.futures`.
- **No side effects.** No clock, randomness, I/O, identity generation or persistence.
- **Opaque evidence references.** Evidence references (snapshot identities) are opaque, caller-supplied, non-empty text. Core never interprets an `ecs1` value.

### 5.2 Proposed types

| Type | Purpose |
| --- | --- |
| `EquityLifecycleState` | The eleven EI5-D5 states, with `is_terminal` and an outcome class (`RESOLVED`, `RESOLVED_NO_ENTRY`, `INDETERMINATE`, `UNRESOLVED`, or none for a non-terminal state) |
| `EquityLifecycleEventKind` | `ENTRY_WINDOW_OPENED`, `ENTRY_ACTIVATED`, `ENTRY_INDETERMINATE`, `ENTRY_CANCELLED`, `ENTRY_WINDOW_EXPIRED`, `TARGET_REACHED`, `STOP_REACHED`, `EXIT_INDETERMINATE`, `SESSION_CUTOFF_REACHED`, `EVIDENCE_INCOMPLETE`; and the non-state kind `SOURCE_REVISION_RECORDED` |
| `EquityLifecycleLevel` | `ENTRY`, `STOP`, `TARGET`, in that canonical order |
| `EquityIntrabarOrdering` | `NOT_REQUIRED`, `UNKNOWN`. There is no modeled ordering |
| `EquityLifecycleCancellationReason` | `STOP_TOUCHED_BEFORE_ENTRY` |
| `EquityEvidenceDeficiencyKind` | `MISSING`, `UNFINALIZED`, `CONFLICTING_VALUES`, `UNRESOLVED_REVISIONS`, the exact EI-2A names. The Application maps them one to one |
| `EquityEvidenceDeficiency` | An interval (`opens_at`, `closes_at`) and its kind |
| `EquityLifecycleEvidence` | An opaque evidence reference, the interval, and the acceptable observation references used |
| `EquityLifecycleWindow` | The levels (`EquityRecommendationLevels`), `eligible_from`, `latest_entry_bar_opens_at` and `modeled_exit_at`. It validates `eligible_from <= latest_entry_bar_opens_at < modeled_exit_at`, as `EquityModeledEntryWindow` does |
| `EquityLifecycleBar` | An interval with its acceptable candle's `low` and `high` (Core `Price`, one currency, the levels' currency) and its evidence |
| `EquityLifecycleEvent` | `sequence` (from 1), `kind`, `from_state`, `to_state` (equal for a non-state event), `evaluated_at`, `evidence`, `touched` levels, `ordering`, plus `reason`, `deficiencies`, or revision fields (the cited event's sequence, the superseded and new evidence, materiality `UNASSESSED`) as its kind requires. No price, quantity, order or fill field exists |
| `EquityRecommendationLifecycle` | A window and its events. `state` is the fold of the events; construction validates the whole sequence |

### 5.3 Allowed event shapes

Every event must match exactly one row; anything else is refused at construction.

| Kind | From | To | Touched | Ordering | Other |
| --- | --- | --- | --- | --- | --- |
| `ENTRY_WINDOW_OPENED` | `RECORDED` | `WAITING_FOR_ENTRY` | none | `NOT_REQUIRED` | `evaluated_at >= eligible_from` |
| `ENTRY_ACTIVATED` | `WAITING_FOR_ENTRY` | `ACTIVE` | `ENTRY` | `NOT_REQUIRED` | Interval in the entry window |
| `ENTRY_INDETERMINATE` | `WAITING_FOR_ENTRY` | `INDETERMINATE_ENTRY` | `ENTRY` with `STOP`, `TARGET` or both | `UNKNOWN` | Interval in the entry window |
| `ENTRY_CANCELLED` | `WAITING_FOR_ENTRY` | `CANCELLED_BEFORE_ENTRY` | `STOP` | `NOT_REQUIRED` | Reason `STOP_TOUCHED_BEFORE_ENTRY`; interval in the entry window |
| `ENTRY_WINDOW_EXPIRED` | `WAITING_FOR_ENTRY` | `EXPIRED_BEFORE_ENTRY` | none, or `TARGET` | `NOT_REQUIRED` | Interval opens at `latest_entry_bar_opens_at` |
| `TARGET_REACHED` | `ACTIVE` | `CLOSED_TARGET` | `TARGET` | `NOT_REQUIRED` | Interval in the exit range |
| `STOP_REACHED` | `ACTIVE` | `CLOSED_STOP` | `STOP` | `NOT_REQUIRED` | Interval in the exit range |
| `EXIT_INDETERMINATE` | `ACTIVE` | `INDETERMINATE_EXIT` | `STOP`, `TARGET` | `UNKNOWN` | Interval in the exit range |
| `SESSION_CUTOFF_REACHED` | `ACTIVE` | `SESSION_CUTOFF_REACHED` | none | `NOT_REQUIRED` | Interval closes at `modeled_exit_at` |
| `EVIDENCE_INCOMPLETE` | `WAITING_FOR_ENTRY` or `ACTIVE` | `INCOMPLETE_EVIDENCE` | none | `NOT_REQUIRED` | `evaluated_at` at or after the applicable boundary; at least one deficiency, each at or before the boundary; never when `evaluated_at` equals the boundary and the first deficiency is the boundary bar `MISSING` or `UNFINALIZED` (amended 2026-11-12 (exact boundary)) |
| `SOURCE_REVISION_RECORDED` | any | the same state | none | `NOT_REQUIRED` | Cites an earlier event; materiality `UNASSESSED` |

**Sequence guards** checked by `EquityRecommendationLifecycle`:

- Sequences are contiguous from 1.
- `evaluated_at` is non-decreasing.
- No state-changing event follows a terminal state.
- Bar-driven events cite strictly ascending intervals. The one exception is `SESSION_CUTOFF_REACHED`, which may cite the activation interval when that interval closes at `modeled_exit_at`.
- Exit events cite intervals after the activation interval.
- A `SOURCE_REVISION_RECORDED` event cites an earlier event's interval.

There is no `INVALIDATED` and no Futures-only state or ordering. The Futures `EXPIRED_AFTER_ACTIVATION`, `EXPIRED_WITH_INCOMPLETE_COVERAGE` and `MODELED_STOP_FIRST` are not representable.

### 5.4 Bar evaluation

`evaluate_bar(lifecycle, bar)` returns the events one `ACCEPTABLE` bar produces at the cursor. A touch always takes precedence over that bar's expiry or cutoff.

| State | Entry zone | Stop | Target | Events |
| --- | --- | --- | --- | --- |
| `WAITING_FOR_ENTRY` | touched | not touched | not touched | `ENTRY_ACTIVATED`; then `SESSION_CUTOFF_REACHED` if the bar closes at `modeled_exit_at` |
| `WAITING_FOR_ENTRY` | touched | touched | any | `ENTRY_INDETERMINATE` (EI0-D9; EI4-D5 clarification) |
| `WAITING_FOR_ENTRY` | touched | not touched | touched | `ENTRY_INDETERMINATE` (EI0-D9) |
| `WAITING_FOR_ENTRY` | not touched | touched | not touched | `ENTRY_CANCELLED` (EI4-D5) |
| `WAITING_FOR_ENTRY` | not touched | not touched | any | None; `ENTRY_WINDOW_EXPIRED` if the bar opens at `latest_entry_bar_opens_at` |
| `ACTIVE` | not evaluated | touched | touched | `EXIT_INDETERMINATE` (EI0-D9) |
| `ACTIVE` | not evaluated | touched | not touched | `STOP_REACHED` |
| `ACTIVE` | not evaluated | not touched | touched | `TARGET_REACHED` |
| `ACTIVE` | not evaluated | not touched | not touched | None; `SESSION_CUTOFF_REACHED` if the bar closes at `modeled_exit_at` |

- A bar outside the applicable range is refused, never evaluated: one that opens before `eligible_from`, opens after `latest_entry_bar_opens_at` while waiting, or closes after `modeled_exit_at`.
- A LONG bar cannot reach both the stop and the target without touching the zone, so no other combination exists.
- **Gap-through cases** (amended 2026-11-12; O-1):
  - **While `ACTIVE`:** a range wholly below the stop is `STOP_REACHED`, and one wholly above the target is `TARGET_REACHED`.
  - **While `WAITING_FOR_ENTRY`:** a range wholly below the stop reaches the stop without touching the zone, so it is `ENTRY_CANCELLED` (EI4-D5). The scope basis is in the [EI-5 Decision Register, section 12.3](../product/Equity-Intraday-V1-EI5-Decision-Register.md#123-o-1--gap-through-recognition).
  - **Entry zone:** a range wholly above or below the zone never activates.
  - **Ambiguity still takes precedence:** zone and stop in one bar is `ENTRY_INDETERMINATE`, and stop and target in one bar is `EXIT_INDETERMINATE`.

### 5.5 Window opening and insufficient evidence

- `open_entry_window(lifecycle, evaluated_at, evidence)` returns `ENTRY_WINDOW_OPENED` when the state is `RECORDED` and `evaluated_at >= eligible_from`. Otherwise it returns nothing.
- `incomplete_evidence(lifecycle, evaluated_at, deficiencies, evidence)` returns `EVIDENCE_INCOMPLETE` only when all of these hold:
  - the state is `WAITING_FOR_ENTRY` or `ACTIVE`;
  - `evaluated_at` is at or after the applicable boundary;
  - the deficiencies are non-empty, ordered, and at or after the cursor and at or before the boundary.

  Otherwise it refuses. Below the boundary, the caller receives no event: that is a temporary block.

  *Amended 2026-11-12 (exact boundary).* At `evaluated_at` equal to the boundary, a `MISSING` or `UNFINALIZED` boundary bar is also a temporary block, and no `EVIDENCE_INCOMPLETE` is produced. A stored `EVIDENCE_INCOMPLETE` resting only on that condition is refused on replay. `CONFLICTING_VALUES`, `UNRESOLVED_REVISIONS` and earlier deficient intervals are not excepted.

### 5.6 Replay

`EquityRecommendationLifecycle(window, events)` re-validates every shape and guard and folds the state. The same window and events always give the same state. Core therefore guarantees that a stored event stream that violates EI5-D5 cannot be rehydrated.

## 6. Application: lifecycle evaluation (EI-5C)

### 6.1 Use case

`EvaluateEquityRecommendationLifecycleUseCase` is pure orchestration. In EI-5C it persists nothing.

- **Inputs:**
  - the issued recommendation's envelope. In EI-5C this is a labelled synthetic test double or a verified proposal; from EI-5D it is the issued record;
  - its current lifecycle (events so far);
  - the cursor and the last evaluated snapshot, both derived from the events;
  - the `ecs1` identity of the next snapshot to evaluate.
- **Dependency:** the existing `EquityCandleEvidenceSnapshotStore` (read only, `get_snapshot`), which verifies snapshot fingerprints on load.
- **Output:** an evaluation result:
  - the new events, which the caller appends;
  - the resulting state;
  - `BLOCKED` with the deficient interval, `AWAITING_EVIDENCE` (`NOT_ENDED`), `TERMINAL` or `ADVANCED`.

### 6.2 Snapshot preconditions

The evaluation refuses, with a typed error and no events, when:

| Condition | Error category |
| --- | --- |
| The snapshot is absent or fails verification | Evidence unavailable or invalid (existing EI-2B errors) |
| The listing or trading date differs from the recommendation's | Wrong session (no overnight carry-over) |
| The research-window policy version differs from the proposal's | Wrong window |
| `as_of` is before the modeled `issued_at` | Snapshot before issuance |
| `as_of` is before, or equal to with a different identity, the last evaluated snapshot's | Out of order |
| The journal watermark is lower than the last evaluated snapshot's | Watermark regression (EI3-D9 pattern) |
| The snapshot's grid does not contain the cursor interval, `E` or `modeled_exit_at` | Grid mismatch |

Re-evaluating the last evaluated snapshot is idempotent: no events.

### 6.3 Algorithm

For snapshot `S`:

1. If the state is `RECORDED`, ask Core to open the entry window at `S.as_of`. The event cites `S` and the interval that opens at `eligible_from`.
2. While the state is `WAITING_FOR_ENTRY` or `ACTIVE`, take the cursor interval `I` from `S`. A `RECORDED` recommendation evaluates no bar.
   - **`ACCEPTABLE`:** build an `EquityLifecycleBar` from its acceptable candle, append Core's events, and advance the cursor;
   - **`NOT_ENDED`:** stop with `AWAITING_EVIDENCE`;
   - **deficient:** collect every deficient interval from the cursor to the applicable boundary in `S`. If `S.as_of` is at or after the boundary, append Core's `EVIDENCE_INCOMPLETE`. Otherwise stop with `BLOCKED`, and append no event.
     - *Amended 2026-11-12 (exact boundary).* There is one exception: `S.as_of` equals the boundary and the deficient interval is the bar closing there, `MISSING` or `UNFINALIZED`. It also stops with `BLOCKED` and no event.
3. Detect revisions (section 6.5) for every interval cited by an earlier event.

Only intervals in `S` are read, and only from the cursor forward, so no look-ahead is possible. Every interval `S` reports as ended had ended at `S.as_of` (EI-2A).

### 6.4 Temporary blocking versus terminal evidence

| Situation | Result |
| --- | --- |
| A deficient interval, and `S.as_of` before the boundary | `BLOCKED`; no event; the state is unchanged |
| A later snapshot shows it `ACCEPTABLE` | Monitoring resumes from it |
| A deficient interval, and `S.as_of` at or after the boundary | `INCOMPLETE_EVIDENCE`, listing every deficient interval from the cursor to the boundary |
| `S.as_of` equals the boundary, and the boundary bar is `MISSING` or `UNFINALIZED` (amended 2026-11-12 (exact boundary)) | `BLOCKED`; no event; a later snapshot decides |
| `S.as_of` equals the boundary, and the boundary bar is `ACCEPTABLE` | Evaluated normally: expiry, cutoff, entry, cancellation or a level reach |
| `S.as_of` equals the boundary, and the boundary bar is `CONFLICTING_VALUES` or `UNRESOLVED_REVISIONS`, or an earlier interval is deficient | `INCOMPLETE_EVIDENCE` (unchanged) |
| No snapshot at or after the boundary is ever evaluated | The state stays non-terminal. Nothing is assumed (O-2) |
| An `ACCEPTABLE` interval after a deficient one | Never used while the deficiency stands |

### 6.5 Revision detection

For every interval cited as `ACCEPTABLE` evidence by an earlier event, a later snapshot may show that interval with a different set of acceptable observations, or as `CONFLICTING_VALUES` or `UNRESOLVED_REVISIONS`. Each such case is a correction to cited evidence. A `SOURCE_REVISION_RECORDED` event is then appended, citing the original event and both evidence references, with materiality `UNASSESSED`.

- Detection is idempotent per (cited interval, new evidence).
- It also runs after a terminal state, which is the only evaluation permitted then.
- It never changes the state.
- Late evidence for an interval cited as deficient is not recorded (O-3).

## 7. Application: issuance (EI-5D)

### 7.1 Use case

`IssueEquityTradeRecommendationUseCase.execute(generation_key, issuance_kind)`:

1. **Refuse `ACTUAL`.** Refuse `ACTUAL` before any read.
2. **Re-verify.** Call `VerifyStoredEquityRecommendationGenerationUseCase.execute(generation_key)`:
   - `None` means no stored disposition: refuse with no write;
   - invalid, incompatible, replay-mismatch or source-invalid errors are permanent refusals. The stored record is preserved, with no write (EI4-D11-B);
   - source-unavailable or store-unavailable errors are retryable refusals, with no write.
3. **Require `PROPOSED`.** Refuse `NOT_ELIGIBLE` and `UNRESOLVED`.
4. **History evidence.** Proposed check, pending confirmation with C-2: refuse unless both of these hold:
   - the assessment's frozen issued-history evidence names this authority;
   - the evidence reproduces exactly from the authority's view at its recorded watermark.

   Fixture-based proposals are therefore never issuable.

   *Amended 2026-11-12 (C-2 approved).* The check is confirmed. Issuance also requires:
   - the assessment to be of the new, versioned EI-4C assessment, whose history view is taken as of `t − 1 µs` (section 8.4). A version-1 disposition is refused as not issuable, and is preserved unchanged;
   - the session's current greatest declared `complete_through` to be exactly `t − 1 µs`. A greater one is `BACKDATED_ISSUANCE`.

   This step depends on the EI-4C compatibility prerequisite, which is not implemented.
5. **Derive the record and identity.** Derive the issued record (section 7.2) and its identity (section 7.4).
6. **Authorize and write.** Produce a content-bound `VerifiedEquityRecommendationIssuance` authorization, and call the writer once.
7. **Map the writer's outcome.** `ISSUED` with its sequence, `EQUIVALENT`, or a typed refusal (section 8.3).

### 7.2 The issued record

`EquityIssuedRecommendation`. Every field except the sequence is copied or derived from the verified stored proposal; none is supplied by the caller.

| Field | Source |
| --- | --- |
| Issuance identity | Section 7.4 |
| `generation_key` (`eirk1`), `proposal_fingerprint` (`eirp1`), `assessment_fingerprint` (`eira1`) | The verified disposition |
| `issuance_kind` | `MODELED` |
| Strategy, listing, trading date, exchange | The verified candidate |
| `decision_as_of`, `issued_at`, `eligible_from` | The proposal's `EquityRecommendationTiming` |
| `latest_entry_bar_opens_at`, `modeled_exit_at`, `timing_policy_version` | The proposal's entry window |
| Levels | The proposal's Core definition |
| `not_assessed` controls | The proposal, unchanged |
| `recording_sequence` | Assigned by the authority at commit; not part of the identity |

There is no actionability flag, no quantity committed to any account, and no order or fill field. The record's history projection is the existing `EquityPriorSessionRecommendation`.

### 7.3 Verified-issuer boundary

- **Ports.** `EquityRecommendationIssuanceReader` and `EquityRecommendationIssuanceWriter` are separate. The writer accepts only `VerifiedEquityRecommendationIssuance` and `VerifiedEquityIssuanceCompletenessDeclaration`.
- **Construction.** Only the issuance and declaration use cases construct those authorizations. Each binds to its exact record content and refuses `dataclasses.replace` or swapped content, as in EI4-D11 clarification B.
- **Wiring.** Read-only consumers receive only the reader: the history source, lifecycle evaluation and verification.
- **Scans.** Architecture scans in Application CI refuse writer use outside the two use cases and composition roots, as the EI4-D11 W3 scans do.

### 7.4 Issuance identity

*Amended 2026-11-12 (C-1 approved).*

`eiris1-sha256:` followed by `sha256(len:part ...)` over the parts below, using the exact length-prefixing of `eirk1`:

1. `northstar.equity-recommendation-issuance-key/1`;
2. the generation key text (`eirk1-sha256:…`);
3. the proposal fingerprint text (`eirp1-sha256:…`);
4. the issuance kind value (`MODELED`).

- **Distinct identities.** The frozen-inputs fingerprint `eiri1-sha256:` is a different identity and is unchanged. A search of all six repositories found no existing use of `eiris1` or of the namespace ([EI-5 Decision Register, section 12.1](../product/Equity-Intraday-V1-EI5-Decision-Register.md#121-c-1--the-eiris1-issuance-identity)).
- **Golden vector.** EI-5D includes a golden test vector computed from fixed synthetic inputs.

### 7.5 Completeness declarations

`DeclareEquitySessionIssuanceCompletenessUseCase.execute(exchange, trading_date, complete_through)`:

1. Validate that the instant lies within the session's venue day, using the existing venue-day evidence.
2. Produce a content-bound declaration authorization.
3. Call the writer, which enforces section 8.

The caller supplies no sequence or watermark.

## 8. Application: issued-history authority contracts (EI-5D)

### 8.1 Reader

| Method | Answer |
| --- | --- |
| `load_issuance(issuance_id)` | The issued record, or `None` |
| `load_session_issuance(strategy, listing, trading_date)` | The issued record, or `None` |
| `session_view(exchange, trading_date)` | The current view: `None` when no declaration exists, otherwise an `EquitySessionRecommendationHistory` at the current watermark |
| `session_view_at(exchange, trading_date, watermark)` | The view at an earlier watermark, for re-verification. Refused when the watermark is beyond the current position |

The authoritative `EquitySessionRecommendationHistorySource.session_history(exchange, date, as_of)` returns `session_view`. The caller keeps refusing a view whose `complete_through` is before `as_of`, as EI-4C already does. Read failures raise `EquityRecommendationHistoryUnavailableError`.

### 8.2 Invariants

The writer enforces these. EI-5D's test double implements them exactly, and EI-6's SQLite adapter enforces them by transaction and constraints.

1. One authority-wide `recording_sequence`, assigned at commit to issuances and declarations alike; unique, monotone and never reused.
2. One issuance per issuance identity, per generation key, and per (strategy, listing, trading date).
3. An issuance whose `issued_at <= max declared complete_through` for its session is refused (no backdating).
4. A declaration below the session's current maximum is refused; an equal one is idempotent.
5. View at `W`: issuances with `sequence <= W` and `issued_at <= CT(W)`, in (`issued_at`, key) order, where `CT(W)` is the maximum declared at a sequence at or below `W`.
6. **Stability:** for `W1 < W2`, the view at `W2` restricted to `issued_at <= CT(W1)` equals the view at `W1`.
7. No declaration means unknown (`None`). A declaration with no issuance means an explicit empty tuple.
8. Records are append-only. Nothing is updated, deleted or replaced.

### 8.3 Writer outcomes

| Outcome | Meaning |
| --- | --- |
| `ISSUED` | Recorded, with its sequence |
| `EQUIVALENT` | The same identity, with content that re-derives equal; nothing written |
| `GENERATION_ALREADY_ISSUED` | Another issuance exists for the generation key |
| `SESSION_STRATEGY_ALREADY_ISSUED` | Another issuance exists for the strategy, listing and date |
| `BACKDATED_ISSUANCE` | `issued_at` is at or before the declared `complete_through` |
| `STORED_ISSUANCE_INVALID` | The stored record under the identity does not re-derive; it is preserved and never treated as equivalent |
| `DECLARED` / `DECLARATION_EQUIVALENT` / `COMPLETENESS_REGRESSION` | Declaration outcomes |
| Unavailable / commit unconfirmed | Typed errors; commit uncertainty is resolved by an authoritative reload by identity (EI4-D9-G pattern) |

### 8.4 The C-2 history boundary and issuance protocol (amended 2026-11-12)

The owner approved C-2 on 2026-11-12 ([EI-5 Decision Register, section 12.2](../product/Equity-Intraday-V1-EI5-Decision-Register.md#122-c-2--the-exclusive-history-boundary)). The invariants of section 8.2 are unchanged.

| Item | Rule |
| --- | --- |
| Modeled issuance | `t`, derived exactly as today (EI4-D9-C) |
| Assessment history instant (new EI-4C version) | `b = t − 1 µs`. `PointInTime` carries at most six fractional digits, so `b` is the last representable instant strictly before `t` |
| Assessment requirement | `session_history(exchange, date, as_of=b)` with `complete_through >= b`. The frozen evidence lists exactly the issuances with `issued_at < t`; missing, unavailable, incomplete or foreign history fails closed |
| Issuance requirement | `issued_at = t > complete_through`, and the frozen evidence reproduces from `session_view_at(watermark)` |
| Combined | Issuable only while the session's greatest declared `complete_through` is exactly `b` |
| Protocol | In ascending modeled-issuance order: declare through `t − 1 µs`; assess and issue every proposal at `t`; then declare further |
| Same instant | Issuances at the same `t` are not in each other's assessments; EI5-D1 uniqueness still applies |
| Version 1 | Existing records keep "complete through `t`"; never reinterpreted; never issuable; still verifiable (R-1) |

**Prerequisite.** The new EI-4C assessment version is a required Application implementation prerequisite before EI-5D. It needs its own approved design, which must settle R-1 (two-version verification under ADR-021, section 10) and R-2 (fingerprint prefixes and storage). It is not implemented, and existing EI-4C still reads history as of `t`.

## 9. Application: lifecycle persistence port (planned for EI-6)

`EquityRecommendationLifecycleStore`:

- `load_events(issuance_id)` returns the events in sequence order.
- `append(issuance_id, expected_last_sequence, events)` is a compare-and-append, the Futures pattern. It refuses when the stored last sequence differs, so two evaluators can never fork a lifecycle.
  - An identical append at the same position is idempotent.
  - Appending different content at an occupied position is a conflict.
- Appends pass through a verified boundary like section 7.3. The events must rehydrate through Core (section 5.6) before any write.

EI-5C and EI-5E use in-memory, labelled test doubles only.

## 10. Infrastructure plan (EI-6; not implemented)

| Concern | Plan |
| --- | --- |
| Placement | The same SQLite database authority as the EI-2B, EI-3C, EI-4D and EI4-D10 data. A foreign key from the issuance to the EI-4D disposition; lifecycle events reference the issuance |
| Tables | Issuances; completeness declarations; lifecycle events. Insert-only triggers on all three |
| Constraints | Unique issuance identity, generation key, (strategy, listing, trading date) and recording sequence; unique (issuance, event sequence); a `GLOB 'eiris1-sha256:?*'` check on the issuance identity (C-1, amended 2026-11-12) |
| Transactions | One `BEGIN IMMEDIATE` per issuance, declaration or lifecycle append. Inside it: read the maximum declaration, check uniqueness and backdating, assign the next sequence, insert, commit once |
| Concurrency | SQLite serialization and constraints, never in-memory locks. One authority; no multi-host claim |
| Commit uncertainty | Reload by issuance identity, or by issuance and event sequence, and verify (EI4-D9-G pattern) |
| Verification on load | Re-derive the issued record from the verified proposal; rehydrate events through Core; the history view satisfies the existing contract's validation |
| Pre-connection refusal | Missing or mismatched authorizations are refused before a connection opens (ADR-023 pattern) |
| Cleanup reliability | The guarded close, rollback and discard pattern of the L9 fixes |
| Acceptance | Restart, separate OS processes (spawned), concurrent issuers and declarers, lock contention, uncertain COMMIT and tampering, on real SQLite |

## 11. Slices and ownership

| Slice | Repository | Delivers | Prerequisites |
| --- | --- | --- | --- |
| EI-5A | northstar-docs | The EI-5 Decision Register, ADR-024, this design, and dated notes | — |
| EI-5B | northstar-core | Section 5 and its K and A scenarios | The EI-5A documentation PR merged (amended 2026-11-12; ADR-024, the design and O-1 approved) |
| EI-5C | northstar-application | Section 6 with test doubles; C scenarios for lifecycle | EI-5B merged |
| EI-4C versioned compatibility (amended 2026-11-12) | northstar-application | The new EI-4C assessment version with the `t − 1 µs` history boundary (section 8.4); version 1 unchanged and still verifiable | Its own approved design, settling R-1 and R-2. A required dependency, not an approved implementation PR |
| EI-5D | northstar-application | Sections 7 and 8, and the history source; C scenarios for issuance and history | The EI-4C versioned compatibility prerequisite merged (amended 2026-11-12; C-1 and C-2 approved) |
| EI-5E | northstar-application | E scenarios, from a verified proposal through issuance, history and lifecycle, on test doubles | EI-5C, EI-5D |
| EI-6 | northstar-infrastructure | Sections 9 and 10; S scenarios; monitoring later | EI-5E; O-2 and O-3 before monitoring |

EI-5 depends on EI-4, whose formal closure is pending CI verification ([EI-4 Decision Register, section 15](../product/Equity-Intraday-V1-EI4-Decision-Register.md#15-ei-4-closure-dispositions-2026-11-12)). Whether EI-5B may start first is an owner decision.

> **Amended 2026-11-12.** The owner decided that EI-5B may begin after the EI-5A documentation PR is merged.
>
> - EI-4's formal closure stays pending independent CI verification, and this decision waives no CI requirement.
> - The approved order is EI-5A, EI-5B, EI-5C, the EI-4C versioned compatibility prerequisite, EI-5D, EI-5E, then EI-6.

## 12. Acceptance matrix

**Status for every row: PLANNED.** Nothing here is executed or verified. *Amended 2026-11-12:* no row is blocked by an undecided C-1, C-2 or O-1. RV-04 is marked "DEFERRED: awaits O-3", and IS-16 "requires the EI-4C versioned compatibility prerequisite".

Layers:

- **K:** Core unit test.
- **C:** Application contract test with test doubles.
- **E:** Application synthetic end-to-end contract test.
- **S:** Infrastructure test on real SQLite.
- **A:** Architecture or source scan.

The shared synthetic example is that of [EI-5 Decision Register, section 4.1](../product/Equity-Intraday-V1-EI5-Decision-Register.md#41-synthetic-example-conventions): zone `[100.00, 100.50]`, stop 99.00, target 103.00, `eligible_from` 09:45, latest entry opening 11:00, `E` 11:05, `modeled_exit_at` 15:00.

### 12.1 Permitted transitions (LT)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| LT-01 | `RECORDED`; snapshot `as_of` 09:46:10 | `ENTRY_WINDOW_OPENED` → `WAITING_FOR_ENTRY` | K, C | EI-5B, EI-5C | PLANNED |
| LT-02 | Waiting; bar 10:00 to 10:05, low 100.10, high 100.60 | `ENTRY_ACTIVATED` → `ACTIVE` | K, C | EI-5B, EI-5C | PLANNED |
| LT-03 | Waiting; bar low 98.90, high 100.20 (entry and stop) | `INDETERMINATE_ENTRY`, `UNKNOWN` | K | EI-5B | PLANNED |
| LT-04 | Waiting; bar low 100.40, high 103.20 (entry and target) | `INDETERMINATE_ENTRY`, `UNKNOWN` | K | EI-5B | PLANNED |
| LT-05 | Waiting; bar low 98.80, high 103.30 (all three) | `INDETERMINATE_ENTRY`, `UNKNOWN` | K | EI-5B | PLANNED |
| LT-06 | Waiting; bar 09:50 to 09:55, low 98.95, high 99.80 | `CANCELLED_BEFORE_ENTRY`, `STOP_TOUCHED_BEFORE_ENTRY` | K, C | EI-5B, EI-5C | PLANNED |
| LT-07 | Waiting; every entry-window bar `ACCEPTABLE` and untouched | `EXPIRED_BEFORE_ENTRY` on the bar opening at 11:00 | K, C | EI-5B, EI-5C | PLANNED |
| LT-08 | Waiting; bar 10:30 to 10:35 `MISSING` in a snapshot at 11:06 | `INCOMPLETE_EVIDENCE` | K, C | EI-5B, EI-5C | PLANNED |
| LT-09 | Active; bar low 101.00, high 103.00; and (amended 2026-11-12, O-1) the gap-through bar low 103.50, high 104.00 | `CLOSED_TARGET` for both; no fill or exit price | K, C | EI-5B, EI-5C | PLANNED |
| LT-10 | Active; bar low 99.00, high 100.20; and (amended 2026-11-12, O-1) the gap-through bar low 98.00, high 98.50 | `CLOSED_STOP` for both; no stop-order fill or exit price | K, C | EI-5B, EI-5C | PLANNED |
| LT-11 | Active; bar 10:15 to 10:20, low 98.90, high 103.10 | `INDETERMINATE_EXIT`, `UNKNOWN`, no exit price | K, C | EI-5B, EI-5C | PLANNED |
| LT-12 | Active; complete through 14:55 to 15:00, no touch | `SESSION_CUTOFF_REACHED`, no exit price | K, C | EI-5B, EI-5C | PLANNED |
| LT-13 | Active; bar 13:15 to 13:20 `MISSING` in a snapshot at 15:02 | `INCOMPLETE_EVIDENCE`, listing `MISSING` at 13:15 to 13:20 | K, C | EI-5B, EI-5C | PLANNED |

### 12.2 Forbidden transitions (FT)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| FT-01 | `RECORDED` → `ACTIVE` or any state other than `WAITING_FOR_ENTRY` | Refused at construction | K | EI-5B | PLANNED |
| FT-02 | `RECORDED` → any terminal state | Refused | K | EI-5B | PLANNED |
| FT-03 | `WAITING_FOR_ENTRY` → `CLOSED_TARGET`, `CLOSED_STOP`, `INDETERMINATE_EXIT` or `SESSION_CUTOFF_REACHED` | Refused | K | EI-5B | PLANNED |
| FT-04 | `ACTIVE` → `WAITING_FOR_ENTRY` or `RECORDED` | Refused | K | EI-5B | PLANNED |
| FT-05 | `ACTIVE` → `EXPIRED_BEFORE_ENTRY`, `CANCELLED_BEFORE_ENTRY` or `INDETERMINATE_ENTRY` | Refused | K | EI-5B | PLANNED |
| FT-06 | Any state-changing event after each of the eight terminal states | Refused (parameterized over all eight) | K | EI-5B | PLANNED |
| FT-07 | `INVALIDATED` | Not representable | K | EI-5B | PLANNED |
| FT-08 | `EXPIRED_AFTER_ACTIVATION`, `EXPIRED_WITH_INCOMPLETE_COVERAGE`, `MODELED_STOP_FIRST` | Not representable; no Futures import | K, A | EI-5B | PLANNED |
| FT-09 | A shape mismatch: activation touching the stop, an indeterminate exit with `NOT_REQUIRED`, a cancellation without its reason | Refused | K | EI-5B | PLANNED |

### 12.3 Ambiguity and pre-entry invalidation (AM, PE)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| AM-01 | Same-bar entry and stop (EI4-D5 clarification) | `INDETERMINATE_ENTRY`, never `CANCELLED_BEFORE_ENTRY` | K | EI-5B | PLANNED |
| AM-02 | Same-bar stop and target while active (EI0-D9 example) | `INDETERMINATE_EXIT`, never `CLOSED_*` | K | EI-5B | PLANNED |
| AM-03 | Indeterminate events | Ordering `UNKNOWN`; no price field exists | K, A | EI-5B | PLANNED |
| PE-01 | Stop-only touch while waiting | `CANCELLED_BEFORE_ENTRY`; no `ENTRY_ACTIVATED`; no fill | K, C | EI-5B, EI-5C | PLANNED |
| PE-02 | While waiting: target reached above the zone; and (amended 2026-11-12) a range wholly above the zone, low 100.80, high 101.20 | No event; no activation; gap-through is not transferred to entry | K | EI-5B | PLANNED |
| PE-03 | Stop-only touch on the bar opening at 11:00 | `CANCELLED_BEFORE_ENTRY` takes precedence over expiry | K | EI-5B | PLANNED |
| PE-04 | A waiting bar wholly below the stop (low 98.20, high 98.70) (amended 2026-11-12, O-1 approved) | `CANCELLED_BEFORE_ENTRY`, `STOP_TOUCHED_BEFORE_ENTRY`; no activation; no fill | K | EI-5B | PLANNED |

### 12.4 Entry-window boundaries (EW)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| EW-01 | The bar 09:40 to 09:45 touches the zone | Never evaluated; no event | K, C | EI-5B, EI-5C | PLANNED |
| EW-02 | The bar opening exactly at 09:45 touches the zone | `ACTIVE` | K | EI-5B | PLANNED |
| EW-03 | The bar opening exactly at 11:00 touches the zone | `ACTIVE` | K | EI-5B | PLANNED |
| EW-04 | The bar 11:05 to 11:10 touches the zone while waiting | No activation; expiry occurred at 11:00 to 11:05 | K, C | EI-5B, EI-5C | PLANNED |
| EW-05 | The activation bar closes at `modeled_exit_at` (a synthetic policy with latest opening 14:55) | `ENTRY_ACTIVATED`, then `SESSION_CUTOFF_REACHED` on the same interval | K | EI-5B | PLANNED |
| EW-06 | The activation bar does not count for exits | Exit evaluation starts with the next bar | K | EI-5B | PLANNED |
| EW-07 | A bar 15:00 to 15:05 is present and touches the target | Never read | K, C | EI-5B, EI-5C | PLANNED |

### 12.5 Deficient evidence and temporary blocks (EV, TB)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| EV-01 | Cursor interval `MISSING` before the boundary | `BLOCKED`; no event | C | EI-5C | PLANNED |
| EV-02 | `UNFINALIZED` | `BLOCKED` | C | EI-5C | PLANNED |
| EV-03 | `CONFLICTING_VALUES` | `BLOCKED` | C | EI-5C | PLANNED |
| EV-04 | `UNRESOLVED_REVISIONS` | `BLOCKED` | C | EI-5C | PLANNED |
| EV-05 | `NOT_ENDED` | `AWAITING_EVIDENCE`; no block | C | EI-5C | PLANNED |
| EV-06 | 09:55 `MISSING`; later bars `ACCEPTABLE`, one touching the stop | No progress past 09:55; the later touch is unused | C | EI-5C | PLANNED |
| EV-07 | Two deficient intervals at the boundary | `INCOMPLETE_EVIDENCE` lists both, with their EI-2A outcome names | K, C | EI-5B, EI-5C | PLANNED |
| TB-01 | Deficiency at 10:30; snapshot at 10:45 | `BLOCKED`; state unchanged | C | EI-5C | PLANNED |
| TB-02 | The 10:30 interval `ACCEPTABLE` in a snapshot at 10:50 | Monitoring resumes at 10:30 | C | EI-5C | PLANNED |
| TB-03 | Waiting; deficiency persists in a snapshot at or after 11:05. Amended 2026-11-12 (exact boundary): at exactly 11:05 the boundary bar 11:00 to 11:05 `UNFINALIZED` or `MISSING`; `ACCEPTABLE`; an earlier interval (10:30) `MISSING`; after 11:05, still deficient | `INCOMPLETE_EVIDENCE` after 11:05 and for the earlier interval; at exactly 11:05, `BLOCKED` with no event for the boundary bar not yet final, and normal evaluation when `ACCEPTABLE` | K, C | EI-5B, EI-5C | PLANNED |
| TB-04 | Active; deficiency persists in a snapshot at or after 15:00. Amended 2026-11-12 (exact boundary): at exactly 15:00 the boundary bar 14:55 to 15:00 `MISSING` or `UNFINALIZED`; `ACCEPTABLE`; `CONFLICTING_VALUES`; after 15:00, still deficient | `INCOMPLETE_EVIDENCE` after 15:00 and for `CONFLICTING_VALUES`; at exactly 15:00, `BLOCKED` with no event for the boundary bar not yet final; stop, target, both or cutoff when `ACCEPTABLE` | K, C | EI-5B, EI-5C | PLANNED |
| TB-05 | No snapshot at or after the boundary is evaluated | Non-terminal state unchanged; nothing assumed; no clock read | C, A | EI-5C | PLANNED |

### 12.6 Session end and no look-ahead (SE, LA)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| SE-01 | Cutoff outcome | Unresolved; outcome class `UNRESOLVED`; no exit price | K | EI-5B | PLANNED |
| SE-02 | The bar 14:55 to 15:00 touches the stop | `CLOSED_STOP`, not the cutoff | K | EI-5B | PLANNED |
| SE-03 | A snapshot of the next trading date | Refused; no overnight carry-over | C | EI-5C | PLANNED |
| SE-04 | Outcome classes | Indeterminate and unresolved states are never a win or loss | K | EI-5B | PLANNED |
| LA-01 | A snapshot at 10:12 | Uses only intervals ended by 10:12 | C | EI-5C | PLANNED |
| LA-02 | A snapshot older than the last evaluated | Refused | C | EI-5C | PLANNED |
| LA-03 | The same `as_of` with a different snapshot; then the identical snapshot again | Refused; idempotent | C | EI-5C | PLANNED |
| LA-04 | Snapshot `as_of` before the modeled `issued_at` | Refused | C | EI-5C | PLANNED |
| LA-05 | Another listing or research-window version | Refused | C | EI-5C | PLANNED |
| LA-06 | Journal watermark regression | Refused | C | EI-5C | PLANNED |
| LA-07 | Every event cites the snapshot and interval; `evaluated_at` is the snapshot's `as_of`; no clock call | Verified | C, A | EI-5C | PLANNED |

### 12.7 Revisions (RV)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| RV-01 | `CLOSED_TARGET` on 11:15 to 11:20; a correction visible at 16:00 | `SOURCE_REVISION_RECORDED`, `UNASSESSED`; state unchanged | K, C | EI-5B, EI-5C | PLANNED |
| RV-02 | A cited bar corrected while active | Revision event; earlier events unchanged; monitoring continues | C | EI-5C | PLANNED |
| RV-03 | The same revision seen again | No second event | C | EI-5C | PLANNED |
| RV-04 | Late evidence fills an interval cited by `INCOMPLETE_EVIDENCE` | State unchanged; whether it is recorded is undecided | C | EI-5C | PLANNED — DEFERRED: awaits O-3 |
| RV-05 | Any revision | Never `INVALIDATED`; never a state change | K | EI-5B | PLANNED |

### 12.8 Issuance (IS)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| IS-01 | A verified `PROPOSED` disposition | `ISSUED`, MODELED; state `RECORDED` | C, E | EI-5D, EI-5E | PLANNED |
| IS-02 | Issue again | `EQUIVALENT`; nothing written | C, E | EI-5D, EI-5E | PLANNED |
| IS-03 | Another policy set; same strategy, listing and session | `SESSION_STRATEGY_ALREADY_ISSUED` | C, E | EI-5D, EI-5E | PLANNED |
| IS-04 | Another strategy; same listing and session | `ISSUED` | C | EI-5D | PLANNED |
| IS-05 | A `NOT_ELIGIBLE` disposition | Refused; no write | C | EI-5D | PLANNED |
| IS-06 | An `UNRESOLVED` disposition | Refused; no write | C | EI-5D | PLANNED |
| IS-07 | An unknown generation key | Refused; no write | C | EI-5D | PLANNED |
| IS-08 | Stored record invalid at level A | Permanent refusal; record preserved; no write | C | EI-5D | PLANNED |
| IS-09 | Source evidence unavailable | Retryable refusal; no write | C | EI-5D | PLANNED |
| IS-10 | Replay mismatch at level C | Permanent refusal; no write | C | EI-5D | PLANNED |
| IS-11 | `ACTUAL` requested | Refused before any read | C | EI-5D | PLANNED |
| IS-12 | The issued record | Non-actionable; `not_assessed` controls identical to the proposal's | C | EI-5D | PLANNED |
| IS-13 | Timing instants | Equal to the proposal's; no caller parameter exists | C, A | EI-5D | PLANNED |
| IS-14 | A writer call without a verified authorization | Refused | C, A | EI-5D | PLANNED |
| IS-15 | An authorization with swapped content | Refused | C | EI-5D | PLANNED |
| IS-16 | (Amended 2026-11-12, C-2 approved) A version-1 disposition; or new-version history evidence that is from a fixture, is not taken as of `t − 1 µs`, or does not reproduce at its watermark; or a declared `complete_through` beyond `t − 1 µs` | Refused; the disposition is preserved; no write | C | EI-5D | PLANNED — requires the EI-4C versioned compatibility prerequisite |
| IS-17 | After issuance | The EI-4D disposition and attempts are byte-identical; no disposition is `RECORDED` | C, E | EI-5D, EI-5E | PLANNED |

### 12.9 Identity (ID)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| ID-01 | Golden vector from fixed synthetic parts | Exact expected text | C | EI-5D | PLANNED |
| ID-02 | Two spawned processes, and a restart | Identical identity | C, S | EI-5D, EI-6 | PLANNED |
| ID-03 | Prefix `eiris1-sha256:` (amended 2026-11-12, C-1 approved) | Distinct from every EI-4 prefix; matched by `GLOB 'eiris1-sha256:?*'` and not by `GLOB 'eiri1-sha256:?*'` | C, S | EI-5D, EI-6 | PLANNED |
| ID-04 | Change the generation key, the proposal fingerprint or the kind | A different identity each time | C | EI-5D | PLANNED |
| ID-05 | Part boundaries shifted by concatenation | A different identity (length prefixing) | C | EI-5D | PLANNED |

### 12.10 Concurrency, backdating and history (CC, BD, HW)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| CC-01 | Two processes issue the same disposition | One row; the other `EQUIVALENT` | S | EI-6 | PLANNED |
| CC-02 | Two processes issue different policy sets for one strategy, listing and session | Exactly one `ISSUED` | S | EI-6 | PLANNED |
| CC-03 | An issuance concurrent with a declaration covering its `issued_at` | Either listed in the view, or refused as backdated; never silently outside it | S | EI-6 | PLANNED |
| CC-04 | Concurrent declarations | `complete_through` monotone; regression refused | S | EI-6 | PLANNED |
| CC-05 | Write lock not acquired | Nothing written; typed unavailable | S | EI-6 | PLANNED |
| CC-06 | COMMIT raises after taking effect | Reload by identity resolves it as committed | S | EI-6 | PLANNED |
| BD-01 | `issued_at` 10:10; declared through 10:30 | `BACKDATED_ISSUANCE` | C, S | EI-5D, EI-6 | PLANNED |
| BD-02 | `issued_at` exactly equal to `complete_through` | `BACKDATED_ISSUANCE` | C | EI-5D | PLANNED |
| BD-03 | `issued_at` 10:40, declared through 10:30; and (amended 2026-11-12, C-2) `issued_at` `t`, declared through exactly `t − 1 µs` | `ISSUED` in both cases | C | EI-5D | PLANNED |
| BD-04 | No declaration for the session | `ISSUED`; no completeness claimed | C | EI-5D | PLANNED |
| HW-01 | No declaration | `session_history` is `None`, never empty | C | EI-5D | PLANNED |
| HW-02 | A declaration and no issuance | An explicit empty tuple | C | EI-5D | PLANNED |
| HW-03 | The A, B, declaration, C example of EI5-D4 | The view at watermark 3 lists exactly A and B, before and after C | C, S | EI-5D, EI-6 | PLANNED |
| HW-04 | View content | Exactly sequence at or below `W` and `issued_at` at or before `CT(W)`, in (`issued_at`, key) order | C | EI-5D | PLANNED |
| HW-05 | A lower declaration; then an equal one | `COMPLETENESS_REGRESSION`; `DECLARATION_EQUIVALENT` | C | EI-5D | PLANNED |
| HW-06 | A caller tries to supply a sequence or watermark | No such parameter exists | C, A | EI-5D | PLANNED |
| HW-07 | `session_view_at` beyond the current watermark | Refused | C | EI-5D | PLANNED |
| HW-08 | Authority unreadable | `EquityRecommendationHistoryUnavailableError` | C, S | EI-5D, EI-6 | PLANNED |
| HW-09 | Every produced view | Passes the existing `EquitySessionRecommendationHistory` validation | C | EI-5D | PLANNED |
| HW-10 | A recommendation closed by its lifecycle | Still listed and treated as open by EI-4C (EI4-D8 unchanged) | E | EI-5E | PLANNED |

### 12.11 Restart, replay, no execution and architecture (RR, NB, AR)

| ID | Scenario | Expected | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- |
| RR-01 | Lifecycle events reloaded after restart | The same state through Core replay | S | EI-6 | PLANNED |
| RR-02 | The same snapshot sequence evaluated twice from `RECORDED` | Identical events | C, E | EI-5C, EI-5E | PLANNED |
| RR-03 | Issuances and declarations after restart | Identical views at every watermark | S | EI-6 | PLANNED |
| RR-04 | Replay from a separate OS process | Identical state and views | S | EI-6 | PLANNED |
| RR-05 | A stored event tampered in a test | Refused on reload; preserved | S | EI-6 | PLANNED |
| RR-06 | Two evaluators append from the same position | One succeeds; the other is refused by compare-and-append | C, S | EI-6 | PLANNED |
| NB-01 | Every event and issued record | No order, fill, quantity filled, broker account, P&L or exit price | K, C, A | EI-5B, EI-5D | PLANNED |
| NB-02 | `SESSION_CUTOFF_REACHED`, `INDETERMINATE_*` and `INCOMPLETE_EVIDENCE` | No price | K | EI-5B | PLANNED |
| NB-03 | Lifecycle and issuance packages | Import no execution, broker or paper-trading module | A | EI-5B, EI-5D | PLANNED |
| NB-04 | `ACTIVE` | No position or paper record is created anywhere | E, A | EI-5E | PLANNED |
| AR-01 | The Core lifecycle package | Imports nothing from the Application, Infrastructure or Futures | A | EI-5B | PLANNED |
| AR-02 | Core lifecycle and Application evaluation | No clock or randomness call | A | EI-5B, EI-5C | PLANNED |
| AR-03 | Futures and Options test suites | Unchanged and passing in every EI-5 slice | A | All | PLANNED |

### 12.12 Scenario counts

*Amended 2026-11-12.* C-1, C-2 and O-1 are approved, so PE-04, IS-16 and ID-03 are no longer blocked by an owner decision. The total is unchanged.

| Group | Scenarios | Awaiting a deferred owner decision | Needs a prerequisite implementation first |
| --- | --- | --- | --- |
| LT | 13 | 0 | 0 |
| FT | 9 | 0 | 0 |
| AM, PE | 7 | 0 | 0 |
| EW | 7 | 0 | 0 |
| EV, TB | 12 | 0 | 0 |
| SE, LA | 11 | 0 | 0 |
| RV | 5 | 1 (RV-04, O-3 deferred) | 0 |
| IS | 17 | 0 | 1 (IS-16: the EI-4C versioned compatibility) |
| ID | 5 | 0 | 0 |
| CC, BD, HW | 20 | 0 | 0 |
| RR, NB, AR | 13 | 0 | 0 |
| **Total** | **119** | **1** | **1** |

- **Status.** Every row is PLANNED. Policy approval is not implementation readiness, and no row is executed or verified.
- **EI-5D rows.** Every EI-5D row (IS, ID, BD, HW and the EI-5D parts of CC and RR) is implementable only after the EI-4C versioned compatibility prerequisite is merged. IS-16 is the row that tests it directly.
- **O-2.** It affects no row's expected result. It concerns how EI-6 monitoring chooses snapshots.

## 13. Conflicts and open points

They are recorded with evidence, options and recommendations in the [EI-5 Decision Register, section 6](../product/Equity-Intraday-V1-EI5-Decision-Register.md#6-conflicts-and-open-points-found-during-ei-5a):

| ID | Subject | Blocks | Status (amended 2026-11-12) |
| --- | --- | --- | --- |
| C-1 | The approved `eiri1` prefix is already the frozen-inputs prefix | EI-5D | **APPROVED:** `eiris1` (section 7.4) |
| C-2 | Inclusive completeness at the modeled issuance, with no backdating, blocks issuance | EI-5D | **APPROVED:** `t − 1 µs` boundary in a new EI-4C version (section 8.4); prerequisite before EI-5D, not implemented; R-1 and R-2 open for its design |
| O-1 | Gap-through touches | EI-5B stop and target predicates | **APPROVED:** stop and target reach, with no entry transfer (sections 4, 5.4) |
| O-2 | Terminal `INCOMPLETE_EVIDENCE` depends on the evaluated snapshot | EI-6 monitoring | **DEFERRED** |
| O-3 | Late evidence after terminal `INCOMPLETE_EVIDENCE` | EI-6 | **DEFERRED** |

## 14. Risks and limitations

- **Snapshot selection** decides terminal `INCOMPLETE_EVIDENCE` (O-2). EI-5 tests use explicit synthetic snapshots only.
- **No snapshot enumeration.** Callers must hold snapshot identities. EI-6 monitoring must establish how snapshots are found, without look-ahead.
- **Conservative exclusivity.** EI4-D8 still treats every issued recommendation as open.
- **Trust model.** Personal-use, single-host SQLite. No protection against privileged code or direct file modification (ADR-024, section 14).
- **Inherited EI-4 limitations.** Windows-only process evidence, unverified CI, and no separate-process replay. They are not removed by EI-5.
- **Strict issuance protocol** (amended 2026-11-12). A proposal at `t` is issuable only while its session is declared complete through exactly `t − 1 µs`. EI-6 monitoring must declare and issue in ascending modeled-issuance order, or proposals become permanently non-issuable, failing closed.
- **Two-version verification** (amended 2026-11-12, R-1). The EI-4C prerequisite must keep version-1 records verifiable. That is not yet supported, and ADR-021, section 10 excludes an evaluator registry today, so the prerequisite's design needs explicit architecture approval for it.
- **Gap-through scope** (amended 2026-11-12). The pre-entry use of the stop predicate (PE-04) follows from option (a) as approved. If the owner limits gap-through to `ACTIVE` recommendations, PE-04 changes to "no event".

## 15. Explicit non-authorizations

This design authorizes no implementation. It does not authorize:

- ACTUAL issuance or an actionable recommendation;
- `INVALIDATED`;
- orders, fills, positions or exits;
- monitoring jobs, schedulers or composition roots;
- API or Web;
- real data;
- production cutoffs or latencies;
- changes to EI-4 records, identities or formats;
- changes to Futures or Options behaviour.

The 09:15 to 15:00 IST research window is not a production entry, exit or trading authorization.

## 16. Related documents

- [EI-5 Decision Register](../product/Equity-Intraday-V1-EI5-Decision-Register.md)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md)
- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)
- [EI4-D11 Mandatory Operational Generation-Write Boundary Enforcement Design](Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md)
- [Futures Trade Recommendation Design Specification](Futures-Trade-Recommendation-Design-Specification-v1.0.md) (pattern only)

## 17. Owner approval (2026-11-12)

### 17.1 Approved decisions

| ID | Decision | Incorporated in |
| --- | --- | --- |
| C-1 | Issuance identity prefix `eiris1-sha256:`. Namespace `northstar.equity-recommendation-issuance-key/1`. The frozen-inputs `eiri1-sha256:` is unchanged | Sections 7.4, 10, 12 (ID-03) |
| C-2 | A new, versioned EI-4C assessment requires history complete strictly before the modeled issuance (`t − 1 µs`). It is a required prerequisite before EI-5D, needing its own approved design | Sections 7.1, 8.4, 11, 12 (IS-16, BD-03) |
| O-1 | Stop reached when `low <= stop`, target reached when `high >= target`, gap-through included. The entry zone keeps the intersection rule. Ambiguity precedence and `UNKNOWN` ordering are unchanged | Sections 4, 5.4, 12 (LT-09, LT-10, PE-02, PE-04) |
| Design and ADR-024 | Approved with the amendments above | This document; [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md) |
| EI-5B sequencing | EI-5B may begin after the EI-5A documentation PR is merged | Section 11 |

### 17.2 Status

| Item | Status |
| --- | --- |
| Architecture and engineering design | **APPROVED** (2026-11-12, with amendments) |
| Engineering implementation (EI-5B to EI-5E, EI-6) | **NOT STARTED** |
| EI-4C versioned compatibility prerequisite | **NOT STARTED**; own design and approval needed (R-1, R-2) |
| Acceptance scenarios | **PLANNED** (119; none executed) |
| O-2, O-3 | **DEFERRED** |
| EI-4 formal closure | **PENDING** independent CI verification |
| Operational or production authorization | **NOT GRANTED** |

The full record is in the [EI-5 Decision Register, section 12](../product/Equity-Intraday-V1-EI5-Decision-Register.md#12-owner-approval-of-the-ei-5a-amendments-2026-11-12).
