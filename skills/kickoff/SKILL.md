---
name: kickoff
description: Use once, at the start of a coding challenge or new project in a fresh repo or fork, to turn the brief into SPEC.md and GitHub tracking.
argument-hint: "[brief-path] [<hours>h] [demo|solid|production]"
disable-model-invocation: true
allowed-tools: Bash(git ls-files *) Bash(git status *) Bash(git log *) Bash(gh repo view *) Bash(date *) Bash(ck-github *)
---

# Kickoff

Turn the brief into an approved SPEC.md, then into GitHub tracking. You own the questions; the user owns every answer. Nothing leaves this machine before the user says **ready**.

Arguments: `$ARGUMENTS`. The brief is the argument that is a path, or `CHALLENGE.md`. `<n>h` is the time box; `demo`, `solid` or `production` is the grade. Definitions: `${CLAUDE_PLUGIN_ROOT}/rules/grades.md`.

## 0. Preconditions
- The brief exists. If not, ask for its path and stop.
- SPEC.md does not exist. If it does, stop: kickoff has run; changes go through `/challenge-kit:change-spec`, time and grade through `/challenge-kit:replan`.
- `gh repo view` works (a GitHub remote you can push to). If not, say so: steps 1–6 still run, step 7 needs it.

## 0b. Time box, grade and up-front depth
Record the start: `date -u +%FT%TZ`. Ask in one AskUserQuestion (skip the questions the arguments answered), recommendation first:
- **Time box**: 2h / 4h / 8h (Other for any amount or a deadline). If the deadline is more than one working session away, ask for the working hours before it.
- **Grade**: solid (Recommended) / demo / production, with one line each from rules/grades.md.
- **Research and contract depth**: the grade's defaults (Recommended) / deep research and full contracts / light research and sketch contracts.
- **Issue size**: one issue per service per milestone, the pieces as a checklist inside (Recommended: fewest issues, plan passes and PRs) / fine-grained: one issue per service and layer (more workers can run at once, at the cost of many more issues, plans and PRs).

Keep these for SPEC.md's "Time box and grade" section. They shape every step below, and the final review lists them.

## 1. Research
Pass the research depth, and the time box and grade as a constraint ("fits <n>h at <grade> grade"), to the research skill.
- `docs/research.md` doesn't exist: run the full pass of the `challenge-kit:research` skill (priorities, agenda approval and follow-up loop included).
- It exists: show its agenda table, priorities and date, and ask (AskUserQuestion, recommendation first): **reuse** it (then run its follow-up loop, and its priorities step if it has none) / **refresh** some topics (then run `challenge-kit:research <topics>`) / **redo** it (the full pass, replacing it). Never overwrite it without that answer.

Move on only when the user has ended the research follow-up loop. Everything below draws on `docs/research.md`: its Priorities, each topic's recommendation, and "How it fits together".

## 2. Requirements and out of scope
Draft from the brief and research:
- **R<n>**: one per capability a user or caller can exercise, each with a brief § source, observable acceptance bullets at the chosen grade, and a size (S/M/L). Split a deliverable into several Rs when it has independent parts. An R with a user-facing UI also gets **visual acceptance bullets**, each starting `Visual:`, taken from research's visual design topic and checkable on a screenshot at mobile (375px) and desktop (1280px) widths. At demo grade, propose optional parts as O items.
- **O<n>**: everything the brief rules out, plus what the user rules out below.

Then ask about every gap: things the brief leaves open, implies but doesn't state, or that research flagged. AskUserQuestion, up to 4 questions per batch, 2–4 options each, recommendation first with "(Recommended)". Typical options: in scope / out of scope / a narrower version. Keep going until no gap is left.

## 3. Boundaries, contracts, technology
Propose, from research recommendations:
- **Services**: what each owns, calls, and how it trusts callers; what a consumer does when its provider is down.
- **Contracts**: one per boundary between separately built pieces (API, shared library interface, file or event format), at the chosen contract depth.
- **Technology choices T<n>**: each with why, the alternative, the research section it rests on, and the priorities it serves or costs. Take the combination from research's "How it fits together", not each topic's winner in isolation.

Ask only about choices where research found a real trade-off (a row in its Tensions table) or the brief says the choice will be questioned. Accept the rest as proposed; they appear in the final review anyway. If the user asks a research question here, run `challenge-kit:research ask "<question>"` and come back.

## 4. Milestones and priority
- **M1 Integration**: scaffolding, `make check`, CI job named `check`, contract docs + examples + mocks. If any R has `Visual:` bullets, scaffolding also adds `make screenshots URL=<url> OUT=<dir>`: a pinned Playwright that saves full-page PNGs at 375×812 and 1280×800 (and 768×1024 at production grade), with `.ck/screenshots/` gitignored. It goes first.
- Then one milestone per service or deliverable, then integration/smoke and any data migration last. Every active R maps to exactly one milestone.
- Propose a Priority P0/P1/P2 for each R and milestone: P0 for what others depend on or the brief weights most.

## 4b. Fit check
Per rules/grades.md § Estimates (time already spent on research counts). Show: R · Priority · Size · Estimate · Running total · Fits?, then Time left · Work left · Gap.

If it doesn't fit, first offer the speed levers from rules/grades.md (AskUserQuestion, multiSelect, process levers first, each with its saving here), and re-run the fit. Then, for the rows still below the line, the user picks per R: **stretch** (P2, built only while time remains), **cut** (becomes an O, "cut for time"), or **keep** (needs more hours). Never cut without the user's pick. The chosen levers become the Speed setting.

## 5. Final review
Show **one table of every choice**: the time box, grade, research and contract depth, issue size and speed settings, the fit summary; each R (with priority, size and milestone), each O, each service boundary, each contract, each T, each M. Columns: ID · Choice · Why · Source (brief §, research §, or "you"). Ask the user to change any row. Apply edits and show the table again until they have none.

## 6. Write locally, then the ready gate
- Write `SPEC.md` from `${CLAUDE_PLUGIN_ROOT}/templates/SPEC.md.tmpl` (header line keeps `{{PROJECT_URL}}`/`{{PROJECT_NUMBER}}`/`{{LOG_ISSUE}}` until sync-github fills them), with "Time box and grade" filled in and a changelog row "Initial spec".
- Write `docs/contracts/<name>.md` per contract: purpose, every request/operation with fields and nullability, every error, and an `examples/` folder with one JSON file per documented case. Mocks are Integration issues, not written now.
- Tell the user what you wrote, then say exactly: **"Review SPEC.md and docs/contracts/. Say `ready` to commit and set up GitHub, or tell me what to change."**

Stop. Wait for the user's reply.
- The reply is `ready` (or clearly means only that: "ready", "I'm ready", "go"): continue to step 7.
- Anything else, including approval mixed with a change ("looks good but drop R4"): treat it as edits. Apply them, show the changed rows, and repeat the gate sentence. Never continue on an implied yes.

## 7. After ready
1. Render into the repo, filling every `{{…}}`:
   - `AGENTS.md` from `${CLAUDE_PLUGIN_ROOT}/templates/AGENTS.md.tmpl`, with `{{WORKING_RULES}}` = the bullets of `${CLAUDE_PLUGIN_ROOT}/rules/working-rules.md`, `{{GRADE_RULES}}` = the bullets for the grade from `${CLAUDE_PLUGIN_ROOT}/rules/grades.md`, `{{SPEED_RULES}}` = one bullet per Speed setting (or "None"), and one boundary line per service. Then `ln -s AGENTS.md CLAUDE.md` unless a CLAUDE.md exists (then ask).
   - `.claude/settings.json` from `${CLAUDE_PLUGIN_ROOT}/templates/settings.json.tmpl`. `{{MARKETPLACE_SOURCE}}`: ask the user, recommendation first: `{"source": "github", "repo": "<owner>/challenge-kit"}` if the plugin is pushed there, else `{"source": "directory", "path": "${CLAUDE_PLUGIN_ROOT}"}`. If a settings.json exists, merge keys instead of overwriting.
   - `.github/ISSUE_TEMPLATE/task.md` from `${CLAUDE_PLUGIN_ROOT}/templates/issue-task.md` and `.github/pull_request_template.md` from `${CLAUDE_PLUGIN_ROOT}/templates/pr-template.md`.
   - Never overwrite the brief, an existing README.md or Makefile.
2. Commit on a branch `kickoff-spec`: `docs/research.md`, `SPEC.md`, `docs/contracts/`, `AGENTS.md`, `CLAUDE.md`, `.claude/settings.json` and `.github/`. Nothing else: check `git status` and ask about any other changed file. Push, open a PR titled "Spec, contracts and agent setup". Ask the user to merge it (branch protection comes next, and main must hold SPEC.md for workers).
3. Once merged, run the `challenge-kit:sync-github` skill. It creates issues at the chosen issue size.
4. `ck-github log <log-issue> "kickoff · <n>h <grade> · start <UTC> · deadline <UTC> · <R count> Rs, fit <work left> of <time left>"`.
5. Finish with: the Project URL, the issue count per milestone, the time left, and the next step: **"/challenge-kit:orchestrate"**. Remind the user to open this folder once in the Claude desktop app and accept the trust prompt, or worker sessions can't start.
