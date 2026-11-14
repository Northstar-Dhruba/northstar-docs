# Equity Intraday V1 — EI-4C Versioned History Compatibility Design v1.0

Status: **Approved governance; implementation not started** (2026-11-13). Version 1.0.

- **Governance basis.** R-1, R-2 and N1 to N10 were **OWNER APPROVED, 2026-11-13** ([EI-5 Decision Register, section 13](../product/Equity-Intraday-V1-EI5-Decision-Register.md#13-owner-approval-of-the-ei-4c-versioned-compatibility-governance-2026-11-13)). The architecture is [ADR-025](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md).
- **Not implemented.** Every type, constant, format, field, check and test named here as version 2 is specified, not built. Today's code supports only evaluator version 1 and reads history as of `t`.
- **N10 case C.** Its refusal classification (section 7) was **owner approved on 2026-11-13**: `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, status `DEPENDENCY_UNAVAILABLE`, version 2 only, keyless attempt only. It is not implemented.
- **Acceptance.** Every scenario in section 12 is **PLANNED**.
- **Location.** `architecture/` holds design specifications, and the README's Architecture row already links the folder. A revised design is a new version; this file is not rewritten.

> **Status update, 2026-11-14 (implemented; engineering acceptance recorded).** The status lines above and in section 1 record the state on 2026-11-13 and are not rewritten. Since then:
>
> - **Sequencing (N9).** Slices 3a and 3b were delivered as one combined Application PR, owner approved 2026-11-14; packaging only (section 13).
> - **Implemented.** Slices 1, 2, the combined 3a + 3b and 4 are merged. The design's contracts were implemented as specified: the field name `history_boundary`, its value `"STRICTLY_BEFORE_MODELED_ISSUANCE"` and the case-C reason are used exactly, so no design amendment was needed for them.
> - **SQLite.** Section 11's compatibility is **VERIFIED** on real SQLite, with no migration.
> - **Acceptance.** Section 12's scenarios are reconciled in the [EI-4C Engineering Acceptance Record](../product/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md#10-acceptance-evidence-matrix): 40 VERIFIED, 8 VERIFIED WITH LIMITATION, 1 NOT VERIFIED, 3 PLANNED. Engineering verdict PASS WITH DOCUMENTED LIMITATIONS.
> - **Owner acceptance, 2026-11-14.** EI-4C versioned history compatibility is **ACCEPTED WITH DOCUMENTED LIMITATIONS** ([EI-5 Decision Register, section 14.4](../product/Equity-Intraday-V1-EI5-Decision-Register.md#144-owner-acceptance-of-slice-5-2026-11-14)). VC-04 stays NOT VERIFIED; rollback or downgrade to a pre-3a Application against stored evaluator-v2 records is not authorized without separate compatibility verification.
> - **Unchanged.** EI-5D is not authorized and requires a separate owner milestone authorization; EI-4 formal closure is pending independent CI verification.

Source paths are relative to each repository root. Code was read at:

| Repository | Branch | Commit |
| --- | --- | --- |
| northstar-core | `develop` | `ccff26e` (EI-5B #39) |
| northstar-application | `develop` | `75c6a35` (EI-5C #93) |
| northstar-infrastructure | `develop` | `c105e94` (L9-P #63) |
| northstar-api | `develop` | `7ad37f5` |
| northstar-web | `develop` | `5aeb94c` |
| northstar-docs | `develop` | `498956e` (EI5-D7 boundary addendum #58) |

## 1. Status and scope

| Item | Status |
| --- | --- |
| R-1, R-2, N1 to N10 | **OWNER APPROVED** (2026-11-13); N10 in its stricter exact-completeness form |
| ADR-025 | **ACCEPTED** (2026-11-13) |
| This design | **Approved governance; implementation not started** |
| N10 case-C refusal classification | **OWNER APPROVED** (2026-11-13): `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED` / `DEPENDENCY_UNAVAILABLE`; not implemented |
| Slices 1 to 5 | **NOT STARTED** |
| Existing SQLite schema compatibility | **EXPECTED, NOT VERIFIED** |
| EI-5D | **NOT AUTHORIZED** |
| EI-4 formal closure | **PENDING** independent CI verification |

*2026-11-14:* slices 1, 2, 3a + 3b (combined) and 4 are merged, SQLite compatibility is **VERIFIED**, and slice 5 is recorded and **accepted by the owner with documented limitations on 2026-11-14** (status update above). The table records the 2026-11-13 state.

In scope:

- the version-1 contract, as it is;
- the version-2 contract;
- verification dispatch and label-format pairing;
- the history boundary and N10;
- attempts and cross-version conflicts;
- SQLite compatibility, the acceptance matrix and the slices.

Out of scope: issuance, declarations, the authority, monitoring, and any change to Core, API or Web.

## 2. Governing decisions

| ID | Decision | Section |
| --- | --- | --- |
| N1 (R-1) | Closed two-version evaluator verification; dispatch by the recorded label; no registry | 4 |
| N2 | ADR-025 amends ADR-021 section 10 by a dated pointer | ADR-025 |
| N3 (R-2) | Retain every fingerprint prefix; the `format` in the preimage identifies the version; no schema migration | 3, 9, 11 |
| N4 | `generation-attempt/2` | 9 |
| N5 | A cross-version key collision preserves the first record and returns `EVALUATOR_VERSION_CONFLICT` | 10 |
| N6 | Mandatory Application label-format pairing; the SQLite pairing trigger is deferred | 8 |
| N7 | Keyword-only evaluator selection, defaulting to `/1`; EI-5D selects `/2` | 4 |
| N8 | Explicit version-2 history-boundary field; history as of `b = t − 1 µs` | 5, 6 |
| N9 | The staged slices | 13 |
| N10 | Version-2 `PROPOSED` only when `complete_through == b` | 7 |

## 3. The existing version-1 contract

Constants are in `northstar-application/src/northstar_application/application_services/equity_recommendation_generation_evidence.py` and `equity_recommendation_generation_decoding.py`.

| Item | Exact identifier |
| --- | --- |
| Evaluator label | `EQUITY_RECOMMENDATION_EVALUATOR_VERSION = "northstar.equity-recommendation-evaluator/1"`; `SUPPORTED_EQUITY_RECOMMENDATION_EVALUATOR_VERSIONS` holds only it |
| Policy set | `northstar.equity-recommendation-policy-set/1`, prefix `eirs1-sha256:` |
| Frozen inputs | `northstar.equity-recommendation-generation-inputs/1`, prefix `eiri1-sha256:` |
| Assessment | `northstar.equity-recommendation-assessment/1`, prefix `eira1-sha256:` |
| Proposal | `northstar.equity-recommendation-proposal/1`, prefix `eirp1-sha256:` |
| Generation attempt | `northstar.equity-recommendation-generation-attempt/1`, prefix `eirat1-sha256:` |
| Core outcome (embedded) | `northstar.equity-recommendation-core-outcome/1`; no prefix of its own |
| Generation key | Namespace `northstar.equity-recommendation-generation-key/1`, prefix `eirk1-sha256:`, length-prefixed parts |
| Policy-definition binding (EI4-D10) | `northstar.equity-synthetic-policy-definition/1`, prefix `eipd1-sha256:`; never in a generation record |

**Version-1 behaviour.**

| Area | Behaviour |
| --- | --- |
| History instant | `t = equity_modeled_issuance(decision_as_of, modeled_issuance_latency)`. History is read with `session_history(exchange, trading_date, as_of=t)` and must `covers(t)`, meaning `complete_through >= t` |
| Frozen evidence | `issued_history.as_of = t`. `EquityRecommendationInputs` refuses any other `as_of` |
| Fingerprints | SHA-256 over canonical JSON (sorted keys, no whitespace, no floats). The document's `format` is in the preimage, and the prefix is chosen from it by `EQUITY_RECOMMENDATION_DOCUMENT_PREFIXES` |
| Evaluator label | A stored disposition field that **no fingerprint covers**. The assessment and attempt documents have no evaluator field |
| Decoding | An exact format is required. Another version of the same family raises `IncompatibleEquityRecommendationRecordError`; anything else raises `InvalidEquityRecommendationRecordError` |
| Verification | `EquityRecommendationGenerationVerifier.verify` refuses any label other than `/1` as incompatible. Level B re-establishes the CANDIDATE through `establish_equity_recommendation_candidate`. Level C replays the pure `assess_equity_recommendation_inputs` on the frozen inputs |
| Recording | `RecordEquityRecommendationGenerationUseCase` verifies, gates and authorizes, then writes. On a stored record of another evaluator, it returns `EVALUATOR_VERSION_CONFLICT` **before** comparing content, and does not adopt the stored record |

**Golden-vector coverage today.** `eirs1`, `eirk1` and `eipd1` are pinned by golden vectors. `eiri1`, `eira1`, `eirp1` and `eirat1` are **not**.

## 4. The closed version table and verification dispatch

**The table.** One fixed literal in the Application, with no registration API:

| Evaluator | Inputs | Assessment | Attempt | Proposal | Policy set | Core outcome | Key namespace | History `as_of` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `northstar.equity-recommendation-evaluator/1` | `…-generation-inputs/1` | `…-assessment/1` | `…-generation-attempt/1` | `…-proposal/1` | `…-policy-set/1` | `…-core-outcome/1` | `…-generation-key/1` | `t` |
| `northstar.equity-recommendation-evaluator/2` | `…-generation-inputs/2` | `…-assessment/2` | `…-generation-attempt/2` | `…-proposal/1` | `…-policy-set/1` | `…-core-outcome/1` | `…-generation-key/1` | `b = t − 1 µs` |

Every `…` stands for `northstar.equity-recommendation`; the full strings are in section 3 and section 5.

**Dispatch rules:**

1. **Read the label.** Read the record's recorded evaluator label.
   - Missing or empty: `InvalidEquityRecommendationRecordError`.
   - Not in the table: `IncompatibleEquityRecommendationRecordError`.
2. **Check the pairing.** Require the record's document formats to be that label's row (section 8).
3. **Decode.** Decode with that version's decoders.
4. **Verify.** Re-establish the source CANDIDATE at level B, exactly as today.
5. **Replay.** Replay the assessment with that version's envelope and the shared pure assessment, and require the stored assessment and proposal fingerprints to reproduce exactly.

A version-1 record is never replayed under version-2 rules, and a version-2 record is never accepted as version 1 because its prefix is familiar.

**Evaluator selection (N7).**
- `GenerateEquityTradeRecommendationUseCase` accepts a keyword-only evaluator selection, constrained to the table, and defaults to `/1`. Every existing construction site therefore behaves as today.
- EI-5D composition must select `/2` explicitly.
- The global default is not changed.

**EI-5C.** `EvaluateSyntheticEquityRecommendationLifecycleUseCase` consumes `VerifyStoredEquityRecommendationGenerationUseCase` and does not read the evaluator label.
- After slice 3a it can evaluate a verified `PROPOSED` disposition of either version.
- That remains synthetic lifecycle evaluation, never issuance.
- Its contracts are unchanged.

## 5. The version-2 contract

**Frozen inputs: `northstar.equity-recommendation-generation-inputs/2`.**
- **Same fields as version 1.** `format`, `candidate`, `policy_set`, `policy_set_fingerprint`, `listing_eligibility`, `venue_day`, `issued_history` and `modeled_issued_at`, with the same meaning.
- **One difference.** `issued_history` changes as below.

| `issued_history` field | Version 1 | Version 2 |
| --- | --- | --- |
| `exchange_code`, `trading_date`, `reference`, `recording_watermark`, `recommendations` | As today | Unchanged meaning |
| `history_boundary` | Absent | **Required.** The text `"STRICTLY_BEFORE_MODELED_ISSUANCE"` |
| `as_of` | `t` | `b = t − 1 µs` |
| `complete_through` | Observed; `>= t` | Observed; **exactly `b`** (N10) |

`modeled_issued_at` stays `t` in both versions.

**Construction invariants.** A version-2 inputs value refuses construction unless all of the following hold:
- `history_boundary` is exactly `"STRICTLY_BEFORE_MODELED_ISSUANCE"`;
- `issued_history.as_of` equals the derived `t` minus one microsecond;
- `issued_history.complete_through` equals `issued_history.as_of`;
- every existing version-1 inputs invariant other than `as_of = t` holds;
- every recommendation in the evidence has `issued_at <= b`, which follows from the existing evidence checks.

**Field naming.** The field name `history_boundary` and its value are fixed by this design. Slice 3b uses them exactly, or obtains a dated design amendment first.

**Assessment: `northstar.equity-recommendation-assessment/2`.**
- **Same fields as version 1:** `format`, `synthetic`, `authoritative`, `request`, `generation_key`, `status`, `reasons`, `checks`, `candidate`, `inputs`, `inputs_fingerprint`, `entry_window`, `core_outcome`, `proposal` and `proposal_fingerprint`.
- **What differs:** `inputs` must be a `generation-inputs/2` document, and `inputs_fingerprint` is its `eiri1`.
- **Unchanged check:** the entry window must be timed from `inputs.modeled_issued_at`, which is `t`.

**Proposal: `northstar.equity-recommendation-proposal/1`, unchanged.** It holds the generation key, Core definition and entry window, with timing `issued_at = t`. It does not depend on the issued-history evidence. So for one key, a version-2 proposal can be byte-identical to a version-1 proposal; section 10 explains why that never yields two records.

**Attempt: `northstar.equity-recommendation-generation-attempt/2`.**
- Same fields as version 1: `format`, `request`, `status`, `reasons`, `checks`, `generation_key`, `disposition_kind` and `assessment_fingerprint`.
- A keyed version-2 attempt names the `eira1` fingerprint of its own assessment `/2`.

## 6. History-boundary policy

Let `t = modeled_issued_at` and `b = t − 1 µs`.

- **Arithmetic.** Use the existing integer-microsecond arithmetic (the same arithmetic as `equity_modeled_issuance`), with no float and no clock.
- **Representation.** `PointInTime` keeps at most six fractional digits and canonicalizes to UTC with a `Z` suffix and no trailing fractional zeros (`northstar-core/src/northstar_core/foundation/value_objects/point_in_time.py`). `b` is therefore the last representable instant strictly before `t`.
- **No Core change.** `PointInTime` is not changed. A finer resolution would need a new boundary contract.

Worked values, computed with today's code:

| `t` as given | Canonical `t` | Canonical `b` | Rollover |
| --- | --- | --- | --- |
| `2026-08-12T09:36:00+05:30` (IST) | `2026-08-12T04:06:00Z` | `2026-08-12T04:05:59.999999Z` | Second and minute |
| `2026-08-12T04:06:00.000001Z` | `2026-08-12T04:06:00.000001Z` | `2026-08-12T04:06:00Z` | Fraction drops to none |
| `2026-08-12T04:06:00.5Z` | `2026-08-12T04:06:00.5Z` | `2026-08-12T04:06:00.499999Z` | Within the second |
| `2026-08-13T00:00:00+05:30` (IST midnight) | `2026-08-12T18:30:00Z` | `2026-08-12T18:29:59.999999Z` | IST date |
| `2026-08-13T00:00:00Z` (UTC midnight) | `2026-08-13T00:00:00Z` | `2026-08-12T23:59:59.999999Z` | UTC date |

The first row is the modeled issuance of the existing EI-4C synthetic fixture. The midnight rows exercise arithmetic only; no trading session reaches midnight.

**Within the boundary:**
- the version-2 frozen evidence lists exactly the issuances with `issued_at < t`, which is `issued_at <= b`;
- issuances at exactly `t` are excluded;
- version-1 evidence keeps listing them, which is why version 1 is never reinterpreted.

## 7. N10 exact completeness

Version-2 generation may produce `PROPOSED` only when the authoritative history satisfies `complete_through == b`.

The history step runs before the frozen inputs exist. So every refusal below is an **attempt-only** result:
- `generation_key` is `None` and no disposition is created, so the key is never consumed;
- the attempt is a `generation-attempt/2` document, distinct from any version-1 attempt.

This matches today's behaviour for `RECOMMENDATION_HISTORY_INCOMPLETE`, verified with the existing EI-4C fixture.

| Case | Condition | Reason and status | Retry |
| --- | --- | --- | --- |
| A | `complete_through == b` | The history check passes. `PROPOSED` only if every other check passes. Necessary, not sufficient, for issuance | A re-run after a competing issuance at `t` changes `recording_watermark`, so it gives another `eira1` and `CONTENT_CONFLICT`. The first record stays and remains authoritative |
| B | `complete_through < b` | `RECOMMENDATION_HISTORY_INCOMPLETE`, status `DEPENDENCY_UNAVAILABLE`. **Accurate:** its existing meaning is "the record is complete only through an instant before the required one" | May succeed once the session is declared through exactly `b`; that is a different attempt |
| C | `complete_through > b` | `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, status `DEPENDENCY_UNAVAILABLE`. **Owner approved 2026-11-13**; version 2 only. Fail closed; no `PROPOSED`; no disposition; keyless attempt only | Permanent for that `t`: declarations never decrease, and nothing backdates or reverses one. An identical re-run is the same attempt, which is idempotent |
| D | `None`, unavailable, or another session | `OPEN_RECOMMENDATION_STATE_UNAVAILABLE`, status `DEPENDENCY_UNAVAILABLE`, as today | May succeed once history is established |

### The version-2 history step

For a version-2 generation, after the candidate and policy set are established:

1. **Compute the boundary.** Derive `t` and `b = t − 1 µs`.
2. **Read the history.** Read `session_history(exchange, trading_date, as_of=b)`.
3. **Refuse a missing or foreign history.** `None`, an unavailable source, or another session's history is refused as case D.
4. **Classify by `complete_through`.**
   - `complete_through < b`: case B.
   - `complete_through > b`: case C.
   - Exactly `b`: case A, which freezes `issued_history` with `history_boundary`, `as_of = complete_through = b`, the observed `recording_watermark` and `issued_by(b)`.
5. **Refusals record an attempt only.** Cases B to D yield a keyless `generation-attempt/2`, recorded through the existing `record_attempt` path, and nothing else.
6. **Case A continues.** It goes on to the existing assessment and Core formation.

### The case-C refusal classification (owner approved, 2026-11-13)

| Item | Approved value |
| --- | --- |
| Condition | `complete_through > b`, where `b = t − 1 µs` and `t = modeled_issued_at` |
| Reason | `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`: the session's issued history is declared complete beyond `b`, so the modeled issuance at `t` can no longer satisfy the issuance protocol. A new reason, **not yet implemented** |
| Status | `DEPENDENCY_UNAVAILABLE`, an existing status, reported on the existing `SYNTHETIC_REFERENCES` check |
| Evaluator applicability | `northstar.equity-recommendation-evaluator/2` only |
| Persistence | A keyless `generation-attempt/2` (`generation_key` `None`, `disposition_kind` `None`, `assessment_fingerprint` `None`) through `record_attempt` |
| Disposition | None; never `PROPOSED` |
| Generation key | Not consumed |
| Issuance | No authoritative issuance record |
| Recovery | None. Nothing backdates or reverses a completeness declaration |
| Identity | Append-only and deterministic; identical re-runs are idempotent |

**Why it is classified this way.**
- **Accuracy.** The existing vocabulary in `equity_trade_recommendation_eligibility.py` had no accurate reason. `RECOMMENDATION_HISTORY_INCOMPLETE` means complete only **before** the required instant, which is case B, and is never used for case C. `OPEN_RECOMMENDATION_STATE_UNAVAILABLE` means no record, an unreadable record, or another session's record, which is case D.
- **Precedent.** `POLICY_REFERENCE_CONFLICT` is already a permanent condition under `DEPENDENCY_UNAVAILABLE`, distinguished by reason, never by status.

**Constraints the classification respects.**
- **No new status.** The SQLite attempts table checks `status IN ('PROPOSED', 'NO_CANDIDATE', 'NOT_ELIGIBLE', 'EVALUATION_UNRESOLVED', 'DEPENDENCY_UNAVAILABLE', 'RESEARCH_INTEGRITY_FAILURE')`, so the approved status needs no schema change. Reason values are not constrained by SQLite.
- **Version 1 is untouched.** Version-1 generation keeps `complete_through >= t` and never produces this reason, and version-1 code decodes only `/1` attempts.
- **The recorder's outcomes are unchanged.**

**Not chosen.** The same reason under status `NOT_ELIGIBLE`; and reusing `RECOMMENDATION_HISTORY_INCOMPLETE`.

**Atomicity.**
- N10 is checked at generation time only.
- The future issuance authority must independently re-verify history and no-backdating inside its own serialized write transaction (EI5-D4).
- A declaration that advances after generation and before issuance makes issuance fail closed with `BACKDATED_ISSUANCE`.
- Nothing here implements issuance or a declaration scheduler.

## 8. Label-format pairing

**Mandatory Application enforcement (N6).**

| Label | Inputs | Assessment | Attempt | Proposal | Result |
| --- | --- | --- | --- | --- | --- |
| `/1` | `/1` | `/1` | `/1` | `/1` | Permitted |
| `/2` | `/2` | `/2` | `/2` | `/1` | Permitted |
| `/2` | `/1` | `/1` | any | `/1` | `InvalidEquityRecommendationRecordError` |
| `/1` | `/2` | `/2` | any | `/1` | `InvalidEquityRecommendationRecordError` |
| `/1` or `/2` | Assessment `/2` embedding inputs `/1`, or the reverse | — | — | — | `InvalidEquityRecommendationRecordError` |
| Unknown | — | — | — | — | `IncompatibleEquityRecommendationRecordError` |
| Missing or empty | — | — | — | — | `InvalidEquityRecommendationRecordError` |
| Any | Proposal, policy set or Core outcome other than `/1` | — | — | — | Incompatible within a known family, Invalid otherwise, as today |

**Where it applies:**
- `EquityRecommendationGenerationDisposition` and `EquityRecommendationGenerationAttempt` construction, and so every load by the store adapter;
- the decoders;
- `EquityRecommendationGenerationVerifier` dispatch;
- the recorder, before it issues `VerifiedEquityRecommendationGenerationWrite` or `VerifiedEquityRecommendationAttemptWrite`.

**Attempts.**
- An attempt pairs with the evaluator that produced **its own** audit: `/2` only from a version-2 generation.
- A version-2 attempt may legitimately sit under a version-1 disposition's key as the recorded conflicting attempt of N5. The existing SQLite trigger allows that, because the assessment fingerprints differ.

**SQLite does not enforce pairing.**
- `evaluator_version` is free text (non-empty).
- `assessment_format` and `attempt_format` are free text, checked only to equal the document's `$.format`.
- The database-level pairing trigger is **DEFERRED** and is not part of this prerequisite.
- The EI4-D11 authorized-write boundary remains mandatory.

## 9. Fingerprints and attempt identity

- **Prefixes retained (N3).** `generation-inputs/2` uses `eiri1-sha256:`, `assessment/2` uses `eira1-sha256:` and `generation-attempt/2` uses `eirat1-sha256:`. `EQUITY_RECOMMENDATION_DOCUMENT_PREFIXES` gains these three entries; no prefix changes.
- **Separation.**
  - The `format` string is inside the canonical preimage, so a version-2 document never shares a fingerprint with a version-1 document.
  - The `history_boundary` field and the `b` instants also differ.
  - Nothing infers a version from a prefix.
- **Attempts (N4).**
  - **Distinct across versions.** A version-2 attempt never shares a logical-attempt identity with a version-1 attempt, whether it is keyless (history incomplete, overdeclared or missing) or keyed.
  - **Stable within a version.** Identical invocations are one logical attempt, and retries are idempotent.
  - **Keyed attempts** name their own assessment's `eira1`; the existing `eira1-sha256:` prefix check in the attempt record still holds.
- **Unchanged identities.** `eirk1`, `eirs1`, `eirp1` and `eiris1` (C-1) are unchanged.

## 10. Cross-version conflicts and authorized writer behaviour

**Generation-key collisions (N5).**
- `eirk1` holds no evaluator version, so one key holds at most one disposition, of one version.
- When a version-2 disposition is offered under a key that already holds a version-1 disposition, or the reverse:
  - the store returns its `CONFLICT` outcome and writes no disposition;
  - the recorder returns `EVALUATOR_VERSION_CONFLICT`, with `authoritative` unset;
  - the first record is preserved, not adopted as the competing version's, and not reclassified as `CONTENT_CONFLICT`;
  - the conflicting attempt is recorded when storage is available.
- A candidate already recorded under version 1 needs a genuinely different approved policy-set version for a separate version-2 key. A consumed key is never reused or migrated.

**The same proposal identity under different assessments.**
- A proposal embeds its generation key, so equal `eirp1` values imply the same key, and the collision above applies.
- The `UNIQUE` `proposal_fingerprint` column and the no-replace trigger independently refuse a second row.

**Within one version,** the existing rules apply: equal content is `EQUIVALENT_COMMITTED`, and other content is `CONTENT_CONFLICT`.

**Authorized writers.** EI4-D11 and ADR-023 are unchanged:
- authorizations are issued only by the recorder, after dispatched verification;
- an authorization binds the exact disposition content and its evaluator label;
- a record whose label and formats do not pair cannot be constructed, so it cannot be authorized.

## 11. Existing SQLite schema compatibility

Source: `northstar-infrastructure/src/northstar_infrastructure/persistence/sqlite_equity_recommendation_generation_schema.py`.

| Constraint | Existing rule | Version-2 expectation |
| --- | --- | --- |
| Primary keys | `generation_key` (`GLOB 'eirk1-sha256:?*'`) on dispositions; `attempt_identity` (`GLOB 'eirat1-sha256:?*'`) on attempts | Unchanged families; satisfied |
| `UNIQUE` | `proposal_fingerprint` | Satisfied; same key implies a conflict first |
| Fingerprint GLOBs | `eira1`, `eirp1`, `eiri1`, `eirs1`, `eir1`, `eirt1`, `ecs1`, `ecm1`; attempts `eirat1`, `eirk1`, `eira1` | Families retained; satisfied |
| Format columns | `assessment_format`, `proposal_format`, `attempt_format` are free text, each required to equal the document's `$.format` | `/2` values accepted |
| `evaluator_version` | `NOT NULL`, text, `length > 0` | `/2` accepted. No pairing enforced |
| Status and kind | Attempts `status IN (six values)`; `disposition_kind IN ('PROPOSED', 'NOT_ELIGIBLE', 'UNRESOLVED')` | Version 2 must use existing values; constrains section 7 |
| Disposition JSON checks | `$.format`, `$.generation_key`, `$.status`, `$.proposal_fingerprint`, `$.proposal`, `$.inputs_fingerprint`, `$.inputs.policy_set_fingerprint`, `$.inputs.policy_set.reference.version`, `$.request.policy_set.version`, `$.request.strategy`, `$.request.listing.symbol`, `$.request.listing.exchange_code`, `$.request.trading_date`, `$.inputs.candidate.source.*` (strategy, configuration version, listing, date, `decision_as_of`, decision, entry and snapshot fingerprints), `$.inputs.candidate.candidate_sequence`, `$.inputs.candidate.manifest_fingerprint`; proposal `$.format`, `$.generation_key` | Version 2 keeps every path (section 5) |
| Attempt JSON checks | `$.format`, `$.status`, `$.request.strategy`, `$.request.listing.*`, `$.request.trading_date`, `$.request.policy_set.version`, `$.generation_key`, `$.disposition_kind`, `$.assessment_fingerprint` | Kept |
| Foreign keys | Disposition to the EI-3C entry and EI-2B snapshot; attempt `disposition_key` to its disposition | Unchanged |
| Triggers | No `UPDATE`, `DELETE` or replacing `INSERT` on either table. The CANDIDATE source trigger reads `$.inputs.candidate.research_window.closes_at` and `$.request.research_configuration`, and requires a `SYNTHETIC` snapshot. The attempt-matches-disposition trigger requires the same scope and policy-set version, and the same kind when the assessment fingerprints are equal | Paths kept; a cross-version conflicting attempt has a different `eira1` |
| Migration convention | `CREATE … IF NOT EXISTS`; existing tables, rows, indexes and triggers are never altered; no schema-version table | No migration planned |

- **Finding.** By inspection, no existing constraint conflicts with the version-2 shape of section 5. The one restriction found is the status set, and section 7 respects it.
- **Status.** **EXPECTED, NOT VERIFIED.** Only slice 4's real mixed-version acceptance (PS-01 to PS-08) can verify it.
- **If a constraint fails.** The failure is reported and governed separately, never worked around, and no migration is run without its own approval.
- *Amended 2026-11-14 — status: **VERIFIED** (slice 4, infrastructure #66, merge `f71f8f7`).* Real mixed-version SQLite acceptance (PS-01 to PS-08) passed against the existing schema and adapter with **no migration and no production change**:
  - every row of the table above held for version-2 records as expected; no constraint failed;
  - `sqlite_master`, `PRAGMA user_version` and both tables' column definitions are identical before and after version-2 writes;
  - foreign-key provenance, the source trigger, the insert-only and no-replace triggers, `BEGIN IMMEDIATE` atomicity and authorized writes behave for version 2 as for version 1;
  - version-1 rows stay byte-identical, and version-1 and version-2 rows coexist and replay under their own labels, including in a fresh process.

  **Limitations.** Windows local evidence only; no CI result is recorded. The `evaluator_version` row is unchanged: SQLite does not enforce label-format pairing, and the pairing trigger stays deferred (N6). Evidence: [acceptance record, section 9](../product/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md#9-sqlite-compatibility).

## 12. Acceptance matrix

**Every scenario is PLANNED.** None is executed, and existing version-1 tests are not evidence of version-2 behaviour.

*Note, 2026-11-14.* The Status columns below record the plan of 2026-11-13 and are not rewritten. The executed statuses, with evidence for each of the 52 scenarios, are in the [EI-4C Engineering Acceptance Record, section 10](../product/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md#10-acceptance-evidence-matrix): 40 VERIFIED, 8 VERIFIED WITH LIMITATION, 1 NOT VERIFIED (VC-04), 3 PLANNED (VC-21 to VC-23, EI-5D).

**Columns.** Each scenario states the expected behaviour at each stage: decoding, historical verification, generation, recording and issuance eligibility.

**Layers:**

| Code | Layer |
| --- | --- |
| A | Application with test doubles |
| S | Infrastructure on real SQLite |
| I | EI-5D, with Application test doubles |

### 12.1 Cross-version compatibility (VC)

| ID | Scenario | Decoding | Historical verification | Generation | Recording | Issuance eligibility | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| VC-01 | Valid version-1 record | Decodes | A–C under `/1`, `as_of = t` | `/1` unchanged | As today | Refused: not issuable; preserved | A, S | 3a | PLANNED |
| VC-02 | Valid version-2 record | Decodes; pairs | A–C under `/2`, `as_of = b` | `/2`, case A | `COMMITTED` or `EQUIVALENT_COMMITTED` | Candidate only, subject to every EI-5D check | A, S | 3a, 3b | PLANNED |
| VC-03 | Version-1 record through the new verifier | Decodes | Dispatched to `/1`; result equals the pre-3a golden replay | — | — | Refused | A | 3a | PLANNED |
| VC-04 | Version-2 record met by legacy-only code | `Incompatible` (unknown label and `/2` format) | Not verified | — | `STORED_RECORD_INCOMPATIBLE` | Refused | A | 3a | PLANNED |
| VC-05 | Unknown evaluator label | `Incompatible` | Not verified | Selection refused | `STORED_RECORD_INCOMPATIBLE` | Refused | A, S | 3a | PLANNED |
| VC-06 | Missing or empty label | `Invalid` | Not verified | — | SQLite `NOT NULL` and `length > 0` | Refused | A, S | 3a | PLANNED |
| VC-07 | Label and formats mismatched (each invalid row of section 8) | `Invalid` | Not verified | — | No authorization | Refused | A | 3a | PLANNED |
| VC-08 | Correct version, wrong fingerprint | `Invalid` at level A | Not verified | — | `STORED_RECORD_INVALID` | Refused | A, S | 3a | PLANNED |
| VC-09 | Same generation key across versions | Both decode | Each under its own label | — | Store `CONFLICT`; recorder `EVALUATOR_VERSION_CONFLICT`; first kept; not adopted; competing attempt recorded | Only the stored record | A, S | 3a, 4 | PLANNED |
| VC-10 | Same `eirp1` under different assessments | Decodes | — | — | Same key, so VC-09; `UNIQUE` and no-replace refuse a second row | `eiris1` unaffected | A, S | 3a, 4 | PLANNED |
| VC-11 | Version-1 retry | — | — | Same inputs give the same documents | `EQUIVALENT_COMMITTED`; changed history evidence gives `CONTENT_CONFLICT` | — | A | 3a | PLANNED |
| VC-12 | Version-2 retry | — | — | Same view: same documents. After a competing issuance at `t`: other watermark. After a declaration beyond `b`: case C | `EQUIVALENT_COMMITTED`; or `CONTENT_CONFLICT` with the first kept; or attempt only | — | A | 3b | PLANNED |
| VC-13 | Version-1 and version-2 attempts for equivalent requests | Each pairs | — | — | Distinct `eirat1`, keyless and keyed; each idempotent within its version | — | A, S | 3b, 4 | PLANNED |
| VC-14 | Mixed-version persistence | Both decode | — | — | Both rows coexist under distinct keys | — | S | 4 | PLANNED |
| VC-15 | Mixed-version restart and replay | Both decode | Each verified under its own label in a fresh process | — | — | — | S | 4 | PLANNED |
| VC-16 | Wrong-version authorized write | Mismatched record cannot construct | — | — | No authorization; a swapped label or content is refused by the EI4-D11 binding | — | A, S | 3a, 4 | PLANNED |
| VC-17 | Version 2, history complete exactly through `b` | — | Frozen `as_of = complete_through = b` replays | Case A: `PROPOSED` iff every other check passes | `COMMITTED` | Necessary, not sufficient | A | 3b | PLANNED |
| VC-18 | Version 2, history incomplete before `b` | — | — | Case B: `RECOMMENDATION_HISTORY_INCOMPLETE`, keyless | Attempt `/2` only | — | A | 3b | PLANNED |
| VC-19 | Version 2, history declared beyond `b` | — | — | Case C: `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`, status `DEPENDENCY_UNAVAILABLE`; no `PROPOSED`; keyless. Not `RECOMMENDATION_HISTORY_INCOMPLETE`. A version-1 generation over the same history is unaffected | Keyless attempt `/2` only; no disposition; key not consumed; an identical re-run is idempotent | Refused; no issuance record; no declaration reversed | A | 3b | PLANNED |
| VC-20 | Missing authoritative history | — | — | Case D: `OPEN_RECOMMENDATION_STATE_UNAVAILABLE`, keyless | Attempt `/2` only | — | A | 3b | PLANNED |
| VC-21 | Unverifiable watermark | — | Frozen `W` replays from the frozen evidence alone | Unaffected | Unaffected | Refused when the authority's view at `W` does not reproduce the evidence | I | 6 | PLANNED |
| VC-22 | Issuing a version-1 proposal | Decodes | Verified | — | — | Refused as not issuable; preserved; no write | I | 6 | PLANNED |
| VC-23 | Competing issuance proposals | — | — | Same `t`: neither sees the other. Later `t`: sees the earlier as open (EI4-D8), so `EVALUATION_UNRESOLVED` | — | Same strategy, listing and date: `SESSION_STRATEGY_ALREADY_ISSUED`. A declaration race: `BACKDATED_ISSUANCE` inside the authority transaction | I | 6 | PLANNED |
| VC-24 | Unknown or unapproved document format (e.g. `…-assessment/3`, `…-generation-inputs/9`) | `Incompatible` within a known family; `Invalid` otherwise | Not verified | — | `STORED_RECORD_INCOMPATIBLE` or `STORED_RECORD_INVALID` | Refused | A | 3a | PLANNED |
| VC-25 | Replay after a later Core implementation change | Decodes | `REPLAY_MISMATCH` if formation output changes; never rehashed or migrated; detected by the golden vectors | — | — | Refused | A | 1, 3a | PLANNED |

### 12.2 Version-1 golden freeze (GV)

Generated from the **unchanged** existing implementation, and recorded in tests as exact canonical JSON text and exact fingerprints. No version-2 golden is made by editing a version-1 fixture.

| ID | Scenario | Layer | Slice | Status |
| --- | --- | --- | --- | --- |
| GV-01 | `eiri1`: canonical `generation-inputs/1` text and fingerprint for the existing PROPOSED synthetic scenario | A | 1 | PLANNED |
| GV-02 | `eira1`: canonical `assessment/1` text and fingerprint for PROPOSED, NOT_ELIGIBLE and UNRESOLVED dispositions | A | 1 | PLANNED |
| GV-03 | `eirp1`: canonical `proposal/1` text and fingerprint | A | 1 | PLANNED |
| GV-04 | `eirat1`: keyed attempt, and keyless attempt (`RECOMMENDATION_HISTORY_INCOMPLETE`) | A | 1 | PLANNED |
| GV-05 | `eirk1` and `eirs1` equal the existing golden constants; the evaluator label and every `/1` format string are pinned | A | 1 | PLANNED |
| GV-06 | The pinned version-1 documents replay A–C through the verifier | A | 1 | PLANNED |
| GV-07 | A pinned version-1 persisted row reloads and verifies on real SQLite | S | 2 | PLANNED |

### 12.3 Test hygiene (TH)

| ID | Scenario | Layer | Slice | Status |
| --- | --- | --- | --- | --- |
| TH-01 | Application "future evaluator" constants no longer use `northstar.equity-recommendation-evaluator/2`; every unknown-version refusal assertion is kept | A | 1 | PLANNED |
| TH-02 | Infrastructure constants likewise | S | 2 | PLANNED |
| TH-03 | The replacement label is shown by repository search to be unused and outside the governed set | A, S | 1, 2 | PLANNED |

### 12.4 History-boundary arithmetic and canonical form (HB)

| ID | Scenario | Layer | Slice | Status |
| --- | --- | --- | --- | --- |
| HB-01 | `t` with no fraction: `b` ends `.999999Z` | A | 3b | PLANNED |
| HB-02 | `t` ending `.000001Z`: `b` has no fraction | A | 3b | PLANNED |
| HB-03 | Second and minute rollover | A | 3b | PLANNED |
| HB-04 | Hour rollover | A | 3b | PLANNED |
| HB-05 | UTC midnight: `b` on the previous UTC date | A | 3b | PLANNED |
| HB-06 | An IST-offset `t` canonicalizes to `Z` with the same `b`; the IST-midnight row of section 6 | A | 3b | PLANNED |
| HB-07 | `b`, `history_boundary` and `generation-inputs/2` serialize canonically and round-trip through decoding to equal values | A | 3b | PLANNED |
| HB-08 | Deterministic replay: equal inputs give equal `eiri1` and `eira1`, including in a fresh process | A | 3b | PLANNED |
| HB-09 | Inputs `/2` refuse `as_of != b`, `complete_through != b`, and a missing or different `history_boundary`; inputs `/1` still refuse `as_of != t` | A | 3b | PLANNED |

### 12.5 Mixed-version persistence (PS)

| ID | Scenario | Layer | Slice | Status |
| --- | --- | --- | --- | --- |
| PS-01 | Version-2 PROPOSED, NOT_ELIGIBLE and UNRESOLVED rows insert under the existing schema | S | 4 | PLANNED |
| PS-02 | Version-2 keyed and keyless attempts insert; a version-2 conflicting attempt under a version-1 disposition inserts | S | 4 | PLANNED |
| PS-03 | `UPDATE`, `DELETE` and replacing `INSERT` are refused for version-2 rows | S | 4 | PLANNED |
| PS-04 | A mixed file reloads and verifies both versions after restart, in a fresh process | S | 4 | PLANNED |
| PS-05 | Concurrent version-1 and version-2 writers on one key, in separate processes: one commits, the other gets `EVALUATOR_VERSION_CONFLICT` | S | 4 | PLANNED |
| PS-06 | An unconfirmed commit that reconciles to the other version's record is `EVALUATOR_VERSION_CONFLICT`, never equivalent | S | 4 | PLANNED |
| PS-07 | The EI4-D11 boundary and its architecture scans are unchanged; a wrong-version write is refused before a connection opens | S | 4 | PLANNED |
| PS-08 | A database initialized by the current schema accepts version-2 rows with no table altered and no migration | S | 4 | PLANNED |

### 12.6 Scenario counts

| Group | Count | Status |
| --- | --- | --- |
| VC | 25 | PLANNED |
| GV | 7 | PLANNED |
| TH | 3 | PLANNED |
| HB | 9 | PLANNED |
| PS | 8 | PLANNED |
| **Total** | **52** | **PLANNED** |

These are additional to the 119 scenarios of the [EI-5 design](Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md#12-acceptance-matrix). VC-21 to VC-23 restate EI-5D obligations (IS-16, BD-02, BD-03) from the compatibility side and are owned by EI-5D.

## 13. Implementation sequence

| Slice | Repository | Scope | Prerequisites | Gate | Excluded |
| --- | --- | --- | --- | --- | --- |
| 0 | northstar-docs | ADR-025, register amendments, this design, cross-references | — | Owner review and merge | Any code |
| 1 | northstar-application | Version-1 freeze: GV-01 to GV-06; TH-01, TH-03 | Slice 0 merged | Goldens computed from unchanged code; zero production diff; full suite green | Version 2 |
| 2 | northstar-infrastructure | GV-07; TH-02, TH-03 | Slice 0 merged | Zero production diff; full suite green | Schema, version 2 |
| 3a | northstar-application | Version table; `/2` formats and prefix entries; decoder pairing; version-aware records; verifier dispatch; VC-01 to VC-11, VC-16, VC-24 and VC-25 | Slices 1 **and** 2 merged | Version-1 goldens unchanged; Application suite green; **the full Infrastructure suite run against the slice-3a head** (see below) | Version-2 generation, issuance |
| 3b | northstar-application | Keyword-only evaluator selection, defaulting to `/1`; inputs `/2` at `b` with `history_boundary`; N10 cases A to D; assessments and attempts `/2`; VC-12, VC-13 and VC-17 to VC-20, including the approved case-C reason `RECOMMENDATION_HISTORY_BOUNDARY_EXCEEDED`; HB-01 to HB-09 | 3a merged | As above | Issuance |
| 4 | northstar-infrastructure | PS-01 to PS-08; VC-09, VC-10 and VC-13 to VC-16 on real SQLite | 3b merged | Full suite green; no schema migration unless an independently governed blocker is found | Migrations, the pairing trigger |
| 5 | northstar-docs | Acceptance record: evidence, limitations, CI evidence where available | Slice 4 merged | Owner acceptance | — |
| 6 | northstar-application | EI-5D issuance; VC-21 to VC-23 | Slice 5 accepted | Its own milestone authorization | — |

**Cross-repository dependency.** Infrastructure depends on Application as an editable sibling path (`northstar-infrastructure/pyproject.toml`), so its tests run against the local Application checkout.

**Test-label inventory.** This inventory is indicative; slices 1 and 2 must re-inventory by search. These existing tests use `northstar.equity-recommendation-evaluator/2` as an unsupported or "second" label, often with `SUPPORTED_EQUITY_RECOMMENDATION_EVALUATOR_VERSIONS` monkeypatched:

| Repository | File | Uses |
| --- | --- | --- |
| Application | `tests/application_services/test_record_equity_recommendation_generation.py` | `_FUTURE`, `_second_label`, `_current_and_relabelled`, the `test_evaluator_*` group |
| Infrastructure | `tests/equity_recommendation_generation_fixtures.py` | `FUTURE_EVALUATOR` |
| Infrastructure | `tests/test_sqlite_equity_recommendation_generation.py` | The `second_label` fixture, and the "another evaluator" tests `test_o_…`, `test_o_a_…`, `test_p_…` |
| Infrastructure | `tests/test_equity_recommendation_generation_acceptance_faults.py` | `_FUTURE_EVALUATOR` |

**Pre-implementation detail.** Some of these tests construct **identical version-1 documents under a second supported label**. Slice 3a's pairing makes that premise impossible to construct, so it cannot be kept by renaming the label alone. Therefore:

- slice 2 re-expresses each such test so it holds both before and after slice 3a where possible (for example, an unknown-label row placed below the adapter must load as `Incompatible`);
- slice 2 lists each premise that slice 3a deliberately removes;
- slice 4 replaces those with pairing-refusal and real cross-version tests;
- slice 3a does not merge while the Infrastructure suite fails against it. If that is unavoidable, a paired Infrastructure PR is prepared before slice 3a merges and merged immediately after.

This is a sequencing constraint, not an architecture change.

*Amended 2026-11-14 (owner approved): combined slices 3a + 3b.* The table above remains the sequence approved on 2026-11-13. Slices 3a and 3b were delivered as one Application PR, **"Combined Slices 3a + 3b — Closed Two-Version Verification and Version-2 Generation"** (application #95, merge `d908674`). Only packaging changed:

- both rows' scope, gates and acceptance scenarios stayed binding, and the gate "the full Infrastructure suite run against the slice-3a head" was applied to the combined head (3,267 passed, 27 skipped, local);
- slices 1 (#94) and 2 (#64) merged first;
- the paired Infrastructure PR was not needed after the Application merge: the evaluator-set correction (#65) merged **before** #95 and accepts exactly `{/1}` or `{/1, /2}`;
- slice 4 (#66) followed the combined merge.

The owner record is the [EI-5 Decision Register, section 14.1](../product/Equity-Intraday-V1-EI5-Decision-Register.md#141-n9-sequencing-amendment-combined-slices-3a--3b-owner-approved-2026-11-14). The test-label inventory above was resolved by slices 1 and 2 (TH-01 to TH-03), and slice 4 replaced the removed premises with pairing-refusal and real cross-version tests.

## 14. Deferred work and limitations

**Resolved, 2026-11-13.** The N10 case-C refusal classification is owner approved (section 7); it no longer blocks slice 3b.

**Deferred.**
- **The SQLite pairing trigger (N6).**

**Limitations.** These are recorded, not resolved:

1. A version-1 disposition permanently occupies its generation key.
2. A distinct generation identity needs a different approved policy-set version.
3. Under EI4-D8, later same-session issuances may be blocked by conservative open-state treatment.
4. Replay depends on the version-1 golden vectors and on stable Core formation.
5. Re-running generation after a competing issuance changes the watermark (`CONTENT_CONFLICT`); after a declaration beyond `b`, it is refused (case C).
6. Future monitoring must respect monotone completeness and no-backdating, and issue in ascending modeled-issuance order.
7. Prefixes alone do not identify canonical format versions.
8. The microsecond-resolution rules remain binding.
9. N10 at generation does not replace the issuance authority's atomic check.
10. Existing tests use the now-approved `/2` label as an unsupported future version.
11. EI-4 formal closure remains pending independent CI.
12. O-2, O-3, EI3-D10, EI0-D21 and EI0-D19 remain deferred.
13. ACTUAL issuance remains unauthorized.
14. No production recommendation-history authority exists.

**Policies versus blockers.**
- Items 3, 12, 13 and 14 are deferred **policies or products**.
- The slice 2 and 3a test sequencing (section 13) is the only remaining **pre-implementation constraint**.
- Nothing listed blocks the architecture.

*Update, 2026-11-14.* The list above is kept as recorded.
- **Resolved:** limitation 10 (slices 1 and 2 replaced the `/2` test label), and the pre-implementation sequencing constraint (met; section 13).
- **Added by the acceptance record:** I08 is an equivalent mutant, not killed, and depends on the Application's pairing and construction invariants; the cross-process SQLite evidence is Windows-only; VC-04 is NOT VERIFIED, accepted by the owner as a documented limitation on 2026-11-14, so rollback or downgrade to a pre-3a Application against stored evaluator-v2 records is not authorized without separate compatibility verification. See the [acceptance record, section 13](../product/Equity-Intraday-V1-EI4C-Versioned-History-Compatibility-Engineering-Acceptance-Record.md#13-remaining-limitations).
- **Still deferred:** the SQLite pairing trigger (N6). Limitations 1 to 9 and 11 to 14 remain.

## 15. Related documents

- [ADR-025: Equity Intraday V1 Versioned Evaluation and History Compatibility](../adr/ADR-025-Equity-Intraday-V1-Versioned-Evaluation-and-History-Compatibility.md)
- [EI-5 Decision Register](../product/Equity-Intraday-V1-EI5-Decision-Register.md), sections 12.2 and 13
- [EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md), EI4-D9-I
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [ADR-023: Equity Intraday V1 Mandatory Operational Generation-Write Boundary Enforcement](../adr/ADR-023-Equity-Intraday-V1-Mandatory-Operational-Generation-Write-Boundary-Enforcement.md)
- [ADR-024: Equity Intraday V1 Recommendation Issuance and Lifecycle Architecture](../adr/ADR-024-Equity-Intraday-V1-Recommendation-Issuance-and-Lifecycle-Architecture.md)
- [EI-5 Issuance and Lifecycle Design](Equity-Intraday-V1-EI5-Issuance-and-Lifecycle-Design-v1.0.md)
- [EI4-D11 Mandatory Operational Generation-Write Boundary Enforcement Design](Equity-Intraday-V1-EI4-D11-Operational-Generation-Write-Boundary-Enforcement-Design.md)
