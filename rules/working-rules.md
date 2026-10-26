# Working rules

Project-agnostic rules every challenge-kit session follows. Kickoff copies them into the target repo's `AGENTS.md` under "Working rules". Project-specific rules go in that repo's `AGENTS.md` under "Learned rules" or "Review rules", never here.

## Scope
- Build only what the issue's `Delivers: R<n>` and its approved Decisions cover. A new capability is a question for `change-spec`, not a call you make
- Nothing on SPEC.md's out-of-scope list (`O<n>`) gets built, even partly
- Stay inside the issue's `Touches:` paths: one service, one layer. An endpoint, the page that shows it, a build step and a component are separate issues
- Stay inside the size budget. Well over it means stop and ask, not open the PR
- List every choice the issue didn't decide under "Calls I made" in the PR

## Correctness
- Check library APIs against the installed code, not memory
- Before picking a version, check the peer ranges of every package that depends on it, and note why it's pinned next to the pin
- Make the schema as strict as the source data guarantees: NOT NULL, CHECK and unique constraints wherever every row satisfies them. A field that's legitimately empty in some states stays nullable; there, handle null in every query, sort and serializer that reads it
- Contracts are the only agreement between services. Test against the contract's examples, never against another service's code or branch

## Data and secrets
- Never put real user data or credentials in code, tests, fixtures or commits, even when they're already elsewhere in the repo (a seed dump, a fixture). Generate stand-in data; if you must keep real data, ask first and flag it in the PR

## Process
- Write the code and its tests together, then run them. While working, run only the targets the change touches; run `make check` once, before the PR
- Done means `make check` passes. Report what you ran and what it printed
- Before building tooling for the agent, check for a built-in Claude Code feature that already does it
