# Northstar Architecture Handbook v1.0

## Purpose

This handbook is the canonical architecture reference for Northstar Platform Core v1.0.0. It explains how the platform is organized, why its boundaries exist, and which engineering patterns future bounded contexts must follow.

This is an architecture handbook, not an implementation guide. Business meaning, architectural boundaries, and approved design specifications remain authoritative over code. The handbook should be read with the approved ADRs, design specifications, and reference implementation registries.

---

## 1. Platform Vision

Northstar is a domain-driven platform for representing market context, participant intent, execution truth, and ownership interpretation as separate business concerns.

The platform exists to preserve business meaning across the progression from what the market reports, through what a participant intends, to what executes, and finally to how completed activity is interpreted as current ownership.

Northstar is Domain-Driven because the difficult part of the platform is not data transport. It is preserving distinctions between concepts that look related operationally but carry different business meanings:

- a Quote is not an Order,
- an Order is not a Trade,
- a Trade is not a Portfolio,
- and a Portfolio is not Market Data.

Bounded contexts exist to protect these distinctions. Each bounded context owns a coherent business responsibility, vocabulary, consistency boundary, and set of architectural rules. Concepts may be referenced across boundaries, but ownership must remain explicit.

### Design-First Engineering

Northstar follows Design-First Engineering:

- business architecture is established before detailed design,
- detailed design is approved before implementation,
- implementation expresses approved meaning without inventing new rules,
- reference contract suites verify approved behavior,
- registries record approved reference implementations,
- and the CHANGELOG records completed work.

When implementation exposes an unresolved business question, implementation stops. The architectural gap is resolved first.

### Core Architectural Commitments

- One business meaning belongs to one authoritative concept.
- Aggregate Roots own consistency boundaries.
- Value Objects own precise business meaning without independent identity.
- Entities own identity and lifecycle where approved.
- Composition is preferred over duplication.
- Frameworks and infrastructure do not define Core Domain meaning.
- Reference implementations establish patterns; they do not authorize silent redesign.

## 2. Platform Architecture

The completed platform is organized as a progression of business responsibilities:

```text
Foundation
    |
    v
Core Domain
    |
    v
Market Data
    |
    v
Orders
    |
    v
Trades
    |
    v
Portfolio
```

### Foundation

Foundation provides reusable business primitives:

- identity primitives,
- measurement primitives,
- financial primitives,
- temporal primitives,
- and composed financial meaning.

Foundation does not own market observations, participant intent, execution outcomes, or portfolio interpretation.

### Core Domain

Core Domain expresses stable business concepts that organize market participation and participant attribution:

- Instrument,
- Exchange,
- Listing,
- Participant,
- ParticipantIdentity,
- ParticipantReference,
- ListingStatus,
- and Tradability.

Core Domain does not own Orders, Trades, Portfolio state, or Market Data observations.

### Market Data

Market Data preserves what the market reports:

- Quote,
- Tick,
- OHLCBar,
- OrderBook,
- and their observation-specific state Value Objects.

Market Data is observational, Listing-centric, and temporally meaningful. It does not own participant intent, execution outcomes, or ownership interpretation.

### Orders

Orders preserve participant intent and transaction commitment:

- Order is the Reference Aggregate Root.
- OrderIdentity preserves Order aggregate identity.
- OrderStatus preserves Order lifecycle meaning.
- ParticipantReference provides participant attribution.

Orders may use Market Data as context, but do not own Market Data.

### Trades

Trades preserve completed execution truth:

- Trade is the Reference Immutable Aggregate Root.
- TradeIdentity preserves Trade aggregate identity.
- Listing provides market context.
- PointInTime provides execution context.
- ParticipantReference provides attribution.

Trades do not own Orders, Market Data, or Portfolio state.

### Portfolio

Portfolio preserves current ownership interpretation:

- Portfolio is the Reference Ownership Interpretation Aggregate Root.
- PortfolioIdentity preserves Portfolio aggregate identity.
- ParticipantReference provides participant attribution.
- PointInTime provides snapshot or valuation context.
- Position is the Reference Subordinate Entity.

Portfolio consumes completed Trade outcomes and may use Market Data as external valuation context. It does not own Trades or Market Data.

## 3. Business Flow

The complete business progression is:

```text
Observation
    |
    v
Intent
    |
    v
Execution Truth
    |
    v
Ownership Interpretation
```

### Observation

Market Data observes and preserves reported market state.

- Quote preserves point quotation meaning.
- Tick preserves high-frequency point meaning.
- OHLCBar preserves interval meaning.
- OrderBook preserves market-depth meaning.

Observation is not intent and is not execution.

### Intent

Orders express participant intent and transaction commitment. An Order composes Listing, PointInTime, Quantity, Price, ParticipantReference, OrderIdentity, and OrderStatus according to the approved Order design.

Intent is not execution truth. An Order may exist without producing a Trade.

### Execution Truth

Trades preserve completed execution outcomes. A Trade is created through execution and becomes an independent historical aggregate after creation.

Trade is not an Order and does not become a container for Order lifecycle.

### Ownership Interpretation

Portfolio consumes completed Trade outcomes and interprets them as current ownership state. Position preserves Listing-specific ownership interpretation within the Portfolio aggregate.

Portfolio is mutable because current ownership interpretation evolves. Trade remains immutable because historical execution truth must remain stable.

## 4. Bounded Context Responsibilities

### Foundation

**Business responsibility:** Provide reusable primitive business concepts.

**Owns:** Foundation Value Objects such as Symbol, ExchangeCode, Currency, Quantity, Percentage, Price, Money, Timeframe, and PointInTime.

**Does not own:** Instrument, Listing, Market Data, Orders, Trades, Portfolio, workflow, persistence, or infrastructure meaning.

### Core Domain

**Business responsibility:** Express stable market and participant concepts that organize the platform’s business language.

**Owns:** Instrument, Exchange, Listing, Participant, ParticipantIdentity, ParticipantReference, ListingStatus, and Tradability.

**Does not own:** Orders, Trade execution truth, Portfolio ownership state, Market Data observations, authentication, authorization, accounts, or workflow orchestration.

ParticipantReference is an identity association Value Object. It identifies Participant for consuming concepts without owning Participant or Participant lifecycle.

### Market Data

**Business responsibility:** Preserve reported market state in its correct Listing and temporal context.

**Owns:** Quote, Tick, OHLCBar, OrderBook, and their supporting Observation State Value Objects.

**Does not own:** Orders, Trades, Portfolio, Participant identity, workflow, execution, settlement, or accounting.

### Orders

**Business responsibility:** Preserve participant intent, transaction commitment, and Order lifecycle.

**Owns:** Order, OrderIdentity, OrderStatus, and Order aggregate consistency.

**Does not own:** Market Data, Participant lifecycle, Trade execution outcome, Portfolio state, settlement, accounting, or workflow engines.

### Trades

**Business responsibility:** Preserve completed execution truth and historical execution outcomes.

**Owns:** Trade and TradeIdentity, together with Trade aggregate consistency.

**Does not own:** Orders, Participant lifecycle, Market Data, Portfolio, settlement, accounting, or workflow.

### Portfolio

**Business responsibility:** Preserve current ownership interpretation for one Participant context.

**Owns:** Portfolio, PortfolioIdentity, and subordinate Positions. Portfolio protects the invariant of at most one Position interpretation for each Listing within the Portfolio.

**Does not own:** Orders, Trades, Market Data, Participant lifecycle, settlement, accounting, risk, performance, or workflow.

## 5. Engineering Patterns

Northstar uses distinct engineering families. A family is defined by business responsibility, not by superficial data shape.

### Foundation Value Objects

**Business purpose:** Represent reusable business primitives.

**Engineering philosophy:** Foundation Value Objects are immutable, value-based, framework-independent, validated, normalized where approved, and reusable across bounded contexts.

**Approved references:**

- Symbol
- ExchangeCode
- Currency
- Quantity
- Percentage
- Price
- Money
- Timeframe
- PointInTime

### Observation State Value Objects

**Business purpose:** Preserve specialized market observation meaning.

**Engineering philosophy:** Observation State belongs to one Market Observation and owns only the specialized meaning of that observation. It does not own Listing or temporal context.

**Approved references:**

- QuotedMarketState
- TickState
- BarState
- OrderBookState

### Lifecycle State Value Objects

**Business purpose:** Preserve the lifecycle condition of an owning aggregate.

**Engineering philosophy:** Lifecycle State is subordinate to its aggregate. It is not a generic workflow state and does not own lifecycle transitions independently.

**Approved reference:**

- OrderStatus

### Aggregate Identity Value Objects

**Business purpose:** Preserve stable, opaque identity for an Entity or Aggregate Root.

**Engineering philosophy:** Aggregate identity is immutable, value-based, stable, and independent of business attributes, lifecycle meaning, participant attribution, and Market Data. Identity formats and persistence encodings are not inferred by the family.

**Approved references:**

- OrderIdentity
- ParticipantIdentity
- TradeIdentity
- PortfolioIdentity

### Identity Association Value Objects

**Business purpose:** Preserve cross-aggregate attribution to an identity-bearing business concept.

**Engineering philosophy:** An association Value Object identifies another business concept without owning it, duplicating its meaning, or importing its lifecycle.

**Approved reference:**

- ParticipantReference

### Reference Subordinate Entities

**Business purpose:** Represent mutable business concepts that are meaningful only inside an owning Aggregate Root.

**Engineering philosophy:** Subordinate Entities do not create independent aggregate boundaries unless separately approved. Their identity may be local to the owning aggregate and may remain unresolved until explicitly designed.

**Approved reference:**

- Position

Position is subordinate to Portfolio, composes Listing, inherits participant and temporal context from Portfolio, and does not introduce PositionIdentity in Version 1.

### Reference Aggregate Roots

**Business purpose:** Protect consistency boundaries around independent business concepts.

**Engineering philosophy:** Aggregate Roots own their identity, composition, invariants, and lifecycle or historical consistency according to their business meaning.

**Approved references:**

- Order, the mutable intent Aggregate Root.
- Trade, the immutable historical truth Aggregate Root.
- Portfolio, the mutable ownership interpretation Aggregate Root.

## 6. Reference Aggregate Roots

### Order: Mutable Intent

Order represents participant intent and transaction commitment. It is mutable through its approved aggregate lifecycle, although Version 1.0 does not implement lifecycle transition APIs.

Order owns OrderStatus and composes OrderIdentity, Listing, PointInTime, Quantity, Price, and ParticipantReference. Order equality is based only on OrderIdentity.

Order does not own Market Data, Trade, Portfolio, Participant, or workflow infrastructure.

### Trade: Immutable Historical Truth

Trade represents a completed execution outcome. It is immutable after creation because execution truth must remain historically stable and reproducible.

Trade owns TradeIdentity and composes Listing, PointInTime, Quantity, Price, and ParticipantReference. Trade equality is based only on TradeIdentity.

Trade becomes an independent historical aggregate after creation. It does not own Orders, Market Data, Portfolio, settlement, accounting, or workflow.

### Portfolio: Mutable Ownership Interpretation

Portfolio represents current ownership interpretation for one Participant context. It is mutable because completed Trade outcomes change current holdings and Listing-specific ownership interpretations.

Portfolio owns PortfolioIdentity and subordinate Positions. It composes ParticipantReference and PointInTime. Its aggregate consistency includes the rule that a Portfolio contains at most one Position interpretation for each Listing.

Portfolio consumes Trade outcomes and may use Market Data as external valuation context. It does not own Trade or Market Data.

## 7. Reference Value Objects

### Foundation Value Objects

Use Foundation Value Objects for reusable primitives whose meaning is not owned by a single bounded context:

- Symbol and ExchangeCode for identity primitives.
- Quantity for measurable magnitude.
- Currency, Price, and Money for financial meaning.
- Timeframe and PointInTime for temporal meaning.
- Percentage for normalized proportional meaning.

A new Foundation Value Object requires approval that its meaning is genuinely reusable and primitive.

### Aggregate Identity Value Objects

Use Aggregate Identity Value Objects when an Entity or Aggregate Root requires stable opaque identity that must remain independent of business attributes.

Approved references:

- OrderIdentity
- ParticipantIdentity
- TradeIdentity
- PortfolioIdentity

Do not encode Listing, ParticipantReference, Quantity, Price, lifecycle, Market Data, or Portfolio state into an aggregate identity.

### Observation State Value Objects

Use Observation State Value Objects only when a Market Observation owns specialized observation meaning that must remain separate from Listing and temporal context.

The owning observation remains responsible for Listing and temporal context. The state Value Object remains responsible for observation meaning.

### Lifecycle State Value Objects

Use Lifecycle State Value Objects for aggregate-specific lifecycle meaning, not for generic workflow orchestration. The state belongs to its owning aggregate and must not become a shared status framework without explicit architecture approval.

### Identity Association Value Objects

Use Identity Association Value Objects when one aggregate needs attribution to an independent identity-bearing concept without taking ownership of that concept.

ParticipantReference is the approved example. It identifies Participant without owning Participant, Participant lifecycle, or Participant profile meaning.

## 8. Entity Design

### Aggregate Root Versus Subordinate Entity

An Aggregate Root owns a consistency boundary and is independently meaningful within its bounded context. A subordinate Entity has mutable business meaning but exists only within its owning Aggregate Root.

The reference relationship is:

```text
Portfolio
    |
    +-- Position
```

### Portfolio and Position

Portfolio owns the Position collection and protects aggregate consistency. Position represents one Listing-specific ownership interpretation.

Position composes Listing but does not compose ParticipantReference or PointInTime. Those contexts are inherited from Portfolio at the business level. Position does not own Trade, Orders, Market Data, Risk, Performance, Settlement, or Workflow.

### Why Position Has No PositionIdentity

PositionIdentity is not required for Version 1. The local business identity of a Position is provided by:

```text
Portfolio + Listing
```

Within one Portfolio, at most one Position represents the current ownership interpretation for a given Listing. A separate PositionIdentity would add a second identity concept without approved independent business meaning.

If future business requirements create multiple simultaneous Position interpretations for one Listing within a Portfolio, that is an architectural change requiring review. It must not be solved by implementation invention.

## 9. Aggregate Ownership

Aggregate ownership means that one concept is authoritative for a business boundary and its consistency rules.

Approved ownership examples:

- Portfolio owns Position.
- Order owns OrderStatus.
- Order composes OrderIdentity for its aggregate identity.
- Trade composes TradeIdentity for its aggregate identity.
- Portfolio composes PortfolioIdentity for its aggregate identity.
- ParticipantReference never owns Participant.
- Observation State Value Objects own observation meaning but not Market Observation context.

Ownership is not the same as reference. A concept may reference another concept without owning it. ParticipantReference identifies Participant without importing Participant ownership; Portfolio consumes Trade outcomes without owning Trades.

## 10. Boundary Rules

Northstar bounded contexts follow explicit dependency direction.

### Market Data and Orders

Market Data provides context for Orders. Market Data never owns Orders, participant intent, or workflow state.

```text
Market Data
    |
    v
Orders
```

### Orders and Trades

Orders express intent. Trades preserve completed execution truth. Trade is created through execution of one or more Order relationships, then becomes an independent historical aggregate. Neither aggregate owns the other after creation.

### Trades and Portfolio

Trades provide immutable execution outcomes. Portfolio interprets those outcomes as current ownership state.

```text
Trades
    |
    v
Portfolio
```

Trades never own Portfolio. Portfolio never owns Trades.

### Portfolio and Market Data

Portfolio may consume Market Data for valuation context. Portfolio does not own Quote, Tick, OHLCBar, or OrderBook.

```text
Market Data
    |
    v
Portfolio valuation context
```

### Participant

Participant remains an independent Core Domain Entity. Orders, Trades, Portfolio, Strategy, and Execution use ParticipantReference for attribution. ParticipantReference does not transfer Participant ownership.

### Listing

Listing remains the authoritative market context for market observations, Orders, Trades, and Positions. Instrument and Exchange meaning are reached through Listing; consuming concepts do not duplicate those boundaries.

### Forbidden Boundary Drift

The following are architectural violations unless explicitly approved:

- introducing a generic Market Data Value Object layer,
- embedding Market Data inside Orders, Trades, or Portfolio,
- making ParticipantReference own Participant,
- making Trades own Portfolio,
- making Portfolio own Market Data,
- making Position an independent Aggregate Root,
- encoding business meaning into aggregate identity,
- or introducing workflow, settlement, accounting, risk, or performance into a context that does not own it.

## 11. Identity Philosophy

Identity answers which business object is being referenced. It must not answer what that object currently means, what state it has, or what business attributes it contains.

### Aggregate Identity Value Objects

The approved Aggregate Identity Value Objects are:

- OrderIdentity,
- ParticipantIdentity,
- TradeIdentity,
- PortfolioIdentity.

Each is opaque, stable, immutable, value-based, and specific to its business concept. Each identifies exactly one aggregate or identity-bearing Entity without encoding Listing, Participant attribution, Quantity, Price, lifecycle, Market Data, or ownership interpretation.

### Why PositionIdentity Does Not Exist

Position is subordinate to Portfolio. Its identity is local to the Portfolio and Listing context. The approved Version 1 identity rule is:

```text
Portfolio + Listing
```

PositionIdentity would be unnecessary duplication and would risk creating an independent identity boundary where none is approved.

### Identity and Equality

Aggregate Root equality is identity-based through its Aggregate Identity Value Object. Value Object equality is value-based. Subordinate Position equality remains intentionally unresolved in Version 1 because no PositionIdentity has been approved.

## 12. Design-First Engineering

Northstar development follows this lifecycle:

```text
Business Analysis
        |
        v
Architectural Classification
        |
        v
Design Specification
        |
        v
Implementation
        |
        v
Reference Contract Suite
        |
        v
Registry
        |
        v
CHANGELOG
```

### Business Analysis

Clarifies business purpose, vocabulary, responsibilities, relationships, and unresolved questions.

### Architectural Classification

Determines whether a concept is a Value Object, Entity, Aggregate Root, subordinate Entity, Market Observation, or another approved concept.

### Design Specification

Defines canonical business meaning, ownership, invariants, relationships, temporal semantics, and boundaries.

### Implementation

Expresses approved design with minimal public API and no invented behavior. If design is incomplete, implementation stops.

### Reference Contract Suite

Verifies approved business behavior, composition, identity semantics, value semantics, boundaries, immutability or mutability, and representation without redesigning the concept.

### Registry

Records the approved reference implementation and its engineering pattern for future work.

### CHANGELOG

Records the completed implementation and documentation milestone. It does not replace the design specification or registry.

## 13. Engineering Standards

### Immutability Philosophy

Immutable concepts preserve historical truth and stable value meaning.

- Foundation Value Objects are immutable.
- Observation State Value Objects are immutable.
- Trade is immutable after creation because it records execution truth.
- TradeIdentity and other identity Value Objects are immutable.

Immutability prevents later changes from silently rewriting the meaning of a past observation or execution.

### Mutability Philosophy

Mutable concepts represent business meaning that evolves within an approved lifecycle or consistency boundary.

- Order is mutable as a participant-intent Aggregate Root, although Version 1.0 does not implement transition APIs.
- Portfolio is mutable because ownership interpretation changes as completed Trades are incorporated.
- Position is mutable because Listing-specific ownership interpretation changes within Portfolio.

Mutability must remain inside the owning aggregate boundary and must not become an excuse to introduce workflow or execution behavior.

### Aggregate Consistency

Aggregate Roots protect invariants across their owned concepts:

- Order protects intent and lifecycle consistency.
- Trade protects immutable execution truth.
- Portfolio protects Position collection consistency, including one Position per Listing.

### Value Semantics

Value Objects are defined by business value, are identity-free, and are normally immutable. Their equality and hashing follow their canonical business meaning.

### Entity Semantics

Entities have identity independent of mutable business state. Aggregate Root identity is explicit and stable. Subordinate Entity identity is local to its owning Aggregate Root where approved.

### Validation Philosophy

Validation protects approved business invariants and type boundaries. It must not invent workflow rules, execution rules, persistence rules, or technical constraints that are absent from the design.

### Reference Implementations

Reference implementations are architectural baselines. Future concepts should reuse an approved pattern when their business meaning is the same. A different pattern requires explicit architectural justification and review.

## 14. Future Bounded Contexts

Future bounded contexts must build upon the completed architecture rather than redesign it.

### Strategy

Strategy may express decision or allocation intent. It must remain distinct from Orders, Trades, Portfolio, and Market Data. It may consume approved context but must not own those bounded contexts.

### Execution

Execution may coordinate the process that turns Orders into completed Trades. It must not redefine Order intent or Trade execution truth.

### Risk

Risk may interpret Portfolio, Position, Trade, and Market Data context. It must not become the owner of Portfolio, Market Data, or execution truth.

### Performance

Performance may interpret historical Trades and Portfolio ownership changes. It must not mutate Trade or absorb Portfolio ownership meaning.

### Analytics

Analytics may consume observations, Orders, Trades, and Portfolio interpretations. It must remain an interpretive consumer rather than an owner of those concepts.

Future contexts must:

- reuse Foundation primitives,
- preserve Core Domain boundaries,
- consume Market Data without owning it,
- reference participant identity through ParticipantReference,
- preserve Order, Trade, and Portfolio ownership rules,
- and follow Design-First Engineering.

No redesign of completed bounded contexts is expected. Any material change requires an approved architecture decision.

## 15. Architecture Principles

1. **One business meaning per bounded context.** Each concept has one authoritative owner.
2. **Aggregate Roots own consistency.** Order, Trade, and Portfolio protect their boundaries.
3. **Value Objects own meaning.** Value Objects preserve precise business concepts without independent identity.
4. **Market Data observes.** Market Data records reported market state and does not own business intent or execution.
5. **Orders express intent.** Orders preserve participant instruction and transaction commitment.
6. **Trades preserve execution truth.** Trades record completed historical execution and remain immutable.
7. **Portfolio interprets ownership.** Portfolio transforms completed outcomes into current ownership interpretation.
8. **Positions remain subordinate.** Position belongs to Portfolio and represents Listing-specific ownership interpretation.
9. **Participant is referenced.** ParticipantReference provides attribution without transferring Participant ownership.
10. **Identity is opaque.** Aggregate identities must not encode business attributes or mutable state.
11. **Reuse engineering patterns.** Future concepts should inherit approved patterns when their business meaning is equivalent.
12. **Avoid accidental complexity.** Do not introduce identity, status, workflow, or architectural layers without approved meaning.
13. **Composition over duplication.** Existing concepts are composed rather than redefined.
14. **Historical truth remains stable.** Observations and completed execution outcomes must not be silently rewritten.
15. **Architecture before implementation.** Unresolved business questions stop implementation until architecture is approved.
16. **Framework independence matters.** Core business meaning must remain independent of infrastructure and frameworks.
17. **Contract suites verify approved behavior.** Tests protect design; they do not create it.

## Canonical Architecture Summary

```text
Foundation
    |
    +-- reusable business primitives

Core Domain
    |
    +-- Instrument
    +-- Exchange
    +-- Listing
    +-- Participant
    +-- ParticipantIdentity
    +-- ParticipantReference

Market Data
    |
    +-- Quote
    +-- Tick
    +-- OHLCBar
    +-- OrderBook

Orders
    |
    +-- Order
    |      +-- OrderIdentity
    |      +-- OrderStatus
    |
    +-- ParticipantReference association

Trades
    |
    +-- Trade
           +-- TradeIdentity
           +-- ParticipantReference association

Portfolio
    |
    +-- Portfolio
           +-- PortfolioIdentity
           +-- ParticipantReference association
           +-- Position
                  +-- Listing
```

Northstar’s architecture is complete for Platform Core v1.0.0. Future bounded contexts extend this architecture through approved design, explicit ownership, and disciplined reuse of the reference patterns recorded here.
