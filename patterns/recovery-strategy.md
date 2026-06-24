---
slug: "recovery-strategy"
title: "Recovery Strategy"
type: "pattern"
category: "reliability"
evidence_level: "production"
source: "https://santismm.com/en/patterns/recovery-strategy"
---
# Recovery Strategy

> Evidence: production · Confidence: low · Source: production_system, personal_experience, industry_observation

Canonical: https://santismm.com/en/patterns/recovery-strategy

Give the agent an explicit plan for when things break. Detect failures by validating outputs and catching tool errors; then retry with adjustment, fall back to an alternative path, roll back partial actions, or escalate. Bound retries to avoid runaway loops and cost, make actions idempotent, and distinguish transient from permanent failures. The goal is graceful degradation instead of crashes or silently wrong results.

## Problem

Agents fail constantly: tools time out, APIs return errors, models emit malformed output, plans hit dead-ends, and multi-step workflows leave partial side effects behind. Without an explicit recovery path, an agent either crashes on the first error or — worse — plows ahead on bad data and silently produces confidently wrong results. Naive retry loops make it worse, hammering a failing dependency, burning tokens, and spinning forever. The hard part is not catching one error; it is deciding what kind of failure it is and what response is safe.

## When to use it

Use this pattern in any agent that calls external tools, runs multi-step plans, or takes consequential actions where partial completion is possible. It matters most for long-running or autonomous workflows that no human watches in real time, and for actions with side effects (payments, writes, emails) where a blind retry could duplicate work. It assumes you can validate outputs against some contract and that at least some operations can be made idempotent or compensated. It is less relevant for single-shot, read-only, low-stakes prompts.

## Solution

Treat recovery as a first-class control loop layered around the agent's normal execution. Every tool call and model output passes through a validation gate: catch exceptions and timeouts, and check outputs against a schema or contract before trusting them. On failure, classify it. Transient failures (timeouts, rate limits, 5xx) get a bounded retry with exponential backoff and jitter, ideally against an idempotent operation so a duplicate request is harmless. Permanent failures (invalid arguments, auth errors, contract violations) skip retries and move straight to an alternative: a different tool, a simpler plan, or a fallback answer.

When progress matters, checkpoint state so the agent can resume from the last good step rather than restarting. When a step has already produced side effects and cannot proceed, run compensating actions to roll back — cancel the order, delete the draft, reverse the charge. Wrap the whole loop in hard budgets: maximum attempts, maximum wall-clock time, and a cost ceiling, plus a circuit breaker that stops calling a dependency that keeps failing. When all recovery options are exhausted, escalate cleanly — surface the failure to a human or a supervising agent with enough context to act, rather than guessing.

## Benefits

- The agent produces a partial or fallback result and a clear status instead of crashing or returning confident garbage.
- Hard attempt, time, and cost limits stop runaway retry loops from burning budget on a failing dependency.
- Compensating actions and checkpoints keep external systems and task state coherent when a workflow stops midway.
- When recovery fails, the agent hands off with enough context for a human or supervisor to act, rather than guessing.

## Risks

- Aggressive retries against a struggling dependency add load and can turn a brief blip into a cascading outage.
- Retrying a non-idempotent action can double-charge, double-send, or double-write if request keys are not used.
- Over-eager fallbacks can hide systematic failures, so a broken tool looks healthy while quietly degrading every result.
- Rollback logic is often incomplete or itself fails, leaving systems in an inconsistent state that is hard to detect.

## KPIs

- Recovery success rate: Share of failures resolved automatically by retry or fallback without human help; a healthy value is high and stable, with no quiet downward drift.
- Mean attempts per successful task: How many tries it takes to succeed; watch for creep, which signals a degrading dependency rather than genuine recovery.
- Unbounded-loop / budget-breach rate: How often runs hit retry, time, or cost ceilings; this should be rare, and spikes mean limits or classification need tuning.
- Compensation completeness: Fraction of failed multi-step workflows that end in a consistent state; the target is full rollback with no orphaned side effects.

## Observed failure modes

- Missing or too-high attempt caps let the agent retry a permanent failure forever, burning cost and never making progress.
- Treating a permanent error as transient wastes retries; treating a transient error as permanent gives up too early and triggers needless fallbacks.
- A fallback path returns a plausible but wrong answer with no signal that recovery occurred, so downstream consumers trust bad output.
- The agent fails after a write or external action but before compensation runs, leaving duplicate or dangling records.

## Lessons learned

- The retry/fallback/escalate decision hinges on transient versus permanent; invest in clear classification before tuning backoff curves.
- Idempotency keys turn a risky retry into a safe one; design for it up front rather than bolting on dedup later.
- Hard caps on attempts, time, and cost are non-negotiable; an agent without them will eventually find a way to run forever.
- Log every retry, fallback, and compensation so silent degradation surfaces as a metric instead of a surprise incident.

## FAQs

**How is this different from just adding try/except and a retry loop?**

Try/except handles one error; a recovery strategy decides what kind of failure it is and chooses among retry, fallback, rollback, and escalation under hard budgets. The control logic and idempotency, not the exception handling, are the substance.

**How many retries should I allow?**

Few — typically a small fixed cap with exponential backoff and jitter, plus separate time and cost ceilings. The exact number depends on the dependency, but the loop must always terminate, and permanent failures should not be retried at all.

**What if an action cannot be undone or made idempotent?**

Do not auto-retry across it. Checkpoint before the irreversible step, and on failure escalate to a human or supervising agent rather than guessing. Irreversibility is a signal to slow down, not to retry harder.
