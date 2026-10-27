---
name: report
description: Use when the user reports a bug or asks for a small fix in work that is already merged (or built), in plain words. Files one already-triaged issue per service. New capabilities go to change-spec instead.
argument-hint: "<the bug or fix, in plain words> [#n | PR #n]"
allowed-tools: Bash(gh issue list *) Bash(gh issue view *) Bash(gh issue create *) Bash(gh pr view *) Bash(gh api *) Bash(git ls-files *) Bash(git log *) Bash(git grep *) Bash(ck-github add-item *) Bash(ck-github set-status *) Bash(ck-github reopen-milestone *) Bash(ck-github log *) Bash(ck-github items *) Bash(date *)
---

# Report a bug or small fix

Turn the user's report into a triaged issue the orchestrator can pick up. Report: `$ARGUMENTS`. The user reporting it is the triage: the issue is `ready` or `blocked`, never `needs-triage`. `ck-github` is on PATH from the plugin's `bin/` (fall back to `${CLAUDE_PLUGIN_ROOT}/bin/ck-github`).

## 1. Classify
Read SPEC.md: every R with its acceptance bullets, the Out of scope list and the contracts.
- **Bug or small fix**: an active R's acceptance is broken, or there is a gap inside what that R already promises (a crash, a wrong value, missing error handling, an a11y gap, hardening of an existing flow). Continue.
- **New capability**: it goes past every R's acceptance, touches an O item, or changes a contract's shape. Stop, say why in one line, and give the exact `/challenge-kit:change-spec "<drafted change>"` to run. File nothing.
- **A removed R**: say so and stop.
- Unsure: AskUserQuestion, recommendation first.

## 2. Locate
- The R, and the service from SPEC.md's Services table.
- The paths: `git ls-files` and `git grep` on the names in the report; note file:line where the fault shows.
- The issue that built it: `gh issue list --state all --label req:R<n> --json number,title,state,body`, matched by title and `Touches:`; the PR if the user named one (`gh pr view`).
- A fix that spans two services is two issues, one per service; say so.
- The R or service is ambiguous: AskUserQuestion.

## 3. Draft
From `${CLAUDE_PLUGIN_ROOT}/templates/issue-task.md`:
- First line `<!-- ck:report:<slug> -->` (kebab-case, unique). Title `<service>: <what's wrong>`. No Checklist.
- Goal, then a `## Why` section: what happens, how to reproduce it, the file:line from step 2.
- `Delivers: R<n> (follow-up to #<n>)` · `Touches:` inside the one service · `Depends on:` the original issue if still open, plus any open issue whose `Touches:` overlaps, else `none` · `Contract:` · `Budget:` · `Estimate: S` unless clearly bigger.
- "My reading of R<n>", an empty Decisions table, acceptance criteria that start with "a test that reproduces it fails before the fix and passes after" and end with "`make check` passes", and Out of scope.
- Labels `service:<s>`, `req:R<n>`, and `ready` (no open dependency) or `blocked`. Milestone: the R's milestone in SPEC.md. Priority: the R's priority.

## 4. Confirm
Show title, labels, milestone, Priority and the body. AskUserQuestion: file it (Recommended) / let me edit. Apply edits and show it again. Nothing is filed before a yes.

## 5. File
1. `gh api "repos/{owner}/{repo}/milestones?state=all" --paginate`; if the milestone is closed, `ck-github reopen-milestone "<title>"`.
2. `gh issue create --title "<title>" --body-file <file in your scratchpad> --milestone "<title>" --label <labels>`.
3. If SPEC.md's header has a Project number: `ck-github add-item <project> <issue-url> <P>`, then `ck-github set-status <project> <issue-url> <Ready|Todo>` (Todo when `blocked`).
4. If it has a Progress log issue: `ck-github log <log-issue> "report #<n> · R<n> · <service> · <ready|blocked>"`.

A failed step is reported with its error, never retried silently.

## 6. Report back
The issue link, its labels and status. Next step: the orchestrator plans it in its next round, and the plan still comes to the user for approval. Run `/challenge-kit:orchestrate`, or tell the orchestrator session "I reported #<n>".

## Rules
- Never edit SPEC.md or contracts; that is change-spec's job.
- One issue per service. Never `needs-triage`.
