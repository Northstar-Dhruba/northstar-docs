# ADR-021: Equity Intraday V1 Recommendation Generation Persistence

## Status

Accepted

The product owner approved EI4-D9 -- Recommendation Generation Persistence Identity, Sequencing and Historical Verification -- on 2026-11-07, as ten subdecisions, EI4-D9-A to EI4-D9-J, recorded in the [EI-4 Decision Register, section 9](../product/Equity-Intraday-V1-EI4-Decision-Register.md#9-ei4-d9-owner-approval-record-2026-11-07). This ADR records the persistence architecture those subdecisions require. It records no separate Architecture Review or Engineering Review sign-off.

This is a design decision. **Nothing in it is implemented** when it is recorded. It needs an EI-4C Application compatibility slice first ([section 12](#12-implementation-sequence)).

> **Note, 2026-11-12 (owner architecture sign-off, EI4-C5).** The paragraphs above record the status when this ADR was accepted, on 2026-11-07, and are not rewritten. On 2026-11-12 the product owner approved ADR-021's architecture conformance and its formal sign-off ([EI-4 Decision Register, section 15](../product/Equity-Intraday-V1-EI4-Decision-Register.md#15-ei-4-closure-dispositions-2026-11-12)):
>
> - **Accepted and implemented.** ADR-021 is accepted and implemented within the documented EI-4 scope, through the slices of [section 12](#12-implementation-sequence):
>   - EI-4C compatibility;
>   - EI-4D-A;
>   - EI-4D-B;
>   - EI-4E synthetic acceptance.
>
>   The evidence is in the [EI-4E Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md). The statement above that nothing is implemented is superseded by this note.
> - **Later hardening.** [ADR-022](ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md) and [ADR-023](ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md) subsequently harden its authorization and write boundaries:
>   - ADR-022 adds the policy-binding gate inside the section 6 recording path;
>   - ADR-023 makes the verified recorder the only supported operational writer, and splits the store port of section 11 into a read port and a write port.
> - **Unchanged decisions.** Every architectural decision in sections 1 to 12 is unchanged.
> - **Limitations.** Synthetic engineering acceptance has documented limitations:
>   - Windows-only process evidence;
>   - no replay from a separate OS process;
>   - no natural end-to-end sizing-search-limit scenario;
>   - independent CI not verified.
>
>   EI-4's formal milestone closure is pending independent CI verification.
> - **No production authorization.** This sign-off does not imply production authorization, real data, actual issuance or trading.

> **Note, 2026-11-13 (amended by ADR-025).** On 2026-11-13 the product owner approved R-1 and R-2 of the C-2 correction and accepted [ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md). ADR-025 amends the version-verification restriction of [section 10](#10-restart-and-historical-verification-ei4-d9-i):
>
> - **What changes.** Versioned verification is now governed by ADR-025: a **closed** set of exactly two evaluator versions, `northstar.equity-recommendation-evaluator/1` and `/2`. Each record is verified only by the evaluator it names.
> - **No registry.** No extensible evaluator registry is authorized.
> - **Unchanged.**
>   - Historical version-1 replay remains required.
>   - An unsupported version still fails closed.
>   - Nothing is migrated, reinterpreted or relinked.
>   - Every other decision of this ADR is unchanged.
> - **Original text kept.** The text of section 10 is kept as recorded on 2026-11-07, and so is its historical meaning.
> - **Not implemented.** Evaluator version 2 does not exist in code.

## Date

2026-11-07

## Decision Makers

- Product owner: approved EI4-D9-A to EI4-D9-J on 2026-11-07.
- Northstar Architecture Team: formal review pending.

> **Note, 2026-11-12.** The product owner approved this ADR's architecture conformance and formal sign-off on 2026-11-12 (EI4-C5; see the note under Status). The line above records the status before that sign-off.

## Context

[ADR-020](ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md) set the equity recommendation architecture and left recommendation persistence sequencing to EI4-D9. It requires that "business uniqueness is enforced by the immutable store (EI-4D)", that recommendation definitions and eligibility decisions are immutable once recorded, and that every candidate has one auditable disposition per policy version.

What exists when this ADR is recorded:

- **EI-2B and EI-3C** keep immutable evidence snapshots and authoritative research timelines in one SQLite file. Timeline entries reference snapshots by real foreign keys. Writes run in one `BEGIN IMMEDIATE` transaction, triggers refuse updates, deletes and replacing inserts, and a failure from COMMIT itself is reported as unconfirmed and reconciled by an authoritative reload (EI3-D11 to EI3-D13, the [EI-3C Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI3C-Synthetic-Acceptance-Record.md)).
- **EI-4B** (northstar-core) forms a structural `EquityRecommendationDefinition` or an `EquityRecommendationRejection` from versioned synthetic inputs. It generates no identity.
- **EI-4C** (northstar-application) evaluates the first authoritative `CANDIDATE` and returns an `EquityRecommendationEligibilityAudit` whose status is `PROPOSED`, `NOT_ELIGIBLE`, `EVALUATION_UNRESOLVED`, `NO_CANDIDATE`, `DEPENDENCY_UNAVAILABLE` or `RESEARCH_INTEGRITY_FAILURE`. Nothing is persisted, and:
  - its provisional proposal key, under the namespace `northstar.equity-trade-recommendation-proposal.v1`, binds the candidate and the **generation-policy version only**, so equal keys can carry different risk, cost, increment, timing or eligibility inputs;
  - its documents carry no content fingerprint, and check details include free text such as port exception messages;
  - `modeled_issued_at` is a caller-supplied input;
  - its recorded inputs name the venue-day reference but do not keep the venue-day answer (classification, UTC offset, common boundaries) that timing used;
  - its point-in-time history contract states `complete_through` but no recording watermark.
- **Futures F-5** records recommendation definitions and their generation audit in one transaction, treats an exactly equal replay as idempotent and refuses a different one. It is a pattern here, never a dependency (ADR-017, ADR-020).
- **Milestone wording disagreed.** The [product specification, section 12.1](../product/Equity-Intraday-V1-Product-Specification.md#121-refined-sequence) gives EI-6 "durable insert-only persistence for recommendations, rejections, lifecycle events...", while the EI-4 Decision Register gives EI-4D "immutable recommendation and eligibility-audit persistence". The specification's section 8.2 makes `RECORDED` ("the recommendation is persisted") the first lifecycle state.

## Decision

### 1. Generation records (EI4-D9-A, EI4-D9-J)

EI-4D persists **generation records**, never issued recommendations or lifecycle records:

- **Generation disposition.** At most one per generation identity (section 2), immutable, of exactly one kind:
  - `PROPOSED`, carrying the synthetic proposal document;
  - `NOT_ELIGIBLE`, a confirmed refusal with its ordered reasons and any exact Core reasons;
  - `UNRESOLVED`, a result that could not be established -- an unapproved exclusivity or exposure policy, or Core's exact sizing search reaching its bound -- with its reasons.

  A `NOT_ELIGIBLE` or `UNRESOLVED` disposition never carries a proposal.
- **Logical generation attempt.** An append-only log of distinct, deterministic logical attempts. An attempt that produced, matched or conflicted with a disposition names it. Integrity failures, dependency outages and attempts that found no candidate appear only here: they never acquire a disposition, because no verified candidate and policy key exists for them. Identical invocations may deduplicate to one attempt.

EI-4C outcomes map as follows: `PROPOSED` to a `PROPOSED` disposition; `NOT_ELIGIBLE` to `NOT_ELIGIBLE`; `EVALUATION_UNRESOLVED` to `UNRESOLVED`; `NO_CANDIDATE`, `DEPENDENCY_UNAVAILABLE` and `RESEARCH_INTEGRITY_FAILURE` to attempts only.

Dispositions are not lifecycle states. A stored `PROPOSED` disposition is not `RECORDED`, not issued and not actionable.

### 2. Generation identity (EI4-D9-B)

- A composite, versioned **synthetic policy-set identity** names one generation policy, one risk policy with its model-capital configuration, one cost schedule, one price-increment reference and one timing policy (including the synthetic issuance latency of section 4).
- The **generation identity** is derived by the Application, never by Core, from the policy-set identity and the authoritative first `CANDIDATE` with its strategy, research configuration, listing and trading date. The candidate's `eirt1` entry fingerprint binds the timeline's full configuration, the research decision and its evidence snapshot.
- This refines EI4-D2: a candidate yields zero or one recommendation per policy-set identity. A materially different approved policy set may have a different identity, which permits a distinct disposition; it never authorizes several issued recommendations or any order.
- Listing-eligibility evidence, issued-history evidence and venue-day evidence are **content**, not identity. A change to them under one identity is a conflict (section 5), never a new disposition.
- The identity uses a new, explicitly versioned scheme, in the length-prefixed SHA-256 style of the existing keys, introduced by the EI-4C compatibility slice. The `northstar.equity-trade-recommendation-proposal.v1` namespace keeps its current meaning and is never reinterpreted.

### 3. Canonical content and fingerprints (EI4-D9-C)

- The assessment and the proposal are stored as **format-versioned canonical structured documents**: canonical JSON with sorted keys and no whitespace, exact decimal text, canonical UTC instants and no floating-point numbers.
- A **content fingerprint**, SHA-256 over the canonical structured assessment, binds everything the result depends on:
  - the candidate references: `eir1` decision, `eirt1` entry and `ecs1` snapshot, with the manifest fingerprint;
  - every policy document in the policy set;
  - the listing-eligibility document;
  - the venue-day evidence used for timing;
  - the point-in-time issued-history evidence, with its completeness and recording watermark;
  - the modeled instants;
  - the ordered checks and reasons, any Core reasons, and the Core definition.
- Free-text diagnostics, exception messages and descriptions are excluded from the fingerprint. They may be kept beside the document for operators, and never affect identity, equality or replay.
- Equal identities never prove equal content; equality is decided by the content fingerprint, after verifying the document it covers.

### 4. Modeled issuance (EI4-D9-C)

- In synthetic historical modeling, modeled issuance is **derived**: the verified source decision availability -- the research decision instant -- plus an explicitly versioned synthetic latency from the timing policy. It is never a caller-supplied timestamp.
- `decision_as_of`, modeled issuance, actual operational issuance and the earliest eligible entry stay distinct (EI4-D1). Entry eligibility still begins at the first grid bar that opens at or after the modeled issuance.
- Actual operational issuance is outside EI-4D. Nothing is backdated, and nothing looks ahead.
- No numeric latency and no production cutoff are approved.

### 5. Duplicates and conflicts (EI4-D9-D)

For one generation identity:

| Situation | Outcome |
|---|---|
| Stored disposition with the same verified fingerprint | Idempotent: the stored record is returned and nothing new is written, except a distinct logical attempt |
| Stored disposition with a different fingerprint | Explicit conflict. The stored record stays authoritative; the conflicting attempt is logged when storage is available |
| No stored disposition | The disposition, proposal and attempt are recorded together (section 6) |

There is no last-write-wins, no replacement and no relinking. Commit order establishes what was recorded first; it does not show that the first evidence was objectively correct. The rule is the same for every kind: an `UNRESOLVED` disposition is never upgraded to `PROPOSED` under the same identity, and a different disposition needs a different approved policy identity.

### 6. Atomic recording (EI4-D9-E)

One SQLite `BEGIN IMMEDIATE` transaction, which takes the write lock before reading:

1. reads the disposition stored under the generation identity, if any;
2. applies section 5: an equal fingerprint records only a new distinct attempt; a different fingerprint records only the conflicting attempt;
3. otherwise inserts the disposition, the proposal document when the kind is `PROPOSED`, and the attempt;
4. commits once.

Any failure before COMMIT rolls everything back: no partially authoritative generation result exists, and nothing is reported as recorded. When storage itself fails, the failure is reported and the attempt is never claimed as audited.

### 7. Concurrent writers (EI4-D9-F)

- SQLite serializes writers. Uniqueness of the generation identity and of the attempt identity is enforced by database constraints, never by in-memory locks alone.
- The first successfully committed disposition is authoritative for its key. A competing equal write is idempotent; a competing different write is a conflict.
- The guarantee is at most one disposition per generation identity within **one SQLite database authority**. No multi-host or distributed exactly-once guarantee, and no exactly-once issuance, is claimed.

### 8. Commit-unconfirmed recovery (EI4-D9-G)

A failure from COMMIT itself is uncertain. The Application reloads the generation identity once, authoritatively, and verifies what it finds:

| Reload finds | Result |
|---|---|
| The same verified fingerprint | Committed, or an equivalent committed |
| A different verified fingerprint | Explicit conflict |
| No record, after a successful reload | No matching commit; a controlled, bounded retry may proceed |
| Reload unavailable or inconsistent | Outcome unresolved; nothing is claimed either way |

A COMMIT exception is never assumed to be a rollback. The store never retries by itself.

### 9. Point-in-time issued history (EI4-D9-H)

- EI-4D never writes dispositions, proposals or attempts into the issued-recommendation history that EI-4C reads through `session_history(exchange, date, as_of)`. A proposal can therefore never block its own retry.
- An independent **issued-recommendation authority** belongs to later issuance and lifecycle work (EI-5, EI-6). It must be append-only, never backdate (EI4-D1), and provide verifiable point-in-time history with recording provenance and a monotone **recording watermark**, so an as-of view can be re-verified after the fact. Modeled issuances in historical runs are recorded later in wall-clock time, so completeness is pinned by that watermark rather than by issuance instants alone.
- A recommendation issued later never changes an earlier modeled assessment. EI4-D8's conservative open-state treatment is unchanged.
- Until the authority exists, missing history is unknown, never empty. No adapter asserts an empty history because EI-5 and EI-6 are not built. Explicitly labelled synthetic test fixtures may establish a complete history for acceptance testing; they are not production authority.

### 10. Restart and historical verification (EI4-D9-I)

On every load, a stored generation record is verified at four distinct levels, which are not interchangeable:

| Level | Verified |
|---|---|
| A. Stored-document integrity | Each document re-encodes canonically to its stored text; the content fingerprint recomputes; every constrained column is what its document produces |
| B. Referential integrity | The referenced EI-3C `CANDIDATE` entry and EI-2B snapshot exist, match the record, and rehydrate under EI3-D13 |
| C. Evaluation reproducibility | With a compatible evaluation format, EI-4C eligibility and EI-4B Core formation re-run from the **stored** policy documents, venue-day evidence and point-in-time history evidence, and from the rehydrated candidate, and reproduce the stored assessment and fingerprint exactly |
| D. Actual issuance | Not established by A to C, and outside EI-4D |

Missing, corrupted, inconsistent or unsupported evidence fails closed. There is no historical evaluator registry: an unsupported format version fails closed, and nothing is migrated, reinterpreted or relinked to newer corrected evidence. SQLite enforces structure and foreign keys; SHA-256 verification is the Application's.

> **Amended 2026-11-13 by [ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md), section 1.** The paragraph above is kept as recorded. From 2026-11-13, "supported" means the closed two-version table of ADR-025:
>
> - evaluator `/1` with its `/1` documents;
> - evaluator `/2` with its `/2` inputs, assessment and attempt documents.
>
> A record is replayed only under the evaluator it names. A label-format mismatch is invalid, and an unknown label is incompatible. This is still not a registry: there is no registration API, and a third version needs another ADR amendment. Levels A to D are unchanged. The amendment is not yet implemented.

### 11. Storage placement and referential integrity

- Generation records live in the **same SQLite database authority** as the EI-2B evidence and EI-3C timelines they reference, so references are real foreign keys and one transaction covers a whole record (EI4-D9-E, EI4-D9-F).
- A disposition references its candidate's timeline entry, by its unique `eirt1` fingerprint, and its EI-2B snapshot. Insert-time checks require the entry to be the timeline's `CANDIDATE`, with matching decision, snapshot, manifest, strategy, listing, date and synthetic evidence class.
- Tables are insert-only. Triggers refuse updates, deletes and replacing inserts. A database check ties the proposal document to the `PROPOSED` kind.
- The Application owns the store port and the record types; Infrastructure owns the SQLite schema and adapter. Application never imports Infrastructure.

### 12. Implementation sequence

1. EI4-D9 governance approval and this ADR (northstar-docs).
2. **EI-4C compatibility** (northstar-application): the versioned composite policy-set identity; versioned canonical structured assessment and proposal documents; content fingerprints without diagnostic prose; modeled issuance from a versioned synthetic latency; complete venue-day timing evidence; verifiable point-in-time issued-history evidence with coverage and recording provenance; persistence-oriented disposition and attempt contracts.
3. **EI-4D-A** (northstar-application): persistence ports, immutable records, verification, and record-and-reconcile orchestration.
4. **EI-4D-B** (northstar-infrastructure): SQLite schema, adapter and concurrency protection.
5. **EI-4E**: synthetic end-to-end acceptance on real SQLite -- restart, concurrent writers, uncertain COMMIT, tampering and replay.

EI-5 and EI-6 stay separate milestones and are not implemented by EI-4D.

## Relationship to Earlier ADRs

- **ADR-017.** The layering is unchanged: Core <- Application <- Infrastructure, with no Futures recommendation type used for equities.
- **ADR-018.** Its F-5 obligations -- idempotent retries, transactional uniqueness and insert-only history -- are adopted as patterns. ADR-018 is unchanged.
- **ADR-019.** Unchanged; listings such as `M&M` remain referenceable.
- **ADR-020.** This ADR settles what ADR-020 deferred to EI4-D9. ADR-020 is not superseded. Where ADR-020 section 3 derives identity from the generation-policy version, EI4-D9-B refines that version to the composite policy-set identity.

## Rationale

- **A stable key with a content fingerprint** keeps EI4-D2's one disposition per policy identity while making equality exact, cheap and replayable.
- **A composite policy set** matches the specification's composite `policy_version` (section 8.1) and supports sensitivity runs without minting artificial generation-policy versions.
- **Evidence as content, not identity** prevents re-generating a candidate until a setup passes by varying evidence or instants.
- **Derived modeled issuance** removes a free timestamp that could otherwise change timing results.
- **First commit authoritative, conflicts refused** follows EI3-D12: history is never rewritten, and order is not mistaken for truth.
- **No writes to issued history** keeps generation and issuance separate, so proposals cannot block themselves and no issuance is fabricated.

## Alternatives Considered

- **Keep the generation-policy-only key and require exact equality (option A, or B-1).** Not chosen: any cost or risk variant would need a new generation-policy version, and equality would rest on whole-document comparison with no stored binding.
- **Put every policy and reference version, and the modeled instant, in the identity (option B).** Rejected: it allows several recommendations per candidate and policy and invites retrying variants until one passes.
- **Last write wins, or keep every variant as a new version of the disposition.** Rejected by EI4-D9-D.
- **Give outages and integrity failures dispositions.** Rejected: they are not judgements on a verified candidate, and would block it under a key that does not exist.
- **Let EI-4D write the issued history, or assert an empty history until EI-5.** Rejected by EI4-D9-H.
- **A separate SQLite file for generation records.** Rejected: references to EI-2B and EI-3C would lose foreign keys, and one transaction could not cover the record.
- **In-memory locks for uniqueness.** Rejected: they do not hold across processes.

## Consequences

### Positive Consequences

- Each candidate has exactly one auditable disposition per approved policy identity, verifiable after restart.
- Retries and concurrent writers are idempotent or explicitly conflicting, never silently divergent.
- Rejected and unresolved results stay distinct, and neither can become a proposal.
- Generation, issuance and lifecycle stay separable for EI-5 and EI-6.

### Negative Consequences

- EI-4C needs a compatibility slice before EI-4D, and its current proposal key is not the generation identity.
- Full historical verification re-runs evaluation and costs more as records grow.
- Storage is single-host, single-file SQLite.
- Until an issued-recommendation authority exists, generation needs labelled synthetic history fixtures, and nothing can reach a production history.

## Scope

This ADR records an architecture. It does not:

- implement any contract, schema, adapter, use case or test;
- approve any numeric latency, cutoff, model capital, risk parameter, cost rate, price increment, liquidity threshold, exposure limit or loss limit;
- approve EI4-D5, EI0-D11, the proposed one-per-listing threshold, EI0-D19, EI0-D21 or EI3-D10;
- define issuance or lifecycle (EI-5), their persistence (EI-6), interfaces (EI-7), evaluation (EI-8) or real data (EI-9);
- authorize real data, actual issuance, trading, orders, fills, positions or production use.

## Future Considerations

- The issued-recommendation authority and its recording watermark, with EI-5 and EI-6.
- An issued recommendation referencing its EI-4D proposal by generation identity and content fingerprint.
- A historical evaluator registry, if evaluation formats must change; until then unsupported formats fail closed.
  - *Note, 2026-11-13.* Evaluation formats did change (C-2). [ADR-025](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) adopts a closed two-version table instead of a registry. A general registry stays a future consideration and is not authorized.
- External audit anchoring, which neither EI-3C nor EI-4D provides.

## Related Documents

- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md)
- [EI-3 Decision Register](../product/Equity-Intraday-V1-EI3-Decision-Register.md)
- [EI-3C Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI3C-Synthetic-Acceptance-Record.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md)
- [ADR-017: Repository Layering and Futures Recommendation Domain Boundary](ADR-017-Repository-Layering-and-Futures-Recommendation-Domain-Boundary.md)
- [ADR-018: Futures V1 Research-to-Recommendation Generation Policy](ADR-018-Futures-V1-Research-to-Recommendation-Generation-Policy.md)
- [ADR-019: Symbol Ampersand Extension](ADR-019-Symbol-Ampersand-Extension.md)
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- Engineering Standards ES-001, ES-002, ES-003, ES-004
- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md) (added 2026-11-12)
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md) (added 2026-11-12)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md) (added 2026-11-12)
- [ADR-025: Equity Intraday V1 Versioned Evaluation and History Compatibility](ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md) (added 2026-11-13)
