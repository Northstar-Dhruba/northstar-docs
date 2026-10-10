# Indian Futures Paper MVP: Evidence Register

The evidence cited as `E-n` by the
[Paper-Trading MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md).
Entries are appended; an existing entry is corrected only by adding a dated
note, never by rewriting what was recorded.

## Source labels

| Label | Meaning |
|-------|---------|
| Operator-reported | Stated by the operator. No log, database row, file or hash was inspected when the entry was written. |
| Inspected | Backed by an artifact in the repositories (commit, code or test) that was read when the entry was written. |
| Engineering session | A result produced on the development machine during an engineering task, from the command stated. It is not a CI artifact and does not describe production. |

Never attach secrets: no Upstox token, no `.env` content, no basic-auth hash.
Real Upstox candle data must not be committed (runbook section 0); refer to
private evidence files by date and description only.

## Register

### E-1 October 7 SELL decision

- **Observation:** at the 2026-10-07 session close the production paper
  operation froze a SELL decision for NIFTY / NSE / 2026-10-27, strategy
  `directional-mvp-v1-nifty-paper-ops`, portfolio `nifty-paper-ops-oct26`.
  The portfolio was flat, so the target-exposure rule (target 1) produced
  SELL 1.
- **Source:** operator-reported.
- **Artifacts attached:** none. To attach: the read-only `paper status` or
  dashboard output showing the frozen decision and its order.

### E-2 October 8 simulated fill at 22580

- **Observation:** the October 7 SELL 1 order filled at 22580, the open of
  the 2026-10-08 session, as the next-session OPEN rule requires.
- **Source:** operator-reported.
- **Artifacts attached:** none. To attach: the fill as reported by
  `paper status` or the dashboard (fill quote and instant). Whether 22580 equals
  the provider's 2026-10-08 open has not been independently checked.

### E-3 SHORT 1 position, frozen October 8 decisions and INR valuation

- **Observation:** after the fill the portfolio held SHORT 1 NIFTY 2026-10-27;
  the October 8 decisions were frozen; an INR gross valuation was shown.
- **Source:** operator-reported. By the portfolio rule (`_transition` in
  `northstar-application/.../build_futures_paper_portfolio.py`) a position
  opened by one fill has that fill's quote as its average entry, here 22580;
  that value is derived, not observed.
- **Artifacts attached:** none. The valuation amounts were not recorded; P-3
  of the specification requires an independent recomputation.

### E-4 First automatic Windows scheduled run

- **Observation:** on 2026-10-10 at 08:00 IST the Windows Task Scheduler task
  "Northstar India Operations" ran production operations unattended and
  returned `WAITING`, exit 0 (sessions after the approved final-through date
  are not processed).
- **Source:** operator-reported.
- **Artifacts attached:** none. To attach: the run's
  `%LOCALAPPDATA%\Northstar\india-operations\logs\<run id>.log` excerpt and
  `last-run.json` (neither contains secrets), and Task Scheduler's last run
  result.
- **Note (inspected):** the task passes no `-DeploymentDirectory`. Without the
  default-directory fix in commit `78e8770` the wrapper would stop at
  parameter binding under Windows PowerShell 5.1 before writing any log, so a
  successful run implies the deployed wrapper includes that fix. The deployed
  commit itself has not been recorded.

### E-5 Isolated real-Docker acceptance of the Windows wrapper

- **Observation:** the opt-in harness
  `northstar-api/tests/test_india_windows_docker_acceptance.py` passed 14 of 14
  tests on the Docker Desktop host.
- **Source:** operator-reported for the result. The harness and its merge
  (`northstar-api` PR #36) are inspected.
- **Artifacts attached:** none. To attach: the pytest output (`-v -rs`) and the
  session names it printed.

### E-6 Production SQLite non-interference

- **Observation:** snapshots of the production SQLite database taken before and
  after the isolated acceptance run were byte-identical.
- **Source:** operator-reported.
- **Artifacts attached:** none. The hashes were not provided and are not
  recorded here. To attach: the hash algorithm and the two digests, and how the
  snapshots were taken (read-only).

### E-7 October 9 candle revision and pending finality

- **Observation:** finality mode is `operator-approved` with
  `NORTHSTAR_FUTURES_FINAL_THROUGH=2026-10-08`. The 2026-10-09 session is not
  approved because its Upstox daily candle was seen to change. Observations of
  NIFTY 2026-10-27, session 2026-10-09:

  | Observed | Open | High | Low | Close | Volume (provider units) | Open interest |
  |----------|------|------|-----|-------|-------------------------|---------------|
  | 2026-10-09 | 22356 | 22665 | 22351 | 22625 | 3910335 | 19064630 |
  | 2026-10-10 | 22356 | 22665 | 22351 | 22612.6 | 3910335 | 18942950 |
  | 2026-10-10, about 21 minutes later | 22356 | 22665 | 22351 | 22612.6 | 3910335 | 18942950 |

- **Source:** operator-reported.
- **Note (inspected):** the evidence collector routes a request for the
  current venue date to Upstox's current-day endpoint and any other date to the
  historical endpoint (`northstar-infrastructure/.../market_data/upstox_candle_evidence.py`).
  The first observation therefore came from the current-day endpoint and the
  later two from the historical endpoint, so the change may reflect the endpoint
  rather than a revision over time. This is not yet resolved.
- **Artifacts attached:** none in the repositories; evidence files stay private
  (`Upstox-Finality-Evidence.md`).

### E-8 M1.2 dashboard snapshot consistency correction

- **Defect (engineering session):**
  `northstar-api/tests/test_futures_daily_operation.py::test_dashboard_reads_during_the_daily_write_stay_consistent`
  intermittently raised `ValueError: FuturesPaperTradingSnapshot valuation must
  value the snapshot portfolio.` in its reader thread while still passing (3 of
  23 isolated runs). Cause: the snapshot read the trading history twice. A
  second race could load a fill without its order.
- **Correction (inspected):** `northstar-application` PR #66 (merge `9a0ab57`,
  fix commit `d2db174`, 2026-10-10) and `northstar-api` PR #38 (merge
  `b986e31`, test commit `54810e9`, 2026-10-10), both merged to development.
- **Results (engineering session, 2026-10-10, development machine):**

  | Check | Result |
  |-------|--------|
  | New deterministic regression tests, before the fix | 5 failed, 10 passed (`-k "committed or coherent or once"`) |
  | The same tests, after the fix | 15 passed |
  | Strengthened threaded test against the unfixed code | 3 of 20 runs failed with the original `ValueError` |
  | Strengthened threaded test after the fix | 50 of 50 consecutive runs passed, unhandled thread exceptions treated as errors |
  | `northstar-application` full suite | 2853 passed |
  | `northstar-api` full suite | 681 passed, 18 skipped |
  | `northstar-infrastructure` full suite | 1848 passed, 27 skipped |
  | Ruff check and format (application, API) | clean |

- **Production:** **not deployed.** No production deployment of M1.2 has been
  established.

### E-9 Historical online SQLite backup tests (persistence audit)

- **Observation:** earlier online SQLite backups of the production database
  returned `PRAGMA integrity_check = ok` and no foreign-key violations, and
  production snapshots taken before and after the isolated Docker acceptance
  run were identical (see also E-6). Their output is in the deployment
  machine's evidence directory
  `C:\Local Disk(E)\Codes\NorthstarEvidence\persistence-audit`.
- **Source:** operator-reported. Neither the files nor the method were
  inspected when this entry was written.
- **What this is not:** these were backup checks only. **No restore drill**
  was performed, and they did not use the procedure in the
  [Windows SQLite Backup and Isolated Restore Drill](Indian-Futures-Windows-SQLite-Backup-and-Restore-Drill.md).
  They do not satisfy B-1 or B-2.
- **Artifacts attached:** none. To attach: the file names, the method used,
  and the integrity and foreign-key output.

### E-10 M1.4.3.2 expiry exception detection

- **Change (inspected):** uncommitted working-tree changes in `northstar-api`
  (`cli.py`, `operations.py`, `operational_status.py`, `_cli_rendering.py`,
  `schemas/futures.py`, the Windows wrapper) and `northstar-web` (dashboard
  card). `operations daily` reports `STATUS: EXPIRY EXCEPTION`, exit 7, when
  the operated contract holds a position or pending order and either no
  session is left through expiry (S-1) or the Asia/Kolkata date is after its
  expiration date (S-2). See the
  [Expiry Exception Operator Procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md),
  sections 12 and 13.
- **Results (engineering session, 2026-10-10, development machine):**

  | Check | Result |
  |-------|--------|
  | `northstar-api/tests/test_india_expiry_exception_detection.py` (V-1 to V-5, combinations, boundaries) | 18 passed |
  | `northstar-api` full suite | 890 passed, 18 skipped |
  | Windows wrapper suite (fake docker), including exit 7 | 143 passed, 3 skipped |
  | `northstar-web` tests | 63 passed; `tsc -b` and ESLint clean |
  | Ruff check and format (API) | clean |

- **Production:** **not deployed, not committed, not approved.** It does not
  satisfy X-4 on its own.

### E-11 M1.4.3.3 synthetic expiry exception operator exercise

- **Observation (engineering session, 2026-10-10, development machine):** an
  isolated exercise ran V-1 to V-5 and the additional cases through the
  production CLI and runtime on temporary SQLite, with the fake-Upstox helpers,
  injected clocks and a refused network:
  - every exception case gave exit 7, V-3 and V-5 gave exit 3, and V-4 and the
    on-expiry-date case gave exit 0;
  - the SQL dump of each database was identical before and after every
    detection command;
  - the real Windows wrapper, fed the CLI's actual V-2 output through a fake
    `docker`, recorded outcome `EXPIRY_EXCEPTION`, exit 7.
  Details: [Expiry Exception Operator Procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md),
  section 13.
- **Commits:** `northstar-api` 545b3b6, `northstar-web` 4f6908c,
  `northstar-docs` 99f8517.
- **Not verified:**
  - real Task Scheduler suspension and last result: the wrapper run used a fake
    `docker` and proves only the wrapper's classification;
  - identifying the last actual fill (procedure section 7 limitation);
  - anything on the deployment machine.
- **Artifacts:** synthetic only, preserved on the development machine outside
  every repository in
  `C:\Code\Private\Northstar\NorthstarEvidence\M1.4.3.3-synthetic-2026-10-10`.
  - Contents: 27 files listed in `MANIFEST.sha256` (SHA-256 of the manifest
    itself: `23b565a781c56cfc9ba23f602f07f3a87051fb6e110ef07c7a3cb290f5f5b8c5`).
    They are the exercise script, the local report, the 21 evidence files,
    `CORRECTIONS.md` and the scan and preservation scripts.
  - Redaction: two wrapper files had the temporary directory, Windows account
    and computer name redacted; their original hashes are in `CORRECTIONS.md`.
  - No production data, credential, provider-derived data or database copy is
    included.
- **Production:** M1.4.3.2 is **not deployed**. This entry grants no approval
  and does not satisfy X-4.
