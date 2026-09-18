# Execution Application Ports Design Specification v1.0

## Status

**Approved application-design specification for Northstar Platform v1.1.0.**

This document defines the concrete Application Ports required by the Execution Application Module. It is governed by Architecture Handbook v2.0, Application Layer Design Specification v1.0, Application Contracts Design Specification v1.0, Execution Application Contracts Design Specification v1.0, and Application Ports Design Specification v1.0.

Execution Ports are Application-owned abstractions. They are the canonical orchestration boundary between the Execution Coordinator and Infrastructure. They define required capabilities only; Infrastructure supplies concrete implementations through the Composition Root.

This specification does not define methods, interfaces, fields, technical protocols, frameworks, storage mechanisms, message transports, or adapter implementations.

---

## 1. Port Set

The complete concrete Execution Port set is:

- BrokerPort;
- ExchangePort;
- OrderPersistencePort;
- TradeRecordingPort;
- PortfolioUpdatePort;
- EventPublisherPort; and
- TransactionPort.

ExchangePort is included because it is part of the already-approved generic Application Port family. A particular Execution use case selects BrokerPort or ExchangePort according to the external capability it requires; neither Port is a technical implementation or a Domain owner.

No additional Execution Port is required. A new Port may be introduced only through approved Application design when a new Execution use case has a focused external capability that cannot be fulfilled by this set.

---

## 2. BrokerPort

BrokerPort provides the Application capability to request broker-facing execution actions and receive normalized Application-level results.

### Coordinator Invocation

The Execution Coordinator invokes BrokerPort after the relevant Order Aggregate behavior has been invoked and the Application workflow requires a broker as the external execution intermediary. BrokerPort supports submission, cancellation, replacement, and broker-result processing workflows.

### Contract Relationship

BrokerPort consumes the applicable Execution Application Contract that expresses the requested broker-facing coordination, including the relevant ExecutionRequest or derived Application workflow intent and correlation. It produces an ExecutionResult for a normalized result or an ExecutionFailure for a technical, timeout, uncertain, or broker-reported failure.

BrokerPort does not produce ExecutionResponse or ExecutionOutcome. The Execution Coordinator produces the final Outcome and Response after coordinating all required Domain and Application work.

### Boundary

BrokerPort does not own:

- Order intent, OrderStatus meaning, lifecycle rules, or invariants;
- Trade execution truth;
- Portfolio ownership interpretation;
- broker selection policy unless separately approved as an Application use-case responsibility; or
- broker SDKs, credentials, protocols, connection management, or vendor-specific types.

Infrastructure implements broker connectivity and translates between Application Contracts and broker-specific representations.

---

## 3. ExchangePort

ExchangePort provides the Application capability to request exchange-facing execution actions and receive normalized Application-level results when a use case requires direct exchange interaction.

### Coordinator Invocation

The Execution Coordinator invokes ExchangePort when the approved execution workflow requires an exchange venue as its external boundary rather than a broker intermediary.

### Contract Relationship

ExchangePort consumes the applicable Execution Application Contract carrying the exchange-facing workflow intent and correlation. It produces an ExecutionResult for a normalized result or an ExecutionFailure for an external technical, timeout, uncertain, or exchange-reported failure.

ExecutionResponse and ExecutionOutcome remain Coordinator responsibilities. ExchangePort does not establish Trade truth.

### Boundary

ExchangePort is distinct from BrokerPort by the external capability it represents: direct venue interaction rather than broker-mediated interaction. It does not own market-data meaning, exchange business semantics, matching logic, Order validity, Trade truth, Portfolio interpretation, venue APIs, identifiers, or protocol behavior.

Infrastructure implements concrete exchange connectivity and technical translation.

---

## 4. OrderPersistencePort

OrderPersistencePort provides the capability to retrieve and persist the Order aggregate state required by an Execution workflow.

### Coordinator Invocation

The Execution Coordinator invokes OrderPersistencePort to obtain the relevant Order before requesting Domain behavior and to coordinate persistence after approved Order behavior changes aggregate state.

### Contract Relationship

OrderPersistencePort consumes the relevant Execution Application Contract expressing the required coordination context. It produces an ExecutionResult when the Application persistence step completes as required or an ExecutionFailure when technical persistence cannot complete.

It does not produce Order state as an Application Contract substitute, nor does it produce an ExecutionOutcome or ExecutionResponse.

### Boundary

Application requests persistence; Infrastructure performs persistence. OrderPersistencePort does not validate Order intent, decide Order lifecycle transitions, expose a database representation, create a substitute Order model, or bypass the Order Aggregate Root.

Orders Domain remains the owner of Order business meaning, identity, lifecycle, and consistency. Infrastructure owns database connections, queries, mappings, schemas, and storage technology.

---

## 5. TradeRecordingPort

TradeRecordingPort provides the capability to record or persist a Trade after the Trades Domain model has established approved execution truth.

### Coordinator Invocation

The Execution Coordinator invokes TradeRecordingPort only after it has coordinated confirmed execution information with the Trades Domain model and approved Domain behavior has established a Trade.

### Contract Relationship

TradeRecordingPort consumes the applicable Execution Application Contract expressing the recording coordination context. It produces an ExecutionResult when recording completes as required or an ExecutionFailure when a technical recording failure prevents completion.

The Port does not produce Trade truth, an ExecutionOutcome, or an ExecutionResponse.

### Boundary

Application coordinates recording; the Trade Aggregate remains responsible for business truth. TradeRecordingPort does not create, validate, modify, or reinterpret a Trade. It never treats a broker response, exchange response, or technical record as a Trade substitute.

Infrastructure owns the concrete recording or persistence mechanism.

---

## 6. PortfolioUpdatePort

PortfolioUpdatePort provides the capability to retrieve and persist Portfolio aggregate state required to coordinate an approved Trade outcome with Portfolio.

### Coordinator Invocation

The Execution Coordinator invokes PortfolioUpdatePort to obtain the relevant Portfolio before calling approved Portfolio behavior and to coordinate persistence after Portfolio has interpreted an approved Trade outcome.

### Contract Relationship

PortfolioUpdatePort consumes the applicable Execution Application Contract expressing the Portfolio-update coordination context. It produces an ExecutionResult when required state coordination completes or an ExecutionFailure when technical retrieval or persistence cannot complete.

The Port does not create an ExecutionOutcome or ExecutionResponse. It neither produces nor contains Portfolio ownership interpretation.

### Boundary

Application coordinates update requests; the Portfolio Aggregate owns ownership interpretation. PortfolioUpdatePort does not calculate holdings, modify a Position directly, enforce Portfolio invariants, or create an alternate Portfolio representation.

Infrastructure owns the concrete retrieval and persistence implementation.

---

## 7. EventPublisherPort

EventPublisherPort provides the capability to deliver approved post-transaction event reactions beyond the Application boundary.

### Coordinator Invocation

The Execution Coordinator makes event reactions eligible only after the originating Domain work has completed and the relevant transaction scope reaches its approved completion point. The Domain Event Dispatcher coordinates routing; it invokes EventPublisherPort when an approved reaction must leave the Application boundary.

### Contract Relationship

EventPublisherPort consumes the applicable Execution Application Contract representing an approved publication coordination request. It produces an ExecutionResult when the required delivery coordination completes or an ExecutionFailure when technical delivery cannot complete.

It does not define, create, change, or interpret Domain events. It does not produce an ExecutionOutcome or ExecutionResponse.

### Boundary

Application requests publication; Infrastructure performs delivery. EventPublisherPort does not select messaging technology, define payloads, serialize events, connect to queues or topics, or make technical delivery a Domain responsibility.

---

## 8. TransactionPort

TransactionPort provides the capability needed to coordinate the completion, rollback, and approved compensation scope of an Execution use case.

### Coordinator Invocation

The Execution Coordinator invokes TransactionPort when beginning, completing, rolling back, or coordinating an approved compensation boundary for an Execution workflow. Application determines the conceptual transaction scope and event eligibility; Infrastructure supplies the technical transaction implementation.

### Contract Relationship

TransactionPort consumes the applicable Execution Application Contract expressing the required transaction coordination. It produces an ExecutionResult when the requested coordination completes or an ExecutionFailure when technical transaction coordination cannot complete.

It does not determine the final ExecutionOutcome or produce ExecutionResponse. The Coordinator determines the use-case conclusion after considering Domain behavior and the applicable port Results or Failures.

### Boundary

TransactionPort does not merge Aggregate Roots, decide business validity, relax invariants, select a transaction framework, or make a broker or exchange acknowledgement equivalent to Trade truth. When external interaction cannot share an atomic boundary, the Coordinator coordinates an approved compensation workflow.

---

## 9. Execution Contract Relationship

All Execution Ports use the same approved Execution Application Contract family.

| Execution Port       | Consumes                                         | Produces                            | Does not produce                                              |
| -------------------- | ------------------------------------------------ | ----------------------------------- | ------------------------------------------------------------- |
| BrokerPort           | ExecutionRequest or applicable workflow contract | ExecutionResult or ExecutionFailure | ExecutionOutcome, ExecutionResponse, Trade                    |
| ExchangePort         | ExecutionRequest or applicable workflow contract | ExecutionResult or ExecutionFailure | ExecutionOutcome, ExecutionResponse, Trade                    |
| OrderPersistencePort | Applicable execution coordination contract       | ExecutionResult or ExecutionFailure | Order substitute, ExecutionOutcome, ExecutionResponse         |
| TradeRecordingPort   | Applicable execution recording contract          | ExecutionResult or ExecutionFailure | Trade truth, ExecutionOutcome, ExecutionResponse              |
| PortfolioUpdatePort  | Applicable Portfolio-update contract             | ExecutionResult or ExecutionFailure | Portfolio interpretation, ExecutionOutcome, ExecutionResponse |
| EventPublisherPort   | Approved publication coordination contract       | ExecutionResult or ExecutionFailure | Domain-event meaning, ExecutionOutcome, ExecutionResponse     |
| TransactionPort      | Applicable transaction coordination contract     | ExecutionResult or ExecutionFailure | Business decision, ExecutionOutcome, ExecutionResponse        |

ExecutionRequest initiates or continues coordination. Ports consume the appropriate request or workflow context and return step-level ExecutionResult or ExecutionFailure. The Execution Coordinator evaluates these results, invokes the necessary Domain behavior, determines ExecutionOutcome, and returns ExecutionResponse to the inbound boundary.

No Port may expose Domain entities, Aggregate Roots, mutable aggregate state, Infrastructure DTOs, broker messages, exchange messages, database rows, HTTP models, protocol frames, or vendor-specific types as its contract boundary.

---

## 10. Domain Relationship

Execution Ports support Application workflows that invoke Domain behavior. They never own business meaning.

### Orders

BrokerPort, ExchangePort, and OrderPersistencePort support workflows involving an Order, but none owns Order intent, lifecycle, identity, state, or invariants. The Execution Coordinator invokes approved Order Aggregate behavior before and during the relevant coordination path.

### Trades

TradeRecordingPort supports recording after Trades Domain establishes immutable execution truth. BrokerPort and ExchangePort may report normalized external results, but neither external result nor any Port establishes a Trade.

### Portfolio

PortfolioUpdatePort supports obtaining and persisting Portfolio state around an approved Portfolio Aggregate operation. It does not calculate ownership, mutate Position state directly, or enforce Portfolio consistency.

### Strategy

Execution Ports do not evaluate Strategy policy or make Strategy decisions. An Execution workflow may originate from approved Strategy context, but Strategy remains entirely Domain-owned.

Domain remains independent of all Execution Ports. Ports invoke required external capabilities; they never own the concepts those capabilities help Application coordinate.

---

## 11. Infrastructure Relationship

Infrastructure implements Execution Ports.

```text
Execution Coordinator
  ↓ calls
Execution Port
  ↓ implemented by
Infrastructure adapter
  ↓ translates to and from
Technical protocol, storage, or transport
```

Each adapter translates the relevant Execution Application Contract to and from the concrete technology required to fulfill its focused Port capability. Infrastructure may use a broker SDK, exchange API, FIX engine, REST client, WebSocket library, database, message transport, or transaction manager without allowing those types or semantics to enter the Port boundary.

Execution Ports remain technology-independent, protocol-independent, and framework-independent. The Composition Root selects and wires concrete Infrastructure implementations. The Execution Coordinator and Domain never select or depend on them.

---

## 12. Dependency Review

The required dependency direction is:

```text
Execution Coordinator
  ↓
Execution Application Ports
  ↓ implemented by
Infrastructure adapters

Execution Coordinator
  ↓ invokes
Domain Aggregate Roots

Domain
  ↓
remains independent
```

Execution Ports are Application abstractions. The Coordinator depends on those abstractions and on approved Domain boundaries. Infrastructure depends inward to implement the Ports. Domain does not depend on the Coordinator, Execution Contracts, Ports, or Infrastructure.

This preserves dependency inversion and prevents protocol, storage, or vendor concerns from entering Application orchestration or Domain business meaning.

---

## 13. Engineering Principles

1. **Single responsibility.** Each Execution Port represents one focused external capability.
2. **Dependency inversion.** The Execution Coordinator depends on Port abstractions; Infrastructure supplies implementations.
3. **Interface segregation.** Port responsibilities remain narrow and do not combine unrelated technical capabilities.
4. **Technology independence.** Ports do not name or require vendors, databases, transports, or runtime systems.
5. **Protocol independence.** Ports contain no broker protocol, exchange protocol, FIX, REST, or WebSocket semantics.
6. **Framework independence.** Port contracts do not depend on application frameworks, ORMs, transaction managers, or messaging frameworks.
7. **Application Contract boundary.** Ports consume and produce only approved Execution Application Contracts.
8. **Orchestration support only.** Ports support Execution coordination and never own Domain decisions or business state.
9. **Aggregate respect.** Ports cannot bypass Aggregate Roots or directly mutate Domain-owned state.
10. **Minimal abstraction.** No new Port is added without an approved focused Application capability.

---

## 14. Anti-Patterns

Execution Application Ports must never:

- contain broker or exchange SDKs, credentials, connection management, or vendor-specific types;
- contain SQL, database queries, ORM models, persistence mappings, schemas, or storage configuration;
- contain REST routes, HTTP models, WebSocket frames, FIX messages, or protocol handling;
- contain queue, topic, bus, serializer, retry, or transport implementation;
- implement Domain business rules, validation, lifecycle transitions, identity semantics, aggregate invariants, or Strategy decisions;
- expose Domain entities, Aggregate Roots, mutable aggregate state, or direct aggregate mutation paths;
- expose Infrastructure DTOs, database rows, protocol errors, connection details, or adapter configuration;
- construct or reinterpret Trade truth;
- directly modify Portfolio or Position state;
- merge broker, exchange, persistence, event, and transaction concerns into one generic gateway; or
- select concrete Infrastructure implementations outside the Composition Root.

---

## 15. Implementation Readiness

Execution Application Ports are fully specified at the Application design level.

The specification defines the complete concrete Port set, each Port's capability, invocation point, contract inputs and outputs, Domain boundary, Infrastructure responsibility, dependency direction, and prohibited concerns.

The following remain intentional implementation details that must conform to this specification:

- interface and method definitions;
- concrete Execution Contract mappings;
- error and result representations;
- adapter registration and Composition Root wiring;
- technical transaction and persistence mechanisms; and
- protocol, transport, messaging, and vendor choices.

**Execution Port implementation may begin.**

---

## 16. Self Review

- Matches Architecture Handbook v2.0.
- Matches Application Layer Design Specification v1.0.
- Matches Application Contracts and Execution Application Contracts designs.
- Matches Application Ports Design Specification v1.0.
- Domain remains unchanged and retains all business meaning.
- Infrastructure remains unchanged and retains all technical implementation ownership.
- Execution Ports own required capabilities only and do not implement them.
- No methods, interfaces, frameworks, protocols, storage technologies, message transports, or adapter implementations are prescribed.

---

EXECUTION APPLICATION PORTS

Design Specification

Approved

Ready for Implementation

---
