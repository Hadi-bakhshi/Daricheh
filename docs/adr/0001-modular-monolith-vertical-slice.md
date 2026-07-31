# ADR-0001: Modular Monolith + Vertical Slice Architecture

**Status:** Accepted

## Decision
The backend is a single deployable (modular monolith), internally organized by feature (vertical slice) rather than by technical layer. Modules: Gateway, Plugins, Authentication, Consumers, Workspaces. Each module owns its own `Domain`, `Infrastructure`, `Commands`, `Queries`, and `Endpoints`.

## Context
Solo developer, portfolio-and-eventually-real-users project. Needs to move fast without accumulating the coordination overhead of microservices, but should keep clean seams so modules could be extracted later if the project grows a team or real scale needs.

## Alternatives considered
- **Clean/Onion architecture** — horizontal layers (Domain/Application/Infrastructure) tend to produce anemic cross-cutting projects for a project this size; feature code ends up scattered across three projects for every change.
- **Microservices from day one** — operational overhead (N deployables, N CI pipelines, distributed debugging) with no corresponding benefit at current scale.

## Consequences
- Fast to add a feature: touch one module's folder.
- Module boundaries are enforced by an architecture test (NetArchTest), not just convention — see CONTRIBUTING.md.
- If the project needs to scale to a team or split into services later, module boundaries are already the seam; extraction is additive, not a rewrite.
