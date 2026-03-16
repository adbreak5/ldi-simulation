# Simulation Databank — `plan/databank.md`

Purpose
- Centralized, append-only record of important decisions, actions, and status updates for the LDI simulation project.

Usage rules
- Every substantive action or decision (design choice, parameter selection, data-source choice, experiment run, CI result) must be recorded as a short entry here.
- Entries should be timestamped (ISO 8601), include the actor (user/agent/subagent), a 1-line summary, and 1–3 lines of supporting context.
- Agents and subagents must read this file before planning and must reference its latest state when deciding next steps.
- When an agent applies a change, it must append an audit entry with the plan summary, diff link or patch, and test/lint outputs.

Format (example entry)
- 2026-03-16T12:34:00Z | agent:senior-quant-advisor | DECISION: Use BOE yield-curve zip as primary source | RATIONALE: direct CSV/XLS, daily updates. ACTIONS: fetched, stored in `data/boe/`.

Minimal content now
- 2026-03-16T00:00:00Z | user | INIT: Databank created. Project baseline from `plan/codeplan/plan_1.md`.
- 2026-03-16T18:30:00Z | agent:senior-quant-advisor | DECISION: Created detailed modified execution plan in `plan/codeplan/plan_2.md` | RATIONALE: tighten delivery under 2.5-week window while preserving publishable quality. ACTIONS: added explicit workstreams, quality gates, decision points, day-by-day roadmap, and escalation criteria.
