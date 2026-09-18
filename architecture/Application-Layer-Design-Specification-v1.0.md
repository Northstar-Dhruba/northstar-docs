# Application Layer Design Specification v1.0

## Status

**Approved architectural and application-design specification for Northstar Platform v1.1.0.**

This document defines the canonical design boundary for every future Northstar Application module. It is governed by Architecture Handbook v2.0 and preserves the completed, frozen Domain architecture.

This specification is intentionally implementation-independent. It defines responsibilities, ownership, relationships, and dependency rules. It does not prescribe classes, frameworks, libraries, protocols, persistence technology, messaging technology, or package structure.

---

## 1. Application Purpose

The Application Layer is Northstar's use-case coordination boundary. It turns an inbound request, scheduled operation, or approved Domain-triggered workflow into an ordered interaction with Domain concepts and external capabilities.

It exists because many platform outcomes require more than one Domain aggregate or bounded context but must not create a new business owner. For example, the progression from Strategy decision to Order intent, confirmed execution outcome, Trade creation, and Portfolio interpretation requires coordination across existing owners.

The Application Layer remains separate from Domain because Domain owns business meaning. Domain defines aggregates, entities, Value Objects, invariants, lifecycle semantics, and Domain events. Application determines when approved Domain behavior is invoked, not what the behavior means.

The Application Layer remains separate from Infrastructure because Infrastructure owns technical implementation. Databases, broker adapters, exchange adapters, transport protocols, messaging systems, authentication providers, and networking mechanisms are interchangeable technical concerns. Application names needed capabilities through ports rather than embedding their implementation.

---

## 2. Application Responsibilities

The Application Layer owns orchestration of a use case. Its responsibilities are:

- receiving an application intent through an inbound boundary;
- coordinating use-case flow across one or more Domain concepts;
- invoking approved behavior on Aggregate Roots and other Domain abstractions;
- coordinating Application Services, commands, queries, and workflow steps;
- defining transaction and consistency boundaries for a use case;
- retrieving and persisting Domain state through outbound ports;
- coordinating Execution as an Application capability;
- dispatching Domain events after successful Domain work;
- calling outbound ports for technical or external capabilities; and
- returning application-level outcomes through inbound boundaries.

Application may own transient process state necessary to coordinate, resume, retry, or observe a use case. This state describes the progress of an application workflow. It is not a substitute source of truth for Order intent, Trade execution truth, Portfolio ownership interpretation, Strategy policy, or Market Data observation.

### Application Versus Domain

| Application responsibility                       | Domain responsibility                                             |
| ------------------------------------------------ | ----------------------------------------------------------------- |
| Coordinates the order of an approved use case.   | Defines business meaning and approved behavior.                   |
| Selects the transaction or consistency scope.    | Protects aggregate invariants within its consistency boundary.    |
| Invokes aggregate operations.                    | Decides whether a requested state change is valid.                |
| Coordinates cross-context workflow.              | Owns each bounded context's concepts and semantics.               |
| Dispatches Domain events to reactions and ports. | Produces Domain events when approved business facts occur.        |
| Invokes external capabilities through ports.     | Remains independent of external systems and technical mechanisms. |

Application must not implement an alternate business model, duplicate validation owned by Domain, or infer new rules from workflow convenience.

---

## 3. Application Building Blocks

The following concepts belong in the Application Layer. This specification classifies their responsibilities only; detailed component design requires separate approval.

### Application Services

Application Services provide cohesive application capabilities. They coordinate a complete use case or a related group of use cases, invoke Domain behavior, and collaborate through ports. They are not repositories for Domain rules or business state ownership.

### Use Cases

Use Cases represent a concrete intent initiated by a user, system, schedule, or approved event-driven flow. A Use Case defines the application-level sequence needed to achieve that intent and the application-level outcome. It delegates business decisions to Domain concepts.

### Command Handlers

Command Handlers coordinate requests intended to change business state. They validate application request shape and authorization context where applicable, establish the appropriate transaction boundary, invoke Domain behavior, persist resulting state through ports, and coordinate permitted event dispatch.

A Command Handler does not decide a Domain invariant or bypass an Aggregate Root to mutate its internal state.

### Query Handlers

Query Handlers coordinate read-oriented requests. They return requested information without changing Domain state. They may use approved read ports or projections, but they do not introduce business decisions or become an alternate owner of Domain truth.

### Execution Coordinator

The Execution Coordinator is the Application capability responsible for coordinating approved Order intent with external broker or exchange interaction and confirmed execution outcomes. It coordinates existing Domain concepts and external ports; it is not a Domain bounded context and does not define execution business meaning.

### Domain Event Dispatcher

The Domain Event Dispatcher coordinates post-transaction reactions to Domain events. It directs those reactions to Application workflows or outbound ports after the relevant Domain work has completed successfully. It does not turn a technical messaging mechanism into a Domain concern.

### Ports

Ports are Application-owned abstractions that describe capabilities needed outside the Application Layer. They allow Application to request persistence, broker or exchange communication, messaging, authentication, scheduling, or other external capabilities without depending on a concrete technology.

### Transaction Boundaries

Transaction boundaries define the conceptual scope in which an application use case must complete atomically or be resolved through approved compensating behavior. They coordinate the timing of aggregate persistence and event dispatch while preserving the independent consistency boundaries of Aggregate Roots.

---

## 4. Domain Relationship

The Application Layer coordinates. The Domain decides.

Application invokes Domain behavior through its public, approved boundaries. It does not inspect or manipulate internal aggregate state to recreate Domain decisions. The Domain remains the sole owner of business invariants, identity semantics, lifecycle rules, and the meaning of Domain events.

### Strategy

Application supplies approved inputs and invokes Strategy behavior within a Strategy-oriented use case. Strategy owns decision policy and aggregate consistency. Application may coordinate the outcome of an approved decision with an Order workflow, but Strategy neither submits an Order nor communicates with a broker.

### Orders

Application coordinates the creation, retrieval, persistence, and approved lifecycle invocation of Orders. Order owns participant intent, transaction commitment, OrderStatus meaning, and Order aggregate invariants. Application does not redefine an Order as a broker request or execution result.

### Trades

Application coordinates the establishment of Trade execution truth from a confirmed external outcome. Trade owns the immutable historical meaning of that outcome. Application does not alter a Trade's historical truth, treat an unconfirmed Order as a Trade, or place broker protocol state inside Trade.

### Portfolio

Application coordinates incorporation of an approved Trade outcome into the relevant Portfolio. Portfolio owns mutable ownership interpretation, its Position collection, and its aggregate invariants. Application does not calculate or independently store a competing ownership interpretation.

### Cross-Context Coordination

Application may coordinate these existing concepts in an approved sequence:

```text
Strategy decision
  ↓
Order intent
  ↓
External execution interaction
  ↓
Confirmed Trade execution truth
  ↓
Portfolio ownership interpretation
```

This sequence does not transfer ownership between bounded contexts. It only coordinates interactions among their existing authoritative concepts.

---

## 5. Infrastructure Relationship

Infrastructure provides technical implementations required by Application. Application depends on abstractions; Infrastructure supplies implementations.

### Outbound Ports

Outbound ports describe capabilities Application calls beyond its boundary. They may represent persistence, broker communication, exchange communication, event delivery, authentication, scheduling, or other external needs. An outbound port is defined by the capability Application requires, not by a vendor, wire protocol, or database engine.

### Inbound Ports

Inbound ports describe Application capabilities offered to delivery mechanisms. They provide a stable boundary through which API adapters, web-facing adapters, scheduled jobs, command-line tools, or event consumers can invoke an application use case.

Inbound ports describe application intent and outcome. They do not expose Domain internals as a delivery contract and do not make Application responsible for transport implementation.

### Infrastructure Implementations

Infrastructure implements outbound ports because it owns concrete technical details, including:

- broker and exchange adapters;
- FIX connectivity and protocol handling;
- REST and WebSocket clients or servers;
- persistence engines and database mappings;
- message transports and delivery mechanisms;
- authentication and authorization integrations; and
- networking, retries, configuration, observability, and runtime concerns.

Application does not implement these concerns because doing so would couple use-case intent to replaceable technical choices. Infrastructure does not own ports' application intent or their Domain meaning; it only fulfills the required capability.

---

## 6. Execution Coordination

Execution is an Application capability with Infrastructure support. It is neither a Domain bounded context nor an owner of a new execution aggregate.

### Where Orchestration Begins

Execution orchestration begins when an Application use case receives approved Order intent that must be acted upon externally. The Application Layer coordinates retrieval of the relevant Order, invocation of approved Order behavior, transaction scope, and the outbound interaction required to seek execution.

### Where Domain Execution Ends

Domain responsibility ends at the meaning and validity of its existing concepts:

- Strategy owns decision policy.
- Order owns participant intent and Order lifecycle meaning.
- Trade owns confirmed immutable execution truth.
- Portfolio owns current ownership interpretation.

Domain does not choose a broker protocol, submit network requests, track wire-level acknowledgements, or execute a cross-context process.

### Where Infrastructure Begins

Infrastructure begins when Application invokes an outbound broker or exchange port. Infrastructure selects and runs the concrete adapter and protocol mechanism, such as FIX, REST, or WebSocket, then translates the technical outcome into the Application-facing result required by the use case.

### Coordinated Execution Flow

```text
Application receives approved Order intent
  ↓
Application invokes Orders Domain behavior
  ↓
Application invokes broker or exchange outbound port
  ↓
Infrastructure performs technical communication
  ↓
Application receives confirmed outcome
  ↓
Application invokes Trades Domain behavior
  ↓
Application invokes Portfolio Domain behavior as required
```

Application coordinates this flow without redefining Order intent, broker semantics, Trade truth, or Portfolio ownership.

---

## 7. Transaction Boundaries

A transaction boundary is the Application-defined scope within which a use case coordinates state changes and their observable consequences. Its purpose is to preserve a coherent application outcome while respecting the fact that each Aggregate Root owns its own consistency boundary.

Application transaction boundaries:

- establish when a use case begins and completes;
- coordinate loading and persistence through outbound ports;
- determine when resulting Domain events may be dispatched;
- make explicit where atomic completion is required; and
- identify where an approved compensating workflow is necessary when an external operation cannot be atomic.

A transaction boundary does not merge Aggregate Roots into one business owner, weaken aggregate invariants, or move business rules from Domain to Application. It is a coordination boundary, not a replacement for Domain consistency.

No framework, library, storage model, transaction manager, or messaging mechanism is prescribed by this specification.

---

## 8. Domain Events

Domain events record approved business facts that occur within the Domain. The Domain publishes or records the event as part of its business behavior because the fact belongs to Domain meaning.

Application owns coordination of reactions after the relevant Domain work succeeds. The Domain Event Dispatcher may initiate further Application workflows or call outbound ports according to the approved use case. Infrastructure fulfills technical delivery when an event must leave the Application boundary.

```text
Domain establishes business fact
  ↓
Domain produces Domain event
  ↓
Application transaction completes
  ↓
Application dispatches approved reactions
  ↓
Infrastructure delivers technical integration when required
```

This separation preserves Domain purity. Domain does not know queues, topics, buses, webhooks, consumers, or delivery guarantees. Application owns which approved reaction is coordinated; Infrastructure owns how a technical message is delivered.

This specification does not prescribe messaging technologies, delivery semantics, or event-bus implementations.

---

## 9. Ports

Ports are the dependency-inversion boundary between Application intent and external capability.

They belong to the Application Layer because Application determines what capability a use case needs. A port expresses that requirement as an abstraction independent of broker vendor, exchange, protocol, persistence engine, authentication provider, or message transport.

Application depends on port abstractions. Infrastructure supplies their concrete implementations. The Composition Root selects and wires those implementations into the Application runtime.

Ports must not:

- expose infrastructure-specific types as Application concepts;
- embed a vendor, protocol, database, or messaging contract into Application intent;
- move Domain invariants into adapter interfaces; or
- provide a route around Aggregate Root behavior.

---

## 10. Dependency Rules

Northstar uses inward dependency and mandatory dependency inversion.

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

The required rules are:

- Domain depends on nothing upward. It never depends on Application, Infrastructure, API, web, transport, persistence, messaging, protocols, or frameworks.
- Application depends on Domain abstractions and Application-owned port abstractions.
- Infrastructure depends inward to implement Application ports and adapt external technologies.
- Inbound adapters invoke Application through inbound ports. They do not bypass Application to mutate Domain state directly.
- The Composition Root owns dependency wiring. Neither Domain nor Application selects a concrete Infrastructure implementation.

Dependency inversion protects business meaning from technology churn. Replacing a broker, database, protocol library, message transport, or authentication provider must not require redesigning Domain behavior or Application use-case intent.

---

## 11. DDD Review

### Ownership

Ownership is preserved. Domain bounded contexts retain ownership of all business concepts. Application owns only the process of coordinating a use case and any necessary workflow state. Infrastructure owns only technical implementation.

### Orchestration

Cross-context sequencing, Execution coordination, persistence timing, external capability invocation, and post-Domain reactions are correctly placed in Application. No Domain aggregate is made responsible for workflow that exceeds its owned consistency boundary.

### Dependency Inversion

Ports make Application dependent on abstractions instead of technical implementations. Infrastructure fulfills those abstractions and remains replaceable. Domain remains free of Infrastructure knowledge.

### Separation of Concerns

Strategy decides, Orders express intent, Trades preserve execution truth, and Portfolio interprets ownership. Application coordinates these meanings. Infrastructure integrates with the outside world. The separation is explicit and preserves the frozen Domain architecture.

### Domain Purity

Domain purity remains intact because Domain contains no broker adapters, exchange clients, protocol handling, persistence code, messaging technology, application workflow, or Composition Root wiring.

---

## 12. Engineering Principles

1. **Orchestration only.** Application coordinates approved behavior; it does not own business meaning.
2. **Use-case orientation.** Application modules are organized around meaningful application intents and outcomes, not technical transports or database tables.
3. **Domain decides.** Application invokes Domain behavior and respects its invariants, identities, and lifecycle semantics.
4. **No business ownership.** Application process state is not a replacement for Domain-owned state.
5. **No infrastructure logic.** Protocols, persistence engines, message transports, and vendor adapters remain in Infrastructure.
6. **Dependency inversion.** Application names required capabilities through ports; Infrastructure implements them.
7. **Framework independence.** Application design remains independent of frameworks and libraries so that implementation choices do not define use-case meaning.
8. **Aggregate respect.** Application never bypasses Aggregate Roots or directly mutates internal Domain state.
9. **Explicit boundaries.** Transactions, event reactions, external calls, and ownership transitions are visible in use-case design.
10. **Minimal coordination.** Application introduces only the workflow required to coordinate existing concepts and avoids generic abstractions without approved need.

---

## 13. Anti-Patterns

The Application Layer must never:

- own business invariants, identity rules, lifecycle semantics, or Domain meaning;
- implement broker protocols, exchange protocols, FIX sessions, REST clients, or WebSocket clients;
- contain database mappings, persistence-engine logic, message-transport implementation, or network-retry mechanisms;
- duplicate or reinterpret Domain behavior for convenience;
- bypass Aggregate Roots to mutate Domain-owned state;
- create an alternate source of truth for Order, Trade, Portfolio, Strategy, or Market Data;
- define a new Domain model for Execution;
- depend on Infrastructure implementations or vendor-specific types;
- select concrete Infrastructure adapters outside the Composition Root;
- publish technical messages directly from Domain;
- treat an external acknowledgement as completed Trade truth without invoking the Trades Domain model; or
- merge cross-context workflow into a single aggregate or bounded context without approved architectural change.

---

## 14. Self Review

- Domain remains unchanged and retains business meaning.
- Infrastructure remains unchanged and retains technical implementation ownership.
- Application owns orchestration only.
- Execution remains an Application capability with Infrastructure support.
- Dependency inversion is preserved through Application-owned ports and Infrastructure implementations.
- Aggregate Root boundaries are respected.
- No implementation assumptions, frameworks, libraries, protocols, or concrete adapters are prescribed.
- The specification is suitable as the engineering guide for future Application module design.

---

APPLICATION LAYER

Design Specification

Approved

Ready for Application Module Design

---
