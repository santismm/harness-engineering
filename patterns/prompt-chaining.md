---
slug: "prompt-chaining"
title: "Prompt Chaining"
type: "pattern"
category: "orchestration"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/prompt-chaining"
---
# Prompt Chaining

> Evidence: industry_observation · Confidence: high · Source: industry_observation, paper

Canonical: https://santismm.com/en/patterns/prompt-chaining

Prompt chaining decomposes a task into a fixed sequence of LLM calls, where each step works on the output of the previous one. It trades a little latency for much higher accuracy and control, and is the simplest workflow pattern: use it whenever a task cleanly splits into ordered subtasks.

## Problem

A single prompt asked to do several things at once produces lower-quality, harder-to-control output, and is difficult to debug when it goes wrong.

## When to use it

Use prompt chaining when a task decomposes into a clear, ordered sequence of subtasks — for example outline, then draft, then edit — and each step benefits from the previous step's result.

## Solution

Break the task into discrete steps and run one LLM call per step, passing each output to the next. Optionally add programmatic checks (gates) between steps to validate intermediate results before continuing.

Because each call has one focused job, prompts are simpler, outputs are more reliable, and failures are localized to a specific step that you can inspect and fix.

## Benefits

- Higher accuracy by giving each call one focused job.
- Easier to debug — failures localize to a step.
- Validation gates can catch errors between steps.

## Risks

- Higher total latency from sequential calls.
- Errors can compound down the chain if not checked.
- Too many steps add cost and brittleness.

## KPIs

- End-to-end success rate: Share of chains that produce a correct final result; errors compound across steps.
- Per-step error rate: Failure rate at each link — a 95%-reliable step chained five times yields ~77% end to end.
- Total latency & cost: Sum across every call in the chain; more steps mean more of both.
- Recovery rate: How often a failed intermediate step is caught and corrected rather than silently propagated.

## Observed failure modes

- Error propagation: a mistake early in the chain corrupts every downstream step.
- Latency and cost accumulation as the chain grows longer.
- Brittle hand-offs when one step's output format doesn't match the next step's expected input.
- Lost context across steps, so later links forget constraints set earlier.

## Lessons learned

- Validate or gate-check between steps so errors are caught before they propagate.
- Keep chains as short as the task allows; every extra step multiplies failure probability.
- Pin the output contract of each step so hand-offs don't break silently.
- Use chaining for genuinely sequential work; parallelize independent steps instead.

## FAQs

**How is prompt chaining different from an agent?**

Prompt chaining follows a fixed, predefined sequence. An agent decides its own steps dynamically. Prefer chaining when the path is known in advance.

**When should I add gates between steps?**

Whenever an intermediate result must meet a condition before proceeding — it stops errors from propagating down the chain.

**Does chaining increase cost?**

Yes, modestly — more calls mean more tokens and latency — but the gain in reliability usually outweighs it for multi-part tasks.
