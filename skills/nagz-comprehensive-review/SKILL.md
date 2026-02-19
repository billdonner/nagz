---
name: nagz-comprehensive-review
description: "Comprehensive documentation and code review across Nagz repos (nagz, nagzerver, nagz-web, nagz-ios). Use when asked to start from AGENTS.md, review docs for completeness/consistency, and then review code for incompleteness, edge cases, and inefficiencies, preferably in parallel across repos."
---

# Nagz Comprehensive Review

## Quick Start

1. Open `AGENTS.md` in `~/nagz` and extract constraints, repo layout, and review expectations.
2. Review documentation first, then code.
3. Review repositories in parallel where feasible.
4. Report findings with severity and concrete file references.

## Documentation Review Workflow

- Start with `~/nagz/Docs/*.md` and `~/nagzerver/docs/*.md`.
- Scan for:
  - Incomplete flows, missing prerequisites, or gaps between requirements and implementation.
  - Stale or contradictory statements (e.g., versioning, ports, commands, limits).
  - Missing cross-repo sync steps when API or model changes occur.
  - App review/compliance checklists that are incomplete or mismatch current features.
- Use fast searches for TODO/FIXME/XXX or unresolved placeholders in docs.
- Summarize doc issues by severity and point to exact files.

## Code Review Workflow

- Run quick scans per repo for:
  - TODO/FIXME/XXX, `pass`, `NotImplementedError`, or stubbed methods.
  - Error handling gaps, missing validation, unsafe defaults.
  - Edge cases in date/time logic, rate limiting, auth token handling, pagination, and concurrency.
  - Inefficiencies: N+1 queries, unbounded loops, redundant network calls, large in-memory transforms.
- Prefer parallel repo scans when practical (separate searches per repo).
- For each repo, prioritize:
  - `nagzerver`: endpoints, models, background scheduler, rate limiting, policy enforcement.
  - `nagz-web`: API client usage, auth/session handling, error boundaries, state providers.
  - `nagz-ios`: async/await usage, actor isolation, caching, sync, and API version checks.
- If you can't fully inspect everything, sample high-risk modules and call out what was sampled.

## Output Format

- Begin with a short summary (2–4 bullets) of overall health and top risks.
- Then list findings ordered by severity (Critical, High, Medium, Low).
- Each finding must include:
  - Problem statement
  - Impact/risk
  - Evidence with file references (exact file paths)
  - Suggested fix or follow-up
- End with:
  - What you did not review (if any)
  - Suggested next steps (if any)

## Execution Constraints

- Do not request escalated permissions unless strictly required.
- Prefer `rg` for searches and avoid heavyweight tooling unless necessary.
