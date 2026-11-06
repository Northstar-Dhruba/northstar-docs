# Equity Intraday V1 EI-3C Synthetic Acceptance Record

## Status

**Engineering acceptance record, 2026-11-07.**

- **Verdict:** PASS WITH DOCUMENTED LIMITATIONS.
- **Milestone:** EI-3C COMPLETE — SYNTHETIC RESEARCH PERSISTENCE ACCEPTED.
- This record is engineering acceptance evidence. It is **not** an owner approval: it approves no decision and changes no decision status. The owner approvals that EI-3C implements are recorded in the [EI-3 Decision Register, section 11](Equity-Intraday-V1-EI3-Decision-Register.md#11-ei-3c-owner-approval-record-2026-11-07).
- Accepting EI-3C is **not** real-data readiness, production readiness, trading authorization or acceptance of any profitability or performance claim.

## 1. Scope

EI-3C makes synthetic equity intraday research timelines durable and authoritative. It builds on the immutable evidence of EI-2B, the EI-3A research evaluator and the EI-3B timeline scheduler, and consists of:

- **EI-3C-A** (northstar-application): the `EquityIntradayResearchTimelineStore` port, `AdvanceEquityIntradayResearchTimelineUseCase`, historical rehydration and verification, bounded stale-head retries and unconfirmed-commit reconciliation;
- **EI-3C-B** (northstar-infrastructure): `SQLiteEquityIntradayResearchTimelineStore`, its append-only tables, constraints and triggers, atomic compare-and-append and foreign keys to EI-2B snapshots in the same SQLite file.

Everything accepted here is synthetic: invented candles, finality statements, applicability records and calibration.

## 2. Revisions accepted

These are the revisions verified during final acceptance. Every working tree was clean, and no uncommitted change was needed.

| Repository | Branch | HEAD | Merged change |
| --- | --- | --- | --- |
| northstar-application | `develop` | `80f7020` | EI-3C-A, PR #84 |
| northstar-infrastructure | `develop` | `837735f` | EI-3C-B, PR #51 |
| northstar-docs | `develop` | `0e90dc8` | EI-3C governance, PR #46 |

## 3. Acceptance evidence

Every result below was produced on the development machine during the acceptance review. None is a CI artifact, and none describes a deployed environment.

| Check | Result |
| --- | --- |
| Independent acceptance script, real SQLite on disposable databases | 59 of 59 checks passed |
| Full Application test suite | 4,033 passed, 0 failed, 0 skipped |
| Full Infrastructure test suite | 2,663 passed, 0 failed, 27 skipped |
| SQLite `integrity_check` and `foreign_key_check` | passed on all 12 acceptance databases |
| Ruff check, Ruff format check and `git diff --check` (Application and Infrastructure) | clean |

The 27 Infrastructure skips are pre-existing and unrelated to EI-3C: opt-in live Databento acceptance tests, and one test that runs only on POSIX systems.

The independent acceptance script was written for the review and kept outside every repository. It used the merged code, not the merged test fixtures, with its own synthetic prices and configuration.

### 3.1 Behaviour demonstrated on real SQLite

1. Synthetic EI-2C ingestion and EI-2B immutable snapshot issuance.
2. EI-3A research decisions: NO_SIGNAL, LONG_CANDIDATE and a later qualifying evaluation.
3. EI-3B roles: one CANDIDATE, then REPEAT_QUALIFYING.
4. Authoritative EI-3C persistence of the timeline.
5. Restart through fresh Application and Infrastructure instances.
6. Identical recovered decision fingerprints and timeline hash chain.
7. Idempotent retries: resubmitted snapshots were DUPLICATE, with no new rows.
8. A correction-driven REJECTED entry.
9. Byte-identical preservation of the earlier entries.
10. Concurrent-writer protection and bounded stale-head retries.
11. Reconciliation of unconfirmed commits.
12. Refusal of invalid direct SQL writes.

The completed correction scenario held one timeline with four entries.

## 4. Governance compliance

The three owner decisions approved on 2026-11-07 ([EI-3 Decision Register, section 11.1](Equity-Intraday-V1-EI3-Decision-Register.md#111-approved-decisions)) passed acceptance. Their approved text is not repeated here.

| Decision | Subject | Acceptance result |
| --- | --- | --- |
| EI3-D11 | The first committed entry establishes timeline authority | Passed |
| EI3-D12 | Serialized concurrent commits; established history is immutable | Passed |
| EI3-D13 | Historical replay verification; incompatible or mismatched history fails closed | Passed |

Every earlier approval is unchanged: EI3-D1 to EI3-D5, EI3-D8, EI0-D3 and EI0-D6 (2026-11-05), and EI3-D6, EI3-D7 and EI3-D9 (2026-11-06). This record makes no new owner approval.

## 5. What enforces each property

- **SQLite** enforces append-only rows, the next sequence, the previous-entry link, strictly increasing decision instants on an exact integer microsecond key, non-decreasing watermarks, at most one CANDIDATE, REPEAT_QUALIFYING only after a CANDIDATE, foreign keys to EI-2B snapshots with matching provenance and synthetic evidence class, and decision documents consistent with their entries.
- **The Application** verifies, on every load, the SHA-256 `eir1` decision fingerprints and `eirt1` entry fingerprints, the canonical decision documents, and the reproduction of every decision from its immutable snapshot. SQLite does not verify SHA-256 fingerprints.

## 6. Accepted limitations

None of these blocks synthetic EI-3C closure.

- Which of several equivalent snapshots represents an instant, and so the fingerprint chain from that entry on, can depend on which was committed first. Research outcomes for equivalent evidence are the same.
- Full historical rehydration on every advancement costs more as a timeline grows.
- Historical evaluation is compatible only with the supported EI-3A decision format. No historical evaluator registry exists.
- Decoding the stored configuration is coupled to the existing strategy configuration structure.
- SHA-256 fingerprints are verified by the Application, not independently by SQLite.
- SQLite cannot refuse a raw identity row without entries; such a row is rejected on load.
- Every COMMIT failure is conservatively classified as unconfirmed.
- Storage is single-host, single-file SQLite.
- There is no external audit anchoring and no API composition root.

## 7. Production and execution restrictions

Every production and execution restriction stays in force, including those in [section 8 of the EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md#8-production-readiness-restrictions). In particular, there is:

- no authorized real-data provider and no completed data-rights approval;
- no approved production finality source;
- no complete production listing and applicability authority;
- no completed corporate-action or surveillance source;
- no approved trading execution policy;
- no cost, sizing, risk or recommendation acceptance;
- no broker or deployment authorization.

EI0-D7, EI0-D8, EI0-D20, EI0-D21, EI3-D10 and the outstanding CE decisions keep their existing status ([EI-3 Decision Register, section 11.2](Equity-Intraday-V1-EI3-Decision-Register.md#112-decisions-still-pending)). This record changes no governance status.

## 8. Related documents

- [Equity Intraday V1 EI-3 Decision Register](Equity-Intraday-V1-EI3-Decision-Register.md)
- [Equity Intraday V1 Product Specification](Equity-Intraday-V1-Product-Specification.md)
- [NSE Cash-Equity Session Evidence Register](../research/NSE-Cash-Equity-Session-Evidence-Register.md)
