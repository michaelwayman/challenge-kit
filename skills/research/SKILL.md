---
name: research
description: Use when a project's brief, repo or a proposed change needs facts gathered before decisions are made, or when the user asks to research or re-research a topic.
argument-hint: "[topic ...]"
allowed-tools: Bash(git ls-files *) Bash(git log *) Bash(ls *) Bash(date +%F)
---

# Research

Learn the project before deciding anything about it. Output: `docs/research.md`, built from `${CLAUDE_PLUGIN_ROOT}/templates/research.md.tmpl`.

Brief: `CHALLENGE.md` unless SPEC.md or the user names another path.

## With no arguments (full pass; kickoff runs this)

1. **Survey, yourself, cheaply.** Read the brief in full. Inventory the repo: `git ls-files`, top-level layout, languages, existing services and data sources, build/run commands (Makefile, compose, package scripts), CI, docs. Ten or so reads at most; this only feeds the agenda.
2. **Propose an agenda** of 4–8 topics. Each topic informs at least one decision the spec will make. Cover, where the brief has them:
   - the source data or existing system and its real shape
   - what the brief says is evaluated or required, and what it leaves open
   - each deliverable's technology options
   - integration points between deliverables
   - risks: data quirks, version constraints, anything the survey found surprising

   Show it as a table: # · Topic · Decisions it informs · Focus questions. Ask the user to approve, edit, drop or add topics (AskUserQuestion, recommendation first). Do not dispatch before approval.
3. **Dispatch** one `challenge-kit:researcher` agent per approved topic, all in one message so they run in parallel. Each prompt: the topic, the decisions it informs, the brief's path, the focus questions, and "return the format in your instructions".
4. **Synthesize** into `docs/research.md`: the agenda table, then each agent's section in agenda order. Resolve contradictions between sections by checking the evidence yourself, and say which one held. Then tell the user, in chat, the recommendations in one line each and the open questions that need them.

## With topics (`/challenge-kit:research <topic> ...`)

Skip the survey. Confirm the topic list with the user, dispatch one researcher per topic, then replace that topic's section in `docs/research.md` (or append a new numbered one) and update the agenda table and date. Leave other sections alone.

## Rules
- Facts come from researchers with evidence. Your own claims in `docs/research.md` need evidence too.
- Research recommends; it decides nothing. Decisions happen in kickoff or change-spec, with the user.
- Never write SPEC.md here.
