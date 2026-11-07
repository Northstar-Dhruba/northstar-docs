# Equity Intraday V1 Product Specification

## Status

**Draft for product-owner review (EI-0). Not approved as a whole: only owner decision EI0-D4 is approved.**

This document proposes the smallest product definition for Northstar's first actionable equity capability: NSE cash-equity, LONG-only, intraday trade recommendations built from completed 5-minute candles.

- Every owner decision is listed in the [owner decision register](#14-owner-decision-register). EI0-D4 is **APPROVED** (2026-11-01, [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md)). Every other decision is marked **PROPOSED** or **PENDING**.
- Apart from EI0-D4, nothing here has been approved by the product owner. Nothing here has been approved by Architecture Review or Engineering Review.
- No strategy, parameter, threshold or cost rate is approved for production.
- This document changes no ADR status and no existing contract. Where an implementation milestone needs a binding policy, it will be recorded in its own ADR. The Futures precedent is [ADR-018](../adr/ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md).

> **Note, 2026-11-05 (EI-3A governance, owner approval).** This note supersedes the status lines above, which describe the draft as of 2026-11-01. Details are in the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md).
> - The owner **APPROVED** EI0-D3 (LONG only) and EI0-D6 (interval-end UTC candle timestamps) on 2026-11-05. EI0-D4 stays approved (2026-11-01).
> - The owner also **APPROVED**, on 2026-11-05 and for synthetic-only EI-3A research, EI3-D1 (decision instant), EI3-D2 (evidence completeness), EI3-D3 (research outcome vocabulary), EI3-D4 (initial research strategy), EI3-D5 (synthetic calibration governance) and EI3-D8 (volume baseline). They are recorded only in that register.
> - Every other decision in this document keeps its status. In particular, EI0-D7 (the 14:25 / 14:30 / 15:00 IST schedule) and EI0-D8 (initial strategy and control) are **not** approved.
> - No strategy, parameter, threshold or cost rate is approved for production. The rest of this document remains a draft.

Statements about NSE market structure use three labels:

| Label | Meaning |
| --- | --- |
| **VERIFIED IN CODEBASE** | Confirmed by reading Northstar's current code or approved documents. It says nothing about NSE. |
| **AWAITING OFFICIAL VALIDATION** | A widely published NSE fact that has not been checked against an official NSE circular for the period in use. It must not be encoded as verified reference data until it is. |
| **POLICY PROPOSAL** | A Northstar product choice. It is not an exchange rule. |

---

## 1. Product Position

Northstar is a private, single-user decision-support platform. Its workflow is:

```text
Market data -> Research -> Candidate selection -> Risk validation
  -> Actionable recommendation -> Monitoring -> Historical evaluation -> Private presentation
```

Equity Intraday V1 applies that workflow to NSE cash equities within one trading session.

- **Recommendation-led, not execution-led.** V1 places no orders, connects to no broker for execution and shows no execution controls.
- **Private.** Recommendations are not distributed to anyone else. V1 is not a subscriber or advisory service.
- **Not an order.** A recommendation is a traceable trade setup. It is never an order, a fill, a position or a broker holding.
- **No profitability claim.** Engineering acceptance uses invented synthetic candles. Synthetic results show that the code is correct, never that the strategy has an edge.

---

## 2. Reference Inputs

### 2.1 Repository baseline at drafting

| Repository | Branch | HEAD |
| --- | --- | --- |
| northstar-core | develop | `3123a6a1` |
| northstar-application | develop | `1d851b9f` |
| northstar-infrastructure | develop | `74f5b0b0` (includes S-1A, PR #44) |
| northstar-api | develop | `7ad37f52` |
| northstar-web | develop | `5aeb94c4` |
| northstar-docs | develop | `3557a5fb` |

### 2.2 Missing reference

The optimized **Equity Intraday Master Roadmap** is not present in the development workspace.

- [Roadmap.md](../Roadmap.md) contains no equity-intraday material.
- This specification therefore takes its milestone sequence from the EI-0 instruction and reviews that sequence in [section 12](#12-milestone-dependencies).
- If the roadmap is added later, it must be reconciled with this document.

### 2.3 Governing documents

- [ADR-002 Architecture Constitution](../adr/ADR-002-Architecture-Constitution.md)
- [ADR-019 Symbol Ampersand Extension](../adr/ADR-019-Symbol-Ampersand-Extension.md)
- [Equity Intraday V1 EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md) (added 2026-11-05)
- [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md) (added 2026-11-05)
- [ADR-007 PointInTime Foundation Extension](../adr/ADR-007-PointInTime-Foundation-Extension.md)
- [ADR-017 Repository Layering and Futures Recommendation Domain Boundary](../adr/ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md) (Proposed)
- [ADR-018 Futures V1 Research-to-Recommendation Generation Policy](../adr/ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md) (Proposed)
- [Domain dependency rules](../domain/05-dependency-rules.md)
- [ES-003 Testing Standards](../engineering/standards/ES-003-Testing-Standards.md) and [ES-007 Documentation Standards](../engineering/standards/ES-007-Documentation-Standards.md)
- [Upstox Finality Evidence](../operations/Upstox-Finality-Evidence.md), which is the precedent for provider finality evidence

---

## 3. Current Contracts and Their Reuse

All facts in this section are **VERIFIED IN CODEBASE** at the baseline above.

### 3.1 Reused directly

| Contract | Location | Use in V1 |
| --- | --- | --- |
| `Symbol`, `ExchangeCode`, `Currency`, `Price`, `Quantity`, `PointInTime`, `Timeframe` | `northstar_core.foundation.value_objects` | Identity, INR prices, share quantities and UTC instants. `Timeframe("5m")` is already in the approved vocabulary. |
| `ListingReference` | Core domain | The canonical stock identity, `(Symbol, ExchangeCode("NSE"))`. |
| `Listing`, `ListingStatus`, `Tradability` | Core domain | `ListingStatus` already has `Created`, `Active`, `Suspended`, `Delisted` and `Retired`. These feed universe eligibility. |
| `HistoricalOHLCVBar` | `northstar_core.market_data` | The canonical candle: symbol, exchange, `point_in_time`, timeframe, OHLC, volume and an optional `adjusted_close`. It is timeframe-generic. |
| `HistoricalReplaySnapshot` | `northstar_core.market_data` | Point-in-time visibility. It already refuses bars after the replay instant and duplicate logical bars. |
| `HistoricalMarketDataSource` port and the ingest and replay use cases | `northstar_application` | Generic provider boundary, ingestion and cumulative replay. S-1A proved this chain with synthetic NSE/INR data. |
| `StrategyIdentity`, `ResearchHorizon` | `northstar_core.strategy` | Strategy naming and horizon labelling. |

### 3.2 Reused as patterns, not as code

Futures types are bound to `FuturesContract`, `FuturesOHLCVBar` and `FuturesProductReference`. They must not be renamed or subclassed for equities. ADR-017 keeps the Futures recommendation domain Futures-specific.

| Futures pattern | Location | Equity treatment |
| --- | --- | --- |
| `FuturesTradeRecommendation`: identity, entry zone, stop, target, validity, source decision reference, policy version, issued-at and thesis | `recommendations.futures.value_objects` | Model the same structure in a new `recommendations.equity` package. |
| Lifecycle events with `IntrabarOrdering` values `NOT_REQUIRED`, `MODELED_STOP_FIRST` and `UNKNOWN` | `recommendations.futures.lifecycle` | Reuse the evidence discipline. For equity exits, V1 proposes `UNKNOWN` rather than `MODELED_STOP_FIRST`; see [section 8.3](#83-intrabar-ambiguity). |
| `ObservationCoverage` and `CoverageDeficiency` on time expiry | Futures lifecycle | Reuse the idea that time expiry needs complete, verified coverage. |
| Generation rejection reasons, reward-to-risk and policy version (ADR-018) | `recommendations.futures.generation` | Use the same structure. Equity adds cost-aware validation and position sizing; see [section 9](#9-risk-model). |
| One open recommendation per product (ADR-018 P-1) | ADR-018 | Becomes one open recommendation per listing. |
| A hand-verified, versioned session reference that never uses the network or a calendar library | `nse_futures_session.py`, `nse_futures_calendar_reference.py` | Build a separate NSE cash-equity reference with its own evidence. The Futures calendar is not assumed to apply to equities. |

### 3.3 Not reusable as-is

- **`HistoricalSnapshotEvaluatorService`** only supports `1d` bars, requires 20 bars and uses raw closes. It stays unchanged. V1 needs a new intraday evaluator; it does not need a flag on this one.
- **The existing 5/20 close-average plus volume rule** is a research baseline. It is not an approved intraday strategy. On 5-minute bars it would mix sessions unless it is explicitly bounded to one session.
- **The research labels `BUY`, `SELL` and `HOLD`** are research vocabulary. A research `SELL` never becomes an equity SHORT and never becomes an overnight cash-equity short.
- **`SQLiteHistoricalMarketDataStore`** upserts on logical identity: a conflicting bar replaces the stored one with no correction audit. That is acceptable for research fixtures but not for recommendation evidence; see [section 6.6](#66-candle-finality-and-late-corrections).
- **The `/analyze` and `/watchlist/refresh` API surfaces and the web watchlist** are current-state analysis. They are not recommendation surfaces, and they may analyse an unfinished bar.
- **`Symbol`** uses `^[A-Z0-9._-]{1,32}$` and rejects `&`, so NSE symbols such as `M&M` cannot be represented at this baseline. [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md) approves extending it to `^[A-Z0-9._&-]{1,32}$`.

> **Note, 2026-11-05 (correction).** The `Symbol` bullet above describes the drafting baseline only and is obsolete. EI-1B implemented [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md): `Symbol` now accepts `^[A-Z0-9._&-]{1,32}$`, and `M&M` is a valid symbol. The reference implementation is RVO-001 `Symbol` v1.1 in the [Reference Implementation Registry](../engineering/registry/Reference-Implementations.md). No decision changes.

---

## 4. V1 Product Definition (Summary)

| Area | V1 definition | Status |
| --- | --- | --- |
| Market | NSE cash equities, INR | PROPOSED |
| Series | `EQ` only | PROPOSED |
| Direction | LONG only. Research SELL is rejected, never mapped to SHORT. | PROPOSED, and recommended |
| Interval | Completed 5-minute candles only | PROPOSED |
| Universe | A small, versioned, owner-configured list, e.g. no more than 10 stocks, filtered by explicit eligibility rules | PROPOSED |
| Strategy | Exactly one deterministic, versioned intraday strategy, selected in EI-3 | PENDING |
| Holding | Intraday only. No recommendation remains active past the session's planned exit cutoff. | PROPOSED |
| Entry cutoff | 14:30 IST | PROPOSED |
| Planned exit cutoff | 15:00 IST | PROPOSED |
| Orders | None. No automatic or manual broker execution from Northstar. | PROPOSED, and recommended |
| Engineering data | Invented synthetic candles | PROPOSED |
| Real data | Deferred to EI-9, behind a data-rights gate | PROPOSED |
| Evidence | Immutable recommendation, lifecycle events and evidence references; all-call performance | PROPOSED |

> **Note, 2026-11-05 (EI-3A governance, owner approval).** In the table above, **Direction** (EI0-D3) is now **APPROVED** (owner, 2026-11-05): LONG only, and a bearish signal is never converted into a short. **Strategy** remains PENDING (EI0-D8). For synthetic EI-3A research only, the owner approved the opening-range breakout with a session VWAP approximation and volume confirmation (EI3-D4). Every other row keeps its status. See the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md).

### 4.1 Challenges to the supplied proposals

1. **LONG-only is the right V1 scope.**
   - Intraday short selling of cash equities depends on broker product rules, intraday square-off and stock-borrowing arrangements. None of that exists in Northstar.
   - Excluding SHORT removes a whole class of error: an unsettled cash-equity short being carried overnight.
   - This document does not claim that intraday shorting is impossible under NSE rules. It is deferred as a product choice.
2. **Use one cutoff pair, not per-stock cutoffs.** Per-stock cutoffs would add configuration without evidence that they help.
3. **A 15:00 exit cutoff needs an explicit unresolved state.** A recommendation that is still active at 15:00 must not be closed at a price Northstar did not observe as an exit. See [section 8](#8-recommendation-and-lifecycle).
4. **Indicators must be bounded to one session, or explicitly span sessions.** A 20-bar average on 5-minute data that silently crosses the overnight gap mixes two auctions. Any cross-session input must be named and justified.
5. **Transaction cost and position sizing cannot wait until EI-8.** Small intraday targets can be smaller than round-trip costs. A recommendation whose modeled cost consumes most of its reward should never be issued. Both must be in EI-4; see [section 12.2](#122-prerequisites-moved-earlier).
6. **Candle-correction auditing must move into EI-2.** Recommendations cite candles as evidence. If a cited candle can be silently replaced, the evidence trail is broken.
7. **"Small liquid stock set" needs measurable rules.** Liquidity eligibility belongs in EI-3 candidate screening, not in owner judgement at run time.

---

## 5. Instrument and Universe

### 5.1 Canonical identity

- **Canonical stock identity** is `ListingReference(Symbol(<NSE trading symbol>), ExchangeCode("NSE"))`, with currency INR. This structure is VERIFIED IN CODEBASE.
- **Identity across exchanges.** The same company on BSE is a different listing and never shares recommendations, lifecycle or evidence with NSE. S-1A already tests this separation.
- **ISIN.** The ISIN is recorded as provider-mapping evidence, not as part of Core identity. This is PROPOSED. Whether an `Isin` value object belongs in Core is a decision for after V1.
- **Renamed or re-coded symbols.** A trading-symbol change creates a new listing reference. Continuity between the old and new symbol is an effective-dated mapping record, never an in-place rewrite. This is PROPOSED.

### 5.2 Symbol compatibility (ampersand)

- NSE trading symbols include `&`, for example `M&M`. That such symbols exist is AWAITING OFFICIAL VALIDATION against the NSE securities list for the period in use.
- `Symbol` is an approved Foundation value object, reference implementation RVO-001. Its pattern `^[A-Z0-9._-]{1,32}$` rejects `&`.
- **APPROVED (owner, EI0-D4, 2026-11-01):** [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md) extends the `Symbol` character set to `^[A-Z0-9._&-]{1,32}$`.
  - It is the smallest widening needed. Every existing valid symbol stays valid, and every other `Symbol` rule is unchanged.
  - `Symbol` stays exchange-neutral: listing and series eligibility remain separate concerns.
  - `UnderlyingReference` is not changed.
  - It is implemented in EI-1B. RVO-001 becomes v1.1 when the Core change is merged.
- **Not allowed instead:** provider-specific encodings such as `M_M` or `M%26M` must never become canonical Northstar symbols.

### 5.3 Series and eligibility

The NSE series vocabulary is AWAITING OFFICIAL VALIDATION. As commonly published:

- `EQ` is the normal rolling-settlement series.
- `BE` and `BZ` are trade-for-trade or surveillance series, in which intraday netting is not available.
- Other series exist.

PROPOSED V1 rules:

| Rule | Proposal |
| --- | --- |
| Series | `EQ` only. Any other series is ineligible: `SERIES_NOT_SUPPORTED`. |
| Listing status | `Active` only. `Suspended`, `Delisted` and `Retired` are ineligible: `LISTING_NOT_ACTIVE`. |
| Exchange surveillance measures | PENDING. Whether stocks under exchange surveillance frameworks are excluded, and how that evidence is sourced. Until a source exists, such status cannot be inferred and is not modelled. |
| Liquidity | Median daily traded value over a trailing window of completed sessions must be at least a configured threshold. The threshold is PENDING. Ineligible: `INSUFFICIENT_LIQUIDITY`. |
| Price floor | Last completed close must be at least a configured minimum. The minimum is PENDING. Ineligible: `PRICE_BELOW_MINIMUM`. |
| Data availability | The stock must have complete, final 5-minute data for the required look-back. Ineligible: `INCOMPLETE_INTRADAY_HISTORY`. Missing bars are never interpolated or carried forward. |
| Corporate-action day | Excluded on a day with a known corporate action that changes price continuity, such as an ex-date for a split, bonus or large dividend. Ineligible: `CORPORATE_ACTION_DAY`. This needs a corporate-action input; until one exists, the gap is reported (see [section 15](#15-material-risks)). PENDING. |

### 5.4 Suspended, delisted and unavailable securities

- **Before issuance:** an ineligible stock produces a recorded rejection reason, not a silent omission.
- **After issuance:** a listing suspended during the session, or a halt in data, ends the recommendation as `INCOMPLETE_EVIDENCE`, or `INVALIDATED` if suspension evidence exists. It is never resolved by assumption.
- **Unavailable data** produces no recommendation, with reason `INCOMPLETE_INTRADAY_HISTORY` or `SESSION_DATA_UNAVAILABLE`.

### 5.5 Source-specific instrument mapping

- Provider instrument keys, such as an Upstox instrument key or a vendor token, live only in Infrastructure.
- A mapping record holds:
  - the provider;
  - the provider key;
  - the `ListingReference`;
  - the series;
  - the ISIN;
  - the effective-from and effective-to dates;
  - an evidence reference.
- A provider key that maps to more than one listing, or to none, fails closed.
- Core and Application never see provider keys. This follows ADR-013's precedent for options.

### 5.6 Configurable universe

- A universe is a versioned list of listing references: `EquityUniverseVersion` (PROPOSED). Changing it creates a new version.
- **Proposed cap:** no more than 10 stocks. PENDING.
- Every recommendation records the universe version it was screened under.
- The universe is owner-configured, not discovered by a scanner. Market-wide screening is after V1.

---

## 6. Session, Timeframe and Candle Finality

### 6.1 Facts awaiting official validation

These must be confirmed against official NSE circulars for every reference year before they enter a reference module. Each entry needs an evidence row, following the [NSE Historical Futures Reference Evidence Register](../research/NSE-Historical-Futures-Reference-Evidence-Register.md) precedent.

| Item | As commonly published | Status |
| --- | --- | --- |
| Normal market (continuous trading), cash equity | 09:15 to 15:30 IST | AWAITING OFFICIAL VALIDATION |
| Pre-open call auction, cash equity | Starting 09:00 IST, ending before 09:15 IST | AWAITING OFFICIAL VALIDATION |
| Post-close or closing-price session | After 15:30 IST | AWAITING OFFICIAL VALIDATION |
| Trading holidays and weekends | Published annually by circular | AWAITING OFFICIAL VALIDATION, per year |
| Special sessions (e.g. Muhurat, Budget day, mock or DR sessions) | Notified case by case | AWAITING OFFICIAL VALIDATION, per event |
| Instrument-specific phases (e.g. call-auction-only securities, special pre-open for IPO listing or relisting days) | Exist for some securities and days | AWAITING OFFICIAL VALIDATION |
| UTC offset of IST | +05:30, no daylight saving | Handled through the `Asia/Kolkata` zone, as in `nse_futures_session.py` |

> **Note, 2026-11-05 (correction, CE-C-05).** The single "09:15 to 15:30 IST" continuous market in the table above is obsolete. The owner accepted the following facts on 2026-11-04, and they govern: see the [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md), section 18.3, for every source, boundary and limitation.
> - **Open.** The normal market opens at 09:15 IST from 2026-04-28 (CE-F-01).
> - **Before 2026-08-03.** The normal market closes at 15:30 IST for all normal-market securities, from 2026-04-28 to 2026-08-02 (CE-F-02).
> - **From 2026-08-03, CAS-applicable securities.** Continuous trading ends at 15:15 IST, and the Closing Auction Session runs separately from 15:15 to 15:35 (CE-F-10, CE-F-12, CE-F-16). The auction is not continuous trading.
> - **Non-CAS securities.** Continuous trading runs from 09:15 to 15:30 IST as the current rule from the page update on 2026-08-12 (CE-F-13). Whether that applies from 2026-08-03 to 2026-08-11 is CE-D15, which is **PENDING**; this note does not resolve it.
> - **Pre-open** is 09:00 to 09:15 IST, with timings that change from 2026-09-07 (CE-F-03, CE-F-04). It is not continuous trading.
> - **CAS eligibility.** Historical CAS membership is never inferred (CE-D14). V1 relies on the common boundary (CE-D6) and on the approved 09:15 to 15:00 IST research window (CE-D17), which is a Northstar policy and not the NSE trading session.

Broker intraday auto square-off times are broker product rules, not exchange facts. V1 does not use them.

### 6.2 Session model

- **Separate equity resolver.** A new `EquityTradingSessionResolver` Application port returns, for a listing and a date:
  - the session label, which is the IST civil date;
  - `continuous_opens_at` and `continuous_closes_at`, as UTC instants;
  - a session kind: normal or special.
  It is separate from `FuturesTradingSessionResolver`, which is typed to futures products.
- **Resolution order.** Infrastructure resolves sessions from a hand-verified, versioned NSE cash-equity reference module, using the same order as the Futures resolver:
  1. a year that has not been loaded raises;
  2. a published special session is a session;
  3. a weekend is not a session;
  4. a published holiday is not a session;
  5. otherwise, the normal hours of the regime in force.
- **Unknown timings fail closed.** A special session whose timings are not notified raises, and V1 does not trade it. Excluding special sessions entirely is PROPOSED.
- **Instrument-specific phases** are not modelled in V1. Any listing and day subject to one is ineligible (`SESSION_PHASE_NOT_SUPPORTED`). PROPOSED.

### 6.3 5-minute grid

- Bars are aligned to the continuous-trading open: `[open + 5k, open + 5(k+1))` minutes.
- Pre-open auction prints are not a 5-minute bar. Whether the opening price appears as the first bar's open is a provider-normalization question for EI-9.
- If the session length is not a whole number of 5-minute intervals, the trailing partial interval is labelled. It is never stretched or merged. PROPOSED.

### 6.4 Timestamp and timezone semantics

- **Storage.** All instants are UTC `PointInTime` values. IST is used only to resolve session dates and to display times.
- **Bar timestamps.** A bar's `point_in_time` is the interval's **end**, its completion instant. This is PROPOSED, and consistent with daily bars, where the point in time is the session close.
  - Example: the 09:15 to 09:20 IST bar is stamped 09:20 IST, which is 03:50Z.
  - Providers that stamp the interval start must be normalized by the adapter. This must be proven with evidence in EI-9.

> **Note, 2026-11-05 (EI-3A governance, owner approval).** EI0-D6 is **APPROVED** (owner, 2026-11-05): a five-minute candle is identified by its canonical UTC interval-end timestamp, UTC storage stays distinct from IST session interpretation, and a bar-start timestamp is never a canonical candle identity. The bullet above that calls this PROPOSED is superseded. See the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md).
- **No host time.** Nothing reads the host timezone or host clock for market semantics. "Now" is always a supplied instant.

### 6.5 Completed-bar eligibility

A bar may be used for a decision at instant `T` only if all of these hold:

1. its `point_in_time` is at or before `T`;
2. it belongs to the resolved session and the 5-minute grid;
3. it is **final** under the finality policy in [section 6.6](#66-candle-finality-and-late-corrections).

The bar still forming at `T` is never visible, not even partially. There is no tick-level, quote-level or "last traded price" input in V1.

### 6.6 Candle finality and late corrections

- **Finality.** A 5-minute bar is final when two conditions hold:
  - it is at least a configured settle delay past its interval end (PENDING; it must be justified by provider evidence);
  - the source has not marked it provisional.
  For synthetic data, finality is declared by the fixture.
- **Corrections.** Every correction to an accepted bar creates a new, immutable bar revision. It never overwrites.
  - Each revision records what it supersedes, its acquisition instant and its source.
  - Every earlier revision stays identifiable and retrievable, for audit.
  - Replay can be run as of a knowledge instant: it then uses only revisions acquired at or before that instant. This is how a past decision is reproduced exactly.
  - The Futures [Revised Candle Approval Procedure](../operations/Indian-Futures-Revised-Candle-Approval-Procedure.md) is the operational precedent.
- **Store change.** This requires an insert-only intraday bar store, or a revision table. The current generic store upserts, so EI-2 must not reuse it unchanged for recommendation evidence.

#### 6.6.1 Effect of a revision on recommendations

A revision is an evidence-integrity fact. It is not, by itself, a reason to invalidate anything.

1. **As-issued evidence is preserved.**
   - A recommendation cites the exact revision of every bar it used when it was issued.
   - Its research decision, thesis, levels, lifecycle events and outcomes are never rewritten.
2. **A source-revision event is appended.**
   - Each recommendation that cites the revised bar gets an immutable `SOURCE_REVISION_RECORDED` event, linking the superseded revision and the new one.
   - The same applies when the revised bar was used by one of the recommendation's lifecycle events.
   - The event is evidence-integrity history. It is not a lifecycle transition.
3. **Materiality is assessed deterministically.** The assessment replays the recommendation's own decision and lifecycle rules against the revised bars. Possible results:
   - `NOT_MATERIAL`: the same decision, levels and lifecycle outcome would follow.
   - `MATERIAL_TO_DECISION`: the decision would not have been issued, or its entry, stop or target would differ.
   - `MATERIAL_TO_LIFECYCLE`: the decision is unchanged, but an entry, stop, target or coverage determination would differ.
   - `UNASSESSED`: no approved materiality policy applies, or the assessment could not complete.
4. **Effect on a recommendation that is not yet terminal** (`RECORDED`, `WAITING_FOR_ENTRY`, `ACTIVE`):
   - Only `MATERIAL_TO_DECISION`, under an approved policy (EI0-D21), may move it to `INVALIDATED` with reason `SOURCE_EVIDENCE_MATERIALLY_REVISED`. The event cites the assessment.
   - `MATERIAL_TO_LIFECYCLE` does not rewrite earlier events. Monitoring continues on the revisions known at each later decision instant.
   - `UNASSESSED` fails closed: the recommendation is flagged as evidence under review and is no longer presented as actionable. This is a presentation flag derived from the event, not a lifecycle state, and the recommendation is not invalidated.
5. **Effect on a terminal recommendation.** It never changes state. The assessment is recorded, and any effect is shown only in a separately labelled corrected-data restatement ([section 10.5](#105-as-issued-and-restated-results)).
6. **Never automatic.** A revision does not invalidate every recommendation that cites the bar. It never creates a new recommendation or a new trading outcome.

**PENDING (owner, EI0-D21):** the materiality policy. Until it is approved, every assessment is `UNASSESSED`, and the fail-closed rules in this section and in [section 10.5](#105-as-issued-and-restated-results) apply.

### 6.7 Entry and exit cutoffs (POLICY PROPOSALS)

| Cutoff | Proposal | Meaning |
| --- | --- | --- |
| Signal cutoff | No decision after the bar ending at 14:25 IST | The last decision's earliest entry is still before the entry cutoff. |
| Entry cutoff | 14:30 IST | A recommendation still waiting for entry when the bar ending at 14:30 completes becomes `EXPIRED_BEFORE_ENTRY`. A touch inside a bar that ends after 14:30 does not activate it. |
| Planned exit cutoff | 15:00 IST | The intraday idea ends. An active recommendation's outcome is recorded as described below. No exit fill or exit price is recorded. |

The cutoffs govern four separate things. They must not be conflated.

1. **Recommendation eligibility.** After the signal cutoff, no new decision is made. After the entry cutoff, no entry may be recommended or activated.
2. **The requirement to stop following the idea.** At the planned exit cutoff, an active recommendation stops being eligible for continued intraday exposure. It is no longer presented as a live idea, and it is never carried into the next session.
3. **Exit evidence.** The cutoff alone does **not** show that a hypothetical position exited at any price. Exit evidence exists only when a completed, final bar up to and including the bar ending at 15:00 touched the stop or the target.
4. **The recorded lifecycle outcome**, evaluated on the bar ending at 15:00 and never on a later bar:
   - If that bar, or an earlier one, gives valid exit evidence, the ordinary rule records the evidenced outcome: `CLOSED_TARGET`, `CLOSED_STOP` or `INDETERMINATE_EXIT`.
   - If coverage through 15:00 is complete and verified and no exit was evidenced, the outcome is `SESSION_CUTOFF_REACHED`. This is an **unresolved** outcome: not a win, not a loss, and with no exit price.
   - If any required bar through 15:00 is missing, non-final, or has an unverified timestamp, the outcome is `INCOMPLETE_EVIDENCE`. Gaps and unverified timestamps fail closed. They never become `SESSION_CUTOFF_REACHED` and never resolve by assumption.

Bars after 15:00 are never used to resolve the recommendation.

- All three are policy parameters in the recommendation policy version. They are expressed as IST wall-clock times and converted per session.
- On a special or shortened session, the cutoffs do not apply automatically: V1 does not trade that session. PROPOSED.
- The cutoffs leave a buffer before the normal close, as commonly published. That buffer is a product choice, not an exchange requirement.

> **Note, 2026-11-05 (EI-3A governance).** EI0-D7, the schedule in this section, remains **PROPOSED** and is **not** approved.
> - Under EI3-D1 (approved 2026-11-05), a research decision is made at its evidence snapshot's `as_of` instant, which is strictly after the decision bar's interval end, because finality evidence must be received first. A decision on the bar ending 14:25 is therefore made after 14:25, and no complete bar then both opens after the decision and ends by the 14:30 entry cutoff.
> - How the signal cutoff relates to finality latency and to the entry cutoff is EI3-D6, which is **PENDING**. This section is not corrected until EI3-D6 and EI0-D7 are decided. See the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md), sections 5.1 and 6.1.

> **Note, 2026-11-06 (EI-3B, owner approval).** EI3-D6 is now **APPROVED** (owner, 2026-11-06) as research-only scheduling: research uses the approved window, the latest signal candle ends at 15:00 IST, and no entry-dependent cutoff is enforced. EI0-D7, this section's schedule, stays **PROPOSED** and is deferred to execution and risk design. See the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md#10-ei-3b-owner-approval-record-2026-11-06).

> **Note, 2026-11-07 (EI-4A, owner approval).** EI0-D7 is now **APPROVED for synthetic policy structure only**. The bar-end 14:25 / 14:30 / 15:00 IST interpretation of the table above is **not** approved as an operational schedule. The approved structure distinguishes the signal-bar end, decision availability, the latest permissible entry-bar opening, any minimum remaining session time, the mandatory modeled exit boundary, shortened or special sessions and post-session research, with versioned synthetic values. No production cutoff time is approved. See the [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md#3-approved-decisions).

---

## 7. Signal and Strategy

### 7.1 Inputs and outputs

**Inputs**, all point-in-time and frozen into the decision's evidence:

- The completed, final 5-minute bars of the current session up to the decision instant.
- A bounded, named set of prior-session references. Proposed: the previous session's high, low and close, and the trailing liquidity statistics.
- Each listing's eligibility facts and universe version.

**Outputs**, an immutable `EquityIntradayResearchDecision`:

- the listing;
- the decision instant;
- the strategy identity and configuration version;
- the research outcome: `LONG_CANDIDATE` or `NO_SIGNAL`. Bearish research is recorded as `NO_SIGNAL` with its signal values, never as SHORT;
- the signal values;
- the explanation reasons;
- references to the cited bars.

### 7.2 Point-in-time visibility

- Snapshots are built per decision instant using the existing replay discipline: no bar after the instant, and no duplicate logical bars.
- A test must show that appending later, divergent bars leaves earlier decisions unchanged. S-1A has this test for daily bars.
- Indicators are computed only from the frozen evidence. Nothing recomputes them later from a mutable store.

> **Note, 2026-11-05 (EI-3A governance, owner approval).** For synthetic EI-3A research, the owner approved (2026-11-05):
> - **EI3-D1:** the decision instant is the `as_of` instant of the immutable EI-2B evidence snapshot. Only evidence visible and valid at that instant contributes.
> - **EI3-D2:** a missing observation, an unfinalized candle, conflicting values or an unresolved source revision in a required interval rejects the evaluation. Nothing is interpolated, carried forward or chosen.
> - **EI3-D3:** the outcomes are `LONG_CANDIDATE`, `NO_SIGNAL` or an explicit rejection with a reason. `BUY`, `HOLD` and `SELL` are not reused as executable outcomes.
>
> The replay discipline in section 7.2 is met through EI-2B evidence snapshots, not the generic replay snapshot. See the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md).

### 7.3 Indicator configuration

- Every parameter belongs to the strategy configuration version: look-back lengths, thresholds, opening-range length and volume ratio.
- Changing any parameter creates a new version, following ADR-018 F4-D7.
- Core holds no defaults. The Application supplies a configuration, and parameters are validated at construction.
- Arithmetic is exact `Decimal`, with no binary floating point.

### 7.4 Initial strategy selection (PENDING)

V1 needs exactly one strategy. This document proposes a candidate and a control; it approves neither.

| Role | Definition | Status |
| --- | --- | --- |
| **Candidate strategy** | **Opening-range breakout with a session-VWAP filter (LONG)**. The opening range is the first `n` completed 5-minute bars, proposed `n = 3`. Signal: a completed bar closes above the opening-range high, that close is above the session VWAP computed from completed bars, and bar volume is at least `v` × the session's mean bar volume so far. | PENDING. This is a hypothesis for EI-3, not an approved edge. |
| **Research control** | The existing 5/20 close-average plus volume rule, bounded to bars of the current session. | PROPOSED, as a comparison baseline only |
| **Not adopted** | SMA 9/21 crossover (the Instagram-series example) | Not approved. It may be evaluated later as a research control. |

Why this candidate is proposed:

- it needs only the current session's OHLCV;
- it has a natural, explainable invalidation level (the opening-range low or the breakout bar's low);
- it has a natural time structure that suits the entry and exit cutoffs.

"Session VWAP" here means an approximation from 5-minute bars, the sum of typical price × volume over the sum of volume. It must be named as such, because it is not the exchange's traded VWAP.

> **Note, 2026-11-05 (EI-3A governance, owner approval).** For synthetic EI-3A research only, the owner approved the candidate strategy above, the opening-range breakout with a session VWAP approximation and volume confirmation (EI3-D4), with explicit, versioned, visibly synthetic calibration (EI3-D5). The volume comparison excludes the breakout candle from its own mean and never uses a future bar (EI3-D8).
> - EI0-D8 stays **PENDING**: the research control, real-data research and recommendation generation are not approved.
> - The proposed `n = 3` and any value of `v` are **not** approved parameter values.
> - Nothing here approves an entry, exit, stop, target, sizing, brokerage or slippage rule, or supports any claim of profitability or significance. See the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md).

### 7.5 Candidate screening and rejection reasons

Each universe stock, at each decision instant, gets exactly one recorded outcome: a candidate, or one or more rejection reasons. The proposed reasons are:

- **Eligibility:**
  - `SERIES_NOT_SUPPORTED`
  - `LISTING_NOT_ACTIVE`
  - `INSUFFICIENT_LIQUIDITY`
  - `PRICE_BELOW_MINIMUM`
  - `CORPORATE_ACTION_DAY`
  - `SESSION_PHASE_NOT_SUPPORTED`
- **Data:**
  - `INCOMPLETE_INTRADAY_HISTORY`
  - `SESSION_DATA_UNAVAILABLE`
  - `OPENING_RANGE_INCOMPLETE`
  - `NON_FINAL_BAR`
- **Signal:**
  - `NO_SIGNAL`
  - `BEARISH_RESEARCH_NOT_SUPPORTED`, the LONG-only rule
- **Timing:**
  - `AFTER_SIGNAL_CUTOFF`
  - `SESSION_NOT_SUPPORTED`
- **Open recommendations:**
  - `DUPLICATE_OPEN_RECOMMENDATION`
  - `MAX_SIMULTANEOUS_RECOMMENDATIONS`
  - `DAILY_LOSS_LIMIT_REACHED`
- **Risk:** see [section 9.3](#93-risk-validation-rejection-reasons).

A rejection is evidence and is stored with the same point-in-time discipline as a candidate.

---

## 8. Recommendation and Lifecycle

### 8.1 Minimum recommendation contract

The proposed contract is `EquityTradeRecommendation`, in a new `northstar_core.recommendations.equity` package. It is immutable once recorded.

| Field | Meaning |
| --- | --- |
| `identity` | Caller-supplied and deterministic. It is derived by the Application from listing, session, strategy version and decision instant, as in Futures F-5. Core generates no identifiers. |
| `listing` | `ListingReference`, exchange NSE |
| `currency` | INR. All price fields must share it. |
| `strategy_identity`, `strategy_configuration_version` | The research that produced it |
| `policy_version` | The recommendation and risk policy version: cutoffs, risk parameters, cost schedule version and universe version |
| `source_decision` | Reference to the `EquityIntradayResearchDecision` |
| `issued_at` | The decision instant, which is the completion instant of the last cited bar |
| `direction` | `LONG` only in V1 |
| `rationale` | Ordered explanation reasons and signal values |
| `entry` | An entry zone `[low, high]`, aligned to the price increment, or a trigger level. One representation must be chosen in EI-4: PENDING. |
| `stop` | Stop-loss level, below the entry |
| `target` | Target level, above the entry |
| `validity` | `eligible_from`, `entry_cutoff`, `exit_cutoff`, all UTC instants of one session |
| `risk` | `risk_per_share`, which is worst entry minus stop; `risk_amount` in INR; reward-to-risk measured gross and net of modeled costs |
| `suggested_quantity` | A whole number of shares from the sizing rule. It is a suggestion, not an order. |
| `model_capital_reference` | The explicit model-capital input used for sizing |
| `cost_estimate` | The modeled round-trip cost and slippage for the suggested quantity, with the cost schedule version |
| `evidence` | References to the exact revision of each cited bar as known at issuance, plus the eligibility facts and the universe version |

- A recommendation never carries an order identifier, fill price, fill quantity, broker account or P&L.
- **Price increment.** The NSE price increment for each stock is AWAITING OFFICIAL VALIDATION, and it is supplied as a policy input, as in ADR-018. V1 should support a single configured increment per universe version, and fail closed if a stock's increment is unknown. PENDING.

> **Note, 2026-11-05 (EI-3A governance).** The `issued_at` row above says the decision instant is "the completion instant of the last cited bar". For **research decisions**, that is superseded by EI3-D1 (approved 2026-11-05): the decision instant is the `as_of` instant of the EI-2B evidence snapshot, and a bar's interval end never by itself makes a candle final or a decision available. For **recommendations**, `issued_at` and `eligible_from` (section 8.2) are **not** decided by EI3-D1. They remain open for EI-4 and EI-5, to be settled consistently with EI3-D1, EI3-D6 and EI0-D7. See the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md), section 6.1.

> **Note, 2026-11-07 (EI-4A, owner approval).** EI4-D1 now decides the recommendation instants: `decision_as_of` (the EI-3 research decision instant), `issued_at` (actual issuance of an operational recommendation) and `eligible_from` are distinct, and entry evidence comes only from bars that open at or after the information-availability and issuance instant. Where section 8.2 would let the bar completing at `eligible_from` evidence an entry although it opened before the decision was available, EI4-D1 governs. EI0-D14 is **APPROVED for the initial synthetic model**: the `entry` row above is a modeled zone anchored to the signal close, built from versioned parameters and aligned to a supplied price increment. See the [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md#3-approved-decisions).

### 8.2 Lifecycle states

Lifecycle is derived only from completed, final bars at or after `eligible_from`, using the bar revisions known at each event's decision instant. Each transition is an immutable event that cites its evidence.

"Terminal" means monitoring has ended. It does not mean the outcome is resolved. Only `CLOSED_TARGET`, `CLOSED_STOP` and `EXPIRED_BEFORE_ENTRY` are resolved outcomes. The other terminal states record why no evidenced result exists.

| State | Terminal | Entered when |
| --- | --- | --- |
| `RECORDED` | No | The recommendation is persisted. `eligible_from` is the completion instant of the next bar. |
| `WAITING_FOR_ENTRY` | No | `eligible_from` is reached. |
| `ACTIVE` | No | A completed bar's range touches the entry, and does not touch stop or target in the same bar. |
| `CLOSED_TARGET` | Yes | While active, a completed bar touches the target only. |
| `CLOSED_STOP` | Yes | While active, a completed bar touches the stop only. |
| `EXPIRED_BEFORE_ENTRY` | Yes | The entry cutoff passes with no activation, and coverage is complete and verified. |
| `INDETERMINATE_ENTRY` | Yes | One bar touches the entry and the stop and/or target. The order inside the bar is `UNKNOWN`. |
| `INDETERMINATE_EXIT` | Yes | While active, one bar touches both stop and target. The order is `UNKNOWN`; see [section 8.3](#83-intrabar-ambiguity). |
| `SESSION_CUTOFF_REACHED` | Yes, **unresolved** | Still active at the planned exit cutoff, with complete and verified coverage and no evidenced exit. It is not a win or a loss. **No exit price is recorded.** See [section 6.7](#67-entry-and-exit-cutoffs-policy-proposals). |
| `INCOMPLETE_EVIDENCE` | Yes, unresolved | A required bar is missing, not final or has an unverified timestamp before resolution, or the session's data is unavailable. Coverage deficiencies are listed, as in the Futures `CoverageDeficiency`. |
| `INVALIDATED` | Yes, unresolved | Before it became terminal, either: there is evidence of a listing suspension or ineligibility after issuance; or an approved materiality assessment found a source revision `MATERIAL_TO_DECISION` (`SOURCE_EVIDENCE_MATERIALLY_REVISED`, [section 6.6.1](#661-effect-of-a-revision-on-recommendations)). A candle revision alone never causes it. The reason and its evidence are recorded. |

Rules:

- **No overnight carry-over.**
  - Every recommendation is resolved only from bars of its issuing session, up to the planned exit cutoff.
  - If that session's data cannot be assessed, the outcome is `INCOMPLETE_EVIDENCE`.
  - No recommendation is ever active on, or resolved from, the next session.
- **No look-ahead.** A transition at bar `B` uses only bars up to and including `B`.
- **No gap filling.** A missing bar is never interpolated, assumed flat or skipped over. Resolution stops and becomes `INCOMPLETE_EVIDENCE`.
- **Event order.** Events are insert-only, and the lifecycle is rebuilt from them deterministically. The same inputs always give the same states.
- **Later reconciliation is appended, never rewritten.** Evidence that arrives after a terminal state never changes that state or its events. Examples are a late bar that fills a gap, or a candle revision. It is recorded as an explicit, audited event that cites the original terminal event, and any effect appears only in a labelled restatement ([section 10.5](#105-as-issued-and-restated-results)).
- **Touch semantics.** "Touch" means the bar's low is at or below the level and its high is at or above it, compared exactly. Whether a gap-through counts as a touch at the level is not decided here, because V1 records no exit fill. It matters only for the modeled outcomes in [section 10](#10-backtesting-and-performance).

### 8.3 Intrabar ambiguity

- When a single 5-minute bar contains both stop and target, or entry and an exit, the order of those events is unknowable from OHLC data.
- V1 records `UNKNOWN` ordering and an indeterminate terminal state. This is PROPOSED.
- This differs on purpose from the Futures `MODELED_STOP_FIRST` lifecycle rule. A modeled ordering belongs in performance modeling, as a labelled assumption, not in the factual lifecycle record.
- Performance reports a range for indeterminate outcomes; see [section 10.3](#103-indeterminate-and-no-entry-outcomes).
- **PENDING (owner):** keep this rule, or align with Futures stop-first.

### 8.4 Session-end handling

- `SESSION_CUTOFF_REACHED` records one fact: the setup was active, coverage was complete and verified, and neither stop nor target was touched before the planned exit time.
- It is an **unresolved** outcome, not a completed exit.
  - It has no exit price and no fill.
  - It is never classified as a win or a loss.
  - It does not show that a position could have exited at the cutoff bar's close or at any other price.
- The cutoff ends the idea's eligibility for continued exposure ([section 6.7](#67-entry-and-exit-cutoffs-policy-proposals)). It does not create exit evidence.
- If a stop or target touch through the bar ending at 15:00 exists, the evidenced outcome is recorded instead. If coverage is incomplete or unverified, the outcome is `INCOMPLETE_EVIDENCE`.

---

## 9. Risk Model

Recommendation-level risk is a property of a hypothetical setup sized against a **model capital** input. It is not broker account risk, portfolio risk or margin. V1 reads no broker balance, position or margin.

### 9.1 Inputs (all PENDING numeric values)

| Parameter | Proposed default (for review only) |
| --- | --- |
| Model capital (INR) | Explicit owner input. No default. |
| Risk per recommendation | 0.5% of model capital |
| Maximum exposure per recommendation (entry notional) | 20% of model capital, with no leverage |
| Maximum simultaneous open recommendations | 3 |
| Minimum stop distance | At least a fraction of the entry price, **and** at least a multiple of the modeled round-trip cost per share |
| Maximum stop distance | At most a fraction of the entry price |
| Minimum net reward-to-risk | 1.5, after modeled costs |
| Daily modeled loss limit | No new recommendations in a session once the closed recommendations' modeled net loss reaches 2R |
| Cost schedule | A versioned schedule. See [section 9.4](#94-transaction-costs). |
| Slippage assumption | A fixed number of basis points per side, versioned |

### 9.2 Sizing rule

```text
worst_entry          = entry.high                     (LONG)
risk_per_share       = worst_entry - stop
cost_per_share       = modeled round-trip cost and slippage per share at worst_entry
quantity_by_risk     = floor(risk_budget / (risk_per_share + cost_per_share))
quantity_by_exposure = floor(max_exposure / worst_entry)
suggested_quantity   = min(quantity_by_risk, quantity_by_exposure)
```

- If `suggested_quantity` is less than 1, the setup is rejected with `QUANTITY_BELOW_ONE_SHARE`.
- Quantities are whole shares. No fractional shares are used.

### 9.3 Risk-validation rejection reasons

- `INVALID_PRICE_ORDER`
- `DEGENERATE_ENTRY_ZONE`
- `STOP_TOO_TIGHT`
- `STOP_TOO_WIDE`
- `COST_EXCEEDS_REWARD_THRESHOLD`
- `NET_REWARD_TO_RISK_BELOW_MINIMUM`
- `QUANTITY_BELOW_ONE_SHARE`
- `EXPOSURE_LIMIT`
- `PRICE_INCREMENT_UNKNOWN`
- `INSUFFICIENT_SESSION_TIME`: the validity window does not fit before the entry cutoff.

### 9.4 Transaction costs

- Indian cash-equity intraday costs include several components: brokerage, securities transaction tax, exchange transaction charges, SEBI fees, stamp duty and GST.
- Their rates and bases are **AWAITING OFFICIAL VALIDATION**, and they change over time. This document states no rate.
- V1 needs a versioned `EquityCostSchedule`. It is effective-dated, its rates are owner-supplied and each rate cites evidence. Every estimate records the schedule version it used.
- Costs used for generation are *estimates for validation*. Costs used in evaluation are *modeled costs*. Neither is a broker contract note.

### 9.5 Duplicates and contradictions

- Only one open recommendation per listing is allowed (the ADR-018 P-1 pattern, applied per listing). A same-listing candidate while one is open is rejected `DUPLICATE_OPEN_RECOMMENDATION`.
- With LONG-only and one strategy, a contradicting recommendation cannot arise. No unreachable code is added for it.
- State that already breaks these rules fails closed and is reported. It is never repaired automatically.

### 9.6 Daily loss limit

- The limit applies to **modeled recommendation outcomes**, not to an account.
- When the limit is reached:
  - generation stops for the rest of the session (`DAILY_LOSS_LIMIT_REACHED`);
  - recommendations already open continue to be monitored.
- Indeterminate outcomes count at their worst modeled value for this purpose. That choice is conservative, and PENDING.

---

## 10. Backtesting and Performance

### 10.1 Separation of concepts

| Concept | Meaning | V1 |
| --- | --- | --- |
| Lifecycle outcome | Factual state from completed bars | Yes |
| Gross theoretical outcome | R-multiple from the entry, stop and target levels of resolved outcomes, before costs. No cutoff exit price is used. | Yes, labelled theoretical |
| Net modeled outcome | Gross outcome minus modeled costs and slippage for the suggested quantity | Yes, labelled modeled |
| Paper order or fill | A simulated order that has its own fill model | **Not in V1** |
| Broker fill or realized P&L | A real execution | **Not in V1** |
| Corrected-data restatement | Re-evaluation of the same as-issued levels against later bar revisions | Yes, reported separately ([section 10.5](#105-as-issued-and-restated-results)) |

### 10.2 Reporting rules

- **All-call reporting.** Every recommendation issued is reported, including those that never entered, and every rejection is counted. Nothing is filtered after the fact.
- **Sample counts.** Every statistic is shown with its sample count. When nothing was measured, a statistic is `None`, never zero. This follows the convention of `CalculateHistoricalResearchMetricsUseCase`.
- **Statistics, gross and net:**
  - win rate and loss rate over resolved outcomes;
  - average win and average loss in R;
  - expectancy in R;
  - profit factor, reported only when the gross loss is greater than 0;
  - maximum drawdown of the sequential cumulative net R, ordered by resolution instant.

### 10.3 Indeterminate and no-entry outcomes

- **No entry.** `EXPIRED_BEFORE_ENTRY` is reported as its own category, with 0 R exposure. It is not counted as a win or a loss.
- **Indeterminate.**
  - `INDETERMINATE_ENTRY` and `INDETERMINATE_EXIT` are reported separately.
  - Statistics are shown in two versions: one with each indeterminate case scored at its worst bound, and one at its best bound.
  - If the two versions differ materially, the conclusion is marked inconclusive.
- **Unresolved evidence.** `INCOMPLETE_EVIDENCE` and `INVALIDATED` are excluded from statistics, but always counted and listed.
- **Session-end cutoff.** `SESSION_CUTOFF_REACHED` is reported as its own unresolved category.
  - It is always counted and listed.
  - It is excluded from win rate, loss rate, average win and loss, expectancy, profit factor and drawdown.
  - No exit price is assumed for it, not even as a modeled bound.
  - A high share of cutoff outcomes is itself reported as a limitation of the strategy's evidence.

### 10.4 Robustness requirements before any real-data conclusion

- **Independent periods.** Use at least two independent, non-overlapping historical periods. Their dates are PENDING and depend on authorized data.
- **In-sample versus out-of-sample.**
  - Parameters are fixed on the in-sample period and frozen by configuration version before the out-of-sample period is evaluated.
  - The out-of-sample period is evaluated once per frozen version.
- **Parameter sensitivity.**
  - Use a small, pre-declared grid around the chosen parameters.
  - A result that holds only at one parameter point is treated as overfit.
- **Overfitting safeguards.** Record every configuration version that was ever evaluated. Report the number of versions tried alongside the chosen one.
- **Minimum sample.** A minimum number of resolved recommendations is required before any statistic is presented as meaningful. The number is PENDING.
- **No profitability claim** is made from synthetic data, from in-sample results alone, or from samples below the minimum.

### 10.5 As-issued and restated results

- **As-issued results are the primary record.**
  - They use the lifecycle outcomes recorded from the bar revisions known at each event's decision instant.
  - Later revisions never alter them.
- **Corrected-data restatement is separate and labelled.**
  - It re-evaluates the same recommendation levels against later revisions.
  - It is reported in a separate view, with the revisions and assessments it relies on.
  - It is never merged into as-issued statistics, and it creates no new recommendation, lifecycle event or trading outcome.
- **Unassessed revisions fail closed for performance claims.** While any recommendation in a reporting period has an `UNASSESSED` source revision ([section 6.6.1](#661-effect-of-a-revision-on-recommendations)):
  - that period's statistics are marked provisional;
  - no robustness conclusion from [section 10.4](#104-robustness-requirements-before-any-real-data-conclusion) may be drawn from them.
- **Terminal recommendations stay reconstructable.** Every terminal recommendation can be reconstructed exactly as issued: its decision, its events and the bar revisions they cited.

---

## 11. Private Dashboard (V1)

The dashboard is read-only and private. Its proposed screens are:

| Screen | Content |
| --- | --- |
| Universe | The universe version, each stock's eligibility for the current or a selected session, and its rejection reasons |
| Candidates | Per decision instant: candidate or rejection, signal values and explanation |
| Recommendations | Entry, stop, target, validity times in IST, risk amount, suggested quantity, cost estimate and policy version |
| Lifecycle | The event timeline with the cited bars, showing an `UNKNOWN` ordering and coverage deficiencies explicitly |
| History and evidence | Past recommendations, the bars they cite with revisions, and their configuration versions |
| Performance | All-call statistics: gross and net, best and worst indeterminate bounds, sample counts, and permanent limitation notices (synthetic data, modeled costs, not fills) |

Not in V1:

- broker or execution controls;
- order buttons;
- a no-code strategy builder;
- an AI report assistant;
- market-wide scanning;
- notifications to anyone else.

The Futures dashboard is the integration pattern: an API read model over persisted state, and web cards. Its components are not reused for equities until direct reuse is shown to be safe.

---

## 12. Milestone Dependencies

### 12.1 Refined sequence

| Milestone | Scope | Depends on |
| --- | --- | --- |
| **EI-0** | This specification and its owner decision register | — |
| **EI-1** | NSE equity identity and session foundations: equity session port and 5-minute grid; `Symbol` `&` (EI-1B, [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md)); the NSE cash-equity calendar reference with an evidence register; series and listing eligibility facts; the `EquityUniverseVersion` | EI-0 decisions D1, D2, D4, D5 |
| **EI-2** | Synthetic intraday OHLCV ingestion, replay and finality, **including immutable, insert-only bar revisions** (moved earlier). Acceptance: earlier revisions stay retrievable, and replay as of a knowledge instant reproduces the revisions known then. | EI-1 |
| **EI-3** | One intraday strategy, research control, explainable candidate screening, **liquidity and price eligibility** (moved earlier) | EI-2, D8 |
| **EI-4** | Risk-validated recommendation generation, **including cost schedule, cost-aware validation and position sizing** (moved earlier), and an equity generation-policy ADR | EI-3, D10 to D14 |
| **EI-5** | Intraday lifecycle and session-end handling. Acceptance covers:<br>• evidenced exits through the 15:00 bar take precedence;<br>• `SESSION_CUTOFF_REACHED` is unresolved, with no exit price;<br>• gaps and unverified timestamps give `INCOMPLETE_EVIDENCE`;<br>• no next-session resolution;<br>• `SOURCE_REVISION_RECORDED` events and materiality assessment, which stays `UNASSESSED` until D21 is approved. | EI-4, D9 (D21 for any revision-driven `INVALIDATED`) |
| **EI-6** | Durable insert-only persistence for recommendations, rejections, lifecycle events, source-revision events, materiality assessments and reconciliation records; deterministic monitoring runs | EI-5 |
| **EI-7** | Private read-only API and web | EI-6 |
| **EI-8** | Trade-level historical evaluation: net modeled outcomes, indeterminate bounds, unresolved cutoff outcomes kept out of win and loss statistics, as-issued results separate from corrected-data restatements, in-sample and out-of-sample results, sensitivity | EI-6. It reuses the EI-4 cost schedule rather than introducing one. |
| **EI-9** | Authorized real NSE equity intraday data, the provider adapter and timestamp-convention evidence | Data-rights gate D16, EI-2 |
| **EI-10** | Forward observation and operational acceptance | EI-7, EI-8, EI-9 |

### 12.2 Prerequisites moved earlier

1. **Transaction cost and position sizing: into EI-4, not EI-8.**
   - Without them, generation can issue setups whose reward is consumed by costs, and quantities that exceed the exposure limit or fall below one share.
   - EI-4 needs the cost *schedule* and validation. EI-8 adds trade-level *evaluation* on top of the same schedule.
2. **Candle revision audit: into EI-2.** Recommendation evidence depends on immutable bars, and the generic store upserts.
3. **Liquidity eligibility: into EI-3.** It is a screening rule, not a risk rule. It needs only the trailing data that EI-2 provides.
4. **The data-rights inquiry runs in parallel from EI-0.**
   - It is a non-engineering owner task, so the EI-9 gate does not become the critical path.
   - Engineering never depends on its outcome before EI-9.
5. **A corporate-action input is needed before EI-9.**
   - With synthetic data, V1 can exclude corporate-action days through a fixture input.
   - With real data, an authoritative corporate-action source is needed. Otherwise prior-session references and liquidity windows can be corrupted.

### 12.3 After V1

These follow V1, in no committed order:

- advanced explainable screening;
- additional strategy families;
- historical chart replay;
- AI sentiment and financial research;
- market-regime analysis;
- parameter optimisation, with the overfitting safeguards of [section 10.4](#104-robustness-requirements-before-any-real-data-conclusion);
- a visual strategy builder.

Equity SHORT, and broker execution, remain deferred.

---

## 13. Data Rights and Provider Independence

- **Provider independence.** Core and Application contracts name no provider. Upstox, Zerodha and any vendor are Infrastructure adapters only.
- **Broker and data provider are separate.** The trading broker and the market-data provider need not be the same, and no contract assumes they are.
- **API availability is not authorization.** Being able to call an API does not establish the right to use its data in any of the ways below.

Before EI-9, the owner must hold written evidence, such as provider terms, an agreement or a written confirmation, for each of these uses:

| Use | Evidence required |
| --- | --- |
| Permitted API use | Personal, non-commercial use of intraday historical and live candles is allowed for this account. |
| Historical storage | Local storage of candles, and how long they may be kept, is permitted. |
| Backtesting and simulated outcomes | Use in research, backtests and modeled outcomes is permitted. |
| Private backups | Backup copies, including off-machine copies, are permitted. |
| Derived research | Derived indicators, decisions and recommendations may be stored and kept after the raw data must be deleted. |
| Retention | Retention limits and the obligations when access ends |
| Data integrity | Documented candle timestamp convention (start or end), finality and correction behaviour, completeness guarantees, and pre-open handling |

Evidence is recorded in a register following the [Upstox Finality Evidence](../operations/Upstox-Finality-Evidence.md) precedent. Northstar does not contact providers as part of EI-0.

---

## 14. Owner Decision Register

Only EI0-D4 is approved. Every other decision remains PROPOSED or PENDING.

> **Note, 2026-11-05 (EI-3A governance, owner approval).** The sentence above describes the register as of 2026-11-01. EI0-D3 and EI0-D6 are now **APPROVED** (owner, 2026-11-05), as shown in the table. Decisions EI3-D1 to EI3-D8 are recorded only in the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md): EI3-D1 to EI3-D5 and EI3-D8 are approved, and EI3-D6 and EI3-D7 are **PENDING**. None of them is listed in the table below. Every other status below is unchanged.

> **Note, 2026-11-06 (EI-3B, owner approval).** EI3-D6, EI3-D7 and EI3-D9 are now **APPROVED** (owner, 2026-11-06), and EI3-D10 is recorded as **PENDING**, all in the [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md#10-ei-3b-owner-approval-record-2026-11-06). The statuses in the table below are unchanged; EI0-D7 stays **PROPOSED**.

> **Note, 2026-11-07 (EI-4A, owner approval).** In the [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md#3-approved-decisions), the owner **APPROVED** EI0-D7 (revised timing structure, for synthetic policy structure only) and EI0-D14 (entry zone, for the initial synthetic model), together with EI4-D1 to EI4-D4 and EI4-D6 to EI4-D8. The EI0-D7 and EI0-D14 rows below describe their earlier status. Every other status below is unchanged: in particular EI0-D10 to EI0-D13, approving real values, stay **PENDING**, and no production cutoff time is approved.

| ID | Decision | Proposal | Status |
| --- | --- | --- | --- |
| EI0-D1 | Market and currency | NSE cash equities, INR | PROPOSED |
| EI0-D2 | Series | `EQ` only | PROPOSED |
| EI0-D3 | Direction | LONG only. Research SELL is rejected, never mapped to SHORT. | APPROVED (owner, 2026-11-05, [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md)) |
| EI0-D4 | `Symbol` `&` support | Extend the `Symbol` pattern to `^[A-Z0-9._&-]{1,32}$` through a new Foundation ADR ([ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md)); `UnderlyingReference` unchanged | APPROVED (owner, 2026-11-01) |
| EI0-D5 | Universe | Versioned, owner-configured, no more than 10 stocks | PROPOSED |
| EI0-D6 | Bar timestamp convention | Interval end | APPROVED (owner, 2026-11-05, [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md)) |
| EI0-D7 | Cutoffs | Signal 14:25, entry 14:30, planned exit 15:00 IST. No special sessions in V1. At the planned exit, an evidenced exit is recorded; otherwise `SESSION_CUTOFF_REACHED` (unresolved, no exit price) or `INCOMPLETE_EVIDENCE` ([section 6.7](#67-entry-and-exit-cutoffs-policy-proposals)). | PROPOSED |
| EI0-D8 | Initial strategy | Opening-range breakout plus a VWAP-approximation filter, with the bounded 5/20 rule as control | PENDING (selection and test in EI-3) |
| EI0-D9 | Intrabar ambiguity | `UNKNOWN` and indeterminate in the lifecycle. Modeled ordering only in evaluation bounds. | PENDING |
| EI0-D10 | Model capital | Explicit owner input | PENDING (value) |
| EI0-D11 | Risk parameters | [Section 9.1](#91-inputs-all-pending-numeric-values) defaults | PENDING (values) |
| EI0-D12 | Cost schedule | Versioned, owner-supplied, evidence-cited rates | PENDING (rates) |
| EI0-D13 | Price increment | Configured per universe version. Fail closed if unknown. | PENDING |
| EI0-D14 | Entry representation | Entry zone (Futures pattern) or trigger level | PENDING |
| EI0-D15 | Liquidity and price thresholds | Configurable minimums | PENDING (values) |
| EI0-D16 | Data-rights gate | The [section 13](#13-data-rights-and-provider-independence) evidence before EI-9 | PROPOSED |
| EI0-D17 | Corporate-action handling | Exclude corporate-action days. An authoritative source before EI-9. | PENDING |
| EI0-D18 | Exchange surveillance exclusion | Exclude, if an evidence source is available | PENDING |
| EI0-D19 | Daily modeled loss limit | 2R per session, with indeterminate outcomes counted at their worst bound | PENDING |
| EI0-D20 | Robustness minimums | Minimum sample and independent periods | PENDING (values) |
| EI0-D21 | Candle-revision materiality policy | A deterministic assessment ([section 6.6.1](#661-effect-of-a-revision-on-recommendations)). Only `MATERIAL_TO_DECISION` on a recommendation that is not yet terminal may lead to `INVALIDATED`. Until approved, assessments are `UNASSESSED` and fail closed. | PENDING |

---

## 15. Material Risks

| Risk | Consequence | Mitigation |
| --- | --- | --- |
| Futures models are renamed for equities | Hidden futures assumptions (contract cutoff, expiry, SHORT mapping) leak into equities | A separate `recommendations.equity` package. Futures is used as a pattern only (ADR-017). |
| The generic store's upsert is used for evidence | Cited candles change silently, and evidence is falsified | Insert-only revisions in EI-2 |
| Bars stamped at interval start are mis-normalized | Off-by-one-bar look-ahead | End-stamp convention, adapter evidence in EI-9, and look-ahead tests in EI-2 and EI-3 |
| Unverified NSE facts are encoded as reference data | Wrong sessions on exactly the exceptional days | An evidence register per year. Unloaded years and un-notified special sessions fail closed. |
| Cross-session indicators | Overnight gaps distort signals | Session-bounded indicators by default. Named cross-session inputs only. |
| Corporate actions are absent | Prior-session references and liquidity windows are corrupted on ex-dates | Exclude those days. An authoritative source before EI-9. |
| Costs or sizing appear late | Uneconomic setups are issued | Moved into EI-4 |
| Intrabar ordering is invented | Overstated or understated outcomes | Lifecycle uses `UNKNOWN`. Evaluation shows best and worst bounds. |
| The `Symbol` change ripples into Futures, Options and provider parsing | Regressions in existing products | [ADR-019](../adr/ADR-019-Symbol-Ampersand-Extension.md), plus regression checks of every `Symbol` consumer in EI-1B |
| Overfitting through parameter search | A false edge | Pre-declared grids, frozen versions, out-of-sample once, and a count of versions tried |
| Data rights assumed from API access | Unauthorized storage or derived use | The D16 gate before EI-9 |
| A universal trading framework creeps in | Coupling across products | Product-specific packages. Only truly generic components are shared. |

---

## 16. First EI-1 Implementation Slice (Proposed, Not Started)

**EI-1A: Equity trading-session port and completed 5-minute interval grid.**

This slice was chosen first because it needs no pending owner decision and asserts no NSE date.

1. **Application.** Add an `EquityTradingSessionResolver` port and an `EquityTradingSession` value:
   - fields: listing, session date, kind, `continuous_opens_at`, `continuous_closes_at`;
   - errors: an explicit resolution error for unloaded years and unknown special-session timings.
2. **Application or Core.** Add a pure function that enumerates the completed 5-minute interval ends of a session up to an instant.
   - The bar still forming is excluded.
   - A trailing partial interval is labelled.
3. **Infrastructure.** Add a resolver skeleton that reads a versioned reference module, with the same resolution order as `nse_futures_session.py`.
   - In this slice it is exercised **only with an explicitly synthetic reference fixture**.
   - No real NSE holiday, special session or hours are loaded until the evidence register entries exist.
4. **Tests.** Cover:
   - IST to UTC conversion through `Asia/Kolkata`;
   - the unloaded year failing closed;
   - weekend and holiday exclusion from the fixture;
   - special session precedence;
   - grid alignment;
   - exclusion of the bar still forming at every instant;
   - no host timezone or clock dependence;
   - no network.

Out of scope for EI-1A:

- `Symbol` `&` (EI-1B, after D4);
- real NSE calendar data (EI-1C, after evidence);
- strategy, generation, persistence, API and web.

---

## 17. Related Documents

- [Equity Intraday V1 EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md)
- [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md)
- [ADR-017](../adr/ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md), [ADR-018](../adr/ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md)
- [Futures Trade Recommendation Design Specification](../architecture/Futures-Trade-Recommendation-Design-Specification-v1.0.md)
- [Futures Recommendation Generation Design Specification](../architecture/Futures-Recommendation-Generation-Design-Specification-v1.0.md)
- [Listing Status Design Specification](../architecture/Listing-Status-Design-Specification-v1.0.md)
- [PointInTime Design Specification](../architecture/PointInTime-Design-Specification-v1.0.md)
- [Indian Futures Revised Candle Approval Procedure](../operations/Indian-Futures-Revised-Candle-Approval-Procedure.md)
- [Upstox Finality Evidence](../operations/Upstox-Finality-Evidence.md)
- [NSE Historical Futures Reference Evidence Register](../research/NSE-Historical-Futures-Reference-Evidence-Register.md)
