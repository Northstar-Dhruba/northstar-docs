# Historical Futures Dataset Specification v1.0 (R-1A)

Status: **specification; the manifest contract is implemented in development
(R-1A), not released.** No historical dataset exists, none is authorized, and
no importer is implemented. Everything described as R-1B, R-1C or R-1D is
future work.

- Implementation:
  `northstar-application/src/northstar_application/application_services/futures_historical_dataset_manifest.py`.
- Builds on the per-candle `fdc1` identity
  (`futures_daily_candle_identity.py`) and the R-0 evaluator
  (`evaluate_futures_recommendation_policy.py`).
- Changes no Core type, no Infrastructure schema, no adapter and no operation.

**Location.** `architecture/` holds the design specifications. The README's
Architecture row already links the folder, so the index is unchanged.

## 1. Purpose

R-2 will evaluate the F-3/F-4/F-2 recommendation rules on real history. That
needs a dataset whose content and origin can be named and checked. This
specification defines:

- the canonical bar a dataset holds (section 3);
- the contract reference facts it depends on (section 4);
- the immutable manifest that identifies one dataset version (sections 6 to 9);
- the boundary between synthetic and authorized evaluation (section 10).

The research universe starts with NIFTY futures on NSE (`NIFTY@NSE`).

## 2. Per-contract storage

A dataset stores **individual contracts**, each under its own expiration date,
exactly as `futures_ohlcv` already keys them (product, exchange, expiration,
timeframe, instant). Expired contracts need no new model: an expired contract
is another expiration date.

A continuous or back-adjusted series is **not** a substitute. F-3 chooses
between real contracts, the levels F-4 computes must be prices of a contract
that traded, and a roll adjustment would invent prices.

## 3. Canonical price bar

| Field | Rule |
|---|---|
| Product | Exchange product code, e.g. `NIFTY` |
| Exchange | `NSE` |
| Expiration | `YYYY-MM-DD`, the contract's exchange-published expiration date |
| Trading date | The local NSE civil date of the session (Asia/Kolkata) |
| Timeframe | `1d`: the exchange's native daily bar |
| Open, high, low, close | Exact decimals, unadjusted, individual-contract prices |
| Volume | Whole contracts in the stored bar |

Normalization rules:

- **Decimals.** Prices are parsed as exact decimal text; they are never
  converted through floating point. Core's `QuoteValue` keeps every digit.
- **Timestamp.** The bar's instant is the **verified session close** for its
  trading date, resolved from the NSE calendar. It is never the provider's
  timestamp label, which is used only to read the trading date.
- **Volume units.** The source's unit must be declared (`CONTRACTS` or
  `UNDERLYING_UNITS`). Underlying units are converted to contracts only with
  the lot size from a named contract reference, and an inexact division is
  refused. Nothing is converted silently.
- **No fabrication.** A missing candle stays missing. It is never filled,
  interpolated, carried forward or rebuilt from another contract.

The stored bar is exactly what `fdc1` identifies. Open interest, settlement,
provider symbols and raw provider volume are not part of it.

## 4. Contract reference requirements

These facts come from authoritative references, each fact with its source,
following the pattern of the NSE calendar reference
(`nse_futures_calendar_reference.py`). This specification states no values.

| Fact | Needed for |
|---|---|
| Listing date | Telling "not yet listed" apart from a missing bar |
| Expiration date | Contract identity |
| Last tradable session | The contract cutoff. Never derived from the expiration date. |
| Cutoff authority | The source the last session was taken from |
| Lot size and effective date | Converting volume; R-2 point value |
| Tick size and effective date | The F-4 price increment, which ADR-018 requires from authoritative NSE specifications |
| Session calendar and special sessions | Session closes, holidays, Muhurat, budget and disaster-recovery sessions |

**Calendar limitation.** The NSE calendar reference loads only 2024, 2025 and
2026. Any other year raises. The 2026-11-08 Muhurat session has no notified
timings and fails closed. A dataset outside the loaded years cannot be
timestamped or checked for completeness until R-1B extends the calendar.

## 5. Conditional R-2 valuation fields

These are **not** R-1A inputs. They become required only when the owner's R-2
valuation conventions need them:

- Daily settlement prices.
- Final settlement price (for an exit at expiry).
- Open interest.
- Traded value.
- Intraday evidence, if the owner chooses to resolve stop-or-target ordering
  within a day (today `UNKNOWN`/`INDETERMINATE`).
- Historical broker and statutory cost references by date.

## 6. Manifest

`FuturesHistoricalDatasetManifest` is an immutable value. It identifies and
validates a dataset; it is not a second price database and never holds
prices.

| Field | Meaning |
|---|---|
| `schema_version` | `northstar.futures-historical-dataset-manifest/1`. Any other value is refused. |
| `dataset_id` | The immutable name of this version: `[A-Za-z0-9][A-Za-z0-9._-]{0,127}` |
| `source.category` | `SYNTHETIC`, `PROVIDER_API`, `EXCHANGE_PRODUCT`, `LICENSED_VENDOR` or `OWNER_SUPPLIED`. A category, never a permission. |
| `source.provider` | The provider or source name |
| `source.release_reference` | The source's file or release version, or explicitly absent |
| `source.acquired_at` | Retrieval or acquisition instant (UTC) |
| `source.rights_reference` | The written permission the data is held under. Required for every real category, forbidden for `SYNTHETIC`. |
| `product`, `contracts` | The product and its contract universe |
| `timeframe` | `1d` only |
| `source_volume_unit` | `CONTRACTS` or `UNDERLYING_UNITS` |
| `normalization_version` | The version of the normalization that produced the bars |
| `coverage` | Bounds, calendar reference, contract reference, expected sessions (section 9) |
| `bars` | Ordered references: expiration, trading date, `fdc1` fingerprint |
| `supersedes` | The dataset ID this version revises, or explicitly absent |

Absent optional values are stored as explicit nulls. The source's missing
metadata is never invented.

## 7. Dataset ID and fingerprints

Four things are kept apart.

| Item | Form | What it proves |
|---|---|---|
| Candle fingerprint | `fdc1-sha256:<hex>` | One bar's contract, trading date, prices and contract volume, unchanged since fingerprinted |
| Dataset fingerprint | `fhd1-sha256:<hex>` | The whole ordered candle inventory and every material source, normalization and coverage field, unchanged |
| Dataset ID | Caller's name | Nothing; it names a version |
| Rights reference | Text | Nothing; it is the caller's statement |

**Dataset fingerprint `fhd1`.** SHA-256 over an injective encoding. Each part
is UTF-8, prefixed by its byte length and `:`, in this order:

```
northstar.futures-historical-dataset/1
<manifest schema>
<category> <provider> <release: "absent" | "present" value>
<normalization version> <timeframe> <source volume unit>
<product code> <exchange code>
"contracts" <count> <expiration>...
"coverage" <first date> <last date>
<calendar reference: absent | present value>
<contract reference: absent | present value>
"expected" absent | present <count> (<expiration> <trading date>)...
"bars" <count> (<expiration> <trading date> <fdc1 fingerprint>)...
```

Contracts, expected sessions and bars are sorted by expiration and then
trading date, so input order never changes the digest. Dates are
`YYYY-MM-DD`, and enums are their stable values.

Deliberately **not hashed**: the dataset ID, `supersedes`, the acquisition
instant and the rights reference. They name, link and attest a version; they
are not its content. Identical content therefore always has one fingerprint,
whatever it is called or when it was fetched.

**What a hash does not prove.** A fingerprint shows only that content equals
content fingerprinted earlier. It does not prove the source was legitimate,
the prices genuine, the data complete or its use licensed. Nothing here is
signed.

## 8. Versioning and revisions

- A dataset is immutable once identified. One dataset ID names exactly one
  fingerprint; `verify_futures_historical_dataset_catalog` refuses a
  collection where it names two.
- A revised or corrected file is a **new dataset** with a new ID. It never
  overwrites the original.
- A revision names the dataset it revises in `supersedes`. A manifest can
  never supersede itself.
- `verify_futures_historical_dataset_revision(revision, original)` requires:
  - both manifests supplied;
  - `supersedes` equal to the original's ID;
  - the same product;
  - different content.
- A superseded dataset is never assumed to exist. A `supersedes` that names
  a manifest not supplied stays unresolved. Revision cycles among supplied
  manifests are refused.
- Revision history is never inferred from a filename.

R-1A implements no persistence or revision lookup. Storing manifests and
refusing a reused ID across runs is R-1C.

## 9. Coverage and missing sessions

`FuturesHistoricalDatasetCoverage` holds:
- the first and last trading dates;
- an optional calendar reference;
- an optional contract reference;
- optional expected-session evidence.

**Completeness** is derived, never claimed.

| Evidence | Completeness | Rules |
|---|---|---|
| No expected sessions | `UNVERIFIED` | The bounds must be exactly the first and last stored trading dates. |
| Expected sessions, with a calendar reference and a contract reference | `COMPLETE` or `INCOMPLETE` | Every bar must be an expected session. The bounds are the expected extremes. Every missing session is listed. |

- Expected sessions without both references are refused. A missing
  calendar or contract reference keeps completeness `UNVERIFIED`.
- A gap at the start or end of the coverage is listed, never trimmed.
- Bar counts are reported per contract, including contracts with no bars.
- Duplicate contract sessions are refused. So is one candle fingerprint
  attached to two sessions.

Completeness is relative to the named references. Those references are not
verified by R-1A. Building expected sessions from the verified calendar and
contract references is R-1B.

## 10. Synthetic and authorized evaluation

The R-0 evaluator accepts **only** `SYNTHETIC` evidence, and R-1A leaves that
unchanged. R-1A adds no `AUTHORIZED_DATASET` classification to it.

- `manifest.synthetic_evaluation_evidence()` returns R-0 `SYNTHETIC` evidence
  naming `<dataset_id>@<fingerprint>`. For any real source category it is
  refused.
- `rights_verified` and `is_market_evidence` are always False. A rights
  reference such as `approved` is recorded, never trusted.
- There is no approval field, flag or setting.

**Future activation contract (R-1D).** Authorized-dataset evaluation may be
added only together with:
1. a trusted, owner-controlled record of verified written data-use
   permission, kept outside the manifest the data supplier or caller
   produces;
2. an R-1C import that has validated the dataset against its manifest;
3. completeness derived from R-1B's verified references;
4. a new evaluation classification that refuses evaluation unless all three
   are present.

None of these exist yet.

## 11. Written permission and retention

Before any real data is retained, the owner needs **written** terms that
cover:
- private, single-user research use;
- local retention;
- private backups;
- keeping derived results (fingerprints and statistics);
- what happens to retained data when access ends.

Technical access or a public download is not permission.

- G-1 (Upstox written data-use permission) is unresolved.
- No alternative historical dataset is approved.
- Real data must never be committed to a repository or used as a test
  fixture.

## 12. Follow-up work

| Slice | Work |
|---|---|
| R-1B | Verified contract reference (listing, last tradable session with its authority, lot and tick by effective date) as a `FuturesContractCutoffSource`. Calendar years beyond 2024 to 2026. Expected-session derivation. |
| R-1C | Canonical-file importer into a separate evaluation SQLite file using the unchanged `futures_ohlcv` schema. Integrity checks. Manifest storage and ID-reuse refusal. |
| R-1D | End-to-end acceptance: synthetic first, then an authorized dataset once permission exists. Activation contract of section 10. |
