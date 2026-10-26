---
name: orchestrate
description: Use to run the build after kickoff - plan rounds of issues for approval, start worker sessions in parallel lanes, answer workers, and catch up after merges. Also for "stop #n" or "redo #n".
argument-hint: "[milestone ... | stop <n> | redo <n>]"
disable-model-invocation: true
allowed-tools: Bash(gh issue *) Bash(gh pr list *) Bash(gh pr view *) Bash(gh api *) Bash(ck-github *) Bash(git fetch *) Bash(git log *) Bash(date *) Agent
---

# Orchestrate

You are the orchestrator. You plan, start workers, answer them, and keep GitHub current. You never build service code, never merge, never push to main, never poll CI. Arguments: `$ARGUMENTS`. Milestones: the ones named, else every open milestone. Project number and Progress log issue: SPEC.md's header line. Time box, grade and Speed settings: SPEC.md "Time box and grade"; definitions in `${CLAUDE_PLUGIN_ROOT}/rules/grades.md`. An issue's `Grade:`/`Speed:` override wins over the project's.

## stop / redo
- `stop <n>`: find the worker session for #n (linked sessions list) and stop it. Leave its branch. Say what it had done.
- `redo <n>`: stop it if running, ask what was wrong (or take it from the user's words), rewrite the issue's "My reading of" / Decisions / Budget accordingly, remove `plan-approved`, and put it in the next round. If the fault is in the requirement itself, run `challenge-kit:change-spec` instead.

Otherwise, run these steps; run them again every time you wake.

## 1. Catch up
- `gh issue list --state all --limit 1000 --json number,state,labels,milestone,body` and `ck-github items <project>`.
- An issue labelled `blocked` whose `Depends on:` are all closed → swap `blocked` for `ready`; Status Ready.
- An issue with neither `ready`, `blocked` nor `needs-triage` (the user just triaged it) → `ready` or `blocked` by its dependencies.
- Status: Done for closed issues; In review for issues with an open PR; never move an item backwards by hand-set status.
- An empty open milestone → close it (`gh api -X PATCH repos/<repo>/milestones/<n> -f state=closed`).
- `needs-triage` issues are the user's; leave them.
- `bundled` issues are built through their bundle; never a candidate on their own.
- For each issue that closed since the last wake: `ck-github log <log> "done #n · R<n> · estimate <m> · actual <m> · ratio <r>"` (actual: its `started` comment to its PR's createdAt).

## 1b. Clock check
`date -u`, then per rules/grades.md § Estimates: elapsed, time left, work left, calibration factor (update SPEC.md's factor through a replan PR only when it moved more than 20%; otherwise just use it).
- Fits: show one line, `2h10m elapsed · 1h30m left · ~1h05m of work`, and `ck-github log` it.
- Doesn't fit: show the gap and run `challenge-kit:replan` (it offers process levers before cuts). Plan nothing new until it's done or the user says carry on.
- Past 75% of the time box: start no `stretch` issue. Only the reserve left: start nothing; recommend `/challenge-kit:integrate`. Log each of these once.

## 2. Plan a round
Candidates: `ready`, not `plan-approved`, not `needs-triage`, not `bundled`, no open dependency, in the chosen milestones. A bundle is one candidate.

**Order**: Priority P0, then P1, then P2 (Project field); `stretch` last. Ties: the issue that unblocks the most others, then the lowest number. A blocked issue never starts, whatever its priority.

Speed settings change this step: `planning: batch` plans every candidate of the milestone in one round; `planning: self` skips the Plan agents (the round table shows the issue's own reading, and the worker records its decisions in the PR); `auto-approve: S` writes S plans without asking and shows them as approved.

For the top candidates of each lane (one lane per milestone; up to the lane's slots), dispatch read-only `Plan` agents (model sonnet), all in one message. Each returns, for its issue: Decisions rows (question, options, chosen, why); one sentence **"My reading of R<n>"**; a **Budget** (files, lines, times the budget-factor if set); a size S/M/L; anything outside `Delivers:` it would need; what it deliberately leaves out.

Show **one table per lane**, in pick order: # · Issue · Priority · My reading of R<n> · Key decisions · Budget · Estimate · Needs outside scope. Ask the user (AskUserQuestion): approve all / change some. Route "needs outside scope" items to `challenge-kit:change-spec`, not into the plan. For approved issues: write the rows, reading, budget and Estimate into the body (body-file round trip), add `plan-approved`.

## 3. Run lanes
For each lane, start the next approved issue when the lane has a free slot (2, or `slots-per-lane`) and the two issues' `Touches:` don't overlap and neither depends on the other. Start it with `start_session`: initiation "user_asked", context "fresh", use_worktree true, branch name `<n>-<two-words>`, brief starting `/challenge-kit:work-issue <n>`. Set Status In progress (`ck-github set-status`) and comment `started <UTC>` on the issue (`gh issue comment`): calibration measures from it.

## 4. When a worker reports
1. Check its PR: `gh pr view --json number,files,statusCheckRollup`. Read its full final message.
2. Run the `challenge-kit:review-pr` skill for the PR (with `review: single`, only when the worker's reviewer reported gaps).
3. Start the lane's next issue.
4. Answer questions, by kind (SendMessage to the worker):
   - how to build inside its service → "decide, log it under Calls I made"
   - new capability, out-of-scope item, or a misread requirement → `challenge-kit:change-spec`; tell the worker to pause or stop
   - contract change → open a `contract` issue (additive and needed by an approved plan: you build it; otherwise `needs-triage` and ask the user)
   - over budget → ask the user: raise the budget or cut back; if the overrun breaks the fit, `challenge-kit:replan`
   - marked `ask` or crossing a boundary → AskUserQuestion, then answer
5. Tell the user which PRs are ready to merge, and to say "I merged #n" (or rerun `/challenge-kit:orchestrate`) after merging: nothing wakes you on a merge.

## Rules
- New issues you open get `needs-triage`, never `ready`.
- Never edit SPEC.md or contracts directly; change-spec and contract issues do.
