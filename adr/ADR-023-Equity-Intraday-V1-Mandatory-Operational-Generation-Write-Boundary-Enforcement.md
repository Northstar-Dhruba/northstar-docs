# ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement

## Status

Accepted

The product owner approved EI4-D11, Mandatory Verified Recording Boundary and Invalid-Record Recovery Governance, on 2026-11-08 ([EI-4 Decision Register, section 10.3](../product/Equity-Intraday-V1-EI4-Decision-Register.md#103-ei4-d11----mandatory-verified-recording-boundary-and-invalid-record-recovery-governance-aa-2)). On 2026-11-10 the owner approved six engineering decisions of the [EI4-D11 enforcement design](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md) (EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8 and DQ10), with two mandatory clarifications ([EI-4 Decision Register, section 13](../product/Equity-Intraday-V1-EI4-Decision-Register.md#13-ei4-d11-enforcement-design-owner-approval-record-2026-11-10)). This ADR records the enforcement architecture those decisions establish. It records no separate Architecture Review or Engineering Review sign-off.

| Item | Status |
| --- | --- |
| EI4-D11 governance requirement (EI4-D11-A to EI4-D11-E) | OWNER APPROVED (2026-11-08) |
| EI4-D11 engineering enforcement design | OWNER APPROVED (2026-11-10): this ADR |
| EI4-D11 implementation | **NOT YET IMPLEMENTED** |
| EI4-D11 engineering acceptance | **NOT YET ESTABLISHED** |
| Invalid-record recovery | **NOT APPROVED** |

This is a design decision. **Nothing in it is implemented** when it is recorded, and nothing in it has been tested. An accepted ADR is not completed code.

> **Note, 2026-11-11 (EI4-D11 engineering acceptance).** The table above records the status when this ADR was accepted. The decision text below is unchanged.
>
> - **Implementation.** Slices W1, W2-I, W3 and W4-I are now implemented and merged. The Application port and the SQLite adapter expose exactly `load_disposition`, `load_attempt`, `write_generation` and `write_attempt`; the raw `record_generation` and `record_attempt` are removed from both (section 7).
> - **Acceptance.** Engineering acceptance is recorded with the verdict PASS WITH DOCUMENTED LIMITATIONS (synthetic engineering acceptance) in the [EI4-D11 Engineering Acceptance Record](../product/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Engineering-Acceptance-Record.md) and the [EI-4 Decision Register, section 14](../product/Equity-Intraday-V1-EI4-Decision-Register.md#14-ei4-d11-engineering-acceptance-record-2026-11-11): 28 scenarios VERIFIED and 5 VERIFIED WITH LIMITATION. CI is not independently verified.
> - **Unchanged.** The trust boundary of section 2 is unchanged: no protection is claimed against privileged code, direct SQL or file modification, source tampering or forged capability objects. No production composition root exists, and invalid-record recovery is not approved.

## Date

2026-11-10

## Decision Makers

- Product owner: approved EI4-D11 on 2026-11-08, and EI4D11-DQ1, DQ2, DQ3, DQ5, DQ8 and DQ10 with clarifications A and B on 2026-11-10.
- Northstar Architecture Team: formal review pending.

## Context

[ADR-021](ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md) made EI-4D generation records immutable and assigned verification to the Application:
- SQLite enforces structure, insert-only history, foreign keys and the insert-time CANDIDATE checks;
- fingerprint, source and replay verification are the Application's (section 10).

[ADR-022](ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md) added policy-version enforcement. Its recording gate sits **inside** the verified Application recorder, and its engineering acceptance is recorded in the [EI4-D10 Engineering Acceptance Record](../product/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md).

**The EI-4E AA-2 finding** ([EI-4E record, section 14.2](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#142-aa-2--verified-writer-boundary-and-recovery)):
- A direct `SQLiteEquityRecommendationGenerationStore.record_generation` call commits a forged disposition that is structurally valid but historically contradictory (`test_e20_contradictory_candidate_evidence_is_caught_by_the_application_verifier`).
- The verified recorder refuses the same record.
- Because `eirk1` does not cover assessment content, and rows are immutable, the forged row occupies the genuine key. The genuine record is then refused as `STORED_RECORD_INVALID`.

**The source evidence** behind the [design's write-path inventory](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md#4-confirmed-write-path-inventory), read at Application `cd2301b` and Infrastructure `97700b3`:
- **No production composition root** wires the generation recorder or store. The API and CLI wire futures and options only.
- **The raw adapter's write methods are public**, and they are the confirmed bypass surface.
- **Read-only consumers hold write capability.** `VerifyStoredEquityRecommendationGenerationUseCase` and the EI4-D10 audit take the full read/write port, although they only read.
- **The EI4-D10 gate is not reached** by a direct adapter call.

## Decision

### 1. Approved decisions

| ID | Decision (owner approved 2026-11-10) | Section |
| --- | --- | --- |
| EI4D11-DQ1 | The operational trust boundary | 2 |
| EI4D11-DQ2 | Layered enforcement (Option D) | 3 |
| EI4D11-DQ3 | Attempt-only writes are inside the boundary, under their own authorization | 5 |
| EI4D11-DQ5 | Legacy raw write methods are removed after a phased migration | 7 |
| EI4D11-DQ8 | Windows is the required acceptance platform | 10 |
| EI4D11-DQ10 | This ADR records the architecture | — |
| Clarification A | The five-PR order: the recorder migrates to authorized writes (W1) before Infrastructure refuses raw writes (W2-I) | 8 |
| Clarification B | Authorization binds to the exact verified write contents, not only `eirk1`; reuse only for the identical operation during bounded reconciliation | 4 |

EI4D11-DQ4 (authorization representation), DQ6 (test migration), DQ7 (PR sequence) and DQ9 (investigation surface) are engineering choices within these decisions. They need no separate approval, and they are constrained by clarifications A and B.

### 2. Operational trust boundary (DQ1)

**The boundary covers:**
- supported operational Application workflows;
- Application composition roots;
- public Application read and write interfaces;
- supported Infrastructure persistence interfaces;
- accidental or ordinary use of the raw persistence adapter;
- mandatory verification before every operational generation write.

**It does not claim protection against:**
- arbitrary privileged Python execution, including importing private names;
- direct SQL or modification of the SQLite file;
- malicious source-code changes;
- forgery by privileged code. A private issuer token or Python capability object is **not** cryptographically unforgeable;
- multi-host authorization;
- production trading security.

The architecture is appropriate to current personal-use synthetic research only. ADR-022's DQ9 file-level trust limitation is unchanged.

### 3. Layered enforcement (DQ2)

| Layer | Owner | Rule |
| --- | --- | --- |
| 1. Read/write port separation | Application | `EquityRecommendationGenerationRecordReader` (`load_disposition`, `load_attempt`) for read-only consumers. `EquityRecommendationGenerationRecordWriter` (`write_generation`, `write_attempt`) for the recorder only |
| 2. Verified recorder as the operational write boundary | Application | `RecordEquityRecommendationGenerationUseCase` is the only supported caller of the writer port |
| 3. Recorder-issued verified-write authorization | Application | Only the recorder issues `VerifiedEquityRecommendationGenerationWrite` and `VerifiedEquityRecommendationAttemptWrite`, after its verification succeeds |
| 4. Adapter validation before SQLite | Infrastructure, using an Application validation function | Refuse before opening a connection |
| 5. Architecture and dependency checks | CI, both repositories | Source scans: who issues, who writes, who wires the adapter |
| 6. Existing SQLite guarantees | Infrastructure | Insert-only triggers, constraints, foreign keys, `BEGIN IMMEDIATE`, conflict handling. Defence in depth, unchanged |

**The design does not rely solely on:**
- constructor naming;
- optional dependency injection;
- documentation conventions;
- a process-local flag;
- a public "verified" boolean;
- an independently constructible authorization accepted without issuer checks.

**What the layers guarantee.** Within the boundary of section 2, no supported path makes a generation disposition or attempt durable unless the verified recorder issued an authorization for exactly that content. Any other supported attempt is refused before SQLite is opened, or detected by CI scans.

**What they cannot guarantee.** They cannot stop privileged code, raw SQL or file edits. They cannot repair or release a key already occupied by an invalid record (section 9).

### 4. Exact write-content binding (clarification B)

**A generation authorization corresponds to one exact verified operation.** It holds, by containment, using existing canonical documents and identities only:

- the disposition, including its proposal when it has one;
- the logical attempt;
- the verifier's `EquityVerifiedRecommendationGeneration`: levels A to C, frozen-evidence replay, and source and snapshot provenance;
- the gate's verified `EquitySyntheticPolicySetVerification`.

**At issuance**, the authorization captures the covered identity of the operation from existing canonical values:
- generation key, disposition kind and evaluator version;
- `eira1` assessment fingerprint;
- `eirp1` proposal fingerprint or none;
- `eirat1` attempt identity;
- the set's `eirs1`.

No new fingerprint or identity scheme is introduced.

**Validation** (an Application-owned function, called by the adapter) re-derives that identity from the contained records, which re-verify their own documents on construction. It refuses when:
- any value differs;
- the verifier result does not hold exactly that disposition;
- the policy verification is not verified for that disposition's set;
- the attempt is not that disposition's attempt;
- the issuer contract is not met.

An authorization therefore **never** covers a different disposition, proposal or attempt, even under the same `eirk1`. A generation key, policy labels, a verification flag or a caller-supplied identifier are never sufficient.

**The writer API takes the authorization, not separate records.** If an implementation also passes records, the adapter must require them to equal the covered content exactly, and refuse otherwise.

**Reuse.** The recorder issues one authorization per `record()` operation. It may reuse that same authorization **only** for the identical operation during the existing bounded reconciliation (EI4-D9-G; at most `max_writes` writes).
- It never mints a second authorization during reconciliation.
- It never mints one to cover contradictory contents under the same generation key.

A different verified disposition under an occupied key is a separate operation. Its write yields `CONFLICT` under EI4-D9-D and never replaces the first writer. No one-time nonce is introduced, because a nonce would block the approved bounded retries.

### 5. Attempt-only authorization (DQ3)

The boundary covers complete generation recording **and** attempt-only recording.

**`VerifiedEquityRecommendationAttemptWrite`** binds to one exact attempt-only record: its `eirat1` identity and its document, with no disposition kind. The recorder issues it for the actual attempt being persisted.

**It does not:**
- require policy evidence an attempt cannot have (a failure before complete policy-bearing inputs existed);
- fabricate an assessment, a proposal or a successful disposition;
- change the attempt's identity or status formats.

An unverified caller cannot fabricate authoritative attempt evidence through a supported adapter path.

### 6. Pre-connection refusal

The adapter's `write_generation` and `write_attempt` call the Application validation function **first**: before `_connect`, before `BEGIN IMMEDIATE`, before any insert. They refuse, with the Application-defined `UnverifiedEquityRecommendationGenerationWriteError`:
- a missing authorization;
- the wrong type;
- an unissued or forged authorization;
- mismatched exact contents;
- an unauthorized attempt-only write.

The error is a programming error: no supported caller can produce it, so it never becomes a recording outcome. Infrastructure never imports the issuer token.

After validation, the existing transaction runs **unchanged**:
- its single COMMIT boundary;
- W2's cleanup, which never masks failures;
- first-writer immutability;
- the atomic disposition, proposal and attempt;
- EQUIVALENT and CONFLICT semantics;
- typed failures.

### 7. Legacy raw write methods (DQ5)

`record_generation(disposition, attempt)` and `record_attempt(attempt)` do not remain as permanently supported write paths.

1. **During migration** they exist only as documented compatibility mechanisms (section 8).
2. **Then** they are removed from the Application port (W3) and the Infrastructure adapter (W4-I).

Read-only historical access (`load_disposition`, `load_attempt`) remains. Test-only corruption injection (raw SQL on a disposable database) is kept separate from the production interfaces, labelled, and never presented as a supported path.

### 8. Phased migration (clarification A; DQ7)

| Slice | Repository | Content | Temporary interfaces | Enforcement after merge |
| --- | --- | --- | --- | --- |
| W1 | application | Reader and writer ports; both authorization types; issuance in the recorder; validation function; the error. **The recorder already calls `write_generation` and `write_attempt`.** `VerifyStored…` and the audit take the reader | The combined `EquityRecommendationGenerationStore` keeps its four abstract methods. The writer methods get **concrete defaults** that validate the authorization, then delegate through dynamic dispatch (`self.record_generation(…)`, `self.record_attempt(…)`) | **None in Infrastructure yet**: compatibility, not enforcement. W1 alone does not complete EI4-D11 |
| W2-I | infrastructure | The adapter overrides `write_*` with pre-connection validation. Legacy `record_*` refuse operational use. Tests are migrated, with test-only raw-SQL corruption isolated and labelled | Legacy methods remain only as refusing methods | Raw writes refused |
| W3 | application | Remove `record_*` from the port; make `write_*` abstract; architecture tests; read-only consumers on the reader only | — | Port carries no raw write |
| W4-I | infrastructure | Remove the refusing `record_*`; end-to-end acceptance; confirm every fixture and composition surface uses the supported contract | — | Complete |
| W5 | docs | EI4-D11 engineering acceptance record | — | — |

**Merge compatibility invariant.**
- Each slice is tested against the other repository's **current `develop`** before merge. Compiling is not evidence of compatibility.
- Multi-repository merges are not atomic, and required CI checks are never bypassed.
- A compatibility gap found in the actual source is reported for engineering review **before** implementation. Enforcement is never weakened to close it.

The constraints already identified are listed in the [design, section 22.4](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md#224-migration-compatibility-constraints-found-in-source).

### 9. Invalid committed evidence (EI4-D11-B to D)

Unchanged and fail-closed:
- `STORED_RECORD_INVALID` or `STORED_RECORD_INCOMPATIBLE`;
- `SOURCE_EVIDENCE_*` and `REPLAY_MISMATCH`;
- `EVALUATOR_VERSION_CONFLICT`;
- `CONTENT_CONFLICT`;
- `COMMIT_RECONCILIATION_UNRESOLVED`.

Original records and source evidence are preserved. Investigation uses only the existing read-only surfaces: typed outcomes, `VerifyStoredEquityRecommendationGenerationUseCase`, and the EI4-D10 audit.

**No** repair, recovery, replacement, deletion, migration or maintenance write path is introduced. An occupied key stays occupied until a separately approved recovery design exists (EI4-D11-D).

### 10. Acceptance scope (DQ8)

Windows is the required development and engineering acceptance platform for this milestone. Acceptance must cover:
- local SQLite;
- independent connections;
- Windows spawned processes;
- restart and durable persistence;
- failed authorization before any SQLite connection;
- the existing immutable-record guarantees.

POSIX separate-process acceptance and multi-host distributed authorization are **not** claimed. Both are explicit limitations.

The proposed acceptance matrix (33 scenarios, PROPOSED / NOT EXECUTED) is in the [design, section 17](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md#17-acceptance-matrix).

### 11. Historical compatibility

- Replay and read-only verification use the reader port only and need no authorization.
- Replay never reads the policy catalog (ADR-022, section 9).
- Authorizations are in-memory Application objects and are **never persisted**.
- No schema change, row migration, backfill or canonical-format change is made.
- Records written before enforcement remain readable and verifiable under their original rules.

## Relationship to Earlier ADRs

- **ADR-021.** Unchanged and not superseded.
  - It remains the EI-4D persistence authority: identities, atomic recording, conflict rules, bounded commit reconciliation, verification levels and storage placement.
  - This ADR governs **which path may write** to that store, not how the store records.
- **ADR-022.** Unchanged and not superseded.
  - ADR-022 validates policy content through the supported Application workflow, through its generation check and recording gate.
  - This ADR makes that verified recorder mandatory for operational writes, so the gate can no longer be bypassed through a supported adapter path.
- **EI4-D10 engineering acceptance.** It remains PASS WITH DOCUMENTED LIMITATIONS, with its historical evidence unchanged. This ADR addresses a distinct, remaining risk: its limitation L7.
- **ADR-017 and ADR-020.** Layering and dependency direction are unchanged (Core ← Application ← Infrastructure).

## Rationale

- **Runtime refusal at the adapter** is the only layer that stops a new caller wiring the raw adapter, rather than only detecting it in CI.
- **Least privilege** removes the write capability that read-only consumers hold today without using it.
- **Containment and existing identities** give exact content binding with no new identity scheme, and keep the bounded retries of EI4-D9-G working.
- **The staged five-PR order** keeps every PR green against the other repository's `develop`. It avoids the red window between EI4-D10's S3 and S4 merges.

## Alternatives Considered

The [design, section 7](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md#7-alternatives-evaluated) records the full comparison.

- **A. Composition-root rule and scans only.** Rejected: CI detection without runtime refusal.
- **B. Port separation only.** Rejected: least privilege without refusal.
- **C. Authorization only.** Rejected: leaves read-only consumers write-capable.
- **A database-level verification marker.** Rejected: SQLite cannot verify, the marker is forgeable, and it would need table rebuilds.
- **A single-use nonce.** Rejected: it breaks the approved bounded retries.

## Consequences

### Positive Consequences

- A forged or unverified generation record can no longer be committed through any supported path. The AA-2 occupied-key scenario can then arise only from privileged access.
- Read-only consumers can no longer write.
- No identity, document, schema or stored record changes.

### Negative Consequences

- Breaking port and adapter changes, staged over five PRs.
- 38 direct adapter call sites in Infrastructure tests must migrate.
- e20's mechanism changes. The adapter now refuses the forged record, and the fail-closed consequence is tested by planting it through labelled test-only raw SQL.
- Keys already occupied by invalid records stay occupied (no recovery).
- Privileged bypass and Python-level forgeability remain.
- POSIX and multi-host behaviour are untested.

## Scope

This ADR records an architecture. It does not:

- implement any slice (W1 to W5), contract, adapter change, test or fixture;
- approve or implement invalid-record recovery, or any overwrite, repair or deletion of immutable evidence;
- introduce a new `eirs1`, `eirk1`, `eirat1` or other identity scheme, or change historical canonical evidence;
- extend EI4-D11 to the futures, options or paper-trading stores;
- introduce production operational entry points that do not exist today, or a new database authority;
- authorize recommendation issuance, the lifecycle `RECORDED` state, broker integration, paper or live trading, positions, fills, real-market readiness, EI-5 or EI-6.

Every restriction of the [EI-4 Decision Register, section 7](../product/Equity-Intraday-V1-EI4-Decision-Register.md#7-production-readiness-restrictions) applies.

## Future Considerations

- Slices W1 to W5, and the EI4-D11 engineering acceptance record.
- A separately governed invalid-record recovery design (EI4-D11-D).
- Architecture scans in any repository that adds the first operational composition root.
- POSIX or multi-host acceptance, if ever required.

## Related Documents

- [EI4-D11 Mandatory Operational Generation-Write Boundary Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md)
- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md), sections 10.3 and 13
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md)
- [EI4-D10 Engineering Acceptance Record](../product/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md)
- [Equity Intraday V1 EI-4E Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md), section 14.2
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- [ADR-017: Repository Layering and Futures Recommendation Domain Boundary](ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md)
