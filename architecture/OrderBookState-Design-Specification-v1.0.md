# OrderBookState Design Specification v1.0

## 1. Business Purpose

OrderBookState is an Order Book-specific Value Object.

It exists to preserve the market-depth business meaning observed by one Order Book for one Listing within one PointInTime context.

OrderBookState expresses visible market-depth structure. It does not represent execution, ownership, accounting state, workflow state, prediction, or business intent.

OrderBookState does not redefine Order Book, Market Observation, Listing, PointInTime, Price, or any other approved Foundation or Core Domain concept.

## 2. Business Meaning

OrderBookState represents the visible market-depth structure reported at one specific PointInTime.

Its meaning concerns observable market liquidity and the organization of that liquidity across market-depth levels. It preserves what was visible in the market-depth observation, not what was executed or what any participant intended to do.

OrderBookState does not represent:

- completed Trades,
- ownership,
- Portfolio state,
- workflow decisions,
- execution instructions,
- prediction,
- or business intent.

OrderBookState is observation meaning, not action meaning.

## 3. Relationship to Order Book

Order Book composes OrderBookState.

Order Book owns:

- Listing context,
- PointInTime context,
- and observation context.

OrderBookState owns only market-depth observation meaning.

Neither concept redefines the other:

- Order Book does not redefine market-depth meaning owned by OrderBookState.
- OrderBookState does not own Listing, PointInTime, or the broader observation context owned by Order Book.

OrderBookState belongs exclusively to Order Book. It is not a reusable Market Data architectural layer.

## 4. Relationship to Price

Price remains the approved reusable Foundation financial primitive.

Price contributes denomination-aware financial meaning to market-depth observation. It provides the approved quotation-value concept through which market-depth levels may express financial valuation.

In Version 1.0, OrderBookState composes one or more Price values.

This composition does not redefine Price or transfer Price ownership into OrderBookState as an independent business responsibility. Future refinement may extend the composition only through approved architecture and design changes.

## 5. Business Responsibilities

OrderBookState is responsible for:

- preserving market-depth observation meaning,
- preserving visible liquidity semantics,
- composing approved quotation-value concepts,
- expressing market-depth interpretation,
- supporting downstream liquidity analysis,
- and preserving the distinction between observed depth and executed activity.

In Version 1.0, the approved quotation-value concepts are composed through one or more Price values.

OrderBookState does not own:

- Listing,
- PointInTime,
- Instrument,
- Exchange,
- ListingStatus,
- Tradability,
- Orders,
- Trades,
- Positions,
- Portfolios,
- workflow state,
- accounting state,
- or business decisions.

## 6. Business Invariants

The following invariants must always hold:

- OrderBookState contains market-depth observation meaning.
- OrderBookState belongs exclusively to Order Book.
- OrderBookState remains distinct from execution and ownership concepts.
- OrderBookState is immutable.
- OrderBookState is value-based and has no independent identity.
- OrderBookState preserves visible market-depth meaning.
- Version 1.0 market-depth quotation meaning is composed through one or more Price values.
- OrderBookState does not own Listing or PointInTime context.
- OrderBookState does not redefine Market Observation or any composed Foundation concept.

## 7. Composition Review

Inside OrderBookState:

- market-depth observation meaning,
- approved quotation-value concepts,
- and, in Version 1.0, one or more Price values.

Order Book provides:

- Listing context,
- PointInTime context,
- and the broader observation context.

Price contributes approved denomination-aware financial meaning and remains a reusable Foundation Value Object.

Currency remains part of Price meaning and is not independently elevated into OrderBookState business ownership.

PointInTime belongs to Order Book because it identifies when the market-depth state was observed. It does not belong inside OrderBookState meaning.

OrderBookState does not contain or redefine Listing, Instrument, Exchange, Orders, Trades, Portfolio, or workflow state.

## 8. Validation Rules

Business validation requires:

- market-depth observation meaning exists,
- market-depth meaning is internally consistent,
- approved quotation-value meaning exists,
- the meaning remains interpretable as visible market depth,
- the meaning remains distinct from execution and action meaning,
- and the state remains consistent with Order Book semantics.

Version 1.0 requires the approved quotation-value meaning to be represented through one or more Price values.

Validation must not introduce workflow rules, execution rules, ownership rules, or assumptions about participant intent.

## 9. Normalization

OrderBookState requires business normalization where normalization produces a canonical interpretation of market-depth meaning.

Normalization should preserve:

- visible market-depth structure,
- quotation-value meaning,
- denomination-aware financial meaning,
- market-depth interpretation,
- and historical observation meaning.

Price normalization remains authoritative for Price itself.

OrderBookState normalization must not:

- redefine Price,
- alter the business meaning of visible depth,
- introduce Listing or PointInTime ownership,
- transform market depth into a Trade,
- or transform OrderBookState into Quote, Tick, or OHLC Bar meaning.

Any additional normalization rules require approved Order Book architecture and design refinement.

## 10. Equality

OrderBookState equality is value-based.

Two OrderBookState values are equal when their canonical market-depth observation meaning is equal.

Equality is based on business value rather than identity, ownership, workflow state, or observation context.

In Version 1.0, canonical quotation-value meaning includes the normalized composition of one or more Price values. Listing and PointInTime are evaluated by Order Book and are not part of OrderBookState's independent business meaning.

## 11. Examples

### Valid Examples

#### Visible Market-Depth Meaning

An OrderBookState representing the visible buy-side and sell-side market-depth structure for a Listing at a specific PointInTime.

Business meaning:
The state preserves what market depth was observable at that moment.

#### Price-Based Depth Meaning

An OrderBookState whose market-depth interpretation is expressed using one or more approved Price values.

Business meaning:
Price provides reusable denomination-aware financial meaning without becoming a new market-depth ownership concept.

### Invalid Examples

- A completed Trade: this represents execution outcome, not market-depth observation meaning.
- An Order instruction: this represents business intent or workflow action, not observed market state.
- A Portfolio valuation: this represents accounting or ownership interpretation, not OrderBookState meaning.
- A Listing with no market-depth meaning: this provides context but does not constitute OrderBookState.
- A PointInTime by itself: this provides temporal context but does not constitute market-depth meaning.
- A prediction of future liquidity: this represents analysis or intent rather than observed market depth.

## 12. Boundary Review

OrderBookState owns only market-depth observation meaning.

OrderBookState does not own:

- Listing,
- PointInTime,
- Instrument,
- Exchange,
- Orders,
- Trades,
- Portfolio,
- workflow state,
- accounting state,
- or business decisions.

Order Book composes OrderBookState with Listing and PointInTime to form the complete Reference Market-Depth Observation.

No new Market Data Value Object architectural layer is introduced. OrderBookState remains exclusively subordinate to Order Book.

## 13. DDD Review

### Explicit Business Meaning

OrderBookState has one explicit purpose: preserving visible market-depth observation meaning.

### Immutable Value Semantics

OrderBookState is immutable and value-based. It has no independent identity and cannot represent a mutable lifecycle.

### Ownership

OrderBook owns the observation context. OrderBookState owns only the specialized market-depth meaning.

### Cohesion

The state is highly cohesive because its responsibilities are limited to interpreting and preserving market-depth meaning.

### Coupling

The state has low coupling. It depends on the approved Price concept for financial meaning and remains independent of Listing, PointInTime, execution, ownership, and workflow concepts.

### Historical Correctness

OrderBookState preserves the market-depth meaning observed at the relevant PointInTime. A later market-depth observation must not silently mutate an earlier state.

### Future Extensibility

Future market-depth refinements may extend OrderBookState only through approved architecture and design changes. Such refinement must preserve its exclusive ownership of market-depth meaning and must not create a generic Market Data Value Object layer.

## Approval Summary

```text
Order Book
      |
      v
OrderBookState
      |
      v
market-depth observation meaning
      |
      v
one or more approved Price values in Version 1.0
```

OrderBookState remains an Order Book-specific Value Object.

Order Book remains responsible for Listing and PointInTime context.

Price remains the approved reusable Foundation financial primitive.

OrderBookState remains immutable, value-based, identity-free, and exclusively responsible for market-depth observation meaning.
