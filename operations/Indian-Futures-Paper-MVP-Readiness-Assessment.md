# Indian Futures Paper MVP: Readiness Assessment

Status: **read-only readiness audit (M1.5.1, 2026-10-11)**. This document
changes no acceptance status, approves nothing and authorizes no release,
candle approval or production action. The authoritative statuses remain those
of the [Paper MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md)
(the "specification") and the
[Paper MVP Evidence Register](Indian-Futures-Paper-MVP-Evidence-Register.md).

**Verdict: NOT READY FOR SIGN-OFF** (section 14).

**Sources:**
- [Deployment Runbook](Indian-Futures-Deployment-Runbook.md);
- [Expiry Exception Operator Procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md);
- [Independent Paper P&L Reconciliation](Indian-Futures-Independent-Paper-PnL-Reconciliation.md);
- [Windows SQLite Backup and Isolated Restore Drill](Indian-Futures-Windows-SQLite-Backup-and-Restore-Drill.md).

Source paths are relative to the Northstar workspace root.

**Operator-reported facts not recorded in the evidence register.** The
M1.5.1 brief states two facts with no log or file inspected:
- drill blocks A and P passed on the deployment machine;
- production still runs the older build.

They are labelled operator-reported wherever used.

## 1. Executive summary

**What is done:**
- 28 of the specification's 41 requirements (sections 2 to 12) are PASS on
  inspected code and automated tests.
- Expiry exception detection (M1.4.3.2) and the independent P&L verifier
  (M1.4.4.2) are merged into `develop`.
- The synthetic operator exercise (M1.4.3.3) is PARTIALLY VERIFIED.

**What is not done:**
- 10 requirements are open: X-4 and D-4 are BLOCKED; P-3, N-4, X-3, V-3, D-2,
  D-3, B-1 and B-2 are PENDING.
- 3 more, O-5, P-4 and V-5, are DEFERRED; their limitations are not yet
  approved.
- No policy decision has been approved: D-EXP-1 to D-EXP-7, U-1 to U-8,
  TG-1 to TG-5 and L-1 to L-8.
- Production runs a build that does not include M1.2 or the expiry
  detection.
- G-1 (Upstox data permission) is unresolved and blocks D-4, the recovery
  drill (B-1, B-2) and production P-3.

**Time-sensitive:** the October contract expires on 2026-10-27, with a
SHORT 1 position open and finality approved only through 2026-10-08. The
normal flatten needs sessions through 2026-10-19 approved and processed. Any
such approval also approves the revised 2026-10-09 session, because finality
is a single final-through date. That approval is the operator's decision
under N-4, and this document does not recommend it (section 12).

## 2. Verified milestones and repository baseline

**Repositories.** Recorded 2026-10-11 from local refs; nothing was fetched.
All six are on `develop` with clean working trees.

| Repository | `develop` | Local `main` | `develop` commits not in `main` | `main` commits not in `develop` |
|------------|-----------|--------------|---------------------------------|---------------------------------|
| northstar-core | e54ccdf | 26c1d04 (local `origin/main` is 2ca2081, 5 commits ahead of local `main`) | 12 | 1 merge |
| northstar-application | 9a0ab57 | f101f3f | 12 | 2 merges |
| northstar-infrastructure | aa94428 | 9003b3c | 12 | 0 |
| northstar-api | cfa7774 | 76cc3e9 | 18 | 4 (release branches) |
| northstar-web | 4f6908c | 006243a | 2 | 2 merges |
| northstar-docs | 3101a02 | 5eaf482 | 26 | 6 (release branches) |

**Main-only commits** are release-branch merges whose content reached
`develop` through separate feature merges. For example, the scheduler
registration script and task XML are identical on both branches, and the
wrapper differs only by the exit-7 change. `develop` therefore contains
`main`.

**`develop` also carries unrelated India options work** (core, application,
infrastructure, api). A release cut from `develop` ships it too (section 5).

**Milestones:**

| Milestone | State | Evidence | Kind |
|-----------|-------|----------|------|
| M1.2 dashboard snapshot consistency | Merged (application PR #66, api PR #38); **not deployed** | E-8 | Development |
| M1.3 acceptance specification and expiry policy | Documented; policy proposed | Specification 13.7 | Documentation |
| M1.4.1 Windows backup and restore runbook | Written and statically validated | Drill document | Documentation |
| M1.4.2 drill: blocks A and P | **Operator-reported as passed** on the deployment machine; no register entry or artifact | M1.5.1 brief | Deployment (operator-reported) |
| M1.4.2 drill: blocks B to J | Not run; waiting on G-1 | -- | -- |
| M1.4.3.1 and M1.4.3.2 expiry detection (S-1, S-2, exit 7) | Merged (api #39 `545b3b6`, web #8 `4f6908c`, docs #20); **not deployed** | E-10 | Development |
| M1.4.3.3 operator exercise | **PARTIALLY VERIFIED**; synthetic; real Task Scheduler not tested | E-11 | Synthetic |
| M1.4.4.1 and M1.4.4.2 independent P&L verifier | Merged (api #40 `cfa7774`, docs #22 `3101a02`); synthetic PR-1 to PR-16, no mismatch | E-12 | Synthetic |

## 3. Acceptance requirement matrix

The specification is authoritative for each requirement's wording; the
conditions below are summaries.

**Evidence kinds:**
- **Code+tests:** inspected code and automated tests; development,
  reproducible.
- **Operator-reported:** production, not reproducible from the repositories.
- **Synthetic:** a reproducible engineering result.

### 3.1 PASS (28)

| ID | Condition (summary) | Evidence | Kind | Remains unverified |
|----|---------------------|----------|------|--------------------|
| S-1 | One exact dated contract from configuration | Spec section 2 row | Code+tests; E-1 | -- |
| S-2 | Contract's own economics, 65 INR per point | Spec section 2 | Code+tests; E-3 | -- |
| S-3 | One built-in strategy; one portfolio per contract | Spec section 2 | Code+tests | -- |
| R-1 | 20-bar BUY, SELL or HOLD rule | Spec section 3 | Code+tests | -- |
| R-2 | A decision is frozen once | Spec section 3 | Code+tests | -- |
| R-3 | No look-ahead; no clock decides | Spec section 3 | Code+tests | -- |
| R-4 | Missing or invalid data decides nothing | Spec section 3 | Code+tests | -- |
| O-1 | Target +N or -N; the order is the difference | Spec section 4 | Code+tests | -- |
| O-2 | HOLD or a met target creates no order | Spec section 4 | Code+tests | -- |
| O-3 | A reversal is one order through zero | Spec section 4 | Code+tests | -- |
| O-4 | Deterministic order identity | Spec section 4 | Code+tests | -- |
| F-1 | Fill at the next stored bar's OPEN | Spec section 5 | Code+tests; E-2 | -- |
| F-2 | Pending until that bar is stored | Spec section 5 | Code+tests | -- |
| P-1 | Direction, whole contracts, weighted average | Spec section 6 | Code+tests; E-12 synthetic | -- |
| P-2 | Gross realized and unrealized P&L at the contract's point value | Spec section 6 | Code+tests; E-12 synthetic | Production (P-3) |
| N-1 | `disabled` or `operator-approved` finality only | Spec section 7 | Code+tests | -- |
| N-2 | WAITING, exit 0, no clock, no request | Spec section 7 | Code+tests | From M1.4.3.2, an exposed contract past expiry reports exit 7 (not yet in production) |
| N-3 | Nothing writes `.env` or final-through | Spec section 7 | Code+tests | -- |
| C-1 | One range acquired; decided oldest first | Spec section 8 | Code+tests | -- |
| C-2 | Idempotent reruns | Spec section 8 | Code+tests | -- |
| C-3 | Facts never updated or deleted | Spec section 8 | Code+tests | -- |
| C-4 | One writer at a time | Spec section 8 | Code+tests | -- |
| X-1 | K = 5 flatten rule | Spec section 9 | Code+tests | Production (X-3) |
| X-2 | ROLLOVER REQUIRED after E | Spec section 9 | Code+tests | With M1.4.3.2, only when flat with nothing pending (S-1) |
| V-1 | Read-only dashboard content | Spec section 10 | Code+tests | The `northstar-web:india` image and Caddy are not recorded on the deployment machine (runbook section 2) |
| V-2 | Dashboard consistent during writes | Spec section 10; E-8 | Code+tests ("merged to development") | Production (V-3) |
| V-4 | Run log and `last-run.json` | Spec section 10 | Code+tests | -- |
| D-1 | Indian stack isolated from CME | Spec section 11 | Code+tests | -- |

These 28 are exactly the PASS identifiers of the specification's section 16
summary.

### 3.2 PENDING (8), BLOCKED (2) and DEFERRED (3)

| ID | Condition (summary) | Status | Evidence and kind | Unverified | Blocking dependencies | Decision owner |
|----|---------------------|--------|-------------------|------------|-----------------------|----------------|
| P-3 | October valuation equals an independent recomputation | PENDING | Synthetic only (E-12) | The October portfolio | G-1 (reading or retaining production values) | Gate owner; approver |
| P-4 | Fees, taxes, slippage, cash, margin | DEFERRED (L-6) | Excluded by design | -- | Approval of L-6 | Approver |
| O-5 | FLAT only by the expiry flatten | DEFERRED (L-5) | X-1 covers the flatten | -- | Approval of L-5 | Approver |
| N-4 | Written procedure for a revised candle | PENDING | E-7 (operator-reported) | No procedure exists | None technical | Operator to write; approver to accept |
| X-3 | October flatten observed in production | PENDING | Not reached | Everything | Approver decides whether it is required (sign-off item 9). If required: approvals through 2026-10-19 before expiry, which N-4 governs | Approver; operator |
| X-4 | Expiry exception approved, enforced, verified | BLOCKED | E-10 and E-11 (development, synthetic) | Approval, deployment, real scheduler, reconciliation | D-EXP-1 to D-EXP-7; release; deployment-machine verification | Approver |
| V-3 | M1.2 deployed | PENDING | E-8 (not deployed) | Deployment | Controlled release | Operator; approver |
| V-5 | Automatic notifications | DEFERRED (L-7) | -- | -- | Approval of L-7 | Approver |
| D-2 | Hourly task runs exactly `operations daily` | PENDING | E-4 (operator-reported) | E-4 artifacts | Attach a log and `last-run.json`, or approver acceptance | Operator; approver |
| D-3 | Isolated real-Docker acceptance | PENDING | E-5 and E-6 (operator-reported) | Test output and hashes | Attach them, or approver acceptance | Operator; approver |
| D-4 | G-1 resolution recorded | BLOCKED | -- | -- | G-1 | Gate owner |
| B-1 | Verified backup on the Windows deployment machine | PENDING | Blocks A and P operator-reported (not in the register); E-9 is a different method | Blocks B, D, E | G-1 | Operator (supervised) |
| B-2 | One isolated restore and comparison | PENDING | -- | Blocks F to J | B-1; G-1 | Operator (supervised) |

## 4. Evidence coverage and gaps

**Coverage:**
- The code-level behaviour behind every PASS is reproducible on the
  development machine.
- Synthetic evidence is preserved outside the repositories with SHA-256
  manifests: M1.4.3.3 (E-11) and M1.4.4.2 (E-12).

**Gaps:**
1. **E-1 to E-7 and E-9 are operator-reported, with no artifacts.** Sign-off
   item 3 needs artifacts or explicit approver acceptance. Production
   artifacts (logs, `paper status` output) contain Upstox-derived values, so
   retaining them falls under G-1.
2. **Blocks A and P** are reported passed in the M1.5.1 brief but recorded
   nowhere. Their `drill.json` and `results\lock-test-passed.json` (drill
   document, sections 4A and 4P) should be recorded as operator-reported
   evidence.
3. **Stale wording in E-10 and E-12.** They describe their code as
   "uncommitted"; it has since been merged (api `545b3b6` and `cfa7774`). The
   register is append-only, so a dated note is the correction.
4. **The deployed build is unidentified** (E-4 note). The only clue is that the
   E-4 run implies the wrapper's default-directory fix, which is on both
   branches: `78e8770` on `develop`, an equivalent release commit `5aa23ae` on
   `main`.
5. **No full-suite counts at deployed commits** (sign-off item 10). E-8's
   counts are development counts at earlier commits.

## 5. Production release readiness

**What `develop` adds over the documented production state:**

| Change | Repository | Effect | Notes |
|--------|------------|--------|-------|
| M1.2 dashboard consistency | application `d2db174` (PR #66), api PR #38 | The dashboard never mixes two persisted states | V-3 |
| S-1 and S-2 expiry detection, exit 7 | api `545b3b6` | `STATUS: EXPIRY EXCEPTION`; reported before any provider request | X-4 enforcement (detection only) |
| Wrapper exit 7 | api `deploy/india/windows/Invoke-NorthstarIndiaOperations.ps1` | `last-run.json` outcome `EXPIRY_EXCEPTION` | The registered task XML is unchanged |
| Dashboard expiry warnings | api (`operational_status.py`, schema) and web `5862618` | Stage `expiry_exception`; `pending_orders`; "as of the latest decision" | Needs the web image, which is not recorded on the deployment machine |
| Independent P&L verifier | api `cfa7774` (tests only) | No runtime effect | -- |
| India options features | core, application, infrastructure, api | Option CLI commands and option tables (created only if an option command runs) | **Out of Phase 1 scope**; the approver should accept shipping it or cut a narrower release |

**Compatibility:**
- Futures tables and stores are byte-identical between `main` and `develop`.
- `initialize_database` (`northstar-api/src/northstar_api/runtime.py`) creates
  only futures tables, and `CREATE TABLE IF NOT EXISTS` is a no-op for
  existing ones.
- No migration is needed. No `.env` setting is added or renamed.
- The scheduled task definition is unchanged.

**Release gaps:**

| Topic | State |
|-------|-------|
| Deployed image identity | **Not established.** Nothing records the deployed commit or image ID (E-4). The drill preflight records `imageId` (drill document, block A). A release must record the commits and the image ID before and after |
| Rollback | Runbook section 15 documents an image retag (`northstar-api:india-previous`), but its stop and restart lines are Linux/systemd. Section 18H only removes the scheduled task. A Windows-specific upgrade and rollback procedure was **not written** at this assessment. M1.5.2 has since prepared one for operator review ([Windows Controlled Upgrade and Rollback](Indian-Futures-Windows-Controlled-Upgrade-and-Rollback.md)); it is not approved and has never been run |
| Mandatory preflight | The wrapper's own checks (Docker engine, Compose config, image present: runbook section 18) and the drill's read-only preflight (drill document, block A). Plus: no run active, task disabled during the swap, `.env` unchanged, the `northstar-india-data` volume untouched |
| Checks needing the deployment machine | First supervised run after the upgrade (exit code and `last-run.json`); dashboard reads (V-3) if the web stack is run; exit-7 recognition by the **real** Task Scheduler (not verified in M1.4.3.3); full suites at deployed commits (sign-off item 10) |

**Can a release proceed before G-1?**
- Technically yes. It changes no data handling, schema or configuration, and
  stores nothing new.
- But the runbook (section 0) says G-1 must be cleared before persistent
  operation, which is already running. Whether to keep operating, and
  therefore whether to release, is the gate owner's and approver's decision.
- Releasing changes the image ID, so the prepared drill (blocks A and P)
  would STOP at block B ("imageId changed since block A"). Blocks A and P
  must then be repeated before B (drill document, section 4B).
- Not releasing leaves production without S-1 and S-2 through the October
  expiry: after 2026-10-27 a stalled run would report WAITING with exit 0.

## 6. Expiry exception readiness

| Item | State |
|------|-------|
| EXP-1 and EXP-2 policies (specification 13.7) | Proposed; not approved (D-EXP-1) |
| S-1 and S-2 (exit 7) | Implemented, tested and merged (E-10); not approved (D-EXP-2, D-EXP-3); **not deployed** |
| Wrapper outcome `EXPIRY_EXCEPTION` | Verified with a fake `docker` (E-11); real Task Scheduler **not verified** |
| Dashboard | Stage `expiry_exception` (S-1 only); no real-date check (S-2 not applied to the dashboard) |
| Operator containment (procedure sections 6 to 9) | Exercised synthetically (E-11). Real scheduler suspension not verified; escalation is a human step |
| D-EXP-1 to D-EXP-7 | All open. D-EXP-6 (reconciliation or settlement method) has no candidate at all |
| TG-1 to TG-5 (procedure section 10) | Proposed. TG-1 counts only with a recorded S-1-protected deployed build; TG-2 is required in every case |
| Previous-contract visibility (D-EXP-7) | Confirmed gap (E-11): reconfiguring hides an exception left on the previous contract |
| Reconciliation and settlement | None exists (L-8, U-5); nothing may be edited in the database |
| Last actual fill | No read-only interface lists fill identity, time and price (procedure section 7) |
| X-4 | **BLOCKED** |

**Minimum remaining for X-4** (specification 13.9):
1. Approve the EXP-1 and EXP-2 policies and decide D-EXP-1 to D-EXP-7,
   including approving S-1 and S-2 and choosing D-EXP-6 and the TG gate
   (D-EXP-7).
2. Deploy the detection (section 5) with a recorded build.
3. Verify on the deployment machine, without real data loss or edits:
   - that a run's exit 7 is recorded by the real Task Scheduler and
     `last-run.json`;
   - the containment the operator chose under D-EXP-5.

   A safe way to do this is an isolated acceptance project, as with E-5.
4. If D-EXP-6 needs any reconciliation mechanism, it is a separate approved
   milestone.

## 7. Independent P&L readiness

| Item | State |
|------|-------|
| Accounting rules | Audited (reconciliation document, sections 2 to 5) |
| Independent verifier | Implemented and merged; standard library only; enforced import boundary (E-12) |
| PR-1 to PR-16 | Synthetic: no mismatch on the CLI, dashboard or analysis |
| PR-13 rounding | 6.5e-22 INR, inside the provisional 1e-15 INR threshold (U-3, **not approved**) |
| Gross and net | Gross only; net not applicable (L-6, P-4 DEFERRED) |
| Mark approval and age | Approval is UNKNOWN (finality is not persisted per bar, U-6); no age limit (U-7) |
| U-1 to U-8 | All open |
| Production P-3 (October portfolio) | **PENDING**; needs G-1. The verifier must not be run on the production database or a copy before G-1 |

Synthetic results show the accounting rules are implemented correctly. They
do not show that the production October valuation is correct, and they say
nothing about profitability.

## 8. SQLite recovery readiness

| Item | State |
|------|-------|
| Windows drill document | Written and statically validated; never fully run |
| Block A (preflight; prepares the drill) | Operator-reported passed; not recorded |
| Block P (lock test on a disposable volume) | Operator-reported passed; not recorded |
| Block B (the only block that mounts production, read-only) and C to J | Not run. Block B performs the production backup, which retains Upstox-derived data: **G-1** |
| Safeguards | Production mounted only once, read-only, in block B. The flock is taken through a read-only descriptor. Disposable `sqlite-*` volumes with markers, labels and creation records. Block B checks the preflight identity (`engineId`, `productionVolume`, `productionVolumeCreatedAt`, `imageId`), the script hashes and `results\lock-test-passed.json` |
| Evidence handling | The drill directory sits under `C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills`; real backups never go into a repository; retention falls under G-1 |

**Resuming after G-1 is resolved.** These steps follow the drill document's
own guards:
1. Confirm that the drill directory from blocks A and P still exists, and
   that its scripts are unchanged.
2. If **anything** in the recorded identity changed (most likely the image ID
   after a release), block B STOPs. Prepare a new drill: block A, then block
   P.
3. Run blocks B, C, D and E, then F, G and H (optionally H2), then I and J,
   under supervision.
4. Record `LOCK TEST OK`, `BACKUP OK`, the SHA-256 match, `VERIFY OK`,
   `RESTORE OK` and `COMPARE OK` in the evidence register (B-1, B-2).

## 9. Data-permission and retention gates

G-1 (specification 11.1; runbook section 0) is unresolved. Only its owner can
record a resolution.

**Depends on G-1:**
- D-4;
- the recovery drill blocks B to J (B-1, B-2);
- P-3;
- attaching production artifacts that contain Upstox-derived values (part of
  sign-off item 3);
- whether persistent operation, and therefore a release, should continue.

**Does not depend on G-1:**
- development work;
- synthetic verification;
- policy decisions;
- writing procedures (N-4, the Windows upgrade and rollback procedure).

## 10. Outstanding policy decisions

None of these is decided here.

| Decision | Where | Owner |
|----------|-------|-------|
| EXP-1 and EXP-2 policies; D-EXP-1 to D-EXP-7 | Specification 13.7; procedure section 9 | Approver |
| TG-1 to TG-5 contract-transition gate | Procedure section 10 | Approver |
| U-1 to U-8 (portfolio total, precision, tolerance, net, settlement, mark finality and age, economics dating) | Reconciliation document, section 12 | Approver |
| L-1 to L-8 and the DEFERRED O-5, P-4, V-5 | Specification section 14 | Approver |
| X-3: is live observation of the October flatten required? | Sign-off item 9 | Approver |
| N-4 procedure (including the 2026-10-09 revision) | Specification section 7 | Operator writes; approver accepts |
| Accept E-1 to E-7 as operator-reported, or require artifacts | Sign-off item 3 | Approver |
| Release scope (whether India options code may ship) and timing before or after G-1 | Section 5 | Approver; gate owner |
| G-1 | Specification 11.1 | Gate owner |

## 11. Dependency-ordered completion plan

**A. Development, now** (no approval needed to start; outputs are proposals):
1. Write the Windows upgrade and rollback procedure, with recorded commits and
   image IDs (section 5). **Prepared in M1.5.2 for operator review**:
   [Windows Controlled Upgrade and Rollback](Indian-Futures-Windows-Controlled-Upgrade-and-Rollback.md). It is not approved and
   has never been run; its approvals AP-1 to AP-8 belong to step 4.
2. Draft the N-4 revised-candle decision procedure. It may contain no
   automatic rule and no threshold.
3. Add dated notes to E-10 and E-12 (merged commits), and record blocks A and
   P as operator-reported with their artifacts.

**B. Operator and approver policy** (can run in parallel with A):
4. Decide D-EXP-1 to D-EXP-7 and TG-1 to TG-5; U-3 and U-2 at least; L-1 to
   L-8; X-3; the release scope; and acceptance of operator-reported evidence.

**C. Controlled deployment** (after 1 and the release decisions in 4):
5. Disable the task, tag the previous image, build `develop` at the chosen
   commits, record identities, make a supervised run, then re-enable.
   Rollback is by retag. This delivers V-3 and the deployed half of X-4.

**D. G-1** (gate owner; the critical path):
6. Record G-1. Then, in order:
   - drill block A and block P again if the identity changed;
   - blocks B to J (B-1, B-2);
   - P-3: the independent recomputation of the October portfolio by an
     approved production method;
   - production artifacts for sign-off item 3.

**E. Production verification** (after 5 and, where noted, 6):
7. Exit-7 recognition by the real scheduler and the chosen containment,
   preferably in an isolated acceptance project (X-4).
8. X-3, if required.
9. Full suites at the deployed commits (sign-off item 10).

**F. Outside Phase 1:**
- live orders and broker integration (L-1, L-2);
- automatic approval, rollover and settlement (L-3, L-4, L-8);
- net P&L (L-6) and notifications (L-7);
- a read-only fill listing;
- the November contract until the Muhurat calendar is loaded (runbook
  section 13);
- India options.

**Shortest safe path to sign-off:**

| Step | Depends on |
|------|------------|
| 4 (decisions) | -- |
| 1 and 5 (release) | 4 |
| 7 | 5 |
| 6 (G-1, drill, P-3) | G-1 |
| 9 | 5 |
| Sign-off | All of the above |

G-1 is the only step outside engineering control.

## 12. Risks and STOP conditions

**October 2026 timeline.**
- E = 2026-10-27 (calendar expiry).
- E-6 = 2026-10-16: the K = 5 flatten decision.
- E-5 = 2026-10-19: the flatten fill, at the open of the first stored bar
  after the decision. 2026-10-20 is a holiday.

**Today:**
- Approved data finality reaches 2026-10-08 (E-7). The revised 2026-10-09
  session is unapproved.
- The persisted state is SHORT 1 (E-3, operator-reported).
- **No closing fill exists, and no flatten order exists**: no session after
  2026-10-08 has been decided. The position is not closed.
- A pending flatten order would exist only after a decision at the
  2026-10-16 close. It is filled only once the 2026-10-19 session is approved
  and processed.

**Time-sensitive dependencies:**

| Dependency | Why it is time-sensitive |
|------------|--------------------------|
| Normal flatten | Needs sessions through 2026-10-19 approved and processed while the contract can still be acquired (through 2026-10-27). Final-through is one date, so any approval past 2026-10-08 also approves 2026-10-09. That decision belongs to N-4 and the operator, and must **never** be made just to meet the deadline |
| If processing stays stalled past 2026-10-27 | EXP-1 arises: the position survives expiry |
| Before S-1 and S-2 are deployed | Production reports WAITING with exit 0 after expiry. The operator must apply the procedure's "before M1.4.3.2" checks with the real date |
| Deploying detection before 2026-10-28 (IST) | Makes S-2 report exit 7 on the first exposed run after expiry. Not deploying keeps the reliance on manual checks |
| X-3 | If the approver requires live observation, it can only happen within these dates |

**STOP conditions** (in addition to the procedure's section 11 and the drill
document's section 3):
- any step that would approve a candle to meet a date, or approve 2026-10-09
  without the N-4 decision;
- any production database read, copy or backup before G-1, other than what the
  gate owner allows;
- a release without the previous image tagged and the deployed identities
  recorded;
- configuring the next contract while TG-1 to TG-5 are not satisfied (the
  November contract is blocked anyway);
- exit 7, which means following the expiry procedure: no rollover, no
  database edit;
- claiming the position flat without a persisted closing fill shown by
  `paper status`.

## 13. Phase 1 sign-off prerequisites

From specification section 15, with the current state of each:

| # | Prerequisite | State |
|---|--------------|-------|
| 1 | Every requirement PASS, or DEFERRED with its limitation approved | Not met: 10 open requirements; the 3 deferrals' limitations are not approved |
| 2 | X-4 met as in 13.9 | Not met (section 6) |
| 3 | E-1 to E-7 backed by artifacts or accepted as operator-reported | Not met |
| 4 | P-3 recomputed | Not met (G-1) |
| 5 | M1.2 deployed, commits recorded (V-3) | Not met |
| 6 | Backup and restore drill (B-1, B-2) | Not met (G-1); blocks A and P operator-reported only |
| 7 | G-1 status recorded (D-4) | Not met |
| 8 | N-4 procedure written | Not met |
| 9 | Approver's X-3 decision | Not made |
| 10 | Full suites at deployed commits | Not met |

The approver and operator sign-off table in specification section 15 is
empty.

## 14. Final readiness verdict

**NOT READY FOR SIGN-OFF.** None of the ten sign-off prerequisites is met.

| Group | Blockers |
|-------|----------|
| Policy | X-4 policy, all D-EXP, TG and U decisions, the limitations, X-3 |
| Engineering | Approval of the Windows release procedure (prepared in M1.5.2) and the release (V-3, X-4 deployment) |
| Data permission | G-1 (D-4, B-1, B-2, P-3) |

The code-level Paper MVP behaviour is verified on development and synthetic
evidence. Nothing in this assessment changes a status or grants an approval:
X-4 BLOCKED, G-1 unresolved, B-1 and B-2 PENDING, P-3 PENDING, M1.4.3.3
PARTIALLY VERIFIED.
