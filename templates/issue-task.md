---
name: Task
about: One service, one PR. Per-milestone issues list their pieces in a Checklist
title: ""
labels: []
---

<!-- ck:{{M<n>:<service> | R<n>:<slug>}} -->

## Goal
{{One or two sentences: what exists when this is done}}

Delivers: {{R<n>, ...}}
Touches: {{paths, one service}}
Depends on: {{#n, #m | none}}
Contract: {{docs/contracts/<name>.md | none}}
Budget: {{~files, ~lines}}
Estimate: {{S / M / L}} (~{{minutes}} min)
<!-- Optional, from replan: Grade: <g> (override) · Speed: <levers> -->

## My reading of {{R<n>, ...}}
{{One sentence per R: how this issue interprets the requirement. The user approves this with the plan}}

## Checklist
<!-- Per-milestone issues only (delete for fine-grained). Dependency order; one commit per item; tick it when its acceptance passes. -->
- [ ] {{piece}} ({{R<n>}}): {{its acceptance bullets}}

## Decisions

<!-- Kickoff seeds boundary decisions; /orchestrate adds the plan's rows; the worker adds how-to rows. Chosen = `ask` makes the worker stop and ask. -->

| Question | Options | Chosen | Why |
|---|---|---|---|

## Acceptance criteria
- [ ] {{From R<n>'s acceptance bullets, narrowed to this issue}}
- [ ] `make check` passes

## Out of scope
- {{What this issue deliberately doesn't do, including neighbouring issues' work}}
