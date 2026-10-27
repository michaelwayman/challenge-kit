---
name: work-issue
description: Use when a worker session is started to build one GitHub issue to its approved plan, open its PR, and stop.
argument-hint: "<issue-number>"
arguments: [issue]
allowed-tools: Bash(gh issue view *) Bash(gh issue edit *) Bash(gh pr create *) Bash(gh pr view *) Bash(git *) Bash(make *) Bash(ck-github set-status *)
---

# Work one issue

You are a worker. You build issue #$issue in this worktree, open one PR, and stop. The user merges.

!`gh issue view $issue`

## 1. Read
- The issue above (if it didn't load, run `gh issue view $issue`): Goal, `Delivers:`, `Touches:`, `Depends on:`, `Contract:`, `Budget:`, `Estimate:`, any `Grade:`/`Speed:` override, "My reading of", Decisions, acceptance criteria, Out of scope.
- An issue with a **Checklist** (per-milestone issues): build the items in order to their acceptance bullets, one commit per item, ticking each in the issue body as it passes.
- A **bundle** issue (`Bundles:` line): its checklist is your order. Build each bundled issue to its own reading, Decisions and acceptance, one commit per item, ticking the checklist. `Crosses boundaries: no` means `Touches:` stays per service exactly as for single issues.
- `SPEC.md`: the delivered R (acceptance bullets) and the whole Out of scope list.
- The root `AGENTS.md` and the `AGENTS.md` nearest each `Touches:` path. Its Working, Grade, Speed, Review and Learned rules bind you; the issue's override wins over the project's Grade and Speed rules. Build to the grade: no more, no less.
- The contract doc and its `examples/`, if `Contract:` names one.

Stop and message the orchestrator (SendMessage) instead of building if: the issue lacks the `plan-approved` label (unless Speed says `planning: self`), or a `Depends on:` issue is still open.

## 2. Branch
`git fetch origin && git rebase origin/main`. If on main, `git switch -c <issue>-<two-words> --no-track`.

## 3. Decide
Make the how-to decisions inside `Touches:` yourself, and add them as rows to the issue's Decisions table (fetch the body to a scratch file, edit, `gh issue edit $issue --body-file`). Stop and message the orchestrator instead when a decision:
- is a row marked `ask`
- adds a capability `Delivers:` doesn't cover, or touches anything on the Out of scope lists
- changes a contract, or needs a path outside `Touches:`

With `planning: self`, write your one-sentence reading of R<n> into the issue first, then build. If "My reading of R<n>" turns out wrong once you're in the code, stop and say so; don't build your own reading.

## 4. Build and check
- Code and tests together; run the targets you touch while working. If a project skill or `AGENTS.md` names a project-specific skill for this work, use it.
- Commit. Run `make check` once. It must pass.
- `git diff --name-only origin/main...HEAD`: every file must be under a `Touches:` path.
- **Budget**: compare `git diff --stat origin/main...HEAD` totals to `Budget:`. More than 1.5× the lines or files: stop, cut back, or message the orchestrator with the reason and the number. Don't open the PR over budget without the orchestrator's go-ahead.

## 5. Review
Dispatch `challenge-kit:reviewer` and `challenge-kit:contract-checker` in one message, each with the issue number and the base `origin/main`. If the repo has its own `.claude/agents/reviewer.md` or `contract-checker.md`, use those instead (an ejected copy). Fix every finding, or note why you disagree. Re-run `make check` if you changed code.

## 6. Ship
- `git push -u origin HEAD`
- `gh pr create --base main --title "<issue title> (#$issue)" --body-file <file>`, the body following `.github/pull_request_template.md`: `Closes #$issue` (a bundle: also `Closes #n` for every bundled issue), Verification (what you ran, what it printed), Size (budget vs actual, estimate vs actual minutes), Review (each finding and what happened), Calls I made.
- `bind_pr` with the PR, then `set_monitor` with Auto-fix on.
- `ck-github set-status <project-number> <issue-url> "In review"` (project number: SPEC.md header).

## 7. Report and stop
Your final message: the PR link; each acceptance criterion and how it was checked; size vs budget; review findings and outcomes; anything the orchestrator must know (contract or cross-service notes). Then stop. Don't wait for CI; Auto-fix handles it.
