---
name: record-correction
description: Use when the user corrects an agent's output or behavior (a worker, the reviewer, the orchestrator, any challenge-kit skill) and the fix should apply to future work, not just this instance.
argument-hint: "<what went wrong>"
allowed-tools: Bash(date +%F) Bash(git ls-files *) Bash(git log *)
---

# Record a correction

Turn feedback into a rule so it doesn't recur. Feedback: `$ARGUMENTS`.

## 1. State it
One sentence: what the agent did, what it should have done, and the trigger (issue/PR number). Confirm with the user if you had to guess any part.

## 2. Route it
| The feedback is about | Goes to |
|---|---|
| this project's code, data, conventions or domain ("we always use the ORM", "the API must never return X") | the repo's `AGENTS.md` → **Learned rules** (or the `AGENTS.md` of the one service it concerns) |
| what reviewers must check in this project ("flag views without the permission class") | the repo's `AGENTS.md` → **Review rules** |
| how a challenge-kit skill or agent behaves on any project ("the reviewer is too noisy", "kickoff asked too many questions", "workers should never …") | a patch to the plugin: `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md`, `agents/<name>.md` or `rules/working-rules.md` |

Unsure → ask: "Only this project, or every project that uses challenge-kit?"

## 3. Write the rule
- General enough to cover the next case, specific enough to check: a rule a reviewer could find broken in a diff.
- Prefer a mechanism over prose when one fits: a test, a lint rule, a permission rule in `.claude/settings.json`. Propose it alongside the rule.
- A near-duplicate exists → amend it instead of adding one.

Show the exact text and target file. Edit only after the user says yes.

## 4. Apply
- **Project**: edit the `AGENTS.md` section (never a `CLAUDE.md` symlink path), appending `(<date>, #<n>)`. Leave it uncommitted and say so.
- **Plugin**: edit the file in `${CLAUDE_PLUGIN_ROOT}`. Say that it's a change to the plugin's own repo, that it applies to new sessions after `/reload-plugins` (or a plugin update for cached installs), and that the user should commit it there. If the project has ejected copies in `.claude/`, offer to apply the same edit there.
