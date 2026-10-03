# Indian Futures Deployment Runbook (NIFTY / NSE / Upstox)

Operator runbook for the isolated Indian Futures paper-trading environment.
The deployment artifacts live in `northstar-api/deploy/india/`; this document
explains how to deploy, operate, back up, roll back and roll over that stack
without affecting the existing CME/ES deployment.

## 0. Gates before a persistent deployment

Two open gates apply. Neither blocks building or testing the deployment
artifacts, but both must be respected before running the stack persistently.

1. **Provider / legal gate.** Written permission from Upstox to retain and use
   its market data on a persistent server is unresolved. Technical API access
   does not grant retention or use rights. A persistent server deployment that
   stores Upstox candles in SQLite is a deployment/legal decision that must be
   cleared first. Real Upstox data must not be committed to any repository or
   used as a test fixture.
2. **November Muhurat gate.** The NSE calendar reference fails closed on
   2026-11-08 (Muhurat trading) until the official session timings are loaded
   into it. The November contract cannot be operated until then (see
   section 13). Never bypass the calendar.

## 1. Topology

Two stacks share one Docker host and nothing else.

|                 | CME stack (unchanged)                        | Indian stack                                    |
|-----------------|----------------------------------------------|-------------------------------------------------|
| Checkout        | `/opt/northstar`                             | `/opt/northstar-india` (separate checkout)      |
| Deploy dir      | `/opt/northstar/northstar-api/deploy`        | `/opt/northstar-india/northstar-api/deploy/india` |
| Compose project | `northstar`                                  | `northstar-india` (pinned by `name:`)           |
| Services        | `api`, `operations`, `web`                   | `india-api`, `india-operations`, `india-web`    |
| Images          | `northstar-api:local`, `northstar-web:local` | `northstar-api:india`, `northstar-web:india`    |
| Data volume     | `northstar_northstar-data`                   | `northstar-india-data`                          |
| Caddy volumes   | `northstar_caddy-data`, `northstar_caddy-config` | `northstar-india-caddy-data`, `northstar-india-caddy-config` |
| Environment     | `deploy/.env`                                | `deploy/india/.env`                             |
| systemd         | `northstar-daily.service` / `.timer`         | `northstar-india-daily.service` / `.timer`      |
| Site            | Caddy on 80/443                              | Caddy on `127.0.0.1:8081` (SSH tunnel)          |
| Provider        | Databento                                    | Upstox                                          |

Inside each container the database is `/data/northstar.sqlite3`; the Docker
volume is what separates the two databases. The Indian volume has an explicit
Docker name, so it is `northstar-india-data` even if a command passes another
project name, and the Indian compose file never references the CME volume.

Indian services:

- `india-api` -- the read-only dashboard API over the Indian volume. It
  receives the contract, strategy, portfolio, target and the safe finality
  settings (mode, final-through, go-live) so the dashboard can show the
  operational status. It never receives the Upstox token. No port is
  published; only `india-web` reaches it.
- `india-operations` -- `northstar operations daily` with
  `NORTHSTAR_FUTURES_MARKET_DATA_PROVIDER=upstox` fixed in `compose.yaml`.
  It is not a long-lived service: each `docker compose run --rm` reads `.env`
  afresh, runs once and is removed. It is the only service that receives
  `UPSTOX_ANALYTICS_TOKEN`.
- `india-web` -- Caddy serving the static dashboard behind basic auth, with
  the Indian `Caddyfile` mounted read-only over the one baked into the shared
  web image. Only `GET`/`HEAD` on `/api/health` and `/api/futures/dashboard`
  are proxied to the API; every other `/api/*` path or method gets 404, and
  the legacy equity POST endpoints (`/analyze`, `/watchlist/refresh`) are not
  proxied at all. There is no control, run or approval endpoint.

### Why a separate checkout and distinct image tags

Both stacks build from `northstar-api/deploy/Dockerfile.api` and
`Dockerfile.web`. If they shared a checkout or an image tag, rebuilding India
would silently change the image CME runs on its next timer tick. Therefore:

- the Indian stack is built from its own checkout (`/opt/northstar-india`),
  never from CME's `/opt/northstar`;
- the Indian images are tagged `northstar-api:india` and `northstar-web:india`;
  CME keeps `northstar-api:local` and `northstar-web:local`.

Building in one checkout never touches the other's sources or images.

## 2. Machines and paths

Every command block in this runbook is labelled **DEVELOPMENT MACHINE** or
**DEPLOYMENT MACHINE**. They are separate machines; never mix them.

- **Development machine (Windows):** workspace `C:\Code\Private\Northstar`,
  PowerShell, the `northstar-api` virtualenv. Used to author the deployment
  artifacts and test them statically, and for code review. Docker, Caddy,
  systemd, a real `.env`, a real token, volumes and containers are never used
  here, and nothing here touches the deployed database.
- **Deployment machine:** a separate host holding the Indian checkout at
  `/opt/northstar-india`; every `docker compose` command below is run from
  `/opt/northstar-india/northstar-api/deploy/india` unless stated otherwise.
  These artifacts assume what the existing CME deployment already uses: Linux
  with Docker Engine, Docker Compose v2 and systemd. Those are requirements of
  the deployment machine, not facts about the development machine; a host
  without systemd would need a different scheduler.

Development machine, static checks of the deployment artifacts (text facts
only; they do not run Docker, Compose or Caddy):

**DEVELOPMENT MACHINE**

```powershell
cd C:\Code\Private\Northstar\northstar-api
.\.venv\Scripts\python.exe -m pytest -q tests\test_india8e_indian_deployment.py
```

### Pending deployment-machine validation

The artifacts were authored and tested statically on the development machine
only. None of the following has been performed yet; each is deferred to the
deployment machine:

- `docker compose config` against `deploy/india/compose.yaml` with the real
  `.env`;
- Caddy validation of `deploy/india/Caddyfile` (for example `caddy validate`
  inside the `northstar-web:india` image);
- building the `northstar-api:india` and `northstar-web:india` images;
- creation of the `northstar-india-data` and Caddy volumes;
- container startup and the `india-api` health check;
- installation of the `northstar-india-daily` service and timer;
- the real `.env` and secrets (Upstox token, basic-auth login);
- economics, historical bootstrap and every database step inside the
  deployment.

## 3. Environment and secrets

`deploy/india/.env` is created from `deploy/india/.env.example` on the host
and is git-ignored (every `.env` in `northstar-api` is). It is excluded from
both image build contexts.

| Class | Setting | Notes |
|-------|---------|-------|
| Secret | `UPSTOX_ANALYTICS_TOKEN` | `india-operations` only. Needed for `market-data sync` and for operator-approved runs; disabled finality neither reads nor needs it. Never printed, never passed to the API. |
| Secret | `NORTHSTAR_BASIC_AUTH_HASH` (with `NORTHSTAR_BASIC_AUTH_USER`) | Caddy login for the Indian dashboard; `india-web` only. |
| Deployment | `NORTHSTAR_DATABASE` | `/data/northstar.sqlite3` on the Indian volume. |
| Deployment | `NORTHSTAR_FUTURES_PRODUCT`, `NORTHSTAR_FUTURES_EXCHANGE`, `NORTHSTAR_FUTURES_EXPIRATION` | The one exact dated contract, e.g. `NIFTY` / `NSE` / `2026-10-27`. |
| Deployment | `NORTHSTAR_STRATEGY`, `NORTHSTAR_PORTFOLIO`, `NORTHSTAR_TARGET` | Strategy label, monthly portfolio identity, fixed target. |
| Deployment | `NORTHSTAR_FUTURES_DAILY_BAR_FINALITY` | `disabled` (example default; fails closed) or `operator-approved`. |
| Deployment | `NORTHSTAR_FUTURES_FINAL_THROUGH` | Last trading date the operator has verified as final; required when operator-approved. |
| Deployment | `NORTHSTAR_FUTURES_GO_LIVE` | First session to decide on, used only while no decision is frozen. |
| Deployment | `NORTHSTAR_INDIA_WEB_PORT`, `NORTHSTAR_WEB_ORIGIN` | Loopback port of the Indian site and its origin. |
| Fixed in compose | `NORTHSTAR_FUTURES_MARKET_DATA_PROVIDER=upstox` | Not in `.env`. |
| Frozen in code | K = 5 pre-expiry flatten, strategy semantics and warm-up, next-open fills, NSE calendar | Never environment settings. |

There is deliberately no setting for K, the strategy window, automatic
rollover, or any time-of-day finality threshold (no "21:00", no "next
morning", no delay in minutes, no IST cut-off).

## 4. First deployment

All steps on the deployment host unless stated otherwise.

### A. Create the Indian checkout and configuration

Check out every sibling repository side by side under `/opt/northstar-india`
(the images build from that workspace root), at the commit being deployed.

**DEPLOYMENT MACHINE**

```sh
cd /opt/northstar-india/northstar-api/deploy/india
cp .env.example .env
chmod 600 .env
docker run --rm caddy:2-alpine caddy hash-password     # value for NORTHSTAR_BASIC_AUTH_HASH
```

Edit `.env`: set the basic-auth user and hash, the Upstox token (needed for
the bootstrap sync), and confirm the contract, strategy and portfolio.

### B. Start with finality disabled

Keep `NORTHSTAR_FUTURES_DAILY_BAR_FINALITY=disabled` and leave
`NORTHSTAR_FUTURES_FINAL_THROUGH` and `NORTHSTAR_FUTURES_GO_LIVE` empty.

**DEPLOYMENT MACHINE**

```sh
docker compose build
```

### C. Contract economics

Set the economics of the exact dated contract once, before its first fill.
For the October 2026 contract: NIFTY / NSE / 2026-10-27, 65 INR per quote
point per contract (a lot of 65).

**DEPLOYMENT MACHINE**

```sh
docker compose run --rm --no-deps india-operations northstar economics set \
  --database /data/northstar.sqlite3 --product NIFTY --exchange NSE \
  --expiration 2026-10-27 --point-value 65 --currency INR
```

### D. Bootstrap historical daily bars (no paper decisions)

Acquire completed history for the strategy warm-up (at least twenty
sessions). For the October experiment: from 2026-07-29 through the last
completed session before go-live, `<PRE_GO_LIVE>`. This only stores market
data; do not run `paper run` during bootstrap.

**DEPLOYMENT MACHINE**

```sh
docker compose run --rm --no-deps india-operations northstar market-data sync \
  --database /data/northstar.sqlite3 --product NIFTY --exchange NSE \
  --expiration 2026-10-27 --start 2026-07-29 --end <PRE_GO_LIVE> --provider upstox
```

### E. Verify an untouched portfolio

The cutoff can be any instant at or after the `<PRE_GO_LIVE>` close (NSE
regular sessions close at 10:00 UTC).

**DEPLOYMENT MACHINE**

```sh
docker compose run --rm --no-deps india-operations northstar paper status \
  --database /data/northstar.sqlite3 \
  --strategy directional-mvp-v1-nifty-paper-ops --portfolio nifty-paper-ops-oct26 \
  --as-of <PRE_GO_LIVE>T10:00:00+00:00
```

Expect `Orders: 0`, `Fills: 0`, `Pending: 0` and `Flat` under
`PORTFOLIO (all contracts)`.

Then start the dashboard and check it (section 8):

**DEPLOYMENT MACHINE**

```sh
docker compose up -d
docker compose ps                     # india-api healthy, india-web up
```

The Operational Status card shows finality `disabled` and backlog
`Go-live not configured`.

### F. Set the explicit go-live session

Set `NORTHSTAR_FUTURES_GO_LIVE=<GO_LIVE>`, the first NSE trading session after
`<PRE_GO_LIVE>`. It must be a trading session no later than expiry; the
operation refuses anything else.

**DEPLOYMENT MACHINE**

```sh
docker compose up -d india-api        # recreate the API with the new value
```

The dashboard now shows the next required session `<GO_LIVE>`, finality
`UNKNOWN`, backlog caught up (nothing is processable while disabled).

### G. Enable operator-approved finality when ready

Only when the go-live session's daily candle has been verified as final
(section 5), set:

**DEPLOYMENT MACHINE**

```
NORTHSTAR_FUTURES_DAILY_BAR_FINALITY=operator-approved
NORTHSTAR_FUTURES_FINAL_THROUGH=<GO_LIVE>
```

### H. Process sessions chronologically

Install the timer (section 6) or run the operation once now:

**DEPLOYMENT MACHINE**

```sh
docker compose up -d india-api
sudo systemctl start northstar-india-daily.service
journalctl -u northstar-india-daily.service -n 200
```

Market data may be acquired as a range, but paper decisions are taken one
session at a time, oldest first, each at its own close. Sessions are never
skipped (section 10).

## 5. Operator-approved finality (daily workflow)

Automatic Upstox finality is not implemented. Finality is an explicit
operator decision recorded in `NORTHSTAR_FUTURES_FINAL_THROUGH`. For each new
session `D`:

1. **Verify** outside Northstar that the provider's daily candle for `D` is
   final. Northstar does not and cannot decide this; no clock time does either.
2. **Approve** by editing `.env`: `NORTHSTAR_FUTURES_FINAL_THROUGH=D`. Approve
   one session at a time; never approve a session whose candle you have not
   verified.
3. **Refresh the dashboard's view** of the setting (the operation itself reads
   `.env` afresh on every run, so it needs nothing):

   **DEPLOYMENT MACHINE**

   ```sh
   cd /opt/northstar-india/northstar-api/deploy/india
   docker compose up -d india-api
   ```

4. **Run** now, or let the next hourly timer tick pick it up:

   **DEPLOYMENT MACHINE**

   ```sh
   sudo systemctl start northstar-india-daily.service
   ```

5. **Inspect** the log and the dashboard. Expect `STATUS: COMPLETED -- 1
   session(s) processed`, then on later ticks `STATUS: WAITING` at `D+1` with
   finality `NOT_YET_FINAL`. The dashboard shows `D` as the latest decision
   session and the backlog caught up.
6. **Move forward** only after `D` is accepted: verify `D+1` and repeat.

If the operation has fallen behind, approving through a later date makes the
next run acquire every approved session as one range and decide each in order.

To stop processing at once, set `NORTHSTAR_FUTURES_DAILY_BAR_FINALITY=disabled`
(and `docker compose up -d india-api`); later runs then wait without contacting
the provider.

## 6. Scheduled operation

**DEPLOYMENT MACHINE**

```sh
cd /opt/northstar-india/northstar-api/deploy/india
sudo cp systemd/northstar-india-daily.service systemd/northstar-india-daily.timer /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now northstar-india-daily.timer
systemctl list-timers northstar-india-daily.timer
```

The service runs `docker compose --project-name northstar-india run --rm
--no-deps india-operations` from the Indian deploy directory only. The timer
fires hourly. The trigger time is only a check for approved work: it never
establishes finality, because finality is `disabled` or operator-approved. An
hourly check picks up a new approval within the hour; a run with nothing
approved makes no provider request and exits 0 (`WAITING`). The CME units
`northstar-daily.service` / `.timer` are separate and untouched.

## 7. Exit codes and retry meanings

| Exit | Meaning for `operations daily` | Operator action |
|------|-------------------------------|-----------------|
| 0 | `COMPLETED` (sessions processed), `WAITING` (nothing approved/final), or `SKIPPED` (another runner holds the lock) | None. |
| 2 | Invalid input | Fix the command. |
| 3 | Configuration: invalid settings, operator-approved without token or final-through, go-live missing/invalid, non-NSE contract, or **rollover required** after expiry | Fix `.env`; at rollover follow section 13. |
| 4 | Data: e.g. contract economics missing for valuation | Set economics; rerun. |
| 5 | State: a manual writer collided with an active runner, a persisted conflict, or SQLite busy (`database busy ...; retry later`) | Retry when the other writer finishes; investigate conflicts. |
| 6 | Provider or calendar: Upstox failure or edge block, or the NSE calendar failing closed (e.g. Muhurat) | Retry later; nothing after the failure was decided. |
| 1 | Internal | Inspect logs. |

The operation stops at the first failure, so no later session is ever decided
past one that did not complete. A rerun resumes from the latest frozen
decision; completed sessions are idempotent.

## 8. Dashboard access

The Indian site listens only on host loopback `127.0.0.1:8081` (the CME site
keeps 80/443; no DNS changes). From the development machine, open an SSH
tunnel and browse locally:

**DEVELOPMENT MACHINE**

```powershell
ssh -N -L 8081:127.0.0.1:8081 <user>@<deployment-host>
# then open http://localhost:8081/#futures and sign in with the Indian basic-auth login
```

On the host itself:

**DEPLOYMENT MACHINE**

```sh
curl -u <user> http://127.0.0.1:8081/api/health                 # {"status":"ok"}
curl -u <user> http://127.0.0.1:8081/api/futures/dashboard
```

The Operational Status card shows finality mode and final-through, the
next session's finality, latest market and decision sessions, the next
required session, backlog, expiry safety and rollover. It reads only
persisted facts and the API's settings; it never contacts Upstox, reads no
clock and writes nothing. It is the latest processed state, not a record of
the last run; run outcomes are only in the operation log (section 12).

## 9. Single writer and locks

The runner owns `DatabaseOperationsLock`, an OS file lock beside the database
(`/data/northstar.sqlite3.operations.lock` on the Indian volume). No shell
lockfile or `flock` wraps compose.

- Another scheduled runner while one is active: logs `DAILY OPERATION:
  SKIPPED` and exits 0.
- A manual writer (`economics set`, `market-data sync`, `paper run`) while the
  runner is active: exits 5 (STATE); retry afterwards.
- The read-only dashboard stays available throughout.

## 10. Chronological catch-up

- Market data may be caught up as a range: one acquisition covers every
  approved session.
- Paper decisions may not skip sessions: each session is decided at its own
  close, oldest first. Never "catch up" by running `paper run` only at the
  latest cutoff; that silently skips the intermediate decisions.
- After a gap, approve through the latest verified session and run once; the
  operation processes the backlog in order and stops at the first failure.

## 11. Expiry flatten (code-frozen, K = 5)

For the October contract, expiry E = 2026-10-27 (2026-10-20 is an NSE
holiday):

- decision at E-6 (2026-10-16) close: flatten required; the closing order
  fills at the E-5 (2026-10-19) open;
- E-5 through E: protected; the portfolio stays flat and no signal may reopen
  or increase the contract.

The dashboard's Expiry safety block shows the window (`Outside flatten
window`, `Flatten session`, `No-reopen window`), whether flatten is required,
whether the contract position is flat, and whether reopening is blocked.

## 12. Logs

There is no run-journal table; the operation's stdout/stderr is the record.

**DEPLOYMENT MACHINE**

```sh
journalctl -u northstar-india-daily.service -n 200          # latest runs
journalctl -u northstar-india-daily.service --since today
cd /opt/northstar-india/northstar-api/deploy/india
docker compose logs --tail 200 india-api                    # dashboard API
docker compose logs --tail 200 india-web                    # Caddy
```

Each operation run logs (`northstar.operations` lines plus the rendered
report): the finality mode, each session assessed and its finality outcome,
the acquired range, each session's decision and paper order (with
`Expiry flatten: yes` when the guard governs it), fills, P&L, the final
`STATUS:` line, and on failure `<CLASS> ERROR: ...` with the exit
classification. The token is redacted and never printed.

## 13. Manual contract rollover

Nothing rolls automatically. After expiry is processed every run exits 3
(`Rollover required`), and the dashboard shows backlog `Rollover required`.

1. Confirm the current contract reached the protected state and E was
   processed (dashboard: `No-reopen window`, position flat, rollover required).
2. Stop the timer and take a backup (section 14):

   **DEPLOYMENT MACHINE**

   ```sh
   sudo systemctl stop northstar-india-daily.timer
   ```

3. Edit `.env` for the next exact contract, with finality reset:

   **DEPLOYMENT MACHINE**

   ```
   NORTHSTAR_FUTURES_EXPIRATION=2026-11-23
   NORTHSTAR_PORTFOLIO=nifty-paper-ops-nov26
   NORTHSTAR_FUTURES_DAILY_BAR_FINALITY=disabled
   NORTHSTAR_FUTURES_FINAL_THROUGH=
   NORTHSTAR_FUTURES_GO_LIVE=<next go-live session>
   ```

   Keep the same strategy identity unless the strategy version changes. A new
   monthly portfolio identity starts flat; the expired month's facts stay in
   the same Indian database.
4. Set the next contract's own economics (verify that expiry's lot size; do
   not copy it from another expiration):

   **DEPLOYMENT MACHINE**

   ```sh
   docker compose run --rm --no-deps india-operations northstar economics set \
     --database /data/northstar.sqlite3 --product NIFTY --exchange NSE \
     --expiration 2026-11-23 --point-value <INR per point> --currency INR
   ```

5. Bootstrap the next contract's history (at least twenty sessions, ending
   before its go-live) with `market-data sync` as in section 4D, and verify
   the new portfolio is flat as in section 4E.
6. `docker compose up -d india-api`, check the dashboard, then approve
   sessions explicitly as in section 5 and restart the timer.

### November blocker

The November contract (expiration 2026-11-23) must not be activated until the
official NSE Muhurat session timings for 2026-11-08 are loaded into the NSE
calendar reference. Until then the calendar fails closed on that date. The
operation plans every session from go-live through expiry, so with the
November contract configured every run fails closed (exit 6) and the
dashboard backlog shows unavailable -- not only after 2026-11-08. Bootstrap
syncs whose range ends before 2026-11-08 still work. Do not bypass or edit
around the calendar; loading the timings is a separate code change.

## 14. Backups

Indian backups are separate from CME's and use SQLite's online backup API,
never a raw copy of the live file. Stop the writer first; the API may keep
reading.

**DEPLOYMENT MACHINE**

```sh
cd /opt/northstar-india/northstar-api/deploy/india
sudo install -d -m 700 /var/backups/northstar-india
sudo systemctl stop northstar-india-daily.timer
systemctl is-active northstar-india-daily.service      # wait until inactive
docker compose exec india-api python -c "import sqlite3; s = sqlite3.connect('/data/northstar.sqlite3'); d = sqlite3.connect('/data/backup.sqlite3'); s.backup(d); d.close(); s.close()"
docker compose cp india-api:/data/backup.sqlite3 ./northstar-india-backup.sqlite3
sudo mv ./northstar-india-backup.sqlite3 /var/backups/northstar-india/northstar-india-$(date -u +%FT%H%MZ).sqlite3
docker compose exec india-api rm /data/backup.sqlite3
sudo systemctl start northstar-india-daily.timer
```

Retention is manual. CME backups remain wherever the CME runbook puts them
and are never touched by these commands.

**Restore (concept and commands).** Stop every Indian reader and writer,
write the chosen backup into the Indian volume through the SQLite backup API,
then start again:

**DEPLOYMENT MACHINE**

```sh
cd /opt/northstar-india/northstar-api/deploy/india
sudo systemctl stop northstar-india-daily.timer
systemctl is-active northstar-india-daily.service      # wait until inactive
docker compose stop india-web india-api
sudo install -m 644 /var/backups/northstar-india/<backup>.sqlite3 /tmp/northstar-india-restore.sqlite3
docker compose run --rm --no-deps --volume /tmp/northstar-india-restore.sqlite3:/restore.sqlite3:ro \
  india-operations python -c "import sqlite3; s = sqlite3.connect('/restore.sqlite3'); d = sqlite3.connect('/data/northstar.sqlite3'); s.backup(d); d.close(); s.close()"
sudo rm /tmp/northstar-india-restore.sqlite3
docker compose up -d
sudo systemctl start northstar-india-daily.timer
```

## 15. Rollback

Rolling back the Indian stack never affects CME, which keeps running.

Stop the Indian stack, keeping its data:

**DEPLOYMENT MACHINE**

```sh
sudo systemctl disable --now northstar-india-daily.timer
cd /opt/northstar-india/northstar-api/deploy/india
docker compose stop
```

Keep a known-good image and configuration before every rebuild, so a rollback
is a retag rather than a rebuild:

**DEPLOYMENT MACHINE**

```sh
docker tag northstar-api:india northstar-api:india-previous
docker tag northstar-web:india northstar-web:india-previous
cp .env .env.previous
```

Roll back to them:

**DEPLOYMENT MACHINE**

```sh
docker tag northstar-api:india-previous northstar-api:india
docker tag northstar-web:india-previous northstar-web:india
cp .env.previous .env
docker compose up -d
sudo systemctl enable --now northstar-india-daily.timer
```

The Indian data volume `northstar-india-data` is preserved throughout. If the
Indian containers and network must be removed, use plain `docker compose down`
from the Indian directory with no volume flag; named volumes are kept.

## 16. CME coexistence

- During Indian engineering acceptance, keep the CME stack unchanged and
  running: its containers, its `northstar_northstar-data` volume, its timer
  and its backups.
- Before deploying India on the same host, use the separate checkout and the
  distinct image tags described in section 1.
- After Indian acceptance and an agreed soak period, the CME timer may be
  disabled manually (`sudo systemctl disable --now northstar-daily.timer`).
  Nothing in this story does that automatically.
- Never delete CME data. The CME database is historical reference evidence.

Commands that must never be run on this host, for either stack: `docker
compose down` with the `-v`/`--volumes` flag, `docker volume rm` of any
Northstar volume, and `docker volume prune`.

## 17. Windows process note (development machine)

On Windows a virtualenv's `python.exe` can be a launcher that starts the real
interpreter as a child process. Killing only the launcher can leave the real
interpreter running, and that process still holds the operations lock
legitimately. This is not a stale lock: the lock is released as soon as the
process that owns it exits. When stopping a Windows runner manually, stop the
actual interpreter process:

**DEVELOPMENT MACHINE**

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'python.exe'" |
  Select-Object ProcessId, ParentProcessId, CommandLine
Stop-Process -Id <interpreter process id>
```

Linux containers under systemd are not affected by this launcher detail.
