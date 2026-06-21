---
slug: "human-escalation"
title: "Human Escalation"
type: "pattern"
category: "safety"
evidence_level: "industry_observation"
source: "https://santismm.com/en/patterns/human-escalation"
---
# Human Escalation

> Evidence: industry_observation · Confidence: high · Source: industry_observation, paper

Canonical: https://santismm.com/en/patterns/human-escalation

Hand the whole task to a human when the agent detects it is out of its depth — low confidence, repeated failure, ambiguity, or sensitive situations — and pass full context so the person can take over without re-investigating. Unlike an approval gate, which pauses one action for sign-off, escalation transfers ownership so the agent stops driving. The hard part is calibrating triggers to avoid both over- and under-escalation.

## Problem

An autonomous agent will inevitably encounter cases it cannot handle well: inputs outside its training distribution, requests it keeps failing to satisfy, genuinely ambiguous goals, or emotionally and legally sensitive moments. If it presses on anyway, it produces confidently wrong answers, loops, or harmful actions — and the user discovers the failure too late. Yet routing everything to humans defeats the point of automation and overwhelms staff. The system needs a disciplined way to recognize the edge of its competence and transfer the task before damage is done.

## When to use it

Use this pattern wherever an agent acts with meaningful autonomy and the cost of a wrong outcome exceeds the cost of a human glancing at it: customer support, claims and case handling, financial or medical triage, content moderation, and operational copilots. It assumes a human queue or on-call function exists to receive escalations and that the agent can observe signals about its own performance. It is most valuable when failures are silent — when a confidently wrong answer is worse than no answer — and when a subset of cases is known to be hard, rare, or regulated.

## Solution

Define explicit escalation triggers and wire them into the agent's main loop as first-class exit conditions, not afterthoughts. Common triggers are confidence below a threshold (from model scores, self-critique, or a verifier), loop or repeated-failure detection (the agent retries the same step without progress), structural ambiguity (multiple valid interpretations of the goal), and sensitivity signals (negative sentiment, safety keywords, high-value accounts, or regulated topics). Each trigger should map to a routing decision: which human or team, with what priority. Treat thresholds as tunable parameters owned by the team, reviewed against real outcomes, because they encode the trade-off between automation rate and error rate.

When a trigger fires, the agent must perform a clean handoff: stop acting, package the full context — original request, what it attempted, intermediate results, its current best guess, and why it escalated — and route it to the right queue via a ticket or live handoff. The receiving human should be able to take over without re-investigating from scratch; context quality is what makes escalation feel like help rather than a dropped ball. Always provide a graceful fallback message to the end user ("I'm bringing in a specialist") so the experience degrades smoothly. Finally, log every escalation with its trigger and resolution so appropriateness can be measured and triggers retuned.

## Benefits

- Hard cases reach a human before the agent produces a confidently wrong outcome, capping the blast radius of mistakes.
- Only genuinely difficult cases are handed off, so routine volume stays automated and staff focus on what needs judgment.
- A clean handoff with context means users are helped rather than bounced, and humans resume without starting over.
- Logged triggers and resolutions provide the evidence trail regulators and risk owners expect for meaningful human oversight.

## Risks

- Thresholds set too conservatively push easy cases to humans, erasing automation gains and burying staff in noise.
- Thresholds set too loosely let the agent power through cases it should have handed off, causing silent bad outcomes.
- If the payload is thin, the human re-investigates from scratch and escalation feels like a dropped task, not assistance.
- Model self-confidence often does not track real accuracy, so naive score thresholds escalate the wrong cases in both directions.

## KPIs

- Escalation rate: Share of tasks handed to humans. Watch the trend and the distribution, not a target number — a sudden spike or drop signals a miscalibrated trigger or a shift in input mix.
- Escalation appropriateness: Of escalated cases, how many genuinely needed a human (true positives) versus could have been handled. Sampled human review of escalations is the most reliable read.
- Missed-escalation rate: Of automated resolutions, how many later turned out to be wrong and should have been escalated. The hardest and most important signal; mine complaints, reopens, and audits to find them.
- Handoff context sufficiency: How often the receiving human can take over without re-contacting the user or re-investigating. Track via agent feedback on whether the package was complete.

## Observed failure modes

- Triggers tuned once and never revisited fall out of step as inputs and models change, silently shifting the automation/error balance.
- Cases route into a queue that no one owns or that is overwhelmed, so escalated users wait indefinitely — worse than a wrong answer.
- An agent optimized to avoid escalation learns to express false confidence, suppressing the very signal the pattern depends on.
- Handoff strips formatting, intermediate reasoning, or attachments, forcing the human to rebuild the situation and erasing the speed benefit.

## Lessons learned

- Validate that your confidence signal correlates with actual accuracy before thresholding on it; pair model scores with a verifier or self-critique.
- The difference between a good and bad escalation is almost entirely the handoff payload; invest there before tuning thresholds.
- Treat thresholds as living parameters reviewed against sampled escalations and missed escalations, owned by the team, not frozen at launch.
- Even a perfect trigger fails sometimes; a graceful holding message and an owned queue prevent failures from becoming abandonments.

## FAQs

**How is this different from a human-approval gate?**

An approval gate pauses one specific high-impact action and asks a human to sign off, then the agent continues. Escalation transfers ownership of the whole task — the agent stops driving because it shouldn't proceed at all. Use a gate for 'should I do this one thing?' and escalation for 'I'm out of my depth, please take over.'

**What's the right escalation rate?**

There is no universal number; it depends on task difficulty mix and the cost of errors. Optimize for appropriateness, not a target rate: escalate cases that genuinely need a human and minimize both unnecessary handoffs and missed escalations. Review the rate as a signal of miscalibration, not as a goal in itself.

**Can I just escalate whenever model confidence is low?**

It's a useful trigger but rarely sufficient alone, because model self-confidence often does not track real accuracy. Combine it with loop detection, ambiguity checks, and sensitivity signals, and validate that your confidence measure actually correlates with correct outcomes before trusting a threshold.
