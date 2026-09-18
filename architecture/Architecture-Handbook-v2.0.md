# Northstar Architecture Handbook v2.0

## Status

**Canonical architecture reference for Northstar Platform v1.1.0.**

This handbook supersedes Architecture Handbook v1.0. It is the single architectural reference for future Northstar development. It records the completed and frozen Domain architecture, the approved Application Layer architecture, and the required Infrastructure boundary.

This is an architectural reference, not an implementation guide. Approved design specifications, ADRs, reference implementation registries, and reference contract suites remain authoritative for their respective details. Implementation must not invent business rules or alter an approved boundary.

---

## 1. Platform Vision

Northstar is a financial intelligence platform. It represents the progression from reported market context, through participant decision and intent, to completed execution and current ownership interpretation. Trading and broker connectivity are platform capabilities, not the platform's sole identity.

The platform exists to preserve business meaning as information changes form:

```text
Market observation -> decision -> intent -> execution truth -> ownership interpretation
```

Northstar uses Domain-Driven Design because these concepts are operationally related but not interchangeable. A Quote is not an Order, an Order is not a Trade, and a Trade is not a Portfolio. Each has distinct vocabulary, lifecycle, owner, and consistency needs. Bounded contexts protect those distinctions.

Northstar uses Clean Architecture to keep business meaning independent of delivery mechanisms and technology choices. Domain code must not be shaped by a database, a protocol, a broker, or a user interface. Technical capabilities serve the inner business model through explicit boundaries.

### Design-First Engineering

Design-First Engineering is mandatory. Business analysis and architectural classification precede implementation so that code expresses approved meaning rather than becoming the source of it. When a business question, ownership boundary, or invariant is unresolved, implementation stops until architecture and design resolve it.

---

## 2. Complete Platform Architecture

Northstar is organized into three layers:

```text
Domain
  ^
  |
Application
  ^
  |
Infrastructure
```

The diagram expresses dependency direction: outer layers depend inward. Operational requests flow through the layers toward Domain behavior; results, events, and integrations flow outward through approved ports.

### Domain

The Domain Layer owns business meaning: bounded contexts, aggregates, entities, Value Objects, invariants, lifecycle semantics, and business events. It is framework-independent and has no knowledge of Application, Infrastructure, transport, persistence, or protocols.

### Application

The Application Layer owns use-case coordination. It invokes Domain behavior, manages workflow and transaction boundaries, dispatches Domain events, and calls external capabilities through ports. It coordinates business concepts but does not define or own their meaning.

### Infrastructure

Infrastructure owns technical mechanisms: databases, message brokers, networks, protocols, authentication integrations, and external-system adapters. It implements ports required by Application and must not become the owner of business rules.

### Ownership Boundaries

- Domain owns business concepts and their consistency.
- Application owns the orchestration of a use case and its process state.
- Infrastructure owns technical implementation details.
- No layer may silently take ownership from another layer.

---

## 3. Domain Architecture

The Domain architecture is complete and frozen for the following bounded contexts. Future work extends this architecture through approved design; it does not redesign the completed contexts.

### Foundation

**Business responsibility:** Provide reusable primitive business meaning.

**Owns:** Identity, measurement, financial, temporal, and composed financial Value Objects, including Symbol, ExchangeCode, Quantity, Percentage, Currency, Price, Money, Timeframe, and PointInTime.

**Consumes:** Nothing upward. Foundation is reused by Domain concepts that require its approved primitive meanings.

**Influences:** All Domain contexts through stable, framework-independent Value Objects.

**Does not own:** Instruments, Listings, participants, market observations, Orders, Trades, Portfolio state, Strategy policy, workflow, or technical concerns.

### Core Domain

**Business responsibility:** Express stable market and participant concepts that organize the shared business language.

**Owns:** Instrument, Exchange, Listing, Participant, ParticipantIdentity, ParticipantReference, ListingStatus, and Tradability.

**Consumes:** Foundation primitives.

**Influences:** Market Data, Orders, Trades, Portfolio, and Strategy through shared market context and participant attribution.

**Does not own:** Market observations, participant intent, execution truth, ownership interpretation, strategy lifecycle, application workflow, or integrations.

Listing remains the authoritative market context for observations, Orders, Trades, and Positions. ParticipantReference identifies Participant for consuming concepts without owning Participant or Participant lifecycle.

### Market Data

**Business responsibility:** Preserve what the market reports in the correct Listing and temporal context.

**Owns:** Quote, Tick, OHLCBar, OrderBook, and their observation state Value Objects: QuotedMarketState, TickState, BarState, and OrderBookState.

**Consumes:** Listing and Foundation temporal and financial primitives.

**Influences:** Strategy decisions, Order context, and Portfolio valuation context.

**Does not own:** Participant intent, Orders, execution outcomes, Trades, Portfolio state, Strategy policy, workflow, or data-acquisition technology.

Market Data is observational. It reports market state; it does not decide or act.

### Orders

**Business responsibility:** Preserve participant intent, transaction commitment, and Order lifecycle meaning.

**Owns:** Order, OrderIdentity, OrderStatus, and Order aggregate consistency.

**Consumes:** Listing, ParticipantReference, and approved Foundation Value Objects. It may consume Market Data as context without owning it.

**Influences:** Application execution coordination and the creation of completed execution outcomes.

**Does not own:** Market Data, Participant lifecycle, broker communication, Trade execution truth, Portfolio state, settlement, accounting, or workflow engines.

An Order expresses intent. It may exist without producing a Trade.

### Trades

**Business responsibility:** Preserve completed execution truth and historical execution outcomes.

**Owns:** Trade, TradeIdentity, and Trade aggregate consistency.

**Consumes:** Listing, ParticipantReference, and Foundation Value Objects supplied in an approved execution outcome.

**Influences:** Portfolio ownership interpretation, historical analysis, and downstream application event handling.

**Does not own:** Orders, Market Data, Participant lifecycle, broker protocol state, Portfolio state, settlement, accounting, or workflow.

A Trade is an independent immutable historical aggregate after creation.

### Portfolio

**Business responsibility:** Preserve current ownership interpretation for one participant context.

**Owns:** Portfolio, PortfolioIdentity, and subordinate Position entities. Portfolio protects the invariant that it contains at most one Position interpretation per Listing.

**Consumes:** ParticipantReference, PointInTime, completed Trade outcomes, and optionally Market Data as valuation context.

**Influences:** Current holdings, exposure, and future Risk, Performance, and Analytics interpretations.

**Does not own:** Orders, Trades, Market Data, Participant lifecycle, execution, settlement, accounting, risk, performance, or workflow.

Portfolio interprets completed activity; it never becomes the owner of historical Trades.

### Strategy

**Business responsibility:** Preserve a defined decision policy or framework for reasoning and action selection.

**Owns:** Strategy, StrategyIdentity, and Strategy aggregate consistency.

**Consumes:** Approved market context and other approved inputs through Application use cases.

**Influences:** Decision intent that may lead to Order-oriented application workflows.

**Does not own:** Market Data, Orders, Trades, Portfolio state, broker communication, execution outcomes, research infrastructure, or workflow orchestration.

Strategy decides within its approved Domain meaning. It does not submit an Order or communicate with a broker.

---

## 4. Application Architecture

The Application Layer represents the platform's use-case boundary. It translates an inbound request, schedule, or Domain-triggered workflow into a coordinated sequence of Domain operations and external port invocations.

It solves coordination concerns that do not belong to one aggregate: cross-context workflow ordering, transaction and consistency boundaries, external interaction timing, and post-domain reactions. It may retain process state necessary to execute or resume a workflow, but it is not a primary owner of business state.

### Responsibilities

The Application Layer is responsible for:

- use-case orchestration across bounded contexts;
- command handling and workflow coordination;
- execution coordination;
- defining transaction and consistency boundaries;
- retrieving and persisting aggregates through ports;
- invoking approved Domain behavior;
- dispatching Domain events after successful Domain work;
- invoking external ports; and
- shaping application-level results for inbound adapters.

The Application Layer must not:

- define or duplicate Domain invariants;
- own Order, Trade, Portfolio, Strategy, or Market Data business meaning;
- implement protocol clients, database technology, or messaging technology;
- infer new lifecycle transitions or execution semantics; or
- become a generic business-rules layer outside the Domain.

### Domain Coordination

Application coordinates existing Domain concepts rather than creating an alternate model:

- It invokes Strategy behavior to obtain or process approved decision intent.
- It invokes Orders behavior to create or manage participant intent.
- It coordinates the creation of Trade execution truth from confirmed external outcomes.
- It invokes Portfolio behavior to incorporate approved execution outcomes into current ownership interpretation.

The Domain remains authoritative for every invariant, identity rule, and lifecycle rule in these interactions.

---

## 5. Infrastructure Architecture

Infrastructure supplies technical capabilities required to run the platform. It owns concrete technology and external-system integration, not business meaning.

### Infrastructure Responsibilities

Infrastructure owns implementations for:

- broker adapters;
- exchange adapters;
- FIX connectivity and protocol handling;
- REST clients and servers;
- WebSocket clients and servers;
- persistence engines, database repositories, and storage mechanisms;
- messaging transports, queues, topics, and event-bus technology;
- authentication and authorization integrations;
- networking, retries, observability, and configuration;
- schedulers and other runtime mechanisms.

Infrastructure adapters translate Application port calls to a technical protocol and translate technical responses into Application-facing results. A broker-specific status, database schema, or network retry is not Domain meaning.

Infrastructure must not decide whether an Order is valid, redefine a Trade, calculate Portfolio ownership rules, or embed Strategy policy. It may enforce technical reliability, security, and transport concerns without owning the business decision being carried.

---

## 6. Business Flow

The complete platform progression is:

```text
Market Data
  ↓
Observation
  ↓
Strategy
  ↓
Decision
  ↓
Order
  ↓
Intent
  ↓
Application Execution
  ↓
Infrastructure
  ↓
Broker / Exchange
  ↓
Trade
  ↓
Execution Truth
  ↓
Portfolio
  ↓
Ownership Interpretation
```

### Market Data to Observation

Market Data preserves reported information as a Listing-specific and temporally meaningful observation. It does not imply a recommendation, participant intention, or execution.

### Observation to Strategy

Application supplies approved observation context to Strategy-oriented use cases. Strategy evaluates that context according to its Domain policy. The application flow does not move market-observation meaning into Strategy.

### Strategy to Decision

Strategy produces or participates in a decision within its approved business boundary. A decision is not an Order and has no broker effect by itself.

### Decision to Order

Application coordinates the conversion of approved decision intent into an Order workflow by invoking the Orders Domain model. Order retains ownership of participant intent and its lifecycle meaning.

### Order to Intent

The resulting Order is the authoritative transaction commitment. Application may persist it and begin execution coordination, but it does not reinterpret it as a Trade.

### Application Execution to Infrastructure

The Execution Coordinator invokes an outbound broker or exchange port. Infrastructure implements that port using an appropriate adapter, such as FIX, REST, or WebSocket. The Domain knows neither the protocol nor the adapter.

### Broker or Exchange to Trade

Infrastructure reports a confirmed external outcome to Application. Application validates the use-case flow and invokes Trades Domain behavior to establish a Trade. Trade owns completed execution truth and remains immutable.

### Trade to Portfolio

Application coordinates the incorporation of an approved Trade outcome into the relevant Portfolio. Portfolio updates its Position interpretation while retaining its own aggregate invariants.

### Portfolio to Ownership Interpretation

Portfolio is the authoritative current interpretation of Listing-specific ownership for its participant context. It does not rewrite the Trade that informed it.

---

## 7. Engineering Patterns

Engineering families are selected by business responsibility, not superficial data shape. The reference implementation registry records the canonical implementation pattern for each approved component.

### Foundation Value Objects

**Business purpose:** Represent reusable primitive business meaning.

**Engineering philosophy:** Immutable, value-based, validated, normalized where approved, framework-independent, and reusable across bounded contexts.

**Approved reference implementations:** Symbol, ExchangeCode, Quantity, Percentage, Currency, Price, Money, Timeframe, and PointInTime.

### Observation State Value Objects

**Business purpose:** Preserve specialized market-observation meaning without duplicating Listing or temporal context.

**Engineering philosophy:** Immutable state belongs to one observation type. The owning observation retains Listing and time; the state object owns only its specialized observation meaning.

**Approved reference implementations:** QuotedMarketState, TickState, BarState, and OrderBookState.

### Lifecycle State Value Objects

**Business purpose:** Preserve the lifecycle condition of an owning aggregate.

**Engineering philosophy:** Lifecycle state is aggregate-specific and subordinate. It is not a shared workflow framework and does not independently orchestrate transitions.

**Approved reference implementation:** OrderStatus.

### Aggregate Identity Value Objects

**Business purpose:** Preserve stable opaque identity for an aggregate root or approved identity-bearing Entity.

**Engineering philosophy:** Identity is immutable, value-based, stable, and independent of business attributes, lifecycle, participant attribution, Market Data, or persistence encoding.

**Approved reference implementations:** OrderIdentity, ParticipantIdentity, TradeIdentity, PortfolioIdentity, and StrategyIdentity.

### Identity Association Value Objects

**Business purpose:** Preserve attribution to an independent identity-bearing concept without taking ownership of it.

**Engineering philosophy:** An association identifies another business concept but neither imports its lifecycle nor duplicates its meaning.

**Approved reference implementation:** ParticipantReference.

### Reference Subordinate Entities

**Business purpose:** Represent mutable business meaning that exists within an owning aggregate rather than as an independent aggregate boundary.

**Engineering philosophy:** Local identity is sufficient when the business concept is only meaningful inside the aggregate. A new aggregate identity is not introduced without approved independent meaning.

**Approved reference implementation:** Position.

### Reference Aggregate Roots

**Business purpose:** Protect consistency around independently meaningful business concepts.

**Engineering philosophy:** An aggregate root owns its identity, composition, invariants, and lifecycle or historical consistency according to its business role.

**Approved reference implementations:** Order, Trade, Portfolio, and Strategy.

---

## 8. Reference Aggregate Roots

### Order: Mutable Intent

Order represents participant intent and transaction commitment. It is mutable through its approved aggregate lifecycle. It owns OrderStatus and composes OrderIdentity, Listing, PointInTime, Quantity, Price, and ParticipantReference. Its equality is identity-based through OrderIdentity.

### Trade: Immutable Historical Truth

Trade represents a completed execution outcome. It is immutable after creation because historical execution truth must remain stable and reproducible. It owns TradeIdentity, composes Listing, PointInTime, Quantity, Price, and ParticipantReference, and has identity-based equality.

### Portfolio: Mutable Ownership Interpretation

Portfolio represents current ownership interpretation for one participant context. It is mutable because completed Trade outcomes change current holdings. It owns PortfolioIdentity and subordinate Positions, and protects one Position interpretation per Listing.

### Strategy: Mutable Decision Policy

Strategy represents a decision policy or framework whose approved configuration and lifecycle may evolve. It owns StrategyIdentity and its aggregate consistency. Strategy remains distinct from the application workflow that evaluates or acts on its decisions.

---

## 9. Entity Design

### Aggregate Root

An Aggregate Root is independently meaningful inside a bounded context and owns a consistency boundary. It controls changes to its owned concepts and protects its approved invariants.

### Subordinate Entity

A subordinate Entity has mutable business meaning but exists only within its owning Aggregate Root. It is not independently loaded, persisted, or referenced as an aggregate unless an approved architecture change establishes that boundary.

### Portfolio and Position

The reference relationship is:

```text
Portfolio
  |
  +-- Position
```

Portfolio owns the Position collection and aggregate consistency. Position represents one Listing-specific ownership interpretation. Participant attribution and temporal context are inherited from Portfolio at the business level rather than duplicated in Position.

### Local Identity Without PositionIdentity

Position has local identity through:

```text
Portfolio + Listing
```

Within one Portfolio, at most one Position may interpret ownership for a Listing. PositionIdentity does not exist because Position has no approved independent identity or lifecycle outside Portfolio. Adding it would duplicate identity and imply an aggregate boundary that the Domain does not own.

---

## 10. Identity Philosophy

Identity answers which business object is referenced. It does not describe the object's attributes, current state, relationships, or history.

### Opaque Identity

Opaque identity is a stable identifier whose representation does not encode mutable or business-significant attributes. Format and persistence encoding are not inferred from the Aggregate Identity family.

### Aggregate Identity

Aggregate identity identifies an independently meaningful aggregate root or approved Entity. It supports identity-based equality while remaining distinct from the aggregate's mutable state.

Approved Aggregate Identity Value Objects are:

- OrderIdentity;
- ParticipantIdentity;
- TradeIdentity;
- PortfolioIdentity; and
- StrategyIdentity.

### Identity Association

An identity association identifies another aggregate without transferring ownership. ParticipantReference is the approved reference: it attributes Orders, Trades, Portfolio, and Strategy-related concepts to Participant while leaving Participant ownership and lifecycle in Core Domain.

### Local Identity

Local identity is determined only inside an owning aggregate. Position uses Portfolio plus Listing as its approved local identity. It therefore does not need a separate PositionIdentity.

---

## 11. Application Building Blocks

The following are approved Application Layer concepts. This handbook classifies them; detailed implementation design requires separate approval.

### Application Services

Application Services expose cohesive application capabilities and coordinate a use case. They are thin with respect to business rules and delegate business decisions to Domain objects.

### Use Cases

Use Cases express a user, system, schedule, or event-driven intent in application terms. They define the orchestration path, required collaborators, and expected result without becoming Domain models.

### Command Handlers

Command Handlers receive requests that intend to change business state. They validate application-level request shape, open the appropriate transaction boundary, invoke Domain behavior, and coordinate port use.

### Query Handlers

Query Handlers serve read-oriented requests without changing Domain state. They may use read ports or projections but must not place business invariants into query logic.

### Execution Coordinator

The Execution Coordinator is an Application capability that coordinates the path from approved Order intent through an external broker or exchange interaction to confirmed Trade creation and Portfolio impact. It does not own Order, Trade, Portfolio, broker protocol, or execution business semantics.

### Ports

Ports are Application-facing contracts for external capabilities, including persistence, broker or exchange communication, messaging, identity, and other integrations. Application depends on ports; Infrastructure implements them.

### Domain Event Dispatcher

The Domain Event Dispatcher coordinates post-domain reactions after a successful transaction boundary. It routes Domain events to application workflows or outbound integration ports without making technical messaging a Domain responsibility.

### Transaction Boundaries

Transaction boundaries define the atomic or compensating scope of an application use case. They coordinate persistence and event-dispatch timing but do not relax aggregate invariants or create cross-aggregate ownership.

---

## 12. Dependency Rules

Northstar uses inward dependency and dependency inversion.

```text
Presentation / inbound adapters
  ↓
Application
  ↓
Domain

Infrastructure adapters
  ↓ implements
Application ports
```

### Required Rules

- Domain depends on nothing upward. It never depends on Application, Infrastructure, API, web, transport, persistence, or framework code.
- Application depends on Domain abstractions and its own port contracts.
- Infrastructure depends inward to implement Application ports and use Domain-compatible types where required.
- The Composition Root wires concrete Infrastructure adapters to Application ports and exposes inbound delivery mechanisms.
- Presentation and API adapters invoke Application capabilities; they do not bypass Application to mutate Domain state directly.

### Dependency Inversion

Application names the capability it needs through a port. Infrastructure supplies an implementation for that port. This keeps broker vendors, database engines, message transports, and protocol libraries replaceable without changing Domain meaning or application use-case intent.

---

## 13. Design-First Engineering

The official Northstar workflow is:

```text
Business Analysis
  ↓
Architectural Classification
  ↓
Design Specification
  ↓
Implementation
  ↓
Reference Contract Suite
  ↓
Registry
  ↓
CHANGELOG
```

### Business Analysis

Clarifies business purpose, ubiquitous language, ownership, relationships, and unresolved questions.

### Architectural Classification

Determines the correct architectural role: Value Object, Entity, Aggregate Root, subordinate Entity, Market Observation, Application capability, port, or Infrastructure adapter.

### Design Specification

Defines canonical meaning, invariants, identity, composition, temporal semantics, mutability, and boundaries before code is written.

### Implementation

Implements approved design with a minimal public API. It must not introduce rules merely because they appear convenient.

### Reference Contract Suite

Verifies the approved contract: construction, validation, value or identity semantics, mutability, composition, boundaries, and representation as relevant. Tests protect design; they do not create it.

### Registry

Records the approved reference implementation and the engineering family future work must follow.

### CHANGELOG

Records the completed implementation and documentation milestone. It is historical record, not a substitute for design or registry authority.

This workflow exists to prevent accidental architecture, duplicate ownership, invented identity, and technical coupling from entering the platform through implementation.

---

## 14. Engineering Principles

### Aggregate Consistency

Aggregate Roots own and protect the invariants inside their approved consistency boundaries. Cross-aggregate workflow belongs to Application, not to an aggregate that does not own the other concept.

### Value Semantics

Value Objects are defined by their business value, not independent identity. They are normally immutable, validated, and equality-based on canonical meaning.

### Entity Semantics

Entities retain identity independent of mutable state. Aggregate Root identity is explicit and stable. Subordinate Entity identity is local where approved.

### Identity Philosophy

Identity is opaque and specific to the concept it identifies. It must not encode state, participant attribution, Listing, Price, Quantity, lifecycle, or persistence detail.

### Mutability and Immutability

Mutability is permitted only where business meaning evolves within an owned boundary. Order, Portfolio, Position, and Strategy are mutable according to their approved roles. Foundation and state Value Objects are immutable. Trade is immutable because it records completed historical truth.

### Validation

Validation enforces approved business invariants and type boundaries. It must not invent workflow, protocol, persistence, or operational restrictions that have not been designed.

### Separation of Concerns

Domain owns business meaning. Application coordinates use cases. Infrastructure integrates technology. Presentation delivers interaction. A concern crossing this boundary requires explicit architectural review.

### Reference Implementations

Approved references are the baseline for future concepts with equivalent business roles. Reuse the appropriate family rather than creating parallel patterns.

### Avoiding Accidental Complexity

Do not introduce an aggregate identity, generic status framework, workflow engine, new abstraction, or bounded context without approved business need. Composition is preferred over duplication.

---

## 15. Anti-Patterns

Northstar intentionally avoids the following architectural mistakes:

- Domain depending on Infrastructure, API, web, or framework code.
- Application owning business meaning, lifecycle rules, or aggregate invariants.
- Infrastructure containing business rules or becoming the authoritative business model.
- Duplicating Aggregate Roots or creating alternate sources of truth.
- Creating an unnecessary Aggregate Identity Value Object, particularly PositionIdentity.
- Duplicating ownership between contexts, such as Portfolio owning Trades or Orders owning Market Data.
- Cross-bounded-context state ownership.
- Protocol logic, broker status types, FIX sessions, REST clients, or WebSocket concerns inside Domain.
- Treating a broker confirmation as a Domain aggregate without Application coordination and approved Trade creation.
- Treating execution orchestration as a new Domain bounded context.
- Allowing Strategy to submit orders or communicate directly with a broker.
- Using contract tests to invent business requirements.
- Introducing generic frameworks that obscure approved business families.

---

## 16. Future Platform Direction

Future platform work extends the architecture through Design-First Engineering and explicit ownership review.

### Application

Application will gain approved use cases, commands, queries, execution workflows, process coordination, port contracts, event-dispatch flows, and transaction-boundary designs. These additions must coordinate the frozen Domain architecture rather than move business meaning out of it.

### Infrastructure

Infrastructure will gain concrete broker, exchange, FIX, REST, WebSocket, persistence, messaging, authentication, and networking adapters. Technology choices remain replaceable behind Application ports.

### Risk

Risk may interpret Portfolio, Position, Trade, and Market Data context. It must not take ownership of those existing concepts or encode its interpretation into them without approved design.

### Performance

Performance may interpret historical Trades and Portfolio ownership changes. It must not mutate historical Trade truth or absorb Portfolio ownership meaning.

### Analytics

Analytics may consume market observations, decisions, Orders, Trades, and Portfolio interpretations. It remains an interpretive consumer rather than the owner of the concepts it analyzes.

Any future capability that changes completed Domain ownership, aggregate boundaries, or dependency direction requires an approved architectural decision before implementation.

---

## 17. Architectural Principles

1. **One business meaning per bounded context.** Each concept has one authoritative owner.
2. **One owner for every business concept.** Referencing a concept never transfers ownership.
3. **Aggregate Roots own consistency.** Their invariants are not duplicated in Application or Infrastructure.
4. **Value Objects own precise meaning.** They carry value semantics without independent identity.
5. **Identity is opaque.** Aggregate identity does not encode mutable business attributes.
6. **Market Data observes.** It reports market state and does not express intent or execution.
7. **Strategy decides.** It owns decision policy, not workflow or broker connectivity.
8. **Orders express intent.** They preserve transaction commitment, not execution truth.
9. **Trades preserve execution truth.** They are immutable historical aggregates.
10. **Portfolio interprets ownership.** It turns completed outcomes into current holdings without owning Trade history.
11. **Application coordinates.** It orchestrates use cases but does not own business meaning.
12. **Infrastructure integrates.** It implements technology and external communication without defining business rules.
13. **Reuse engineering patterns.** Equivalent business roles inherit approved reference families.
14. **Architecture before implementation.** Unresolved meaning stops code until design is approved.
15. **Avoid accidental complexity.** New identities, abstractions, and layers require real business justification.

---

## Canonical Architecture Summary

```text
Domain
  Foundation
  Core Domain
  Market Data
  Orders
  Trades
  Portfolio
  Strategy

Application
  Application Services
  Use Cases
  Command Handlers
  Query Handlers
  Execution Coordinator
  Ports
  Domain Event Dispatcher
  Transaction Boundaries

Infrastructure
  Broker and Exchange Adapters
  FIX / REST / WebSocket
  Persistence
  Messaging
  Authentication
  Networking

Composition Root
  Wires Infrastructure implementations to Application ports
```

Northstar Platform v1.1.0 has a complete layered architecture. Future development must preserve the frozen Domain architecture, use Application for coordination, keep Infrastructure technical, and follow the approved Design-First Engineering workflow.

---

## Architecture Review

- Domain architecture documented.
- Application architecture documented.
- Infrastructure architecture documented.
- Engineering patterns documented.
- Aggregate Root philosophy documented.
- Identity philosophy documented.
- Dependency rules documented.
- Design-First workflow documented.
- Anti-patterns documented.
- Handbook is suitable as the canonical architecture reference.

---

APPLICATION LAYER

Architecture Review

Approved

Ready for Architecture Design

---
