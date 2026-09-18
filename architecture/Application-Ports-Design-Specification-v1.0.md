# Application Ports Design Specification v1.0

## Status

**Approved application-design specification for Northstar Platform v1.1.0.**

This document defines the canonical Application Port model for Northstar. It is governed by Architecture Handbook v2.0, Application Layer Design Specification v1.0, Application Contracts Design Specification v1.0, and the approved Execution specifications.

Application Ports define required capabilities only. They do not implement those capabilities, select technologies, define Domain behavior, or expose Infrastructure representations. Infrastructure supplies concrete implementations through the Composition Root.

---

## 1. Application Port Purpose

An Application Port is an Application-owned abstraction that expresses a capability required by an Application use case beyond its immediate orchestration boundary.

Ports exist to preserve dependency inversion. Application identifies what capability it needs, such as storing an Order, submitting an approved request to a broker, or delivering an approved event. Infrastructure implements how that capability is fulfilled.

Ports belong to the Application Layer because their purpose is determined by Application use-case coordination. They remain independent of Infrastructure because a port describes a required capability, not a database, broker SDK, exchange vendor, HTTP client, FIX session, WebSocket connection, queue, or framework.

```text
Application use case
  ↓ depends on
Application Port
  ↓ implemented by
Infrastructure Adapter
```

A Port does not implement a capability. It does not own a business concept, contain business rules, or become an Infrastructure DTO boundary.

---

## 2. Port Family

The canonical port families required for the Execution Application Module are:

1. Broker Port
2. Exchange Port
3. Order Persistence Port
4. Trade Recording Port
5. Portfolio Update Port
6. Event Publisher Port
7. Transaction Port

These families are sufficient for the approved Execution workflows. No additional baseline Port family is required.

Authentication, scheduling, idempotency persistence, configuration, observability, and read-model needs may require future Application-specific Port designs when an approved use case demonstrates a real boundary need. They are not introduced as generic Ports in this baseline because they would add abstraction without an approved Execution responsibility.

Each Port must remain narrowly responsible for one external capability. A Port family does not imply a shared technical implementation, a shared protocol, or a shared Domain owner.

---

## 3. Broker Port

### Capability

The Broker Port provides the capability to request broker-facing execution actions and receive normalized Application-level results. Its role is to support the submission, cancellation, replacement, and result-processing workflows coordinated by Execution.

### Application Responsibility

Application uses the Broker Port after invoking the required Domain behavior. The Port accepts and returns Application Contracts that convey execution workflow intent, correlation, Results, and Failures.

### What It Must Never Own

The Broker Port must never own:

- Order intent, Order lifecycle meaning, or Order invariants;
- Trade creation or Trade execution truth;
- Portfolio ownership interpretation;
- broker selection policy unless separately designed as an Application use-case responsibility;
- broker SDKs, credentials, protocols, connection management, or vendor-specific types; or
- a direct path that bypasses the Orders Domain aggregate.

Infrastructure adapters own broker protocol, vendor, and connection implementation.

---

## 4. Exchange Port

### Capability

The Exchange Port provides the capability to request exchange-facing actions and receive normalized Application-level results when the exchange is the relevant external execution boundary.

### Distinction from Broker Port

A Broker Port represents the capability of interacting with a broker as an execution intermediary. An Exchange Port represents the capability of interacting with an exchange venue directly.

The distinction is architectural, not technical: the Port selected by a use case reflects the external capability it needs. Neither Port owns exchange market meaning, Order validity, Trade truth, or Portfolio interpretation. Both remain independent of a protocol or vendor.

### What It Must Never Own

The Exchange Port must not contain exchange APIs, venue-specific identifiers, FIX details, REST or WebSocket mechanics, matching behavior, market-data ownership, or Domain business rules. These concerns remain in Infrastructure or their owning Domain contexts.

---

## 5. Order Persistence Port

### Capability

The Order Persistence Port provides the capability to retrieve and persist Order aggregate state required by an Application workflow.

### Responsibility

Application requests an Order aggregate through the Port, invokes approved Order behavior, and requests persistence of the resulting aggregate state. The Port supports that use-case need without defining any Order business behavior.

Infrastructure performs persistence. It owns database connections, queries, mappings, schemas, transactions at the technical level, and storage technology. The Port must not expose any of those details to Application.

### Boundary

The Port never validates Order intent, determines a lifecycle transition, constructs a substitute Order representation, or bypasses the Order aggregate.

---

## 6. Trade Recording Port

### Capability

The Trade Recording Port provides the capability to record or persist a Trade aggregate after the Trades Domain model has established approved execution truth.

### Responsibility

Application invokes approved Trades Domain behavior and requests Trade recording through the Port. The Port supports making the resulting immutable Trade available beyond the immediate Application workflow without constructing or interpreting it.

### Domain Boundary

Trade business rules, identity, immutability, and the decision that confirmed execution information becomes Trade truth remain entirely inside Trades Domain. The Port never validates Trade meaning, creates a competing Trade state, or treats a technical response as a Trade.

Infrastructure owns the actual recording or persistence mechanism.

---

## 7. Portfolio Update Port

### Capability

The Portfolio Update Port provides the capability to retrieve and persist Portfolio aggregate state required to coordinate an approved Trade outcome with Portfolio.

### Responsibility

Application uses the Port to obtain the relevant Portfolio, invokes approved Portfolio behavior, and requests persistence of the resulting state. This supports the execution workflow without moving ownership interpretation outside Portfolio.

### Domain Boundary

Application coordinates; Portfolio decides. The Port never calculates holdings, changes a Position directly, enforces the one-Position-per-Listing invariant, or constructs an alternate Portfolio representation.

Infrastructure owns the concrete retrieval and persistence implementation.

---

## 8. Event Publisher Port

### Capability

The Event Publisher Port provides the capability to deliver approved post-transaction event reactions beyond the Application boundary.

### Responsibility

Application requests publication only after the originating Domain behavior has completed and the relevant transaction boundary has reached its approved completion point. The Domain Event Dispatcher coordinates which approved reactions are routed to the Port.

Infrastructure performs technical delivery, including transport connection, serialization, routing, queue or topic interaction, retries, and delivery mechanics.

### Boundary

The Event Publisher Port does not invent Domain events, alter event meaning, define event payloads, select a messaging technology, or make technical delivery a Domain responsibility.

---

## 9. Transaction Port

### Capability

The Transaction Port provides the capability needed to coordinate the completion, rollback, and approved compensation scope of an Application use case.

### Responsibility

Application determines when an execution workflow begins, when its Domain changes are ready for completion, and when events may become eligible for dispatch. The Transaction Port supports that coordination without defining Aggregate Root consistency or transaction technology.

Infrastructure supplies the concrete transaction mechanism. Database technology, transaction manager behavior, storage scope, and other technical mechanisms remain Infrastructure concerns.

### Boundary

The Transaction Port does not merge Aggregate Roots, relax their invariants, decide business validity, or make an external broker acknowledgement equivalent to completed Trade truth. When external work cannot be atomic, Application coordinates an approved compensation workflow.

---

## 10. Application Contract Relationship

Ports consume and produce Application Contracts.

- Inbound Ports accept Application Requests and return Application Responses or Outcomes.
- Outbound Ports accept Application Contracts expressing a required external capability and return Application Results or Failures.

Ports must never expose Domain entities, Aggregate Roots, mutable aggregate state, or Infrastructure DTOs as their boundary model. They must not expose broker messages, exchange responses, FIX tags, REST payloads, WebSocket frames, database rows, ORM models, queue messages, or vendor types.

Application Contracts remain the stable coordination language. Domain concepts are referenced only through approved identity or association semantics, and Infrastructure adapters translate technical representations at the edge.

---

## 11. Domain Relationship

Application Ports interact with Domain only through Application use cases and approved Domain abstractions.

A Port exists because Application needs a capability to coordinate a Domain workflow. It does not own the Domain concept associated with that capability:

- Broker and Exchange Ports do not own Order, Trade, or Portfolio meaning.
- Order Persistence Port does not own Order state or lifecycle.
- Trade Recording Port does not own Trade truth or immutability.
- Portfolio Update Port does not own Portfolio interpretation or Position consistency.
- Event Publisher Port does not own Domain-event meaning.
- Transaction Port does not own aggregate consistency or business validity.

Domain remains independent of all Ports. Domain Aggregates decide their own behavior; Application invokes that behavior and uses Ports to fulfill external capabilities.

---

## 12. Infrastructure Relationship

Infrastructure implements Application Ports. Each Infrastructure adapter translates between an Application Contract and the concrete technology required to fulfill the Port's capability.

```text
Application Port
  ↓ implemented by
Infrastructure adapter
  ↓ translates to and from
Technical protocol, storage, or transport
```

A Port remains technology-independent even when its Infrastructure adapter uses a specific broker SDK, exchange API, FIX engine, REST client, WebSocket library, database, or message transport.

Infrastructure must not redefine a Port's application purpose, leak its types into Application, or use a Port to bypass the owning Domain aggregate. The Composition Root selects and wires Infrastructure implementations to Application Ports.

---

## 13. Engineering Principles

1. **Dependency inversion.** Application depends on Port abstractions; Infrastructure supplies implementations.
2. **Interface segregation.** Each Port describes one focused capability and does not become a broad technical service boundary.
3. **Technology independence.** Ports express capability without naming vendors, databases, protocols, or transport systems.
4. **Framework independence.** Port meaning does not depend on web, ORM, messaging, or runtime frameworks.
5. **Protocol independence.** Port contracts do not contain FIX, REST, WebSocket, or broker-specific semantics.
6. **Single responsibility.** A Port owns only the external capability requested by its Application use case.
7. **Application Contract boundary.** Ports consume and produce Application Contracts, not Domain entities or Infrastructure DTOs.
8. **No business ownership.** Ports do not own Domain invariants, lifecycle rules, identity semantics, or business state.
9. **No technical implementation.** Ports define required behavior only; adapters implement technology.
10. **Minimal abstraction.** A new Port requires an approved Application use-case need that cannot be served by an existing focused Port.

---

## 14. Anti-Patterns

Application Ports must never:

- include broker SDKs, exchange SDKs, or vendor-specific types;
- contain SQL, database queries, ORM models, persistence mappings, or schema definitions;
- contain REST routes, HTTP request or response models, WebSocket frames, or FIX messages;
- contain queue, topic, bus, serializer, or transport implementation;
- implement business validation, lifecycle transitions, aggregate invariants, or strategy decisions;
- expose Domain aggregates, entities, mutable state, or direct aggregate mutation paths;
- expose Infrastructure DTOs, database rows, protocol errors, connection details, or adapter configuration;
- create a generic technical gateway that combines unrelated capabilities;
- allow Application to select a concrete Infrastructure implementation; or
- duplicate the ownership of Orders, Trades, Portfolio, Strategy, or Market Data.

---

## 15. Implementation Readiness

Application Ports are fully specified at the Application design level.

The approved Port model defines the required Execution Port families, the capability and responsibility of each, their relationship to Application Contracts, and their required Domain and Infrastructure boundaries.

The following remain intentional implementation details, subject to this specification:

- concrete Port names and interface definitions;
- method and parameter shapes;
- concrete Application Contract types and mappings;
- adapter registration and Composition Root wiring;
- technical error translation and retry mechanisms; and
- transaction, storage, protocol, and messaging technologies.

**Application Port implementation may begin.**

---

## 16. Self Review

- Matches Architecture Handbook v2.0.
- Matches Application Layer Design Specification v1.0.
- Matches Application Contracts Design Specification v1.0.
- Supports the approved Execution Application Module and Execution Coordinator designs.
- Domain remains unchanged and retains all business meaning.
- Infrastructure remains unchanged and retains all technical implementation ownership.
- Ports own required capabilities only and do not implement them.
- No concrete interface, technology, framework, protocol, schema, or adapter assumption is introduced.

---

APPLICATION PORTS

Design Specification

Approved

Ready for Implementation

---
