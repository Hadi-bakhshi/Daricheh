# ADR-0002: CQRS with Direct Handler Calls, No Mediator Library

**Status:** Accepted

## Decision
Commands and Queries are plain classes with a corresponding Handler class, invoked directly from the Minimal API endpoint (`app.MapPost(...) => CreateRouteEndpoint.Handle`). No MediatR, no Wolverine.

## Context
CQRS is a natural fit (gateway config changes are commands, dashboard/route listings are queries). The question was whether to route those calls through a mediator library.

## Alternatives considered
- **MediatR** — solves a decoupling problem (caller doesn't know handler) that doesn't exist yet for a solo developer working in vertical slices. Adds reflection-based indirection and an extra layer to step through when debugging.
- **Wolverine** — messaging-capable mediator, but RabbitMQ + outbox already covers the messaging need; adopting Wolverine means learning a second abstraction over infrastructure already understood deeply.

## Consequences
- One fewer dependency, one fewer indirection layer to debug through.
- If cross-cutting pipeline behaviors (validation, logging, transactions) become repetitive across handlers, revisit — a source-generated option (e.g. `Mediator` by martinothamar) avoids the reflection complaints against classic MediatR and is the fallback if this decision needs to be reversed.
