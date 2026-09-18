# Core Domain Engineering Guide

## 1. Introduction

The Foundation defines reusable primitive business concepts.

The Core Domain composes those concepts into business models with explicit responsibilities, behavior, relationships, and lifecycle meaning.

Core Domain engineering focuses on expressing approved business architecture rather than discovering or redefining it during implementation.

Business semantics remain authoritative in Design Specifications, ADRs, and the Ubiquitous Language.

## 2. Engineering Philosophy

### Design-First Engineering

Business architecture and design are established before implementation begins.

### Architecture Before Implementation

Implementation follows approved boundaries and must not compensate for missing architectural decisions.

### ADR-Driven Evolution

Material architectural changes are documented and approved through Architecture Decision Records.

### Reference Implementations

Approved implementations establish proven patterns for future work within the same architectural context.

### Business Behavior Over Primitive Data

Entities and value objects express meaningful business behavior instead of exposing business rules as unrestricted primitive data manipulation.

### Composition Over Duplication

Existing Foundation and Core Domain concepts are composed and reused rather than redefined inside other concepts.

### Framework Independence

Core Domain code remains independent of frameworks and infrastructure so business meaning can be evaluated without technical coupling.

### Explicit Business Responsibilities

Each responsibility belongs to one authoritative business concept. Boundaries are explicit and protected from accidental overlap.

## 3. Core Domain Architecture

The approved Core Domain architecture is:

```text
Foundation Value Objects
          |
          v
Core Domain Value Objects
          |
          v
Core Domain Entities
          |
          v
Business Workflows
```

### Foundation Value Objects

Foundation Value Objects provide reusable primitive business concepts that can be shared across architectural domains.

### Core Domain Value Objects

Core Domain Value Objects express reusable business meaning specific to the Core Domain without possessing independent entity identity.

### Core Domain Entities

Core Domain Entities represent business concepts with mutable behavior, relationships, and lifecycle meaning.

### Business Workflows

Business Workflows coordinate approved entities and value objects to support higher-level domain activity.

## 4. Core Domain Value Objects

Core Domain Value Objects:

- express business meaning,
- are immutable,
- are value-based,
- have no independent identity,
- belong only to the Core Domain,
- and are reusable by multiple entities.

Approved reference implementations:

- ListingStatus
- Tradability

Core Domain Value Objects differ from Foundation Value Objects because their meaning is specific to the Core Domain rather than being a primitive concept intended for broad domain reuse.

They differ from Core Domain Entities because they do not possess independent identity or an autonomous lifecycle.

## 5. Core Domain Entities

Core Domain Entities:

- represent business concepts,
- compose Foundation Value Objects,
- compose Core Domain Value Objects,
- own explicit business behavior,
- remain mutable where business lifecycle requires change,
- and defer entity identity semantics until those semantics are approved.

Approved reference implementations:

- Instrument
- Exchange
- Listing

Entities remain framework-independent so their business responsibilities and behavior are not coupled to persistence, APIs, messaging, or infrastructure concerns.

## 6. Business Behavior

Entities expose explicit business operations that communicate business intent.

Examples include:

- `rename()`
- `update_status()`
- `update_tradability()`
- `update_description()`

Explicit business operations are preferred over unrestricted primitive property mutation because they provide a clear place for approved business rules and preserve responsibility boundaries.

Operations must remain limited to behavior supported by the approved design. They must not introduce workflow behavior or unapproved lifecycle transitions.

## 7. Boundary Protection

Approved entity boundaries are:

```text
Instrument
    |
    v
Intrinsic tradable identity

Exchange
    |
    v
Trading venue

Listing
    |
    v
Market-specific manifestation
```

Responsibilities belong to exactly one authoritative concept.

Business concepts must not be duplicated across entities.

Exchange owns ExchangeCode.

Listing references Exchange and does not duplicate ExchangeCode.

Instrument remains independent of market-specific participation.

Exchange remains independent of any individual Instrument manifestation.

Listing owns market participation context without redefining Instrument or Exchange meaning.

## 8. Value Object Composition

Core Domain Entities compose Foundation Value Objects and Core Domain Value Objects.

The approved composition examples are:

```text
Instrument
    |
    v
  Symbol

Exchange
    |
    v
ExchangeCode

Listing
    |
    v
 Currency
    |
    v
ListingStatus
    |
    v
Tradability
```

Composition preserves business meaning and prevents primitive obsession.

It also keeps each business concept in its authoritative architectural layer.

## 9. Implementation Rules

Implementation must:

- follow approved architecture,
- reuse approved business concepts,
- never redefine business concepts,
- never broaden responsibilities,
- stop when architecture is incomplete,
- and report architectural gaps instead of implementing assumptions.

Architecture always precedes implementation.

Implementation must not use technical convenience to resolve an unresolved business decision.

## 10. Testing Philosophy

Core Domain Entities use Reference Entity Contract Suites.

Core Domain Value Objects use Reference Value Object Contract Suites.

Contract suites verify:

- business correctness,
- entity and value object boundaries,
- approved composition,
- mutability or immutability as appropriate,
- and explicit business behavior.

They do not verify implementation details or require a particular internal mechanism when the business contract is satisfied.

Entity identity tests remain deferred when the Core Domain Entity Identity Model has not been approved.

## 11. Documentation Lifecycle

The approved engineering workflow is:

```text
Architecture Analysis
          |
          v
Architecture Review
          |
          v
Design Specification
          |
          v
Implementation
          |
          v
Implementation Review
          |
          v
Reference Contract Suite
          |
          v
Contract Review
          |
          v
CHANGELOG
          |
          v
Reference Registry
          |
          v
Merge
```

### Architecture Analysis

Identifies business boundaries, relationships, responsibilities, and unresolved architectural questions.

### Architecture Review

Approves or corrects the business architecture before detailed design proceeds.

### Design Specification

Defines the canonical business meaning, invariants, relationships, and responsibilities of one approved concept.

### Implementation

Expresses the approved design without adding business assumptions.

### Implementation Review

Verifies that the implementation faithfully follows the approved architecture and design.

### Reference Contract Suite

Defines the reusable business contract for the approved implementation.

### Contract Review

Confirms that the contract suite verifies business behavior and boundary protection.

### CHANGELOG

Records the completed engineering milestone and approved feature scope.

### Reference Registry

Records the approved implementation as a reference for future work in its bounded context.

### Merge

Integrates the reviewed and documented feature into the shared development line.

## 12. Reference Implementations

Approved engineering references include:

### Reference Foundation Value Object

- Symbol

### Reference Core Domain Value Objects

- ListingStatus
- Tradability

### Reference Core Domain Entities

- Instrument
- Exchange
- Listing

Future implementations should follow these proven engineering patterns unless a future ADR explicitly changes the architecture.

## 13. Future Development

The Core Domain Foundation is complete.

Future work now focuses on business workflows built on the approved Foundation and Core Domain concepts.

Examples include:

- Market Data
- Order
- Trade
- Position
- Portfolio
- Analytics
- Risk

These concepts build upon the approved Core Domain Foundation and should not modify or redefine it without the appropriate architectural process.
