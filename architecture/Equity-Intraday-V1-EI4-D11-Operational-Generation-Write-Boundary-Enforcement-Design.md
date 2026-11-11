# EI4-D11 — Mandatory Operational Generation-Write Boundary Enforcement Design

Status at proposal (historical): **PROPOSAL — NOT OWNER APPROVED** (2026-11-10).

> **Status update, 2026-11-10: OWNER APPROVED — 2026-11-10.** The owner approved EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8 and DQ10, with two mandatory clarifications ([section 22](#22-owner-approval-2026-11-10)). The approved architecture is recorded in [ADR-023](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md) and the [EI-4 Decision Register, section 13](../product/Equity-Intraday-V1-EI4-Decision-Register.md#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10).
>
> - **Implementation:** NOT YET IMPLEMENTED.
> - **Engineering acceptance:** NOT YET ESTABLISHED.
> - **Invalid-record recovery:** NOT APPROVED.
> - The clarifications are incorporated into sections 8, 10, 14, 16 and 17, each marked "amended 2026-11-10". The text below otherwise keeps the proposal as reviewed: where it says the design is unapproved or a proposal, that is the status at proposal.

> **Status update, 2026-11-11: IMPLEMENTED; ENGINEERING ACCEPTANCE RECORDED.** The bullets above record the status before implementation.
>
> - **Implementation.** Slices W1 to W4-I are implemented and merged: Application #91 (`298a3a5`) and #92 (`8937f12`); Infrastructure #58 (`6b90597`) and #59 (`6282cae`). W5 is the acceptance record.
> - **Acceptance.** Engineering acceptance, PASS WITH DOCUMENTED LIMITATIONS (synthetic engineering acceptance), is recorded in the [EI4-D11 Engineering Acceptance Record](../product/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Engineering-Acceptance-Record.md). It traces every scenario of the [acceptance matrix](#17-acceptance-matrix) to its evidence: 28 VERIFIED, 5 VERIFIED WITH LIMITATION (rows 12, 21, 23, 29 and 31), 0 NOT VERIFIED. The matrix below keeps its proposal status.
> - **Unchanged.** The proposal, alternatives, approvals and limitations below are unchanged. Invalid-record recovery is not approved, and no production composition root exists.

- **Governance basis.** EI4-D11, Mandatory Verified Recording Boundary and Invalid-Record Recovery Governance, EI4-D11-A to EI4-D11-E: **OWNER APPROVED, 2026-11-08** ([EI-4 Decision Register, section 10.3](../product/Equity-Intraday-V1-EI4-Decision-Register.md#103-ei4-d11----mandatory-verified-recording-boundary-and-invalid-record-recovery-governance-aa-2)).
- **The approved rule and this design are different things.** The rule is approved; this enforcement design is **not**. Every contract, slice and test below is a proposal. None exists, and nothing here authorizes implementation.
- **Not implemented and not accepted.** EI4-D11 enforcement is not implemented, and its engineering acceptance is not established.
- **Unchanged.** Nothing here changes code, schema, tests, fixtures, any canonical identity (`eirs1`, `eirk1`, `eirat1`, `eipd1`) or any stored record.
- **No recovery.** No invalid-record recovery is designed or authorized ([section 11](#11-invalid-committed-evidence)).
- **Open questions.** Engineering decisions that need the owner are listed in [section 18](#18-engineering-decisions-requiring-approval), as EI4D11-DQ1 to EI4D11-DQ10.

**Location.** `architecture/` holds design specifications. This file follows the [EI4-D10 enforcement design](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md) precedent: a proposal, named without a version suffix, added as a single file. The README's Architecture row already links the folder, so no index changes.

Source paths are relative to each repository root. Code was read at:

| Repository | Branch | Commit |
| --- | --- | --- |
| northstar-core | `develop` | `bc081a2` |
| northstar-application | `develop` | `cd2301b` (EI4-D10-S1 #89, S3 #90) |
| northstar-infrastructure | `develop` | `97700b3` (S2 #55, W2 #56, S4 #57) |
| northstar-api | `develop` | `7ad37f5` |
| northstar-web | `develop` | `5aeb94c` |
| northstar-docs | `develop` | `64956d5` (EI4-D10 S5 #54) |

## 1. Status and scope

| Item | Status |
| --- | --- |
| EI4-D11 governance (A to E) | APPROVED, 2026-11-08 |
| This enforcement design | **PROPOSAL — NOT OWNER APPROVED** |
| EI4-D11 implementation | **NOT IMPLEMENTED** |
| EI4-D11 engineering acceptance | **NOT ESTABLISHED** |
| Invalid-record recovery | **NOT APPROVED**; not designed here |

> **Note, 2026-11-10.** The table records the status at proposal. The enforcement design is now **OWNER APPROVED (2026-11-10)**. Implementation is NOT YET IMPLEMENTED, engineering acceptance is NOT YET ESTABLISHED, and invalid-record recovery is NOT APPROVED ([section 22](#22-owner-approval-2026-11-10)).

**Scope.** The write path for synthetic equity recommendation-generation records (EI-4D dispositions, proposals and logical attempts) in the Application and Infrastructure repositories.

**Not in scope:**
- futures, options and paper-trading persistence;
- the EI4-D10 policy catalog, which is accepted and unchanged;
- issuance, broker activity, EI-5 and EI-6.

## 2. Existing approved governance

**EI4-D11-A** (verbatim):

- "The EI-4D-A verified Application recorder (`RecordEquityRecommendationGenerationUseCase`) is the required operational path for committing synthetic equity recommendation-generation dispositions."
- "The raw EI-4D-B SQLite adapter (`SQLiteEquityRecommendationGenerationStore`) is an Infrastructure persistence mechanism. It is not an independently authorized operational generation interface."

**The other subdecisions:**

| Subdecision | What it requires |
| --- | --- |
| EI4-D11-B | A stored record that fails canonical, source, snapshot or replay verification is never accepted as equivalent, repaired, migrated, replaced, treated as issued, or used as authoritative evidence. The existing distinctions are preserved |
| EI4-D11-C | The invalid record and its source evidence are preserved, not deleted, overwritten or relinked |
| EI4-D11-D | Recovery needs a separately approved design. Deleting or updating immutable rows, replacing an occupied key, rewriting fingerprints, and any privileged production repair bypass are **not authorized** |
| EI4-D11-E | "A future engineering slice must establish enforceable architectural restrictions, and appropriate regression tests, around the required operational recording boundary. A documentation-only convention is not technical enforcement." SQLite enforces structure only; source and replay verification are the Application's |

**Settled elsewhere; not re-decided here:**
- ADR-021: identities, atomic recording, conflict rules, bounded commit reconciliation.
- ADR-022: the policy catalog, and DQ9's file-level trust limitation.
- EI4-D10: the recording gate, accepted.

**Not settled by governance:** the enforcement mechanism, its trust boundary, its interfaces and its migration. Those are this design's subject, and its proposals.

## 3. Current implemented architecture

**Application** (`src/northstar_application/`):

| Component | Role today |
| --- | --- |
| `ports/equity_recommendation_generation_store.py`: `EquityRecommendationGenerationStore` | One ABC with four methods: `load_disposition`, `load_attempt`, `record_generation(disposition, attempt)` and `record_attempt(attempt)`. Reads and writes are one capability |
| `application_services/record_equity_recommendation_generation.py`: `RecordEquityRecommendationGenerationUseCase` | The verified recorder. `execute(audit)`, `record(disposition, attempt)` and `record_attempt(attempt)` are public. `record` runs the EI4-D10 policy gate, then source verification and replay through the verifier, then `store.record_generation`, with bounded reconciliation (EI4-D9-G) and the W1 mappings. `record_attempt` checks only the record type and that no disposition kind is present |
| `application_services/verify_equity_recommendation_generation.py`: `EquityRecommendationGenerationVerifier`, `VerifyStoredEquityRecommendationGenerationUseCase` | Levels A to C: documents, the EI-3C candidate and EI-2B snapshot, frozen replay. `VerifyStored…` takes the **full** store port, but only reads |
| `application_services/audit_equity_recommendation_policy_bindings.py`: `AuditEquityRecommendationPolicyBindingsUseCase` | The read-only EI4-D10 audit. It also takes the **full** store port, but only reads |
| `application_services/verify_equity_synthetic_policy_bindings.py` | The S3 policy-set verifier used by the gate |

**Records** (`application_services/equity_recommendation_generation_records.py`):
- A disposition and an attempt verify their own documents on construction.
- The attempt identity `eirat1` is the fingerprint of the attempt document: it is content-addressed.
- The generation key `eirk1` binds the candidate and the policy set, **not** the assessment content.

**Infrastructure** (`src/northstar_infrastructure/persistence/`):
- `sqlite_equity_recommendation_generation.py`: `SQLiteEquityRecommendationGenerationStore` implements the port.
  - It is exported publicly from `persistence/__init__.py`.
  - Each write is one `BEGIN IMMEDIATE` transaction, with a single `_commit` boundary and W2's cleanup that never masks failures.
  - It refuses a database without the EI-2B and EI-3C tables.
  - It rebuilds every stored row through the Application record types.
- `sqlite_equity_recommendation_generation_schema.py`:
  - insert-only triggers;
  - CHECK constraints, including the six attempt statuses;
  - foreign keys to the CANDIDATE entry and snapshot;
  - the insert-time CANDIDATE checks of ADR-021, section 11.

**Composition and entry points:**
- No production composition root constructs the generation recorder or store.
- `northstar-api` (both scripts, `northstar-api` and the `northstar` CLI in `src/northstar_api/cli.py`) wires futures and options adapters only, and contains no equity-intraday symbol.
- `northstar-web` and `northstar-core` contain no reference.

**There is no operational equity generation-write entry point today.**

## 4. Confirmed write-path inventory

Every reference found by searching all five code repositories. Categories:
- **A** production operational entry point;
- **B** Application orchestration;
- **C** Infrastructure adapter;
- **D** test fixture or test;
- **E** historical read-only verifier or audit;
- **F** maintenance utility.

| # | Repository | File and symbol | Caller / entry point | Operation | Writes | Uses verified recorder | Raw store directly | Verification coverage | Category and risk | Remediation proposed |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | application | `record_equity_recommendation_generation.py`, `record` → `self._store.record_generation` | Recorder internals | Disposition, proposal and attempt in one write | Both | Is the recorder | Through the port | EI4-D10 gate; levels A to C; reconciliation; W1 mappings | B; the approved path | Becomes the only issuer of write authorizations ([section 8](#8-recommended-architecture)) |
| 2 | application | Same module, `record_attempt` → `self._store.record_attempt` | Recorder internals | Attempt-only | Attempts | Is the recorder | Through the port | Type check, no disposition kind; the attempt document self-verifies | B | Issues an attempt write authorization ([section 10](#10-attempt-only-path)) |
| 3 | application | `verify_equity_recommendation_generation.py`, `VerifyStoredEquityRecommendationGenerationUseCase` | Read-only verification | `load_disposition` | None | — | Holds the **full** port | Levels A to C | E; holds a write capability it never uses | Take the reader port only |
| 4 | application | `audit_equity_recommendation_policy_bindings.py`, `AuditEquityRecommendationPolicyBindingsUseCase` | Read-only audit | `load_disposition` | None | — | Holds the **full** port | Classification only | E; holds a write capability it never uses | Take the reader port only |
| 5 | infrastructure | `sqlite_equity_recommendation_generation.py`, `SQLiteEquityRecommendationGenerationStore.record_generation` and `.record_attempt` | **Any importer**; the class is public | Raw insert-only writes | Both | No | Is the raw store | Structure, foreign keys and insert-time CANDIDATE checks only | **C; the confirmed bypass surface** | Accept only recorder-issued write authorizations; remove the raw signatures |
| 6 | infrastructure | `tests/test_sqlite_equity_recommendation_generation.py` (22 direct `record_generation` / `record_attempt` call sites), `tests/test_sqlite_equity_recommendation_generation_reliability.py` (12) | Adapter contract tests and the W2 fault tests | Direct adapter writes with fault seams | Both | No | Yes | The adapter under test | D; legitimate adapter testing | Obtain authorizations from the real recorder ([section 15](#15-cross-repository-migration)) |
| 7 | infrastructure | `tests/test_equity_recommendation_generation_acceptance_faults.py`: `test_e17_a_real_constraint_violation_rolls_back_the_whole_write`; `_competitor` (used by e18); `test_e21_a_corrupt_stored_attempt_is_typed_by_the_adapter_and_mapped_by_the_recorder` | EI-4E fault tests | Direct adapter writes: a constraint seam, a competing writer, a typed error on write | Both | No | Yes | — | D; legitimate | As row 6; a competitor records through its own recorder |
| 8 | infrastructure | Same file: `test_e20_contradictory_candidate_evidence_is_caught_by_the_application_verifier` | The AA-2 demonstration | A forged, structurally valid disposition committed directly | Both | No (deliberately) | Yes | Structure only; the forged record later fails verification | D; **demonstrates the gap** | Becomes a negative test: the adapter refuses; the forged row can be planted only by the labelled raw-SQL `bypass()` |
| 9 | infrastructure | `tests/test_sqlite_equity_synthetic_policy_catalog.py`, `tests/test_equity_synthetic_policy_enforcement_acceptance.py`, the EI-4E fixtures | Through `Pipeline.recorder` / `World.recorder` | Recorder | Both | Yes | No | Full | D; compliant | None |
| 10 | application | `tests/application_services/test_record_equity_recommendation_generation.py` (`_Store`), `test_equity_synthetic_policy_enforcement.py`, `test_equity_synthetic_policy_catalog.py` (`_Records`) | In-memory port doubles | Double writes | Both (in memory) | Recorder under test | Doubles only | — | D | The doubles implement the new ports |
| 11 | api | — | No equity generation wiring; both scripts are futures and options | — | — | — | — | — | No A or B caller exists | Future wiring must use the recorder ([section 14](#14-infrastructure-enforcement)) |
| 12 | core, web | — | No reference | — | — | — | — | — | — | — |

No maintenance or migration utility (F) exists.

## 5. Remaining bypass risk

- **The confirmed surface is inventory row 5:** a public adapter whose `record_generation(disposition, attempt)` accepts any well-formed record.
- **Why dispositions matter most.** `eirk1` does not cover assessment content, so a structurally valid, historically contradictory disposition can be committed under a genuine key. Because rows are immutable, it then occupies that key. EI-4E `test_e20_…` demonstrates exactly this. EI4-D11-B already makes such a record fail closed, but it cannot be replaced (EI4-D11-D).
- **Attempts carry a smaller risk.** An attempt is content-addressed, so a direct `record_attempt` cannot occupy another attempt's identity. It can, however, add well-formed but fabricated attempt-only evidence, for example an outage that never happened.
- **The EI4-D10 gate does not close this.** The gate sits inside `record()`; a direct adapter call never reaches it.
- **Read-only consumers hold write capability.** `VerifyStored…` and the audit hold a write-capable port they do not need.
- **No operational caller exists today**, so the risk is **latent**: the next composition root, CLI command or background job could wire the raw adapter.

## 6. Trust and threat boundary

**In scope**, where enforcement must hold:
- ordinary supported Application operations, and any future production composition root (API, CLI, background job);
- the public Application services and ports;
- the supported Infrastructure persistence interface, including `SQLiteEquityRecommendationGenerationStore`;
- accidental or ordinary misuse: a new caller wiring the raw adapter, passing unverified records, or recording a forged disposition;
- source-level checks in CI, and runtime refusal when mandatory verification is absent.

**Out of scope**, where nothing here claims protection:
- a privileged user editing the SQLite file (ADR-022 DQ9 is unchanged);
- arbitrary Python execution with unrestricted module and filesystem access, including importing private names;
- a malicious administrator changing executable code;
- cryptographic publisher or writer identity, multi-host authorization, broker authentication and live-trading security.

**The honest claim.** Within the in-scope boundary, the only supported way to make a generation record durable is through the verified recorder. Every other supported path is refused at runtime before a transaction opens, or caught by architecture tests in CI.

A Python capability object is not unforgeable. Private constructors and module-private tokens stop **accidental and ordinary** misuse, not privileged code, and this design does not claim otherwise.

## 7. Alternatives evaluated

| Criterion | A. Composition-root rule + scans | B. Separate read and write ports | C. Verified-write authorization | **D. Combined: B + C + scans + database immutability** |
| --- | --- | --- | --- | --- |
| Stops a new caller wiring the raw adapter | Only if a scan sees it; nothing at runtime | No; the writer port is still callable with raw records | **Yes, at runtime**: the adapter refuses anything not issued by the recorder | **Yes**, at runtime and in CI |
| Least privilege for read-only consumers | No | **Yes** | No | **Yes** |
| Binding to exact content | n/a | n/a | Yes, by containment ([section 8.2](#82-the-verified-write-authorization)) | Yes |
| Commit-uncertainty retries | Unaffected | Unaffected | Reusable within bounded reconciliation; idempotent by first-writer immutability | Same |
| New canonical fingerprint | No | No | **No**; containment needs none | No |
| Forgeable by privileged Python | n/a | n/a | Yes (documented) | Yes (documented) |
| Test impact | Small | Doubles change | Infrastructure adapter tests must obtain authorizations | As C |
| Cross-repository breakage | None | Additive | Breaking unless staged | Staged without a red state ([section 16](#16-proposed-implementation-slices)) |
| Meets EI4-D11-E's "not a documentation convention" | Weak: CI-only | Partly: types are not enforcement | Yes | **Yes** |

**Also considered and rejected:**
- **A database-level verification marker** (a column or trigger requiring a token). SQLite cannot verify SHA-256, source or replay (ADR-021, section 10). A marker is forgeable by any writer, and adding it needs a rebuild of insert-only tables. Rejected.
- **A single-use, nonce-based authorization.** It would break the bounded retry with the same records that EI4-D9-G requires after an unconfirmed COMMIT, and it adds state to persist. Rejected; reuse is harmless under first-writer immutability.
- **Optional dependency injection or naming conventions.** Not enforcement (EI4-D11-E). Rejected.

**Recommendation: Option D.** Option C alone gives runtime refusal but leaves read-only consumers with write capability. Option B alone is least privilege without refusal. Option A alone is a convention checked in CI. Combined, they give:
- **runtime refusal** at the last point before a durable write;
- **least privilege** in the Application contracts;
- **CI detection** of new callers;
- the existing **database immutability** as defence in depth.

Residual bypasses are listed in [section 19](#19-risks-and-limitations).

## 8. Recommended architecture

```text
                       northstar-application (owns the ports and the records)
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │ EquityRecommendationGenerationRecordReader      (port: load_disposition,     │
 │   ▲ used by VerifyStored…, the audit, the recorder     load_attempt)         │
 │                                                                              │
 │ EquityRecommendationGenerationRecordWriter      (port: write_generation(     │
 │   ▲ used ONLY by the recorder                     VerifiedGenerationWrite),  │
 │   │                                               write_attempt(             │
 │   │                                                 VerifiedAttemptWrite))   │
 │   │                                                                          │
 │ RecordEquityRecommendationGenerationUseCase  ── the ONLY issuer of           │
 │   gate (EI4-D10) → levels A–C (verifier) →      Verified*Write authorizations │
 │   issue authorization → writer → reconcile                                    │
 └──────────────────────────────────────────────────────────────────────────────┘
                                   │ implements (dependency points to Application)
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │ northstar-infrastructure                                                     │
 │ SQLiteEquityRecommendationGenerationStore: reader + writer                   │
 │   write_*(authorization): validate authorization ──► BEGIN IMMEDIATE ─► COMMIT │
 │   (refused before any connection or transaction when invalid)                │
 └──────────────────────────────────────────────────────────────────────────────┘
```

The dependency direction is unchanged: Core ← Application ← Infrastructure. Application owns the ports, records and authorization types; Infrastructure owns SQLite.

### 8.1 Ports

The current `EquityRecommendationGenerationStore` is replaced, in stages ([section 16](#16-proposed-implementation-slices)), by:

- **`EquityRecommendationGenerationRecordReader`:** `load_disposition` and `load_attempt`, with the same contract as today. `VerifyStoredEquityRecommendationGenerationUseCase` and `AuditEquityRecommendationPolicyBindingsUseCase` depend on this only.
- **`EquityRecommendationGenerationRecordWriter`:** `write_generation(VerifiedEquityRecommendationGenerationWrite)` and `write_attempt(VerifiedEquityRecommendationAttemptWrite)`.
  - These keep exactly today's transactional semantics, outcomes (`EquityRecommendationStoreWrite`) and typed errors: atomicity, EQUIVALENT and CONFLICT, the attempt recorded with a conflict, and Unavailable, NotCommitted and CommitUnconfirmed.
  - Only the argument changes.

### 8.2 The verified-write authorization

**`VerifiedEquityRecommendationGenerationWrite`**, an Application record, holds:

- `verified`: the `EquityVerifiedRecommendationGeneration` the verifier returned (levels A to C), which contains the disposition;
- `policy`: the gate's `EquitySyntheticPolicySetVerification`, which must be verified;
- `attempt`: the logical attempt of that disposition.

**Binding by containment.** The writer persists `authorization.verified.disposition` and `authorization.attempt`, and nothing else. A verified authorization cannot be paired with other records, so no new fingerprint is needed.

**Construction checks** (`__post_init__`):
- the policy verification is verified, and its set identity and `eirs1` are the disposition's;
- the attempt is the disposition's attempt: the same key, assessment fingerprint and kind, as `record` checks today;
- the issuer token is the recorder module's private token.

So `dataclasses.replace` with other content fails, unless the caller also forges verification objects and imports the private token. That is privileged code, outside the boundary.

**`VerifiedEquityRecommendationAttemptWrite`** holds an attempt-only `EquityRecommendationGenerationAttempt` (no disposition kind) and the issuer token ([section 10](#10-attempt-only-path)).

**Reuse.** An authorization is immutable and content-bound. The recorder reuses it for its bounded retries after an unconfirmed COMMIT (EI4-D9-G), and re-recording it is EQUIVALENT under first-writer immutability. It is not single-use.

**Validation by Infrastructure.** The adapter checks only:
- the exact authorization type;
- that its records are present and consistent;
- the issuer token.

It computes no SHA-256 (ADR-021, section 10) and re-verifies nothing. The Application issued the authorization only after verifying.

> **Amended 2026-11-10 (clarification B, exact write-content binding).** These rules are normative and supersede the looser wording above where they differ.
>
> - **Covered operation.** A generation authorization corresponds to one exact verified operation:
>   - the disposition, including its proposal when it has one;
>   - the logical attempt;
>   - the verifier's levels A to C result: source, snapshot and frozen-evidence replay;
>   - the verified policy-binding check.
> - **Covered identity.** At issuance it captures that operation's identity from **existing** canonical values only:
>   - generation key, kind and evaluator version;
>   - `eira1` assessment fingerprint;
>   - `eirp1` proposal fingerprint or none;
>   - `eirat1` attempt identity;
>   - `eirs1`.
>
>   No new fingerprint scheme is introduced. `eirk1` alone is never sufficient: it does not cover assessment content. Policy labels, a "verified" flag and caller-supplied identifiers are never sufficient either.
> - **Validation.** An **Application-owned validation function**, which the adapter calls, re-derives that identity from the contained records and refuses on any difference. It also refuses when:
>   - the verifier result does not hold exactly that disposition;
>   - the policy check is not verified for that disposition's set;
>   - the attempt is not that disposition's attempt;
>   - the issuer contract is not met.
>
>   Infrastructure never imports the issuer token.
> - **No separate payload channel.** The writer takes the authorization alone, so there is no channel for a different payload. If an implementation also passes records, the adapter must require exact equality with the covered content.
> - **Reuse.** One authorization is issued per `record()` operation. It is reused **only** for the identical operation during the existing bounded EI4-D9-G reconciliation.
>   - It is never minted again during reconciliation.
>   - It is never minted to cover contradictory content under the same generation key.
>
>   A different verified disposition under an occupied key is a separate operation, which yields `CONFLICT` under EI4-D9-D. No one-time nonce is used.
> - **Attempt-only.** `VerifiedEquityRecommendationAttemptWrite` binds to the exact attempt's `eirat1` identity and document ([section 10](#10-attempt-only-path)).
> - **Limitation.** Within the approved personal-use trust model, the issuer capability is not tamper-proof against privileged code.

### 8.3 Where verification becomes mandatory

1. **The recorder** issues an authorization only after the EI4-D10 gate passes and the verifier returns a verified result. The S3 recorder already performs both; today it discards the verifier's result. EI4-D11 keeps it and wraps it.
2. **The last point before a durable write** is the adapter's write entry. An unsupported call must be rejected there, **before `_connect`, before `BEGIN IMMEDIATE`**. This applies to:
   - an argument that is not the exact authorization type;
   - an authorization without the issuer token;
   - records that are inconsistent.

   The proposed error is a new typed `UnverifiedEquityRecommendationGenerationWriteError` (Application port), raised as a programming error. It is never mapped to a recording outcome, because no supported caller can produce it.

   > **Amended 2026-11-10.** The refusal also covers mismatched exact contents (clarification B) and an unauthorized attempt-only write (EI4D11-DQ3). It happens before opening a SQLite connection, beginning a transaction or inserting evidence.

**What S3 already covers, and what EI4-D11 adds:**
- **Already covered by S3:** the policy gate, source and replay verification, W1 mappings and reconciliation inside the recorder. The catalog verifier is reused, not reimplemented.
- **Added by EI4-D11:**
  - making the recorder the only issuer of writes;
  - refusing unverified writes at the adapter;
  - removing write capability from read-only consumers;
  - architecture tests.

## 9. Generation disposition path

```text
recorder.record(disposition, attempt)
  1. EI4-D10 gate  ─ not verified ─► POLICY_REFERENCE_* (no write; unchanged)
  2. verifier.verify(disposition) ─ refuses ─► SOURCE_EVIDENCE_* / REPLAY_MISMATCH / STORED_RECORD_INCOMPATIBLE (unchanged)
  3. authorization = VerifiedEquityRecommendationGenerationWrite(verified, policy, attempt, issuer)
  4. writer.write_generation(authorization)   ── one BEGIN IMMEDIATE transaction (unchanged)
       COMMITTED / EQUIVALENT / CONFLICT ─► interpreted exactly as today
       CommitUnconfirmed ─► reload; retry with the SAME authorization, at most max_writes (EI4-D9-G)
```

Every existing outcome, invariant and identity is preserved:
- first-writer authority;
- atomic disposition, proposal and attempt;
- deterministic `eirat1`;
- content and evaluator-version conflicts;
- the W1 mappings;
- bounded reconciliation.

A disposition that fails any check is never written and never authorized. A failed policy-bearing disposition is never recorded as verified.

## 10. Attempt-only path

- **Genuine attempt-only results** (no candidate, outages, early refusals, EI4-D10 gathering failures) have no inputs and no policy content. They must **not** be forced to manufacture a policy bundle or pass the policy gate.
- **The recorder's checks** for `record_attempt` stay as today: it is an `EquityRecommendationGenerationAttempt` with no disposition kind, and its document self-verifies with a content-addressed `eirat1`.
- **Issuance.** The recorder then issues a `VerifiedEquityRecommendationAttemptWrite`, and the adapter's `write_attempt` accepts only that.
- **What this closes.** It closes the smaller attempt bypass of [section 5](#5-remaining-bypass-risk): fabricated attempt evidence through the adapter. It adds no verification an attempt cannot have.
- **Owner decision.** Whether attempts are inside the boundary at all is EI4D11-DQ3. EI4-D11-A names dispositions.

> **Amended 2026-11-10 (EI4D11-DQ3, OWNER APPROVED).** Attempt-only writes are inside the boundary.
>
> - **A separate contract.** `VerifiedEquityRecommendationAttemptWrite` is a separate authorization. The recorder issues it for the actual attempt being persisted, bound to that attempt's exact `eirat1` identity and document.
> - **Nothing fabricated.** It requires no policy evidence, and fabricates no assessment, proposal or successful disposition. Attempt identity and status formats are unchanged.
> - **The bypass is closed.** A fabricated direct attempt-only write through a supported adapter path is refused before SQLite opens.

## 11. Invalid committed evidence

These behaviours are unchanged and required by EI4-D11-B and EI4-D11-C. The supported flow never overwrites or deletes:

| Situation | What a blocked caller observes |
| --- | --- |
| Invalid stored disposition or attempt | Recorder: `STORED_RECORD_INVALID`, `attempt_recorded` None or as established; verifier: typed error |
| Incompatible stored record or evaluator | `STORED_RECORD_INCOMPATIBLE`; `EVALUATOR_VERSION_CONFLICT` for the same content under another evaluator |
| Missing provenance; contradictory committed content | Stored other content that fails verification: `STORED_RECORD_INVALID`, or `SOURCE_EVIDENCE_UNAVAILABLE` when the source cannot be read; never adopted (the e20 consequence) |
| Valid conflicting content | `CONTENT_CONFLICT`; the first writer stays authoritative |
| Uncertain commit | `COMMIT_RECONCILIATION_UNRESOLVED`; nothing claimed |

**Surfacing for investigation.** The existing typed outcomes and details, `VerifyStoredEquityRecommendationGenerationUseCase` and the EI4-D10 audit are the investigation surfaces, and all are read-only. This design adds **no** recovery, replacement, deletion, migration or maintenance write path. An occupied key stays occupied until a separately approved recovery design exists (EI4-D11-D). That is a known, accepted consequence ([section 19](#19-risks-and-limitations)).

## 12. Historical replay compatibility

- Replay and read-only verification use the **reader** port only and need no authorization.
- Replay never reads the policy catalog (ADR-022, section 9), and needs no publication evidence that never existed.
- No row, document, fingerprint or identity is migrated. The authorization is an in-memory Application object and is **never persisted**, so the schema and canonical formats are unchanged.
- Records written before enforcement remain readable and verifiable by their original rules. Re-recording them goes through the recorder like any new write.

## 13. Public interface and port changes

| Change | Kind | Where |
| --- | --- | --- |
| `EquityRecommendationGenerationRecordReader` | New port | `ports/equity_recommendation_generation_store.py` |
| `EquityRecommendationGenerationRecordWriter`, with `write_generation` and `write_attempt` | New port | Same |
| `VerifiedEquityRecommendationGenerationWrite`, `VerifiedEquityRecommendationAttemptWrite` | New records; no public factory | `application_services/` |
| `UnverifiedEquityRecommendationGenerationWriteError` | New error | `ports/` |
| `record_generation(disposition, attempt)` and `record_attempt(attempt)` on the port | **Removed**, after a staged transition | `ports/` |
| `VerifyStored…` and the audit | Constructor parameter type narrowed to the reader | Their modules |
| The recorder's constructor | Takes a reader-and-writer store; public signature unchanged in shape | `record_equity_recommendation_generation.py` |

No canonical document, fingerprint, key, outcome value or attempt status changes.

## 14. Infrastructure enforcement

- **`SQLiteEquityRecommendationGenerationStore`** implements both ports. `write_generation` and `write_attempt` validate the authorization ([section 8.3](#83-where-verification-becomes-mandatory)) and then run today's `_write` unchanged. That includes the single `_commit` boundary and W2's cleanup.
- **The raw `record_generation` and `record_attempt` are removed.** In the transition they refuse with `UnverifiedEquityRecommendationGenerationWriteError` ([section 16](#16-proposed-implementation-slices)).
- **The schema, triggers, constraints and foreign keys are unchanged** and remain defence in depth.
- **Architecture tests (source scans) in both repositories:**
  - outside the recorder module, nothing constructs a `Verified*Write` or references the issuer token;
  - outside the recorder and the adapter, nothing calls `write_generation` or `write_attempt`;
  - no production module (Application, Infrastructure or API source) constructs `SQLiteEquityRecommendationGenerationStore` for writing except a composition root that passes it to the recorder (none exists today);
  - tests reach the issuer only through the real recorder ([section 15](#15-cross-repository-migration)).
- **Future composition roots** (API, CLI, background) must construct the recorder and pass the adapter only to it. The architecture tests enforce this in each repository that adds such wiring.

> **Amended 2026-11-10.**
>
> - **Validation.** The adapter validates through the Application-owned validation function of [section 8.2](#82-the-verified-write-authorization): exact contents, issuer contract and type. It never imports the issuer token.
> - **Test-only corruption.** Raw-SQL injection (the existing `bypass()` in the EI-4E fixtures) stays separate from the production adapter surface and labelled test-only. It is never a supported operational path.
> - **Read-only access stays.** `load_disposition` and `load_attempt` remain available (EI4D11-DQ5).

## 15. Cross-repository migration

**Affected production modules:**
- **Application:** `ports/equity_recommendation_generation_store.py`, `ports/__init__.py`, `application_services/record_equity_recommendation_generation.py`, `application_services/verify_equity_recommendation_generation.py`, `application_services/audit_equity_recommendation_policy_bindings.py`, a new `application_services/equity_recommendation_generation_write_authorization.py`, `application_services/__init__.py`.
- **Infrastructure:** `persistence/sqlite_equity_recommendation_generation.py`.

**Affected tests:**
- **Application:** `tests/application_services/test_record_equity_recommendation_generation.py` (the `_Store` double), `test_equity_synthetic_policy_enforcement.py`, `test_equity_synthetic_policy_catalog.py` (the `_Records` double), and a new `test_equity_recommendation_generation_write_boundary.py`.
- **Infrastructure:** `tests/test_sqlite_equity_recommendation_generation.py`, `tests/test_sqlite_equity_recommendation_generation_reliability.py`, `tests/test_equity_recommendation_generation_acceptance_faults.py` (e17, e18 competitor, e20, e21), `tests/equity_recommendation_generation_fixtures.py`, `tests/equity_recommendation_acceptance_fixtures.py`, and a new `tests/test_equity_recommendation_generation_write_boundary_acceptance.py`.

**How tests obtain authorizations** (EI4D11-DQ6):
- Adapter-level tests capture a **real** authorization by running the real recorder against a capturing writer test double, then pass it to the SQLite adapter with its fault seams. No test imports the issuer token.
- A forged or contradictory record cannot be authorized. Tests that need one committed (the e20 consequence) plant it with the existing, labelled test-only raw-SQL `bypass()`. This is privileged access, outside the boundary.

**CI coupling.** Infrastructure CI checks out Application `develop`. Multi-repository commits are not atomic. The slices below are ordered so that **each PR is green against the other repository's `develop`**, with no temporary red state. That avoids the S3/S4 coupling of EI4-D10, where Infrastructure was red between the merges.

## 16. Proposed implementation slices

| Slice | Repository | Purpose | Production changes | Test changes | Depends on | Compatibility | Acceptance evidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **W1. Additive contracts** | application | Introduce the reader and writer ports and the authorization records; the recorder issues authorizations | Add the reader port, with the current store port extending it. Add the writer port, whose `write_*` methods have a **concrete default** delegating to the legacy `record_*`. Add the authorization records and private issuer. The recorder keeps the verifier's result and calls `write_*`. `VerifyStored…` and the audit take the reader | Contract tests: issuance only after verification; forged or mismatched authorizations refused at construction; reuse in reconciliation | — | **Non-breaking.** The existing SQLite adapter inherits the defaults and Infrastructure stays green | Application suite green; Infrastructure suite green against the new Application `develop` |
| **W2-I. Adapter enforcement** | infrastructure | The adapter accepts only authorizations | Override `write_*` with validation before `_connect`. Make legacy `record_*` refuse with `UnverifiedEquityRecommendationGenerationWriteError`. No schema change | Adapter tests capture authorizations through the real recorder. e20 becomes "adapter refuses, raw SQL plants". Competitors record through their own recorders. New negative tests | Application W1 merged | **Non-breaking for Application.** Infrastructure is green against Application `develop` | Infrastructure suite green; the e20 negative path; refusal before any transaction |
| **W3. Remove the raw signatures** | application | The port has no raw write | Remove `record_generation` and `record_attempt` from the port; make `write_*` abstract. Add architecture tests | Doubles implement the new ports | Infrastructure W2-I merged | The adapter already implements `write_*`; its legacy methods become extra, refusing methods. **Green in both** | Both suites green; architecture tests |
| **W4-I. Remove legacy adapter methods; acceptance** | infrastructure | Clean the adapter; end-to-end acceptance | Remove the refusing `record_*` | Architecture scans; the end-to-end acceptance module ([section 17](#17-acceptance-matrix)) | Application W3 merged | Green | The full acceptance matrix |
| **W5. Documentation acceptance** | docs | Acceptance record | — | — | W4-I | — | Record and register entry |

Each slice is one PR, independently reviewable and merged in the order shown. Required CI checks are never bypassed. Slice names avoid the EI4-D10 `S1`–`S5` labels; the `W` prefix here is unrelated to the earlier W1 and W2 reliability fixes, and implementation may choose other names.

> **Amended 2026-11-10 (clarification A, OWNER APPROVED order W1 → W2-I → W3 → W4-I → W5).**
>
> - **W1 migrates the recorder first.** It adds the read-only and authorized-write port contracts, both authorization types, recorder issuance, the validation function, the error, and the temporary compatibility defaults. The verified recorder **already calls** `write_generation` and `write_attempt` in W1, before W2-I refuses legacy raw writes.
> - **W1's defaults are compatibility, not enforcement.** The defaults validate the authorization, then delegate through dynamic dispatch to the legacy `record_*`. W1 alone does not complete EI4-D11.
> - **W2-I enforces.** It refuses, before any SQLite connection: a missing, wrong-type or unissued authorization; mismatched exact contents; and an unauthorized attempt-only write. Legacy `record_*` refuse operational use. Tests are migrated, and test-only raw-SQL corruption is isolated and labelled.
> - **W3 and W4-I remove the raw write signatures**, from the port and then from the adapter.
> - **W5 records engineering acceptance.**
> - **The compatibility invariant.** Each slice is tested against the other repository's current `develop` before merge. Compiling is not evidence of compatibility.
>
> The constraints already found in source are in [section 22.4](#224-migration-compatibility-constraints-found-in-source). Any further gap is reported for engineering review before implementation, never closed by weakening enforcement.

## 17. Acceptance matrix

All tests below are proposed. **Status for every row: PROPOSED / NOT EXECUTED.**

Layers:
- **C:** Application contract test.
- **S:** Infrastructure SQLite test.
- **E:** end to end on real SQLite.
- **A:** architecture or source scan.

| # | Requirement | Layer | Expected result | Negative assertion | Slice | Proposed module |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Valid generation through the verified recorder | C, E | COMMITTED; disposition and attempt stored atomically | No write before the gate and levels A to C pass | W1, W4-I | `test_equity_recommendation_generation_write_boundary.py`; `test_equity_recommendation_generation_write_boundary_acceptance.py` |
| 2 | Valid attempt-only recording (amended 2026-11-10) | C, E | COMMITTED attempt through the recorder, under a `VerifiedEquityRecommendationAttemptWrite` for that exact attempt; no disposition | No policy bundle demanded for a genuine failed-input attempt; no assessment or proposal fabricated | W1, W4-I | Same |
| 3 | Direct raw-store write refused (amended 2026-11-10; negative test A) | S, E | `UnverifiedEquityRecommendationGenerationWriteError` from a direct, unsupported call to the legacy generation-write entry (W2-I); the method is absent after W4-I | No SQLite connection opened (counting `_connect`); no transaction begun; no disposition or attempt inserted; original evidence byte-identical | W2-I, W4-I | Same S/E modules |
| 4 | Missing mandatory recorder dependency | C | TypeError at construction (catalog, verifier, store) | No partially wired recorder | W1 | C module |
| 5 | Missing verification evidence | C, S | A raw disposition or attempt passed to `write_*` is refused | Zero rows | W1, W2-I | C, S modules |
| 6 | Forged or mismatched authorization (amended 2026-11-10; negative test B) | C, S | An authorization issued for one verified operation cannot write a different disposition, proposal or attempt, **even under the same generation key**. Refused at construction (`dataclasses.replace`), or by the validation function when contents are swapped after issuance. Also refused: an unverified policy, the wrong attempt, a missing token | Never written; no connection opened | W1, W2-I | C, S modules |
| 7 | Unpublished policy definition | C, E | `POLICY_REFERENCE_NOT_ESTABLISHED`; no authorization issued | Zero writes | W1, W4-I | C, E modules |
| 8 | Conflicting policy binding | C, E | `POLICY_REFERENCE_CONFLICT`; no authorization | Zero writes | W1, W4-I | C, E modules |
| 9 | Unavailable policy catalog | C, E | `POLICY_REFERENCE_UNAVAILABLE` | Zero writes | W1, W4-I | C, E modules |
| 10 | Invalid frozen inputs | C | Refused at record construction or replay (`REPLAY_MISMATCH`) | No authorization | W1 | C module |
| 11 | Invalid source provenance (amended 2026-11-10; negative test D, guarantee 1) | C, E | `SOURCE_EVIDENCE_INVALID` from the recorder; the e20 forgery is **also refused by the adapter** when written directly, without an authorization | Zero rows from any supported path; no connection opened | W1, W2-I | C; acceptance faults (e20, part 1) |
| 12 | Invalid existing committed evidence (amended 2026-11-10; negative test D, guarantee 2) | E | The e20 forged row, planted through **labelled test-only raw SQL** (not a supported path), still fails closed: `VerifyStored…` refuses it naming the field, and the genuine record gets `STORED_RECORD_INVALID` | Never overwritten, deleted or adopted; no recovery path exercised | W2-I | Acceptance faults (e20, part 2) |
| 13 | Incompatible stored records | S, E | `STORED_RECORD_INCOMPATIBLE` | Nothing migrated | W2-I | Existing e21, migrated |
| 14 | Conflicting committed content | C, E | `CONTENT_CONFLICT`; the first writer is authoritative | Not replaced | W1, W4-I | C, E modules |
| 15 | Evaluator-version conflict | C, S | `EVALUATOR_VERSION_CONFLICT` | Not adopted | W1, W2-I | C; existing `test_o_*`, migrated |
| 16 | Definite rollback | S | `NOT_COMMITTED`; zero rows | No commit claim | W2-I | Reliability module, migrated |
| 17 | COMMIT succeeds, acknowledgment lost | S, E | `EQUIVALENT_COMMITTED` after reload; the same authorization is reused for the identical operation | No duplicate; no second authorization minted | W2-I | Reliability, E modules |
| 18 | COMMIT uncertain, reload fails | S | `COMMIT_RECONCILIATION_UNRESOLVED` | Nothing claimed | W2-I | Reliability module |
| 19 | Duplicate equivalent recording | C, E | `EQUIVALENT_COMMITTED`; one row | No second disposition | W1, W4-I | C, E modules |
| 20 | Independent SQLite connections | S | Two recorders: one COMMITTED, one EQUIVALENT or CONFLICT | One authoritative row | W2-I | Existing `test_aa_*`, `test_ab_*`, migrated |
| 21 | Windows separate processes | E | Spawned recorders serialize | One row | W2-I | Existing e19, migrated |
| 22 | Historical replay without the current catalog | C, E | Verified through the reader only | No authorization or write needed | W1, W4-I | C, E modules |
| 23 | No evidence backfill or migration | E | Rows byte-identical across the slices | No row rewritten | W4-I | E module |
| 24 | Existing EI-4D-B, EI-4E and EI4-D10 acceptance | S, E | Suites pass, with only the listed expectation changes (e20, competitors) | No weakened gate | W2-I, W4-I | Existing suites |
| 25 | No issuance or broker behaviour | A, E | No issuance, history or lifecycle write | — | W4-I | E module; scans |
| 26 | Read-only consumers cannot write | A, C | `VerifyStored…` and the audit hold only the reader | No `write_*` reachable through their dependencies | W1, W3 | C module; scans |
| 27 | Only the recorder issues authorizations | A | Source scan: no other constructor or token use in either repository | — | W3, W4-I | Scan modules |
| 28 | Refusal precedes any transaction (amended 2026-11-10) | S | An unverified call to either write path, generation or attempt-only, opens no connection (counting `_connect`) | No lock is held; no transaction begun | W2-I | S module |
| 29 | Composition-root rule | A | No production module constructs the adapter for writing outside a recorder wiring | — | W3, W4-I | Scans (Application, Infrastructure; API when equity wiring appears) |
| 30 | Fabricated direct attempt-only write refused (added 2026-11-10; negative test C) | S, E | A direct attempt-only write without a recorder-issued `VerifiedEquityRecommendationAttemptWrite`, or with one for another attempt, is refused | No connection opened; no attempt inserted; the genuine attempt path (row 2) is unaffected | W2-I, W4-I | S, E modules |
| 31 | Migration ordering (added 2026-11-10; negative test E) | C, A, S | After W1, every recorder write path calls `write_generation` or `write_attempt` and never the legacy `record_*`. The Infrastructure suite at Infrastructure `develop` passes against W1 through the compatibility defaults (dynamic dispatch). At W2-I every recorder path still passes once the legacy methods refuse | No recorder path depends on a raw write when W2-I lands | W1, W2-I | C module; scans; Infrastructure suite against W1 |
| 32 | Reuse only for the identical operation (added 2026-11-10) | C, S | During bounded reconciliation the recorder reuses the one authorization of that operation, at most `max_writes` writes. No authorization is minted during reconciliation, and none covers different content under the same key | Retries stay bounded; no nonce blocks a legitimate retry | W1, W2-I | C, reliability modules |
| 33 | Restart and durable persistence (added 2026-11-10; EI4D11-DQ8) | E | Records written through authorizations reload after the database file is closed, moved and reopened. Re-recording the same verified operation is `EQUIVALENT_COMMITTED` | No authorization is persisted; no row changes | W4-I | E module |

Rows 26 to 29 are added because the actual architecture requires them. POSIX and multi-host behaviour is not proposed for acceptance (EI4D11-DQ8).

> **Note, 2026-11-10.** Rows 2, 3, 6, 11, 12, 17 and 28 are amended for clarifications A and B and the approved decisions; their identifiers are unchanged. Rows 30 to 33 are added. Coverage of the required negative tests:
>
> - **A:** row 3.
> - **B:** row 6.
> - **C:** rows 2 and 30.
> - **D:** rows 11 and 12, which prove different guarantees.
> - **E:** rows 31, 26, 29 and 32.
>
> The matrix holds **33 scenarios**. Every one is **PROPOSED / NOT EXECUTED**, and no EI4-D11 engineering acceptance is established.

## 18. Engineering decisions requiring approval

None of these is approved. "Owner approval" marks decisions that change the governed boundary, compatibility or acceptance scope. The others are engineering choices within an approved option, listed for review.

| ID | Question | Options | Recommended | Rationale | Risks | Compatibility | Owner approval |
| --- | --- | --- | --- | --- | --- | --- | --- |
| EI4D11-DQ1 | Operational trust boundary | (a) as [section 6](#6-trust-and-threat-boundary); (b) also claim protection against privileged code | (a) | Honest; consistent with ADR-022 DQ9 | Privileged bypass stays possible | None | **YES** |
| EI4D11-DQ2 | Enforcement mechanism | A, B, C, D ([section 7](#7-alternatives-evaluated)) | D | Runtime refusal, least privilege and CI detection | Python-level forgeability (documented) | Port change, staged | **YES** |
| EI4D11-DQ3 | Are attempt-only writes inside the boundary? | (a) yes, through an attempt authorization; (b) dispositions only, as EI4-D11-A literally names | (a) | Stops fabricated attempt evidence through the adapter; needs no policy bundle | Slightly more test migration | Same slices | **YES** |
| EI4D11-DQ4 | Authorization binding and reuse | Containment and cross-checks, reusable; a new fingerprint; single-use nonce | Containment, reusable | No new identity scheme; EI4-D9-G retries keep working | None material | None | NO: an engineering detail of DQ2 |
| EI4D11-DQ5 | Raw adapter write methods | Remove; keep and deprecate; keep refusing | Remove after a refusing transition | Leaves no ordinary bypass | Breaking for any external caller (none exists) | Staged in W2-I and W4-I | **YES** |
| EI4D11-DQ6 | Test access to authorizations | Capture through the real recorder; import the private token in tests; a public test factory | Capture through the recorder; forged rows only by labelled raw SQL | Tests never hold the issuer; a public factory would be a bypass | e20's expectation changes | Test-only | NO: but e20's changed expectation is disclosed |
| EI4D11-DQ7 | Cross-repository sequence | Five ordered PRs with no red state; one coordinated pair, as S3/S4 were | The five ordered PRs | No red window; CI always meaningful | More PRs | [Section 16](#16-proposed-implementation-slices) | NO: engineering; recorded for review |
| EI4D11-DQ8 | Acceptance platforms | Windows only (as EI4-D10); Windows and POSIX | Windows only, with POSIX stated as not established | Matches the available evidence | POSIX untested | None | **YES** |
| EI4D11-DQ9 | Investigation surface for blocked keys | Existing read-only outcomes, verifier and audit; a new read-only listing; a recovery path | The existing read-only surfaces | Recovery is not approved (EI4-D11-D) | An occupied key stays occupied | None | NO: no new capability; recovery stays pending |
| EI4D11-DQ10 | Architecture record | A new ADR (ADR-023) after approval; amend ADR-021 | A new ADR after approval | Mirrors EI4-D10-DQ8; ADR-021 stays unchanged | None | Docs only | **YES** |

Nothing here asks the owner to reapprove EI4-D11-A to EI4-D11-E, ADR-021 or ADR-022.

> **Note, 2026-11-10.** EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8 and DQ10 are **OWNER APPROVED, 2026-11-10**, each as its "Recommended" column states, with clarifications A and B ([section 22](#22-owner-approval-2026-11-10)). DQ4, DQ6, DQ7 and DQ9 remain engineering choices that need no separate approval. DQ4 is now constrained by clarification B, and DQ7 is the approved W1 → W2-I → W3 → W4-I → W5 order.

## 19. Risks and limitations

- **Privileged bypass remains.** Raw SQL with file access, dropping triggers, or importing the private issuer token (ADR-022 DQ9). Not claimed as protected.
- **Python-level forgeability.** An authorization is a capability object, not cryptographic proof.
- **Occupied keys.** A key already occupied by an invalid record, including any planted before enforcement, stays occupied. Recovery needs EI4-D11-D's separate design.
- **No existing operational caller.** Enforcement is validated by tests and scans. When the first composition root is added, its repository must include the architecture scan (row 29).
- **Test migration.** 38 direct adapter call sites in Infrastructure tests change: 22 in the EI-4D-B adapter tests, 12 in W2, and 4 in the EI-4E faults. e20's assertion changes from "the adapter commits it" to "the adapter refuses; raw SQL plants it". The EI-4E record's historical evidence is unchanged.
- **Platform coverage.** Windows only (EI4D11-DQ8).
- **Other stores.** Futures, options and paper persistence are not addressed. Their adapters remain as they are.
- **L9 cleanup findings** from EI4-D10 acceptance remain separate.

## 20. Explicit non-authorizations

This proposal authorizes nothing. It does not authorize:
- implementation of any slice;
- invalid-record recovery, deletion, update, replacement of an occupied key, fingerprint rewriting, or any maintenance or repair write path;
- schema changes or historical backfill;
- changes to `eirs1`, `eirk1`, `eirat1`, `eipd1`, canonical document versions or the evaluator version;
- changes to the EI4-D10 architecture, the S3 gate or ADR-022's DQ9 boundary;
- API endpoints;
- recommendation issuance, the lifecycle `RECORDED` state, broker or paper orders, fills, positions, P&L, live data or production trading;
- EI-5 or EI-6.

Every restriction of the [EI-4 Decision Register, section 7](../product/Equity-Intraday-V1-EI4-Decision-Register.md#7-production-readiness-restrictions) applies.

## 21. Related documents

- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md), sections 9 to 12
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md)
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- [EI4-D10 Policy-Version Identity Enforcement Design](Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md)
- [EI4-D10 Engineering Acceptance Record](../product/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md)
- [Equity Intraday V1 EI-4E Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md), section 14.2
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md) (added 2026-11-10, [section 22](#22-owner-approval-2026-11-10))

## 22. Owner approval (2026-11-10)

This section records an approval given on **2026-11-10**. It does not rewrite sections 1 to 21, which keep the proposal as reviewed, except for the dated amendments that incorporate the clarifications. The approved architecture is recorded in [ADR-023](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md) and the [EI-4 Decision Register, section 13](../product/Equity-Intraday-V1-EI4-Decision-Register.md#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10).

### 22.1 Approved decisions

| ID | Approved | Status |
| --- | --- | --- |
| EI4D11-DQ1 | The trust boundary of [section 6](#6-trust-and-threat-boundary): supported workflows, composition roots, public Application interfaces, supported Infrastructure persistence interfaces, ordinary raw-adapter misuse. Not privileged Python, direct SQL or file edits, malicious code, cryptographic forgery, multi-host authorization or trading security. ADR-022 DQ9 unchanged | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ2 | Option D: layered port separation, the recorder as the boundary, recorder-issued authorization, pre-connection adapter validation, CI architecture checks, and SQLite immutability as defence in depth | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ3 | Attempt-only writes are inside the boundary under a separate attempt authorization; no fabricated policy evidence, assessment, proposal or disposition | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ5 | Legacy raw write methods are removed after a phased migration; temporary compatibility methods only during it; read-only access stays; test-only corruption stays separate | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ8 | Windows is the required acceptance platform. POSIX separate-process and multi-host behaviour are not claimed | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ10 | ADR-023 records the architecture | **OWNER APPROVED, 2026-11-10** |
| EI4D11-DQ4, DQ6, DQ7, DQ9 | Engineering choices; no separate approval. DQ4 is constrained by clarification B; DQ6 preserves e20 (rows 11 and 12); DQ7 is the order of 22.2; DQ9 uses existing read-only surfaces | Not owner decisions |

### 22.2 Clarification A: safe migration order

The approved order is **W1 → W2-I → W3 → W4-I → W5**:
- **W1** migrates the verified recorder to the authorized-write interface before **W2-I** refuses legacy raw writes.
- W1's temporary defaults delegate to the legacy methods. They are a compatibility mechanism, **not** enforcement: W1 alone does not complete EI4-D11.
- Each slice is tested against the other repository's current `develop`. Merges are not atomic, and required CI is never bypassed.

Incorporated in [section 16](#16-proposed-implementation-slices).

### 22.3 Clarification B: exact write-content binding

- **Exact binding.** An authorization binds to the exact verified operation: the disposition with its proposal, the attempt, the verified source and frozen-evidence results, and the verified policy check. It never binds by `eirk1` alone.
- **Existing identities only.** It uses existing canonical documents and identities; no new fingerprint scheme is approved.
- **Adapter refusal.** The adapter refuses mismatched contents before any SQLite connection.
- **Reuse.** Reuse is only for the identical operation during bounded EI4-D9-G reconciliation. There is no blanket nonce, and no second authorization for contradictory content.
- **Limitation.** The issuer capability is not tamper-proof against privileged code.

Incorporated in [sections 8.2](#82-the-verified-write-authorization) and [8.3](#83-where-verification-becomes-mandatory).

### 22.4 Migration compatibility constraints found in source

These were read at Application `cd2301b` and Infrastructure `97700b3`. They constrain implementation without weakening enforcement. They are reported here for engineering review before W1 begins.

1. **W1 keeps the existing port intact.**
   - Infrastructure `tests/test_sqlite_equity_recommendation_generation.py::test_a_the_adapter_implements_the_merged_application_port` asserts that:
     - the port `EquityRecommendationGenerationStore` declares exactly `load_disposition`, `load_attempt`, `record_generation` and `record_attempt` as abstract methods;
     - the adapter's public names are exactly those four.
   - To stay green against Infrastructure `develop`, W1 must keep that port name with those four abstract methods, and add `write_*` only as **concrete** defaults (on the combined port, or a mixin it inherits).
   - W2-I must rewrite that test so it holds both before and after W3. Otherwise W3's removal of `record_*` turns Infrastructure red.
2. **W1's defaults dispatch dynamically.**
   - Infrastructure test subclasses override adapter methods. `_Paused` (in the same module) overrides `record_generation` to pause a racing writer.
   - W1's default `write_generation` must therefore delegate through `self.record_generation(...)`, not a direct internal call; otherwise those race tests stop pausing and fail.
   - W2-I moves such overrides to `write_generation`.
3. **The issuer check is Application-owned.** The adapter's issuer and content check must be an Application-owned function on the authorization contract. Infrastructure never imports the private issuer token.
4. **The recorder keeps the verifier's result.** Today the recorder's `_verified` discards the `EquityVerifiedRecommendationGeneration` that the verifier returns. W1 must keep it, to contain it in the authorization.

### 22.5 Status

| Item | Status |
| --- | --- |
| EI4-D11 governance (EI4-D11-A to EI4-D11-E) | APPROVED (2026-11-08, unchanged) |
| EI4-D11 enforcement design | **OWNER APPROVED (2026-11-10)** |
| Implementation (W1 to W4-I) | **NOT YET IMPLEMENTED** |
| Engineering acceptance (W5) | **NOT YET ESTABLISHED.** All 33 scenarios are PROPOSED / NOT EXECUTED |
| Invalid-record recovery | **NOT APPROVED** |

> **Note, 2026-11-11.** The table above records the status at approval. Implementation (W1 to W4-I) is now merged, and engineering acceptance is recorded as PASS WITH DOCUMENTED LIMITATIONS in the [EI4-D11 Engineering Acceptance Record](../product/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Engineering-Acceptance-Record.md). Invalid-record recovery is still not approved.

The EI4-D10 architecture and its engineering acceptance are unchanged. Every non-authorization of [section 20](#20-explicit-non-authorizations) still applies.
