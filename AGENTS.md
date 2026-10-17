# AGENTS.md

This repository's rules for agents of both harnesses. Codex reads only this
file; Claude Code reads it through the `@AGENTS.md` import on the first line of
`CLAUDE.md`, which holds only Claude-specific notes.

## What this repository is

A **plugin marketplace** (Claude Code and Codex CLI) shipping a single plugin, `co-rar` — a model-invoked skill ("CO-RAR — Continuous Resilient Adversarial Reasoning in Codeless Design"). There is no application code, build system, linter, or test suite. Everything is Markdown content plus two JSON manifests.

## Verification commands

Validate manifests after editing:

```bash
jq . .claude-plugin/marketplace.json
jq . plugins/co-rar/.claude-plugin/plugin.json
```

End-to-end test in Codex, without a model call — `codex debug prompt-input` renders the prompt the model would get, skill list included. The subshell's own `CODEX_HOME` keeps the user's Codex untouched; the prompt is captured first, so a failing `codex` fails the check instead of hiding behind a pipe:

```bash
(
  set -euo pipefail
  CODEX_HOME="$(mktemp -d)"   # assigned apart from export: `export X=$(…)` returns export's status
  export CODEX_HOME
  trap 'rm -rf "$CODEX_HOME"' EXIT
  codex plugin marketplace add ~/Project/co-rar
  codex plugin add co-rar@co-rar
  prompt="$(codex debug prompt-input x </dev/null)"
  grep -q 'co-rar:co-rar' <<<"$prompt" && echo 'OK: co-rar:co-rar is in the skill list'
)
```

## Structure and how the pieces relate

Two-level layout: the repo root is the marketplace (`.claude-plugin/marketplace.json` is the catalog); the plugin lives in `plugins/co-rar/` with its own `.claude-plugin/plugin.json`. The skill content is under `plugins/co-rar/skills/co-rar/`.

- **`SKILL.md`** is the entry point and the core of the frame: the task diagnostic (necessary conditions N1–N3, sufficient S1–S4, anti-conditions A1–A3), the seven design principles P1–P7, operational metrics (TtR, ADR, Drift Resistance, Adversarial Coverage, Mutation Cadence), and the architecture-review checklist of eight failure modes.
- **`references/`** are progressive-disclosure playbooks, each expanding a specific part of SKILL.md: `feedback-loops.md` → P5, `adversarial-critic.md` → P6, `mutation-stratification.md` → P7, `anti-patterns.md` → the failure-mode catalogue (AP1, AP2, …), `diagnostic-examples.md` → worked N/S/A classifications, `multi-agent-topology.md` → a four-agent reference implementation (Orchestrator, Critic, Builder, Auditor).

## Editing rules that follow from this structure

- **The skill's trigger surface is the `description` field** in SKILL.md frontmatter — that text alone decides when the model auto-invokes the skill. Edit it deliberately; keep the trigger phrases and the explicit "Do NOT use for regulated deterministic domains" exclusion.
- **Identifiers (N1–N3, S1–S4, A1–A3, P1–P7, AP1–…) are cross-referenced** across SKILL.md, every file in `references/`, and README.md. Renumbering or renaming one requires updating all references.
- **The version lives only in `plugin.json`** — the single source of truth; `marketplace.json` carries no versions (knowledge-work-plugins convention). The plugin is also listed in the umbrella marketplace github.com/umar-s/devpowers, which references this repo via `git-subdir` with `ref: main` — no release step is needed there.
- **README.md mirrors the skill**: its "What you get" / "When it triggers" sections and the layout tree must stay consistent with SKILL.md and the actual file structure when content changes.
- All content is written in English, prose-heavy, with the principle/anti-pattern format used in existing files (bold shape/why/inversion pattern in `anti-patterns.md`, ID-headed sections elsewhere). Match it when adding material.
