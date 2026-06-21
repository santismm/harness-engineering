---
slug: "evaluator-optimizer"
title: "Evaluator-Optimizer"
type: "pattern"
category: "reliability"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/evaluator-optimizer"
---
# Evaluator-Optimizer

> Evidence: industry_observation · Confidence: high · Source: industry_observation, paper

Canonical: https://santismm.com/en/patterns/evaluator-optimizer

One LLM generates a response while a second LLM evaluates it against criteria and returns feedback; the generator revises and the loop repeats until the evaluation passes. It raises quality on tasks with clear evaluation criteria, at the cost of extra calls.

## Problem

A single-pass output may miss requirements, and there is no built-in mechanism to check and improve it before it is used.

## When to use it

Use evaluator-optimizer when you can articulate clear evaluation criteria and iterative refinement measurably improves the result — for example translation quality, code that must pass tests, or writing against a rubric.

## Solution

A generator produces a candidate; an evaluator (a separate LLM call or a deterministic check) scores it against explicit criteria and returns actionable feedback. The generator revises, and the cycle repeats until criteria are met or a budget is reached.

Separating generation from evaluation mirrors how a human writer benefits from an editor: the critic catches issues the author misses, and explicit criteria keep the loop converging.

## Benefits

- Higher quality on tasks with clear criteria.
- Catches errors a single pass would ship.
- Feedback is explicit and actionable.

## Risks

- Extra calls add latency and cost.
- A weak evaluator gives misleading feedback.
- Loops can fail to converge without a budget.

## KPIs

- Acceptance rate: Share of candidate outputs the evaluator accepts on first pass — too high means the bar is too low, too low means the generator or rubric is off.
- Iterations to accept: Average evaluate→revise loops before acceptance; rising counts flag a weak generator or vague criteria.
- Cost & latency per accepted output: Total tokens and wall-clock across all loop iterations, not just the final call — the loop multiplies both.
- Eval–human agreement: How often the evaluator's verdict matches a human reviewer on a sampled set; the loop is only as good as the evaluator.

## Observed failure modes

- Reward hacking: the generator learns to satisfy the evaluator's wording rather than the real goal.
- Weak or miscalibrated evaluator: it accepts bad outputs or rejects good ones, so the loop adds cost without quality.
- Infinite or oscillating loops when no candidate ever clears the bar — without an iteration cap the cost is unbounded.
- Criteria drift: vague or shifting rubrics make acceptance non-deterministic and hard to audit.

## Lessons learned

- Cap iterations and define a fallback (return best-so-far, or escalate) so the loop always terminates.
- Make acceptance criteria explicit and stable; an evaluator is only as good as its rubric.
- Validate the evaluator against human judgement before trusting it as a gate.
- Use the loop only where quality justifies the multiplied cost — not for cheap, low-stakes outputs.

## FAQs

**How is this different from reflection?**

Reflection has the same model self-critique. Evaluator-optimizer separates the roles: a distinct evaluator judges the generator, which often gives sharper, less biased feedback.

**Can the evaluator be deterministic?**

Yes. For code, a test runner is an ideal evaluator; for structured output, a schema check works. Use a model judge for nuanced criteria.

**How many iterations?**

Set a budget (e.g. 2–3) and stop when criteria pass. Unbounded loops waste cost and may not converge.
