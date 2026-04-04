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

## How To Use This Repository

- Start with `Basics` if you want a foundation-first path.

## Roadmap

Planned topics for the `Basics` section include:

- replication
- partitioning / sharding
- consensus
- quorum
- caching
- load balancing
- message queues
- rate limiting
- idempotency
