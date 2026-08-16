# Changelog

All notable changes to this plugin are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versioning follows
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The authoritative version lives in `plugins/co-rar/.claude-plugin/plugin.json`;
every release below is tagged `vX.Y.Z` in git. Installs track the default
branch, so a tag marks history rather than a download.

## [1.0.1] — 2026-08-10

### Changed
- **Context bleed is named as a topology anti-pattern.** Sharing one context
  across agents that are supposed to disagree does not produce independent
  judgements — it produces one judgement restated, which is exactly the failure
  an adversarial loop exists to prevent.
- **Findings that converge independently outrank singletons.** When separate
  agents reach the same conclusion without seeing each other's reasoning, that
  agreement is signal; a single agent's lone finding is a hypothesis.

## [1.0.0] — 2026-06-12

### Added
- Initial release: CO-RAR — Continuous Resilient Adversarial Reasoning in
  Codeless Design, packaged as a Claude Code plugin with a single
  model-invoked skill. A frame for designing and critiquing systems where
  reasoning lives inside execution, the system is expected to mutate under
  environmental pressure instead of breaking, and quality is held by a
  continuous adversarial loop rather than by upfront correctness —
  anti-spam/anti-fraud, warmup and deliverability, agentic pipelines, AI-first
  backends.

[1.0.1]: https://github.com/umar-s/co-rar/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/umar-s/co-rar/releases/tag/v1.0.0
