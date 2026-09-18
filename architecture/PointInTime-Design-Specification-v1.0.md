# PointInTime Design Specification v1.0

## 1. Business Purpose

PointInTime represents a specific temporal location having business meaning.

It exists to express when a relevant business fact, observation, action, state, or historical record occurred.

PointInTime belongs in the Foundation because it is a reusable temporal primitive that applies across multiple bounded contexts.

Natural consumers include:

- Market Observation,
- Quote,
- Tick,
- Trade,
- Order,
- Portfolio history,
- Analytics,
- Risk.

PointInTime provides common temporal meaning without allowing higher layers to define competing local concepts for the same business question.

## 2. Business Meaning

PointInTime answers:

> When?

It represents one unique, globally unambiguous, specific temporal location using one canonical temporal business meaning.

PointInTime is distinct from Timeframe.

Timeframe answers:

> What interval?

PointInTime identifies a location in time. Timeframe identifies an interval scale.

These concepts complement one another:

```text
Timeframe
    |
    v
Interval identity

PointInTime
    |
    v
Specific temporal location
```

Neither replaces the other.

## 3. Relationship to Timeframe

Both Timeframe and PointInTime belong to the Temporal Foundation family.

Timeframe represents temporal interval identity.

PointInTime represents a specific temporal location.

A business concept may require:

- PointInTime alone, when it is meaningful at a specific temporal location,
- Timeframe alone, when it represents interval identity,
- or both, when it represents an interval-oriented fact with a specific temporal context.

PointInTime does not represent duration, and Timeframe does not represent a specific temporal location.

## 4. Business Responsibilities

PointInTime is responsible for:

- expressing one specific temporal location,
- providing reusable temporal business meaning,
- supporting temporal consistency across bounded contexts,
- allowing business facts to be associated with a clear temporal context.

PointInTime does not represent:

- duration,
- interval identity,
- chronology,
- workflow state,
- business decisions,
- market observations,
- technical storage representation,
- Listing,
- Instrument,
- Exchange,
- Order,
- Trade,
- or Portfolio state.

## 5. Business Invariants

The following invariants must always hold:

- Every PointInTime represents one specific temporal location.
- PointInTime is immutable.
- PointInTime has value semantics.
- PointInTime has no independent identity.
- PointInTime preserves temporal business meaning.
- PointInTime remains distinct from interval identity.
- PointInTime remains independent of any one bounded context.
- PointInTime does not redefine Timeframe or higher-level business concepts.

## 6. Reuse Analysis

PointInTime belongs in the Foundation because its business meaning is reusable across architectural domains.

### Market Observation

Market Observation uses PointInTime to preserve when market state was observed.

### Quote

Quote uses PointInTime to preserve when quoted market state was reported.

### Tick

Tick uses PointInTime to preserve fine-grained temporal observation context.

### Trade

Trade uses PointInTime to preserve when an execution outcome occurred.

### Order

Order uses PointInTime to preserve when order-related business facts become relevant.

### Portfolio History

Portfolio history uses PointInTime to preserve historical business context.

### Analytics

Analytics uses PointInTime to compare and interpret business facts across time.

### Risk

Risk uses PointInTime to evaluate exposure and market context at meaningful temporal locations.

Higher layers compose PointInTime rather than redefining it. This preserves one canonical temporal business concept across the platform.

## 7. Validation Rules

Business validation requires:

- temporal value exists,
- temporal value represents one specific point in time,
- temporal value contains complete temporal meaning,
- temporal value includes an explicit UTC designation or numeric UTC offset,
- temporal meaning is unambiguous,
- the value is neither an interval nor a duration.

Validation must preserve the distinction between a specific temporal location and an interval identity.

The following business representations are invalid:

- a local date-time without an explicit offset,
- a date-only value,
- a time-only value,
- a Timeframe value,
- a duration value,
- a relative temporal expression,
- a timezone identifier without one specific temporal location,
- multiple temporal locations.

## 8. Normalization

PointInTime has one canonical business representation for each specific temporal location.

Approved normalization includes:

- trimming surrounding whitespace,
- conversion of explicit offsets to the equivalent UTC value,
- canonical UTC representation,
- removal of insignificant fractional-second zeroes.

Normalization must preserve the represented temporal location exactly and ensure:

- one canonical temporal business meaning,
- deterministic comparison,
- no ambiguity caused by equivalent temporal presentation.

Normalization must not:

- change business meaning,
- convert PointInTime into Timeframe,
- merge different temporal locations,
- normalize relative expressions,
- normalize timezone names,
- or introduce workflow-specific normalization.

## 9. Equality

PointInTime equality is based on business value.

Equality is evaluated after canonical normalization.

Two PointInTime values are equal when they represent the same canonical temporal location, regardless of their original explicit offset representation.

Different temporal locations are not equal.

Equality does not depend on:

- object identity,
- the business concept that composes the PointInTime,
- presentation variation,
- or workflow context.

## 10. Examples

### Valid Examples

- `2026-08-17T09:30:00Z`: represents one specific temporal location.
- `2026-08-17T14:30:00+05:00`: represents the same PointInTime as `2026-08-17T09:30:00Z`.
- The temporal location at which a Quote was reported.
- The temporal location at which a Trade occurred.
- The temporal location associated with a Tick observation.
- The temporal location used to reconstruct a Portfolio historical state.
- The temporal location at which an Order became relevant to a workflow.

Each example represents one specific temporal business context.

### Invalid Examples

- `1m`: represents Timeframe interval identity, not PointInTime.
- `1h`: represents Timeframe interval identity, not PointInTime.
- `2026-08-17T09:30:00`: lacks an explicit UTC designation or numeric UTC offset.
- `2026-08-17`: represents a date only, not one specific temporal location.
- `09:30:00Z`: represents a time only, not one specific temporal location.
- `UTC`: represents a timezone identifier, not one specific temporal location.
- “the next trading session”: represents a future interval or workflow context, not one specific temporal location.
- “during the day”: represents an interval, not one specific temporal location.
- “current market state”: lacks a specific temporal location.
- a duration: represents elapsed temporal extent, not one temporal location.
- a market observation: represents higher-level business meaning that may compose PointInTime.

## 11. Future Dependency

PointInTime naturally supports:

- Market Observation,
- Quote,
- Tick,
- Trade,
- Order,
- Portfolio history,
- Analytics,
- Risk.

PointInTime remains reusable and independent of each dependent concept.

It does not own or control the lifecycle, behavior, state, or identity of higher-level concepts.

## 12. Boundary Review

PointInTime owns only:

- specific temporal business meaning.

PointInTime does not own:

- Timeframe,
- Listing,
- Instrument,
- Exchange,
- Orders,
- Trades,
- Market Observation,
- workflow state,
- or implementation representation.

PointInTime remains a Foundation Value Object.

## 13. DDD Review

PointInTime provides:

- explicit business meaning,
- immutability,
- value semantics,
- no independent identity,
- broad bounded-context reuse,
- low coupling,
- high cohesion,
- and a canonical temporal vocabulary.

It is correctly classified as a Foundation Value Object because it represents a reusable primitive concept rather than a workflow-specific concern or Core Domain entity.

## Approval Summary

PointInTime is the approved Foundation Value Object representing a specific temporal location with business meaning.

It complements Timeframe:

- Timeframe represents interval identity.
- PointInTime represents specific temporal location.

PointInTime provides the canonical reusable answer to the business question:

> When?

PointInTime now has approved business representation, canonical normalization, validation, and equality semantics.

PointInTime implementation may proceed.
