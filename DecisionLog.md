## Business logic must never depend on the API or the UI.

axiom-core is the source of truth for all domain logic. Both axiom-api and axiom-web consume it, but it never imports from them.