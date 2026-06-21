---
slug: "routing"
title: "Routing"
type: "pattern"
category: "orchestration"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/routing"
---
# Routing

> Evidence: industry_observation · Confidence: high · Source: industry_observation, paper

Canonical: https://santismm.com/en/patterns/routing

Routing classifies an input and directs it to the most appropriate specialized handler, prompt or model. It improves quality by letting each path be optimized for its case, and controls cost by sending easy requests to cheap models and hard ones to capable models.

## Problem

A single prompt or model handling every kind of input does each one worse, and using one expensive model for everything wastes money on easy requests.

## When to use it

Use routing when inputs fall into distinct categories that benefit from different handling — different prompts, tools, models or workflows — and the categories can be classified reliably.

## Solution

A lightweight classifier (an LLM call or a model) labels the input, then a router sends it to the matching downstream handler. Each handler is specialized and optimized for its category.

Routing also enables cost-performance tiering: route simple queries to a fast, cheap model and complex ones to a stronger reasoning model, paying for capability only when it is needed.

## Benefits

- Each path is optimized for its case, raising quality.
- Cost control by tiering models to difficulty.
- Separation of concerns keeps each handler simple.

## Risks

- Misclassification sends inputs down the wrong path.
- The classifier adds a step and some latency.
- Category drift over time degrades routing accuracy.

## KPIs

- Routing accuracy: Share of inputs sent to the correct handler/model; the single metric that defines the pattern's value.
- Cost savings vs. always-best-model: Money saved by routing easy inputs to cheaper models instead of the top one for everything.
- Misroute cost: The downstream damage of wrong routes — a misroute can cost far more than the savings it chased.
- Router latency overhead: Time the routing decision itself adds before any real work begins.

## Observed failure modes

- Misclassification: the router sends an input to the wrong model or path, degrading the answer.
- Ambiguous inputs that don't fit any route cleanly and get forced into a poor one.
- Router becomes a bottleneck or single point of failure for every request.
- Drift: input distribution shifts over time and the router's categories go stale.

## Lessons learned

- Optimize for the cost of a misroute, not just routing accuracy — some wrong routes are far costlier than others.
- Add a default / fallback route for inputs that match nothing well.
- Keep the router cheap and fast; if it costs as much as the work, it defeats the purpose.
- Monitor input drift and re-tune routes as the distribution changes.

## FAQs

**What classifies the input?**

Usually a lightweight LLM call or a dedicated classifier model; for clear-cut cases, deterministic rules can route without a model.

**How does routing save cost?**

By tiering: easy requests go to cheap, fast models and only hard ones reach expensive reasoning models, so you pay for capability only when needed.

**What if the classifier is wrong?**

Provide a sensible default route and monitor misroutes; a fallback handler and good observability limit the impact of misclassification.
