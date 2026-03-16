---
description: Describe when these instructions should be loaded by the agent based on task context
# applyTo: 'Describe when these instructions should be loaded by the agent based on task context' # when provided, instructions will automatically be added to the request context when the pattern matches an attached file
---

<!-- Tip: Use /create-instructions in chat to generate content with agent assistance -->

Provide project context and strict coding & data rules that AI agents should follow when generating code, answering questions, or reviewing changes for the LDI simulation project.

When to load these instructions
- Load for any agent action that will edit code, fetch or process data, run experiments, or prepare results for publication.

Core expectations (short)
- Plan first: produce a short, numbered plan (steps, time estimate, acceptance criteria) before edits.
- Ask one targeted clarification question at a time and wait for the user's reply.
- Keep changes minimal: prefer small, well-tested edits over broad refactors.

Coding rules
- Use repository's `simulation/pyproject.toml` for style/type configuration; respect existing typing and conventions.
- Run and pass these checks before proposing commits: `ruff check . --select ALL`, `mypy --ignore-missing-imports .`, and `pytest -q` for affected tests.
- Add unit tests for any behavioral change; include a smoke E2E test for the CLI runner when adding features.
- Do not introduce new third-party dependencies without explicit justification and approval.

Data & reproducibility
- Prefer public data sources in the following fallback order: Bank of England (BOE) → DMO → FRED → ICMA → synthetic.
- If programmatic data fetch is used, include the exact commands/URLs and cite license/terms of use.
- Bundle synthetic-data seed values and a small sample dataset to allow immediate reproducibility.

Publication standards
- Document assumptions, parameter choices, and limitations in a `docs/` or `plan/` file alongside results.
- Include short reproducibility instructions (venv activation, install, run commands) with any major result.

Security & ethics
- Never include secrets, credentials, or private/proprietary data in the repo or outputs.
- For any external dataset with restricted reuse, record the license and include an explanation of how it was used.

Enforcement & workflow
- Agents must present a dry-run patch (unified diff) and test output for user approval before applying changes.
- All automated modifications must be accompanied by an audit entry: plan, diff, tests run, and final commit SHA.
- Add CI that runs `ruff`, `mypy`, and `pytest` on PRs and fails the check if any test or type/linting error occurs.

Contact points
- If data tables are unclear or unavailable, raise a single question in chat and await a user reply. Do not proceed with assumptions that materially alter results.

Copilot / assistant-specific rule
- When using GitHub Copilot or any assistant mode, always ask exactly one focused clarifying question if unsure about intent or missing information; wait for the user's reply before making changes. Do not make bold assumptions that materially change behavior, scope, or data usage.
- To apply these rules in Copilot Chat, copy the relevant sections from this file into the Copilot Chat "Custom instructions" or system prompt so the assistant uses them by default.

Databank requirement
- All important actions, decisions, and status updates MUST be recorded in `plan/databank.md` as an append-only entry with ISO 8601 timestamp, actor, 1-line summary, and brief context.
- Agents and subagents must read `plan/databank.md` before planning and must reference its entries when proposing changes.
- Before applying any change, the agent must append a planned-entry to the databank (dry-run entry) and after successful execution, append the final audit entry with test outputs and commit SHA.
- Use the format shown in `plan/databank.md` for consistency.

Repository layout and data logging rules
- `simulation/`: contains all simulation code, modules, scripts, CLI runners, and tests. Do not place large data dumps or final report materials here.
- `report/`: final dissertation/report sources and compiled output (LaTeX, PDF). Keep final write-up artifacts here.
- `plan/`: planning materials, `codeplan/` outputs, the `databank.md`, UCL marking schemes, notes, reproducibility runbooks, and any raw or processed data dumps collected or web-scraped for calibration. Use `plan/data/` as the canonical location for scraped/downloaded data files and CSVs.
- All web-scraped or externally downloaded datasets used for experiments MUST be stored under `plan/data/` (or a subfolder) and logged in `plan/databank.md` with timestamp, source URL, local path, and license/terms summary before they are used in simulations.
- Agents must never commit external data directly into `simulation/` or `report/`; instead write and reference data from `plan/data/` and include a databank entry describing file provenance and usage.
