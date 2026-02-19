# Nagz — Claude Code Progress Log

Auto-updated by Claude Code sessions. Monitor remotely via GitHub:
`https://github.com/billdonner/nagz/blob/main/PROGRESS.md`

---

## 2026-02-19 — Session 4: Metrics Endpoint + Gap Analysis + Cross-Repo Sync

### Metrics Endpoint (nagzerver)
- **Created `GET /api/v1/metrics`** — unauthenticated monitoring endpoint
- Returns 22 metrics: uptime, RPS (with sparkline history), memory RSS/VMS, CPU%, user/family counts, nags by status, ack rate, nag events, deliveries by channel, failed deliveries, AI mediation events, gamification events, open abuse reports, active blocks
- **RateCounter utility class** — thread-safe sliding-window request counter with sparkline history
- **ServerInfo model** added to response (name, version, uptime)
- **Added psutil dependency** to pyproject.toml
- **Request-counting middleware** wired into FastAPI app
- **mark_start()** called in lifespan handler for uptime tracking

### Gap Analysis (all 3 repos vs specs)
- Full spec-vs-implementation audit across 62 API endpoints and 20+ features
- **Result: excellent coverage** — all v1.0 endpoints implemented on server, 57/62 on iOS, 62/62 on web (generated client)
- All 6 v2.0 Siri intents implemented

### Gap Fixes
- **iOS: Added per-user preference endpoints** (`getUserPreferences`, `updateUserPreferences`) — guardians can now view/edit child preferences
- **iOS: Added `POST /accounts`** endpoint for authenticated registration flow
- **Spec: Added Section 20 (Metrics)** to API_SURFACE.md documenting the new endpoint
- **Web: Regenerated TypeScript client** from updated openapi.json — picked up metrics endpoint + 15 new model types

### Cross-Repo Sync
- Regenerated openapi.json (56 paths) from nagzerver FastAPI app
- Copied to nagz-web and ran Orval to regenerate TypeScript client
- All 525 tests passing (190 + 126 + 209)
- Committed and pushed all 4 repos

---

## 2026-02-19 — Session 3: Deep Consistency Audit

### Documentation Consistency Audit (cross-codebase)
- **Identified 11 issues** across docs in nagz, nagzerver, nagz-ios, nagz-web
- **Synced 3 divergent docs** from nagzerver → nagz/Docs: REQUIREMENTS.md (self-nag rule), PREFERENCES.md (flat schema), AI_ARCHITECTURE.md (iOS 17+)
- **Fixed API_SURFACE.md** in both repos: added `GET /gamification/badges`, fixed participant role annotation
- **Fixed CATALOG.md** in nagz: corrected release doc paths (Docs/ vs nagz/Docs/), added CODE_REVIEW_PLAN/FINDINGS entries
- **Fixed CATALOG.md** in nagzerver: added 3 unindexed docs (DEMO, APP_STORE_DESCRIPTION, PRODUCT_DESCRIPTION)
- **Fixed port registry** in ~/CLAUDE.md: nagz-web 3000 → 5173 (Vite default, matches CORS config)
- **Deduplicated iOS clientAPIVersion**: VersionChecker now delegates to Constants.Version (single source)
- **Updated MEMORY.md**: user preference for maximum auto-approval of all tool calls
- **Committed and pushed** all 3 repos (nagz, nagzerver, nagz-ios)

---

## 2026-02-19 — Session 2 (continued)

### Doc Consistency Audit & Fixes
- **Audited** all docs across 4 repos for consistency (17 inconsistencies found)
- **Fixed test counts**: nagzerver 183→190, total 518→525 in CLAUDE.md, AGENTS.md
- **Fixed port references**: dev server 8001→9800 in AGENTS.md, nagz-ios/CLAUDE.md
- **Fixed schema_version**: PREFERENCES.md examples updated from 1→2
- **Added default join role**: `participant` documented in REQUIREMENTS.md (both repos)
- **Updated APP_STORE_SUBMISSION.md**: privacy/TOS URLs reference server endpoints
- **Updated CATALOG.md**: added DEPLOYMENT_PLAN.md and CODE_REVIEW entries
- **Updated AGENTS.md**: fixed stale CODE_REVIEW_FINDINGS.md → CODE_REVIEW_2026-02-19.md
- **Fixed iOS CLAUDE.md**: `@unchecked Sendable` → `@MainActor` for AppDelegate
- **Committed and pushed** all 3 repos (nagz, nagzerver, nagz-ios)

### Deployment Plan
- **Created** `Docs/DEPLOYMENT_PLAN.md` — comprehensive 7-phase plan
- Covers: Fly.io server, Vercel web, Apple Developer, TestFlight, App Store metadata, App Review, post-launch
- Timeline: 5 weeks, cost: ~$22-32/month
- **Committed and pushed** to nagz

## 2026-02-19 — Session 1

### Comprehensive Code Review Fixes (101 findings)
- **Top 5 critical fixes** applied across all 3 repos:
  1. Port alignment 8001→9800 (iOS + web)
  2. GuardianRoute auth bypass fix (web)
  3. Logout cleanup — family_id leak (web + iOS)
  4. Password max_length=128 bcrypt DoS prevention (server)
  5. Rate limit TOCTOU race condition — atomic ZADD pattern (server)

- **Remaining 17 nagzerver fixes**: JWT secret warning, joiner default role, block enforcement, UUID consistency, enum validation, badge logic, streak reset, 8 database indexes, unused imports, explicit updated_at

- **5 nagz-web fixes**: dev credential tree-shaking, dead code removal, snooze from current time, axios import fix

- **7 nagz-ios fixes**: force-unwrap elimination, `@MainActor` AppDelegate, cached ISO8601DateFormatter, snooze from current time, intent error messages

- **All tests passing**: 190 (server) + 126 (web) + 209 (iOS) = 525
- **Committed and pushed** all repos

---

*Last updated: 2026-02-19T21:00Z*
