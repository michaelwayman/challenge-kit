---
name: sync-github
description: Use when SPEC.md has changed and GitHub (labels, milestones, issues, the Project) must match it, or when the user asks to sync or repair the GitHub tracking. Run by kickoff and change-spec.
allowed-tools: Bash(ck-github *) Bash(${CLAUDE_PLUGIN_ROOT}/bin/ck-github *) Bash(gh issue list *) Bash(gh issue view *) Bash(gh repo view *) Bash(gh auth status *) Bash(git ls-files *)
---

# Sync GitHub with SPEC.md

SPEC.md is the source of truth; GitHub mirrors it. This skill is idempotent: running it twice changes nothing the second time. `ck-github` is on PATH from the plugin's `bin/` (fall back to `${CLAUDE_PLUGIN_ROOT}/bin/ck-github`); `ck-github` with no arguments prints its usage.

## 1. Preconditions
- `gh auth status` lists the `project` scope. If not, stop and tell the user to run `gh auth refresh -s project`.
- SPEC.md exists and the default branch `main` exists on the remote.

## 2. Read the desired state from SPEC.md
- Services → `service:<s>` labels. Active and removed R → `req:R<n>` labels (keep labels of removed Rs; closed issues still carry them).
- Milestones: `M<n> <name>` titles, exactly as SPEC.md's Milestones table names them.
- Issues: break each active R into issues. One issue = one service, one layer, about one PR: an endpoint, the page that shows it, a build step and a component are separate issues. Scaffolding, CI and each contract's mock are Integration issues. Each issue body follows `${CLAUDE_PLUGIN_ROOT}/templates/issue-task.md`, including its first line `<!-- ck:R<n>:<slug> -->` (slug: kebab-case, stable, unique within the R). Integration issues use `ck:M<n>:<slug>`.
- Priority: the R's priority unless the user set another for the issue.
- Dependencies: `Depends on:` other issues' slugs until they have numbers. An issue with an open dependency is `blocked`, otherwise `ready`.

## 3. Read the current state
- `gh issue list --state all --limit 1000 --json number,title,state,labels,milestone,body` and index by the `ck:` marker.
- `ck-github items <project-number>` for Status and Priority. The project number is in SPEC.md's header line; if it says `{{PROJECT_NUMBER}}`, there is no Project yet.

## 4. Show the diff, then wait
One table: Action (create / update / close / reopen-milestone / add-to-project) · Issue or object · What changes · Why (which R or M). Updates list the changed fields only. Ask the user to confirm (AskUserQuestion: apply all / let me edit). Apply nothing before a yes.

Never in the diff:
- changing a Priority that is already set in the Project (the user owns it after creation)
- editing an issue that has an open PR, beyond a comment saying what changed in SPEC.md
- deleting anything: removed work is closed as not planned, with a comment naming the SPEC.md changelog entry

## 5. Apply, in this order
1. `ck-github labels service:<s>... req:R<n>...`
2. `ck-github milestones "<M title>"...`; `ck-github reopen-milestone "<title>"` for each closed milestone that gets new open work
3. `ck-github protect check` (only on the first run, or when the user asks)
4. `ck-github project "<repo name> challenge-kit" [number]`. If it printed a new number, write it and the URL into SPEC.md's header line
5. Issues: create (`gh issue create --title --body-file --milestone --label`), then a second pass that rewrites `Depends on:` slugs as `#<n>`. Update with `gh issue edit --body-file` (fetch the body, edit only the sections that changed, keep the Decisions rows). Close with `gh issue close --reason "not planned" --comment`
6. For every issue: `ck-github add-item <number> <issue-url> <P>` then `ck-github set-status <number> <issue-url> <Ready|Todo>` to match its label (Todo for blocked and needs-triage)

Use body files in your scratchpad, never inline heredocs with backticks.

## 6. Report
Counts per action, the Project URL, and any step that failed with its error. A failed step is reported, never retried silently. Remind the user once, if the board is new: in the Project's Board view, "Group by → Milestone" groups the columns by milestone (the API can't set grouping).
