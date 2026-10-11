# Indian Futures: Windows Controlled Upgrade and Rollback

Status: **prepared for operator review; not approved; never run**
(M1.5.2, 2026-10-11; safety review corrections the same day).
- Nothing here has been executed on any machine.
- The commands were statically validated only (section 12).
- This document authorizes no release, approves no requirement, changes no
  finality and grants no G-1 permission.
- The current state is unchanged: the production paper stack is **not**
  known to run M1.2, S-1 or S-2.

Related:
- [Deployment Runbook](Indian-Futures-Deployment-Runbook.md) (sections 1, 4,
  8, 14, 15 and 18);
- [Paper MVP Readiness Assessment](Indian-Futures-Paper-MVP-Readiness-Assessment.md);
- [Expiry Exception Operator Procedure](Indian-Futures-Expiry-Exception-Operator-Procedure.md);
- [Windows SQLite Backup and Isolated Restore Drill](Indian-Futures-Windows-SQLite-Backup-and-Restore-Drill.md);
- [Paper MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md).

**Paths and labels.** Every command block is for the **DEPLOYMENT MACHINE
(Windows, Windows PowerShell 5.1, as the Docker Desktop user)**. The
deployment root is `C:\Local Disk(E)\Codes\Northstar`, as in the expiry
procedure and the drill. Source paths in the text are relative to the
Northstar workspace root.

**How to run a block.** Paste each block whole into the PowerShell window. Every block
is wrapped in `. { ... }` so that the console reads it as one statement before running
it. A `throw` (STOP) therefore ends the whole block: no later line of it runs. The
variables and functions it defines stay available in the same session, which is what
7.1 to 7.8 rely on.

**Classes of step.** Every step is one of four classes:

| Class | Meaning |
|-------|---------|
| **R** | Read-only release check. Writes no production data. Creates only release records |
| **C** | Change to code, images, containers or the scheduler. Never to data |
| **I** | Isolated synthetic check (an acceptance project). Never production |
| **W** | Real supervised operation that **may write** production SQLite. Each needs its own explicit operator authorization |

## 1. Purpose and supported scope

**Purpose.** To replace the India paper stack's code, without touching its
data, with:
- M1.2 (dashboard consistency);
- S-1 and S-2 (expiry detection, CLI exit 7);
- the wrapper's exit-7 classification;
- the web expiry warnings.

The independent P&L verifier (M1.4.4.2) is test-only and is not deployed.

**Deployment architecture** (verified in the repositories):

| Component | Definition | Identity |
|-----------|------------|----------|
| API image | `northstar-api/deploy/Dockerfile.api`. Build context: the **workspace root**. It copies `northstar-core`, `northstar-application`, `northstar-infrastructure` and `northstar-api`, then runs `uv sync --frozen --no-dev --no-editable` | Tag `northstar-api:india`. Used by the `india-api` service (`CMD northstar-api`, port 8000) and the `india-operations` service (`northstar operations daily`) |
| Web image | `northstar-api/deploy/Dockerfile.web`. It copies `northstar-web` and builds with Node 22.23.2 (`npm ci`, `npm run build`) on `caddy:2-alpine` | Tag `northstar-web:india`; service `india-web` |
| Compose | `northstar-api/deploy/india/compose.yaml`: project `northstar-india`. `india-api` has a healthcheck on `http://127.0.0.1:8000/health` and `restart: unless-stopped`. `india-operations` has profile `operations` and `restart: "no"`. `india-web` publishes `127.0.0.1:${NORTHSTAR_INDIA_WEB_PORT:-8081}:8080`, depends on a healthy `india-api`, and mounts `./Caddyfile` read-only. Only `india-api` and `india-operations` mount `northstar-india-data` | -- |
| Data | Named volume `northstar-india-data` mounted at `/data`; database `/data/northstar.sqlite3`. Also Caddy volumes `northstar-india-caddy-data` and `northstar-india-caddy-config` | Never removed, never recreated |
| Environment | `northstar-api/deploy/india/.env` (secrets: Upstox token, basic-auth hash). Neither build context contains it: `Dockerfile.api.dockerignore` excludes `northstar-*/deploy`, and `Dockerfile.web.dockerignore` is an allowlist. Git ignores `.env` only, **not** copies such as `.env.rollback-*`; so no copy may be placed inside a checkout (section 12) | Never changed by a release |
| Wrapper | `northstar-api/deploy/india/windows/Invoke-NorthstarIndiaOperations.ps1`, run **from the checkout** by the task. It names its container `northstar-india-operations-<run id>`, has `-PreflightOnly` and isolated `-ProjectName northstar-india-acceptance-<id>` modes, and records `finalityMode` and `finalThrough` in `last-run.json` | Changes the moment the checkout changes |
| Scheduler | Task `Northstar India Operations` (`northstar-india-operations.task.xml`): hourly, `IgnoreNew`, 55-minute limit. Registered by `Register-NorthstarIndiaOperationsTask.ps1` | Unchanged by this release |
| Locks | `DatabaseOperationsLock` (a lock file beside the database, held by every writer). A writer must run in a container that mounts `northstar-india-data`; status reads (`paper status`, dashboard) take no lock | Writers are found by volume, not by name (section 6) |
| Read-only status | `GET /health`; `northstar paper status` (runbook section 4E); wrapper `-PreflightOnly`; `last-run.json`; `Get-ScheduledTaskInfo` | -- |
| Provenance | **None in the images.** The Dockerfiles set no source labels, and locally built images carry no registry digest. The deployed commit has never been recorded (evidence register E-4) | Section 3 |

**Not covered:**
- any change to SQLite data;
- restoring a database;
- changing finality, `.env` or the task definition;
- the Linux/systemd variant (runbook section 15). Its `.env.previous` copy
  must not be used on this machine (section 12);
- any G-1 decision.

## 2. Version and environment prerequisites

- Runbook section 18A holds: Docker Desktop in Linux-containers mode, the
  Docker Desktop user signed in, and Windows PowerShell 5.1.
- The deployment checkout holds the repositories side by side under the
  deployment root (runbook section 4A).
- Each of the five **runtime** repositories must be a **clean Git work tree**
  with a remote named `origin`. Block 5 checks this; if any fails, STOP: the
  transfer method must be defined first.
- **Release commits:** every repository has a CI workflow
  (`.github/workflows/python-ci.yml`; `frontend-ci.yml` for web). Its result,
  or full local suites, must be recorded for the **exact** release commits
  before upgrading (acceptance sign-off item 10). Neither was checked here
  (offline).
- **Build-time network:** `docker compose build` pulls the base images
  (`ghcr.io/astral-sh/uv:0.12.17`, `python:3.13-slim`, `node:22.23.2-alpine`,
  `caddy:2-alpine`) and the npm and uv dependencies. These are not provider
  requests. `python:3.13-slim` and `caddy:2-alpine` are floating tags, so a
  rebuild is not bit-reproducible. The recorded image IDs are therefore the
  identity.

## 3. Release dependency manifest

**Required changes, by commit:**

| Change | Repository | Commit(s) on `develop` | Files |
|--------|------------|------------------------|-------|
| M1.2 dashboard consistency | northstar-application | `d2db174` (merge `9a0ab57`, PR #66) | `build_futures_paper_trading_valuation.py`, `get_futures_paper_trading_snapshot.py`, `run_futures_paper_trading_decision.py` |
| M1.2 regression test | northstar-api | `54810e9` (merge `b986e31`, PR #38) | `tests/test_futures_daily_operation.py` |
| S-1, S-2, exit 7, API schema, wrapper exit 7 | northstar-api | `71c32c1` (merge `545b3b6`, PR #39) | `cli.py`, `operations.py`, `operational_status.py`, `_cli_rendering.py`, `schemas/futures.py`, the wrapper, tests |
| Web expiry warnings | northstar-web | `5862618` (merge `4f6908c`, PR #8) | `src/futures/FuturesCards.tsx`, `types.ts`, `fixtures.ts`, tests |
| Verifier (test-only) | northstar-api | `d464f9c` (merge `cfa7774`, PR #40) | `tests/` only; outside the image (`northstar-*/tests` is ignored) |

**Unchanged between `main` and `develop`:** `compose.yaml`,
`Dockerfile.api`, `Dockerfile.web`, both `.dockerignore` files, both
Caddyfiles, the task XML, the registration script, and every repository's
`pyproject.toml`, `uv.lock` and web package files.

**Persistent data:**
- The futures tables and stores are identical on both branches.
- `initialize_database` (`northstar-api/src/northstar_api/runtime.py`)
  creates only futures tables, and `CREATE TABLE IF NOT EXISTS` is a no-op on
  existing ones.
- **No persistent-data change, so no data blocker.**

**Unrelated changes on `develop`:** India options. They are api PRs #30 to
#33 (`0bc9cda`, `2442fc6`, `ba69583`, `47bef8d`), application PRs #61 to #65,
and the corresponding core and infrastructure commits.
- They add only CLI subcommands, plus option tables that are created only
  when an option command runs.
- They add no HTTP route: the routers serve `/analyze`,
  `/watchlist/refresh`, `/health`, `/futures/dashboard` and
  `/futures/analysis`, and Caddy proxies only the last three.
- `operations daily` does not use them.
- Shipping them is an explicit acceptance item of AP-1.

**Two release approaches** (checked with `git apply --check` on `git archive`
exports of `main`; no repository was changed):

| Aspect | A: complete `develop` at recorded commits | B: narrow release (`main` plus the futures commits) |
|--------|-------------------------------------------|-----------------------------------------------------|
| Applies cleanly | It is the tested state | `d2db174`, `54810e9` and `5862618` apply to `main`. **`71c32c1` does not**: `cli.py` and `_cli_rendering.py` conflict with four options CLI commits (`e719684`, `53dc0ee`, `bf84bde`, `189b494`). A hand-merged backport would be needed |
| Functional dependencies | Complete | All symbols S-1 and S-2 use exist on `main` (snapshot `pending_orders`, `latest_daily_bar`, `latest_frozen_decision`, `runtime.snapshot`, `operational_status`, `ExpirySafetyResponse`, `expiry_guard_for`); M1.2 needs only application |
| Unrelated features | Ships inert options CLI code | None |
| Tested combination | Development suites ran on `develop` (E-10, E-12); still to be confirmed at the exact commits | **Never tested**; needs release branches in three repositories, conflict resolution and full CI |
| Cross-repository | Five built repositories at their `develop` heads | Mixed `main` plus patches. Local `northstar-core` `main` is 5 commits behind its `origin/main`, so the true `main` is not established offline |
| Rollback | By image ID and checkout commits | The same, plus branch bookkeeping |
| Documentation | The runbook on `develop` describes `develop` behaviour | It would need a separate documentation baseline |
| Risk of omitting a dependency | Lowest | Higher: a manual backport |

**Recommendation: approach A.** Release manifest, as full commit identities:

| Runtime repository | Commit (`develop` head, 2026-10-11) |
|--------------------|-------------------------------------|
| northstar-core | `e54ccdff023db6a97db4a3debfdab4fb15e5b875` |
| northstar-application | `9a0ab57288b440dec83d1680addfb12b8fc3e703` |
| northstar-infrastructure | `aa94428880de99f7c697577e5dfbe4987ffa9f0a` |
| northstar-api | `cfa7774b503b32c15263f8f2c0d905df0c1db622` |
| northstar-web | `4f6908ca48b1ed82ea70787021c9bffcaf3ab49f` |

**Documentation revision.** It is approved **separately** and is not built.
It must be the `northstar-docs` commit that contains this runbook at the
reviewed version. That commit does not exist yet: the current `develop` head
`3101a02` predates this document. The runbook's `northstar-docs` checkout is
not moved by section 7.3; AP-1 records which documentation commit was
followed.

If the approver rejects shipping options code, approach B becomes a separate
engineering task: a backport of `71c32c1` with full CI. It is not part of
this procedure.

## 4. Operator approvals

All are **PENDING**. Each is recorded with the name, date, time and the
approved values; this document grants none.

| # | Approval | Status | Notes |
|---|----------|--------|-------|
| AP-1 | Release scope: approach A at the five runtime commits of section 3. This explicitly **accepts shipping the unrelated India options code** (api PRs #30 to #33, application PRs #61 to #65 and the related core and infrastructure commits), and names the documentation commit followed | PENDING | Or approach B after its own engineering task |
| AP-2 | CI or full-suite results at those five commits are recorded | PENDING | Acceptance sign-off item 10 |
| AP-3 | Continuing persistent operation while G-1 is unresolved | PENDING | Gate owner and approver (section 13) |
| AP-4 | The deployment window, including disabling the hourly task (class C) | PENDING | Runs are idempotent and catch up afterwards |
| AP-5 | Which of `india-api` and `india-web` to replace or start (class C) | PENDING | Default: only those running in block 5 |
| AP-6 | One supervised `operations daily` run after the upgrade (class **W**) | PENDING | Only if the eligibility check of 7.7 passes. Otherwise it is a normal operational run, outside this procedure |
| AP-7 | Re-establishing the recorded scheduler state, after upgrade **or** rollback (class C) | PENDING | Never automatic; every failure path leaves the task disabled |
| AP-8 | Running `paper status` before and after (class R) | PENDING | Its output holds Upstox-derived values: G-1 retention; it stays on the deployment machine |

## 5. Preflight checks (class R)

**Block 5** records the current identities and enforces the runtime STOP
checks itself.
- It creates only a new release-record directory and writes nothing else.
- It does not mount the production volume.
- It never prints or stores a secret: the resolved Compose model is only
  hashed, and Git remote URLs are never read.

Note the printed stamp; every later block uses it.

```powershell
. {
    # Block 5 (class R): read-only preflight. Creates only a new release-record directory.
    $ErrorActionPreference = 'Stop'
    $Root = 'C:\Local Disk(E)\Codes\Northstar'
    $Deploy = Join-Path $Root 'northstar-api\deploy\india'
    $Records = 'C:\Local Disk(E)\Codes\NorthstarEvidence\releases'
    $TaskName = 'Northstar India Operations'
    $Stamp = (Get-Date).ToUniversalTime().ToString("yyyyMMdd'T'HHmmss'Z'")
    $Release = Join-Path $Records $Stamp
    $Utf8 = New-Object System.Text.UTF8Encoding($false)
    if (Test-Path -LiteralPath $Release) { throw "STOP: $Release already exists" }
    New-Item -ItemType Directory -Path $Release | Out-Null
    function Save([string] $Name, [object[]] $Lines) {
        [System.IO.File]::WriteAllLines((Join-Path $Release $Name), [string[]] $Lines, $Utf8)
    }
    # Native commands under a local 'Continue': in Windows PowerShell 5.1 a redirected stderr
    # line would otherwise become a terminating error under 'Stop'.
    function Invoke-Native([string] $Exe, [string[]] $Arguments, [switch] $AllowFailure) {
        $ErrorActionPreference = 'Continue'
        $output = & $Exe @Arguments 2>$null
        $code = $LASTEXITCODE
        if ($code -ne 0 -and -not $AllowFailure) { throw "STOP: $Exe $($Arguments[0]) $($Arguments[1]) exited $code" }
        [pscustomobject]@{ Code = $code; Output = @($output) }
    }
    function Get-ImageId([string] $Reference) {
        $result = Invoke-Native docker @('image', 'inspect', $Reference, '--format', '{{.Id}}') -AllowFailure
        if ($result.Code -ne 0) { return 'ABSENT' }
        [string] $result.Output[0]
    }
    Set-Location -LiteralPath $Deploy

    # 1. Host, Docker context, engine and client versions.
    $context = [string] (Invoke-Native docker @('context', 'show')).Output[0]
    $engine = [string] (Invoke-Native docker @('info', '--format', '{{.ID}}|{{.ServerVersion}}|{{.OperatingSystem}}|{{.OSType}}')).Output[0]
    Save 'host.txt' @("computer=$env:COMPUTERNAME", "account=$env:USERNAME", "utc=$Stamp", "powershell=$($PSVersionTable.PSVersion)")
    Save 'docker.txt' @("context=$context", "engine=$engine",
        "client=$((Invoke-Native docker @('version', '--format', '{{.Client.Version}}')).Output[0])",
        "compose=$((Invoke-Native docker @('compose', 'version', '--short')).Output[0])")
    if ($context -notin @('desktop-linux', 'default')) { throw "STOP: Docker context is '$context'" }
    if ($engine.Split('|')[3] -ne 'linux' -or -not $engine.Split('|')[0]) { throw "STOP: engine is not a Linux engine with an ID: $engine" }

    # 2. Source: each repository's commit, branch and cleanliness; 'origin' must exist (URL never read).
    $Repos = @('northstar-core', 'northstar-application', 'northstar-infrastructure', 'northstar-api', 'northstar-web', 'northstar-docs')
    $source = foreach ($repo in $Repos) {
        $path = Join-Path $Root $repo
        $head = Invoke-Native git @('-C', $path, 'rev-parse', 'HEAD') -AllowFailure
        if ($head.Code -ne 0) { "$repo|NOT-A-GIT-WORK-TREE"; continue }
        $branch = [string] (Invoke-Native git @('-C', $path, 'rev-parse', '--abbrev-ref', 'HEAD')).Output[0]
        $dirty = (Invoke-Native git @('-C', $path, 'status', '--porcelain')).Output.Count
        $origin = (Invoke-Native git @('-C', $path, 'remote', 'get-url', 'origin') -AllowFailure).Code -eq 0
        "$repo|$($head.Output[0])|$branch|uncommitted=$dirty|origin=$origin"
    }
    Save 'source-checkout.txt' $source
    foreach ($line in $source | Where-Object { $_ -notlike 'northstar-docs|*' }) {
        if ($line -like '*NOT-A-GIT-WORK-TREE*' -or $line -notlike '*|uncommitted=0|origin=True') { throw "STOP: runtime checkout not clean, not Git or without origin: $line" }
    }

    # 3. Images, service containers and the production volume's metadata (never mounted here).
    $apiImage = Get-ImageId 'northstar-api:india'
    $webImage = Get-ImageId 'northstar-web:india'
    Save 'images.txt' @("northstar-api:india|$apiImage", "northstar-web:india|$webImage")
    if ($apiImage -eq 'ABSENT') { throw 'STOP: northstar-api:india is not present' }
    $volume = [string] (Invoke-Native docker @('volume', 'inspect', 'northstar-india-data', '--format', '{{.Name}}|{{.Driver}}|{{.CreatedAt}}|{{json .Labels}}')).Output[0]
    Save 'volume.txt' @($volume)
    if ($volume -notlike 'northstar-india-data|local|*' -or $volume -notlike '*com.docker.compose.project*northstar-india*') { throw "STOP: unexpected volume identity: $volume" }
    $running = foreach ($service in @('india-api', 'india-web')) {
        foreach ($id in (Invoke-Native docker @('compose', 'ps', '-q', $service)).Output) {
            if ($id) { "$service|$id|$((Invoke-Native docker @('inspect', '--format', '{{.Image}}', $id)).Output[0])" }
        }
    }
    Save 'services-running.txt' @($running)
    foreach ($line in @($running)) {
        $service, $id, $image = $line.Split('|')
        $expected = if ($service -eq 'india-api') { $apiImage } else { $webImage }
        if ($image -ne $expected) { throw "STOP: $service runs $image, not the tag's $expected; rollback target ambiguous" }
    }

    # 4. Writers and scheduler: nothing but the india-api service may mount the production volume.
    $mounting = @((Invoke-Native docker @('ps', '-q', '--filter', 'volume=northstar-india-data')).Output | Where-Object { $_ })
    $apiIds = @((Invoke-Native docker @('compose', 'ps', '-q', 'india-api')).Output | Where-Object { $_ })
    $writers = @($mounting | Where-Object { $apiIds -notcontains $_ })
    $wrappers = @(Get-CimInstance Win32_Process -Filter "Name = 'powershell.exe'" | Where-Object { $_.CommandLine -like '*Invoke-NorthstarIndiaOperations.ps1*' })
    $task = Get-ScheduledTask -TaskName $TaskName
    $info = Get-ScheduledTaskInfo -TaskName $TaskName
    Save 'scheduler.txt' @("state=$($task.State)", "lastRunTime=$($info.LastRunTime)", "lastTaskResult=$($info.LastTaskResult)", "nextRunTime=$($info.NextRunTime)")
    Export-ScheduledTask -TaskName $TaskName | Out-File -LiteralPath (Join-Path $Release 'scheduler-definition.xml') -Encoding utf8
    $lastRun = Join-Path $env:LOCALAPPDATA 'Northstar\india-operations\last-run.json'
    if (Test-Path -LiteralPath $lastRun) { Copy-Item -LiteralPath $lastRun -Destination (Join-Path $Release 'last-run.before.json') }

    # 5. Configuration identity without secrets.
    $files = @('compose.yaml', 'Caddyfile', '.env', 'windows\Invoke-NorthstarIndiaOperations.ps1',
        'windows\Register-NorthstarIndiaOperationsTask.ps1', 'windows\northstar-india-operations.task.xml',
        '..\Dockerfile.api', '..\Dockerfile.web')
    Save 'file-hashes.txt' @(foreach ($file in $files) {
        "$file|$((Get-FileHash -LiteralPath (Join-Path $Deploy $file) -Algorithm SHA256).Hash.ToLowerInvariant())"
    })
    Save 'settings.txt' @(Select-String -LiteralPath (Join-Path $Deploy '.env') -Pattern '^\s*NORTHSTAR_(FUTURES_EXPIRATION|PORTFOLIO|STRATEGY|FUTURES_DAILY_BAR_FINALITY|FUTURES_FINAL_THROUGH|INDIA_WEB_PORT)\s*=' | ForEach-Object { $_.Line })
    $model = (Invoke-Native docker @('compose', 'config')).Output -join "`n"
    $digest = [System.Security.Cryptography.SHA256]::Create().ComputeHash([System.Text.Encoding]::UTF8.GetBytes($model))
    Save 'compose-config.sha256.txt' @((-join ($digest | ForEach-Object { $_.ToString('x2') })))
    Remove-Variable model

    # 6. The wrapper's own preflight with the current image (runs nothing, keeps last-run.json).
    & powershell.exe -NoProfile -ExecutionPolicy Bypass -File (Join-Path $Deploy 'windows\Invoke-NorthstarIndiaOperations.ps1') -PreflightOnly
    $preflight = $LASTEXITCODE
    Save 'wrapper-preflight.txt' @("exit=$preflight")

    # 7. Manifest of this record, then the remaining STOP checks.
    Get-ChildItem -LiteralPath $Release -File | Get-FileHash -Algorithm SHA256 |
        ForEach-Object { "$($_.Hash.ToLowerInvariant())  $(Split-Path -Leaf $_.Path)" } |
        Out-File -LiteralPath (Join-Path $Release 'MANIFEST.sha256') -Encoding ascii
    if ($preflight -ne 0) { throw "STOP: wrapper preflight exited $preflight" }
    if ($writers.Count -gt 0) { throw "STOP: $($writers.Count) container(s) other than india-api mount the production volume" }
    if ($wrappers.Count -gt 0) { throw 'STOP: a wrapper process is running' }
    "Preflight recorded. Stamp: $Stamp"
}
```

**Then check by eye.** `settings.txt` must show the expected contract,
portfolio and finality, with finality last recorded through 2026-10-08. This
release must leave those settings unchanged.

**If the old image cannot be linked to a release.** This is expected, because
nothing records the deployed commit (E-4).
- Record it as **"previous release: unidentified source"** with its image ID.
- `source-checkout.txt` describes only what is checked out now, not what the
  image was built from.
- Rollback then restores that **image ID** and the **preserved deployment
  files**, not a named source version.

**Optional, needs AP-8 (class R):** the portfolio before the upgrade. This is
the existing read-only report (runbook section 4E). It mounts the volume but
takes no lock and writes nothing. Its output contains Upstox-derived values
and stays in the release record on the deployment machine (G-1).

```powershell
. {
    $Stamp = 'PASTE-STAMP-FROM-BLOCK-5'
    $Deploy = 'C:\Local Disk(E)\Codes\Northstar\northstar-api\deploy\india'
    $Release = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\releases' $Stamp
    if (-not (Test-Path -LiteralPath (Join-Path $Release 'MANIFEST.sha256'))) { throw 'STOP: run block 5 first' }
    Set-Location -LiteralPath $Deploy
    docker compose run --rm --no-deps india-operations northstar paper status --database /data/northstar.sqlite3 --strategy directional-mvp-v1-nifty-paper-ops --portfolio nifty-paper-ops-oct26 --as-of 2026-10-27T18:30:00+00:00 |
        Out-File -LiteralPath (Join-Path $Release 'paper-status.before.txt') -Encoding utf8
    "paper status exit $LASTEXITCODE"
}
```

## 6. Quiescence and scheduler controls (class C, needs AP-4)

The sequence is:
1. Record the scheduler definition and state (block 5).
2. Prevent new scheduled invocations (`Disable-ScheduledTask`; an active run
   finishes, and no new one starts: runbook section 18C).
3. Establish that any active writer has completed: no wrapper process, and
   `last-run.json` not `RUNNING`.
4. Verify that no lock holder remains: no container other than the
   `india-api` service mounts `northstar-india-data`. Every writer, scheduled
   or a manual `docker compose run` whatever its name, must mount the
   volume.
5. Only then change the checkout or the images (section 7).

The check must succeed **twice in a row**, 30 seconds apart. **On any
failure the task stays disabled.** It is re-enabled only under AP-7
(section 7.8).

```powershell
. {
    # Block 6 (class C, AP-4): quiesce. On any STOP the task stays Disabled.
    $ErrorActionPreference = 'Stop'
    $Deploy = 'C:\Local Disk(E)\Codes\Northstar\northstar-api\deploy\india'
    $TaskName = 'Northstar India Operations'
    $LastRun = Join-Path $env:LOCALAPPDATA 'Northstar\india-operations\last-run.json'
    function Invoke-Native([string] $Exe, [string[]] $Arguments) {
        $ErrorActionPreference = 'Continue'
        $output = & $Exe @Arguments 2>$null
        if ($LASTEXITCODE -ne 0) { throw "STOP: $Exe $($Arguments[0]) $($Arguments[1]) exited $LASTEXITCODE" }
        @($output)
    }
    function Test-Quiet {
        $wrappers = @(Get-CimInstance Win32_Process -Filter "Name = 'powershell.exe'" | Where-Object { $_.CommandLine -like '*Invoke-NorthstarIndiaOperations.ps1*' })
        $mounting = @(Invoke-Native docker @('ps', '-q', '--filter', 'volume=northstar-india-data') | Where-Object { $_ })
        $apiIds = @(Invoke-Native docker @('compose', 'ps', '-q', 'india-api') | Where-Object { $_ })
        $writers = @($mounting | Where-Object { $apiIds -notcontains $_ })
        $outcome = ''
        if (Test-Path -LiteralPath $LastRun) { $outcome = [string] ([System.IO.File]::ReadAllText($LastRun) | ConvertFrom-Json).outcome }
        $state = [string] (Get-ScheduledTask -TaskName $TaskName).State
        ($state -eq 'Disabled') -and ($wrappers.Count -eq 0) -and ($writers.Count -eq 0) -and ($outcome -ne 'RUNNING')
    }
    Set-Location -LiteralPath $Deploy
    Disable-ScheduledTask -TaskName $TaskName | Out-Null
    $deadline = (Get-Date).AddMinutes(60)
    $quietTwice = 0
    while ($quietTwice -lt 2) {
        if (Test-Quiet) { $quietTwice++ } else { $quietTwice = 0 }
        if ($quietTwice -lt 2) {
            if ((Get-Date) -gt $deadline) { throw 'STOP: not quiet after 60 minutes; the task stays Disabled; investigate' }
            Start-Sleep -Seconds 30
        }
    }
    "Quiesced: task Disabled; no wrapper process; no writer container; last-run.json not RUNNING"
}
```

Never `Unregister-ScheduledTask`: the definition stays exactly as recorded.

## 7. Controlled upgrade sequence

Run 7.1 to 7.8 in order, in one window and **one PowerShell session**:
- 7.2 to 7.8 use the variables and functions of 7.1;
- 7.3, 7.4 and 7.7 re-check quiescence first;
- any STOP leaves the task disabled and leads to section 9.

### 7.1 Variables and checks for the remaining blocks

```powershell
. {
    $ErrorActionPreference = 'Stop'
    $Stamp = 'PASTE-STAMP-FROM-BLOCK-5'
    $Root = 'C:\Local Disk(E)\Codes\Northstar'
    $Deploy = Join-Path $Root 'northstar-api\deploy\india'
    $Release = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\releases' $Stamp
    $Secrets = Join-Path $env:LOCALAPPDATA "Northstar\release-secrets\$Stamp"
    $TaskName = 'Northstar India Operations'
    $LastRun = Join-Path $env:LOCALAPPDATA 'Northstar\india-operations\last-run.json'
    $Approved = [ordered]@{
        'northstar-core' = 'e54ccdff023db6a97db4a3debfdab4fb15e5b875'
        'northstar-application' = '9a0ab57288b440dec83d1680addfb12b8fc3e703'
        'northstar-infrastructure' = 'aa94428880de99f7c697577e5dfbe4987ffa9f0a'
        'northstar-api' = 'cfa7774b503b32c15263f8f2c0d905df0c1db622'
        'northstar-web' = '4f6908ca48b1ed82ea70787021c9bffcaf3ab49f'
    }
    function Invoke-Native([string] $Exe, [string[]] $Arguments, [switch] $AllowFailure) {
        $ErrorActionPreference = 'Continue'
        $output = & $Exe @Arguments 2>$null
        $code = $LASTEXITCODE
        if ($code -ne 0 -and -not $AllowFailure) { throw "STOP: $Exe $($Arguments[0]) $($Arguments[1]) exited $code" }
        [pscustomobject]@{ Code = $code; Output = @($output) }
    }
    function Assert-Quiet {
        $wrappers = @(Get-CimInstance Win32_Process -Filter "Name = 'powershell.exe'" | Where-Object { $_.CommandLine -like '*Invoke-NorthstarIndiaOperations.ps1*' })
        $mounting = @((Invoke-Native docker @('ps', '-q', '--filter', 'volume=northstar-india-data')).Output | Where-Object { $_ })
        $apiIds = @((Invoke-Native docker @('compose', 'ps', '-q', 'india-api')).Output | Where-Object { $_ })
        if ([string] (Get-ScheduledTask -TaskName $TaskName).State -ne 'Disabled') { throw 'STOP: the task is not Disabled; run block 6' }
        if ($wrappers.Count -gt 0) { throw 'STOP: a wrapper process is running' }
        if (@($mounting | Where-Object { $apiIds -notcontains $_ }).Count -gt 0) { throw 'STOP: a writer container mounts the production volume' }
    }
    if ($Stamp -notmatch '^\d{8}T\d{6}Z$') { throw 'STOP: set $Stamp to the value printed by block 5' }
    if (-not (Test-Path -LiteralPath (Join-Path $Release 'MANIFEST.sha256'))) { throw 'STOP: block 5 did not complete' }
    Set-Location -LiteralPath $Deploy
    Assert-Quiet
    # Services to replace (AP-5): by default exactly those block 5 found running.
    $Services = @([System.IO.File]::ReadAllLines((Join-Path $Release 'services-running.txt')) | Where-Object { $_ } | ForEach-Object { $_.Split('|')[0] } | Select-Object -Unique)
    $WebPortLine = @([System.IO.File]::ReadAllLines((Join-Path $Release 'settings.txt')) | Where-Object { $_ -match '^\s*NORTHSTAR_INDIA_WEB_PORT\s*=' })
    $WebPort = if ($WebPortLine.Count -gt 0) { ($WebPortLine[0] -split '=', 2)[1].Trim() } else { '8081' }
    "Services: $($Services -join ', '); web port: $WebPort"
}
```

### 7.2 Preserve the previous release (class C)

Preservation is by **tag on the existing image ID**: no image is removed or
rebuilt. The non-secret deployment files are copied into the release record.
`.env` is copied **outside every checkout and outside the evidence directory**,
into the operator's own profile, with the same access control as the
original (section 12).

```powershell
. {
    $Previous = Join-Path $Release 'previous'
    New-Item -ItemType Directory -Path (Join-Path $Previous 'windows') | Out-Null
    foreach ($image in @('northstar-api', 'northstar-web')) {
        if ((Invoke-Native docker @('image', 'inspect', "${image}:india", '--format', '{{.Id}}') -AllowFailure).Code -eq 0) {
            Invoke-Native docker @('tag', "${image}:india", "${image}:india-rollback-$Stamp") | Out-Null
        }
    }
    foreach ($file in @('compose.yaml', 'Caddyfile')) { Copy-Item -LiteralPath (Join-Path $Deploy $file) -Destination $Previous }
    Copy-Item -Path (Join-Path $Deploy 'windows\*') -Destination (Join-Path $Previous 'windows')
    New-Item -ItemType Directory -Path $Secrets | Out-Null
    $envCopy = Join-Path $Secrets '.env'
    Copy-Item -LiteralPath (Join-Path $Deploy '.env') -Destination $envCopy
    Set-Acl -LiteralPath $envCopy -AclObject (Get-Acl -LiteralPath (Join-Path $Deploy '.env'))
    if ((Get-FileHash -LiteralPath $envCopy -Algorithm SHA256).Hash -ne (Get-FileHash -LiteralPath (Join-Path $Deploy '.env') -Algorithm SHA256).Hash) { throw 'STOP: .env copy differs' }
    (Invoke-Native docker @('image', 'ls', '--format', '{{.Repository}}:{{.Tag}}|{{.ID}}')).Output | Where-Object { $_ -like 'northstar-*:india*' } |
        Out-File -LiteralPath (Join-Path $Release 'images.after-preserve.txt') -Encoding utf8
    "Previous release preserved; .env copy held under the operator profile only"
}
```

### 7.3 Bring the runtime checkouts to the approved commits (class C)

Only the five runtime repositories move; `northstar-docs` does not. This uses
ordinary Git with network access to `origin`. Fetch output can show the remote
URL: it goes to the console only and is never recorded.

```powershell
. {
    Assert-Quiet
    foreach ($repo in $Approved.Keys) {
        $path = Join-Path $Root $repo
        if ((Invoke-Native git @('-C', $path, 'status', '--porcelain')).Output.Count -ne 0) { throw "STOP: $repo has uncommitted or untracked files" }
        Invoke-Native git @('-C', $path, 'fetch', 'origin') | Out-Null
        Invoke-Native git @('-C', $path, 'checkout', '--detach', $Approved[$repo]) | Out-Null
        $head = [string] (Invoke-Native git @('-C', $path, 'rev-parse', 'HEAD')).Output[0]
        if ($head -ne $Approved[$repo]) { throw "STOP: $repo is at $head, not $($Approved[$repo])" }
    }
    # Every deployment file Compose and the task use must be exactly the approved commit's file.
    $api = Join-Path $Root 'northstar-api'
    foreach ($file in @('deploy/india/compose.yaml', 'deploy/india/Caddyfile', 'deploy/Dockerfile.api', 'deploy/Dockerfile.web',
            'deploy/Dockerfile.api.dockerignore', 'deploy/Dockerfile.web.dockerignore', 'deploy/india/windows/Invoke-NorthstarIndiaOperations.ps1',
            'deploy/india/windows/northstar-india-operations.task.xml', 'deploy/india/windows/Register-NorthstarIndiaOperationsTask.ps1')) {
        $blob = [string] (Invoke-Native git @('-C', $api, 'rev-parse', "HEAD:$file")).Output[0]
        $disk = [string] (Invoke-Native git @('-C', $api, 'hash-object', $file)).Output[0]
        if ($blob -ne $disk) { throw "STOP: $file differs from the approved commit" }
    }
    "Runtime checkouts at the approved commits"
}
```

### 7.4 Build the images (class C)

This is runbook section 4B's command, from the India deployment directory.
- Building replaces the `northstar-api:india` and `northstar-web:india` tags.
- Running containers keep their old image until 7.6.
- The Compose model hash is compared with block 5's: a difference means the
  previous deployment files were not what `develop` holds, and must be
  reviewed before continuing.

```powershell
. {
    Assert-Quiet
    Set-Location -LiteralPath $Deploy
    docker compose build
    if ($LASTEXITCODE -ne 0) { throw 'STOP: build failed; containers unchanged; rollback tags remain' }
    $built = foreach ($reference in @('northstar-api:india', 'northstar-web:india')) {
        "$reference|$((Invoke-Native docker @('image', 'inspect', $reference, '--format', '{{.Id}}')).Output[0])"
    }
    $built | Out-File -LiteralPath (Join-Path $Release 'images.built.txt') -Encoding utf8
    $model = (Invoke-Native docker @('compose', 'config')).Output -join "`n"
    $digest = -join ([System.Security.Cryptography.SHA256]::Create().ComputeHash([System.Text.Encoding]::UTF8.GetBytes($model)) | ForEach-Object { $_.ToString('x2') })
    Remove-Variable model
    $digest | Out-File -LiteralPath (Join-Path $Release 'compose-config.after.sha256.txt') -Encoding ascii
    if ($digest -ne [System.IO.File]::ReadAllText((Join-Path $Release 'compose-config.sha256.txt')).Trim()) { throw 'STOP: the resolved Compose model changed; review before continuing' }
    $built
}
```

### 7.5 Verify the release identity without touching data (class R)

`docker run --rm --pull never` starts a throwaway container from the local
image with **no** production volume; its anonymous `/data` volume is removed
with it. An unsupported flag also fails this check: STOP.

```powershell
. {
    $probe = Invoke-Native docker @('run', '--rm', '--pull', 'never', 'northstar-api:india', 'python', '-c', 'from northstar_api.cli import ExitCode; print(ExitCode.EXPIRY_EXCEPTION.value)') -AllowFailure
    if ($probe.Code -ne 0 -or [string] $probe.Output[-1] -ne '7') { throw "STOP: the built image does not carry EXPIRY_EXCEPTION = 7 (exit $($probe.Code))" }
    & powershell.exe -NoProfile -ExecutionPolicy Bypass -File (Join-Path $Deploy 'windows\Invoke-NorthstarIndiaOperations.ps1') -PreflightOnly
    if ($LASTEXITCODE -ne 0) { throw "STOP: wrapper preflight exited $LASTEXITCODE" }
    "wrapper|$((Get-FileHash -LiteralPath (Join-Path $Deploy 'windows\Invoke-NorthstarIndiaOperations.ps1') -Algorithm SHA256).Hash.ToLowerInvariant())" |
        Out-File -LiteralPath (Join-Path $Release 'wrapper.after.txt') -Encoding utf8
    "Release identity verified"
}
```

### 7.6 Replace the read-only services (class C, needs AP-5)

This replaces only `$Services`: by default those block 5 found running.
- `--no-build` uses the images from 7.4.
- Compose recreates a container whose image changed and keeps every volume.
- Containers are found through `docker compose ps -q`, never by an assumed
  name.

```powershell
. {
    if ($Services.Count -eq 0) { 'No service was running before: 7.6 skipped (starting one needs AP-5)'; return }
    Set-Location -LiteralPath $Deploy
    docker compose up -d --no-build @Services
    if ($LASTEXITCODE -ne 0) { throw 'STOP: the services did not start; go to section 10' }
    $builtApi = ([System.IO.File]::ReadAllLines((Join-Path $Release 'images.built.txt')) | Where-Object { $_ -like 'northstar-api:india|*' }).Split('|')[1]
    if ($Services -contains 'india-api') {
        $apiId = [string] (Invoke-Native docker @('compose', 'ps', '-q', 'india-api')).Output[0]
        if ((Invoke-Native docker @('inspect', '--format', '{{.Image}}', $apiId)).Output[0] -ne $builtApi) { throw 'STOP: india-api does not run the built image' }
        $deadline = (Get-Date).AddMinutes(5)
        do { Start-Sleep -Seconds 10; $health = [string] (Invoke-Native docker @('inspect', '--format', '{{.State.Health.Status}}', $apiId)).Output[0] }
        until ($health -eq 'healthy' -or (Get-Date) -gt $deadline)
        if ($health -ne 'healthy') { throw "STOP: india-api is '$health'; go to section 10" }
    }
    if ($Services -contains 'india-web') {
        try { Invoke-WebRequest -Uri "http://127.0.0.1:$WebPort/" -UseBasicParsing -TimeoutSec 10 | Out-Null; $web = 'no authentication challenge' }
        catch { $web = if ($_.Exception.Response) { [string] [int] $_.Exception.Response.StatusCode } else { 'no answer' } }
        if ($web -ne '401') { throw "STOP: india-web answered '$web', expected the basic-auth challenge 401" }
    }
    docker compose ps | Out-File -LiteralPath (Join-Path $Release 'compose-ps.after.txt') -Encoding utf8
    "Services replaced: $($Services -join ', ')"
}
```

### 7.7 Supervised run (class **W**, needs AP-6)

**This may write production SQLite.** It is runbook section 18C step 2, with
the task still disabled. WAITING is **not assumed**: eligibility is checked
from recorded facts first.

The run decides no new session only when all of the following hold:
- the previous run's outcome was `WAITING`;
- the finality mode and final-through date the wrapper recorded for it in
  `last-run.json` equal the current `.env` values;
- no writer has run since (quiescence).

Planning reads no clock and depends only on the persisted latest decision,
the finality setting and the calendar, so the plan is unchanged. The new
build adds one case: from the day after the configured expiration date (India
calendar), an exposed contract reports exit 7.

If any condition fails, this is a normal operational run that may acquire
provider data and write decisions, orders and fills. It is **outside this
procedure**: STOP and decide separately.

```powershell
. {
    $SupervisedRunAuthorizedBy = ''   # AP-6: the operator's name and time; empty means NOT authorized
    if (-not $SupervisedRunAuthorizedBy) { throw 'STOP: AP-6 not recorded; the supervised run is not authorized' }
    Assert-Quiet
    $before = [System.IO.File]::ReadAllText((Join-Path $Release 'last-run.before.json')) | ConvertFrom-Json
    $setting = @{}
    foreach ($line in [System.IO.File]::ReadAllLines((Join-Path $Release 'settings.txt'))) { $name, $value = $line -split '=', 2; $setting[$name.Trim()] = $value.Trim() }
    $now = @{}
    Select-String -LiteralPath (Join-Path $Deploy '.env') -Pattern '^\s*NORTHSTAR_FUTURES_(DAILY_BAR_FINALITY|FINAL_THROUGH)\s*=' | ForEach-Object { $name, $value = $_.Line -split '=', 2; $now[$name.Trim()] = $value.Trim() }
    if ([string] $before.outcome -ne 'WAITING') { throw "STOP: the previous outcome was '$($before.outcome)'; this run may process sessions" }
    if ($now['NORTHSTAR_FUTURES_FINAL_THROUGH'] -ne $setting['NORTHSTAR_FUTURES_FINAL_THROUGH'] -or [string] $before.finalThrough -ne $now['NORTHSTAR_FUTURES_FINAL_THROUGH']) { throw 'STOP: final-through changed since the previous run' }
    if ($now['NORTHSTAR_FUTURES_DAILY_BAR_FINALITY'] -ne $setting['NORTHSTAR_FUTURES_DAILY_BAR_FINALITY'] -or [string] $before.finalityMode -ne $now['NORTHSTAR_FUTURES_DAILY_BAR_FINALITY']) { throw 'STOP: finality mode changed since the previous run' }
    $indiaDate = [System.TimeZoneInfo]::ConvertTimeBySystemTimeZoneId([DateTime]::UtcNow, 'India Standard Time').Date
    $expiry = [DateTime]::ParseExact($setting['NORTHSTAR_FUTURES_EXPIRATION'], 'yyyy-MM-dd', [System.Globalization.CultureInfo]::InvariantCulture)
    Set-Location -LiteralPath (Join-Path $Deploy 'windows')
    powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\Invoke-NorthstarIndiaOperations.ps1
    $code = $LASTEXITCODE
    Copy-Item -LiteralPath $LastRun -Destination (Join-Path $Release 'last-run.after.json')
    $after = [System.IO.File]::ReadAllText($LastRun) | ConvertFrom-Json
    "exit $code; outcome $($after.outcome); India date $($indiaDate.ToString('yyyy-MM-dd')); expiration $($expiry.ToString('yyyy-MM-dd'))"
    if ($code -eq 0 -and $after.outcome -eq 'WAITING') { 'As expected: WAITING' }
    elseif ($code -eq 7 -and $indiaDate -gt $expiry) { 'EXPIRY_EXCEPTION after the expiration date: intended detection; follow the expiry procedure, do not roll back' }
    else { throw "STOP: unexpected outcome (exit $code, $($after.outcome)); see section 9" }
}
```

### 7.8 Re-establish the scheduler state (class C, needs AP-7)

Restore exactly the state recorded in `scheduler.txt`, and only on explicit
operator authorization:
- if it was `Ready` (enabled), run the command below;
- if it was `Disabled`, leave it disabled.

The same applies after a rollback.

```powershell
. {
    Enable-ScheduledTask -TaskName 'Northstar India Operations'
    Get-ScheduledTask -TaskName 'Northstar India Operations' | Select-Object TaskName, State
}
```

## 8. Post-upgrade verification

| Check | How | Class |
|-------|-----|-------|
| API startup and health | 7.6: container runs the built image; health `healthy` | R |
| Web/API schema compatibility | Open the dashboard (runbook section 8). The Operational Status card renders, with the expiry block labelled "Window assessed as of the latest decision", `Pending orders` shown, and no error | R |
| Release identity | 7.3 commits and deployment-file blobs; 7.4 image IDs and Compose model hash; 7.5 exit-7 probe | R |
| M1.2 consistency | Covered by the development tests (E-8). In production, only that dashboard reads during normal runs never error; never forced by a concurrent write | R (observation) |
| Normal WAITING | 7.7, only after its eligibility check | **W** (AP-6) |
| Normal flat ROLLOVER REQUIRED | Not reachable in production now; covered by V-3 and V-5 (E-10, E-11) | Synthetic only |
| Exit 7 through the wrapper | Synthetic and fake-Docker (E-11). On the deployment machine only an isolated acceptance project with a seeded synthetic exposure may trigger it; the existing harness covers WAITING only, so this is **not yet available**. Never create an exception in the production portfolio | I (not yet feasible) |
| Wrapper and real-Docker isolation with the new image | The opt-in harness `northstar-api/tests/test_india_windows_docker_acceptance.py` (as E-5) | I (needs operator authorization) |
| Paper trading state preserved | `paper status` after 7.7, compared with `paper-status.before.txt`: same orders, fills, pending and position after WAITING | R (AP-8; G-1 retention) |
| Scheduler behaviour | After 7.8: `Get-ScheduledTaskInfo` `LastTaskResult` and `last-run.json` after the next hourly run | R (observation) |

## 9. Rollback triggers

**Roll back (section 10) when:**
- 7.2 to 7.6 STOP after any tag, checkout or container changed;
- `india-api` is not running the built image or is not healthy within 5
  minutes;
- `india-web` does not answer 401 without credentials;
- the dashboard fails to render its operations data;
- the supervised run exits other than 0 (WAITING) or 7 after the expiration
  date: for example 1, or 4 or 5 where the previous build did not fail;
- `paper status` shows changed orders, fills or positions after a WAITING run.

**STOP and escalate, without rolling back, when:**
- 7.7's eligibility check fails, or AP-6 is missing: skip the run;
- exit 7 after the expiration date: follow the expiry procedure;
- exit 6 provider errors;
- exit 3 configuration errors that also occur with the previous build;
- any sign of a different volume, project or Docker context.

**Escalate before any further step when:**
- block 5 cannot establish the previous image or checkout;
- a rollback STOPs.

**Every branch leaves the task disabled.** Re-enabling is only ever AP-7.

## 10. Complete rollback procedure

A rollback restores:
- the **API and web image tags** (previous image IDs);
- the **runtime checkouts** (recorded commits and branches);
- the **deployment files** (wrapper, Compose file, Caddyfile; verified by
  hash);
- the **Compose model** (hash);
- **`.env`**, only if it changed;
- the **service topology**: the same services running, on the previous images;
- the **scheduler state**, only under AP-7.

Restoring container images never restores data. The SQLite database and every
volume are **never** touched: the release made no persistent-data change, and
anything a run wrote is valid for both builds.

```powershell
. {
    # Block 10 (class C): rollback. On any STOP the task stays Disabled.
    $ErrorActionPreference = 'Stop'
    $Stamp = 'PASTE-STAMP-FROM-BLOCK-5'
    $Root = 'C:\Local Disk(E)\Codes\Northstar'
    $Deploy = Join-Path $Root 'northstar-api\deploy\india'
    $Release = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\releases' $Stamp
    $Previous = Join-Path $Release 'previous'
    $Secrets = Join-Path $env:LOCALAPPDATA "Northstar\release-secrets\$Stamp"
    $TaskName = 'Northstar India Operations'
    function Invoke-Native([string] $Exe, [string[]] $Arguments, [switch] $AllowFailure) {
        $ErrorActionPreference = 'Continue'
        $output = & $Exe @Arguments 2>$null
        $code = $LASTEXITCODE
        if ($code -ne 0 -and -not $AllowFailure) { throw "STOP: $Exe $($Arguments[0]) $($Arguments[1]) exited $code; escalate" }
        [pscustomobject]@{ Code = $code; Output = @($output) }
    }
    function Get-ModelHash {
        $model = (Invoke-Native docker @('compose', 'config')).Output -join "`n"
        -join ([System.Security.Cryptography.SHA256]::Create().ComputeHash([System.Text.Encoding]::UTF8.GetBytes($model)) | ForEach-Object { $_.ToString('x2') })
    }
    if (-not (Test-Path -LiteralPath $Previous)) { throw 'STOP: no preserved previous release for this stamp; escalate' }
    Set-Location -LiteralPath $Deploy

    # 1. Quiesce, bounded: no new runs, no wrapper process, no writer container.
    Disable-ScheduledTask -TaskName $TaskName | Out-Null
    $deadline = (Get-Date).AddMinutes(60)
    do {
        $wrappers = @(Get-CimInstance Win32_Process -Filter "Name = 'powershell.exe'" | Where-Object { $_.CommandLine -like '*Invoke-NorthstarIndiaOperations.ps1*' })
        $mounting = @((Invoke-Native docker @('ps', '-q', '--filter', 'volume=northstar-india-data')).Output | Where-Object { $_ })
        $apiIds = @((Invoke-Native docker @('compose', 'ps', '-q', 'india-api')).Output | Where-Object { $_ })
        $busy = $wrappers.Count + @($mounting | Where-Object { $apiIds -notcontains $_ }).Count
        if ($busy -gt 0) {
            if ((Get-Date) -gt $deadline) { throw 'STOP: not quiet after 60 minutes; the task stays Disabled; escalate' }
            Start-Sleep -Seconds 30
        }
    } while ($busy -gt 0)

    # 2. Runtime checkouts: the recorded commit, and the recorded branch if there was one.
    foreach ($line in [System.IO.File]::ReadAllLines((Join-Path $Release 'source-checkout.txt'))) {
        $fields = $line.Split('|')
        if ($fields[0] -eq 'northstar-docs' -or $fields[1] -eq 'NOT-A-GIT-WORK-TREE') { continue }
        $path = Join-Path $Root $fields[0]
        if ($fields[2] -ne 'HEAD') { Invoke-Native git @('-C', $path, 'checkout', $fields[2]) | Out-Null }
        else { Invoke-Native git @('-C', $path, 'checkout', '--detach', $fields[1]) | Out-Null }
        if ([string] (Invoke-Native git @('-C', $path, 'rev-parse', 'HEAD')).Output[0] -ne $fields[1]) { throw "STOP: $($fields[0]) is not back at $($fields[1]); escalate" }
    }

    # 3. Deployment files: the preserved copies, verified against block 5's hashes.
    Copy-Item -LiteralPath (Join-Path $Previous 'compose.yaml') -Destination $Deploy
    Copy-Item -LiteralPath (Join-Path $Previous 'Caddyfile') -Destination $Deploy
    Copy-Item -Path (Join-Path $Previous 'windows\*') -Destination (Join-Path $Deploy 'windows')
    $expected = @{}
    foreach ($line in [System.IO.File]::ReadAllLines((Join-Path $Release 'file-hashes.txt'))) { $name, $hash = $line.Split('|'); $expected[$name] = $hash }
    foreach ($name in @('compose.yaml', 'Caddyfile', 'windows\Invoke-NorthstarIndiaOperations.ps1', 'windows\northstar-india-operations.task.xml', '..\Dockerfile.api', '..\Dockerfile.web')) {
        if ((Get-FileHash -LiteralPath (Join-Path $Deploy $name) -Algorithm SHA256).Hash.ToLowerInvariant() -ne $expected[$name]) { throw "STOP: $name does not match the pre-release hash; escalate" }
    }

    # 4. Configuration: restore .env from the profile copy only if it changed.
    if ((Get-FileHash -LiteralPath (Join-Path $Deploy '.env') -Algorithm SHA256).Hash.ToLowerInvariant() -ne $expected['.env']) {
        Copy-Item -LiteralPath (Join-Path $Secrets '.env') -Destination (Join-Path $Deploy '.env')
    }
    if ((Get-ModelHash) -ne [System.IO.File]::ReadAllText((Join-Path $Release 'compose-config.sha256.txt')).Trim()) { throw 'STOP: the Compose model is not the pre-release one; escalate' }

    # 5. Images: point the tags back at the preserved image IDs.
    foreach ($image in @('northstar-api', 'northstar-web')) {
        if ((Invoke-Native docker @('image', 'inspect', "${image}:india-rollback-$Stamp", '--format', '{{.Id}}') -AllowFailure).Code -eq 0) {
            Invoke-Native docker @('tag', "${image}:india-rollback-$Stamp", "${image}:india") | Out-Null
        }
    }

    # 6. Topology: the services running before run again on their previous image IDs;
    #    a service started by the release that was not running before is stopped (volumes kept).
    $before = @{}
    foreach ($line in [System.IO.File]::ReadAllLines((Join-Path $Release 'services-running.txt'))) { if ($line) { $service, $id, $image = $line.Split('|'); $before[$service] = $image } }
    foreach ($service in @('india-api', 'india-web')) {
        $runningNow = @((Invoke-Native docker @('compose', 'ps', '-q', $service)).Output | Where-Object { $_ })
        if ($before.ContainsKey($service)) {
            docker compose up -d --no-build $service
            if ($LASTEXITCODE -ne 0) { throw "STOP: $service did not start on the previous image; escalate" }
            $id = [string] (Invoke-Native docker @('compose', 'ps', '-q', $service)).Output[0]
            if ((Invoke-Native docker @('inspect', '--format', '{{.Image}}', $id)).Output[0] -ne $before[$service]) { throw "STOP: $service is not on its previous image; escalate" }
        }
        elseif ($runningNow.Count -gt 0) {
            docker compose stop $service
            if ($LASTEXITCODE -ne 0) { throw "STOP: could not stop $service; escalate" }
        }
    }

    # 7. The previous wrapper's own preflight, then the record.
    & powershell.exe -NoProfile -ExecutionPolicy Bypass -File (Join-Path $Deploy 'windows\Invoke-NorthstarIndiaOperations.ps1') -PreflightOnly
    "wrapper preflight exit $LASTEXITCODE"
    (Invoke-Native docker @('image', 'ls', '--format', '{{.Repository}}:{{.Tag}}|{{.ID}}')).Output | Where-Object { $_ -like 'northstar-*:india*' } |
        Out-File -LiteralPath (Join-Path $Release 'images.after-rollback.txt') -Encoding utf8
    "Rolled back. The task stays Disabled until AP-7."
}
```

**After a rollback:**
- If `india-api` was running before, check its health as in 7.6.
- One supervised run (7.7) only under a new AP-6.
- Restore the recorded scheduler state only under AP-7.
- If `northstar-web:india` was `ABSENT` in block 5, the tag still names the
  new web image; no container uses it unless `india-web` was running before.

**Never:**
- `docker compose down` (with or without volumes);
- any volume removal or prune;
- any image removal before the release is accepted;
- any database copy, restore or recreation.

## 11. Recovery and SQLite boundaries

- The release and the rollback mount nothing new on `northstar-india-data`.
  Production is mounted only by:
  - the existing `india-api` service;
  - the optional `paper status` report (class R);
  - the supervised run (class W).
- No command copies, restores, recreates or edits the database. The data
  volume and the Caddy volumes are never removed.
- **No persistent-data change:** this release adds no table, column or
  migration to the futures store (section 3). Had it done so, it would be a
  blocker requiring its own approved procedure.
- A database recovery is never part of a code rollback. It needs the drill
  document's isolated procedure and, for production, a separately approved
  procedure.

## 12. Evidence and secrets

**Release record.**
- Location: `C:\Local Disk(E)\Codes\NorthstarEvidence\releases\<stamp>`, on
  the deployment machine only.
- Contents:
  - block 5's files and `MANIFEST.sha256`;
  - `previous\` (non-secret deployment files);
  - `images.after-preserve.txt`, `images.built.txt` and
    `compose-config.after.sha256.txt`;
  - `wrapper.after.txt`, `compose-ps.after.txt` and `last-run.after.json`;
  - `paper-status.before.txt` and `paper-status.after.txt` (AP-8);
  - `images.after-rollback.txt`.

**Secrets and sensitive items:**

| Item | Handling |
|------|----------|
| `.env` copy | Held only in `%LOCALAPPDATA%\Northstar\release-secrets\<stamp>\.env`: the operator's own profile, outside every Git checkout and outside the evidence directory, with the original's access control applied. **Never** beside `.env` in the checkout: Git ignores only `.env`, so a copy there would be untracked, would make 7.3 STOP, and could be committed. The Linux runbook's `.env.previous` must not be used on Windows |
| `.env` copy cleanup | After the release is accepted, or the rollback is complete and accepted, the operator deletes that one file and its `<stamp>` folder by exact path, never by wildcard, and records that it was done |
| Resolved Compose model | Contains the token. Only its SHA-256 is stored; it is never written, printed or logged |
| Wrapper logs and `last-run.json` | The wrapper never logs secrets. `last-run.json` holds outcome, finality mode and final-through only. Run logs may contain Upstox-derived values (G-1) |
| Scheduler definition | `scheduler-definition.xml` names the Windows account (no password: the task uses an interactive token). It stays on the deployment machine |
| Git remote URLs | Never read or stored. Block 5 only checks that `origin` exists. Fetch output may print the URL to the console; never transcribe it |
| Hashes | File and image hashes, including `.env`'s SHA-256, reveal no secret and may be recorded |
| Host identity | `host.txt` names the computer and account; it stays on the deployment machine |

**Record in the evidence register:** the approvals AP-1 to AP-8; the release
commits, the documentation commit, image IDs and wrapper hash; each check in
section 8; and any STOP. V-3 changes only through the specification's own
process.

**Static validation of this document** (development machine, 2026-10-11):
- every PowerShell block parses under Windows PowerShell 5.1;
- the repository's destructive-command rule and a stricter list pass: no
  `compose down`, volume removal or prune, `docker rm` or `rmi`, `sqlite3`,
  finality change or task registration;
- `.env` is copied only to the profile folder and restored only by the
  guarded rollback;
- the Compose model is only hashed;
- cited files, commits and symbols exist.

## 13. G-1 and acceptance limitations

- **Does the release create new retained Upstox-derived data?** No. A build
  pulls base images and packages, never provider data, and the release changes
  no data handling. But it **continues** a persistent operation that acquires
  and stores Upstox candles, and the runbook (section 0) says G-1 must be
  cleared for that. Technical access is not permission. Whether continued
  production acquisition is permissible is **undetermined** here: it is the
  gate owner's decision (AP-3).
- **No production backup.** This procedure takes none and permits none. Drill
  block B and everything after it remain blocked by G-1.
- **Still blocked by G-1:**
  - D-4;
  - drill blocks B to J (B-1, B-2);
  - P-3;
  - retaining production outputs as acceptance evidence.
- **Drill blocks A and P** passed against the **earlier** image identity
  (operator-reported). That evidence stays as **historical** evidence of the
  procedure, but it does not carry over to the new build. Block B compares
  `imageId`, `engineId`, `productionVolume` and `productionVolumeCreatedAt`
  with block A's record and STOPs on any change ("prepare a new drill"). After
  this release, **block A's preflight and block P's disposable-volume lock
  test must be repeated** for the new image before block B.
  - Neither needs G-1: A reads metadata, and P uses disposable volumes.
  - Block B and everything after it do.
- **Acceptance:** a successful release would allow V-3 to be assessed, and
  would provide the deployed half of X-4. X-4 stays BLOCKED (approvals, the
  real-scheduler exit-7 check, D-EXP-6 and D-EXP-7). G-1 stays unresolved,
  B-1, B-2 and P-3 stay PENDING, and M1.4.3.3 stays PARTIALLY VERIFIED.
- **October timing:** deploying before 2026-10-28 (IST) means S-2 reports
  exit 7 on the first exposed run after the 2026-10-27 expiry. Without it,
  production keeps reporting WAITING with exit 0. The release must not be
  combined with any finality approval.

## 14. STOP conditions

Stop, change nothing further, leave the task disabled and escalate when:
- any approval in section 4 is missing;
- the Docker context, engine, project `northstar-india` or volume
  `northstar-india-data` differs from block 5's checks;
- a runtime repository is not a clean Git work tree with `origin`, or a
  commit or deployment file differs from section 3;
- a running service uses an image other than its tag's;
- the previous image cannot be preserved by tag;
- the system is not quiet (a wrapper process or writer container) within 60
  minutes;
- the build fails, the Compose model changes, or the image lacks exit 7;
- `.env` or `settings.txt` would change: a release never changes finality or
  configuration;
- 7.7's eligibility check fails;
- any step would run `compose down`, remove a volume or image, prune, copy or
  recreate the database, or register or unregister the task;
- any step would approve a session or change `NORTHSTAR_FUTURES_FINAL_THROUGH`;
- an exception would have to be created in the production portfolio to test
  exit 7;
- the `.env` copy would be placed inside a checkout or the evidence directory.

## 15. Outstanding decisions and readiness verdict

**Decisions:**
- AP-1 to AP-8 (section 4), all PENDING, especially AP-1 (scope, including
  the options code, and the documentation commit) and AP-3 (operation under
  G-1);
- whether an isolated exit-7 acceptance scenario should be built for the
  deployment machine (section 8);
- whether to add source labels to the images in future, so the image itself
  records its commits.

**Assumptions checked only at run time on the deployment machine:**
- Docker context and engine;
- clean runtime checkouts with `origin`;
- the volume's identity;
- `docker compose ps -q` and the `volume=` filter;
- support for `--pull never` and `--no-build`;
- the web port;
- the `India Standard Time` time-zone ID.

Each has a STOP check in its block.

**Release blockers today:**
- missing approvals;
- CI or full suites not recorded at the release commits;
- the documentation commit not yet created;
- the previous build's source is unidentified (expected: preserve it by image
  ID).

**Verdict: READY FOR OPERATOR REVIEW.** The procedure is complete for review.
It is not approved, not production-validated, and authorizes no release.
