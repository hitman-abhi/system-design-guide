# Consistency Models

## 1. Overview

Consistency models define what a system promises readers will observe after writes happen. They answer a deceptively simple question:

> If one client updates data, when and how will other clients see that update?

That question sits at the center of distributed systems. Once data is replicated across nodes, cached across layers, and accessed concurrently by many clients, "what value should a read return?" stops being trivial. A consistency model gives a precise contract for that behavior.

This topic matters because distributed systems do not fail only by crashing. They also fail by becoming confusing. A system may accept a write but show stale data to another user. Two users may observe the same events in different orders. A client may read older data than it saw a moment ago. Consistency models exist to define which of these outcomes are allowed and which are forbidden.

## 2. The Core Problem

A single-threaded program updating local memory is simple: there is one machine, one copy of data, and one clear execution order.

Distributed systems are different:

- data is replicated across machines
- writes arrive concurrently from multiple clients
- messages can be delayed or reordered
- replicas may lag behind each other
- clients may read from different nodes across requests

Once those conditions exist, there is no automatic guarantee that every client will observe the same state at the same time.

Example:

1. Client A updates a user's profile picture.
2. The write reaches one replica.
3. Client B reads from another replica before replication completes.

What should Client B see?

- the new picture
- the old picture
- an error
- a response that depends on which replica handled the request

There is no single natural answer. The answer depends on the system's consistency model.

## 3. Formal Statement

A consistency model is a specification of the visibility and ordering guarantees a storage system provides for reads and writes.

More concretely, a consistency model defines:

- when a write becomes visible to readers
- whether all clients observe writes in the same order
- whether a client can observe its own writes immediately
- whether a later read can return older data than an earlier read
- what anomalies are allowed under concurrency and replication

Consistency models are not about whether data is "correct" in a business sense. They are about observable behavior at the interface between clients and the system.

They also do not say everything about a system. A system can have a strong consistency model and still be slow, unavailable, or operationally fragile. Likewise, a system can have a weak consistency model and still be highly reliable for its intended use case.

## 4. Key Terms

### 4.1 Read

A read retrieves a value from the system.

The key question is not just whether the read succeeds, but what version of the data it is allowed to return.

### 4.2 Write

A write changes state in the system.

In distributed systems, a write often becomes durable or visible in stages:

- accepted by one node
- replicated to other nodes
- acknowledged to the client
- observed by future readers

Different consistency models impose different rules on when that write must become visible.

### 4.3 Visibility

Visibility describes which writes a given read is allowed to observe.

If a write has happened, visibility rules determine whether:

- all readers must see it immediately
- only some readers may see it
- the writer itself must see it
- other readers may still see older state for some time

### 4.4 Ordering

Ordering defines how operations appear relative to one another.

In a strong model, all clients may observe writes in a single global order. In weaker models, different clients may observe different orders temporarily.

### 4.5 Stale Read

A stale read returns an older value even though a newer write has already completed somewhere in the system.

Stale reads are not always bugs. In many systems they are an accepted tradeoff for lower latency or higher availability.

### 4.6 Linearizability

Linearizability is one of the strongest and most intuitive consistency models.

It guarantees:

- once a write is acknowledged, every later read must return that write or something newer
- operations appear to occur atomically in a single real-time order

This makes a distributed system behave like one correct shared object, at least from the perspective of completed operations.

### 4.7 Sequential Consistency

Sequential consistency guarantees that all clients observe operations in the same order, but not necessarily the real-time order in which they occurred.

This is weaker than linearizability.

### 4.8 Eventual Consistency

Eventual consistency guarantees that if writes stop, all replicas will eventually converge to the same state.

It does not guarantee immediate visibility, monotonicity, or a single shared order during convergence.

### 4.9 Session Guarantees

Session guarantees are client-centric promises that improve usability even when the global model is weak.

Examples:

- read-your-writes
- monotonic reads
- monotonic writes
- writes-follow-reads

These are often more useful to product behavior than the high-level label alone.

## 5. What It Really Means

Consistency models are not academic decoration. They are user-visible behavior contracts.

A weak model can produce experiences such as:

- a user updates a setting and immediately sees the old value
- a post appears in one device but not another
- inventory looks available in one request and unavailable in the next
- two services act on different versions of the same record

A strong model can avoid much of that confusion, but it usually requires more coordination:

- more network round trips
- higher latency
- lower tolerance for partitions or lag
- lower write throughput at scale

So the real engineering question is not "Which model is best?" It is:

- how wrong is the system allowed to look
- for how long
- to whom
- under what failure conditions

That is the practical meaning of consistency models.

## 6. Why the Constraint Exists

Consider a service with three replicas in different zones.

1. Client A writes `account_status = suspended`.
2. Replica `R1` accepts the write.
3. Replication to `R2` and `R3` is delayed.
4. Client B reads from `R2`.

Now the system has to choose what behavior to allow.

Option 1:

- block the read until `R2` has caught up or a quorum confirms the latest value

Result:

- the read is likely correct
- latency increases
- availability may drop during network issues

Option 2:

- return the value currently stored on `R2`

Result:

- the read is fast and available
- it may be stale
- two clients can see different realities for a period of time

This is why consistency models exist. Once systems replicate data and tolerate delay, they need explicit rules for what readers are allowed to observe.

## 7. Main Variants or Modes

### 7.1 Strong Consistency / Linearizability

Strong consistency usually refers to linearizable behavior.

Guarantees:

- once a write is acknowledged, later reads return that value or a newer one
- all clients observe a single coherent timeline of completed operations

Strengths:

- simple mental model
- fewer user-visible anomalies
- easier reasoning for correctness-critical workflows

Costs:

- higher latency from coordination
- reduced availability under partition or leader failure
- more operational complexity at global scale

Where it fits:

- financial ledgers
- lock services
- metadata stores
- control planes

### 7.2 Sequential Consistency

Sequential consistency preserves a single global order of operations, but that order does not have to match wall-clock time.

Implication:

- all clients agree on operation order
- the order may still violate real-time intuition

This is stronger than eventual consistency, but weaker than linearizability.

### 7.3 Causal Consistency

Causal consistency guarantees that causally related operations are seen in the same order by all clients.

Example:

1. Alice posts a message.
2. Bob replies to that message.

A causally consistent system ensures no client sees Bob's reply before Alice's original post.

At the same time, unrelated writes may be observed in different orders by different clients.

This model is a useful middle ground because it preserves meaningful user intent without requiring a total global order for everything.

### 7.4 Eventual Consistency

Eventual consistency guarantees convergence if updates stop.

What it allows during normal operation:

- stale reads
- temporary replica divergence
- conflicting concurrent writes
- different clients observing different values

Strengths:

- high availability
- low latency
- better tolerance of partitions and replication lag

Costs:

- confusing user-visible behavior unless carefully managed
- need for conflict resolution
- harder reasoning across services

Where it fits:

- social feeds
- caching layers
- analytics pipelines
- shopping carts with merge semantics

### 7.5 Read-Your-Writes Consistency

This is a session guarantee, not a full global consistency model.

Guarantee:

- after a client writes data, that same client will see its own write on future reads

This is often enough to make systems feel much more intuitive, even if other users still see stale data temporarily.

### 7.6 Monotonic Reads

Guarantee:

- once a client has seen a value, it will not later see an older value

This avoids the frustrating experience where a user refreshes and sees the system appear to move backward.

### 7.7 Monotonic Writes

Guarantee:

- a client's writes are applied in the order they were issued

This matters for workflows where order is semantically important, such as state transitions.

### 7.8 Writes-Follow-Reads

Guarantee:

- if a client reads some state and then issues a write based on it, the write is ordered after the observed state

This helps preserve causality in user-driven flows.

## 8. Supporting Mechanisms and Related Ideas

### 8.1 Replication Strategy

Consistency behavior is tightly coupled to replication.

- synchronous replication usually improves consistency but increases latency
- asynchronous replication improves responsiveness but increases the chance of stale reads

The storage model and the replication strategy cannot be reasoned about separately.

### 8.2 Quorums

Quorum systems influence consistency by requiring overlap between reads and writes.

With:

- `N` replicas
- `W` write acknowledgments
- `R` read participants

If `R + W > N`, successful reads and writes overlap at at least one replica.

That overlap can support strong guarantees, but only if the system also handles versioning, leader rules, or conflict resolution correctly.

### 8.3 Conflict Resolution

Weaker consistency often shifts burden from coordination time to reconciliation time.

Common strategies:

- last-write-wins
- vector clocks
- application-defined merge logic
- CRDTs
- compensating actions

Every weak consistency model needs a clear story for what happens when concurrent updates collide.

### 8.4 CAP and PACELC

Consistency models connect directly to broader system tradeoffs.

- CAP explains the tension between consistency and availability during partitions
- PACELC explains that outside partitions, the tension is often latency versus consistency

A consistency model is rarely chosen in isolation. It is usually chosen together with a latency target, availability target, and failure model.

### 8.5 Isolation Levels vs Consistency Models

These two ideas are often mixed up, but they solve different problems.

- **Consistency models** describe what distributed readers and writers can observe across replicas and time
- **Isolation levels** describe how concurrent transactions interact inside a database

Examples of isolation levels:

- read committed
- repeatable read
- serializable

A system can have strong transactional isolation within one node and still expose weak distributed consistency across replicas.

## 9. Real-World Examples

### 9.1 User Profile Update

A user changes their display name and refreshes the page.

If the system provides read-your-writes consistency, the user sees the new value immediately. Without it, the user may think the update failed even though it succeeded.

This is a good example of why client-centric guarantees matter.

### 9.2 Banking Ledger

Banking ledgers usually need linearizable or otherwise very strong consistency guarantees.

Reason:

- stale reads can cause double spending
- conflicting updates can violate account invariants
- operator reconciliation is not an acceptable primary correctness model

### 9.3 Social Feed

A social feed can often tolerate eventual consistency.

Reason:

- temporary lag is acceptable
- ordering is valuable but does not always need global precision
- availability and throughput are usually more important than immediate synchronization

### 9.4 Distributed Locks

A lock service must behave with very strong consistency.

If two clients both believe they hold the same lock, the lock service has failed in its core purpose.

### 9.5 DNS

DNS is broadly eventual in practice.

- records propagate gradually
- caches serve stale responses for a period of time
- the system remains highly available and scalable

This tradeoff works because the domain tolerates temporary staleness better than widespread lookup failure.

## 10. Common Misconceptions

### 10.1 "Eventual Consistency Means Data Is Random"

It does not.

Eventual consistency still defines a rule:

- if updates stop, replicas converge

The weakness is not lack of structure. The weakness is that the model allows temporary divergence and delayed visibility.

### 10.2 "Strong Consistency Is Always Better"

Strong consistency is easier to reason about, but it is not free.

It often costs:

- latency
- throughput
- operational complexity
- availability under failure

If the domain can tolerate weaker guarantees, paying those costs everywhere is wasteful.

### 10.3 "Read Replicas Automatically Give the Same View"

They do not.

Read replicas commonly lag behind primaries. Unless the system routes reads carefully or waits for replica catch-up, readers may see stale state.

### 10.4 "Consistency and Isolation Are the Same Thing"

They are related but different.

Isolation focuses on concurrent transaction behavior. Consistency models focus on visibility and ordering across distributed reads and writes.

### 10.5 "One System Needs One Consistency Model Everywhere"

Large systems often use multiple models:

- strong consistency for metadata and money movement
- weaker consistency for feeds, caches, and analytics
- session guarantees to smooth the user experience

This is usually a sign of good design, not inconsistency in architecture.

## 11. Design Guidance

Choose consistency based on the business cost of disagreement.

Questions to ask:

- what happens if two users see different values for 5 seconds
- what happens if a user cannot immediately see their own write
- what happens if updates are applied in different orders on different replicas
- what happens if two concurrent writes conflict
- what happens during a partition or replica lag spike

Use stronger models when:

- correctness is safety-critical
- stale reads cause financial or security harm
- order matters globally
- coordination state drives the behavior of other systems

Use weaker models when:

- latency and availability matter more than immediate global agreement
- temporary divergence is acceptable
- conflicts can be merged safely
- the product can tolerate delayed visibility

Practical pattern:

- keep core invariants in strongly consistent stores
- use weaker models for high-volume, user-tolerant paths
- add session guarantees to improve perceived correctness
- be explicit about stale-read behavior in APIs and product flows

The worst design is not choosing the wrong model. It is choosing one accidentally.

## 12. Reusable Takeaways

- Consistency models define what reads are allowed to observe after writes.
- Stronger consistency reduces ambiguity but usually requires more coordination.
- Weaker consistency improves scalability and availability, but allows more anomalies.
- Session guarantees can make weakly consistent systems feel much more intuitive.
- Eventual consistency guarantees convergence, not immediate agreement.
- Consistency models and transaction isolation solve different problems.
- The right model depends on the cost of stale, reordered, or conflicting data.

## 13. Summary

Consistency models are the rules that govern how distributed state becomes visible to readers.

They matter because the hardest part of a distributed system is often not storing data, but making that data appear coherent across nodes, services, and users.

The central tradeoff is straightforward:

- stronger guarantees require more coordination
- weaker guarantees allow more flexibility, lower latency, and better fault tolerance

Understanding that tradeoff is essential for designing systems that are both scalable and predictable.
