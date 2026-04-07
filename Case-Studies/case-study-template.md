# Case Study Template

Use this structure for end-to-end system design case studies such as:

- TinyURL
- Dropbox
- WhatsApp
- Uber
- live streaming
- stock trading

The expected depth here is higher than a lightweight practice outline.

A good case study should be strong enough that a reader can use it to sharpen design reasoning for senior-level system design discussions:

- requirements clarification
- scale estimation
- architectural tradeoffs
- bottleneck analysis
- failure handling
- evolution over time

The goal is not to produce a rigid checklist answer.

The goal is to explain how multiple system design concepts interact inside one real product problem, and why certain choices are better than others under specific constraints.

## Recommended Structure

### 1. Problem Statement

Describe the product in plain language.

Explain:

- what is being built
- who uses it
- what the user is trying to achieve
- why the system becomes interesting at scale

This should make the rest of the document feel grounded in a real product rather than an abstract architecture exercise.

### 2. Scope and Assumptions

Define the boundaries before designing.

Clarify:

- what is in scope
- what is out of scope
- simplifying assumptions
- whether the design is for MVP, large-scale production, or global scale

This section prevents the rest of the case study from becoming vague or overbuilt.

### 3. Functional Requirements

List the key product behaviors the system must support.

Examples:

- shorten URL
- upload file
- send chat message
- start live stream
- place stock order

Keep these concrete and product-facing.

### 4. Non-Functional Requirements

List the qualities that matter for the system.

Examples:

- low latency
- high availability
- strong consistency
- durability
- fault tolerance
- ordering
- cost efficiency
- security and abuse prevention

This section usually drives the architecture more than the functional list does.

### 5. Capacity and Scale Estimation

Estimate the rough scale of the system.

Include where relevant:

- daily active users
- read QPS
- write QPS
- storage growth
- bandwidth
- cache size
- peak vs average load

The goal is not perfect arithmetic. The goal is to make later architecture choices feel justified.

### 6. Core Data Model

Define the important entities and access patterns.

Examples:

- users
- URLs
- files
- messages
- orders
- rides

Explain:

- what is the source of truth
- what IDs and keys matter
- what the dominant reads and writes are

### 7. APIs or External Interfaces

Describe the key system interfaces.

Examples:

- create short URL
- resolve short URL
- upload chunk
- fetch messages
- place order

This connects the requirements to concrete system interactions.

### 8. High-Level Design

Present the main components and how they interact.

This should usually cover:

- clients
- API layer
- application services
- cache
- databases
- object storage
- queues or streams
- background workers

Use diagrams whenever they make the architecture easier to reason about.

### 9. Request Flows

Walk through the major paths.

Common flows:

- write path
- read path
- async processing path
- retry or failure path

This section often makes the design much clearer than a component list alone.

### 10. Deep Dive Areas

This is where the case study becomes valuable.

Pick the parts that actually make the system hard.

Examples:

- schema design
- shard key strategy
- cache design
- consistency model
- ranking or matching algorithm
- message ordering
- deduplication
- replication strategy
- file chunking

Do not try to deep-dive everything. Pick the areas where architectural choices really matter.

### 11. Bottlenecks and Failure Modes

Discuss where the design can break.

Examples:

- hot keys
- hot partitions
- cache stampede
- queue buildup
- duplicate processing
- region failure
- stale reads
- metadata bottlenecks

A strong case study must explain where the design is weak, not only where it looks elegant.

### 12. Scaling Strategy

Explain how the system evolves as usage and product complexity grow.

Examples:

- start with one DB, then shard
- add cache
- add read replicas
- add async workers
- move to event-driven pipelines
- adopt multi-region routing

This section is useful because real systems are usually staged, not built in their final form on day one.

### 13. Tradeoffs and Alternatives

Explain what was chosen and why.

Examples:

- SQL vs NoSQL
- synchronous vs asynchronous flow
- denormalized reads vs transactional simplicity
- push vs pull
- strong consistency vs latency

This section is often where readers learn the most.

### 14. Real-World Considerations

Discuss production realities around the design.

Examples:

- abuse prevention
- observability
- compliance
- security
- cost control
- data retention
- migrations
- operational tooling

### 15. Summary

End with:

- the final recommended direction
- the key design tradeoff
- why the design fits the system

## Suggested Minimal Skeleton

```md
# <System Name>

## 1. Problem Statement

## 2. Scope and Assumptions

## 3. Functional Requirements

## 4. Non-Functional Requirements

## 5. Capacity and Scale Estimation

## 6. Core Data Model

## 7. APIs or External Interfaces

## 8. High-Level Design

## 9. Request Flows

## 10. Deep Dive Areas

## 11. Bottlenecks and Failure Modes

## 12. Scaling Strategy

## 13. Tradeoffs and Alternatives

## 14. Real-World Considerations

## 15. Summary
```

## Writing Rules

- Start from product behavior before naming infrastructure.
- Keep assumptions explicit.
- Use scale estimates to justify decisions rather than as decoration.
- Use diagrams for architecture and important flows.
- Pick a few deep-dive areas and explain them thoroughly.
- Make bottlenecks and failure modes explicit.
- Show how the design evolves with traffic and product growth.
- Avoid pretending there is only one correct architecture.

## Practical Advice

If a case study can only be understood by looking at the component diagram, it is too shallow.

If it can only be understood by reading 20 pages of prose without architecture structure, it is too loose.

The best case studies sit in the middle:

- clear product framing
- grounded scale
- explicit tradeoffs
- realistic failure thinking
- architecture that feels buildable
