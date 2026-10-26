---
name: eject
description: Use when the project's repo should contain its own visible copy of the challenge-kit skills and agents, for example so reviewers or graders can see the AI tooling, or to customize them for this project only.
disable-model-invocation: true
allowed-tools: Bash(ls *) Bash(git status *) Bash(diff *)
---

# Eject

Copy the plugin's skills and agents into this repo's `.claude/` as a snapshot.

1. List what will be copied: every `${CLAUDE_PLUGIN_ROOT}/skills/*/SKILL.md` → `.claude/skills/ck-<name>/SKILL.md`, every `${CLAUDE_PLUGIN_ROOT}/agents/*.md` → `.claude/agents/<name>.md`, plus `bin/ck-github` → `.claude/bin/ck-github`. Show any that already exist and differ (`diff`).
2. Ask the user to confirm. Note the trade-off: ejected copies don't receive plugin updates; `record-correction` will offer to patch both.
3. Copy. In the copies, replace `${CLAUDE_PLUGIN_ROOT}` with the plugin's absolute path for templates and rules, `ck-github` with `.claude/bin/ck-github`, and `challenge-kit:<agent>` with `<agent>`. Skills get a `ck-` prefix so they don't collide with the plugin's `/challenge-kit:<name>`.
4. Add a line to `AGENTS.md` under the intro: "Ejected challenge-kit <version> on <date>: `.claude/skills/ck-*`, `.claude/agents/`. Workers use these copies."
5. Leave it uncommitted; tell the user what changed.

Workers and `review-pr` already prefer `.claude/agents/reviewer.md` and `contract-checker.md` when they exist.
