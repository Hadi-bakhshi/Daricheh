# ADR-0005: `IGatewayEngine` Abstraction Over the Reverse Proxy

**Status:** Accepted

## Decision
No module talks to YARP directly. All modules depend on an `IGatewayEngine` interface (register route, apply config snapshot, health status). `YarpGatewayEngine` is the only implementation today.

## Context
YARP is the right choice for v1 (in-process, .NET-native, built for exactly this dynamic-config use case) — but the product being built is the platform (admin portal, plugin engine, config management), not the proxy itself. The proxy is a swappable internal component.

## Alternatives considered
- **Depend on YARP types directly throughout the codebase** — cheaper today, but couples every module that touches routing to YARP's specific config model, making a future swap (e.g. an Envoy/xDS-based `EnvoyGatewayEngine`) a rewrite instead of a new implementation.

## Consequences
- Adding a second engine implementation later (e.g. Envoy via xDS) requires no changes outside `PulseGate.GatewayEngine` and its DI registration.
- Slight upfront cost: one more interface, one more indirection — accepted because the swap this protects against is specifically named on the public roadmap.
