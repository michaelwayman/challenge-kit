---
name: change-spec
description: Use when a requirement should be added, removed, altered or clarified, when something moves into or out of scope, when a technology choice or service boundary changes, or when a worker asks for a capability its issue doesn't cover.
argument-hint: "<the change, in plain words>"
allowed-tools: Bash(gh issue list *) Bash(gh issue view *) Bash(gh pr list *) Bash(gh pr view *) Bash(git status *) Bash(git diff *) Bash(date +%F) Bash(ck-github items *)
---

# Change the spec

The only way SPEC.md changes after kickoff. Request: `$ARGUMENTS`.

## 1. Parse into operations
Restate the request as a list of operations, each on a stable ID:

| Operation | Effect in SPEC.md |
|---|---|
| add R | new `R<next>` with source "user, <date>", priority, milestone, acceptance bullets |
| alter / clarify R | edit its text or acceptance bullets |
| remove R | `Status: removed (<date>, <reason>)`; the entry stays |
| add / alter / remove O | edit the Out of scope list; a removed O is struck through with the date |
| move R → out of scope | remove the R, add an O that says "was R<n>" |
| move O → in scope | add an R that says "was O<n>"; strike the O with "now R<n>" |
| change T or a boundary | edit the row; list every contract and service it touches |
| change priority | edit the R or M priority |

IDs are never renumbered or reused. If the request is ambiguous (which R? narrower or wider?), ask before going further: AskUserQuestion, recommendation first.

## 2. Knowledge check
If an operation needs facts `docs/research.md` doesn't have (a new library, an unexplored part of the source), offer `challenge-kit:research ask "<question>"` (or `research <topic>` for a whole new area) first, and use what it finds. If the change shifts a priority ("we now care most about build speed"), offer `challenge-kit:research steer "<priority>"`: its "If priorities change" and Tensions sections show which T choices should move.

## 3. Impact table
Find what each operation touches:
- issues: `gh issue list --state all --label req:R<n> --json number,title,state,labels,milestone`
- open PRs for those issues: `gh pr list --search "<issue numbers>" --json number,title,headRefName`
- contracts and milestones named by the R or boundary
- Project priority: `ck-github items <project-number>`

Show one table: Operation · Affected (issue / PR / contract / milestone) · Proposed action · Priority. Proposed actions:
- **new R**: new issues (sketched as title + Touches + budget), proposed priority
- **altered R**: open issues get their Goal/acceptance updated, lose `plan-approved`, gain a comment linking the changelog entry; an open PR gets a comment saying what changed and is not edited
- **removed R / moved out**: open issues closed as not planned with a comment; an open PR gets a comment recommending it be closed (the user decides)
- **boundary / T change**: a `contract` issue if any contract changes; affected issues lose `plan-approved`
- **milestone with new work but closed**: reopen

Ask the user to confirm or edit (AskUserQuestion: apply / let me edit). Nothing is written before a yes.

## 4. Apply
1. Branch `spec-<short-slug>` from up-to-date main. Edit SPEC.md (and `docs/contracts/` only for the contract-doc part of a boundary change). Add a changelog row: date, the operations in one line, "change-spec".
2. Commit, push, open a PR "Spec: <summary>". Ask the user to merge it.
3. Once merged, run the `challenge-kit:sync-github` skill. Its diff should match the impact table; if it doesn't, stop and show the difference.
4. If an orchestrator session asked for this change, report the outcome back to it (SendMessage) so it can answer the worker.

## Rules
- Never edit SPEC.md outside this skill's step 4, never without the confirmed impact table.
- Never close a PR or delete an issue. Close issues as not planned; recommend PR closes.
