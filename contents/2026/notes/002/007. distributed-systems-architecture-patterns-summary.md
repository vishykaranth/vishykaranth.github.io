# Distributed Systems & Architecture Patterns — Context from Key Books

Sources referenced: *Designing Data-Intensive Applications* (Kleppmann), *Distributed Systems* (Tanenbaum & van Steen), *Site Reliability Engineering* (Google), *Release It!* (Nygard), *Building Microservices* (Newman), *Microservices Patterns* (Richardson), *Patterns of Enterprise Application Architecture* (Fowler), *The Art of Scalability* (Abbott & Fisher).

## Core Fundamentals (DS)
- Failure model: crash vs byzantine; unreliable networks (loss, reorder, duplication), partitions, clocks skew → CAP/FLP realities.
- Replication: leader–follower, multi-leader, leaderless/quorum; sync vs async; RYW/monotonic/causal vs linearizable guarantees.
- Partitioning/sharding: hash/range/directory; secondary indexes across shards; rebalancing and hotspot mitigation; consistent hashing.
- Consensus & coordination: Paxos/Raft for membership/config/log replication; fencing tokens for lease-based safety; failure detectors (accrual/phi).
- Messaging semantics: at-most/at-least/effectively-once; idempotent producers/consumers; ordering scopes (key/partition).
- Consistency models: eventual, causal, snapshot, serializable; anomalies (lost update, write skew, phantom) and remedies (locks, OCC, SSI).
- Time & ordering: logical clocks (Lamport/vector), total order vs causal order; stream offsets and watermarks.

## Architecture Patterns (Systems & Services)
- Layering & modularity: hexagonal/ports-adapters, bounded contexts (DDD), strangler-fig for legacy carve-out.
- Integration: sync HTTP/gRPC vs async event-driven; outbox + CDC for reliable async; sagas for long-running, cross-service workflows.
- CQRS & Event Sourcing: separate read/write models; append-only logs as source of truth; projections/materialized views.
- Caching: aside, read/write-through, write-behind; stampede protection (locks, jitter, request coalescing); tiered cache (CDN/edge/app).
- Resilience: retries with backoff+jitter, circuit breakers, bulkheads, rate limiting, load shedding, timeouts, hedging (carefully).
- Data locality & domains: colocate data with services; avoid shared mutable DBs; apply domain-aligned sharding keys.
- Deployment topologies: single-region vs multi-region (active-active/active-passive); blast-radius isolation via cells/zones.
- Observability baked-in: RED/USE, structured logs, traces with IDs; SLOs/error budgets guiding change velocity.

## Reliability & Operations (SRE + Release It!)
- Design for steady-state and failure modes: backpressure, bounded queues, bulkheading.
- Chaos/fault injection and game days; runbooks; automated remediation where safe.
- Release safety: blue/green, canary, feature flags; fast rollback; config is code.
- Capacity & scaling: plan for headroom; autoscale on leading signals (queue depth, P99); cost/perf trade-offs.

## Data & Storage Patterns (DDIA)
- Storage engines: B-Tree vs LSM; impact on write/read amplification and compaction.
- Streams/logs: log as system of record; downstream projections; batch (MapReduce/Spark) vs stream processors (Flink/Kafka Streams).
- Schema evolution: forward/backward compatibility; contracts on the wire (Avro/Protobuf/JSON schema).
- Conflict handling: single-writer vs CRDTs/vector clocks/LWW; choose per domain.

## Microservices-Specific (Newman/Richardson/Fowler)
- Service boundaries: split by domain capability; avoid chatty synchronous graphs; prefer async integration where possible.
- Service contracts: versioning, backward-compatible changes, consumer-driven contracts.
- Cross-cutting: authn/z (OAuth2/OIDC/JWT/mTLS), rate limits, quotas, multitenancy isolation (schema/cluster/row).
- Common pitfalls: distributed monolith (tight sync coupling), over-fragmentation, missing platform (discovery, config, secrets, telemetry).

## Scalability Lenses (Art of Scalability)
- Scale cube: X (clone), Y (functional split), Z (data partition); apply where bottlenecks dictate.
- Org/process mirrors architecture (Conway’s Law); align teams to bounded contexts/capabilities.

## Practical Heuristics
- Start modular (monolith with boundaries); extract when domain is stable and comms are coarse-grained.
- Choose consistency per operation; make invariants explicit; use async + idempotency for side effects.
- Keep critical paths simple and synchronous only where needed; everything else can be async/event-driven.
- Prefer append-only logs + projections for auditability and recovery.
- Always design for observability, backpressure, and safe degradation from day one. 

