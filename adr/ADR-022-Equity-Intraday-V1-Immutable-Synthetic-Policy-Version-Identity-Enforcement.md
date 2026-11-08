# ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement

## Status

Accepted

The product owner approved EI4-D10, Immutable Policy-Version Identity, on 2026-11-08 ([EI-4 Decision Register, section 10.2](../product/Equity-Intraday-V1-EI4-Decision-Register.md#102-ei4-d10----immutable-policy-version-identity-aa-1)). On the same date the owner approved all nine engineering design decisions of the [EI4-D10 enforcement design](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md), EI4-D10-DQ1 to EI4-D10-DQ9, as recommended, including the DQ6 conflict-handling clarification and the DQ9 trust limitation ([EI-4 Decision Register, section 11](../product/Equity-Intraday-V1-EI4-Decision-Register.md#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08)). This ADR records the enforcement architecture those decisions establish. It records no separate Architecture Review or Engineering Review sign-off.

| Item | Status |
| --- | --- |
| EI4-D10 governance policy | APPROVED (2026-11-08) |
| EI4-D10 technical enforcement design | APPROVED (2026-11-08): this ADR |
| EI4-D10 engineering implementation | **NOT YET IMPLEMENTED.** Implementation pending |
| EI4-D10 implementation acceptance | **NOT YET ESTABLISHED.** Independent acceptance pending |

This is a design decision. **Nothing in it is implemented** when it is recorded, and nothing in it has been tested. An accepted ADR is not completed code.

## Date

2026-11-08

## Decision Makers

- Product owner: approved EI4-D10 and EI4-D10-DQ1 to EI4-D10-DQ9 on 2026-11-08.
- Northstar Architecture Team: formal review pending.

## Context

[ADR-021](ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md) made the generation key (`eirk1`) bind a composite synthetic policy-set identity, whose `eirs1` fingerprint covers the set label, the five component labels and their full canonical content.

The [EI-4E Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#141-aa-1--policy-version-label-governance) then found (AA-1) that **a published version label can name different content over time**:

- The EI-4C generator asks an injected `EquitySyntheticRecommendationReferenceSource` for each label. It accepts any answer that names the label asked for, and only a wrong-label answer is refused (`POLICY_REFERENCE_MISMATCH`).
- Changed content changes `eirs1`, and therefore `eirk1`. A reused label yields a new key and a second disposition for the same candidate, instead of a conflict.
- EI-4E's `test_e12_c_a_reused_version_label_with_changed_content_is_another_identity` demonstrated this on real SQLite.

What exists when this ADR is recorded, as verified in the design (core `bc081a2`, application `bed91de`, infrastructure `5dda518`):

- No production adapter implements the synthetic reference ports. Only in-memory test fixtures do, and no policy registry exists.
- Core canonicalizes Decimals, and validates `EquitySyntheticVersion` as case-sensitive, never trimmed and `synthetic-`-prefixed. Its own docstring leaves "a version reused with different content" to "later Application persistence".
- Only committed dispositions persist policy content: the set label and `eirs1` as columns, and the full policy-set document inside the assessment JSON.
- The attempts table constrains `status` to six values with a CHECK constraint. Reasons are stored only inside JSON.

## Decision

### 1. Approved decisions

| ID | Decision (owner approved 2026-11-08, as recommended) | Section |
| --- | --- | --- |
| EI4-D10-DQ1 | One closed namespace, `northstar.equity-intraday-v1.synthetic` | 2 |
| EI4-D10-DQ2 | Six policy types: five components plus `POLICY_SET` | 2 |
| EI4-D10-DQ3 | One immutable price-increment amount per published label | 3 |
| EI4-D10-DQ4 | The catalog lives in the existing single-file SQLite authority, behind an Application port and an Infrastructure adapter | 4 |
| EI4-D10-DQ5 | Layered enforcement: publication, catalog registration, catalog-backed resolution, generation validation, recording gate, catalog-free replay | 7 |
| EI4-D10-DQ6 | `DEPENDENCY_UNAVAILABLE` status with the distinct reason `POLICY_REFERENCE_CONFLICT`, a permanent integrity conflict that is never retried as transient | 8 |
| EI4-D10-DQ7 | No automatic historical backfill; read-only historical binding audit | 9 |
| EI4-D10-DQ8 | The enforcement architecture is recorded in this ADR | — |
| EI4-D10-DQ9 | Publication relies on the existing SQLite file trust boundary, as a deliberate, limited assumption | 10 |

### 2. Namespace and policy types (DQ1, DQ2)

- **Namespace.** Exactly one closed namespace, `northstar.equity-intraday-v1.synthetic`, for the current synthetic equity intraday recommendation policy catalog.
  - It is an Application constant, never caller-supplied text, and it is part of every immutable binding identity.
  - It does **not** extend to Futures, Options, live equity execution, commodities or any other Northstar vertical.
- **Policy types.** Exactly six; the list is not expanded silently.

  | Policy type | Definition |
  | --- | --- |
  | `GENERATION_POLICY` | `EquitySyntheticGenerationPolicy` |
  | `RISK_POLICY` | `EquitySyntheticRiskPolicy` |
  | `COST_SCHEDULE` | `EquitySyntheticCostSchedule` |
  | `PRICE_INCREMENT` | `EquityPriceIncrement` |
  | `TIMING_POLICY` | `EquitySyntheticEntryTimingPolicy` |
  | `POLICY_SET` | The composite set label and its five component labels |

- **Uniqueness** is scoped to `(namespace, policy_type, published_version_label)`. It is never global across policy types: the same label under two types is two independent bindings. The five components and the composite set have separate identities.
- **Labels** keep `EquitySyntheticVersion`'s existing rules: case-sensitive, never trimmed, refused with surrounding whitespace, `synthetic-`-prefixed.
- **Out of scope.** Listing eligibility (per-listing evidence; content under EI4-D9-B) and the EI-3 research configuration (governed by EI-3C's configuration-conflict rule) stay with their existing mechanisms.

### 3. Canonical binding identities (DQ3)

| Identity | Format | Fingerprint | Status |
| --- | --- | --- | --- |
| Component binding | New: `northstar.equity-synthetic-policy-definition/1`, a document of namespace, policy type, label and the component's existing canonical `document()` | New: `eipd1-sha256:` | Approved design; **not implemented** |
| Composite set binding | The existing `EquitySyntheticPolicySetReference.document()` | Existing `eirs1` | Unchanged algorithm |
| Generation identity | Existing | Existing `eirk1` | Unchanged algorithm |

- The new component format uses the existing canonical encoding (`canonical_equity_recommendation_json`) and fingerprinting (`equity_recommendation_fingerprint`). It adds one prefix entry; no new serialization scheme.
- A set binding is verified by rebuilding `EquityResolvedSyntheticPolicySet` from the bound components and re-deriving `eirs1`.
- No binding document is embedded in frozen inputs, assessments, proposals or attempts. `eirs1`, `eirk1`, `eiri1`, `eira1`, `eirp1`, `eirat1`, every existing canonical document version and `EQUITY_RECOMMENDATION_EVALUATOR_VERSION` are unchanged.
- **Price increment (DQ3).**
  - One published `PRICE_INCREMENT` label binds exactly one immutable canonical amount. A changed amount needs a new label, and a label never acquires several amounts.
  - This keeps one `eirs1` per set label, because the increment document is inside the policy-set document.
  - No production NSE tick size is introduced or inferred from synthetic amounts. A richer increment definition, such as a per-listing table, needs separately governed versioning.

### 4. Catalog placement and ownership (DQ4)

- The catalog is durable, immutable and lives in the **same SQLite database file** as the EI-2B evidence, the EI-3C timelines and the EI-4D dispositions and attempts: the existing single-file authority.
- **Not approved:** a separate policy database file, any second authority, process-local memory as authoritative storage, and direct SQL from the Application.
- **Application** owns:
  - the catalog port and typed binding errors;
  - the publication use case and the catalog-backed reference resolver;
  - generation-time integrity validation and the recorder pre-write gate;
  - the read-only historical binding audit.
- **Infrastructure** owns:
  - the SQLite catalog schema and the binding adapter;
  - transactions, unique constraints, foreign keys and insert-only protections;
  - durable lookup, concurrency, and commit-failure classification.

  It verifies no SHA-256, as with EI-4D-B (ADR-021 section 10).
- **Core** keeps its policy value objects, canonical calculations and formation behaviour, unchanged.
- **Dependency direction** stays Core ← Application ← Infrastructure (ADR-017, ADR-020 section 8). No Infrastructure dependency enters the Application or Core.
- **Not an issuance authority.** The catalog is a policy authority only. It never becomes an issued-recommendation authority or an alternative issuance path (EI4-D9-H, EI4-D9-J).

### 5. Publication lifecycle

1. A definition is supplied to an **explicit** publication use case.
2. It is validated by its Core or Application type.
3. Its canonical binding document and fingerprint are derived.
4. Its binding is established, or loaded if it already exists.
5. An identical existing binding is **equivalent**: idempotent, nothing written.
6. A different existing binding under the same identity is a **conflict**: rejected, nothing written.
7. Published bindings become available to generation through the catalog-backed resolver.
8. Frozen generation inputs keep the policy content as historical evidence, exactly as today.

Publication is the only way a binding is created. Policy reads, generation, recording, verification and historical replay never publish implicitly. An unverified reference answer never defines historical truth.

### 6. Atomicity, concurrency and commit uncertainty

- One policy bundle (components, then an optional set) is published in **one `BEGIN IMMEDIATE` transaction**, following the EI-4D-B pattern. It is all or nothing, so a set can never exist without its components.
- **Database rules:**
  - unique binding identities as primary keys;
  - real foreign keys from a set to its components;
  - insert-only triggers that refuse UPDATE, DELETE and replacing inserts;
  - a monotone publication sequence, with no clock.
- **Cross-connection and cross-process correctness** comes from SQLite's lock and constraints, never from a process-local lock.
- **The adapter** reports typed outcomes and errors: committed, equivalent, conflict, unavailable, not committed, commit unconfirmed, invalid, incompatible. It never retries, sleeps or reads a clock: there are no hidden retries.
- **Commit uncertainty.** A COMMIT failure is never reported as a rollback. The Application reconciles by reloading the bundle once:

  | Reload finds | Result |
  | --- | --- |
  | All equal | Equivalent, reconciled |
  | Any different | Conflict |
  | None, after a successful reload | Bounded retry with the same bundle, at most three writes |
  | Mixed, or the reload is unavailable or invalid | Unresolved; nothing claimed |

### 7. Layered enforcement (DQ5)

| Layer | Owner | Rule |
| --- | --- | --- |
| A. Explicit publication | Application use case | Establishes a label-to-definition binding. Reads and replay never publish |
| B. Atomic catalog registration | Infrastructure adapter, Application reconciliation | Identical bindings are idempotent; conflicting reuse is rejected |
| C. Catalog-backed resolution | Application resolver | Generation references resolve only to published, fingerprint-verified definitions |
| D. Generation validation | EI-4C, while gathering inputs | Validates the resolved component and set bindings. A policy-reference failure is never a market-eligibility rejection |
| E. Recording gate | EI-4D-A verified recorder, before writing | Checks the disposition's frozen policy content and set reference against the catalog. A record that bypassed ordinary input gathering does not bypass this check by reaching the recorder |
| F. Historical replay | Unchanged | Replays from frozen inputs and never depends on catalog availability. A separate read-only audit checks publication evidence |

`assess_equity_recommendation_inputs`, policy-set construction and the four verification levels of ADR-021 section 10 are unchanged.

**DQ5 alone does not stop direct SQLite adapter writes.** Making the verified recorder the mandatory operational write path is EI4-D11's separate enforcement (section 11).

### 8. Failure classification (DQ6)

The top-level generation outcome for policy-reference failures stays the existing **`DEPENDENCY_UNAVAILABLE`**. No new persisted attempt status is created, and the immutable attempts table and its approved status semantics are not modified.

| Case | Classification | Handling |
| --- | --- | --- |
| A. Temporary catalog outage | `DEPENDENCY_UNAVAILABLE`, with the existing `*_UNAVAILABLE` reason | Fails closed. It may be retried only under a separately approved operational retry policy; none exists today. It never creates a binding |
| B. A published label bound to different content | `DEPENDENCY_UNAVAILABLE` at the compatible top level, with the distinct machine-readable reason **`POLICY_REFERENCE_CONFLICT`** | A **permanent integrity conflict** for that policy identity, never transient downtime. Fails closed. No automatic retry, and no silent republication with changed content |
| C. Unknown or unpublished label | `DEPENDENCY_UNAVAILABLE`, with the existing "not held" `*_UNAVAILABLE` reason, or `POLICY_SET_UNAVAILABLE` for a set | Fails closed. No default policy is invented, and absence is never publication evidence |
| D. Corrupt or unsupported stored binding | Typed invalid versus incompatible binding errors; `POLICY_REFERENCE_CONFLICT` at generation | Fails closed. Evidence is preserved and nothing is migrated |
| E. Same key, different generation content | Existing `CONTENT_CONFLICT` | Unchanged; distinct from a policy-reference conflict |
| F. Eligibility rejection | Existing `NOT_ELIGIBLE` reasons | Unchanged; distinct from a policy-reference conflict |

At the recording gate, the proposed outcomes `POLICY_REFERENCE_CONFLICT`, `POLICY_REFERENCE_NOT_ESTABLISHED` and `POLICY_REFERENCE_UNAVAILABLE` write nothing.

**The machine-readable reason must propagate** through every relevant contract:

- the EI-4C reason list and attempt document;
- the recording result;
- the publication result;
- the historical audit result.

Consumers must distinguish a permanent conflict from a transient outage by that reason, not by the top-level status. The future Application and Infrastructure slices must validate this.

### 9. Historical compatibility (DQ7)

- **No automatic historical backfill.**
  - No publication history is fabricated.
  - No existing policy evidence, EI-4D disposition, EI-4D attempt, `eirs1` fingerprint, `eirk1` key or canonical document is rewritten, re-keyed, migrated or deleted.
- **Read-only audit.** A historical binding audit classifies stored records against the catalog, beside ADR-021's verification levels A to C and never folded into them:
  1. historically verified binding;
  2. binding not established;
  3. historical policy-version conflict;
  4. invalid canonical evidence;
  5. incompatible historical format, where applicable.
- **No retrospective compliance.** A record created before enforcement is not described as publication-compliant at its creation time unless evidence establishes that. Dispositions carry no publication sequence, so for those records it cannot be established.
- **Replay independence.** Historical replay stays reproducible from frozen inputs when the catalog is unavailable.
- No automatic migration and no retrospective approval is authorized.

### 10. Publication trust boundary (DQ9)

- **Accepted, deliberately and narrowly:** for current personal-use synthetic research, publication relies on the existing SQLite database-file trust boundary.
- **Supported writes are protected.** SQLite constraints and triggers protect ordinary supported writes.
- **Privileged access can bypass it.** Local file or database access with sufficient privilege can bypass application-level publication authority. For example, it can insert a new binding directly, or drop triggers, as EI-4E's test-only `bypass()` shows.
- **This boundary does not establish:**
  - tamper-proof publication;
  - protection against an operator with database-file write access;
  - cryptographic proof of who published a definition;
  - multi-host distributed authorization;
  - a production identity or permission system;
  - protection against every direct database-write bypass.
- No authentication or access-control system is introduced.
- The assumption is **not** acceptable for real trading without separate review.
- It does not weaken EI4-D11's mandatory verified recording requirement.

### 11. Relationship to EI4-D11

- EI4-D10 policy-catalog enforcement and EI4-D11 verified-recorder boundary enforcement are distinct. Both governance policies are approved, and implementing EI4-D10 does not complete EI4-D11.
- Until EI4-D11's operational boundary is separately implemented and accepted, the recording gate of section 7 can be bypassed by direct use of the Infrastructure adapter.
- Policy-publication authority is not recording authority.
- No invalid-record recovery design is approved (EI4-D11-D). Nothing here introduces deletion, overwrite or mutation of immutable evidence.

### 12. Implementation sequence and acceptance requirements

| Slice | Repository | Scope | Excludes |
| --- | --- | --- | --- |
| S1. Application contracts | northstar-application | Immutable binding contracts, catalog port, publication use case with reconciliation, catalog-backed resolver, historical binding audit, typed error vocabulary, deterministic contract tests | Any SQLite implementation |
| S2. Infrastructure catalog | northstar-infrastructure | Same-file SQLite catalog, additive immutable schema, atomic publication, uniqueness and concurrency, commit uncertainty, restart verification, integration tests | Generation-gate changes |
| S3. Application enforcement | northstar-application | Catalog-backed generation validation, recorder pre-write gate, fail-closed outcomes, replay independence | Coordinated with S4; changes generator and recorder construction |
| S4. Infrastructure end-to-end acceptance | northstar-infrastructure | Explicit synthetic policy publication in fixtures, updated EI-4E integration, conflicting-label rejection, restart and concurrency acceptance, revalidation of earlier synthetic scenarios | Coordinated with S3 |
| S5. Documentation acceptance | northstar-docs | Enforcement acceptance evidence, test references, remaining limitations, engineering verdict | Any retroactive change to EI-4E evidence |

S3 and S4 must be coordinated, and neither is represented as independently accepted full enforcement before integration. No slice is implemented, and none is started by this ADR.

**Acceptance requirements.** The design's acceptance matrix ([section 13](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md#13-acceptance-matrix)) holds **33 scenarios**: the original 29, unchanged, plus scenarios 30 to 33 ([section 13.1](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md#131-scenarios-30-to-33)), which give every owner-required test an explicit row. The owner-required tests map to it as follows:

| Required test | Matrix rows |
| --- | --- |
| Initial publication | 1 |
| Idempotent identical publication | 2 |
| Conflicting reuse of one label | 3, 11 |
| Namespace separation | 4 |
| Policy-type separation | 5 |
| Price-increment label immutability | 3, instantiated for `PRICE_INCREMENT` |
| Invalid and unsupported canonical definitions | 8, 12 |
| Concurrent identical and conflicting writers; independent connections | 15 |
| Separate Windows processes | 16 |
| Restart durability | 14 |
| COMMIT before and after a lost acknowledgment; bounded reconciliation | 17, 18, 19 |
| Corrupt catalog entries | 13 |
| Historical binding audit; no invented backfill | 22 |
| Historical replay with the catalog unavailable | 21 |
| Existing EI-4E compatibility | 23 |
| Preservation of `eirs1` and `eirk1` | 24 |
| No modification of existing dispositions | 25 |
| No actual issuance or broker activity | 26 |
| Catalog unavailability during EI-4C generation | 30 |
| Catalog unavailability at the EI-4D-A recording gate | 31 |
| Distinct permanent-conflict classification: `POLICY_REFERENCE_CONFLICT` versus a temporary outage, by machine-readable value | 32 |
| No automatic retry or republication of a permanent integrity conflict | 33 |

Scenarios 30, 31 and 33 test the EI4-D10 recording gate inside the EI-4D-A recorder. They do not test EI4-D11's separate requirement that every operational write pass through that recorder (section 11).

Every acceptance test, including scenarios 30 to 33, is **PROPOSED / NOT EXECUTED**. No engineering acceptance has passed.

## Relationship to Earlier ADRs

- **ADR-017 and ADR-020 section 8.** Layering and dependency direction are unchanged.
- **ADR-020.** Section 2 already adopts "named, versioned generation policies, where any parameter change is a new version". This ADR makes conflicting label reuse fail closed. ADR-020 is not superseded or edited.
- **ADR-021.** Unchanged and not superseded:
  - its identity scheme, canonical formats, atomic recording, conflict rules, commit-unconfirmed recovery and four verification levels stand;
  - the catalog shares the single SQLite authority of its section 11;
  - the recording gate adds a pre-write check before its section 6 transaction, without changing that transaction;
  - no contradiction was found, and ADR-021 is not edited.

## Rationale

- **An explicit, durable catalog** is the only option that holds across processes and restarts without letting the first read or the first generation define truth.
- **Same-file placement** keeps one SQLite authority and one backup, and lets the recording gate read the policies and the records from the same file.
- **Layered enforcement** gives resolution only published content. The recording gate is the wiring-independent guarantee, which matters because there is no composition root; replay stays catalog-free.
- **A reason, not a status,** carries the permanent conflict. It needs no rebuild of the insert-only attempts table, and follows the existing `POLICY_REFERENCE_MISMATCH` precedent of reference-answer failures under `DEPENDENCY_UNAVAILABLE`. Calling it permanent prevents it from being retried like an outage.
- **No backfill** avoids inventing history, and keeps every stored fingerprint and record unchanged.

## Alternatives Considered

The [design, section 6](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md#6-alternatives) records the full comparison.

- **Application-only in-memory binding.** Rejected: not durable, not cross-process.
- **A SQLite registry used directly, without an Application port.** Rejected: wrong dependency direction.
- **A separate catalog database file.** Rejected by DQ4: a second authority to keep consistent and back up.
- **Embedding binding evidence in frozen inputs.** Rejected by DQ7: it would change the `eiri1` and `eira1` document versions.
- **Implicit binding by the first committed disposition.** Rejected: the first read or generation would define truth.
- **A new attempt status for policy conflicts.** Rejected by DQ6: it requires rebuilding the insert-only attempts table.
- **Placing `POLICY_REFERENCE_CONFLICT` under `RESEARCH_INTEGRITY_FAILURE`.** Rejected by DQ6: it misnames the failure as research.
- **Per-listing price-increment bindings.** Rejected by DQ3 for now: one set label would map to several `eirs1` values.
- **Technical publisher authentication.** Not adopted by DQ9: there is no identity layer in a personal-use system. This is recorded as a limitation, not a protection.

## Consequences

### Positive Consequences

- A published label means one definition for good, and reuse with other content fails closed at publication, at generation and at recording.
- `eirs1`, `eirk1`, every canonical document, the evaluator version and every stored record are unchanged.
- Historical replay keeps its independence from any live source.
- Permanent policy conflicts are machine-distinguishable from outages.

### Negative Consequences

- Recording depends on catalog availability, and a catalog outage blocks new dispositions (fail closed).
- The generator and recorder constructors change, so S3 and S4 must be coordinated, and the expected outcome of EI-4E `test_e12_c_…` changes.
- For records before enforcement, "published before generated" cannot be proved.
- A price-increment label is one amount for every listing and date.
- The file-level trust limitation of section 10 remains.
- The recording gate is a complete guarantee only after EI4-D11 is enforced.

## Scope

This ADR records an architecture. It does not:

- implement any contract, schema, DDL, adapter, use case, fixture or test;
- change `eirs1`, `eirk1`, any canonical document version or the evaluator version;
- mutate, migrate, re-key, delete or backfill any historical evidence or record;
- approve or implement any invalid-record recovery (EI4-D11-D), or implement EI4-D11;
- authorize direct-adapter operational recording, actual recommendation issuance, the lifecycle `RECORDED` state, broker or paper orders, fills, positions, realized P&L, live data integrations, data-use licensing claims or production trading;
- approve any production policy value or default. Every policy is synthetic, and EI0-D10 to EI0-D13 stay pending;
- approve EI4-D5, EI0-D11, the same-listing exclusivity threshold, EI0-D19, EI0-D21 or EI3-D10;
- implement EI-5 or EI-6.

## Future Considerations

- Slices S1 to S5, and the enforcement acceptance record.
- EI4-D11's separate enforcement of the verified-recorder boundary, and any separately approved invalid-record recovery design.
- A further policy namespace, for example for approved real policies after EI0-D10 to EI0-D13, which needs a new decision.
- A richer price-increment definition, which needs separately governed versioning.
- Publisher authentication, if the system ever leaves personal-use synthetic research.

## Related Documents

- [EI4-D10 Policy-Version Identity Enforcement Design](../architecture/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Enforcement-Design.md)
- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md), sections 10 and 11
- [EI-4E Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- [ADR-017: Repository Layering and Futures Recommendation Domain Boundary](ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md)
- Engineering Standards ES-001, ES-003, ES-004, ES-007
