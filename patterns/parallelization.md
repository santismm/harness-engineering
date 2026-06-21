---
slug: "parallelization"
title: "Parallelization"
type: "pattern"
category: "orchestration"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/parallelization"
---
# Parallelization

> Evidence: industry_observation · Confidence: high · Source: industry_observation

Canonical: https://santismm.com/en/patterns/parallelization

Parallelization runs multiple LLM calls at the same time and aggregates the results. Two flavors: sectioning (split a task into independent subtasks run in parallel) and voting (run the same task several times to improve reliability or coverage). It cuts latency and can raise quality.

## Problem

Running independent subtasks one after another wastes time, and a single sample of a hard task can be unreliable.

## When to use it

Use parallelization when subtasks are independent (sectioning), or when multiple attempts at the same task improve confidence or coverage (voting).

## Solution

Sectioning: split the work into independent pieces, run them concurrently, and combine the outputs. Voting: run the same prompt multiple times (or with variations) and aggregate by majority, union or a judge.

Both reduce wall-clock time versus sequential execution; voting additionally trades extra cost for higher reliability on tasks where a single sample is risky.

## Benefits

- Lower latency by running calls concurrently.
- Voting improves reliability and coverage.
- Each parallel call stays simple and focused.

## Risks

- Voting multiplies token cost.
- Aggregation logic can be tricky to get right.
- Subtasks assumed independent may actually interact.

## KPIs

- Latency reduction vs. sequential: Wall-clock saved by running calls concurrently; the whole point of the pattern.
- Aggregation quality: Whether merging the parallel outputs preserves correctness — the hard part is the join, not the fan-out.
- Concurrency cost: Total tokens across all parallel branches; you trade money for speed, so watch the multiplier.
- Rate-limit / throttle rate: How often parallel calls hit provider rate limits, which silently serializes or fails them.

## Observed failure modes

- Aggregation errors: parallel results are correct individually but combined wrongly (double-counting, contradictions).
- Rate limiting turns intended parallelism back into slow, serialized calls.
- Cost surprise: N parallel branches cost N× even when only one result is used.
- Partial failure handling: one branch fails and the aggregator either blocks or silently drops it.

## Lessons learned

- Design the aggregation step first — combining results well is harder than splitting the work.
- Respect provider rate limits with batching or backoff, or parallelism evaporates.
- Only parallelize independent sub-tasks; dependencies force a sequence anyway.
- Decide explicitly how partial failures are handled before they happen in production.

## FAQs

**What is the difference between sectioning and voting?**

Sectioning splits one task into different independent subtasks; voting runs the same task multiple times to aggregate for reliability.

**Does voting always improve quality?**

Often, on tasks where samples vary, but it multiplies cost. Reserve it for high-stakes steps where a single sample is risky.

**How do I combine parallel results?**

Depending on the case: concatenate sections, take a majority vote, union the findings, or use a judge model to synthesize.
