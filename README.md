# challenge-kit

A Claude Code plugin that takes a coding challenge (or any new multi-part project) from its brief to a spec, GitHub tracking, and parallel, reviewed work.

It knows nothing about any particular challenge. A guided research step learns each project; everything project-specific is written into the project's own repo.

## Install

From GitHub (private repo: you need read access to `michaelwayman/challenge-kit`):

```bash
claude plugin marketplace add michaelwayman/challenge-kit
```

Or from a local clone:

```bash
claude plugin marketplace add ~/Projects/challenge-kit
```

Then, either way:

```bash
claude plugin install challenge-kit@challenge-kit
```

Kickoff can also enable the plugin for everyone who opens a project, by writing the GitHub marketplace into that project's `.claude/settings.json`.

Needs `gh` (with the `project` scope: `gh auth refresh -s project`) and `jq`. While working on the kit itself, `claude --plugin-dir ~/Projects/challenge-kit` loads it straight from disk; `/reload-plugins` picks up edits.

## The flow

1. **`/challenge-kit:kickoff [brief]`**. Reads the brief (`CHALLENGE.md` by default) and the repo, then runs research. Drafts `SPEC.md` (requirements, out of scope, services and boundaries, contracts, technology choices, milestones with priorities) and asks about every gap. Shows one table of every choice for a final review. Writes locally and **waits for you to say `ready`**. Then it renders `AGENTS.md`, `.claude/settings.json` and the GitHub templates, opens a PR, and runs sync-github once it merges.
2. **`/challenge-kit:orchestrate`**. Each round, it proposes the next issues in priority order, one table per milestone lane. Each row shows its reading of the requirement, its key decisions and its size budget, and you approve or change them. It then starts up to two worker sessions per lane, each in its own worktree, answers their questions and reviews their PRs. You merge, then say "I merged #n".
3. **`/challenge-kit:integrate`**. Once the build milestones close: contracts against the real services, an end-to-end smoke test, and failures filed as issues for you to triage.

Anytime:

| You want to | Run |
|---|---|
| Add, drop, change or clarify a requirement; move something in or out of scope; change a tech choice | `/challenge-kit:change-spec "<the change>"` |
| Research or re-research a topic | `/challenge-kit:research <topic>` |
| Ask a follow-up research question (recorded in `docs/research.md`) | `/challenge-kit:research ask "which is faster here, A or B?"` |
| Steer research with a priority (weighed, not a mandate) | `/challenge-kit:research steer "handle 100 users/minute"` |
| Re-sync GitHub with SPEC.md | `/challenge-kit:sync-github` |
| Stop or redo a worker's issue | `/challenge-kit:orchestrate stop 42` / `redo 42` |
| Turn a correction into a rule | `/challenge-kit:record-correction "<what went wrong>"` |
| Put visible copies of the skills and agents in the repo | `/challenge-kit:eject` |

**Choosing the order**: every issue sits in the repo's GitHub Project with a **Priority** (P0/P1/P2). Kickoff and change-spec propose priorities, and you edit them in their review tables or later in the board. The orchestrator always picks P0 before P1 before P2, but never an issue whose dependencies are still open.

## What lives where

| In the plugin (this repo) | In the project repo |
|---|---|
| `skills/`: kickoff, research, change-spec, sync-github, orchestrate, work-issue, review-pr, integrate, record-correction, eject | `SPEC.md`: the single source of truth, with stable IDs (R, O, T, M) and a changelog |
| `agents/`: researcher, reviewer, contract-checker (read-only, Sonnet) | `docs/research.md`: findings with evidence |
| `templates/`: SPEC, AGENTS, research, settings, issue, PR | `docs/contracts/`: one doc + examples per boundary |
| `rules/working-rules.md`: generic rules every project gets | `AGENTS.md` (+ `CLAUDE.md` symlink): working, review and learned rules for this project |
| `bin/ck-github`: labels, milestones, protection, Project | `.claude/settings.json`: permissions and the plugin enabled for this project |

Agents run inside the project repo, so they follow its `AGENTS.md`. Feedback that's specific to the project goes there. Feedback about how the kit itself behaves goes into this repo. `record-correction` decides which.

## Keeping workers on track

- **Before code**: each round's table shows how the planner reads each requirement and a size budget. A misreading is a sentence you correct, not a PR.
- **Before the PR**: the worker stays inside `Touches:`, checks its diff against the budget, and runs the reviewer. The reviewer treats a misreading, scope creep and anything out of scope as findings.
- **After the PR**: review-pr posts an independent review. Comment on the PR (Auto-fix acts on it), `orchestrate redo <n>`, or fix the requirement itself with `change-spec`.

Design notes: [docs/design.md](docs/design.md).
