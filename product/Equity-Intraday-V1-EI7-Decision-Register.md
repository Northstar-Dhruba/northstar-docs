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

> **Note, 2026-11-21 (EI-7B Docs PR-1).** The bullets above record the state on 2026-11-16 and are not rewritten. Since then the owner has approved **EI7-CF1, EI7-CF2 and EI7-CF4 on 2026-11-20**, and the refinements **EI7-P1 to EI7-P8**, the evidence-class decision **EI7-EC1** and the final confirmations **EI7-FC1 to EI7-FC6 on 2026-11-21**. The governing contracts are in [section 13](#13-approved-contracts-ei7-cf1-ei7-cf2-and-ei7-cf4-2026-11-20-and-2026-11-21), which prevails over the proposal text of section 8 and of the design wherever they differ. EI7-CF3, EI7-CF5, EI7-CF6 and EI7-CF7 stay **PROPOSED — PENDING OWNER APPROVAL**. **No EI-7 implementation is authorized**, and EI7-N01 to EI7-N24 stay **PLANNED — NOT TESTED**.

> **Note, 2026-11-28 (EI-7E governance documentation PR).** The bullets and the note above are not rewritten. Since 2026-11-21: EI-7C and EI-7D were separately authorized by the owner on 2026-11-21 and merged (northstar-application #100, `16bb21a`, 2026-11-21; northstar-infrastructure #70, `caa2c46`, 2026-11-28). On **2026-11-28** the owner approved **EI7-CF3** (private access), **EI7-CF5** (Equity runtime and database), **EI7-SI1** (storage-engine integrity classification) and **EI7-SI2** (integrity-check coverage), recorded in [section 14](#14-approved-contracts-ei7-cf3-ei7-cf5-ei7-si1-and-ei7-si2-2026-11-28). These are contract approvals only. **SI-A, SI-B, EI-7E, EI-7F, EI-7G and EI-7H stay NOT AUTHORIZED**, as does any deployment change. EI7-CF6 and EI7-CF7 stay **PROPOSED — PENDING OWNER APPROVAL**. EI7-N01 to EI7-N24 stay **PLANNED — NOT TESTED**.

**Source of the decisions.** The owner's EI-7B instruction of 2026-11-15, which reproduces the approved directions. The owner's working document, `Northstar_EI7B_Governance_Decision_Register_OWNER_APPROVED_2026-11-15.md`, was not available in this repository or on the engineering machine when this record was prepared. The decisions below are recorded from the instruction text; nothing is added to them.

## 1. Purpose and identifiers

This register is the authoritative record of the owner's EI-7 decisions. It changes no earlier register: the EI-3, EI-4 and EI-5 registers keep their decisions, and the EI-6A, EI-6B and EI-6C acceptances are unchanged.

| Identifier | Meaning |
| --- | --- |
| R-1 to R-12 | The approved EI-7 architectural directions of section 4. Outside this register they are cited as **EI-7 R-1** to **EI-7 R-12**, so that they are not confused with the EI-4C contract points R-1 and R-2 settled by [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) |
| EI7-CF1 to EI7-CF7 | Contract-finalization decisions, pending (section 8). EI7-CF1, EI7-CF2 and EI7-CF4 approved 2026-11-20 (section 13) |
| EI7-P1 to EI7-P8 | Owner-approved refinements of EI7-CF1, EI7-CF2 and EI7-CF4, 2026-11-21 ([section 13.5](#135-refinements-ei7-p1-to-ei7-p8)). Cited with the `EI7-` prefix so that they are not confused with EI6C-P1 to EI6C-P12 |
| EI7-EC1 | Owner-approved SYNTHETIC evidence-class enforcement, 2026-11-21 ([section 13.6](#136-ei7-ec1-synthetic-evidence-class-enforcement)) |
| EI7-FC1 to EI7-FC6 | The owner's six final confirmations of 2026-11-21, labelled C1 to C6 in the owner's instruction ([section 13.12](#1312-final-owner-confirmations-ei7-fc1-to-ei7-fc6-2026-11-21)). Cited with the `EI7-FC` prefix so that they are not confused with the EI-6B carried-forward limitations C1 to C10 or the EI-5 conflicts C-1 and C-2 |
| EI7-N01 to EI7-N24 | Planned acceptance scenarios (section 10; design section 16) |
| EI-7C to EI-7H | Proposed engineering slices (design section 18) |
| EI7-CF3-A to EI7-CF3-H; EI7-CF5-A to EI7-CF5-H | The parts of the approved EI7-CF3 and EI7-CF5 contracts, 2026-11-28 ([sections 14.2](#142-ei7-cf3-approved-private-access-contract) and [14.3](#143-ei7-cf5-approved-equity-runtime-and-database-contract)); labelled CF3-A to CF3-H and CF5-A to CF5-H in the owner's instruction |
| EI7-SI1, EI7-SI2 | The owner's storage-integrity decisions of 2026-11-28, labelled SI-1 and SI-2 in the owner's instruction ([sections 14.4](#144-ei7-si1-approved-storage-engine-integrity-classification) and [14.5](#145-ei7-si2-approved-integrity-check-coverage)). Cited with the `EI7-` prefix, as EI7-P and EI7-FC are |
| SI-A, SI-B | The two implementation slices of EI7-SI1: SI-A in northstar-application, SI-B in northstar-infrastructure ([section 14.6](#146-implementation-order-and-gates)). Each needs a separate owner authorization |

| Status | Meaning |
| --- | --- |
| **APPROVED — 2026-11-15** | An owner-approved architectural direction |
| **APPROVED — 2026-11-20** | An owner-approved technical contract (EI7-CF1, EI7-CF2, EI7-CF4) |
| **APPROVED — 2026-11-21** | An owner-approved refinement or final confirmation of an approved contract (EI7-P1 to EI7-P8, EI7-EC1, EI7-FC1 to EI7-FC4, EI7-FC6) |
| **ACCEPTED — 2026-11-21** | The owner's acceptance of a stated fact about this record's sources (EI7-FC5) |
| **APPROVED — 2026-11-28** | An owner-approved technical contract (EI7-CF3, EI7-CF5, EI7-SI1, EI7-SI2) |
| **TECHNICAL CLOSURE** | An implementation detail the 2026-11-28 approvals leave to the named slice, which must settle it within the approved contract without a new architectural choice ([section 14.7](#147-technical-closure-items)) |
| **PENDING TECHNICAL DETAIL** | A detail the approvals explicitly left open, to be settled before the named slice ([section 13.10](#1310-pending-technical-details-and-outstanding-decisions)) |
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

> **Note, 2026-11-28.** The CF3 conditions are approved: [section 14.2](#142-ei7-cf3-approved-private-access-contract). This direction is unchanged; no Caddy, bind or deployment change is authorized by that approval.

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
| EI-6A declared-session history (`equity_issuance_session_view`; `EquitySessionRecommendationHistory`) | Unchanged, including its ascending `(issued_at, recommendation_key)` order (EI7-P1, 2026-11-21; section 13) |
| EI-4C versioned evaluation ([ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md)) | Unchanged: the closed evaluator table and the evaluator-`/2`-only issuance rule are preserved by EI7-CF4 (section 13.4) |
| EI3-D13 research rehydration (`rehydrate_equity_intraday_research_timeline`) | Decision semantics unchanged. EI7-P4 (2026-11-21) permits, within a separately authorized EI-7C only, a narrow reader-only Protocol adjustment of its snapshot dependency (section 13.5) |

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

> **Note, 2026-11-21.** The approved contracts add accepted limitations of their own; they are listed in [section 13.9](#139-accepted-limitations). Every limitation above stays in force.

> **Note, 2026-11-28.** EI7-CF3, EI7-CF5, EI7-SI1 and EI7-SI2 add the accepted limitations of [section 14.8](#148-accepted-limitations). Every limitation above and in section 13.9 stays in force.

## 8. Contract-finalization decisions (pending)

R-1 to R-12 are approved at the architectural level. The contracts below are engineering recommendations for the owner's decision. **None is approved by this record.**

> **Note, 2026-11-21 (EI-7B Docs PR-1).** This section records the proposals as prepared on 2026-11-16 and is not rewritten. The owner approved **EI7-CF1, EI7-CF2 and EI7-CF4 on 2026-11-20**, refined by EI7-P1 to EI7-P8 and EI7-EC1 on 2026-11-21. Their governing text is [section 13](#13-approved-contracts-ei7-cf1-ei7-cf2-and-ei7-cf4-2026-11-20-and-2026-11-21); the proposal text superseded there is listed in [section 13.7](#137-superseded-proposal-text). EI7-CF3, EI7-CF5, EI7-CF6 and EI7-CF7 below stay **PROPOSED — PENDING OWNER APPROVAL**.

| ID | Contract | Status |
| --- | --- | --- |
| EI7-CF1 | Session discovery contract | **PROPOSED — PENDING OWNER APPROVAL** (as prepared). **APPROVED — 2026-11-20**; [section 13.2](#132-ei7-cf1-approved-session-discovery-contract) |
| EI7-CF2 | API contract freeze | **PROPOSED — PENDING OWNER APPROVAL** (as prepared). **APPROVED — 2026-11-20**; [section 13.3](#133-ei7-cf2-approved-private-read-api-contract) |
| EI7-CF3 | Private access enforcement | **PROPOSED — PENDING OWNER APPROVAL** (as prepared). **APPROVED — 2026-11-28**; [section 14.2](#142-ei7-cf3-approved-private-access-contract) |
| EI7-CF4 | Integrity and lineage | **PROPOSED — PENDING OWNER APPROVAL** (as prepared). **APPROVED — 2026-11-20**; [section 13.4](#134-ei7-cf4-approved-integrity-lineage-and-research-status-contract) |
| EI7-CF5 | Database and ADR clarification | **PROPOSED — PENDING OWNER APPROVAL** (the ADR-024 clarification itself is recorded; section 6.1) (as prepared). **APPROVED — 2026-11-28**; [section 14.3](#143-ei7-cf5-approved-equity-runtime-and-database-contract) |
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

> **Note, 2026-11-21.** **APPROVED — 2026-11-20**, refined 2026-11-21; the governing contract is [section 13.2](#132-ei7-cf1-approved-session-discovery-contract). Superseded or settled here: the recorded view is ordered by ascending `recording_sequence`, while the declared view keeps the `(issued_at, issuance_id)` order stated above (EI7-P1); the limit is 50 (approved); overflow is HTTP 409 (EI7-P2); every response states the recording position and the selected watermark, undeclared sessions included (section 13.2). The strategy-filter question stays open for a future strategy.

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

> **Note, 2026-11-21.** **APPROVED — 2026-11-20**, refined 2026-11-21; the governing contract is [section 13.3](#133-ei7-cf2-approved-private-read-api-contract). Superseded or settled here: the identifier is `contract_id: "northstar.equity-private-read/1"`, not `contract: "northstar.equity-api/1"` (EI7-P8); every integer, not only prices, quantities and amounts, is a canonical decimal string, and the Equity read contract has no JSON numbers; overflow, watermark-ahead and evidence-class refusals are HTTP 409 (EI7-P2, EI7-P7, EI7-EC1); the three optional endpoints stay deferred.

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

> **Note, 2026-11-28.** **APPROVED — 2026-11-28**; the governing contract is [section 14.2](#142-ei7-cf3-approved-private-access-contract). Superseded or settled here: HEAD is permitted only where EI-7E explicitly supports it and verifies it as non-mutating, otherwise it is denied (EI7-CF3-D); the documentation routes must be disabled or unreachable at the application boundary, not only at Caddy (EI7-CF3-E); the route encoding of `StrategyIdentity` is canonical base64url of its UTF-8 text in one segment (EI7-CF3-F). "Never `0.0.0.0` by default outside a container" stands; how the loopback-only model applies to the container topology is a technical closure item of EI-7E (section 14.7). The open questions on a separate process, a read-only file mount and the CME stack are EI-7E closure items, bounded by EI7-CF3-B and the stop conditions of section 14.6. Still **NOT AUTHORIZED**: any configuration or deployment change.

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

> **Note, 2026-11-21.** **APPROVED — 2026-11-20**, refined 2026-11-21; the governing contract is [section 13.4](#134-ei7-cf4-approved-integrity-lineage-and-research-status-contract). Superseded or settled here: generation status without an issuance is not derived from a policy-set version in EI-7C, neither by request parameter nor by configuration; it is `NOT_QUERYABLE`, distinct from UNKNOWN (EI7-P3). A corrupt primary research timeline fails the whole research response; in a recommendation detail, optional research context degrades to a panel only while the required lineage stays authoritative (EI7-P5). A non-SYNTHETIC snapshot refuses the affected response with HTTP 409 and is never reported as a panel (EI7-EC1).

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

> **Note, 2026-11-28.** **APPROVED — 2026-11-28**; the governing contract is [section 14.3](#143-ei7-cf5-approved-equity-runtime-and-database-contract). Superseded or settled here: the proposed setting name is `NORTHSTAR_EQUITY_SQLITE_PATH`, replacing the example `NORTHSTAR_EQUITY_DATABASE` (EI7-CF5-B); an unconfigured runtime answers `EQUITY_NOT_CONFIGURED`, HTTP 503, not retryable (EI7-CF5-D; EI7-FC4); the initial operating model is the rollback journal, and WAL-mode files are refused (EI7-CF5-F). "Only the separately authorized operator path (EI7-CF7) writes the Equity file" stands. File location conventions and a read-only file mount are EI-7E closure items (section 14.7).

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

> **Note, 2026-11-21.** Rows added by EI-7B Docs PR-1. The rows above are unchanged: the approval of EI7-CF1, EI7-CF2 and EI7-CF4 satisfies a contract dependency of EI-7C, EI-7D and EI-7E but authorizes none of them.

| Item | Repository | Status |
| --- | --- | --- |
| EI-7B Docs PR-1 (record of EI7-CF1, EI7-CF2, EI7-CF4, EI7-P1 to EI7-P8 and EI7-EC1) | northstar-docs | **AUTHORIZED, 2026-11-21** (documentation only); uncommitted for owner review |
| EI7-P4 reader-only Protocol adjustment of the EI-3C research rehydration dependency | northstar-application | **NOT AUTHORIZED** by EI-7B Docs PR-1; eligible for implementation within a separately owner-authorized EI-7C Application slice (EI7-FC6) |
| Policy-set-based generation lookup or any generation-status search by candidate, policy set, strategy or date | — | **NOT AUTHORIZED**; deferred by EI7-P3 to separate governance |
| `EquitySyntheticPolicyCatalogReader` or any catalog read in an EI-7 read path | northstar-application; northstar-infrastructure | **NOT AUTHORIZED**; deferred with the lookup above |
| Giving the recorded-session discovery port to EI-6C monitoring | — | **NOT AUTHORIZED** (EI6C-P7 unchanged) |
| Synthetic operator CLI | northstar-api | **NOT AUTHORIZED** (EI7-CF7 pending; EI-7G row above) |

> **Note, 2026-11-28.** Rows added by the EI-7E governance documentation PR. The rows above are not rewritten; where they say EI-7C or EI-7D is not authorized, the rows below record the later state.

| Item | Repository | Status |
| --- | --- | --- |
| EI-7C Application read and query contracts | northstar-application | Authorized by the owner on **2026-11-21**; merged as #100 (`16bb21a`, 2026-11-21). No engineering acceptance record is in this repository; its tests are in the merged suite |
| EI-7D Infrastructure read-only adapters | northstar-infrastructure | Authorized by the owner on **2026-11-21**; merged as #70 (`caa2c46`, 2026-11-28). No engineering acceptance record is in this repository; its tests are in the merged suite |
| EI-7E governance documentation PR (this record of EI7-CF3, EI7-CF5, EI7-SI1 and EI7-SI2) | northstar-docs | **AUTHORIZED, 2026-11-28** (documentation only); uncommitted for owner review |
| SI-A Application storage-integrity compatibility | northstar-application | **NOT AUTHORIZED** |
| SI-B Infrastructure storage-integrity compatibility | northstar-infrastructure | **NOT AUTHORIZED** |
| EI-7E Private equity API | northstar-api | **NOT AUTHORIZED**; its contract dependencies EI7-CF3 and EI7-CF5 are approved; it also waits for SI-A and SI-B (section 14.6) |
| EI-7F Equity Web experience | northstar-web | **NOT AUTHORIZED**; EI7-CF6 pending |
| Caddy, bind, CORS, documentation-route, compose or other deployment change | northstar-api | **NOT AUTHORIZED** (approved contracts only) |
| Automated, background, scheduled or startup integrity checking or repair | — | **NOT AUTHORIZED**; separate governance required (EI7-SI2) |

## 10. Acceptance scenario register

The 24 scenarios EI7-N01 to EI7-N24, their expected behaviours, verification layers and owning slices are defined in [design section 16](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#16-acceptance-scenarios).

| Count | Value |
| --- | --- |
| Scenarios | 24 (EI7-N01 to EI7-N24) |
| PLANNED — NOT TESTED | 24 |
| Conditional | EI7-N22, on separate authorization of the operator path (EI7-CF7) |
| Evidence recorded | None |

> **Note, 2026-11-21.** The scenarios were reconciled with the approved contracts in [design section 23.8](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#238-acceptance-reconciliation). No scenario was added or removed; the revised expected behaviours of EI7-N03, N04, N07, N10, N11, N14, N15 and N16 and the required coverage items are recorded there. All 24 stay **PLANNED — NOT TESTED**.

> **Note, 2026-11-28.** The scenarios were reconciled again with EI7-CF3, EI7-CF5, EI7-SI1 and EI7-SI2 in [design section 24.14](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#2414-acceptance-reconciliation), which names the slice that must evidence each item (SI-A, SI-B, EI-7E, EI-7F or EI-7H). No scenario is added, removed or renumbered. The merged EI-7C and EI-7D suites contain layer-level tests, but no acceptance record of either slice exists here, so all 24 stay **PLANNED — NOT TESTED**.

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

## 13. Approved contracts EI7-CF1, EI7-CF2 and EI7-CF4 (2026-11-20 and 2026-11-21)

This section records owner approvals given after sections 1 to 12 were written. Sections 1 to 12 are not rewritten; dated notes there point here. Where the proposal text of section 8 or of the [design](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md) differs from this section, **this section governs**. Python names and response field names remain proposals (section 11); the semantics below are binding. **Nothing here is implemented, and nothing here authorizes an implementation.**

### 13.1 Approval record

| Date | Decision | Status |
| --- | --- | --- |
| 2026-11-15 | R-1 to R-12 (section 4) | **APPROVED — 2026-11-15**, unchanged |
| 2026-11-20 | EI7-CF1, EI7-CF2 and EI7-CF4, approved as recommended, preserving the explicitly pending technical details | **APPROVED — 2026-11-20** |
| 2026-11-21 | EI7-P1 to EI7-P8, as refined by the owner | **APPROVED — 2026-11-21** |
| 2026-11-21 | EI7-EC1, SYNTHETIC evidence-class enforcement | **APPROVED — 2026-11-21** |
| 2026-11-21 | EI7-FC1 to EI7-FC6, the owner's final confirmations (section 13.12) | **APPROVED — 2026-11-21** (EI7-FC5 **ACCEPTED — 2026-11-21**) |
| — | EI7-CF3, EI7-CF5, EI7-CF6 and EI7-CF7 | **PROPOSED — PENDING OWNER APPROVAL** |

- **Sources** (as accepted by the owner under EI7-FC5): the owner's explicit EI-7B decisions of 2026-11-20 (the CF1, CF2 and CF4 final contract feasibility audit instruction) and of 2026-11-21 (the EI-7B Docs PR-1 instruction and its finalization instruction); the source-feasibility audit; the merged EI-7B governance (this register and the design as merged at `d9bc867`); and the current source contracts verified at the commits of section 3. The owner-approved CF contract package is not present in the six local repositories. It was not opened, read or committed in preparing this record, and nothing here is attributed to its text. Its absence does not revoke the owner's decisions, which are recorded from the instruction text.
- **Engineering evidence.** The EI-7B CF1, CF2 and CF4 read-only feasibility audit (2026-11-20) read all six repositories at the code commits of section 3, with northstar-docs at `d9bc867`. It found the contracts implementable without a schema change, a new Core model, a new write authority or any weakening of EI-6A verification, given the additive read-only ports of section 13.8. It ran no tests; its findings rest on source inspection only. Two of its recommendations were not adopted: the identifier `northstar.equity-api/1` (EI7-P8) and a panel-level evidence-class refusal (EI7-EC1).
- **Mechanics.** The read sequence, presentation derivation, wire examples, use-case inventory and acceptance reconciliation are in [design section 23](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#23-approved-contract-reconciliation-2026-11-21).

> **Note, 2026-11-28.** The last row of the table above records the state on 2026-11-21. EI7-CF3 and EI7-CF5 were approved on 2026-11-28 ([section 14.1](#141-approval-record)); EI7-CF6 and EI7-CF7 stay pending.

### 13.2 EI7-CF1: approved session discovery contract

**APPROVED — 2026-11-20**; refined by EI7-P1, EI7-P2, EI7-P6 and EI7-P7 on 2026-11-21.

1. **Scope.** One exchange (NSE in V1) and one trading date per request. No strategy filter, no global list-all, no date-range or cross-session discovery, no pagination and no continuation.
2. **Bound.** At most **50** recorded issuances for the session at the selected watermark. The bound is fixed for `northstar.equity-private-read/1`; changing it needs a dated owner decision, and it must be revisited if EI0-D5 is approved with a larger universe. It restricts the new EI-7 presentation and query layer only, never the historical issuance authority: EI-6A records any number of issuances, and its `session_view` behaviour and canonical order are unchanged (EI7-FC1).
3. **Complete or refused.** A successful response holds the complete, fully verified set. Otherwise **both** new EI-7 session projections, recorded and declared, refuse with HTTP 409 `DISCOVERY_LIMIT_EXCEEDED`: no partial authoritative list and no silent truncation (EI7-P2; EI7-FC1). An overflow refusal is **not a corruption verdict**: it says nothing about the integrity of the session's records. The refusal states the limit. It need not state a total count, and states one only where the count is established authoritatively.
4. **Two presentations, one read.** Both presentations are derived from one bounded, verified read of the session at one recording watermark W, and each keeps its own order (EI7-P1):
   - **recorded view** (new): every recorded issuance of the session with `recording_sequence <= W`, in ascending `recording_sequence`;
   - **declared view** (existing EI-6A semantics): exactly the issuances that the existing rule `equity_issuance_session_view` admits at W (`recording_sequence <= W` and `issued_at <= complete_through(W)`), in that rule's existing order, ascending `(issued_at, recommendation_key)`. An EI-6A record's recommendation key is its `issuance_id`, so this is the `(issued_at, issuance_id)` order of section 8.1.

   The two orders can differ: two issuances at one instant are ordered by identity, not by sequence (Infrastructure fixture `test_hw03_s_a_view_keeps_its_content_after_later_recording`).
5. **Watermark.** Every response states the authority's recording position N at the read and the selected recording watermark W (N when none is requested). An optional `watermark` request value must satisfy W ≤ N; otherwise HTTP 409 `WATERMARK_AHEAD`. A syntactically valid non-negative decimal above 2^63−1 is ahead of the authority by definition and is refused with 409 `WATERMARK_AHEAD` before it is bound into SQLite (EI7-P7). Malformed, signed or non-canonical values are request-validation failures, HTTP 422 `UNSUPPORTED_SCOPE`.
6. **Declaration at the same watermark.** The declaration state is evaluated at W by the existing EI-6A rule `equity_declared_complete_through`: `NONE`, or `DECLARED` with `complete_through`. An undeclared session is a successful response with `declaration: NONE` and N and W stated. It is not an error, not "no recommendations" and not complete. Recorded-view items carry `covered_by_declaration` (`issued_at <= complete_through(W)`). A session is only ever "declared complete through" a stated instant.
7. **Discovery content and lifecycle per item** (list and detail consistency, 2026-11-21). The authoritative content of a discovery response is the issuance identities and the session recording facts at W: position, watermark, declaration state, recording sequences and coverage. The full lifecycle belongs to the verified recommendation detail. An item may carry an optional lifecycle summary, read by an **independent** `load_lifecycle` call per item, after the session read. Those summaries are not pinned to W, are not atomic with the session read or with one another, and must never be presented as one globally consistent lifecycle observation. Each summary carries its own status; a failed summary is shown as that item's `invalid` or `unavailable` status, never omitted, defaulted or disguised as a successful observation. An issuance-authority failure fails the whole response. The EI-6A and EI-6B transaction semantics are unchanged.
8. **Verification and precedence (EI7-P6).** In order: (1) the whole-ledger check; (2) the recording position and the applicable watermark; (3) the declaration state; (4) the session's bounded recorded count at W; (5) above 50, the typed overflow refusal, before any per-issuance re-derivation; (6) otherwise every listed issuance re-derived and verified exactly as EI-6A verifies it today. A LEDGER failure always takes precedence over overflow. A successful response is never unverified.
9. **EI-6A unchanged.** `EquityRecommendationIssuanceReader`, `session_view`, `session_view_at`, `EquitySessionRecommendationHistory` and their ordering invariants do not change. Today `session_view` returns `None`, with no watermark, for an undeclared session; the new read contract corrects that presentation gap without changing the EI-6A authority.
10. **EI-6C unchanged.** The monitor is never given the discovery port; EI6C-P7 still governs monitoring discovery.

### 13.3 EI7-CF2: approved private read API contract

**APPROVED — 2026-11-20**; refined by EI7-P2, EI7-P7, EI7-P8 and EI7-EC1 on 2026-11-21.

1. **Five essential GET route families:**
   - `GET /equity/health`;
   - `GET /equity/sessions/{exchange}/{date}/recommendations` (declared view);
   - `GET /equity/sessions/{exchange}/{date}/recorded-issuances` (recorded view);
   - `GET /equity/recommendations/{issuance_id}` (verified detail, including the complete authoritative lifecycle timeline);
   - `GET /equity/research/{strategy}/{exchange}/{symbol}/{date}`.

   **Deferred**, each needing a later dated decision: the lookup endpoint, a separate lifecycle endpoint and a generic snapshot endpoint.
2. **Ownership.** Immutable, Application-owned query views. The Application classifies every domain error. API response models are mapped from the views. Equity route modules import no Infrastructure type and catch no Infrastructure error.
3. **Contract identifier.** `contract_id: "northstar.equity-private-read/1"` at the **top level** of every equity success and error envelope, in every example and every contract test (EI7-P8; EI7-FC4). No other identifier is used.
4. **Wire rules.**
   - Every integer and decimal value is a string. No JSON number appears anywhere in the Equity V1 read contract.
   - Integer strings are canonical non-negative base 10: `0`, or a non-zero digit followed by digits, with no sign, leading zero, exponent or separator. This covers recording positions, watermarks, recording sequences, lifecycle positions and event sequences, timeline sequences, quantities and counts. None may lose precision in JavaScript, whose exact range ends at 2^53−1 (9007199254740991), below the SQLite INTEGER maximum of 2^63−1.
   - Decimals (prices, amounts, multipliers) are the exact canonical decimal text of the domain value, never a binary float.
   - Instants are canonical UTC `PointInTime` text; dates are ISO `YYYY-MM-DD`; identifiers and enumeration names are opaque, verbatim strings.
   - Unknown, unavailable, not declared and not queryable are explicit, typed states, never `null`, zero or an empty list. `null` means only "not applicable" or "not stored".
   - No rationale prose is invented from structured data. No `recorded_at`, `last_monitored_at`, fill, execution price or performance figure is present or fabricated.
   - Fixed labels: `evidence_class: "SYNTHETIC"`; `issuance_kind: "MODELED"` and `actionable: false` wherever an issuance is present.
5. **Error envelope.** On equity routes only: `{"contract_id": "northstar.equity-private-read/1", "error": {"code", "message", "retryable", "scope"}}`. No traceback, SQLite text, file path, configuration value or raw request input is returned. Equity request validation must not use a handler that echoes input. Futures and legacy routes are unchanged.
6. **Non-mutating.** GET and HEAD only. No DDL or schema initialization during a request or at equity startup. No generation, issuance, declaration, monitoring, scheduling or clock read. No broker execution and no performance claim.
7. **Route encoding.** `StrategyIdentity` is opaque text and may contain characters, such as `/`, that do not survive as one path segment. The route encoding is a pending technical detail to verify before EI-7E (section 13.10). This record imposes no new, incompatible restriction on strategy identities.

**Status mapping:**

| Code | HTTP | Retryable | Meaning |
| --- | --- | --- | --- |
| `ISSUANCE_NOT_FOUND` | 404 | No | The authority holds no issuance with the requested identity |
| `WATERMARK_AHEAD` | 409 | No | The requested watermark is above the recording position, including any value above 2^63−1 (EI7-P7) |
| `DISCOVERY_LIMIT_EXCEEDED` | 409 | No | More than 50 recorded issuances at the watermark (EI7-P2) |
| `EVIDENCE_CLASS_UNSUPPORTED` | 409 | No | A retrieved snapshot's class is not SYNTHETIC (EI7-EC1) |
| `UNSUPPORTED_SCOPE` | 422 | No | A malformed or unsupported request: exchange, date, symbol, strategy, identity or watermark syntax |
| `PERSISTED_AUTHORITY_INVALID` | 500 | No | A corrupt ledger, declaration or issuance; a failed lineage comparison; a missing or non-`PROPOSED` disposition for an issuance; an unverifiable primary research timeline |
| `LIFECYCLE_HISTORY_INVALID` | 500 | No | Corrupt lifecycle history: fails a detail response; an item status in the lists |
| `DEPENDENCY_UNAVAILABLE` | 503 | Yes | A required reader is unavailable, including missing tables or an unrecoverable hot journal under `mode=ro` |
| `EQUITY_NOT_CONFIGURED` | 503 | No | The Equity read runtime is not configured, when applicable; the exact trigger depends on EI7-CF5 |

- Further source-supported distinctions may be kept where needed, such as the panel statuses of section 13.6.
- **Not errors:** a session with `declaration: NONE` (HTTP 200); a generation status of `NOT_QUERYABLE`, which is never an issuance failure; an UNKNOWN research timeline.

> **Note, 2026-11-28.** Item 7 and the `EQUITY_NOT_CONFIGURED` row are settled at the contract level by [section 14](#14-approved-contracts-ei7-cf3-ei7-cf5-ei7-si1-and-ei7-si2-2026-11-28): the route encoding is canonical base64url of the identity's UTF-8 text, with no new restriction on identities (EI7-CF3-F); `EQUITY_NOT_CONFIGURED` is triggered by an absent Equity database setting (EI7-CF5-D). `PERSISTED_AUTHORITY_INVALID` also covers a permanent SQLite storage-integrity failure (`SQLITE_CORRUPT`, `SQLITE_NOTADB`) through the Application-owned contract of EI7-SI1; no code, status or retryability in the table changes, and no code is added. The five route families and the HEAD rule are refined by EI7-CF3-D: HEAD only where explicitly supported and verified as non-mutating.

### 13.4 EI7-CF4: approved integrity, lineage and research-status contract

**APPROVED — 2026-11-20**; refined by EI7-P3, EI7-P5 and EI7-EC1 on 2026-11-21.

**Source-verified facts (feasibility audit, 2026-11-20):**

- The supported generation evaluators are a closed table, `northstar.equity-recommendation-evaluator/1` and `/2` ([ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md)). Issuance accepts only `/2`, enforced by a database trigger and by issuance re-derivation on every read. Nothing is migrated; the frozen EI-4C compatibility rules are preserved.
- The generation key is `eirk1`. It binds the strategy, research configuration version, listing, trading date, candidate entry fingerprint, policy-set version and `eirs1` policy-set fingerprint; the evaluator version is not part of it. The issuance identity is `eiris1`, binding the generation key, the proposal fingerprint and MODELED.
- The issued recommendation is reconstructed canonically by `equity_issued_recommendation(disposition)`.
- `load_lifecycle` already checks the whole ledger, re-derives the issuance from its stored disposition and replays the lifecycle in one read transaction, but does not return the disposition. A separate exact-key `load_disposition(issuance.generation_key)` supplies the structured sizing, version and research-reference facts.

**Recommendation detail: required comparisons.** Each must hold, or the whole detail fails with `PERSISTED_AUTHORITY_INVALID` (`LIFECYCLE_HISTORY_INVALID` for lifecycle history):

| # | Comparison |
| --- | --- |
| 1 | The issuance's generation key equals the exact-key disposition's key; the disposition exists and is `PROPOSED` |
| 2 | Strategy |
| 3 | Listing |
| 4 | Trading date |
| 5 | Policy-set version and `eirs1` fingerprint, and the component, timing, listing-eligibility, universe and research-configuration versions |
| 6 | Research candidate reference: decision instant, research decision fingerprint, candidate entry fingerprint, evidence snapshot |
| 7 | Levels and modeled timing |
| 8 | Sizing and risk definition |
| 9 | Evaluator compatibility: the disposition's evaluator is `/2` |
| 10 | `equity_issued_recommendation(disposition)` equals the recorded issued recommendation, identity included |

- **No re-evaluation.** A GET proves the lineage and integrity of stored facts. It does not re-evaluate market history or replay the generation (R-6).
- **Consistency.** The issuance and lifecycle come from one read transaction. The disposition comes from a separate connection over insert-only rows, guarded by comparisons 1 to 10. No cross-connection atomicity is claimed.
- **Exact identity only.** No latest or current policy-set version is inferred, configured or guessed. There is no broad generation-status search.
- **Provenance limits**, stated in the source-of-truth rules and in every presentation of these facts:
  - the historical publication of the policy set before generation cannot be established from persisted facts, because publication history is not stored; the detail shows the bound policy identity, never a publication time;
  - the authenticity of the research candidate against its timeline is established only by the accepted EI-3 rehydration verification (EI3-D13). Without it, the detail shows the frozen candidate reference bound by `eirk1`, labelled as such.

**Research view (EI7-P3, EI7-P5):**

- The persisted research timeline is loaded and verified by EI3-D13 rehydration. If it cannot be verified or rehydrated, the whole research response fails with a typed integrity error; it is never downgraded to UNKNOWN.
- Rehydration uses the stored, authoritative configuration document, never a current configuration or a client-supplied value. A missing or inconsistent required configuration fails the whole research response with a typed integrity error (EI7-FC3).
- Every snapshot retrieved for the research response is gated on its evidence class **before** any research outcome is presented. A non-SYNTHETIC snapshot refuses the whole response with HTTP 409 `EVIDENCE_CLASS_UNSUPPORTED`. The existing evaluator's reconstruction of such a snapshot as a `REJECTED` decision is never presented as an ordinary research outcome, and no evidence payload is exposed (EI7-FC3).
- Research outcomes and reasons are shown distinctly, as stored; persisted `NO_SIGNAL` and `REJECTED` outcomes show their authoritative reasons (EI7-FC2).
- When the authority records an issuance for the strategy, listing and date, its exact generation key is used. The disposition must exist, be `PROPOSED` and cite the timeline's candidate entry; otherwise the response fails with `PERSISTED_AUTHORITY_INVALID`.
- Otherwise the generation status is `NOT_QUERYABLE`, with a typed reason. No policy-set version is accepted as a request parameter or taken from configuration, and no disposition or attempt is scanned.
- `NOT_ELIGIBLE` and `UNRESOLVED` are never inferred. They may be shown only when their exact dispositions are authoritatively discoverable; in the EI-7C scope no authorized exact lookup reaches them, so they are not shown (EI7-FC2).
- The absence of generation attempts is never inferred: `NOT_QUERYABLE` says that no lookup was possible, not that nothing was attempted (EI7-FC2).
- The statuses UNKNOWN, `NOT_QUERYABLE`, `PROPOSED`, `NOT_ELIGIBLE`, `UNRESOLVED` and `ISSUED` stay distinct in types, responses and labels.

**UNKNOWN and NOT_QUERYABLE are distinct:**

| State | Meaning |
| --- | --- |
| UNKNOWN | An authorized exact lookup was performed and the authoritative source holds no record; for example, no persisted research timeline for the strategy, listing and date |
| `NOT_QUERYABLE` | The exact identity an authorized lookup needs is not available within the approved scope, so no lookup was performed; the response states why. Not an error and not an issuance failure |

An exact-key lookup that finds nothing means UNKNOWN only where that exact lookup is authorized and available. For an issued recommendation, a missing required disposition is a persisted-authority integrity failure, never UNKNOWN.

### 13.5 Refinements EI7-P1 to EI7-P8

**APPROVED — 2026-11-21.**

| ID | Decision | Supersedes or settles |
| --- | --- | --- |
| EI7-P1 | Recorded view in ascending `recording_sequence`. Declared view in the existing EI-6A order, ascending `(issued_at, recommendation_key)`, the key being the `issuance_id`; the EI-6A history invariants are unchanged. Both may be derived from one verified recording watermark; each keeps its order. Acceptance tests must prove the distinction | The section 8.1 sort key, for the recorded view |
| EI7-P2 | Overflow is HTTP 409 `DISCOVERY_LIMIT_EXCEEDED`: no partial list, no silent truncation; an exact total need not be exposed if it cannot be established authoritatively. The limit of 50 is fixed for this contract version | The section 8.1 open status question; design section 12, "422 (open)" |
| EI7-P3 | Policy-set-based generation lookup is deferred. The EI-7C research view supports a verified persisted timeline, the exact issuance-linked generation identity when an authoritative issuance exists, `NOT_QUERYABLE` otherwise, and distinct research outcomes and reasons. No policy-set request parameter; no scan of dispositions or attempts; no guessed current or latest version. Broader generation discovery needs separate governance | Section 8.4, generation status without an issuance, and its open question |
| EI7-P4 | A narrowly scoped, reader-only Protocol for the snapshot dependency of the existing research rehydration, exposing only the read it needs. No fake `issue()` or other write method to satisfy an existing ABC; no change to the write-side contract or to research decision semantics. Whether the existing runtime `isinstance` validation can safely accept the structural Protocol, preserving every writer-capable implementation, is verified within a separately authorized EI-7C, which alone may make the limited change to EI-3C Application code | New |
| EI7-P5 | A primary research timeline that cannot be verified or rehydrated fails the whole research response with a typed integrity error. In a recommendation detail, optional research context may show an `invalid` or `unavailable` panel only while the required issuance, lifecycle and generation lineage stays authoritative; evidence required for identity or lineage fails the whole detail. No invented provenance; required corruption is never UNKNOWN | The section 8.4 open question on research-origin corruption |
| EI7-P6 | Discovery precedence: ledger; position and watermark; declaration state; bounded count; overflow before per-issuance re-derivation; otherwise complete verification. A LEDGER failure precedes overflow. Accepted limitation: overflow may be returned before an individual row's corruption is discovered; a success is never unverified; EI-6A verification of successful reads is not weakened | New |
| EI7-P7 | A syntactically valid non-negative decimal watermark above 2^63−1 is ahead of the recorded authority: HTTP 409 `WATERMARK_AHEAD`, refused before SQLite binding, so no Python `OverflowError` escapes. Malformed, signed, non-canonical or unsupported syntax receives the request-validation error, HTTP 422 | Section 8.2, "watermarks as non-negative integers" (refined) |
| EI7-P8 | The contract identifier is exactly `northstar.equity-private-read/1`, used consistently in response and error contracts, examples and contract tests | Section 8.2, `contract: "northstar.equity-api/1"`, and the feasibility audit's identical suggestion |

### 13.6 EI7-EC1: SYNTHETIC evidence-class enforcement

**APPROVED — 2026-11-21.** A refinement of R-10, enforced server-side:

| Condition | Result |
| --- | --- |
| A cited optional snapshot cannot be found | An explicit `unavailable` evidence panel (`EVIDENCE_NOT_FOUND`), only while every mandatory issuance, lifecycle and generation fact is verified |
| A cited ancillary snapshot is corrupt, and the corruption does not compromise required identity, lineage or domain authority | An explicit `invalid` evidence panel; its content withheld |
| Corrupt evidence compromises required provenance | The whole response fails (`PERSISTED_AUTHORITY_INVALID`) |
| A snapshot is retrieved and its class is not SYNTHETIC | The affected response is refused with HTTP 409 `EVIDENCE_CLASS_UNSUPPORTED`. Its contents are not exposed, its class is never converted to SYNTHETIC, and it is never presented as a missing optional snapshot |

- An evidence-class refusal stays distinguishable from persisted-authority corruption (500) and from a missing snapshot (panel).
- It is an evidence policy, not an access decision. The user-authentication boundary is EI7-CF3 and is unaffected.

### 13.7 Superseded proposal text

The text below stays in place, marked by dated notes, and no longer governs.

| Location | Superseded text | Governing text |
| --- | --- | --- |
| Section 8.1, sort key | `(issued_at, issuance_id)` for both views; `recording_sequence` not the sort key | EI7-P1: the recorded view by `recording_sequence`; the declared view unchanged |
| Section 8.1, open questions | The limit; the overflow status | 50; HTTP 409 (EI7-P2) |
| Section 8.2, serializers | `contract: "northstar.equity-api/1"` | `contract_id: "northstar.equity-private-read/1"` (EI7-P8) |
| Section 8.2, precision | Exact decimal strings for prices, quantities and amounts | Every integer and decimal a string; no JSON numbers (section 13.3) |
| Section 8.2, request validation | Watermarks as non-negative integers | Canonical decimal strings; above 2^63−1 is 409 (EI7-P7) |
| Section 8.4, generation status without an issuance | Derived from a policy-set version; otherwise UNKNOWN | `NOT_QUERYABLE` unless an issuance-linked key exists (EI7-P3) |
| Section 8.4, optional-field degradation | Research-origin and snapshot panels report `invalid` or `unavailable` for any failure | EI7-P5 and EI7-EC1: an unverifiable primary timeline fails the research response; a non-SYNTHETIC snapshot refuses the response with 409 |
| Design sections 6.1, 9, 11, 12, 16, 18, 19 and 20 | The corresponding proposals | [Design section 23](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#23-approved-contract-reconciliation-2026-11-21), through dated notes in each |

### 13.8 Approved EI-7C scope and read-only ports

**Minimal EI-7C scope (still NOT AUTHORIZED):** the Application views and query use cases behind the five route families; the recorded-session read port; the verified detail with the comparisons of section 13.4; the research view of EI7-P3; typed errors; test doubles; architecture tests.

| Port (Application) | Kind | Purpose |
| --- | --- | --- |
| `EquityRecordedSessionIssuanceReader` | New, read-only | The bounded recorded-session read of section 13.2 |
| `EquityIntradayResearchTimelineReader` | New, read-only | Load a stored research timeline; the existing timeline store also appends |
| `EquityCandleEvidenceSnapshotReader` | New, read-only | Get a snapshot; the existing snapshot store also issues (EI7-P4) |
| `EquityRecommendationLifecycleReader` | Existing, reused | The verified issuance and lifecycle |
| `EquityRecommendationIssuanceReader` | Existing, reused | `load_issuance`; `load_session_issuance` |
| `EquityRecommendationGenerationRecordReader` | Existing, reused | Exact-key `load_disposition` |
| `EquitySyntheticPolicyCatalogReader` | **Deferred** (EI7-P3) | Only for a future, separately governed policy-set-based lookup |

- Port names are proposals (section 11). The new ports are separate from the existing ones, so no existing implementation or test double must change.
- **EI7-P4 authorization rule (EI7-FC6):** "The narrow P4 compatibility change is eligible for implementation within a separately owner-authorized EI-7C Application slice. It is not authorized by EI-7B Docs PR-1."
- **EI-7D (still NOT AUTHORIZED)** supplies the read-only Infrastructure adapters: `mode=ro`; `PRAGMA query_only = ON`, confirmed; no DDL and no schema creation on any read; reuse of the existing verified decoders; typed unavailability and integrity failures; real-SQLite equivalence and restart tests. No write-capable reader adapter is introduced; the existing adapters stay the write authorities.

> **Note, 2026-11-28.** "Still NOT AUTHORIZED" above records the state on 2026-11-21. EI-7C and EI-7D were authorized on 2026-11-21 and merged (section 9, rows of 2026-11-28). EI7-SI1 adds one Application-owned error contract to this scope, delivered by SI-A and applied to the EI-7D adapters by SI-B, each separately authorized ([section 14.4](#144-ei7-si1-approved-storage-engine-integrity-classification)).

### 13.9 Accepted limitations

In addition to section 7, which stays in force:

- an over-limit session may be refused for overflow before a corrupt individual issuance row is discovered (EI7-P6); a success is never unverified;
- because both presentations come from one bounded read, a session with more than 50 recorded issuances at W is refused in the declared view too, even if fewer are declared (approved, EI7-FC1); such a session can be inspected only issuance by issuance through the detail read;
- every discovery and detail read checks the whole ledger, so its cost grows with the ledger; optional per-item lifecycle summaries are up to 50 further independent reads, not atomic with the session read or with one another (section 13.2, item 7);
- full unissued-generation history is not observable in EI-7C: `NOT_ELIGIBLE` and `UNRESOLVED` dispositions of unissued candidates, and every generation attempt, stay invisible (EI7-FC2);
- the stored research configuration has no fingerprint of its own; its identity is verified through its canonical document, its strategy and version columns, the timeline identity document and the re-derived decision and entry fingerprints (EI7-FC3);
- the disposition is read on a separate connection; no cross-connection atomicity;
- the historical publication of a policy set cannot be established from persisted facts;
- research-candidate authenticity requires EI3-D13 rehydration;
- generation outcomes other than an issued `PROPOSED` disposition, and attempts without a disposition, are not discoverable in EI-7C;
- lifecycles have no watermark; `position` lets a client detect change;
- a `mode=ro` reader cannot recover a hot rollback journal (EI-6C L8);
- no time of recording and no monitoring time exist, so none is shown.

> **Note, 2026-11-28.** Further limitations accepted with EI7-CF3, EI7-CF5, EI7-SI1 and EI7-SI2 are in [section 14.8](#148-accepted-limitations).

### 13.10 Pending technical details and outstanding decisions

| Item | Status | Needed before |
| --- | --- | --- |
| Route encoding of `StrategyIdentity` as a path segment | PENDING TECHNICAL DETAIL | EI-7E |
| Whether the runtime check in `rehydrate_equity_intraday_research_timeline` (today `isinstance` against `EquityCandleEvidenceSnapshotStore`) can accept the reader-only Protocol, with its compatibility tests (EI7-P4) | Direction **APPROVED — 2026-11-21** (EI7-FC6); the code-level verification stays a PENDING TECHNICAL DETAIL of the authorized slice | EI-7C research use case |
| The research configuration identity that rehydration requires | **SETTLED — 2026-11-21** (EI7-FC3): the stored authoritative configuration document, verified; never a current or client-supplied configuration | — |
| An explicit evidence-class check of the snapshots read for the research response | **SETTLED — 2026-11-21** (EI7-FC3): class gating before any outcome presentation; 409 on non-SYNTHETIC | — |
| `contract_id` placement; `recording_position` and `recording_watermark` names and meaning; `EQUITY_NOT_CONFIGURED` retryability | **SETTLED — 2026-11-21** (EI7-FC4) | — |
| The typed reasons of `NOT_QUERYABLE`; panel and item status names; the remaining response field names | PENDING TECHNICAL DETAIL (semantics approved) | EI-7C contract tests |
| The trigger of `EQUITY_NOT_CONFIGURED` | Depends on EI7-CF5 | EI-7E |
| A strategy filter once a second strategy exists; the limit if EI0-D5 approves a larger universe | **OPEN** | A future dated decision |
| Policy-set-based generation lookup | Deferred by EI7-P3; separate governance | — |
| EI7-CF3 private access enforcement; EI7-CF5 database runtime configuration and startup isolation | **PROPOSED — PENDING OWNER APPROVAL** | EI-7E |
| EI7-CF6 Web navigation and presentation | **PROPOSED — PENDING OWNER APPROVAL** | EI-7F |
| EI7-CF7 synthetic operator workflow | **PROPOSED — PENDING OWNER APPROVAL**; operator CLI **NOT AUTHORIZED** | EI-7G; EI7-N22 |
| EI4-C4 | **PENDING** | — |

> **Note, 2026-11-28.** The table above is not rewritten. Three of its rows are settled at the contract level on 2026-11-28: the route encoding of `StrategyIdentity` (direction **APPROVED — 2026-11-28**, EI7-CF3-F; its padding, length and response-echo details are technical closure items); the trigger of `EQUITY_NOT_CONFIGURED` (**SETTLED — 2026-11-28**, EI7-CF5-D: the Equity database setting is absent); and EI7-CF3 and EI7-CF5 (**APPROVED — 2026-11-28**). The remaining implementation details are listed in [section 14.7](#147-technical-closure-items). The EI7-CF6, EI7-CF7, strategy-filter, policy-set lookup and EI4-C4 rows are unchanged.

### 13.11 Status

| Item | Status |
| --- | --- |
| R-1 to R-12 | **APPROVED — 2026-11-15** |
| EI7-CF1, EI7-CF2, EI7-CF4 | **APPROVED — 2026-11-20** |
| EI7-P1 to EI7-P8; EI7-EC1 | **APPROVED — 2026-11-21** |
| EI7-FC1 to EI7-FC6 | **APPROVED — 2026-11-21** (EI7-FC5 accepted) |
| EI7-P4 code change | Eligible within a separately owner-authorized EI-7C; **NOT AUTHORIZED** by EI-7B Docs PR-1 |
| EI7-CF3, EI7-CF5, EI7-CF6, EI7-CF7 | **PROPOSED — PENDING OWNER APPROVAL** |
| EI-7C to EI-7H | **NOT AUTHORIZED** |
| EI7-N01 to EI7-N24 | **PLANNED — NOT TESTED** (24) |
| EI-6B E1 to E9; EI-6C L1 to L13 | In force |
| EI-6C follow-ups F1 to F10 | **NOT AUTHORIZED** |
| EI4-C4 | **PENDING** |
| Unchanged boundaries | Single host; synthetic-only evidence; MODELED-only issuance; no cross-connection atomicity; no durable monitoring-run provenance; no broker execution, orders or fills; no real or provider market-data activation; no scheduler; no deployment change; no EI-8 performance claim |

> **Note, 2026-11-28.** The table above records the state on 2026-11-21. The current status is in [section 14.9](#149-status).

### 13.12 Final owner confirmations EI7-FC1 to EI7-FC6 (2026-11-21)

The owner confirmed six points on **2026-11-21**, labelled C1 to C6 in the instruction. They are recorded here and applied in sections 13.1 to 13.11 and in [design sections 23.2 to 23.10](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#2310-final-owner-confirmations-ei7-fc1-to-ei7-fc6). They change no R-1 to R-12, EI7-CF1, EI7-CF2, EI7-CF4, EI7-P1 to EI7-P8 or EI7-EC1 approval; they settle details those approvals left open.

| ID | Owner confirmation | Status | Applied in |
| --- | --- | --- | --- |
| EI7-FC1 | **Discovery overflow scope.** At most 50 recorded issuances per session. Above 50, **both** new EI-7 session projections refuse with HTTP 409 `DISCOVERY_LIMIT_EXCEEDED`; no partial authoritative list. The 50-record restriction applies to the new EI-7 presentation and query layer, not to the historical issuance authority. EI-6A `session_view` behaviour and canonical order are unchanged. An overflow refusal is not a corruption verdict | **APPROVED — 2026-11-21** | Sections 13.2 and 13.9; design 23.2 |
| EI7-FC2 | **Unissued generation visibility.** Persisted `NO_SIGNAL` and `REJECTED` research outcomes show their authoritative reasons. An unissued research candidate without exact generation identity is `NOT_QUERYABLE`. The absence of generation attempts is never inferred. `NOT_ELIGIBLE` and `UNRESOLVED` are shown only when their exact dispositions are authoritatively discoverable. EI-7C adds no policy-set-based generation search, no global disposition browsing and no attempt search. UNKNOWN, `NOT_QUERYABLE`, `PROPOSED`, `NOT_ELIGIBLE`, `UNRESOLVED` and `ISSUED` stay distinct. **Limitation, stated plainly: full unissued-generation history is not observable in EI-7C** | **APPROVED — 2026-11-21** | Sections 13.4 and 13.9; design 23.5 |
| EI7-FC3 | **Research configuration identity and SYNTHETIC evidence.** Persisted research is rehydrated with the stored authoritative configuration document. Its identity, fingerprints and applicable historical relationships are verified. A current configuration or client-supplied guess is never substituted. Missing or inconsistent required configuration is a typed whole-response research integrity failure. A retrieved non-SYNTHETIC snapshot is refused explicitly with HTTP 409 `EVIDENCE_CLASS_UNSUPPORTED`; the existing evaluator's reconstruction of unsupported evidence as an ordinary `REJECTED` outcome is never displayed; evidence payloads are not leaked or reclassified as SYNTHETIC. Optional evidence-panel degradation stays permitted only while required provenance is intact. Future tests must prove class gating before outcome presentation | **APPROVED — 2026-11-21** | Sections 13.4, 13.9 and 13.10; design 23.5 and 23.8 |
| EI7-FC4 | **Contract conventions.** `EQUITY_NOT_CONFIGURED` is HTTP 503 with `retryable: false`. `contract_id` is at the top level of success and error envelopes. The identifier is exactly `northstar.equity-private-read/1`. `recording_position` is the observed authoritative ledger position N. `recording_watermark` is the requested or selected replay boundary W. Both are canonical exact decimal strings on the JSON wire; the no-JSON-numbers rule stands | **APPROVED — 2026-11-21** | Sections 13.2, 13.3 and 13.10; design 23.1 and 23.3 |
| EI7-FC5 | **Source-document provenance.** The owner-approved CF contract package is not present in the six local repositories. This record cites the explicit owner decisions of 2026-11-20 and 2026-11-21, the source-feasibility audit, the merged EI-7B governance and the verified current source contracts. It does not claim that the package was opened, read or committed | **ACCEPTED — 2026-11-21** | Section 13.1 |
| EI7-FC6 | **P4 Protocol authorization.** A reader-only Protocol may be introduced for research rehydration. Existing valid writer-capable stores stay compatible. A fake `issue()` is forbidden. `runtime_checkable` Protocol validation alone does not prove runtime behaviour. Future EI-7C tests cover compliant and non-compliant implementations, structural acceptance and preserved rehydration semantics. Exact rule: "The narrow P4 compatibility change is eligible for implementation within a separately owner-authorized EI-7C Application slice. It is not authorized by EI-7B Docs PR-1." | **APPROVED — 2026-11-21** | Sections 9, 13.8 and 13.11; design 18 (note) and 23.6 |

**Source facts behind EI7-FC3** (verified read-only at the commits of section 3):

- the timeline row binds `strategy` and `configuration_version` to its canonical `configuration_document`, and its identity document embeds the same configuration (Infrastructure `sqlite_equity_intraday_research_timeline_schema.py`, lines 69 to 81);
- the stored configuration has **no fingerprint of its own**. "Fingerprint" verification therefore means the decision and entry fingerprints that EI3-D13 rehydration re-derives, together with the exact canonical-document comparison that `rehydrate_equity_intraday_research_timeline` already performs (it raises `EquityIntradayResearchTimelineConfigurationConflictError` on a mismatch);
- for an issued recommendation, the applicable historical relationship is the research configuration version bound into `eirk1` and the configuration in the disposition's decoded inputs, which must equal the stored timeline's;
- the existing evaluator turns a non-SYNTHETIC snapshot into a `REJECTED` decision with reason `EVIDENCE_NOT_SYNTHETIC` (Application `evaluate_equity_intraday_research.py`, line 649), so the evidence-class gate must run before any re-derived outcome is used or presented.

**List and detail consistency** (verified 2026-11-21 against section 13.2 and design sections 11.3, 13 and 23.2): no EI-7 text promises an atomic lifecycle snapshot across the listed issuances. Discovery is authoritative for issuance identity and session recording facts at W. The full lifecycle belongs to the verified detail, whose issuance and lifecycle come from one read transaction. Optional per-item summaries are independent reads, not atomic with W or with one another, and each failure is an explicit item status.

## 14. Approved contracts EI7-CF3, EI7-CF5, EI7-SI1 and EI7-SI2 (2026-11-28)

This section records owner approvals given after section 13 was written. Sections 1 to 13 are not rewritten; dated notes there point here. Where earlier proposal text in this register or in the [design](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md) differs from this section, **this section governs**. Nothing in section 13 is changed: R-1 to R-12, EI7-CF1, EI7-CF2, EI7-CF4, EI7-P1 to EI7-P8, EI7-EC1 and EI7-FC1 to EI7-FC6 stand as approved. The engineering mechanics are in [design section 24](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#24-approved-private-access-runtime-and-storage-integrity-design-2026-11-28). **These are contract approvals. Nothing here is implemented, and nothing here authorizes an implementation or a deployment change.**

### 14.1 Approval record

| Date | Decision | Status |
| --- | --- | --- |
| 2026-11-15 | R-1 to R-12 | **APPROVED — 2026-11-15**, unchanged |
| 2026-11-20 | EI7-CF1, EI7-CF2, EI7-CF4 | **APPROVED — 2026-11-20**, unchanged |
| 2026-11-21 | EI7-P1 to EI7-P8; EI7-EC1; EI7-FC1 to EI7-FC6 | **APPROVED — 2026-11-21** (EI7-FC5 **ACCEPTED — 2026-11-21**), unchanged |
| 2026-11-28 | EI7-CF3, private access and deployment security | **APPROVED — 2026-11-28** |
| 2026-11-28 | EI7-CF5, separate Equity database configuration and strictly read-only runtime | **APPROVED — 2026-11-28** |
| 2026-11-28 | EI7-SI1, a dedicated permanent storage-integrity error contract | **APPROVED — 2026-11-28** |
| 2026-11-28 | EI7-SI2, accepted integrity-check coverage without whole-file verification on ordinary GET requests | **APPROVED — 2026-11-28** |
| — | EI7-CF6, EI7-CF7 | **PROPOSED — PENDING OWNER APPROVAL**, unchanged |

- **Source.** The owner's EI-7E governance instruction of 2026-11-28, which states the four decisions. No separate owner working document was read; nothing is attributed to one. The decisions are recorded from the instruction text; the source facts cited below were verified read-only on 2026-11-28 at northstar-core `29bd34a`, northstar-application `16bb21a`, northstar-infrastructure `caa2c46`, northstar-api `7ad37f5`, northstar-web `5aeb94c` and northstar-docs `1e8138c`.
- **Engineering evidence behind EI7-SI1 and EI7-SI2.** The EI-7D final pre-PR audit (2026-11-23), whose read-only adapters are merged at `caa2c46`, found that every SQLite engine failure, `SQLITE_CORRUPT` and `SQLITE_NOTADB` included, reaches EI-7C as the reader family's unavailability error and so as retryable `DEPENDENCY_UNAVAILABLE`; that the accepted issuance and lifecycle integrity errors accept only their record categories (ISSUANCE, DECLARATION, LEDGER; EVENT, HISTORY, WINDOW); that the EI-7C snapshot integrity error asserts that a stored snapshot exists; and that corruption on a page no query touches is not detected. The audit stopped short of an Application change. These are findings of that audit, not acceptance evidence.
- **Not authorized:** SI-A, SI-B, EI-7E, EI-7F, EI-7G, EI-7H; any Caddy, bind, CORS, documentation-route, compose or other deployment change; broker or market-data integration; database creation, migration or file moves.

### 14.2 EI7-CF3: approved private access contract

**APPROVED — 2026-11-28.** Refines R-5; supersedes the proposal of [section 8.3](#83-ei7-cf3-private-access-enforcement) where they differ.

- **Previously open.** The exact environment and route-enforcement contracts (section 8.3): bind, topology, allow-list, documentation routes, same or separate process, read-only file mount, and whether the CME stack serves Equity.
- **Approved decision:**

| Part | Decision |
| --- | --- |
| EI7-CF3-A Topology | Owner browser → SSH tunnel → Caddy private gateway → Northstar API on loopback → Application read queries → Infrastructure read-only SQLite adapters. The deployment must prevent any access that bypasses this gateway. Caddy basic authentication is the private authentication mechanism. No in-application bearer-token architecture is approved, and no general public or subscriber service is authorized |
| EI7-CF3-B Network binding | A loopback-only private exposure model. The recommended backend listener is `127.0.0.1`. Existing Futures binding, port and composition are not silently changed, and are verified before EI-7E implementation. A shared listener or application composition that would expose Equity routes without the gateway fails the private-access acceptance requirements. External direct reachability of the backend is a deployment blocker |
| EI7-CF3-C Authentication | Caddy basic authentication protects every private route. Credentials are never committed, logged or returned in a response body. Access uses the approved SSH-tunnel model; basic authentication over unprotected public HTTP is not acceptable |
| EI7-CF3-D Default-deny routes | Only the five approved Equity GET route families of [section 13.3](#133-ei7-cf2-approved-private-read-api-contract) are exposed, with their approved query parameters, including the `watermark` semantics of section 13.2. No snapshot, separate lifecycle, generic browse or write route. HEAD is permitted only where explicitly supported and verified as non-mutating; otherwise it is denied. Every other method and path is denied by default, without a permissive wildcard that could expose future routes |
| EI7-CF3-E API documentation | `/docs`, `/redoc` and `/openapi.json` are disabled or unreachable **at the application boundary**; hiding them at the gateway alone is insufficient. Their actual behaviour is verified in EI-7E |
| EI7-CF3-F Strategy identity encoding | A valid `StrategyIdentity` may contain `/` and is not restricted further. The route segment is the canonical base64url encoding of the identity's UTF-8 text, one segment, with a canonical round trip and validation through the existing `StrategyIdentity` only; there is no second identity authority. Invalid encodings are request-validation failures |
| EI7-CF3-G Same-origin access | The initial browser experience uses the authenticated same-origin gateway. No wildcard CORS. The browser never needs the backend's internal listener |
| EI7-CF3-H Security verification | The negative tests of [design section 24.13](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#2413-security-and-failure-tests) are required; they stay planned until executed in the implementing slice |

- **Rationale.** Reuses the private boundary already deployed for the Indian Futures stack (Caddy basic authentication, host-loopback publication, SSH tunnel, exact-path GET allow-list; `deploy/india/Caddyfile`, `deploy/india/compose.yaml`), and makes "private by deployment" checkable at both the gateway and the application.
- **Non-goals.** No in-app authentication, user accounts, tokens or roles; no public or multi-user service; no new route family; no write route; no change to Futures routes; no deployment change by this record.
- **Known limitations.** Privacy is a deployment condition, not intrinsic authentication (R-5): an API process run outside the boundary is unprotected. Source facts verified on 2026-11-28 that EI-7E must resolve, not assume: the API entry point hard-codes `0.0.0.0:8000` (`northstar_api/__init__.py`), which in the Indian stack listens only inside a container with no published port, behind Caddy published on host loopback; the CME stack's Caddy publishes ports 80 and 443 and proxies every `/api/*` path (`deploy/Caddyfile`), so Equity routes added to the shared application would be reachable there without a default-deny allow-list; FastAPI's documentation routes are on today (`northstar_api/app.py`).
- **Implementation dependency.** EI-7E, after SI-A and SI-B are merged (section 14.6); any deployment change needs its own authorization.
- **Required acceptance evidence.** EI7-N18 and the EI7-CF3 items of [design section 24.14](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#2414-acceptance-reconciliation), executed in EI-7E (application boundary) and evidenced against the deployed topology before exposure.

### 14.3 EI7-CF5: approved Equity runtime and database contract

**APPROVED — 2026-11-28.** Refines R-4 and R-7; supersedes the proposal of [section 8.5](#85-ei7-cf5-database-and-adr-clarification) where they differ.

- **Previously open.** The setting name, file location conventions, a read-only file mount, startup isolation and the trigger of `EQUITY_NOT_CONFIGURED` (sections 8.5 and 13.10).
- **Approved decision:**

| Part | Decision |
| --- | --- |
| EI7-CF5-A Separate database authority | Equity uses an explicitly configured SQLite file distinct from the Futures file. No implicit fallback to the Futures database, no combined Equity and Futures schema or migration, no global database discovery |
| EI7-CF5-B Explicit configuration | A dedicated Equity SQLite path setting. Proposed name `NORTHSTAR_EQUITY_SQLITE_PATH`. Verified 2026-11-28: no Equity setting exists; Futures-specific settings use the `NORTHSTAR_FUTURES_` prefix; the Futures file is `NORTHSTAR_DATABASE`, parsed as a path (`northstar_api/settings.py`); no variable of that name exists, so it conflicts with no setting. It replaces the example `NORTHSTAR_EQUITY_DATABASE` of section 8.5. EI-7E confirms the spelling as a technical closure item |
| EI7-CF5-C Startup behaviour | Constructing the Equity read-only adapters opens no database, creates no file, initializes no schema, applies no migration, issues no recommendation, records no lifecycle event, triggers no monitoring, writes no evidence and attempts no journal recovery. An unconfigured Equity runtime never falls back to a writable database |
| EI7-CF5-D Unconfigured runtime | Equity requests are refused with `EQUITY_NOT_CONFIGURED`, HTTP 503, `retryable: false` (EI7-FC4), never answered as an empty but healthy database. The top-level identifier stays `contract_id: "northstar.equity-private-read/1"` |
| EI7-CF5-E Health | `GET /equity/health` is a narrowly scoped, non-mutating dependency probe. It distinguishes configured and safely readable; not configured; dependency unavailable; and integrity failure where detectable. A successful health response never claims that every page is valid, every recommendation verified, every lifecycle audited, the Equity universe complete, or broker or market data available. It is never a writer-driven schema initialization and carries no unrelated application metric. Body fields not already specified are part of the EI-7E wire-contract closure, within the approved error envelope |
| EI7-CF5-F Journal mode | The initial verified operating model is SQLite's rollback journal. WAL-mode databases are refused by the EI-7D readers. Residual limitation: the WAL header check runs before SQLite opens the file, so a concurrent mode transition between the two is not eliminated by the check alone. Deployment validation and journal-mode operating controls are required before API exposure. No claim of unconditional filesystem non-mutation across unsupported modes. `immutable=1`, automatic recovery and any read-write fallback are not authorized |
| EI7-CF5-G Hot journal | A hot rollback journal that needs recovery is a typed dependency unavailability. The read-only runtime never repairs or recovers it by a write; recovery stays outside the GET path, in an appropriately authorized existing write workflow |
| EI7-CF5-H Futures compatibility | Existing Futures behaviour is retained. The Futures runtime initializes its own schema at startup (`build_database_runtime`, `northstar_api/runtime.py`), so the API **process** is not described as write-free; the newly exposed **Equity read stack** stays read-only |

- **Rationale.** Makes R-7's separation enforceable at configuration time, keeps the R-4 read-only boundary true from construction to response, and records the EI-7D operating model (rollback journal; WAL refused; hot journal unavailable) as the governing one.
- **Non-goals.** No Equity database creation, migration or file move; no WAL support; no recovery in a read path; no change to the Futures settings or to the Futures schema initialization.
- **Known limitations.** The header-check-to-open race (EI7-CF5-F); a hot journal makes reads unavailable until a writer recovers it (EI-6C L8); readers delay writers' commits under the rollback journal; two files to back up, with no cross-file consistency.
- **Implementation dependency.** EI-7E, after SI-A and SI-B; deployment journal-mode controls before exposure.
- **Required acceptance evidence.** EI7-N12, N17 and N20 items of [design section 24.14](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#2414-acceptance-reconciliation): unconfigured, missing-file, missing-table, hot-journal and WAL cases; no write at construction, startup or request; Futures regression unchanged.

### 14.4 EI7-SI1: approved storage-engine integrity classification

**APPROVED — 2026-11-28.** Closes the question exposed by the EI-7D audit (section 14.1).

- **Previously open.** The merged EI-7D read-only adapters map SQLite engine-level corruption through the reader-family unavailability errors, so `SQLITE_CORRUPT` and `SQLITE_NOTADB` can reach EI-7C as retryable `DEPENDENCY_UNAVAILABLE`. The existing issuance and lifecycle integrity categories describe decoded-authority failures, not database-engine failures, and must not be overloaded.
- **Approved decision.**
  - A dedicated, **Application-owned**, typed permanent storage-integrity contract represents a permanent failure of the storage read boundary. Tentative name `EquityReadStorageIntegrityError`; the name and module are finalized in SI-A under existing naming conventions.
  - It never requires pretending that a specific issuance, lifecycle or snapshot exists when SQLite cannot establish that fact.
  - EI-7C maps it to the existing code `PERSISTED_AUTHORITY_INVALID`: HTTP 500, not retryable, a safe fixed public message, no raw SQLite text, no database path and no stored payload. HTTP serialization stays with EI-7E. **No new public error code is introduced.**
  - Classification:

| SQLite condition | Classification |
| --- | --- |
| `SQLITE_CORRUPT` | Permanent storage-integrity error |
| `SQLITE_NOTADB` | Permanent unusable-storage error: the configured storage target is unusable without intervention. It may indicate a misconfigured path rather than literal corruption; the classification does not diagnose the cause |
| `SQLITE_BUSY` | Dependency unavailable |
| `SQLITE_LOCKED` | Dependency unavailable |
| `SQLITE_CANTOPEN` | Dependency unavailable |
| `SQLITE_READONLY_ROLLBACK` (hot journal) | Dependency unavailable |
| Missing required tables | Dependency unavailable, under the approved initial contract |

  - SI-B detects these by SQLite result codes, including extended codes, never by matching human-readable messages, and keeps the original exception safely for internal diagnosis.
  - Implementation order: **SI-A** (Application compatibility: the typed contract and EI-7C handling, no Infrastructure change), then **SI-B** (Infrastructure compatibility: the EI-7D read-only adapters raise the contract, no Application change). Each needs a separate owner authorization.
  - The existing EI-6A, EI-6B, EI-4D and older write-side adapters keep their behaviour unless separately authorized. No existing authority error enumeration is modified to fit the new error.
- **Rationale.** Keeps permanent corruption from being presented as a retryable outage, without misusing record-level categories or asserting facts SQLite cannot establish, and reuses the approved public vocabulary.
- **Non-goals.** No new public code; no change to the EI-6A or EI-6B error categories; no change to writer adapters; no repair; no full-file check (EI7-SI2).
- **Known limitations.** `SQLITE_NOTADB` cannot distinguish a misconfigured path from damaged storage. Detection covers only what each operation reads (EI7-SI2). Until SI-A and SI-B are merged, the EI-7D behaviour recorded in section 14.1 stays in force and EI-7E is not ready (section 14.6).
- **Implementation dependency.** SI-A, then SI-B, then EI-7E.
- **Required acceptance evidence (planned, not verified by this record).** `SQLITE_CORRUPT`; `SQLITE_NOTADB`; busy and locked failures; a missing file and missing tables; propagation through the session, detail and research queries and the optional lifecycle summaries; no raw internal message leakage; correct retryable semantics; writer-side backward compatibility; the read-only adapters' non-mutation preserved ([design section 24.10](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#2410-storage-engine-error-compatibility-contract)).

### 14.5 EI7-SI2: approved integrity-check coverage

**APPROVED — 2026-11-28.**

- **Previously open.** Whether ordinary GET requests must prove the integrity of the whole database file.
- **Approved decision.**
  - The read-only adapters verify the authoritative data each operation accesses, under its existing accepted contract. They do not promise to detect corruption on pages no query accesses.
  - The EI-7D whole-ledger issuance verification stays required and is not weakened (EI7-P6).
  - No full `PRAGMA integrity_check` on every GET. No background integrity automation, automatic repair service, monitoring scheduler, periodic maintenance job or implicit startup repair. Any future offline or periodic integrity inspection needs separate governance.
  - **Communication rule.** A successfully verified query result, a healthy read dependency and a database-wide integrity certification are different claims and are never conflated in health responses, logs or user-interface language.
- **Rationale.** Per-request whole-file checks would make every GET cost proportional to the file and add no authority the verified decoders do not already provide for the data presented.
- **Non-goals.** Integrity certification; scheduled checks; repair.
- **Known limitations.** Corruption outside the pages a request reads stays undetected by that request, and a healthy probe says nothing about it.
- **Implementation dependency.** None of its own; it constrains SI-B, EI-7E (health) and EI-7F (language).
- **Required acceptance evidence.** The health and wording items of [design section 24.14](../architecture/Equity-Intraday-V1-EI7-Private-API-and-Web-Design-v1.0.md#2414-acceptance-reconciliation); the existing whole-ledger tests stay green.

### 14.6 Implementation order and gates

| Stage | Slice | Repository | Precondition | Status |
| --- | --- | --- | --- | --- |
| 1 | EI-7E governance documentation PR (this record) | northstar-docs | Owner review | **AUTHORIZED, 2026-11-28** (documentation only) |
| 2 | SI-A Application compatibility: the typed permanent storage-integrity contract and EI-7C handling; no Infrastructure change | northstar-application | Stage 1 merged; separate owner authorization | **NOT AUTHORIZED** |
| 3 | SI-B Infrastructure compatibility: the EI-7D read-only adapters classify SQLite engine failures with the SI-A contract; no Application change | northstar-infrastructure | SI-A merged; separate owner authorization | **NOT AUTHORIZED** |
| 4 | EI-7E private API: the five GET families and the EI7-CF3 and EI7-CF5 runtime and security contracts | northstar-api | SI-A and SI-B merged and their compatibility verified; separate owner authorization | **NOT AUTHORIZED** |
| 5 | EI-7F Web | northstar-web | EI7-CF6 approved; separate owner authorization | **NOT AUTHORIZED** |
| 6 | EI-7G operator CLI; EI-7H end-to-end acceptance | northstar-api; all | Separate governance (EI7-CF7) and authorization | **NOT AUTHORIZED**; EI-7H **not accepted** |

**EI-7E stop conditions.** EI-7E stops and reports, rather than deciding, if: a route or composition would expose Equity routes outside the approved gateway, including through the CME stack's `/api/*` proxy; the backend would be reachable directly from outside the host; documentation routes cannot be disabled at the application boundary without an unreviewed Futures-visible change; the health probe would need a write, a schema initialization, a route family beyond the five, or an Application port beyond SI-A; the Equity setting would fall back to the Futures file; `StrategyIdentity` encoding would need a restriction on valid identities; SI-A or SI-B is not merged; or any GET path would reach a writer.

### 14.7 Technical closure items

Implementation details the approvals leave to the named slice. Each is settled within the approved contract, stated in that slice's PR, and needs an owner decision only if it cannot be.

| Item | Bound | Slice |
| --- | --- | --- |
| Final name and module of the storage-integrity error, and its relation to the reader-family errors, such that no unavailability handler can receive it first and present it as retryable | EI7-SI1 | SI-A |
| Presentation of a storage-integrity failure met in an optional read (an evidence panel, a per-item lifecycle summary): whole-response `PERSISTED_AUTHORITY_INVALID` or the panel's or item's `invalid` status; never `unavailable`, never retryable, never silent | EI7-SI1; EI7-EC1; section 13.2 item 7 | SI-A |
| Mapping of EI-7D's pre-open refusals that carry no SQLite code (an unreadable or unresolvable path; a WAL-mode file): today dependency unavailability, kept unless the owner decides otherwise | EI7-SI1; EI7-CF5-F | SI-B |
| Exact spelling of the Equity setting (`NORTHSTAR_EQUITY_SQLITE_PATH` proposed); treatment of a set but empty value; file location conventions; whether the API container mounts the Equity file read-only | EI7-CF5-A to EI7-CF5-D | EI-7E |
| Health probe: its location given that Equity route modules import no Infrastructure type (section 13.3 item 2), the exact read it performs, and its body fields | EI7-CF5-E; EI7-SI2 | EI-7E |
| How the loopback-only model applies to the container topology, where the API listens on the compose network behind Caddy, and the default bind of the non-container entry point, without silently changing Futures behaviour | EI7-CF3-B; EI7-CF5-H | EI-7E |
| Whether disabling the documentation routes applies to the whole process (a Futures-visible change, to be explicit and tested) or only to an Equity-only application | EI7-CF3-E | EI-7E |
| Whether HEAD is supported on the five families, and its non-mutation proof | EI7-CF3-D | EI-7E |
| Treatment of unknown query parameters on the five families | EI7-CF3-D | EI-7E |
| Base64url padding (with or without `=`), maximum segment length, and whether responses echo the decoded identity, the encoded segment or both | EI7-CF3-F | EI-7E |
| How the CME stack is kept from reaching Equity routes (a separately authorized deployment change, or a composition in which they are absent) | EI7-CF3-A; EI7-CF3-B | EI-7E |
| Deployment journal-mode validation and operating controls | EI7-CF5-F | EI-7E deployment |

### 14.8 Accepted limitations

In addition to sections 7 and 13.9, which stay in force:

- privacy depends on deployment: an API process outside the approved boundary is unprotected (R-5; EI7-CF3);
- the WAL header check and the SQLite open are two steps; a concurrent mode transition between them is not eliminated by the check alone (EI7-CF5-F);
- a hot rollback journal makes every Equity read unavailable until an authorized write workflow recovers it (EI7-CF5-G; EI-6C L8);
- the API process as a whole may write the Futures file at startup; only the Equity read stack is read-only (EI7-CF5-H);
- `SQLITE_NOTADB` is reported as unusable storage, whatever its cause (EI7-SI1);
- corruption on pages that no request reads is not detected, and a healthy dependency probe is not an integrity certification (EI7-SI2);
- until SI-A and SI-B are merged, SQLite engine corruption reaches EI-7C as retryable unavailability, as merged in EI-7D (section 14.1).

### 14.9 Status

| Item | Status |
| --- | --- |
| R-1 to R-12 | **APPROVED — 2026-11-15** |
| EI7-CF1, EI7-CF2, EI7-CF4 | **APPROVED — 2026-11-20** |
| EI7-P1 to EI7-P8; EI7-EC1; EI7-FC1 to EI7-FC6 | **APPROVED — 2026-11-21** (EI7-FC5 accepted) |
| EI7-CF3, EI7-CF5, EI7-SI1, EI7-SI2 | **APPROVED — 2026-11-28** |
| EI7-CF6, EI7-CF7 | **PROPOSED — PENDING OWNER APPROVAL** |
| EI-7C | Authorized 2026-11-21; merged (#100, `16bb21a`); no acceptance record here |
| EI-7D | Authorized 2026-11-21; merged (#70, `caa2c46`); no acceptance record here |
| SI-A, SI-B, EI-7E, EI-7F, EI-7G, EI-7H | **NOT AUTHORIZED** |
| EI7-N01 to EI7-N24 | **PLANNED — NOT TESTED** (24) |
| Technical closure items | Section 14.7, each with its slice |
| EI-6B E1 to E9; EI-6C L1 to L13 | In force |
| EI-6C follow-ups F1 to F10 | **NOT AUTHORIZED** |
| EI4-C4 | **PENDING** |
| Unchanged boundaries | Single host; synthetic-only evidence; MODELED-only issuance; no cross-connection atomicity; no durable monitoring-run provenance; no broker execution, orders or fills; no real or provider market-data activation; no scheduler; no deployment change; no EI-8 performance claim |
