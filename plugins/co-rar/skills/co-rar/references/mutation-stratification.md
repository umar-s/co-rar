# Mutation Stratification (P7)

A CO-RAR system mutates itself continuously. P7 governs which mutations happen autonomously and which require human arbitration. Getting this split wrong in either direction breaks the system: too much autonomy makes it unsafe, too much arbitration makes it unable to keep up with the environment it was supposed to track.

This file documents how to classify mutations, how to demonstrate that a mutation class is safe to automate, and how to keep the arbitration channel functional.

## The default is arbitration

Reverse the burden of proof. The default for any proposed mutation is "requires human approval." A mutation class is granted autonomy only after explicit demonstration that the class is:

1. **Bounded** — the space of possible changes within the class is enumerable and small.
2. **Reversible** — every change can be rolled back without residual effect.
3. **Measurable** — there is a metric that reliably tells you within a short window whether the change helped or hurt.

If you cannot demonstrate all three, the mutation is not autonomous. This rule prevents the slide where every mutation gradually becomes "obviously safe to automate" until the system is autonomously rewriting itself in ways nobody can audit.

## Micro-mutations: examples and rules

Examples of mutations that typically qualify as micro:

- **Prompt phrasing edits** within an established prompt structure (rewording an instruction, adjusting an example, changing tone descriptors). Bounded by the prompt template, reversible by reverting the diff, measurable by output quality on holdout set.
- **Model setting tuning** within established ranges — temperature, top_p, reasoning depth within a window agreed by the team.
- **Few-shot example rotation** — swapping examples in a prompt's few-shot section based on recent performance.
- **Tool selection within an established tool set** — the AI deciding to use tool A vs tool B on a given task, when both are pre-approved.

Rules for micro-mutations:

- **Always deployed on a fraction first.** Never replace production wholesale. 5-10% of traffic, measured against the control on real-world outcomes.
- **Auto-rollback on regression.** If the measured outcome on the fraction is worse than control beyond a threshold, automatic revert without human action.
- **Logged immutably.** Every mutation, its rationale, its measurement window, its outcome. This log is the audit trail that justifies the autonomy.
- **Aggregated review.** Humans don't review individual mutations, but review aggregated patterns weekly. The question is "is the micro-mutation system as a whole behaving well," not "was this specific change right."

## Macro-mutations: examples and rules

Examples that typically require arbitration:

- **Adding or removing a tool from the AI's scope.** Changes the action surface, can cascade in non-obvious ways.
- **Restructuring a prompt at the architectural level** — removing a section, adding a new sub-agent, changing the role definition.
- **Changing the model family.** A switch from one model to a fundamentally different one is not a setting change; it shifts capability profile and failure modes.
- **Removing a validator or guardrail.** Especially one near anti-conditions A1/A2. Even if measurement supports it, this is a structural decision with cascading risk.
- **Changing the external behavior contract.** What the system promises to do, what it refuses to do, what triggers escalation. These are policy decisions, not implementation details.

Rules for macro-mutations:

- **AI proposes, human disposes.** The AI generates the proposed change with rationale, projected impact, identified risks, and rollback plan. The human approves, modifies, or rejects.
- **Proposals are structured artifacts**, not freeform suggestions. A template: what is changing, why (what telemetry or critic finding triggered this), what the projected effect is, what could go wrong, how to roll back, what success looks like in N days.
- **Approval is logged with reasoning.** Not just yes/no — the human's reasoning for approving (or the modification they imposed) becomes context for future proposals.
- **Macro-mutation cadence is bounded.** Even if the system wanted to make ten macro-mutations a week, the human review channel can sustain perhaps one or two per week before degrading to rubber-stamping. Cap the cadence to preserve the quality of arbitration.

## The dangerous middle

Some mutations look micro but behave macro. The trap to watch:

**Prompt edits that shift the system's effective behavior contract.** A "small wording change" in a prompt can change what the system refuses to do or what it escalates. This is a macro change wearing micro clothing. The rule: if the prompt edit changes what the system *will* do (not just how well), it is macro.

**Cumulative micro-drift.** A hundred small autonomous prompt edits over a quarter can drift the system far from where it started, in ways no individual change would have triggered review. Mitigate by periodic "drift audit": diff the current prompt against the baseline of N weeks ago, surface the cumulative change for review even though each step was micro.

**Settings changes near limits.** Lowering temperature from 0.7 to 0.6 is micro. Lowering from 0.7 to 0 changes the qualitative behavior of the model and is macro despite being "just a number change."

## When the arbitration channel becomes a bottleneck

The macro-mutation channel is a constrained resource (human attention) competing with an unconstrained source (AI-generated proposals). It will become a bottleneck unless actively managed.

Symptoms:

- Backlog of pending macro proposals grows weekly.
- Humans batch-approve to clear the queue without genuine review.
- AI learns to formulate proposals in ways that get rubber-stamped, regardless of merit.
- High-quality proposals get the same attention as low-quality ones.

Counter-measures:

- **AI pre-prioritizes proposals.** Not all macro changes are equally urgent; the AI ranks them and the human reviews top-ranked first.
- **Triage triage.** A separate fast-pass channel where the human spends 30 seconds per proposal classifying as "needs real review," "auto-approve with audit," or "reject." Only the first gets deep review.
- **Quality feedback on proposals.** The human's approval/rejection/modification feeds back into how the AI formulates future proposals. Bad proposals should die early, good proposals should compress.
- **Increase the micro-class scope.** If the bottleneck is chronic, reconsider whether some currently-macro changes can be demoted to micro by adding tighter measurement and rollback.

## When autonomy must be revoked

Sometimes a mutation class that was granted autonomy turns out to be unsafe. Signals:

- A micro-mutation caused a regression that auto-rollback caught, but only after non-trivial damage.
- Aggregate patterns show the autonomous loop is drifting toward worse outcomes despite each individual mutation looking neutral.
- A new dimension of risk appeared (regulatory change, new adversary capability) that invalidates the previous bounded/reversible/measurable demonstration.

Revoking autonomy is not failure of P7; it is P7 working. The system should support fast revocation: a class moves from autonomous back to arbitrated within hours, not weeks. Plan the revocation mechanism when granting autonomy, not after needing it.

## Anti-patterns

**Everything is autonomous.** No arbitration channel. The system mutates freely and nobody can explain its current behavior. This is the "fully autonomous AI" fantasy and it produces systems that work until they catastrophically don't.

**Everything requires arbitration.** Every prompt edit goes to a human. The system cannot keep up with the environment. The human becomes the bottleneck and either burns out or starts rubber-stamping.

**Classification by file type.** "Prompt changes are micro, code changes are macro." Wrong axis. Behavioral impact, not artifact type, determines classification. A prompt change that adds tool access is macro; a code change that adjusts a log format is micro.

**No revocation path.** Once a class is granted autonomy, there is no clean mechanism to revert. The autonomy ratchets only one direction. Eventually the system has too much autonomy and no way to scale it back without disruption.

**Audit log without audit.** Every mutation is logged. Nobody reads the log. The log exists to satisfy a compliance checkbox, not to actually constrain the autonomous loop. Equivalent to no log.
