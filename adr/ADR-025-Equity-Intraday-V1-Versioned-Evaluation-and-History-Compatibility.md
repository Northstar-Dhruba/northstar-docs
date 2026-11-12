# ADR-025: Equity Intraday V1 Versioned Evaluation and History Compatibility

## Status

Accepted — 2026-11-13

On 2026-11-13 the product owner approved R-1 and R-2, the two contract points that the C-2 correction left open ([EI-5 Decision Register, section 12.2](../product/Equity-Intraday-V1-EI5-Decision-Register.md#122-c-2--the-exclusive-history-boundary)), together with the decisions N1 to N10 recorded in the [EI-5 Decision Register, section 13](../product/Equity-Intraday-V1-EI5-Decision-Register.md#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13). On the same date it separately approved the N10 case-C refusal classification (section 9). This ADR records the architecture those decisions require. The engineering specification and acceptance matrix are in the [EI-4C Versioned History Compatibility Design v1.0](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md).

This ADR **amends** [ADR-021, section 10](ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md#10-restart-and-historical-verification-ei4-d9-i), which says there is no historical evaluator registry. It does not replace ADR-021, and ADR-021 keeps its original text with a dated pointer to this record.

| Item | Status |
| --- | --- |
| R-1 (two-version verification), R-2 (fingerprint prefixes and storage) | **RESOLVED** — owner approved 2026-11-13 (N1, N3) |
| N1 to N10 | **OWNER APPROVED** (2026-11-13), with N10 in the stricter exact-completeness form |
| The N10 case-C refusal classification for an overdeclared version-2 history (section 9) | **OWNER APPROVED** (2026-11-13): `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, status `DEPENDENCY_UNAVAILABLE`, version 2 only, keyless attempt only |
| This architecture record | **ACCEPTED** (2026-11-13) |
| Implementation (slices 1 to 5 of section 13) | **NOT STARTED** |
| Acceptance | **PLANNED**; no scenario executed |
| Existing SQLite schema compatibility with version 2 | **EXPECTED, NOT VERIFIED** until slice 4 runs |
| EI-5D | **NOT AUTHORIZED**; requires the accepted prerequisite |
| EI-4 formal closure | **PENDING** independent CI verification |
| Operational or production authorization | **NOT GRANTED** |

This is a design decision. **Nothing in it is implemented.** Evaluator version 2, the `/2` document formats, the version table, label-format pairing and the history-boundary field do not exist in code. Today's EI-4C still reads the issued history as of the modeled issuance `t`, and today's verifier replays only `northstar.equity-recommendation-evaluator/1`. It records no Architecture Review or Engineering Review sign-off.

## Date

2026-11-13

## Decision Makers

- Product owner: approved R-1, R-2 and N1 to N10 on 2026-11-13, and accepted this ADR.
- Northstar Architecture Team: formal review pending.

## Context

### The original incompatibility (C-2)

Source read at core `ccff26e`, application `75c6a35`, infrastructure `c105e94`, api `7ad37f5`, web `5aeb94c`, docs `498956e`.

- **EI-4C reads history at the modeled issuance.** `GenerateEquityTradeRecommendationUseCase` derives `t = equity_modeled_issuance(decision_as_of, latency)` and calls `session_history(exchange, trading_date, as_of=t)`. It refuses a record that does not `covers(t)`, meaning `complete_through >= t`, with `RECOMMENDATION_HISTORY_INCOMPLETE` (`generate_equity_trade_recommendation.py`).
- **The frozen inputs pin that instant.** `EquityRecommendationInputs` refuses issued-history evidence whose `as_of` is not exactly `t` (`equity_recommendation_generation_evidence.py`).
- **No backdating.** EI5-D4 refuses an issuance whose `issued_at <= complete_through`.

So every version-1 proposal generated against an authority needs `complete_through >= t`, and its own issuance at `t` is then refused. The owner resolved this on 2026-11-12 (C-2): a new, versioned EI-4C assessment takes its history as of `b = t − 1 µs`. That correction left two contract points open: R-1, two-version verification, and R-2, fingerprint prefixes and storage.

### What the version-1 contract actually binds

| Identity | Format or namespace (exact) | Prefix | Binds the evaluator version? |
| --- | --- | --- | --- |
| Policy set | `northstar.equity-recommendation-policy-set/1` | `eirs1-sha256:` | No |
| Generation key | `northstar.equity-recommendation-generation-key/1` | `eirk1-sha256:` | No: strategy, configuration version, listing, date, first CANDIDATE `eirt1`, policy-set version and `eirs1` only |
| Frozen inputs | `northstar.equity-recommendation-generation-inputs/1` | `eiri1-sha256:` | No |
| Assessment | `northstar.equity-recommendation-assessment/1` | `eira1-sha256:` | No: the document has no evaluator field |
| Proposal | `northstar.equity-recommendation-proposal/1` | `eirp1-sha256:` | No; it is also independent of the issued-history evidence |
| Generation attempt | `northstar.equity-recommendation-generation-attempt/1` | `eirat1-sha256:` | No: no evaluator and no instant |
| Core outcome (embedded, not separately fingerprinted) | `northstar.equity-recommendation-core-outcome/1` | — | No |

- The evaluator label, `EQUITY_RECOMMENDATION_EVALUATOR_VERSION = "northstar.equity-recommendation-evaluator/1"`, is a stored disposition field that **no fingerprint covers**. Today it is checked only for membership in `SUPPORTED_EQUITY_RECOMMENDATION_EVALUATOR_VERSIONS`, which holds exactly that one label (`equity_recommendation_generation_decoding.py`). Nothing ties the label to the document formats.
- Each fingerprint is SHA-256 over the canonical JSON of a document that contains its own `format` field, so the format string is part of the hash preimage. The prefix is chosen from the format by `EQUITY_RECOMMENDATION_DOCUMENT_PREFIXES`.
- A document whose format names another version of a known family raises `IncompatibleEquityRecommendationRecordError`; any other defect raises `InvalidEquityRecommendationRecordError`.
- The future issuance identity `eiris1` (C-1) is a different identity from the frozen-inputs fingerprint `eiri1`. This ADR does not change it.

### Decision drivers

1. **Historical replay must not weaken.** Every version-1 record must stay verifiable, at levels A to C, under its original meaning (EI4-D9-I, EI3-D13).
2. **No reinterpretation.** A version-1 record is never rehashed, relabelled, migrated or read under version-2 rules.
3. **A new meaning needs a new canonical identity.** A version-2 document must never be accepted as version 1 because its prefix is familiar.
4. **Fail closed.** Unknown, missing or mismatched versions are refused.
5. **No unbounded extension point.** ADR-021 deliberately excluded an evaluator registry; any widening must stay closed and governed.
6. **No schema migration** unless an actual constraint requires one. The generation tables are insert-only and their initialization never alters existing objects.
7. **Issuance safety.** Generation must not be mistaken for issuance, and the authority's transaction stays the only place where no-backdating is enforced.

## Decision

### 1. A closed set of two evaluator versions (N1, R-1)

Northstar supports exactly two governed evaluator versions:

- `northstar.equity-recommendation-evaluator/1`: the original frozen semantics, history as of `t`;
- `northstar.equity-recommendation-evaluator/2`: strict-before-issuance history, as of `b = t − 1 µs`, with exact completeness (section 9).

They are defined by **one closed, explicitly enumerated version table** in the Application. It maps each evaluator version to its approved canonical document versions (section 3).

- Verification dispatches on the record's **recorded evaluator label**, and each version is verified only under its own approved interpretation.
- Shared pure assessment logic may be reused wherever behaviour is unchanged.
- An unknown label is `IncompatibleEquityRecommendationRecordError`; a missing label is `InvalidEquityRecommendationRecordError`; a label-format mismatch is `InvalidEquityRecommendationRecordError` (section 6).
- **There is no dynamic registration API and no evaluator plugin architecture.** The table is a fixed literal in source. Adding a third evaluator requires another governance amendment: an ADR and an owner decision.

**How this amends ADR-021, section 10.** ADR-021 said: "There is no historical evaluator registry: an unsupported format version fails closed." From 2026-11-13:

- the supported set is the closed two-version table above, not a single version;
- a record is replayed only by the evaluator it names; a version-1 record is never replayed under version-2 history rules;
- an unsupported version still fails closed, and nothing is migrated, reinterpreted or relinked;
- **no extensible evaluator registry is authorized.** ADR-021's "historical evaluator registry" future consideration stays a future consideration.

Every other part of ADR-021 section 10 is unchanged: the four verification levels, and that levels A to C never establish issuance.

### 2. Historical version-1 preservation

Version-1 records keep, exactly:

- their evaluation instant, `as_of = t`, and their inclusive "complete through `t`" meaning;
- their frozen inputs, canonical formats, fingerprints (`eiri1`, `eira1`, `eirp1`, `eirat1`), key (`eirk1`) and evaluator label;
- their replay behaviour and their persisted disposition and attempt identities.

Nothing rehashes, converts, relabels or relaxes them. Version-1 proposals are **never issuable** under the EI-5 protocol, but they stay verifiable.

**Version-1 golden vectors are a mandatory prerequisite.** No fixed reference vector pins `eiri1`, `eira1`, `eirp1` or `eirat1` today; only `eirs1`, `eirk1` and `eipd1` are pinned. Slice 1 (section 13) establishes them from the unchanged current code before any version-2 behaviour is introduced.

### 3. Canonical format versioning (N3, N4)

| Document | Version 1 (evaluator `/1`) | Version 2 (evaluator `/2`) |
| --- | --- | --- |
| Frozen generation inputs | `northstar.equity-recommendation-generation-inputs/1` | `northstar.equity-recommendation-generation-inputs/2` |
| Assessment | `northstar.equity-recommendation-assessment/1` | `northstar.equity-recommendation-assessment/2` |
| Generation attempt | `northstar.equity-recommendation-generation-attempt/1` | `northstar.equity-recommendation-generation-attempt/2` |
| Proposal | `northstar.equity-recommendation-proposal/1` | **unchanged**: `northstar.equity-recommendation-proposal/1` |
| Policy set | `northstar.equity-recommendation-policy-set/1` | **unchanged** |
| Generation key namespace | `northstar.equity-recommendation-generation-key/1` | **unchanged** |
| Core outcome | `northstar.equity-recommendation-core-outcome/1` | **unchanged** |

- A format changes only where canonical meaning changes: the inputs (their history instant and boundary), the assessment that embeds them, and the attempt whose refusal now refers to a different instant.
- The existing SHA-256 encoding, canonical JSON rules and version-1 hash preimages are unchanged. The version-2 documents keep the version-1 field structure, plus the boundary field of section 8, so the JSON paths that SQLite reads stay valid (section 11).
- The generation-key derivation, proposal identity and policy-set identity are unchanged.

### 4. Fingerprint family naming (N3, R-2)

The existing prefixes are retained: `eirs1`, `eirk1`, `eiri1`, `eira1`, `eirp1` and `eirat1`.

- **A prefix names a fingerprint family, not a format version.** `eira1-sha256:` is the prefix of every assessment fingerprint, whether the document is `…-assessment/1` or `…-assessment/2`.
- **The canonical document's `format` identifies the version,** and it is inside the hash preimage, so a version-2 document can never share a fingerprint with a version-1 document of the same content.
- Nothing may infer a version from a prefix. Version is read from the decoded `format` and checked against the evaluator label.
- `eiris1` remains the approved issued-recommendation identity (C-1) and is not the `eiri1` frozen-inputs family.

### 5. Versioned attempts and refusals (N4)

Version-2 attempts use `northstar.equity-recommendation-generation-attempt/2`, with the `eirat1-sha256:` prefix.

- **Separation.** A version-2 refusal, for example history incomplete before `b`, never receives the same logical-attempt identity as a version-1 refusal that assessed a different instant. The attempt document holds no instant and no evaluator field, so without a new format the two would collide.
- **Stability.** Within one version, identical invocations remain one logical attempt (EI4-D9 clarification B), and retries are idempotent.
- **Reference.** A keyed version-2 attempt names the `eira1` fingerprint of its own version-2 assessment.
- **Unchanged.** Append-only storage and idempotence are unchanged. Version-1 attempt documents are never reinterpreted.

### 6. Evaluator-label and format pairing (N6)

Application enforcement is **mandatory**. A record's evaluator label and its canonical format versions must form an approved combination of the version table:

| Evaluator label | Inputs | Assessment | Attempt | Proposal, policy set, Core outcome, key | Verdict |
| --- | --- | --- | --- | --- | --- |
| `/1` | `/1` | `/1` | `/1` | `/1` | Permitted |
| `/2` | `/2` | `/2` | `/2` | `/1` | Permitted |
| `/2` | `/1` | `/1` | — | `/1` | **Invalid** |
| `/1` | `/2` | `/2` | — | `/1` | **Invalid** |
| Any other non-empty label | — | — | — | — | **Incompatible** |
| Missing or empty | — | — | — | — | **Invalid** |

- The check applies wherever the existing architecture constructs, decodes, verifies or authorizes a record: disposition and attempt construction, decoding, `EquityRecommendationGenerationVerifier` dispatch, and the recorder before it issues a write authorization.
- An assessment `/2` that embeds inputs `/1`, or the reverse, is invalid.
- Relabelling without canonical document consistency is never accepted. Today a relabelled record still constructs whenever its label is supported; pairing removes that gap.
- **SQLite does not enforce this pairing.** Its `evaluator_version`, `assessment_format` and `attempt_format` columns are free text, checked only against the document's `format`. An additional database-level pairing trigger is **DEFERRED** and is not part of this prerequisite. The EI4-D11 authorized-write boundary (ADR-023) remains mandatory.

### 7. Explicit evaluator selection (N7)

The generation entry point, `GenerateEquityTradeRecommendationUseCase`, accepts a **keyword-only evaluator selection**, constrained to the closed set.

- **Default:** `northstar.equity-recommendation-evaluator/1`. This preserves existing callers, avoids lockstep changes across the existing construction sites in two repositories, and keeps the version-1 synthetic tests and historical replay.
- **EI-5D composition must explicitly select version 2.**
- Version 1 stays supported for compatibility and verification, but is **not issuable**.
- The global default is not changed to version 2.

### 8. Strict-before-issuance history (N8)

Let `t = modeled_issued_at` and `b = t − 1 µs`, computed with integer microsecond arithmetic on the existing `PointInTime`. `PointInTime` carries at most six fractional digits and canonicalizes to UTC with a `Z` suffix, with trailing fractional zeros removed. So `b` is the last representable instant strictly before `t`.

- Version 1 keeps `issued_history.as_of = t`.
- Version 2 takes `session_history(exchange, trading_date, as_of=b)`, freezes `issued_history.as_of = b`, and keeps `modeled_issued_at = t`. The proposal's timing still states `issued_at = t`.
- Version-2 frozen inputs carry an **explicit canonical history-boundary field** inside `issued_history`. It participates in the `generation-inputs/2` document, and therefore in `eiri1` and in the embedding `eira1`. Its exact name and value are fixed by the [design, section 5](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md#5-the-version-2-contract).
- `PointInTime` in Core is not changed. A future change of timestamp resolution would require a new contract version.

### 9. Exact history completeness (N10)

**Owner approved, 2026-11-13, in its stricter form.** Version-2 generation may record `PROPOSED` only when the authoritative session history satisfies

```
complete_through == b
```

and not merely `complete_through >= b`.

| Case | History | Version-2 generation |
| --- | --- | --- |
| A | `complete_through == b` | The history-boundary condition is satisfied. `PROPOSED` only if every other approved check passes. **Necessary, not sufficient,** for issuance |
| B | `complete_through < b` | No `PROPOSED`. `RECOMMENDATION_HISTORY_INCOMPLETE` (existing reason, status `DEPENDENCY_UNAVAILABLE`); attempt only, with no generation key and no disposition |
| C | `complete_through > b` | No `PROPOSED`. The session is declared complete beyond the version-2 boundary. Fail closed. Reason `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, status `DEPENDENCY_UNAVAILABLE`, approved 2026-11-13. Attempt only, with no generation key and no disposition, so the key is **not** consumed |
| D | Missing, unavailable or another session's history | No `PROPOSED`. `OPEN_RECOMMENDATION_STATE_UNAVAILABLE` (existing reason, status `DEPENDENCY_UNAVAILABLE`); attempt only |

**Case-C refusal classification: OWNER APPROVED, 2026-11-13.** When `complete_through > b`:

| Item | Approved value |
| --- | --- |
| Reason | `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, a new reason. It is not implemented, so it does not exist in production code |
| Status | `DEPENDENCY_UNAVAILABLE`, an existing status |
| Evaluator applicability | `northstar.equity-recommendation-evaluator/2` only. Version 1 is unchanged and never produces it |
| Persistence | A keyless attempt only (`northstar.equity-recommendation-generation-attempt/2`, `generation_key` `None`) |
| Disposition | None: no `PROPOSED` disposition, and no disposition of any kind |
| Generation key | Not consumed |
| Issuance | No authoritative issuance record is created |
| Recovery | None. Nothing backdates or reverses a completeness declaration. Declarations never decrease, so the refusal is permanent for that `t` |
| Identity | Append-only and deterministic. An identical re-run is the same logical attempt, which is idempotent |

**Why this classification.**
- **Accuracy.** It never reuses `RECOMMENDATION_HISTORY_INCOMPLETE`, which means history complete only **before** the required instant (case B). Case C is not reinterpreted as case B.
- **Precedent.** It follows `POLICY_REFERENCE_CONFLICT`, a permanent condition under `DEPENDENCY_UNAVAILABLE` that callers distinguish by reason, never by status.
- **No schema change.** No new status is introduced, so the existing SQLite `status IN (…)` check on the attempts table is unaffected. Reason values are not constrained by SQLite.

Details are in the [design, section 7](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md#7-n10-exact-completeness).

**Atomicity.** The exact generation-time boundary does not replace the issuance authority's check. The future authority must re-verify history and no-backdating inside its own serialized write transaction. If declarations advance after generation and before issuance, issuance fails closed (`BACKDATED_ISSUANCE`).

### 10. Generation-key conflicts across versions (N5)

The generation-key semantics are unchanged, and `eirk1` does not contain the evaluator version. The same key therefore cannot hold both a version-1 and a version-2 disposition. When a competing version meets an existing disposition under its key:

- the first stored disposition is preserved, never overwritten or replaced;
- the recorder returns the existing `EVALUATOR_VERSION_CONFLICT` outcome, and the store its `CONFLICT` outcome;
- the competing record is **not** adopted, and the conflict is not reclassified as `CONTENT_CONFLICT`. This is the recorder's current behaviour: it compares evaluator versions before content, and leaves `authoritative` unset;
- the conflicting logical attempt is recorded when storage is available (EI4-D9-D).

**Consequence.** A candidate already recorded under version 1 can obtain a separate version-2 generation key only under a **genuinely different approved policy-set version**. Nothing reuses or migrates a consumed key.

### 11. Existing SQLite schema compatibility (N3)

**No schema migration is planned.** Under this design, version-2 records are expected to satisfy the existing schema without change:

- the prefix GLOB checks are on families that are retained;
- the format and evaluator columns are free text, checked against the document's `format`;
- the structured-column checks and the CANDIDATE trigger read JSON paths that the version-2 documents preserve;
- status values stay within the existing set.

The detailed inventory is in the [design, section 11](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md#11-existing-sqlite-schema-compatibility).

This is **expected, not verified**. It becomes verified only when real mixed-version SQLite acceptance (slice 4) passes. If an actual constraint conflicts with the version-2 shape, the conflict is reported and governed separately. It is never worked around silently.

### 12. Authorized writer boundaries and issuance safety

**Writers.** EI4-D11 and ADR-023 are unchanged:

- the verified recorder is the only operational writer;
- it issues a write authorization only after dispatched verification;
- the authorization binds the exact disposition content and its evaluator label.

**Issuance (EI-5).** These requirements are preserved:

- MODELED only;
- generation is not issuance;
- version-1 proposals are non-issuable;
- version-2 verification is required before issuance;
- `eiris1` remains the issued identity;
- one issuance per generation key, and at most one per (strategy, listing, trading date);
- no backdating; explicit completeness declarations; stable recording watermarks;
- append-only issuance history;
- unknown or unverifiable history fails closed.

The issuance authority's serialized transaction enforces uniqueness and no-backdating. **Application pre-checks are never authoritative.**

**Remaining restriction.** EI4-D8's conservative open-state treatment is unchanged. A later modeled issuance in the same exchange session sees an earlier issued recommendation as open, so it resolves as `EVALUATION_UNRESOLVED`, which is non-issuable. It is not relaxed here.

### 13. Implementation sequence (N9)

| Slice | Repository | Scope | Before |
| --- | --- | --- | --- |
| 0 | northstar-docs | This ADR, the register amendments, the compatibility design and its acceptance matrix | — |
| 1 | northstar-application | Version-1 freeze: golden vectors and pinned canonical documents; replace the conflicting future-evaluator test labels. No production change | 3a |
| 2 | northstar-infrastructure | Test hygiene: replace the conflicting labels; a version-1 persisted-row replay fixture. No production change | 3a |
| 3a | northstar-application | Two-version verification: version table, `/2` formats, decoder pairing, version-aware records and verifier dispatch. No version-2 generation | 3b |
| 3b | northstar-application | Version-2 generation: explicit selection, `/2` inputs at `b`, the N10 exact boundary, `/2` assessments and attempts, typed non-`PROPOSED` outcomes. No issuance | 4 |
| 4 | northstar-infrastructure | Mixed-version acceptance on real SQLite. No schema migration unless an independently governed blocker is found | 5 |
| 5 | northstar-docs | Acceptance record, limitations, CI evidence where available | 6 |
| 6 | northstar-application | EI-5D issuance, a separately scoped milestone, after slice 5 | — |

Slices 1 and 2 may proceed independently after this governance is approved, but both must merge before slice 3a. Core, API and Web have no slice. Gates and the cross-repository test dependency are in the [design, section 13](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md#13-implementation-sequence).

## Relationship to Earlier ADRs

- **ADR-020.** Unchanged. EI4-D2's zero or one recommendation per candidate under a policy identity is reinforced by section 10.
- **ADR-021.** Section 10 is amended by section 1 of this ADR, through a dated pointer; its original text and historical meaning are kept. Every other ADR-021 decision is unchanged.
- **ADR-022.** Unchanged. The policy-binding gate applies to dispositions of either version.
- **ADR-023.** Unchanged. The verified recorder stays the only operational writer, and the authorization binds the evaluator label.
- **ADR-024.** Unchanged and still accepted. This ADR settles the R-1 and R-2 points ADR-024 left to the prerequisite's design, and adds the N10 exact-completeness rule to the version-2 assessment. ADR-024's no-backdating rule, issuance protocol and `eiris1` identity are unchanged.

## Rationale

- **The only behavioural difference is the history instant and its completeness rule.** `assess_equity_recommendation_inputs` is already pure over frozen inputs. One closed table plus shared pure code is the smallest design that keeps both interpretations exact.
- **Separate code copies would not freeze version 1.** Both copies would still call the shared EI-4B Core formation. Golden vectors and stored-fixture replay are what actually detect drift.
- **The format string is already in the preimage.** Retaining the prefixes therefore gives full cryptographic separation without touching the SQLite GLOB checks. SQLite cannot alter a CHECK constraint, so new prefixes would need a table rebuild or parallel tables.
- **Versioning only what changed** keeps the generation key, proposal and policy-set identities stable, and therefore EI4-D2's one-disposition rule.
- **Exact completeness at generation (N10)** stops a version-2 `PROPOSED` disposition from being recorded when the issuance protocol already makes it non-issuable. It does not consume the key.

## Alternatives Considered

- **Separate immutable verifier implementations per version.** Rejected: they duplicate the decoding, evidence and generation modules (about 3,000 lines) without freezing the shared Core formation.
- **An extensible evaluator registry.** Rejected: it contradicts ADR-021 section 10's intent and adds a plugin trust surface for no approved need.
- **Keep a version-1-only verifier and let version-1 records become incompatible.** Rejected under C-2: it weakens historical replay.
- **New fingerprint prefixes** (`eiri2`, `eira2`, `eirat2`). Rejected:
  - the SQLite GLOB checks would need a table rebuild that rewrites stored rows, or parallel tables that break single-table key uniqueness and the attempt foreign key;
  - `CREATE TABLE IF NOT EXISTS` would silently keep the old checks on existing files.
- **Prefixes changed only where meaning changes.** Rejected: inputs, assessment and attempt all change meaning, so this reduces to new prefixes.
- **A generation-key namespace that includes the evaluator.** Rejected: it would allow two dispositions per candidate and policy identity, which EI4-D2 and EI4-D9-D forbid.
- **Reclassify a cross-version collision as `CONTENT_CONFLICT`, adopting the stored record.** Rejected (N5): the existing `EVALUATOR_VERSION_CONFLICT` is more precise and never adopts.
- **Version-2 generation accepting `complete_through >= b`** (the review's original N10 recommendation). Superseded by the owner's stricter N10.
- **A database-level pairing trigger now.** Deferred (N6).
- **Classifying N10 case C as `RECOMMENDATION_HISTORY_INCOMPLETE`, or under status `NOT_ELIGIBLE`.** Not chosen (2026-11-13).
  - The first misdescribes overdeclared history as incomplete.
  - The owner approved `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED` under `DEPENDENCY_UNAVAILABLE` instead (section 9).

## Consequences

### Positive Consequences

- Version-1 history keeps its exact meaning and stays verifiable.
- Version-2 records are cryptographically distinct from version 1 under familiar prefixes.
- No schema migration is expected.
- Unknown, missing and mismatched versions all fail closed.
- Generation records only proposals that the strict issuance protocol could still issue at generation time.

### Negative Consequences

- A version-1 disposition permanently occupies its generation key; a different approved policy-set version is needed for a distinct identity.
- Under EI4-D8, later same-session issuances may be blocked by the conservative open-state treatment.
- Re-running version-2 generation after a competing issuance at the same `t` changes the recording watermark, which gives `CONTENT_CONFLICT`. After a declaration beyond `b`, the re-run is refused as case C.
- Replay depends on the version-1 golden vectors and on stable Core formation. A Core change that alters formation is a `REPLAY_MISMATCH` that fails closed.
- Monitoring (EI-6) must declare and issue in ascending modeled-issuance order and respect monotone completeness.
- Prefixes alone do not identify format versions.
- Existing Application and Infrastructure tests use `northstar.equity-recommendation-evaluator/2` as an unsupported future label. That label is now reserved, and those tests must change first (slices 1 and 2).
- Several Infrastructure tests construct identical version-1 documents under a monkeypatched second label. Once pairing exists (slice 3a), that premise can no longer be constructed, so they must be re-expressed in the coordinated order described in the design.

### Acceptance requirements

The acceptance matrix of the [compatibility design, section 12](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md#12-acceptance-matrix) must pass, slice by slice, before the prerequisite is accepted. At minimum:

- version-1 golden vectors and replay;
- version-2 formation and microsecond-boundary arithmetic;
- cases A to D of N10;
- label-format pairing;
- cross-version key conflicts;
- distinct attempt identities;
- mixed-version persistence, restart, concurrency and commit-uncertain reconciliation on real SQLite.

Every scenario is **PLANNED**. Existing version-1 tests are not evidence of version-2 behaviour.

## Scope

This ADR records an accepted architecture. It does not:

- implement evaluator version 2, any `/2` format, the version table, pairing, golden vectors or any test;
- alter any SQLite schema, run a migration, or add the deferred pairing trigger;
- implement the approved N10 case-C reason `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED` (slice 3b);
- implement or authorize EI-5D, issuance, declarations, a declaration scheduler, monitoring or the authority;
- change `PointInTime`, Core, API or Web;
- relax EI4-D8, or approve EI0-D11, the exclusivity threshold, EI0-D19, EI0-D21, EI3-D10, O-2 or O-3;
- authorize ACTUAL issuance, real data, actionable recommendations, orders, fills, positions or production use;
- close EI-4, whose formal closure stays pending independent CI verification.

## Future Considerations

- A database-level label-format pairing trigger (deferred under N6).
- A third evaluator version, only through another ADR amendment.
- Relaxing EI4-D8 with authoritative lifecycle state.
- A finer timestamp resolution, which would need a new boundary contract.

## Related Documents

- [EI-5 Decision Register](../product/Equity-Intraday-V1-EI5-Decision-Register.md), sections 12.2 and 13
- [EI-4C Versioned History Compatibility Design v1.0](../architecture/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Design-v1.0.md)
- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md), EI4-D9-I
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design](../architecture/Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md)
