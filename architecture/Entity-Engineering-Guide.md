# Entity Engineering Guide

## Purpose

This guide provides directional guidance for Entity-oriented Core Domain modeling.

## Entity vs Value Object

Entities differ from Value Objects.

Entities:

- possess identity,
- have lifecycle,
- may change state,
- compose Foundation Value Objects.

Value Objects remain immutable and represent stable business meaning.

## Modeling Direction

Core Domain entities should reuse approved Foundation Value Objects for shared semantics such as identity, measurement, financial value, and temporal references.

This guide is directional and intentionally avoids implementation detail at this stage.
