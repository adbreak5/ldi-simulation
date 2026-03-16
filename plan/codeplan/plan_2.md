# Modified Execution Plan (Time-Crunch, Publishable Quality)

Date: 2026-03-16
Owner: senior-quant-advisor workflow
Primary reference: `plan/codeplan/plan_1.md`

## 1. Execution Intent

Deliver a dissertation-quality LDI crisis simulation under severe time pressure by maximizing reproducibility, minimizing scope risk, and prioritizing the highest-marking outputs first.

This modified plan keeps your original research ambition but tightens delivery mechanics:
- MVP first, heterogeneity later.
- Public data first, synthetic fallback quickly.
- Hard quality gates (ruff, mypy, pytest) before each milestone.
- Write-up starts before all experiments finish.

## 2. Non-Negotiable Constraints

- Deadline pressure: 2.5 weeks effective execution window.
- Folder discipline:
  - `simulation/` = code, scripts, tests.
  - `plan/` = planning artifacts, rubrics, web-scraped/downloaded datasets, databank.
  - `report/` = final dissertation write-up.
- Data-provenance discipline:
  - All web-scraped/downloaded data goes under `plan/data/`.
   - Every data decision/action must be logged in `plan/databank.md`.

## 3. Quality Gates (Must Pass)

Before closing any major task block:
- `ruff check . --select ALL`
- `mypy --ignore-missing-imports .`
- `pytest -q` (for impacted tests)

Before moving from model development to bulk experiments:
- One end-to-end scenario run succeeds from clean environment.
- At least one margin-math unit test and one smoke test pass.

## 4. Workstreams and Deliverables

## 4.1 Workstream A: Data Acquisition and Calibration Inputs

Goal:
- Build a minimum viable calibration dataset quickly without blocking simulation development.

Action sequence:
1. Create `plan/data/` structure:
   - `plan/data/raw/boe/`
   - `plan/data/raw/dmo/`
   - `plan/data/raw/fred/`
   - `plan/data/raw/icma/`
   - `plan/data/processed/`
2. Pull BOE yield curve package first.
3. Pull DMO aggregated yields/turnover second.
4. Pull FRED supplemental series (after series ID validation).
5. If haircut series remains incomplete, parameterize haircut regimes and proceed with documented synthetic proxy.
6. Log every pull in `plan/databank.md` with URL, timestamp, local path, and license note.

Critical decision points:
- DP-A1: If BOE/DMO access fails for more than 4 hours, switch to synthetic calibration for MVP and continue.
- DP-A2: If ICMA/BoE papers do not provide machine-readable haircut time series, define haircut shock bands and explicitly mark as proxy assumption.

Deliverables:
- Data inventory table in `plan/`.
- Reproducible fetch script(s) or command log.
- Provenance entries in databank.

## 4.2 Workstream B: Core Simulation Engine (MVP)

Goal:
- Deliver a robust single-fund mechanistic simulation that reproduces the margin spiral sequence.

Action sequence:
1. Finalize state variables:
   - cash, free gilts, repo-encumbered gilts, swap position, recapitalization delay.
2. Implement deterministic step logic:
   - shock yields/rates,
   - update VM and repo margin,
   - check shortfall,
   - execute fire-sale,
   - apply market impact,
   - move to next step.
3. Implement baseline linear impact.
4. Implement non-linear/square-root toggle (feature flag).
5. Add policy intervention toggle (BoE buffer rule proxy).

Critical decision points:
- DP-B1: If model complexity slows progress, freeze to single-fund until all MVP outputs are generated.
- DP-B2: If non-linear impact destabilizes numerics, keep linear baseline as primary and run non-linear as sensitivity extension.

Deliverables:
- Runnable CLI for one scenario producing JSON time-series.
- One deterministic seed path for reproducibility.
- Unit tests for VM and haircut margin components.

## 4.3 Workstream C: Stress Grid and Phase Boundary Mapping

Goal:
- Produce publishable heatmaps with clear cascade boundary interpretation.

Action sequence:
1. Define fixed parameter grids for:
   - E1: yield shock × recap delay,
   - E2: leverage × haircut shock,
   - E3: with/without policy threshold,
   - E4: linear vs non-linear impact.
2. Build batch runner with checkpointed outputs.
3. Save outputs to structured files (CSV/Parquet/NPY) and figures.
4. Annotate phase transitions with threshold notes.

Critical decision points:
- DP-C1: If full 4-experiment run is too slow, prioritize E1 and E3 first (highest policy narrative value), then E2, then E4.
- DP-C2: If compute budget is constrained, reduce grid granularity but keep boundary-identification clarity.

Deliverables:
- Core heatmap set with captions and assumptions.
- Summary table of threshold shifts under policy intervention.

## 4.4 Workstream D: Report Production and Critical Analysis

Goal:
- Convert outputs into first-class rubric-aligned narrative while code is still fresh.

Action sequence:
1. Draft methodology while implementing model logic (do not wait).
2. Draft results immediately after first stable heatmaps.
3. Build dedicated limitations section:
   - data proxy limitations,
   - haircut observability,
   - model abstraction bounds,
   - external validity caveats.
4. Map each chapter to UCL marking rubric criteria.

Critical decision points:
- DP-D1: If time is slipping, stop adding model features and allocate all remaining effort to analysis and presentation quality.

Deliverables:
- Report-ready figures with reproducible generation notes.
- Rubric coverage checklist in `plan/`.

## 5. Day-by-Day Roadmap (18 Days)

Day 1
- Freeze scope and acceptance criteria.
- Setup folder/data conventions (`plan/data/`, databank entries).

Day 2
- Implement/validate core accounting functions (VM + haircut margin).
- Add first unit tests.

Day 3
- Complete single-step simulation loop.
- Validate deterministic repeatability with fixed seed.

Day 4
- Add linear impact path and CLI scenario run.
- Produce first JSON output.

Day 5
- Build first E1 coarse grid runner.
- Generate first coarse heatmap.

Day 6
- Add non-linear impact toggle.
- Compare linear vs non-linear on a small subset.

Day 7
- Add policy intervention toggle.
- Produce preliminary E3 comparison plot.

Day 8
- Run quality sweep (ruff/mypy/pytest) and bug fixes.
- Lock MVP milestone.

Day 9
- Integrate real/synthetic calibration inputs.
- Re-run E1 and E3 with updated parameters.

Day 10
- Implement E2 grid (leverage × haircut shock).
- Save outputs and metadata.

Day 11
- Optional heterogeneity layer only if previous blocks are stable.
- If unstable, skip and document rationale.

Day 12
- Compute robustness checks (seed sensitivity, grid sensitivity).
- Prepare result summary tables.

Day 13
- Figure cleanup and captioning.
- Begin final results narrative.

Day 14
- Freeze simulation features.
- Final reproducibility pass.

Day 15
- Write methodology and implementation sections.

Day 16
- Write results and policy interpretation.

Day 17
- Write critical analysis and limitations.

Day 18
- Final polish, formatting, checklist validation, submission packaging.

## 6. Decision Escalation Rules

Escalate to user immediately (single focused question) if any of the following occurs:
- Required public dataset unavailable or licensing unclear.
- A model change would materially alter the interpretation of results.
- Runtime exceeds feasible daily budget and requires experiment downscoping.
- Validation checks repeatedly fail after a reasonable fix attempt.

## 7. Definition of Done

Technical done:
- CLI run + reproducible core figures generated from clean setup.
- Lint/type/tests pass.
- Databank contains complete decision and provenance trail.

Academic done:
- Research question answered with phase-boundary evidence.
- Policy intervention effect quantified and critically interpreted.
- Limitations explicitly analyzed against rubric expectations.

## 8. Immediate Next Actions (Start Now)

1. Create `plan/data/` skeleton and add provenance README.
2. Execute BOE and DMO first-pass downloads into `plan/data/raw/`.
3. Validate/record FRED series IDs and add fetch command block.
4. Build and run first deterministic single-scenario pipeline.
5. Append databank entries after each of the four steps above.
