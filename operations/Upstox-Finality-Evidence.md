# Upstox Futures Daily-Candle Finality Evidence

How to collect evidence about when Upstox stops revising a futures daily
candle. This is evidence collection only. It never makes, approves or
recommends a finality decision.

## Purpose

Production finality is `disabled` or `operator-approved`
(`NORTHSTAR_FUTURES_DAILY_BAR_FINALITY`, `NORTHSTAR_FUTURES_FINAL_THROUGH`).
There is no automatic Upstox finality rule, because a same-day Upstox daily
candle has been seen to change between fetches (close and open interest
changed; open, high, low and volume did not). None of these is accepted:
"21:00 means final", "next morning means final", "N hours after the close
means final".

`northstar finality-evidence observe` records what Upstox returns for one
exact contract and trading date at one moment. Repeating it over time builds
the evidence needed to see which fields revise, when, and for how long after
the NSE close. A later report (8G-A2) analyses that evidence. Any automatic
finality policy would be a separate, explicit decision made from it.

## Strict separation from production

The command:

- writes only to the evidence file you name;
- never opens a Northstar database, and has no `--database` option;
- reads only `UPSTOX_ANALYTICS_TOKEN` from the environment -- never
  `NORTHSTAR_DATABASE` or any finality or go-live setting;
- never writes market data, freezes decisions, paper-trades or changes
  `NORTHSTAR_FUTURES_FINAL_THROUGH`;
- never says a candle is final, stable or safe.

## Usage

The trading date must be an NSE futures session according to Northstar's NSE
calendar. A weekend or holiday is refused (exit 2). A date the calendar
cannot resolve, such as the 2026-11-08 Muhurat session whose timings are not
yet loaded, fails closed (exit 6).

**DEVELOPMENT MACHINE** (`C:\Code\Private\Northstar`, PowerShell). The
evidence file lives outside every repository:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\northstar-evidence" | Out-Null
$env:UPSTOX_ANALYTICS_TOKEN = "<token>"   # this session only; never committed
cd C:\Code\Private\Northstar\northstar-api
.\.venv\Scripts\northstar.exe finality-evidence observe `
  --evidence "$env:USERPROFILE\northstar-evidence\upstox-candles.jsonl" `
  --product NIFTY --exchange NSE --expiration 2026-10-27 --trading-date <YYYY-MM-DD>
```

**DEPLOYMENT MACHINE.** Running it there needs a private evidence location
and deployment wiring that does not exist yet. That is deferred until the
provider/legal gate below is cleared; no scheduler is defined.

Run it as many times as you like for the same contract and date. Every run
appends one line. An identical repeat is recorded too: an unchanged candle
over time is itself evidence.

Exit codes follow the CLI:

| Exit | Meaning |
|------|---------|
| 0 | One observation appended, with or without a candle |
| 2 | Invalid input, a non-NSE contract or a non-session date |
| 3 | `UPSTOX_ANALYTICS_TOKEN` missing, or an unusable evidence path |
| 5 | The evidence file ends with an incomplete line; it is left untouched and nothing is appended |
| 6 | Provider or calendar failure (access block, authentication, invalid or expired instrument, unavailable, malformed response, calendar fail-closed); nothing is appended |

## What is recorded

One JSON object per line (UTF-8, keys sorted, newline-terminated), schema
`northstar.upstox-daily-candle-observation/1`:

- `requested_at`, `received_at`: canonical UTC instants read from the CLI's
  clock immediately before the candle request and immediately after the
  response;
- `contract` (`product`, `exchange`, `expiration`) and `trading_date`:
  Northstar identity;
- `instrument_key`, `lot_size`: Upstox provider metadata in force at the
  time, never Northstar identity;
- `request`: interval `days/1`, `from` and `to` (both the trading date);
- `candle`: the provider timestamp verbatim, open, high, low, close, raw
  volume and open interest, or `null` when Upstox answered with no candle;
- `volume_contracts`: raw volume divided by the lot size when that is a whole
  number, otherwise `null` with `volume_contracts_note` explaining why;
- `provider`, `collector`: provenance.

Every number is the exact text Upstox sent (`25010.50` stays `"25010.50"`),
decoded without floats. Open interest is evidence only; Northstar's canonical
daily bar has no open interest. The token, request and response headers,
cookies and the request URL are never recorded.

A response with more than one candle, or a candle for another date, is
refused and nothing is recorded. Provider failures are never recorded.

## One writer per evidence file

The file is append-only: lines are never rewritten, deduplicated or
truncated, and each append is flushed and synced. Plain append mode does not
make concurrent writers safe, so exactly one process may write a given
evidence file at a time. A reader refuses a malformed line and reports an
incomplete final line; it never skips either.

## Token safety

The token is read through the CLI's secret handling, sent only as the
Authorization header and redacted from all output. It is never stored,
logged or shown. A successful candle fetch is not proof that the token is
valid: the historical endpoint has answered even with a placeholder token.

## Limitations

- **Expired contracts.** The contract is resolved through the current
  Upstox instrument master, which lists only tradable contracts, and an
  expired contract's former key is rejected (`UDAPI100011`). Once a contract
  leaves the master its candles can no longer be observed, so revisions after
  that point are invisible and evidence for its last sessions, especially the
  expiry session, may be right-censored.
- **No conclusion from timing.** The last observed revision bounds nothing
  beyond the last observation. Evidence never proves that a candle will not
  change again.

## Provider and legal boundary

Written Upstox permission to retain and use its data is still pending.
Technical API access does not grant retention or use rights. Until it is
granted:

- real evidence files stay private and local, outside every Git repository;
- they are never committed and never become test fixtures (tests use
  synthetic data only);
- persistent collection on a server is a deployment and legal decision
  (see the [Indian Futures Deployment Runbook](Indian-Futures-Deployment-Runbook.md), section 0).
