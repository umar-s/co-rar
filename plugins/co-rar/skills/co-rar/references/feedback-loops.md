# Reactive Feedback Loop Patterns (P5)

This file documents patterns for closing the **reactive** loop — adapting based on real-world signal that arrives after production has acted. For the **proactive** loop (synthetic adversarial attacks that find failures before reality does), see `adversarial-critic.md`. For the **mutation pipeline** that both loops feed into, see `mutation-stratification.md`. A complete CO-RAR system needs all three.

P5 alone is necessary but insufficient. A system with only P5 is paying real-world cost for every lesson — fine if the cost is low, dangerous if not.

## Why P5 is hard

Closed-loop adaptation requires three things that are individually solved but rarely combined:

1. A signal that genuinely measures real-world outcome (not an internal proxy).
2. A mechanism to translate that signal into a change in the system's behavior.
3. A way to do (2) without a human in every loop.

Most systems have (1) on a dashboard, lack (2) entirely, and assume (3) is impossible. P5 says: design all three from the start.

## Pattern 1 — Telemetry-driven prompt revision

**Use when**: Signal is observable within hours-to-days, and the system has a small number of long prompts driving behavior.

**Shape**:
- Telemetry collects per-output outcomes (delivered/bounced, replied/ignored, flagged/clean).
- On a schedule (daily, weekly), a separate "meta" agent reviews the last period's outcomes alongside the current prompts.
- The meta agent proposes prompt edits: what worked, what stopped working, what to try.
- Edits are applied automatically (with a rollback if the next period degrades) or queued for human approval (with SLA).

**What it replaces**: Engineers manually editing prompts when someone complains.

**Failure mode to watch**: The meta agent overfits to noise. Mitigation: require a minimum sample size before acting; A/B test changes when sample size permits.

## Pattern 2 — Adversary hypothesis prompt

**Use when**: An active adversary is updating their behavior and your system must track it.

**Shape**:
- Maintain a short natural-language document: "Current model of what the adversary is doing." Updated weekly or on telemetry trigger.
- This document is injected into every production prompt as context.
- When telemetry contradicts the current hypothesis (sudden drop in success rate), a triage process updates the hypothesis: review recent failures, identify what changed, write the new theory.

**What it replaces**: Tribal knowledge in a Slack channel that nobody can search.

**Failure mode**: Hypothesis becomes stale and nobody updates it. Mitigation: make the hypothesis age-tagged and surface its age in the system; force a review when it crosses a threshold.

## Pattern 3 — Champion-challenger configurations

**Use when**: You have enough volume to A/B and outcomes arrive within reasonable time.

**Shape**:
- Production runs the "champion" configuration (model + prompt + settings) on 90% of traffic.
- A "challenger" runs on 10% — same task, different configuration.
- After enough volume, compare on the real-world metric. If challenger wins, promote.
- A scheduled process proposes new challengers (often itself an AI task: "look at recent failures, propose a configuration change worth testing").

**What it replaces**: Engineers guessing which change to ship next.

**Failure mode**: Too many challengers running simultaneously contaminates the signal. Mitigation: discipline — one challenger at a time per axis.

## Pattern 4 — Delayed-signal proxies

**Use when**: True signal takes weeks to materialize (e.g., long sales cycle) but you cannot wait that long to adapt.

**Shape**:
- Identify intermediate signals that correlate with the true outcome and arrive sooner. For warmup: reply rate is a fast proxy for eventual deliverability.
- Calibrate the proxy against the true signal periodically. When they diverge, the proxy needs re-calibration.
- Use the proxy for fast loops; use the true signal for slow validation that the proxy is still valid.

**Failure mode**: Optimizing the proxy harms the true outcome (Goodhart). Mitigation: never let the proxy fully replace the true signal; keep periodic ground-truth checks.

## Pattern 5 — Human-in-the-loop with bounded budget

**Use when**: Full automation is risky but full human review is unaffordable.

**Shape**:
- Define an annual or monthly "human review budget" — number of outputs a human will look at.
- The system samples outputs for review intelligently: uncertain cases, configuration changes, outliers, periodic random sample.
- Human feedback feeds back into the prompts or the meta-agent's training context.
- The budget is fixed; the system optimizes which outputs use it.

**What it replaces**: Either "no human ever looks" (unsafe) or "humans look at everything" (unaffordable).

**Failure mode**: Budget gets spent on the wrong outputs. Mitigation: a sampling strategy that is itself reviewed and tuned.

## Anti-pattern: the dashboard-only loop

**Shape**: Metrics are computed, displayed on a dashboard, and a human is expected to "monitor."

**Why it fails**: Humans do not monitor consistently. The dashboard is consulted after a complaint, by which time the degradation is weeks old. There is no defined action — the loop is open even though it looks closed.

**Fix**: Every metric on the dashboard must have a defined action when it crosses a threshold. The action is either automatic (pattern 1 or 3) or a notification with an SLA (pattern 5). "Someone will notice" is not a loop.

## Picking a pattern

- Volume high, signal fast, outcome easy to measure → Pattern 3 (champion-challenger).
- Volume moderate, signal medium-speed, single dominant prompt → Pattern 1 (telemetry-driven revision).
- Active adversary, fast classifier updates → Pattern 2 (adversary hypothesis) layered with Pattern 1.
- Long true-signal latency → Pattern 4 (proxies) plus one of the above for the fast loop.
- High stakes or low volume → Pattern 5 (bounded human review) as the anchor, with Pattern 1 layered when ready.

Most mature CO-RAR systems combine 2-3 of these patterns. Pure single-pattern setups are usually missing a dimension.
