---
name: research
description: Use when a project's brief, repo or a proposed change needs facts gathered before decisions are made, when the user asks a question about options, trade-offs or performance that should be researched and recorded, when the user wants to steer research with priorities, or when the user asks to research or re-research a topic.
argument-hint: "[topic ... | ask \"<question>\" | steer \"<priority>\"]"
allowed-tools: Bash(git ls-files *) Bash(git log *) Bash(ls *) Bash(date +%F)
---

# Research

Learn the project before deciding anything about it, steered by the user's priorities, in a loop the user drives. Output: `docs/research.md`, shaped like `${CLAUDE_PLUGIN_ROOT}/templates/research.md.tmpl`.

Brief: `CHALLENGE.md` unless SPEC.md or the user names another path. Arguments: `$ARGUMENTS`.

## Modes
| Arguments | Do |
|---|---|
| none | Full pass: steps 1–6 |
| `<topic> ...` | Confirm the topics, step 4 for them only (replace or append their sections), then steps 5–6 |
| `ask "<question>"` | Step 6 with that question, then keep looping |
| `steer "<priority>"` | Step 2 with that priority, then step 5, then step 6 |

## 1. Survey (yourself, cheaply)
Read the brief in full. Inventory the repo: `git ls-files`, layout, languages, existing services and data, build/run commands, CI, docs. About ten reads; this only feeds the agenda.

## 2. Priorities
Ask the user what should steer the research: performance targets, scale, build speed, team familiarity, cost, anything. Record each in the Priorities table:
- **constraint**: a must, from the brief or the user ("handle 100 users/minute"). Options that fail it are out.
- **preference**: traded off against the others, with a weight high / medium / low ("fastest framework", "fast CI builds").

Ask the kind and weight when the user doesn't say. Read the brief for constraints the user didn't mention and propose them. If kickoff passed a time box and grade, or SPEC.md has them, they are a constraint row ("fits 4h at demo grade"): prefer options that are quick to set up and that the grade doesn't over-serve. A priority never decides a topic on its own: it is one input to every topic's evaluation.

With `steer`: add or change that priority, then re-evaluate every topic it touches (step 4 for those topics, with the new priority) before step 5.

## 3. Agenda
Propose topics by research depth (`${CLAUDE_PLUGIN_ROOT}/rules/grades.md`; standard if none was set): light 2–3, standard 4–6, deep up to 8. Each informs at least one decision the spec will make: the source data or existing system, what the brief evaluates or leaves open, each deliverable's technology options, integration points, risks. **Every user-facing UI deliverable** (a page, component, screen or app UI) also gets a **visual design** topic, on top of the depth count (one shared topic when the deliverables share a look): layout and hierarchy at mobile and desktop widths, spacing and type scale, color and the existing theme or brand tokens, how each state looks (loading, empty, error, focus), and reference patterns for that kind of UI. Its recommendation must end in checkable visual acceptance bullets. The topic that covers the source data or existing system always has the focus question "which fields do the brief's outputs need that the source doesn't have?", answered with a Field map. Show: # · Topic · Decisions it informs · Priorities it bears on · Focus questions. The user approves, edits, drops or adds (AskUserQuestion, recommendation first). Dispatch nothing before approval.

## 4. Research topics
Dispatch one `challenge-kit:researcher` per topic, all in one message. Each prompt: the topic, the decisions it informs, the brief's path, the focus questions, **the full Priorities table**, and "return the topic format in your instructions". Write each returned section into `docs/research.md` in agenda order. Where two sections contradict, check the evidence yourself and say which held.

## 5. How it fits together
Rewrite this section from all topics, priorities and follow-ups:
- **The recommended combination**: each topic's recommendation as one system; which priorities each serves and which it costs.
- **Tensions**: every place where serving one priority works against another (a faster runtime vs faster CI builds; horizontal scaling vs operational simplicity). For each: where it bites, the options, the recommended resolution given the weights, and what would flip it.
- **If priorities change**: which recommendations move if a preference is dropped or reweighted.

A topic's recommendation may differ from the option that wins its headline priority. Say so, and why.

## 6. Follow-up loop
Show the user, in chat: the recommended combination in a few lines, the top tensions, and the open questions. Then ask: **"Any follow-up questions? Name a topic number, or just ask. Say `done` when finished."**

For each question:
1. **Route** it to a topic (or several). If none fits, it becomes a new numbered topic in the agenda.
2. **Answer** with a `challenge-kit:researcher` in question mode: the question, the topic's current section, the Priorities table, and any condition the user gave ("considering we integrate with GitHub"). Several questions at once: one researcher each, in one message.
3. **Show** the answer in chat, short: the direct answer first, then what it depends on.
4. **Record** it under that topic's Follow-ups: date, question, answer, evidence. If it changes the topic's recommendation, update the recommendation and add "Recommendation updated: was X, now Y, because …".
5. **Rewrite** step 5 if anything changed.
6. Ask again.

The loop ends only when the user says `done` (or clearly ends it). When kickoff ran this skill, return to kickoff then.

## Rules
- Every finding, answer and claim in `docs/research.md` has evidence. Comparisons ("which is faster") name what was compared, under what conditions, with what source; unverified claims go under Open questions.
- Research recommends; it decides nothing. Decisions happen in kickoff or change-spec, with the user.
- Never write SPEC.md here.
