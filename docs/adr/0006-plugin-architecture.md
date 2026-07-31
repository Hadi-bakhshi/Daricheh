# ADR-0006: Native Plugin Architecture via `AssemblyLoadContext`, WASM Planned

**Status:** Accepted

## Decision
Plugins implement `IGatewayPlugin` (middleware-shaped: `InvokeAsync(HttpContext, Func<Task> next)`). Native plugins are `.dll`s loaded into a dedicated, collectible `AssemblyLoadContext` per plugin, discovered from a `/plugins` folder, loadable/unloadable/reloadable without restarting the gateway process. Each Route has an ordered list of Plugin Instances (`route_id, plugin_id, config jsonb, enabled, order`) — this list *is* the pipeline; no separate pipeline entity.

## Context
The plugin system is the platform's main extensibility and differentiation point. It needs to support enable/disable/reorder from the UI without a restart, and eventually support plugins written in languages other than C#.

## Alternatives considered
- **WASM (Extism) from v1** — the right long-term direction (true sandboxing, language-agnostic), but a project in itself (host function bindings, marshaling). Deferred to a second plugin loader behind the same `IGatewayPlugin`-shaped contract, so adding it later is additive, not a rewrite of the pipeline.
- **Flat JSON array of plugin keys per route** — rejected in favor of a queryable join table (`plugin_instances`) so each instance carries its own config, enabled flag, and explicit order without a second lookup.

## Consequences
- Every plugin: unique key, self-describing JSON config schema, self-validates its own config, stateless (Redis for shared state like rate-limit counters is fine; direct DB access is not).
- Module boundary rule applies: plugins never reference module internals directly, only the `PulseGate.Plugins.Abstractions` contract.
