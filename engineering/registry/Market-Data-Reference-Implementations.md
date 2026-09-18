# Market Data Reference Implementations Registry

This registry records the approved Reference Market Observation family in the Northstar Market Data bounded context. Each entry represents a reviewed, approved, and designated baseline for future Market Observations in the same architectural family.

The Market Data bounded context now contains the approved Reference Market Observation family:

- Quote, the Reference Point Market Observation.
- OHLCBar, the Reference Interval Market Observation.
- Tick, the Reference High-Frequency Point Market Observation.
- OrderBook, the Reference Market-Depth Observation.

These implementations establish the engineering reference patterns for future Market Observations. Each observation remains within the approved Market Observation architecture and owns one observation-specific state Value Object.

The Foundation and Core Domain registries remain independent. This registry records only approved Market Data reference implementations.

---

## Reference Family Overview

The approved Market Observation families are:

### Point Observations

- Quote
- Tick

### Interval Observations

- OHLCBar

### Market-Depth Observations

- OrderBook

## Reference Market Observations

The registry is observation-centric. Supporting Observation State Value
Objects are recorded with their owning Market Observation and are not
independent Reference Market Observations.

| Identifier | Observation | Classification                                    | Supporting Observation State | Status   | Version |
| ---------- | ----------- | ------------------------------------------------- | ---------------------------- | -------- | ------- |
| RMO-001    | Quote       | Reference Point Market Observation                | QuotedMarketState            | Approved | v1.0    |
| RMO-002    | OHLCBar     | Reference Interval Market Observation             | BarState                     | Approved | v1.0    |
| RMO-003    | Tick        | Reference High-Frequency Point Market Observation | TickState                    | Approved | v1.0    |
| RMO-004    | OrderBook   | Reference Market-Depth Observation                | OrderBookState               | Approved | v1.0    |

### RMO-001 — Quote

| Field                        | Value                              |
| ---------------------------- | ---------------------------------- |
| Version                      | v1.0                               |
| Reference Observation        | Quote                              |
| Classification               | Reference Point Market Observation |
| Supporting Observation State | QuotedMarketState                  |
| Status                       | Approved                           |
| Repository                   | northstar-core                     |
| Package                      | market_data                        |
| Design                       | Approved                           |
| Implementation               | Approved                           |
| Reference Contract Suite     | Approved                           |

**Reference Contract Test Suite**

`tests/market_data/test_quote.py`

Quote establishes the Reference Point Market Observation pattern. It composes Listing, PointInTime, and QuotedMarketState.

---

### RMO-002 — OHLCBar

| Field                        | Value                                 |
| ---------------------------- | ------------------------------------- |
| Version                      | v1.0                                  |
| Reference Observation        | OHLCBar                               |
| Classification               | Reference Interval Market Observation |
| Supporting Observation State | BarState                              |
| Status                       | Approved                              |
| Repository                   | northstar-core                        |
| Package                      | market_data                           |
| Design                       | Approved                              |
| Implementation               | Approved                              |
| Reference Contract Suite     | Approved                              |

**Reference Contract Test Suite**

`tests/market_data/test_ohlc_bar.py`

OHLCBar establishes the Reference Interval Market Observation pattern. It composes Listing, PointInTime, Timeframe, and BarState.

---

### RMO-003 — Tick

| Field                        | Value                                             |
| ---------------------------- | ------------------------------------------------- |
| Version                      | v1.0                                              |
| Reference Observation        | Tick                                              |
| Classification               | Reference High-Frequency Point Market Observation |
| Supporting Observation State | TickState                                         |
| Status                       | Approved                                          |
| Repository                   | northstar-core                                    |
| Package                      | market_data                                       |
| Design                       | Approved                                          |
| Implementation               | Approved                                          |
| Reference Contract Suite     | Approved                                          |

**Reference Contract Test Suite**

`tests/market_data/test_tick.py`

Tick establishes the Reference High-Frequency Point Market Observation pattern. It composes Listing, PointInTime, and TickState.

---

### RMO-004 — OrderBook

| Field                        | Value                              |
| ---------------------------- | ---------------------------------- |
| Version                      | v1.0                               |
| Reference Observation        | OrderBook                          |
| Classification               | Reference Market-Depth Observation |
| Supporting Observation State | OrderBookState                     |
| Status                       | Approved                           |
| Repository                   | northstar-core                     |
| Package                      | market_data                        |
| Design                       | Approved                           |
| Implementation               | Approved                           |
| Reference Contract Suite     | Approved                           |

**Reference Contract Test Suite**

`tests/market_data/test_order_book.py`

OrderBook establishes the Reference Market-Depth Observation pattern. It composes Listing, PointInTime, and OrderBookState.

---

## Engineering Summary

All Reference Market Observations share:

- immutable observation semantics,
- Listing composition,
- PointInTime composition,
- dedicated Observation State ownership,
- value equality,
- hashing,
- deterministic representation,
- aggregate ownership boundaries,
- Reference Contract Suites,
- and engineering consistency through one common implementation pattern.

OHLCBar additionally composes Timeframe because it is an interval observation.

## Observation State Summary

The following supporting Observation State Value Objects are owned by their
corresponding Market Observations. They are not independent Reference Market
Observations.

```text
QuotedMarketState
        |
        v
Reference Point Observation State

BarState
        |
        v
Reference Interval Observation State

TickState
        |
        v
Reference High-Frequency Point Observation State

OrderBookState
        |
        v
Reference Market-Depth Observation State
```

## Architecture Summary

The approved Market Observation hierarchy is:

```text
Market Observation
    |
    +-- Quote
    |      |
    |      +-- QuotedMarketState
    |
    +-- Tick
    |      |
    |      +-- TickState
    |
    +-- OHLCBar
    |      |
    |      +-- BarState
    |
    +-- OrderBook
           |
           +-- OrderBookState
```

Market Observation remains the abstract business concept.

No Market Data architectural layer was introduced. Each Market Observation
owns exactly one Observation State Value Object, and each Observation State
Value Object remains subordinate to its owning Market Observation.

## Architecture Consistency Review

- Foundation remains unchanged.
- Core Domain remains unchanged.
- Market Observation remains unchanged.
- Quote remains the Reference Point Market Observation.
- Tick remains the Reference High-Frequency Point Market Observation.
- OHLCBar remains the Reference Interval Market Observation.
- OrderBook remains the Reference Market-Depth Observation.
- No Market Data architectural layer exists.
- No implementation behavior changed.

## Registry Status

All four Reference Market Observations and all four Observation State Value Objects are approved and recorded in this registry.
