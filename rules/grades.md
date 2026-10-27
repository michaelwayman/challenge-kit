# Grades, up-front depth, estimates and speed levers

Project-agnostic. Kickoff, research, replan, bundle, orchestrate, work-issue, reviewer and integrate read this file. SPEC.md § "Time box and grade" records which settings a project uses; an R or issue may carry an override (`Grade: demo (override)`, `Speed: ...`), which wins over the project default.

## Grades

| | demo | solid | production |
|---|---|---|---|
| Requirements | core flows only; optional parts proposed as out of scope | everything the brief asks for | everything the brief asks for, plus edge cases it implies |
| Tests | happy path per R | core paths plus error cases | plus edge cases, validation and contract tests |
| Errors and validation | fail loudly; no polish | clean errors at the boundaries | strict validation, consistent error bodies |
| CI | `make check` locally | `check` job in CI | plus required check and branch protection |
| Observability and hardening | none; building it is scope creep | basic logging | logging, health checks, config by environment |
| Review | reviewer blocks only on correctness, misreading and scope | full reviewer | full reviewer plus contract-checker on every PR |
| Integrate | smoke test only | contracts plus smoke test | contracts, consumers against real providers, smoke test, `make test-integration` |
| Estimate multiplier | 0.6 | 1.0 | 1.6 |
| Reserve (integration and review overhead) | 15% of the time box | 25% | 35% |
| Default research depth | light | standard | deep |
| Default contract depth | sketch | full | full |

### Grade rules rendered into AGENTS.md
Kickoff (and replan, on a grade change) copies the bullets for the project's grade under AGENTS.md "Grade rules".

**demo**
- Build the happy path each acceptance bullet names. Error cases, validation beyond what the framework gives, logging, health checks and config layers are out unless an acceptance bullet asks
- One test per R that walks its happy path
- A missing edge case is not a review finding; building one nobody asked for is scope creep

**solid**
- Happy path plus the error cases a caller can trigger: bad input, missing record, provider down
- Tests for core paths and those error cases; CI runs `make check` as a job named `check`
- Errors at service boundaries are clean and documented in the contract; basic request logging

**production**
- Everything in solid, plus edge cases the brief implies: empty data, limits, concurrency, idempotency where a caller can retry
- Strict input validation and one error-body shape per service; logging, a health endpoint, configuration by environment
- Contract tests on the provider side in CI; branch protection requires `check`
- A missing edge case the R implies is a review finding

## Up-front depth

| Setting | Options |
|---|---|
| Research depth | **light**: 2–3 topics, repo first, web only for a choice that is actually open, follow-up loop offered once · **standard**: 4–6 topics · **deep**: up to 8 topics, web benchmarks with stated conditions, follow-up loop until `done` |
| Contract depth | **sketch**: operations, fields and status codes · **full**: every error, an `examples/` file per case, a mock per contract as an Integration issue |
| Issue size | **per milestone** (default): one issue per service per milestone, its pieces as a checklist, one PR each · **fine**: one issue per capability and layer, one PR each; `challenge-kit:bundle` can regroup them later |

Up-front depth is independent of grade: deep research and full contracts with a demo build and per-milestone issues is a valid combination.

## Estimates
- Size per R and per issue: **S** 20, **M** 45, **L** 90 agent wall-clock minutes, times the grade multiplier, times the calibration factor (SPEC.md; starts at 1.0).
- **Work left** = sum of open estimates ÷ parallel lanes in use, but never less than the longest dependency chain's sum.
- **Time left** = available hours (or deadline − now) − reserve − time already spent.
- **Fits** when work left ≤ time left.
- Calibration: per merged issue, actual minutes (the `started` comment to PR opened) ÷ estimate. The factor is the median ratio over merged issues; SPEC.md is updated when it moves by more than 20%.

## Speed levers
Offered by kickoff's fit check and by replan, before any cut. Savings are rough; the fit is re-run after the user picks.

| Group | Lever | Effect | Rough saving | Setting in SPEC.md |
|---|---|---|---|---|
| Planning | Batch-plan | plan a whole milestone in one round and one approval | ~5 min per round saved | `planning: batch` |
| Planning | Auto-approve S | only M and L plans come to the user | ~2 min per S issue | `auto-approve: S` |
| Planning | Self-planning | no Plan agents; the worker writes its reading and decisions into the PR, and stops only on `ask` rows | ~5 min per issue | `planning: self` |
| Per-ticket effort | Minimal implementation | size budgets × 0.7; meet the acceptance bullets and nothing more, same grade | ~25% per issue | `budget-factor: 0.7` |
| Per-ticket effort | One review pass | the worker's reviewer only; review-pr runs only when the reviewer reported gaps | ~5 min per PR | `review: single` |
| Per-ticket effort | Bundle tickets | fine-grained issues only: several issues, one worker, one PR (runs `bundle`) | ~10 min per issue folded in | bundles on GitHub |
| Parallelism | More slots | up to 3 workers per lane where `Touches:` don't overlap | divides work left | `slots-per-lane: 3` |
| Quality | Lower the grade | e.g. production → solid | the multiplier ratio | `grade:` |
| Scope | Cut or stretch Rs | lowest priority and least depended-on first | that R's estimate | change-spec operations |

Scope comes last unless the user picks it.
