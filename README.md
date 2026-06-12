# co-rar

**CO-RAR — Continuous Resilient Adversarial Reasoning in Codeless Design**, packaged as a Claude Code plugin.

A frame for designing and critiquing systems where reasoning belongs inside execution, where the system is expected to mutate under environmental pressure rather than break, and where quality is maintained by a continuous adversarial loop rather than by upfront correctness. Think anti-spam/anti-fraud, email warmup and deliverability, growth automation, agentic pipelines, AI-first backends — any system facing an adversary or a moving target that adapts faster than you can ship code.

## What you get

A model-invoked skill (`co-rar`) that Claude applies automatically when designing or reviewing qualifying systems:

- **Task diagnostic** — necessary (N1–N3), sufficient (S1–S4), and anti-conditions (A1–A3) that decide whether a task belongs to the CO-RAR class at all.
- **Seven design principles (P1–P7)** — inversion of control, the 5-axis quality model, backend-as-prompt, headless deep-reasoning sessions, reactive feedback loop, proactive adversarial critic, stratified mutation.
- **Operational metrics** — TtR, ADR, Drift Resistance, Adversarial Coverage, Mutation Cadence.
- **Architecture-review checklist** — eight failure modes of LLM-generated architectures, each mapped to its remedy.
- **Reference playbooks** — feedback loops under delayed/noisy signal, adversarial critic design, micro/macro mutation governance, a four-agent reference topology, anti-pattern catalogue, worked diagnostic examples.

## When it triggers

The skill activates when you ask Claude to design or critique systems where output quality is not contract-checkable, an active adversary or moving target exists, or you're building AI-first backends, agentic pipelines, or codeless workflows meant to adapt under pressure. It explicitly does **not** apply to regulated deterministic domains (finance, medical, legal) where tail-risk dominates flexibility.

## Install

### Option A — local marketplace (one machine)

```bash
git clone https://github.com/umar-s/co-rar ~/Project/co-rar
```

In Claude Code, add the directory as a local marketplace:

```
/plugin marketplace add ~/Project/co-rar
/plugin install co-rar
```

### Option B — direct from GitHub

```
/plugin marketplace add umar-s/co-rar
/plugin install co-rar
```

After install, restart Claude Code so the skill is registered. Verify: ask Claude to "critique this architecture with CO-RAR" — it should announce it is using the `co-rar` skill.

## Layout

This repo is a Claude Code **marketplace** that ships a single plugin. The root holds the marketplace manifest; the plugin itself lives in `plugins/co-rar/`.

```
co-rar/                                        # marketplace root
├── .claude-plugin/marketplace.json            # marketplace catalog
├── README.md
└── plugins/
    └── co-rar/                                # the plugin
        ├── .claude-plugin/plugin.json         # plugin metadata
        └── skills/
            └── co-rar/
                ├── SKILL.md                   # diagnostic, principles P1-P7, metrics
                └── references/
                    ├── diagnostic-examples.md       # worked N/S/A classifications
                    ├── anti-patterns.md             # trained-default architectures + inversions
                    ├── feedback-loops.md            # P5: reactive loop patterns
                    ├── adversarial-critic.md        # P6: proactive critic design
                    ├── mutation-stratification.md   # P7: micro/macro mutation governance
                    └── multi-agent-topology.md      # 4-agent reference implementation
```
