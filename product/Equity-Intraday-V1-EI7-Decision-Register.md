# Equity Intraday V1 EI-7 Decision Register

## Status

**Owner decision record. Decisions approved 2026-11-15; record prepared 2026-11-16 (EI-7B Docs PR-0). Scope: the architectural directions for EI-7, the private, read-only API and Web recommendation-inspection experience for synthetic-only NSE Equity Intraday V1.**

- The product owner **approved** the twelve architectural directions R-1 to R-12 on **2026-11-15** ([section 4](#4-approved-decisions)).
- They are directions, not contracts. The exact Application types and signatures, endpoint paths, response fields, HTTP statuses, error payloads, the discovery limit and the Web route patterns are **PROPOSED — PENDING OWNER APPROVAL** as EI7-CF1 to EI7-CF7 ([section 8](#8-contract-finalization-decisions-pending)). Their detail is in the [EI-7 Private API and Web Design](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md).
- **No EI-7 implementation is authorized.** The engineering slices EI-7C to EI-7H each need a separate owner authorization ([section 9](#9-implementation-authorization-register)).
- The 24 acceptance scenarios EI7-N01 to EI7-N24 are **PLANNED — NOT TESTED** ([section 10](#10-acceptance-scenario-register)).
- Nothing here authorizes real or provider market data, ACTUAL issuance, new issuance or trading authority, broker integration, orders or fills, an operator CLI, a scheduler, a Caddy, bind or deployment change, a database migration or file move, or any EI-8 metric.
- The [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md) remains a draft wherever no owner decision approves it. Its section 11 screens stay proposals. [ADR-017](../adr/ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md) stays Proposed.
- This record has not been through Architecture Review or Engineering Review.

**Source of the decisions.** The owner's EI-7B instruction of 2026-11-15, which reproduces the approved directions. The owner's working document, `Northstar_EI7B_Governance_Decision_Register_OWNER_APPROVED_2026-11-15.md`, was not available in this repository or on the engineering machine when this record was prepared. The decisions below are recorded from the instruction text; nothing is added to them.

## 1. Purpose and identifiers

This register is the authoritative record of the owner's EI-7 decisions. It changes no earlier register: the EI-3, EI-4 and EI-5 registers keep their decisions, and the EI-6A, EI-6B and EI-6C acceptances are unchanged.

| Identifier | Meaning |
| --- | --- |
| R-1 to R-12 | The approved EI-7 architectural directions of section 4. Outside this register they are cited as **EI-7 R-1** to **EI-7 R-12**, so that they are not confused with the EI-4C contract points R-1 and R-2 settled by [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) |
| EI7-CF1 to EI7-CF7 | Contract-finalization decisions, pending (section 8) |
| EI7-N01 to EI7-N24 | Planned acceptance scenarios (section 10; design section 16) |
| EI-7C to EI-7H | Proposed engineering slices (design section 18) |

| Status | Meaning |
| --- | --- |
| **APPROVED — 2026-11-15** | An owner-approved architectural direction |
| **ACCEPTED** (earlier) | An earlier owner decision or accepted milestone, unchanged by EI-7 |
| **PROPOSED — PENDING OWNER APPROVAL** | An engineering recommendation for a technical contract |
| **OPEN** | An unresolved point with no recommendation yet |
| **PLANNED — NOT TESTED** | An acceptance scenario with no evidence |
| **NOT AUTHORIZED** | An implementation or capability nobody may build or enable yet |

## 2. Objective and V1 boundaries

**Objective, as stated by the owner (EI-7A instruction, 2026-11-15).** A complete synthetic Equity Intraday V1 product experience in which the single owner can discover research results for an eligible instrument; understand why a candidate was or was not produced; inspect an authoritative recorded recommendation with its entry, stop, target, risk and validity; see its issuance identity and provenance; follow its lifecycle; inspect its final or unresolved outcome; review its evidence; tell synthetic modeled outcomes apart from actual trades; and repeat the same demonstration after a restart without inventing or duplicating history.

**V1 boundaries, as approved:**

| Boundary | Source |
| --- | --- |
| Private, read-only API and Web; no HTTP-triggered generation, issuance, declaration or monitoring | R-1 |
| SYNTHETIC evidence only, with explicit refusal of any other class | R-10; EI0-D14 (synthetic model) |
| MODELED issuance only; LONG only; issued recommendations are non-actionable | EI5-D1; EI0-D3; ADR-024 |
| Single host; one authoritative Equity SQLite file, separate from Futures | R-7; ADR-024 section 14 |
| Private by deployment, not by in-app authentication | R-5; ADR-022 DQ9 |
| Modeled outcomes only; no EI-8 metrics | R-9 |

EI-8 historical evaluation follows EI-7 and is not part of it.

## 3. EI-7A reconnaissance findings

The EI-7A read-only reconnaissance (2026-11-15) read every repository at these commits, all on `develop` with clean working trees: northstar-core `29bd34a`, northstar-application `617b538`, northstar-infrastructure `9d9c960`, northstar-api `7ad37f5`, northstar-web `5aeb94c`, northstar-docs `ac3a804`. The same commits were confirmed again on 2026-11-16 before this record was written.

| # | Finding | Evidence |
| --- | --- | --- |
| A-1 | No EI-7 specification, design, ADR or decision existed. EI-7 was defined only as "Private read-only API and web" in the draft specification | Specification section 12.1; EI-5 Decision Register section 23.6 |
| A-2 | One recommendation can be read correctly: `EquityRecommendationLifecycleReader.load_lifecycle(issuance_id)` returns the re-derived issuance and its verified, Core-replayed lifecycle from one SQLite read transaction | `ports/equity_recommendation_lifecycle_reader.py:22`; `SQLiteEquityRecommendationLifecycleStore.load_lifecycle` |
| A-3 | Discovery is narrow: no list-all, no pagination, no query by strategy or date range. The only multi-issuance read is `session_view` / `session_view_at` per exchange and date, which shows an issuance only once a later completeness declaration covers it | `ports/equity_recommendation_issuance.py:239-262`; Infrastructure `test_ms32_the_three_scope_modes_against_the_persisted_authority` |
| A-4 | No composition root composes any equity use case outside tests. Infrastructure architecture tests forbid constructing the issuance and lifecycle adapters inside Infrastructure | `test_sqlite_equity_recommendation_issuance.py:1302`; `test_sqlite_equity_recommendation_lifecycle.py:1686`; ADR-023 line 48 |
| A-5 | The issuance and lifecycle adapters open `mode=rw` and run their schema initializer, `CREATE … IF NOT EXISTS` and a commit, on every call, reads included. Only the EI-6C catalog is read-only by construction | `sqlite_equity_recommendation_issuance.py:376-396`; `sqlite_equity_recommendation_lifecycle.py:297-318` |
| A-6 | The API has five endpoints, none for equity, and no in-app authentication. It binds `0.0.0.0:8000`, leaves the interactive API docs on, and writes the Futures schema at startup. Futures privacy is enforced by Caddy basic auth, a loopback port, an SSH tunnel and an exact-path GET allow-list | `northstar_api/__init__.py:12`; `runtime.py:264-287`; Futures deployment runbook sections 1 and 8 |
| A-7 | The Web has no router, no equity recommendation screen and no synthetic or MODELED label. Its Futures loading, card and test patterns are reusable | `src/App.tsx:697-743`; `src/futures/` |
| A-8 | A synthetic journey cannot run outside tests: `EquitySyntheticListingEligibilitySource` has no non-test implementation, and every write step needs an explicit caller that does not exist | Application `ports/equity_recommendation_references.py:509`; test doubles only |
| A-9 | The accepted production references cannot carry real data: the NSE session catalog ends at 2026-10-19 and the production listing-applicability source always answers UNKNOWN | `nse_accepted_equity_session_evidence.py:488-490`; `nse_equity_research_window_reference.py:133-146` |
| A-10 | No stored time of recording exists for lifecycle events, and monitoring runs are not stored (EI6C-P12) | `sqlite_equity_recommendation_lifecycle_schema.py`; EI-5 Decision Register section 22.5 |

## 4. Approved decisions

| ID | Decision | Status |
| --- | --- | --- |
| R-1 | EI-7 scope: private, read-only inspection | **APPROVED — 2026-11-15** |
| R-2 | Application-owned query architecture | **APPROVED — 2026-11-15** |
| R-3 | Bounded recommendation discovery | **APPROVED — 2026-11-15** |
| R-4 | Strict SQLite read-only boundary | **APPROVED — 2026-11-15** |
| R-5 | Private access boundary | **APPROVED — 2026-11-15** |
| R-6 | Verified recommendation details | **APPROVED — 2026-11-15** |
| R-7 | Equity and Futures database separation | **APPROVED — 2026-11-15** |
| R-8 | Web deep linking | **APPROVED — 2026-11-15** |
| R-9 | Modeled outcome display | **APPROVED — 2026-11-15** |
| R-10 | Evidence class | **APPROVED — 2026-11-15** |
| R-11 | Time representation | **APPROVED — 2026-11-15** |
| R-12 | Research visibility | **APPROVED — 2026-11-15** |

### 4.1 R-1: EI-7 scope

**APPROVED — 2026-11-15.**

- **Approved:** a private, read-only API and Web recommendation-inspection experience. No HTTP request generates, issues, declares or monitors anything. A synthetic operator CLI is governed separately.
- **Not authorized:** the operator CLI's implementation; any scheduler; any new issuance or trading authority.
- **Options considered:** (a) read-only API and Web only; (b) the same plus the synthetic journey CLI in one milestone.
- **Rationale:** EI-6C may write lifecycle events through the EI-6B recorder, so it is not a query; keeping every write behind a separately governed, explicitly invoked operation keeps HTTP free of write authority.
- **Accepted tradeoff:** until the operator CLI is separately authorized, a demonstration database can be produced only by test code (EI7-N22 is conditional).

### 4.2 R-2: Application-owned query architecture

**APPROVED — 2026-11-15.**

- **Approved:** immutable, Application-owned read and query view models; thin API controllers; the existing authoritative reader ports; no direct Infrastructure manipulation in HTTP controllers; no writer, recorder, issuer, monitor or clock dependency in any read use case.
- **Still open:** exact types, signatures and response contracts (EI7-CF2, EI7-CF4).
- **Options considered:** (A) thin API handlers calling the ports directly; (B) Application-owned query use cases with thin handlers.
- **Rationale:** option B keeps the joining of issuance, generation record and lifecycle, the outcome-class mapping and the error classification in the Application, next to the domain; it is testable without HTTP; and it matches the draft specification's integration pattern, "an API read model over persisted state".
- **Accepted tradeoff:** more Application code before the first endpoint.

### 4.3 R-3: Recommendation discovery

**APPROVED — 2026-11-15.**

- **Approved:** the existing declaration-aware session view; an additional, bounded read of recorded issuances for one exchange and one trading date; the declaration and completeness semantics kept; no global list-all and no unrestricted date-range browsing.
- **Still open:** the numeric result limit; the deterministic ordering; overflow handling; the continuation or pagination policy; the watermark semantics; how complete and incomplete sessions are labelled (EI7-CF1).
- **Constraint:** the EI-6C catalog limit of 300 snapshots is not automatically reused as the issuance-discovery limit.
- **Options considered:** (a) declared session view only; (b) (a) plus a bounded per-session recorded read; (c) a paginated date-range read.
- **Rationale:** the declared view alone hides every new issuance until a later declaration covers it; a bounded per-session read uses the existing `equity_recommendation_issuances_by_session (exchange_code, trading_date)` index and is not a list-all.
- **Accepted tradeoff:** browsing across dates is not possible; a session must be chosen. EI6C-P7's "no new EI-6A list-all interface" stays in force for EI-6C monitoring discovery; the bounded read is a separately governed read surface, not given to the monitor.

### 4.4 R-4: Strict SQLite read-only boundary

**APPROVED — 2026-11-15.**

- **Approved:** additive read-only reader paths; `mode=ro` and `PRAGMA query_only = ON` where compatible with the existing adapters; no schema DDL or initialization during equity GET processing; the existing EI-6A and EI-6B write authorities unchanged.
- **Not authorized by this record:** the implementation.
- **Options considered:** (a) accept the existing read-write readers; (b) add read-only readers.
- **Rationale:** the existing readers execute DDL and commit on every call (finding A-5), so a GET is not read-only by construction. The EI-6C catalog is the accepted precedent for `mode=ro` with confirmed `query_only`.
- **Accepted tradeoff:** a `mode=ro` reader cannot recover a hot rollback journal (EI-6C L8); it reports unavailability until a read-write connection does.

### 4.5 R-5: Private access boundary

**APPROVED — 2026-11-15.**

- **Approved architectural direction:** Caddy basic authentication; a loopback or internal network restriction; an SSH tunnel; a restricted endpoint allow-list; a safe API bind configuration; publicly accessible interactive API documentation disabled; no new in-app token system for synthetic personal-use V1.
- **Recorded explicitly:** **deployment privacy is a condition, not intrinsic application authentication.** The API is private only when deployed behind this boundary; the single intended user does not make it private (ADR-022 DQ9 records the absence of an identity layer as a limitation, not a protection).
- **Not authorized by this record:** any Caddy change, runtime bind change or deployment action.
- **Still open:** the exact environment and route-enforcement contracts (EI7-CF3).
- **Options considered:** (a) reuse the Futures deployment boundary; (b) add an in-app token.
- **Rationale:** (a) is the boundary already used, documented and pinned by a test for Futures (`test_india8e_indian_deployment.py:440`).
- **Accepted tradeoff:** an API process run outside that boundary is unprotected; deployment must be refused unless the CF3 conditions hold.

### 4.6 R-6: Verified recommendation details

**APPROVED — 2026-11-15.**

- **Approved:** begin with `load_lifecycle(issuance_id)` for the authoritative issuance and lifecycle; decode the stored generation facts; match the exact authoritative generation identity; validate the lineage and the relevant version identities; do not replay the complete generation on every GET; refuse corrupt or inconsistent required evidence.
- **Still open:** which fields are required and which optional; the exact error classification (EI7-CF4).
- **Options considered:** (a) decode the stored generation record; (b) replay the full generation (`VerifyStoredEquityRecommendationGenerationUseCase`) on every read.
- **Rationale:** the stored records are insert-only and trigger-protected, and the issued recommendation is already re-derived from the stored disposition on every load; full replay needs eight dependencies per request.
- **Accepted tradeoff:** a GET proves lineage and integrity of the stored facts, not a fresh re-evaluation of the generation.

### 4.7 R-7: Equity and Futures database separation

**APPROVED — 2026-11-15.**

- **Approved:** separate SQLite database files for Equity and Futures; one authoritative Equity database file for the Equity lifecycle; the approved single-host scope preserved; no cross-database transaction authority.
- **Not authorized:** any database migration, file move or deployment change.
- **ADR compatibility:** recorded as a dated clarification of [ADR-024, section 14](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md#14-trust-model), not a substantive amendment (section 6.1).
- **Options considered:** (a) share the Futures file; (b) a separate Equity file.
- **Rationale:** the Futures API runtime initializes the Futures schema in its file at startup, and every Equity single-file rule lists Equity objects only.
- **Accepted tradeoff:** two files to back up; no query or transaction may span them.

### 4.8 R-8: Web deep linking

**APPROVED — 2026-11-15.**

- **Approved:** the smallest suitable routing mechanism; stable session and recommendation URLs; refresh and browser-navigation support; the existing hash routing preferred only if technically sufficient; no unnecessary frontend rewrite.
- **Still open:** the exact route patterns (EI7-CF6).

### 4.9 R-9: Modeled outcome display

**APPROVED — 2026-11-15.**

- **Approved for display:** synthetic recommendation levels; validity; modeled lifecycle events and outcomes; sizing and cost estimates where authoritatively supported; evidence deficiencies and unresolved information.
- **Not authorized:** executed fills; realized profit or loss; win-rate reporting; strategy performance; slippage-adjusted returns; EI-8 historical evaluation metrics.
- **Constraint:** `ACTIVE` (and `ENTRY_ACTIVATED`) must not be presented as a broker execution.

### 4.10 R-10: Evidence class

**APPROVED — 2026-11-15.**

- **Approved:** SYNTHETIC evidence only; explicit server-side refusal of unsupported evidence classes; no silent conversion of production evidence into synthetic evidence.
- **Still open:** the exact HTTP error contract (EI7-CF2).

### 4.11 R-11: Time representation

**APPROVED — 2026-11-15.**

- **Approved:** API timestamps in canonical UTC; primary UI display in IST; an accessible UTC reference.
- **Distinguished instants:**

| Instant | Meaning | Source |
| --- | --- | --- |
| Session time | The NSE session and research-window boundaries, interpreted in IST | Session and research-window references |
| Evidence `as_of` | The knowledge instant of an EI-2B snapshot | `EquityCandleEvidenceSnapshot.as_of` |
| Snapshot `issued_at` | **Caller-declared**; not proof of when the snapshot was created (EI-6C L1) | `EquityCandleEvidenceSnapshot.issued_at` |
| Recommendation `issued_at` | **Modeled**: the decision instant plus the timing policy's modeled issuance latency | `EquityRecommendationTiming.issued_at`; `EquityIssuedRecommendation.issued_at` |
| Lifecycle `evaluated_at` | **Modeled**: the evaluated snapshot's `as_of` | `EquityLifecycleEvent.evaluated_at` |

- **Constraint:** no actual event-recording timestamp and no monitoring-run timestamp may be shown or fabricated; neither exists (finding A-10).

### 4.12 R-12: Research visibility

**APPROVED — 2026-11-15.**

- **Approved:** only persisted research and generation evidence is presented; missing history means UNKNOWN; the distinctions among research candidate, `NO_SIGNAL`, `REJECTED`, `PROPOSED`, `NOT_ELIGIBLE`, `UNRESOLVED`, issued and lifecycle status are preserved.
- **Constraint:** refused or duplicate research history is never invented. Whether to persist refused or duplicate research outcomes stays a separate open EI-3 decision ([EI-3 Decision Register, section 11.2](Equity-Intraday-V1-EI3-Decision-Register.md#112-decisions-still-pending): "not decided here").

## 5. Compatibility with EI-3 to EI-6

| Earlier authority | EI-7 effect |
| --- | --- |
| EI3-D3, EI3-D7 (research outcome vocabulary and labels) | Preserved by R-12: `LONG_CANDIDATE` is a research result, not a recommendation |
| EI4-D9-A, EI4-D9-H (dispositions; missing history is unknown) | Preserved by R-12: `PROPOSED` is never shown as issued |
| EI4-D11 (verified recording boundary) | Read composition must not construct the generation store for writing (section 6.2) |
| EI5-D1 to EI5-D8; ADR-024 | Unchanged. Issued recommendations stay non-actionable and MODELED-only |
| EI-6A issuance authority | Unchanged and not revoked. R-3's bounded read is additive and read-only |
| EI-6B lifecycle authority | Unchanged. All lifecycle writes stay with the EI-6B recorder |
| EI-6C, EI6C-P1 to EI6C-P12, limit 300 | Unchanged. Monitoring is never invoked from HTTP; its policy, limit and no-checkpoint rule are untouched; monitoring runs stay non-durable |
| EI6C-FINAL-ACCEPTANCE | Unchanged; it did not authorize EI-7, and this record authorizes no EI-7 implementation |
| EI4-C4 | **PENDING**, unchanged |

## 6. Governance clarifications

### 6.1 ADR-024 single-file wording and R-7

ADR-024 section 14 says "one host and one SQLite file", and section 13 places issuance and lifecycle tables in "the same SQLite database authority" as the EI-4D records. Every document that states the single-file rule lists only Equity objects:

- ADR-021: EI-2B evidence and EI-3C timelines "in one SQLite file"; a separate file for generation records rejected;
- ADR-022: the policy catalog "in the same SQLite database file as the EI-2B evidence, the EI-3C timelines and the EI-4D dispositions and attempts";
- the EI-6A storage design: "the same SQLite file as the EI-2B evidence, the EI-3C timelines and the EI-4D generation records".

None places Futures or Options tables in that file, and the Futures API runtime initializes only Futures schemas in its own file (`runtime.py:264-287`). The single-file rule therefore means one authoritative **Equity** file, not one file for the whole platform. A dated clarification recording this and R-7 is added to [ADR-024, section 14](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md#14-trust-model). No issuance, lifecycle, schema or trust-model contract changes, and no new ADR is needed. Its remaining contract points are EI7-CF5.

### 6.2 EI4-D11 composition roots and read-only consumers

The EI4-D11 design requires future composition roots (API, CLI, background) to "construct the recorder and pass the adapter only to it", and forbids constructing `SQLiteEquityRecommendationGenerationStore` "for writing" except for that purpose; it also requires least privilege for read-only consumers. An EI-7 read composition is compatible only if it never holds the write-capable generation store: it reads generation records through `EquityRecommendationGenerationRecordReader`, served by a read-only path (R-4), and northstar-api adds the architecture scan that EI4-D11 requires of the first repository with such wiring.

### 6.3 EI-5 Decision Register section 23.6

Its row "EI-7; EI-8 — NOT AUTHORIZED" is not rewritten. A dated note there points to this register: EI-7's architectural directions are approved here; EI-7 implementation remains not authorized; EI-8 is unchanged.

### 6.4 Product specification

Dated notes in the status section and in section 11 point here. The draft text, including the six proposed dashboard screens and the Performance screen, is not rewritten and is not promoted by EI-7.

### 6.5 ADR-017

It stays **Proposed**; no repository evidence shows a later approval. The layering EI-7 relies on (Web → API → Application → Core, the API as composition root) is recorded here by R-2 for EI-7 only.

## 7. Inherited limitations

EI-7 resolves none of these by presenting them. Each stays in force:

- EI-6C L1 to L13 and the scenario limitations MS-02, MS-20, MS-21 and MS-36 ([EI-6C acceptance record, section 11](Equity-Intraday-V1-EI6C-Lifecycle-Monitoring-and-Snapshot-Selection-Engineering-Acceptance-Record.md#11-limitations));
- EI-6B E1 to E9 and the carried-forward C1 to C10 ([EI-6B acceptance record, section 16](Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Engineering-Acceptance-Record.md#16-limitations-and-verification-gaps));
- the EI-6C follow-ups F1 to F10, **NOT AUTHORIZED**;
- EI4-C4, **PENDING**;
- synthetic-only evidence and MODELED-only issuance;
- no persistence of monitoring runs or of the monitoring instant (EI6C-P12);
- no single atomic view across separate read connections (EI-6C L7);
- incomplete declared-session discovery (EI6C-P7; EI-6C L10);
- single-host SQLite (EI-6C L9);
- existing reliability and performance limits: every issuance read checks the whole ledger; a quiet monitoring rerun takes about 128 s at the limit of 300 (EI-6C L4); readers delay writers' commits under the rollback journal;
- no authoritative listing universe or synthetic listing-eligibility source outside tests (findings A-8 and A-9).

## 8. Contract-finalization decisions (pending)

R-1 to R-12 are approved at the architectural level. The contracts below are engineering recommendations for the owner's decision. **None is approved by this record.**

| ID | Contract | Status |
| --- | --- | --- |
| EI7-CF1 | Session discovery contract | **PROPOSED — PENDING OWNER APPROVAL** |
| EI7-CF2 | API contract freeze | **PROPOSED — PENDING OWNER APPROVAL** |
| EI7-CF3 | Private access enforcement | **PROPOSED — PENDING OWNER APPROVAL** |
| EI7-CF4 | Integrity and lineage | **PROPOSED — PENDING OWNER APPROVAL** |
| EI7-CF5 | Database and ADR clarification | **PROPOSED — PENDING OWNER APPROVAL** (the ADR-024 clarification itself is recorded; section 6.1) |
| EI7-CF6 | Web navigation and response presentation | **PROPOSED — PENDING OWNER APPROVAL** |
| EI7-CF7 | Synthetic operator dependency | **PROPOSED — PENDING OWNER APPROVAL**; implementation **NOT AUTHORIZED** |

### 8.1 EI7-CF1: session discovery contract

- **Approved direction:** R-3.
- **Options for the limit:**
  - (a) reuse 300: rejected without independent justification; it bounds snapshots per issuance per monitoring run, a different resource;
  - (b) 50 per session;
  - (c) 10, exactly the proposed universe;
  - (d) unbounded: excluded by R-3.
- **Recommended contract:**
  - **Scope:** one exchange (NSE in V1) and one trading date per request; no strategy filter in V1.
  - **Limit:** at most **50** issuances per response, for both the declared and the recorded view.
  - **Sort key:** `(issued_at, issuance_id)` ascending, the order of the existing session-view rule (`equity_issuance_session_view`); `recording_sequence` is returned but is not the sort key.
  - **Overflow:** a typed refusal, `DISCOVERY_LIMIT_EXCEEDED`; never truncation and never a partial list.
  - **Pagination:** none in V1; one bounded page.
  - **Watermark:** every response returns the authority's `recording_watermark`. An optional `watermark` parameter reproduces an earlier view: the declared view through `session_view_at`; the recorded view as the issuances with `recording_sequence <= watermark`, with the declaration state at that watermark. A watermark beyond the authority's position is refused (`WATERMARK_AHEAD`).
  - **Declaration visibility:** a session-level `declaration` (`NONE`, or `DECLARED` with `complete_through`); in the recorded view, a per-item `covered_by_declaration` flag (`issued_at <= complete_through`). A session is never labelled "complete"; only "declared complete through" a stated instant.
  - **Lifecycle state per item:** included, read per item through `load_lifecycle`; a failure for one item is reported on that item, as EI-6C isolates issuances, while a failure of the issuance authority fails the whole response.
- **Rationale for 50:** an issuance is unique per strategy, listing and trading date (EI-6A), so a session holds at most strategies × listings issuances. V1 has one approved research strategy (EI3-D4) and a proposed universe of no more than 10 stocks (EI0-D5, PROPOSED). Fifty leaves five times that headroom without pagination, while capping the per-request cost of whole-ledger checks and per-item lifecycle reads. If EI0-D5 is approved with a larger universe, the limit must be revisited by a dated decision.
- **Compatibility impact:** an additive read port and adapter (EI-7C, EI-7D); no change to `EquityRecommendationIssuanceReader`, the EI-6A schema or EI-6C.
- **Acceptance tests:** EI7-N07, N15, N16, N17, N21.
- **Open questions:** the number; whether overflow maps to HTTP 422 or another status; whether a strategy filter is needed once a second strategy exists.
- **Status:** **PROPOSED — PENDING OWNER APPROVAL.**

### 8.2 EI7-CF2: API contract freeze

- **Approved direction:** R-1, R-2, R-9, R-10, R-11.
- **Options:**
  - endpoint set: all eight inventoried endpoints, or a minimal essential set with optional ones deferred;
  - error envelope: FastAPI's default `{"detail": …}`, or a typed envelope for equity routes;
  - versioning: a path version (`/equity/v1/…`), or an unversioned path with a contract identifier in every response.
- **Recommended contract:**
  - **Essential:** `GET /equity/health`; `GET /equity/sessions/{exchange}/{date}/recommendations`; `GET /equity/sessions/{exchange}/{date}/recorded-issuances`; `GET /equity/recommendations/{issuance_id}`, with the complete lifecycle timeline embedded; `GET /equity/research/{strategy}/{exchange}/{symbol}/{date}`.
  - **Optional or deferred:** `GET /equity/recommendations/lookup` (covered by the recorded view); `GET /equity/recommendations/{issuance_id}/lifecycle` (duplicates the detail's timeline); `GET /equity/snapshots/{snapshot_id}` (an optional evidence-inspection surface).
  - **Serializers:** API-owned response models mapped from the Application views; no Core or Application type is serialized directly; every response carries `contract: "northstar.equity-api/1"` and fixed labels (`evidence_class: "SYNTHETIC"`, `issuance_kind: "MODELED"` where an issuance is present, `actionable: false`).
  - **Error envelope:** `{"error": {"code", "message", "retryable", "scope"}}` on equity routes only; Futures routes unchanged; no traceback, SQLite text, file path or input echo.
  - **HTTP status mapping:** in [design section 12](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#12-errors).
  - **Precision and nulls:** prices, quantities and amounts as exact decimal strings; instants as canonical UTC `PointInTime` text; identities and enum names verbatim; `null` only for "not applicable" or "not stored", and an explicit status object, never `null`, for unknown or unavailable.
  - **Request validation:** exchange limited to NSE; ISO dates; symbols through Core `Symbol`; strategies through `StrategyIdentity`; issuance identities through `is_equity_recommendation_issuance_identity`; snapshot identities by their `ecs1-sha256:` form; watermarks as non-negative integers; a failure is a typed `UNSUPPORTED_SCOPE`.
  - **Versioning:** unversioned `/equity/` paths with the response `contract` identifier; any breaking change needs a new identifier and an owner decision.
- **Rationale:** the smallest set that supports the owner's journey and the negative research cases; error fidelity without leaking internals.
- **Compatibility impact:** API only; the Futures and legacy routes are unchanged.
- **Acceptance tests:** EI7-N05, N10 to N15, N17, N23, N24.
- **Open questions:** whether the optional endpoints ship in V1; the overflow and refusal statuses; the contract identifier's exact text.
- **Status:** **PROPOSED — PENDING OWNER APPROVAL.**

### 8.3 EI7-CF3: private access enforcement

- **Approved direction:** R-5.
- **Options:** the equity API in the existing FastAPI application with a separate router and runtime, or a separate process; the Equity file mounted read-only into the API container, or not.
- **Recommended contract:**
  - **Development bind:** `127.0.0.1` by default and configurable; never `0.0.0.0` by default outside a container (today `0.0.0.0:8000` is hard-coded).
  - **Topology:** browser → SSH tunnel → host loopback port → Caddy with basic authentication → API on an internal network with no published port → the Equity SQLite file, opened read-only.
  - **Authentication boundary:** Caddy basic authentication (bcrypt hash) on every equity path; no in-app token.
  - **Allow-list:** GET and HEAD only; one anchored pattern per equity route template, with restricted segment character classes; every other path or method returns 404; no catch-all prefix proxy for equity paths.
  - **Documentation routes:** `/docs`, `/redoc` and `/openapi.json` unreachable through Caddy and disabled in the equity deployment profile; the OpenAPI document is still produced in tests for the allow-list check.
  - **Security tests:** the allow-list matches the OpenAPI path templates exactly; unauthenticated requests are refused; non-GET methods are refused; documentation routes are unreachable; no equity route writes.
  - **Deployment is prohibited while:** basic authentication is absent; the API port is reachable beyond loopback or the internal network; documentation routes are reachable; any non-GET equity route exists; non-SYNTHETIC evidence could be displayed; the allow-list test fails; Futures or provider secrets reach the equity API; or the read-only checks of R-4 fail.
- **Rationale:** reuses the Futures boundary already documented and tested; adds the conditions that make "private by deployment" checkable.
- **Compatibility impact:** API and deployment files only, in a separately authorized slice. The CME stack, which proxies `/api/*` in full, must not expose equity routes unless they are allow-listed.
- **Acceptance tests:** EI7-N17, N18.
- **Open questions:** same process or separate process; read-only file mount; whether the CME stack serves equity at all.
- **Status:** **PROPOSED — PENDING OWNER APPROVAL.** No configuration change is authorized.

### 8.4 EI7-CF4: integrity and lineage

- **Approved direction:** R-6, R-12.
- **Recommended contract:**
  - **Required invariants of a recommendation detail:**
    1. `load_lifecycle(issuance_id)` returns a verified lifecycle;
    2. the disposition for the issuance's `generation_key` exists and is `PROPOSED`;
    3. its stored proposal and assessment decode, and their fingerprints equal the issuance's `proposal_fingerprint` and `assessment_fingerprint`;
    4. `equity_issued_recommendation(disposition)` reproduces the issued recommendation exactly, identity included;
    5. the disposition's policy-set version and fingerprint, research reference, strategy, listing, trading date and decision instant agree with the decoded proposal and the issuance.
  - **Corruption refusal:** any failure of 1 to 5 fails the whole response closed: `PERSISTED_AUTHORITY_INVALID`, or `LIFECYCLE_HISTORY_INVALID` for lifecycle history. No partial success that looks authoritative.
  - **Optional-field degradation:** supporting panels (the research origin matched by `candidate_entry_fingerprint`, the evidence snapshot summaries) report `available`, `unavailable` or `invalid` with a typed reason; they never fabricate data, and their failure does not hide the verified recommendation.
  - **Generation status without an issuance (EI7-N03):** derived only from stored records: the stored `CANDIDATE` timeline entry, the published policy set and `equity_recommendation_generation_key`, then `load_disposition`. This needs the policy-set version as an input; without it, or without a stored disposition, the status is UNKNOWN. Generation attempts without a disposition (`NO_CANDIDATE`, `DEPENDENCY_UNAVAILABLE`, `RESEARCH_INTEGRITY_FAILURE`) have no discovery read and are shown as UNKNOWN.
  - **Read consistency:** the issuance and lifecycle come from one read transaction; the disposition comes from a separate connection over insert-only rows, and the lineage check fails closed on any mismatch.
  - **Cross-connection limits:** no atomic view across endpoints or connections is claimed (EI7-N21).
- **Rationale:** proves lineage cheaply, with no generation replay, and keeps corruption distinct from absence.
- **Compatibility impact:** reads only; existing decoding functions reused; no contract change.
- **Acceptance tests:** EI7-N03, N04, N10, N11, N21, N24.
- **Open questions:** whether the policy-set version for N03 is a request parameter or configuration; whether a corrupt research origin should fail the detail rather than its panel.
- **Status:** **PROPOSED — PENDING OWNER APPROVAL.**

### 8.5 EI7-CF5: database and ADR clarification

- **Approved direction:** R-7.
- **Recorded now:** the dated clarification of ADR-024 section 14 (section 6.1), which changes no contract.
- **Recommended contract for the remaining points:**
  - **Configuration:** a separate Equity database setting, distinct from `NORTHSTAR_DATABASE`; the name, for example `NORTHSTAR_EQUITY_DATABASE`, is a proposal.
  - **Futures startup:** the Futures `initialize_database` is never called on the Equity file, and no equity code is called on the Futures file.
  - **Equity readers:** never create, initialize or migrate the file; a missing file or missing tables are a typed unavailability.
  - **Writers:** only the separately authorized operator path (EI7-CF7) writes the Equity file.
  - **Backups:** cover both files; no cross-file consistency is claimed.
- **Rationale:** the separation is approved; these points make it enforceable.
- **Compatibility impact:** none on existing contracts; no Equity database is deployed today, so nothing moves.
- **Acceptance tests:** EI7-N17.
- **Open questions:** the setting name; file location conventions; whether the API container mounts the file read-only.
- **Status:** **PROPOSED — PENDING OWNER APPROVAL** for the remaining points.

### 8.6 EI7-CF6: Web navigation and response presentation

- **Approved direction:** R-8, R-9, R-11, R-12.
- **Options:** extend the existing hash routing, or add a router library.
- **Recommended contract:**
  - **Deep links,** extending the hash routing, which is sufficient:
    - `#/equity` — session chooser;
    - `#/equity/sessions/NSE/{date}` — session recommendations, declared and recorded;
    - `#/equity/recommendations/{issuance_id}` — detail, with the identity URI-encoded;
    - `#/equity/research/{strategy}/NSE/{symbol}/{date}` — research.
    `#futures` stays unchanged. Refresh and back and forward work through the existing `hashchange` handling.
  - **Session selection:** no implicit "today" date; the owner chooses a date, and the last choice may be remembered in browser storage as a convenience.
  - **Detail layout:** header with listing, date, strategy, SYNTHETIC and MODELED badges and the current state; then panels for levels and validity, synthetic sizing and costs, provenance, rationale, lifecycle timeline and evidence.
  - **Timeline:** an ordered list by event sequence, each event showing the transition, its modeled time, the cited snapshot, the evidence interval, deficiencies, intrabar ordering and materiality; open states say that no later event is recorded and that monitoring runs are not recorded.
  - **Labels:** a persistent banner, "Synthetic evidence · MODELED issuance · Not a trade, order or fill"; "Entry activated (modeled)", never "executed" or "filled".
  - **Time:** IST visibly first; UTC visible as secondary text, not in a tooltip only; `<time datetime>` in UTC.
  - **Error and empty states:** a typed message per error code; "Not declared" and "Unknown" never shown as zero.
- **Rationale:** smallest change that gives stable URLs; reuses the Futures loading, card and test patterns.
- **Compatibility impact:** Web only. The legacy "Equity" asset-analysis view is a current-state analysis surface, not a recommendation surface (specification section 3.3), and should be labelled so.
- **Acceptance tests:** EI7-N06, N08, N09, N19, N23, N24.
- **Open questions:** whether the legacy view stays the default; the banner's exact wording.
- **Status:** **PROPOSED — PENDING OWNER APPROVAL.**

### 8.7 EI7-CF7: synthetic operator dependency

- **Approved direction:** R-1 (a synthetic operator CLI, governed separately).
- **Options:**
  - (a) an explicit operator CLI in northstar-api;
  - (b) test-only demonstration;
  - (c) a fixture-seeding script.
- **Recommended contract for a future governance decision:**
  - **Delivery:** an explicit CLI in northstar-api, following the existing CLI conventions (operations lock, redaction, exit codes).
  - **Behaviour:** builds the fixed synthetic journey over the one Equity file: ingestion, snapshots, research timeline, policy publication, declarations before generation and after issuance, generation recording, MODELED issuance, and monitoring of an explicit issuance at an explicit instant.
  - **Idempotency:** a rerun after restart is EQUIVALENT or produces no new events.
  - **Never:** an HTTP trigger, a scheduler or a clock read.
  - **Wiring:** writers composed only as EI4-D11 requires.
  - **Inputs:** a non-test synthetic listing-eligibility source in Infrastructure, and composition of the existing synthetic session and applicability references.
- **Rationale:** the only path to a repeatable demonstration database outside tests, without hidden autonomous behaviour.
- **Compatibility impact:** a write composition root, which needs its own architecture scans; EI-6C rules unchanged.
- **Acceptance tests:** EI7-N22 (conditional), N20.
- **Open questions:** the command set; the fixture world; where the eligibility source lives.
- **Status:** **PROPOSED — PENDING OWNER APPROVAL.** Implementation **NOT AUTHORIZED**.

## 9. Implementation authorization register

| Item | Repository | Status |
| --- | --- | --- |
| EI-7B Docs PR-0 (this register, the design and the dated notes) | northstar-docs | **AUTHORIZED, 2026-11-15** (documentation only); uncommitted for owner review |
| EI-7C Application read and query contracts | northstar-application | **NOT AUTHORIZED** |
| EI-7D Infrastructure read-only adapters | northstar-infrastructure | **NOT AUTHORIZED** |
| EI-7E Private equity API | northstar-api | **NOT AUTHORIZED** |
| EI-7F Equity Web experience | northstar-web | **NOT AUTHORIZED** |
| EI-7G Explicit synthetic journey operator, and the synthetic listing-eligibility source | northstar-api; northstar-infrastructure | **NOT AUTHORIZED** |
| EI-7H End-to-end synthetic acceptance | All | **NOT AUTHORIZED** |
| Caddy, bind or deployment change | northstar-api (`deploy/`) | **NOT AUTHORIZED** |
| Equity database creation, migration or file move | — | **NOT AUTHORIZED** |
| Scheduler or automated monitoring | — | **NOT AUTHORIZED** |
| Real or provider data; ACTUAL issuance; broker integration; orders or fills | — | **NOT AUTHORIZED** |
| EI-8 metrics; EI-8 restatement | — | **NOT AUTHORIZED** |
| EI-6C follow-ups F1 to F10 | — | **NOT AUTHORIZED** |

## 10. Acceptance scenario register

The 24 scenarios EI7-N01 to EI7-N24, their expected behaviours, verification layers and owning slices are defined in [design section 16](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#16-acceptance-scenarios).

| Count | Value |
| --- | --- |
| Scenarios | 24 (EI7-N01 to EI7-N24) |
| PLANNED — NOT TESTED | 24 |
| Conditional | EI7-N22, on separate authorization of the operator path (EI7-CF7) |
| Evidence recorded | None |

## 11. Decision-change and supersession rules

- An approved decision changes only through a later dated owner decision recorded in this register. The superseded text stays, marked by a dated note; it is never rewritten.
- A contract-finalization approval is recorded as a new dated section naming the exact approved contract. The design's proposed text is then marked by a dated note.
- Engineering evidence and acceptance records never change a decision's status.
- Earlier owner decisions (EI-3 to EI-6, EI4-D11, ADR-022 to ADR-025) prevail over anything here unless an owner decision expressly supersedes them. No R-n relaxes EI6C-P1 to EI6C-P12, EI4-D11, EI5-D1 or the EI-6A and EI-6B authorities.
- Python names in the design are proposals; an implementation PR may rename one by stating the reason, but may not change its semantics.

## 12. Related documents

- [EI-7 Private API and Web Design v1.0](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md)
- [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md), sections 3.3, 6.4, 11 and 12.1
- [EI-5 Decision Register](Equity-Intraday-V1-EI5-Decision-Register.md), sections 22 and 23
- [EI-6C Lifecycle Monitoring and Snapshot Selection Engineering Acceptance Record](Equity-Intraday-V1-EI6C-Lifecycle-Monitoring-and-Snapshot-Selection-Engineering-Acceptance-Record.md)
- [EI-6B Durable Recommendation Lifecycle Persistence Engineering Acceptance Record](Equity-Intraday-V1-EI6B-Durable-Recommendation-Lifecycle-Persistence-Engineering-Acceptance-Record.md)
- [EI-6A Durable Issuance Authority Engineering Acceptance Record](Equity-Intraday-V1-EI6A-Durable-Issuance-Authority-Engineering-Acceptance-Record.md)
- [EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md)
- [EI-4 Decision Register](Equity-Intraday-V1-EI4-Decision-Register.md)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md)
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)
- [EI4-D11 Operational Generation Write Boundary Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md)
- [Indian Futures Deployment Runbook](../operations/Indian-Futures-Deployment-Runbook.md), sections 1 and 8
