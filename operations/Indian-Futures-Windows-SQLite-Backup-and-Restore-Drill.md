# Indian Futures: Windows SQLite Backup and Isolated Restore Drill

Status: **written, statically validated, never run** (2026-10-10). Nothing
here has been run against production. The first supervised run is milestone
M1.4.2.

This runbook takes a consistent online backup of the production paper-trading
SQLite database on the **Windows deployment machine with Docker Desktop**,
verifies it, restores it into a **new disposable Docker volume**, and compares
the restored copy with the backup. It never restores into production. A real
production restore is not covered here; it needs its own approved procedure.

Related: [Indian Futures Deployment Runbook](Indian-Futures-Deployment-Runbook.md)
(sections 14 and 18), [Paper MVP Acceptance Specification](Indian-Futures-Paper-MVP-Acceptance.md)
(B-1, B-2), [Paper MVP Evidence Register](Indian-Futures-Paper-MVP-Evidence-Register.md).

## 1. Persistence facts this procedure relies on

Verified in the repositories (paths relative to `northstar-api`):

| Fact | Source |
|------|--------|
| The database is `/data/northstar.sqlite3` inside the containers | `deploy/india/compose.yaml` (`NORTHSTAR_DATABASE: ${NORTHSTAR_DATABASE:-/data/northstar.sqlite3}`); `deploy/india/.env.example` sets the same value |
| `/data` is the named volume `northstar-india-data`, mounted read-write into exactly two services, `india-api` and `india-operations` | `deploy/india/compose.yaml` (`- northstar-india-data:/data`, twice) |
| The volume name is pinned with `name: northstar-india-data`, so **any** Compose project name resolves to the production volume | `deploy/india/compose.yaml` (`volumes:` section) |
| SQLite runs in its default rollback-journal mode (`delete`); no source sets a `PRAGMA` | no `PRAGMA` in `northstar-api/src` or `northstar-infrastructure/src`; `northstar-api/tests/test_futures_daily_operation.py` asserts `journal_mode == delete` |
| The only database writer is serialised by `DatabaseOperationsLock`: an exclusive, non-blocking `flock` on `/data/northstar.sqlite3.operations.lock`, held for a whole operation; a held lock raises `OperationsAlreadyActiveError` | `src/northstar_api/operations_lock.py` |
| The scheduled task runs `docker compose run --rm -T --no-deps india-operations` hourly; a run that finds the lock held reports `SKIPPED` and exits 0 | `deploy/india/windows/Invoke-NorthstarIndiaOperations.ps1`; runbook section 18 |
| The image `northstar-api:india` runs as user `northstar` (uid 10001), has no `ENTRYPOINT`, and only `/data` exists for it: `mkdir /data && chown northstar:northstar /data`, `VOLUME ["/data"]` | `deploy/Dockerfile.api` |
| Futures facts live in `futures_forward_research_records` (decisions), `futures_paper_orders`, `futures_paper_fills` (fills reference orders), `futures_ohlcv` (daily bars) and `futures_contract_economics`; positions are derived from fills, not stored | `northstar-infrastructure/src/northstar_infrastructure/persistence/*_schema.py`, `.../market_data/sqlite_futures_schema.py` |
| No backup or restore script exists in the repositories; runbook section 14 is written for the Linux/systemd host | `deploy/` tree; runbook section 14 |

Because uid 10001 can write only where the image's `/data` ownership applies,
every **disposable** volume in this procedure is mounted at `/data`. On its
first mount Docker copies the image directory's ownership into a new, empty
named volume. Mounting a new volume anywhere else would leave it owned by root,
and the scripts could not write to it. That is why a drill container's `/data`
is never assumed to be production: production is identified only by the
recorded volume identity and by the markers described in section 2.

Operator-provided for the Windows deployment machine (not inspectable from the
repositories; the preflight checks them):

| Item | Value |
|------|-------|
| Production `deploy\india` directory | `C:\Local Disk(E)\Codes\Northstar\northstar-api\deploy\india` |
| Scheduled task | `Northstar India Operations` (enabled, hourly) |
| Existing evidence root | `C:\Local Disk(E)\Codes\NorthstarEvidence` (historical audit output in `persistence-audit`) |
| New drill directories (this procedure) | `C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills\<stamp>` |

## 2. Design and safety boundary

- **Docker Compose is not used.** Every container is started with
  `docker run` and explicit `--mount` options, so no `.env`, Compose
  interpolation or project name is involved. Changing a Compose project name
  would not isolate anything: the volume name is pinned.
- **Every Docker call is checked.** Arguments are built as PowerShell arrays,
  so there are no line continuations. Every call goes through a helper that
  stops on a non-zero exit, or its exit code is checked explicitly. A volume is
  treated as absent only when a successful `docker volume ls` does not list its
  exact name; an inspect error is never read as absence.
- **One preflight, run twice.** Block A runs the production preflight, records
  its result (engine identity, production volume and its `CreatedAt`, image ID)
  and saves the preflight script into the drill. Block B runs the same script
  again immediately before mounting production and stops if anything differs
  from the record. Every drill script's SHA-256 is recorded in block A and
  checked before the script is used.
- **Locking is proven before production is touched.** Block P runs only on a
  disposable volume. It shows that a read-only descriptor takes the exclusive
  `flock`, that it conflicts with Northstar's own `DatabaseOperationsLock` in
  another container in both directions, that a second non-blocking acquisition
  fails, and that the lock is released when its process exits. Its success
  record is written only after every check passed and its own containers and
  volume are gone. Block B refuses to run unless that record is valid, complete
  and for the same drill, and nothing of block P remains.
- **Production is touched once, read-only.** Only block B mounts
  `northstar-india-data`, and only with `readonly`. The backup script refuses
  to continue if the database is writable at that mount. It holds the
  operations lock through a read-only descriptor for the duration of the copy.
  If the hourly task starts meanwhile, that run reports `SKIPPED` and exits 0;
  the next hourly run proceeds normally. The task is **not** disabled.
- **The backup uses SQLite's backup API** (`sqlite3.Connection.backup`, every
  page in one step) from a read-only connection, into the container's own
  `/tmp`. The finished, closed file is then copied byte for byte into the drill
  directory, so SQLite never writes through the Windows bind mount.
- **Disposable volumes are new, labelled, recorded and marked.**
  `sqlite-restore-drill-<stamp>` and `sqlite-lock-test-<stamp>` are created by
  this procedure, labelled `northstar.purpose` and `northstar.drill=<stamp>`,
  and their creation record (name, labels, `CreatedAt`) is written to the
  drill's `results` directory. Each is re-verified against that record before
  every mount and before removal. The scripts refuse a volume that is not empty
  when they first claim it, write a marker into it, and later refuse any `/data`
  without this drill's marker or with a database where none should be.
- **No container has a network** (`--network none`), receives a token, or
  pulls an image (`--pull never`). No provider request or trading operation
  happens.
- **Cleanup removes only what this drill demonstrably created**: the two
  disposable volumes, each only with a matching creation record and labels and
  no container using it. Backups, manifests, results and failure logs are kept;
  retention is manual. Block I's `docker volume rm` is the only removal in this
  runbook. Runbook section 16's rule still holds: no Northstar stack volume is
  ever removed or written here, and no prune or `compose down` is used.

## 3. STOP conditions

Every block throws `STOP: ...` and changes nothing further when one of these
holds. Fix the cause or abandon the drill; never work around a STOP.

| Where | STOP when |
|-------|-----------|
| A, B (preflight) | `compose.yaml` or `.env` is missing; `compose.yaml` does not pin `name: northstar-india-data`, does not mount it at `/data` in exactly two services, or its database default is not `/data/northstar.sqlite3`; `.env` overrides `NORTHSTAR_DATABASE` with another path |
| A, B (preflight) | Docker is not on context `desktop-linux` or `default`, is not Docker Desktop, does not run Linux containers, or reports no engine ID |
| A, B (preflight) | `northstar-india-data` is not listed, is not the `local` driver, or is not labelled with Compose project `northstar-india` |
| A, B (preflight) | The scheduled task is missing or running, or a `northstar-india-operations-*` container is running |
| A, B (preflight) | `northstar-api:india` is not available locally |
| A | The drill directory already exists, or would be inside the Northstar checkout |
| B | Engine ID, production volume, its `CreatedAt` or the image ID differ from block A's record; `lock-test-passed.json` is missing, invalid, for another stamp, not `passed`, or lacks a check; the lock-test volume or any container of this drill still exists; a drill script's SHA-256 differs from block A's record |
| B | The database is writable at the read-only mount, the lock file is missing, the lock is held, or a rollback journal (`northstar.sqlite3-journal`) is present |
| P | The lock test already passed, or the lock-test volume or a probe or holder container name already exists; the volume is not empty at its first mount; any lock expectation fails; a holder exits before `LOCKED`, does not report it within 60 s, stops before its competing probes finish, does not exit within 90 s, or exits non-zero; a lock-test container remains |
| B, E, F | `PRAGMA integrity_check` is not `ok`, or `PRAGMA foreign_key_check` reports a violation |
| D, E, F | A SHA-256 does not match the manifest |
| E | A database is visible at `/data` in a container that mounts no volume there |
| F | The restore volume already exists, or is not empty at its first mount |
| F, H, H2, I, P | A disposable volume's name, labels or `CreatedAt` differ from its creation record, or its marker is missing |
| H | The backup and the restored copy differ |
| I | A container of this drill other than a label-verified lock-test holder still exists, or a container still uses a volume to be removed |
| Any | A Docker command exits non-zero |

## 4. Procedure

Run each block in a **new** Windows PowerShell 5.1 window on the deployment
machine, as the Docker Desktop user. Blocks share nothing but the drill stamp:
paste the stamp printed by block A into `$Stamp` at the top of every later
block. Run the blocks in order: A, P, B, D, E, F, H, (H2), I, then J.

### A. Preflight and prepare the drill

Runs the production preflight first and **creates nothing unless every check
passes**. Then it creates the new drill directory and writes into it: the
preflight script, five Python scripts, and `drill.json`, which records the
preflight identity and every script's SHA-256. Note the printed stamp.

```powershell
$ErrorActionPreference = 'Stop'
$Deploy = 'C:\Local Disk(E)\Codes\Northstar\northstar-api\deploy\india'
$EvidenceRoot = 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills'
$Checkout = 'C:\Local Disk(E)\Codes\Northstar'
$Utf8 = New-Object System.Text.UTF8Encoding($false)

$preflightPs1 = @'
param([Parameter(Mandatory = $true)] [string] $Deploy)
# Read-only production preflight. Changes nothing; throws STOP on any doubt.
$ErrorActionPreference = 'Stop'
function Invoke-Docker([string[]] $Arguments) {
    $output = & docker @Arguments
    if ($LASTEXITCODE -ne 0) { throw "STOP: docker $($Arguments[0]) $($Arguments[1]) failed with exit code $LASTEXITCODE" }
    $output
}

$composePath = Join-Path $Deploy 'compose.yaml'
$envPath = Join-Path $Deploy '.env'
foreach ($path in @($composePath, $envPath)) {
    if (-not (Test-Path -LiteralPath $path -PathType Leaf)) { throw "STOP: $path not found" }
}
$compose = [System.IO.File]::ReadAllText($composePath)
if ($compose -notmatch '(?m)^\s+name:\s+northstar-india-data\s*$') { throw 'STOP: compose.yaml does not pin the volume name northstar-india-data' }
if ([regex]::Matches($compose, '(?m)^\s+-\s+northstar-india-data:/data\s*$').Count -ne 2) { throw 'STOP: compose.yaml does not mount northstar-india-data at /data in exactly two services' }
if ($compose -notmatch '(?m)^\s+NORTHSTAR_DATABASE:\s+\$\{NORTHSTAR_DATABASE:-/data/northstar\.sqlite3\}\s*$') { throw 'STOP: the compose.yaml database default is not /data/northstar.sqlite3' }
# Only the NORTHSTAR_DATABASE line of .env is read; nothing is printed from it.
$override = Select-String -LiteralPath $envPath -Pattern '^\s*NORTHSTAR_DATABASE\s*=' | Select-Object -Last 1
if ($override -and ($override.Line -split '=', 2)[1].Trim().Trim('"').Trim("'") -cne '/data/northstar.sqlite3') {
    throw 'STOP: .env overrides NORTHSTAR_DATABASE with another path'
}

$context = (@(Invoke-Docker @('context', 'show')) -join '').Trim()
if (@('desktop-linux', 'default') -cnotcontains $context) { throw "STOP: Docker context is '$context', not the local Docker Desktop engine" }
$info = (@(Invoke-Docker @('info', '--format', '{{json .}}')) -join "`n") | ConvertFrom-Json
if ($info.OSType -ne 'linux') { throw 'STOP: the engine is not running Linux containers' }
if ([string] $info.OperatingSystem -notlike '*Docker Desktop*') { throw "STOP: the engine is not Docker Desktop ($($info.OperatingSystem))" }
if (-not $info.ID) { throw 'STOP: the engine reported no identity' }

if (@(Invoke-Docker @('volume', 'ls', '--format', '{{.Name}}')) -cnotcontains 'northstar-india-data') { throw 'STOP: the production volume northstar-india-data is not listed' }
$volume = ((@(Invoke-Docker @('volume', 'inspect', 'northstar-india-data')) -join "`n") | ConvertFrom-Json)[0]
if ($volume.Name -cne 'northstar-india-data' -or $volume.Driver -ne 'local' -or $volume.Labels.'com.docker.compose.project' -cne 'northstar-india') {
    throw 'STOP: northstar-india-data is not the northstar-india Compose volume'
}

try { $task = Get-ScheduledTask -TaskName 'Northstar India Operations' } catch { throw 'STOP: the scheduled task Northstar India Operations was not found' }
if ([string] $task.State -eq 'Running') { throw 'STOP: a scheduled run is in progress; wait until it finishes' }
$running = @(Invoke-Docker @('ps', '--filter', 'name=northstar-india-operations-', '--format', '{{.Names}}'))
if ($running.Count -gt 0) { throw "STOP: an operations container is running: $($running -join ', ')" }
$imageId = (@(Invoke-Docker @('image', 'inspect', '--format', '{{.Id}}', 'northstar-api:india')) -join '').Trim()
if (-not $imageId) { throw 'STOP: the image northstar-api:india is not available locally' }

[pscustomobject]@{
    engineId = [string] $info.ID
    engineServerVersion = [string] $info.ServerVersion
    engineOperatingSystem = [string] $info.OperatingSystem
    productionVolume = [string] $volume.Name
    productionVolumeCreatedAt = [string] $volume.CreatedAt
    imageId = $imageId
    taskState = [string] $task.State
}
'@

$backupPy = @'
"""Consistent online backup. /data: production volume, READ-ONLY. /work: this drill."""

import fcntl
import hashlib
import json
import os
import shutil
import sqlite3
import sys
from datetime import UTC, datetime
from pathlib import Path

STAMP = sys.argv[1]
SOURCE = Path("/data/northstar.sqlite3")
LOCK = Path("/data/northstar.sqlite3.operations.lock")
HOT_JOURNAL = Path("/data/northstar.sqlite3-journal")
TEMPORARY = Path("/tmp/northstar-backup.sqlite3")
TARGET = Path("/work") / f"northstar-india-{STAMP}.sqlite3"
MANIFEST = Path("/work") / "backup-manifest.json"


def stop(code, reason):
    print(f"STOP: {reason}", flush=True)
    sys.exit(code)


def sha256(path):
    digest = hashlib.sha256()
    with path.open("rb") as handle:
        for block in iter(lambda: handle.read(1 << 20), b""):
            digest.update(block)
    return digest.hexdigest()


def describe(path):
    connection = sqlite3.connect(f"file:{path}?mode=ro", uri=True)
    try:
        integrity = [row[0] for row in connection.execute("PRAGMA integrity_check")]
        violations = connection.execute("PRAGMA foreign_key_check").fetchall()
        names = [row[0] for row in connection.execute(
            "SELECT name FROM sqlite_master WHERE type = 'table' ORDER BY name")]
        counts = {n: connection.execute(f'SELECT COUNT(*) FROM "{n}"').fetchone()[0] for n in names}
        mode = connection.execute("PRAGMA journal_mode").fetchone()[0]
    finally:
        connection.close()
    return {"integrity_check": integrity, "foreign_key_violations": len(violations),
            "journal_mode": mode, "row_counts": counts}


if not SOURCE.is_file():
    stop(10, f"{SOURCE} not found; is the production volume mounted at /data?")
if os.access(SOURCE, os.W_OK):
    stop(11, "the production database is writable here; it must be mounted read-only")
if not LOCK.is_file():
    stop(12, f"{LOCK} not found; refusing to read without the operations lock")
if TARGET.exists() or MANIFEST.exists():
    stop(13, "this drill already holds a backup; prepare a new drill")

lock = os.open(LOCK, os.O_RDONLY)
try:
    try:
        # The same exclusive, non-blocking flock DatabaseOperationsLock takes.
        fcntl.flock(lock, fcntl.LOCK_EX | fcntl.LOCK_NB)
    except BlockingIOError:
        stop(20, "an operations writer holds the database lock; retry after it finishes")
    if HOT_JOURNAL.exists():
        stop(21, "a rollback journal is present (an interrupted write); investigate before any backup")
    source = sqlite3.connect(f"file:{SOURCE}?mode=ro", uri=True)
    target = sqlite3.connect(TEMPORARY)
    try:
        source.backup(target)  # every page in one step: one consistent snapshot
    finally:
        target.close()
        source.close()
finally:
    os.close(lock)  # closing the descriptor releases the lock

checks = describe(TEMPORARY)
with TEMPORARY.open("rb") as source_file, TARGET.open("xb") as target_file:
    shutil.copyfileobj(source_file, target_file)
    target_file.flush()
    os.fsync(target_file.fileno())
manifest = {
    "schema": "northstar.sqlite-backup/1",
    "stamp": STAMP,
    "created_utc": datetime.now(UTC).isoformat(timespec="seconds"),
    "source_volume": "northstar-india-data",
    "source_path": str(SOURCE),
    "method": "sqlite3 backup API, read-only source, operations lock held",
    "backup_file": TARGET.name,
    "bytes": TARGET.stat().st_size,
    "sha256": sha256(TARGET),
    "sqlite_version": sqlite3.sqlite_version,
    **checks,
}
with MANIFEST.open("x", encoding="utf-8") as handle:
    json.dump(manifest, handle, indent=2, sort_keys=True)
    handle.write("\n")
print(json.dumps(manifest, indent=2, sort_keys=True))
if checks["integrity_check"] != ["ok"] or checks["foreign_key_violations"]:
    stop(30, "the backup failed its integrity or foreign-key check; keep it as evidence")
TEMPORARY.unlink()
print("BACKUP OK")
'@

$verifyPy = @'
"""Independent check of the exported backup. /work: this drill, READ-ONLY. Nothing at /data."""

import hashlib
import json
import sqlite3
import sys
from pathlib import Path

STAMP = sys.argv[1]
BACKUP = Path("/work") / f"northstar-india-{STAMP}.sqlite3"
MANIFEST = Path("/work") / "backup-manifest.json"


def stop(code, reason):
    print(f"STOP: {reason}", flush=True)
    sys.exit(code)


def sha256(path):
    digest = hashlib.sha256()
    with path.open("rb") as handle:
        for block in iter(lambda: handle.read(1 << 20), b""):
            digest.update(block)
    return digest.hexdigest()


if any(Path("/data").iterdir()):
    stop(10, "something is visible at /data; this check must mount no volume there")
manifest = json.loads(MANIFEST.read_text(encoding="utf-8"))
digest = sha256(BACKUP)
if digest != manifest["sha256"]:
    stop(20, "the backup's SHA-256 does not match its manifest")
connection = sqlite3.connect(f"file:{BACKUP}?mode=ro&immutable=1", uri=True)
try:
    integrity = [row[0] for row in connection.execute("PRAGMA integrity_check")]
    violations = connection.execute("PRAGMA foreign_key_check").fetchall()
finally:
    connection.close()
print(json.dumps({"backup_file": BACKUP.name, "sha256": digest, "integrity_check": integrity,
                  "foreign_key_violations": len(violations)}, indent=2))
if integrity != ["ok"] or violations:
    stop(30, "the exported backup failed its integrity or foreign-key check")
print("VERIFY OK")
'@

$restorePy = @'
"""Restore into a NEW drill volume. /work: this drill, READ-ONLY. /data: the new drill volume."""

import hashlib
import json
import sqlite3
import sys
from pathlib import Path

STAMP = sys.argv[1]
BACKUP = Path("/work") / f"northstar-india-{STAMP}.sqlite3"
MANIFEST = Path("/work") / "backup-manifest.json"
DATA = Path("/data")
MARKER = DATA / ".northstar-restore-drill"
RESTORED = DATA / "northstar.sqlite3"


def stop(code, reason):
    print(f"STOP: {reason}", flush=True)
    sys.exit(code)


def sha256(path):
    digest = hashlib.sha256()
    with path.open("rb") as handle:
        for block in iter(lambda: handle.read(1 << 20), b""):
            digest.update(block)
    return digest.hexdigest()


# A new drill volume is empty; production never is. Claim it before writing.
if any(DATA.iterdir()):
    stop(10, "/data is not empty; it must be a new drill volume, never production")
manifest = json.loads(MANIFEST.read_text(encoding="utf-8"))
if sha256(BACKUP) != manifest["sha256"]:
    stop(20, "the backup's SHA-256 does not match its manifest")
with MARKER.open("x", encoding="utf-8") as handle:
    handle.write(STAMP + "\n")
source = sqlite3.connect(f"file:{BACKUP}?mode=ro&immutable=1", uri=True)
target = sqlite3.connect(RESTORED)
try:
    source.backup(target)
finally:
    target.close()
    source.close()
connection = sqlite3.connect(f"file:{RESTORED}?mode=ro", uri=True)
try:
    integrity = [row[0] for row in connection.execute("PRAGMA integrity_check")]
    violations = connection.execute("PRAGMA foreign_key_check").fetchall()
finally:
    connection.close()
print(json.dumps({"restored": str(RESTORED), "integrity_check": integrity,
                  "foreign_key_violations": len(violations)}, indent=2))
if integrity != ["ok"] or violations:
    stop(30, "the restored copy failed its integrity or foreign-key check")
print("RESTORE OK")
'@

$comparePy = @'
"""Compare backup and restored copy. /work and /data (the drill volume) are both READ-ONLY."""

import hashlib
import json
import sqlite3
import sys
from pathlib import Path

STAMP = sys.argv[1]
BACKUP = Path("/work") / f"northstar-india-{STAMP}.sqlite3"
MARKER = Path("/data/.northstar-restore-drill")
RESTORED = Path("/data/northstar.sqlite3")
KEY_TABLES = ("futures_forward_research_records", "futures_paper_orders",
              "futures_paper_fills", "futures_ohlcv", "futures_contract_economics")


def stop(code, reason):
    print(f"STOP: {reason}", flush=True)
    sys.exit(code)


def snapshot(path):
    connection = sqlite3.connect(f"file:{path}?mode=ro&immutable=1", uri=True)
    try:
        schema = sorted(repr(row) for row in connection.execute(
            "SELECT type, name, tbl_name, sql FROM sqlite_master"))
        names = [row[0] for row in connection.execute(
            "SELECT name FROM sqlite_master WHERE type = 'table' ORDER BY name")]
        tables = {}
        for name in names:
            rows = sorted(repr(row) for row in connection.execute(f'SELECT * FROM "{name}"'))
            tables[name] = {"rows": len(rows),
                            "sha256": hashlib.sha256("\n".join(rows).encode()).hexdigest()}
        paper = {}
        for name in ("futures_paper_orders", "futures_paper_fills"):
            if name in names:
                cursor = connection.execute(f'SELECT * FROM "{name}"')
                columns = [column[0] for column in cursor.description]
                paper[name] = sorted((dict(zip(columns, row)) for row in cursor.fetchall()),
                                     key=repr)
        latest = None
        if "futures_ohlcv" in names:
            latest = connection.execute("SELECT MAX(point_in_time) FROM futures_ohlcv").fetchone()[0]
    finally:
        connection.close()
    return schema, tables, paper, latest


# Only this drill's restore volume carries this drill's marker; production never does.
if not MARKER.is_file() or MARKER.read_text(encoding="utf-8").strip() != STAMP:
    stop(10, "/data is not this drill's restore volume")
backup_schema, backup_tables, backup_paper, latest = snapshot(BACKUP)
restored_schema, restored_tables, restored_paper, _ = snapshot(RESTORED)
equal = (backup_schema == restored_schema and backup_tables == restored_tables
         and backup_paper == restored_paper)
print(json.dumps({
    "equal": equal,
    "schema_equal": backup_schema == restored_schema,
    "key_tables": {n: {"backup": backup_tables.get(n), "restored": restored_tables.get(n)}
                   for n in KEY_TABLES},
    "differing_tables": sorted(n for n in set(backup_tables) | set(restored_tables)
                               if backup_tables.get(n) != restored_tables.get(n)),
    "latest_daily_bar": latest,
    "paper_orders": backup_paper.get("futures_paper_orders"),
    "paper_fills": backup_paper.get("futures_paper_fills"),
}, indent=2, default=str))
if not equal:
    stop(40, "the backup and the restored copy differ")
print("COMPARE OK")
'@

$lockprobePy = @'
"""Lock behaviour on a DISPOSABLE volume at /data. Never production: needs this drill's marker.

Modes: create; hold-readonly SECONDS; hold-northstar SECONDS;
expect-held-readonly; expect-held-northstar; expect-free-readonly; expect-free-northstar.
"""

import fcntl
import os
import sys
import time
from pathlib import Path

from northstar_api.operations_lock import DatabaseOperationsLock, OperationsAlreadyActiveError

DATA = Path("/data")
DATABASE = DATA / "northstar.sqlite3"
LOCK = DATA / "northstar.sqlite3.operations.lock"
MARKER = DATA / ".northstar-lock-test"
MODE = sys.argv[1]
STAMP = sys.argv[2]


def stop(code, reason):
    print(f"STOP: {reason}", flush=True)
    sys.exit(code)


def readonly_try():
    descriptor = os.open(LOCK, os.O_RDONLY)
    try:
        fcntl.flock(descriptor, fcntl.LOCK_EX | fcntl.LOCK_NB)
    except BlockingIOError:
        os.close(descriptor)
        return None
    return descriptor  # held until the process exits


def northstar_try():
    lock = DatabaseOperationsLock(DATABASE)
    try:
        lock.__enter__()
    except OperationsAlreadyActiveError:
        return None
    return lock  # held until the process exits


if DATABASE.exists():
    stop(10, "a database is visible at /data; the lock test must only use its disposable volume")
if MODE == "create":
    if any(DATA.iterdir()):
        stop(11, "/data is not empty; it must be the new lock-test volume")
    with MARKER.open("x", encoding="utf-8") as handle:
        handle.write(STAMP + "\n")
    with DatabaseOperationsLock(DATABASE):
        pass
    if not LOCK.is_file():
        stop(12, "the lock file was not created")
    print("CREATED")
    sys.exit(0)
if not MARKER.is_file() or MARKER.read_text(encoding="utf-8").strip() != STAMP:
    stop(13, "/data is not this drill's lock-test volume")
if MODE in ("hold-readonly", "hold-northstar"):
    held = readonly_try() if MODE == "hold-readonly" else northstar_try()
    if held is None:
        stop(20, "the lock was not free when the holder started")
    print("LOCKED", flush=True)
    time.sleep(int(sys.argv[3]))
    print("RELEASING", flush=True)  # exiting closes the descriptor and releases the lock
    sys.exit(0)
if MODE.startswith("expect-"):
    _, state, kind = MODE.split("-")
    acquired = (readonly_try() if kind == "readonly" else northstar_try()) is not None
    if state == "held" and acquired:
        stop(50, f"a {kind} acquisition succeeded while another process held the lock")
    if state == "free" and not acquired:
        stop(51, f"a {kind} acquisition failed although the lock should have been released")
    print(f"OK {MODE}")
    sys.exit(0)
stop(2, f"unknown mode {MODE}")
'@

# 1. Production preflight first: nothing is created unless it passes.
$identity = & ([scriptblock]::Create($preflightPs1)) -Deploy $Deploy

# 2. A new drill directory, outside the checkout.
$Stamp = [DateTime]::UtcNow.ToString("yyyyMMdd'T'HHmmss'Z'").ToLowerInvariant()
$Drill = [System.IO.Path]::GetFullPath((Join-Path $EvidenceRoot $Stamp))
if ($Drill.StartsWith($Checkout.TrimEnd('\') + '\', [StringComparison]::OrdinalIgnoreCase)) { throw 'STOP: the drill directory would be inside the Northstar checkout' }
if (Test-Path -LiteralPath $Drill) { throw "STOP: $Drill already exists" }
[void] [System.IO.Directory]::CreateDirectory($Drill)
[void] [System.IO.Directory]::CreateDirectory((Join-Path $Drill 'results'))

# 3. The scripts, and the SHA-256 of each, recorded in drill.json.
$scripts = [ordered]@{
    'preflight.ps1' = $preflightPs1; 'backup.py' = $backupPy; 'verify.py' = $verifyPy
    'restore.py' = $restorePy; 'compare.py' = $comparePy; 'lockprobe.py' = $lockprobePy
}
$hashes = [ordered]@{}
foreach ($name in $scripts.Keys) {
    $path = Join-Path $Drill $name
    [System.IO.File]::WriteAllText($path, $scripts[$name], $Utf8)
    $hashes[$name] = (Get-FileHash -LiteralPath $path -Algorithm SHA256).Hash.ToLowerInvariant()
}
$drillJson = [ordered]@{
    schema = 'northstar.restore-drill/2'; stamp = $Stamp; directory = $Drill
    preparedUtc = [DateTime]::UtcNow.ToString('o'); computer = $env:COMPUTERNAME
    identity = $identity; scripts = $hashes
} | ConvertTo-Json -Depth 4
[System.IO.File]::WriteAllText((Join-Path $Drill 'drill.json'), $drillJson, $Utf8)

"Preflight passed: engine $($identity.engineId), $($identity.productionVolume) created $($identity.productionVolumeCreatedAt), image $($identity.imageId)"
"Drill prepared: $Drill"
"Drill stamp (paste into every later block): $Stamp"
```

### P. Lock behaviour test (disposable volume only; required before B)

Creates `sqlite-lock-test-<stamp>`, records it, and proves the locking
behaviour block B relies on. Production is never mounted. Each step checks the
volume against its record before mounting it.

| Step | Holder (keeps the lock for 40 s) | Then expected |
|------|----------------------------------|---------------|
| 1 | read-only descriptor `flock` (as the backup takes it) | `DatabaseOperationsLock` in another container is refused; a second read-only `flock` is refused; after the holder exits, `DatabaseOperationsLock` succeeds |
| 2 | `DatabaseOperationsLock` (as an operation takes it) | a read-only `flock` is refused (the backup would STOP); after the holder exits, it succeeds |

Holders run detached **without** `--rm`, under exact names and this drill's
labels, so their exit code is always readable. Every wait is bounded: 60 s for
a holder to report `LOCKED` (failing at once if it exits first) and 90 s for it
to exit. Whatever happens, a holder is then stopped if still running (bounded
by `--time 5`), its logs are saved to `results\lock-test.txt`, and only that
exact, label-verified container is removed; the original failure is kept.

`results\lock-test-passed.json` is written **last**: after every probe and
holder check has passed, after no container of this test remains, and after
the lock-test volume has passed its identity checks, been removed, and been
confirmed absent by a successful listing. A failed or interrupted block P never
leaves that record, so block B cannot proceed; run block I to clean up, then
run block P again.

```powershell
$ErrorActionPreference = 'Stop'
$Stamp = 'PASTE-STAMP-FROM-A'
$Drill = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills' $Stamp
$Utf8 = New-Object System.Text.UTF8Encoding($false)
if ($Stamp -notmatch '^\d{8}t\d{6}z$') { throw 'STOP: set $Stamp to the value printed by block A' }
$LockVolume = "sqlite-lock-test-$Stamp"
$Record = Join-Path $Drill 'results\lock-test-volume.json'
$Log = Join-Path $Drill 'results\lock-test.txt'
$Passed = Join-Path $Drill 'results\lock-test-passed.json'
$ExpectedChecks = @(
    '0 create', '1 holder locked', '1a expect-held-northstar', '1b expect-held-readonly', '1 holder exit 0',
    '1c expect-free-northstar', '2 holder locked', '2a expect-held-readonly', '2 holder exit 0',
    '2b expect-free-readonly', 'no lock-test container remains', 'lock-test volume removed'
)
$Completed = New-Object System.Collections.Generic.List[string]

function Invoke-Docker([string[]] $Arguments) {
    $output = & docker @Arguments
    if ($LASTEXITCODE -ne 0) { throw "STOP: docker $($Arguments[0]) $($Arguments[1]) failed with exit code $LASTEXITCODE" }
    $output
}
function Write-Log([string[]] $Lines) { [System.IO.File]::AppendAllLines($Log, [string[]] @($Lines), $Utf8) }
function Test-Volume([string] $Name) { @(Invoke-Docker @('volume', 'ls', '--format', '{{.Name}}')) -ccontains $Name }
function Test-Container([string] $Name) { @(Invoke-Docker @('ps', '-a', '--format', '{{.Names}}')) -ccontains $Name }
function Get-ContainerRecord([string] $Name) {
    $c = (@(Invoke-Docker @('container', 'inspect', '--format', '{{json .}}', $Name)) -join "`n") | ConvertFrom-Json
    [pscustomobject]@{ name = ([string] $c.Name).TrimStart('/'); running = [bool] $c.State.Running; exitCode = [int] $c.State.ExitCode
        purpose = [string] $c.Config.Labels.'northstar.purpose'; drill = [string] $c.Config.Labels.'northstar.drill' }
}
function Remove-OwnedHolder([string] $Name) {
    # Only this drill's exact, label-verified lock-test container; logs are kept first.
    if ($Name -notmatch '^sqlite-lock-test-\d{8}t\d{6}z-[12]$') { throw "STOP: refusing container name $Name" }
    if (-not (Test-Container $Name)) { return }
    $c = Get-ContainerRecord $Name
    if ($c.name -cne $Name -or $c.purpose -ne 'lock-test' -or $c.drill -ne $Stamp) { throw "STOP: $Name is not this drill's lock-test container; not touching it" }
    if ($c.running) { Invoke-Docker @('stop', '--time', '5', $Name) | Out-Null }
    Write-Log (@("== $Name logs (exit $((Get-ContainerRecord $Name).exitCode))") + @(Invoke-Docker @('logs', $Name)))
    Invoke-Docker @('rm', $Name) | Out-Null
    if (Test-Container $Name) { throw "STOP: $Name still exists after removal" }
}
function Get-VolumeRecord([string] $Name) {
    $volume = ((@(Invoke-Docker @('volume', 'inspect', $Name)) -join "`n") | ConvertFrom-Json)[0]
    [pscustomobject]@{ name = [string] $volume.Name; createdAt = [string] $volume.CreatedAt
        purpose = [string] $volume.Labels.'northstar.purpose'; drill = [string] $volume.Labels.'northstar.drill' }
}
function Assert-OwnedVolume {
    if (-not (Test-Volume $LockVolume)) { throw "STOP: $LockVolume is not listed" }
    if (-not (Test-Path -LiteralPath $Record)) { throw "STOP: no creation record for $LockVolume" }
    $recorded = [System.IO.File]::ReadAllText($Record) | ConvertFrom-Json
    $now = Get-VolumeRecord $LockVolume
    if ($now.name -cne $recorded.name -or $now.createdAt -ne $recorded.createdAt -or $now.purpose -ne 'lock-test' -or $now.drill -ne $Stamp) {
        throw "STOP: $LockVolume is not the volume this drill created"
    }
}
function Assert-Script([string] $Name) {
    $expected = ([System.IO.File]::ReadAllText((Join-Path $Drill 'drill.json')) | ConvertFrom-Json).scripts.$Name
    if ((Get-FileHash -LiteralPath (Join-Path $Drill $Name) -Algorithm SHA256).Hash.ToLowerInvariant() -ne $expected) { throw "STOP: $Name changed since block A" }
}
function Get-ProbeArgs([string] $Name, [bool] $ReadOnly, [string[]] $Probe, [bool] $Detached) {
    Assert-OwnedVolume  # before every mount of the test volume
    $mount = "type=volume,source=$LockVolume,target=/data"
    if ($ReadOnly) { $mount += ',readonly' }
    # Probes run in the foreground with --rm: the CLI returns their exit code.
    # Holders run detached WITHOUT --rm, so their exit code stays readable.
    $runArgs = @('run')
    if ($Detached) { $runArgs += '-d' } else { $runArgs += '--rm' }
    $runArgs + @(
        '--name', $Name,
        '--label', 'northstar.purpose=lock-test', '--label', "northstar.drill=$Stamp",
        '--network', 'none', '--pull', 'never',
        '--mount', "type=bind,source=$Drill,target=/work,readonly",
        '--mount', $mount,
        'northstar-api:india', 'python', '-I', '-B', '/work/lockprobe.py'
    ) + $Probe
}
function Invoke-Probe([string] $Step, [string] $Mode, [bool] $ReadOnly) {
    $name = "sqlite-lock-test-$Stamp-$Step"
    if (Test-Container $name) { throw "STOP: a container named $name already exists" }
    $runArgs = Get-ProbeArgs $name $ReadOnly @($Mode, $Stamp) $false
    $out = & docker @runArgs
    $code = $LASTEXITCODE
    Write-Log (@("== $Step $Mode exit $code") + @($out))
    $out
    if ($code -ne 0) { throw "STOP: lock probe $Step ($Mode) exited $code" }
    $Completed.Add("$Step $Mode")
}
function Invoke-Holder([string] $Step, [string] $Mode, [bool] $ReadOnly, [scriptblock] $WhileHeld) {
    $name = "sqlite-lock-test-$Stamp-$Step"
    if (Test-Container $name) { throw "STOP: a container named $name already exists" }
    $failure = $null
    try {
        Invoke-Docker (Get-ProbeArgs $name $ReadOnly @($Mode, $Stamp, '40') $true) | Out-Null
        # Startup: LOCKED within 60 s, failing at once if the holder exits first.
        $deadline = [DateTime]::UtcNow.AddSeconds(60)
        while (@(Invoke-Docker @('logs', $name)) -notcontains 'LOCKED') {
            $state = Get-ContainerRecord $name
            if (-not $state.running) { throw "STOP: holder $name exited (code $($state.exitCode)) before reporting LOCKED" }
            if ([DateTime]::UtcNow -gt $deadline) { throw "STOP: holder $name did not report LOCKED within 60 s" }
            Start-Sleep -Seconds 1
        }
        $Completed.Add("$Step holder locked")
        & $WhileHeld
        # The competing probes are only meaningful if the holder held the lock throughout.
        if (-not (Get-ContainerRecord $name).running) { throw "STOP: holder $name stopped before its competing probes finished" }
        # Exit: the holder releases after 40 s; never wait more than 90 s.
        $deadline = [DateTime]::UtcNow.AddSeconds(90)
        while ((Get-ContainerRecord $name).running) {
            if ([DateTime]::UtcNow -gt $deadline) { throw "STOP: holder $name did not exit within 90 s" }
            Start-Sleep -Seconds 1
        }
        $exit = (Get-ContainerRecord $name).exitCode
        if ($exit -ne 0) { throw "STOP: holder $name exited $exit" }
        $Completed.Add("$Step holder exit 0")
    } catch {
        $failure = $_
    }
    # Always: stop if still running (bounded), keep its logs, remove only this exact container.
    try { Remove-OwnedHolder $name } catch {
        Write-Log @("== cleanup of $name failed: $($_.Exception.Message)")
        if (-not $failure) { $failure = $_ }
    }
    if ($failure) { throw $failure }
}

$drillPath = Join-Path $Drill 'drill.json'
if (-not (Test-Path -LiteralPath $drillPath)) { throw "STOP: $Drill is not a prepared drill" }
Assert-Script 'lockprobe.py'
if (Test-Path -LiteralPath $Passed) { throw 'STOP: the lock test already passed for this drill' }
if (Test-Volume $LockVolume) { throw "STOP: $LockVolume already exists; run block I first, never reuse a test volume" }
Invoke-Docker @('volume', 'create', '--label', 'northstar.purpose=lock-test', '--label', "northstar.drill=$Stamp", $LockVolume) | Out-Null
$created = Get-VolumeRecord $LockVolume
if ($created.name -cne $LockVolume -or $created.purpose -ne 'lock-test' -or $created.drill -ne $Stamp) { throw 'STOP: the lock-test volume does not carry its labels' }
[System.IO.File]::WriteAllText($Record, ($created | ConvertTo-Json), $Utf8)

# 1. Every probe and holder check. Any failure throws; the record is never written.
Invoke-Probe '0' 'create' $false
Invoke-Holder '1' 'hold-readonly' $true {
    Invoke-Probe '1a' 'expect-held-northstar' $false
    Invoke-Probe '1b' 'expect-held-readonly' $true
}
Invoke-Probe '1c' 'expect-free-northstar' $false
Invoke-Holder '2' 'hold-northstar' $false {
    Invoke-Probe '2a' 'expect-held-readonly' $true
}
Invoke-Probe '2b' 'expect-free-readonly' $true

# 2. No container of this test remains.
$remaining = @(Invoke-Docker @('ps', '-a', '--filter', "label=northstar.drill=$Stamp", '--filter', 'label=northstar.purpose=lock-test', '--format', '{{.Names}}'))
if ($remaining.Count -gt 0) { throw "STOP: lock-test containers remain: $($remaining -join ', ')" }
$Completed.Add('no lock-test container remains')

# 3. Remove the lock-test volume: owned, unused, removed, then confirmed absent.
Assert-OwnedVolume
if (@(Invoke-Docker @('ps', '-a', '--filter', "volume=$LockVolume", '--format', '{{.Names}}')).Count -gt 0) { throw 'STOP: a container still uses the lock-test volume' }
Invoke-Docker @('volume', 'rm', $LockVolume) | Out-Null
if (Test-Volume $LockVolume) { throw "STOP: $LockVolume still exists after removal" }
$Completed.Add('lock-test volume removed')

# 4. Only now: the success record, with every completed check.
if ((@($Completed) -join '|') -cne ($ExpectedChecks -join '|')) { throw "STOP: the lock test did not complete every check: $(@($Completed) -join ', ')" }
$record = [ordered]@{
    schema = 'northstar.lock-test/2'; stamp = $Stamp; result = 'passed'
    volume = $LockVolume; volumeRemoved = $true; checks = @($Completed)
    completedUtc = [DateTime]::UtcNow.ToString('o')
} | ConvertTo-Json
[System.IO.File]::WriteAllText($Passed, $record, $Utf8)
"LOCK TEST OK; $LockVolume removed"
```

### B. Consistent online backup (the only block that mounts production)

Re-runs the recorded preflight, compares it with block A's record, requires a
passed lock test, and only then mounts `northstar-india-data` **read-only**
(and the drill directory read-write). It takes the operations lock, backs up
with the SQLite backup API, checks the copy, and writes
`northstar-india-<stamp>.sqlite3` and `backup-manifest.json` into the drill
directory.

```powershell
$ErrorActionPreference = 'Stop'
$Stamp = 'PASTE-STAMP-FROM-A'
$Deploy = 'C:\Local Disk(E)\Codes\Northstar\northstar-api\deploy\india'
$Drill = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills' $Stamp
$Utf8 = New-Object System.Text.UTF8Encoding($false)
if ($Stamp -notmatch '^\d{8}t\d{6}z$') { throw 'STOP: set $Stamp to the value printed by block A' }

function Invoke-Docker([string[]] $Arguments) {
    $output = & docker @Arguments
    if ($LASTEXITCODE -ne 0) { throw "STOP: docker $($Arguments[0]) $($Arguments[1]) failed with exit code $LASTEXITCODE" }
    $output
}
$drillPath = Join-Path $Drill 'drill.json'
if (-not (Test-Path -LiteralPath $drillPath)) { throw "STOP: $Drill is not a prepared drill" }
$prepared = [System.IO.File]::ReadAllText($drillPath) | ConvertFrom-Json
foreach ($name in @('preflight.ps1', 'backup.py')) {
    if ((Get-FileHash -LiteralPath (Join-Path $Drill $name) -Algorithm SHA256).Hash.ToLowerInvariant() -ne $prepared.scripts.$name) { throw "STOP: $name changed since block A" }
}
# Block P must have completed every check and its own cleanup for THIS drill.
$passedPath = Join-Path $Drill 'results\lock-test-passed.json'
if (-not (Test-Path -LiteralPath $passedPath)) { throw 'STOP: run block P first; the lock test has not passed for this drill' }
try { $passed = [System.IO.File]::ReadAllText($passedPath) | ConvertFrom-Json } catch { throw 'STOP: lock-test-passed.json is not valid JSON' }
$expectedChecks = @(
    '0 create', '1 holder locked', '1a expect-held-northstar', '1b expect-held-readonly', '1 holder exit 0',
    '1c expect-free-northstar', '2 holder locked', '2a expect-held-readonly', '2 holder exit 0',
    '2b expect-free-readonly', 'no lock-test container remains', 'lock-test volume removed'
)
if ($passed.schema -cne 'northstar.lock-test/2' -or $passed.stamp -cne $Stamp -or $passed.result -cne 'passed' -or
    $passed.volumeRemoved -ne $true -or $passed.volume -cne "sqlite-lock-test-$Stamp" -or
    (@($passed.checks) -join '|') -cne ($expectedChecks -join '|')) {
    throw 'STOP: lock-test-passed.json does not record a complete lock test for this drill'
}
if (@(Invoke-Docker @('volume', 'ls', '--format', '{{.Name}}')) -ccontains "sqlite-lock-test-$Stamp") { throw 'STOP: the lock-test volume still exists; block P did not finish' }
if (@(Invoke-Docker @('ps', '-a', '--filter', "label=northstar.drill=$Stamp", '--format', '{{.Names}}')).Count -gt 0) { throw 'STOP: containers of this drill still exist; run block I' }
if (Test-Path -LiteralPath (Join-Path $Drill "northstar-india-$Stamp.sqlite3")) { throw 'STOP: this drill already holds a backup' }

# The production preflight again, immediately before production is mounted.
$preflight = [scriptblock]::Create([System.IO.File]::ReadAllText((Join-Path $Drill 'preflight.ps1')))
$identity = & $preflight -Deploy $Deploy
foreach ($field in @('engineId', 'productionVolume', 'productionVolumeCreatedAt', 'imageId')) {
    if ([string] $identity.$field -cne [string] $prepared.identity.$field) { throw "STOP: $field changed since block A; prepare a new drill" }
}
$name = "sqlite-backup-$Stamp"
if (@(Invoke-Docker @('ps', '-a', '--format', '{{.Names}}')) -ccontains $name) { throw "STOP: a container named $name already exists" }

$runArgs = @(
    'run', '--rm', '--name', $name,
    '--label', 'northstar.purpose=sqlite-backup', '--label', "northstar.drill=$Stamp",
    '--network', 'none', '--pull', 'never',
    '--mount', 'type=volume,source=northstar-india-data,target=/data,readonly',
    '--mount', "type=bind,source=$Drill,target=/work",
    'northstar-api:india', 'python', '-I', '-B', '/work/backup.py', $Stamp
)
$out = & docker @runArgs
$code = $LASTEXITCODE
[System.IO.File]::WriteAllLines((Join-Path $Drill 'results\backup.txt'), [string[]] @($out), $Utf8)
$out
if ($code -ne 0) { throw "STOP: the backup exited $code. Production was mounted read-only." }
```

### C. Copy to the Windows evidence directory

Block B already wrote the backup into the drill directory through its bind
mount, so there is no separate copy step and the backup is never written into
the production volume. The drill directory is the evidence location. Block D
confirms the file on the Windows side.

### D. SHA-256 and backup metadata

Computes the SHA-256 on Windows, compares it with the manifest, and records
`backup-metadata.json`.

```powershell
$ErrorActionPreference = 'Stop'
$Stamp = 'PASTE-STAMP-FROM-A'
$Drill = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills' $Stamp
$Utf8 = New-Object System.Text.UTF8Encoding($false)
if ($Stamp -notmatch '^\d{8}t\d{6}z$') { throw 'STOP: set $Stamp to the value printed by block A' }

function Invoke-Docker([string[]] $Arguments) {
    $output = & docker @Arguments
    if ($LASTEXITCODE -ne 0) { throw "STOP: docker $($Arguments[0]) $($Arguments[1]) failed with exit code $LASTEXITCODE" }
    $output
}
$backup = Join-Path $Drill "northstar-india-$Stamp.sqlite3"
$manifestPath = Join-Path $Drill 'backup-manifest.json'
$metadataPath = Join-Path $Drill 'backup-metadata.json'
if (-not (Test-Path -LiteralPath $backup -PathType Leaf) -or -not (Test-Path -LiteralPath $manifestPath)) { throw 'STOP: no backup in this drill; run block B first' }
if (Test-Path -LiteralPath $metadataPath) { throw 'STOP: metadata already recorded for this drill' }

$manifest = [System.IO.File]::ReadAllText($manifestPath) | ConvertFrom-Json
$hash = (Get-FileHash -LiteralPath $backup -Algorithm SHA256).Hash.ToLowerInvariant()
if ($hash -ne $manifest.sha256) { throw 'STOP: the Windows SHA-256 does not match the manifest' }
$volume = ((@(Invoke-Docker @('volume', 'inspect', 'northstar-india-data')) -join "`n") | ConvertFrom-Json)[0]

$metadata = [ordered]@{
    schema = 'northstar.sqlite-backup-metadata/1'
    stamp = $Stamp
    recordedUtc = [DateTime]::UtcNow.ToString('o')
    computer = $env:COMPUTERNAME
    backupFile = [System.IO.Path]::GetFileName($backup)
    bytes = (Get-Item -LiteralPath $backup).Length
    sha256 = $hash
    manifestSha256Matches = $true
    integrityCheck = $manifest.integrity_check
    foreignKeyViolations = $manifest.foreign_key_violations
    dockerServerVersion = (@(Invoke-Docker @('version', '--format', '{{.Server.Version}}')) -join '').Trim()
    imageId = (@(Invoke-Docker @('image', 'inspect', '--format', '{{.Id}}', 'northstar-api:india')) -join '').Trim()
    productionVolume = [string] $volume.Name
    productionVolumeCreatedAt = [string] $volume.CreatedAt
}
[System.IO.File]::WriteAllText($metadataPath, ($metadata | ConvertTo-Json), $Utf8)
"SHA-256 $hash matches the manifest; metadata written to $metadataPath"
```

### E. Integrity and foreign-key checks of the exported file

An independent check of the file as it now sits on Windows. The container
mounts only the drill directory, read-only; its `/data` is the image's own
empty directory.

```powershell
$ErrorActionPreference = 'Stop'
$Stamp = 'PASTE-STAMP-FROM-A'
$Drill = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills' $Stamp
$Utf8 = New-Object System.Text.UTF8Encoding($false)
if ($Stamp -notmatch '^\d{8}t\d{6}z$') { throw 'STOP: set $Stamp to the value printed by block A' }
$prepared = [System.IO.File]::ReadAllText((Join-Path $Drill 'drill.json')) | ConvertFrom-Json
if ((Get-FileHash -LiteralPath (Join-Path $Drill 'verify.py') -Algorithm SHA256).Hash.ToLowerInvariant() -ne $prepared.scripts.'verify.py') { throw 'STOP: verify.py changed since block A' }

$runArgs = @(
    'run', '--rm', '--name', "sqlite-restore-drill-$Stamp-verify",
    '--label', 'northstar.purpose=restore-drill', '--label', "northstar.drill=$Stamp",
    '--network', 'none', '--pull', 'never',
    '--mount', "type=bind,source=$Drill,target=/work,readonly",
    'northstar-api:india', 'python', '-I', '-B', '/work/verify.py', $Stamp
)
$out = & docker @runArgs
$code = $LASTEXITCODE
[System.IO.File]::WriteAllLines((Join-Path $Drill 'results\verify.txt'), [string[]] @($out), $Utf8)
$out
if ($code -ne 0) { throw "STOP: verification exited $code" }
```

### F. Restore into a new disposable volume

Creates `sqlite-restore-drill-<stamp>`, records it, verifies it against the
record, and restores the backup into it. Neither the production volume nor any
Compose file is involved.

```powershell
$ErrorActionPreference = 'Stop'
$Stamp = 'PASTE-STAMP-FROM-A'
$Drill = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills' $Stamp
$Utf8 = New-Object System.Text.UTF8Encoding($false)
if ($Stamp -notmatch '^\d{8}t\d{6}z$') { throw 'STOP: set $Stamp to the value printed by block A' }
$DrillVolume = "sqlite-restore-drill-$Stamp"
$Record = Join-Path $Drill 'results\drill-volume.json'

function Invoke-Docker([string[]] $Arguments) {
    $output = & docker @Arguments
    if ($LASTEXITCODE -ne 0) { throw "STOP: docker $($Arguments[0]) $($Arguments[1]) failed with exit code $LASTEXITCODE" }
    $output
}
function Test-Volume([string] $Name) { @(Invoke-Docker @('volume', 'ls', '--format', '{{.Name}}')) -ccontains $Name }
function Get-VolumeRecord([string] $Name) {
    $volume = ((@(Invoke-Docker @('volume', 'inspect', $Name)) -join "`n") | ConvertFrom-Json)[0]
    [pscustomobject]@{ name = [string] $volume.Name; createdAt = [string] $volume.CreatedAt
        purpose = [string] $volume.Labels.'northstar.purpose'; drill = [string] $volume.Labels.'northstar.drill' }
}
function Assert-OwnedVolume {
    if (-not (Test-Volume $DrillVolume)) { throw "STOP: $DrillVolume is not listed" }
    $recorded = [System.IO.File]::ReadAllText($Record) | ConvertFrom-Json
    $now = Get-VolumeRecord $DrillVolume
    if ($now.name -cne $recorded.name -or $now.createdAt -ne $recorded.createdAt -or $now.purpose -ne 'restore-drill' -or $now.drill -ne $Stamp) {
        throw "STOP: $DrillVolume is not the volume this drill created"
    }
}

$prepared = [System.IO.File]::ReadAllText((Join-Path $Drill 'drill.json')) | ConvertFrom-Json
if ((Get-FileHash -LiteralPath (Join-Path $Drill 'restore.py') -Algorithm SHA256).Hash.ToLowerInvariant() -ne $prepared.scripts.'restore.py') { throw 'STOP: restore.py changed since block A' }
$verified = Join-Path $Drill 'results\verify.txt'
if (-not (Test-Path -LiteralPath $verified) -or @([System.IO.File]::ReadAllLines($verified))[-1] -ne 'VERIFY OK') { throw 'STOP: run blocks B, D and E first; the backup is not verified' }
if (Test-Volume $DrillVolume) { throw "STOP: $DrillVolume already exists; never reuse a drill volume" }

Invoke-Docker @('volume', 'create', '--label', 'northstar.purpose=restore-drill', '--label', "northstar.drill=$Stamp", $DrillVolume) | Out-Null
$created = Get-VolumeRecord $DrillVolume
if ($created.name -cne $DrillVolume -or $created.purpose -ne 'restore-drill' -or $created.drill -ne $Stamp) { throw 'STOP: the drill volume does not carry its labels' }
[System.IO.File]::WriteAllText($Record, ($created | ConvertTo-Json), $Utf8)

Assert-OwnedVolume
$runArgs = @(
    'run', '--rm', '--name', "sqlite-restore-drill-$Stamp-restore",
    '--label', 'northstar.purpose=restore-drill', '--label', "northstar.drill=$Stamp",
    '--network', 'none', '--pull', 'never',
    '--mount', "type=bind,source=$Drill,target=/work,readonly",
    '--mount', "type=volume,source=$DrillVolume,target=/data",
    'northstar-api:india', 'python', '-I', '-B', '/work/restore.py', $Stamp
)
$out = & docker @runArgs
$code = $LASTEXITCODE
[System.IO.File]::WriteAllLines((Join-Path $Drill 'results\restore.txt'), [string[]] @($out), $Utf8)
$out
if ($code -ne 0) { throw "STOP: the restore exited $code; run block I to remove the drill volume" }
```

### G. Validate the restored copy

The restore script (block F) already checked the restored copy with
`PRAGMA integrity_check` and `PRAGMA foreign_key_check`. No production writer
is started at any point: the scheduled task and its `india-operations` service
are not involved, and no container receives the Upstox token.

### H. Compare the backup and the restored copy

Both are mounted read-only. The comparison covers the full schema and every
table's row count and content digest, highlights the Futures tables, and prints
the stored paper orders and fills. Those rows hold no secrets, and they are
what evidence entries E-1 to E-3 need.

```powershell
$ErrorActionPreference = 'Stop'
$Stamp = 'PASTE-STAMP-FROM-A'
$Drill = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills' $Stamp
$Utf8 = New-Object System.Text.UTF8Encoding($false)
if ($Stamp -notmatch '^\d{8}t\d{6}z$') { throw 'STOP: set $Stamp to the value printed by block A' }
$DrillVolume = "sqlite-restore-drill-$Stamp"
$Record = Join-Path $Drill 'results\drill-volume.json'

function Invoke-Docker([string[]] $Arguments) {
    $output = & docker @Arguments
    if ($LASTEXITCODE -ne 0) { throw "STOP: docker $($Arguments[0]) $($Arguments[1]) failed with exit code $LASTEXITCODE" }
    $output
}
function Test-Volume([string] $Name) { @(Invoke-Docker @('volume', 'ls', '--format', '{{.Name}}')) -ccontains $Name }
function Get-VolumeRecord([string] $Name) {
    $volume = ((@(Invoke-Docker @('volume', 'inspect', $Name)) -join "`n") | ConvertFrom-Json)[0]
    [pscustomobject]@{ name = [string] $volume.Name; createdAt = [string] $volume.CreatedAt
        purpose = [string] $volume.Labels.'northstar.purpose'; drill = [string] $volume.Labels.'northstar.drill' }
}
function Assert-OwnedVolume {
    if (-not (Test-Volume $DrillVolume)) { throw "STOP: $DrillVolume is not listed" }
    if (-not (Test-Path -LiteralPath $Record)) { throw "STOP: no creation record for $DrillVolume" }
    $recorded = [System.IO.File]::ReadAllText($Record) | ConvertFrom-Json
    $now = Get-VolumeRecord $DrillVolume
    if ($now.name -cne $recorded.name -or $now.createdAt -ne $recorded.createdAt -or $now.purpose -ne 'restore-drill' -or $now.drill -ne $Stamp) {
        throw "STOP: $DrillVolume is not the volume this drill created"
    }
}

$prepared = [System.IO.File]::ReadAllText((Join-Path $Drill 'drill.json')) | ConvertFrom-Json
if ((Get-FileHash -LiteralPath (Join-Path $Drill 'compare.py') -Algorithm SHA256).Hash.ToLowerInvariant() -ne $prepared.scripts.'compare.py') { throw 'STOP: compare.py changed since block A' }
Assert-OwnedVolume
$runArgs = @(
    'run', '--rm', '--name', "sqlite-restore-drill-$Stamp-compare",
    '--label', 'northstar.purpose=restore-drill', '--label', "northstar.drill=$Stamp",
    '--network', 'none', '--pull', 'never',
    '--mount', "type=bind,source=$Drill,target=/work,readonly",
    '--mount', "type=volume,source=$DrillVolume,target=/data,readonly",
    'northstar-api:india', 'python', '-I', '-B', '/work/compare.py', $Stamp
)
$out = & docker @runArgs
$code = $LASTEXITCODE
[System.IO.File]::WriteAllLines((Join-Path $Drill 'results\compare.txt'), [string[]] @($out), $Utf8)
$out
if ($code -ne 0) { throw "STOP: the comparison exited $code" }
```

#### H2. Optional: Northstar's own read-only report of the restored copy

Runs `northstar paper status` against the **restored copy only**, after the
drill volume is verified against its record. The strategy and portfolio are
the production identities. Set `$AsOf` to the `latest_daily_bar` value printed
by block H. Run H2 only after H: opening the copy may create missing tables
(never rows), which would alter the file.

```powershell
$ErrorActionPreference = 'Stop'
$Stamp = 'PASTE-STAMP-FROM-A'
$AsOf = 'PASTE-latest_daily_bar-FROM-H'
$Drill = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills' $Stamp
$Utf8 = New-Object System.Text.UTF8Encoding($false)
if ($Stamp -notmatch '^\d{8}t\d{6}z$') { throw 'STOP: set $Stamp to the value printed by block A' }
if ($AsOf -notmatch '^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d+)?(Z|[+-]\d{2}:\d{2})$') { throw 'STOP: set $AsOf from block H' }
$DrillVolume = "sqlite-restore-drill-$Stamp"
$Record = Join-Path $Drill 'results\drill-volume.json'

function Invoke-Docker([string[]] $Arguments) {
    $output = & docker @Arguments
    if ($LASTEXITCODE -ne 0) { throw "STOP: docker $($Arguments[0]) $($Arguments[1]) failed with exit code $LASTEXITCODE" }
    $output
}
function Test-Volume([string] $Name) { @(Invoke-Docker @('volume', 'ls', '--format', '{{.Name}}')) -ccontains $Name }
function Get-VolumeRecord([string] $Name) {
    $volume = ((@(Invoke-Docker @('volume', 'inspect', $Name)) -join "`n") | ConvertFrom-Json)[0]
    [pscustomobject]@{ name = [string] $volume.Name; createdAt = [string] $volume.CreatedAt
        purpose = [string] $volume.Labels.'northstar.purpose'; drill = [string] $volume.Labels.'northstar.drill' }
}
function Assert-OwnedVolume {
    if (-not (Test-Volume $DrillVolume)) { throw "STOP: $DrillVolume is not listed" }
    if (-not (Test-Path -LiteralPath $Record)) { throw "STOP: no creation record for $DrillVolume" }
    $recorded = [System.IO.File]::ReadAllText($Record) | ConvertFrom-Json
    $now = Get-VolumeRecord $DrillVolume
    if ($now.name -cne $recorded.name -or $now.createdAt -ne $recorded.createdAt -or $now.purpose -ne 'restore-drill' -or $now.drill -ne $Stamp) {
        throw "STOP: $DrillVolume is not the volume this drill created"
    }
}

$compared = Join-Path $Drill 'results\compare.txt'
if (-not (Test-Path -LiteralPath $compared) -or @([System.IO.File]::ReadAllLines($compared))[-1] -ne 'COMPARE OK') { throw 'STOP: run block H first' }
Assert-OwnedVolume
$runArgs = @(
    'run', '--rm', '--name', "sqlite-restore-drill-$Stamp-status",
    '--label', 'northstar.purpose=restore-drill', '--label', "northstar.drill=$Stamp",
    '--network', 'none', '--pull', 'never',
    '--mount', "type=volume,source=$DrillVolume,target=/data",
    'northstar-api:india', 'northstar', 'paper', 'status', '--database', '/data/northstar.sqlite3',
    '--strategy', 'directional-mvp-v1-nifty-paper-ops', '--portfolio', 'nifty-paper-ops-oct26', '--as-of', $AsOf
)
$out = & docker @runArgs
$code = $LASTEXITCODE
[System.IO.File]::WriteAllLines((Join-Path $Drill 'results\paper-status.txt'), [string[]] @($out), $Utf8)
$out
"paper status exited $code (4 means P&L unavailable, for example missing economics)"
```

### I. Cleanup: only resources this drill demonstrably created

First, if an interrupted block P left a detached lock-test holder
(`sqlite-lock-test-<stamp>-1` or `-2`), it verifies the container's exact name
and labels, gives a running holder up to 60 s to release by itself, stops it if
needed (bounded by `--time 5`), appends its logs to `results\lock-test.txt`,
and removes only that container. Any other leftover container of this drill
stops the cleanup for inspection.

Then it removes this drill's restore volume and, if block P stopped before
removing it, its lock-test volume. Each is removed only when it is listed, has a
creation record in `results`, carries this drill's labels, has the recorded
`CreatedAt`, and no container uses it. Absence is confirmed by a successful
listing. Nothing else is removed: not the production volume, not the drill
directory, its backup, manifests, results or logs. Safe to run at any point,
including when the drill stopped before any volume was created.

```powershell
$ErrorActionPreference = 'Stop'
$Stamp = 'PASTE-STAMP-FROM-A'
$Drill = Join-Path 'C:\Local Disk(E)\Codes\NorthstarEvidence\sqlite-backup-drills' $Stamp
if ($Stamp -notmatch '^\d{8}t\d{6}z$') { throw 'STOP: set $Stamp to the value printed by block A' }

function Invoke-Docker([string[]] $Arguments) {
    $output = & docker @Arguments
    if ($LASTEXITCODE -ne 0) { throw "STOP: docker $($Arguments[0]) $($Arguments[1]) failed with exit code $LASTEXITCODE" }
    $output
}
function Test-Volume([string] $Name) { @(Invoke-Docker @('volume', 'ls', '--format', '{{.Name}}')) -ccontains $Name }
function Get-VolumeRecord([string] $Name) {
    $volume = ((@(Invoke-Docker @('volume', 'inspect', $Name)) -join "`n") | ConvertFrom-Json)[0]
    [pscustomobject]@{ name = [string] $volume.Name; createdAt = [string] $volume.CreatedAt
        purpose = [string] $volume.Labels.'northstar.purpose'; drill = [string] $volume.Labels.'northstar.drill' }
}

function Get-ContainerRecord([string] $Name) {
    $c = (@(Invoke-Docker @('container', 'inspect', '--format', '{{json .}}', $Name)) -join "`n") | ConvertFrom-Json
    [pscustomobject]@{ name = ([string] $c.Name).TrimStart('/'); running = [bool] $c.State.Running; exitCode = [int] $c.State.ExitCode
        purpose = [string] $c.Config.Labels.'northstar.purpose'; drill = [string] $c.Config.Labels.'northstar.drill' }
}

# An interrupted block P can leave a detached lock-test holder (no --rm). Only
# those exact, label-verified containers are handled here: give a running one up
# to 60 s to release by itself, stop it if needed (bounded), keep its logs, then
# remove it. Any other leftover container stops the cleanup for inspection.
$leftover = @(Invoke-Docker @('ps', '-a', '--filter', "label=northstar.drill=$Stamp", '--format', '{{.Names}}'))
foreach ($name in $leftover) {
    if ($name -notmatch "^sqlite-lock-test-$Stamp-[12]$") { throw "STOP: container $name of this drill still exists; inspect it before re-running this block" }
    $c = Get-ContainerRecord $name
    if ($c.name -cne $name -or $c.purpose -ne 'lock-test' -or $c.drill -ne $Stamp) { throw "STOP: $name is not this drill's lock-test container; not touching it" }
    $deadline = [DateTime]::UtcNow.AddSeconds(60)
    while ((Get-ContainerRecord $name).running -and [DateTime]::UtcNow -lt $deadline) { Start-Sleep -Seconds 1 }
    if ((Get-ContainerRecord $name).running) { Invoke-Docker @('stop', '--time', '5', $name) | Out-Null }
    $results = Join-Path $Drill 'results'
    if (-not (Test-Path -LiteralPath $results)) { throw "STOP: $results is missing; not removing $name without keeping its logs" }
    $lines = @("== block I: $name logs (exit $((Get-ContainerRecord $name).exitCode))") + @(Invoke-Docker @('logs', $name))
    [System.IO.File]::AppendAllLines((Join-Path $results 'lock-test.txt'), [string[]] $lines, (New-Object System.Text.UTF8Encoding($false)))
    Invoke-Docker @('rm', $name) | Out-Null
    if (@(Invoke-Docker @('ps', '-a', '--format', '{{.Names}}')) -ccontains $name) { throw "STOP: $name still exists after removal" }
    "Removed leftover lock-test holder $name (logs kept in results\lock-test.txt)"
}

$owned = @(
    @{ Name = "sqlite-restore-drill-$Stamp"; Purpose = 'restore-drill'; Record = Join-Path $Drill 'results\drill-volume.json' },
    @{ Name = "sqlite-lock-test-$Stamp"; Purpose = 'lock-test'; Record = Join-Path $Drill 'results\lock-test-volume.json' }
)
foreach ($entry in $owned) {
    $name = $entry.Name
    if ($name -notmatch '^sqlite-(restore-drill|lock-test)-\d{8}t\d{6}z$') { throw "STOP: refusing volume name $name" }
    if (-not (Test-Volume $name)) { "$name is not listed; nothing to remove"; continue }
    if (-not (Test-Path -LiteralPath $entry.Record)) { throw "STOP: $name exists but this drill has no creation record for it; not removing it" }
    $recorded = [System.IO.File]::ReadAllText($entry.Record) | ConvertFrom-Json
    $now = Get-VolumeRecord $name
    if ($now.name -cne $recorded.name -or $now.createdAt -ne $recorded.createdAt -or $now.purpose -ne $entry.Purpose -or $now.drill -ne $Stamp) {
        throw "STOP: $name does not match this drill's creation record; not removing it"
    }
    if (@(Invoke-Docker @('ps', '-a', '--filter', "volume=$name", '--format', '{{.Names}}')).Count -gt 0) { throw "STOP: a container still uses $name" }
    Invoke-Docker @('volume', 'rm', $name) | Out-Null
    if (Test-Volume $name) { throw "STOP: $name still exists after removal" }
    "Removed $name"
}

# The production volume was not recreated during the drill. This compares only
# the volume's identity; it is not proof that the database contents are unchanged.
$production = ((@(Invoke-Docker @('volume', 'inspect', 'northstar-india-data')) -join "`n") | ConvertFrom-Json)[0]
$drillPath = Join-Path $Drill 'drill.json'
if (Test-Path -LiteralPath $drillPath) {
    $prepared = [System.IO.File]::ReadAllText($drillPath) | ConvertFrom-Json
    if ([string] $production.CreatedAt -cne [string] $prepared.identity.productionVolumeCreatedAt) { throw 'STOP: the production volume was recreated since block A; investigate' }
    "Production volume not recreated (CreatedAt $($production.CreatedAt))."
}
"Cleanup complete. Drill evidence kept in $Drill"
```

### J. Record the drill

Add an entry to the
[Paper MVP Evidence Register](Indian-Futures-Paper-MVP-Evidence-Register.md)
for every run, whether it passed or stopped:

- the stamp, the date, and who ran it;
- the outcome of blocks P, B, D, E, F and H (`LOCK TEST OK`, `BACKUP OK`, the
  SHA-256 match, `VERIFY OK`, `RESTORE OK`, `COMPARE OK`), or the first `STOP`
  message;
- the backup's SHA-256 and size, and the key-table row counts from
  `results\compare.txt`;
- the drill directory path. Do not copy the database or its rows into a
  repository.

B-1 of the acceptance specification needs `BACKUP OK`, the SHA-256 match and
`VERIFY OK`; B-2 additionally needs `RESTORE OK` and `COMPARE OK`. Neither is
PASS before a supervised run records those outcomes.

## 5. Assumptions to confirm on the deployment machine

These follow from the code and Docker's documented behaviour but have not been
observed on the deployment machine. Each fails closed if wrong. Block P checks
the first two before production is touched.

1. A `flock` taken through a read-only descriptor, on a read-only mount,
   conflicts with `DatabaseOperationsLock` in another container in the same
   Docker Desktop VM, in both directions, and is released when its process
   exits. (Block P.)
2. A new named volume mounted at `/data` is owned by uid 10001 on first mount,
   copied from the image's `/data`, so the scripts can write to it. (Block P's
   `create` step, then block F.)
3. Docker Desktop shares `C:\Local Disk(E)\Codes\NorthstarEvidence` with
   containers, and uid 10001 can write the backup there. If it cannot, block B
   fails before writing a manifest.
4. The production `.env` does not override `NORTHSTAR_DATABASE`, and the lock
   file already exists because operations have run. (Preflight; backup script.)
5. No `northstar.sqlite3-journal` exists between operations. The database uses
   the default rollback journal, so a leftover journal means an interrupted
   write.
6. There is enough free space for the database twice in Docker Desktop's disk
   (in `/tmp` of the backup container and in the restore volume) and once in
   the drill directory.
7. `docker info` reports `OperatingSystem` containing `Docker Desktop` and a
   non-empty `ID`; `docker volume inspect` reports `CreatedAt`.
8. `paper status` (H2) accepts the stored `point_in_time` text as `--as-of`.

## 6. Rollback

Nothing in production is changed, so there is nothing to roll back there. The
production volume is mounted read-only, and only during block B. The lock file
is opened read-only and the lock is released when block B's container exits.

If a block stops, run block I. It removes only the volumes this drill
recorded. The drill directory, with every output written so far, stays as
evidence; delete it by hand only when it is no longer needed. If the hourly
task reported `SKIPPED` during block B, the next hourly run processes normally.
