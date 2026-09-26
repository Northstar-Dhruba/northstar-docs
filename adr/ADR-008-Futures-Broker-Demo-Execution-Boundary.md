# ADR-008: Futures Broker Demo Execution Boundary

## Status

Accepted

## Date

2026-09-27

## Decision Makers

Northstar Architecture Team

## Context

Northstar already runs internal Futures paper trading in production:

- a frozen Futures forward research record states what the strategy advised about one contract at one decision instant;
- a target-position policy turns that advice into at most one simulated execution intent;
- a simulated fill is taken at the OPEN of the next persisted daily bar;
- the paper portfolio, valuation and profit and loss are folded from those fills.

Paper execution is deterministic. Every paper run re-derives every frozen decision through its cutoff, recomputes the simulated order and fill, and verifies that the persisted history still agrees. That works only because a simulated fill can be reproduced from stored market data.

Epic 10 adds execution through a broker DEMO account alongside paper trading, so the two can run in parallel for a comparison period before any real-money execution is considered. Broker execution differs in kind:

- it is external: the broker, not Northstar, decides whether an order is accepted, when it executes, at what price, and what the account holds;
- it is asynchronous: an order is submitted, then acknowledged, filled, partly filled, cancelled or rejected over time;
- it cannot be replayed: a broker execution happens once, and a broker cannot be asked to reproduce it.

The existing paper execution types are simulation types. FuturesExecutionIntent names a PaperPortfolioIdentity and describes a simulated trade; FuturesPaperFill is a simulated fill; FuturesPosition carries an average entry in the product's own quotation convention. None of them can describe a broker fact without either misrepresenting it or changing the paper contract.

Demo and paper execution must coexist without either being able to corrupt the other.

## Decision

Introduce a separate `broker_execution` bounded context in northstar-core for broker execution facts, and keep every paper-trading type unchanged.

### Core boundary

`northstar_core.broker_execution` holds immutable value objects only:

- `BrokerEnvironment`: the closed environment vocabulary. It has exactly one member, `DEMO`.
- `BrokerAccountReference`: an opaque Northstar reference to one configured broker account: broker code, account code and environment. It carries no credential.
- `ClientOrderIdentity`: the Northstar-owned identity of a broker order.
- `BrokerOrderId`, `BrokerExecutionId`: broker-owned external identifiers.
- `FuturesBrokerExecutionIntent`: account, contract, side, contract count, strategy and decision instant.
- `FuturesBrokerOrder`: a client order identity and its intent. It holds no broker fact.
- `BrokerOrderStatus`: the broker-normalized vocabulary `WORKING`, `FILLED`, `CANCELLED`, `REJECTED`.
- `FuturesBrokerOrderObservation`: what the broker reported about one order at one instant.
- `FuturesBrokerExecution`: one broker-reported execution against one order.
- `FuturesBrokerPosition`: broker-reported signed net contracts in one contract; flat is absence.

The package reuses the shared Futures execution vocabulary: FuturesContract, OrderSide, FuturesContractCount, StrategyIdentity, PointInTime and QuoteValue. It does not reuse or generalize FuturesExecutionIntent, PaperPortfolioIdentity, FuturesPaperOrder, FuturesPaperFill or FuturesPosition. The dependency is one-way: broker execution may import those shared values, and paper trading never imports broker execution.

### Consumption of research

Demo execution consumes persisted frozen Futures research records only. It never runs research, never invokes the analysis generator and never freezes a decision.

### Broker-specific concerns

Broker instrument symbols, identifier formats, price conventions, order types, endpoints, credentials and SDKs belong to Infrastructure. Core and Application name contracts by FuturesContract and quotations by QuoteValue only.

### Broker facts

Broker statuses, executions and positions enter Northstar only through a broker gateway port. Northstar never constructs one to stand in for something the broker did not report, and never represents a simulated fill as a broker execution.

### Later stories

The submission journal, idempotency, reconciliation, risk guards, broker adapter and operational process are Application, Infrastructure and API work in later Epic 10 stories. This ADR fixes their architectural commitments now so those stories implement them rather than re-decide them.

### Process isolation

Broker demo execution will ultimately run as its own command and process, separate from `northstar operations daily`, so a broker failure cannot stop market-data acquisition, research freezing or paper trading.

## Architecture Invariants

Items marked **(commitment)** are architectural commitments whose persistence, Application or Infrastructure implementation belongs to later Epic 10 stories. The remaining items are enforced from Story 10.1 or are existing properties the story must preserve.

1. **Frozen research only.** Demo execution consumes only persisted frozen Futures research records.
2. **One shared target policy.** One target-position policy will drive both execution engines. For the same frozen decision, paper and demo may make different trades from different current positions, but never pursue different targets.
3. **Distinct execution types.** Demo and paper execution types are distinct; no paper execution type is accepted where a broker execution type is required, and no broker execution type is accepted where a paper type is required.
4. **Paper engine unchanged.** The existing paper engine (its values, ports, stores, tables, identities, runtime and daily operation) remains unchanged by broker demo execution.
5. **Gateway-only broker facts.** Broker facts enter Northstar only through the broker gateway.
6. **(commitment) One order per decision.** One frozen decision may produce at most one broker order per account.
7. **(commitment) Claimed attempts.** A broker submission call requires a committed, uniquely claimed attempt record. A process that cannot claim the attempt does not call the broker.
8. **(commitment) Resume, never recompute.** A persisted broker order is resumed on retry, never recomputed or rewritten.
9. **(commitment) Look up in-doubt submissions.** A submission whose outcome is unknown is looked up at the broker by its client order identity, never blindly resubmitted.
10. **(commitment) Fail closed.** An unresolved submission, a working order or a position mismatch blocks new demo execution on the account. Northstar never places corrective orders.
11. **Broker specifics stay in Infrastructure.** Broker symbols, identifier formats and price conventions stay in Infrastructure.
12. **DEMO only.** DEMO is the only representable broker environment. There is no LIVE value.
13. **Credentials stay server-side configuration.** Broker credentials never enter Core values, the database, logs or the web client.
14. **Failure isolation.** A demo execution failure cannot stop market-data acquisition, research freezing or paper trading.
15. **(commitment) Insert-only journal.** Broker journal persistence will be insert-only.
16. **No wall clock in Core.** `broker_execution` reads no wall clock, generates no identifier and reads no environment. Instants such as an observation time enter as PointInTime values supplied by the caller.

## Rationale

### Why a Separate Package Rather Than Generalizing Paper Trading

Generalizing FuturesExecutionIntent to accept either a paper portfolio or a broker account would change the paper contract, its persistence and its tests to serve a second execution engine. Two small values that each say plainly what they describe are a smaller change, and the same choice was already made for FuturesAssetAnalysis rather than genericizing AssetAnalysis.

Distinct types also give the strongest available isolation: a demo intent cannot be stored as a simulated order, and a paper intent cannot be submitted to a broker, because neither value is accepted where the other is required.

### Why Broker Facts Are Core Values

Paper execution facts (orders, fills and positions) are already Core values. Broker facts carry comparable domain rules: a filled count cannot exceed the requested count, a FILLED order is completely filled, a REJECTED order filled nothing, and an execution follows the decision it executes. Keeping those rules in Core means every later layer receives facts that are already coherent.

### Why Only DEMO Is Representable

Excluding real-money execution structurally means no configuration flag can enable it by mistake. Adding it later is a deliberate change to a Core vocabulary, reviewed as such.

### Why No Separate Partially-Filled or Submitted Status

A partial fill is a filled count above zero on an order that is not FILLED; a separate status would state one fact twice. Whether Northstar has prepared or attempted a submission is Northstar's own journal state, not a broker report, and belongs to the Application journal.

### Why Broker Positions Carry No Average Entry

Brokers report average prices in their own conventions. Until a convention is frozen, reconciliation compares signed net contracts only.

### Why Executions Must Follow the Decision Strictly

A decision instant is the completion of the session decided upon. An order reaches a broker only after that decision is frozen from the completed session, so a genuine execution is always later. PointInTime is canonical UTC at up to microsecond precision and is compared chronologically, so strict ordering is exact and matches the existing rule for simulated fills.

## Alternatives Considered

- **Reuse FuturesExecutionIntent with a PaperPortfolioIdentity naming the demo account.** Rejected: it misnames a broker account as a paper portfolio and lets demo and paper values cross boundaries.
- **Generalize FuturesExecutionIntent over its owner.** Rejected: it changes the released paper contract and its persistence.
- **Reuse FuturesPosition for broker positions.** Rejected: it requires an average entry in the product convention, which broker reports do not reliably provide.
- **Model broker execution with the legacy Orders, Trades and Portfolio aggregates.** Rejected: they are built on Listing identity, which the Futures model deliberately does not use.
- **Represent execution as an Application-only concept, as the generic Execution Application documents describe.** Rejected for Futures: see Relationship to Earlier Documentation.

## Relationship to Earlier Documentation

The Execution Application documents (Module, Contracts, Ports and Coordinator design specifications v1.0) describe a generic execution coordinator over Order, Trade and Portfolio aggregates, and treat execution as an Application capability without its own domain model.

For Futures, this ADR supersedes any conflicting stance in those documents. Futures broker execution facts are Core values in `broker_execution`, built on FuturesContract rather than Listing, and orchestrated by Futures-specific Application use cases in later stories. Those documents are not rewritten by this ADR.

## Consequences

### Positive Consequences

- Broker demo execution can be built without touching the released paper engine.
- Paper and demo cannot contaminate each other's history through type confusion.
- Real-money execution cannot be represented.
- Later stories inherit fixed decisions on idempotency, reconciliation and isolation.

### Implementation Consequences

- Story 10.1 adds the Core `broker_execution` package and this ADR only.
- OrderSide and FuturesContractCount are reused from `paper_trading` in place. Moving them to a shared package is deferred cleanup.
- The target-position policy will be extracted from the paper intent use case in a later Application story, with the paper behaviour unchanged.

### Future Consequences

- Demo profit and loss will need the paper position fold generalized or paralleled for broker executions.
- A real-money environment, if ever approved, requires a new ADR and a deliberate change to BrokerEnvironment.

## Scope

This ADR establishes the broker demo execution boundary and freezes its invariants.

It does not:

- choose a broker,
- design Application ports or use cases,
- design persistence or schemas,
- design the broker adapter or contract mapping implementation,
- define risk-control configuration,
- change any paper-trading value, port, store, table or behaviour,
- or rewrite the Execution Application documents.

## Future Considerations

- Broker selection is a separate current-market research task that must complete before the Application broker ports are frozen. The chosen broker must support client-assigned order references that can be looked up at the broker.
- Commission, fees, margin and account state are deferred.
- Automatic contract rollover remains deferred; demo positions are managed by explicit contract expiration.

## Related Documents

- ADR-002: Architecture Constitution
- ADR-007: PointInTime Foundation Extension
- Execution Application Module Design Specification v1.0
- Execution Application Contracts Design Specification v1.0
- Execution Application Ports Design Specification v1.0
- Execution Coordinator Implementation Design Specification v1.0
