# ADR-0003: Result Pattern for Business Outcomes, Exceptions for Infrastructure Failures Only

**Status:** Accepted

## Decision
Handlers return a `Result` / `Result<T>` (Success, ValidationError, NotFound, Unauthorized, Forbidden, Conflict) for any expected business outcome. Exceptions are reserved for infrastructure failure (DB unreachable, Redis down) and programmer error (impossible state, bugs).

## Context
"Route already exists" or "invalid plugin config" are expected, frequent outcomes — not exceptional control flow.

## Alternatives considered
- **Exceptions for everything** — expensive (stack unwinding), and turns expected business outcomes into try/catch soup at every call site.

## Consequences
- Endpoints have one job: convert `Result` → HTTP response. No business logic in endpoints.
- Every handler follows the same template: Validate → Load → Apply business rules → Persist → Publish domain events → Return Result.
- A shared `PulseGate.Shared.Result` type is used across all modules — this is one of the few things every module is allowed to depend on.
