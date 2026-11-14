# EI-6C — Equity Recommendation Lifecycle Monitoring and Snapshot Selection Design v1.0

## Status

**OWNER APPROVED — 2026-11-15 (EI6C-P1 to EI6C-P12; O-2 option A; EI3-D10 option (a)).** Version 1.0.

| Item | Status |
| --- | --- |
| EI-6C governance and this design | **APPROVED** (owner, 2026-11-15) |
| EI-6C implementation | **NOT AUTHORIZED** — needs a separate owner implementation authorization |
| Acceptance scenarios MS-01 to MS-40 ([section 12](#12-acceptance)) | **PLANNED**; none executed |
| O-2 | **RESOLVED: option A, Δ = 0** ([section 8](#8-o-2-and-the-exact-boundary-rule)) |
| EI3-D10 | **RESOLVED: option (a), strict EI-3A rule retained** |
| O-3 | **Unchanged: option (b)** (2026-11-14) |
| EI-6A, EI-6B | **ACCEPTED WITH DOCUMENTED LIMITATIONS** (2026-11-14, 2026-11-15); not reopened |
| Core change; EI-6A or EI-6B contract change | **NOT AUTHORIZED** |
| Operational or production authorization | **NOT GRANTED** |

- **Governance.** The owner decisions are recorded in the [EI-5 Decision Register, section 22](../product/Equity-Intraday-V1-EI5-Decision-Register.md#22-owner-authorization-of-ei-6c-governance-2026-11-15). EI3-D10 is recorded in the [EI-3 Decision Register, section 12](../product/Equity-Intraday-V1-EI3-Decision-Register.md#12-ei3-d10-owner-decision-2026-11-15). The architecture is [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), section 13 (amended 2026-11-15).
- **What this is.** The EI-6C stage of the [EI-5 Issuance and Lifecycle Design v1.0](Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md) (section 11, "monitoring and operational snapshot policy"). It builds on the [EI-6B Lifecycle Persistence Design](Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md), which it does not change.
- **Not implemented.** No module, port, test or table described here exists. The new contracts of [section 4](#4-new-contracts-proposed) are proposed engineering contracts until implementation is separately authorized.
- **Binding level.** The decisions, the eligibility rule, the selection and recording semantics, the determinism boundaries ([section 10](#10-determinism-boundaries)) and the outcome semantics are binding. Python names are proposals; an implementation PR may rename one by stating the reason.

Code was read at:

| Repository | Branch | Commit |
| --- | --- | --- |
| northstar-core | `develop` | `29bd34a` |
| northstar-application | `develop` | `b7df16b` (#98) |
| northstar-infrastructure | `develop` | `dd6a434` (#68) |
| northstar-docs | `develop` | `5c7ddca` (#68) |

## 1. Scope (EI6C-P1)

**In scope:** a deterministic Application-layer operation, invoked explicitly with a caller-supplied monitoring instant `T`, that monitors the lifecycles of **existing authoritative MODELED issuances** by selecting EI-2B snapshots and delegating every lifecycle write to the EI-6B recording use case.

**Not in scope (not orchestrated by EI-6C):** EI-3 research; EI-4 generation; EI-6A declarations or issuance; EI-2 snapshot creation; market-data collection; API or Web operations; scheduling or background polling; broker execution. EI-6C reads no system clock. No second lifecycle state machine and no competing issuance or lifecycle authority is introduced.

**Data flow:**

```text
caller-supplied T
  -> EI-6C monitor (Application; no clock)
       -> issuance scope (EI-6A reader, unchanged)            section 7
       -> per issuance:
            load_lifecycle (EI-6B store, unchanged)
            snapshot catalog (new, read-only)                  section 5
            deterministic selection                             section 5
            one snapshot per call -> RecordEquityRecommendationLifecycleUseCase (EI-6B, unchanged)
                 -> EquityRecommendationLifecycleStore (authoritative lifecycle history)
       -> per-issuance results                                  section 9
```

## 2. Existing contracts reused (verified at the commits above)

| Concern | Symbol | Location (northstar-application unless stated) |
| --- | --- | --- |
| Evidence snapshot | `EquityCandleEvidenceSnapshot` (`snapshot_id`, `as_of`, `issued_at`, `watermark`, `intervals`, `evidence_class`, `manifest_fingerprint`) | `application_services/equity_candle_evidence_snapshot.py` |
| Evidence class | `EquityCandleEvidenceClass` (`SYNTHETIC`, `PRODUCTION`) | same module, line 163 |
| Snapshot store | `EquityCandleEvidenceSnapshotStore.issue`, `.get_snapshot` — the only two operations; no enumeration | `ports/equity_candle_evidence_journal.py`, lines 132–141 |
| Interval outcomes | `EquityCandleIntervalOutcome` | `application_services/equity_intraday_candle_validation.py` |
| Snapshot persistence | `SQLiteEquityCandleEvidenceSnapshotStore`; table `equity_candle_evidence_snapshots` (`as_of`, `issued_at` as text; no venue column; key `snapshot_id` only); index `equity_candle_evidence_snapshots_scope (symbol, exchange_code, trading_date)` | northstar-infrastructure `persistence/sqlite_equity_candle_evidence_schema.py`, lines 100–124 |
| EI-5C rules | `EvaluateSyntheticEquityRecommendationLifecycleUseCase` (`_require_session`, `_require_order`), `_observation`, `_record_revisions` | `application_services/evaluate_synthetic_equity_recommendation_lifecycle.py` |
| Issuance authority | `EquityRecommendationIssuanceReader`: `load_issuance`, `load_session_issuance`, `session_view`, `session_view_at` | `ports/equity_recommendation_issuance.py`, line 239 |
| Session view rule | `equity_issuance_session_view`: lists issuances with `issued_at <= complete_through` | `application_services/equity_recommendation_issuance_rules.py`, lines 96–125 |
| Lifecycle recording | `RecordEquityRecommendationLifecycleUseCase.execute(issuance_id, snapshot_ids)`; `EquityLifecycleRecordingOutcome`; `EquityLifecycleRecordingRefusal` (including the `SNAPSHOT_*` reasons); `EquityLifecycleRecordingRefusedError`; `EquityLifecycleRecordingUnresolvedError` | `application_services/record_equity_recommendation_lifecycle.py` |
| Lifecycle store | `EquityRecommendationLifecycleStore` (`load_lifecycle`, `append`); `equity_lifecycle_append_decision`; `SQLiteEquityRecommendationLifecycleStore` | `ports/equity_recommendation_lifecycle_store.py`; northstar-infrastructure `persistence/sqlite_equity_recommendation_lifecycle.py` |
| Core | `EquityRecommendationLifecycle` (`advance`, `cursor`, `last_evaluated_at`, `is_terminal`) | northstar-core `recommendations/equity/lifecycle/lifecycle.py` |

**Observed recorder behaviour relied on:**

- Within one call, snapshots must strictly increase in `as_of` and must not regress in watermark; across calls only `as_of >= last event` is checked (`_require_order`).
- Refusals of the EI-5C rules are typed (`EquityLifecycleRecordingRefusedError`). Only `EquitySyntheticLifecycleRefusedError` is caught in `_evaluate` (`record_equity_recommendation_lifecycle.py`, line 379); a Core lifecycle rejection or an unverified-append error escapes untyped.
- The recorder evaluates a whole call before appending; a refusal writes nothing for that call.

## 3. Decisions applied

| ID | Decision (approved 2026-11-15) | Sections |
| --- | --- | --- |
| EI6C-P1 | Lifecycle-only scope (L1) | 1 |
| EI6C-P2 | O-2 option A, Δ = 0 | 8 |
| EI6C-P3 | EI3-D10 option (a), strict EI-3A retained | 8.3 |
| EI6C-P4 | Read-only snapshot catalog; dual-timestamp eligibility | 4, 5 |
| EI6C-P5 | Chronological, incremental, one snapshot per recorder call; no thinning; no batching in v1 | 5, 6 |
| EI6C-P6 | No durable checkpoint (K1); finite catalog limit with typed refusal | 6.4 |
| EI6C-P7 | EXPLICIT, STRATEGY_LISTING, DECLARED_SESSION scope modes | 7 |
| EI6C-P8 | Duplicates, same-instant conflicts, later revisions, late evidence | 5.4 |
| EI6C-P9 | Per-issuance failure isolation | 9 |
| EI6C-P10 | Concurrency through EI-6B compare-and-append | 6.3 |
| EI6C-P11 | Typed per-issuance outcomes | 9 |
| EI6C-P12 | Non-durable selection-policy provenance; one fixed policy | 11 |

## 4. New contracts proposed

Proposed engineering contracts. They are approved as semantics; their code is not authorized.

| Contract | Kind | Responsibility |
| --- | --- | --- |
| Snapshot catalog (for example `EquityCandleEvidenceSnapshotCatalog`) | Read-only Application port; Infrastructure SQLite adapter over the existing snapshot table and scope index | Lists eligible snapshot entries for one issuance scope at `T` (section 5) |
| Monitoring use case (for example `MonitorEquityRecommendationLifecyclesUseCase.execute(scope, monitoring_instant)`) | Application use case | Resolves issuances, selects snapshots, records incrementally, reports results |
| Scope modes | Value types | EXPLICIT, STRATEGY_LISTING, DECLARED_SESSION (section 7) |
| Selection rules | Pure Application policy | Eligibility, ordering, groups, regression, late evidence (section 5) |
| Failure classification and results | Value types | Section 9 |
| Catalog limit | Governed constant with a typed error | Section 6.4 |
| Policy identity | One versioned constant, for example `northstar.equity-lifecycle-monitoring-policy/1`. **Implementation detail, to be verified; not a separately owner-approved identifier** | Section 11 |

No new durable table, journal, clock authority or lifecycle field is introduced.

## 5. Temporal eligibility and selection (EI6C-P4, P5, P8)

### 5.1 Eligibility

For a monitoring instant `T`, a snapshot is eligible only when **both**:

- `as_of <= T`, and
- `issued_at <= T`.

Both comparisons use parsed, timezone-aware instants (`PointInTime`, UTC), never lexical SQLite text ordering; text order is not chronological (`…:00.5Z` sorts before `…:00Z`). Equality is eligible.

**Declared, not proven.** `issued_at` is supplied by the caller under the EI-2B contract (`equity_candle_evidence_snapshot.py`, line 870, requires only `issued_at >= as_of`), and nothing persisted records when a snapshot was actually created (`journaled_at` is informative only). The dual guard therefore refuses a snapshot that **declares** issuance after `T`. It does **not** prove historical availability at `T`, and it does not prevent privileged backdating.

### 5.2 Scope filtering and verification

A catalog entry is a candidate for an issuance only when, for that issuance's verified candidate:

- symbol, exchange code and trading date match (SQL, through the scope index);
- the evidence class is `SYNTHETIC`;
- timeframe and window policy version match;
- the snapshot decodes and verifies through the existing verified decoding (`get_snapshot`), so identity and manifest integrity hold;
- `as_of` is at or after the issuance's `issued_at` (earlier snapshots are not candidates).

**Venue.** The table has no venue column, so the venue reference is checked from the authenticated snapshot contents. A snapshot that passes the SQL scope but carries another venue reference is **not** silently dropped: it is a typed snapshot refusal at its position (section 6.2).

The catalog reads its entries in one consistent database read view.

### 5.3 Ordering and same-instant groups

Candidates are ordered by parsed `as_of`, then by `snapshot_id`. Snapshots sharing an `as_of` form a **same-instant group**, which is processed completely before the next instant.

Selection starts after the authoritative history:

- with no lifecycle event, at the issuance's `issued_at`;
- otherwise at the last persisted event's `evaluated_at` (the `as_of` of its cited snapshot). At that instant only conflict detection applies (section 5.4); the cited snapshot and its equivalents are not re-recorded.

**Watermark and chronology.** Along the selected sequence the watermark must not decrease, starting from the watermark of the snapshot the last persisted event cites (loaded by its reference through `get_snapshot`). A decrease stops the issuance (`STOPPED_AT_REGRESSION`). A cited snapshot that cannot be loaded fails the issuance closed.

### 5.4 Duplicates, conflicts, revisions and late evidence (EI6C-P8)

| Situation | Treatment |
| --- | --- |
| **Duplicate evidence:** several eligible snapshots at one `as_of` with an identical manifest fingerprint | One is evaluated: the lowest `snapshot_id` of the currently eligible group |
| **Same-instant conflict:** eligible snapshots at one `as_of` with different manifests | Not resolved. The issuance stops at that instant (`STOPPED_AT_CONFLICT`); earlier committed events stay |
| **Same-instant evidence after a recorded event:** a newly eligible snapshot at the last event's `as_of` | Identical manifest: nothing recorded, the event's reference unchanged (no rewrite). Different manifest: `STOPPED_AT_CONFLICT`; nothing appended |
| **Later-instant source revision:** a later snapshot corrects evidence an earlier event cited | Inherited EI-5C/EI-6B behaviour: `SOURCE_REVISION_RECORDED`, once, materiality `UNASSESSED` (EI5-D8) |
| **Late historical evidence:** an eligible snapshot whose `as_of` precedes the last persisted event | Never evaluated; reported with the non-durable diagnostic `LATE_HISTORICAL_SNAPSHOT_IGNORED`; history is not reinterpreted |
| **Evidence contradicting recorded history:** a later snapshot shows a cited interval `MISSING` or `UNFINALIZED` | Refused by the recorder (`SNAPSHOT_CONTRADICTS_CITED_EVIDENCE`): `STOPPED_AT_SNAPSHOT_REFUSAL`; earlier events stay |

Already committed events remain authoritative in every case.

## 6. Incremental recording (EI6C-P5, P6, P10)

### 6.1 Reference algorithm

For each issuance, after scope resolution and the catalog read:

1. Refuse with `CATALOG_LIMIT_EXCEEDED` when the candidates exceed the governed limit (section 6.4); record nothing.
2. Apply section 5 to obtain the ordered candidates; stop at the first conflict or regression.
3. For each selected snapshot, in order, call `RecordEquityRecommendationLifecycleUseCase.execute(issuance_id, (snapshot_id,))` — **one snapshot per call**.
4. Continue after `APPENDED`, `EQUIVALENT` or `NO_NEW_EVENTS`; stop as section 6.2 states otherwise.

There is no snapshot thinning and no batch-recording optimization in EI-6C v1. Every lifecycle write goes through the recorder, Core and the EI-6B store; no lifecycle row is inserted directly.

### 6.2 Prefix safety and stops

| Event during a call | Result | Committed earlier events |
| --- | --- | --- |
| A typed `SNAPSHOT_*` refusal (EI-5C rule) | `STOPPED_AT_SNAPSHOT_REFUSAL` | Kept |
| `CONFLICT` (another writer advanced the history) | `CONFLICT`; the next run reloads | Kept |
| A typed non-snapshot refusal (unavailable, corrupt, generation, window provenance) | `REFUSED`, with the recorder's retryable flag | Kept |
| `EquityLifecycleRecordingUnresolvedError` | `UNRESOLVED`; nothing claimed | Kept |
| Any other exception escaping the recorder (Core rejection, append verification) | `ISSUANCE_EVALUATION_FAILED`; the issuance fails closed. The failure is **not** assumed to be specific to one snapshot | Kept |

The EI-6B public contract is unchanged; untyped errors are classified at the EI-6C boundary.

### 6.3 Concurrency, uncertainty and recovery

EI-6B's atomic compare-and-append is the only concurrency mechanism; no durable lock or second write authority is added. A concurrent caller may observe `APPENDED`, `EQUIVALENT`, `NO_NEW_EVENTS`, `CONFLICT`, `STORE_UNAVAILABLE` or an unresolved result, depending on when it reads the authoritative history. Because the store's rule is exact, an identical single-snapshot request made after another monitor progressed further is a `CONFLICT`. Commit uncertainty is reconciled inside the recorder, as accepted for EI-6B. After a crash or in a fresh process, a monitor starts from durable state alone.

### 6.4 Progress and the catalog limit (K1)

No durable monitoring checkpoint, run ledger or quiet-evaluation cursor exists. Progress is derived from the authoritative lifecycle history, the last persisted event, its cited snapshot and watermark, and the current eligible catalog. Quiet evaluations leave no durable record, and repeated evaluation of snapshots after the last event is accepted.

The catalog has a finite, governed limit on candidates per issuance per run. Exceeding it raises a typed error; a candidate prefix is never silently truncated (without a checkpoint, a truncated quiet prefix would be re-read forever). **The numerical limit is not yet fixed.** It is set during the governed Application design from the actual synthetic fixtures, measured cost and a documented acceptance test (MS-30); no illustrative value is approved.

## 7. Issuance scope (EI6C-P7)

| Mode | Existing authority | Behaviour | Completeness |
| --- | --- | --- | --- |
| EXPLICIT | `load_issuance(issuance_id)` | Each named issuance; an unknown identity is a typed not-found result for it; corrupt authority keeps the EI-6A D6-5 error, typed for that issuance | As named |
| STRATEGY_LISTING | `load_session_issuance(strategy, listing, trading_date)` | Zero or one issuance; finds issuances not yet covered by a declaration | One key per request |
| DECLARED_SESSION | `session_view` / `session_view_at(exchange_code, trading_date, watermark)` | Issuances with `issued_at <= complete_through` at the view's watermark | **Incomplete:** an issuance made after the latest declaration (for example at `t` after the declaration through `t − 1 µs`) is omitted until a later declaration covers it |

No new EI-6A list-all interface is approved.

## 8. O-2 and the exact-boundary rule

### 8.1 Option A, Δ = 0 (EI6C-P2)

Snapshots are selected explicitly and deterministically (section 5), with no evidence-settlement allowance, no deferral window and no artificial timeout. Every eligible snapshot is evaluated in order.

- **Unchanged:** EI5-D7, including the exact-boundary clarification (2026-11-12): at `as_of` exactly equal to the applicable boundary, a `MISSING` or `UNFINALIZED` boundary bar blocks temporarily; after the boundary a remaining deficiency becomes terminal `INCOMPLETE_EVIDENCE`. Boundaries: the entry-window end (`entry_window_ends_at`) while waiting, `modeled_exit_at` while active.
- **Accepted consequence:** when synthetic evidence models delayed finality, selection can produce terminal `INCOMPLETE_EVIDENCE` before later evidence arrives.
- **No finality bound:** no finite waiting allowance is a guaranteed finality bound (EI3-D6 assumes no maximum provider finality latency). An allowance based on actual provider characteristics is deferred to future real-data governance and is not authorized here.

### 8.2 O-3 and terminal immutability

O-3 option (b) is unchanged: after terminal `INCOMPLETE_EVIDENCE`, later gap-filling evidence produces no lifecycle event and reopens nothing. Terminal states stay immutable; Core-permitted `SOURCE_REVISION_RECORDED` events stay as EI5-D8 governs them. Product specification section 8.2's audited-event requirement stays deferred to EI-8.

### 8.3 EI3-D10 (EI6C-P3)

EI-3A's strict rule is retained: an ended but unfinalized candle rejects a research evaluation. No new EI-3 evaluator version and no change to EI-3C timeline contracts or EI-4 generation fingerprints. EI-6C does not orchestrate research, so this decision constrains EI-6C only by ruling out any relaxation. Real-data finality latency remains a future readiness concern.

## 9. Results and failure isolation (EI6C-P9, P11)

Each issuance in a run has its own result; a failure for one never suppresses another's.

| Outcome | Origin | Meaning |
| --- | --- | --- |
| `APPENDED` | EI-6B | At least one event appended this run |
| `EQUIVALENT` | EI-6B | The batch was already recorded |
| `CONFLICT` | EI-6B | Another writer advanced the history; nothing adopted |
| `NO_NEW_EVENTS` | EI-6B | Candidates evaluated; no event (a successful no-op) |
| `NO_ELIGIBLE_SNAPSHOT` | EI-6C | No candidate at `T` (a successful no-op) |
| `STOPPED_AT_CONFLICT` | EI-6C | Same-instant conflict |
| `STOPPED_AT_REGRESSION` | EI-6C | Watermark or chronology regression |
| `STOPPED_AT_SNAPSHOT_REFUSAL` | EI-6C, mapping EI-5C `SNAPSHOT_*` | Typed snapshot refusal |
| `CATALOG_LIMIT_EXCEEDED` | EI-6C | Resource limit exceeded; nothing recorded |
| `ISSUANCE_EVALUATION_FAILED` | EI-6C | Unexpected untyped Core or verification failure; fail closed |
| `REFUSED` | EI-6B refusals | Typed refusal (permanent corruption, unavailability, provenance), with the retryable flag |
| `UNRESOLVED` | EI-6B | Commit uncertainty not resolved |
| `LATE_HISTORICAL_SNAPSHOT_IGNORED` | EI-6C diagnostic, non-durable | Accompanies any outcome |

Successful no-op outcomes (`NO_NEW_EVENTS`, `NO_ELIGIBLE_SNAPSHOT`) are never conflated with failures, and a failure is never reported as a quiet outcome. Exact type names and result shape are verified during implementation without weakening these semantics.

## 10. Determinism boundaries

**Guaranteed:**

1. Identical authoritative history, eligible catalog, monitoring instant and policy produce deterministic selection and idempotent behaviour, subject to unchanged authoritative inputs.
2. Successful lifecycle appends remain authoritative.
3. Replay from the same stored inputs is deterministic.
4. Atomic compare-and-append prevents duplicate or divergent committed lifecycle authority.
5. Core preserves terminal-state immutability.

**Conditional.** Several runs at increasing instants may equal one run at the latest instant only when all hold: the catalog is unchanged; the eligible snapshot sets form stable ordered prefixes (no snapshot with an earlier `as_of` becomes eligible, through a later `issued_at`, after a later-`as_of` snapshot was eligible); same-instant groups stay stable and complete; the recorder's preconditions hold; no failure interrupts the comparison; the same approved policy is used. **These conditions are not guaranteed for arbitrary monitoring instants.**

**Not guaranteed:** run-frequency independence for all valid catalogs; equality between one late run and several earlier runs; knowledge-time accuracy from caller-supplied timestamps; convergence after late historical snapshots; identical outcomes under different process interleavings; reconstruction of quiet evaluations after restart; durable proof of monitoring-policy identity.

Existing EI-6B events are never rewritten in pursuit of these properties.

## 11. Policy provenance and historical compatibility (EI6C-P12)

The lifecycle event ledger proves its stored events, their evidence references, chronology and replay validity. It does **not** necessarily prove which policy selected a snapshot, the run identity, the caller's monitoring instant, which quiet snapshots were considered, or a historical policy change. This is accepted for synthetic EI-6C v1.

- One fixed governed policy; no operator-selectable allowance; no mid-issuance policy switching.
- No change to accepted EI-6B event fields; no durable policy-authority table.
- **Compatibility:** lifecycle histories produced before EI-6C, including those recorded directly through the EI-6B recorder, remain authoritative. They are never rejected or rewritten under the new policy, and no retrospective policy-consistency check invalidates them. Monitoring continues from their last event.
- A future policy change requires new governance and compatibility analysis.

## 12. Acceptance

**Status for every row: PLANNED.** Layers: C (Application contract), S (real SQLite), P (separate OS processes), A (architecture scan). Inherited evidence (EI-5C, EI-6A, EI-6B) is not re-counted; these rows are new EI-6C requirements.

| ID | Requirement | Layer | Status |
| --- | --- | --- | --- |
| MS-01 | No eligible snapshot at `T`: `NO_ELIGIBLE_SNAPSHOT`; nothing written | C, S | PLANNED |
| MS-02 | Temporal eligibility: only snapshots with both `as_of <= T` and `issued_at <= T` are eligible; equality is eligible; comparisons on parsed instants; the documented limitation that the guard does not detect backdated issuance | C, S | PLANNED |
| MS-03 | Same-instant groups: equivalent manifests deduplicated to the lowest eligible `snapshot_id`; conflicting eligible manifests refused at that instant (`STOPPED_AT_CONFLICT`); earlier committed events preserved | C, S | PLANNED |
| MS-04 | Conditional run-frequency equivalence: several runs equal one later run **only** when the eligible sets form stable ordered prefixes and every section 10 condition holds | C, S | PLANNED |
| MS-05 | An identical rerun with unchanged inputs writes nothing | C, S | PLANNED |
| MS-06 | A run at a smaller `T` after a larger-`T` run, with an unchanged catalog, writes nothing | C, S | PLANNED |
| MS-07 | A quiet evaluation: `NO_NEW_EVENTS`; no lifecycle row; no checkpoint, cursor or run table | C, S | PLANNED |
| MS-08 | An event-producing path through monitoring to a terminal state | C, S | PLANNED |
| MS-09 | O-2 option A: no settlement allowance; EI5-D7's exact-boundary rule preserved; a post-boundary deficiency becomes terminal `INCOMPLETE_EVIDENCE`; entry-window and modeled-exit boundaries both covered | C, S | PLANNED |
| MS-10 | No artificial deferral: an otherwise eligible snapshot is never postponed or suppressed through an invented Δ window; later-arriving evidence does not reopen a terminal lifecycle (O-3) | C, S | PLANNED |
| MS-11 | EI3-D10 option (a): EI-3A still rejects an ended but unfinalized candle; EI-3C and EI-4 contracts and fingerprints unchanged (no-regression) | C | PLANNED |
| MS-12 | A deficient or not-yet-final bar before the applicable boundary blocks temporarily, with no event | C, S | PLANNED |
| MS-13 | A later-instant source revision is recorded once as `SOURCE_REVISION_RECORDED`; same-instant conflicts and late contradictory evidence do not rewrite earlier history | C, S | PLANNED |
| MS-14 | A terminal lifecycle records no state change; only Core-permitted revisions; otherwise `NO_NEW_EVENTS` | C, S | PLANNED |
| MS-15 | O-3 option (b) through monitoring: late gap-filling evidence after terminal `INCOMPLETE_EVIDENCE` records nothing; rows unchanged | C, S | PLANNED |
| MS-16 | Concurrent monitors over the same scope: outcomes from {`APPENDED`, `EQUIVALENT`, `NO_NEW_EVENTS`, `CONFLICT`, `STORE_UNAVAILABLE`, unresolved}; no duplicate event; one committed history; no requirement that every second caller is `EQUIVALENT` | S, P | PLANNED |
| MS-17 | Divergent concurrent monitors: the first commit wins; the loser is `CONFLICT`; the history is a Core-valid replay of the commit order; no scheduling-independent winner required | S, P | PLANNED |
| MS-18 | A process that ends before COMMIT leaves nothing; a later run completes | S, P | PLANNED |
| MS-19 | Commit uncertainty is reconciled through the recorder (`EQUIVALENT` reconciled, bounded retry, or `UNRESOLVED`) | C, S | PLANNED |
| MS-20 | Corrupt issuance or lifecycle authority: a typed permanent failure for that issuance | C, S | PLANNED |
| MS-21 | Snapshots of another listing, trading date, timeframe or window version are never candidates; another venue reference is a typed snapshot refusal | C, S | PLANNED |
| MS-22 | A fresh process with identical authoritative inputs reproduces the selection and outcomes; it does not reconstruct past quiet runs or selection-policy identity | S, P | PLANNED |
| MS-23 | Several issuances in one run are isolated: a failure for one does not suppress the others' results | C, S | PLANNED |
| MS-24 | Architecture: no clock, network, scheduler or durable write outside the EI-6B recorder; the catalog is read-only; no direct lifecycle-row insert; no second state machine | A | PLANNED |
| MS-25 | Non-`SYNTHETIC` snapshots are never selected | C, S | PLANNED |
| MS-26 | No regression: the EI-5, EI-6A and EI-6B suites pass unchanged | C, S, P | PLANNED |
| MS-27 | Watermark regression is detected relative to earlier selected evidence and to the last persisted event's cited snapshot | C, S | PLANNED |
| MS-28 | Changing-catalog path dependence: an earlier committed event stays authoritative when late conflicting evidence appears; no convergence with a hypothetical single later run is required | C, S | PLANNED |
| MS-29 | Parsed-instant ordering: fractional-second and whole-second ISO instants ordered chronologically (`…:00Z` before `…:00.5Z`) | C, S | PLANNED |
| MS-30 | Candidate overflow raises a typed `CATALOG_LIMIT_EXCEEDED`; never silent truncation; the numerical limit fixed and verified during governed implementation | C, S | PLANNED |
| MS-31 | One fixed policy for EI-6C v1: no operator-selectable Δ; no mid-issuance policy switching | C, A | PLANNED |
| MS-32 | EXPLICIT, STRATEGY_LISTING and DECLARED_SESSION scope modes, including a newly issued recommendation not yet visible in a declaration view | C, S | PLANNED |
| MS-33 | Historical compatibility: previously valid EI-6B history stays authoritative under the policy; no retrospective policy-consistency or deferral-window check invalidates it | C, S | PLANNED |
| MS-34 | Fixed catalog, unstable eligibility: with differing `issued_at` values, a fixed catalog can produce different results across monitoring instants when the eligible sets are not stable ordered prefixes; accepted path dependence | C, S | PLANNED |
| MS-35 | Regression coverage fails if the `issued_at <= T` requirement is removed; `as_of`-only eligibility is not approved | C | PLANNED |
| MS-36 | A snapshot that becomes eligible after a later event is persisted is diagnosed (`LATE_HISTORICAL_SNAPSHOT_IGNORED`), not evaluated retroactively; history unchanged | C, S | PLANNED |
| MS-37 | A newly eligible lower-ID equivalent snapshot does not rewrite an already committed event's reference | C, S | PLANNED |
| MS-38 | A later EI-5C refusal preserves the events committed earlier through incremental recording | C, S | PLANNED |
| MS-39 | An unexpected Core or verification error fails the issuance closed with `ISSUANCE_EVALUATION_FAILED`; earlier committed events stay authoritative | C, S | PLANNED |
| MS-40 | A snapshot with `as_of <= T` but `issued_at > T` is not selected; a snapshot with `issued_at = T` is (exact equality); demonstrated by a retrospective monitoring call | C, S | PLANNED |

| Count | Value |
| --- | --- |
| Scenario rows | 40 (MS-01 to MS-40, no gaps or duplicates) |
| PLANNED | 40 |
| Any other status | 0 |

**Verification strategy.** Local Windows synthetic and real-SQLite runs, then independent Linux CI for each implementation PR before owner acceptance, with the spawned-process and fresh-process rows (P) shown to execute on Linux. The gate's exact form is set at implementation authorization.

## 13. Limitations

- **Caller-supplied timestamps:** `issued_at` eligibility is declared, not proven; backdating is undetectable.
- **Path dependence:** live monitoring history depends on which snapshots existed at each run (section 10).
- **Non-durable policy provenance** (section 11).
- **Re-evaluation cost** after the last event, bounded by the typed limit rather than by progress.
- **DECLARED_SESSION completeness** (section 7).
- **Option A:** possible premature synthetic terminal `INCOMPLETE_EVIDENCE` under modeled delayed finality; no real-data finality guarantee.
- **Untyped recorder errors** fail the whole issuance (section 6.2).
- **Single host;** synthetic evidence only; MODELED only.
- **Inherited:** the EI-6B limitations E1 to E9 and the carried-forward restrictions (EI4-D8, VC-04, N6, I08, IS-16, NB-04, EI-6A HW-03); EI-4 formal CI closure (EI4-C4) outstanding.

## 14. Explicit non-authorizations

Not authorized: implementation code in any repository; research, generation, declaration or issuance orchestration; new snapshot issuance behaviour; background polling or scheduling; live or real market data and provider integration; ACTUAL issuance; broker orders or fills; position management; real-money trading; production deployment; API or Web integration; EI-8 restatement; terminal-state reopening; historical lifecycle rewriting; EI4-D8 relaxation; multi-host durability; changes to the EI-6A or EI-6B persistence contracts.

## 15. Implementation sequence (after a separate authorization)

| PR | Repository | Delivers |
| --- | --- | --- |
| PR-1 | northstar-application | Catalog port; selection policy; monitoring use case; results; C and A tests |
| PR-2 | northstar-infrastructure | Read-only SQLite catalog adapter (no schema change); S and P tests |
| PR-3 | northstar-docs | Engineering acceptance record; Linux CI evidence; owner decision |

## 16. Related documents

- [EI-5 Decision Register](../product/Equity-Intraday-V1-EI5-Decision-Register.md), sections 6, 17.3, 19 to 22
- [EI-3 Decision Register](../product/Equity-Intraday-V1-EI3-Decision-Register.md), sections 10.2, 11.2 and 12
- [ADR-024](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), section 13
- [EI-5 Issuance and Lifecycle Design v1.0](Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [EI-6B Lifecycle Persistence Design v1.0](Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md)
- [EI-6B Engineering Acceptance Record](../product/Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Engineering-Acceptance-Record.md)
- [Equity Intraday V1 Product Specification, section 8.2](../product/Equity-Intraday-V1-Product-Specification.md#82-lifecycle-states)
