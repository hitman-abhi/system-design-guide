# Concept Template

Use this structure for any topic that needs to be explained as an in-depth knowledge-base article.

The goal of the template is:

- start with intuition before detail
- define terms precisely
- explain tradeoffs, not just definitions
- connect theory to real systems
- end with practical guidance and durable takeaways

## Recommended Structure

### 1. Overview

Explain in 2-4 short paragraphs:

- what the concept is
- why it matters
- what central question it helps answer

Optional pattern:

> One sentence that captures the central idea or tension.

### 2. The Core Problem

Describe the underlying problem the concept exists to address.

Focus on:

- the engineering context
- the failure mode or scaling problem
- why simpler systems do not face the same issue

### 3. Formal Statement

State the concept in its precise form.

Include:

- the canonical definition
- any conditions or assumptions
- what the concept does and does not claim

### 4. Key Terms

Break down all important terms one by one.

For each term:

- define it precisely
- separate formal meaning from loose industry usage
- add a small example if needed

### 5. What It Really Means

Translate the formal definition into plain engineering language.

This section should answer:

- what practical tradeoff does this create
- how should a builder think about it
- what is commonly misunderstood

### 6. Why the Constraint Exists

Walk through a concrete scenario that shows why the rule or limitation is unavoidable.

Good format:

1. set up a small example
2. show the system choices
3. explain why each choice gives up something

### 7. Main Variants or Modes

If the concept has categories, branches, or operating modes, describe them here.

For each variant:

- what it optimizes for
- how it behaves
- where it fits
- what tradeoff it introduces

Examples:

- strong vs eventual consistency
- push vs pull
- synchronous vs asynchronous replication
- leader-based vs leaderless designs

### 8. Supporting Mechanisms and Related Ideas

Cover adjacent concepts that deepen understanding.

Examples:

- formulas
- algorithms
- protocols
- related models
- extensions and refinements

This section is where you connect the concept to the rest of the system design landscape.

### 9. Real-World Examples

Show how the concept appears in real systems.

For each example:

- where it shows up
- why that choice makes sense
- what tradeoff is being accepted

### 10. Common Misconceptions

List the incorrect simplifications people often repeat.

For each misconception:

- state the misconception clearly
- explain why it is incomplete or wrong
- replace it with a more accurate statement

### 11. Design Guidance

Turn the concept into decision-making guidance.

Useful prompts:

- when should this model be preferred
- when is it too expensive
- what business or product constraints affect the choice
- what failure modes matter most

### 12. Reusable Takeaways

Summarize the durable lessons in 4-8 bullets.

This should be the section a reader can skim later and still recover the core idea.

### 13. Summary

Close with a short synthesis:

- restate the main idea
- restate the most important tradeoff
- end with why it matters in system design

## Writing Rules

- Prefer precise terms over motivational language.
- Avoid audience-specific or role-specific labels.
- Use examples whenever a definition could otherwise feel abstract.
- Distinguish formal meaning from common shorthand.
- When a concept is commonly misunderstood, say so directly and correct it.
- Focus on behavior under failure, scale, or concurrency where relevant.
- Use lists for structure, but keep the prose explanatory.

## Optional Section Additions

Not every concept needs all of these, but they are useful when relevant:

- **History or Origin**: for foundational ideas or papers
- **Math or Formula**: for throughput, latency, hashing, probability, or quorum topics
- **Comparison With Similar Concepts**: when two ideas are often confused
- **API / Implementation View**: when the concept has practical coding implications
- **Operational Risks**: when production behavior matters as much as the theory

## Minimal Skeleton

```md
# <Concept Name>

## 1. Overview

## 2. The Core Problem

## 3. Formal Statement

## 4. Key Terms

## 5. What It Really Means

## 6. Why the Constraint Exists

## 7. Main Variants or Modes

## 8. Supporting Mechanisms and Related Ideas

## 9. Real-World Examples

## 10. Common Misconceptions

## 11. Design Guidance

## 12. Reusable Takeaways

## 13. Summary
```
