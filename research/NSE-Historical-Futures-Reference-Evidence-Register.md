# NSE Historical Futures Reference Evidence Register (R-1B-2B-1)

Status: **evidence register; no fact is accepted.** Every owner review below
is PENDING. Nothing here is a verified contract reference, and no
verified-reference code exists: Infrastructure's
`HistoricalFuturesReferenceKind` has only `SYNTHETIC` (R-1B-1).

> **Note, 2026-10-18 (R-1B-2B-2).**
> - The owner approved governance decisions D-1 and D-2 (section 7.1). They
>   approve no fact.
> - An evidence acceptance gate now exists in Infrastructure (section 6.1).
>   Its production registry is empty.
> - Accepted facts: **0**. Verified historical contracts: **0**. Every
>   extracted fact (NSE-F-01 to NSE-F-22) remains individually PENDING, and
>   `HistoricalFuturesReferenceKind` is still only `SYNTHETIC`.

This register records which official NSE documents were located and
inspected, which facts were extracted from them, and what is still missing
before any historical NIFTY futures contract reference can be built. It
starts from the R-1B-2A evidence discovery of 2026-10-17, whose verdict was
**PARTIAL EVIDENCE**.

Related specifications:

- [Historical Futures Dataset Specification v1.0](../architecture/Historical-Futures-Dataset-Specification-v1.0.md)
  (R-1A): section 4 lists the contract reference requirements.
- R-1B-1 contract reference foundation:
  `northstar-infrastructure/src/northstar_infrastructure/market_data/historical_futures_contract_reference.py`.
- The existing NSE calendar reference:
  `northstar-infrastructure/src/northstar_infrastructure/market_data/nse_futures_calendar_reference.py`.

**Location.** No research or reference-evidence folder existed, so this
register starts `research/`; the README index links it.

## 1. Rules of this register

- Entries are appended. A recorded entry is corrected only by a dated note,
  never by rewriting it.
- **No source text is reproduced.** NSE's terms for reusing its circulars
  have not been reviewed (section 8), so each fact is an original summary
  with a page, table or annexure reference. Read the source for the wording.
- **Source PDFs are not committed.** Only identifiers, URLs, retrieval
  details and fingerprints are recorded.
- Search-engine results, broker notices and other secondary pages were used
  only to locate documents. None is evidence here.
- Nothing in this register is market-price data, and nothing here permits
  retaining any.

### 1.1 Evidence states

| State | Meaning |
|---|---|
| SOURCE LOCATED | A reference or URL for the document exists. It was not opened. |
| SOURCE INSPECTED | The document was retrieved from the official URL and read. |
| FACT EXTRACTED | A specific fact was identified in an inspected source and is recorded below with its location. |
| FACT REVIEWED | The owner examined the fact, its source and its interpretation, and recorded ACCEPTED, REJECTED or DEFERRED. |
| FACT ACCEPTED FOR INTEGRATION | The owner ACCEPTED the fact through the review gate (section 6) for a stated reference purpose. Only these may later enter a version-controlled reference catalog. |

A source being inspected does not make any of its facts reviewed or
accepted. Today every fact is FACT EXTRACTED and owner review is PENDING.

### 1.2 Fingerprints

A fingerprint is the SHA-256 of the exact bytes retrieved. It shows that a
later copy is byte-identical to the copy observed here. It does **not** show
that NSE issued the document, that the document is current, or that it has
not been superseded. NSE circulars are not signed in any way Northstar
checks.

The fingerprints below were computed on 2026-10-17, during R-1B-2B-1, from
the bytes retrieved during R-1B-2A (and, for EV-10, during R-1B-2B-1). The
retrieved copies sit in a temporary engineering-session folder and are not
retained by Northstar.

## 2. Sources

All circulars were retrieved on 2026-10-17 over HTTPS with HTTP 200 and
content type `application/pdf`.

| ID | Issuer | Document | Published | Effective (as stated) | Official URL |
|---|---|---|---|---|---|
| NSE-EV-01 | NSE, F&O | NSE/FAOP/47854, Circular Ref. 28/2021: revision in market lot of index derivative contracts | 2021-03-31 | 2021-04-30 | https://archives.nseindia.com/content/circulars/FAOP47854.pdf |
| NSE-EV-02 | NSE, F&O | NSE/FAOP/61415, Circular Ref. 37/2024: revision in market lot of index derivative contracts | 2024-04-02 | 2024-04-26 | https://nsearchives.nseindia.com/content/circulars/FAOP61415.pdf |
| NSE-EV-03 | NSE, F&O | NSE/FAOP/64625, Circular Ref. 128/2024: contract size revision for index derivatives | 2024-10-18 | New contracts introduced from 2024-11-20 | https://nsearchives.nseindia.com/content/circulars/FAOP64625.pdf |
| NSE-EV-04 | NSE, F&O | NSE/FAOP/67135, Circular Ref. 39/2025: revision in tick size of index futures | 2025-03-13 | Trade date 2025-04-15 | https://nsearchives.nseindia.com/content/circulars/FAOP67135.pdf |
| NSE-EV-05 | NSE, F&O | NSE/FAOP/68589, Circular Ref. 103/2025: revision in expiry day of index and stock derivatives | 2025-06-17 | Contracts expiring on or after 2025-09-01 | https://nsearchives.nseindia.com/content/circulars/FAOP68589.pdf |
| NSE-EV-06 | NSE, F&O | NSE/FAOP/68747, Circular Ref. 111/2025: revision in expiry day, update | 2025-06-25 | New Tuesday contracts from 2025-06-26 EOD | https://nsearchives.nseindia.com/content/circulars/FAOP68747.pdf |
| NSE-EV-07 | NSE, F&O | NSE/FAOP/70616, Circular Ref. 176/2025: revision in market lot of index derivative contracts | 2025-10-03 | 2025-10-28 (EOD) | https://nsearchives.nseindia.com/content/circulars/FAOP70616.pdf |
| NSE-EV-08 | NSE, F&O | NSE/FAOP/71876, Circular Ref. 217/2025: change in expiry and maturity date of contracts | 2025-12-18 | Trading systems from 2025-12-29 | https://nsearchives.nseindia.com/content/circulars/FAOP71876.pdf |
| NSE-EV-09 | NSE | Equity derivatives contract specifications web page (snapshot); the page states it was updated on 11/08/2026 | Not stated | Current rules only | https://www.nseindia.com/static/products-services/equity-derivatives-contract-specifications |
| NSE-EV-10 | NSE, F&O | NSE/FAOP/71777, Circular Ref. 212/2025: trading holidays for calendar year 2026 | 2025-12-12 | Calendar year 2026 | https://nsearchives.nseindia.com/content/circulars/FAOP71777.pdf |

| ID | Bytes | SHA-256 | Inspection |
|---|---|---|---|
| NSE-EV-01 | 70783 | `069c95668aee2eb4d82ec88eef3da110fda1c41fad5b6c2acd692d53eda6b7c7` | INSPECTED in R-1B-2A (1 page) |
| NSE-EV-02 | 99534 | `463b464c998fa067248b6ee9f8be27824722ab9b3795870d4e5f1a27f4ce4a7c` | INSPECTED in R-1B-2A (2 pages) |
| NSE-EV-03 | 104109 | `ee1169239eed9cc06519537c6af53751e031903fe709b2bf296283412dc028f8` | INSPECTED in R-1B-2A (3 pages, annexure on page 3) |
| NSE-EV-04 | 90701 | `ce09d78c4dbcffcff978c0912e08cd3858e504931873f4840c143278556fe2c6` | INSPECTED in R-1B-2A (2 pages) |
| NSE-EV-05 | 102889 | `efa5efbc334d39e18dc2138c0d48f779dd298d0e51ffe8a1b669728b443e1f63` | INSPECTED in R-1B-2A (2 pages) |
| NSE-EV-06 | 181366 | `e1b56024a511135ffd5c6c3c097881dd0ac2c37b15b5f61e94fe84e51cf66762` | INSPECTED in R-1B-2A (4 pages, annexure on pages 3-4) |
| NSE-EV-07 | 207805 | `0718cf0cc7e6c74105dc946497369270fca807f59912586560c9552cad8050ef` | INSPECTED in R-1B-2A (2 pages, annexure on page 2) |
| NSE-EV-08 | 110421 | `9fd9ecafba6e070458330133a439f83670748d71c431dd57a3ed6abad3fdadd2` | INSPECTED in R-1B-2A (1 page) |
| NSE-EV-09 | 376099 | `398f5ddccf6d28799fe3d41d9de07b959ceb2881662ce24a5eab41a3d8675bff` | INSPECTED in R-1B-2A. HTML of a live page: the fingerprint identifies that retrieval only, and the page changes over time. |
| NSE-EV-10 | 176660 | `5a2079cd78b2e6b536ef0d28300e63b645721bed22cc82a91facf5945f3296ea` | INSPECTED in R-1B-2B-1 (2 pages) |

The contents of NSE-EV-01 to NSE-EV-09 were read during R-1B-2A and were not
re-read for this register. The fingerprints were computed in this task from
the same retrieved bytes.

**Located but not inspected:**

- NSE/FAOP/68685 (2025-06-23), the expiry-day circular that NSE-EV-06
  partially modifies. Referenced only in NSE-EV-06.
- NSE/FAOP/72262 (2026-01-12), cited by the existing calendar reference for
  the 2026-01-15 holiday. Not inspected in R-1B-2A or this task.
- NSE/FAOP/62175 and NSE/FAOP/67134 (stock-futures tick size). Retrieved
  during R-1B-2A but not reviewed, and nothing below relies on them.
- SEBI circulars cited by the NSE circulars, including
  SEBI/HO/MRD/TPD-1/P/CIR/2024/132 (2024-10-01),
  SEBI/HO/MRD-PoD2/CIR/P/2024/00181 (2024-12-30) and
  SEBI/HO/MRD/TPD-1/P/CIR/2025/76 (2025-05-26).

## 3. Extracted facts

Common to every fact: product `NIFTY@NSE` (index futures where the source
distinguishes); **owner review PENDING**; not accepted for integration.

"EOD" means the change applies after the close of the stated day, so the
first affected session is the next trading session. That reading of EOD is
itself an interpretation awaiting owner review (D-3, section 7).

### 3.1 Lot size

| Fact | Evidence | Level | Rule | From | Through | Limits and open questions |
|---|---|---|---|---|---|---|
| NSE-F-01 | EV-01 p.1 | Product rule | NIFTY market lot revised from 75 to 50. Applies to the July 2021 monthly expiry onwards and to weekly contracts from the August 2021 weekly expiry. The May and June 2021 contracts keep 75. Existing long-term options are revised after the June 2021 expiry. | Circular effective 2021-04-30 | Not stated | Before 2024: outside the initial target. Does not show the lot stayed 50 until 2024. |
| NSE-F-02 | EV-02 p.1 tables and items 1-5; p.2 effective statement | Product rule with transition | NIFTY lot was 50 on 2024-04-02 and is revised to 25. The April 2024 monthly contract, expiring 2024-04-25, is not revised. All weekly, monthly, quarterly and half-yearly contracts available for trading from trade date 2024-04-26 carry 25. Contracts generated at EOD 2024-04-25 carry 25. First monthly contract with 25: expiry 2024-05-30. | Trade date 2024-04-26 (inclusive) | Not stated | **Open:** the wording appears to revise contracts already listed before 2024-04-26 (e.g. May and June 2024) from that trade date, which would make it a mid-life change. Requires owner interpretation (D-4). The circular does not show how long 50 had applied. |
| NSE-F-03 | EV-03 p.1 table and final paragraph; p.2 first paragraph; p.3 annexure | Product rule, transition by contract type | NIFTY lot revised from 25 to 75 for new index derivative contracts introduced from 2024-11-20. Existing weekly and monthly contracts keep 25 until they expire. Existing quarterly and half-yearly contracts change at 2024-12-26 EOD. Annexure: last monthly expiry with 25 is 30-Jan-2025; first with 75 is 27-Feb-2025. | Introduction on or after 2024-11-20; quarterly and half-yearly after 2024-12-26 EOD | Not stated | Applying this to a contract needs its introduction date (unavailable, section 4). The annexure notes its dates may change if a holiday is declared. |
| NSE-F-04 | EV-07 p.1 tables and bullets; p.2 annexure | Product rule, transition by contract type, EOD wording | NIFTY lot revised from 75 to 65. Circular effective 2025-10-28 EOD. Weekly and monthly contracts keep 75 through the 30-Dec-2025 expiry. Existing quarterly and half-yearly contracts change at 2025-12-30 EOD. Annexure: last weekly with 75 is 23-Dec-2025, first with 65 is 06-Jan-2026; last monthly with 75 is 30-Dec-2025, first with 65 is 27-Jan-2026. | 2025-10-28 EOD; quarterly and half-yearly after 2025-12-30 EOD | Not stated | Which contracts were introduced after 2025-10-28 EOD is not stated per contract. Holiday caveat in the annexure. |

### 3.2 Tick size

| Fact | Evidence | Level | Rule | From | Through | Limits and open questions |
|---|---|---|---|---|---|---|
| NSE-F-05 | EV-04 p.1 table and paragraphs | Product rule (formula), not a value inventory | Index futures tick by index level: 0.05 up to 15,000; 0.10 above 15,000 up to 30,000; 0.20 above 30,000. First determination 2025-04-11 EOD from the 2025-03-28 index close. After that, reviewed monthly: the index close on the last trading day of a month sets the next month's tick. Applies to all expiries. | Trade date 2025-04-15 | Not stated | **This is a rule, not a monthly tick inventory.** Applying it needs index closes, which are market data. NSE states that the applicable tick appears in the contract and spread master files (p.2). No monthly tick value is established here. |
| NSE-F-06 | EV-04 p.1 table | Product statement at one date | Before this revision, the index futures tick was 0.05 in every band. | Stated as current on 2025-03-13 | Before 2025-04-15 | When 0.05 started is not shown, so it does not establish the 2024 tick. |
| NSE-F-07 | EV-04 p.2 | Product statement | Index options keep a uniform 0.05 tick. | 2025-03-13 | Not stated | Options only; recorded so it is not confused with futures. |

### 3.3 Expiry convention and dates

| Fact | Evidence | Level | Rule | From | Through | Limits and open questions |
|---|---|---|---|---|---|---|
| NSE-F-08 | EV-05 p.1 items 1-3 | Product rule with transition | SEBI agreed to NSE's proposed Tuesday expiry day. Already-introduced contracts keep their expiry day, except long-dated index options. New contracts expiring on or before 2025-08-31 keep Thursday. Contracts expiring on or after 2025-09-01 expire on Tuesday, and monthly contracts on the last Tuesday of the month. | Expiries on or after 2025-09-01 | Not stated | Says NSE would issue operational guidelines later (EV-06). |
| NSE-F-09 | EV-06 p.1 table; p.1-2 items 1-5 | Product rule with transition | NIFTY monthly, quarterly and half-yearly: from last Thursday to last Tuesday of the expiry month. Weekly: from Thursday to Tuesday. Existing contracts expiring on or before 2025-08-31 are unchanged, except long-dated NIFTY and BANKNIFTY options, realigned at 2025-07-31 EOD. New contracts expiring on or after 2025-09-01 are introduced with Tuesday expiry from 2025-06-26 EOD. No other contract-specification change. | 2025-06-26 EOD for new contracts | Not stated | Confirms the pre-change NIFTY monthly convention (last Thursday) as of 2025-06-25 only. It does not show that convention for 2024. Partially modifies NSE/FAOP/68685 (not inspected). |
| NSE-F-10 | EV-06 annexure pp.3-4 | Published schedule | NIFTY monthly expiries 26-Jun-25, 31-Jul-25 and 28-Aug-25 unchanged. A new monthly contract expiring 28-Oct-25 is generated at 2025-07-31 EOD. Quarterly examples: 24-Dec-25 because 25-Dec-25 is a holiday, realigned to 30-Dec-25. | As published 2025-06-25 | - | **Open:** the annexure says "monthly contracts" without separating futures from options. Dates may change for declared holidays. These are scheduled dates, not confirmed actual expiries. The September 2025 monthly futures expiry is not listed. |
| NSE-F-11 | EV-02 p.1 items 1 and 5 | Published schedule | April 2024 monthly expiry 2024-04-25; first monthly expiry with the revised lot 2024-05-30. | As published 2024-04-02 | - | Scheduled, not confirmed actual. |
| NSE-F-12 | EV-03 p.3 annexure | Published schedule | NIFTY monthly expiries 30-Jan-2025 and 27-Feb-2025; weekly 19-Dec-2024 and 02-Jan-2025. | As published 2024-10-18 | - | Scheduled; holiday caveat in the annexure. |
| NSE-F-13 | EV-07 p.2 annexure | Published schedule | NIFTY monthly expiries 30-Dec-2025 and 27-Jan-2026; weekly 23-Dec-2025 and 06-Jan-2026; NIFTY quarterly and half-yearly 31-Mar-2026. | As published 2025-10-03 | - | The 31-Mar-2026 date was later changed by NSE-F-14. |
| NSE-F-14 | EV-08 p.1 table | Dated notice for one month | Contracts with expiry and maturity date 31-Mar-2026 move to 30-Mar-2026, issued pursuant to the 2026 holiday circular (EV-10). Updated contract files to be loaded before trading on 2025-12-29. | Trading systems from 2025-12-29 | - | **A specific dated notice, not a general algorithm.** It names no product: it applies to the contracts of that month. Whether NIFTY futures are included is the natural reading but needs owner review (D-5). Shows the scheduled date, not the last tradable session. |

### 3.4 Current specification snapshot (NSE-EV-09)

| Fact | Section | Rule | Limits |
|---|---|---|---|
| NSE-F-15 | Trading Cycle | NIFTY 50 index futures have three consecutive monthly contracts: near, mid and far month. | **Current rules only**, as of the page's 11/08/2026 update. Does not establish earlier rules or the day a new contract is introduced. |
| NSE-F-16 | Expiry Day | Index futures expire on the Tuesday of the expiry period; if that Tuesday is a trading holiday, the previous trading day. Refers to EV-06. | Current rules only. Does not say whether the expiry day is the last trading day. |
| NSE-F-17 | Price Steps | Index futures tick bands as NSE-F-05, citing EV-04. | Current rules only; no monthly value. |

### 3.5 Calendar (NSE-EV-10)

| Fact | Evidence | Rule | Limits |
|---|---|---|---|
| NSE-F-18 | EV-10 p.1 table | The 15 weekday F&O trading holidays for 2026, including 2026-03-31 (Tuesday) and 2026-11-24 (Tuesday). | Agrees with the existing calendar reference, which adds 2026-01-15 from NSE/FAOP/72262 (not inspected here). The calendar is unchanged. |
| NSE-F-19 | EV-10 p.2 table and footnote | Muhurat trading will be held on Sunday 2026-11-08; its timings are to be notified later. | **Unresolved.** No timing notice was found as of 2026-10-17. The calendar's fail-closed entry stays. |
| NSE-F-20 | EV-10 p.1, with the expiry convention in NSE-F-08/F-09 | 2026-11-24 is a trading holiday and the last Tuesday of November 2026. | **Unresolved:** no NSE notice changing the November 2026 expiry was located. Moving to the previous trading day is the *current* rule on EV-09, but no dated notice for this month was found, and none is inferred. |

### 3.6 Listing

| Fact | Evidence | Rule | Limits |
|---|---|---|---|
| NSE-F-21 | EV-02 p.1 item 3 | New contracts generated at EOD 2024-04-25 are available for trading from 2024-04-26. | Shows when one generation of new contracts was first tradable, without naming them individually. Not a per-contract listing record. |
| NSE-F-22 | EV-06 annexure | New NIFTY contracts are generated at EOD on stated dates (for example, the 28-Oct-25 monthly at 2025-07-31 EOD). | Gives the generation date only. The first trading session is not stated and is not inferred. |

## 4. Missing-evidence matrix (2024-2026)

No monthly contract universe is fabricated here. The rows are the fields
R-1B-1's `HistoricalFuturesContractReference` needs.

| Requirement | What the inspected sources support | Status |
|---|---|---|
| Per-contract listing (first trading) date | NSE-F-21 for one generation; NSE-F-22 gives generation dates only | **UNAVAILABLE** per contract |
| Actual last tradable session | Nothing. No source says the expiry day is the last trading day. | **UNAVAILABLE** |
| Actual expiration date | Scheduled dates for some months (NSE-F-10 to F-13); one dated revision (NSE-F-14); conventions (NSE-F-08, F-09, F-16) | **PARTIAL**: scheduled, not confirmed. Not derived from weekday conventions. |
| Contract-specific lot applicability | Lot rules with transitions (NSE-F-02 to F-04) | **PARTIAL**: needs introduction dates and owner interpretation (D-4) |
| Monthly applicable tick size | Formula only (NSE-F-05); 0.05 before 2025-04-15 at one date (NSE-F-06) | **UNAVAILABLE** after 2025-04-15; **PARTIAL** before |
| Special-session handling | 2024-2026 special sessions are in the calendar reference (circulars cited there, not reinspected here); Muhurat 2026 timings unnotified (NSE-F-19) | **PARTIAL**: Muhurat 2026 unresolved |
| November 2026 expiry | NSE-F-20 | **UNRESOLVED** |
| Authoritative per-contract provenance | None. NSE names the contract master files (section 5). | **UNAVAILABLE** |
| Years before 2024 | NSE-F-01 only | **UNAVAILABLE** (calendar and contract facts) |

Consequently no complete 2024-2026 NIFTY futures contract can be built, and
none should be.

## 5. Contract-master feasibility checklist

NSE's circulars (EV-02, EV-03, EV-04, EV-07, EV-08) tell members to load
updated contract files, including the MII contract file named
`NSE_FO_contract_ddmmyyyy.csv.gz`, from the member extranet or the NSE
reports page `https://www.nseindia.com/all-reports-derivatives`. EV-04 p.2
states that the applicable index tick size appears in the contract and spread
master files, and that the MII files are put on the website daily. **No
contract master file was retrieved or inspected.** A file name says nothing
about whether old files can be downloaded or whether the owner may keep them.

| # | Question | Answer from inspected documentation |
|---|---|---|
| 1 | Are historical daily contract master files available for 2024 to 2026? | UNKNOWN. EV-04 indicates that current files are published daily; historical availability is not stated. |
| 2 | Does a file identify the first trading date, or can it be established from an authorized complete daily history? | UNKNOWN |
| 3 | Does it identify expiration and last tradable session separately? | UNKNOWN |
| 4 | Does it give contract-specific tick size? | Stated for the current files: EV-04 p.2 says the applicable index tick is in the contract and spread master files. Historical files: UNKNOWN. |
| 5 | Does it give lot size? | UNKNOWN. The lot circulars ask members to load updated contract files at each revision, which suggests it but does not confirm the field. |
| 6 | Are monthly tick revisions reflected? | UNKNOWN |
| 7 | Are file versions and corrections identified? | UNKNOWN |
| 8 | Does it contain market-price fields (for example base prices) needing separate treatment? | UNKNOWN |
| 9 | May the owner keep these files locally for private research? | UNKNOWN |
| 10 | May private backups and derived reference records be kept? | UNKNOWN |
| 11 | What happens when any access entitlement ends? | UNKNOWN |

## 6. Owner review gate

1. **Identify.** A developer records the source: ID, issuer, URL, dates,
   retrieval, fingerprint.
2. **Extract.** The developer records one specific fact with its page or
   annexure reference, as an original summary.
3. **Qualify.** The developer records the fact's applicability (product,
   contract class, boundaries, inclusive or EOD semantics) and every open
   interpretation.
4. **Review.** The owner reads the fact against the cited source.
5. **Decide.** The owner records ACCEPTED, REJECTED or DEFERRED, with date,
   reference purpose and notes, as a dated entry in this register. The
   developer never records an owner decision without the owner's explicit
   confirmation.
6. **Eligible.** Only ACCEPTED facts may enter a later version-controlled
   reference catalog, for the purpose stated in the acceptance.
7. **Tested.** Regression tests show each implemented value equals the
   accepted fact, with its evidence and fact IDs.

Limits:

- Owner review is a governance step, not cryptographic source
  authentication.
- A citation string is not approval, and no code may treat it as one.
- A fingerprint proves byte identity with an observed document, not that NSE
  issued or signed it.
- Version control records who changed what. It is not independent
  verification.
- There is no automatic VERIFIED classification and no self-approval.

### 6.1 Implemented gate (R-1B-2B-2, 2026-10-18)

The mechanism exists; it accepts nothing. Code:
`northstar-infrastructure/src/northstar_infrastructure/market_data/historical_futures_reference_evidence.py`
and the trust anchor
`northstar-infrastructure/src/northstar_infrastructure/market_data/nse_accepted_reference_evidence.py`.

**What each part is**

- `HistoricalFuturesDocumentIdentity`: evidence ID (`NSE-EV-nn`), NSE
  document ID, an https URL on an official NSE host, and the SHA-256 where
  verified.
- `HistoricalFuturesAcceptedFact` binds one fact ID (`NSE-F-nn`) to:
  - its document;
  - its category;
  - the product and the exact contract;
  - the value;
  - its effective boundaries and their semantics (`SESSION_INCLUSIVE` or
    `AFTER_END_OF_DAY`);
  - the accepted interpretation, limitations and unresolved conditions;
  - the owner's acceptance: decision ID, owner, date and purpose.

  It has no "accepted" field. Building one accepts nothing.
- `HistoricalFuturesAcceptedEvidenceRegistry` is one version of a set of
  facts. It refuses a repeated or conflicting fact ID, and an evidence ID
  naming two documents.
- `PRODUCTION_ACCEPTED_EVIDENCE` is the only production registry. It is
  **empty** (`nse-accepted-evidence/0`).

**What the gate does**

`HistoricalFuturesEvidenceGate` authorizes a value only when an accepted fact
states exactly that category, product, contract and value, within its closed
effective period and with the same boundary semantics. It refuses:

- end-of-day boundaries, until D-3 is decided;
- any fact with an unresolved condition;
- expiry rules, scheduled expiries and product lot or tick rules, which
  never authorize a contract value.

Authority comes only from the gate. It is labelled `nse-accepted-evidence`
only when the registry is the production object itself, and
`test-accepted-evidence` otherwise. It never grants market-data rights.

**Evidence-backed cutoffs**

`HistoricalFuturesEvidenceCutoffSource` implements the existing Application
cutoff port:

- The last session is the accepted LAST_TRADABLE_SESSION fact's own value
  for exactly that contract. The caller supplies no dates.
- It is checked against the NSE calendar resolver.
- A scheduled expiry, an expiry rule, or a lot or tick fact never stands in
  for it, and nothing is derived from the expiration date.
- No accepted fact means no cutoff. Two accepted facts for one contract, or
  an unverifiable session, fail closed.

Today it returns no cutoff for any contract.

**What is unchanged**

- The R-1B-1 synthetic path, including its `synthetic-reference` authority.
- `HistoricalFuturesReferenceKind`, still only `SYNTHETIC`. A full
  `HistoricalFuturesContractReference` cannot yet be built from accepted
  evidence; only an evidence-backed cutoff can (section 9).

**Trust boundary.** The trust boundary is code review of the trust anchor, an
identity check on the production registry object, and a private token
required to build an authority. None of it is cryptographic. Anyone able to
edit the code can change it, so review of `nse_accepted_reference_evidence.py`
is the control.

**Adding an accepted fact later.** Only after the owner accepts that exact
fact:

1. Identify the official document (section 2).
2. Extract the specific fact (section 3).
3. Resolve its scope and uncertainty. A fact with an unresolved condition is
   not usable.
4. Present the exact fact, scope and interpretation to the owner.
5. The owner explicitly accepts or rejects it.
6. Record the decision here: decision ID, date, purpose and the accepted
   boundaries.
7. Add only that fact to `PRODUCTION_ACCEPTED_EVIDENCE` and bump its version.
8. Add a regression test pinning the fact against this register.
9. Record any later correction or revision as a dated note here and as a new
   registry version; never rewrite an earlier one.

## 7. Owner decisions required

D-1 and D-2 were approved on 2026-10-18 (section 7.1). The others have not
been made. Each is recorded here when the owner confirms it.

| ID | Decision | State |
|---|---|---|
| D-1 | Accept NSE F&O circulars, read from NSE's official archive, as the authority for historical contract reference facts. | APPROVED 2026-10-18 (7.1) |
| D-2 | Adopt the review gate in section 6. | APPROVED 2026-10-18 (7.1) |
| D-3 | How EOD wording maps to effective periods (first affected session = next trading session). | PENDING |
| D-4 | Whether NSE-F-02 changed the lot of contracts already listed before 2024-04-26 (a mid-life change). | PENDING |
| D-5 | Whether NSE-F-14 applies to NIFTY futures. | PENDING |
| D-6 | Whether "expiry day equals last tradable session" may be accepted, and on what evidence. None was found. | PENDING |
| D-7 | The per-contract source to pursue: the contract master (section 5), a licensed NSE reference product, or another authorized source, after a terms review. | PENDING |
| D-8 | How R-2 treats monthly index futures tick sizes against F-4's single `price_increment`. | PENDING |
| D-9 | Whether PDF copies, quotations or only identifiers and fingerprints may be kept (section 8). | PENDING |

### 7.1 Recorded decisions

Recorded 2026-10-18 from the product owner's written instruction for
R-1B-2B-2. The owner's wording is reproduced exactly. No signature or other
acceptance timestamp was given, and none is recorded.

**D-1 Primary reference authority: APPROVED.**

> Official NSE circulars, exchange notices and contract specifications are
> acceptable primary authorities for historical Futures reference facts.
>
> However, each fact's actual applicability must be reviewed separately.
>
> An official document does not automatically validate every proposed
> interpretation.

**D-2 Individual fact acceptance: APPROVED.**

> Every extracted reference fact requires the owner's explicit acceptance
> before that fact may enter the verified reference catalog.
>
> Acceptance of one fact does not imply acceptance of related facts, an
> entire document or a derived contract record.

**What these decisions do not do.**

- D-1 and D-2 are governance decisions only. They approve none of
  NSE-F-01 to NSE-F-22, each of which remains PENDING.
- Accepted facts: 0. Verified historical contracts: 0.
- D-3 to D-9 remain PENDING.
- G-1 (Upstox written data-use permission) remains unresolved, and no
  real-market data use is authorized.

## 8. Source use and retention

- **Terms not reviewed.** NSE's website terms of use, and any terms covering
  the reuse of its circulars, have not been reviewed. Until they are, this
  register holds identifiers, URLs, fingerprints and original summaries only.
- **Copies not kept.** The retrieved copies were temporary engineering-session
  files and are not part of any repository.
- **Reference documents are not market data.** Reading or keeping exchange
  reference circulars grants no right to retain historical OHLCV. NSE contract
  master files may contain price-like fields (question 8) and need the same
  terms review before any retrieval.
- **G-1 unresolved.** G-1 (Upstox written data-use permission) remains
  unresolved. Private, single-user use is not assumed to permit unrestricted
  access or retention.
- **Not authorized by this register:** downloading OHLCV, importing bhavcopy
  or contract master files, authorized-data evaluation, and any production
  configuration change.

## 9. Compatibility with existing contracts

- `HistoricalFuturesReferenceSource(kind, citation)`: today `kind` can only
  be `SYNTHETIC`. A later accepted-evidence catalog would cite
  `NSE-EV-nn/NSE-F-nn` and carry a reviewed kind. That kind does not exist
  and is not added here. Citation text alone never makes a source
  authoritative (R-1B-1).
- `HistoricalFuturesContractReference` needs a listing date, a last tradable
  session and tick and lot periods, each sourced. Section 4 shows none of
  these is available per contract, so no real contract entry is possible.
  Lot and tick periods can already represent mid-life changes (NSE-F-02,
  F-03, F-04) and monthly ticks (NSE-F-05).
- `HistoricalFuturesContractReferenceCatalog.version` would name the accepted
  catalog, and an R-1A manifest's `coverage.contract_reference` would cite
  it. Completeness stays relative to the named references, as R-1A states.
- `HistoricalFuturesContractCutoffSource` resolves a last session only
  through the NSE calendar resolver. Its `synthetic-reference` authority is
  unchanged. An accepted-evidence authority would name the evidence ID and
  fingerprint.
- The NSE calendar reference already cites circulars per fact, and EV-10
  agrees with it. No calendar change is made.
- R-0 remains SYNTHETIC-only.

> **Note, 2026-10-18 (R-1B-2B-2).**
>
> **What exists now:**
> - The evidence-linked authority this section anticipated now exists for
>   cutoffs only: `HistoricalFuturesEvidenceCutoffSource` names the registry
>   version, fact, evidence, document, fingerprint and decision (section 6.1).
> - Its production registry is empty, so it resolves no cutoff.
>
> **What did not change:**
> - No reviewed kind was added to `HistoricalFuturesReferenceKind`. Adding a
>   public member would let a source be built from strings alone.
> - A full `HistoricalFuturesContractReference` from accepted evidence (listing,
>   lot and tick periods) therefore remains future work. That work must keep
>   the gate at the catalog boundary.
> - R-0 and R-1A semantics are unchanged.
