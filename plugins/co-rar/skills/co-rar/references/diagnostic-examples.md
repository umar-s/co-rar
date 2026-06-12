# Diagnostic Examples

Worked examples of applying the CO-RAR diagnostic to real tasks. Use these as calibration when classifying a new task.

## Example 1: Email warmup copywriting (CO-RAR-class, canonical)

**Task**: Generate unique conversational emails for 1000 mailboxes warming up against Gmail/Outlook reputation systems. 20-30k emails/day, must look human, must vary in tone/timing/depth, must avoid spam triggers, must adapt as classifiers change.

**Diagnostic**:
- N1 ✓ — "Good email" is not codable. Verified by deliverability and reply rate.
- N2 ✓ — Every email needs context-aware composition.
- N3 ✓ — Dictionary-based generators produce garbage that classifiers eat.
- S1 ✓ — Gmail/Outlook are active adversaries with weekly classifier updates.
- S2 ✓ — Warmup tradecraft is not in training data.
- S3 ✓ — Long tail of conversation shapes.
- Anti-conditions: none. Failed emails are cheap individually; system-level failure is recoverable.

**Verdict**: CO-RAR-class.

**Trained-default architecture (wrong)**: Python orchestrator with template library, dictionary of safe words, regex spam-trigger filter, single GPT call to "rewrite this template more naturally," output passed through validators.

**CO-RAR architecture**: Long-session reasoning agent per mailbox, given: mailbox persona, recent conversation history, time-of-day, deliverability telemetry from the last 24h. Agent decides what to send, when, to whom, in what tone. Tools: send_email, check_inbox, mark_read. No validators; instead, a separate critic-pass model reviews drafts against the current adversary hypothesis. Telemetry feeds back daily into the adversary hypothesis prompt.

## Example 2: Invoice line-item extraction from PDFs (NOT CO-RAR-class)

**Task**: Extract vendor, date, line items, totals from incoming invoice PDFs and post to accounting system.

**Diagnostic**:
- N1 ✗ — Quality IS contract-checkable. The totals must reconcile, the vendor must exist in the vendor table, the date must parse.
- Even if N1 were borderline, this task lives near anti-condition A1 (accounting requires auditability).

**Verdict**: Not CO-RAR-class. Conventional architecture is correct: LLM extracts structured fields, deterministic code validates against vendor table and arithmetic, exceptions queued for human review.

This is the case where the trained default is right. Do not over-apply CO-RAR.

## Example 3: Customer support triage and response (mixed)

**Task**: Handle inbound support tickets — classify, respond to simple ones, escalate hard ones.

**Diagnostic**:
- Classification sub-task: N1 borderline (categories are defined, but boundary cases are fuzzy). N2 ✓. N3 ✓.
- Response sub-task: N1 ✓ (quality is felt, not measured). N2 ✓. N3 ✓.
- S3 ✓ for both.
- A1 partial — some responses (refund commitments, legal statements) have audit requirements.

**Verdict**: Mixed. The response generation is CO-RAR-class; the audit-bound subset is not. Split the system: CO-RAR-shaped reasoning agent handles the main flow; a deterministic gate routes audit-bound categories (refunds above $X, anything mentioning lawsuit/lawyer/regulator) to a separate path with human approval.

This is the common case: most real systems are mixed. The skill is to identify which sub-tasks belong in which regime, not to force the whole system into one.

## Example 4: Trading-strategy execution (NOT CO-RAR-class, despite appearances)

**Task**: Execute a trading strategy that reacts to market conditions.

**Surface appearance**: adversarial (other traders), moving target (market), fuzzy quality (P&L).

**Diagnostic**:
- Anti-condition A2 dominates: a bad trade can cause irrecoverable loss. The flexibility gain from reasoning-in-execution does not justify the tail risk of a model hallucinating a 100x position size.
- The strategy *design* may benefit from CO-RAR-style reasoning (research, backtesting, hypothesis generation). The *execution* must remain deterministic with hard limits.

**Verdict**: Split. Reasoning informs strategy, deterministic code executes trades within hard guardrails. The guardrails here are not the trained-default carg-cult; they are insurance against catastrophic tail.

## Example 5: Code review of pull requests (CO-RAR-class)

**Task**: Review incoming PRs for style, bugs, design issues.

**Diagnostic**:
- N1 ✓ — "Good review" is not codable.
- N2 ✓ — Each PR needs context.
- N3 ✓ — Linters and static analyzers catch a fraction; the rest needs judgment.
- S3 ✓ — Long tail of code patterns.
- S2 partial — codebase-specific conventions are not in training data.
- Anti-conditions: none.

**Verdict**: CO-RAR-class.

**Trained-default (wrong)**: Run linter, run security scanner, post results, optionally call AI to "summarize" — AI as decoration.

**CO-RAR architecture**: Long-session reasoning agent with access to the full repo, the PR diff, recent commit history, related issues. Agent decides what to comment on, what to investigate further, what to approve. No upfront rule about "always comment on functions over 50 lines" — that is the validator-stack anti-pattern.
