# Indian Options Daily Market Data (NIFTY / NSE / Upstox)

How an operator acquires the historical daily candles of one exact NIFTY option
contract from Upstox into a Northstar database. This is manual acquisition only.
It never selects a contract, never reads the current day's candle and never
declares a candle final.

Decisions: ADR-013 (persisted provider listings), ADR-014 (canonical option
daily bars and sessions) and ADR-015 (observed-subset acquisition).

## 1. Prerequisite: a persisted listing

The contract must already have an Upstox listing stored in the same database.
Listings come only from the public instrument master, while the contract is
listed:

```powershell
.\.venv\Scripts\northstar.exe options instruments sync `
  --database <PATH> --product NIFTY --exchange NSE
```

Check one contract's stored listing (read-only):

```powershell
.\.venv\Scripts\northstar.exe options instruments show `
  --database <PATH> --product NIFTY --exchange NSE `
  --expiration 2026-10-27 --strike 22600 --right PUT
```

Market-data sync never falls back to the live master. A contract with no stored
listing exits 4. Run `options instruments sync` regularly, so contracts are
captured while they are listed.

## 2. The command

```powershell
$env:UPSTOX_ANALYTICS_TOKEN = "<token>"   # this session only; never committed
cd C:\Code\Private\Northstar\northstar-api
.\.venv\Scripts\northstar.exe options market-data sync `
  --database <PATH> `
  --product NIFTY --exchange NSE `
  --expiration 2026-10-27 --strike 22600 --right PUT `
  --start 2026-09-01 --end 2026-10-07
```

- `--product NIFTY --exchange NSE` is the only supported product.
- `--expiration`, `--strike` and `--right` name exactly one contract. The right is exactly `CALL` or `PUT`, never `CE` or `PE`. There is no strike or expiry selection.
- `--start` and `--end` are inclusive trading dates, `YYYY-MM-DD`. `--end` may not be later than the contract's expiration.
- `UPSTOX_ANALYTICS_TOKEN` is required and is the only environment value read. It is sent only as a request header and never printed.
- The command takes the database's operations lock for its whole run.

## 3. What it does

1. It resolves every NIFTY option session in the range. The option session regime begins on **2026-08-03** (09:15 to 15:40 IST). A range that starts earlier, crosses a calendar year that is not loaded, or touches a special session fails closed with exit 4.
2. It requests the contract's candles once, from Upstox's **historical** endpoint only, using the stored instrument key. It reads no clock and never calls the current-day endpoint.
3. Each candle becomes one canonical option daily bar, stamped at its session's close. Its volume is the provider volume divided exactly by the stored exchange lot, so it counts option contracts.
4. The provider's raw open interest for each candle is stored beside the bar, unconverted and with no unit claimed.
5. The bars and their open interest are written in one transaction, or nothing is written.

## 4. Sessions without a provider candle

Listed NIFTY options often go a whole session without a trade, and Upstox then
returns no candle for that session. Such a session is valid. It is reported,
never filled:

```
OPTIONS MARKET DATA SYNC
Provider: upstox
Contract: NIFTY@NSE 2026-10-27 22600 CALL
Date range: 2026-09-15 .. 2026-09-22 (trading dates, historical endpoint only)
SYNC: COMPLETED
Sessions in range: 6
Sessions with a persisted daily bar: 5 (identical bars already stored count as persisted)
Sessions without a provider candle: 1
Missing trading dates: 2026-09-18
Provider open-interest records persisted: 5 (raw provider values, not normalized)
Finality: not assessed
```

- No bar and no open-interest row exist for a missing date. Nothing is fabricated: no zero bar, carried-forward bar or settlement price.
- A missing date is **not** proof that nothing traded. Upstox records no listing history, so it may predate the contract's listing, or be a candle not published yet.
- A range with no candle at all still exits 0, with every session listed as missing and nothing stored.
- **Venue-today:** the historical endpoint does not serve the current venue date. A range ending today reports today as a missing date. Sync it again on a later day.

## 5. Repeat syncs

Repeating a sync is safe. Identical bars and open interest already stored are
accepted unchanged and count as persisted. A later sync may add a candle that
the provider exposes afterwards. A candle that differs from the stored one, in
any price, volume or open interest, is a conflict: exit 5, with nothing written
and nothing changed.

## 6. Expired contracts

Upstox stops accepting an expired contract's instrument key (`UDAPI100011`). The
stored listing still resolves locally, but the provider refuses the request
(exit 6, `UpstoxInvalidInstrumentKeyError`). Acquire a contract's sessions
before it expires; afterwards they may be unobtainable from Upstox.

## 7. No finality

A stored candle records what Upstox returned at the time of the sync. The
command makes no claim that it will not be revised, and there is no option
finality, final-through setting, evidence collector or scheduled option
operation.

## 8. Exit codes

| Exit | Meaning | Operator action |
|------|---------|-----------------|
| 0 | Completed, including sessions without a provider candle | None. |
| 2 | Invalid input: product, exchange, expiration, strike, right, a date, a reversed range, or `--end` after the expiration | Fix the command. |
| 3 | `UPSTOX_ANALYTICS_TOKEN` missing or not a single line, or an unusable database path | Fix the environment or path. |
| 4 | No stored listing for the contract; a provider candle on a date that is not an option session; or a range before 2026-08-03, in an unloaded year or across a special session | Run `options instruments sync` while listed, or narrow the range. |
| 5 | A stored bar or open interest differs, a corrupt stored listing or market data, SQLite busy, or another writer holds the operations lock | Retry when the other writer finishes; investigate conflicts. |
| 6 | Upstox failure: access block, authentication, expired or invalid instrument key, unavailable, malformed candle or open interest, or a volume that is not a whole number of lots | Retry later; an expired key cannot be retried. |
| 1 | Internal defect, such as captured open interest not matching the bars | Report it; nothing was stored. |

Every failure after acquisition starts ends with `Sync stopped. Nothing from
this range was stored.`
