---
slug: "reflection"
title: "Reflection"
type: "pattern"
category: "reliability"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/reflection"
---
# Reflection

> Evidence: industry_observation · Confidence: high · Source: industry_observation, paper

Canonical: https://santismm.com/en/patterns/reflection

Reflection has a model critique its own output and then revise it, using the critique as feedback. It is a lightweight, single-model way to catch mistakes and improve quality on reasoning, coding and writing tasks — at the cost of extra calls.

## Problem

Models often produce a flawed first answer they could improve if prompted to review their own work, but a single pass gives them no chance to.

## When to use it

Use reflection when a self-review step measurably improves output and you want a simpler alternative to a two-model evaluator loop — common in reasoning and coding tasks.

## Solution

After generating an answer, prompt the same model to critique it against the goal (and any tool feedback such as test results or errors), then to produce a revised answer informed by that critique. Repeat for a bounded number of iterations.

Reflection works best when grounded in real signals — execution errors, test output, retrieved facts — rather than pure self-assessment, which can be overconfident.

## Benefits

- Improves quality with a single model — no second system.
- Effective when grounded in tool or test feedback.
- Simple to add to an existing call.

## Risks

- Self-critique can be overconfident or miss its own errors.
- Extra calls add latency and cost.
- Without grounding, gains are limited.

## KPIs

- Quality lift from reflection: Measured improvement in output quality with the reflection step versus without; if it's not measurable, the step isn't earning its cost.
- Self-correction rate: Share of genuine errors the model catches and fixes on review — distinct from cosmetic edits.
- Added latency & cost: Reflection at least doubles calls; track the overhead against the quality it buys.
- Over-revision rate: How often reflection degrades an already-good answer by second-guessing it.

## Observed failure modes

- Self-evaluation blind spots: a model often can't see its own errors, so reflection misses them.
- Over-revision: the model 'fixes' a correct answer into a worse one.
- Cost and latency double (or more) for marginal or no quality gain.
- False confidence: the model asserts the output is now correct when it isn't.

## Lessons learned

- Measure the lift; reflection is worth it only where it demonstrably improves quality.
- Prefer external signals (tests, tools, a separate evaluator) over pure self-critique when stakes are high.
- Cap reflection to one or two passes — returns diminish fast and cost compounds.
- Give the reflection step concrete criteria, not a vague 'improve this'.

## FAQs

**Reflection or evaluator-optimizer?**

Reflection uses one model to self-critique (simpler); evaluator-optimizer uses a separate evaluator (sharper, less biased). Choose by how reliable self-assessment is for your task.

**Does reflection always help?**

It helps most when grounded in real feedback like test results or errors. Pure self-assessment can be overconfident and add little.

**How many reflection rounds?**

Keep it bounded — often one or two. Diminishing returns and rising cost make long loops rarely worth it.
