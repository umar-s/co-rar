---
name: co-rar
description: Continuous Resilient Adversarial Reasoning in Codeless Design. Use when designing or critiquing systems where output quality is not contract-checkable, an active adversary or moving target exists (anti-spam, anti-fraud, growth hacks, gray-zone automation), or the user is building AI-first backends, agentic pipelines, codeless workflows, or systems meant to adapt under environmental pressure rather than remain static. Trigger aggressively for warmup systems, deliverability, cold outreach automation, AI agents outsmarting classifiers, codeless architecture, production degradation, or "AI bolted onto scripts" complaints. Also trigger when reviewing architectures heavy in validators, dictionaries, regex gates, or rule engines wrapping a thin AI layer, or static no-code workflows lacking self-modification. Provides diagnostic, prescriptive architecture, and operational metrics for systems-as-organisms. Do NOT use for regulated deterministic domains (finance, medical, legal) where tail-risk dominates flexibility.
---

# CO-RAR — Continuous Resilient Adversarial Reasoning in Codeless Design

A frame for designing systems where reasoning belongs inside execution, where the system is expected to mutate under environmental pressure rather than break, and where quality is maintained by a continuous adversarial loop rather than by upfront correctness.

## The core insight

LLMs are trained on the public, mature corpus of how the internet was built for 30 years: scripts orchestrate, rules validate, AI (if present) is a narrow optional component wrapped in guardrails. Static systems are deployed, monitored, and patched in discrete release cycles. This is a sensible default for the median task the median engineer faces. It is the wrong default for a growing class of tasks where the median solution produces medianly-bad results — and bad results, in this class, mean systemic decay rather than minor inconvenience.

When a task in this class is handed to an LLM without explicit override, the LLM will:

1. Build a system whose primary control flow is deterministic code.
2. Insert an AI call as a leaf node, wrapped in validators that strip most of its value.
3. Treat the system as a finished artifact to be monitored, not as an organism to be evolved.
4. Defend this architecture as "robust" and "best practice."
5. Produce a system that runs, ships, gets praised, and silently decays under real-world pressure.

CO-RAR is the frame for recognizing tasks in this class and designing for them correctly. It has three pillars:

- **Codeless Design** — the system's logic lives in modifiable form (prompts, visual workflows, declarative configs), not in compiled code.
- **Adversarial Reasoning** — an AI critic continuously attacks the current design with synthetic stress-tests, in addition to learning from real-world failures.
- **Continuous Resilience** — the system mutates its own structure in response to both attacks and reality, rather than preserving a fixed shape.

## Resilience vs Stability

This distinction is the philosophical core and worth stating sharply.

A **stable** system preserves its current shape. Under pressure it either holds or breaks. Traditional software is stable: it does exactly what it did yesterday until a release changes it. When the environment shifts faster than the release cycle, a stable system fails by becoming wrong while still running.

A **resilient** system preserves its function by changing its shape. Under pressure it mutates. Biological organisms are resilient: the cells turn over, the immune system updates, the overall function persists across a constantly-changing substrate. A CO-RAR system aims for biological resilience, not mechanical stability.

This is not a metaphor. It is an engineering target with concrete consequences:

- A stable system measures success by uptime and feature completion. A resilient system measures success by sustained outcome under adversarial pressure.
- A stable system treats change as risk. A resilient system treats lack of change as risk.
- A stable system patches after failure. A resilient system mutates before failure.

If your instinct when something breaks is "lock down what works" — you are designing for stability. CO-RAR demands the opposite instinct: "what about the system needs to change so this class of break stops mattering."

## When CO-RAR applies (diagnostic)

A task belongs in the CO-RAR class if **all** necessary conditions and **at least one** sufficient condition hold, and **no** anti-conditions apply.

### Necessary (all must hold)

**N1 — Quality is not contract-checkable.** No `is_good(output) -> bool` you can write in code. Quality is either inherently fuzzy or only verifiable through real-world reaction.

**N2 — Reasoning is needed in execution, not just in design.** Each individual output requires fresh context analysis. You cannot think once and codify a template.

**N3 — Static heuristics cost more than they save.** Dictionaries, regex validators, rule engines produce outputs worse than the cost of the reasoning call they replaced.

### Sufficient (at least one strengthens the case)

**S1 — Active adversary or moving target.** Anti-spam, anti-fraud, platform algorithms, competitive countermoves. The environment adapts faster than you can ship code.

**S2 — Closed or fast-evolving knowledge corpus.** Best practices unpublished or obsolete within weeks. Training data is structurally behind.

**S3 — Long edge-case tail.** Each case is slightly unique. Exhaustive cataloguing is impossible.

**S4 — Expected lifetime exceeds release cycle frequency.** The system must operate correctly across timescales longer than the engineering team can re-release. Even without an adversary, if the world will shift faster than the team can ship patches, the system must self-modify.

### Anti-conditions (any one disqualifies the task)

**A1 — Regulatory determinism required.** Compliance demands "show me the rule that produced this output." Reasoning-in-execution and autonomous mutation both fail audit.

**A2 — Tail-risk dwarfs flexibility gains.** Worst-case output causes irrecoverable damage. Deterministic gates are insurance, not cargo cult.

**A3 — Task is one-shot.** No feedback loop, no iteration, no adversary. CO-RAR overhead is unjustified.

## The seven principles of CO-RAR design

When a task qualifies, design by these principles. Each is a default to be overridden only with specific argument.

### P1 — Inversion of control

**AI orchestrates, code is the set of levers AI pulls. Not the reverse.**

If you catch yourself writing "the script decides when to call the AI" — stop. That is the trained default reasserting itself. The correct shape: AI receives the task and available tools, AI decides what to do, code is the tool surface.

Diagnostic question when reviewing an architecture: "What is the top-level control loop?" If it is a Python `for` loop calling an LLM as a subroutine, the architecture is inverted.

### P2 — Quality is a 5-axis balance, not a code problem

When output is wrong, the bug is not in the code. The bug is on one of these axes:

1. **Model choice** — capability per dollar per token.
2. **Model settings** — temperature, top_p, reasoning depth, tool budget.
3. **Layering** — how many model passes review/improve/critique each other.
4. **Prompt** — the actual instruction surface.
5. **Scope of access** — what the model can see and do.

Debugging means moving along these axes, not editing Python. If a teammate proposes a code fix to a quality problem, ask which of the five axes they ruled out first.

### P3 — Backend collapses to prompt + levers

If the backend logic of a feature can be stated in natural language, it should live in natural language — as a prompt — not in Python. The frontend stays code (deterministic, fast, accessible). The backend, for CO-RAR-class features, becomes a prompt with access to a defined set of levers.

Test: write the backend behavior as instructions to a smart contractor. If the instructions are vastly shorter and clearer than the equivalent code, the prompt **is** the implementation.

### P4 — Headless deep-reasoning sessions for non-interactive work

Anything that does not need sub-second UI response is a candidate for a headless long-reasoning session with broad environment access. Batched generation, periodic re-evaluation, overnight analysis, content production at volume — these run dramatically better as long sessions with full server access than as short stateless calls stitched by a queue.

A one-hour reasoning session at higher per-token cost frequently beats a thousand short calls in total cost, because short calls duplicate context-loading and lack cross-task awareness.

### P5 — Reactive feedback loop from reality

The system must include:

- **Telemetry on real-world outcome**, not internal proxies. Did the email land? Did the lead reply? Did the platform flag us? Internal LLM-as-judge scores drift together with the producing model.
- **Automatic or semi-automatic revision** of prompts, model choice, or architecture based on telemetry. The 5 axes from P2 are the levers; telemetry tells you which to pull.
- **Adversary-tracking**: an explicit hypothesis about what the opposing system is doing, updated when telemetry contradicts it.

A CO-RAR system without P5 is a demo, not a system.

### P6 — Proactive adversarial pressure (red team in the loop)

P5 alone is reactive: you wait for reality to hurt you, then learn. That is necessary but insufficient. CO-RAR systems also include an internal critic that *generates* failure scenarios before reality does.

The critic loop:

- A separate AI agent, distinct from the production agent, holds the role of attacker.
- Its job: given the current production design, generate synthetic adversarial inputs and "worst-case" scenarios that should break the system.
- Production runs against these synthetic attacks before they reach reality.
- Failures discovered by the critic trigger the same revision mechanism as P5 (prompt edits, model swaps, architectural changes).

Why this matters: real-world signal arrives slowly and at the cost of real damage. The adversarial critic discovers failure modes at the speed of compute and at the cost of tokens. A mature CO-RAR system has the critic running continuously, generating fresh attacks every few hours, with coverage targets across the system's decision surface.

The critic is not a replacement for P5. It is a layer that catches failures the reactive loop would catch eventually, but earlier and cheaper. Both loops must exist.

### P7 — Stratified mutation (micro autonomous, macro arbitrated)

The system mutates itself continuously, but not all mutations are equal. Stratify them.

**Micro-mutations** — small, local, reversible changes to prompts, settings, layer counts. Autonomous. Rolled out on a fraction of traffic, measured, kept or rolled back. No human in the loop. Frequency: hours to days.

**Macro-mutations** — architectural changes: new tools added to the AI's scope, fundamental prompt restructuring, removal of a validator, change in the system's external behavior contract. Requires human arbitration before deployment. The AI proposes, generates the diff, explains the rationale; a human approves. Frequency: weeks.

**Why the split**: autonomous micro-mutation is necessary because the world moves faster than humans can review every change. Autonomous macro-mutation is unsafe because architectural changes can cascade in non-obvious ways and because they often touch the boundary with anti-conditions A1/A2.

The default for any proposed mutation is "requires arbitration." Autonomy is granted to a class of mutations only after explicit demonstration that the class is bounded, reversible, and measurable. Reverse the burden of proof: the AI must show why a mutation class is safe to automate, not why it needs review.

## Operational metrics

CO-RAR systems are measured by their ability to sustain function under pressure, not by absence of bugs at release. These are the metrics; target values depend on the domain and the speed at which ground-truth signal arrives.

- **TtR (Time to Resiliency)** — Elapsed time from first observation of a new failure mode (in reality or in the adversarial critic) to a deployed mutation that addresses it. Bounded below by signal latency: if Gmail takes three days to react, TtR cannot be five minutes. Optimize TtR relative to signal latency, not in absolute terms.
- **ADR (Anti-Degradation Rate)** — Fraction of failures caught by the adversarial critic (P6) before they manifest in reality. Higher ADR means the proactive loop is doing its job. ADR = 0 means you have only a reactive loop and are paying real-world cost for every lesson.
- **Drift Resistance** — Stability of the primary outcome metric across measured environmental shifts. The point is not "no drift in metrics" but "outcome metric holds while environment changes." A system whose outcome holds across a 30% environmental shift without human intervention is exhibiting resilience.
- **Adversarial Coverage** — Fraction of the system's decision surface that the critic has attacked within a recent window. Low coverage means large parts of the system are untested against synthetic attacks and rely entirely on reality to expose flaws.
- **Mutation Cadence (split)** — Rate of micro-mutations (autonomous) vs macro-mutations (arbitrated). A healthy system has high micro-cadence and low-but-nonzero macro-cadence. Zero micro-cadence means the autonomous loop is broken. Zero macro-cadence means humans have stopped engaging with structural evolution and the system will eventually hit a ceiling.

Do not commit to specific target numbers for these metrics in the abstract. Targets are per-domain and emerge from measurement.

## The human role: Architect of Meaning

In a CO-RAR system the human is not coder, tester, or log-monitor. Those roles are delegated. The human occupies two positions:

**Intent Specifier.** Sets the top-level objectives (what outcome matters), the hard constraints (ethical, legal, financial guardrails the system must never cross regardless of what the metrics say), and the value hierarchy when objectives conflict.

**Arbiter of Value Conflicts.** When the adversarial loop discovers an irreducible trade-off — "to defend against fraud we must add friction that costs 12% conversion" — the AI does not pick. It formalizes the dilemma (here are the options, here are the projected outcomes, here is the value tension) and escalates. The human picks. The pick is not a one-off; it updates the system's value hierarchy and influences future autonomous decisions.

The two positions combined make the human responsible for the *direction* of the system's evolution while delegating its *mechanism*. This is the only durable division of labor: humans cannot keep up with the mechanism, and AI cannot legitimately set the direction.

## The boundary still exists

The trained default is "scripts orchestrate, AI is a leaf." CO-RAR inverts this for the qualifying class. But the boundary between deterministic code and reasoning-in-execution has not disappeared — it has moved.

For each sub-task inside a CO-RAR system, decide explicitly which side of the boundary it falls on. Default to reasoning; require specific argument to push something into deterministic code. Valid arguments:

- The sub-task is genuinely deterministic (parsing fixed format, computing a hash).
- The sub-task has hard latency requirements reasoning cannot meet.
- The sub-task is on the audit-required path (anti-condition A1).
- The sub-task has been measured and reasoning genuinely loses on quality-per-dollar.

Invalid arguments (trained-default reasserting itself):

- "AI is unreliable" — restate as which specific failure mode, on which axis, measured how.
- "We should validate AI output" — validate against what? If against a rule, why didn't the rule generate it? If against another model, that is layering (P2), not validation.
- "It's more efficient" — efficient on what metric. CO-RAR-class tasks are quality-dominated.

## Diagnostic walkthrough for a candidate task

When asked to design or critique a system, walk these steps in order:

1. **Classify**: N1+N2+N3 + at least one S, no A. If not, CO-RAR does not apply.
2. **Locate control**: Where does top-level decision-making live? If in code, propose inversion (P1).
3. **Audit the validator stack**: For each validator, ask whether it encodes a rule that could just generate the right output. If yes, replace with a reasoning pass.
4. **Map the 5 axes**: Identify which axis is currently being tuned. Usually only the prompt; large gains often hide in model choice and layering.
5. **Check headless candidates** (P4): What work does not need sub-second response? Move to long-session deep reasoning.
6. **Demand the reactive loop** (P5): What real-world signal closes the loop? If none, design stops until specified.
7. **Demand the proactive loop** (P6): Where is the adversarial critic? If absent, the system is learning only from real damage.
8. **Stratify mutations** (P7): Which changes are autonomous, which require arbitration? If everything is autonomous, the system is unsafe. If everything requires arbitration, the system cannot keep up.

## Critiquing AI-generated architectures

Failure modes to look for, in order of frequency:

1. **AI as decoration**: AI output gets discarded or overridden by downstream rules.
2. **Validator-heavy gating**: dozens of dictionaries, regexes, length checks. Each fixed a specific bad output, none generalize.
3. **Wrong direction of control**: AI called from inside a `for` loop instead of running the loop.
4. **Single-pass production**: one model call generates the final output. No critique, no improvement, no diversity pass.
5. **No feedback loop**: no way to learn that yesterday's outputs were worse than last week's.
6. **Reactive-only loop**: telemetry exists but no adversarial critic. System learns only from real damage.
7. **Unstratified mutation**: either all changes require human approval (system can't keep up) or all changes are autonomous (system can't be trusted).
8. **Defaulted model and settings**: cheapest model, temperature 0.7, no thinking budget. The 5 axes at factory settings.

Each has a remedy under P1–P7. Name the failure mode, point to the principle, propose the change.

## Further reading

- `references/diagnostic-examples.md` — worked examples of classifying tasks and proposing CO-RAR-shaped architectures.
- `references/anti-patterns.md` — catalogue of trained-default architectures the LLM will reach for, with the CO-RAR inversion for each.
- `references/feedback-loops.md` — patterns for closing the P5 reactive loop in environments where real-world signal is delayed, noisy, or expensive.
- `references/adversarial-critic.md` — patterns for designing the P6 proactive loop: how the internal critic is built, what attacks it generates, how its findings feed mutation.
- `references/mutation-stratification.md` — patterns for P7: how to classify mutations as micro vs macro, how to demonstrate a class is safe to automate, how to maintain the macro-arbitration channel without it becoming a bottleneck.
- `references/multi-agent-topology.md` — a reference implementation of CO-RAR as a four-agent system (Orchestrator, Critic, Builder, Auditor). Consult when the production system is large enough that a single-agent loop would bottleneck or lose coherence; not all CO-RAR systems need this overhead, and the file includes explicit criteria for when to use it vs when to stay single-agent.
