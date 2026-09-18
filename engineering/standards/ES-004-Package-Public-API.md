# ES-004: Package Public API

## Purpose

This standard defines how packages expose their public surface in Northstar.

## Scope

This standard applies to all packages and subpackages that define public functionality for other packages or applications.

## Mandatory Rules

- Every package MUST expose its public API through its package initializer file, namely __init__.py.
- Public API symbols MUST be declared explicitly.
- __all__ MUST be used for export declarations.
- Wildcard imports MUST NOT be used.
- Consumers MUST import from packages rather than from implementation modules.
- Private implementation modules MUST NOT be part of the supported public interface.

## Recommended Practices

- Package boundaries SHOULD be explicit and stable.
- Internal modules SHOULD contain implementation details that are not intended for direct external use.
- Imports SHOULD flow from package-level APIs rather than from concrete implementation files.
- Public APIs SHOULD remain small and intentional.

## Examples

### Good

```python
from northstar_core.foundation.value_objects import Symbol
```

### Bad

```python
from northstar_core.foundation.value_objects.symbol import Symbol
```

### Allowed Imports

- Package-to-package imports through the public API.
- Internal imports within the same package when they are implementation details.

### Forbidden Imports

- Imports from private modules by external packages.
- Wildcard imports.
- Circular imports introduced by exposing implementation details as part of the public surface.

## Common Mistakes

- Exposing implementation modules as the public contract.
- Allowing package internals to leak into other packages.
- Using wildcard exports that make the API unclear.
- Creating package boundaries that are not aligned with responsibility.

## Review Checklist

- [ ] The public API is explicitly declared.
- [ ] The package initializer is the entry point for consumers.
- [ ] Private internals are not exposed accidentally.
- [ ] Imports follow the package boundary model.

## Related Standards

- ES-001: Value Objects
- ES-002: Reference Implementations
- ES-005: Code Review Checklist

## References

- Architecture Constitution
- Domain Handbook
- Python package design conventions
