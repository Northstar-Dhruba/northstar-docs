# Quoted Market State Design Specification v1.0

## 1. Business Purpose

Quoted Market State is a Quote-specific Value Object.

It exists to preserve the quote-specific business meaning reported by the market for one Listing within one temporal context.

Quoted Market State does not redefine Quote, Listing, PointInTime, Market Observation, or Foundation Value Objects.

## 2. Relationship to Price

Quoted Market State composes one or more approved quotation-value concepts.

In Version 1.0, Quoted Market State composes one or more Price values.

Price is the currently approved Foundation Value Object representing denomination-aware monetary valuation and is the primary approved financial building block for quoted market meaning.

Future architectural refinement may extend this composition only through an approved architecture change, without redefining Price itself.

## 3. Business Responsibilities

Quoted Market State is responsible for:

- composing approved quotation-value concepts,
- preserving quote-specific business meaning,
- preserving the distinction between quoted meaning and execution meaning.

For Version 1.0, the approved quotation-value concepts are composed as one or more Price values.

Quoted Market State does not own Listing, PointInTime, workflow state, or execution outcomes.

## 4. Business Invariants

The following invariants must always hold:

- Every Quoted Market State contains nonempty approved quotation-value meaning.
- Quoted Market State remains Quote-specific.
- Quoted Market State remains immutable.
- Quoted Market State remains value-based and has no independent identity.
- Version 1.0 quotation-value meaning is composed as a non-empty tuple of Price values.

## 5. Composition Review

Inside Quoted Market State:

- one or more approved quotation-value concepts.

- in Version 1.0, one or more Price values.

Price is the currently approved financial primitive contributing denomination-aware monetary valuation.

Future Quote refinements may determine how Price contributes to quote-specific meaning without redefining Price.

## 6. Validation Rules

Business validation requires:

- approved quotation-value meaning exists,
- quote-specific meaning is unambiguous,
- composed meaning remains consistent with Quote semantics.

Version 1.0 implementation validation also requires:

- quotation-value meaning is provided as a tuple,
- the tuple is non-empty,
- each composed value is a Price value.

## 7. Normalization

Quoted Market State normalization should preserve each approved quotation-value meaning.

In Version 1.0, normalization preserves tuple composition and each composed Price meaning.

Any normalization inherited from Price remains authoritative for Price itself.

Normalization must preserve quote-specific meaning and must not redefine Quote or Foundation concepts.

## 8. Equality

Quoted Market State equality is value-based.

Equality is evaluated on canonical quotation-value meaning.

In Version 1.0, canonical quotation-value meaning is the normalized tuple of composed Price values.

## 9. Examples

### Valid Example

Quoted market meaning represented using approved quotation-value concepts for one Listing at one temporal context.

Version 1.0 example form: a non-empty tuple of Price values.

Business meaning remains quote-specific and distinct from execution outcomes.

## Approval Summary

```text
Quoted Market State
        |
        v
one or more approved quotation-value concepts
        |
        v
currently supported by Price
```

Version 1.0 implementation form:

```text
Quoted Market State
        |
        v
non-empty tuple[Price, ...]
```

Quoted Market State remains Quote-specific.

Price remains the approved reusable financial primitive while Quote remains free to refine quote-specific meaning without architectural redesign.
