# CO-RAR Anti-Patterns

The architectures an LLM will reach for when given a CO-RAR-class task without explicit override, paired with the inversion each one needs. Use this as a checklist when reviewing your own or another model's proposed architecture.

## AP1 — The Validator Sandwich

**Shape**: Input → static validator → AI call → static validator → output.

**Why it appears**: Training data is saturated with "guardrail" patterns. The model treats validators as universally virtuous.

**Why it fails on CO-RAR tasks**: The validators encode the designer's incomplete model of "good output." They reject novel-but-correct outputs and accept bland-and-wrong ones. They also prevent the AI from learning, because feedback never reaches the AI — it reaches the validator authors.

**Inversion**: Replace pre-validators with richer prompting (state the constraints to the AI directly). Replace post-validators with a critic-pass model (a second AI call that reviews the first one's output against the goal, not against rules). If a validator is truly necessary (audit, hard safety), make it visible to the producing AI in its prompt so the AI does not waste cycles producing what will be rejected.

## AP2 — AI as Garnish

**Shape**: A largely deterministic system with one AI call somewhere in the middle, whose output is either ignored, heavily post-processed, or used only for display.

**Why it appears**: The model defaults to "AI for the part that humans would call creative; code for everything else." This bisects the system in the wrong place.

**Diagnostic**: If you removed the AI call entirely and replaced it with a fixed string, would the system still mostly work? If yes, the AI is garnish.

**Inversion**: Move the AI to the orchestration layer. The AI decides which sub-tasks to run, in what order, with what parameters. Code provides the capabilities (tools), not the control flow.

## AP3 — The Pipeline of Tiny LLM Calls

**Shape**: Stage 1 LLM call → stage 2 LLM call → stage 3 LLM call, each stateless, each with its own narrow prompt.

**Why it appears**: This mirrors the microservices pattern from web development, which the model learned exhaustively.

**Why it fails**: Context fragments at every stage boundary. Each stage re-derives what the previous stage already knew. Errors compound and become undebuggable because the failure is in the seams between stages.

**Inversion**: Collapse to one long-session reasoning call with tool access. The same work, done with full cross-stage context, is usually cheaper and substantially better. Pipelines remain appropriate when stages have genuinely different SLAs (real-time vs batch) or different access rights.

## AP4 — Temperature Theater

**Shape**: Quality is poor. The proposed fix is to lower temperature ("make it more deterministic") or raise it ("make it more creative").

**Why it appears**: Temperature is the most visible knob and the most-discussed in training data.

**Why it fails**: Temperature is one of five axes (P2). Tuning it in isolation typically wastes the budget that should have gone to model choice or layering.

**Inversion**: When quality is poor, audit all five axes before touching temperature. The usual real fix is a stronger model and/or a critic pass, not a different temperature.

## AP5 — The Dictionary of Forbidden Things

**Shape**: A growing list of banned words, banned patterns, banned tones, maintained by adding to it every time a bad output is observed.

**Why it appears**: This is how anti-spam was built in 1998 and the pattern is everywhere in training data.

**Why it fails on CO-RAR adversarial tasks**: Adversaries adapt. Static word lists are a defender's tool against static attackers. When the adversary is the platform's classifier, the list reflects yesterday's signal and is irrelevant today.

**Inversion**: Replace the list with a current-adversary-model prompt. "Recent observation: classifier is flagging messages with X structural pattern; avoid that pattern this week." Update the prompt from telemetry (P5), not from a growing append-only file.

## AP6 — Single-Pass Production

**Shape**: One prompt, one output, ship it.

**Why it appears**: Token cost and latency optics. Also: the cheap path looks engineering-virtuous.

**Why it fails on quality-dominated tasks**: The first draft of anything is rarely the best draft. A second pass that critiques the first, or a parallel generation that selects the best of N, almost always beats single-pass.

**Inversion**: Default to at least: generate → critique → revise. For high-stakes outputs, generate N candidates and have a separate model pick the best. The token cost is real; the quality gain almost always exceeds it on CO-RAR-class work.

## AP7 — The Cron Job Mind

**Shape**: Work is split into small jobs that run on schedules, each one stateless, each one re-establishing context from scratch.

**Why it appears**: This is how batch processing was built for decades.

**Why it fails**: Cross-job awareness is impossible. Job N cannot know what job N-1 just decided. The system cannot adapt mid-batch when telemetry shifts.

**Inversion**: For non-interactive CO-RAR work, run a long-session agent that handles the batch as a coherent task with internal memory. Cron triggers the session; the session does the thinking. See P4.

## AP8 — Feedback Loop Theater

**Shape**: The system has dashboards, metrics, logs. None of them are connected to anything that changes the system's behavior.

**Why it appears**: Observability is well-trained-on; closed-loop adaptation is not.

**Why it fails**: P5 is violated. The system degrades silently while looking healthy on dashboards.

**Inversion**: For every metric, specify the action it triggers when it moves. If the answer is "a human looks at it eventually," the loop is open. Close it: either make the action automatic (re-tune prompt, swap model), or make the metric trigger a notification that demands action within a defined SLA.

## AP9 — Model Defaulting

**Shape**: The cheapest available model is used for everything, on the assumption that "we can upgrade later if needed."

**Why it appears**: Cost-consciousness is a trained virtue.

**Why it fails on quality-dominated tasks**: Quality floor is set by model capability. A weak model with a great prompt is usually worse than a strong model with a mediocre prompt. The cost saved is illusory if the output is unusable.

**Inversion**: Start with the strongest model available, measure quality, then probe whether a weaker model meets the bar. Default-up, not default-down, for CO-RAR-class work.

## AP10 — Schema Worship

**Shape**: The AI is forced to output rigid JSON conforming to a schema designed before the task was understood.

**Why it appears**: Structured outputs are a well-trained pattern and APIs love schemas.

**Why it fails**: The schema constrains what the AI can express. Information that does not fit a field is lost. The schema designer has substituted their pre-task model of the output for the AI's in-task understanding.

**Inversion**: For CO-RAR-class internal reasoning, let the AI produce natural-language artifacts. Convert to structured form only at the boundary where structure is genuinely required (database write, API call). Often the conversion is itself an AI task.
