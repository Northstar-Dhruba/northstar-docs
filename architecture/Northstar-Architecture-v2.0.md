# Northstar Architecture v2.0

## Architecture Baseline

**Status: Approved**

Northstar Architecture v2.0 is the official architecture baseline for Northstar Platform v1.1.0. It records the completed architectural foundation and serves as the constitutional reference for all future implementation work.

This document is executive-level architecture documentation. It summarizes approved boundaries and decisions; it does not prescribe implementation, frameworks, libraries, protocols, package structures, or concrete adapters.

---

## 1. Architecture Overview

Northstar Architecture v2.0 represents the completed architectural foundation of a financial intelligence platform. It defines the authoritative separation between Domain business meaning, Application orchestration, and Infrastructure technical implementation.

This milestone matters because it establishes a stable basis for implementation. The platform now has explicit ownership, dependency direction, reference patterns, and an approved process for adding capabilities without allowing technology, workflow convenience, or duplicated models to distort business meaning.

The architecture phase is complete. Future work must implement and extend this baseline rather than redesign it.

---

## 2. Architectural Vision

Northstar is guided by the following principles:

- **Domain-Driven Design:** Business concepts are modeled according to their distinct meaning, vocabulary, lifecycle, and owner.
- **Clean Architecture:** Business meaning remains independent of delivery mechanisms and technical choices.
- **Design-First Engineering:** Business analysis, architectural classification, and approved design precede implementation.
- **Dependency Inversion:** Application depends on abstractions; Infrastructure supplies technical implementations.
- **Explicit Ownership:** Every business concept has one authoritative owner. Referencing a concept never transfers ownership.
- **Separation of Concerns:** Domain decides, Application coordinates, and Infrastructure integrates.

Northstar is a financial intelligence platform. Observation, decision, intent, execution, and ownership interpretation are related capabilities with distinct business meanings. The architecture protects those distinctions.

---

## 3. Complete Platform Architecture

Northstar uses a layered architecture with inward dependencies:

```text
Domain
  ^
  |
Application
  ^
  |
Infrastructure
```

Operational requests pass inward from delivery mechanisms through Application to Domain behavior. Technical capabilities are invoked outward through Application ports and fulfilled by Infrastructure adapters.

### Domain

Domain owns business meaning: bounded contexts, Aggregate Roots, entities, Value Objects, invariants, lifecycle semantics, and Domain events. It is independent of Application, Infrastructure, transport, persistence, protocols, and frameworks.

### Application

Application owns use-case orchestration. It coordinates Domain behavior, cross-context workflows, transaction boundaries, Domain-event reactions, and calls to external capabilities through ports. It does not own business meaning.

### Infrastructure

Infrastructure owns technical implementation. It provides adapters, protocols, persistence, messaging, authentication integration, networking, and runtime mechanisms. It does not define business rules or become an alternate business model.

---

## 4. Domain Model

The completed Domain architecture is frozen. Its bounded contexts have the following responsibilities and ownership.

### Foundation

**Business responsibility:** Provide reusable primitive business meaning.

**Ownership:** Identity, measurement, financial, temporal, and composed financial Value Objects.

**Downstream influence:** Supplies stable, framework-independent primitives to all Domain contexts.

### Core Domain

**Business responsibility:** Express stable market and participant concepts shared across the platform.

**Ownership:** Instrument, Exchange, Listing, Participant, ParticipantIdentity, ParticipantReference, ListingStatus, and Tradability.

**Downstream influence:** Supplies authoritative market context and participant attribution to Market Data, Orders, Trades, Portfolio, and Strategy.

### Market Data

**Business responsibility:** Preserve what the market reports in Listing and temporal context.

**Ownership:** Quote, Tick, OHLCBar, OrderBook, QuotedMarketState, TickState, BarState, and OrderBookState.

**Downstream influence:** Provides observation context for Strategy decisions, Order context, and Portfolio valuation interpretation.

### Orders

**Business responsibility:** Preserve participant intent, transaction commitment, and Order lifecycle meaning.

**Ownership:** Order, OrderIdentity, OrderStatus, and Order aggregate consistency.

**Downstream influence:** Provides authoritative intent to Application Execution and may lead to completed Trade outcomes.

### Trades

**Business responsibility:** Preserve completed execution truth and historical execution outcomes.

**Ownership:** Trade, TradeIdentity, and immutable Trade aggregate consistency.

**Downstream influence:** Provides completed outcomes for Portfolio ownership interpretation, historical analysis, and downstream application reactions.

### Portfolio

**Business responsibility:** Preserve current ownership interpretation for one participant context.

**Ownership:** Portfolio, PortfolioIdentity, and subordinate Position entities, including the one Position interpretation per Listing invariant.

**Downstream influence:** Provides current holdings and exposure context to future Risk, Performance, and Analytics capabilities.

### Strategy

**Business responsibility:** Preserve a defined decision policy or framework for reasoning and action selection.

**Ownership:** Strategy, StrategyIdentity, and Strategy aggregate consistency.

**Downstream influence:** Produces approved decision intent that can lead to Order-oriented application workflows.

---

## 5. Application Model

The Application Layer is Northstar's use-case boundary. It coordinates existing Domain concepts and external capabilities into a coherent platform outcome.

Application owns:

- use-case orchestration;
- Application Services;
- use cases;
- command handling;
- query handling;
- Execution coordination;
- transaction and consistency boundaries;
- outbound and inbound port contracts; and
- Domain-event dispatch coordination.

Application invokes approved behavior on Strategy, Orders, Trades, and Portfolio. It may retain process state required to coordinate a workflow, but it does not own Strategy policy, Order intent, Trade truth, Portfolio ownership interpretation, or Market Data observation.

Execution is an Application capability. It coordinates approved Order intent, external broker or exchange interaction, confirmed Trade creation, and Portfolio impact without becoming a Domain bounded context or an alternate business model.

---

## 6. Infrastructure Model

Infrastructure fulfills the technical capabilities Application requires through ports.

Infrastructure responsibilities include:

- broker adapters;
- exchange adapters;
- FIX connectivity and protocol handling;
- REST and WebSocket communication;
- persistence engines and database mappings;
- messaging transports and delivery mechanisms;
- authentication and authorization integrations; and
- networking, configuration, observability, retries, scheduling, and runtime concerns.

Infrastructure translates between Application port contracts and concrete technologies. It does not determine whether an Order is valid, redefine Trade truth, calculate Portfolio ownership, or embed Strategy policy.

---

## 7. Business Flow

The complete Northstar business progression is:

```text
Observation
  ↓
Decision
  ↓
Intent
  ↓
Application Execution
  ↓
Infrastructure
  ↓
Execution Truth
  ↓
Ownership Interpretation
```

### Observation

Market Data preserves reported market context. An observation is neither a recommendation nor an instruction to act.

### Decision

Strategy evaluates approved context according to its Domain policy. A decision has no external effect by itself.

### Intent

Application coordinates the transition from approved decision intent to an Order workflow. Orders own participant intent and transaction commitment.

### Application Execution

Application coordinates the approved Order with an outbound broker or exchange port. It determines workflow sequencing and transaction boundaries without defining broker protocol behavior.

### Infrastructure

Infrastructure implements the selected port and performs the required technical communication with a broker or exchange.

### Execution Truth

A confirmed external outcome returns to Application, which invokes the Trades Domain model to establish immutable Trade execution truth.

### Ownership Interpretation

Application coordinates incorporation of the approved Trade outcome into Portfolio. Portfolio owns the mutable, current interpretation of Listing-specific ownership and does not rewrite historical Trade truth.

---

## 8. Reference Architecture

Northstar uses approved engineering families selected by business responsibility, not superficial data shape.

### Aggregate Roots

Aggregate Roots protect independent consistency boundaries, identity, composition, and lifecycle or historical truth. Approved references are Order, Trade, Portfolio, and Strategy.

### Aggregate Identity Value Objects

Aggregate Identity Value Objects provide stable, opaque identity independent of mutable business attributes. Approved references are OrderIdentity, ParticipantIdentity, TradeIdentity, PortfolioIdentity, and StrategyIdentity.

### Identity Association

Identity Association Value Objects attribute a consuming concept to an independently owned identity without transferring ownership. ParticipantReference is the approved reference.

### Subordinate Entities

Subordinate Entities have mutable business meaning within an owning Aggregate Root without creating an independent aggregate boundary. Position is the approved reference and is owned by Portfolio.

### Foundation Value Objects

Foundation Value Objects provide reusable primitive meaning: identity, measurement, financial, temporal, and composed financial concepts. Approved references include Symbol, ExchangeCode, Quantity, Percentage, Currency, Price, Money, Timeframe, and PointInTime.

### Observation State

Observation State Value Objects preserve specialized market-observation meaning while their owning observation retains Listing and temporal context. Approved references are QuotedMarketState, TickState, BarState, and OrderBookState.

### Lifecycle State

Lifecycle State Value Objects preserve aggregate-specific lifecycle condition without becoming a generic workflow mechanism. OrderStatus is the approved reference.

---

## 9. Dependency Model

Northstar applies inward dependency and dependency inversion:

```text
Presentation and inbound adapters
  ↓
Application
  ↓
Domain

Infrastructure adapters
  ↓ implements
Application outbound ports

Composition Root
  ↓ wires
Infrastructure implementations to Application ports
```

Domain depends on nothing upward. Application depends on Domain abstractions and Application-owned port abstractions. Infrastructure depends inward to implement ports and adapt external technologies.

Ports express the capabilities a use case requires without naming a vendor, protocol, database, or messaging system. The Composition Root selects concrete implementations and wires them to the Application runtime. Neither Domain nor Application selects Infrastructure implementations.

Dependency inversion keeps broker integrations, persistence engines, message transports, protocol libraries, and authentication providers replaceable without redesigning business meaning or use-case intent.

---

## 10. Architectural Decisions

### ParticipantReference

ParticipantReference is the approved identity association Value Object. It enables participant attribution in consuming contexts without importing Participant ownership or lifecycle. This preserves Core Domain ownership and prevents duplicated participant models.

### Aggregate Identity Family

OrderIdentity, ParticipantIdentity, TradeIdentity, PortfolioIdentity, and StrategyIdentity provide opaque, stable identities that remain independent of mutable business attributes. This protects identity-based equality and prevents lifecycle or business state from leaking into identifiers.

### Position Without PositionIdentity

Position is subordinate to Portfolio. Its approved local identity is Portfolio plus Listing, with at most one Position interpretation for a Listing within a Portfolio. Omitting PositionIdentity prevents an unnecessary second identity and avoids falsely implying an independent aggregate boundary.

### Execution Outside the Domain

Execution is an Application capability with Infrastructure support. It coordinates Order intent, external communication, Trade creation, and Portfolio impact without redefining existing Domain concepts. This preserves Domain purity and keeps protocol concerns outside Domain.

### Strategy as Decision Policy Aggregate

Strategy is an Aggregate Root that owns decision policy and its consistency. It does not own Order submission, broker connectivity, execution truth, or application workflow. This preserves the distinction between deciding and acting.

### Portfolio Position Collection Invariants

Portfolio owns Positions and enforces at most one Position interpretation per Listing. This gives current ownership interpretation a single authoritative owner and prevents duplicate or conflicting holding state.

---

## 11. Architectural Principles

1. **One owner for every business meaning.**
2. **Aggregate Roots own consistency.**
3. **Value Objects own precise meaning without independent identity.**
4. **Identity is opaque and independent of mutable state.**
5. **Market Data observes.**
6. **Strategy decides.**
7. **Orders express intent.**
8. **Trades preserve immutable execution truth.**
9. **Portfolio interprets mutable ownership.**
10. **Application orchestrates.**
11. **Infrastructure integrates.**
12. **Dependency inversion is mandatory.**
13. **Reference implementations guide equivalent future concepts.**
14. **Accidental complexity is avoided.**
15. **Architecture precedes implementation.**

---

## 12. What Is Now Frozen

The following are Northstar architectural baselines:

- Domain architecture and bounded-context ownership;
- Application architecture and Execution placement;
- Infrastructure direction through ports and adapters;
- engineering standards and Design-First Engineering workflow;
- approved reference implementation families; and
- dependency rules and Composition Root responsibility.

Future implementation must extend these foundations. It must not redesign Domain ownership, move business meaning into Application or Infrastructure, bypass Aggregate Roots, or reverse dependency direction without an approved architectural decision.

---

## 13. Implementation Phase

Northstar now enters the **Platform Implementation Phase**.

Future work will primarily deliver approved modules and adapters in the established architecture, including:

- Application modules and use cases;
- Infrastructure modules;
- broker and exchange adapters;
- persistence;
- messaging;
- APIs and other inbound delivery mechanisms; and
- user interfaces.

Architectural work remains necessary when a genuinely new business boundary or ownership question arises. It is no longer the default activity for completed areas; implementation must follow the established baseline.

---

## 14. Future Evolution

Risk, Performance, and Analytics are future areas that must reuse the completed architecture.

- **Risk** may interpret Portfolio, Position, Trade, and Market Data context without taking ownership of them.
- **Performance** may interpret historical Trades and Portfolio ownership changes without rewriting Trade truth or absorbing Portfolio meaning.
- **Analytics** may consume observations, decisions, Orders, Trades, and Portfolio interpretations as an interpretive capability rather than an alternate owner.

Future capabilities must use Design-First Engineering, explicit ownership analysis, approved Application coordination, and port-based Infrastructure integration. A material change to a frozen boundary requires an approved architectural decision before implementation.

---

## 15. Conclusion

Northstar Architecture v2.0 is the completed architectural foundation of the Northstar platform. It establishes a clear Domain model, an Application orchestration boundary, a replaceable Infrastructure direction, engineering reference families, and mandatory dependency rules.

Future engineering will build upon this architecture rather than redesign it.

---

## Architecture Baseline Review

- Domain architecture documented.
- Application architecture documented.
- Infrastructure direction documented.
- Engineering standards summarized.
- Major architectural decisions documented.
- Platform Implementation Phase clearly defined.
- No redesign introduced.

**Northstar Architecture v2.0 is approved as the official architecture baseline for all future development.**
