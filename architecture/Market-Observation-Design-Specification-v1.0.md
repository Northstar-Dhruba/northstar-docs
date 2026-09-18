# Market Observation Design Specification v1.0

## 1. Business Purpose

Market Observation represents an observation of market state associated with a Listing and a temporal context.

It exists to preserve what the market reported about a specific market manifestation at a particular point in time or across a defined interval.

Market Observation is the foundational business concept of the Market Data family because future observation types share common meaning:

- market state is observed,
- the observation belongs to a Listing,
- temporal context is intrinsic,
- and the observation supports downstream analysis and workflows.

Market Observations exist independently of:

- Orders,
- Trades,
- Positions,
- Portfolios.

Those concepts consume market observations but do not define or own the observed market state.

## 2. Business Meaning

Market Observation records what the market reported.

It represents observed market state, not business intent or completed business action.

Market Observation does not:

- execute business actions,
- create business decisions,
- authorize Orders,
- execute Trades,
- own workflow state,
- or manage Portfolio or Position state.

## 3. Architectural Classification

Market Observation is:

- an Observation,
- immutable,
- time-bound,
- Listing-centric,
- and foundational to the Market Data family.

Market Observation is not:

- one generic Entity,
- one generic Value Object,
- a Domain Event,
- or a Read Model.

It is an observation-oriented business concept whose meaning depends on the observed market state, its Listing context, and its temporal context.

## 4. Business Responsibilities

Market Observation is responsible for:

- preserving observed market state,
- preserving the associated Listing context,
- preserving temporal meaning,
- supporting downstream business workflows,
- supporting historical analysis,
- supporting reproducible interpretation of market conditions.

Market Observation does not own:

- Instrument identity,
- Exchange identity,
- Listing identity,
- ListingStatus,
- Tradability,
- Price ownership,
- Money ownership,
- Order execution,
- Trade execution,
- Position state,
- or Portfolio state.

Instrument and Exchange meaning are reached through Listing. Market Observation does not redefine any of them.

## 5. Business Invariants

The following invariants must always hold:

- Every Market Observation references exactly one Listing.
- Every Market Observation has temporal meaning.
- Every Market Observation represents one observed market state.
- Every Market Observation preserves its Listing context.
- Every Market Observation is immutable.
- Every Market Observation preserves historical meaning.
- Every Market Observation remains distinct from business decisions and actions.
- Market Observation does not redefine Instrument, Exchange, or Listing meaning.

## 6. Relationships

The primary relationship is:

```text
Listing
   |
   v
Market Observation
```

Market Observation depends directly on Listing because market state must be interpreted within a specific market manifestation.

Instrument and Exchange are reached through Listing:

- Instrument provides intrinsic tradable meaning.
- Exchange provides venue meaning.
- Listing provides the market-specific context.
- Market Observation records observed state within that context.

## 7. Downstream Relationships

```text
Market Observation
        |
        v
      Order
        |
        v
  Trade Analysis
        |
        v
     Position
        |
        v
    Portfolio
        |
        v
    Analytics
        |
        v
       Risk
```

Market Observation supports these concepts but does not own them.

- Order may use observations to inform execution decisions.
- Trade Analysis may compare executions with observed market state.
- Position may use observations for valuation and exposure analysis.
- Portfolio may consume observation-derived position information.
- Analytics uses observations as analytical input.
- Risk uses observations to evaluate market exposure and uncertainty.

## 8. Temporal Semantics

Time is intrinsic to Market Observation.

Without temporal context, an observation cannot express when the market reported the observed state or which interval the state represents.

### Point-in-Time Observations

A point-in-time observation describes market state at a specific moment.

Examples include:

- a quote,
- a tick,
- a point-in-time market depth observation.

### Interval Observations

An interval observation summarizes market state over a defined period.

Examples include:

- OHLC Bars,
- interval volume observations,
- interval market activity summaries.

### Sequencing

Temporal ordering allows observations to be interpreted as part of a historical sequence.

Sequence meaning supports:

- historical reconstruction,
- comparison,
- trend analysis,
- and detection of changes in observed market state.

### Historical Context

A Market Observation preserves what was reported in its original temporal context.

Historical interpretation must not silently replace the original observation with a later market state.

### Relationship to Timeframe

Timeframe provides the approved temporal interval identity for interval observations.

Timeframe identifies the interval scale. Market Observation provides the observed market state associated with that interval.

Timeframe does not replace the observation’s temporal meaning, and Market Observation does not redefine Timeframe.

## 9. Validation Rules

Business validation requires:

- Listing must exist.
- Temporal context must exist.
- Observed market state must exist.
- Observation meaning must not be ambiguous.
- The observation must belong to one Listing context.
- The temporal meaning must distinguish point-in-time and interval observation context where applicable.
- The observation must represent reported market state rather than an unexecuted decision or workflow instruction.

## 10. Normalization

Market Observation requires business normalization where normalization produces a canonical interpretation of observed state.

Normalization should preserve:

- the Listing context,
- the temporal context,
- the observation type,
- the meaning of the reported market state,
- and historical accuracy.

Normalization must not:

- alter the reported business meaning,
- merge observations from different Listings,
- collapse distinct temporal contexts,
- or replace an original observation with a later interpretation.

Any observation-specific normalization rules should be defined by the relevant specialized observation design.

## 11. Immutability

Market Observation must be immutable because it represents a historical report of market state.

Immutability supports:

- historical correctness,
- reproducibility,
- analytical determinism,
- consistent comparison,
- and reliable reconstruction of past market conditions.

If a reported observation is later corrected, superseded, or invalidated, that change should be represented through a separate approved business concept or observation relationship rather than silently mutating the original meaning.

The correction and provenance model requires separate design if future requirements establish it.

## 12. Examples

### Valid Examples

#### Point-in-Time Quote Observation

A quote observation associated with an AAPL Listing on NASDAQ at a specific moment.

Business meaning:
The market reported a particular state for that Listing at that time.

#### Interval Observation

An OHLC observation associated with an AAPL Listing on NASDAQ over a defined one-day Timeframe.

Business meaning:
The observation summarizes reported market state across the approved interval.

#### Market Depth Observation

An observation of visible market depth for a Listing at a specific moment.

Business meaning:
The market reported a depth state for that Listing at that time.

### Invalid Examples

- An Order instruction: this represents business intent, not observed market state.
- A completed Trade: this represents an execution outcome, not Market Observation by default.
- A Position balance: this represents holdings or exposure, not market state.
- A Portfolio valuation: this represents an aggregate business view, not one observation.
- An observation without Listing context: its market manifestation is ambiguous.
- An observation without temporal context: its historical meaning is incomplete.
- A future prediction presented as reported market state: it is analysis or decision support, not an observation.
- An observation that combines data from multiple Listings without preserving their separate contexts: market meaning is ambiguous.

## 13. Future Specializations

Future observation types specialize Market Observation.

Approved examples include:

- Quote,
- OHLC Bar,
- Tick,
- Order Book,
- Auction Observation.

These are specialized Market Observations.

They should preserve the approved foundational meaning:

- association with Listing,
- intrinsic temporal context,
- immutable observed state,
- historical correctness,
- and support for downstream workflows.

Specialized observations may introduce business semantics specific to their observation type, but they must not redefine Market Observation or move Listing ownership elsewhere.

## 14. DDD Review

Market Observation provides:

- explicit business meaning,
- immutable historical semantics,
- temporal correctness,
- Listing-centric ownership,
- high cohesion around observed market state,
- low coupling to downstream workflows,
- and future extensibility through specialization.

It does not become a generic Entity, generic Value Object, Domain Event, or Read Model.

Its boundary is focused on preserving observations. Workflow decisions, execution outcomes, holdings, aggregate portfolio views, and analytical conclusions remain separate concepts.

## Approval Summary

Market Observation is the foundational business concept of the Market Data family.

It is an immutable, time-bound observation of market state associated directly with Listing.

Future Quote, OHLC Bar, Tick, Order Book, and Auction Observation concepts should specialize Market Observation without redefining its core business meaning.
