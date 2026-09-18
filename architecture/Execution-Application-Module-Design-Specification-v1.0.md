# Execution Application Module Design Specification v1.0

## Status

**Approved application-design specification for Northstar Platform v1.1.0.**

This document defines the canonical design boundary for the Execution Application Module. It is governed by Architecture Handbook v2.0 and Application Layer Design Specification v1.0.

Execution is an Application capability with Infrastructure support. It is not a Domain bounded context, does not introduce a new Domain model, and does not own business meaning. This specification is implementation-independent and does not prescribe classes, frameworks, libraries, protocols, persistence engines, messaging technologies, or concrete adapters.

---

## 1. Module Purpose

The Execution Application Module coordinates the platform workflow that acts on approved Order intent through external broker or exchange interaction and processes confirmed execution outcomes.

It exists because the path from Order intent to Trade execution truth and Portfolio ownership interpretation crosses existing Domain ownership boundaries and external technical systems. No single Domain aggregate owns that process. Execution provides the Application-level coordination required to preserve each existing owner's business meaning.

Execution belongs in the Application Layer because it orders use-case steps, invokes Domain behavior, establishes transaction boundaries, calls outbound ports, coordinates failures, and dispatches permitted reactions. These are orchestration responsibilities.

Execution is not a Domain model because it does not own participant intent, execution truth, Portfolio interpretation, decision policy, or broker semantics. Orders own intent, Trades own confirmed historical truth, Portfolio owns current ownership interpretation, and Strategy owns decision policy.

---

## 2. Responsibilities

The Execution Application Module is responsible for:

- coordinating execution-oriented use cases;
- coordinating approved Order submission, cancellation, and replacement workflows;
- invoking outbound broker or exchange capabilities through ports;
- receiving and coordinating confirmed external execution outcomes;
- invoking the Trades Domain model to establish confirmed Trade execution truth;
- coordinating the Portfolio workflow that incorporates an approved Trade outcome;
- establishing execution-use-case transaction and consistency boundaries;
- coordinating Domain-event reactions after successful Domain work;
- coordinating retry and failure behavior for external interactions; and
- returning application-level outcomes through inbound boundaries.

The Execution Application Module does not own:

- Strategy decision policy;
- Order business meaning, lifecycle rules, or intent;
- Trade business meaning or historical execution truth;
- Portfolio ownership interpretation or Position invariants;
- Market Data observations;
- broker, exchange, FIX, REST, or WebSocket protocols;
- persistence mechanisms or database mappings; or
- messaging delivery technology.

Execution coordinates existing concepts. It does not create an alternate source of truth for them.

---

## 3. Primary Application Use Cases

The following are the primary Execution Application Module use cases. Their detailed design and implementation require separate approval.

### Submit Order

Coordinates approved Order intent for external submission. It invokes the relevant Orders behavior, establishes the appropriate application transaction boundary, and calls the required broker or exchange outbound port. It does not decide whether the Order's business state is valid or implement protocol communication.

### Cancel Order

Coordinates a request to cancel an existing Order through its approved Domain lifecycle and an external execution capability. It respects the Order aggregate's decision and communicates the resulting application intent through the appropriate port.

### Replace Order

Coordinates a request to replace an existing Order through approved Order behavior and the required external capability. It does not invent replacement semantics, lifecycle transitions, or broker-specific rules.

### Receive Execution Result

Coordinates an execution result received through an inbound boundary from an external adapter. It distinguishes application workflow handling from Domain meaning, establishes the relevant processing boundary, and determines which approved Domain workflow must be invoked.

### Confirm Trade

Coordinates confirmed external execution information with the Trades Domain model. Trade creation or recording remains a Domain action; the use case does not directly construct an alternate Trade state or treat an unconfirmed result as execution truth.

### Trigger Portfolio Update

Coordinates an approved Trade outcome with the Portfolio Domain model. Portfolio decides how current ownership interpretation and Position state evolve. The use case does not directly modify Portfolio or Position state.

---

## 4. Execution Coordinator

The Execution Coordinator is the Application component responsible for ordering the complete execution workflow across Domain abstractions and external ports.

It coordinates:

- approved Order intent;
- the relevant Order lifecycle invocation;
- outbound broker or exchange requests through ports;
- confirmed external execution outcomes;
- Trade creation through the Trades Domain model;
- Portfolio-update workflows through the Portfolio Domain model;
- transaction-boundary transitions; and
- permitted Domain-event reactions.

### Where Coordination Starts

Coordination starts when an inbound use case receives approved Order intent requiring an external execution action, or when an approved external result reaches the Execution Module through an inbound boundary.

### Where Coordination Ends

Coordination ends when the Execution use case has reached its defined application outcome: Domain changes have been processed through their owning aggregates, required state has been coordinated through ports, and approved post-transaction reactions have been handed to the Domain Event Dispatcher or outbound ports.

The Coordinator does not own business rules. It delegates all business decisions to the applicable Domain Aggregate Root and does not substitute workflow sequencing for Domain validation.

---

## 5. Command Handlers

The following command-handling responsibilities belong in the Execution Application Module:

### SubmitOrderCommandHandler

Receives a request to submit approved Order intent, validates application request shape and applicable authorization context, coordinates the execution transaction boundary, invokes approved Orders behavior, and requests external submission through an outbound port.

### CancelOrderCommandHandler

Receives a request to cancel an Order, coordinates the approved Order lifecycle invocation, and requests external cancellation through an outbound port after the Domain model permits it.

### ReplaceOrderCommandHandler

Receives a request to replace an Order, coordinates the approved Order behavior and external replacement request, and preserves the Order aggregate as the owner of all lifecycle decisions.

These handlers coordinate commands. They do not implement broker protocols, determine Domain validity, mutate aggregate internals, or choose concrete Infrastructure implementations.

---

## 6. Domain Interaction

The Execution Application Module invokes Domain behavior through approved public boundaries. The Application coordinates; the Domain decides.

### Strategy

Execution may receive approved decision intent originating from a Strategy-oriented use case. Strategy remains the owner of decision policy. Execution does not evaluate market observations, reinterpret Strategy policy, or communicate directly with Strategy internals.

### Order

Execution coordinates the submission, cancellation, and replacement workflows of Orders by invoking approved Order behavior. Order remains the owner of participant intent, OrderStatus meaning, lifecycle rules, identity, and aggregate consistency.

### Trade

Execution coordinates confirmed external outcomes with the Trades Domain model. Trade remains the owner of immutable execution truth. Execution does not directly create a competing Trade state, modify completed Trade history, or equate an external acknowledgement with Domain truth without the approved Trade workflow.

### Portfolio

Execution coordinates the incorporation of an approved Trade outcome into Portfolio. Portfolio remains the owner of ownership interpretation, Position collection consistency, and the one Position interpretation per Listing invariant. Execution does not directly modify Portfolio or Position state.

---

## 7. Ports

The Execution Application Module depends on Application-owned ports that express capabilities required by its use cases. These are responsibilities, not interface designs.

### Broker Port

Provides the capability to request broker-facing actions and receive broker-facing results without exposing broker-specific protocol or vendor behavior to Execution orchestration.

### Exchange Port

Provides the capability to request exchange-facing actions and receive exchange-facing results where an exchange integration is the relevant external boundary.

### Order Persistence Port

Provides the capability to retrieve and persist Order aggregate state needed by execution workflows. It does not define Order business rules or database behavior.

### Trade Recording Port

Provides the capability to coordinate persistence or recording of approved Trade aggregate outcomes after the Trades Domain model establishes execution truth.

### Portfolio Update Port

Provides the capability to retrieve and persist Portfolio aggregate state needed for an approved Portfolio-update workflow. It does not calculate ownership interpretation or directly mutate Positions.

### Event Publisher Port

Provides the capability to deliver approved post-transaction notifications beyond Application without exposing a messaging technology to the Execution Module.

### Transaction Boundary Port

Provides the capability to coordinate the persistence and completion scope required by an execution use case. It remains an application abstraction and does not prescribe transaction technology.

Additional ports may be approved only when they express a real Application capability required by an Execution use case. A port must not encode broker protocol details, persistence technology, or Domain invariants.

---

## 8. Domain Events

The Execution Application Module coordinates reactions to approved Domain events relevant to execution workflows. Conceptually, these include:

- Order Submitted;
- Execution Confirmed;
- Trade Recorded; and
- Portfolio Update Requested.

The listed events describe orchestration-relevant facts only. This specification does not define their payloads, names as implementation contracts, transport, or delivery semantics.

The Domain produces events when approved business facts occur. The Execution Module coordinates permitted application reactions only after the relevant Domain work and transaction boundary succeed. Infrastructure implements the technical delivery required by an Event Publisher Port.

Execution must not publish a technical message directly from Domain, turn event delivery into a Domain responsibility, or use an event to bypass an owning Aggregate Root.

---

## 9. Transaction Boundaries

Execution transaction boundaries define the conceptual scope of an execution-oriented use case. They make state changes, external calls, and post-transaction reactions explicit without merging independent Aggregate Roots into one business owner.

### Application Coordination Begins

Application coordination begins at an inbound Execution use case: a submit, cancel, replace, external result, confirmed-trade, or Portfolio-update request. The module identifies the intended workflow, acquires required Domain state through ports, and establishes the applicable coordination scope.

### Domain Work Occurs

Domain work occurs when Execution invokes approved behavior on the relevant Aggregate Roots. Order decides intent and lifecycle validity. Trade decides the establishment of execution truth. Portfolio decides ownership interpretation and Position consistency.

### Infrastructure Interaction Occurs

Infrastructure interaction occurs only when Execution invokes an outbound port. Infrastructure adapters perform the concrete broker, exchange, persistence, or event-delivery operation and return an Application-facing result.

An Execution transaction boundary defines when Domain changes are coordinated for persistence and when resulting Domain events may be dispatched. It identifies where an approved compensating workflow is required when an external interaction cannot share one atomic boundary. It does not prescribe a framework, library, storage model, or messaging technology.

---

## 10. Error Handling

Execution owns application-level coordination of failures. It does not own business validation.

### Retry Coordination

Execution may coordinate whether and when an approved external operation is retried according to application policy and available technical capability. It does not implement network retry mechanics or alter Domain meaning to make a retry convenient.

### Failure Propagation

Execution returns or propagates meaningful application outcomes to the calling inbound boundary. It preserves the distinction between a Domain rejection, an external communication failure, and an orchestration failure.

### External Communication Failures

Infrastructure detects and reports technical communication failures through port contracts. Execution coordinates the use-case response, retry, compensation, or escalation path as approved. It does not embed protocol-specific error handling.

### Orchestration Failures

Execution identifies failures in sequencing, required dependencies, incomplete workflow progress, or post-transaction coordination. It resolves them through approved application flow without bypassing Domain boundaries or treating incomplete external work as confirmed Trade truth.

### Business Validation

Business validation remains inside Domain. Execution invokes the relevant Aggregate Root and respects its outcome. It must not duplicate or replace validation for Order intent, Trade truth, Portfolio ownership, or Strategy policy.

---

## 11. Module Boundaries

### Execution Module Owns

- execution orchestration;
- use-case coordination;
- command flow;
- inbound and outbound port invocation;
- transaction-boundary coordination;
- error and retry coordination at the application level; and
- Domain-event reaction coordination.

### Execution Module Does Not Own

- Orders;
- Trades;
- Portfolio;
- Strategy;
- Market Data;
- broker or exchange protocols;
- FIX, REST, or WebSocket implementation;
- persistence implementation;
- messaging implementation;
- Domain invariants or lifecycle semantics; or
- Composition Root selection of concrete adapters.

---

## 12. Dependency Review

The required dependency direction is:

```text
Execution Application Module
  ↓
Domain abstractions and Application ports
  ↓
Infrastructure implementations
```

Execution depends on Domain abstractions to invoke business behavior and on Application-owned ports to request external capabilities. Infrastructure depends inward to implement those ports.

```text
Inbound adapter
  ↓
Execution Application Module
  ↓ invokes
Domain Aggregate Roots

Execution Application Module
  ↓ calls
Application ports
  ↓ implemented by
Infrastructure adapters

Composition Root
  ↓ wires
Infrastructure implementations to Execution dependencies
```

The Execution Module must not depend on Infrastructure implementations, vendor-specific types, protocol libraries, database abstractions, or concrete message transports.

---

## 13. Engineering Principles

1. **Orchestration only.** Execution orders approved use-case steps and does not own business meaning.
2. **Domain decides.** Order, Trade, Portfolio, and Strategy retain their approved rules, invariants, and lifecycle semantics.
3. **No protocol ownership.** Broker, exchange, FIX, REST, and WebSocket implementation remain Infrastructure concerns.
4. **No persistence ownership.** Execution requests persistence through ports and does not contain database or mapping logic.
5. **Dependency inversion.** Execution depends on Domain abstractions and Application ports; Infrastructure supplies implementations.
6. **Framework independence.** Module design must remain independent of frameworks, libraries, vendors, and transport choices.
7. **Use-case orientation.** Module responsibilities are organized around execution intents and outcomes, not technical systems or data tables.
8. **Aggregate respect.** Execution never bypasses Aggregate Roots or directly mutates Domain-owned state.
9. **Explicit outcomes.** Submission, confirmation, failure, retry, and Portfolio-update coordination remain visible in the use-case design.
10. **Minimal coordination.** The module introduces only the workflow required to coordinate existing owners and avoids generic abstractions without approved need.

---

## 14. Anti-Patterns

The Execution Application Module must never:

- implement broker or exchange logic;
- implement FIX, REST, WebSocket, or other protocol handling;
- implement Domain business rules, validation, identity semantics, or lifecycle rules;
- bypass Aggregate Roots to mutate Order, Trade, Portfolio, Position, or Strategy state;
- directly modify Portfolio or Position state;
- directly create Trade state outside approved Trades Domain behavior;
- treat an unconfirmed external response as immutable Trade execution truth;
- embed persistence mappings, database queries, or message-transport implementation;
- depend on concrete Infrastructure implementations or vendor-specific types;
- select a concrete adapter outside the Composition Root;
- create a new Execution Domain model or bounded context; or
- duplicate Market Data, Strategy, Orders, Trades, or Portfolio ownership.

---

## 15. Self Review

- Matches Architecture Handbook v2.0.
- Matches Application Layer Design Specification v1.0.
- Execution remains an Application capability with Infrastructure support.
- Domain remains unchanged and retains business ownership.
- Infrastructure remains unchanged and retains technical ownership.
- Dependency inversion is preserved through Application-owned ports and Infrastructure implementations.
- No implementation assumptions, frameworks, libraries, protocols, adapters, or payload designs are introduced.
- The module is suitable as the reference design for future Application modules.

---

EXECUTION APPLICATION MODULE

Design Specification

Approved

Ready for Module Implementation

---
