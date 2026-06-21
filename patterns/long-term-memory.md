---
slug: "long-term-memory"
title: "Long-Term Memory"
type: "pattern"
category: "retrieval"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/long-term-memory"
---
# Long-Term Memory

> Evidence: industry_observation · Confidence: high · Source: industry_observation, paper

Canonical: https://santismm.com/en/patterns/long-term-memory

Give an agent persistent memory across sessions so it remembers facts, user preferences, and prior outcomes beyond a single context window. A write path decides what to store, summarizes it, and deduplicates it; a read path retrieves only the relevant memories into context when needed. Unlike semantic caching, which caches whole answers to skip recomputation, long-term memory stores durable facts and state and recomposes them into fresh reasoning each time.

## Problem

The context window is finite and resets between sessions. An agent that only sees the current conversation forgets a user's stated preferences, decisions made last week, and the outcome of prior tasks. Stuffing all history into every prompt is impossible past a certain scale and degrades reasoning as the window fills with low-value tokens. Teams need a way to persist the small set of facts that matter and surface them precisely when they are relevant.

## When to use it

Use this when an agent serves the same users or works on the same long-running tasks repeatedly: assistants that learn preferences, support agents that track a customer's history, coding agents that remember project conventions, or multi-step workflows spanning days. It assumes you can store data outside the model (a vector store, database, or memory framework) and that you control both when memories are written and how they are retrieved into the prompt.

## Solution

Separate the write path from the read path. On the write path, after a turn or task completes, an extraction step decides what is worth remembering: stable facts, preferences, commitments, and outcomes — not transient chatter. Candidate memories are summarized into compact, self-contained statements, checked against existing memories to deduplicate and to detect contradictions, then written to a store with metadata: a memory type, a timestamp, a source, and the user or scope it belongs to. Writing less but writing well is the goal; noisy memories poison later retrieval.

On the read path, before the agent reasons, you retrieve candidate memories relevant to the current task — typically by semantic similarity plus filters on scope and recency — rank them, and inject only the top few into context. Treat retrieval as a precision problem: a handful of correct memories beats a large, loosely related set. Distinguish memory types so retrieval can be targeted: episodic (what happened), semantic (durable facts and preferences), and procedural (how to do a recurring task). Periodically consolidate and expire memories so the store stays small, current, and free of contradictions.

## Benefits

- The agent recalls preferences, decisions, and outcomes from prior sessions, so users do not have to repeat context and the agent behaves consistently over time.
- Retrieving a few relevant memories keeps the window focused on high-value tokens instead of dumping full history, which preserves reasoning quality and reduces cost.
- As stable facts and preferences accumulate, the agent tailors responses more accurately with each interaction without retraining the model.
- Because memories live in an external store with metadata, you can inspect, correct, export, and delete what the agent knows — important for trust and compliance.

## Risks

- Without consolidation and expiry, the store accumulates outdated facts and conflicting statements, and the agent confidently acts on the wrong one.
- Persisting user data raises retention, consent, and access-control obligations; memories can leak sensitive information across sessions or users if scope is not enforced.
- Low precision injects irrelevant or wrong memories that mislead reasoning; low recall silently drops the memory that mattered, making failures hard to diagnose.
- Over-eager writing inflates the store, slows retrieval, raises storage and embedding costs, and dilutes the signal that good retrieval depends on.

## KPIs

- Retrieval precision of injected memories: Of the memories placed in context, the share that were actually relevant. This is the metric that most directly governs answer quality; good looks like the injected set being almost entirely on-topic, with irrelevant memories rare.
- Retrieval recall on memory-dependent tasks: On tasks that require a known stored fact, how often that fact is actually retrieved. Good looks like the right memory surfacing reliably; persistent misses point to extraction or indexing gaps.
- Memory store size and growth rate: Total memories and how fast they accumulate per active user. Good looks like growth tracking genuinely new durable facts, not unbounded climb — a runaway curve signals over-eager writing.
- Staleness and contradiction rate: Share of retrieved memories that are outdated or conflict with a newer truth. Good looks like a low and stable rate, evidence that consolidation and expiry are keeping pace with change.

## Observed failure modes

- Writing everything turns the store into noise; retrieval then surfaces low-value or wrong memories. Fix by raising the bar for what gets written and reviewing extraction quality.
- An old fact is retrieved and acted on after the truth changed, with no signal that it is outdated. Mitigate with timestamps, recency-weighted ranking, and explicit supersession on write.
- A memory from one user, tenant, or project is retrieved into another's context because scope filters were missing or wrong — a privacy and correctness failure at once.
- To compensate for poor ranking, teams inject many memories, refilling the window with marginal tokens and degrading the very reasoning memory was meant to support.

## Lessons learned

- Quality is decided when you choose what to remember. A small, clean, deduplicated store retrieves far better than a large noisy one.
- A few correct memories outperform many loosely related ones. Tune for relevance and rank tightly rather than maximizing how much you inject.
- Store metadata and provide ways to view, edit, expire, and delete memories. This is essential for debugging, trust, and meeting privacy obligations.
- Facts go stale and contradict each other. Build consolidation, supersession, and expiry early; retrofitting them onto a large polluted store is painful.

## FAQs

**How is this different from semantic caching?**

Semantic caching stores and replays whole answers to avoid recomputing similar requests. Long-term memory stores durable facts, preferences, and outcomes, then recomposes them into fresh reasoning for each new task. One reuses outputs; the other remembers state.

**What should the agent actually remember?**

Stable, reusable signal: user preferences, decisions and commitments, outcomes of prior tasks, and recurring procedures. Avoid transient chatter and anything you cannot justify retaining. Writing less but writing well is what makes later retrieval precise.

**How do you handle PII and privacy?**

Treat the store as governed data: enforce scope so memories never cross users or tenants, minimize what you persist, support consent and deletion, and set retention and access controls. Inspectability and an expiry policy are part of meeting these obligations.
