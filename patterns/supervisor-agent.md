---
slug: "supervisor-agent"
title: "Supervisor Agent"
type: "pattern"
category: "orchestration"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/supervisor-agent"
---
# Supervisor Agent

> Evidence: industry_observation · Confidence: high · Source: industry_observation, paper

Canonical: https://santismm.com/en/patterns/supervisor-agent

A supervisor agent is a persistent coordinator that manages a team of specialized sub-agents. It reads the conversation state, decides which specialist should act next, routes messages to it, and integrates returned results toward the goal. Unlike a one-shot decomposer, the supervisor stays in the loop across many turns, delegating by capability and re-planning until the task is done or handed back to the user.

## Problem

A single agent given many tools, instructions, and domains becomes unfocused: its prompt bloats, tool selection degrades, and it confuses unrelated concerns. Real workflows need different expertise at different steps (research, coding, billing, compliance), but no single flat agent reliably picks the right capability at the right moment or keeps long multi-step interactions coherent.

## When to use it

Use a supervisor when work spans several distinct, reusable specialist capabilities that must collaborate over a multi-turn conversation or loop, when routing decisions depend on evolving state rather than a fixed plan, and when you need a clear, central place to enforce policy, manage handoffs, and observe which agent did what. It fits heterogeneous teams of agents more than uniform parallel workers.

## Solution

The supervisor owns the control loop and the shared conversation state. On each turn it inspects the latest messages and goal, then decides whether to answer directly, delegate to a named specialist, or finish. Delegation is by capability: each sub-agent has a declared scope (for example a code agent, a data agent, a knowledge agent), and the supervisor routes the relevant slice of context to the chosen one. The specialist runs its own focused tool loop and returns a result or a request for clarification, which the supervisor records before deciding the next step.

Control returns to the supervisor after every specialist turn, so it remains the single decision point rather than letting agents call each other freely. The supervisor integrates partial results, resolves conflicts between specialists, decides when a goal is satisfied, and decides when to hand back to the user. Guardrails such as step budgets, allowed-transition rules, and explicit termination conditions keep the loop from cycling. Structured handoff messages and a shared trace make every delegation auditable, so teams can see who was asked to do what and why.

## Benefits

- Focused specialists with smaller, cleaner prompts
- Centralized routing and policy enforcement
- Modular agents that can evolve independently
- Clear audit trail of who did what

## Risks

- Infinite or ping-pong handoff loops
- Coordination overhead inflates latency and cost
- Supervisor becomes a routing bottleneck
- Context loss across handoffs degrades quality

## KPIs

- Task success / goal-completion rate: Share of sessions reaching the intended outcome without human rescue; the headline quality signal for the supervisor team.
- Handoffs per resolved task: Average delegations to completion; watch for upward drift signaling indecision or routing thrash, not richer work.
- Coordination overhead: Extra tokens, calls, and latency attributable to the supervisor versus a single agent; good means routing earns its cost.
- Routing accuracy: Fraction of delegations sent to the correct specialist on first try, judged against labeled cases.

## Observed failure modes

- Two agents hand work back and forth without progress until a budget cuts the loop
- Supervisor mis-routes to the wrong specialist and never recovers the thread
- Critical context is dropped in the handoff, so the specialist solves the wrong problem
- Specialists' partial results conflict and the supervisor merges them incoherently

## Lessons learned

- Enforce hard step budgets and explicit termination so loops always end
- Make handoffs structured, with intent and scope, not raw message dumps
- Keep specialist scopes narrow and non-overlapping to reduce routing ambiguity
- Instrument every delegation; you cannot debug a multi-agent loop you cannot see

## FAQs

**How is this different from orchestrator-workers?**

Orchestrator-workers decomposes one task into parallel, often homogeneous worker calls and merges them. A supervisor is a persistent coordinator over heterogeneous specialists across a multi-turn loop, re-deciding routing as state evolves rather than executing a fixed plan.

**How do I prevent infinite handoff loops?**

Route control back to the supervisor after each specialist turn, forbid free peer-to-peer calls, set a step or token budget, define allowed transitions, and add explicit termination conditions so the loop cannot cycle indefinitely.

**When should a specialist hand back to the supervisor?**

Whenever it finishes its scoped task, needs a capability it does not own, hits ambiguity needing a decision, or detects it is the wrong agent for the request. The supervisor then integrates and picks the next step.
