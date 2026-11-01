# NSE Cash-Equity Session Evidence Register (EI-1C-1)

Status: **evidence register; no fact is verified or accepted.**

- Opened 2026-11-02 for EI-1C-1.
- **No NSE source was retrieved in this task.** Every NSE source below is SOURCE NOT RETRIEVED, and every fact is at most DISCOVERED.
- Accepted facts: **0**. Production session reference data: **none**.
- The only equity session resolver in Northstar is the synthetic one (EI-1A). It refuses non-synthetic references by design.

> **Note, 2026-11-02 (EI-1C-1B).**
> - The owner supplied the official NSE URLs for five sources, and the findings of an external review of them (section 13).
> - **No NSE document was inspected in this task.** The register's manual-retrieval rule (section 2, rule 5) and the Futures register's terms note (its section 8.1) rule out automated requests to NSE hosts. No manually downloaded copy was available in the workspace.
> - The sources are now SOURCE LOCATED (owner, official URL). The external-review findings are recorded as REPORTED, not as evidence.
> - Every fact remains **DISCOVERED**. Verified facts: **0**. Accepted facts: **0**.
> - Verification is **blocked** until the owner supplies manually downloaded copies (section 13.4).

> **Note, 2026-11-02 (EI-1C-1B, local source inspection).** This note supersedes the status lines above.
> - The owner supplied six manually obtained copies: NSE/CMTR/75479, 76941, 71775 and 72260, and browser prints of the CAS page (CE-S-03) and the market-timings page (CE-S-02). Each was inspected and fingerprinted (section 14). No NSE host was contacted, and no copy is committed.
> - **Verified facts: 10**, all awaiting owner acceptance. **Accepted facts: 0.** Production session reference data: **none**.
> - Normal-day continuous hours for CAS-eligible securities are still **not** verified (CE-F-12). No normal trading day can yet be resolved from verified facts.

This register records what Northstar needs to know before an evidence-backed NSE cash-equity session resolver can exist:

- where the authoritative NSE material is believed to be;
- which candidate facts have been discovered, and from what;
- what each fact would need before it can be accepted.

It does not establish any NSE trading rule.

Related documents:

- [Equity Intraday V1 Product Specification](../product/Equity-Intraday-V1-Product-Specification.md), sections 6.1 to 6.7 and the owner decision register
- [NSE Historical Futures Reference Evidence Register](NSE-Historical-Futures-Reference-Evidence-Register.md), the governance precedent
- [ADR-014 Canonical Option Daily Market Data and Session Semantics](../adr/ADR-014-Canonical-Option-Daily-Market-Data-and-Session-Semantics.md), the precedent for refusing borrowed or back-projected hours
- EI-1A contracts:
  - `northstar-application/src/northstar_application/ports/equity_trading_session_resolver.py`
  - `northstar-application/src/northstar_application/application_services/equity_session_interval_grid.py`
  - `northstar-infrastructure/src/northstar_infrastructure/market_data/synthetic_equity_session.py`

## 1. Purpose and scope

**In scope.** These NSE **Capital Market (cash equity) segment** session facts, for the EQ series:

- normal continuous-market hours;
- the pre-open session;
- the Closing Auction Session (CAS) and which securities it applies to;
- effective dates of rule changes;
- trading holidays;
- special sessions, including Muhurat trading;
- unscheduled halts and session revisions.

**Out of scope:**

- Futures and Options session facts, except as labelled cross-references;
- market-data licensing;
- historical prices;
- live calendars;
- any code or contract change.

## 2. Source authority rules

These rules follow the Futures register's precedent (its section 1 and governance decisions D-1 and D-2). That precedent has **not** been approved for cash equities. Adopting it here is owner decision CE-D1 (section 9).

1. **Primary authority.** A circular from NSE's Capital Market segment (`NSE/CMTR/...`) or an official NSE web page is primary authority, and only when Northstar has read it. SEBI circulars are primary authority for the regulatory framework those circulars implement.
2. **Segment matters.** An F&O circular (`NSE/FAOP/...`) is **not** evidence for a cash-segment fact. Each segment's holidays, hours and special-session timings are separate exchange notices and are verified separately. An F&O fact may only corroborate, and is labelled as such.
3. **Secondary material locates documents only.** Search-engine summaries, broker notices, news articles and third-party holiday lists may be used to *locate* a document. They are never evidence for a fact, and any figure taken from them is marked SECONDARY.
4. **No inference from titles.** A circular's content is never inferred from its title, its number or a summary of it.
5. **Manual retrieval only.** The Futures register (section 8.1) records that NSE's website Terms of Use prohibit automated collection, and that future retrieval should be manual until NSE clarifies.
   - This task therefore made **no** automated request to any NSE host.
   - NSE documents must be obtained manually, for example downloaded by the owner in a browser, and then inspected.
6. **No source text is reproduced, and no copies are committed.** Facts are original summaries with page or section references, as in the Futures register.
7. **No back-projection.** A rule is applied only from its stated effective date. Hours in force in 2026 are never assumed for earlier dates, and the same applies to every other rule.
8. **Append only.** Entries are corrected by dated notes, never rewritten.

### 2.1 Evidence states

| State | Meaning |
|---|---|
| SOURCE LOCATED | An identifier or URL for the document is known. It has not been opened. |
| SOURCE NOT RETRIEVED | The document was not obtained in this task. |
| DISCOVERED | A candidate fact is known only from a located-but-unread source, a secondary source, or a cross-segment reference. It is not evidence. |
| VERIFIED FROM SOURCE | The fact was read in a primary source that Northstar inspected, at a recorded page or section. |
| PENDING ACCEPTANCE | The fact is verified and awaits the owner's review. |
| ACCEPTED | The owner explicitly accepted the fact for a stated reference purpose. Only accepted facts may enter a production reference module. |

Being official does not make a source accepted. Being verified does not make a fact accepted.

## 3. Official source inventory

All NSE sources are **SOURCE NOT RETRIEVED**. "Located via" says how each identifier became known. The dates and descriptions come from that locating material and are themselves unverified.

### 3.1 Official NSE pages (discovery references named by the owner)

| ID | Page | URL | State |
|---|---|---|---|
| CE-S-01 | Holidays and exchange communication | https://www.nseindia.com/resources/exchange-communication-holidays | SOURCE LOCATED (owner); NOT RETRIEVED |
| CE-S-02 | Market timings | https://www.nseindia.com/static/market-data/market-timings | SOURCE LOCATED (owner); NOT RETRIEVED |
| CE-S-03 | Closing Auction Session | https://www.nseindia.com/static/products-services/closing-auction-session | SOURCE LOCATED (owner); NOT RETRIEVED |
| CE-S-04 | Equity market pre-open | https://www.nseindia.com/static/products-services/equity-market-pre-open | SOURCE LOCATED (owner); NOT RETRIEVED |

Live web pages describe current rules only. Effective dates for history must come from dated circulars, as with NSE-EV-09 in the Futures register.

### 3.2 Capital Market circulars

Archive URLs follow the pattern `https://nsearchives.nseindia.com/content/circulars/CMTR<number>.pdf`. The pattern was seen in search results; the documents were not opened.

| ID | Reference | Date (as located) | Subject (as located; unverified) | Located via | State |
|---|---|---|---|---|---|
| CE-S-10 | NSE/CMTR/72394 | 2026-01-19 | CAS | Owner | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-11 | NSE/CMTR/73362 | 2026-03-18 | CAS | Owner; search result | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-12 | NSE/CMTR/73845 | 2026-04-22 | CAS | Owner | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-13 | NSE/CMTR/74466 | 2026-05-29 | CAS | Owner; secondary citation | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-14 | NSE/CMTR/74969 | 2026-07-01 | CAS or pre-open timings (cited by secondary material) | Search summary | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-15 | NSE/CMTR/75479 | 2026-07-30 | Equity-segment trading-modality changes for CAS live from 2026-08-03 | Search result and summary | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-16 | NSE/CMTR/75401, 75366, 74982, 74792 | not established | Appeared in CAS-related search results; relevance unknown | Search results | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-17 | NSE/CMTR/71775 | 2025-12-12 or 2025-12-15 (sources conflict) | Capital Market trading holidays, calendar year 2026 | Secondary only | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-18 | NSE/CMTR/72260 | 2026-01-12 | Adds Thursday 2026-01-15 as a Capital Market holiday; partially modifies 71775 | Search result and summary | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-19 | NSE/CMTR/76941 | 2026-10-19 | Special live session for Diwali Muhurat trading, Sunday 2026-11-08, Capital Market segment | Search result; secondary reports | SOURCE LOCATED; NOT RETRIEVED |

### 3.3 Related non-CMTR references

| ID | Reference | Relevance | State |
|---|---|---|---|
| CE-S-30 | NSE/CMPT/74898 | Appeared with CAS search results; possibly clearing or risk modalities for CAS (unverified) | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-31 | NSE/FAOP/76940 (2026-10-19) | Possibly the F&O counterpart of the Muhurat notice. Cross-segment only. | SOURCE LOCATED; NOT RETRIEVED |
| CE-S-32 | SEBI circular on CAS (secondary material dates it January 2026) | Regulatory framework | Identifier NOT LOCATED |
| CE-S-33 | SEBI circulars on market-wide circuit breakers | Halts (section 7.3) | Identifier NOT LOCATED |

### 3.4 Cross-segment evidence already held by Northstar

These were read in earlier Futures and Options work and are cited in `nse_futures_calendar_reference.py`, `nse_option_session_reference.py`, ADR-014 and the Futures register. They are **F&O-segment** facts. Here they are cross-references only, and none is evidence for the cash segment.

| ID | Reference | F&O fact as recorded in Northstar |
|---|---|---|
| CE-X-01 | NSE/FAOP/74467 (2026-05-29) | F&O normal-market close 15:40, effective 2026-08-03 |
| CE-X-02 | NSE/FAOP/75472 (2026-07-30) | CAS-related F&O modalities live from 2026-08-03 |
| CE-X-03 | NSE/FAOP/74970 (2026-07-01) | F&O pre-open phases revised from 2026-09-07; 09:00 start and 09:15 open unchanged |
| CE-X-04 | NSE/FAOP/71777 (2025-12-12), Futures NSE-EV-10 | 2026 F&O trading holidays; Muhurat on Sunday 2026-11-08 with timings to be notified (Futures NSE-F-19) |
| CE-X-05 | NSE/FAOP/72262 (2026-01-12) | F&O holiday added on 2026-01-15 (cited by the calendar reference; not reinspected in the Futures register) |

## 4. Candidate evidence facts

Common to every fact below:

- the state is **DISCOVERED**;
- nothing is VERIFIED FROM SOURCE;
- owner acceptance is PENDING;
- the segment is NSE Capital Market, EQ series, unless stated otherwise.

"SECONDARY" marks a figure taken from non-authoritative material. It is recorded only to direct manual verification and must not be used.

### 4.1 Normal continuous market and pre-open

| Fact | Candidate rule | Applicability | Effective | Basis | Needed for VERIFIED |
|---|---|---|---|---|---|
| CE-F-01 | The normal market opens at 09:15 IST. | All CM securities in continuous trading | Not established | EI-0 section 6.1 (widely published; AWAITING OFFICIAL VALIDATION); SECONDARY | CE-S-02 plus a dated circular per regime |
| CE-F-02 | Before 2026-08-03, the normal market closed at 15:30 IST for all continuously traded EQ securities. | EQ, continuous trading | Before the CAS go-live; start of regime not established | SECONDARY; the F&O analogue is in CE-X-01 | Dated CM circular(s); the regime's start date for the reference period |
| CE-F-03 | The pre-open call auction precedes 09:15 and is not continuous trading. It is excluded from the 5-minute grid. | CM | Not established | CE-S-04 (not retrieved); EI-0 section 6.3 | CE-S-04 plus the dated circular defining its phases |
| CE-F-04 | The CM pre-open phases were restructured with effect from 2026-09-07. | CM | 2026-09-07 | SECONDARY; the F&O analogue is in CE-X-03 | The CM circular itself; whether the 09:15 open changed |

### 4.2 Closing Auction Session

| Fact | Candidate rule | Applicability | Effective | Basis | Needed for VERIFIED |
|---|---|---|---|---|---|
| CE-F-10 | CAS was introduced in the CM segment, with trading-modality changes live from 2026-08-03. | CAS-eligible securities only | From 2026-08-03 | CE-S-15 (located); corroborated cross-segment by CE-X-01 and CE-X-02 | CE-S-15 and the CAS circulars CE-S-10 to CE-S-14 |
| CE-F-11 | CAS eligibility is limited to F&O-eligible stocks in the first phase. One secondary source says about 220 stocks, with active F&O contracts on both NSE and BSE. | Instrument level | From 2026-08-03 | SECONDARY only | The circular defining eligibility, and how the list is published and revised |
| CE-F-12 | For CAS-eligible securities, continuous trading ends **before** 15:30. One secondary source gives 15:15, with CAS order entry about 15:20 to 15:30 and matching to about 15:35. | CAS-eligible only | From 2026-08-03 | SECONDARY, conflicting: another summary of a *mock-trading* schedule shows both closes at 15:30 | CE-S-13, CE-S-14, CE-S-03 |
| CE-F-13 | Non-eligible securities continue normal trading until 15:30 IST. | Non-CAS EQ securities | From 2026-08-03 | Owner brief (describing CE-S-03); SECONDARY | CE-S-03 and the eligibility circular |
| CE-F-14 | CAS and any post-close session are not continuous trading. They are excluded from the 5-minute grid. | All | — | Northstar policy (EI-0 sections 6.3 and 6.5), not an exchange fact | Owner acceptance only |

### 4.3 Holidays

| Fact | Candidate rule | Effective | Basis | Needed for VERIFIED |
|---|---|---|---|---|
| CE-F-20 | 2026 CM trading holidays are listed in NSE/CMTR/71775. | Calendar 2026 | CE-S-17 (located; date conflicts) | Retrieve CE-S-17 and record each date with its page |
| CE-F-21 | Thursday 2026-01-15 is an added CM holiday (Maharashtra municipal elections). | 2026-01-15 | CE-S-18 (located); corroborated cross-segment by CE-X-05 | Retrieve CE-S-18 |
| CE-F-22 | Every later amendment to the 2026 CM holiday list is known. | — | **Not established.** No search for amendments after 2026-01-12 was exhaustive. | A complete review of CM circulars for 2026 |

No date-by-date 2026 CM holiday list is recorded here. Third-party lists were seen and are not evidence. The F&O list (CE-X-04) is not substituted for the CM list.

### 4.4 Special sessions

| Fact | Candidate rule | Effective | Basis | Needed for VERIFIED |
|---|---|---|---|---|
| CE-F-30 | A Muhurat trading session is held on Sunday 2026-11-08. | 2026-11-08 | CE-S-19 (located); corroborated cross-segment by CE-X-04 | Retrieve CE-S-19 |
| CE-F-31 | Muhurat 2026 timings: secondary reports give a normal market of about **14:20 to 15:30 IST**, with separate pre-open and closing windows. **SECONDARY, unverified; not to be used.** | 2026-11-08 | Secondary reports of CE-S-19 | Retrieve CE-S-19 and record every phase, including CAS applicability on that day |
| CE-F-32 | Other 2026 CM special sessions (for example the Union Budget session on Sunday 2026-02-01, or mock or DR sessions with live trading) are known. | — | **Not established.** The F&O calendar reference lists F&O special sessions; the CM equivalents were not located. | A CM circular per session |

### 4.5 Halts, outages and session revisions

| Fact | Candidate rule | Basis | Needed for VERIFIED |
|---|---|---|---|
| CE-F-40 | Index-based market-wide circuit breakers can halt trading intraday, and trading may resume with a pre-open. | General knowledge; no source located (CE-S-33) | The SEBI framework circular and the NSE implementation circular |
| CE-F-41 | NSE has extended or revised sessions after technical outages, by notices issued on the day. | General knowledge; no source located | Per-event notices; such events cannot be known in advance |
| CE-F-42 | Individual securities may be suspended or traded only in periodic call auctions, so they do not trade continuously. | General knowledge; EI-0 sections 5.3 and 6.2 | Surveillance and call-auction circulars; an instrument-level source |

## 5. Effective dates and applicability

- **No regime is established for any date.** Nothing above is VERIFIED FROM SOURCE, so a production reference could not resolve any date today. That is the intended fail-closed state.
- **Periods the evidence must cover** (candidate boundaries, all DISCOVERED):

| Period | What is needed |
|---|---|
| Before 2026-08-03 | The CM normal-market hours regime in force, with its start date. Not projected from later rules. |
| From 2026-08-03 | Hours differ by CAS eligibility (CE-F-10 to CE-F-13). A per-instrument, per-date eligibility source is required. |
| From 2026-09-07 | Possible pre-open restructuring (CE-F-04). Whether the 09:15 continuous open changed must be confirmed. |
| Special days | Each needs its own notice. A missing or unread notice means the date fails closed. |

- **Required fields for each accepted fact:**
  - exact rule;
  - segment;
  - applicability scope;
  - instrument applicability, where relevant;
  - effective-from date;
  - effective-through date, or "until superseded";
  - source ID and reference number;
  - page or section;
  - amendments or superseding source;
  - verification state;
  - remaining uncertainty.

## 6. CAS versus non-CAS analysis

1. **When CAS applies.** Candidate: from 2026-08-03 (CE-F-10). This is DISCOVERED only.
2. **Which securities are eligible.** Candidate: F&O-eligible stocks in the first phase (CE-F-11). This is SECONDARY only. How the list is published, its form, and how often it is revised are **unknown**.
3. **How eligibility changes.** Unknown. Stocks entering or leaving F&O, later phases, and suspensions could all change it. Each change needs an effective date.
4. **Eligibility on a historical date.** It can be established only if NSE publishes dated eligibility lists or effective-dated changes that Northstar can retrieve and keep under the applicable terms. This is not yet shown, and it is the main open question for EI-1C.
5. **Is the exchange/date-only resolver sufficient?** **No, not for the close of continuous trading after 2026-08-03.**
   - Under CE-F-12 and CE-F-13, two EQ securities on the same date can have different continuous-market ends.
   - `EquityTradingSessionResolver.resolve(exchange_code, trading_date)` returns one set of phases per venue and date. It cannot express that difference.
   - It does remain sufficient for venue-level facts: whether a date is a session, holidays, special sessions, and the opening phase.
6. **Where instrument applicability belongs.** In a **separate, instrument-level reference lookup**, not inside the venue session port.
   - The session stays the venue's envelope.
   - An eligibility lookup answers, for a listing and session date, one of `CAS_ELIGIBLE`, `NOT_CAS_ELIGIBLE` or `UNKNOWN`, with its evidence.
   - A pure Application rule derives the listing's continuous phases from both answers.
   - This keeps EI-1A's port and grid unchanged, and keeps provider- and instrument-specific data out of the venue calendar.
   - It is a proposal for a separate design decision (CE-D5), not a change made here.
7. **When eligibility is unknown.** Recommended: fail closed for that listing and date. No grid, no candle expectations and no research for the window that depends on the unknown close. Recorded as reason `SESSION_PHASE_NOT_SUPPORTED`, already proposed in EI-0 section 7.5.
   - **Alternative for owner consideration (CE-D6):** expose only intervals that end before the earliest *verified* continuous-market end that applies to both groups.
     - The EI-0 proposed cutoffs (signal 14:25, entry 14:30, planned exit 15:00 IST) all fall before any reported CAS boundary.
     - This alternative could therefore serve V1 without knowing eligibility, but only once that common boundary is VERIFIED FROM SOURCE.
     - It must not be used on the strength of the SECONDARY 15:15 figure.

Neither "every equity trades continuously until 15:30" nor "every equity stops at 15:15" may be assumed. Both are rejected here.

## 7. Holidays and special-session coverage

### 7.1 Date classes a production reference must distinguish

| Class | Source required | EI-1A representation |
|---|---|---|
| Ordinary weekday session | The holiday circular (to confirm it is not a holiday) and the hours regime | Normal session |
| Official trading holiday | The CM holiday circular and amendments | Closed date with reason |
| Weekend without a session | The holiday circular (weekend holidays are listed separately) and the absence of a special-session notice | Closed date with reason. No weekday arithmetic: the reference states the date. |
| Special session on a weekend or holiday | A dated special-session notice with timings | Special session, which overrides closed |
| Special session with modified hours | Same | Special session with its own phases |
| Announced but untimed special session | The notice of the date | Special session without timings, which raises |
| Later notice altering the schedule | The amending notice | A new reference version. The earlier version stays reproducible. |

### 7.2 Muhurat, 2026-11-08

- **What is known:**
  - The F&O holiday circular (CE-X-04) announced the session with timings "to be notified".
  - A CM notice dated 2026-10-19 (CE-S-19) has been **located but not read**.
- **Timings:** secondary reports of the timings (CE-F-31) are **not used**.
- **Until CE-S-19 is retrieved and its phases verified,** 2026-11-08 must remain a special session without established timings, and it must fail closed.
- **V1 trading:** EI-0 proposes that V1 does not trade special sessions (EI0-D7).

### 7.3 Unscheduled events

- Halts, outages and session extensions are not knowable in advance, so no session reference can encode them ahead of time.
- **Proposed handling (CE-D8):**
  - The reference holds the scheduled session only.
  - Gaps in actual candles are caught by EI-2 completeness checks and resolve as `INCOMPLETE_EVIDENCE` (EI-0 section 8.2).
  - A later NSE notice that revises a past session is recorded as a new reference version, citing the notice. It never edits the earlier version.

## 8. Conflicting or obsolete documentation

| ID | Conflict | Treatment |
|---|---|---|
| CE-C-01 | **Muhurat 2026 timings.** The Futures register's NSE-F-19 records that no timing notice was found as of 2026-10-17. CE-S-19 and CE-S-31 are dated 2026-10-19. | The Futures record is not modified here. It was correct when written. Whether the Futures calendar's fail-closed entry for 2026-11-08 can now be resolved is a separate F&O task. |
| CE-C-02 | **2026 CM holiday circular identity.** Secondary sources give NSE/CMTR/71775 dated 12 December 2025, and also a holiday circular dated 15 December 2025 with "Ref. No. 67/2025". | Unresolved until CE-S-17 is retrieved. |
| CE-C-03 | **CAS timings.** One secondary source gives a 15:15 continuous end and a 15:15 to 15:35 auction. A mock-trading schedule summary shows both closes at 15:30. | Mock schedules are not live rules. Unresolved until the CAS circulars are read. |
| CE-C-04 | **2026 holiday count:** 15, 16 or 20 in different summaries (weekday-only, with the 2026-01-15 addition, or with weekend holidays). | It doesn't matter once each date is recorded from the circular. Counts are not evidence. |
| CE-C-05 | **EI-0 section 6.1** lists one normal market of 09:15 to 15:30 IST (marked AWAITING OFFICIAL VALIDATION). It predates this analysis and does not reflect CAS-eligibility-dependent closes from 2026-08-03. | Not edited in this task. A specification correction is proposed after the CAS facts are verified (section 10). |
| CE-C-06 | **Segment timings.** F&O closes at 15:40 from 2026-08-03 (CE-X-01). That figure must never be read as a cash-equity close. | Recorded so that no one borrows it. |

> **Note, 2026-11-02 (EI-1C-1B): progress on the conflicts above.** No conflict is resolved, because no source was inspected.
> - **CE-C-01:** the owner located NSE/CMTR/76941 at its official archive URL. An external review reports that it gives official timings for the 2026-11-08 Muhurat session. That would supersede the "timings to be notified" position, but only once the circular is inspected. Futures NSE-F-19 is still not modified.
> - **CE-C-02:** the owner located NSE/CMTR/71775 at its official archive URL, and an external review reports it as the base 2026 Capital Market holiday list. The identity of the base circular is narrowed to 71775. Its publication date, and the meaning of "Ref. No. 67/2025", stay unresolved until it is inspected.
> - **CE-C-03:** an external review reports that the official CAS page distinguishes continuous trading for CAS-eligible and non-CAS securities. Whether the 15:30 figures in NSE/CMTR/75479 belong to mock sessions or to live rules is now an explicit inspection item (section 13.3, item G). It stays unresolved.
> - **CE-C-04 and CE-C-05:** unchanged.
> - **CE-C-06:** unchanged, and still applies.

> **Note, 2026-11-02 (EI-1C-1B, local source inspection): conflicts after inspecting the sources.** Details are in section 14.7.
> - **CE-C-01: resolved for the cash segment.** NSE/CMTR/76941 gives official Muhurat timings (CE-F-31). Futures NSE-F-19 is still not modified, and its F&O counterpart (CE-S-31) was not inspected.
> - **CE-C-02: resolved.** The base circular is NSE/CMTR/71775, dated 2025-12-12, circular reference 172/2025. "Ref. No. 67/2025" is not this circular's number, and no fact depends on it.
> - **CE-C-03: resolved as to mock versus live.** Every clock in NSE/CMTR/75479 belongs to the 2026-08-01 mock session (CE-F-19). The normal-day close for CAS-eligible securities is still unverified (CE-F-12).
> - **CE-C-04: resolved.** There are 15 weekday holidays in 71775, plus 1 added by 72260, giving 16. Another 4 weekend holidays are listed separately, giving 20 entries in all.
> - **CE-C-05:** unchanged. The correction to EI-0 section 6.1 is now confirmed as needed.
> - **CE-C-06:** unchanged. The CAS page also gives the derivatives segment 09:15 to 15:40, which is consistent with CE-X-01 and is still never a cash fact.
> - New conflicts **CE-C-07 to CE-C-09** are in section 14.7.

## 9. Unresolved decisions

These are owner decisions for EI-1C. All are **PENDING**.

| ID | Decision | Recommendation |
|---|---|---|
| CE-D1 | Accept NSE Capital Market circulars read from NSE's official archive, plus official NSE pages for current rules only, as the primary authority for cash-equity session facts. This is the analogue of Futures D-1. | Accept |
| CE-D2 | Adopt a per-fact review gate for cash-equity session facts: VERIFIED FROM SOURCE, then PENDING ACCEPTANCE, then ACCEPTED, as in Futures D-2. | Accept |
| CE-D3 | Retrieval method and retention. The owner obtains the listed documents manually; Northstar records identifiers, fingerprints and original summaries only. Retention is aligned with the pending Futures D-9. | Manual retrieval; no committed copies |
| CE-D4 | F&O circulars may corroborate but never establish a cash-equity fact. | Accept |
| CE-D5 | Represent CAS applicability as a separate instrument-level eligibility lookup, outside the venue session port. This is an Application design decision (section 11). | Separate lookup |
| CE-D6 | Behaviour when CAS eligibility is unknown: fail closed per listing and date, or use a verified common-boundary window. | Fail closed, until the common boundary is verified and approved |
| CE-D7 | Historical coverage the first production reference must support. For example, only dates from 2026-09-07 (the current regime), or also from 2026-08-03 or earlier. This decides how many regimes must be verified. | Start at the most recent verified regime; extend only with evidence |
| CE-D8 | Unscheduled halts and revisions are handled as described in section 7.3. | Accept |
| CE-D9 | Special sessions are recorded in the reference so they are never mistaken for holidays or normal days, even though V1 does not trade them (EI0-D7). | Accept |
| CE-D10 | Muhurat 2026-11-08: record it as untimed (fail closed) until CE-S-19 is verified; V1 excludes it either way. | Accept |

These do not change any EI0 decision. EI0-D7 (cutoffs and no special sessions) and EI0-D18 (surveillance exclusion) remain as they are.

## 10. Acceptance status and owner review

- **Accepted facts: 0. Verified facts: 0. Sources retrieved in this task: 0.**
- **Production readiness: none.** No resolver may load any fact from this register as verified.
- **Owner actions needed before any fact can be verified:**
  1. Decide CE-D1 to CE-D3.
  2. Manually obtain at least:
     - CE-S-17 and CE-S-18 (2026 holidays);
     - CE-S-19 (Muhurat);
     - CE-S-15 and CE-S-10 to CE-S-14 (CAS);
     - the current versions of CE-S-02, CE-S-03 and CE-S-04.
  3. Hand them to an inspection task. That task records fingerprints and per-fact page references, and moves facts to VERIFIED FROM SOURCE.
  4. Review each verified fact, and record ACCEPTED, REJECTED or DEFERRED.
- **Follow-up after the CAS facts are verified:** correct EI-0 section 6.1 (CE-C-05) to show the eligibility-dependent close.

## 11. Compatibility with EI-1A contracts

| Need | Current EI-1A support | Classification |
|---|---|---|
| Different continuous-market hours by date | Phases are per session. The synthetic reference has one normal phase set, with no dated regimes. | **Infrastructure adapter change.** A production adapter needs effective-dated regimes, like `nse_futures_calendar_reference.regime_for`. |
| Multiple eligible phases in one session | Supported: ordered phases with breaks | No change needed |
| Pre-open, CAS and post-close excluded from the grid | Supported: only stated continuous phases are gridded | No change needed |
| Exchange/date session authority | `EquityTradingSession.reference` carries one identity string | **Reference-data addition** for a reference version id. Per-fact evidence ids are an **adapter change**. |
| Instrument-specific continuous-market end (CAS) | Not representable. The port is venue and date only. | **Application port extension requiring a separate design decision** (CE-D5): a new eligibility lookup plus a phase-derivation rule. EI-1A's port and grid are unchanged. |
| Effective-dated rules | Not in the synthetic reference | **Infrastructure adapter change** |
| Special-session overrides | Supported, including an untimed special session that raises | No change needed |
| Incomplete or disputed reference | Unloaded year, unstated date and untimed special session all raise. A disputed fact is simply not loaded, so its date raises. | **Reference-data rule**, no contract change. Disputed facts stay out of the reference. |
| Reference versioning and as-known-at reproduction | One immutable reference per resolver instance | **Infrastructure adapter change**, for named versions |
| Halts and session revisions | Not representable in advance | Handled outside the reference (section 7.3). No change needed. |

## 12. Proposed smallest EI-1C-2 implementation slice

Nothing can be loaded as production data until facts are ACCEPTED. The smallest useful next slice therefore builds the **gate**, not the data. This mirrors the Futures R-1B-2B-2 acceptance gate.

**EI-1C-2: evidence-gated NSE cash-equity session reference adapter.**

It is Infrastructure only, and has no Application change.

1. **A versioned reference structure**, holding:
   - effective-dated normal-market regimes, as phase sets;
   - closed dates with reasons;
   - special sessions, timed or untimed;
   - per-entry evidence ids that must name ACCEPTED register facts;
   - a reference version.
2. **A resolver** implementing `EquityTradingSessionResolver` with the EI-1A resolution order:
   - it refuses any entry without accepted evidence;
   - it never infers a date from the day of the week;
   - it never projects a regime outside its effective dates.
3. **An empty production registry.** Until facts are accepted, every NSE date raises. Tests use explicitly labelled test evidence, as in the Futures `TEST_EVIDENCE_AUTHORITY_PREFIX` pattern.
4. **No CAS in this slice.** Venue-level phases are limited to what is accepted for *all* EQ securities. The CAS eligibility lookup (CE-D5) is a later slice, EI-1C-3, which starts with an Application design decision.

**Prerequisites:**

- CE-D1 to CE-D3 decided;
- for any real date to resolve, the relevant facts verified and accepted. The slice itself can be built and tested before that.

## 13. EI-1C-1B verification attempt (2026-11-02)

### 13.1 What happened

- **Request.** The owner asked for six official sources to be verified, and supplied their URLs and the findings of an earlier external review.
- **No inspection was possible.**
  - No automated request was made to any NSE host. This follows section 2, rule 5, and the Futures register's section 8.1.
  - The only tool available would also have returned a model-written summary rather than the document's bytes. That cannot support a fingerprinted, page-referenced inspection.
  - No manually downloaded copy was present in the workspace.
- **Result.** Nothing was inspected, so no source or fact moves to VERIFIED FROM SOURCE.

### 13.2 Source states after this task

| Source | Official URL (owner-supplied) | Finding reported by the external review (not evidence) | State |
|---|---|---|---|
| CE-S-15, NSE/CMTR/75479 | https://nsearchives.nseindia.com/content/circulars/CMTR75479.pdf | CAS-related trading changes effective from 2026-08-03 | SOURCE LOCATED (owner, official URL); NOT RETRIEVED |
| CE-S-19, NSE/CMTR/76941 | https://nsearchives.nseindia.com/content/circulars/CMTR76941.pdf | Official timings for the 2026-11-08 Muhurat session | SOURCE LOCATED (owner, official URL); NOT RETRIEVED |
| CE-S-17, NSE/CMTR/71775 | https://nsearchives.nseindia.com/content/circulars/CMTR71775.pdf | Base 2026 Capital Market holiday list | SOURCE LOCATED (owner, official URL); NOT RETRIEVED |
| CE-S-18, NSE/CMTR/72260 | https://nsearchives.nseindia.com/content/circulars/CMTR72260.pdf | Additional Capital Market holiday on 2026-01-15 | SOURCE LOCATED (owner, official URL); NOT RETRIEVED |
| CE-S-03, CAS page | https://www.nseindia.com/static/products-services/closing-auction-session | Continuous trading differs between CAS-eligible and non-CAS securities | SOURCE LOCATED (owner); NOT RETRIEVED |
| CE-S-01, holidays page | https://www.nseindia.com/resources/exchange-communication-holidays | Supplied as "market timing information" | SOURCE LOCATED (owner); NOT RETRIEVED |

**URL correction.** The URL supplied as "market timing information" is the holidays and exchange-communication page, CE-S-01. The market-timings page, CE-S-02 (https://www.nseindia.com/static/market-data/market-timings), was not re-supplied. Both are wanted.

### 13.3 Inspection checklist for the manual copies

For each document, record:

- the SHA-256 and byte size of the copy;
- the circular number, date, segment and page count;
- for every fact, the page or section, the effective date, the applicability, and any amendment or supersession.

Summarise the facts in original words. Quote nothing and commit nothing.

| Item | Question | Source(s) | Facts affected |
|---|---|---|---|
| A | The effective date of CAS-related cash-market changes, and whether it is a LIVE effective date | CE-S-15, CE-S-03 | CE-F-10 |
| B | Continuous-trading hours for CAS-eligible securities on a normal LIVE day, from what date | CE-S-03, CE-S-15, and the CAS circulars CE-S-10 to CE-S-14 if cited | CE-F-12 |
| C | Continuous-trading hours for non-CAS securities on a normal LIVE day, from what date | CE-S-03, CE-S-02 | CE-F-02, CE-F-13 |
| D | Every 2026 Capital Market trading holiday, with weekend holidays separated, plus any special-session footnotes | CE-S-17 | CE-F-20, CE-F-22, CE-F-32 |
| E | The added 2026-01-15 holiday, and exactly which part of 71775 it modifies | CE-S-18 | CE-F-21 |
| F | Every phase of the 2026-11-08 Muhurat session (pre-open, continuous, CAS, any post-close), and whether CAS applies that day | CE-S-19 | CE-F-30, CE-F-31 |
| G | Which clocks in CE-S-15 belong to **mock** sessions and which describe **live** trading. Mock clocks must never enter live reference data. | CE-S-15 | CE-F-12, CE-C-03 |
| H | How CAS eligibility is published (list, criteria, frequency) and whether past membership can be established for a date | CE-S-03, CE-S-10 to CE-S-15 | CE-F-11 |

### 13.4 Copies requested from the owner

Manually downloaded official copies of:

1. NSE/CMTR/75479 (CE-S-15);
2. NSE/CMTR/76941 (CE-S-19);
3. NSE/CMTR/71775 (CE-S-17);
4. NSE/CMTR/72260 (CE-S-18);
5. the CAS page (CE-S-03), saved from a browser with its "updated on" date visible;
6. the market-timings page (CE-S-02), saved the same way;
7. if CE-S-15 or the CAS page cites them, the CAS circulars defining live timings and eligibility (CE-S-10 to CE-S-14).

Place them in a folder outside every repository, for example the engineering-session scratch folder, and name the path in the next task.

### 13.5 CAS eligibility after this task

- **Three separate things, kept separate:**
  - whether the venue has a session on a date;
  - whether a given security is CAS-eligible on that date;
  - the continuous-trading phases that apply to that security on that date.
- **No assumptions:** no EQ security is assumed CAS-eligible, and historical membership is not inferred from today's F&O universe.
- **Historical membership: UNRESOLVED.** No source for past CAS membership has been identified.
- **Behaviour while unknown:** the section 6 recommendation stands. Where eligibility is required and unknown, fail closed for that listing and date (CE-D6).

### 13.6 Totals after this task

| State | Facts |
|---|---|
| DISCOVERED | 18: CE-F-01 to 04, 10 to 14, 20 to 22, 30 to 32, 40 to 42 |
| VERIFIED FROM SOURCE | 0 |
| PENDING ACCEPTANCE | 0 |
| ACCEPTED | 0 |

Seven of the DISCOVERED facts now have an owner-located official source and a reported external-review finding: CE-F-10, 12, 13, 20, 21, 30 and 31. That does not change their state.

CE-D1 to CE-D10 remain **PENDING**.

### 13.7 Minimum evidence for EI-1C-2

- **The adapter itself** (section 12) can be built with test evidence, once CE-D1 to CE-D3 are decided.
- **For any real NSE date to resolve,** at minimum:
  - CE-S-17 and CE-S-18 inspected, giving the 2026 holidays;
  - a continuous-market regime for **all** EQ securities, from items B, C and G;
  - CE-S-19 inspected, for 2026-11-08;
  - each resulting fact ACCEPTED by the owner.
- **Any session after 2026-08-03** additionally needs CE-D5 and CE-D6 decided. Otherwise the per-security close must fail closed.

## 14. EI-1C-1B local source inspection (2026-11-02)

The owner manually obtained six official NSE documents and supplied them as local files. They are held outside every repository.

- No NSE host was contacted in this task.
- Nothing is quoted. Facts are original summaries with page references.
- No copy is committed.

### 14.1 Files inspected

The fingerprint identifies the supplied copy only. For a page print, it identifies that one capture of a page that changes over time.

| Source | Supplied file | Bytes | SHA-256 | Pages | Form |
|---|---|---|---|---|---|
| CE-S-15, NSE/CMTR/75479 | `CMTR75479.pdf` | 241038 | `17e415f619be6a132f86a4edbc127412f44e90f87e838151d86484e2204022c1` | 6 | Circular PDF. Metadata creation time 2026-07-30 18:16 IST. |
| CE-S-19, NSE/CMTR/76941 | `CMTR76941.pdf` | 161272 | `ce94ee1c54d98eccdf3c30fa0673ae2b6a49ea4ed4a72fb83d826f4359f71d58` | 2 | Circular PDF. Metadata creation time 2026-10-19 15:14 IST. |
| CE-S-17, NSE/CMTR/71775 | `CMTR71775.pdf` | 176256 | `aa97e0afc0ce394097f2fc62631c68e3c2e4c7c23541ed35f21d9fc06b0dcacb` | 2 | Circular PDF. Metadata creation time 2025-12-12 11:56 IST. |
| CE-S-18, NSE/CMTR/72260 | `CMTR72260.pdf` | 78974 | `c5d32a838b46d5044717b830a4793479a51a40f8e4e19d3896c65682d15b6ebc` | 1 | Circular PDF. Metadata creation time 2026-01-12 15:29 IST. |
| CE-S-03, CAS page | `NSE-CAS.pdf` | 1754349 | `11bae310d99b7659a823aa6fae0f5d8cfc838f2f2ca337091e44e4077ba39eb5` | 8 | Browser print made 2026-11-02 02:15 IST. The page shows it was updated on 12/08/2026. |
| CE-S-02, market-timings page | `NSE-Market-Timings.pdf` | 886030 | `a57b20044afb042eca30018212ead7bd3090fe4f0fcc0b6b145ed14c1fe957a2` | 5 | Browser print made 2026-11-02 02:17 IST. The page shows it was updated on 04/08/2025. |

**Notes on the files:**

- **Location.** The owner gave the folder as `NorthstarEvidence\NSE-Cash-Equity-Sessions\Official-Sources`. That folder did not exist. The six files were found together in another owner folder outside every repository.
- **Date format.** NSE's "updated on" dates are read as day/month/year. A month/day reading of 12/08/2026 would be a date after the capture, which is impossible.
- **Capture gaps.** Both prints lose content where the page breaks: the site's fixed header covers the top of each printed page. Missing content is recorded as missing and never reconstructed.
  - CE-S-03 is missing the first CAS timetable rows, from 15:15 up to the limit-only order-entry window, and the top of its circulars list.
  - CE-S-02 is missing the rows before the first block-deal close and the rows after the closing-session open.
- **URLs.** Both prints carry the page's official path with a `utm_source` query parameter. The canonical URLs are those in sections 3.1 and 13.2.
- **Not supplied:** the holidays page (CE-S-01). It is not needed for any fact here.

### 14.2 Source identities

| Source | Department | Download reference | Circular reference | Date | Subject (paraphrased) |
|---|---|---|---|---|---|
| CE-S-15 | Capital Market | NSE/CMTR/75479 | 108/2026 | 2026-07-30 | Mock trading on Saturday 2026-08-01 and go-live of the CAS trading modalities in the equity cash segment; new NEAT+ version |
| CE-S-19 | Capital Market | NSE/CMTR/76941 | 149/2026 | 2026-10-19 | Diwali Muhurat trading session |
| CE-S-17 | Capital Market | NSE/CMTR/71775 | 172/2025 | 2025-12-12 | Trading holidays for calendar year 2026 |
| CE-S-18 | Capital Market | NSE/CMTR/72260 | 07/2026 | 2026-01-12 | Trading holiday on 2026-01-15 in the Capital Market segment |
| CE-S-03 | Equities, Closing Auction Session page | — | — | Updated 2026-08-12 | CAS applicability, timings and mechanics |
| CE-S-02 | Market Timings page, Capital Market tab | — | — | Updated 2025-08-04 | Capital Market session timetable |

**New identifiers cited by the inspected sources** (SOURCE LOCATED; not retrieved):

| ID | Reference | Cited by | Relevance |
|---|---|---|---|
| CE-S-05 | NSE continuous-markets FAQs, section on CAS in the CM and F&O segments (`https://www.nseindia.com/static/trade/continuous-markets-FAQs`) | CE-S-15 p. 1; CE-S-03 p. 5 | CAS mechanics and possibly eligibility |
| CE-S-20 | NSE/CMTR/73927 (2026-04-28) | CE-S-03 p. 5 | Self-trade prevention in call auctions. Not a session fact. |
| CE-S-21 | Circular with download number 71999 (2025-12-24) | CE-S-15 p. 1 | Mock trading schedule. Relevant to CE-F-32. |

CE-S-11 (73362), CE-S-12 (73845) and CE-S-13 (74466) are confirmed as CAS circulars by name in CE-S-15 (p. 1) and CE-S-03 (p. 6). Their contents are **not** inferred from those listings (rule 4). CE-S-10 (72394) and CE-S-14 (74969) do not appear in the visible parts of either source.

### 14.3 Verified facts

For every fact below:

- the state is **VERIFIED FROM SOURCE**;
- owner acceptance is **PENDING** (section 14.9);
- nothing is ACCEPTED.

CE-F-15, CE-F-16, CE-F-17 and CE-F-19 are new IDs. Each one records something a source established that no earlier candidate stated exactly.

| Fact | Verified content (original summary) | Applicability | Effective | Evidence |
|---|---|---|---|---|
| CE-F-10 | The changes in equity-segment trading modalities that come from introducing CAS take effect in live trading from Monday 2026-08-03. | CM equity segment; CAS-eligible securities per CE-F-15 | From 2026-08-03 | CE-S-15 p. 1; Monday confirmed at Annexure 1, p. 5 |
| CE-F-13 | Continuous trading for securities not in CAS runs from 09:15 to 15:30 IST. | Non-CAS securities | Current rule as of the page update on 2026-08-12. The page states no effective-from date. | CE-S-03 p. 3. CE-S-19 p. 2 confirms the 15:30 non-CAS close for 2026-11-08. |
| CE-F-15 | CAS phase 1 applies only to cash-segment stocks on which derivative contracts are available. | Instrument level | Current as of 2026-08-12 | CE-S-03 p. 2 |
| CE-F-16 | CAS is a separate 20-minute session from 15:15 to 15:35 IST on all trading days. The visible sub-periods: a limit-order-only entry window from 15:25 to 15:30, closing at a random time in its last 2 minutes; order matching and trade confirmation from 15:30 to 15:35; a transition from 15:35 to 15:50; a post-close session from 15:50 to 16:00. The CAS reference price is the VWAP of trades from 15:00 to 15:15. | CAS-eligible securities (CE-F-15) | Current as of 2026-08-12. Whether this timetable applied unchanged from 2026-08-03 is **not** stated. | CE-S-03 pp. 2-3. The rows from 15:15 to 15:25 are lost in the capture (section 14.1). |
| CE-F-17 | The inspected Capital Market sources treat the pre-open and CAS as sessions distinct from normal-market continuous trading. CE-S-19 names the normal market the continuous trading session (CTS), and lists pre-open, CAS, transition and post-close periods as separate rows. CE-S-03 calls CAS a separate session. | CM | 2026-11-08 (CE-S-19); current as of 2026-08-12 (CE-S-03) | CE-S-19 pp. 1-2; CE-S-03 p. 2 |
| CE-F-19 | Every timetable in NSE/CMTR/75479 is for the **mock** session on Saturday 2026-08-01. That day had three mock trading sessions: from the primary site, then the DR site, then the primary site. The circular gives no live-day session clock. Mock trades carry no fund pay-in or payout, so 2026-08-01 is not a live session. | CM mock environment | 2026-08-01 only | CE-S-15 pp. 1-3 (timetables), p. 5 (Annexure 1) |
| CE-F-20 | 2026 CM trading holidays (weekdays): 01-26, 03-03, 03-26, 03-31, 04-03, 04-14, 05-01, 05-28, 06-26, 09-14, 10-02, 10-20, 11-10, 11-24 and 12-25. Holidays falling on a weekend are listed separately: 02-15 (Sunday), 03-21 (Saturday), 08-15 (Saturday) and 11-08 (Sunday, Diwali Laxmi Pujan). A footnote says Muhurat trading will be held on 2026-11-08, with timings to follow. | CM segment, calendar year 2026 | Notified 2025-12-12; modified by CE-F-21 | CE-S-17 p. 1 (weekday list), p. 2 (weekend list and footnote) |
| CE-F-21 | Thursday 2026-01-15 is an added CM trading holiday (Maharashtra municipal corporation elections). It partially modifies NSE/CMTR/71775. | CM segment only | 2026-01-15; notified 2026-01-12 | CE-S-18 p. 1 |
| CE-F-30 | A special **live** CM trading session is held on Sunday 2026-11-08 for Diwali Muhurat trading. All of its trades carry settlement obligations. | CM segment | 2026-11-08 | CE-S-19 pp. 1-2; announced in CE-S-17 p. 2 |
| CE-F-31 | Muhurat 2026-11-08 timings, IST (detailed in section 14.6): the continuous trading session opens at 14:20. It closes at 15:15 for CAS securities and at 15:30 for non-CAS securities. CAS runs from 15:15 to 15:35. | CM segment; split by CAS eligibility; separate hours for special pre-open stocks | 2026-11-08 | CE-S-19 pp. 1-2 |

All 22 weekdays named in CE-S-15, CE-S-17 and CE-S-18 were checked against the calendar and match.

### 14.4 Facts inspected but not promoted

| Fact | Finding | State |
|---|---|---|
| CE-F-01 | CE-S-02 shows a normal-market open of 09:15. That page was last updated 2025-08-04, predates CAS and gives no effective dates. CE-S-03 confirms 09:15 for non-CAS securities only (CE-F-13). No inspected source states the normal-day open for CAS securities. | DISCOVERED |
| CE-F-02 | CE-S-02 shows a single normal-market close of 15:30 for the CM segment. It gives no effective dates and is stale after 2026-08-03 (CE-C-07). A dated CM circular is still needed for the regime in force before CAS. | DISCOVERED |
| CE-F-03 | CE-S-02 lists a regular pre-open from 09:00 to 09:08, before the 09:15 open, on a stale page. The separateness of pre-open is verified in CE-F-17. Its exclusion from the grid is policy (CE-F-14). | DISCOVERED |
| CE-F-04 | No inspected source addresses a pre-open change from 2026-09-07. | DISCOVERED |
| CE-F-11 | The criterion is verified as CE-F-15. The figure of about 220 stocks, the condition about both NSE and BSE, the publication method and how often the list is revised appear in **no** inspected source. They stay SECONDARY. | DISCOVERED (remainder) |
| CE-F-12 | **Strong support, but no explicit statement.** CE-S-03 places CAS from 15:15 and computes its reference price from trades up to 15:15, but the visible text never states when continuous trading ends for CAS securities on a normal day. CE-S-19 states 15:15 for 2026-11-08 only. The 15:15 normal-day close is **not** inferred. | DISCOVERED |
| CE-F-14 | Northstar policy, not an exchange fact. The exchange basis for it is now verified in CE-F-17. | Owner acceptance only |
| CE-F-22 | NSE/CMTR/76941 continues 71775 only to set Muhurat timings. No review of 2026 CM circulars for other holiday amendments was made. | DISCOVERED |
| CE-F-32 | CE-S-15 cites a mock trading schedule (CE-S-21). Mock sessions are not live (CE-F-19). No other 2026 CM special session was inspected. | DISCOVERED |
| CE-F-40 to 42 | Not addressed. CE-S-19 and CE-S-02 list a call auction session for illiquid securities, which is consistent with CE-F-42 but is not an instrument-level source. | DISCOVERED |

### 14.5 Effective dates and applicability

**Chain for holidays and special sessions:**

| Date | Source | Effect |
|---|---|---|
| 2025-12-12 | 71775 | Base list of 2026 CM holidays; Muhurat announced without timings |
| 2026-01-12 | 72260 | Adds 2026-01-15, in partial modification of 71775 |
| 2026-10-19 | 76941 | Continues 71775 and sets the timings for 2026-11-08 |

**Chain for CAS:**

| Date | Source | Effect |
|---|---|---|
| 2026-03-18 | 73362 | Cited CAS circular (operating guidelines). Not inspected. |
| 2026-04-22 | 73845 | Cited CAS circular (security master files). Not inspected. |
| 2026-05-29 | 74466 | Cited CAS circular (trading modalities). Not inspected. |
| 2026-07-30 | 75479 | Live go-live date 2026-08-03; mock session on 2026-08-01 |
| 2026-08-12 | CE-S-03 page update | Current CAS applicability and timings |
| 2026-10-19 | 76941 | Confirms CAS operates on the 2026-11-08 special session too |

**What is not established:**

- the exact clocks in force for CAS securities from 2026-08-03 to 2026-08-12;
- any CM regime before 2026-08-03 with a start date;
- any change from 2026-09-07.

**Applicability by instrument.** Three groups of securities can have different continuous phases on the same date:

- **CAS-eligible stocks** (CE-F-15);
- **non-CAS securities** (CE-F-13);
- **stocks in a special pre-open session**, meaning IPOs and relistings. CE-S-19 gives these a normal market from 15:05 to 15:30 on 2026-11-08. This is an instrument-level exception like CAS, and the venue-level session port cannot express it either.

### 14.6 CAS live versus mock, and the Muhurat session

**Mock versus live (checklist item G).**

- Every clock in CE-S-15 is a mock clock for 2026-08-01 (CE-F-19). That includes the 15:30 closes behind CE-C-03.
- No CE-S-15 clock may enter live reference data.
- The live normal-day CAS clocks come from CE-S-03 (CE-F-16).

**Muhurat, Sunday 2026-11-08 (CE-S-19), times in IST:**

| Phase | Time | In the 5-minute grid? |
|---|---|---|
| Block deal session | 13:50 to 14:05 | No |
| Pre-open order entry. All orders until 14:10; limit orders only until 14:15, closing at a random time in the last 2 minutes. | 14:05 to 14:15 | No |
| Pre-open matching | 14:15 to 14:17 | No |
| Pre-open buffer | 14:17 to 14:20 | No |
| Continuous trading session (CTS), CAS securities | 14:20 to 15:15 | Yes: 11 five-minute intervals |
| Continuous trading session (CTS), non-CAS securities | 14:20 to 15:30 | Yes: 14 five-minute intervals |
| Reference-price calculation and transition to CAS | 15:15 to 15:20 | No |
| CAS order entry. All orders until 15:25; limit orders only until 15:30, closing at a random time in the last 2 minutes. | 15:20 to 15:30 | No |
| CAS matching | 15:30 to 15:35 | No |
| CAS transition | 15:35 to 15:50 | No |
| Post-close session | 15:50 to 16:00 | No |
| Special pre-open session (IPO and relisted stocks), closing at a random time in the last 10 minutes | 14:05 to 14:50 | No |
| Normal market for special pre-open stocks | 15:05 to 15:30 | Instrument-specific |
| Call auction for illiquid securities, closing at a random time in the last minute | 14:25 to 15:10 | No |
| Trade modification window | 14:20 to 16:15 | No |

- The interval counts are arithmetic on the stated clocks. Every continuous boundary is 5-minute aligned, so EI-1A's grid can represent each one.
- V1 still does not trade special sessions (EI0-D7).
- CE-D10's precondition (CE-S-19 verified) is now met. The decision itself stays PENDING.

### 14.7 Conflicts and unresolved evidence

CE-C-01 to CE-C-06 are covered by the dated note in section 8.

| ID | Conflict | Treatment |
|---|---|---|
| CE-C-07 | **Stale timings page.** CE-S-02, last updated 2025-08-04 and captured 2026-11-02, still shows one CM normal market from 09:15 to 15:30 and a closing session from 15:40, with no CAS. CE-S-15 and CE-S-03 put CAS live from 2026-08-03, with CAS from 15:15 to 15:35 and a post-close session from 15:50. | CE-S-02 is not used for any rule after 2026-08-03. It is not used for effective dates for any period either. |
| CE-C-08 | **Subject of NSE/CMTR/74466.** CE-S-15 (p. 1) gives 74466 a subject about the derivatives segment. CE-S-03 (p. 6) gives 74466 as the cash-segment trading modalities, and NSE/FAOP/74467 as the derivatives change. | Unresolved until 74466 is read. No verified fact depends on it. |
| CE-C-09 | **Label in CE-S-19.** The circular expands CTS as "Continuous Trading Session" in one row, and as a closing trading session in the row for the transition to CAS. | The clocks are unaffected. Recorded only. |

**Still unresolved:**

- CE-F-12, the normal-day continuous close for CAS securities;
- historical CAS membership (section 13.5). No inspected source publishes a membership list, an as-of rule or a revision process;
- completeness of 2026 holiday amendments (CE-F-22).

### 14.8 Totals after this task

There are 22 facts: the original 18, plus CE-F-15, 16, 17 and 19.

| State | Facts |
|---|---|
| DISCOVERED | 12: CE-F-01 to 04, 11, 12, 14 (policy), 22, 32, 40 to 42 |
| VERIFIED FROM SOURCE, owner acceptance pending | 10: CE-F-10, 13, 15, 16, 17, 19, 20, 21, 30, 31 |
| PENDING ACCEPTANCE (owner review opened) | 0 |
| ACCEPTED | 0 |

CE-D1 to CE-D10 remain **PENDING**. The verified facts are proposed for acceptance in section 14.9. They move to PENDING ACCEPTANCE when the owner opens that review. CE-D1 and CE-D2 decide whether this review gate applies at all.

### 14.9 Proposed for owner acceptance

The owner may accept, reject or defer each fact. Acceptance would be for the cash-equity session reference only.

1. **CE-F-20 and CE-F-21:** the 16 weekday CM holidays and 4 weekend holidays of 2026, as listed in section 14.3.
   - Proposed with the condition that CE-F-22 (later amendments) is open. Any amending circular creates a new reference version.
2. **CE-F-30 and CE-F-31:** a live CM special session on 2026-11-08, with the phases in section 14.6.
   - It is recorded so the date is not mistaken for a holiday. V1 excludes it.
3. **CE-F-10:** the CAS trading-modality changes are live from 2026-08-03.
4. **CE-F-13:** non-CAS continuous trading from 09:15 to 15:30 IST.
   - Proposed as current as of 2026-08-12. Its effective-from date stays open.
5. **CE-F-15:** CAS phase 1 applies to cash-segment stocks with available derivative contracts.
   - Proposed as a criterion only. It is not a list, and it is not historical membership.
6. **CE-F-16:** the CAS session from 15:15 to 15:35, with the visible sub-periods.
   - Proposed as current as of 2026-08-12. The first rows are not captured.
7. **CE-F-17:** pre-open and CAS are separate from continuous trading.
8. **CE-F-19:** the clocks in 75479 are mock-only, and 2026-08-01 is not a live session.

**Separately, a policy for owner decision rather than a verified fact:** CE-F-14, extended to pre-open (CE-F-03). Only continuous trading phases enter the 5-minute grid. Pre-open, CAS, the transition periods, post-close, block deals and call auctions are excluded.

### 14.10 Documents still missing

1. **NSE/CMTR/74466** (2026-05-29), on the CAS trading modalities. This is the most likely dated source for CE-F-12 and for the clocks in force from 2026-08-03. It would also resolve CE-C-08.
2. **NSE/CMTR/73845** (2026-04-22), on security master changes for CAS. It is the first place to check whether eligibility is published per security (checklist item H).
3. **NSE/CMTR/73362** (2026-03-18), the CAS operating guidelines.
4. **A complete capture of CE-S-03** that shows the CAS rows from 15:15 to 15:25 and the full circulars list, for example a full-page screenshot.
5. **The CM circular on pre-open changes from 2026-09-07,** if one exists (CE-F-04). Candidate: CE-S-14, 74969.
6. **A dated CM circular establishing the regime before CAS, with its start date.** Needed only if CE-D7 covers dates before 2026-08-03.
7. **A review of the 2026 CM circular list for holiday amendments** (CE-F-22).
8. **Optional:**
   - CE-S-05 (CAS FAQs);
   - CE-S-04 (pre-open page);
   - CE-S-10 (72394), whose relevance is unconfirmed;
   - CE-S-31 (the F&O Muhurat counterpart), for the separate Futures follow-up.

### 14.11 EI-1C-2 readiness

- **The gate adapter (section 12)** can be built and tested with labelled test evidence once CE-D1 to CE-D3 are decided. This is unchanged.
- **Real data that could load after acceptance:**
  - the 2026 closed dates (CE-F-20 and 21);
  - the 2026-11-08 special session (CE-F-30 and 31).
- **Normal trading days:** **not ready.**
  - There is no verified continuous phase for all EQ securities on a normal day, because the open and close for CAS securities are not explicit (CE-F-01, CE-F-12).
  - Special pre-open stocks have their own hours as well.
  - The regime before CAS is undated (CE-F-02).
  - A resolver loaded with only the accepted facts above could report holidays and the Muhurat date. Every ordinary weekday would still raise. That is correct fail-closed behaviour, but it is not useful for V1.
- **CE-D5 and CE-D6** remain prerequisites for any session after 2026-08-03.

---

*Secondary material used only to locate identifiers in this task* (not evidence):

- motilaloswal.com
- yesinvest.in
- outlookmoney.com
- angelone.in
- bajajbroking.in
- multibagg.ai
- kotakneo.com
- arihantplus.com
- teamleaseregtech.com
- share.market
- indiabonds.com
- sahi.com
- upstox.com
- rupeezy.in
- businesstoday.in

These were reached through search-engine result listings. No NSE host was contacted.
