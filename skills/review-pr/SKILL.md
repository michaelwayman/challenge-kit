---
name: review-pr
description: Use when a worker's PR has opened and needs an independent review against its issue, SPEC.md, contracts and the repo's rules before the user reviews it.
argument-hint: "<pr-number>"
arguments: [pr]
allowed-tools: Bash(gh pr view *) Bash(gh pr diff *) Bash(gh pr checks *) Bash(gh pr comment *) Bash(gh issue view *)
---

# Review a PR

Independent second review of PR #$pr, on the real diff and CI, posted for the user to read before they review.

1. `gh pr view $pr --json number,title,body,closingIssuesReferences,headRefName` → the issue number (from "Closes #").
2. Dispatch `challenge-kit:reviewer` and `challenge-kit:contract-checker` in one message, each with the issue number and PR #$pr (not a base ref). Use the repo's `.claude/agents/` copies if they exist.
3. Compare with the PR's Review section: findings the worker already fixed or answered are dropped; the rest stand.
4. Post one comment with `gh pr comment $pr --body-file <file>`:

   ```
   ## challenge-kit review
   Reviewer: <verdict> · Contracts: <verdict> · CI: <passing / failing / pending>

   <findings table, only the ones that stand; "None" if empty>

   Size: <actual> vs <budget>
   ```
5. Report the verdicts and finding count to whoever ran you. Auto-fix on the worker's session picks up the comment; don't message the worker yourself.
