---
slug: "goal-decomposition"
title: "Goal Decomposition"
type: "pattern"
category: "orchestration"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/goal-decomposition"
---
# Goal Decomposition

> Evidence: industry_observation · Confidence: high · Source: industry_observation, paper

Canonical: https://santismm.com/en/patterns/goal-decomposition

Goal decomposition has an agent break a high-level goal into an ordered set of smaller, tractable sub-tasks — a plan — before acting, then execute and monitor that plan, re-planning when steps fail. The explicit plan becomes an inspectable artifact you can review, gate, and debug. Use it when a goal needs several dependent steps and reactive, step-at-a-time agents drift or stall; skip it for simple, single-shot tasks.

## Problem

A single LLM call handed a broad, multi-step goal tends to improvise. Reactive agents that choose one action at a time can lose the thread on long horizons: they repeat work, skip prerequisites, or chase a dead end without realizing the overall objective is now unreachable. Because no plan exists as an artifact, you cannot review intended steps before they run, cannot tell whether a failure came from a bad strategy or a bad execution, and cannot easily resume after an interruption. The agent's reasoning is implicit, transient, and hard to audit.

## When to use it

This pattern fits goals that decompose into multiple interdependent steps with a meaningful ordering — research-then-synthesize, migrate-then-verify, gather-then-reconcile-then-report. It assumes the model can produce a reasonable plan from the goal and available tools, and that steps are observable enough to detect failure. It is most valuable where steps are costly, side-effecting, or hard to undo, so reviewing the plan before execution pays off. It is a poor fit when the next action is obvious from the current state, or when the environment changes so fast that any upfront plan is stale before the second step.

## Solution

Split the agent into a planning phase and an execution phase. The planner reads the goal, the available tools, and the current state, and emits an explicit, ordered plan: a list (or graph) of sub-tasks with their dependencies and expected outputs. Treating the plan as a first-class artifact is the core idea — it can be logged, shown to a human for approval, scored against policy, and diffed across runs. Encode dependencies explicitly so independent sub-tasks can run in parallel and dependent ones wait for their inputs, rather than forcing a brittle linear sequence the model invented.

An executor then runs the plan step by step, feeding each step's result forward and checking it against the step's expected output. When a step fails, returns something unusable, or invalidates a downstream assumption, hand control back to the planner to re-plan from the current state instead of blindly continuing — this closed loop is what separates robust decomposition from one-shot planning. Keep plans as shallow as the goal allows: prefer a few well-chosen steps over a deep tree, gate re-planning with a budget so the agent cannot loop forever, and let trivial goals bypass planning entirely.

## Benefits

- Long-horizon goals stay coherent because intended steps are decided up front, not improvised one at a time.
- The explicit plan is inspectable: it can be reviewed, approved, audited, and diffed before any side effect runs.
- Failures are easier to localize — a bad plan is distinguishable from a bad step execution.
- Independent sub-tasks expose parallelism and let work resume from the last completed step after an interruption.

## Risks

- A flawed initial decomposition propagates: every downstream step inherits a wrong assumption or missing prerequisite.
- Over-planning adds latency and cost on simple goals that a reactive agent would finish in one step.
- Plans go stale in fast-changing environments, so executing a step still based on an outdated world state.
- Unbounded re-planning loops where the agent repeatedly rewrites the plan without making real progress.

## KPIs

- Goal completion rate: Share of goals fully achieved end-to-end; good looks like decomposition beating a reactive baseline on the same multi-step tasks.
- Re-plan frequency: How often a run triggers re-planning; a healthy band means the loop catches real failures without thrashing on every step.
- Steps per goal vs. minimum: Plan length relative to a sensible minimum; watch for over-planning that inflates steps on simple goals.
- Plan-approval pass rate: Fraction of plans accepted by reviewers or policy checks before execution; low rates signal systematically weak decomposition.

## Observed failure modes

- Bad decomposition propagates: a wrong early assumption corrupts every dependent step downstream.
- Re-planning loop: the agent rewrites the plan repeatedly without converging or making progress.
- Stale plan execution: a step runs against a world state that changed since the plan was made.
- Over-decomposition: a trivial goal is split into needless steps, adding latency, cost, and failure surface.

## Lessons learned

- Make the plan a real artifact — log it, show it, diff it — so failures are debuggable rather than mysterious.
- Always close the loop: detect step failure and re-plan from current state instead of continuing blindly.
- Bound both plan depth and re-planning with explicit budgets to prevent shallow goals from spiraling.
- Let trivial goals skip the planner; reserve decomposition for genuinely multi-step, dependent work.

## FAQs

**How is this different from a reactive ReAct-style agent?**

A reactive agent decides one action at a time from the current state, with no plan as an artifact. Goal decomposition commits to an ordered plan up front, making intended steps inspectable and dependency ordering explicit. In practice the two are often combined: plan first, then execute reactively within each step and re-plan when a step fails.

**What happens when a step fails mid-plan?**

Hand control back to the planner to re-plan from the current state rather than continuing blindly. The closed re-planning loop is what makes decomposition robust. Bound it with a budget so a persistently failing step cannot trigger endless rewrites without progress.

**When does planning hurt more than it helps?**

On simple, single-step goals where the next action is obvious, or in environments that change faster than a plan stays valid. There, upfront planning adds latency and a stale-plan risk. Detect trivial goals and let them bypass the planner, reserving decomposition for genuinely multi-step, dependent work.
