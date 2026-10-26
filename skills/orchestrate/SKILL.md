---
name: orchestrate
description: Use to run the build after kickoff - plan rounds of issues for approval, start worker sessions in parallel lanes, answer workers, and catch up after merges. Also for "stop #n" or "redo #n".
argument-hint: "[milestone ... | stop <n> | redo <n>]"
disable-model-invocation: true
allowed-tools: Bash(gh issue *) Bash(gh pr list *) Bash(gh pr view *) Bash(gh api *) Bash(ck-github *) Bash(git fetch *) Bash(git log *) Agent
---

# Orchestrate

You are the orchestrator. You plan, start workers, answer them, and keep GitHub current. You never build service code, never merge, never push to main, never poll CI. Arguments: `$ARGUMENTS`. Milestones: the ones named, else every open milestone. Project number: SPEC.md's header line.

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

## 2. Plan a round
Candidates: `ready`, not `plan-approved`, not `needs-triage`, no open dependency, in the chosen milestones.

**Order**: Priority P0, then P1, then P2 (Project field). Ties: the issue that unblocks the most others, then the lowest number. A blocked issue never starts, whatever its priority.

For the top candidates of each lane (one lane per milestone; up to 2 per lane), dispatch read-only `Plan` agents (model sonnet), all in one message. Each returns, for its issue: Decisions rows (question, options, chosen, why); one sentence **"My reading of R<n>"**; a **Budget** (files, lines); anything outside `Delivers:` it would need; what it deliberately leaves out.

Show **one table per lane**, in pick order: # · Issue · Priority · My reading of R<n> · Key decisions · Budget · Needs outside scope. Ask the user (AskUserQuestion): approve all / change some. Route "needs outside scope" items to `challenge-kit:change-spec`, not into the plan. For approved issues: write the rows, reading and budget into the body (body-file round trip), add `plan-approved`.

## 3. Run lanes
For each lane, start the next approved issue when the lane has a free slot (max 2) and the two issues' `Touches:` don't overlap and neither depends on the other. Start it with `start_session`: initiation "user_asked", context "fresh", use_worktree true, branch name `<n>-<two-words>`, brief starting `/challenge-kit:work-issue <n>`. Set Status In progress (`ck-github set-status`).

## 4. When a worker reports
1. Check its PR: `gh pr view --json number,files,statusCheckRollup`. Read its full final message.
2. Run the `challenge-kit:review-pr` skill for the PR.
3. Start the lane's next issue.
4. Answer questions, by kind (SendMessage to the worker):
   - how to build inside its service → "decide, log it under Calls I made"
   - new capability, out-of-scope item, or a misread requirement → `challenge-kit:change-spec`; tell the worker to pause or stop
   - contract change → open a `contract` issue (additive and needed by an approved plan: you build it; otherwise `needs-triage` and ask the user)
   - over budget → ask the user: raise the budget or cut back
   - marked `ask` or crossing a boundary → AskUserQuestion, then answer
5. Tell the user which PRs are ready to merge, and to say "I merged #n" (or rerun `/challenge-kit:orchestrate`) after merging: nothing wakes you on a merge.

## Rules
- New issues you open get `needs-triage`, never `ready`.
- Never edit SPEC.md or contracts directly; change-spec and contract issues do.
