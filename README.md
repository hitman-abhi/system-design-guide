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

- [Concept Template](Basics/concept-template.md): Reusable structure for writing in-depth concept pages.
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


## How To Use This Repository

- Start with `Basics` if you want a foundation-first path.
- Move to `Intermediate` once the fundamentals feel natural and you want to study distributed coordination and real system composition.
- Use `Advanced` for protocol-level correctness, storage internals, and deeper distributed systems theory.

## Recommended Progression

The sections are intended to build on each other:

1. `Basics`: understand the core building blocks and tradeoffs.
2. `Intermediate`: learn how those building blocks interact in production systems.
3. `Advanced`: study correctness theory, coordination protocols, and internals.
