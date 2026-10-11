# Futures Trade Recommendation Design Specification v1.0

Status: **design for F-2; implemented in development in `northstar-core`, not released.** Governed by ADR-017 (Proposed; its product decisions are approved). This specification adds no architectural decision beyond ADR-017. Section 23 lists the questions it deliberately leaves open.

Revision: this version includes the B1 (observation eligibility) and B2 (time-driven expiry) corrections approved after the independent F-2 review, and the DOC-1 (outcome coverage) and DOC-2 (ES-002 ordering deviation) corrections from the final review. Section 24 records them.

## 1. Business Purpose and Meaning

A **Trade Recommendation** is an actionable trade setup for one Futures contract: a LONG or SHORT direction, an entry zone, a stop, a target and a validity window, issued at a known instant on the strength of a frozen research decision.

Its **Recommendation Lifecycle** records, from completed market observations and explicit time evidence, whether and when the setup became active and how it ended.

The lifecycle is an observational model. It states what completed bars show about the setup's price levels, and which documented modeling rule was applied where a bar cannot show chronology. It is not an execution record:

- a Recommendation Event is not an order, a fill, a position or a broker execution;
- F-2 records no entry price, exit price, quantity, return or profit and loss.

F-2 delivers only the immutable contracts and the pure lifecycle behavior. It does not generate, persist, publish or monitor recommendations.

## 2. Existing Types Reused

| Type | Package | Use |
| --- | --- | --- |
| `FuturesContract` | `northstar_core.futures` | The contract the setup trades. |
| `FuturesOHLCVBar` | `northstar_core.futures` | The completed observation the lifecycle evaluates. `point_in_time` is the interval completion instant, not publication or arrival time. Identity is `(contract, point_in_time, timeframe)`. |
| `QuoteValue` | `northstar_core.derivatives` | Every price level. Canonical Decimal and ordered; comparisons are exact, never binary floating point. No tick or rounding is applied. |
| `PointInTime` | `northstar_core.foundation.value_objects` | Issuance, decision, eligibility, validity and as-of instants. Compared only through `PointInTime.compare`; it has no ordering operators. |
| `Timeframe` | `northstar_core.foundation.value_objects` | The research decision's timeframe. F-2 accepts only `1d`. |
| `StrategyIdentity` | `northstar_core.strategy` | The strategy that made the source research decision. |
| `ExplanationReason` | `northstar_core.strategy` | One thesis rationale with its supporting signals. |

Not reused, under ADR-017:
- `RecommendationAction` (research vocabulary);
- `OrderSide` and every Paper Trading or Broker Execution type.

`FuturesRecommendation` is not embedded. The source decision is referenced by the natural key of the frozen research record instead (section 4).

## 3. New Types and Public API

Package `northstar_core.recommendations.futures`. All of these are exported from that package:

| Type | Kind | Purpose |
| --- | --- | --- |
| `TradeDirection` | `StrEnum` | `LONG`, `SHORT`. |
| `FuturesTradeRecommendationIdentity` | value object | Caller-supplied opaque identity. |
| `RecommendationPolicyVersion` | value object | Caller-supplied opaque version of the generation policy that produced the setup. |
| `FuturesResearchDecisionReference` | value object | Natural key of the source frozen research record. |
| `EntryZone` | value object | Inclusive entry price range. |
| `StopLoss` | value object | Fixed stop price. |
| `PriceTarget` | value object | Fixed target price. |
| `RecommendationValidity` | value object | First eligible observation, last eligible observation and contract cutoff. |
| `FuturesTradeRecommendation` | value object | The immutable trade setup. |
| `RecommendationStatus` | `StrEnum` | Derived lifecycle state. |
| `RecommendationEventKind` | `StrEnum` | Kind of lifecycle event. |
| `RecommendationBoundary` | `StrEnum` | `ENTRY`, `STOP`, `TARGET`: a price level a bar touched. |
| `IntrabarOrdering` | `StrEnum` | How an event treats chronology inside its bar. |
| `RecommendationExpiryReason` | `StrEnum` | Why a validity expiry occurred. |
| `ObservationCoverage` | `StrEnum` | `COMPLETE_VERIFIED_COVERAGE`, `INCOMPLETE_OR_UNVERIFIED_COVERAGE`. |
| `CoverageDeficiency` | `StrEnum` | `MISSING_OBSERVATIONS`, `UNAPPROVED_OBSERVATIONS`, `UNVERIFIED_OBSERVATIONS`. |
| `TimeExpiryEvidence` | value object | Caller-supplied evidence for a time-driven expiry. |
| `FuturesTradeRecommendationEvent` | value object | One immutable lifecycle event. |
| `FuturesTradeRecommendationLifecycle` | value object | Recommendation, ordered events and the last evaluated observation, with the pure `advance` and `expire` transitions. |

**Error types.** Each non-enum value object has a matching `Invalid…Error(ValidationError)`. For the lifecycle this is `InvalidFuturesTradeRecommendationLifecycleError`, which reports an invalid stored history. Two further errors report input a transition refuses:
- `InvalidFuturesTradeRecommendationObservationError` for a bar refused by `advance`;
- `InvalidFuturesTradeRecommendationExpiryError` for time evidence refused by `expire`.

**Modules.**
- `value_objects/` holds one module per value object. Two use short names to keep imports within the line limit: `recommendation_identity.py` and `research_decision_reference.py`.
- `lifecycle/` holds:
  - `recommendation_status.py`;
  - `time_expiry_evidence.py` (the evidence and its two enumerations);
  - `recommendation_event.py` (the event and its four enumerations);
  - `futures_trade_recommendation_lifecycle.py`.

**Structural form.** Every value object is `@dataclass(frozen=True, slots=True)`. Validation runs in `__post_init__` through module-level helpers, and each type has explicit `__str__` and `__repr__`.

**ES-002 ordering deviation.** ES-002 names `@dataclass(frozen=True, slots=True, order=True)` as the default form, and requires any deviation to be documented and reviewed. These types deliberately omit `order=True`, because none has a business-meaningful total order:
- a setup, zone, validity window, event, evidence or lifecycle has no "less than" another;
- generated field-by-field ordering would be arbitrary, and could be mistaken for ranking or chronology.

This is the same deviation `PointInTime`, `Price`, `Money` and `FuturesContract` make. Chronology is expressed only through `PointInTime.compare`, and no artificial comparison is implemented.

**Rationale.**
- *Architectural:* ordering is domain behavior, and a type should expose only behavior that has business meaning. A generated total order on a setup, window, event, evidence or lifecycle would invite comparisons that the domain does not define, such as treating field order as precedence or time.
- *Engineering:* omitting `order=True` keeps equality and hashing (both required, and tested) while making accidental `<` comparisons fail loudly. It also matches the established practice for non-orderable Core values.

**Approval status.**
- The product owner has approved omitting `order=True` for these F-2 types, where no meaningful total order exists.
- ES-002 additionally requires both an Architecture Review and an Engineering Review for any deviation from the reference style, and a deviation that is not documented and approved must be rejected in code review.
- Neither reviewer sign-off is recorded yet. Both are **pending** (see the Approval Summary), and must be recorded in the F-2 pull-request reviews or in this specification before merge.

## 4. Recommendation Identity and Provenance

- **`FuturesTradeRecommendationIdentity(identity: str)`** is opaque, stripped and non-empty. Core never generates, hashes or parses it. A future Application service will derive it deterministically, at minimum from the source research reference and the policy version. That derivation is not specified here.
- **`RecommendationPolicyVersion(version: str)`** is opaque, stripped and non-empty. It names the recommendation-generation policy (F-3/F-4) and its version.
- **`FuturesResearchDecisionReference(contract, timeframe, decision_instant, strategy_identity)`** carries exactly the natural key of a frozen futures research record. It is a reference, not the research decision itself. `natural_key` returns those four values.

Core does not enforce one recommendation per identity. A recommendation with a different definition under the same identity is a different value; the F-5 persistence layer must enforce uniqueness and conflict on a mismatched rewrite.

## 5. Direction

`TradeDirection.LONG` profits from a rise and `TradeDirection.SHORT` from a fall. Neither is derived from, or convertible to, a Research Action, Target Exposure or Order Side. F-2 implements no mapping from BUY, SELL or HOLD to a direction.

## 6. Entry Zone, Stop, Target and Validity

- **`EntryZone(low, high)`:** QuoteValues with `low <= high`. A single-price zone (`low == high`) is valid. Both bounds are inclusive.
- **`StopLoss(price)`:** one fixed QuoteValue. There is no trailing, moving or revisable stop in F-2.
- **`PriceTarget(price)`:** one fixed QuoteValue. V1 has exactly one target. There is no target sequence, partial exit or allocation.
- **`RecommendationValidity(eligible_from, valid_through, contract_cutoff)`:** three PointInTimes with `eligible_from <= valid_through <= contract_cutoff`. All three are completion instants of observations.
  - **`eligible_from`** (inclusive) is the completion instant of the first trading session that began strictly after the recommendation was issued.
    - A daily bar that completes after issuance can still contain prices traded before it. Completing after issuance is therefore not enough; only bars completing at or after `eligible_from` are evaluated.
  - **`valid_through`** (inclusive) is the completion instant of the last observation the recommendation may consume.
  - **`contract_cutoff`** is the completion instant of the last observation of the contract. Calendar expiry date and last tradable session are not assumed to be the same.

**Trusted caller contract.** All three instants are resolved by Application from the authoritative trading-session resolver. Core computes no session, calendar or expiry schedule. It enforces only the order of the instants it is given.
- A timestamp alone cannot prove that the session closing at `eligible_from` began after issuance.
- A timestamp alone cannot prove that `contract_cutoff` matches the contract's expiry (section 23).

## 7. Domain Invariants

A `FuturesTradeRecommendation` cannot be constructed unless all of the following hold:

1. Every field has its exact type. No implicit conversion from `float`, `str` or `Decimal` is attempted.
2. `contract == source.contract`.
3. `source.timeframe == Timeframe("1d")`. F-2 defines daily-candle semantics only.
4. **LONG:** `stop < entry.low <= entry.high < target`.
5. **SHORT:** `target < entry.low <= entry.high < stop`.
6. `source.decision_instant <= issued_at`: a setup is never issued before its research decision.
7. `issued_at < validity.eligible_from`: the first eligible observation completes strictly after issuance. Together with `RecommendationValidity`, this gives `issued_at < eligible_from <= valid_through <= contract_cutoff`.
8. `thesis` is a non-empty tuple of `ExplanationReason`.

Price checks use `QuoteValue` comparison, which compares canonical Decimals exactly. Numerical validity does not prove exchange tradability: no tick-size model exists yet.

## 8. Immutable Recommendation

```
FuturesTradeRecommendation(
    identity: FuturesTradeRecommendationIdentity,
    contract: FuturesContract,
    direction: TradeDirection,
    entry: EntryZone,
    stop: StopLoss,
    target: PriceTarget,
    validity: RecommendationValidity,
    source: FuturesResearchDecisionReference,
    policy_version: RecommendationPolicyVersion,
    issued_at: PointInTime,
    thesis: tuple[ExplanationReason, ...],
)
```

`strategy_identity` is a read-only property of `source`. The recommendation has no status field. Its state lives only in the lifecycle (section 9).

## 9. Lifecycle Statuses

The status is derived from the event history and is never stored.

| Status | Meaning | Terminal | Outcome classification |
| --- | --- | --- | --- |
| `WAITING_FOR_ENTRY` | No eligible observation has touched the entry zone. | No | — |
| `ACTIVE` | An observation touched the entry zone without touching stop or target. | No | — |
| `CLOSED_TARGET` | While active, an observation touched the target and not the stop. | Yes | Candidate: verified only under the coverage conditions below |
| `CLOSED_STOP` | While active, an observation touched the stop. If it also touched the target, the stop-first policy applied (modeled). | Yes | Candidate: verified only under the coverage conditions below; modeled when stop-first |
| `EXPIRED_BEFORE_ENTRY` | Validity ended while waiting: on an observed bar, or by time with asserted complete verified coverage. | Yes | Candidate: verified only under the coverage conditions below |
| `EXPIRED_AFTER_ACTIVATION` | Validity ended while active: on an observed bar, or by time with asserted complete verified coverage. No exit price is claimed. | Yes | Candidate: verified only under the coverage conditions below |
| `INDETERMINATE_ENTRY` | The entry observation also touched stop and/or target, so whether and how the setup activated cannot be known from daily data. | **Yes** | **Never a verified outcome** |
| `EXPIRED_WITH_INCOMPLETE_COVERAGE` | Validity ended by time while some required observation was missing, unapproved or unverified. | **Yes** | **Never a verified outcome** |

- **`INDETERMINATE_ENTRY` is terminal.** Continuing would require assuming that entry happened and that the exit level touched in the same bar did not come first. That is a chronology a daily candle does not contain, and L-3 forbids inventing it. An indeterminate recommendation is neither a winner nor a loser.
- **`EXPIRED_WITH_INCOMPLETE_COVERAGE` is terminal.** Its meaning:
  - it does not mean the stop or target was not reached;
  - it does not mean the recommendation was safely executed;
  - it is not a normal verified expiry.

  Whether the recommendation had activated is still visible in its events.

**Coverage conditions for a verified outcome.** Core evaluates only the bars it receives. A terminal status records what those bars, or the supplied time evidence, showed. It does not prove that the market-data coverage behind it was complete:

- **Bar-driven outcomes prove nothing about coverage.** `CLOSED_TARGET`, `CLOSED_STOP`, and `EXPIRED_BEFORE_ENTRY` or `EXPIRED_AFTER_ACTIVATION` reached on a bar do not show that every eligible session was observed. For example, if a session that touched the stop is never supplied, a later bar touching the target yields `CLOSED_TARGET`. Likewise, supplying only a bar after `valid_through` yields `EXPIRED_BEFORE_ENTRY`.
- **Time expiry carries explicit evidence, but not proof.** A time-driven expiry records `TimeExpiryEvidence`. Even so, `COMPLETE_VERIFIED_COVERAGE` is a trusted caller assertion (section 17.3). Core checks only that it is consistent with the evaluated history; it does not establish that the assertion is true.
- **Missing sessions are never treated as quiet.** A session that was not observed must never be interpreted as a session without a stop or target touch. A gap in the supplied bars is a coverage deficiency, not evidence that a level was not reached.

A terminal status is therefore a **verified outcome** only when the future Application layer guarantees complete, approved, gap-free, chronological session observations from `eligible_from` through the terminal event. Two cases need more:
- a time expiry must also carry `COMPLETE_VERIFIED_COVERAGE` supplied by a trusted coverage check;
- a `MODELED_STOP_FIRST` closure is also a modeled outcome, not an observed one.

`INDETERMINATE_ENTRY` and `EXPIRED_WITH_INCOMPLETE_COVERAGE` are never verified outcomes.

F-6 must apply these conditions. The status, the event kind, the intrabar ordering marker and the time evidence give it the data to do so, but the coverage guarantee itself must come from Application (F-5). F-2 implements no aggregation.

## 10. Lifecycle Event Structure

```
FuturesTradeRecommendationEvent(
    recommendation: FuturesTradeRecommendationIdentity,
    kind: RecommendationEventKind,
    observation: FuturesOHLCVBar | None,
    touched: tuple[RecommendationBoundary, ...],
    ordering: IntrabarOrdering,
    expiry_reason: RecommendationExpiryReason | None = None,
    time_evidence: TimeExpiryEvidence | None = None,
)
```

Every event has exactly one kind of evidence:
- **`observation`:** the completed bar that caused the event, kept whole so the observed prices remain auditable. Every kind except `VALIDITY_EXPIRED_BY_TIME` has one and no time evidence.
- **`time_evidence`:** the evidence of a `VALIDITY_EXPIRED_BY_TIME` event, which has no bar. A non-bar event never borrows a bar's identity.
- `evidenced_at` returns the bar's completion instant, or the evidence's `as_of`.

The other fields:
- `touched` lists the boundaries the bar touched, in canonical order ENTRY, STOP, TARGET.
- `ordering` states how chronology inside the bar was treated:
  - `NOT_REQUIRED`: at most one price boundary was involved, so no ordering question arises.
  - `MODELED_STOP_FIRST`: both stop and target were touched while active, and the stop-first policy, a modeling rule, decided the outcome.
  - `UNKNOWN`: entry and an exit level were touched in one bar, and the order is unknowable.

Allowed combinations (any other combination is rejected at construction):

| `kind` | Evidence | `touched` | `ordering` | `expiry_reason` |
| --- | --- | --- | --- | --- |
| `ENTRY_ACTIVATED` | bar | `(ENTRY,)` | `NOT_REQUIRED` | `None` |
| `ENTRY_INDETERMINATE` | bar | `(ENTRY, STOP)`, `(ENTRY, TARGET)` or `(ENTRY, STOP, TARGET)` | `UNKNOWN` | `None` |
| `TARGET_REACHED` | bar | `(TARGET,)` | `NOT_REQUIRED` | `None` |
| `STOP_REACHED` | bar | `(STOP,)` | `NOT_REQUIRED` | `None` |
| `STOP_REACHED` | bar | `(STOP, TARGET)` | `MODELED_STOP_FIRST` | `None` |
| `VALIDITY_EXPIRED` | bar | `()` | `NOT_REQUIRED` | `VALIDITY_WINDOW_ENDED` or `CONTRACT_CUTOFF_REACHED` |
| `VALIDITY_EXPIRED_BY_TIME` | time evidence | `()` | `NOT_REQUIRED` | `VALIDITY_WINDOW_ENDED` or `CONTRACT_CUTOFF_REACHED` |

The expiry reason is `CONTRACT_CUTOFF_REACHED` when `valid_through` equals `contract_cutoff`, and `VALIDITY_WINDOW_ENDED` otherwise.

There is no event bus, outbox or persistence.

## 11. Pure Transitions

`FuturesTradeRecommendationLifecycle(recommendation, events=(), last_observation=None)` is immutable.

- **`last_observation`** is the latest bar the lifecycle evaluated, including bars that produced no event. It is the minimum observation reference needed to prove idempotence and ordering: a bar that changes nothing leaves no event, so without it an event history alone could not show that a later bar was already consumed.
- **`lifecycle.advance(bar: FuturesOHLCVBar) -> FuturesTradeRecommendationLifecycle`** evaluates one completed bar (sections 12 to 17).
- **`lifecycle.expire(evidence: TimeExpiryEvidence) -> FuturesTradeRecommendationLifecycle`** expires an open recommendation by time (section 17).

In both cases, the new events are those beyond the input's `events`.

Both transitions are pure and deterministic. They:
- read no clock;
- generate no identifier;
- mutate nothing;
- perform no I/O;
- depend on no global mutable state.

`status`, `is_terminal` and `events` are read-only. Constructing a lifecycle directly from stored parts validates the whole history (section 19).

## 12. Temporal Ordering

`advance(bar)` applies these steps in order:

1. **Contract.** `bar.contract` must equal the recommendation's contract, else `InvalidFuturesTradeRecommendationObservationError`.
2. **Timeframe.** `bar.timeframe` must equal `source.timeframe` (`1d`), else the same error.
3. **Eligibility.** If `bar.point_in_time < validity.eligible_from`, the bar is ineligible: its session began before the recommendation was issued, or it completed before issuance.
   - It is **ignored and not consumed**: the lifecycle is returned unchanged, `last_observation` does not move, and no error is raised, even when it arrives after later bars.
   - It can never activate or close the recommendation.
4. **Order.** If `last_observation` exists:
   - an earlier bar is refused as out of order;
   - a bar at the same instant that equals `last_observation` returns the lifecycle unchanged (idempotent);
   - a bar at the same instant with different values is refused as a revised observation.
5. **Terminal.** A terminal lifecycle is returned unchanged. It is never reopened, whether it ended by a bar or by time.
6. **Evaluate.** Otherwise the bar is evaluated (sections 13 to 17). The result carries the new events and `last_observation = bar`.

Instants are compared only with `PointInTime.compare`.

## 13. Daily-Candle Interpretation

A completed bar shows its open, high, low and close. It does not show the order in which its high and low occurred, or when any level traded within the session. The lifecycle therefore asks only which boundaries the bar's range `[low, high]` touched. Touching is inclusive: equality counts.

| Boundary | LONG touched when | SHORT touched when |
| --- | --- | --- |
| Entry | `low <= entry.high` and `high >= entry.low` | same |
| Stop | `low <= stop` | `high >= stop` |
| Target | `high >= target` | `low <= target` |

The entry test is a range intersection. A bar trading entirely outside the entry zone does not activate a recommendation. This holds even when the whole bar is on the economically favorable side of the zone, for example a LONG bar entirely below the zone.

This models the approved entry-zone touch rule. It does not model general exchange limit-order execution, under which such a bar might have filled. Open-price disambiguation is deferred (section 23).

## 14. Entry Activation (WAITING_FOR_ENTRY)

| Entry touched | Stop touched | Target touched | Result |
| --- | --- | --- | --- |
| no | any | any | No event; remain waiting. |
| yes | no | no | `ENTRY_ACTIVATED`, so `ACTIVE`. |
| yes | yes | no | `ENTRY_INDETERMINATE (ENTRY, STOP)`, so `INDETERMINATE_ENTRY`. |
| yes | no | yes | `ENTRY_INDETERMINATE (ENTRY, TARGET)`, so `INDETERMINATE_ENTRY`. |
| yes | yes | yes | `ENTRY_INDETERMINATE (ENTRY, STOP, TARGET)`, so `INDETERMINATE_ENTRY`. |

On the activation bar, no exit is evaluated. If an exit level had been touched, the bar would be indeterminate instead. Stop-first is never applied to an entry bar.

While waiting, a bar that touches the stop or target without touching the entry zone changes nothing (section 23).

## 15. Target and Stop (ACTIVE)

| Stop touched | Target touched | Result |
| --- | --- | --- |
| no | no | No event; remain active. |
| no | yes | `TARGET_REACHED`, so `CLOSED_TARGET`. |
| yes | no | `STOP_REACHED (STOP,)`, so `CLOSED_STOP`. |
| yes | yes | `STOP_REACHED (STOP, TARGET)` with `MODELED_STOP_FIRST`, so `CLOSED_STOP`. |

Stop-first is the approved conservative L-3 policy. The event records that both boundaries were observed in one daily range and that the intraday order is unknown. The closure is a modeled outcome, not an observed or verified execution sequence.

## 16. Ambiguity Semantics

| Situation | Ordering | Outcome classification |
| --- | --- | --- |
| Single boundary touched | `NOT_REQUIRED` | Observed |
| Active, stop and target both touched | `MODELED_STOP_FIRST` | Modeled (conservative) |
| Waiting, entry and an exit level both touched | `UNKNOWN` | Indeterminate; terminal; no winner or loser |

No case assigns an entry or exit fill.

## 17. Expiry and Validity

### 17.1 Bar-driven expiry

After the eligibility and order checks, an eligible bar is evaluated against validity:

- **`bar.point_in_time > valid_through`:** the window has already passed. Prices are not evaluated. A non-terminal lifecycle receives `VALIDITY_EXPIRED` with `touched = ()`, so it becomes `EXPIRED_BEFORE_ENTRY` or `EXPIRED_AFTER_ACTIVATION`.
- **`bar.point_in_time == valid_through`:** the bar is evaluated as in sections 14 and 15. If the lifecycle is still non-terminal afterwards, `VALIDITY_EXPIRED` follows on the same observation. An activation on the last eligible bar is therefore followed by `VALIDITY_EXPIRED`, giving `EXPIRED_AFTER_ACTIVATION`.
- **`bar.point_in_time < valid_through`:** the bar is evaluated normally.

When `eligible_from == valid_through`, the single eligible bar is evaluated and then the window ends.

### 17.2 Time-driven expiry

After a contract's last session no later bar can exist. The final bar itself may be missing, or stamped at an instant other than the supplied cutoff. In either case bar-driven expiry alone could leave a recommendation open forever.

`expire(evidence)` ends an open recommendation by time, applying these steps in order:

1. **Type.** Anything other than `TimeExpiryEvidence` is refused with `InvalidFuturesTradeRecommendationExpiryError`.
2. **Terminal.** A terminal lifecycle is returned unchanged, whatever the evidence. Repeated expiry is idempotent.
3. **Consistency.** `evidence.evaluated_through` must be the completion instant of `last_observation`, or both must be absent. Otherwise the evidence does not describe this lifecycle's evaluated history and is refused, even if premature.
4. **Never premature.** If `evidence.as_of < valid_through`, the lifecycle is returned unchanged.
5. **Expire.** Otherwise a `VALIDITY_EXPIRED_BY_TIME` event records the evidence, and `last_observation` is unchanged:
   - with `COMPLETE_VERIFIED_COVERAGE`: `EXPIRED_BEFORE_ENTRY` or `EXPIRED_AFTER_ACTIVATION`;
   - with `INCOMPLETE_OR_UNVERIFIED_COVERAGE`: `EXPIRED_WITH_INCOMPLETE_COVERAGE`.

**The validity-end bar is never discarded.** When a bar at `valid_through` is available, the caller advances it before expiring by time, so its prices are evaluated, and that bar ends the lifecycle itself. Complete verified coverage asserts exactly that every eligible observation through the window was evaluated in order. If the bar is missing, no outcome is fabricated: the time expiry records the deficiency.

**A time expiry is final.** A late bar arriving afterwards is ignored once the order checks pass (section 12, step 5). It cannot rewrite the expiry as a target or stop closure.

### 17.3 Time-expiry evidence

```
TimeExpiryEvidence(
    as_of: PointInTime,
    evaluated_through: PointInTime | None,
    coverage: ObservationCoverage,
    attested_by: str,
    deficiency: CoverageDeficiency | None = None,
)
```

- `as_of` is the instant the expiry is evaluated at. It is supplied by the caller, never read from a clock.
- `evaluated_through` is the completion instant of the last bar the lifecycle evaluated. It is checked against the lifecycle (17.2, step 3) and must not be later than `as_of`.
- `COMPLETE_VERIFIED_COVERAGE` names no deficiency and requires `evaluated_through`: coverage of nothing cannot be complete.
- `INCOMPLETE_OR_UNVERIFIED_COVERAGE` names exactly one `CoverageDeficiency`: missing, unapproved or unverified observations.
- `attested_by` is opaque, non-empty text naming who made the coverage assessment, so provenance and responsibility stay visible.

Core never manufactures `COMPLETE_VERIFIED_COVERAGE`. It is not inferred from:
- the passage of time;
- the existence of a last bar;
- a non-null `last_observation`;
- an empty event list;
- a request to expire.

The assertion is a trusted caller statement, not proof, and Core cannot verify a data provider, approval manifest or exchange calendar. Only a future trusted Application service may supply it, after checking:
- coverage against the trading-session calendar;
- the approved stored bars through `valid_through`.

Tests construct synthetic verified evidence. Nothing here is provider-specific.

**What Core can and cannot enforce.** Core enforces:
- that the evidence matches the evaluated history (`evaluated_through`);
- that it is not premature;
- that the bar-event history before it replays exactly;
- that the cursor would have produced no unrecorded events.

Core cannot enforce that a verified assertion is true. For example, it cannot detect a session that was never supplied to `advance`. Bar-driven events likewise assume the caller supplies every eligible, approved session bar in chronological order without gaps. Core cannot verify that assumption, so a bar-driven outcome is no stronger than the caller's coverage guarantee (section 9).

## 18. Deterministic Replay and Idempotency

- Folding the same bars, and the same evidence, in the same order always produces the same lifecycle, compared by value.
- Re-applying the latest evaluated bar is a no-op.
- An ineligible bar is ignored and not consumed.
- Applying any bar to a terminal lifecycle is a no-op once the contract, timeframe, eligibility and order checks pass.
- Applying any time evidence to a terminal lifecycle is a no-op.
- No event is ever duplicated or removed.
- Bars are deduplicated by value and natural key, never by Python object identity.
- An earlier bar, or a revised value for the latest bar, is refused rather than re-sorted or merged.

## 19. Exceptions and Invalid Histories

- **Value objects.** Each raises its own `Invalid…Error` for a wrong type, empty identity, price-order violation, temporal violation, inconsistent event combination or inconsistent coverage evidence.
- **`advance`** raises `InvalidFuturesTradeRecommendationObservationError` for:
  - a non-bar argument;
  - another contract or timeframe;
  - an out-of-order bar;
  - a revised latest bar.
- **`expire`** raises `InvalidFuturesTradeRecommendationExpiryError` for:
  - a non-evidence argument;
  - evidence whose `evaluated_through` does not match the lifecycle.
- **Stored parts.** Constructing `FuturesTradeRecommendationLifecycle` raises `InvalidFuturesTradeRecommendationLifecycleError` unless the history replays exactly. Bar events:
  - every event belongs to this recommendation;
  - at most one `VALIDITY_EXPIRED_BY_TIME` event exists, and only as the last event;
  - bar events are grouped by observation, with observations strictly increasing and each eligible (at or after `eligible_from`, for the contract and timeframe);
  - each group equals what `advance` produces from the preceding state, so forged, reordered, missing-precondition or contradictory events are rejected;
  - nothing follows a terminal bar event.

  The cursor and any time expiry:
  - `last_observation` is eligible, present whenever bar events exist, not earlier than the last bar event's bar, and equal to it at the same instant;
  - when terminal by a bar event, `last_observation` equals that event's bar;
  - otherwise it produces no event when evaluated from the final bar-event state;
  - a trailing time expiry follows a non-terminal state, is not premature, matches `last_observation` through `evaluated_through`, and equals what `expire` produces from its evidence, including the expiry reason.

Malformed histories are never sorted or repaired.

## 20. Public Exports

`northstar_core.recommendations.futures.__all__` has exactly 32 names:
- the 19 types of section 3;
- 13 error types:
  - one `Invalid…Error` for each of the 11 non-enum value objects;
  - `InvalidFuturesTradeRecommendationObservationError`;
  - `InvalidFuturesTradeRecommendationExpiryError`.

The enumerations have no error types; an invalid member raises `ValueError`.

Sub-packages:
- `value_objects` (17 names) exports the setup types: `TradeDirection` through `FuturesTradeRecommendation` and their errors;
- `lifecycle` (15 names) exports the status, coverage evidence, event and lifecycle types and their errors.

`northstar_core.recommendations` is a namespace parent with an empty `__all__`.

## 21. Package Boundaries

Under ADR-017, the package may import only:
- Foundation, Derivatives, Futures, Strategy and its own modules;
- `__future__`, `dataclasses` and `enum`.

It never imports:
- Paper Trading, Broker Execution, Market Data, Orders, Trades, Portfolio or Domain;
- a clock, randomness, environment, network, persistence or any outer repository.

No Core package outside it imports it. Strategy in particular never does. Importing it in a fresh interpreter must not load Paper Trading or Broker Execution. Boundary tests enforce all of this through the AST and a subprocess import.

## 22. Test Matrix

| Area | Cases |
| --- | --- |
| Value objects | Valid and invalid construction; wrong types; immutability; equality and hashing; `str` and `repr`; identity and policy-version normalization; single-price zone; LONG and SHORT invariants including boundary equality; contract, timeframe and temporal relationships; `eligible_from` ordering (equal to `valid_through` allowed, later rejected); issuance strictly before `eligible_from`; validity versus cutoff; empty thesis; event kind and evidence combinations; coverage evidence invariants |
| Lifecycle (bars) | (1) waiting, no entry; (2, 3) LONG and SHORT activation; (4, 5) stop; (6, 7) target; (8, 9) active both-touch, stop-first; (10, 11) entry and stop ambiguity; (12, 13) entry and target ambiguity; (14) entry, stop and target; (15, 16) gaps above and below the zone; (17) bars before issuance and before `eligible_from`; (18) bar at `valid_through`, including activation on it; (19) bar after `valid_through`; (20) contract-cutoff expiry reason and a wrong-contract bar; (21) duplicate bar; (22) out-of-order bar and revised bar; (23) contradictory history; (24) forged or incompatible event; (25) terminal reprocessing; (26) deterministic replay and reconstruction from stored parts; (27) no mutation; (28) no inferred intraday order (no fill or price on events) |
| B1 regression | LONG and SHORT mid-session issuance; the same-session bar touching entry, stop, target or both never activates or closes and is not consumed; first eligible bar exactly at `eligible_from` activates; a later bar closes; one microsecond before `eligible_from` ignored; `eligible_from == valid_through`; replay cannot bypass eligibility through events or the cursor |
| B2 regression | Final bar missing; final bar earlier than the cutoff then verified expiry; last observation before `valid_through`; cutoff with incomplete and with verified evidence; `as_of` before, at and after `valid_through`; repeated expiry; expiry after `CLOSED_TARGET`, `CLOSED_STOP`, `INDETERMINATE_ENTRY` and an expired lifecycle; evidence contradicting the evaluated history; late bars after time expiry; the validity-end bar evaluated before time expiry; forged, misplaced, premature, duplicated, foreign or wrongly-reasoned time expiries; reconstruction; no verified classification for incomplete coverage |
| L-3 preservation | LONG and SHORT normal entry, ambiguous entry, active stop-first, gaps on either side of the zone, no same-bar exit after activation |
| Boundaries | Approved imports only; no Paper Trading, Broker Execution or outer layers; no clock calls; no reverse import from Strategy or any other Core package; fresh-interpreter import isolation and no import cycle; exact public surface; no deferred concept (order, fill, position, pnl, broker, margin and similar) |

## 23. Non-Goals, Open Questions and Later Dependencies

**Not in F-2:**
- recommendation generation;
- mapping BUY/SELL/HOLD to LONG/SHORT;
- price-level, ATR or volatility rules;
- tick alignment;
- risk sizing and margin;
- persistence, publication, monitoring and notification;
- performance, returns and P&L;
- paper comparison;
- API and Web;
- Options;
- open-price disambiguation of daily bars (N1, deferred: L-3 is unchanged).

**Left open deliberately:** no rule is invented for these; each needs a product decision.

- **Invalidation before entry (N7, deferred to F-3/F-4).** Whether a waiting setup should be cancelled when price touches or gaps through the stop or target without touching the entry zone. F-2 remains waiting and never silently cancels.
- **Cancellation and revision.** Operator cancellation, and revision of levels or validity. Both would be new event kinds.
- **Session resolution.** How Application resolves `eligible_from`, `valid_through` and `contract_cutoff` from the NSE trading-session calendar and the contract's expiry date. Core cannot verify the correspondence.
- **Identity derivation and uniqueness (N4, deferred to F-5).** The deterministic identity algorithm, and enforcement of one definition per identity.

**Later dependencies:**
- F-3/F-4 build generation and risk validation on these contracts.
- **F-5** drives `advance` only from complete, approved, gap-free, chronological session bars, and drives `expire` from a trusted coverage check.
  - When it detects a missing, unapproved or unverifiable session, it must not feed later bars as if that session were quiet.
  - It must hold the lifecycle and, once validity has ended, expire it with `INCOMPLETE_OR_UNVERIFIED_COVERAGE` and the matching deficiency. Core has no incomplete state before validity ends.
- **F-6** derives outcomes under the coverage conditions of section 9:
  - it must exclude `INDETERMINATE_ENTRY` and `EXPIRED_WITH_INCOMPLETE_COVERAGE` from verified outcomes;
  - it must treat `MODELED_STOP_FIRST` closures as modeled;
  - it must count other terminal statuses as verified only where F-5's coverage guarantee holds.
- **F-5 persistence guidance (N3).** Do not introduce persistence or real candle fixtures in F-2. When F-5 persists:
  - Store the referenced OHLCV candle key and its approved `fdc1` fingerprint rather than duplicating raw OHLCV values in new recommendation tables. Rebuild bars from the insert-only, reviewed `futures_ohlcv` store.
  - Store recommendation events insert-only, keyed by recommendation identity and sequence. A `VALIDITY_EXPIRED_BY_TIME` event has no candle, so the event table must hold time evidence (as-of, evaluated-through, coverage, deficiency, attested-by) without a candle reference.
  - Insert-only rules forbid updating a stored cursor. Either recompute `last_observation` by replaying stored bars from `eligible_from`, or record each evaluated bar in an append-only evaluation journal.
  - Application and Infrastructure must validate the permission and retention conditions of their data sources (G-1 remains pending). Data-source authorization and recommendation regulatory permissions are separate.
- **Glossary (N6).** A narrowly scoped documentation follow-up should add Entry Zone, Eligible From, Contract Cutoff, Intrabar Ordering, Indeterminate Entry, Time Expiry and Observation Coverage to the canonical Ubiquitous Language. They are not changed in this task.

## 24. Revision Record

| Change | Reason |
| --- | --- |
| **B1:** added `RecommendationValidity.eligible_from`; eligibility is `bar.point_in_time >= eligible_from`; issuance must precede `eligible_from` | A daily bar completing after a mid-session issuance contained pre-issuance prices and could activate the recommendation. |
| **B2:** added `expire(TimeExpiryEvidence)`, `VALIDITY_EXPIRED_BY_TIME`, `ObservationCoverage`, `CoverageDeficiency` and `EXPIRED_WITH_INCOMPLETE_COVERAGE` | With the final bar missing or stamped before the cutoff, no further bar of the contract could ever arrive, and a recommendation could stay open forever. |
| **N2, N3, N5:** clarified gap semantics, F-5 persistence guidance and the ES-002 ordering rationale | Review findings. |
| **N1, N4, N6, N7:** recorded as deferred | Review findings. |
| **DOC-1:** section 9 "Verified outcome" column replaced by an outcome classification with explicit coverage conditions; sections 17.3 and 23 aligned | The final review found the column overstated what bar-driven outcomes prove: Core evaluates only the bars it receives. |
| **DOC-2:** ES-002 ordering deviation recorded with rationale, product-owner approval and pending reviewer sign-offs (section 3 and Approval Summary) | The final review found the deviation documented but not yet formally approved under ES-002. |

## Approval Summary

| Item | Value |
| --- | --- |
| Governing decision | ADR-017 (Proposed; product decisions approved, including L-3, B1 and B2) |
| Repository | `northstar-core` |
| Package | `recommendations.futures` |
| Deviation from ES-002 | No `order=True` where no meaningful total order exists; rationale in section 3 |
| ES-002 deviation: product owner | Approved |
| ES-002 deviation: Architecture Review | **Pending**: no sign-off recorded |
| ES-002 deviation: Engineering Review | **Pending**: no sign-off recorded |
| Final independent review | Completed: approve with non-blocking notes. DOC-1 and DOC-2 are addressed in this revision. |
| Status | Implemented in development; ADR-017 still Proposed; reviewer sign-offs above pending; not released |
