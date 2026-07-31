# ADR-0004: Outbox Pattern + RabbitMQ for Live Config Propagation

**Status:** Accepted

## Decision
Any write to Service/Route/Plugin-instance config is saved to Postgres and an `outbox_messages` row in the same transaction. A relay process publishes outbox rows to RabbitMQ. Every running gateway node consumes the relevant queue and hot-swaps its in-memory YARP config via `IProxyConfigProvider` — no restart required.

## Context
Config changes made in the Admin UI must reach every gateway replica reliably and without downtime. This also doubles as the mechanism behind "enable/disable/reload a plugin without restart."

## Alternatives considered
- **Poll the database** — simplest, but adds latency and load proportional to replica count × poll frequency.
- **etcd/Consul (watch-based)** — the more idiomatic tool for this exact problem (config distribution), but introduces a coordination service the team has no operating experience with, for a benefit (sub-second propagation) the polling/AMQP combination already gets close enough to. Revisit if propagation latency becomes a real problem.
- **Kafka** — wrong tool for low-volume, transactionally-guaranteed control-plane events; reserved (if ever added) for high-volume analytics telemetry instead.

## Consequences
- Guaranteed delivery via the transactional outbox — no lost config changes even if RabbitMQ is briefly unavailable (relay retries from the outbox table).
- Every gateway node also gets a `gateway_nodes` heartbeat + "applied config version" row, making config lag observable in the dashboard rather than just assumed.
