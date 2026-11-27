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

> **Note, 2026-11-21 (EI-7B Docs PR-1).** The status above records the design as prepared on 2026-11-16 and is not rewritten. The owner approved **EI7-CF1, EI7-CF2 and EI7-CF4 on 2026-11-20**, and the refinements **EI7-P1 to EI7-P8**, **EI7-EC1** and the final confirmations **EI7-FC1 to EI7-FC6 on 2026-11-21** ([register section 13](../product/Equity-Intraday-V1-EI7-Decision-Register.md#13-approved-contracts-ei7-cf1-ei7-cf2-and-ei7-cf4-2026-11-20-and-2026-11-21)). For discovery, the five essential endpoints, response and error contracts, and integrity and research status, the governing text is that register section and [section 23](#23-approved-contract-reconciliation-2026-11-21) of this design; dated notes in sections 6.1, 9, 11, 12, 13, 16, 18, 19 and 20 point there. EI7-CF3, EI7-CF5, EI7-CF6 and EI7-CF7 stay **PROPOSED — PENDING OWNER APPROVAL**. EI-7C to EI-7H stay **NOT AUTHORIZED**. EI7-N01 to EI7-N24 stay **PLANNED — NOT TESTED**. On 2026-11-21 the six repositories were confirmed at the code commits below, with northstar-docs at `d9bc867`.

> **Note, 2026-11-28 (EI-7E governance documentation PR).** The status and the note above are not rewritten. EI-7C and EI-7D were authorized on 2026-11-21 and are merged (northstar-application `16bb21a`; northstar-infrastructure `caa2c46`). The owner approved **EI7-CF3, EI7-CF5, EI7-SI1 and EI7-SI2 on 2026-11-28** ([register section 14](../product/Equity-Intraday-V1-EI7-Decision-Register.md#14-approved-contracts-ei7-cf3-ei7-cf5-ei7-si1-and-ei7-si2-2026-11-28)); their design mechanics are in [section 24](#24-approved-private-access-runtime-and-storage-integrity-design-2026-11-28), which governs private access, the Equity runtime and database, health, journal mode and storage-engine error classification. Dated notes in sections 4, 7, 8, 10, 11, 12, 15, 16, 18, 19, 20, 21 and 23 point there. SI-A, SI-B and EI-7E to EI-7H stay **NOT AUTHORIZED**; EI7-CF6 and EI7-CF7 stay **PROPOSED — PENDING OWNER APPROVAL**; EI7-N01 to EI7-N24 stay **PLANNED — NOT TESTED**. Source facts for section 24 were verified on 2026-11-28 at northstar-core `29bd34a`, northstar-application `16bb21a`, northstar-infrastructure `caa2c46`, northstar-api `7ad37f5`, northstar-web `5aeb94c` and northstar-docs `1e8138c`.

> **Note, 2026-11-28 (EI-7E technical-closure governance documentation).** The status and the notes above are not rewritten. SI-A and SI-B were since separately authorized and merged (northstar-application #101, `090bb54`; northstar-infrastructure #71, `f65e66a`). After the EI-7E read-only technical-closure audit, the owner approved the technical-closure decisions **EI7E-D01 to EI7E-D07B** (D-01 to D-07, with D-07A and D-07B) on **2026-11-28** ([register section 15](../product/Equity-Intraday-V1-EI7-Decision-Register.md#15-ei-7e-technical-closure-decisions-d-01-to-d-07-2026-11-28)). Their mechanics are in [section 25](#25-ei-7e-approved-technical-closure-architecture-2026-11-28): the dedicated Equity application, CME isolation, the India network direction, database isolation, the read-only health probe, the V1 session-list boundary, strategy encoding and unexpected-failure safety. Dated notes in sections 8, 11, 12, 16, 18, 19, 20, 21, 23 and 24 point there. The EI-7E slices P1, P2, E1, D1 and V1, and EI-7F to EI-7H, stay **NOT AUTHORIZED**; operational Equity exposure is prohibited until EI7E-D05 is satisfied; EI7-N01 to EI7-N24 stay **PLANNED — NOT TESTED**. The audit read northstar-core `29bd34a`, northstar-application `090bb54`, northstar-infrastructure `f65e66a`, northstar-api `7ad37f5`, northstar-web `5aeb94c` and northstar-docs `0d6fe0b`.

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

> **Note, 2026-11-28.** The approved topology, binding and authentication boundary (EI7-CF3-A to EI7-CF3-C) are in [sections 24.1 and 24.2](#241-private-access-topology). The "GET/HEAD allow-list" above is refined: HEAD only where EI-7E explicitly supports it and verifies it as non-mutating (EI7-CF3-D; [section 24.3](#243-method-and-route-restrictions)).

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

> **Note, 2026-11-21.** The dependencies above are superseded by [section 23.6](#236-ei-7c-use-cases-and-ports). Both session views read the new recorded-session port, not `session_view`. Research and evidence reads use the reader-only timeline and snapshot ports, never the write-capable stores (EI7-P4). `EquitySyntheticPolicyCatalogStore` is not a dependency of any EI-7C use case, because policy-set-based lookup is deferred (EI7-P3). `GetEquityEvidenceSnapshotUseCase` is deferred with the snapshot endpoint. The detail steps of section 6.2 stand, with the ten comparisons of [register section 13.4](../product/Equity-Intraday-V1-EI7-Decision-Register.md#134-ei7-cf4-approved-integrity-lineage-and-research-status-contract).

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

> **Note, 2026-11-28.** The read-only adapters are merged (EI-7D, `caa2c46`). The approved startup, journal-mode, hot-journal and storage-engine error rules are in [sections 24.7, 24.9 and 24.10](#247-equity-database-configuration-and-read-only-startup) (EI7-CF5, EI7-SI1); integrity coverage is in [section 24.11](#2411-integrity-verification-coverage) (EI7-SI2).

## 8. API composition root and Equity and Futures isolation

- **Composition root.** An equity runtime in northstar-api, alongside but separate from the Futures `DatabaseRuntime`; it constructs read-only adapters and query use cases only.
- **Separate files (R-7).** The Equity file holds the EI-2B evidence, EI-3C timelines, the policy catalog, EI-4D records and the EI-6A and EI-6B tables, linked by foreign keys. The Futures file is configured separately. No query, join or transaction spans them.
- **Settings.** A separate Equity database setting (a name such as `NORTHSTAR_EQUITY_DATABASE` is a proposal, EI7-CF5). Equity routes answer `DEPENDENCY_UNAVAILABLE` (proposed 503) when it is absent. Futures settings never become required for equity, nor equity settings for Futures.
- **No cross-initialization.** The Futures `initialize_database` never runs on the Equity file; no equity code runs on the Futures file.
- **Health.** `/health` and its Futures probe are unchanged; `/equity/health` is separate.
- **Architecture scans in northstar-api.** The equity composition constructs no writer, recorder, issuer, monitor or write-capable store, as EI4-D11 requires of the first repository with such wiring.
- **No migration.** No Equity database is deployed today, so nothing moves; R-7 governs future placement.

> **Note, 2026-11-28.** Superseded in part by EI7-CF5 ([section 24.7](#247-equity-database-configuration-and-read-only-startup)): the proposed setting is `NORTHSTAR_EQUITY_SQLITE_PATH`, not `NORTHSTAR_EQUITY_DATABASE`; when it is absent Equity routes answer `EQUITY_NOT_CONFIGURED` (HTTP 503, not retryable), not `DEPENDENCY_UNAVAILABLE`. The Futures runtime still initializes its own schema at startup, so only the Equity read stack, not the whole API process, is described as read-only (EI7-CF5-H).

> **Note, 2026-11-28 (technical closure).** EI7E-D01 places the Equity composition root in a **dedicated Equity application and process**, not alongside the Futures runtime in the existing application, and EI7E-D04 fixes its database isolation ([sections 25.1](#251-dedicated-equity-api-composition) and [25.4](#254-equity-database-isolation)). Because the Equity process never builds the Futures runtime, it inherits no Futures startup write. The existing `/health` and its Futures probe are unchanged.

## 9. Bounded discovery

| View | Source | Shows | Proposed contract (EI7-CF1) |
| --- | --- | --- | --- |
| Declared | Existing `session_view` / `session_view_at` | Issuances covered by the latest declaration (or the declaration at a watermark) | Limit 50; `(issued_at, issuance_id)` order; `declaration` state; `recording_watermark` |
| Recorded | New bounded read | Every recorded issuance of the session, covered or not | Limit 50; same order; per-item `covered_by_declaration`; `recording_watermark`; optional `watermark` |

- **Why both.** Issuance at `t` requires a declaration through `t − 1 µs`, and `session_view` lists only issuances with `issued_at <= complete_through`. A new issuance is therefore absent from the declared view until a later declaration covers it (EI7-N07).
- **Proposed port.** An additive, read-only protocol, separate from `EquityRecommendationIssuanceReader` so that no existing implementation or test double must change; for example `EquityRecordedSessionIssuanceReader.recorded_session_issuances(exchange_code, trading_date, limit)`, returning the issuances with their recording sequences, the authority's watermark and the declaration state. It is served by the existing index `equity_recommendation_issuances_by_session (exchange_code, trading_date)`, with the same verification as `session_view`.
- **Not a list-all.** One exchange and one date; no range; no strategy-wide or global listing. EI6C-P7 still governs EI-6C monitoring discovery, and the monitor is not given this port.
- **Overflow.** A typed refusal, never truncation (proposed `DISCOVERY_LIMIT_EXCEEDED`).

> **Note, 2026-11-21.** EI7-CF1 is approved and refined; see [register section 13.2](../product/Equity-Intraday-V1-EI7-Decision-Register.md#132-ei7-cf1-approved-session-discovery-contract) and [sections 23.1 and 23.2](#231-recorded-session-read). Superseded above: the recorded view is ordered by ascending `recording_sequence`, not "same order" (EI7-P1); the declared view keeps `(issued_at, issuance_id)`. Both views are built from one bounded read of the new port at one watermark, not from `session_view`. The port also takes an optional watermark and returns the recording position, the selected watermark and the declarations through it. Overflow is HTTP 409 (EI7-P2).

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

> **Note, 2026-11-28.** The five essential GET families are the only Equity routes that may be exposed, under default deny (EI7-CF3-D; [section 24.3](#243-method-and-route-restrictions)). The three optional rows above stay deferred and are not exposable. The `{strategy}` segment is the canonical base64url encoding of the identity (EI7-CF3-F; [section 24.5](#245-strategy-identity-encoding)).

**A separate lifecycle endpoint is not warranted in V1.** The detail must call `load_lifecycle` anyway, because that is the read that returns the authoritative issuance; it receives the complete, verified timeline in the same transaction. A lifecycle holds a small number of events (one per bar-driven transition plus any recorded source revisions), so there is no size reason to split it. A separate endpoint would duplicate the read and invite inconsistent pairings.

## 11. Proposed response views

Common to every view: owned by northstar-application; exact decimal strings for prices, quantities and amounts; canonical UTC instants; identities and enum names verbatim; `null` only for "not applicable" or "not stored"; unknown or unavailable reported as an explicit status object; no secrets, file paths or SQLite text; labels `evidence_class: "SYNTHETIC"` and, where an issuance is present, `issuance_kind: "MODELED"` and `actionable: false`.

> **Note, 2026-11-21.** The approved wire rules are in [register section 13.3](../product/Equity-Intraday-V1-EI7-Decision-Register.md#133-ei7-cf2-approved-private-read-api-contract) and [section 23.3](#233-wire-representation). Every response carries `contract_id: "northstar.equity-private-read/1"` (EI7-P8). **Every** integer, including `recording_sequence`, `recording_watermark`, `position` and event `sequence`, is a canonical decimal string; the Equity read contract has no JSON numbers. In section 11.3, `recording_watermark` is present whether or not the session is declared, together with `recording_position`. Section 11.5 is superseded by [section 23.5](#235-research-view). Section 11.6 is deferred with the snapshot endpoint, and its refusal is HTTP 409 under EI7-EC1.

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

> **Note, 2026-11-28 (technical closure).** In the V1 HTTP projections of sections 11.3 and 11.4, the per-item lifecycle summary is omitted, and no lifecycle read is made to build a session list (EI7E-D06; [section 25.6](#256-v1-session-list-presentation-boundary)). The lifecycle stays in the detail.

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

> **Note, 2026-11-28.** Superseded by EI7-CF5-E and EI7-SI2 ([section 24.8](#248-health-semantics)): four distinguishable outcomes (readable, not configured, dependency unavailable, integrity failure where detectable); no database-wide claim. The field names above remain proposals for the EI-7E wire-contract closure.

> **Note, 2026-11-28 (technical closure).** Settled by EI7E-D02. A readable probe answers HTTP 200 `{"contract_id": "northstar.equity-private-read/1", "status": "READABLE"}`, and the three failure outcomes use the error envelope. The probe is an Application-owned port with an Infrastructure adapter (P1, P2); its required tables are checked through the EI-7D read-only transaction ([section 25.5](#255-read-only-health-probe)).

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

> **Note, 2026-11-21.** The table above is superseded by the approved status mapping in [register section 13.3](../product/Equity-Intraday-V1-EI7-Decision-Register.md#133-ei7-cf2-approved-private-read-api-contract). `DISCOVERY_LIMIT_EXCEEDED` and `EVIDENCE_CLASS_UNSUPPORTED` are HTTP 409, not "422 (open)". A non-SYNTHETIC snapshot refuses the affected response and is never a panel (EI7-EC1). A watermark above 2^63−1 is 409 `WATERMARK_AHEAD` (EI7-P7). `EQUITY_NOT_CONFIGURED` (503) is added where applicable under EI7-CF5. `EVIDENCE_NOT_FOUND` is a panel status only while the snapshot endpoint is deferred. `NOT_QUERYABLE` is a research status, not an error. Every error body carries `contract_id`.

> **Note, 2026-11-28.** `EQUITY_NOT_CONFIGURED` is triggered by an absent Equity database setting (EI7-CF5-D). A permanent storage-engine failure (`SQLITE_CORRUPT`, `SQLITE_NOTADB`) maps to `PERSISTED_AUTHORITY_INVALID`, HTTP 500, not retryable, through the Application-owned storage-integrity contract of EI7-SI1; busy, locked, cannot-open, hot-journal and missing-table conditions stay `DEPENDENCY_UNAVAILABLE` ([section 24.10](#2410-storage-engine-error-compatibility-contract)). No public error code is added.

> **Note, 2026-11-28 (technical closure).** EI7E-D07B keeps the mapping above unchanged. It adds that an unexpected, unclassified failure answers HTTP 500 through a fixed, generic **non-contract fallback**. That fallback is not a member of the domain-error vocabulary and is never labelled `PERSISTED_AUTHORITY_INVALID` without evidence. Unknown paths and unsupported methods keep their framework 404 and 405 ([section 25.8](#258-unexpected-http-failure-safety)).

## 13. Read consistency

- **Detail.** The issuance and lifecycle are one transaction. The disposition is a separate read of insert-only rows, protected by the lineage check.
- **Lists.** A list is one transaction; each item's lifecycle summary is a later read and may show more events. Issuances are insert-only and lifecycles append-only, so a later read only ever extends an earlier one.
- **Repeatability.** The recorded view accepts a `watermark` to reproduce an earlier list exactly. Lifecycles have no watermark; `position` lets a client detect change.
- **Caching.** None beyond the client's current page. No view across endpoints or connections is atomic (EI-6C L7; EI7-N21).

> **Note, 2026-11-21.** Under EI7-CF1, both session views come from one read transaction at one watermark and accept the same optional `watermark`. Ledger rows with `recording_sequence <= W` are insert-only, so two reads pinned to the same W return the same issuances, even across connections and processes. The per-item lifecycle summaries are not pinned. Nothing else in this section changes.

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

> **Note, 2026-11-28.** The approved security contract (EI7-CF3) and its required negative tests are in [sections 24.1 to 24.6 and 24.13](#2413-security-and-failure-tests). "GET only" stands; HEAD only where explicitly supported and verified as non-mutating.

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

> **Note, 2026-11-21.** The rows above are kept as prepared. The revised expected behaviours of EI7-N03, N04, N07, N10, N11, N14, N15 and N16 under the approved contracts, and the coverage every implementation must add, are in [section 23.8](#238-acceptance-reconciliation). No scenario is added or removed; all 24 stay **PLANNED — NOT TESTED**.

> **Note, 2026-11-28.** A second reconciliation, for EI7-CF3, EI7-CF5, EI7-SI1 and EI7-SI2, with the slice responsible for each item, is in [section 24.14](#2414-acceptance-reconciliation). No scenario is added, removed or renumbered; all 24 stay **PLANNED — NOT TESTED**.

> **Note, 2026-11-28 (technical closure).** A third reconciliation, for EI7E-D01 to EI7E-D07B, separating existing layer evidence from the evidence owed by P1 and P2, E1, D1 and V1, and EI-7F and EI-7H, is in [section 25.12](#2512-acceptance-and-negative-test-matrix). All 24 stay **PLANNED — NOT TESTED**.

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

> **Note, 2026-11-21.** The EI7-CF1, CF2 and CF4 dependencies of EI-7C are met (approved 2026-11-20); the slice itself is still **NOT AUTHORIZED**. EI-7C's "out of scope: new abstract methods on existing ports" stands: the three new ports are separate ([section 23.6](#236-ei-7c-use-cases-and-ports)). The one permitted exception to "no change to existing code" is the narrow EI7-P4 reader-only adjustment of the EI-3C research rehydration dependency (EI7-FC6): "The narrow P4 compatibility change is eligible for implementation within a separately owner-authorized EI-7C Application slice. It is not authorized by EI-7B Docs PR-1." EI-7D's deliverables no longer include a policy catalog reader (EI7-P3). EI-7E still needs EI7-CF3 and EI7-CF5; EI-7F still needs EI7-CF6; EI-7G still needs EI7-CF7 and separate governance. The revised sequence is in [section 23.9](#239-sequencing).

> **Note, 2026-11-28.** EI-7C and EI-7D are authorized (2026-11-21) and merged. EI7-CF3 and EI7-CF5 are approved, and two slices, SI-A (Application) and SI-B (Infrastructure), now precede EI-7E. The governing sequence is [section 24.12](#2412-dependency-ordering-and-gates).

> **Note, 2026-11-28 (technical closure).** SI-A and SI-B are merged. EI-7E is now delivered as G, P1, P2, E1, D1 and V1, each separately authorized ([section 25.11](#2511-implementation-sequence); [register section 15.9](../product/Equity-Intraday-V1-EI7-Decision-Register.md#159-dependency-and-authorization-gates)). The EI-7E outline below stays as written. Its deployment items are D1 and V1, and its "safe bind configuration" is EI7E-D03.

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

> **Note, 2026-11-21.** EI7-CF1, EI7-CF2 and EI7-CF4 are no longer open (approved 2026-11-20). The policy-set question is settled by EI7-P3: neither a request parameter nor configuration; policy-set-based lookup is deferred to separate governance. The open and pending items are listed in [register section 13.10](../product/Equity-Intraday-V1-EI7-Decision-Register.md#1310-pending-technical-details-and-outstanding-decisions).

> **Note, 2026-11-28.** EI7-CF3 and EI7-CF5 are no longer open (approved 2026-11-28). The implementation details they leave to SI-A, SI-B and EI-7E are the technical closure items of [register section 14.7](../product/Equity-Intraday-V1-EI7-Decision-Register.md#147-technical-closure-items) and [section 24.16](#2416-technical-closure-items). EI7-CF6 and EI7-CF7 stay open.

> **Note, 2026-11-28 (technical closure).** Most EI-7E closure items are settled by EI7E-D01 to EI7E-D07B. The state of each is in [register section 14.7, note](../product/Equity-Intraday-V1-EI7-Decision-Register.md#147-technical-closure-items). Still open, for E1: unknown query parameters; the precedence between `EQUITY_NOT_CONFIGURED` and request validation; the non-container default bind. Still open, for D1 and V1: the Equity port, the file location and journal-mode controls.

## 20. Limitations

EI-7 inherits, and does not resolve, the limitations listed in the [register, section 7](../product/Equity-Intraday-V1-EI7-Decision-Register.md#7-inherited-limitations). In addition:

- read cost grows with the issuance ledger, because every issuance read checks the whole ledger;
- the declared view hides new issuances until a later declaration, by design;
- a `mode=ro` reader cannot recover a hot rollback journal;
- generation status for unissued candidates depends on deriving the generation key; attempts without a disposition cannot be discovered;
- there is no authoritative universe, so no instrument list can be shown.

> **Note, 2026-11-21.** The limitations accepted with the approved contracts, including overflow before row-level corruption, the declared view's refusal above the limit, and the two provenance limits, are listed in [register section 13.9](../product/Equity-Intraday-V1-EI7-Decision-Register.md#139-accepted-limitations).

> **Note, 2026-11-28.** The limitations accepted with EI7-CF3, EI7-CF5, EI7-SI1 and EI7-SI2 (deployment-dependent privacy, the WAL check-to-open race, hot-journal unavailability, a process-wide Futures startup write, `SQLITE_NOTADB` as unusable storage, no detection of unread corruption) are in [register section 14.8](../product/Equity-Intraday-V1-EI7-Decision-Register.md#148-accepted-limitations).

> **Note, 2026-11-28 (technical closure).** Further limitations and deployment blockers are in [register section 15.10](../product/Equity-Intraday-V1-EI7-Decision-Register.md#1510-source-facts-residual-limitations-and-deployment-blockers) and [section 25.13](#2513-source-facts-residual-risks-and-limitations):
> - READABLE health is not an integrity certification;
> - strategy identities are transport-bounded;
> - V1 lists carry no lifecycle state;
> - no Equity database is provisioned;
> - hot-journal recovery is unowned.

## 21. Non-authorizations

This design authorizes nothing. In particular it does not authorize: any EI-7C to EI-7H implementation; the operator CLI or the synthetic listing-eligibility source; a scheduler; a Caddy, bind or deployment change; database creation, migration or file moves; real or provider data; ACTUAL issuance or new issuance functionality; broker integration, orders or fills; EI-8 metrics or restatement; the EI-6C follow-ups F1 to F10; changes to Core or to the EI-2B, EI-4D, EI-6A, EI-6B or EI-6C contracts. The existing EI-6A synthetic MODELED issuance authority is not revoked. EI4-C4 stays pending.

> **Note, 2026-11-28.** The sentence above is not rewritten; EI-7C and EI-7D were later authorized separately (2026-11-21). Section 24 likewise authorizes nothing: not SI-A, SI-B or EI-7E to EI-7H, and not any Caddy, bind, CORS, documentation-route, compose or other deployment change, integrity-check automation or repair.

> **Note, 2026-11-28 (technical closure).** Section 25 likewise authorizes nothing. It does not authorize:
> - P1, P2 or E1;
> - the D1 deployment change or the V1 verification;
> - Equity database provisioning, population, copying, recovery or repair;
> - any Equity writer.

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

## 23. Approved contract reconciliation (2026-11-21)

Added by EI-7B Docs PR-1. Sections 1 to 22 are not rewritten; dated notes there point here. The decisions are recorded in [register section 13](../product/Equity-Intraday-V1-EI7-Decision-Register.md#13-approved-contracts-ei7-cf1-ei7-cf2-and-ei7-cf4-2026-11-20-and-2026-11-21). This section gives the engineering mechanics those decisions imply, using the evidence of the EI-7B CF1, CF2 and CF4 read-only feasibility audit (2026-11-20; source inspection only, no tests run). Where sections 1 to 22 differ, register section 13 and this section govern. The owner's final confirmations EI7-FC1 to EI7-FC6 (2026-11-21) are applied in sections 23.1 to 23.9 and summarised in [section 23.10](#2310-final-owner-confirmations-ei7-fc1-to-ei7-fc6). Every name below is a proposal, except `contract_id`, `recording_position` and `recording_watermark`, which EI7-FC4 confirms. **Nothing here is implemented or authorized.**

### 23.1 Recorded-session read

The new read-only Application port `EquityRecordedSessionIssuanceReader` (EI-7C) takes a canonical exchange code, a trading date, the limit and an optional watermark. It returns one immutable result:

| Result concept | Meaning |
| --- | --- |
| Exchange code, trading date | The session |
| Recording position N (`recording_position`) | The observed authoritative ledger position at the read; the ledger is dense, 1 to N |
| Recording watermark W (`recording_watermark`) | The requested or selected replay boundary: N when none is requested; 0 ≤ W ≤ N |
| Declarations through W | The session's verified completeness declarations with `recording_sequence <= W`, and the derived `complete_through(W)`, or an explicit undeclared state |
| Issuances | Every recorded issuance of the session with `recording_sequence <= W`, each re-derived and verified, in ascending `recording_sequence`; never more than the limit |
| Authority reference | The EI-6A issuance authority reference |

Its typed refusals are a new discovery-limit refusal, which carries the limit, and the existing EI-6A errors: `EquityIssuanceWatermarkBeyondAuthorityError`, `InvalidStoredEquityRecommendationIssuanceAuthorityError` (categories LEDGER, DECLARATION and ISSUANCE) and the unavailability error.

**One read-only transaction, in the order EI7-P6 requires:**

1. Validate a requested W in Python: a non-negative integer, not a Boolean. A value above 2^63−1 is refused as beyond the authority before any SQL parameter is bound. Python's `sqlite3` raises `OverflowError`, which is not a `sqlite3.Error`, for such a value, so it must never reach a bind (EI7-P7).
2. Begin the transaction and run the existing whole-ledger check, which yields N. A LEDGER failure here precedes every other outcome.
3. Set W to the requested value or N. If W > N, refuse with watermark-beyond.
4. Read and verify the session's declarations with the existing helper, and derive `complete_through(W)` with the existing pure rule `equity_declared_complete_through`.
5. Count the session's issuance rows with `recording_sequence <= W` through the existing index `equity_recommendation_issuances_by_session (exchange_code, trading_date)`. Above the limit, refuse with the discovery-limit refusal, without re-deriving any row.
6. Otherwise read those rows in `recording_sequence` order and re-derive each with the existing helper, unchanged; then end the transaction.

**Source basis.** Steps 2, 4 and 6 are performed today by private verified helpers of the EI-6A adapter, which the EI-6B adapter already reuses. The index exists. No schema change is needed. The existing `session_view` and `session_view_at` are unchanged and stay available to existing callers.

### 23.2 Session presentations

Both views are built in the Application from one result of section 23.1:

| View | Items | Order | Declaration |
| --- | --- | --- | --- |
| Recorded (`/recorded-issuances`) | All issuances of the result, each with `covered_by_declaration` (`issued_at <= complete_through(W)`; `false` when undeclared) | Ascending `recording_sequence` | `NONE`, or `DECLARED` with `complete_through(W)` |
| Declared (`/recommendations`) | `equity_issuance_session_view(exchange, date, W, issuances, declarations)` over the same verified records | The rule's own order, ascending `(issued_at, recommendation_key)`; the key is the `issuance_id` | Same |

- When the rule returns `None`, which happens when the session is undeclared at W, the declared view says `declaration: NONE` and still states N and W. It is never shown as "no recommendations".
- Neither view re-sorts the other. The EI-6A type `EquitySessionRecommendationHistory` keeps enforcing its `(issued_at, key)` order.
- **What discovery is authoritative for.** Issuance identities and the session recording facts at W: N, W, declaration state, recording sequences and `covered_by_declaration`. The full lifecycle belongs to the verified detail read (section 23.4), whose issuance and lifecycle come from one transaction.
- **Optional per-item lifecycle summary** (`state`, `is_terminal`, `outcome_class`, `position`). It comes from an independent `load_lifecycle` call per item, made after the session read. It is not pinned to W, not atomic with the session read or with the other items, and may reflect events recorded after W. No response field or label may present the summaries as one consistent lifecycle observation. Each summary carries its own status: a failure is that item's explicit `invalid` (`LIFECYCLE_HISTORY_INVALID`) or `unavailable` status, never an omitted, defaulted or successful-looking summary. An issuance-authority failure fails the response.
- **Overflow (EI7-FC1).** Above 50 recorded issuances at W, both projections refuse with HTTP 409. The bound belongs to this EI-7 projection layer, not to the EI-6A authority, and the refusal is not a corruption verdict.

> **Note, 2026-11-28 (technical closure).** The optional per-item lifecycle summary above is not used by the V1 HTTP session lists (EI7E-D06; [section 25.6](#256-v1-session-list-presentation-boundary)). The Application capability is unchanged.

### 23.3 Wire representation

Illustrative only. Values in angle brackets are placeholders, not real identities.

```json
{
  "contract_id": "northstar.equity-private-read/1",
  "exchange_code": "NSE",
  "trading_date": "<YYYY-MM-DD>",
  "evidence_class": "SYNTHETIC",
  "recording_position": "<N>",
  "recording_watermark": "<W>",
  "declaration": { "status": "DECLARED", "complete_through": "<canonical UTC instant>" },
  "items": [
    {
      "issuance_id": "<issuance identity>",
      "recording_sequence": "<sequence>",
      "issued_at": "<canonical UTC instant, modeled>",
      "issuance_kind": "MODELED",
      "actionable": false,
      "covered_by_declaration": true,
      "lifecycle": { "status": "available", "state": "<state>", "is_terminal": false, "position": "<event count>" }
    }
  ]
}
```

```json
{
  "contract_id": "northstar.equity-private-read/1",
  "error": {
    "code": "DISCOVERY_LIMIT_EXCEEDED",
    "message": "More than 50 recorded issuances in this session; no partial list is returned.",
    "retryable": false,
    "scope": "response"
  }
}
```

- `contract_id` is at the top level of both envelopes (EI7-FC4).
- `lifecycle` in an item is the optional, independently read summary of section 23.2. Its `status` is per item, and it is not part of the watermark-pinned session facts.
- Integer strings match `^(0|[1-9][0-9]*)$`. The request `watermark` uses the same syntax: anything else is HTTP 422 `UNSUPPORTED_SCOPE`, and a valid value above 2^63−1 is HTTP 409 `WATERMARK_AHEAD`.
- Booleans are JSON booleans. No value is a JSON number.
- Clients treat identities and integer strings as opaque text and compare integers exactly, never through a binary float.

> **Note, 2026-11-28 (technical closure).** The item-level `lifecycle` object in the first example above is not part of the V1 session-list responses (EI7E-D06). Every other field and both envelopes stand. Unexpected failures outside the vocabulary use the non-contract fallback of EI7E-D07B ([section 25.8](#258-unexpected-http-failure-safety)).

### 23.4 Verified detail and evidence panels

1. Validate the identity with `is_equity_recommendation_issuance_identity`; otherwise HTTP 422.
2. `load_lifecycle(issuance_id)`. `None` gives HTTP 404 `ISSUANCE_NOT_FOUND`.
3. `load_disposition(issuance.generation_key)` on a separate connection. `None` gives HTTP 500 `PERSISTED_AUTHORITY_INVALID`.
4. Apply the ten comparisons of [register section 13.4](../product/Equity-Intraday-V1-EI7-Decision-Register.md#134-ei7-cf4-approved-integrity-lineage-and-research-status-contract). Any failure fails the whole detail.
5. Build the required facts, the complete lifecycle timeline and the two provenance-limit statements.
6. Build the optional panels under EI7-P5 and EI7-EC1:

| Panel input | Not found | Corrupt | Class not SYNTHETIC | Reader unavailable |
| --- | --- | --- | --- | --- |
| Snapshot cited by a lifecycle event (its `evidence_reference` has no foreign key and no database class check; the EI-6B recorder enforces it) | Panel `unavailable`, `EVIDENCE_NOT_FOUND` | Panel `invalid`, content withheld | Whole detail refused, HTTP 409 `EVIDENCE_CLASS_UNSUPPORTED` | Panel `unavailable`, retryable |
| Candidate evidence snapshot of the disposition | Panel `invalid` (the foreign key makes absence a broken reference) | Panel `invalid` | Whole detail refused, HTTP 409 | Panel `unavailable`, retryable |
| Research origin, the timeline entry rehydrated by EI3-D13 | Panel `invalid` (the disposition's candidate entry has a foreign key, so absence is a broken reference) | Panel `invalid`, because the required lineage is proven from the disposition's frozen reference, not from the timeline | Whole detail refused, HTTP 409 | Panel `unavailable`, retryable |

A panel never hides, replaces or downgrades a required fact. If a failure compromises the required identity or lineage, the whole detail fails instead.

### 23.5 Research view

1. Validate strategy, exchange, symbol and date; otherwise HTTP 422. The route encoding of `StrategyIdentity` is pending before EI-7E.
2. Load the stored timeline through `EquityIntradayResearchTimelineReader`. If none is stored, the timeline is UNKNOWN, and step 4 still runs.
3. Rehydrate and verify it by EI3-D13 through the reader-only snapshot port (EI7-P4), in this order (EI7-FC3):
   1. **Configuration.** Decode the stored, authoritative configuration document exactly. Verify that its re-encoded canonical text, its strategy and version, and the configuration embedded in the timeline identity document all agree. For a recorded issuance, also verify that the research configuration version bound into the generation key, and the configuration in the disposition's decoded inputs, equal it. A current configuration or a client-supplied value is never substituted. Missing or inconsistent configuration gives HTTP 500, a typed whole-response research integrity failure.
   2. **Evidence-class gate.** Check every snapshot retrieved for the timeline for class **before** any decision is re-derived, used or presented. Non-SYNTHETIC gives HTTP 409 `EVIDENCE_CLASS_UNSUPPORTED` for the whole response, with no payload. The evaluator's reconstruction of such a snapshot as a `REJECTED` decision (`EVIDENCE_NOT_SYNTHETIC`) is never presented as a research outcome.
   3. **Re-derivation.** Re-derive and verify the decisions and the decision and entry fingerprints. Any verification or rehydration failure gives HTTP 500, typed integrity, for the whole response (EI7-P5).
   4. **Availability.** An unavailable reader gives HTTP 503.
4. `load_session_issuance(strategy, listing, date)`:
   - **if present:** `load_disposition(issuance.generation_key)`. It must exist, be `PROPOSED` and cite the verified timeline's `CANDIDATE` entry; otherwise HTTP 500 `PERSISTED_AUTHORITY_INVALID`. An issuance with no stored timeline is also an integrity failure. The status shows the generation as `PROPOSED` and issued, with the issuance identity.
   - **if absent:** the generation status is `NOT_QUERYABLE` with a typed reason. Proposed reasons are `NO_RESEARCH_TIMELINE`, `NO_CANDIDATE_ENTRY` and `NO_RECORDED_ISSUANCE`. The issuance part states that the authority records none for the strategy, listing and date at the read, never "not eligible".
5. Never: a policy-set parameter, a catalog read, a global disposition browse, a scan of dispositions or attempts, or a guessed current or latest version. `NOT_ELIGIBLE` and `UNRESOLVED` are shown only when their exact dispositions are authoritatively discoverable; in EI-7C none is, so they are not shown. `NOT_QUERYABLE` never implies that no generation was attempted (EI7-FC2).
6. Persisted `NO_SIGNAL` and `REJECTED` entries are shown with their authoritative stored reasons (EI7-FC2).

| Status | Kind | Where it appears in EI-7C |
| --- | --- | --- |
| UNKNOWN | Research timeline | No timeline is stored for the strategy, listing and date |
| `NOT_QUERYABLE` | Generation | No issuance-linked exact key exists |
| `PROPOSED` | Generation | The issuance-linked disposition verifies |
| `ISSUED` | Issuance | The authority records an issuance, whose identity is shown |
| `NOT_ELIGIBLE`, `UNRESOLVED` | Generation | Not shown in EI-7C, because no exact disposition is authoritatively discoverable. The distinction stays intact in the Application types for a separately governed lookup |

**Limitation (EI7-FC2):** full unissued-generation history is not observable in EI-7C.

> **Note, 2026-11-28.** Step 1's pending route encoding is settled by EI7-CF3-F: the `{strategy}` segment is the canonical base64url encoding of the identity's UTF-8 text ([section 24.5](#245-strategy-identity-encoding)). A permanent SQLite storage failure met by the research query maps to `PERSISTED_AUTHORITY_INVALID` under EI7-SI1 ([section 24.10](#2410-storage-engine-error-compatibility-contract)); step 3's availability rule covers only the dependency-unavailable conditions of that section.

### 23.6 EI-7C use cases and ports

| Use case (proposed) | Reader dependencies |
| --- | --- |
| `ListEquitySessionRecommendationsUseCase` | `EquityRecordedSessionIssuanceReader`, `EquityRecommendationLifecycleReader` |
| `ListEquityRecordedSessionIssuancesUseCase` | `EquityRecordedSessionIssuanceReader`, `EquityRecommendationLifecycleReader` |
| `GetEquityRecommendationDetailUseCase` | `EquityRecommendationLifecycleReader`, `EquityRecommendationGenerationRecordReader`; for panels, `EquityCandleEvidenceSnapshotReader` and `EquityIntradayResearchTimelineReader` |
| `GetEquityResearchTimelineUseCase` | `EquityIntradayResearchTimelineReader`, `EquityCandleEvidenceSnapshotReader`, `EquityRecommendationIssuanceReader`, `EquityRecommendationGenerationRecordReader` |
| `GetEquityEvidenceSnapshotUseCase` | Deferred with the snapshot endpoint |

`/equity/health` is one read-only check by the equity runtime (EI-7E, subject to EI7-CF5).

> **Note, 2026-11-28.** EI7-CF5 is approved; the health semantics are in [section 24.8](#248-health-semantics). EI7-SI1 adds one Application-owned storage-integrity contract, delivered by SI-A ([section 24.10](#2410-storage-engine-error-compatibility-contract)).

- **New ports:** `EquityRecordedSessionIssuanceReader`, `EquityIntradayResearchTimelineReader` and `EquityCandleEvidenceSnapshotReader`. Each is read-only and separate from the existing ports.
- **Reused ports:** `EquityRecommendationLifecycleReader`, `EquityRecommendationIssuanceReader` and `EquityRecommendationGenerationRecordReader`.
- **Deferred port:** `EquitySyntheticPolicyCatalogReader`.

**Architecture tests (AR):**

- no use case depends on a writer, recorder, issuer, monitor, clock, write-capable store or policy catalog store;
- the EI-6C monitor does not depend on the recorded-session port;
- equity Application modules import no Infrastructure.

**EI7-P4 authorization (EI7-FC6).** "The narrow P4 compatibility change is eligible for implementation within a separately owner-authorized EI-7C Application slice. It is not authorized by EI-7B Docs PR-1."

**EI7-P4 compatibility tests** (written within an authorized EI-7C):

- **compliant implementations:** every existing valid writer-capable snapshot store still passes the rehydration check and yields an identical rehydrated timeline; an object exposing only `get_snapshot` is accepted structurally;
- **non-compliant implementations:** an object without `get_snapshot` is still refused with `TypeError`. A `runtime_checkable` Protocol checks method presence only, and that check alone does not prove runtime behaviour. So the refusal, and the behaviour of an object whose `get_snapshot` returns the wrong type or raises, must be tested explicitly;
- **preserved semantics:** rehydration outcomes, errors and fingerprints are identical whether a reader-only object or a writer-capable store is supplied;
- **no write surface:** the reader-only Protocol declares no write method, and no class gains a fake or refusing `issue()`;
- **regression:** the existing EI-3C tests stay unchanged and green.

### 23.7 EI-7D read-only adapters

**Adapters:**

- a read-only issuance reader, implementing the existing issuance reader and the new recorded-session port;
- read-only lifecycle, generation-record, research-timeline and snapshot readers.

**Requirements:**

- `mode=ro`, so a missing file is never created;
- `PRAGMA query_only = ON`, confirmed;
- no DDL or schema initialization;
- one short read transaction per call;
- reuse of the existing verified decoders;
- missing tables reported as typed unavailability;
- a hot journal reported as typed unavailability (EI-6C L8).

**Tests:**

- equivalence with the existing adapters' reads on the same files;
- file bytes unchanged after reads (EI7-N17);
- reads succeed on a read-only file;
- typed unavailability and typed corruption;
- watermark binding above 2^63−1 refused before SQLite;
- fresh-process restart reads.

**Out of scope:**

- a policy catalog adapter;
- any change to the existing adapters, the schemas or the write paths.

> **Note, 2026-11-28.** Merged as northstar-infrastructure #70 (`caa2c46`). As merged, every SQLite engine failure, `SQLITE_CORRUPT` and `SQLITE_NOTADB` included, is the reader family's unavailability error, and WAL-mode files are refused before SQLite opens them. EI7-SI1 assigns permanent storage failures to a new Application-owned contract, applied to these adapters in SI-B only ([section 24.10](#2410-storage-engine-error-compatibility-contract)); the existing adapters stay unchanged.

### 23.8 Acceptance reconciliation

**Revised expected behaviours (2026-11-21).** All remain **PLANNED — NOT TESTED**.

| ID | Revised expected behaviour |
| --- | --- |
| EI7-N03 | `NOT_ELIGIBLE` and `UNRESOLVED` are never inferred or shown without an authorized exact lookup. Without an issuance-linked key the generation status is `NOT_QUERYABLE` with a typed reason, never issued. The two dispositions stay distinct in the Application types |
| EI7-N04 | No stored timeline gives a research timeline of UNKNOWN; no exact identity gives a generation status of `NOT_QUERYABLE`. Neither is "none" or zero, and the two are never conflated |
| EI7-N07 | An issuance recorded before a covering declaration is in the recorded view with `covered_by_declaration: false` and absent from the declared view. An undeclared session answers HTTP 200 with `declaration: NONE` and states N and W |
| EI7-N10 | A missing cited snapshot is an `unavailable` panel while the required facts are verified. The snapshot endpoint is deferred |
| EI7-N11 | Covers each comparison 1 to 10 of register section 13.4, and an unverifiable primary research timeline. Each fails the whole response with HTTP 500, never as an empty success, UNKNOWN or a panel |
| EI7-N14 | A retrieved non-SYNTHETIC snapshot refuses the affected response with HTTP 409 `EVIDENCE_CLASS_UNSUPPORTED`. Its content is not exposed and it is never relabelled. The refusal is distinct from corruption (500) and from absence (panel) |
| EI7-N15 | W > N gives 409 `WATERMARK_AHEAD`. A valid W above 2^63−1 gives 409 before any SQLite bind, and no `OverflowError` escapes. Signed, leading-zero, exponent or other malformed values give 422 `UNSUPPORTED_SCOPE`. Positions and watermarks are integer strings |
| EI7-N16 | The recorded view is in ascending `recording_sequence` and the declared view in ascending `(issued_at, issuance_id)`, using a fixture where the two orders differ. 50 issuances succeed; 51 are refused with 409 and no partial list. LEDGER corruption precedes overflow |

**Required coverage.** Every implementation slice must evidence these items.

| Coverage item | Scenarios | Layers | Slices |
| --- | --- | --- | --- |
| Sequence order of the recorded view versus the preserved declared-view order | N16, N07 | PC, IS, AS | C, D, E |
| 50 versus 51 recorded issuances; no partial list | N16 | PC, IS, AS | C, D, E |
| LEDGER failure precedence over overflow | N16, N11 | PC, IS | C, D |
| Overflow returned before an individual row's corruption is found (accepted limitation); never an unverified success | N16, N11 | IS | D |
| Declaration projected at the same watermark as the issuances | N07, N15 | PC, IS, AS | C, D, E |
| Undeclared session with an exact watermark and position | N07 | PC, IS, AS, WC | C, D, E, F |
| Integer strings beyond the JavaScript safe range (positions, watermarks, sequences, quantities); no JSON numbers | N15, N20 | AS, WC | E, F |
| Watermark above the SQLite int64 range | N15 | PC, IS, AS | C, D, E |
| Research rehydration through reader-only ports (EI7-P4 compatibility tests) | N01, N02, N17 | AU, PC, AR | C |
| Corrupt or unverifiable primary research timeline | N11 | AU, IS, AS | C, D, E |
| Optional evidence-panel degradation: missing, corrupt, reader unavailable | N10, N12 | AU, AS, WC | C, E, F |
| Non-SYNTHETIC snapshot refusing the entire affected response | N14 | AU, IS, AS | C, D, E |
| Exact generation lineage, including version and evaluator mismatch | N11, N24 | AU, PC, IS | C, D |
| Missing generation identity gives `NOT_QUERYABLE` | N03, N04 | AU, AS, WC | C, E, F |
| No inference of `NOT_ELIGIBLE` or `UNRESOLVED` without source evidence | N03 | AU, AS | C, E |
| No read-side write, DDL, scheduling, monitoring or current-clock dependency | N17 | IS, AS, AR, XP | C, D, E |
| Restart and replay consistency at a pinned watermark | N20, N21 | IS, AS, XP | D, E, H |
| No EI-8 performance reporting | N23 | AS, WC, AR | E, F |
| Evidence-class gating before research outcome presentation: a non-SYNTHETIC snapshot gives 409, and the evaluator-derived `REJECTED` (`EVIDENCE_NOT_SYNTHETIC`) is never presented (EI7-FC3) | N14, N02 | AU, PC, AS | C, E |
| Missing or inconsistent stored research configuration fails the whole research response; no current or client configuration is substituted (EI7-FC3) | N11 | AU, IS, AS | C, D, E |
| Persisted `NO_SIGNAL` and `REJECTED` reasons shown as stored; `NOT_QUERYABLE` never shown as "no attempts" (EI7-FC2) | N01, N02, N03 | AU, AS, WC | C, E, F |
| EI7-P4 compliant and non-compliant implementations, structural acceptance and preserved rehydration semantics (EI7-FC6) | N01, N02 | AU, PC, AR | C |
| Overflow refuses both projections; EI-6A `session_view` unchanged; overflow not reported as corruption (EI7-FC1) | N16, N11 | PC, IS, AS | C, D, E |
| Top-level `contract_id`; `EQUITY_NOT_CONFIGURED` 503 not retryable; `recording_position` and `recording_watermark` as strings (EI7-FC4) | N12, N15 | AS | E |
| Optional per-item lifecycle summaries are independent reads, never presented as atomic with W; each failure is an explicit item status | N21, N11 | PC, IS, AS | C, D, E |

No item is VERIFIED or PASS. Evidence is recorded only from executed tests, with independent Linux CI, in the acceptance record of the slice that runs them.

### 23.9 Sequencing

| Step | Repository | Precondition | Status |
| --- | --- | --- | --- |
| EI-7B Docs PR-1 (this record) | northstar-docs | Owner review | Authorized for documentation only; uncommitted |
| EI-7C | northstar-application | Docs PR-1 merged; separate owner authorization | **NOT AUTHORIZED** |
| EI-7D | northstar-infrastructure | EI-7C merged; separate authorization | **NOT AUTHORIZED** |
| EI-7E | northstar-api | EI-7C and EI-7D merged; EI7-CF3 and EI7-CF5 approved; separate authorization | **NOT AUTHORIZED** |
| EI-7F | northstar-web | EI7-CF6 approved; built against the frozen contract; merged after EI-7E | **NOT AUTHORIZED** |
| EI-7G | northstar-api; northstar-infrastructure | EI7-CF7 approved under separate governance | **NOT AUTHORIZED** |
| EI-7H | All | EI-7E and EI-7F, and EI-7G for EI7-N22 | **NOT AUTHORIZED** |

CI checks out sibling repositories at `develop`, so each slice merges only after the slices it depends on.

> **Note, 2026-11-28.** The table above records the state on 2026-11-21. The governing sequence is [section 24.12](#2412-dependency-ordering-and-gates).

### 23.10 Final owner confirmations EI7-FC1 to EI7-FC6

Confirmed by the owner on **2026-11-21** and recorded in [register section 13.12](../product/Equity-Intraday-V1-EI7-Decision-Register.md#1312-final-owner-confirmations-ei7-fc1-to-ei7-fc6-2026-11-21), which governs. The design effect:

| ID | Design effect | Sections |
| --- | --- | --- |
| EI7-FC1 | Above 50 recorded issuances, both EI-7 session projections refuse with 409; the bound is an EI-7 projection limit, not an authority limit; EI-6A `session_view` and its order are unchanged; overflow is not a corruption verdict | 23.2, 23.8 |
| EI7-FC2 | Research reasons shown as stored; `NOT_QUERYABLE` without exact identity; no inference about attempts; `NOT_ELIGIBLE` and `UNRESOLVED` only when exactly discoverable; six distinct statuses; full unissued-generation history not observable in EI-7C | 23.5, 23.8 |
| EI7-FC3 | Stored configuration only, verified, with typed whole-response failure; evidence-class gate before any research outcome is used or presented; no payload leak or relabelling; panel degradation only with required provenance intact | 23.4, 23.5, 23.8 |
| EI7-FC4 | `EQUITY_NOT_CONFIGURED` 503, not retryable; top-level `contract_id: "northstar.equity-private-read/1"`; `recording_position` = N; `recording_watermark` = W; both canonical decimal strings; no JSON numbers | 23.1, 23.3 |
| EI7-FC5 | Sources are the owner's explicit decisions, the feasibility audit, the merged EI-7B governance and verified source; the CF contract package is not present locally and was not opened or committed | 23 (introduction) |
| EI7-FC6 | P4 reader-only Protocol: eligible within a separately owner-authorized EI-7C; not authorized by Docs PR-1; no fake `issue()`; compliant, non-compliant, structural and semantic tests | 18 (note), 23.6 |

**List and detail consistency.** No section of this design promises an atomic lifecycle snapshot across listed issuances:

- section 11.3 and section 13 already describe per-item lifecycles as later, separate reads;
- section 23.2 makes discovery authoritative only for issuance identity and session recording facts at W, and makes the per-item summaries optional, independent, non-atomic and individually statused;
- the detail read (section 23.4) keeps the EI-6A and EI-6B single-transaction issuance and lifecycle semantics.

## 24. Approved private access, runtime and storage-integrity design (2026-11-28)

Added by the EI-7E governance documentation PR. Sections 1 to 23 are not rewritten; dated notes there point here. The decisions are recorded in [register section 14](../product/Equity-Intraday-V1-EI7-Decision-Register.md#14-approved-contracts-ei7-cf3-ei7-cf5-ei7-si1-and-ei7-si2-2026-11-28), which governs; this section gives the design mechanics. Everything approved in register section 13 and design section 23 stands, in particular: SYNTHETIC-only exposure; MODELED-only outcomes; the recorded view in `recording_sequence` order and the EI-6A declared-view order; one watermark W for both views; N and W as canonical decimal strings; at most 50 recorded issuances, no partial success and no pagination; no guessed generation outcome; required lineage verification; the verified lifecycle; stored research rehydration; non-SYNTHETIC refusal; canonical UTC timestamps on the wire and IST with UTC in the Web; no executed-trade or realized profit-and-loss claim; no write from a GET; and no route family beyond the five. **Nothing here is implemented or authorized.** Source facts are those verified on 2026-11-28 at the commits listed in the status note; EI-7E re-verifies them before it changes anything.

### 24.1 Private access topology

```text
Owner browser
   │ SSH tunnel to the host
   ▼
Caddy private gateway  (host loopback publication only; basic auth; default-deny allow-list)
   │ internal network; no published API port
   ▼
Northstar API  (Equity routes: five GET families; read-only Equity runtime)
   │ Application EI-7C query use cases
   ▼
Infrastructure EI-7D read-only adapters  (mode=ro; query_only confirmed)
   │
   ▼
Equity SQLite file  (separate from the Futures file; rollback journal)
```

| Requirement (EI7-CF3-A, EI7-CF3-B) | Design rule |
| --- | --- |
| No bypass of the gateway | The API listener is reachable only from the gateway. A route, listener or composition that reaches an Equity route without passing the approved gateway fails private-access acceptance; direct external reachability of the backend is a deployment blocker |
| Loopback-only exposure | Only the gateway is published, and only on host loopback. The recommended backend listener is `127.0.0.1` |
| Futures unchanged | Existing Futures binding, port and composition are re-verified before EI-7E and never changed silently |
| No public service | No subscriber, public or multi-user service; no in-application bearer token |

**Source facts at `7ad37f5` (northstar-api), to be re-verified by EI-7E:**

| Fact | Source | Consequence for EI-7E |
| --- | --- | --- |
| The Indian stack publishes only Caddy, on `127.0.0.1:${NORTHSTAR_INDIA_WEB_PORT:-8081}`; the API service publishes no port | `deploy/india/compose.yaml` | This is the approved shape: the gateway on host loopback, the API unpublished |
| The API entry point hard-codes `uvicorn` on `0.0.0.0:8000`; in the Indian stack that address is inside the API container, reached by Caddy as `india-api:8000` | `northstar_api/__init__.py`; `deploy/india/Caddyfile` | A literal loopback bind inside that container would cut Caddy off. How the loopback model applies to the container topology, and the default of the non-container entry point, are technical closure items, settled without a silent Futures change |
| The Indian Caddy allows GET and HEAD on three exact Futures paths and answers every other `/api/*` path with 404 | `deploy/india/Caddyfile` | The precedent for EI7-CF3-D; Equity paths are added only by a separately authorized deployment change |
| The CME stack's Caddy publishes ports 80 and 443 and reverse-proxies every `/api/*` path to the same API image | `deploy/Caddyfile`; `deploy/compose.yaml` | Equity routes added to the shared application would be reachable there without the SSH tunnel and without a default-deny allow-list. EI-7E must prevent this before any Equity route exists in a deployable image (section 24.15) |

> **Note, 2026-11-28 (technical closure).** These facts were re-verified by the EI-7E technical-closure audit at northstar-api `7ad37f5`. EI7E-D01 keeps Equity routes out of the application that the CME stack serves ([section 25.2](#252-cme-isolation)). EI7E-D03 sets the India direction: the dedicated Equity process shares the India Caddy network namespace and binds `127.0.0.1`, subject to deployment verification ([section 25.3](#253-india-private-network-topology)). The Futures listener is unchanged. Nothing is deployed.

### 24.2 Authentication boundary

- Caddy basic authentication protects every private route, with a hashed credential supplied through the deployment environment, as the Futures stacks do today (`NORTHSTAR_BASIC_AUTH_USER`, `NORTHSTAR_BASIC_AUTH_HASH`). No credential value appears in any repository, log or response body.
- Basic authentication is used only over the SSH tunnel. Basic authentication over unprotected public HTTP is not acceptable.
- The application performs no authentication of its own (R-5). "Private" therefore means deployed behind this boundary and nothing more; an API process run outside it is unprotected (register section 14.8).

### 24.3 Method and route restrictions

| Family | Path | Query parameters | Methods |
| --- | --- | --- | --- |
| Health | `/equity/health` | None | GET |
| Declared session view | `/equity/sessions/{exchange}/{date}/recommendations` | Optional `watermark`, a canonical decimal string (register section 13.2, items 5 and 6; EI7-P7) | GET |
| Recorded session view | `/equity/sessions/{exchange}/{date}/recorded-issuances` | Optional `watermark`, as above | GET |
| Verified detail | `/equity/recommendations/{issuance_id}` | None | GET |
| Research | `/equity/research/{strategy}/{exchange}/{symbol}/{date}`; `{strategy}` encoded as in section 24.5 | None | GET |

- **Default deny.** At both the gateway and the application, every other path and method is refused. The gateway lists each route template by an anchored pattern with restricted segment character classes, never a prefix wildcard that could expose a future route. No snapshot, separate lifecycle, lookup, generic browse or write route exists.
- **HEAD.** Permitted only where EI-7E explicitly supports it and its tests prove it non-mutating and equivalent to the GET without a body; otherwise refused at both layers.
- **Paths.** Alternate slashes, trailing slashes, dot segments and percent-encoded separators never reach a different handler, and no redirect names the internal listener or leaves the gateway path space. The exact trailing-slash behaviour is verified in EI-7E (section 24.13).

> **Note, 2026-11-28 (technical closure).** GET only in V1. HEAD is not enabled without a separate, verified approval; FastAPI answers it with 405 on a GET route. The dedicated application disables trailing-slash redirects ([section 25.1](#251-dedicated-equity-api-composition)). The treatment of unknown query parameters stays an E1 closure item, with no new public code.

### 24.4 Documentation endpoints

`/docs`, `/redoc` and `/openapi.json` are disabled or unreachable **at the application boundary**; a gateway rule alone is insufficient (EI7-CF3-E). Today FastAPI is constructed with its default documentation routes (`northstar_api/app.py`). The OpenAPI document may still be generated in-process by tests, for the allow-list comparison, without being served. Whether the routes are disabled for the whole process, which Futures would observe, or only for an Equity-only application is a technical closure item; either way the Futures effect must be explicit and tested.

> **Note, 2026-11-28 (technical closure).** Settled by EI7E-D01. The documentation routes are disabled in the dedicated Equity application only. The existing Futures application and its documentation routes are unchanged.

### 24.5 Strategy identity encoding

`StrategyIdentity` is opaque text (Core `northstar_core/strategy/value_objects/strategy_identity.py`): any string that is non-empty after Core strips surrounding whitespace, `/` included. It is not restricted further (EI7-CF3-F).

| Step | Rule | On failure |
| --- | --- | --- |
| Encode | The route segment is the base64url encoding (RFC 4648, section 5: `A`–`Z`, `a`–`z`, `0`–`9`, `-`, `_`) of the identity's UTF-8 bytes, as one path segment | — |
| 1 | The segment uses only the base64url alphabet, and padding only as the padding closure item decides | HTTP 422 `UNSUPPORTED_SCOPE` |
| 2 | It decodes as base64url | 422 |
| 3 | Re-encoding the decoded bytes gives exactly the segment: no alternate padding, no non-zero trailing bits, no standard-alphabet characters, so one identity has exactly one segment | 422 |
| 4 | The bytes decode as strict UTF-8 | 422 |
| 5 | `StrategyIdentity(text)` succeeds **and** its `identity` equals `text`. Core strips surrounding whitespace, so without this check two segments could name one identity | 422 |

- The decoded `StrategyIdentity` is the only identity used; there is no second identity authority, table or alias.
- Errors use the approved envelope and never echo the raw input (register section 13.3, item 5).
- Technical closure items: padding (with or without `=`), a maximum segment length, and whether responses echo the decoded identity, the canonical segment or both.
- Web deep links for research (EI7-CF6, still pending) would carry the same canonical segment; that is for EI-7F under EI7-CF6.

> **Note, 2026-11-28 (technical closure).** The closure items above are settled by EI7E-D07A ([section 25.7](#257-canonical-strategy-identity-encoding)):
> - **unpadded** base64url, with `=` rejected;
> - strict canonical re-encoding;
> - no Application-defined maximum length, since only transport limits apply and they are verified, never truncating;
> - responses carry the decoded canonical identity from the Application view, never the segment in its place.

### 24.6 Same-origin browser access

- The browser reaches the API only through the authenticated same-origin gateway, as the Futures dashboard does through `/api/*` today.
- No wildcard CORS. The existing CORS middleware allows `GET` from one configured origin (`NORTHSTAR_WEB_ORIGIN`; `northstar_api/app.py`) and is not widened for Equity.
- The browser never needs, or is told, the backend's internal listener address.

### 24.7 Equity database configuration and read-only startup

| Rule | Design |
| --- | --- |
| Separate authority (EI7-CF5-A) | One explicitly configured Equity SQLite file, distinct from the Futures file (`NORTHSTAR_DATABASE`). No fallback to the Futures file, no combined schema or migration, no discovery of databases by search or default path |
| Setting (EI7-CF5-B) | `NORTHSTAR_EQUITY_SQLITE_PATH` (proposed; conflicts with no existing setting; follows the per-area `NORTHSTAR_<AREA>_` convention of the `NORTHSTAR_FUTURES_` settings). Its spelling is confirmed in EI-7E |
| Construction (EI7-CF5-C) | Building the Equity runtime and its read-only adapters opens no database, creates no file, initializes no schema, applies no migration, issues nothing, records no lifecycle event, triggers no monitoring, writes no evidence and attempts no journal recovery. The EI-7D adapters already hold only a path at construction |
| Unconfigured (EI7-CF5-D) | No setting: every Equity request, health included, is refused with `EQUITY_NOT_CONFIGURED`, HTTP 503, `retryable: false`, with top-level `contract_id: "northstar.equity-private-read/1"`. Never an empty but healthy database, never a writable fallback |
| Configured but absent file | The readers report dependency unavailability (no file is created; EI-7D `mode=ro`) |
| Futures (EI7-CF5-H) | Futures settings and startup are unchanged. The Futures runtime initializes its own schema at startup (`build_database_runtime`, `northstar_api/runtime.py`), so the API process is not described as write-free; the Equity read stack is read-only, and Equity code never touches the Futures file |

> **Note, 2026-11-28 (technical closure).** Settled by EI7E-D04 ([section 25.4](#254-equity-database-isolation)):
> - the spelling `NORTHSTAR_EQUITY_SQLITE_PATH` is confirmed;
> - an unset or whitespace-only value is not configured;
> - relative paths are refused and Futures aliasing is prevented (E1 obligations);
> - the Equity storage is dedicated and mounted read-only, with no Futures volume or setting in the Equity container.
>
> Under EI7E-D01, the Equity process is separate and does not build the Futures runtime.

### 24.8 Health semantics

`GET /equity/health` is one narrowly scoped, non-mutating dependency probe through the same strictly read-only connection path as the readers (EI7-CF5-E).

| Outcome | Meaning | Response |
| --- | --- | --- |
| Readable | Configured, and the probe read the Equity file safely | HTTP 200 |
| Not configured | No Equity setting | 503 `EQUITY_NOT_CONFIGURED`, not retryable |
| Dependency unavailable | Missing file, missing tables, lock beyond the busy timeout, hot journal, WAL-mode refusal or any other unavailability | 503 `DEPENDENCY_UNAVAILABLE`, retryable |
| Integrity failure, where detectable | The probe's own read meets a permanent storage-integrity failure (EI7-SI1) | 500 `PERSISTED_AUTHORITY_INVALID`, not retryable |

- **Claims a success must not make (EI7-SI2):** that every database page is valid; that every recommendation is verified; that every lifecycle has been audited; that the Equity universe is complete; that broker or market data is available. A healthy probe is a healthy read dependency, not an integrity certification.
- No writer-driven schema initialization; no unrelated application metric; no Futures dependency; no path or SQLite text in the body.
- The probe's location, its exact read and its body fields are technical closure items of EI-7E. Equity route modules import no Infrastructure type (register section 13.3, item 2); if the probe would need an Application port beyond SI-A, EI-7E stops (section 24.15).

> **Note, 2026-11-28 (technical closure).** Settled by EI7E-D02 ([section 25.5](#255-read-only-health-probe)):
> - the probe is an Application-owned read-only port (P1), implemented by an Infrastructure SQLite adapter (P2) through the EI-7D read-only transaction, and consumed by the dedicated Equity runtime (E1);
> - it checks `query_only` and the required tables, and scans no data;
> - a readable probe answers 200 with `{"contract_id": "northstar.equity-private-read/1", "status": "READABLE"}`.
>
> The API runs no SQL for health.

### 24.9 Journal mode, WAL refusal and hot journal

| Condition | Behaviour (as merged in EI-7D unless stated) | Classification |
| --- | --- | --- |
| Rollback journal (`delete`), no hot journal | Served; database bytes, schema and journal unchanged; no auxiliary file created | — |
| Hot rollback journal | SQLite refuses the read-only connection (`SQLITE_READONLY_ROLLBACK`); the database and journal files are left untouched. Recovery is outside every GET path, by an appropriately authorized existing write workflow (EI7-CF5-G) | Dependency unavailable |
| WAL-mode file | Refused after reading the 100-byte header (bytes 18 and 19) and before SQLite opens the file, because any SQLite read of a WAL database creates `-wal` and `-shm` files | Dependency unavailable |
| Residual race | The header check and the open are two steps; a mode transition between them is not eliminated by the check alone (EI7-CF5-F) | Accepted limitation |

- **Required before API exposure:** deployment validation of the journal mode of the configured file, and operating controls that keep it in the rollback journal. Their form is a technical closure item.
- **Not authorized:** WAL support, `immutable=1`, automatic recovery, any read-write fallback, and any claim of unconditional filesystem non-mutation across unsupported modes.

> **Note, 2026-11-28 (technical closure).** The rules above stand.
> - EI7E-D04 requires a read-only mount of the dedicated Equity storage once deployed. It is expected to mitigate, not eliminate, the check-to-open race; V1 verifies that.
> - EI7E-D05 prohibits operational exposure until database provisioning and **recovery**, including the owner of hot-journal recovery, are authorized and verified. No Equity recovery workflow is authorized today.
> - Journal-mode validation and controls stay D1 and V1 obligations ([section 25.10](#2510-deployment-prerequisites)).

### 24.10 Storage-engine error compatibility contract

EI7-SI1 introduces one Application-owned, typed **permanent storage-integrity error** (tentative name `EquityReadStorageIntegrityError`; finalized in SI-A). It represents a permanent failure of the storage read boundary and never asserts that a particular issuance, lifecycle or snapshot exists.

| SQLite condition | Detected by (SI-B) | Classification | EI-7C code | HTTP | Retryable |
| --- | --- | --- | --- | --- | --- |
| `SQLITE_CORRUPT`, any extended code | Primary result code | Permanent storage integrity | `PERSISTED_AUTHORITY_INVALID` | 500 | No |
| `SQLITE_NOTADB` | Primary result code | Permanent unusable storage (possibly a misconfigured path; the cause is not diagnosed) | `PERSISTED_AUTHORITY_INVALID` | 500 | No |
| `SQLITE_BUSY` | Primary result code | Dependency unavailable | `DEPENDENCY_UNAVAILABLE` | 503 | Yes |
| `SQLITE_LOCKED` | Primary result code | Dependency unavailable | `DEPENDENCY_UNAVAILABLE` | 503 | Yes |
| `SQLITE_CANTOPEN` | Primary result code | Dependency unavailable | `DEPENDENCY_UNAVAILABLE` | 503 | Yes |
| `SQLITE_READONLY_ROLLBACK` | Extended result code | Dependency unavailable | `DEPENDENCY_UNAVAILABLE` | 503 | Yes |
| Missing required tables | Unchanged default: every other SQLite failure | Dependency unavailable (approved initial contract) | `DEPENDENCY_UNAVAILABLE` | 503 | Yes |

- **Detection.** By SQLite result codes, including extended codes (in Python, the error's `sqlite_errorcode`), never by message text. The accepted precedent is the EI-6C snapshot catalog's classification of `SQLITE_CORRUPT` and `SQLITE_NOTADB` from the primary code (`sqlite_equity_candle_evidence_snapshot_catalog.py`, `_failure`). The original exception is kept as the cause for internal diagnosis; the public message is fixed and carries no SQLite text, path or stored payload.
- **SI-A (northstar-application):** defines the typed contract; EI-7C maps it to `PERSISTED_AUTHORITY_INVALID` on the session, detail and research queries, and handles it on the optional reads (evidence panels, per-item lifecycle summaries) so that it is never presented as `unavailable` or retryable. No Infrastructure change.
- **SI-B (northstar-infrastructure):** the EI-7D read-only adapters raise the contract for the permanent conditions above and keep the family unavailability errors for the others. No Application change. The existing EI-6A, EI-6B, EI-4D and older write-side adapters are unchanged; no existing authority error enumeration is modified.
- **Unchanged:** the decoded-record integrity errors (ISSUANCE, DECLARATION, LEDGER; EVENT, HISTORY, WINDOW; the generation record error; the timeline and snapshot integrity errors) and their mappings.

### 24.11 Integrity verification coverage

- Each operation verifies the authoritative data it reads, under its accepted contract: the whole-ledger issuance check on every issuance read (EI7-P6, unchanged and not weakened), issuance re-derivation, lifecycle replay, generation-record decoding, research rehydration and snapshot verification.
- Corruption on pages no query of the request reads is not detected. No full `PRAGMA integrity_check` runs on a GET; no background, scheduled, startup or automatic integrity checking or repair exists. Any future offline or periodic inspection needs separate governance (EI7-SI2).
- **Three distinct claims, never conflated** in responses, logs or user-interface language:

| Claim | Established by |
| --- | --- |
| A verified query result | The decoded, re-derived and verified data of that response |
| A healthy read dependency | A successful `/equity/health` probe |
| Database-wide integrity | Nothing in EI-7; no response or page may assert it |

### 24.12 Dependency ordering and gates

```text
Stage 1  EI-7E governance docs (this PR) ─▶ Stage 2  SI-A (application) ─▶ Stage 3  SI-B (infrastructure)
                                                                                   │
                                                     Stage 4  EI-7E (api) ◀────────┘  (SI-A and SI-B merged and verified)
                                                                   │
                                     Stage 5  EI-7F (web; EI7-CF6 approval) ◀──┘
                                     Stage 6  EI-7G (operator; EI7-CF7) and EI-7H (end to end)
```

| Stage | Slice | Gate | Status |
| --- | --- | --- | --- |
| 1 | This documentation PR | Owner review | Authorized for documentation only |
| 2 | SI-A | Separate authorization; Application tests for the contract and every query's handling | **NOT AUTHORIZED** |
| 3 | SI-B | SI-A merged; separate authorization; real-SQLite tests of section 24.13 | **NOT AUTHORIZED** |
| 4 | EI-7E | SI-A and SI-B merged and compatibility verified; separate authorization | **NOT AUTHORIZED** |
| 5 | EI-7F | EI7-CF6 approved; separate authorization | **NOT AUTHORIZED** |
| 6 | EI-7G; EI-7H | Separate governance and authorization | **NOT AUTHORIZED**; EI-7H not accepted |

CI checks out sibling repositories at `develop`, so each stage merges only after the stages it depends on.

### 24.13 Security and failure tests

All **PLANNED — NOT TESTED**. Each is evidenced only by executed tests in the slice named.

| Test | Expected | Slice; layer |
| --- | --- | --- |
| Unauthenticated request through the gateway | Refused by Caddy before the API | EI-7E deployment verification; EI-7H |
| Request bypassing Caddy | The backend is unreachable from outside the host and from any path but the gateway | EI-7E deployment verification; EI-7H |
| Unapproved method (POST, PUT, PATCH, DELETE, OPTIONS; HEAD unless supported) | Refused at the gateway and at the application | EI-7E; AS, AR |
| Unapproved path, including the deferred lookup, lifecycle and snapshot routes and any `/equity/*` not in the five families | 404 at both layers | EI-7E; AS, AR |
| Encoded-path bypass (`%2F`, double encoding, dot segments, encoded `?` or `#`) | Never reaches another handler or a broader allow-list rule | EI-7E; AS, deployment |
| Alternate and trailing slashes; redirects | No alternate spelling reaches another handler; no redirect exposes the internal listener or leaves the gateway path space | EI-7E; AS, deployment |
| `/docs`, `/redoc`, `/openapi.json` | Unreachable at the application boundary and the gateway | EI-7E; AS, deployment |
| Wrong `StrategyIdentity` encoding (padding variant, standard alphabet, non-canonical trailing bits, invalid UTF-8, surrounding whitespace, empty) | 422 `UNSUPPORTED_SCOPE`, input not echoed; a valid identity containing `/` round-trips | EI-7E; AS |
| Direct external connectivity | No published API port; the gateway published on host loopback only | EI-7E deployment verification |
| Credentials in logs or responses | None, on success and on every error path | EI-7E; AS, deployment |
| GET invoking a write | File bytes, schema and sidecar files unchanged across every route and error path; no writer, recorder, issuer or monitor composed | EI-7E; AS, AR, XP |
| Futures regression from Equity composition | Futures routes, settings, startup, binding, CORS and gateway rules unchanged except as explicitly authorized and tested | EI-7E; AS, AR |
| Unconfigured runtime | `EQUITY_NOT_CONFIGURED` 503, not retryable, on every Equity route and on health; no file created; no fallback | EI-7E; AS |
| Configured file missing; missing tables; lock; hot journal; WAL-mode file | 503 `DEPENDENCY_UNAVAILABLE`, retryable; nothing created or changed | SI-B (IS); EI-7E (AS) |
| `SQLITE_CORRUPT`, `SQLITE_NOTADB` | The storage-integrity contract; 500 `PERSISTED_AUTHORITY_INVALID`, not retryable, through the session, detail and research queries and in the optional reads; no raw SQLite text or path | SI-A (AU, PC); SI-B (IS); EI-7E (AS) |
| Writer-side compatibility | The EI-6A, EI-6B and EI-4D write adapters keep their accepted error behaviour | SI-B; IS |
| Read-only adapters' non-mutation after SI-B | The EI-7D non-mutation, restart and equivalence suites stay green | SI-B; IS, XP |
| Health language | No success response or log claims page validity, full verification, audit completeness, universe completeness or market-data availability | EI-7E; AS. EI-7F; WC |

### 24.14 Acceptance reconciliation

**Second reconciliation (2026-11-28).** No scenario is added, removed or renumbered; all 24 stay **PLANNED — NOT TESTED**. Section 23.8 stands. The merged EI-7C and EI-7D suites contain layer-level tests, but no acceptance record of either slice exists here, so no status changes.

| Scenario | Additional expected behaviour | Evidence owed by |
| --- | --- | --- |
| EI7-N11 Corrupt persisted authority | `SQLITE_CORRUPT` and `SQLITE_NOTADB` reach EI-7C as the permanent storage-integrity contract and fail the response with 500 `PERSISTED_AUTHORITY_INVALID`, not retryable; never retryable unavailability | SI-A (AU, PC); SI-B (IS); EI-7E (AS); EI-7H |
| EI7-N12 Dependency unavailable | Busy, locked, cannot-open, missing file, missing tables, hot journal and WAL refusal are 503, retryable; an unconfigured runtime is `EQUITY_NOT_CONFIGURED`, 503, not retryable; the Web offers Retry only for retryable errors | SI-B (IS); EI-7E (AS); EI-7F (WC) |
| EI7-N13 Unsupported scope | A non-canonical or invalid strategy segment is 422 `UNSUPPORTED_SCOPE` without input echo | EI-7E (AS) |
| EI7-N14 Non-SYNTHETIC refusal | Unchanged; evidenced through the API and Web as well | EI-7E (AS); EI-7F (WC); EI-7H |
| EI7-N15 Watermarks | Unchanged; historical watermark replay through the API, with N and W as strings | EI-7E (AS) |
| EI7-N17 GET and startup do not write | Constructing the Equity runtime opens nothing; an unconfigured runtime never falls back; health writes nothing; the Futures startup write stays confined to the Futures file; SI-B keeps the adapters' non-mutation | SI-B (IS); EI-7E (AS, AR, XP); EI-7H |
| EI7-N18 Private access enforcement | Every EI7-CF3 test of section 24.13: authentication, gateway bypass, methods, paths, encoded and alternate paths, redirects, documentation routes, direct connectivity, credentials, Futures regression | EI-7E (AS, AR, deployment verification); EI-7H |
| EI7-N19 Deep links | Unchanged; a research deep link uses the canonical strategy segment, subject to EI7-CF6 | EI-7F (WC) |
| EI7-N20 Restart-stable records | Unchanged; also through the API runtime from the Equity setting alone | EI-7E (XP); EI-7H |
| EI7-N23 No EI-8 performance metrics | Health carries no metric; no response, log or page claims execution, fills or realized results | EI-7E (AS); EI-7F (WC) |
| EI7-N24 Distinctions | Also: a verified query result, a healthy dependency and database-wide integrity are distinct claims in responses, logs and labels (EI7-SI2) | EI-7E (AS); EI-7F (WC) |

### 24.15 EI-7E implementation stop conditions

EI-7E stops and reports, rather than deciding, if:

1. SI-A or SI-B is not merged, or their compatibility is not verified;
2. any composition, listener or proxy would expose an Equity route outside the approved gateway, including through the CME stack's `/api/*` proxy, or the backend would be reachable directly from outside the host;
3. the loopback model cannot be met in the container topology without a Futures behaviour change that is not explicit, reviewed and tested;
4. the documentation routes cannot be disabled at the application boundary without such a Futures change;
5. the health probe would need a write, a schema initialization, a route family beyond the five, or an Application port beyond SI-A;
6. the Equity setting would fall back to, share, or discover the Futures file;
7. the `StrategyIdentity` encoding would need any restriction on valid identities;
8. any GET path would reach a writer, recorder, issuer, monitor or write-capable store;
9. a new public error code or a change to the approved HTTP vocabulary would be needed;
10. any deployment, Caddy, compose, bind or CORS change would be made without its own authorization.

> **Note, 2026-11-28 (technical closure).** Condition 1 is met: SI-A and SI-B are merged. Condition 5's "an Application port beyond SI-A" no longer stops EI-7E for the EI7E-D02 probe port, which P1 and P2 deliver before E1. A probe needing anything beyond EI7E-D02 still stops it. Every other condition stands. The per-slice conditions are in [section 25.14](#2514-stop-conditions).

### 24.16 Technical closure items

The same items as [register section 14.7](../product/Equity-Intraday-V1-EI7-Decision-Register.md#147-technical-closure-items), which governs. Each is settled within the approved contract and stated in its slice's PR:

- **SI-A:** the final name, module and type relations of the storage-integrity error; its presentation in optional reads (whole response or the panel's or item's `invalid` status; never `unavailable`).
- **SI-B:** the classification of EI-7D's pre-open refusals that carry no SQLite code (unreadable or unresolvable path; WAL-mode file), kept as dependency unavailability unless the owner decides otherwise.
- **EI-7E:** the setting's spelling and empty-value treatment; file location and a read-only mount; the health probe's location, read and body fields; the container application of the loopback model and the non-container default bind; process-wide or Equity-only documentation disabling; HEAD support; the treatment of unknown query parameters; base64url padding, maximum segment length and response echo; how the CME stack is kept from reaching Equity routes; deployment journal-mode validation and controls.

> **Note, 2026-11-28 (technical closure).** The state of each item is in the note to [register section 14.7](../product/Equity-Intraday-V1-EI7-Decision-Register.md#147-technical-closure-items).
> - **SI-A and SI-B items:** settled as merged.
> - **Settled by EI7E-D01 to EI7E-D07B:** the setting and empty value, and the read-only mount (D04); health (D02); the container loopback model (D03); documentation routes (D01); HEAD (not supported in V1); base64url (D07A); CME (D01).
> - **Still open:** unknown query parameters and the non-container default bind, both for E1; the Equity file location and journal-mode controls, for D1 and V1.

## 25. EI-7E approved technical closure architecture (2026-11-28)

Added by the EI-7E technical-closure governance documentation. Sections 1 to 24 are not rewritten; dated notes there point here. The decisions are recorded in [register section 15](../product/Equity-Intraday-V1-EI7-Decision-Register.md#15-ei-7e-technical-closure-decisions-d-01-to-d-07-2026-11-28), which governs; this section gives their interfaces, behaviour, dependencies and tests. Everything in register sections 13 and 14 and design sections 23 and 24 stands, in particular:

- **Routes:** exactly the five GET families of section 24.3, with the `watermark` parameter on the two session routes only.
- **Security:** default deny; GET only in V1; no wildcard CORS; no in-application bearer token; no basic authentication over unencrypted public connections.
- **Wire format:** top-level `contract_id`; no JSON numbers; canonical decimal strings; canonical UTC instants; the distinction between N and W; SYNTHETIC evidence and MODELED issuance only; no fill, P&L or broker activity.
- **Storage:** the EI-7D strictly read-only SQLite access; the SI-A and SI-B permanent error mapping; the rollback-journal-only model; WAL refusal; no repair on a GET; no full integrity scan per GET; no claim that unread corrupt pages are detected (EI7-SI2).
- **Data states:** `NONE` (not declared), `NOT_QUERYABLE` and `UNKNOWN` stay data states, never HTTP errors.

Module and type names below are proposals (register section 11). **Nothing here is implemented or authorized.**

### 25.1 Dedicated Equity API composition

EI7E-D01: one dedicated FastAPI application in its own process, beside, never inside, the existing Futures application.

| Component (proposed) | Responsibility | Must not |
| --- | --- | --- |
| Equity application factory | Builds the dedicated FastAPI application: the five GET routes, the Equity error handlers, the Equity runtime | Register on, mount into or import the existing `northstar_api.app`; serve `/docs`, `/redoc` or `/openapi.json` |
| Equity entry point | A separate console entry point that serves only the Equity application | Change the existing `northstar-api` entry point |
| Equity settings | Read `NORTHSTAR_EQUITY_SQLITE_PATH` only (EI7E-D04) | Read or require any Futures setting; fall back to `NORTHSTAR_DATABASE` |
| Equity runtime (composition root) | Construct the EI-7D read-only readers, the EI-7C query use cases and the P1 probe with its P2 adapter, opening nothing | Construct a writer, recorder, issuer, monitor, scheduler, write-capable store or the Futures `DatabaseRuntime` |
| Equity routes and serializers | Decode the request, call one Application use case, map the immutable view to the wire | Import any Infrastructure type, `sqlite3` or SQL; catch any Infrastructure error (register section 13.3, item 2) |
| Equity error handlers | Map Application refusals and runtime conditions to the approved envelope; the non-contract fallback (EI7E-D07B) | Serialize an exception, a cause, a traceback or request input |

**Construction of the dedicated application** (E1 obligations, verified against FastAPI 0.141.1 by the audit's probe):

- `docs_url`, `redoc_url` and `openapi_url` disabled: the documentation routes answer 404 at the application boundary. The OpenAPI document can still be generated in process by tests, for the allow-list comparison.
- `redirect_slashes` disabled, so a trailing slash is a 404, never a redirect. By default FastAPI answers a trailing slash with a 307 whose location is built from the `Host` header; behind the gateway's prefix stripping, that location leaves the `/api` path space.
- `debug` disabled.
- GET routes only. FastAPI does not add HEAD to a GET route, so HEAD, OPTIONS and every other method answer 405 at the application; HEAD stays unsupported in V1.
- No wildcard CORS. Same-origin access through the gateway needs none (EI7-CF3-G).
- A request-validation handler that never echoes input; FastAPI's default handler echoes it.

**Architecture tests (AR), E1:**
- no Equity path in the existing `create_app().openapi()`;
- no import from the Equity package of `northstar_api.app`, `northstar_api.routers` or `northstar_api.runtime`, each of which builds Futures objects at import or startup;
- no `sqlite3` import and no Infrastructure import in the Equity route and serializer modules;
- the EI4-D11 scan: no writer, recorder, issuer, monitor or write-capable store anywhere in the Equity composition.

### 25.2 CME isolation

| Layer | Mechanism | Evidence owed |
| --- | --- | --- |
| Application | The CME stack runs the existing `northstar-api` entry point, which serves only `northstar_api.app:app`; the Equity routes are absent from it by construction (EI7E-D01) | E1: AR test of `create_app().openapi()` and of imports |
| Gateway | The CME Caddyfile is unchanged; its `/api/*` proxy reaches only the existing application | The existing pin `test_the_cme_deployment_is_unchanged` stays green |
| Deployment | No CME compose, Caddy or command change | — |

Running the Equity entry point in the CME stack would be a deployment change, not authorized by any record.

### 25.3 India private network topology

EI7E-D03, the approved direction, subject to deployment verification:

```text
Owner browser
   │ SSH tunnel (ssh -N -L 8081:127.0.0.1:8081 …)
   ▼
host 127.0.0.1:8081 ──▶ india-web: Caddy :8080   (basic auth; anchored GET allow-list; default deny)
                          │                     │
                          │ project network     │ shared network namespace, loopback only
                          ▼                     ▼
                  india-api :8000         Equity API 127.0.0.1:<port>   (dedicated process)
                  (Futures, unchanged)          │ EI-7D read-only readers; P2 probe
                                                ▼
                                         Equity SQLite file  (dedicated storage, read-only mount)
```

- The Equity process joins the India Caddy container's network namespace (the equivalent of `network_mode: "service:india-web"`) and binds `127.0.0.1:<port>`. Only processes in that namespace (Caddy) can connect. `india-operations` and `india-api` share the project network with `india-web`, but not its loopback interface.
- `<port>` is chosen by D1 so that it does not conflict with Caddy's `:8080` or any other listener in the namespace, and it is verified in V1. This record fixes no port.
- Futures is unchanged: `india-api` keeps `0.0.0.0:8000` on the project network, and Caddy keeps reaching it as `india-api:8000`.
- Points D1 must design and V1 must verify:
  - lifecycle coupling: a recreated `india-web` takes the shared namespace with it;
  - the Equity process's startup ordering with respect to Caddy;
  - the gateway's anchored GET templates and default deny (section 24.3), with segment classes no wider than the approved syntax:
    - exchange: canonical `ExchangeCode` text;
    - date: `YYYY-MM-DD`;
    - issuance: `eiris1-sha256:` followed by 64 hexadecimal digits;
    - strategy: the base64url alphabet;
    - symbol: canonical `Symbol` text;
  - Caddy's case, decoding and dot-segment handling on the deployed version (the web image uses the floating `caddy:2-alpine` tag);
  - how the basic-authentication header is handled upstream, so that credentials never appear in API logs or responses;
  - that the API does not rely on forwarded headers for any decision: uvicorn trusts `X-Forwarded-*` from `127.0.0.1` by default, which is Caddy's address in the shared namespace.
- A container-local `0.0.0.0` listener for the Equity process does not satisfy EI7E-D03. If Compose or Docker Desktop cannot support the shared namespace, D1 stops; it does not fall back to another topology without an owner decision.

### 25.4 Equity database isolation

| Configuration state (EI7E-D04) | Behaviour |
| --- | --- |
| `NORTHSTAR_EQUITY_SQLITE_PATH` unset, empty or whitespace-only | Not configured: every Equity route, health included, answers 503 `EQUITY_NOT_CONFIGURED`, `retryable: false` |
| A relative path | Refused, never resolved against the working directory. The mechanism is fixed in E1, with no new public code |
| The same file as `NORTHSTAR_DATABASE`, when both are present (resolved path, or same file for existing files) | Refused, without opening or changing either file |
| An absolute path to a missing file, a missing table, a lock, a hot journal or a WAL-mode file | 503 `DEPENDENCY_UNAVAILABLE`, retryable, at request time; nothing is created (EI-7D) |
| A configured, readable file | Served by the EI-7D readers |

- Construction opens nothing: the EI-7D readers hold only a path until a call.
- Deployment (D1): dedicated Equity storage, mounted read-only in the Equity container. No Futures volume, no `NORTHSTAR_DATABASE` and no Futures secret is given to the Equity container. No writable fallback, schema creation, migration, monitoring, issuance or generation at startup.
- E1 acceptance obligations: validate the configuration without opening a writable database; cover missing, empty, relative, equivalent-spelling, symbolic-link and aliasing cases.

### 25.5 Read-only health probe

EI7E-D02. The Application owns the probe contract (P1); Infrastructure implements it (P2); the dedicated runtime consumes it (E1).

```text
GET /equity/health
  → Equity runtime: not configured?  → 503 EQUITY_NOT_CONFIGURED (not retryable)
  → P1 Application probe (read-only port, typed outcome)
      → P2 SQLite adapter, through the EI-7D read-only transaction:
          WAL header check before open; open mode=ro; confirm query_only;
          BEGIN; check that the required Equity tables exist; ROLLBACK; close
  → READABLE              → 200 {"contract_id": "northstar.equity-private-read/1", "status": "READABLE"}
  → dependency unavailable → 503 DEPENDENCY_UNAVAILABLE (retryable)
  → storage integrity      → 500 PERSISTED_AUTHORITY_INVALID (not retryable)
```

**P1 requirements (Application, separately authorized):**

- One read-only port, separate from the existing ports, with no write, repair, recovery or clock dependency.
- Typed outcomes that keep the three distinctions: readable; dependency unavailable (retryable); permanent storage integrity (not retryable).
- The permanent case reuses the SI-A contract `EquityReadStorageIntegrityError` and its existing mapping to `PERSISTED_AUTHORITY_INVALID`. Missing required tables are dependency unavailability, as in EI7-SI1.
- The existing refusal vocabulary is used; no public code is added. `EQUITY_NOT_CONFIGURED` stays a runtime condition of the API, not a query refusal (EI-7C).
- READABLE claims only that the dependency was read safely (EI7-SI2): never database-wide integrity, complete or verified history, market data, broker connectivity, execution, performance or operational readiness.
- No outcome message carries SQLite text, a path or a stored payload.

**P2 requirements (Infrastructure, separately authorized):**

- Implement the P1 port with the EI-7D `read_only_transaction` machinery and its SI-B classification. One connection and one short read transaction; schema-catalog reads only; no data-row scan, no issuance enumeration, no ledger walk.
- The required table set is the set of tables the EI-7D readers of the five families read. P2 states it in its PR.
- **Never:** creation, initialization, migration, recovery, repair, checkpoint, journal-mode change, `immutable=1`, read-write fallback or `PRAGMA integrity_check`.
- Real-SQLite tests on disposable files:
  - readable;
  - missing file, missing table, empty file;
  - WAL-mode file; hot journal; held lock;
  - a damaged schema page (`SQLITE_CORRUPT`) and a foreign file (`SQLITE_NOTADB`);
  - file bytes and sidecar files unchanged.

**E1 requirements:**
- The four outcomes with exact bodies.
- The health route imports no Infrastructure type, runs no SQL, and substitutes no snapshot or issuance lookup for the probe.
- No metric, timestamp, path or SQLite text in any health response.

### 25.6 V1 session-list presentation boundary

EI7E-D06: the V1 declared and recorded session responses are built from one call of `ListEquityDeclaredSessionRecommendationsUseCase` or `ListEquityRecordedSessionIssuancesUseCase` and nothing else.

| Kept | Omitted in V1 |
| --- | --- |
| `contract_id`, `exchange_code`, `trading_date`, `evidence_class`, `recording_position` (N), `recording_watermark` (W), the declaration state at W; per item the issuance identity, `recording_sequence`, modeled `issued_at`, `issuance_kind`, `actionable` and, in the recorded view, `covered_by_declaration`; each view's approved order; the limit of 50; the watermark rules | The per-item `lifecycle` object of the section 23.3 example, and any list-level lifecycle status |

- `GetEquityLifecycleSummaryUseCase` is not called by a session route. It stays in the Application, unchanged.
- E1 test: a session request makes no lifecycle read, so its outcome never depends on lifecycle availability.

### 25.7 Canonical strategy identity encoding

EI7E-D07A, within EI7-CF3-F. The ten decoding steps of register section 15.8 refine the five steps of section 24.5:

| Input segment | Result |
| --- | --- |
| Canonical unpadded base64url of a valid identity's UTF-8 bytes | The decoded identity is used and returned as the Application view states it |
| Any character outside `A`–`Z`, `a`–`z`, `0`–`9`, `_`, `-` (including `=`, `+`, `/`, `%`) | 422 `UNSUPPORTED_SCOPE` |
| Non-canonical trailing bits, or a length that cannot be canonical | 422 |
| Bytes that are not strict UTF-8 | 422 |
| Text that `StrategyIdentity` rejects (empty) or canonicalizes differently (surrounding whitespace) | 422 |

- Errors never echo the segment. Responses carry the decoded canonical identity, never the segment in its place.
- `%2F` is decoded by uvicorn before routing and so splits the path; the base64url alphabet never needs percent-encoding.
- **Transport limitation:** no Application maximum exists. The inspected stack is uvicorn 0.52.4 with h11 0.16.0; httptools is not in the lock. Its request-head limit (about 16 KiB by default) and the gateway's limits bound the addressable identity length. These are transport facts to verify in E1 and V1, not domain restrictions. An over-long request is refused by the transport, never truncated.
- **E1 tests:**
  - identities containing `/`; Unicode;
  - identities whose encoding would contain `+`, `=` or `%` in other alphabets;
  - empty segments; padded segments;
  - invalid UTF-8; non-canonical trailing bits; surrounding and other non-canonical whitespace;
  - `%2F`, `%252F` and other encoded separators;
  - percent-encoding interactions;
  - safe response echo.

### 25.8 Unexpected HTTP failure safety

EI7E-D07B. Three layers, kept distinct:

| Layer | Trigger | Response |
| --- | --- | --- |
| Framework | Unknown path; unsupported method | The framework's 404 or 405, handled safely; never mapped to a domain code |
| Contracted | `EquityReadQueryRefusedError` (EI-7C, SI-A); the not-configured runtime; request validation | The approved envelope with the section 13.3 status, `retryable` and a fixed API-owned message |
| Non-contract fallback | Any other exception | HTTP 500 with a fixed generic response; not a member of the domain vocabulary; never `PERSISTED_AUTHORITY_INVALID` without evidence of a persisted-authority failure |

- No response on any layer carries a traceback, SQL, SQLite driver text, file path, configuration value, credential, request input or chained cause.
- `debug` is disabled. The fallback's exact representation is fixed in E1, and tested. No logging change is authorized by this record.

### 25.9 Runtime and HTTP integration

| Route | Request decoding (E1) | Application call | Notes |
| --- | --- | --- | --- |
| `/equity/health` | None | P1 probe | Section 25.5 |
| `/equity/sessions/{exchange}/{date}/recommendations` | Exchange, date, optional `watermark` | `ListEquityDeclaredSessionRecommendationsUseCase.execute` | No lifecycle reads (section 25.6) |
| `/equity/sessions/{exchange}/{date}/recorded-issuances` | Same | `ListEquityRecordedSessionIssuancesUseCase.execute` | Same |
| `/equity/recommendations/{issuance_id}` | Issuance identity | `GetEquityRecommendationDetailUseCase.execute` | Full lifecycle; evidence panels |
| `/equity/research/{strategy}/{exchange}/{symbol}/{date}` | Strategy codec (section 25.7), listing, date | `GetEquityResearchTimelineUseCase.execute` | Data states stay data |

- **Watermark.** Canonical decimal syntax only (section 23.3). A syntactically valid value above 2^63−1 must reach the Application's `WATERMARK_AHEAD` refusal (EI7-P7) without an unbounded integer conversion.
- **Errors.** One handler maps `EquityReadQueryRefusedError.reason` to the section 13.3 status and takes `retryable` from the refusal. The API never inspects SQLite errors; permanent storage failures reach it only as the Application's `PERSISTED_AUTHORITY_INVALID` (SI-A, SI-B).
- **Still E1 closure items**, settled within the approved contracts and stated in the E1 PR:
  - the treatment of unknown and repeated query parameters (FastAPI ignores unknown ones and keeps the last of a repeated one), without a new public code;
  - the precedence between `EQUITY_NOT_CONFIGURED` and request validation;
  - the default bind of the dedicated entry point outside a container (EI7-CF3-B; "never `0.0.0.0` by default outside a container" stands).

### 25.10 Deployment prerequisites

Before any operational exposure (D1 and V1, each separately authorized):

1. EI7E-D05 satisfied: an Equity database exists, and its provisioning, source authority, recovery and acceptance evidence are authorized and verified. D1 provisions and populates nothing.
2. The topology of section 25.3, and the ten checks of register section 15.4, verified on the deployment machine.
3. The anchored GET allow-list and default deny at the India gateway. Their equality with the dedicated application's OpenAPI templates is tested.
4. The read-only mount, and the absence of the Futures volume, settings and secrets in the Equity container.
5. Journal-mode validation (rollback journal) and operating controls (EI7-CF5-F). A recovery owner for a hot journal under EI7E-D05.
6. The India gateway's own deployment acceptance, which is not yet recorded (runbook section 2).
7. CME and Futures unchanged and functional.

### 25.11 Implementation sequence

```text
G (docs, this record) ─▶ P1 (application: probe port) ─▶ P2 (infrastructure: SQLite probe)
                                                                  │
                                       E1 (api: dedicated Equity app, code only) ◀┘
                                                                  │
                         D1 (api deploy/india: gateway, network, read-only mount) ◀┘  separate deployment authorization
                                                                  │
                         V1 (deployment-machine verification; EI7E-D05 prerequisites) ◀┘
```

Each step needs its own authorization ([register section 15.9](../product/Equity-Intraday-V1-EI7-Decision-Register.md#159-dependency-and-authorization-gates)). EI-7F, EI-7G and EI-7H stay separately governed.

### 25.12 Acceptance and negative-test matrix

**Third reconciliation (2026-11-28).** All 24 scenarios stay **PLANNED — NOT TESTED** for full EI-7 acceptance. "Existing layer evidence" means tests in the merged EI-7C, EI-7D, SI-A and SI-B suites; none has an acceptance record. No result is recorded here.

| Scenario | Existing layer evidence | P1 / P2 owed | E1 owed (AS, AR, XP) | D1 / V1 owed | EI-7F / EI-7H owed |
| --- | --- | --- | --- | --- | --- |
| N01 `NO_SIGNAL` | EI-7C research view; EI-7D timeline reader | — | Stored outcome and reason serialized, never a candidate | — | F; H |
| N02 `REJECTED` | Same | — | Distinct from N01; the evaluator's `EVIDENCE_NOT_SYNTHETIC` never presented | — | F; H |
| N03 Generation status | EI-7C types and view rules | — | Only `PROPOSED` or `NOT_QUERYABLE` with its reason; never "issued" without an issuance | — | F |
| N04 UNKNOWN versus `NOT_QUERYABLE` | EI-7C | — | Both as HTTP 200 data, never conflated | — | F |
| N05 Unknown issuance | EI-7C detail | — | 404 `ISSUANCE_NOT_FOUND`; a malformed identity is 422 | — | F |
| N06 `RECORDED`, no events | EI-7C; EI-7D | — | Empty events, `position` `"0"`, no last-check time | — | F |
| N07 Recorded before declaration | EI-7C; EI-7D | — | `covered_by_declaration: false`; absent from the declared view; `declaration: NONE` with N and W as HTTP 200 | — | F |
| N08 Modeled lifecycle | EI-7C | — | Modeled vocabulary; nothing executed or filled | — | F |
| N09 `INCOMPLETE_EVIDENCE` | EI-7C; EI-7D | — | Terminal, `UNRESOLVED`, deficiencies, no exit price | — | F |
| N10 Missing cited snapshot | EI-7C panels | — | Panel statuses serialized; required facts intact | — | F |
| N11 Corrupt persisted authority | EI-7C lineage; SI-A (AU, PC); SI-B real damaged pages (IS) | Probe CORRUPT and NOTADB outcomes | 500, not retryable, fixed message, over disposable real SQLite for every route and health; a programming error never mapped to it (EI7E-D07B) | — | H |
| N12 Dependency unavailable | SI-B: busy, cannot open, missing file and tables, hot journal, WAL (IS) | Probe unavailable outcomes | 503 `DEPENDENCY_UNAVAILABLE` retryable; 503 `EQUITY_NOT_CONFIGURED` not retryable on all five routes, including whitespace-only configuration | Deployed health with the deployed file | F (Retry only when retryable) |
| N13 Unsupported scope | EI-7C validation | — | The section 25.7 codec matrix; exchange, date, symbol and identity validation; 422 without echo | — | — |
| N14 Non-SYNTHETIC | EI-7C (EI7-EC1) | — | 409, no payload | — | F; H |
| N15 Watermarks | EI-7C; EI-7D | — | Canonical syntax; 409 above N and above 2^63−1; 422 malformed; N and W as strings | — | — |
| N16 Bounded discovery | EI-7C; EI-7D | — | 50 succeed, 51 refused with 409 in both views; both orders; no lifecycle read in lists (EI7E-D06) | — | — |
| N17 No read-side writes | EI-7D; SI-B non-mutation suites | Probe non-mutation: bytes, sidecars, no DDL | Construction opens nothing; no fallback; health writes nothing; EI4-D11 scan; no Futures imports or Futures schema initialization in the Equity process | Read-only mount; no Futures volume | H |
| N18 Private access | — | — | Five routes only; 404 or 405 elsewhere; HEAD 405; no documentation routes; no trailing-slash redirect; encoded paths; no CORS widening; no Equity path in `create_app().openapi()`; `debug` off | The ten checks of register section 15.4; allow-list equal to the OpenAPI templates; CME unchanged | H |
| N19 Deep links | — | — | — | — | F (EI7-CF6) |
| N20 Restart | EI-7D fresh-process reads | Fresh-process probe | Fresh API process from the Equity setting alone gives identical responses | Equity restart and lifecycle (section 15.4, check 9) | H |
| N21 Read consistency | EI-7C; EI-7D | — | Detail issuance and lifecycle in one transaction; lists pinned to W | — | H |
| N22 Synthetic operator | — | — | — | — | G, H (conditional; EI7-CF7) |
| N23 Vocabulary and safe failures | — | — | No performance vocabulary; health carries no metric; the fallback is fixed and safe; no driver text, path or credential in any response | No credential in logs or responses | F |
| N24 Typed distinctions | EI-7C types | READABLE never an integrity certification | A verified result, a healthy dependency and database-wide integrity never conflated; candidate, proposal, issuance, lifecycle and trade distinct | — | F |

The section 24.13 negative tests stand. These are added:
- the D07A codec matrix;
- the D07B fallback;
- the D06 no-lifecycle-read test;
- the D01 OpenAPI and import tests;
- the D04 configuration and aliasing tests;
- the D02 four-outcome health tests;
- the D03 topology checks.

### 25.13 Source facts, residual risks and limitations

The verified source facts are listed in [register section 15.10](../product/Equity-Intraday-V1-EI7-Decision-Register.md#1510-source-facts-residual-limitations-and-deployment-blockers). They describe source as inspected, not deployed behaviour.

| Risk | Severity | State after EI7E-D01 to EI7E-D07B | Remaining gate |
| --- | --- | --- | --- |
| Equity reachable through the CME `/api/*` proxy | Critical | Resolved at the architecture-decision level (EI7E-D01); not implemented | E1 AR tests |
| Gateway bypass by a container on the India project network | High | Direction approved (EI7E-D03) | D1, V1 |
| The India gateway has no recorded deployment acceptance | High | Unchanged | V1 |
| No provisioned Equity database; hot-journal recovery unowned | High | Exposure prohibited (EI7E-D05) | Later explicit authorization |
| Health would need SQL in the API | High | Resolved at the governance level (EI7E-D02) | P1, P2, E1 |
| Input echo, trailing-slash redirects, documentation routes | Medium | Required off in the dedicated application (section 25.1) | E1 |
| Floating Caddy tag; gateway matcher, case and encoding semantics | Medium | Unverified | D1, V1 |
| Credentials forwarded upstream by the gateway | Low | Unverified | D1, V1 |
| WAL check-to-open race | Low | Accepted (EI7-CF5-F); the read-only mount is expected to mitigate it | V1 |
| Transport-bounded strategy identities | Low | Accepted (EI7E-D07A) | E1, V1 verification |

Accepted limitations are those of register sections 7, 13.9, 14.8 and 15.10.

### 25.14 Stop conditions

The stop conditions of section 24.15 stand. Condition 5 is changed only as register section 14.6 notes: the EI7E-D02 probe port, delivered by P1 and P2, is approved. Each slice also stops and reports, rather than deciding, if:

| Slice | Stops if |
| --- | --- |
| P1 | The probe needs a write, a repair or recovery, a data scan, a new public code or a change to an existing port's contract |
| P2 | The probe cannot be served by the EI-7D read-only transaction; it would need DDL, `integrity_check`, `immutable=1`, a read-write fallback or a scan of data rows; or it cannot keep the files unchanged |
| E1 | An Equity path would appear in the existing application or its OpenAPI; any Futures module, CME file or deployment file would need a change; a session list would need lifecycle reads; the codec would need an identity restriction; a new public code would be needed; health would need anything beyond EI7E-D02 |
| D1 | The shared-namespace topology cannot be supported; any other topology would need an owner decision. Also if CME would change, or a database would be provisioned or populated |
| V1 | Any check of register section 15.4 or section 25.10 fails. Exposure is then refused |
