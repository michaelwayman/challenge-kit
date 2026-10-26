---
name: bundle
description: Use when the user wants a lot of planned work shipped in a few PRs, such as one PR per service, or when kickoff's delivery shape is "few big PRs" or replan picked "Bundle tickets". Also for "bundle undo <n>".
argument-hint: "[service:<s> | M<n> | #<n> ... | undo <n>]"
allowed-tools: Bash(gh issue *) Bash(gh pr list *) Bash(gh api *) Bash(ck-github *)
---

# Bundle issues into one PR

Link several planned issues under one bundle issue that one worker builds in one PR. The original issues, their readings and Decisions stay as they are. Arguments: `$ARGUMENTS`.

## undo <n>
Close bundle #n as not planned with a comment; remove `bundled` from each issue in its `Bundles:` line and set their Status back (Ready or Todo by dependencies). `ck-github log` the undo. Refuse if #n has an open PR; say so.

## 1. Propose
Candidates: open issues in the scope (all open issues if none) without an open PR and without `bundled`. Read SPEC.md's Services table and the issues' `Touches:`, `Depends on:`, Budget and Estimate.

Default grouping: one bundle per service. Integration (M1) issues and `contract` issues stay separate: everything depends on them. Show: Bundle · Issues · Rs delivered · Touches · Summed budget · Estimate · Depends on (outside the bundle).

## 2. Ask (AskUserQuestion, up to 4 questions, recommendation first)
- **Service boundaries**: may any bundle cross a service boundary? Recommended: **no**; contracts stay the only agreement between bundles, and each PR stays reviewable against one side. If yes: which services merge. The bundle records `Crosses boundaries: yes: <services>, approved by the user <date>`.
- **Size**: for each bundle whose summed budget is over ~1,500 lines, offer a split (e.g. models + API / UI) or keep it, noting reviews get slower.
- **Order**: the checklist order (dependency order by default).
- **Leftovers**: issues that don't fit any bundle stay single, or join the nearest one.

## 3. Confirm, then apply
Show the final table and ask: apply / let me edit. Then, per bundle:
1. Write the body from `${CLAUDE_PLUGIN_ROOT}/templates/issue-bundle.md`: `Bundles:`, union of Delivers and Touches, summed Budget and Estimate, the checklist, and each issue's reading, Decisions rows and acceptance copied in. `Depends on:` lists only issues outside the bundle.
2. `gh issue create --title "<service>: <summary> (bundle)" --body-file <file> --milestone <the latest milestone among them> --label <union of service:/req: labels>`, plus `ready`/`plan-approved` only if every bundled issue has them, else `blocked` or neither.
3. For each bundled issue: `ck-github sub-issue <bundle> <issue>` and `gh issue edit <issue> --add-label bundled`.
4. `ck-github add-item <project> <bundle-url> <highest priority among them>`; set Status to match its label.
5. `ck-github log <log-issue> "bundle #<n> · <issues> · crosses boundaries: <no|services>"`.

## Rules
- Never bundle an issue with an open PR, a `contract` issue, or an Integration issue.
- A bundle that crosses a boundary needs the user's explicit yes in step 2.
