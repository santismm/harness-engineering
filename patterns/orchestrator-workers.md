---
slug: "orchestrator-workers"
title: "Orchestrator-Workers"
type: "pattern"
category: "orchestration"
evidence_level: "production"
source: "https://santismm.com/en/patterns/orchestrator-workers"
---
# Orchestrator-Workers

> Evidence: production · Confidence: low · Source: production_system, personal_experience, industry_observation

Canonical: https://santismm.com/en/patterns/orchestrator-workers

An orchestrator LLM dynamically breaks a task into subtasks, delegates each to a worker LLM, and synthesizes the results. Unlike fixed parallelization, the orchestrator decides the subtasks at runtime — making it suited to complex tasks whose decomposition is not known in advance.

## Problem

Some tasks are too complex for a single call and cannot be decomposed up front, because the needed subtasks depend on the input.

## When to use it

Use orchestrator-workers when a task needs dynamic decomposition — the number and nature of subtasks vary by input — and a coordinating model can plan and integrate the work.

## Solution

A lead (orchestrator) model analyzes the task, decides which subtasks are needed, and delegates each to a worker model (often specialized). It then collects and synthesizes the workers' outputs into a final result.

It is the agentic generalization of parallelization: the decomposition is decided at runtime rather than hard-coded, which adds flexibility at the cost of more coordination and unpredictability.

## Benefits

- Handles complex tasks with dynamic decomposition.
- Workers can be specialized per subtask.
- Scales to varied inputs without hard-coded steps.

## Risks

- Coordination overhead, latency and token cost.
- Harder to predict and debug than fixed workflows.
- The orchestrator can mis-plan or loop without limits.

## KPIs

- End-to-end task completion rate: Share of orchestrated jobs that finish correctly across all sub-tasks; the orchestrator owns the whole outcome.
- Worker fan-out & cost: Number of worker calls per job and their combined token cost; orchestration can explode spend if decomposition is sloppy.
- Critical-path latency: Wall-clock of the longest dependent chain, not the sum of workers — this bounds responsiveness.
- Sub-task error rate: How often individual workers fail or return unusable results, driving retries and recovery.

## Observed failure modes

- Bad decomposition: the orchestrator splits the task wrongly, so correct workers still produce a wrong whole.
- Context loss between orchestrator and workers, causing inconsistent or contradictory partial results.
- Cost blow-up from spawning too many workers or deep nesting without budget limits.
- Single point of failure: if the orchestrator misjudges, the entire job fails despite healthy workers.

## Lessons learned

- Invest in the decomposition logic — most failures trace back to how the work was split, not the workers.
- Pass workers the minimum context they need, explicitly, to avoid drift and contradictions.
- Set a budget and depth cap; orchestration without limits is where agent cost spirals.
- Make the orchestrator's plan inspectable so failures can be traced to a specific sub-task.

## FAQs

**How is this different from parallelization?**

Parallelization uses a fixed, predefined split. Orchestrator-workers decides the subtasks dynamically at runtime, so it handles tasks whose shape varies by input.

**Is this a multi-agent system?**

Yes — it is a common multi-agent pattern. Use it only when a task genuinely benefits from dynamic, separable subtasks.

**How do I keep it from running away?**

Set budgets, step limits and stop conditions, and add observability so you can see and bound the orchestrator's planning.
