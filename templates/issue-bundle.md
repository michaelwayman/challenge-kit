---
name: Bundle
about: Several planned issues built by one worker in one PR
title: ""
labels: []
---

<!-- ck:bundle:{{slug}} -->

## Goal
{{One or two sentences: what exists when every bundled issue is done}}

Bundles: {{#a #b #c}}
Delivers: {{R<n>, R<m>}}
Touches: {{union of the bundled issues' paths}}
Crosses boundaries: {{no | yes: <services>, approved by the user YYYY-MM-DD}}
Depends on: {{#n outside the bundle | none}}
Contract: {{docs/contracts/<name>.md, ... | none}}
Budget: {{sum of the bundled budgets}}
Estimate: {{sum}} min

## Checklist
<!-- Dependency order. One commit per item; tick it when its acceptance passes. -->
- [ ] #{{a}} {{title}}
- [ ] #{{b}} {{title}}

## Bundled issues
<!-- Copied from each issue: its reading, Decisions rows and acceptance criteria. The issues stay open as sub-issues and close with the PR. -->

### #{{a}} {{title}}
My reading of {{R<n>}}: {{...}}

| Question | Options | Chosen | Why |
|---|---|---|---|

- [ ] {{acceptance criterion}}

## Acceptance criteria
- [ ] Every bundled issue's criteria above
- [ ] `make check` passes

## Out of scope
- {{Union of the bundled issues' Out of scope items}}
