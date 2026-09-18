# Execution Application Contracts Design Specification v1.0

## Status

**Approved application-design specification for Northstar Platform v1.1.0.**

This document defines the concrete Application Contract set required by the Execution Application Module. It is governed by Architecture Handbook v2.0, Application Layer Design Specification v1.0, Application Contracts Design Specification v1.0, Application Ports Design Specification v1.0, and the approved Execution specifications.

Execution Application Contracts are Application-owned coordination models. They do not create Domain objects, define Infrastructure DTOs, prescribe fields, prescribe serialization, or select frameworks, protocols, storage, or messaging technologies.

---

## 1. Contract Set

The canonical Execution Application Contract set is:

- ExecutionRequest;
- ExecutionResponse;
- ExecutionResult;
- ExecutionOutcome; and
- ExecutionFailure.

This is the complete contract set required for the approved Execution Application Module. Each contract has a distinct responsibility and must not replace another member of the set.

```text
ExecutionRequest
  ↓ initiates or continues
Execution workflow
  ↓ produces step-level
ExecutionResult or ExecutionFailure
  ↓ resolves to
ExecutionOutcome
  ↓ communicated through
ExecutionResponse
```

The diagram describes a coordination relationship, not a required class hierarchy, serialized shape, or invocation mechanism.

---

## 2. ExecutionRequest

ExecutionRequest is the Application Contract that expresses an intent to coordinate an execution use case.

### Creation

ExecutionRequest is created at an inbound Application boundary when an API adapter, scheduled operation, event consumer, command handler, Application Service, or other approved Application workflow requests execution coordination. It may also be created when an Infrastructure adapter has translated an external execution notification into an Application-level intent for processing.

### Producers and Consumers

Inbound adapters and approved Application components create ExecutionRequest. The Execution Coordinator and Execution-oriented Application Services or Command Handlers consume it. The request does not flow into Domain as a substitute for an aggregate and does not flow to Infrastructure as a technical wire format.

### Business Intent

ExecutionRequest represents workflow intent only. It requests coordination of one approved execution-oriented use case, including Order submission, cancellation, replacement, execution-result processing, Trade confirmation coordination, or Portfolio-update coordination.

It identifies the context needed to begin or continue that workflow through approved references and correlation. It does not assert that a business transition is valid; owning Domain aggregates decide that.

### Boundaries

ExecutionRequest may reference the applicable Order, Trade, Portfolio, Strategy, Participant, Listing, request correlation, external-result correlation, and caller or authorization context as necessary for the requested coordination. It must not contain mutable Domain state, broker messages, database representations, transport data, adapter configuration, or a direct aggregate mutation instruction.

---

## 3. ExecutionResponse

ExecutionResponse is the Application Contract that communicates the final Execution use-case conclusion to an inbound caller or adapter.

### Production

ExecutionResponse is produced when the Execution Coordinator reaches the defined application conclusion for an inbound ExecutionRequest. It communicates the resulting ExecutionOutcome and, where relevant, the associated ExecutionFailure in Application terms.

### Producers and Consumers

The Execution Coordinator or an approved Application Service produces ExecutionResponse. API adapters, scheduled callers, event consumers, command handlers, and other inbound Application boundaries consume it and translate it into their own delivery-specific representation when required.

### Boundary

ExecutionResponse is an Application Contract, not a Domain object. It does not expose Order, Trade, Portfolio, Strategy, Position, or Market Data state. It does not contain HTTP responses, broker acknowledgements, protocol statuses, database rows, or message envelopes.

---

## 4. ExecutionResult

ExecutionResult is the Application Contract that communicates a normalized result from one execution workflow step, especially an outbound Port interaction.

### What It Communicates

ExecutionResult communicates the application-relevant result needed by the Coordinator to continue, stop, retry, compensate, or conclude a workflow. It can express the result of an approved broker, exchange, persistence, transaction, or other Execution Port interaction in protocol-independent Application terms.

### Difference from Trade

ExecutionResult is not a Trade. It communicates an interaction or processing result; it is not immutable historical execution truth. Even a confirmed external execution result becomes a Trade only when the Coordinator invokes approved Trades Domain behavior and that Domain behavior establishes Trade meaning.

### Difference from ExecutionResponse

ExecutionResult is internal to the coordination of a workflow step. ExecutionResponse is the external-facing Application conclusion returned to the inbound caller after the complete workflow reaches an ExecutionOutcome. A Result may contribute to an Outcome; it is not the final use-case response by itself.

### Boundary

Infrastructure adapters produce technical results and translate them into ExecutionResult through an approved Port. The Execution Coordinator consumes ExecutionResult. Result must not expose broker DTOs, exchange replies, FIX tags, REST responses, WebSocket frames, database rows, or vendor types.

---

## 5. ExecutionOutcome

ExecutionOutcome is the Application Contract that represents the final state of an Execution workflow.

### Workflow Completion

ExecutionOutcome represents whether the requested coordination completed, was rejected by Domain behavior, failed technically, timed out or remains externally uncertain, or remains incomplete and requires approved follow-up or compensation.

It represents workflow completion, not business truth. It tells the Application boundary what happened to the requested coordination, not what an Order, Trade, Portfolio, or Strategy means.

### Difference from ExecutionResult

ExecutionResult communicates an individual workflow-step result. ExecutionOutcome communicates the final conclusion after the Coordinator has considered the required Domain behavior, Port results, transaction completion, persistence coordination, event-reaction eligibility, and applicable failure processing.

### Difference from Trade

ExecutionOutcome is not a Trade and does not contain or replace Trade state. An Outcome may communicate that Trade confirmation coordination completed, was rejected, or could not be completed. Only the Trades Domain model owns immutable Trade execution truth.

---

## 6. ExecutionFailure

ExecutionFailure is the Application Contract that communicates a non-successful or uncertain Execution coordination condition in Application terms.

### Included Failures

ExecutionFailure represents failures that require Application workflow handling, including:

- external timeout;
- Infrastructure failure reported through a Port;
- broker or exchange failure translated through a Port;
- retry exhaustion;
- uncertain external result;
- incomplete workflow requiring approved follow-up or compensation; and
- orchestration failure, such as failed sequencing, unresolved correlation, or inability to reach the defined completion point.

### Domain Validation Boundary

Domain validation failures remain Domain concerns. An Order, Trade, Portfolio, or Strategy aggregate decides whether requested business behavior is valid. ExecutionFailure may communicate the resulting application-level failure classification to an inbound caller, but it must not restate, reinterpret, or implement the Domain rule.

### Infrastructure Boundary

Infrastructure detects technical failures and translates them through an approved Port. ExecutionFailure does not preserve raw SDK exceptions, transport codes, broker error payloads, database exceptions, or protocol-specific details.

---

## 7. Contract Relationships

Each Execution Application Contract serves a single coordination responsibility.

| Contract          | Responsibility                                                             | Not responsible for                                                |
| ----------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| ExecutionRequest  | Expresses intent to start or continue an Execution use case.               | Validating Domain behavior or carrying aggregate state.            |
| ExecutionResult   | Communicates a normalized result from a workflow step or Port interaction. | Establishing final workflow completion or Trade truth.             |
| ExecutionFailure  | Communicates a non-successful or uncertain application condition.          | Defining Domain validation or retaining raw Infrastructure errors. |
| ExecutionOutcome  | States the final status of the coordinated Execution use case.             | Replacing a Domain aggregate or individual Port result.            |
| ExecutionResponse | Communicates the final Outcome to an inbound caller.                       | Exposing Domain entities or delivery-specific models.              |

The canonical relationship is:

```text
ExecutionRequest
  ↓
Execution Coordinator invokes Domain and Ports
  ↓
ExecutionResult or ExecutionFailure
  ↓
ExecutionOutcome
  ↓
ExecutionResponse
```

ExecutionFailure may directly inform ExecutionOutcome. ExecutionResult may also lead to a Domain rejection, an additional workflow step, a retry, compensation coordination, or successful completion. The contracts remain distinct regardless of path.

---

## 8. Domain Relationship

Execution Application Contracts reference Domain concepts to identify the context that Application must coordinate. They do not replace any Domain model.

### Orders

Contracts may reference an Order and request execution coordination around approved Order behavior. They do not carry Order state, determine OrderStatus, validate intent, or bypass the Order aggregate.

### Trades

Contracts may carry the normalized execution context needed to coordinate confirmed Trade creation. They do not create Trade truth, represent a Trade, or modify immutable Trade history.

### Portfolio

Contracts may identify the Portfolio context required to coordinate incorporation of an approved Trade outcome. They do not contain ownership interpretation, Position state, or Portfolio invariants.

### Strategy

Contracts may reference approved Strategy context or a Strategy-originated decision reference. They do not model decision policy, evaluate observations, or make strategy decisions.

All business rules, identities, lifecycle semantics, and invariants remain inside their owning Domain contexts.

---

## 9. Port Relationship

Application Ports consume and produce Execution Application Contracts.

- An inbound Execution Port accepts ExecutionRequest and returns ExecutionResponse or ExecutionOutcome.
- An outbound Execution Port accepts the applicable Execution Application Contract expressing a required external capability and returns ExecutionResult or ExecutionFailure.

Broker and Exchange Ports use Execution Contracts to translate Application intent to technical communication and technical responses back into Application workflow information. Order Persistence, Trade Recording, Portfolio Update, Event Publisher, and Transaction Ports use them only to the extent required by their focused capabilities.

Ports must never substitute Domain entities or Infrastructure DTOs for Execution Application Contracts. A Port returns an Application-level Result or Failure; it does not return a broker message, database row, or mutable aggregate state as its boundary representation.

---

## 10. Engineering Principles

1. **Immutable contracts.** Each Execution Contract represents a fixed request, result, failure, outcome, or response and must not be mutated after creation.
2. **Framework independence.** Contracts do not depend on web frameworks, ORMs, brokers, message buses, transaction managers, or runtime frameworks.
3. **Serialization independence.** Contract meaning is independent of JSON, database, queue, or other serialized format.
4. **Protocol independence.** Contracts do not contain FIX, REST, WebSocket, or vendor-specific semantics.
5. **No business ownership.** Contracts carry workflow information and references only; Domain retains business meaning.
6. **No Infrastructure ownership.** Contracts contain no technical DTOs, schemas, adapter configuration, connection details, or protocol errors.
7. **Explicit lifecycle of coordination.** Request, Result, Failure, Outcome, and Response remain distinct stages of Application flow.
8. **Idempotency support.** ExecutionRequest and ExecutionResult provide the correlation context required for idempotent coordination without defining technical storage or deduplication mechanisms.
9. **Minimal contract surface.** No Execution Contract is introduced outside the approved Request, Response, Result, Outcome, and Failure family.

---

## 11. Anti-Patterns

Execution Application Contracts must never:

- expose Order, Trade, Portfolio, Position, Strategy, Listing, or Participant entities as delivery models;
- embed mutable aggregate state or direct aggregate mutation instructions;
- treat ExecutionResult, ExecutionOutcome, or ExecutionResponse as a substitute for Trade execution truth;
- embed broker or exchange SDK models, FIX messages, REST payloads, WebSocket frames, queue messages, or vendor types;
- embed database rows, ORM models, persistence schemas, or transaction-engine details;
- embed HTTP request or response models, transport status codes, or serialization behavior;
- contain Domain business logic, business validation, lifecycle rules, identity semantics, or aggregate invariants;
- hide the difference between Domain rejection, Infrastructure failure, timeout, uncertainty, and orchestration failure;
- create another Application Contract family outside the approved set; or
- use a contract to bypass a Domain Aggregate Root or dependency inversion boundary.

---

## 12. Implementation Readiness

Execution Application Contracts are fully specified for implementation at the Application design level.

The specification defines the complete named contract set, the responsibility and lifecycle of every contract, their relationship to Domain concepts and Application Ports, their failure boundaries, their idempotency role, and their prohibited content.

The following remain intentional implementation details that must conform to this specification:

- exact field definitions and constructor shapes;
- concrete contract module layout;
- application request-shape validation;
- type representations for Domain references and correlation;
- serialization mechanisms and API mappings; and
- technical storage, concurrency, and deduplication mechanisms.

**Execution Application Contract implementation may begin.**

---

## 13. Self Review

- Matches Architecture Handbook v2.0.
- Matches Application Layer Design Specification v1.0.
- Matches Application Contracts Design Specification v1.0.
- Supports the approved Execution Application Module and Execution Coordinator designs.
- Domain remains unchanged and retains all business meaning.
- Infrastructure remains unchanged and retains all technical implementation ownership.
- Execution Contracts own Application workflow information only.
- No fields, classes, interfaces, frameworks, protocols, serialization formats, or adapter implementations are prescribed.

---

EXECUTION APPLICATION CONTRACTS

Design Specification

Approved

Ready for Implementation

---
