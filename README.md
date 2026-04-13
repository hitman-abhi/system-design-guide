# System Design Guide

This repository is a structured knowledge base for system design fundamentals. The goal is to explain core concepts in a way that is precise, practical, and reusable, with enough depth to support real engineering reasoning instead of just high-level definitions.

The notes are written as standalone concept pages. Each page focuses on:

- what the concept is
- why it exists
- what tradeoffs it introduces
- how it appears in real systems
- how to think about it when designing software

## Structure

The repository is currently organized by topic area.

### Basics

Foundational distributed systems concepts that support more advanced design decisions.

- [CAP Theorem](Basics/cap-theorem/capTheorem.md): Tradeoffs between consistency, availability, and partition tolerance during network failures.
- [Consistency Models](Basics/consistency-models/consistencyModels.md): Visibility and ordering guarantees for reads and writes in distributed systems.
- [Replication](Basics/replication/replication.md): How systems maintain multiple copies of data for durability, availability, scale, and recovery.
- [Partitioning and Sharding](Basics/partitioning-sharding/partitioningSharding.md): How datasets are split across nodes to scale storage, throughput, and operational capacity.
- [Consistent Hashing](Basics/consistent-hashing/consistentHashing.md): How keys are assigned to nodes with minimal remapping when cluster membership changes.
- [Caching](Basics/caching/caching.md): How reusable data is stored closer to readers to reduce latency and backend load.
- [Load Balancing](Basics/load-balancing/loadBalancing.md): How traffic is distributed across backends to improve utilization, resilience, and tail latency.
- [Horizontal vs Vertical Scaling](Basics/horizontal-vs-vertical-scaling/horizontalVsVerticalScaling.md): How systems buy more capacity either by strengthening one node or distributing work across many.
- [Idempotency](Basics/idempotency/idempotency.md): How systems make retries safe and prevent duplicate side effects.
- [Database Fundamentals](Basics/database-fundamentals/databaseFundamentals.md): A guided foundation chapter covering SQL, ACID, NoSQL families, BASE, and datastore tradeoffs.
- [Networking Basics](Basics/networking-basics/networkingBasics.md): A guided foundation chapter covering DNS, HTTP/HTTPS, the OSI model, and end-to-end request flow.
- [Message Queues](Basics/message-queues/messageQueues.md): How systems decouple producers and consumers, absorb spikes, and process work asynchronously.
- [Rate Limiting](Basics/rate-limiting/rateLimiting.md): How systems protect shared capacity and enforce fairness under load.
- [API Design Basics](Basics/api-design-basics/apiDesignBasics.md): A practical foundation chapter covering resources, HTTP semantics, errors, pagination, idempotency, and compatibility.
- [Indexing Basics](Basics/indexing-basics/indexingBasics.md): How indexes reduce lookup cost, when they help, and what they cost on writes.
- [Latency vs Throughput](Basics/latency-vs-throughput/latencyVsThroughput.md): How to reason about per-request speed versus total system output.
- [Availability and Fault Tolerance Basics](Basics/availability-fault-tolerance-basics/availabilityFaultToleranceBasics.md): How systems stay reachable and correct enough under component failure.
- [Stateless vs Stateful Services](Basics/stateless-vs-stateful-services/statelessVsStatefulServices.md): How service state placement changes scaling, routing, and recovery.

### Intermediate

Topics that build on the fundamentals and focus on how distributed building blocks are composed in real systems.

Section index: [Intermediate](Intermediate/README.md)

- [Quorum](Intermediate/quorum/quorum.md): How replicated systems use read and write thresholds to create overlap and safe coordination.
- [Leader Election](Intermediate/leader-election/leaderElection.md): How distributed systems choose and maintain one valid authority.
- [Distributed Locks](Intermediate/distributed-locks/distributedLocks.md): How systems coordinate exclusive work safely across machines.
- [Service Discovery](Intermediate/service-discovery/serviceDiscovery.md): How clients find healthy service instances in dynamic environments.
- [Circuit Breakers](Intermediate/circuit-breakers/circuitBreakers.md): How systems stop repeatedly calling unhealthy dependencies before cascading failure spreads.
- [Backpressure](Intermediate/backpressure/backpressure.md): How slower consumers push capacity signals upstream to prevent runaway backlog.
- [Load Shedding](Intermediate/load-shedding/loadShedding.md): How systems deliberately reject lower-priority work to preserve core behavior under overload.
- [Batching](Intermediate/batching/batching.md): How systems group work to improve efficiency while trading off some immediacy.
- [Pub/Sub](Intermediate/pub-sub/pubSub.md): How publishers and subscribers are decoupled through fan-out messaging channels.
- [Event-Driven Architecture](Intermediate/event-driven-architecture/eventDrivenArchitecture.md): How systems react to facts asynchronously instead of coordinating all downstream work through direct calls.
- [Saga Pattern](Intermediate/saga-pattern/sagaPattern.md): How multi-step business workflows recover without one distributed ACID transaction.
- [Outbox Pattern](Intermediate/outbox-pattern/outboxPattern.md): How services avoid dual-write inconsistencies between database commits and event publication.
- [CQRS](Intermediate/cqrs/cqrs.md): How command and query paths diverge when write correctness and read efficiency want different models.
- [Read Replicas](Intermediate/read-replicas/readReplicas.md): How systems offload read traffic and isolate workloads while accepting replica lag tradeoffs.
- [Write-Ahead Logs](Intermediate/write-ahead-logs/writeAheadLogs.md): How storage systems make writes durable before applying them to primary data structures.
- [Schema Evolution](Intermediate/schema-evolution/schemaEvolution.md): How data contracts change safely across versions, deployments, and historical records.
- [Secondary Indexes in Distributed Systems](Intermediate/secondary-indexes-distributed-systems/secondaryIndexesDistributedSystems.md): How alternate query paths are maintained across partitioned data without ignoring consistency and write costs.
- [Pagination Strategies](Intermediate/pagination-strategies/paginationStrategies.md): How APIs and data stores split large result sets while managing traversal stability and query cost.
- [API Gateways](Intermediate/api-gateways/apiGateways.md): How edge layers centralize routing, policy enforcement, and client-facing contracts.
- [Webhooks](Intermediate/webhooks/webhooks.md): How push-based HTTP callbacks work, and why retries, signatures, and idempotency matter.
- [CDN Design](Intermediate/cdn-design/cdnDesign.md): How edge caching and traffic control reduce origin load and improve global latency.
- [Failover Strategies](Intermediate/failover-strategies/failoverStrategies.md): How systems detect failure and shift traffic or authority safely to alternate paths.
- [Disaster Recovery Basics](Intermediate/disaster-recovery-basics/disasterRecoveryBasics.md): How systems restore service and data after failures that exceed normal high-availability assumptions.
- [Multi-Region Design](Intermediate/multi-region-design/multiRegionDesign.md): How systems trade coordination cost for regional resilience, compliance, and lower user latency.
- [Security Basics for Distributed Systems](Intermediate/security-basics-distributed-systems/securityBasicsDistributedSystems.md): How trust, identity, transport, and blast radius need to be designed across service boundaries.
- [Authentication vs Authorization](Intermediate/authentication-vs-authorization/authenticationVsAuthorization.md): How identity proof and permission enforcement solve different access-control problems.
- [Session Management](Intermediate/session-management/sessionManagement.md): How authenticated user continuity is maintained safely across requests and deployments.
- [Connection Pooling](Intermediate/connection-pooling/connectionPooling.md): How systems reuse bounded backend connections to reduce setup cost and protect shared capacity.

### Case Studies

End-to-end system design case studies that combine multiple concepts into realistic product designs.

Section index: [Case Studies](Case-Studies/README.md)

- [Distributed Cache](Case-Studies/distributed-cache/distributedCache.md): How to design a Redis-like cache with sharding, replication, failover, eviction, and hot-key handling.
- [Distributed File System](Case-Studies/distributed-file-system/distributedFileSystem.md): How to design a GFS-like storage system with metadata management, chunk replication, append semantics, and repair workflows.
- [Leaderboard System](Case-Studies/leaderboard-system/leaderboardSystem.md): How to design a ranked serving system with durable score ingestion, ordered state maintenance, fast rank reads, and hot-board handling.
- [Logging System](Case-Studies/logging-system/loggingSystem.md): How to design a log platform with durable ingestion, parse and enrich pipelines, indexing strategy, and separation of real-time search from batch processing.
- [Metrics and Monitoring System](Case-Studies/metrics-monitoring-system/metricsMonitoringSystem.md): How to design a Prometheus-like system with high-write ingestion, TSDB storage, aggregations, retention tiers, and alert evaluation.
- [News Feed System](Case-Studies/news-feed-system/newsFeedSystem.md): How to design a large-scale personalized home feed with hybrid fan-out, candidate generation, ranking stages, cache strategy, and hot-author handling.
- [Notification Service](Case-Studies/notification-service/notificationService.md): How to design a shared multi-channel notification platform with preferences, templates, scheduling, retries, provider integration, and delivery tracking.
- [Rate Limiter](Case-Studies/rate-limiter/rateLimiter.md): How to design a low-latency enforcement system with configurable policy, hot counter state, token-bucket or window algorithms, and explicit fail-open or fail-closed behavior.
- [Recommendation System](Case-Studies/recommendation-system/recommendationSystem.md): How to design a Netflix-like recommendation platform with event ingestion, feature stores, candidate retrieval, ranking, and row assembly.
- [Ride Sharing App](Case-Studies/ride-sharing-app/rideSharingApp.md): How to design an Uber-like platform with realtime driver location ingestion, geo-based dispatch, durable trip state, pricing boundaries, and payment workflows.
- [Search System](Case-Studies/search-system/searchSystem.md): How to design an Elasticsearch-like platform with indexing pipelines, distributed shards, relevance ranking, and scatter-gather query execution.
- [Stock Broker Platform](Case-Studies/stock-broker-platform/stockBrokerPlatform.md): How to design a Robinhood-like trading platform with market data fan-out, order entry, risk checks, routing, execution processing, portfolio state, and settlement boundaries.
- [Ticket Booking System](Case-Studies/ticket-booking-system/ticketBookingSystem.md): How to design a high-contention seat inventory system with seat-map reads, temporary holds, transactional booking, payment orchestration, and ticket issuance.
- [TinyURL](Case-Studies/tinyurl/tinyUrl.md): How to design a read-heavy URL shortening service with fast redirects, unique code generation, caching, and async analytics.
- [Video Streaming Service](Case-Studies/video-streaming-service/videoStreamingService.md): How to design a YouTube-style platform with durable uploads, async transcoding, metadata management, adaptive playback, CDN delivery, and engagement logging.
- [WhatsApp Messenger](Case-Studies/whatsapp-messenger/whatsAppMessenger.md): How to design a large-scale messaging system with 1:1 chat, group messaging, delivery flow, and read receipts.


## How To Use This Repository

- Start with `Basics` if you want a foundation-first path.
- Move to `Intermediate` once the fundamentals feel natural and you want to study distributed coordination and real system composition.
- Use `Advanced` for protocol-level correctness, storage internals, and deeper distributed systems theory.

## Recommended Progression

The sections are intended to build on each other:

1. `Basics`: understand the core building blocks and tradeoffs.
2. `Intermediate`: learn how those building blocks interact in production systems.
3. `Advanced`: study correctness theory, coordination protocols, and internals.
