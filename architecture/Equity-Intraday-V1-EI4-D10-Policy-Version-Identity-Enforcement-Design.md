# EI4-D10 — Immutable Policy-Version Identity Enforcement Design

Status at proposal (historical): **PROPOSED — ENGINEERING DESIGN REVIEW PENDING** (2026-11-08).

> **Status update, 2026-11-08: APPROVED DESIGN.** The owner approved all nine design decisions, EI4-D10-DQ1 to EI4-D10-DQ9, as recommended in [section 15](#15-decisions-awaiting-approval), including the DQ6 and DQ9 clarifications ([section 17](#17-owner-approval-2026-11-08)). The approved architecture is recorded in [ADR-022](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md) and the [EI-4 Decision Register, section 11](../product/Equity-Intraday-V1-EI4-Decision-Register.md#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08).
>
> - EI4-D10 enforcement is **NOT YET IMPLEMENTED**, and its acceptance is **NOT YET ESTABLISHED**.
> - The text below is kept as proposed. Where it says the design is unapproved or pending, that is the status at proposal.

> **Status update, 2026-11-10: IMPLEMENTED; ENGINEERING ACCEPTANCE RECORDED.** The bullets above record the status before implementation.
>
> - **Implementation.** Slices S1 to S4 are implemented and merged.
> - **Acceptance.** Engineering acceptance, PASS WITH DOCUMENTED LIMITATIONS, is recorded in the [EI4-D10 Engineering Acceptance Record](../product/Equity-Intraday-V1-EI4-D10-Policy-Version-Identity-Engineering-Acceptance-Record.md). It traces every scenario of the [acceptance matrix](#13-acceptance-matrix) to its evidence.
> - **Unchanged.** The proposal, alternatives and approvals below are unchanged. EI4-D11 is still not implemented.

- **Governance basis:** EI4-D10, Immutable Policy-Version Identity, **OWNER APPROVED, 2026-11-08** ([EI-4 Decision Register, section 10.2](../product/Equity-Intraday-V1-EI4-Decision-Register.md#102-ei4-d10----immutable-policy-version-identity-aa-1)).
- **The approved rule and this design are different things.** The rule is approved. This enforcement design is **not** approved. Every contract, table, outcome and slice below is a proposal. None exists, and nothing here authorizes implementation.
- Nothing here changes code, schema, tests, fixtures, the `eirs1` or `eirk1` formats, any canonical document version or any stored record.
- Design questions that need an owner decision are listed in [section 15](#15-decisions-awaiting-approval) as EI4-D10-DQ1 to EI4-D10-DQ9. Engineering choices that existing standards already cover are not raised as owner decisions.

**Location.** `architecture/` holds design specifications. This file is named without a version suffix because it is a proposal, not a frozen specification, following the precedent of the [Reviewed-Candle Equivalence Design](Indian-Futures-Reviewed-Candle-Equivalence-Design.md). The README's Architecture row already links the folder, so no index changes.

Source paths are relative to each repository root. Code was read at:

| Repository | Branch | Commit |
| --- | --- | --- |
| northstar-core | `develop` | `bc081a2` |
| northstar-application | `develop` | `bed91de` |
| northstar-infrastructure | `develop` | `5dda518` |
| northstar-docs | `develop` | `aa56118` |

## 1. Executive summary

**The problem.** Today a synthetic policy version label is only a name that the injected reference source answers. Nothing records which canonical definition a label was first published with.

- The composite policy-set fingerprint (`eirs1`) binds labels **and** content, and the generation key (`eirk1`) binds that fingerprint.
- So a reused label with new content yields a *new* key, not a conflict, and a second disposition can coexist for the same candidate.
- EI-4E demonstrated this ([AA-1](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#141-aa-1--policy-version-label-governance)).

**Proposed enforcement:**

1. An **immutable synthetic policy catalog**: an Application port, implemented by an Infrastructure SQLite adapter in the existing evidence database.
   - It binds `(namespace, policy type, version label)` to exactly one canonical definition document and its fingerprint.
   - It binds each composite policy-set label to exactly one set of component labels and one `eirs1` fingerprint.
2. **Explicit, atomic publication** through an Application use case:
   - one `BEGIN IMMEDIATE` transaction;
   - insert-only rows, with uniqueness enforced by the database;
   - idempotent equal re-publication, and fail-closed conflicting re-publication;
   - commit-unconfirmed reconciliation modelled on EI4-D9-G.
3. **Resolution from the catalog.** A catalog-backed reference source answers only published, fingerprint-verified definitions, and generation also checks the composite set binding.
4. **A recording gate.** The EI-4D-A recorder checks a disposition's frozen policy content against the catalog before writing. Under EI4-D11-A that recorder is the mandatory operational write boundary, so no disposition commits under a conflicting or unpublished label, whoever wired the generator.
5. **Untouched replay.** Historical replay never consults the catalog. A separate, read-only policy-binding audit classifies older records without rewriting them.

`eirs1`, `eirk1`, every existing canonical document version, the evaluator version and every committed disposition stay unchanged.

## 2. Existing architecture

### 2.1 The current flow

| Step | Code | What it does |
| --- | --- | --- |
| 1. Policy reference request | `EquitySyntheticPolicySetReference` (application `application_services/equity_recommendation_generation_evidence.py`), in `EquityTradeRecommendationRequest.policy_set` | Names a composite set label and five component labels, each an `EquitySyntheticVersion` |
| 2. Reference resolution | `GenerateEquityTradeRecommendationUseCase._inputs`, inner `versioned()` (application `application_services/generate_equity_trade_recommendation.py`) | Calls `EquitySyntheticRecommendationReferenceSource` (application `ports/equity_recommendation_references.py`) once per component; maps outcomes (below) |
| 3. Validated definitions | Core `EquitySyntheticGenerationPolicy`, `EquitySyntheticRiskPolicy`, `EquityPriceIncrement` (core `recommendations/equity/synthetic_inputs.py`), `EquitySyntheticCostSchedule` (`cost_schedule.py`); Application `EquitySyntheticEntryTimingPolicy` (ports module) | Construction validates each definition. `document()` returns its canonical content, including its own version label; Decimals are canonicalized by Core `_exact.text`, so `1.50` and `1.5` are one value |
| 4. Composite policy set | `EquityResolvedSyntheticPolicySet` (evidence module) | Requires each component's `version` to equal the label the reference names. Its `document()` (format `northstar.equity-recommendation-policy-set/1`) holds the reference and all five component documents |
| 5. `eirs1` | `EquityResolvedSyntheticPolicySet.fingerprint`, via `equity_recommendation_fingerprint` and `canonical_equity_recommendation_json` | SHA-256 over the canonical policy-set document, prefixed `eirs1-sha256:` (`EQUITY_RECOMMENDATION_DOCUMENT_PREFIXES`) |
| 6. `eirk1` | `equity_recommendation_generation_key` | Length-prefixed SHA-256 over namespace `northstar.equity-recommendation-generation-key/1`, strategy, research configuration version, listing, date, CANDIDATE entry fingerprint, **policy-set label** and **`eirs1`** |
| 7. Frozen inputs | `EquityRecommendationInputs` | Holds the resolved policy set (full content), eligibility, venue-day and issued-history evidence, and derives the key and an `eiri1` fingerprint |
| 8. Assessment | `assess_equity_recommendation_inputs` | Pure; replay re-runs it from stored inputs |
| 9. Recording | `RecordEquityRecommendationGenerationUseCase` (`record_equity_recommendation_generation.py`) → `EquityRecommendationGenerationStore` → `SQLiteEquityRecommendationGenerationStore` (infrastructure `persistence/sqlite_equity_recommendation_generation.py`) | Verifies source and replay (`EquityRecommendationGenerationVerifier`), then writes the disposition and attempt atomically; conflicts by key |

`versioned()` maps the reference source's answers as follows:

| Source answer | Reason | Status |
| --- | --- | --- |
| `EquitySyntheticReferenceUnavailableError` | `*_UNAVAILABLE` | `DEPENDENCY_UNAVAILABLE` |
| `None` | `*_UNAVAILABLE` ("No … is held") | `DEPENDENCY_UNAVAILABLE` |
| An object of the wrong type, or with `value.version != version` | `POLICY_REFERENCE_MISMATCH` | `DEPENDENCY_UNAVAILABLE` |

Any other exception propagates as a programming error.

**Where policy definitions come from today.** No production adapter implements `EquitySyntheticRecommendationReferenceSource` in any repository. The only implementations are test fixtures: the `References` classes in `tests/equity_recommendation_generation_fixtures.py` and `tests/equity_recommendation_acceptance_fixtures.py`, and the Application test fakes. All are in-memory dictionaries keyed by label. There is no policy registry to extend, and none is assumed.

**What persists policy content today.** Only committed dispositions do. The EI-4D-B `equity_recommendation_generation_dispositions` table stores:

- `policy_set_version` and `policy_set_fingerprint` as columns, checked against the assessment document by a CHECK constraint;
- the full policy-set document, inside `assessment_document` at `inputs.policy_set`.

Component labels exist only inside that JSON. The attempts table stores `policy_set_version`. Its `status` column is constrained by CHECK to the six existing statuses.

### 2.2 Four identities that must not be conflated

| Identity | Form | Bound by | Enforced today |
| --- | --- | --- | --- |
| Version-label identity | `EquitySyntheticVersion`: case-sensitive, never trimmed, begins `synthetic-` | Nothing durable | Only "the answer names the asked label" (`POLICY_REFERENCE_MISMATCH`) |
| Canonical content identity | A component's canonical `document()` | Nothing on its own; no per-component fingerprint exists | Core construction validates values only. The Core module docstring states: "Core … cannot detect a version reused with different content; that is a responsibility of later Application persistence." |
| Composite policy-set identity | `eirs1` over set label + component labels + component contents | `EquityResolvedSyntheticPolicySet` | Recomputed on every decode; stored in each disposition |
| Generation-record identity | `eirk1` over candidate + set label + `eirs1` | `EquityRecommendationInputs` | Database PRIMARY KEY; conflicts by key (EI4-D9-D) |

### 2.3 Where label reuse slips through

- Each generation asks the injected source for a label and accepts whatever definition it answers, as long as the answer names that label.
- `eirs1` then folds the new content in, and `eirk1` folds in `eirs1`. A reused label with new content therefore produces a different generation key.
- EI4-D9-D's same-key conflict rule, which is enforced, is never reached.
- What is verified today:
  - documents and fingerprints, on construction and decode;
  - source evidence and replay, in the verifier;
  - per-key uniqueness, in the database.
- No boundary checks label-to-content uniqueness over time. It is neither verified nor enforced.

## 3. Demonstrated EI-4E finding

`test_e12_c_a_reused_version_label_with_changed_content_is_another_identity` (infrastructure `tests/test_equity_recommendation_generation_acceptance.py`):

- the label `synthetic-ei4e-timing/1` is redefined from a 150-second to a 240-second latency;
- every version label in the request is identical;
- `eirs1` differs and the generation key differs;
- a second disposition commits, and the first is untouched.

[EI-4E record, section 14.1](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#141-aa-1--policy-version-label-governance) records the finding, which the owner approved as EI4-D10. That test asserts the **current** behaviour. Under this design its expected outcome would change: see [section 12](#12-proposed-implementation-slices), slice S4.

## 4. Approved invariant

From EI4-D10, approved 2026-11-08 and not restated with different meaning:

- Within a defined policy namespace and policy type, a published policy-version label identifies exactly one immutable canonical policy definition.
- Conflicting reuse fails closed as a policy-reference integrity conflict.
- A legitimate change requires a new label.
- `eirs1`, `eirk1`, existing canonical document versions, committed dispositions and historical policy evidence are preserved.

The six enforcement requirements of EI4-D10 map to this design as follows:

| EI4-D10 requirement | Design section |
| --- | --- |
| 1. Define and respect the namespace and policy type | [5.1](#51-namespace-and-policy-type) |
| 2. Bind each published label to one immutable definition | [5.2](#52-canonical-definition-binding), [7](#7-publication-and-lookup-flows) |
| 3. Reject conflicting reuse before it becomes accepted evidence | [7](#7-publication-and-lookup-flows), [8](#8-atomic-registration-and-concurrency) |
| 4. Fail closed on incompatible historical or current references | [9](#9-historical-compatibility), [10](#10-failure-classification) |
| 5. Prevent conflicting concurrent registration | [8](#8-atomic-registration-and-concurrency) |
| 6. Preserve historical evidence and immutable records | [9](#9-historical-compatibility) |

## 5. Proposed architecture

### 5.1 Namespace and policy type

**A. Namespace model.** No namespace exists in code today: the product scope, NSE synthetic Equity Intraday V1, is implicit. The labels' `synthetic-` prefix (EI3-D5) separates synthetic from approved values, but it is a value convention, not a namespace.

Proposal (**EI4-D10-DQ1**): one explicit namespace value, `northstar.equity-intraday-v1.synthetic`, stored in every binding.

- It is a closed constant in the Application, not caller-supplied text, so a caller cannot mint namespaces to escape a conflict.
- A future approved-policy namespace, after EI0-D10 to EI0-D13, would be a new constant by a new decision.

**B. Policy type.** The kind of definition a label names. Proposed closed set (**EI4-D10-DQ2**):

| Policy type | Definition type | Origin |
| --- | --- | --- |
| `GENERATION_POLICY` | `EquitySyntheticGenerationPolicy` | Core |
| `RISK_POLICY` | `EquitySyntheticRiskPolicy`, with model capital | Core |
| `COST_SCHEDULE` | `EquitySyntheticCostSchedule` | Core |
| `PRICE_INCREMENT` | `EquityPriceIncrement` | Core |
| `TIMING_POLICY` | `EquitySyntheticEntryTimingPolicy`, including the issuance latency | Application |
| `POLICY_SET` | A composite set label and its five component labels | Application |

Not in scope, with reasons:

- **Listing eligibility.** `EquitySyntheticListingEligibility.version` labels a per-listing, per-date evidence reference, and EI4-D9-B classifies eligibility as content, not identity.
- **Research configuration version.** EI-3C already governs it: a stored timeline with another configuration raises `EquityIntradayResearchTimelineConfigurationConflictError`, giving `RESEARCH_CONFIGURATION_MISMATCH`.
- **Venue-day and issued-history references.** These are evidence, not policies.

**C. Fields reused.**

- `EquitySyntheticVersion` as the label type, with its existing validation.
- The component `document()` methods as canonical content.
- `EquitySyntheticPolicySetReference.document()` for set composition.
- `canonical_equity_recommendation_json` and `equity_recommendation_fingerprint` for encoding and hashing.

**D. Fields missing.** Three things do not exist yet:

- a namespace value;
- a policy-type value;
- a per-component canonical fingerprint. Component documents have no `format` field, so `equity_recommendation_fingerprint` cannot hash them directly.

**E. Canonicalization.** The binding key is the exact tuple `(namespace, policy_type, label)`. The definition is canonical JSON (sorted keys, no whitespace, exact decimal text, no floats) through the existing encoder.

**F. Case.** Labels are **case-sensitive**, as `EquitySyntheticVersion` is today. `synthetic-x/1` and `synthetic-X/1` are distinct labels. Case-folding is not proposed: it would reinterpret existing labels, and nothing requires it.

**G. Whitespace.** Never trimmed. `EquitySyntheticVersion` already refuses surrounding whitespace, and the catalog schema would re-check it (`version_label = trim(version_label)`). Interior characters are kept exactly.

**H. Invalid, missing or ambiguous labels.**

- An invalid label cannot be constructed: Core raises `InvalidEquitySyntheticVersionError`, and publication refuses it.
- A missing label means "not published": the source answers None, and generation reports the existing `*_UNAVAILABLE` reason.
- Ambiguity is impossible by key construction: the full tuple is the primary key.

**I. Registering components and sets.** Components are published first, or in the same transaction as the set. A `POLICY_SET` binding records:

- its five component labels;
- the `eirs1` fingerprint of the resolved set.

Its insertion is refused unless all five component bindings exist under the same namespace (section 8).

**J. The same label under two types.** These are two independent bindings. `synthetic-x/1` as a generation policy and as a risk policy do not conflict. Uniqueness is per `(namespace, policy_type, label)`, never global across types: a global constraint would refuse legitimate labels for no invariant.

**Alternatives considered for the namespace and type model:**

- **Implicit namespace**, where the catalog file is the namespace. Rejected: it cannot be verified from a binding document alone, and it collides if two catalogs are ever merged.
- **Per-exchange namespace.** Rejected for now: only the timing policy carries an exchange code.
- **Label-only uniqueness**, with no type. Rejected: it adds an unneeded global constraint.

### 5.2 Canonical definition binding

**Component binding document**, a new format proposed in an existing family style:

```json
{
  "format": "northstar.equity-synthetic-policy-definition/1",
  "namespace": "northstar.equity-intraday-v1.synthetic",
  "policy_type": "TIMING_POLICY",
  "version": "synthetic-ei4e-timing/1",
  "definition": { "...": "the component's existing canonical document(), unchanged" }
}
```

- It is fingerprinted by the existing `equity_recommendation_fingerprint`, after adding one entry to `EQUITY_RECOMMENDATION_DOCUMENT_PREFIXES`: format `…policy-definition/1` maps to prefix `eipd1-sha256:`. The prefix is unused today.
- It is encoded by the existing `canonical_equity_recommendation_json` and verified by `verify_equity_recommendation_document`. No new serialization scheme is introduced.
- `definition.version` must equal `version`. The document proves the namespace, type, label, canonical definition, fingerprint and format, and it re-verifies at any later time.

**Policy-set binding.** No new content format. The binding stores:

- the existing `EquitySyntheticPolicySetReference.document()`, the set label plus five component labels;
- the existing `eirs1` fingerprint of the resolved set.

It is verified by rebuilding `EquityResolvedSyntheticPolicySet` from the bound component definitions and re-deriving `eirs1`, which reuses `decode_equity_policy_set` (decoding module).

**Compatibility.**

- `eirs1`, `eirk1`, `eiri1`, `eira1`, `eirp1` and `eirat1` are untouched.
- The binding document is never embedded in frozen inputs or dispositions, so no assessment, proposal or attempt format and no evaluator version changes (**EI4-D10-DQ7** records the alternative of embedding it).

### 5.3 Components and responsibilities

| Component | Layer | Responsibility |
| --- | --- | --- |
| `EquitySyntheticPolicyCatalogStore` port (proposed) | Application | Load and insert-only publish bindings; typed errors |
| `PublishEquitySyntheticPoliciesUseCase` (proposed) | Application | Validate definitions; build and fingerprint binding documents; publish atomically; reconcile an unconfirmed commit |
| `CatalogEquitySyntheticRecommendationReferences` (proposed) | Application | Implements `EquitySyntheticRecommendationReferenceSource` over the catalog. Answers only published definitions, after re-verifying each binding fingerprint |
| Generation set check (proposed change) | Application, `GenerateEquityTradeRecommendationUseCase._inputs` | Requires the request's `EquitySyntheticPolicySetReference` to equal the published set binding |
| Recording gate (proposed change) | Application, `RecordEquityRecommendationGenerationUseCase` | Before writing a disposition, verifies its frozen component documents and set reference against the catalog |
| `AuditEquityRecommendationPolicyBindingsUseCase` (proposed) | Application | Read-only classification of stored dispositions against the catalog; never writes |
| `SQLiteEquitySyntheticPolicyCatalogStore` (proposed) | Infrastructure | Schema, insert-only triggers, `BEGIN IMMEDIATE` publication, typed errors. It verifies no SHA-256, as with EI-4D-B (ADR-021 section 10) |

Dependency direction is unchanged: Core ← Application ← Infrastructure. The Application owns the ports and records, and Infrastructure owns SQLite.

## 6. Alternatives

| Criterion | A. Application-only in-memory binding | B. SQLite registry used directly by Infrastructure | C. Catalog port and SQLite adapter, in the evidence database (**recommended**) | C′. As C, in a separate catalog file | D. Binding evidence embedded in frozen inputs | E. Implicit binding by first committed disposition |
| --- | --- | --- | --- | --- | --- | --- |
| Cross-process correctness | No: per-process memory | Yes | Yes: PRIMARY KEY plus `BEGIN IMMEDIATE` | Yes | Not alone: still needs an authority | Yes |
| Restart durability | No | Yes | Yes | Yes | Yes | Yes |
| Transaction atomicity | n/a | Yes | Yes: one transaction per publication | Yes | Changes record writes | Couples publication to recording |
| Historical replay | Unaffected | Unaffected | Unaffected: replay never reads it | Unaffected | Changes `eiri1` and `eira1` for new records | Unaffected |
| Dependency direction | Fine | **Violated**: the Application cannot enforce without a port | Fine | Fine | Fine | Fine |
| Fit with the existing SQLite authority | n/a | Same file | Same file; follows the EI-2B, EI-3C and EI-4D-B conventions | A second file to back up and keep consistent | n/a | Same file |
| Reference provenance | None | Durable | Durable, publication-sequenced | Durable | Per record | Defined by whichever generation ran first |
| Operational simplicity | Simplest | Medium | Medium | Medium, plus a second file | Complex | Simple, but implicit |
| Failure handling | Silent across restarts | Typed | Typed, mirrors EI4-D9-G | Typed | Format migration | Every read becomes a write |
| Migration risk | None | Additive tables | Additive tables; no change to existing tables | New file | **New document versions**; conflicts with "preserve existing canonical document versions" | None |
| EI-5 and EI-6 boundary | n/a | Unclear | Clean: policy catalog, not issuance | Clean | n/a | Couples records to policy authority |
| Verdict | Rejected: cannot hold across processes (ADR-021: "In-memory locks … do not hold across processes") | Rejected: wrong dependency direction | **Recommended** | Alternative for EI4-D10-DQ4 | Rejected for the initial slice; EI4-D10-DQ7 | Rejected: "first read defines truth" is what EI4-D10 forbids |

## 7. Publication and lookup flows

**Publication** is an explicit operator act, never a side effect of generation or of reading a historical policy:

```text
operator tooling or test fixture
  -> PublishEquitySyntheticPoliciesUseCase.execute(bundle: components + optional policy set)
       1. construct and validate each definition (Core and Application types)
       2. build binding documents; derive eipd1 (and eirs1 for a set)
       3. store.publish(bindings)                  -- one atomic transaction
       4. interpret: PUBLISHED | EQUIVALENT | CONFLICT
          | COMMIT_UNCONFIRMED -> reconcile (section 8)
```

**Lookup and generation:**

```text
GenerateEquityTradeRecommendationUseCase(references = CatalogEquitySyntheticRecommendationReferences, ...)
  _inputs:
    for each component label:
        resolver.load(namespace, type, label)
          -> None                      : existing *_UNAVAILABLE reason (not published)
          -> store unavailable         : existing *_UNAVAILABLE reason (source unavailable)
          -> binding fails to verify   : POLICY_REFERENCE_CONFLICT (proposed reason)
          -> verified definition       : used exactly as today
    set check: published POLICY_SET binding for request.policy_set.version
          -> None                      : POLICY_SET_UNAVAILABLE (proposed reason)
          -> components differ         : POLICY_REFERENCE_CONFLICT
    EquityResolvedSyntheticPolicySet  -> eirs1 must equal the set binding's eirs1, else POLICY_REFERENCE_CONFLICT
  assess_equity_recommendation_inputs : unchanged, pure

RecordEquityRecommendationGenerationUseCase.record(disposition, attempt)
  gate (before the existing source and replay verification):
    for each component in disposition.inputs.policy_set, and the set reference:
        binding absent      -> POLICY_REFERENCE_NOT_ESTABLISHED (proposed outcome); nothing written
        binding different   -> POLICY_REFERENCE_CONFLICT (proposed outcome); nothing written
        catalog unavailable -> POLICY_REFERENCE_UNAVAILABLE (proposed outcome); nothing written
  then the existing verification, write and reconcile, unchanged
```

**Lifecycle mapping:**

| Lifecycle step | Where it happens |
| --- | --- |
| 1. Supplied | Publication input |
| 2. Validated | Core and Application construction |
| 3. Canonical identity | `eipd1` and `eirs1` |
| 4. Binding established or loaded | Catalog store |
| 5. Identical re-publication | EQUIVALENT |
| 6. Conflicting re-publication | CONFLICT; nothing written |
| 7. Available for generation | Catalog-backed resolver |
| 8. Retained as historical evidence | The disposition's frozen policy-set content, unchanged as today |

Registration, resolution, validation, generation and recording stay distinct steps with distinct owners.

**Where the invariant lives:**

| Step | Role under this design |
| --- | --- |
| Publication | **Establishes** the binding, atomically |
| Resolution | **Uses** it: answers only published, verified content |
| Policy-set construction | **Not used.** It stays a pure value object, unable to know publication state |
| Generation assessment | **Not used.** It stays pure, so replay cannot depend on the catalog |
| Recording gate | **Guarantees** it, independent of how the generator was wired |

## 8. Atomic registration and concurrency

### 8.1 Candidate schema

Additive to the existing evidence database (**EI4-D10-DQ4**). No existing table is altered.

```sql
-- Proposal only.
CREATE TABLE equity_synthetic_policy_definitions (
    namespace              TEXT NOT NULL CHECK (namespace = 'northstar.equity-intraday-v1.synthetic'),
    policy_type            TEXT NOT NULL CHECK (policy_type IN ('GENERATION_POLICY','RISK_POLICY',
                               'COST_SCHEDULE','PRICE_INCREMENT','TIMING_POLICY')),
    version_label          TEXT NOT NULL CHECK (version_label GLOB 'synthetic-?*'
                               AND version_label = trim(version_label)),
    definition_format      TEXT NOT NULL,
    definition_fingerprint TEXT NOT NULL UNIQUE CHECK (definition_fingerprint GLOB 'eipd1-sha256:?*'),
    definition_document    TEXT NOT NULL CHECK (json_valid(definition_document)),
    publication_sequence   INTEGER NOT NULL UNIQUE CHECK (publication_sequence >= 1),
    PRIMARY KEY (namespace, policy_type, version_label),
    CHECK (json_extract(definition_document, '$.format') IS definition_format
       AND json_extract(definition_document, '$.namespace') IS namespace
       AND json_extract(definition_document, '$.policy_type') IS policy_type
       AND json_extract(definition_document, '$.version') IS version_label
       AND json_extract(definition_document, '$.definition.version') IS version_label)
);

CREATE TABLE equity_synthetic_policy_sets (
    namespace              TEXT NOT NULL,
    set_version_label      TEXT NOT NULL CHECK (set_version_label GLOB 'synthetic-?*'
                               AND set_version_label = trim(set_version_label)),
    reference_document     TEXT NOT NULL CHECK (json_valid(reference_document)),
    policy_set_fingerprint TEXT NOT NULL CHECK (policy_set_fingerprint GLOB 'eirs1-sha256:?*'),
    generation_policy_type TEXT NOT NULL CHECK (generation_policy_type = 'GENERATION_POLICY'),
    generation_version     TEXT NOT NULL,
    -- ... the same column pair for RISK_POLICY, COST_SCHEDULE, PRICE_INCREMENT and TIMING_POLICY
    publication_sequence   INTEGER NOT NULL UNIQUE CHECK (publication_sequence >= 1),
    PRIMARY KEY (namespace, set_version_label),
    FOREIGN KEY (namespace, generation_policy_type, generation_version)
        REFERENCES equity_synthetic_policy_definitions (namespace, policy_type, version_label)
    -- ... one foreign key per component
);
-- Triggers (EI-4D-B pattern): refuse UPDATE, DELETE and replacing inserts on both tables;
-- require publication_sequence = 1 + the highest sequence across both tables (EI-3C next-sequence pattern);
-- require reference_document's labels to equal the component columns.
```

- `publication_sequence` is a monotone recording order. It is not a clock: no wall-clock time is read or stored.
- Constant-valued type columns let real foreign keys bind a set to its components.

### 8.2 Transaction

The adapter's `publish(bindings)` runs one `BEGIN IMMEDIATE` transaction, which takes the write lock before reading:

1. For each binding, in a deterministic order (components, then the set): `SELECT` by primary key.
   - Absent: `INSERT`.
   - Present with identical format, fingerprint and document: equivalent; nothing written.
   - Present with anything different: **conflict**. Stop, and roll back the whole bundle. Nothing is written, and the stored binding is returned for the report.
2. One `COMMIT`.

Outcome: `COMMITTED` if anything was inserted, `EQUIVALENT` if all were already present, or `CONFLICT`.

Typed errors, mirroring the EI-4D-B adapter:

| Error | Meaning |
| --- | --- |
| `…CatalogUnavailableError` | Cannot open or read; nothing written |
| `…NotCommittedError` | Lock not obtained within the bounded busy timeout, or a row refused; rolled back |
| `…CommitUnconfirmedError` | `COMMIT` itself failed |
| `InvalidEquitySyntheticPolicyBindingError` | A stored row does not decode |
| `IncompatibleEquitySyntheticPolicyBindingError` | Unsupported binding format |

The store never retries, sleeps or reads a clock.

### 8.3 Scenarios

| Scenario | Behaviour |
| --- | --- |
| Two identical concurrent publications | Serialized by `BEGIN IMMEDIATE`. The first commits; the second reads equal rows: EQUIVALENT |
| Two conflicting concurrent publications | The first commits; the second reads a different row: CONFLICT, nothing written. If a raw insert ever raced past the read, the PRIMARY KEY refuses it, a refused row means NotCommitted, and nothing is written |
| Separate connections and separate processes | Same: correctness rests on the database's lock and constraints, never on a process-local lock |
| Failure before COMMIT, or a partial bundle | Rolled back: no partial publication. A set can never exist without its components (foreign keys) |
| COMMIT succeeds, acknowledgment lost | The Application reconciles (below) and reports EQUIVALENT_PUBLISHED, reconciled |
| COMMIT outcome uncertain | Reconciled by reload, never assumed rolled back |
| Restart after publication | Rows are durable and verified on every load |
| Lock held or authority unavailable | NotCommitted or Unavailable; reported and never retried by the store |

### 8.4 Commit-unconfirmed reconciliation

In the Application use case, modelled on EI4-D9-G and the EI-4D-A recorder. The use case reloads every binding in the bundle once, authoritatively:

| Reload finds | Result |
| --- | --- |
| All equal to the offered bindings | PUBLISHED or EQUIVALENT, reconciled |
| Any differing | CONFLICT: a competitor won |
| None present, after a successful reload | Bounded retry with the same bundle, at most `max_writes` (3, as `EQUITY_RECOMMENDATION_RECORD_WRITES`), then RETRY_LIMIT_REACHED |
| Some present and some absent | PUBLICATION_RECONCILIATION_UNRESOLVED. Impossible under one transaction, so it is reported, never repaired |
| Reload unavailable, or a row fails to decode | PUBLICATION_RECONCILIATION_UNRESOLVED; nothing is claimed |

There is no unbounded retry loop.

## 9. Historical compatibility

**Principles:**

- No historical record is rewritten, re-keyed, migrated or deleted.
- No binding history is invented: the catalog is never backfilled automatically from dispositions (**EI4-D10-DQ7**).
- Replay (EI4-D9-I, levels A to C) never consults the catalog. Policy-binding status is reported **beside** A to C by the read-only audit use case, never folded into them, so ADR-021 section 10's four levels stay as written.

**Classification per stored disposition and component.** The audit reads `inputs.policy_set` from the stored assessment.

| Class | Condition |
| --- | --- |
| Historically verified | A to C pass, and every component document and the set reference equal their catalog bindings |
| Historical binding not established | A to C pass, but at least one label has no binding |
| Historically incompatible | A to C pass, but a label is bound to different content: a binding conflict |
| Invalid canonical evidence | A fails: the existing `InvalidEquityRecommendationRecordError` or `IncompatibleEquityRecommendationRecordError` |

"Historically verified" is **not** a claim that the binding existed at generation time. Dispositions carry no generation timestamp or publication sequence, so for records written before enforcement the order cannot be proved, and the audit says so. After enforcement, the recording gate makes "published before recorded" true for every new disposition.

**Cases:**

| Case | Treatment |
| --- | --- |
| A. Records whose content matches a binding | Historically verified, with the order caveat above |
| B. Unknown binding history | Binding not established. Never called compliant |
| C. One label reused for different definitions across older records | The audit reports every label with more than one distinct canonical definition across stored dispositions. At most one can match a later explicit publication; the others are historically incompatible. None is changed |
| D. Missing publication evidence | Binding not established. Any **new** recording relying on it fails closed at the gate |
| E. Unsupported canonical versions | The existing Incompatible errors for records; `IncompatibleEquitySyntheticPolicyBindingError` for binding documents |
| F. Replay with the catalog unavailable | A to C unaffected. The audit reports POLICY_BINDING_UNAVAILABLE, not a verdict |
| G. Committed EI-4D dispositions | Immutable and untouched, with keys unchanged. No production generation records exist today: EI-4D has no composition root, and EI-4E used disposable test databases only |

## 10. Failure classification

| Situation | Generation (EI-4C reason → status) | Recording (EI-4D-A outcome) | Publication outcome | Category |
| --- | --- | --- | --- | --- |
| Catalog missing (no tables) or cannot be opened | existing `*_UNAVAILABLE` → `DEPENDENCY_UNAVAILABLE` | `POLICY_REFERENCE_UNAVAILABLE` (proposed) | `UNAVAILABLE` | Dependency unavailability |
| Catalog busy or locked | as above | as above | `NOT_COMMITTED` | Dependency unavailability |
| Label unknown (not published) | existing `*_UNAVAILABLE` ("not held"), or `POLICY_SET_UNAVAILABLE` (proposed) | `POLICY_REFERENCE_NOT_ESTABLISHED` (proposed) | n/a | Dependency unavailability, never a default |
| Label bound to other content | `POLICY_REFERENCE_CONFLICT` (proposed) → see EI4-D10-DQ6 | `POLICY_REFERENCE_CONFLICT` (proposed) | `CONFLICT` | **Policy-reference integrity failure** |
| Stored binding corrupted (fingerprint does not re-derive) | `POLICY_REFERENCE_CONFLICT` | `POLICY_REFERENCE_CONFLICT` | `InvalidEquitySyntheticPolicyBindingError` | Policy-reference integrity failure |
| Unsupported binding format | `POLICY_REFERENCE_CONFLICT` | `POLICY_REFERENCE_CONFLICT` | `Incompatible…` | Version incompatibility |
| Publication COMMIT uncertain | n/a | n/a | Reconciled (8.4) | Commit uncertainty |
| A source answering contradictory versions | existing `POLICY_REFERENCE_MISMATCH` (wrong label), or `POLICY_REFERENCE_CONFLICT` (right label, wrong content) | gate as above | n/a | Integrity failure |
| Valid policy, but no candidate | existing `NO_AUTHORITATIVE_CANDIDATE` → `NO_CANDIDATE`; the catalog is not consulted | attempt-only, as today | n/a | Not a policy failure |
| Market eligibility refusal | existing `LISTING_NOT_ELIGIBLE` and similar → `NOT_ELIGIBLE` | unchanged | n/a | **Eligibility rejection**, never confused with the above |
| Same key, different content | unchanged | existing `CONTENT_CONFLICT` | n/a | Generation-content conflict (EI4-D9-D), distinct from a policy conflict |

Further rules:

- **Placement in gathering.** `POLICY_REFERENCE_CONFLICT` and `POLICY_SET_UNAVAILABLE` are added in gathering only. Gathering failures have no inputs, so they are attempt-only. `assess_equity_recommendation_inputs` and `EQUITY_RECOMMENDATION_EVALUATOR_VERSION` do not change, and stored assessments replay exactly as before.
- **Reasons, not statuses.** New reasons need no schema change: reasons live in the attempt JSON. A new **status** would need the attempts table's `status IN (…)` CHECK changed, which means rebuilding an insert-only evidence table. That is why no new status is proposed (EI4-D10-DQ6).
- **Recording outcomes.** The proposed recording outcomes are Application enum values and are not persisted, so no schema is affected. Like the existing refusal `SOURCE_EVIDENCE_INVALID`, a gate refusal writes nothing and reports `attempt_recorded=False`.
- **No defaults.** Absence of evidence is never read as an empty catalog or a default definition.

## 11. Cross-repository responsibilities

| Repository | Responsibility under this design |
| --- | --- |
| northstar-core | **No change.** It keeps `EquitySyntheticVersion`, the definition types, `document()` and Decimal canonicalization as they are |
| northstar-application | Catalog port and binding records; the binding document format and `eipd1` prefix; the publication use case with reconciliation; the catalog-backed resolver; the generation set check and new reasons; the recording gate and outcomes; the audit use case. Unit tests with in-memory fakes |
| northstar-infrastructure | `SQLiteEquitySyntheticPolicyCatalogStore`: schema, triggers, transaction and typed errors, in the evidence database. SQLite and concurrency tests. Updated test fixtures that publish their synthetic policies |
| northstar-docs | Design review outcome; an architecture record if approved (EI4-D10-DQ8); acceptance record |
| northstar-api, northstar-web | **No change.** No publication endpoint or UI is proposed |

**Authority and trust boundary:**

- Northstar is a personal-use system, with no multi-user identity. Publication is an operator act through `PublishEquitySyntheticPoliciesUseCase`, invoked by local tooling or test setup.
- Generation callers receive only the resolver, which has no publish path. They cannot change a published definition: the store is insert-only and conflicts are refused.
- **Trust assumption, not enforceable today (EI4-D10-DQ9).** Anyone with write access to the SQLite file could insert a *new* binding directly, bypassing the use case. They could not alter an existing one: the triggers and PRIMARY KEY prevent that, unless triggers are dropped, as the EI-4E test-only `bypass()` shows is possible for file owners. This is the same file-level trust EI-2B, EI-3C and EI-4D-B already rely on.

**EI4-D11 interaction:**

- Policy-publication authority is **not** recording authority.
- The verified recorder stays the required generation write boundary (EI4-D11-A). The recording gate above is an EI4-D10 check *inside* that boundary; making the boundary mandatory and technically enforced is EI4-D11's separate slice.
- The raw generation adapter stays an Infrastructure mechanism.
- Neither decision authorizes invalid-record recovery: a historically incompatible disposition is reported, never repaired (EI4-D11-B to EI4-D11-D).

## 12. Proposed implementation slices

None is authorized by this document.

> **Note, 2026-11-08.** The owner approved this slice sequence with the design ([EI-4 Decision Register, section 11.5](../product/Equity-Intraday-V1-EI4-Decision-Register.md#115-approved-implementation-sequence)). No slice is implemented or started by that approval.

| Slice | Repository | Responsibility | Allowed areas | Depends on | Tests | Acceptance gate | Excludes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **S1. Binding contracts and publication** (additive) | application | Catalog port, binding records, `…policy-definition/1` format and `eipd1` prefix, publication use case with reconciliation, catalog-backed resolver, audit use case | `ports/`, `application_services/` (new modules), one prefix entry in `equity_recommendation_generation_evidence.py` | Design approval | Contract and unit tests with in-memory fakes | Full Application suite green; no existing test changes; `eirs1`, `eirk1` and all existing formats unchanged (golden-fingerprint test) | No generator or recorder change; no Infrastructure |
| **S2. SQLite catalog** | infrastructure | `SQLiteEquitySyntheticPolicyCatalogStore`, schema, triggers, `BEGIN IMMEDIATE` publication, typed errors | `persistence/` (new modules), new tests | S1 merged | SQLite tests: constraints, triggers, rollback, commit seam, threads and spawned processes | Full Infrastructure suite green; all 73 EI-4D-B and 72 EI-4E tests unchanged and passing | No change to generation tables, fixtures or EI-4E tests |
| **S3. Enforcement gate** (breaking, coordinated) | application | Generator set check and the `POLICY_REFERENCE_CONFLICT` and `POLICY_SET_UNAVAILABLE` reasons; recorder gate and the three outcomes; the recorder takes the catalog store | `generate_equity_trade_recommendation.py`, `equity_trade_recommendation_eligibility.py` (reason enum and classification), `record_equity_recommendation_generation.py`, their tests | S1, S2 | Unit tests for every row of section 10 | Application suite green; evaluator version and assessment format unchanged | No Infrastructure; no EI4-D11 boundary enforcement |
| **S4. Fixture and acceptance alignment** | infrastructure | EI-4D-B and EI-4E fixtures publish their synthetic policies before generating; `test_e12_c_…` changes its expected outcome from a second COMMITTED disposition to a recording refusal; new EI4-D10 end-to-end acceptance module | `tests/` only | S3 merged (Infrastructure stays red between S3 and S4 unless they merge together) | End-to-end acceptance (section 13) | Full Infrastructure suite green; changed EI-4E expectations listed one by one in the PR | No production change |
| **S5. Governance acceptance** | docs | EI4-D10 enforcement acceptance record; architecture record if EI4-D10-DQ8 approves one | `product/`, `adr/` | S4 | — | Owner review | No status change without owner approval |

S3 and S4 change a constructor that Infrastructure tests use. They must merge as a coordinated pair (S3 then S4 at once), or S3 must land on a branch that S4 completes. The owner should choose which in the review (an engineering choice, not raised as a DQ).

## 13. Acceptance matrix

All tests below are **proposed**. None exists, and none is claimed to pass.

Types:

- **C**: Application contract or unit test, with in-memory fakes.
- **S**: Infrastructure SQLite test.
- **E**: end-to-end acceptance through the real chain on real SQLite.

| # | Scenario | Expected | Type | Slice |
| --- | --- | --- | --- | --- |
| 1 | First publication of components and set | COMMITTED; rows verified on reload | C, S | S1, S2 |
| 2 | Identical re-publication | EQUIVALENT; no row written | C, S | S1, S2 |
| 3 | Same namespace, type and label, different content | CONFLICT; nothing written; stored binding unchanged | C, S, E | S1, S2, S4 |
| 4 | Same label in a different namespace | Refused at construction (closed namespace); no row | C, S | S1, S2 |
| 5 | Same label under two policy types | Two independent bindings, both COMMITTED | C, S | S1, S2 |
| 6 | Different label, different content | COMMITTED as a new binding | C, S | S1, S2 |
| 7 | Decimal `1.50` and `1.5` | One canonical document; EQUIVALENT | C | S1 |
| 8 | Label with surrounding whitespace, a case variant, or no `synthetic-` prefix | Refused (Core); a case variant is a distinct label | C, S | S1, S2 |
| 9 | Unpublished label at generation | Existing `*_UNAVAILABLE` or `POLICY_SET_UNAVAILABLE`; attempt-only | C, E | S3, S4 |
| 10 | Unpublished label at recording, with a non-catalog source | `POLICY_REFERENCE_NOT_ESTABLISHED`; nothing written | C, E | S3, S4 |
| 11 | Conflicting content at recording (the EI-4E AA-1 scenario) | `POLICY_REFERENCE_CONFLICT`; no second disposition | C, E | S3, S4 |
| 12 | Unsupported binding document version | `Incompatible…` at load; `POLICY_REFERENCE_CONFLICT` at generation | C, S | S1, S2, S3 |
| 13 | Corrupted stored binding (test-only bypass) | Invalid at load; fails closed; row unchanged | S, E | S2, S4 |
| 14 | Restart: publish, close everything, reopen | Bindings load and verify | S, E | S2, S4 |
| 15 | Independent-connection concurrency, identical and conflicting | {COMMITTED, EQUIVALENT}; {COMMITTED, CONFLICT}; one row | S | S2 |
| 16 | Separate spawned processes on Windows, identical and conflicting | As 15; distinct process IDs | S | S2 |
| 17 | Lost COMMIT acknowledgment | Reconciled: EQUIVALENT_PUBLISHED | C, S | S1, S2 |
| 18 | COMMIT not made, verified absent | Bounded retry; at most 3 writes; no duplicates | C, S | S1, S2 |
| 19 | Reload unavailable after an uncertain COMMIT | PUBLICATION_RECONCILIATION_UNRESOLVED | C, S | S1, S2 |
| 20 | Partial bundle with a failure mid-transaction | Nothing written; a set never without its components | S | S2 |
| 21 | Historical replay with the catalog unavailable | A to C verify exactly as before; the audit reports unavailable | C, E | S1, S4 |
| 22 | Pre-enforcement records: matching, unbound, reused label | Historically verified (order caveat), not established, incompatible; no row changed | C, E | S1, S4 |
| 23 | Existing EI-4E and EI-4D-B suites | All pass, with only the listed S4 expectation changes | S, E | S2, S4 |
| 24 | `eirs1`, `eirk1` and existing document formats | Golden fingerprints of existing synthetic records unchanged | C | S1, S3 |
| 25 | Existing dispositions | Byte-identical before and after publication and audit | S, E | S2, S4 |
| 26 | No issuance | No issuance or history writes; proposals not issued | E | S4 |
| 27 | Valid policy, no candidate | `NO_CANDIDATE`; the catalog is not consulted | C | S3 |
| 28 | Eligibility refusal under published policies | `NOT_ELIGIBLE` with `LISTING_NOT_ELIGIBLE`; never a policy conflict | C, E | S3, S4 |
| 29 | SQLite `integrity_check` and `foreign_key_check` | Clean on every database not deliberately corrupted, asserted in each S test | S | S2 |
| 30 | Catalog unavailable during EI-4C generation | `DEPENDENCY_UNAVAILABLE` with the existing `*_UNAVAILABLE` reason; attempt-only; nothing invented or published | C, E | S3, S4 |
| 31 | Catalog unavailable at the EI-4D-A recording gate | `POLICY_REFERENCE_UNAVAILABLE`; nothing written; `attempt_recorded=False` | C, E | S3, S4 |
| 32 | Permanent conflict versus temporary outage, classified by machine-readable value | Distinct reasons and outcomes; never `NOT_ELIGIBLE` or `CONTENT_CONFLICT` | C, E | S3, S4 |
| 33 | Permanent conflict is never retried or republished | One attempt only; no republication; stored binding unchanged; no disposition | C, S, E | S1, S2, S3, S4 |

### 13.1 Scenarios 30 to 33

Added on 2026-11-08, before ADR-022's documentation PR, so that every owner-required test of [ADR-022, section 12](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md#12-implementation-sequence-and-acceptance-requirements) has an explicit row.

- Rows 1 to 29 and their identifiers are unchanged. None of them already covered these conditions:
  - rows 9 and 10 cover an *unpublished* label, not an outage;
  - row 19 covers a publication reload, and row 21 covers replay;
  - rows 11 and 12 assert the conflict outcome, but not its distinction from an outage or the absence of retries.
- The matrix now holds **33 scenarios**. Every one is **PROPOSED / NOT EXECUTED**.

| # | Requirement | Scenario and setup | Expected outcome | Fail-closed invariant | Layer | Slice | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 30 | EI4-D10-DQ5 (C, D); EI4-D10-DQ6 case A | **Generation catalog outage.** The catalog-backed resolver raises its unavailable error: a contract fake, or an end-to-end test-only adapter seam or lock | Status `DEPENDENCY_UNAVAILABLE` with the existing component `*_UNAVAILABLE` reason, retained as a machine-readable value. Attempt-only: no generation key, no disposition | No synthetic definition is invented or defaulted. No binding is published (catalog rows unchanged). No disposition is recorded. Nothing retries unless a separately approved operational retry policy applies; none exists | C, E | S3, S4 | PROPOSED / NOT EXECUTED |
| 31 | EI4-D10-DQ5 (E); EI4-D10-DQ6 case A | **Recorder catalog outage.** A verified disposition reaches the EI-4D-A recorder while the catalog cannot be read before persistence | Recording outcome `POLICY_REFERENCE_UNAVAILABLE`, with zero writes. `attempt_recorded=False`: nothing was committed, so nothing is claimed as audited | The pre-write gate refuses the disposition. No implicit policy registration. No unverified record becomes authoritative. Historical replay of already-stored records still runs from frozen inputs without the catalog (row 21) | C, E | S3, S4 | PROPOSED / NOT EXECUTED |
| 32 | EI4-D10-DQ6 (cases A, B, E, F) | **Conflict versus outage**, two runs on the same candidate: (1) a published label is answered with conflicting canonical content; (2) the catalog is temporarily unavailable | At generation, both have status `DEPENDENCY_UNAVAILABLE`. Run 1's reason is `POLICY_REFERENCE_CONFLICT`; run 2's is the existing `*_UNAVAILABLE` reason, so their attempt identities differ. At recording, the outcomes are `POLICY_REFERENCE_CONFLICT` and `POLICY_REFERENCE_UNAVAILABLE` | Callers tell the cases apart by enum value alone, never by parsing diagnostic prose. Neither is ever reported as `NOT_ELIGIBLE` or `CONTENT_CONFLICT` | C, E | S3, S4 | PROPOSED / NOT EXECUTED |
| 33 | EI4-D10; EI4-D10-DQ6 case B | **No retry of a permanent conflict**, with call counters on the resolver and the store: (1) publish a conflicting definition under a published label; (2) generate with conflicting content; (3) record a disposition whose label conflicts | Publication returns CONFLICT after one transaction, never retried: the bounded retry of 8.4 applies only to a *verified absence* after an unconfirmed COMMIT. Generation resolves each label once. The recording gate refuses with zero writes. Each reports the permanent conflict | No automatic retry treats the conflict as an outage. No conflicting definition is republished. No binding is overwritten. No disposition is created from the contradictory policy. The original immutable binding is byte-identical afterwards | C, S, E | S1, S2, S3, S4 | PROPOSED / NOT EXECUTED |

Scenarios 30, 31 and 33 test EI4-D10's recording gate **inside** the EI-4D-A recorder only. Whether every operational write must pass through that recorder is EI4-D11's separate requirement, which these scenarios do not test. A direct Infrastructure adapter write can still bypass the gate until EI4-D11 is implemented and accepted.

## 14. Limitations

- **Order before enforcement.** For records written before enforcement, "published before generated" cannot be proved. The audit says "verified against a binding", never "compliant at generation".
- **File-level trust.** Direct writes to the SQLite file can add new bindings outside the use case (EI4-D10-DQ9).
- **Availability.** Recording depends on the catalog being available. A catalog outage blocks new dispositions; it fails closed by design.
- **Price increments.** Under EI4-D10-DQ3's recommendation, one increment label binds one amount for every listing and date. A future per-listing increment table would need distinct labels or a new decision.
- **No publication time.** `publication_sequence` orders publications; it does not timestamp them.
- **Unchanged EI-4E limitations.** The evaluator version is still not content-fingerprint-bound. Storage is single-host and single-file SQLite. Inputs are synthetic only.
- **Trust in the recorder.** The recording gate relies on EI4-D11-A being enforced before it is a complete guarantee. Until EI4-D11's slice lands, direct adapter writes bypass it, exactly as [AA-2](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md#142-aa-2--verified-writer-boundary-and-recovery) records.

## 15. Decisions awaiting approval

These are design questions, not approvals. Each needs an explicit owner decision before the slice it affects starts.

| ID | Question | Options | Recommendation | Rationale | Compatibility | Scope | Approval needed |
| --- | --- | --- | --- | --- | --- | --- | --- |
| EI4-D10-DQ1 | What is the policy namespace? | (a) One explicit closed constant `northstar.equity-intraday-v1.synthetic`; (b) implicit, the catalog file; (c) per exchange | (a) | EI4-D10 requires a *defined* namespace. A closed constant cannot be minted around a conflict | None for existing records | S1 | Owner |
| EI4-D10-DQ2 | Which policy types are in scope? | (a) Five components + `POLICY_SET`; (b) add listing eligibility; (c) components only | (a) | Eligibility is per-listing evidence and content under EI4-D9-B. Set labels are labels too, and EI-4E's reuse was inside one set label | None | S1–S3 | Owner |
| EI4-D10-DQ3 | How granular is price-increment binding? | (a) One amount per label; (b) per label, listing and date | (a) | Matches Core `EquityPriceIncrement`, which has no listing, and every fixture. Under (b), one set label would map to several `eirs1` values, because the increment is inside the policy-set document | (b) would need per-listing set bindings | S1, S2 | Owner |
| EI4-D10-DQ4 | Where does the catalog live? | (a) The existing evidence database, with additive tables; (b) a separate catalog file | (a) | One SQLite authority and one backup, consistent with ADR-021 section 11. The recording gate and records read one file | Additive tables only; no existing table altered | S2 | Owner |
| EI4-D10-DQ5 | Where is the invariant enforced? | (a) Publication + catalog-backed resolution + generation set check + recording gate; (b) generation only; (c) recording only | (a) | (b) depends on wiring and there is no composition root. (c) lets misleading audits exist in memory. Replay stays catalog-free in every option | Constructor change for the generator and recorder | S3, S4 | Owner |
| EI4-D10-DQ6 | Which generation status does `POLICY_REFERENCE_CONFLICT` take? | (a) `DEPENDENCY_UNAVAILABLE`, like the existing `POLICY_REFERENCE_MISMATCH`; (b) `RESEARCH_INTEGRITY_FAILURE`; (c) a new status | (a), with the distinct reason carrying "policy-reference integrity conflict" | (c) needs the insert-only attempts table rebuilt (its `status IN` CHECK). (b) misnames the failure as research. (a) follows the existing reference-mismatch precedent | (a) and (b) need no schema change | S3 | Owner, because EI4-D10 names the failure class |
| EI4-D10-DQ7 | How are pre-enforcement records and policies treated? | (a) No automatic backfill: explicit publication only; read-only audit classification; (b) backfill from the first committed disposition; (c) embed binding evidence in new frozen inputs | (a) | (b) invents history. (c) changes `eiri1` and `eira1` document versions | (a) changes nothing stored | S1, S4 | Owner |
| EI4-D10-DQ8 | How is the approved enforcement architecture recorded? | (a) A new ADR (ADR-022) after design approval; (b) an addendum to ADR-021 | (a) | Follows the EI4-D9 → ADR-021 precedent and leaves ADR-021 unedited | None | S5 | Owner |
| EI4-D10-DQ9 | Is file-level trust for publication accepted? | (a) Accept and document it; (b) require technical publisher authentication | (a) | Personal-use system with no identity layer. Equal to the EI-2B, EI-3C and EI-4D-B trust today | None | — | Owner |

> **Note, 2026-11-08.** All nine decisions in the table above are **OWNER APPROVED, 2026-11-08**, each as its "Recommendation" column states, with the clarifications recorded in [section 17](#17-owner-approval-2026-11-08). None remains pending. The table is kept as proposed.

**Not raised as owner decisions** (engineering choices under existing standards):

- the `eipd1` prefix and the format name;
- table and column names;
- the 5-second busy timeout and `max_writes = 3`, as in EI-4D-B;
- trigger messages;
- module names;
- whether S3 and S4 merge as one coordinated pair.

**ADR consistency.** No contradiction with ADR-020 or ADR-021 was found:

- ADR-020 section 2 already adopts "named, versioned generation policies, where any parameter change is a new version".
- ADR-021 sections 10 and 11 assign fingerprint and replay verification to the Application, and structure to SQLite. The design keeps the four verification levels unchanged.

No ADR is edited.

## 16. Explicit non-authorizations

This document authorizes nothing. In particular, it does not authorize:

- implementing any slice, contract, schema, adapter, test or fixture change;
- changing `eirs1`, `eirk1`, any canonical document version or the evaluator version;
- altering, re-keying, migrating, deleting or backfilling any stored record or policy evidence;
- any invalid-record recovery mechanism (EI4-D11-D);
- EI4-D11 enforcement;
- production policy values or defaults. Every policy here is synthetic, and EI0-D10 to EI0-D13 stay pending;
- actual recommendation issuance, the lifecycle `RECORDED` state, orders, fills, positions, P&L, live data, exchange licensing or production trading;
- EI-5 or EI-6.

The EI4-D10 decision itself stays **OWNER APPROVED, 2026-11-08**. Only this enforcement design is pending review. The pending decisions EI4-D5, EI0-D11, the same-listing exclusivity threshold, EI0-D19, EI0-D21 and EI3-D10 keep their status.

> **Note, 2026-11-08.** The enforcement design is no longer pending review: it is approved ([section 17](#17-owner-approval-2026-11-08)). Every non-authorization above still applies. Approval of the design authorizes no implementation, data change or recovery by itself.

## 17. Owner approval (2026-11-08)

This section was added after the proposal above. It does not rewrite it: the proposed rationale, alternatives, risks and limitations stand as written.

**OWNER APPROVED, 2026-11-08:** EI4-D10-DQ1, DQ2, DQ3, DQ4, DQ5, DQ6, DQ7, DQ8 and DQ9, each as recommended in [section 15](#15-decisions-awaiting-approval). No other owner approval is recorded here. The approved texts are in the [EI-4 Decision Register, section 11](../product/Equity-Intraday-V1-EI4-Decision-Register.md#11-ei4-d10-enforcement-design-owner-approval-record-2026-11-08), and the approved architecture is [ADR-022](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md).

| Item | Status |
| --- | --- |
| EI4-D10 governance policy | APPROVED |
| EI4-D10 technical enforcement design | APPROVED: this design, ADR-022 |
| EI4-D10 engineering implementation | NOT YET IMPLEMENTED |
| EI4-D10 implementation acceptance | NOT YET ESTABLISHED |

**Clarifications approved with the decisions:**

- **EI4-D10-DQ6.** `POLICY_REFERENCE_CONFLICT` is a **permanent integrity conflict**, never transient catalog downtime:
  - it fails closed, under the compatible top-level `DEPENDENCY_UNAVAILABLE`;
  - it is never retried automatically, and never leads to silent republication with changed content;
  - a temporary catalog outage stays a dependency outage that creates no binding, and is retried only under a separately approved operational retry policy, of which none exists;
  - the reason must propagate through the relevant outcome and audit contracts;
  - no new persisted attempt status is created.
- **EI4-D10-DQ9.** The SQLite file trust boundary is a deliberate, limited assumption for current personal-use synthetic research:
  - it is not tamper-proof publication, protection against privileged file access, cryptographic publisher proof, multi-host authorization or a production identity system;
  - it is not acceptable for real trading without separate review;
  - it does not weaken EI4-D11's mandatory verified recording requirement.

**Acceptance requirements.** Rows 1 to 29 of the [section 13](#13-acceptance-matrix) matrix are preserved. The three owner-required tests that had no explicit row were added as scenarios 30 to 33 ([section 13.1](#131-scenarios-30-to-33)):

- catalog unavailability at generation (30) and at recording (31);
- a permanent conflict classified distinctly from a transient outage (32);
- no automatic retry of a permanent conflict (33).

The matrix holds 33 scenarios. Every scenario is **PROPOSED / NOT EXECUTED**.

**Unchanged.**

- EI4-D11 and its recovery restriction. The EI4-D10 recording gate can still be bypassed by direct Infrastructure adapter use until EI4-D11's boundary is implemented and accepted.
- `eirs1`, `eirk1` and every existing canonical document.
- The EI-4E record and its verdict.
- Every pending decision listed in [section 16](#16-explicit-non-authorizations).

## Related documents

- [ADR-022: Equity Intraday V1 Immutable Synthetic Policy-Version Identity Enforcement](../adr/ADR-022-Equity-Intraday-V1-Immutable-Synthetic-Policy-Version-Identity-Enforcement.md) (added 2026-11-08)
- [Equity Intraday V1 EI-4 Decision Register](../product/Equity-Intraday-V1-EI4-Decision-Register.md), sections 9, 10 and 11
- [Equity Intraday V1 EI-4E Synthetic Acceptance Record](../product/Equity-Intraday-V1-EI4E-Synthetic-Acceptance-Record.md)
- [ADR-020: Equity Intraday V1 Recommendation Generation Architecture](../adr/ADR-020-Equity-Intraday-V1-Recommendation-Generation-Architecture.md)
- [ADR-021: Equity Intraday V1 Recommendation Generation Persistence](../adr/ADR-021-Equity-Intraday-V1-Recommendation-Generation-Persistence.md)
- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md)
- [ES-003: Testing Standards](../engineering/standards/ES-003-Testing-Standards.md)
