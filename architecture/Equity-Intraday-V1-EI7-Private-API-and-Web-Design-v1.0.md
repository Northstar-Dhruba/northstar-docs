# EI-7 — Equity Intraday V1 Private API and Web Design v1.0

## Status

**DESIGN DRAFT — APPROVED ARCHITECTURAL DIRECTIONS, CONTRACTS PENDING.** Version 1.0, prepared 2026-11-16 (EI-7B Docs PR-0).

| Item | Status |
| --- | --- |
| EI-7 architectural directions R-1 to R-12 | **APPROVED — 2026-11-15** ([EI-7 Decision Register, section 4](../product/Equity-Intraday-V1-EI7-Decision-Register.md#4-approved-decisions)) |
| Endpoints, response views, errors, limits and routes in this design | **PROPOSED — PENDING OWNER APPROVAL** (EI7-CF1 to EI7-CF7; [register section 8](../product/Equity-Intraday-V1-EI7-Decision-Register.md#8-contract-finalization-decisions-pending)) |
| Acceptance scenarios EI7-N01 to EI7-N24 ([section 16](#16-acceptance-scenarios)) | **PLANNED — NOT TESTED**; EI7-N22 conditional |
| Engineering slices EI-7C to EI-7H ([section 18](#18-milestone-sequencing)) | **NOT AUTHORIZED** |
| Accepted EI-2B to EI-6C contracts | Unchanged |
| Operational or production authorization | **NOT GRANTED** |

- **Governance.** The decisions are recorded in the [EI-7 Decision Register](../product/Equity-Intraday-V1-EI7-Decision-Register.md). This design adds no decision.
- **Binding level.** Only R-1 to R-12 are binding. Every endpoint, field, error code, status, limit and route below is a proposal until its EI7-CF decision is approved. Python names are proposals; an implementation PR may rename one by stating the reason.
- **Not implemented.** No query use case, read-only adapter, endpoint, Web page or operator command described here exists.

Code was read at:

| Repository | Branch | Commit |
| --- | --- | --- |
| northstar-core | `develop` | `29bd34a` |
| northstar-application | `develop` | `617b538` |
| northstar-infrastructure | `develop` | `9d9c960` |
| northstar-api | `develop` | `7ad37f5` |
| northstar-web | `develop` | `5aeb94c` |
| northstar-docs | `develop` | `ac3a804` |

## 1. Scope

**In scope (R-1, R-9, R-10, R-12):** read-only inspection, by the single owner, of persisted synthetic research timelines, generation records, MODELED issuances, lifecycle histories and evidence snapshots, through GET endpoints and a Web interface.

**Out of scope:** any HTTP-triggered research, generation, declaration, issuance or monitoring; the operator CLI (separately governed, EI7-CF7); scheduling; real or provider data; ACTUAL issuance; orders, fills or broker integration; EI-8 metrics; Core changes; schema changes; changes to the EI-2B, EI-4D, EI-6A, EI-6B or EI-6C contracts.

## 2. Existing implementation inventory

### 2.1 Backend read surface

| Port (Application) | Method | Returns | Notes |
| --- | --- | --- | --- |
| `EquityRecommendationLifecycleReader` | `load_lifecycle(issuance_id)` | `EquityRecordedLifecycle` or `None` | Issuance re-derived from its disposition and lifecycle replayed by Core, in one read transaction. `None` means unknown issuance; a known issuance without events is an empty `RECORDED` lifecycle |
| `EquityRecommendationIssuanceReader` | `load_issuance(issuance_id)` | `EquityRecordedIssuance` or `None` | `recommendation` plus `recording_sequence` |
| | `load_session_issuance(strategy, listing, trading_date)` | `EquityRecordedIssuance` or `None` | At most one per strategy, listing and date |
| | `session_view(exchange_code, trading_date)`; `session_view_at(…, watermark)` | `EquitySessionRecommendationHistory` or `None` | Only issuances with `issued_at <= complete_through`; `None` when undeclared |
| `EquityRecommendationGenerationRecordReader` | `load_disposition(generation_key)`; `load_attempt(attempt_identity)` | Stored records or `None` | By key only |
| `EquityIntradayResearchTimelineStore` | `load(strategy, listing, trading_date)` | `StoredEquityIntradayResearchTimeline` or `None` | The only research read; explanation prose is not stored and is re-derived on rehydration |
| `EquityCandleEvidenceSnapshotStore` | `get_snapshot(snapshot_id)` | `EquityCandleEvidenceSnapshot` or `None` | |
| `EquitySyntheticPolicyCatalogStore` | `load_definition`, `load_policy_set` | Bindings or `None` | |

No port lists listings, a universe, issuances across dates, or research and generation records by session.

### 2.2 API

- FastAPI 0.141.1 on Python 3.13; app factory `create_app(settings, *, runtime, runtime_factory)`; composition root `runtime.py`.
- Five endpoints: `POST /analyze`, `POST /watchlist/refresh` (legacy current-state analysis, not recommendation surfaces), `GET /health`, `GET /futures/dashboard`, `GET /futures/analysis`. None concerns equity recommendations.
- `build_database_runtime` initializes the Futures schema at startup; `/health` probes Futures economics; Futures settings are all-or-nothing.
- No in-app authentication; GET-only CORS for one origin; `uvicorn` bound to `0.0.0.0:8000`; interactive documentation on.
- Tests drive the ASGI app directly (no `httpx`); `test_india8e_indian_deployment.py` pins the Caddy allow-list to the OpenAPI paths.

### 2.3 Web

- React 19.2.8, Vite 8.2.0, TypeScript 6.0.3 (`strict` off), Vitest with Testing Library. No router, data or component library.
- Views: `type View = "equity" | "futures"`, switched by `#futures`. The "Equity" view is the legacy US single-asset analyzer; the Futures view is read-only and tested GET-only.
- Reusable: the `src/futures/api.ts` client pattern, the `LoadState` union with abort on refresh, `section` cards with `dl` facts and `th scope="col"` tables, exact decimal strings, status-plus-reason rendering, fetch-stub tests with forbidden-word checks.

### 2.4 Absent today

An equity composition root; any Application read model; a bounded issuance list; read-only issuance, lifecycle, generation, research and snapshot readers; equity HTTP routes; Web routing and equity pages; synthetic and MODELED labels; a non-test synthetic listing-eligibility source; an operator command.

## 3. Layering and dependency rules

```text
Browser ──HTTP GET──▶ northstar-api   (composition root; thin routers; API response models)
                          │ calls
                          ▼
                     northstar-application   (query use cases → immutable views)
                          │ depends on reader ports only
                          ▼
                     reader ports ◀── implemented by ── northstar-infrastructure read-only adapters
                                                              │ mode=ro, query_only
                                                              ▼
                                                        Equity SQLite file
```

Rules (R-2, R-4):

1. HTTP controllers call Application query use cases only; they never construct or call an Infrastructure adapter, a store or SQL.
2. Query use cases depend only on reader ports. No writer, recorder, issuer, monitor, store write method or clock.
3. New Application read code names `EquityRecommendationLifecycleReader`, never `EquityRecommendationLifecycleStore`; the Application architecture test `test_ar_only_the_recording_use_case_depends_on_the_store_or_issues_authorizations` already enforces this.
4. Generation records are read through `EquityRecommendationGenerationRecordReader`; the write-capable store is never handed to a read path (EI4-D11; register section 6.2).
5. The composition root is in northstar-api. Infrastructure architecture tests forbid constructing the issuance and lifecycle adapters inside Infrastructure.
6. Core is unchanged.

## 4. Trust boundaries

```text
Owner's browser ──SSH tunnel──▶ host loopback port ──▶ Caddy (basic auth; GET/HEAD allow-list)
                                                          │ internal network, no published API port
                                                          ▼
                                                     equity API (read-only routes)
                                                          │ read-only connections
                                                          ▼
                                                     Equity SQLite file (single host)
```

- **Privacy is a deployment condition, not in-app authentication** (R-5). The API protects nothing when run outside this boundary.
- The trust model of ADR-024 section 14 is unchanged: personal-use synthetic research; one host; protection for supported operational paths only; no protection against privileged local access, direct SQL or file edits.
- No provider or broker secret reaches the equity API.

## 5. Source-of-truth mapping

| Fact | Authority | Read | Notes |
| --- | --- | --- | --- |
| Issuance identity, kind, strategy, listing, date | EI-6A issuance authority | `load_lifecycle` (issuance part) | `issuance_kind` is always MODELED |
| Levels (entry zone, stop, target), direction | Issued recommendation, re-derived from the EI-4D disposition | Same | Exact decimals |
| Risk and reward per share; worst entry | Core `EquityRecommendationLevels` properties | Derived | Not stored separately |
| Validity and timing instants | `EquityIssuedRecommendation` | Same | `issued_at` here is **modeled** |
| Sizing and costs | `EquityTradeRecommendationProposal.definition.sizing` | `load_disposition`, decoded | Synthetic model capital, not advice |
| Policy, eligibility and universe versions | Proposal and disposition columns | Same | |
| Research reference | Disposition columns; `EquityResearchCandidateReference` | Same | |
| Research decisions and roles | EI-3C timeline | `load(strategy, listing, date)` | Only persisted timelines |
| Generation status | EI-4D disposition | `load_disposition(key)` | `PROPOSED`, `NOT_ELIGIBLE`, `UNRESOLVED`; otherwise UNKNOWN |
| Lifecycle state, outcome class, events | EI-6B lifecycle, replayed by Core | `load_lifecycle` | `evaluated_at` is **modeled** |
| Cited evidence | EI-2B snapshot | `get_snapshot` | Snapshot `issued_at` is **caller-declared** |
| Declaration state | EI-6A declarations | Session views | Declared completeness, trusted within the ADR-024 trust model |
| Fills, executions, P&L, win rates | — | — | **Do not exist; must not be shown** (R-9) |
| Time an event was recorded; time monitoring ran | — | — | **Not stored; must not be shown or invented** (R-11) |

## 6. Query architecture

### 6.1 Proposed query use cases

All in northstar-application (EI-7C); names proposed:

| Use case | Dependencies | Returns |
| --- | --- | --- |
| `GetEquityRecommendationDetailUseCase` | `EquityRecommendationLifecycleReader`, `EquityRecommendationGenerationRecordReader`; optionally `EquityIntradayResearchTimelineStore` (read only) for the research-origin panel | `EquityRecommendationDetailView` |
| `ListEquitySessionRecommendationsUseCase` | `EquityRecommendationIssuanceReader`, `EquityRecommendationLifecycleReader` | `EquitySessionRecommendationsView` |
| `ListEquityRecordedSessionIssuancesUseCase` | The new bounded port of section 9, `EquityRecommendationLifecycleReader` | `EquityRecordedSessionIssuancesView` |
| `GetEquityResearchTimelineUseCase` | `EquityIntradayResearchTimelineStore` (read only), `EquityRecommendationGenerationRecordReader`, `EquitySyntheticPolicyCatalogStore` (read only), `EquityRecommendationIssuanceReader` | `EquityResearchTimelineView` |
| `GetEquityEvidenceSnapshotUseCase` | `EquityCandleEvidenceSnapshotStore` (read only) | `EquityEvidenceSnapshotView` |

Views are frozen dataclasses of domain values. Serialization to JSON belongs to the API.

### 6.2 Verified recommendation detail

R-6, with the invariants proposed in EI7-CF4:

1. `load_lifecycle(issuance_id)`. `None` gives `ISSUANCE_NOT_FOUND`.
2. `load_disposition(issuance.generation_key)`. A missing disposition, or one that is not `PROPOSED`, gives `PERSISTED_AUTHORITY_INVALID`.
3. Decode the stored proposal (`decode_equity_trade_recommendation_proposal`) and assessment (`decode_equity_recommendation_assessment`, which verifies its fingerprint); their fingerprints must equal the issuance's.
4. `equity_issued_recommendation(disposition)` must reproduce the issued recommendation exactly, identity included.
5. The disposition's policy-set version and fingerprint, research reference, strategy, listing, date and decision instant must agree with the proposal and the issuance.
6. Build the view. Supporting panels (research origin, evidence summaries) degrade per panel; the required facts never do.

No generation is replayed.

## 7. SQLite read-only design

- **Today.** The issuance and lifecycle adapters open `mode=rw` and run their schema initializer, `CREATE … IF NOT EXISTS` statements and a commit, on every call. The EI-2B and EI-3C adapters open the plain path, which can create a file, and initialize their schema too. Only `SQLiteEquityCandleEvidenceSnapshotCatalog` opens `mode=ro` and confirms `PRAGMA query_only = ON`.
- **Proposed (R-4).** Additive read-only adapters implementing the existing reader ports and the new bounded port:
  - open `mode=ro` (a missing file is never created) and confirm `query_only`;
  - never execute DDL or initialize a schema; missing tables are a typed unavailability;
  - reuse the existing verified decoding unchanged, as the EI-6C catalog reused EI-2B's `_stored_snapshot`, so that verification is not weakened (the whole-ledger consistency check and the issuance re-derivation stay);
  - one short read transaction per call.
- **Unchanged.** The existing adapters stay the write authorities, for writers only.
- **Inherited limits.** A `mode=ro` reader cannot recover a hot rollback journal (EI-6C L8). Under the rollback journal a reader's shared lock delays a writer's commit, up to the five-second busy timeout; a concurrent operator write may then refuse with a typed, retryable error.
- **Startup.** The equity runtime opens nothing at construction and initializes nothing; `/equity/health` performs one read-only check.

## 8. API composition root and Equity and Futures isolation

- **Composition root.** An equity runtime in northstar-api, alongside but separate from the Futures `DatabaseRuntime`; it constructs read-only adapters and query use cases only.
- **Separate files (R-7).** The Equity file holds the EI-2B evidence, EI-3C timelines, the policy catalog, EI-4D records and the EI-6A and EI-6B tables, linked by foreign keys. The Futures file is configured separately. No query, join or transaction spans them.
- **Settings.** A separate Equity database setting (a name such as `NORTHSTAR_EQUITY_DATABASE` is a proposal, EI7-CF5). Equity routes answer `DEPENDENCY_UNAVAILABLE` (proposed 503) when it is absent. Futures settings never become required for equity, nor equity settings for Futures.
- **No cross-initialization.** The Futures `initialize_database` never runs on the Equity file; no equity code runs on the Futures file.
- **Health.** `/health` and its Futures probe are unchanged; `/equity/health` is separate.
- **Architecture scans in northstar-api.** The equity composition constructs no writer, recorder, issuer, monitor or write-capable store, as EI4-D11 requires of the first repository with such wiring.
- **No migration.** No Equity database is deployed today, so nothing moves; R-7 governs future placement.

## 9. Bounded discovery

| View | Source | Shows | Proposed contract (EI7-CF1) |
| --- | --- | --- | --- |
| Declared | Existing `session_view` / `session_view_at` | Issuances covered by the latest declaration (or the declaration at a watermark) | Limit 50; `(issued_at, issuance_id)` order; `declaration` state; `recording_watermark` |
| Recorded | New bounded read | Every recorded issuance of the session, covered or not | Limit 50; same order; per-item `covered_by_declaration`; `recording_watermark`; optional `watermark` |

- **Why both.** Issuance at `t` requires a declaration through `t − 1 µs`, and `session_view` lists only issuances with `issued_at <= complete_through`. A new issuance is therefore absent from the declared view until a later declaration covers it (EI7-N07).
- **Proposed port.** An additive, read-only protocol, separate from `EquityRecommendationIssuanceReader` so that no existing implementation or test double must change; for example `EquityRecordedSessionIssuanceReader.recorded_session_issuances(exchange_code, trading_date, limit)`, returning the issuances with their recording sequences, the authority's watermark and the declaration state. It is served by the existing index `equity_recommendation_issuances_by_session (exchange_code, trading_date)`, with the same verification as `session_view`.
- **Not a list-all.** One exchange and one date; no range; no strategy-wide or global listing. EI6C-P7 still governs EI-6C monitoring discovery, and the monitor is not given this port.
- **Overflow.** A typed refusal, never truncation (proposed `DISCOVERY_LIMIT_EXCEEDED`).

## 10. Proposed endpoints

All GET, read-only and idempotent. None invokes research, generation, declaration, issuance or monitoring.

| Endpoint | Purpose | Source | Class (EI7-CF2) |
| --- | --- | --- | --- |
| `GET /equity/health` | Equity read readiness, independent of Futures | One read-only check | Essential |
| `GET /equity/sessions/{exchange}/{date}/recommendations` | Declared session view | `ListEquitySessionRecommendationsUseCase` | Essential |
| `GET /equity/sessions/{exchange}/{date}/recorded-issuances` | Bounded recorded view | `ListEquityRecordedSessionIssuancesUseCase` | Essential |
| `GET /equity/recommendations/{issuance_id}` | Verified detail with the complete lifecycle timeline | `GetEquityRecommendationDetailUseCase` | Essential |
| `GET /equity/research/{strategy}/{exchange}/{symbol}/{date}` | Persisted research timeline and generation status | `GetEquityResearchTimelineUseCase` | Essential for EI7-N01 to N04 |
| `GET /equity/recommendations/lookup` | One issuance by strategy, listing and date | `load_session_issuance` | Optional; the recorded view covers it |
| `GET /equity/recommendations/{issuance_id}/lifecycle` | Timeline alone | `load_lifecycle` | Optional; deferred |
| `GET /equity/snapshots/{snapshot_id}` | Evidence snapshot summary | `GetEquityEvidenceSnapshotUseCase` | Optional inspection surface |

**A separate lifecycle endpoint is not warranted in V1.** The detail must call `load_lifecycle` anyway, because that is the read that returns the authoritative issuance; it receives the complete, verified timeline in the same transaction. A lifecycle holds a small number of events (one per bar-driven transition plus any recorded source revisions), so there is no size reason to split it. A separate endpoint would duplicate the read and invite inconsistent pairings.

## 11. Proposed response views

Common to every view: owned by northstar-application; exact decimal strings for prices, quantities and amounts; canonical UTC instants; identities and enum names verbatim; `null` only for "not applicable" or "not stored"; unknown or unavailable reported as an explicit status object; no secrets, file paths or SQLite text; labels `evidence_class: "SYNTHETIC"` and, where an issuance is present, `issuance_kind: "MODELED"` and `actionable: false`.

### 11.1 `EquityRecommendationDetailView`

| Attribute | Proposal |
| --- | --- |
| Source | `load_lifecycle`; `load_disposition`; decoding of proposal and assessment (section 6.2) |
| Required | **Identity:** `issuance_id`, `generation_key`, `proposal_fingerprint`, `assessment_fingerprint`, `issuance_kind`, `recording_sequence`, authority reference. **Subject:** `strategy`, `listing`, `trading_date`, `direction`. **Timing:** `decision_as_of`, `issued_at` (modeled), `eligible_from`, `latest_entry_bar_opens_at`, `modeled_exit_at`, `timing_policy_version`. **Levels:** entry zone low and high, stop, target, risk and reward per share, worst entry. **Sizing:** currency, quantity, entry notional, entry cost, stop-exit and target-exit costs, gross and net risk and reward, risk and exposure budgets, price increment. **Versions:** policy-set version and fingerprint; generation, risk, cost, increment and timing versions; listing-eligibility and universe versions; research configuration version; evaluator version. **Research reference:** decision instant, research decision fingerprint, candidate entry fingerprint, evidence snapshot ID. **Checks:** `not_assessed`. **Lifecycle:** section 11.2 |
| Optional | Research-origin panel (the timeline entry matched by `candidate_entry_fingerprint`, with its signal values); each with a panel status |
| Identity and lineage | The invariants of EI7-CF4; any failure fails the whole response |
| Ordering | Not applicable (one record) |
| Errors and unknown | `ISSUANCE_NOT_FOUND`; `PERSISTED_AUTHORITY_INVALID`; `LIFECYCLE_HISTORY_INVALID`; `DEPENDENCY_UNAVAILABLE`; panels `available`, `unavailable` or `invalid` |
| Never present | Fill or execution prices, P&L, recorded-at or monitoring times |
| Consistency | Issuance and lifecycle in one transaction; the disposition from a separate connection, guarded by the lineage check |
| Tests before implementation | Contract tests for every invariant and error; serialization of exact decimals; a view built from each lifecycle state |

### 11.2 `EquityRecommendationLifecycleView` (embedded in the detail)

| Attribute | Proposal |
| --- | --- |
| Source | `EquityRecordedLifecycle.lifecycle` |
| Required | `state`, `is_terminal`, `outcome_class` (for terminal states), `position` (event count), `events` |
| Each event | `sequence`, `kind`, `from_state`, `to_state`, `evaluated_at` (modeled), evidence (`snapshot_id`, interval `opens_at` and `closes_at`), `touched`, intrabar `ordering`, `reason`, `deficiencies`, `cited_sequence`, `superseded_evidence`, `materiality` |
| Ordering | By `sequence`, ascending; never by time |
| Unknown | An open state means only that no later event is recorded; monitoring runs are not stored, so the time of the last check is unknown |
| Never present | A recording time; a fill; an exit price for an unresolved outcome |
| Tests | One fixture per state and event kind; `UNKNOWN` ordering shown; `INCOMPLETE_EVIDENCE` deficiencies listed |

**Terminal versus temporary.** Open states are `RECORDED`, `WAITING_FOR_ENTRY` and `ACTIVE`. Terminal states carry an outcome class: `RESOLVED` (`CLOSED_TARGET`, `CLOSED_STOP`), `RESOLVED_NO_ENTRY` (`EXPIRED_BEFORE_ENTRY`, `CANCELLED_BEFORE_ENTRY`), `INDETERMINATE` (`INDETERMINATE_ENTRY`, `INDETERMINATE_EXIT`), `UNRESOLVED` (`SESSION_CUTOFF_REACHED`, `INCOMPLETE_EVIDENCE`). An `UNRESOLVED` outcome is final but has no exit price. "Blocked" and "awaiting evidence" are advance statuses, never stored, and cannot be shown as states.

### 11.3 `EquitySessionRecommendationsView` (declared)

| Attribute | Proposal |
| --- | --- |
| Source | `session_view` or `session_view_at`; per-item `load_lifecycle` |
| Required | `exchange_code`, `trading_date`, `declaration` (`NONE`, or `DECLARED` with `complete_through`); when declared: `recording_watermark` and `items` |
| Each item | `issuance_id`, `strategy`, `listing`, `trading_date`, `issuance_kind`, `issued_at` (modeled), `recording_sequence`; lifecycle summary (`state`, `is_terminal`, `outcome_class`) or an item-level status |
| Ordering | `(issued_at, issuance_id)` ascending |
| Unknown | `declaration: NONE` is not "no recommendations" |
| Errors | `DISCOVERY_LIMIT_EXCEEDED`; `WATERMARK_AHEAD`; `UNSUPPORTED_SCOPE`; `PERSISTED_AUTHORITY_INVALID`; `DEPENDENCY_UNAVAILABLE` |
| Consistency | The list is one transaction; each item's lifecycle is a later, separate read and may have more events |
| Tests | Undeclared session; declared session; watermark replay; overflow; per-item corruption isolation |

### 11.4 `EquityRecordedSessionIssuancesView` (recorded)

As section 11.3, plus a per-item `covered_by_declaration`, and `recording_watermark` always present. Source: the bounded port of section 9. Label: completeness is certified only through the declaration, if any.

### 11.5 `EquityResearchTimelineView`

| Attribute | Proposal |
| --- | --- |
| Source | Timeline `load`; for generation status, the derived key and `load_disposition` (EI7-CF4); for issuance, `load_session_issuance` |
| Required | `strategy`, `listing`, `trading_date`; `timeline`: `STORED` with entries, or `UNKNOWN` when none is stored |
| Each entry | `sequence`, `role` (`CANDIDATE`, `REPEAT_QUALIFYING`, `NO_SIGNAL`, `REJECTED`), decision outcome, no-signal or rejection reason, `decision_instant` (the snapshot's `as_of`), `snapshot_id`, `manifest_fingerprint`, `watermark`, research-window close, decision and entry fingerprints |
| Generation status | `PROPOSED`, `NOT_ELIGIBLE` (with reasons) or `UNRESOLVED` from a stored disposition; otherwise `UNKNOWN` |
| Issuance | The issuance ID if one is recorded; otherwise absent, never "not issued" unless the authority says so |
| Ordering | By entry `sequence` |
| Unknown | Refused and duplicate research outcomes are not stored and are never shown (R-12) |
| Tests | EI7-N01 to N04, N24 |

### 11.6 `EquityEvidenceSnapshotView`

| Attribute | Proposal |
| --- | --- |
| Source | `get_snapshot` |
| Required | `snapshot_id`, `manifest_fingerprint`, `listing`, `trading_date`, `timeframe`, `window_policy_version`, window opens and closes, `evidence_class`, `as_of`, `issued_at` (labelled caller-declared), `watermark`, coverage summary, finality policy version, venue and applicability references |
| Optional | Per-interval bars (OHLCV as exact decimals) and finality outcome |
| Refusal | A non-SYNTHETIC class is refused (`EVIDENCE_CLASS_UNSUPPORTED`) and never relabelled |
| Never present | Raw journal records |
| Tests | EI7-N10, N14 |

### 11.7 `EquityReadHealthView`

`status` (`ok` or `unavailable`) and `configured`; one read-only check that the Equity file opens with `mode=ro` and the required tables exist. No Futures dependency, no write, no file path in the response.

## 12. Errors

**Levels.** A failure of required authority fails the **whole response** closed. A failure of a supporting panel is reported **on the panel**. A field is `null` only when the contract says it is not applicable or not stored, never because a read failed.

| Code (proposed) | Trigger | Level | HTTP (proposed) | Retryable |
| --- | --- | --- | --- | --- |
| `ISSUANCE_NOT_FOUND` | A reader returns `None` for the issuance | Response | 404 | No |
| `EVIDENCE_NOT_FOUND` | `get_snapshot` returns `None` | Response (snapshot endpoint) or panel | 404 or panel | No |
| `UNSUPPORTED_SCOPE` | Invalid exchange, date, symbol, strategy or identity | Response | 422 | No |
| `WATERMARK_AHEAD` | `EquityIssuanceWatermarkBeyondAuthorityError` | Response | 409 | No |
| `DISCOVERY_LIMIT_EXCEEDED` | More issuances than the discovery limit | Response | 422 (open) | No |
| `EVIDENCE_CLASS_UNSUPPORTED` | A non-SYNTHETIC snapshot | Response or panel | 422 (open) | No |
| `PERSISTED_AUTHORITY_INVALID` | `InvalidStoredEquityRecommendationIssuanceAuthorityError`, or a failed lineage invariant | Response | 500 | No |
| `LIFECYCLE_HISTORY_INVALID` | `InvalidStoredEquityRecommendationLifecycleError` | Response (detail); item (lists) | 500 | No |
| `DEPENDENCY_UNAVAILABLE` | Any `…UnavailableError`; unconfigured equity runtime | Response or panel | 503 | Yes |
| Undeclared session | `session_view` returns `None` | State, not an error | 200 with `declaration: NONE` | — |

Corruption is never converted into an empty success. Exact statuses and payloads are EI7-CF2.

## 13. Read consistency

- **Detail.** The issuance and lifecycle are one transaction. The disposition is a separate read of insert-only rows, protected by the lineage check.
- **Lists.** A list is one transaction; each item's lifecycle summary is a later read and may show more events. Issuances are insert-only and lifecycles append-only, so a later read only ever extends an earlier one.
- **Repeatability.** The recorded view accepts a `watermark` to reproduce an earlier list exactly. Lifecycles have no watermark; `position` lets a client detect change.
- **Caching.** None beyond the client's current page. No view across endpoints or connections is atomic (EI-6C L7; EI7-N21).

## 14. Web experience

### 14.1 Navigation

Extend the existing hash handling (R-8): `#/equity`, `#/equity/sessions/NSE/{date}`, `#/equity/recommendations/{issuance_id}` (URI-encoded) and `#/equity/research/{strategy}/NSE/{symbol}/{date}`; `#futures` unchanged. The existing `hashchange` listener gives refresh and back and forward support. No router library is needed.

### 14.2 Session selection

The owner chooses a date; there is no implicit "today". The declared and recorded views appear together, each with its declaration line.

### 14.3 Detail layout

Header: listing, trading date, strategy, SYNTHETIC and MODELED badges, current state with outcome class. Panels: levels and validity; synthetic sizing and costs ("synthetic model capital; not advice"); provenance; rationale (research values, checks and `not_assessed` items); lifecycle timeline; evidence.

### 14.4 Lifecycle presentation

An ordered list by sequence. Each event: transition, modeled time, cited snapshot (linked), evidence interval, deficiencies, `UNKNOWN` ordering shown explicitly, materiality `UNASSESSED`. Terminal states show their outcome class; an open state shows "No later event recorded. Monitoring runs are not recorded."

### 14.5 Provenance display

Issuance ID, generation key, proposal and assessment fingerprints, policy-set and component versions, research reference and evidence snapshot, each copyable in full.

### 14.6 Labels and vocabulary

- A persistent banner: "Synthetic evidence · MODELED issuance · Not a trade, order or fill."
- `ACTIVE` reads "Entry activated (modeled)"; terminal states read "Closed at target (modeled)" and so on.
- Forbidden on equity pages, and tested: "live", "executed", "filled", "fill price", "profit", "P&L", "win rate", "return".
- Candidate, proposal, issued recommendation, lifecycle state and trade are distinct labels (EI7-N24).

### 14.7 Time display

IST first (Asia/Kolkata, UTC+05:30) and UTC visible as secondary text, not in a tooltip only; `<time datetime>` carries the UTC instant (R-11).

### 14.8 Error and empty states

One message per error code; a Retry button only for retryable errors; "Not declared" and "Unknown" never shown as zero; the legacy asset-analysis view labelled as such, not as recommendations.

## 15. Security constraints

- GET only for every equity route; no form, button or control that writes.
- Deployment only behind the R-5 boundary; the prohibitions of EI7-CF3 apply.
- No secrets, file paths, tracebacks or SQLite text in responses or logs reachable by the client.
- SYNTHETIC evidence only, refused otherwise (R-10).
- Request validation before any read; identities validated by their existing shape checks.

## 16. Acceptance scenarios

Layers: **AU** Application unit; **PC** port and contract; **IS** Infrastructure SQLite; **AS** API ASGI; **WC** Web component; **AR** architecture and dependency; **XP** cross-process and restart; **SJ** full synthetic journey.

| ID | Scenario | Expected behaviour (proposed) | Layers | Slices | Status |
| --- | --- | --- | --- | --- | --- |
| EI7-N01 | Persisted `NO_SIGNAL` | The stored entry is shown as no signal, with its reason; never as a rejection, candidate or recommendation | AU, IS, AS, WC | C, D, E, F | **PLANNED — NOT TESTED** |
| EI7-N02 | Persisted `REJECTED` | The rejection and its reason are shown, distinct from no signal | AU, IS, AS, WC | C, D, E, F | **PLANNED — NOT TESTED** |
| EI7-N03 | `NOT_ELIGIBLE` versus `UNRESOLVED` | Generation status only from a stored disposition; the two are distinct; neither is shown as issued | AU, PC, IS, AS, WC | C, D, E, F | **PLANNED — NOT TESTED** |
| EI7-N04 | Missing stored evidence means UNKNOWN | No stored timeline or disposition gives UNKNOWN, never "none" or zero | AU, AS, WC | C, E, F | **PLANNED — NOT TESTED** |
| EI7-N05 | Unknown issuance | Typed not-found; the Web shows a not-found page; no data invented | AU, AS, WC | C, E, F | **PLANNED — NOT TESTED** |
| EI7-N06 | `RECORDED` with no lifecycle events | `RECORDED`, an empty timeline and explicit text; no last-check time | AU, IS, AS, WC | C, D, E, F | **PLANNED — NOT TESTED** |
| EI7-N07 | Recorded before completeness declaration | In the recorded view with `covered_by_declaration: false`; absent from the declared view; the session not labelled complete | PC, IS, AS, WC | C, D, E, F | **PLANNED — NOT TESTED** |
| EI7-N08 | Modeled open and terminal lifecycle | Every open state and outcome class rendered with modeled wording; `ACTIVE` never executed or filled | AU, AS, WC | C, E, F | **PLANNED — NOT TESTED** |
| EI7-N09 | Terminal `INCOMPLETE_EVIDENCE` | Terminal, outcome class `UNRESOLVED`, deficiencies listed, no exit price | AU, IS, AS, WC | C, D, E, F | **PLANNED — NOT TESTED** |
| EI7-N10 | Missing cited snapshot | `EVIDENCE_NOT_FOUND` on the snapshot surface or evidence panel; the verified timeline stays visible | AU, IS, AS, WC | C, D, E, F | **PLANNED — NOT TESTED** |
| EI7-N11 | Corrupt persisted authority | `PERSISTED_AUTHORITY_INVALID` or `LIFECYCLE_HISTORY_INVALID`; whole response fails closed; never an empty list | AU, IS, AS | C, D, E | **PLANNED — NOT TESTED** |
| EI7-N12 | Dependency unavailable | `DEPENDENCY_UNAVAILABLE`, retryable; the Web offers Retry | AU, IS, AS, WC | C, D, E, F | **PLANNED — NOT TESTED** |
| EI7-N13 | Unsupported session or listing | `UNSUPPORTED_SCOPE`; never reported as closed or as having no recommendations | AU, AS | C, E | **PLANNED — NOT TESTED** |
| EI7-N14 | Non-SYNTHETIC refusal | Explicit refusal; never relabelled as synthetic | AU, IS, AS | C, D, E | **PLANNED — NOT TESTED** |
| EI7-N15 | Invalid watermark or position | `WATERMARK_AHEAD` beyond the authority; malformed values refused before any read | PC, IS, AS | C, D, E | **PLANNED — NOT TESTED** |
| EI7-N16 | Discovery bound and ordering | Deterministic `(issued_at, issuance_id)` order; overflow refused, never truncated | PC, IS, AS | C, D, E | **PLANNED — NOT TESTED** |
| EI7-N17 | GET and startup do not write | File bytes unchanged; `mode=ro` and `query_only` confirmed; no DDL; works on a read-only file; Futures schema never written to the Equity file | IS, AS, AR, XP | D, E | **PLANNED — NOT TESTED** |
| EI7-N18 | Private access enforcement | Allow-list equals the OpenAPI templates; unauthenticated and non-GET requests refused; documentation unreachable; no public bind | AS, AR | E | **PLANNED — NOT TESTED** |
| EI7-N19 | Deep links and accessible navigation | Session and recommendation URLs survive refresh and back and forward; keyboard and screen-reader access; IST with visible UTC | WC | F | **PLANNED — NOT TESTED** |
| EI7-N20 | Restart-stable records | A fresh process returns the same identities, events, order and responses | IS, AS, XP | D, E, H | **PLANNED — NOT TESTED** |
| EI7-N21 | Cross-connection consistency limitations | Detail is one transaction for issuance and lifecycle; list then detail may differ by appended events; tests assert prefix stability, not atomicity | PC, IS, XP | C, D | **PLANNED — NOT TESTED** |
| EI7-N22 | Synthetic operator journey — conditional | An explicit CLI builds the journey idempotently; a rerun is EQUIVALENT; never via HTTP | SJ, XP, AR | G, H | **PLANNED — NOT TESTED** (conditional on separate operator authorization) |
| EI7-N23 | No EI-8 performance metrics | No win rate, P&L, returns, Sharpe or aggregate statistics in any response or page; forbidden-vocabulary tests | AS, WC, AR | E, F | **PLANNED — NOT TESTED** |
| EI7-N24 | Research, proposal, issuance and trade distinctions | Candidate, proposal, issued recommendation, lifecycle state and trade stay distinct in types, responses and labels | AU, AS, WC | C, E, F | **PLANNED — NOT TESTED** |

| Count | Value |
| --- | --- |
| Scenario rows | 24 (EI7-N01 to EI7-N24, no gaps or duplicates) |
| PLANNED — NOT TESTED | 24 |
| Conditional | EI7-N22 |

## 17. Test strategy

| Layer | Repository | Scope |
| --- | --- | --- |
| Application unit (AU) | northstar-application | View building for every state, outcome class, event kind, generation status and error |
| Port and contract (PC) | northstar-application | Reader-port contracts, the bounded port, lineage invariants, test doubles for every error class |
| Infrastructure SQLite (IS) | northstar-infrastructure | Read-only adapters equal to the existing reads on the same files; no write; typed corruption and unavailability; read-only file |
| API ASGI (AS) | northstar-api | Routes over a real Equity file seeded by production use cases in test code; GET-only; negative matrix; no leaks; Futures regression unchanged |
| Web component (WC) | northstar-web | Fetch-stub tests; labels; forbidden vocabulary; GET-only; deep links; accessibility queries |
| Architecture (AR) | Application, Infrastructure, API | No writer, recorder, issuer, monitor, write-capable store or clock in read paths; no DDL in read adapters; EI4-D11 scan in northstar-api |
| Cross-process and restart (XP) | Infrastructure, API | Fresh-interpreter reads; readers in other processes |
| Synthetic journey (SJ) | API with the others | Conditional on EI-7G |

Every implementation PR also needs independent Linux CI with the checked-out sibling commits recorded, closing the dependency-provenance gap noted in EI-6C L12.

## 18. Milestone sequencing

Each slice needs its own owner authorization. CI checks out sibling repositories at `develop`, so a slice's CI passes only after the slices it depends on are merged.

```text
EI-7B (docs) ─▶ EI-7C (application) ─▶ EI-7D (infrastructure) ─▶ EI-7E (api) ─▶ EI-7H (end to end)
                       │                                              ▲               ▲
                       └──── shared read-port contract agreed first ──┘               │
                EI-7F (web): built against the frozen API contract; merged after EI-7E ┤
                EI-7G (operator, separately governed) ──────────────────────────────────┘
```

EI-7C and EI-7D are not independent: the bounded port and the reader-port contracts must be agreed in EI-7C before EI-7D can implement them.

**EI-7C — Application read and query contracts**

1. Repository: northstar-application.
2. Dependencies: EI7-CF1, CF2 and CF4 approved; EI-7C authorization.
3. Deliverables: immutable views; query use cases; the additive bounded read port; typed errors.
4. Tests: AU, PC, AR.
5. Gate: every view field traced to a stored fact; no writer dependency; Linux CI green.
6. Authority used: existing reader ports; R-2, R-3, R-6, R-12.
7. Missing authorization: the slice itself; the CF approvals.
8. Out of scope: Core changes; new write paths; new abstract methods on existing ports.

**EI-7D — Infrastructure read-only adapters**

1. Repository: northstar-infrastructure.
2. Dependencies: EI-7C merged.
3. Deliverables: read-only issuance, lifecycle, generation-record, research-timeline and snapshot readers; the bounded recorded-issuance adapter.
4. Tests: IS, AR, XP.
5. Gate: equivalence with existing reads; N17 at adapter level; existing EI-6A, EI-6B and EI-6C tests unchanged and green.
6. Authority used: R-3, R-4.
7. Missing authorization: the slice itself.
8. Out of scope: schema changes; behaviour changes to the existing adapters; write paths.

**EI-7E — Private equity API**

1. Repository: northstar-api.
2. Dependencies: EI-7C and EI-7D merged; EI7-CF2, CF3 and CF5 approved.
3. Deliverables: equity settings and runtime; routers and response models; error mapping; `/equity/health`; architecture scans; safe bind configuration and allow-list tests where separately authorized.
4. Tests: AS, AR, XP.
5. Gate: GET-only; N05 and N10 to N18 at API level; Futures behaviour unchanged.
6. Authority used: R-1, R-2, R-5, R-7, R-10, R-11.
7. Missing authorization: the slice; any Caddy or deployment change.
8. Out of scope: write routes; scheduling; deployment.

**EI-7F — Equity Web experience**

1. Repository: northstar-web.
2. Dependencies: EI7-CF2 and CF6 approved; built against fixtures in parallel with EI-7E, merged after it.
3. Deliverables: hash routes; session, detail and research pages; timeline; provenance; labels; IST formatting.
4. Tests: WC.
5. Gate: N19, N23, N24 and the Web parts of the negative scenarios.
6. Authority used: R-8, R-9, R-11, R-12.
7. Missing authorization: the slice itself.
8. Out of scope: performance displays; controls; authentication.

**EI-7G — Explicit synthetic journey operator**

1. Repositories: northstar-api (CLI) and northstar-infrastructure (synthetic listing-eligibility source).
2. Dependencies: EI7-CF7 approved under separate governance; EI-7D merged.
3. Deliverables: the explicit CLI and the eligibility source.
4. Tests: SJ, XP, AR (EI4-D11 scan; no HTTP trigger; no scheduler).
5. Gate: idempotent rerun; N22.
6. Authority used: R-1 (separate governance), EI4-D11.
7. Missing authorization: governance and implementation.
8. Out of scope: real data; scheduling; any HTTP write.

**EI-7H — End-to-end synthetic acceptance**

1. Repositories: tests across northstar-api and northstar-web, with the others as dependencies.
2. Dependencies: EI-7E, EI-7F and, for N22, EI-7G.
3. Deliverables: journey tests, restart tests, the full negative matrix, an engineering acceptance record.
4. Tests: SJ, XP, AS, WC.
5. Gate: every EI7-N row evidenced, including independent Linux CI.
6. Authority used: all approved directions.
7. Missing authorization: the slice and the final owner acceptance.
8. Out of scope: EI-8; deployment.

## 19. Open decisions

- EI7-CF1 to EI7-CF7 ([register section 8](../product/Equity-Intraday-V1-EI7-Decision-Register.md#8-contract-finalization-decisions-pending)).
- Whether the policy-set version needed to derive a generation key for EI7-N03 is a request parameter or configuration (EI7-CF4).
- Persisting refused or duplicate research outcomes: open in the EI-3 register, not decided here.
- A discovery read for generation attempts without a disposition: none exists; they stay UNKNOWN.
- EI0-D5 (universe), still PROPOSED; its approval may require revisiting the discovery limit.

## 20. Limitations

EI-7 inherits, and does not resolve, the limitations listed in the [register, section 7](../product/Equity-Intraday-V1-EI7-Decision-Register.md#7-inherited-limitations). In addition:

- read cost grows with the issuance ledger, because every issuance read checks the whole ledger;
- the declared view hides new issuances until a later declaration, by design;
- a `mode=ro` reader cannot recover a hot rollback journal;
- generation status for unissued candidates depends on deriving the generation key; attempts without a disposition cannot be discovered;
- there is no authoritative universe, so no instrument list can be shown.

## 21. Non-authorizations

This design authorizes nothing. In particular it does not authorize: any EI-7C to EI-7H implementation; the operator CLI or the synthetic listing-eligibility source; a scheduler; a Caddy, bind or deployment change; database creation, migration or file moves; real or provider data; ACTUAL issuance or new issuance functionality; broker integration, orders or fills; EI-8 metrics or restatement; the EI-6C follow-ups F1 to F10; changes to Core or to the EI-2B, EI-4D, EI-6A, EI-6B or EI-6C contracts. The existing EI-6A synthetic MODELED issuance authority is not revoked. EI4-C4 stays pending.

## 22. Related documents

- [EI-7 Decision Register](../product/Equity-Intraday-V1-EI7-Decision-Register.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md), sections 3.3, 6.4, 11 and 12.1
- [EI-6C Lifecycle Monitoring and Snapshot Selection Design v1.0](Equity-Intraday-V1-EI6C-Lifecycle-Monitoring-and-Snapshot-Selection-Design-v1.0.md)
- [EI-6B Durable Recommendation Lifecycle Persistence Design v1.0](Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Design-v1.0.md)
- [EI-6A Durable Issuance Authority SQLite Storage Design v1.0](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-SQLite-Storage-Design-v1.0.md)
- [EI4-D11 Operational Generation Write Boundary Enforcement Design](Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md), sections 13 and 14
- [EI-6C Engineering Acceptance Record](../product/Equity-Intraday-V1-EI6C-Lifecycle-Monitoring-and-Snapshot-Selection-Engineering-Acceptance-Record.md)
- [Indian Futures Deployment Runbook](../operations/Indian-Futures-Deployment-Runbook.md), sections 1 and 8
