---
slug: "context-compression"
title: "Context Compression"
type: "pattern"
category: "cost"
evidence_level: "production"
source: "https://santismm.com/en/patterns/context-compression"
---
# Context Compression

> Evidence: production · Confidence: low · Source: production_system, personal_experience, industry_observation

Canonical: https://santismm.com/en/patterns/context-compression

Context compression reduces the tokens fed to a model on each call while preserving the information it actually needs to act. Use it on long-running agents and long conversations to cut cost and latency and to stay inside the context window. The three levers are summarizing history, pruning irrelevant context, and compressing prompts. The central risk is lossy: dropping the one detail that mattered. Measure information retained, not just tokens saved.

## Problem

Long-running agents and multi-turn conversations accumulate context: every tool result, prior message, and retrieved document is replayed on the next call. Token count grows roughly linearly with the interaction, so per-call cost and latency climb, and eventually the window overflows and the oldest (sometimes most important) content is silently truncated. Naive fixes — bigger windows, more aggressive truncation — either raise cost or destroy the information the model needs to stay coherent.

## When to use it

Applies when context grows unbounded relative to what any single step needs: conversational assistants with long histories, autonomous agents looping over many tool calls, RAG pipelines that over-retrieve, and batch jobs where prompt size dominates cost. It fits when much of the accumulated context is redundant or stale, when you control prompt assembly, and when you can tolerate some reconstruction error. It is a poor fit when every token is load-bearing (legal, audit, exact-recall tasks) or when interactions are short enough that the window is never pressured.

## Solution

Treat the live context as a budget you actively manage rather than an append-only log. Three complementary levers exist. Summarization replaces a span of history with a shorter synopsis — typically a rolling summary of older turns, refreshed periodically, while recent turns stay verbatim. Pruning removes context that is irrelevant to the current step: deduplicate, drop stale tool output, and select only the retrieved chunks that score above a relevance threshold. Prompt compression (for example LLMLingua) uses a smaller model to delete or rephrase low-information tokens before sending the prompt, trading a small accuracy cost for large token reductions.

Compose these into a pipeline with explicit boundaries: keep a verbatim recent window, a rolling summary of older history, and a retrieval slot filled on demand. Protect a 'pinned' region for facts that must never be compressed — identifiers, constraints, the current goal. Crucially, instrument the result: run an evaluation set comparing answers with and without compression so you can see when quality degrades, and tune the aggressiveness per workload rather than globally. Compression is a quality-versus-cost dial, not a free win.

## Benefits

- Sending fewer tokens directly reduces input cost on every call, which compounds across long agent loops and high-volume traffic.
- Smaller prompts mean less to encode and shorter time-to-first-token, improving responsiveness in interactive and agentic flows.
- Bounding live context lets long conversations and many-step agents continue without overflowing the window or silently truncating.
- Removing redundant and stale context can improve quality by reducing distraction, helping the model attend to what currently matters.

## Risks

- Summaries and pruning can discard the single detail that later turns out to be decisive, producing confidently wrong answers.
- Rolling summaries summarize prior summaries; small omissions compound over many cycles until the thread quietly drifts.
- Running a summarizer or compressor adds its own latency, cost, and failure surface, which can offset savings on short interactions.
- Aggressive eviction can silently remove constraints or instructions the model still depends on, with no obvious error signal.

## KPIs

- Tokens per call (input): The primary cost driver. Track the distribution before and after compression; a healthy result is a clear reduction with no rise in downstream errors.
- Information retention / task quality: Compare answers with and without compression on an eval set. Good looks like quality holding steady within your tolerance as tokens drop.
- End-to-end latency: Net of compression overhead. Good is lower total latency; watch that summarizer or compressor calls do not erase the savings.
- Context-overflow / truncation rate: How often interactions hit the window limit. Good is driving this toward zero without resorting to dropping pinned content.

## Observed failure modes

- A summary omits a constraint mentioned early; many turns later the agent violates it because that fact is simply gone from context.
- Repeated re-summarization amplifies paraphrase errors and omissions until the running summary no longer reflects what actually happened.
- A misconfigured budget compresses identifiers or instructions that were meant to be protected, breaking correctness silently.
- An aggressive relevance threshold filters out context that mattered for an edge case, so quality looks fine in tests but fails in the field.

## Lessons learned

- Token reduction is trivial to maximize and meaningless alone; the real metric is whether the model still answers correctly.
- Explicitly protect identifiers, constraints, and the current goal so no compression stage can evict them.
- Compress old history, not the active context; the most recent exchanges carry the most decision-relevant signal.
- Tune aggressiveness per workload against an eval set; what is safe for chit-chat is reckless for an audit task.

## FAQs

**How is this different from long-term memory?**

Long-term memory persists facts outside the prompt and retrieves them on demand; context compression shrinks the live context sent on each call. They are complementary: memory decides what to bring back, compression decides how compactly it sits in the window.

**Summarize, prune, or compress — which should I use?**

Prune first (free, lossless when removing true redundancy), summarize older history when it grows unbounded, and add prompt compression only when you still need more headroom and can validate the quality cost. Most systems combine all three.

**How do I know compression is hurting quality?**

Run an evaluation set with compression on and off and compare task outcomes, not just token counts. Watch for confidently wrong answers and dropped constraints — those are the signature of lossy compression that has gone too far.
