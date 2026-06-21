---
slug: "semantic-caching"
title: "Semantic Caching"
type: "pattern"
category: "cost"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/semantic-caching"
---
# Semantic Caching

> Evidence: industry_observation · Confidence: medium · Source: industry_observation

Canonical: https://santismm.com/en/patterns/semantic-caching

Semantic caching stores past model responses and reuses them when a new request is semantically similar to a previous one — matching by meaning via embeddings, not exact text. It cuts cost and latency for repetitive or near-duplicate queries common in production.

## Problem

Many production queries are paraphrases of ones already answered, so re-running the full model on each wastes cost and latency.

## When to use it

Use semantic caching when traffic contains many similar or repeated questions and answers are stable enough to reuse — FAQs, support, documentation assistants.

## Solution

Embed each incoming request and search a cache of prior request embeddings. If a sufficiently similar entry exists (above a similarity threshold), return its stored response; otherwise call the model and store the new pair.

Tune the similarity threshold carefully: too loose returns wrong answers for subtly different questions; too strict misses valid hits. Add TTLs and invalidation so cached answers do not go stale.

## Benefits

- Lower cost by avoiding repeat model calls.
- Lower latency on cache hits.
- More consistent answers to similar questions.

## Risks

- A loose threshold serves wrong cached answers.
- Stale cache without TTL or invalidation.
- Personalized or time-sensitive answers cache poorly.

## KPIs

- Cache hit rate: Share of requests served from cache; the lever for both cost and latency savings.
- False-hit rate: How often a semantically 'similar' hit returns a wrong or stale answer — the central risk of caching by meaning.
- Cost & latency saved per hit: Tokens and time avoided on cache hits, the upside you're trading the false-hit risk for.
- Similarity threshold calibration: Whether the match threshold balances hit rate against false hits; too loose hurts quality, too strict kills savings.

## Observed failure modes

- False hits: two queries are similar in embedding space but need different answers, so the cache returns a wrong one.
- Staleness: cached answers go out of date while the underlying facts change.
- Threshold mis-tuning: too loose returns wrong answers, too strict yields almost no hits.
- Cache poisoning: a bad answer gets cached and then served repeatedly.

## Lessons learned

- Tune the similarity threshold against real traffic; it is the make-or-break parameter.
- Never cache where freshness or correctness is critical without an invalidation strategy.
- Validate or sample cache hits to catch false matches before users do.
- Scope caches narrowly (per tenant, per context) to avoid leaking the wrong answer across users.

## FAQs

**How is this different from a normal cache?**

A normal cache matches exact keys; a semantic cache matches by meaning using embeddings, so paraphrased questions still hit.

**What is the main risk?**

A too-loose similarity threshold returns a cached answer for a question that is actually different. Tune the threshold and validate on real traffic.

**How do I avoid stale answers?**

Set TTLs and invalidate entries when the underlying data changes; avoid caching personalized or time-sensitive responses.
