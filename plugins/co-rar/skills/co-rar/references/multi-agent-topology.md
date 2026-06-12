# Multi-Agent Topology for CO-RAR

A reference implementation of how to embody CO-RAR principles in a multi-agent system. This is *one* concrete topology, not the only one. Use it when the task is large enough to justify the overhead of multiple specialized agents; for smaller tasks, a single agent with mode-switched prompts is sufficient.

## When this topology applies

Use the four-agent topology when **at least two** of the following hold:

- **Volume**: The system handles enough decisions per day that a single agent loop would either bottleneck or lose context across decisions (rule of thumb: thousands of decisions per day, but the real test is whether a single agent can hold state coherently).
- **Surface complexity**: The system has multiple distinct decision surfaces (e.g., generation + routing + scoring), each with its own quality concerns.
- **Adversarial pressure**: An active adversary (P6 from main SKILL.md) requires a dedicated critic running continuously, not just on-demand.
- **Autonomy ambition**: The system is intended to mutate itself with minimal human intervention (P7 macro/micro split is in active use, not aspirational).
- **Audit requirements**: Even though the task is not in CO-RAR anti-condition A1, stakeholders demand a documented account of why the system behaves as it does over time.

If only one applies, this is overkill. Start with a single-agent loop and grow into multi-agent when the seams of single-agent become visible.

## The four roles

### Orchestrator

**Responsibility**: Holds the top-level control loop. Watches reality and the agent system itself. Routes work between agents. Owns the system's current view of "what's happening" and "what to do next."

**Inputs**: Telemetry from production (real-world outcomes), aggregated metrics, anomalies, periodic check-in signals (including "wake the Critic for a scheduled adversarial run, not in response to anything broken").

**Outputs**: Reasoning Tasks dispatched to other agents. Each task is a structured artifact: what happened, what should be investigated, what constraints apply, what the success condition for the response is.

**Memory**: Long-term incident history (vector store of past anomalies and how they resolved), current world-state context, current adversary hypothesis (see P6 in main SKILL.md), current version of the system's own configuration.

**Does NOT**: Generate fixes, attack the system, or approve mutations. The Orchestrator is the dispatcher and historian. Keeping it out of the generation/critique loop is what prevents it from becoming a bottleneck and what preserves the separation of powers.

### Critic (Red Team)

**Responsibility**: Continuously attacks the current design. Both reactive (when the Orchestrator surfaces an anomaly) and proactive (on a schedule, generating synthetic stress-tests without external trigger).

**Inputs**: Current system configuration (whatever form it lives in — prompts, workflow graph, tool surfaces), context from the Orchestrator about recent production behavior, current adversary hypothesis.

**Outputs**: Vulnerability Reports. Each report is a structured artifact: what attack was attempted, what production did in response, why this constitutes a failure (or near-failure), what category of design weakness it suggests, and how confident the Critic is in this finding.

**Memory**: Archive of past attacks and their outcomes. This memory is structurally different from the Orchestrator's: the Critic remembers what it has already tried so it does not loop on the same attacks, and it remembers which categories of attack have historically been productive against this system. Without this memory, the Critic regenerates the same findings indefinitely.

**Does NOT**: Propose fixes. The Critic identifies problems; designing solutions is a different cognitive mode and bundling them in the same agent dilutes both. The Critic is also forbidden from defending its findings to the Builder — see "Builder objection mechanism" below.

### Builder (Blue Team)

**Responsibility**: Synthesizes architectural changes in response to Vulnerability Reports. Designs the fix, generates the new configuration, prepares the rollout plan.

**Inputs**: Current configuration, Vulnerability Report from the Critic, hard constraints from the Architect of Meaning (the human), and the Critic's full attack archive (so the Builder can check whether the proposed fix would have caught past attacks too).

**Outputs**: Proposed Mutations. A mutation is a structured artifact: what changes, what attack it addresses, what side effects are projected, how it could be rolled back, what success criteria apply.

**Memory**: Past mutations and their outcomes. The Builder learns which kinds of fixes worked, which created regressions, which closed one vulnerability and opened another. Without this memory, the Builder makes the same mistakes repeatedly.

**Does NOT**: Deploy autonomously without going through the Sandbox Loop (Critic re-validates) and, for macro-mutations, the Escalation Gateway (human approves).

**Builder objection mechanism**: The Builder is not required to act on every Vulnerability Report. If the Builder analyzes a report and concludes that the attack is unrealistic, the projected risk is overstated, or the proposed fix would cost more than the vulnerability is worth, the Builder returns a Counter-Report to the Critic with reasoning. The Critic must then either: (a) refine its attack with additional evidence, (b) acknowledge the Builder's objection and close the report, or (c) escalate the disagreement to the Orchestrator, who can either route the dispute to the Auditor or surface it to the human as a value conflict.

This mechanism exists because a Critic without pushback drifts toward hyperactivity (the autoimmune failure mode): it generates plausible-sounding attacks the Builder dutifully patches, the system accumulates defensive scar tissue, and eventually it becomes brittle in the opposite direction — over-defended, under-functional. The Builder's right to object is what keeps the Critic calibrated.

### Auditor

**Responsibility**: Observes the agent system itself, not the production system. Watches for drift in the agents' behavior: is the Critic overfit to its own findings, is the Builder accumulating subtle regressions, is the Orchestrator dropping signal categories, are micro-mutations cumulatively drifting the system away from baseline.

**Inputs**: All artifacts produced by the other three agents (Vulnerability Reports, Counter-Reports, Mutations, Reasoning Tasks), the immutable log of mutations and their outcomes, periodic snapshots of agent prompts and configurations.

**Outputs**: Audit Findings. These are reports about the *agent system*, not about the production system: "Critic has generated 73% of attacks in the past two weeks against one decision surface, leaving others uncovered." "Builder's last 12 mutations have moved the system progressively toward conservative outputs without explicit instruction to do so." "Orchestrator dropped seven anomaly signals in the past month because they did not match any known pattern; review whether the pattern catalog needs extension."

**Memory**: Long-term baseline of how the agent system behaved at various points; the immutable mutation log; the catalog of known drift patterns in agent systems.

**Does NOT**: Participate in the production adaptation cycle. The Auditor is deliberately outside the cycle — it has no production deadline, no anomaly to respond to, no fix to ship. This is what allows it to see slow drifts that participants in the cycle cannot see because they are part of the drift.

The Auditor is also the only agent whose findings can trigger **autonomy revocation** (P7) without human action: if the Auditor concludes that a class of mutations the system was previously allowed to make autonomously has produced cumulative damage, it can revert that class to requiring arbitration. The human is notified after the fact rather than asked in advance — this prevents the Auditor from becoming yet another bottleneck.

## Interaction protocol

The adaptation cycle has two triggers and five phases.

### Triggers

**Reactive trigger**: Production telemetry crosses a threshold (anomaly, metric drift, real-world failure). Orchestrator initiates the cycle.

**Proactive trigger**: Periodic schedule (hourly, daily, depending on adversarial pressure). Orchestrator initiates the cycle with no specific anomaly — the cycle exists to surface failures the reactive trigger would miss.

Both triggers run the same five-phase cycle. The difference is only the seed context passed to the Critic.

### Phase 1 — Task formulation

Orchestrator assembles the context: what triggered this cycle, what the current production state is, what known constraints apply, what the success condition for the Critic's response is. Dispatches Reasoning Task to the Critic.

### Phase 2 — Attack and reasoning

Critic runs its attack process. For a reactive trigger, this is focused: "given this anomaly, find the design weakness." For a proactive trigger, this is broad: "given the current state and your attack archive, generate a coverage-maximizing set of attacks against decision surfaces under-tested in the recent window."

Critic produces a Vulnerability Report. If no vulnerability found, report explicitly says so — null findings are signal too and the Orchestrator should know coverage was attempted.

### Phase 3 — Reconstruction (with objection)

Builder receives the Vulnerability Report. First decision: act or object. If the Builder concludes the report is sound, it produces a Proposed Mutation. If not, it returns a Counter-Report to the Critic.

In the Counter-Report case, the Critic either refines and resubmits, withdraws, or escalates to the Orchestrator. The Orchestrator decides whether to route the dispute to the Auditor (if it suspects systemic drift in either agent) or to the human (if the dispute reveals a value conflict the agents cannot resolve).

When a Proposed Mutation is produced, it includes everything needed for evaluation: the change, the rationale, the projected effects, the rollback plan.

### Phase 4 — Sandbox loop (Critic re-validates)

Proposed Mutation goes back to the Critic before deployment. The Critic now plays defender's adversary: "given this proposed change, does it close the original vulnerability without opening new ones?" The Critic generates targeted attacks against the proposed new configuration.

If the new configuration survives: mutation is approved for the next phase. If not: report goes back to the Builder with the new finding, and Phase 3 repeats. Bounded by a maximum loop count to prevent oscillation — if Builder and Critic cannot converge in N iterations, the matter escalates to the human as a hard problem.

### Phase 5 — Deployment via Escalation Gateway

The Escalation Gateway is the structural point where P7 stratification is enforced. It receives the approved Proposed Mutation and classifies it:

- **Micro**: bounded, reversible, measurable change. Deploy to fraction of traffic, monitor, auto-rollback on regression. No human involvement.
- **Macro**: structural change. Hold deployment, format the proposal for human review, escalate.
- **Hard-stop**: change would violate a guardrail explicitly set by the human (legal, ethical, financial limit). Reject without deployment; surface to human as "Critic found a real vulnerability but the only fix violates a constraint you set — please advise."

The Gateway's classification is itself auditable — the Auditor reviews classifications periodically to ensure micro-vs-macro is not silently drifting.

## Escalation Gateway: triggers for human review

Macro classification is determined by the *type* of change, not by an arbitrary numeric threshold. Triggers include:

- **Behavioral contract change**: The system would refuse something it used to do, or do something it used to refuse.
- **New tool added or removed**: The action surface expands or contracts.
- **Guardrail proximity**: The proposed change moves the system closer to a guardrail boundary (legal, ethical, financial), even if it does not cross.
- **Novel mutation category**: The proposed change is structurally unlike any past mutation; no historical evidence of its safety.
- **Conflict resolution**: Builder and Critic could not converge; the disagreement is itself the artifact requiring human judgment.
- **Cumulative drift signal**: Auditor flags that recent micro-mutations have drifted the system in a direction that, taken together, would have triggered macro review.
- **Value tension**: The proposed change improves one outcome at measurable cost to another (e.g., security vs. conversion), and the value hierarchy does not unambiguously resolve the trade-off.

The presentation to the human is always structured: what is changing, why, what improves, what costs, what alternatives the Builder considered, what the rollback path is. Never freeform prose — the human's job is judgment, and structured presentation preserves the human's bandwidth for the judgment, not for parsing.

## Agent memory architecture

Each agent has distinct memory because the agents do distinct work. Mixing memories is a common failure mode — it pulls each agent toward generalist behavior and dilutes the value of specialization.

- **Orchestrator memory**: incident history, current world-state, current adversary hypothesis, current system configuration, dispatch log.
- **Critic memory**: attack archive (attempts, outcomes, productive categories), current understanding of the production system's decision surfaces, current adversary hypothesis (shared read-only from Orchestrator).
- **Builder memory**: mutation history with outcomes, configuration history (every past version of the system), pattern library of fixes that have worked or failed.
- **Auditor memory**: baseline snapshots of agent behavior at past points, the immutable mutation log, catalog of known drift patterns, history of past autonomy grants and revocations.

The immutable mutation log is shared across all agents but writable by none except the Builder (writes new entries) and the Auditor (writes drift annotations). This shared log is the system's spine — it is what makes the whole loop auditable to humans and replayable for debugging.

## Anti-patterns specific to multi-agent CO-RAR

**Role bleed**: The Orchestrator starts generating fixes when it thinks the Builder is too slow. The Critic starts proposing solutions because they seem obvious. The Builder starts re-attacking its own work to "save time." Each instance of role bleed dilutes the specialization and the system collapses back toward a single confused agent. Police role boundaries explicitly in prompts.

**Critic hyperactivity (autoimmune)**: Without the Builder's objection mechanism, the Critic generates ever more creative attacks, the Builder patches them all, the system accumulates defensive complexity until it can barely function. The objection mechanism is the immune-tolerance equivalent that prevents this.

**Builder over-deference**: The Builder treats every Vulnerability Report as a directive rather than a hypothesis. This is the inverse of hyperactivity — Critic-driven runaway with the Builder as enabler. The objection mechanism must be culturally and procedurally real, not just nominally available.

**Auditor capture**: The Auditor's findings are routed through the Orchestrator, who routinely deprioritizes them in favor of production tasks. Over time the Auditor learns to soft-pedal findings to get any attention at all. Mitigation: Auditor has a direct channel to the human, independent of the Orchestrator, for findings above a severity threshold.

**Sandbox loop infinite**: Builder and Critic ping-pong forever, each finding fault in the other's work. Without a bounded loop count and escalation path, the system never converges on a deployable change. Always set a maximum N.

**Single-model multi-agent**: All four agents are the same model with different system prompts. This is *not* genuinely multi-agent — they share the same blind spots, same biases, same failure modes. Use different models for different agents when possible, especially Critic ≠ Builder (the Critic should be the strongest model available; cheaping out on the Critic means missing attacks).

**Heavyweight Orchestrator**: The Orchestrator grows to handle "everything that isn't clearly someone else's job" and becomes a bottleneck. Keep the Orchestrator narrow: dispatch and history, nothing else. If something needs to happen that no other agent owns, the right answer is usually to add a fifth specialized agent, not to expand the Orchestrator.

**Auditor inside the cycle**: The Auditor is given production responsibilities to "make use of the spare capacity." This kills its value — once it participates in the cycle, it cannot see drift in the cycle. The Auditor's idleness during normal operation is the price of its perspective.

## When to step down from this topology

The four-agent topology has real overhead: more model calls per cycle, more inter-agent communication, more failure modes (e.g., the Sandbox loop), more memory to maintain. If the production system is small enough that single-agent handling does not fail, this overhead is unjustified.

Signals that the topology is overkill for the task:

- Less than several hundred decisions per day.
- One decision surface, not multiple.
- No active adversary (only the long-tail problem from S3 in main SKILL.md).
- Macro-mutations are rare enough that human review is not a bottleneck.

In these cases, prefer a single agent with mode-switched prompts (one prompt for production work, one for self-critique, one for mutation proposal) and a clear log. You can grow into multi-agent when the seams of single-agent become visible — typically when one agent starts failing in mode-specific ways, or when memory pollution between modes degrades quality.

The topology is a tool, not a virtue. The principles in the main SKILL.md are the virtue; this is one way to embody them at scale.
