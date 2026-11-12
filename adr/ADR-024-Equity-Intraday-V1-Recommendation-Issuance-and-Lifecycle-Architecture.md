# ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture

## Status

Accepted

On 2026-11-12 the product owner approved ten EI-5 policy decisions: EI0-D9, EI4-D5 and EI5-D1 to EI5-D8 ([EI-5 Decision Register](../product/Equity-Intraday-V1-EI5-Decision-Register.md)). This ADR records the architecture those decisions require. It was prepared as **Proposed** for owner review in the EI-5A documentation milestone.

> **Note, 2026-11-12 (owner acceptance with amendments).** The owner accepted this ADR on 2026-11-12, with three approved amendments ([EI-5 Decision Register, section 12](../product/Equity-Intraday-V1-EI5-Decision-Register.md#12-owner-approval-of-the-ei-5a-amendments-2026-11-12)). Each is incorporated in the sections marked "amended 2026-11-12":
>
> - **C-1:** the issuance identity prefix `eiris1` (section 5);
> - **C-2:** an exclusive issued-history boundary for a new, versioned EI-4C assessment, which is a prerequisite before EI-5D (sections 7 and 12);
> - **O-1:** gap-through recognition for the stop and target (section 8).
>
> O-2 and O-3 stay deferred.

| Item | Status |
| --- | --- |
| Governing policy decisions (EI0-D9, EI4-D5, EI5-D1 to EI5-D8) | OWNER APPROVED (2026-11-12) |
| C-1, C-2, O-1 | OWNER APPROVED (2026-11-12) |
| This architecture record | **ACCEPTED** (owner, 2026-11-12, with amendments) |
| EI-4C versioned history-boundary compatibility (prerequisite before EI-5D) | **NOT STARTED**; needs its own approved design |
| Implementation (EI-5B to EI-5E, EI-6) | **NOT STARTED** |
| Acceptance | **PLANNED**; no scenario executed |
| O-2, O-3 | **DEFERRED** |
| Operational or production authorization | **NOT GRANTED** |

This is a design decision. **Nothing in it is implemented**, and nothing in it has been tested. An accepted ADR is not completed code. It records no Architecture Review or Engineering Review sign-off.

> **Note, 2026-11-13 (ADR-025 and milestone status).** This ADR stays **accepted** and unchanged in substance.
>
> - **The prerequisite's architecture.** The EI-4C versioned compatibility prerequisite now has its accepted architecture, [ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md):
>   - R-1 and R-2 are resolved;
>   - a closed two-version evaluator table;
>   - retained prefixes with `/2` inputs, assessment and attempt formats;
>   - mandatory version-1 golden vectors;
>   - the N10 rule that version-2 generation records `PROPOSED` only when `complete_through == t − 1 µs`.
> - **Not implemented.** The prerequisite's implementation is **NOT STARTED**, and EI-5D still requires its acceptance ([EI-5 Decision Register, section 13](../product/Equity-Intraday-V1-EI5-Decision-Register.md#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13)).
> - **Merged since this table was written** ([section 13.6](../product/Equity-Intraday-V1-EI5-Decision-Register.md#136-milestone-status-correction)):
>   - EI-5B (northstar-core #39, `ccff26e`);
>   - EI-5C (northstar-application #93, `75c6a35`).
>
>   They have no Docs acceptance record and no recorded independent CI evidence.
> - **Status table.** The table and paragraph above record the status on 2026-11-12.

## Date

2026-11-12

## Decision Makers

- Product owner: approved EI0-D9, EI4-D5 and EI5-D1 to EI5-D8 on 2026-11-12. On the same date, approved C-1, C-2 and O-1 and accepted this ADR with those amendments.
- Northstar Architecture Team: formal review pending.

## Context

### Problem

EI-4 produces immutable generation records: a stored `PROPOSED` disposition is a synthetic proposal, never an issued recommendation and never `RECORDED` (EI4-D9-A, EI4-D9-J). Nothing yet turns a verified proposal into an issued recommendation, records an authoritative issued history, or follows an issued recommendation through its intraday lifecycle. EI-5 owns that policy (EI4-D9-J), and EI-6 owns its durable records.

### What exists, read at core `bc081a2`, application `8937f12`, infrastructure `c105e94`

- **Generation records (EI-4).**
  - Dispositions, proposals and attempts are stored immutably in one SQLite authority (ADR-021).
  - They are written only through the verified recorder (ADR-023).
  - They are re-verified at levels A to C by `VerifyStoredEquityRecommendationGenerationUseCase` ([ADR-021, section 10](ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md#10-restart-and-historical-verification-ei4-d9-i)).
- **The proposal carries the whole lifecycle envelope.** The canonical `eirp1` proposal document holds:
  - the Core definition and levels: `EquityEntryZone` `[low, high]`, stop and target;
  - `EquityRecommendationTiming` (`decision_as_of`, `issuance_kind`, `issued_at`, `eligible_from`), with MODELED issuance derived by EI4-D9-C;
  - `latest_entry_bar_opens_at`, `modeled_exit_at` and the timing-policy version;
  - the controls that are not assessed.

  EI-4C already confines `latest_entry_bar_opens_at` to an opening, and `modeled_exit_at` to a close, of the proposal's research-window grid.
- **The issued-history read contract.** It already exists in the Application, in `ports/equity_recommendation_references.py`:
  - `EquitySessionRecommendationHistory`, with `complete_through`, `recording_watermark` and `issued_by(instant)`;
  - `EquityPriorSessionRecommendation`, with `recommendation_key`, `issuance_kind`, `issued_at` and `recording_sequence`;
  - `EquitySessionRecommendationHistorySource`.

  Only labelled synthetic fixtures implement the source: no issued-recommendation authority exists (EI4-D9-H).
- **Evidence snapshots (EI-2B).** `EquityCandleEvidenceSnapshot` (`ecs1`) freezes, for one listing, date and `as_of`, every grid interval's outcome: `NOT_ENDED`, `MISSING`, `ACCEPTABLE`, `UNFINALIZED`, `CONFLICTING_VALUES` or `UNRESOLVED_REVISIONS`. It also records the acceptable observations and the journal watermark. The snapshot store exposes `issue` and `get_snapshot(snapshot_id)` only; there is no query by session.
- **The Futures lifecycle (Core).** It is an engineering precedent only:
  - insert-only events with allowed (kind, touched, ordering) shapes;
  - compare-and-append store steps;
  - coverage deficiencies;
  - a modeled stop-first rule and an expired-after-activation state, which equities do not adopt.
- **No equity lifecycle exists** in Core or the Application.

## Decision

### 1. Approved decisions

| ID | Decision (owner approved 2026-11-12) | Section |
| --- | --- | --- |
| EI0-D9 | `UNKNOWN` intrabar ordering; terminal `INDETERMINATE_ENTRY` and `INDETERMINATE_EXIT`; no Futures stop-first | 8, 10 |
| EI4-D5 | A stop-only touch while waiting gives `CANCELLED_BEFORE_ENTRY` (`STOP_TOUCHED_BEFORE_ENTRY`); ambiguity takes precedence | 8 |
| EI5-D1 | Separate, verified, MODELED-only issuance; one per generation key, and one per listing, session and strategy | 2, 3, 4 |
| EI5-D2 | `RECORDED` begins at durable issuance; no clock | 8 |
| EI5-D3 | A separate versioned issuance identity | 5 |
| EI5-D4 | Append-only authority, monotone sequence, completeness declarations, watermark, no backdating | 6, 7 |
| EI5-D5 | The equity state model and transitions; `INVALIDATED` out of scope | 8 |
| EI5-D6 | Entry window, interval ordering and evidence citation | 9 |
| EI5-D7 | Session outcomes; when a block becomes terminal | 10 |
| EI5-D8 | Snapshot-instant evaluation, immutable terminal states, revision events | 10, 11 |

### 2. Separation of generation and issuance

- Generation (EI-4) and issuance (EI-5) are separate operations with separate records. Generating a proposal never issues it, and issuing it never changes it.
- Issuance takes a **generation key** and nothing the proposal already fixes. Every issued field is copied from the re-verified stored proposal, never supplied by a caller:
  - the timing instants (`issued_at`, `eligible_from`, `latest_entry_bar_opens_at`, `modeled_exit_at`);
  - the levels;
  - the strategy, listing and trading date;
  - the controls that are not assessed.
- An issued recommendation refers back to its exact `eirk1` generation key and `eirp1` proposal fingerprint (EI4-D9-J).
- EI-4's disposition kinds, canonical formats, identities and stored records are unchanged. A disposition never becomes a lifecycle state.

### 3. MODELED-only issuance

- EI-5 issues `EquityIssuanceKind.MODELED` only. A request for `ACTUAL` issuance is refused before any read of the authority.
- The issued recommendation's `issued_at` is the proposal's modeled issuance (EI4-D9-C). It is event time, never the recording time (EI5-D2).
- An issued recommendation is **non-actionable**. The type carries no actionability flag that could be set, and every control EI-4C reported as not assessed stays listed as not assessed.

### 4. Issuer verification and authorization

The verified-issuer boundary follows the EI4-D11 enforcement pattern ([ADR-023, section 3](ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md#3-layered-enforcement-dq2)), applied from the start rather than retrofitted:

1. **Read/write port separation.** The issuance authority has a reader port and a writer port. Read-only consumers, such as the history source and lifecycle evaluation, receive only the reader.
2. **One operational issuer.** The issuance use case is the only supported operational path to the writer. It:
   - re-verifies the stored disposition at levels A to C;
   - refuses `NOT_ELIGIBLE`, `UNRESOLVED`, an absent key, any verification failure and `ACTUAL`;
   - derives the issued record and its identity.
3. **Content-bound authorization.** The writer accepts only an issuer-produced authorization bound to the exact issued record's content. It never binds by identity alone. Authorization contents that differ from the record are refused before any connection opens.
4. **Infrastructure validation** of the authorization happens before a database connection is opened (ADR-023, section 6 pattern).
5. **Architecture scans** in CI keep raw issuance writes out of supported Application and composition code.
6. **Database immutability** (insert-only triggers and uniqueness) is defence in depth, never the only control.

Completeness declarations (section 7) are written only through a declaration use case under the same pattern. No caller supplies a recording sequence or watermark.

### 5. Issuance identity

*Amended 2026-11-12: the prefix is `eiris1` (C-1).*

- **Derivation (EI5-D3).** The identity is `eiris1-sha256:` followed by the hexadecimal SHA-256 digest of the length-prefixed parts below, in this order. Each part is written as its UTF-8 byte length in decimal ASCII, a colon, then the bytes, exactly as the existing `eirk1` generation key is built (`equity_recommendation_generation_evidence.py`):

  | # | Part | Form |
  | --- | --- | --- |
  | 1 | Identity namespace | `northstar.equity-recommendation-issuance-key/1` |
  | 2 | Generation key | The full `eirk1-sha256:` text |
  | 3 | Proposal fingerprint | The full `eirp1-sha256:` text |
  | 4 | Issuance kind | `EquityIssuanceKind` value, `MODELED` |

- **Normalization.** Every part is already canonical text. Prefixes are lowercase ASCII and digests are lowercase hexadecimal, as produced by the existing derivations. Nothing is trimmed, case-folded or re-encoded.
- **Properties.**
  - Domain separation comes from the namespace.
  - Length prefixing makes the concatenation unambiguous.
  - The identity is stable across processes and restarts, because no clock, sequence, path or process value enters it.
  - It is distinct from `eirk1` by construction.
- **Prefix (amended 2026-11-12).** The prefix is `eiris1-sha256:`, approved by the owner (C-1).
  - The name `eiri1`, first approved for EI5-D3, was already the frozen-inputs fingerprint prefix and is not reused ([EI-5 Decision Register, conflict C-1](../product/Equity-Intraday-V1-EI5-Decision-Register.md#c-1--the-approved-eiri1-prefix-is-already-in-use)).
  - A search of all six repositories found no existing use of `eiris1` or of the namespace.
  - The frozen-inputs fingerprint (`eiri1`), the generation key (`eirk1`), the proposal fingerprint (`eirp1`) and the issuance identity (`eiris1`) are four distinct identities ([EI-5 Decision Register, section 12.1](../product/Equity-Intraday-V1-EI5-Decision-Register.md#121-c-1--the-eiris1-issuance-identity)).
- **No content fingerprint is added.** The issued record is a pure function of the verified proposal and the identity's parts. Equal identities therefore imply equal content when both are derived from a verified proposal. A stored record that does not re-derive from its proposal is invalid, never equivalent.

### 6. Authoritative issued-history contract

- The authority produces the **existing** `EquitySessionRecommendationHistory` contract unchanged. Its records are the existing `EquityPriorSessionRecommendation`, whose `recommendation_key` is the issuance identity.
- A production implementation of the existing `EquitySessionRecommendationHistorySource` is built over the authority's reader. It replaces nothing: fixtures stay test-only.
- **Views.** For an exchange, a trading date and a watermark `W`:
  - `CT(W)` is the greatest `complete_through` declared for that session at a sequence at or below `W`;
  - the view lists exactly the issuances recorded at a sequence at or below `W` whose `issued_at` is at or before `CT(W)`, in ascending (`issued_at`, key) order.
- **Unknown, never empty.** With no declaration at or below `W`, the session's history is unknown: the source returns `None`, never an empty tuple. An empty tuple is returned only when a declaration exists and lists no issuance. A failure to read or verify raises `EquityRecommendationHistoryUnavailableError`.
- **Re-verification.** The reader also answers the view at a stated earlier watermark, so a frozen history evidence can be reproduced later. A watermark beyond the authority's current position is refused.
- **EI4-D8 is unchanged.** The history carries no lifecycle outcome, and every issued recommendation stays open for exposure and exclusivity evaluation until a separate decision relaxes it.

### 7. Recording sequence, watermark and completeness

- **One sequence.** The authority assigns one monotone `recording_sequence` (an integer, at least 1) to every issuance and every completeness declaration, inside the transaction that records it. Sequences are unique and never reused.
- **Declarations** are append-only facts: "exchange `X`, trading date `D` is complete through instant `T`". For each session:
  - a declaration below the current greatest `T` is refused as a regression;
  - an equal declaration is idempotent;
  - a greater declaration is recorded.

  `T` must lie within the session's venue day.
- **No backdating.** An issuance whose `issued_at` is at or before the session's current greatest declared `T` is refused.
- **Stability invariant.** For `W1 < W2`, the view at `W2` restricted to `issued_at <= CT(W1)` equals the view at `W1`.
  - It holds because any issuance with `issued_at <= CT(W1)` recorded after `W1` would have been refused by the no-backdating rule.
  - It holds after restarts and concurrent writes, because the check and the insert run in one serialized transaction (EI-6: one SQLite authority, `BEGIN IMMEDIATE`).
- **Uniqueness.** The authority enforces, in the same transaction and by database constraints in EI-6:
  - one issuance per issuance identity;
  - one per generation key;
  - one per (strategy, listing, trading date).
- **Conflict C-2.** Inclusive `complete_through` at the modeled issuance, which EI-4C requires, combined with this no-backdating rule, refuses every proposal generated against the authority. The recommended resolution keeps this section unchanged and makes the EI-4C assessment read the history complete strictly **before** its modeled issuance. That needs a versioned EI-4C compatibility change ([EI-5 Decision Register, conflict C-2](../product/Equity-Intraday-V1-EI5-Decision-Register.md#c-2--inclusive-completeness-at-the-modeled-issuance-blocks-issuance)). The owner decided it on 2026-11-12 (next bullet).
- **The C-2 correction (amended 2026-11-12; owner approved).** This section is unchanged. A new, versioned EI-4C assessment takes its history view as of `t − 1 µs`, where `t` is the modeled issuance. `PointInTime` resolves exactly to the microsecond, so that is exactly "complete strictly before `t`", and it uses the existing inclusive contract unchanged.
  - **Issuance window.** A proposal at `t` is issuable only while its session's greatest declared `complete_through` is exactly `t − 1 µs`. Issuance therefore proceeds in ascending modeled-issuance order: declare through `t − 1 µs`, issue at `t`, then declare further.
  - **Existing records.** Version-1 records keep their meaning and are never issuable.
  - **Prerequisite.** The change is an Application prerequisite before EI-5D, with its own design. Two contract points stay open for that design:
    - R-1, two-version verification under ADR-021, section 10;
    - R-2, fingerprint prefixes.
  - *Amended 2026-11-13.* R-1 and R-2 are resolved by [ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md).
    - **Exact completeness.** Version-2 generation additionally requires the session to be complete through **exactly** `t − 1 µs` (N10). History complete beyond it is refused at generation.
    - **Unchanged here.** This section's no-backdating rule and issuance window are unchanged.
    - **The authority's check.** The authority's own transactional check is still required.
  - **Detail:** [EI-5 Decision Register, section 12.2](../product/Equity-Intraday-V1-EI5-Decision-Register.md#122-c-2--the-exclusive-history-boundary).

### 8. Lifecycle states and pure transition model

- **Core owns the model** in a new equity-only package, `northstar_core.recommendations.equity.lifecycle`:
  - the states of EI5-D5;
  - the event kinds;
  - the allowed (kind, from-state, to-state, touched levels, ordering) shapes;
  - the bar rules and transition guards;
  - deterministic replay.

  It is pure. It reads no clock, generates no identity, and imports nothing from the Application, Infrastructure or the Futures recommendation packages.
- **The initial state** is `RECORDED`, derived from the existing issuance record and not from an event.
- **Events.** Every transition is an immutable event, and the state is the fold of the events in sequence order.
- **No other paths.** State-changing events are refused after a terminal state, as is any transition not listed in EI5-D5.
- **Level predicates** (amended 2026-11-12; O-1 approved). These use exact decimal comparison on the bar's acceptable low and high:
  - **Stop reached:** `low <= stop`;
  - **Target reached:** `high >= target`;
  - **Entry zone touched:** `low <= zone.high` and `high >= zone.low`.

  A range lying wholly beyond the stop or the target therefore reaches it (gap-through). The entry zone keeps the intersection rule: a range wholly above or below the zone activates nothing. Reaching a level is a lifecycle observation only; no fill or exit price is inferred. In the table below, "touched" for the stop or target means reached.
- **Bar rules:**

  | State | Entry zone | Stop | Target | Result |
  | --- | --- | --- | --- | --- |
  | `WAITING_FOR_ENTRY` | touched | not touched | not touched | `ACTIVE` |
  | `WAITING_FOR_ENTRY` | touched | touched | any | `INDETERMINATE_ENTRY`, ordering `UNKNOWN` (EI0-D9; EI4-D5 clarification) |
  | `WAITING_FOR_ENTRY` | touched | not touched | touched | `INDETERMINATE_ENTRY`, ordering `UNKNOWN` (EI0-D9) |
  | `WAITING_FOR_ENTRY` | not touched | touched | not touched | `CANCELLED_BEFORE_ENTRY`, `STOP_TOUCHED_BEFORE_ENTRY` (EI4-D5) |
  | `WAITING_FOR_ENTRY` | not touched | not touched | any | No event; on the last entry bar, `EXPIRED_BEFORE_ENTRY` |
  | `ACTIVE` | not evaluated | touched | touched | `INDETERMINATE_EXIT`, ordering `UNKNOWN` (EI0-D9) |
  | `ACTIVE` | not evaluated | touched | not touched | `CLOSED_STOP` |
  | `ACTIVE` | not evaluated | not touched | touched | `CLOSED_TARGET` |
  | `ACTIVE` | not evaluated | not touched | not touched | No event; on the bar closing at `modeled_exit_at`, `SESSION_CUTOFF_REACHED` |

  For a LONG recommendation the stop lies below the entry zone and the target above it, so one continuous bar range cannot reach both the stop and the target without also touching the zone. A waiting bar that reaches only the target, having gapped above the zone, evidences no entry. A waiting bar wholly below the stop reaches the stop without touching the zone, so it is `CANCELLED_BEFORE_ENTRY` (EI4-D5 with the approved stop predicate). Ambiguity always takes precedence: zone and stop in one bar is `INDETERMINATE_ENTRY`, and stop and target in one bar is `INDETERMINATE_EXIT`.

  A touch on a bar always takes precedence over that bar's expiry or cutoff. Exit evaluation starts with the bar after the activation bar, which by rule touched neither exit level.
- **Not adopted from Futures.** No `MODELED_STOP_FIRST` rule, no `EXPIRED_AFTER_ACTIVATION` state and no Futures type. ADR-020, section 2 applies, and a shared kernel would need its own ADR.

### 9. Evidence snapshots and interval ordering

- **Inputs.** The Application evaluates an issued recommendation against EI-2B snapshots named explicitly by their `ecs1` identities and loaded through the existing snapshot store. No snapshot enumeration port is added in EI-5.
- **Accepted snapshots.** A snapshot is evaluated only when:
  - its listing, trading date and research-window policy version are the recommendation's;
  - its `as_of` is at or after the modeled `issued_at`;
  - for one recommendation, successive snapshots have strictly increasing `as_of` and non-decreasing journal watermarks.

  A repeat of the last evaluated snapshot is idempotent. A different snapshot at the same `as_of`, or any regression, is refused (the EI3-D9 pattern).
- **Evaluation instant.** It is the snapshot's `as_of` (EI5-D8).
- **Ranges.** Entry evidence comes only from intervals that open in `[eligible_from, latest_entry_bar_opens_at]`. Exit evidence comes only from later intervals that close at or before `modeled_exit_at`.
- **A cursor.** Intervals are consumed in grid order from a cursor:
  - an `ACCEPTABLE` interval is evaluated, and the cursor advances;
  - `NOT_ENDED` stops evaluation with no block;
  - `MISSING`, `UNFINALIZED`, `CONFLICTING_VALUES` or `UNRESOLVED_REVISIONS` stops evaluation as a block.

  No interval is skipped.
- **Citations.** Every event cites:
  - the snapshot identity and interval (opening and closing instants);
  - the interval's EI-2A outcome;
  - the acceptable observations used.

  Core receives these as opaque, caller-supplied evidence references.

### 10. Terminality and unresolved outcomes

- **Terminal means monitoring has ended.**
  - Resolved outcomes: `CLOSED_TARGET` and `CLOSED_STOP`, plus the no-entry outcomes `EXPIRED_BEFORE_ENTRY` and `CANCELLED_BEFORE_ENTRY`.
  - Indeterminate outcomes: `INDETERMINATE_ENTRY` and `INDETERMINATE_EXIT`.
  - Unresolved outcomes: `SESSION_CUTOFF_REACHED` and `INCOMPLETE_EVIDENCE`.

  Unresolved outcomes are excluded from win and loss metrics.
- **No prices.** `INDETERMINATE_*`, `SESSION_CUTOFF_REACHED` and `INCOMPLETE_EVIDENCE` carry no exit price. No event carries a fill, quantity filled, order, broker account or P&L.
- **Blocks become terminal only at the boundary.** A deficiency blocks monitoring, with no event, until an evaluated snapshot has `as_of` at or after the applicable boundary while a needed interval is still deficient. The boundary is the entry-window end while waiting, and `modeled_exit_at` while active. Only then is terminal `INCOMPLETE_EVIDENCE` recorded, listing every deficient interval from the cursor to the boundary with its EI-2A outcome.
- **Exact-boundary finality** (amended 2026-11-12; EI5-D7 clarification, owner approved). Suppose a snapshot's `as_of` equals the boundary and the bar closing at that boundary is `MISSING` or `UNFINALIZED`. That bar is not yet final only because its close has just been reached, so monitoring stays temporarily blocked: no event, not terminal.
  - **An acceptable boundary bar is evaluated normally.** EI-2A treats an interval as ended when `closes_at <= as_of`, and accepts a candle received and confirmed final exactly at its close.
  - **Genuine deficiencies keep the rule above.** These are `CONFLICTING_VALUES`, `UNRESOLVED_REVISIONS`, any earlier deficient interval, and every evaluation after the boundary.
  - **Replay.** A stored `INCOMPLETE_EVIDENCE` that rests only on the exact-boundary condition is refused.
  - **No latency policy** is introduced ([EI-5 Decision Register, section 12.8](../product/Equity-Intraday-V1-EI5-Decision-Register.md#128-ei5-d7-exact-boundary-finality-clarification)).
- **Snapshot selection.** The terminal rule depends on which post-boundary snapshot is evaluated (open point O-2).
- **No overnight carry-over.** Snapshots of other dates are refused, and no interval after `modeled_exit_at` is read.

### 11. Revision events and materiality limits

- A later snapshot can show an interval that an earlier event cited as `ACCEPTABLE` with different acceptable observations, or as `CONFLICTING_VALUES` or `UNRESOLVED_REVISIONS`. That is a correction to cited evidence. A non-state `SOURCE_REVISION_RECORDED` event is then appended, citing:
  - the original event;
  - the superseded evidence reference;
  - the new evidence reference;
  - materiality `UNASSESSED`.
- Detection is idempotent per (cited interval, new evidence), and it is permitted after a terminal state.
- A revision never changes a state, rewrites an event or produces `INVALIDATED`. EI0-D21 is pending, and no listing-applicability source exists.
- Late evidence that fills an interval cited as deficient is not a correction. No event kind records it in EI-5 (open point O-3), and the state is unchanged.

### 12. Application ports and use cases

The names are proposals; the design gives the detail.

| Component | Kind | Slice |
| --- | --- | --- |
| `EvaluateEquityRecommendationLifecycleUseCase` | Use case: snapshots in, events out; persists nothing in EI-5C | EI-5C |
| `IssueEquityTradeRecommendationUseCase` | Use case: the only operational issuer | EI-5D |
| `DeclareEquitySessionIssuanceCompletenessUseCase` | Use case: the only operational declarer | EI-5D |
| `EquityRecommendationIssuanceReader`, `EquityRecommendationIssuanceWriter` | Ports for the issuance authority | EI-5D |
| An authoritative `EquitySessionRecommendationHistorySource` | Application service over the reader, implementing the existing port | EI-5D |
| `EquityRecommendationLifecycleStore` | Port: event load and compare-and-append; planned | EI-6 |
| A new, versioned EI-4C assessment (amended 2026-11-12) | History view as of `t − 1 µs`, with new evaluator and format versions; version 1 unchanged | The EI-4C compatibility prerequisite, before EI-5D; own design |

The Application owns ports and records. Infrastructure implements them. The Application never imports Infrastructure.

### 13. Infrastructure authority deferred to EI-6

EI-6 implements, in northstar-infrastructure:

- the issuance, declaration and lifecycle-event tables, in the **same SQLite database authority** as the EI-4D records, with foreign keys to the EI-4D disposition;
- insert-only triggers, and uniqueness constraints for every rule of section 7;
- one `BEGIN IMMEDIATE` transaction per issuance, declaration or lifecycle append;
- commit-unconfirmed reconciliation by authoritative reload (the EI4-D9-G pattern);
- restart, separate-process and concurrency acceptance on real SQLite.

EI-5 implements none of it. Monitoring runs and any operational composition root also belong to EI-6 or later.

### 14. Trust model

- **Scope.** The scope of [ADR-023, section 2](ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md#2-operational-trust-boundary-dq1) and the ADR-022 DQ9 limitation applies unchanged:
  - personal-use synthetic research;
  - one host and one SQLite file;
  - protection for supported and ordinary operational paths only.
- **Not protected against.** Privileged Python, direct SQL or file modification, malicious code, forged capability objects and multi-host writers.
- **What declarations certify.** A completeness declaration is trusted because the declaring workflow is trusted within that boundary; it is not a cryptographic proof. The authority guarantees only what section 7 states about sequences, regressions and backdating.

## Relationship to Earlier ADRs

- **ADR-017.** The layering is unchanged: Core ← Application ← Infrastructure.
- **ADR-020.** Futures patterns, not Futures types; no shared kernel. EI4-D6 is unchanged.
- **ADR-021.** This ADR builds the issued-recommendation authority that ADR-021 sections 9 and Future Considerations deferred. It reuses ADR-021's verification levels and same-database placement, and changes none of its decisions.
  - *Amended 2026-11-12.* The C-2 compatibility prerequisite adds a new evaluator version beside version 1. Keeping version 1 verifiable then needs two-version verification (R-1). That is a bounded first step toward ADR-021's "historical evaluator registry" future consideration, which ADR-021, section 10 excludes today.
  - This ADR does not approve that step. The prerequisite's own design must obtain explicit architecture approval for it.
  - *Amended 2026-11-13.* That approval is [ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md). It amends ADR-021 section 10 with a closed two-version table, not a registry.
- **ADR-022.** Unchanged. Issuance reads only verified proposals, whose policy bindings were checked when they were recorded.
- **ADR-023.** Unchanged. Its verified-write pattern is reused for the issuance authority. The generation-write boundary is not extended or altered.

## Rationale

- **Issuance copies, never accepts, timing.** A caller cannot move `issued_at` or `eligible_from`, so backdating and look-ahead through issuance are impossible.
- **The existing history contract is the target.** EI-4C already consumes it, so the history contract itself does not change.
  - *Amended 2026-11-12 (C-2).* What changes, in a new version, is the instant at which the EI-4C assessment reads that contract: one microsecond before its modeled issuance. Version 1 stays as recorded.
- **Gap-through recognition for exits** (amended 2026-11-12, O-1). A level that price traded through is never ignored, and no price is inferred for it.
- **One sequence for issuances and declarations** gives one watermark that pins both what was issued and what was declared complete.
- **No backdating** is what makes a past as-of view still true when replayed.
- **A pure Core model** makes every transition testable from synthetic bars, independently of evidence plumbing and persistence.
- **Explicit snapshots** keep the evaluation reproducible and independent of a clock or of storage enumeration.

## Alternatives Considered

- **Automatic issuance at recording.** Rejected by EI5-D1: generation and issuance stay separate, and a proposal is not an issued recommendation.
- **Reuse `eirk1` as the issuance key.** Rejected by EI5-D3: generation and issuance identities stay distinct.
- **Futures `MODELED_STOP_FIRST`.** Rejected by EI0-D9: a modeled ordering is an evaluation assumption, not a lifecycle fact.
- **Ignore a pre-entry stop touch, as the Futures waiting state does.** Rejected by EI4-D5.
- **Infer completeness from the latest issuance.** Rejected: a past view could not be re-verified, and late-recorded modeled issuances would change it.
- **Caller-supplied watermarks or sequences.** Rejected by EI5-D4.
- **Terminal `INCOMPLETE_EVIDENCE` at the first gap.** Rejected by EI5-D7: a gap blocks monitoring until the boundary.
- **A shared Futures and Equity lifecycle kernel.** Not chosen: it needs its own ADR (ADR-020).
- **A separate database file for the issuance authority.** Rejected: references to EI-4D records would lose foreign keys, and one transaction could not cover the checks.
- **Admit an issuance exactly at `complete_through` (C-2 option b).** Rejected 2026-11-12: a declared completeness would later be contradicted.
- **Make `complete_through` exclusive in the history contract (C-2 option c).** Rejected 2026-11-12: it would change an existing contract and the meaning of stored evidence.
- **Strict two-sided touch for the stop and target (O-1 option b).** Rejected 2026-11-12: a traded-through level would be ignored.
- **Gap-through activation of the entry zone.** Not adopted: the approved O-1 rule covers the stop and target only.

## Consequences

### Positive Consequences

- Every issued recommendation traces to one verified proposal, and every lifecycle event to one snapshot and interval.
- A historical as-of view of issued history stays reproducible and true after later recording.
- Ambiguity, missing evidence and session end are explicit outcomes, never invented prices.
- The Core model can be built and accepted before any issuance or persistence exists.

### Negative Consequences

- EI-5D cannot start until the EI-4C versioned compatibility prerequisite is designed, approved and implemented (C-2, approved 2026-11-12), including R-1 and R-2. C-1 is decided.
  - *Amended 2026-11-13.* The prerequisite is designed and approved ([ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md)). EI-5D still waits for its implementation and acceptance (slices 1 to 5).
- Issuance follows a strict protocol: a proposal at `t` is issuable only while its session is declared complete through exactly `t − 1 µs`. Declaring further first makes it permanently non-issuable, failing closed.
- Recommendations issued at the same modeled instant do not see each other in their assessments.
- Callers must supply snapshot identities. Choosing which snapshots to evaluate becomes an EI-6 monitoring responsibility (O-2).
- Lifecycle state does not yet relax EI4-D8, so exclusivity stays conservative.
- Single-host SQLite only. No protection against privileged local access.

### Acceptance requirements

Each slice must demonstrate the scenarios of the [EI-5 design acceptance matrix](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md#12-acceptance-matrix) that it owns. At minimum:

- every permitted and forbidden transition;
- the ambiguity and pre-entry rules, including gap-through stop and target recognition;
- the entry-window boundaries;
- deficiency blocking and the terminal rule;
- no look-ahead;
- revisions after a terminal state;
- issuance refusals, duplicates and conflicts;
- identity stability;
- backdating refusal, the `t − 1 µs` history boundary, and watermark stability;
- concurrency, restart and replay (EI-6);
- the absence of any order, fill or exit price.

Every scenario is **PLANNED** until executed and recorded in an acceptance record.

## Scope

This ADR records an accepted architecture. It does not:

- implement any Core, Application or Infrastructure component, test or schema;
- implement, or approve as an implementation PR, the EI-4C versioned compatibility change, or decide its open contract points R-1 and R-2 (decided 2026-11-13 in [ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md), not here);
- resolve the deferred open points O-2 and O-3;
- approve ACTUAL issuance, `INVALIDATED`, EI0-D21, EI0-D19, EI0-D11, the exclusivity threshold, EI3-D10 or the relaxation of EI4-D8;
- approve any production cutoff, latency, price, increment or schedule. The 09:15 to 15:00 IST research window stays a research window;
- authorize real data, actionable recommendations, orders, fills, positions, monitoring jobs, API or Web exposure, deployment or trading;
- change Futures or Options behaviour, or any EI-4 record, identity or format.

## Future Considerations

- Owner decisions on the deferred O-2 and O-3. C-1, C-2 and O-1 were decided on 2026-11-12.
- The EI-4C versioned compatibility design, including R-1 (two-version verification) and R-2 (fingerprint prefixes). *Settled 2026-11-13 by [ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md); its implementation remains.*
- `INVALIDATED`, once a listing-applicability source and EI0-D21 exist.
- Relaxing EI4-D8 with authoritative lifecycle state.
- ACTUAL issuance, under separate governance.
- An evidence-settlement allowance and EI3-D10, before real data.
- A shared recommendation kernel, only through its own ADR.

## Related Documents

- [EI-5 Decision Register](../product/Equity-Intraday-V1-EI5-Decision-Register.md)
- [EI-5 Issuance and Lifecycle Design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md), sections 6.6.1, 6.7, 8 and 10.3
- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md)
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md)
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)
- [ADR-017: Repository Layering and Futures Recommendation Domain Boundary](ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md)
- Engineering Standards ES-001, ES-003, ES-004, ES-007
- [ADR-025: Equity Intraday V1 Versioned Evaluation and History Compatibility](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) (added 2026-11-13)
- [EI-4C Versioned History Compatibility Design v1.0](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md) (added 2026-11-13)
