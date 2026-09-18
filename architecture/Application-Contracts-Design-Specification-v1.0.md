# Application Contracts Design Specification v1.0

## Status

**Approved application-design specification for Northstar Platform v1.1.0.**

This document defines the canonical Application Contract model for Northstar. It is governed by Architecture Handbook v2.0, Application Layer Design Specification v1.0, Execution Application Module Design Specification v1.0, and Execution Coordinator Implementation Design Specification v1.0.

Application Contracts are implementation-independent coordination models. They do not define Domain meaning, infrastructure serialization, concrete classes, interfaces, payloads, frameworks, protocols, or storage representations.

---

## 1. Application Contract Purpose

An Application Contract represents information required to invoke, coordinate, observe, or conclude an Application use case. It gives API adapters, Application modules, and Infrastructure adapters a stable language for workflow intent and workflow outcome.

Application Contracts exist because the Application Layer needs to coordinate work without exposing Domain internals to delivery mechanisms or allowing wire formats to define use-case behavior. They separate an application request from an Aggregate Root and separate an application outcome from a broker message, database row, or transport response.

Application Contracts coordinate workflows. They do not own business meaning.

### Separation from Domain Models

Domain models own business concepts, invariants, lifecycle semantics, identities, and Domain events. Application Contracts reference Domain concepts only to identify the context an application workflow must coordinate. They do not replace an Order, Trade, Portfolio, Strategy, Position, Listing, or ParticipantReference.

An Application Contract must never assert that an intended Domain change is valid. It carries a request for coordination; the owning Domain aggregate decides whether the requested business behavior is permitted.

### Separation from Infrastructure DTOs

Infrastructure DTOs represent technology-specific data: broker responses, FIX messages, HTTP request bodies, WebSocket frames, database rows, queue messages, and vendor types. Application Contracts are independent of those representations.

Infrastructure translates incoming technical representations into Application Contracts and translates Application-facing port results into its own technical requests. Neither translation changes Domain meaning or turns technical data into an Application ownership model.

---

## 2. Application Contract Family

Northstar uses the following Application Contract families.

### Requests

A Request expresses an inbound application intent. It identifies the use case to coordinate and contains only the context, references, correlation, and authorization information required to begin that workflow.

A Request does not carry mutable aggregate state, technical transport data, or a caller-provided decision about Domain validity.

### Responses

A Response communicates an Application outcome to an inbound caller or adapter. It is the application-facing conclusion of a use case and is suitable for translation by an API or other delivery mechanism.

A Response does not expose Domain aggregate internals or infrastructure-specific data.

### Results

A Result communicates the normalized outcome of a step in an Application workflow, particularly an outbound-port interaction. It allows the Application Layer to continue, stop, retry, compensate, or propagate the workflow without depending on a technical adapter's representation.

A Result is not a Domain aggregate, a Domain event, or a technical transport response.

### Outcomes

An Outcome communicates the final, meaningful state of an Application use case. It classifies whether the requested coordination completed, was rejected by Domain behavior, failed technically, remains uncertain, or requires approved follow-up coordination.

An Outcome is distinct from a Result. Results describe the outcome of an individual interaction or workflow step; Outcomes describe the final state of the Application use case.

### Failures

A Failure communicates a non-successful application condition while retaining its ownership and category. It distinguishes Domain rejection, external technical failure, timeout, and orchestration failure without embedding an exception type, protocol error code, or infrastructure detail into the Application model.

---

## 3. Execution Request

An Execution Request is an Application Request that expresses an intent to coordinate an execution-oriented use case: Order submission, cancellation, replacement, execution-result processing, Trade confirmation, or Portfolio-update coordination.

### Business Information It Represents

An Execution Request represents application workflow intent, not a new business object. It establishes enough context to identify:

- the execution use case being requested;
- the relevant existing Order or other approved Domain context;
- any required participant, Listing, or workflow references through approved Domain identities or associations;
- correlation with an external execution result when that result is being processed;
- stable request identity or correlation required for idempotent coordination; and
- authorization or caller context required by the inbound Application boundary, where applicable.

### What It References

An Execution Request may reference existing Domain concepts by their approved identities or associations. It references, but never owns, the relevant Order, Participant, Listing, Trade, Portfolio, or Strategy context.

### What It Must Not Contain

An Execution Request must not contain:

- mutable state from an Order, Trade, Portfolio, Position, or Strategy;
- a caller-supplied assertion that a business transition is permitted;
- a Trade substitute or a Portfolio ownership interpretation;
- broker, exchange, FIX, REST, WebSocket, database, or messaging DTOs;
- protocol identifiers, wire-level status, connection details, or adapter-selection instructions;
- persistence schemas or database rows; or
- a way to bypass Aggregate Root behavior.

---

## 4. Execution Result

An Execution Result communicates a normalized application-level result from an execution-related interaction. It allows the Execution Coordinator to determine the next approved coordination step without depending on broker, exchange, protocol, or persistence implementation details.

### Business Outcome It Communicates

An Execution Result communicates the application-relevant state of an execution interaction, such as whether an external request was accepted, rejected, confirmed, incomplete, uncertain, or technically unsuccessful. It is a workflow input, not a record of business truth.

### Producer and Consumer

An Infrastructure adapter produces the technical response and translates it to the Application-facing Execution Result required by the relevant outbound port. The Execution Coordinator consumes the result to coordinate subsequent Application and Domain actions. An inbound adapter may also translate an external execution notification into an Execution Request or comparable Application Contract for result processing.

### Result Is Not Trade

An Execution Result is not a Trade. A Result reports an application-relevant interaction outcome. A Trade is an immutable Domain Aggregate Root representing confirmed execution truth. Only the Trades Domain model can establish Trade meaning through approved Domain behavior.

---

## 5. Execution Outcome

Execution Outcome is required.

An Execution Outcome represents the final state of an Execution use case as observed by the Application boundary. It provides a stable conclusion for the inbound caller and for permitted Application follow-up coordination.

### Difference from Execution Result

An Execution Result describes one normalized interaction or processing step, such as the outcome returned through a Broker Port. An Execution Outcome describes the overall Execution workflow after the Coordinator has applied the required Domain, transaction, persistence, event, and failure coordination.

### Difference from Trade

An Execution Outcome is an Application Contract. It may communicate that Trade confirmation was coordinated, rejected, incomplete, or uncertain. It is not a Trade, does not contain Trade state, and does not own historical execution truth.

### Required Distinctions

The Outcome model must preserve at least these conceptual conclusions:

- completed coordination;
- Domain rejection;
- technical failure;
- timeout or uncertain external state; and
- incomplete workflow requiring approved follow-up or compensation.

The precise representation of these distinctions is an implementation detail, but they must remain distinguishable at the Application boundary.

---

## 6. Failure Contract

Application Failure Contracts make failures visible without moving ownership from Domain or Infrastructure.

### Domain Failures

Domain owns business validation and rejection. A Domain failure means an owning aggregate did not permit the requested behavior according to its invariant, lifecycle, identity, or other approved business rule.

Application receives that rejection, stops or redirects the relevant use case, and reports it as an application outcome. Application does not reinterpret it or recreate the validation rule.

### Infrastructure Failures

Infrastructure owns detection and translation of technical failures arising from connectivity, broker or exchange interaction, persistence, messaging, authentication integrations, and other technical mechanisms.

Application receives a normalized failure through a port, coordinates retry, rollback, compensation, escalation, or propagation according to approved use-case policy, and preserves that the failure is technical rather than a Domain rejection.

### Application Failures

Application owns failures of orchestration: incomplete sequencing, unavailable required Application contract, unresolved workflow correlation, failure to reach the defined use-case completion point, or failure to coordinate an approved post-transaction reaction.

An Application failure must not be represented as an invented Domain state or as a raw Infrastructure error.

### Failure Ownership Summary

| Failure                                                      | Owner                                                           | Application role                                                      |
| ------------------------------------------------------------ | --------------------------------------------------------------- | --------------------------------------------------------------------- |
| Business validation or rejection                             | Domain                                                          | Coordinate and communicate the Domain outcome.                        |
| Broker, exchange, timeout, persistence, or messaging failure | Infrastructure detects; Application classifies for workflow use | Coordinate retry, rollback, compensation, escalation, or propagation. |
| Workflow sequencing or completion failure                    | Application                                                     | Coordinate resolution and communicate the application outcome.        |

---

## 7. Contract Ownership

Application Contracts are owned by the Application Layer.

```text
API and other inbound adapters
  ↓ consume and produce
Application Contracts
  ↓ invoke
Application use cases
  ↓ call through
Application ports
  ↓ implemented by
Infrastructure adapters
```

API uses Application Contracts to translate delivery-specific requests into application intent and application outcomes into delivery-specific responses. Application owns the contract meaning because it owns use-case coordination. Infrastructure consumes and produces Application Contracts at port boundaries while translating to and from technical formats.

Application Contracts must never become Domain entities because they do not own business identity, lifecycle, invariants, or state. They may carry references to Domain concepts, but reference does not transfer ownership.

---

## 8. Port Relationship

Application Ports consume and produce Application Contracts.

- **Inbound ports** accept Requests and return Responses or Outcomes. They define the application capabilities exposed to API adapters, schedulers, event consumers, and other delivery mechanisms.
- **Outbound ports** accept Application Contracts that express a required external capability and return Results or Failures in Application terms. They define what a use case requires from Infrastructure without naming a protocol, vendor, database, or message transport.

Ports do not expose Domain aggregate internals or Infrastructure DTOs. They provide the explicit boundary through which Application Contracts travel while preserving dependency inversion.

---

## 9. Domain Relationship

Application Contracts reference Domain concepts only to coordinate existing business owners.

### Strategy

Contracts may identify Strategy context or an approved Strategy-originated decision reference. They do not describe, calculate, or replace Strategy policy.

### Orders

Contracts may identify an Order and request coordination of an approved Order workflow. They do not carry Order state, validate Order intent, define OrderStatus meaning, or replace the Order aggregate.

### Trades

Contracts may correlate confirmed external execution information with the workflow that will invoke Trades Domain behavior. They do not create Trade truth, carry a Trade substitute, or alter immutable Trade history.

### Portfolio

Contracts may identify the Portfolio context required to coordinate an approved Trade outcome. They do not calculate or contain Portfolio ownership interpretation, Position state, or Portfolio invariants.

Application Contracts are therefore boundary models for coordination, not a parallel Domain model.

---

## 10. Infrastructure Relationship

Infrastructure adapters translate between technology-specific formats and Application Contracts.

At an inbound boundary, an adapter translates a REST request, WebSocket notification, scheduled trigger, message-consumer payload, or other technical input into an Application Request. At an outbound boundary, an adapter translates an Application Contract into the required broker, exchange, persistence, messaging, or authentication interaction.

The adapter then translates the technical result or failure into the Application Result or Failure required by the port contract.

This translation prevents:

- wire formats from becoming use-case models;
- broker DTOs from becoming Domain concepts;
- database rows from becoming aggregate state;
- protocol errors from leaking into Application semantics; and
- Infrastructure types from reversing dependency direction.

---

## 11. Engineering Principles

1. **Immutable contracts.** A Contract represents a specific request, result, outcome, response, or failure and must not be mutated after creation.
2. **Framework independence.** Contracts do not depend on web frameworks, ORMs, brokers, message buses, or runtime frameworks.
3. **Protocol independence.** Contracts do not contain FIX, REST, WebSocket, or vendor-specific semantics.
4. **Serialization independence.** Contract meaning exists independently of JSON, database, queue, or other serialized representation.
5. **Workflow information only.** Contracts carry only coordination intent, correlation, references, and application outcome information.
6. **Domain meaning remains in Domain.** Contracts do not carry aggregate state, business rules, lifecycle decisions, or invariant enforcement.
7. **Infrastructure stays technical.** Contracts do not contain broker DTOs, database rows, transport models, or connection detail.
8. **Explicit failure ownership.** Domain rejection, technical failure, and orchestration failure remain distinguishable.
9. **Idempotent coordination.** Requests and results provide the correlation required to prevent duplicate business effects.
10. **Minimal contract surface.** Each Contract exists only where it expresses a necessary Application boundary or workflow interaction.

---

## 12. Anti-Patterns

Northstar Application Contracts must never:

- expose Domain entities or Aggregate Roots as external delivery models;
- embed mutable Domain aggregate state;
- embed broker, exchange, FIX, REST, WebSocket, database, queue, or vendor DTOs;
- embed database rows, ORM records, or persistence schemas;
- embed HTTP request/response models or transport status semantics;
- contain business logic, business validation, lifecycle transitions, or aggregate invariants;
- use Application Contracts as replacement Domain entities;
- use technical success as evidence of Trade execution truth without the Trades Domain model;
- conflate a Result with an Outcome or conflate either with a Trade;
- expose concrete Infrastructure implementation types through ports;
- omit correlation needed for idempotent execution coordination; or
- hide the distinction between Domain rejection, technical failure, uncertain state, and orchestration failure.

---

## 13. Implementation Readiness

Application Contracts are fully specified for implementation at the Application design level.

The approved model defines ownership, contract families, Execution Request purpose, Execution Result purpose, Execution Outcome purpose, failure ownership, port interaction, Domain references, Infrastructure translation, immutability, idempotency correlation, and prohibited content.

The following remain intentional implementation details that must conform to this specification:

- concrete contract names and type definitions;
- exact field sets and validation limited to application request shape;
- serialization mechanisms;
- API mapping and transport status mapping;
- Infrastructure translation mechanisms; and
- technical storage and concurrency mechanisms used to support correlation.

**Application Contract implementation may begin.**

---

## 14. Self Review

- Matches Architecture Handbook v2.0.
- Matches Application Layer Design Specification v1.0.
- Supports the approved Execution Application Module and Execution Coordinator designs.
- Domain remains unchanged and retains all business meaning.
- Infrastructure remains unchanged and retains all technical implementation ownership.
- Application Contracts own workflow coordination information only.
- No classes, interfaces, payloads, frameworks, protocols, serialization formats, or adapter implementations are prescribed.

---

APPLICATION CONTRACTS

Design Specification

Approved

Ready for Implementation

---
