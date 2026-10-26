---
name: replan
description: Use when the deadline, available hours or quality grade of a running project changes ("the deadline is tomorrow", "I only have 2 more hours", "make R3 demo grade"), when the orchestrator's clock check finds the remaining work no longer fits, or when the user wants the build to go faster for all or part of the project.
argument-hint: "[R<n> | M<n> | #<n> ...] \"<the change, in plain words>\""
allowed-tools: Bash(gh issue list *) Bash(gh issue view *) Bash(gh pr list *) Bash(gh api *) Bash(git status *) Bash(git diff *) Bash(date *) Bash(ck-github items *) Bash(ck-github log *)
---

# Replan

Fit the remaining work into the time the user has, for the whole project or part of it. Request: `$ARGUMENTS`. Definitions (grades, estimates, reserve, levers): `${CLAUDE_PLUGIN_ROOT}/rules/grades.md`.

**Scope**: R IDs, milestones or issue numbers before the quoted change. With a scope, grade and lever changes apply only there, as overrides; without one, they change the project defaults. The fit is always computed for the whole project.

## 1. Read state
- SPEC.md: "Time box and grade", every active R with its Estimate and overrides, Milestones.
- `gh issue list --state all --limit 1000 --json number,title,state,labels,milestone,body,comments`, `gh pr list --state open --json number,title,closingIssuesReferences`, `ck-github items <project>`.
- No "Time box and grade" section (kicked off before this existed): propose one from the user's words, and estimate every open issue (S/M/L) with read-only `Plan` agents, one per milestone, in one message.

## 2. Pin down the time
Turn the user's words into a deadline (`date -u`). If the deadline is more than one working session away ("tomorrow", "Friday"), ask how many working hours are available before it: wall-clock overnight is not build time. Record deadline and available hours.

## 3. Fit
Per rules/grades.md § Estimates: work left vs time left, using the current calibration factor and lanes. Show: Time left · Work left · Gap, and the open issues in scope with Priority · Estimate · Status (in-flight issues count but are not replanned).

## 4. Levers
Show the speed lever table from rules/grades.md, each row with its saving for **this** project (count the rounds, S issues, PRs and issues it touches). Ask (AskUserQuestion, multiSelect) which to use; recommend the ones that close the gap without lowering grade or scope. Re-run the fit with the picks.

Still a gap: propose the next levers, then scope (stretch first, then cut, lowest priority and least depended-on first). Scope and grade change only if the user picks them. If the user would rather keep everything, offer a later deadline or more hours instead.

## 5. Confirm
One impact table: Change · Applies to (project / R / issues) · Effect · Saving. Include settings changed, issues re-budgeted, bundles proposed, Rs moved to stretch or cut, the new fit. Ask: apply / let me edit. Nothing is written before a yes.

## 6. Apply
1. Branch `replan-<short-slug>` from up-to-date main. In SPEC.md: update "Time box and grade" (project scope) or add `Grade: <g> (override)` / `Speed: <levers>` lines to the scoped Rs. Cuts and stretch moves follow `challenge-kit:change-spec`'s operations (stretch = Priority P2 plus listing in Stretch). A changelog row: date, the change in one line, "replan".
2. If grade or speed defaults changed, re-render AGENTS.md's "Grade rules" and "Speed rules" from rules/grades.md.
3. Commit, push, open a PR "Replan: <summary>". Ask the user to merge it.
4. After merge: run `challenge-kit:sync-github` (cuts, priorities, `stretch` labels), then update open issue bodies in scope: Budget × budget-factor, Estimate, override lines; remove `plan-approved` from issues whose plan changed. If "Bundle tickets" was picked, run `challenge-kit:bundle` with the scope.
5. `ck-github log <log-issue> "replan · <user's words> · scope <…> · fit before <…> after <…> · levers <…> · cut <…> · stretch <…> · <PR url>"`.
6. If an orchestrator session asked for this, report back to it (SendMessage).

## Rules
- Process levers before grade, grade before scope, unless the user picks otherwise.
- Never touch an issue with an open PR beyond a comment.
