---
name: integrate
description: Use when the build milestones are closed (or the user asks) to check the real services against their contracts and smoke-test the whole system end to end.
disable-model-invocation: true
allowed-tools: Bash(gh issue *) Bash(gh pr *) Bash(gh api *) Bash(make *) Bash(curl *) Bash(git *) Bash(ck-github *)
---

# Integrate

Prove the pieces work together. Run from the orchestrator session, on up-to-date main.

1. **Bring it up**: `make up` (or what AGENTS.md/README names). Wait until each service answers; if one won't start, that's the first finding.
2. **Contracts against real providers**: for each contract in SPEC.md, send every case in `docs/contracts/<name>/examples/` to the real provider and compare: status, fields, types, nullability, error bodies. Then every documented error the examples don't cover.
3. **Consumers against real providers**: point each consumer at the real provider instead of the mock and run its tests or flows.
4. **Smoke test**: walk every user-facing flow the active Rs name, end to end, the way a user would (browser tools for UI, `curl` for APIs, the CLI for a CLI). One line per R: passed, or what broke.
5. **Keep it**: turn steps 2–4 into `make test-integration`, run by `make check` or CI, on a branch with a PR (an Integration issue; open one if none exists).
6. **File failures**: one issue per failure, from the task template, labelled `needs-triage`, `service:<s>`, `req:R<n>`, in that service's milestone (`ck-github reopen-milestone` if closed), added to the Project with Status Todo. A contract mismatch where the doc is wrong is a `contract` issue instead.
7. **Report**: per contract and per R, pass or the filed issue. Tell the user to triage (remove `needs-triage` from the ones to build) and then run `/challenge-kit:orchestrate`. When all passes, close the Integration milestone.
