@AGENTS.md

<!-- Claude Code expands the import above when it loads this file; everything
     below is Claude-only. Keep this file even when nothing else is in it: a
     CLAUDE.md that appears later without the import hides AGENTS.md from Claude
     Code. -->

## Claude Code

- End-to-end test — install the plugin into Claude Code and confirm the skill registers:

  ```
  /plugin marketplace add ~/Project/co-rar
  /plugin install co-rar
  ```

  Restart Claude Code, then ask it to "critique this architecture with CO-RAR" — it should announce it is using the `co-rar` skill.
