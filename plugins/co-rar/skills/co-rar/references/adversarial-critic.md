# The Adversarial Critic (P6)

P5 is reactive: the system learns from real-world failures after they happen. P6 is proactive: a dedicated AI agent generates attacks against the production design and discovers failures at the speed of compute, before reality charges you for the lesson.

This file documents how to design the critic, what attacks it should generate, and how its output feeds into the mutation pipeline.

## Why the proactive loop is non-negotiable

The reactive loop has two hard limits:

1. **Signal latency.** Real-world failure modes are often discoverable only after damage is done. By the time deliverability drops, your warmup pool is already burned. By the time the fraud team flags a pattern, the fraudsters have moved on.
2. **Signal coverage.** Reality only tests the configurations production actually runs. Configurations you never tried in production are never validated against reality. Large parts of your decision surface are dark.

The adversarial critic addresses both: it surfaces failure modes synthetically (no real damage) and it exercises the full decision surface (including paths production rarely takes).

A CO-RAR system with only P5 is paying real-world cost for every lesson and is blind to large parts of its own behavior. This is a degenerate case, not a viable architecture.

## The critic-production separation

The critic must be a *separate* agent from production, not the production agent in a different mode. Reasons:

- An agent attacking its own work has obvious blind spots. The same prompt that produced the output will defend the output.
- Critic and production should be on different model versions when possible — a stronger critic against the production model surfaces more failures.
- The critic must be free to be hostile. Production prompts are tuned for cooperation with the task; critic prompts must be tuned for hostility toward the design.

A clean separation: production system is one repo / one prompt set / one tool surface. Critic system is another. They share only the interfaces — what production produces and what production is supposed to achieve.

## What the critic generates

Five categories of attack, in rough order of priority:

### 1. Adversary-simulation attacks

The critic plays the role of the real-world adversary (the spam classifier, the fraud detector, the competing system). It analyzes recent production outputs and asks: "If I were the adversary, what pattern would I look for to flag these?"

The output is not just a list of patterns but a generation of inputs designed to make production produce flaggable outputs. The critic then runs these inputs through production and observes whether the outputs are indeed flaggable.

This is the most valuable category for CO-RAR-class tasks with active adversaries.

### 2. Distribution-shift attacks

The critic synthesizes inputs from distributions production has not seen recently — edge cases, low-frequency patterns, plausible-but-rare combinations. It checks whether production handles these gracefully.

Used to catch the failure mode where production silently overfits to the dominant input distribution and breaks on the long tail.

### 3. Prompt-injection and manipulation attacks

The critic generates inputs designed to subvert production's instructions — adversarial user content, embedded instructions, social engineering. Especially important when production has tool access or operates on untrusted input.

This category overlaps with general AI safety but is sharper in CO-RAR systems because production has more autonomy and more leverage.

### 4. Goodhart attacks

The critic looks for inputs where production optimizes its proxy metric while harming the real outcome. If production is optimizing reply rate, the critic finds inputs that maximize replies from low-value sources. If production is optimizing deliverability, the critic finds inputs that land in inboxes but trigger spam reports.

Crucial for any system where the optimized metric is a proxy for the true outcome (which is most systems).

### 5. Boundary-probing attacks

The critic generates inputs near the boundaries between behaviors — between "auto-respond" and "escalate," between "approve" and "review," between "send" and "hold." It surfaces inconsistent or arbitrary behavior at boundaries that production designers usually miss.

## How critic findings feed mutation

Critic findings are inputs to the mutation pipeline (P7), with the same stratification as findings from P5.

**Micro-mutation triggers**: critic finds a prompt edit that reduces failure rate on its attack set without hurting production metrics. Auto-deploy on fraction of traffic, measure, keep or roll back.

**Macro-mutation triggers**: critic finds a structural weakness (a category of attack the current architecture cannot defend regardless of prompt). The fix requires adding a new tool, new layer, or new validator. Generate proposal, escalate to human arbitration.

**Hard-stop triggers**: critic discovers a failure mode that violates a hard guardrail (legal, ethical, safety). System pauses the affected production path until human review. This is the only case where the critic has direct authority to stop production.

## Critic cadence and budget

Two failure modes to avoid:

**Under-running the critic.** The critic generates a few attacks per week, finds little, gets deprioritized. The proactive loop atrophies.

**Over-running the critic.** The critic generates thousands of attacks per hour, most uninteresting, drowning the mutation pipeline in noise. The signal-to-noise of mutation triggers drops, the team learns to ignore critic findings.

Healthy cadence: the critic should generate enough attacks that Adversarial Coverage (fraction of decision surface attacked in a recent window) is meaningful — say, full surface coverage within 24-72 hours — while filtering its own output to surface only attacks that produced unexpected production behavior.

Self-filtering is itself an AI task: the critic runs N attacks, classifies each as "production handled correctly" or "production handled badly," and only the latter feed into the mutation pipeline. Humans review aggregate patterns, not individual attacks.

## Anti-patterns specific to the critic

**Critic = production with a different temperature.** Same prompt, same context, slightly different settings. This is theater; the critic shares production's blind spots.

**Critic outputs ignored.** Findings logged to a dashboard nobody reads. Treat critic output as production telemetry: same SLA, same alerting, same mutation pipeline.

**Critic runs once before deployment.** This is conventional pre-release testing dressed up as adversarial. The proactive loop is *continuous*; one-shot pre-release runs do not qualify.

**Critic only attacks current configuration.** As production mutates, the critic must update its understanding. If the critic attacks last week's design while production has moved on, its findings are stale and misleading. Critic context must include the current production state.

## When P6 can be lighter than full

For systems early in their CO-RAR maturity, full P6 is overhead. A lighter version that still counts:

- A weekly batch where a stronger model reviews recent production outputs and proposes failure modes.
- Findings reviewed manually, fed into prompt edits by hand.
- No continuous run, no auto-deploy.

This is a stepping stone, not a destination. It demonstrates the value of proactive criticism, builds the muscle of acting on critic findings, and creates the substrate to automate over time. A system that stays at this level indefinitely is leaving most of P6's value on the table.
