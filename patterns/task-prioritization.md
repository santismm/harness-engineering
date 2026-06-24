---
slug: "task-prioritization"
title: "Task Prioritization"
type: "pattern"
category: "orchestration"
evidence_level: "production"
source: "https://santismm.com/en/patterns/task-prioritization"
---
# Task Prioritization

> Evidence: production · Confidence: low · Source: production_system, personal_experience, industry_observation

Canonical: https://santismm.com/en/patterns/task-prioritization

Order an agent's candidate tasks by value, urgency, dependencies, and cost instead of processing them first-in-first-out. A scoring function and a priority queue decide what runs next, so limited compute, budget, and time go to the work that matters most. Re-score as state changes, and bound the queue so it cannot grow without limit.

## Problem

An agent that decomposes a goal often ends up with many candidate tasks at once: searches to run, files to read, tools to call, sub-goals to pursue. Processing them in arrival order treats a trivial cleanup step as equal to a blocking, deadline-bound task. Important work waits behind cheap noise, dependencies are violated, and budget is spent on tasks that no longer matter once the situation has changed.

## When to use it

Use this when an agent or orchestrator holds a backlog of independent or loosely coupled tasks and cannot run them all immediately because of compute, rate-limit, cost, or wall-clock constraints. It fits planner and supervisor architectures where one component chooses what executes next. It assumes you can attach signals — impact, deadline, dependency, cost — to each task, and that priorities may shift as new observations arrive.

## Solution

Attach explicit signals to every task: expected impact toward the goal, urgency or deadline, dependency relationships (what must finish first), and estimated cost in tokens, money, or latency. Combine these into a single score with a transparent, auditable function rather than an opaque model judgment. Feed scored tasks into a priority queue so the highest-value ready task runs next. Always respect dependencies first: a task whose prerequisites are unmet is not 'ready' regardless of its score, which keeps the ordering correct and prevents wasted retries.

Make prioritization dynamic. After each step, re-score affected tasks because new results change impact, deadlines approach, and some tasks become obsolete and can be dropped. Protect against starvation by aging — gradually raising the priority of long-waiting tasks — or by reserving capacity for lower tiers. Bound the backlog with an explicit cap and an admission policy: when the queue is full, reject, merge, or evict the weakest tasks instead of letting it grow without limit. Keep the scoring weights configurable and log why each task was chosen so behavior stays explainable.

## Benefits

- Limited compute, budget, and time are spent on high-value, time-critical work instead of whatever arrived first.
- Respecting prerequisites avoids wasted retries and rework caused by running tasks before their inputs exist.
- Re-scoring lets the agent abandon now-irrelevant tasks and promote newly urgent ones as the situation evolves.
- Cost-aware scoring and a bounded queue keep token and latency budgets under control rather than open-ended.

## Risks

- A wrong weighting or bad cost estimate can systematically starve important work or chase low-value tasks; the formula needs review and calibration.
- Without aging or reserved capacity, low-priority tasks may never run, leaving necessary cleanup or background work permanently undone.
- Over-eager re-scoring can cause the agent to switch focus constantly, paying context-switch cost and never finishing anything.
- If decomposition adds tasks faster than they are completed, an uncapped backlog inflates memory, cost, and planning latency.

## KPIs

- Weighted task value completed per unit cost: Captures whether effort lands on high-impact work; good looks like more goal-relevant value delivered per token or dollar than a FIFO baseline.
- Deadline / SLA adherence on time-critical tasks: Shows urgency signals are working; good looks like urgent tasks finishing before their deadline most of the time.
- Starvation indicator (max and tail wait time for low-priority tasks): Reveals whether aging is effective; good looks like bounded worst-case waits with no task stuck indefinitely.
- Queue depth vs. cap and admission/eviction rate: Confirms the backlog stays bounded; good looks like depth held under the cap with eviction reserved for genuinely low-value tasks.

## Observed failure modes

- A high-priority task waits on a low-priority prerequisite that never gets scheduled; the resolver must propagate urgency to blockers.
- Priorities computed once and never refreshed drive decisions on outdated impact or deadline information, so re-scoring must be triggered on relevant state changes.
- Underestimating a task's cost lets it monopolize the budget; estimates need feedback from actual measured consumption.
- An aggressive admission policy drops a task that later turns out to be required, forcing expensive rediscovery; eviction should prefer truly redundant items.

## Lessons learned

- An auditable, configurable formula is easier to debug and tune than an opaque model judgment about what to do next.
- Treat prerequisite completion as a separate readiness check so a high score never lets a task jump ahead of its inputs.
- Add aging or reserved capacity from the start; low-priority background work that never runs becomes a silent correctness gap.
- A hard cap with a clear admission policy is the simplest defense against runaway decomposition inflating cost and latency.

## FAQs

**How is this different from goal decomposition?**

Decomposition produces the tasks; prioritization decides the order in which the resulting tasks are executed. They are complementary: decomposition fills the backlog, prioritization drains it sensibly.

**Should the LLM itself score priorities?**

It can propose signals like estimated impact, but combine them with a transparent, auditable function. A deterministic formula over named signals is easier to calibrate, log, and trust than a single opaque ranking call.

**How do I stop low-priority tasks from never running?**

Use aging to gradually raise the priority of long-waiting tasks, or reserve a fraction of capacity for lower tiers, and monitor tail wait time to confirm nothing is starved.
