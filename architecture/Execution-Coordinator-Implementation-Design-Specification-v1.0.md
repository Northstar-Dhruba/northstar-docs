# Execution Coordinator Implementation Design Specification v1.0

## Status

**Approved application-design specification for Northstar Platform v1.1.0.**

This document defines the implementation contracts required to build the Execution Coordinator. It is governed by Architecture Handbook v2.0, Application Layer Design Specification v1.0, and Execution Application Module Design Specification v1.0.

Execution remains an Application capability. This specification defines application-level contracts and behavior only. It does not define concrete classes, interfaces, frameworks, protocol mappings, persistence schemas, message payloads, or Infrastructure implementations.

---

## 1. Design Scope

The Execution Coordinator coordinates the progression from approved Order intent through external execution interaction to confirmed Trade execution truth and Portfolio ownership interpretation.

Its implementation must be mechanical from the following rule:

```text
Application coordinates the sequence.
Domain decides business meaning.
Infrastructure fulfills technical capabilities.
```

The Coordinator must never become an alternate owner of Order, Trade, Portfolio, Strategy, Market Data, or broker meaning.

---

## 2. Inbound Contract

An inbound execution request expresses an application intent to coordinate one approved execution use case. It may originate from an API adapter, scheduled operation, event consumer, or another approved Application workflow.

### Required Information

The request must provide enough information to:

- identify the requested use case: submission, cancellation, replacement, execution-result processing, Trade confirmation, or Portfolio-update coordination;
- identify the existing Domain aggregate or application workflow to which the request applies;
- establish the participant, Listing, and Order context only through approved Domain identities or references where that context is required;
- correlate an external execution result to the relevant application workflow when processing a result;
- establish a stable request identity or equivalent correlation needed to enforce the idempotency contract; and
- carry the application authorization or caller context required by the inbound boundary, when applicable.

The inbound contract expresses intent and correlation. It does not recreate Domain state.

### Prohibited Information

An inbound execution request must not require or carry:

- mutable internal state of Order, Trade, Portfolio, Position, or Strategy;
- a caller-supplied decision that an Order, Trade, or Portfolio change is valid;
- broker protocol messages, FIX tags, REST payloads, WebSocket frames, database rows, or vendor-specific types;
- persistence implementation details;
- infrastructure connection, routing, or adapter-selection details; or
- a direct instruction to bypass an Aggregate Root.

### Ownership Boundary

The inbound request belongs to the Application boundary. It identifies the requested coordination and its context. Domain aggregates remain the source of truth for business identity, state, invariants, and permitted transitions.

---

## 3. Outbound Contracts

The Execution Coordinator requires Application-owned outbound contracts. Each contract names a capability required by a use case; Infrastructure supplies its implementation.

### Order State Contract

**Purpose:** Retrieve and persist Order aggregate state required by an execution workflow.

**Ownership:** Application owns the required capability; Orders Domain owns Order meaning and lifecycle; Infrastructure owns storage implementation.

**Responsibility:** Make the relevant Order aggregate available to the Coordinator and persist only the state resulting from approved Order behavior.

### Broker Interaction Contract

**Purpose:** Request broker-facing submission, cancellation, or replacement and receive an Application-facing technical outcome.

**Ownership:** Application owns the needed external capability; Infrastructure owns broker adapters, vendor behavior, and protocol implementation.

**Responsibility:** Translate an approved application request into broker communication and return a normalized outcome without leaking protocol or vendor details into Application.

### Exchange Interaction Contract

**Purpose:** Request exchange-facing execution actions where an exchange boundary is the relevant capability.

**Ownership:** Application owns the required capability; Infrastructure owns exchange connectivity and protocol implementation.

**Responsibility:** Fulfill the application request and return the result required to continue the approved workflow.

### Trade Recording Contract

**Purpose:** Persist or record an approved Trade after the Trades Domain model establishes execution truth.

**Ownership:** Application owns the persistence capability; Trades Domain owns Trade meaning; Infrastructure owns the recording implementation.

**Responsibility:** Record only a Trade established through approved Domain behavior. It must not construct, validate, or reinterpret Trade truth.

### Portfolio State Contract

**Purpose:** Retrieve and persist Portfolio aggregate state needed to incorporate an approved Trade outcome.

**Ownership:** Application owns the needed capability; Portfolio Domain owns ownership interpretation; Infrastructure owns storage implementation.

**Responsibility:** Make the Portfolio aggregate available and persist only the state resulting from approved Portfolio behavior.

### Event Publication Contract

**Purpose:** Deliver approved, post-transaction application reactions beyond the Coordinator boundary.

**Ownership:** Application owns reaction coordination; Domain owns Domain-event meaning; Infrastructure owns messaging delivery.

**Responsibility:** Deliver only events approved for publication after the corresponding Domain and transaction work completes.

### Transaction Coordination Contract

**Purpose:** Establish and complete the persistence scope needed for an execution use case.

**Ownership:** Application owns transaction-boundary semantics; Infrastructure owns concrete transaction technology.

**Responsibility:** Support completion, rollback, or approved compensation coordination without imposing a framework or storage model on the Coordinator.

---

## 4. Workflow Contract

The canonical execution workflow is:

```text
Execution Request
  ↓
Load Order
  ↓
Invoke approved Order behavior
  ↓
Submit through Broker or Exchange Port
  ↓
Receive normalized execution result
  ↓
Invoke Trades Domain behavior for confirmed outcome
  ↓
Persist approved Trade
  ↓
Load and invoke Portfolio Domain behavior
  ↓
Persist approved Portfolio state
  ↓
Coordinate Domain-event reactions
```

### Coordination Begins

Coordination begins when a valid inbound execution request reaches the Execution Coordinator. The Coordinator identifies the requested use case, obtains the required aggregates through Application contracts, and opens the appropriate transaction or workflow scope.

### Domain Work

The Coordinator invokes the owning aggregate for every business change:

- Order decides participant intent and lifecycle validity.
- Trade decides whether confirmed information becomes immutable execution truth.
- Portfolio decides how a Trade outcome changes ownership interpretation and Position consistency.
- Strategy remains the owner of decision policy when execution is initiated from a Strategy-oriented workflow.

### External Interaction

The Coordinator requests broker or exchange communication through an outbound port only after the appropriate application and Domain steps have occurred. Infrastructure performs the technical work and returns an Application-facing result.

### Coordination Ends

Coordination ends only when the use case reaches an explicit application outcome:

- the approved Domain changes have been persisted or their failure has been resolved through rollback or approved compensation;
- any required post-transaction Domain-event reactions have been handed to the event-dispatch path; and
- the inbound caller has a meaningful application outcome.

External submission does not, by itself, establish Trade truth. Trade creation is coordinated only from a confirmed execution outcome and through approved Trades Domain behavior.

---

## 5. Transaction Contract

Execution use cases have explicit conceptual transaction boundaries. A transaction boundary coordinates state changes and observable consequences; it does not merge Aggregate Roots or replace their consistency boundaries.

### Transaction Start

The transaction or workflow scope begins when the Coordinator accepts an inbound execution request and starts coordinating Domain state required for that use case.

### Completion

The scope completes when all required Domain changes have been accepted by their owning aggregates and their resulting state has been coordinated for persistence. Domain events may be dispatched only after the relevant completion point succeeds.

### Rollback Responsibility

Infrastructure supplies the technical rollback capability through the transaction coordination contract. Application determines that an execution use case must be rolled back when a failure occurs before its approved completion point. Domain aggregates retain responsibility for their own invariant protection at all times.

### Compensation Responsibility

When an external broker or exchange action cannot participate in the same atomic scope as Domain persistence, the Coordinator owns coordination of the approved compensating workflow. It must not assume that a technical retry, acknowledgement, or partial external result changes Domain truth. The specific compensation policy requires an approved use-case design and remains outside this generic contract.

---

## 6. Failure Contract

The Coordinator coordinates failures at the application level and preserves ownership of their meaning.

| Failure category           | Domain responsibility                      | Application responsibility                                                                                           | Infrastructure responsibility                                      |
| -------------------------- | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Domain rejection           | Decide whether business behavior is valid. | Stop or redirect the use case and return the rejection as an application outcome.                                    | None beyond any completed technical work.                          |
| Infrastructure failure     | None.                                      | Coordinate failure propagation, rollback, retry, compensation, or escalation according to approved use-case policy.  | Detect and report technical failure through the port contract.     |
| Broker or exchange failure | None.                                      | Preserve the distinction between external failure and Domain rejection; coordinate the approved response.            | Translate external failure into an Application-facing outcome.     |
| Timeout                    | None.                                      | Coordinate retry, compensation, or incomplete-workflow handling without inferring a business result.                 | Detect timeout according to its technical mechanism and report it. |
| Orchestration failure      | None.                                      | Identify incomplete sequencing or unmet workflow prerequisites and resolve them without bypassing Domain boundaries. | Fulfill any technical capability involved.                         |

### Retry Coordination

The Coordinator coordinates whether an operation may be retried under approved use-case policy. Infrastructure performs technical retry mechanics when applicable. A retry must preserve the idempotency contract and must not create duplicate Order effects, Trades, Portfolio updates, or event reactions.

### Failure Propagation

The Coordinator returns a meaningful application outcome to the inbound boundary. It must retain the difference between:

- a Domain rejection;
- a technical inability to communicate or persist;
- an uncertain external outcome; and
- an incomplete orchestration process.

---

## 7. Idempotency Contract

The Execution Coordinator must support idempotent execution coordination.

### Business Expectation

A repeated request representing the same execution intent, or a repeated external result representing the same confirmed execution outcome, must not produce duplicate business effects. In particular, repetition must not:

- submit the same approved intent more than once when the outcome is already known;
- create duplicate Trade execution truth;
- apply the same Trade outcome more than once to Portfolio ownership interpretation; or
- dispatch duplicate post-transaction reactions for the same completed business fact.

### Coordination Expectation

The Coordinator uses the inbound request or external-result correlation required by the inbound contract to determine whether the workflow has already reached an outcome. It returns or continues from the established application outcome rather than repeating completed Domain effects.

Idempotency does not alter Domain identity, Aggregate Root rules, or business validation. It is an Application coordination guarantee. The exact storage, key representation, concurrency control, and technical deduplication mechanism are implementation decisions outside this specification.

---

## 8. Event Contract

Domain events record business facts established by Domain behavior. The Coordinator does not invent Domain events or treat technical transport outcomes as Domain facts.

### Publication Order

1. The owning Domain aggregate establishes the business fact and produces the approved Domain event.
2. The Coordinator coordinates persistence and transaction completion for the relevant Domain changes.
3. The Coordinator hands approved reactions to the Domain Event Dispatcher.
4. The Event Dispatcher invokes Application reactions or Event Publication Port capabilities.
5. Infrastructure performs technical delivery when required.

### Reaction Order

Event reactions occur only after the event's originating Domain work reaches its approved completion point. A reaction must not bypass the owning Aggregate Root, create a competing source of truth, or cause Portfolio changes without invoking Portfolio behavior.

### Orchestration Responsibility

The Coordinator owns the sequencing that makes an event eligible for reaction within its use case. The Domain Event Dispatcher owns routing of approved reactions. Infrastructure owns how a message is physically delivered. No messaging payload, technology, delivery guarantee, or event-bus design is prescribed here.

---

## 9. Dependency Review

The required dependency direction is:

```text
Execution Coordinator
  ↓
Application contracts and Domain abstractions
  ↓
Application ports
  ↓ implemented by
Infrastructure implementations
```

The Coordinator depends only on:

- approved Domain abstractions to invoke business behavior; and
- Application-owned contracts and ports to acquire state, interact externally, coordinate transactions, and dispatch reactions.

Infrastructure depends inward to implement the ports. The Composition Root selects and wires concrete implementations.

The Coordinator must not import or depend on broker adapters, exchange adapters, protocol libraries, database technologies, message transports, or vendor-specific types. This preserves mandatory dependency inversion.

---

## 10. Engineering Principles

1. **Orchestration only.** The Coordinator orders approved steps without owning business meaning.
2. **Constructor dependency injection.** Required Domain collaborators and Application contracts are supplied to the Coordinator at construction; it does not create or select Infrastructure dependencies.
3. **Framework independence.** Its design remains independent of runtime framework, transaction manager, persistence engine, and messaging system.
4. **Protocol independence.** Broker and exchange communication is expressed through Application-facing results, never protocol details.
5. **No business ownership.** Order, Trade, Portfolio, Strategy, and Market Data remain owned by their Domain contexts.
6. **No Infrastructure ownership.** The Coordinator invokes ports; it does not implement technical adapters.
7. **Aggregate respect.** All business mutation occurs through the appropriate Aggregate Root's approved behavior.
8. **Explicit outcomes.** Submission, confirmation, rejection, uncertainty, failure, retry, and completion remain distinguishable application outcomes.
9. **Idempotent coordination.** Repetition must not create duplicate business effects.
10. **Minimal design.** Introduce no abstractions beyond the contracts required to coordinate the approved Execution workflow.

---

## 11. Implementation Readiness

The Execution Coordinator is fully specified for implementation at the Application design level.

The required contracts are defined conceptually: inbound intent and correlation, outbound capabilities, workflow sequencing, transaction boundaries, failure ownership, idempotency expectation, event ordering, and dependency direction. Concrete request types, port interfaces, result types, persistence mechanisms, retry limits, compensation rules, and event payloads are implementation details to be designed within these approved constraints.

**Implementation may begin.**

---

## 12. Self Review

- Matches Architecture Handbook v2.0.
- Matches Application Layer Design Specification v1.0.
- Matches Execution Application Module Design Specification v1.0.
- Execution Coordinator owns orchestration only.
- Domain retains all business ownership and validation.
- Infrastructure retains all technical implementation ownership.
- Dependency inversion is preserved through Application-owned contracts and ports.
- No concrete interfaces, classes, frameworks, protocols, schemas, payloads, or adapters are prescribed.

---

EXECUTION COORDINATOR

Implementation Design Specification

Approved

Implementation

Ready to Begin

---
