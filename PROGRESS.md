# Nagz — Claude Code Progress Log

Auto-updated by Claude Code sessions. Monitor remotely via GitHub:
`https://github.com/billdonner/nagz/blob/main/PROGRESS.md`

---

## 2026-02-24 — Session 5 (cont'd): TestFlight Builds + Server Deploy

### TestFlight Build 3 (1.0.0)
- Bumped build number 2 → 3
- Created `ExportOptions.plist` for App Store Connect uploads
- Archived, exported, and uploaded via `xcodebuild`
- Set "What to Test" release notes via App Store Connect API
- Release notes saved to `docs/testflight-notes-build3.md`

### Server Deployment (Fly.io)
- Deployed nagzerver + bundled nagz-web to `bd-nagzerver.fly.dev` (release v8)
- Web app now serves new cartoon icon on login page + favicon
- Verified live: API responding, web SPA serving updated assets

### TestFlight Build 4 — Server Connection Fix
- **Bug:** Production URL pointed to `api.nagz.app` (DNS not configured) — TestFlight users couldn't sign in
- **Fix:** Changed production `baseURL` to `https://bd-nagzerver.fly.dev/api/v1` in `AppEnvironment.swift`
- Bumped build 3 → 4, archived, uploaded, release notes set via API
- Testers should update to build 4 for working sign-in

### App Store Connect API Integration
- Authenticated using `AuthKey_KURS4TFJJ9.p8` + Issuer ID via JWT (ES256)
- App ID: `6759530926`
- Automated: find builds, create/update `betaBuildLocalizations` with release notes
- Token generation script at `/tmp/asc_token.py`

---

## 2026-02-24 — Session 5: Onboarding, Branding, Launch Screen, Website Deploy

### iOS Onboarding Sequence
- **Created 5-page swipeable onboarding carousel** (`OnboardingView.swift`)
- Pages: Never Forget Again (bell), Your Family Hub (people), Smart Escalation (arrows), Earn Points & Streaks (flame), Stay in the Loop (notifications)
- Each page has a 72pt hero SF Symbol, headline, subtitle, and supporting icon row
- "Get Started" button on final page, persisted via `@AppStorage("hasSeenOnboarding")`
- Integrated into `ContentView` — gates before auth flow, shown only once

### Login & Signup Page Branding
- **Login page**: Added blue `bell.badge.fill` SF Symbol (56pt) with "Welcome to Nagz" heading
- **Signup page**: Added purple `person.crop.circle.badge.plus` SF Symbol with "Join the Family" heading

### New App Icon
- Replaced old parchment/serif icon with new cartoon artwork (woman nagging, man covering ears, bold "NAGZ")
- Generated all 13 icon sizes from 1024px master via `sips`
- Updated `LaunchLogo` image asset to match

### Launch Screen
- Initially set to parchment background matching old icon
- Updated to white background + new cartoon icon centered
- Dark mode variant: near-black background
- Configured via `UILaunchScreen` in Info.plist + project.yml

### Branding Across All Repos
- **README logos**: Added centered 200px icon to nagz, nagz-ios, nagzerver READMEs
- **nagzerver marketing site** (`nagz.online`): Added icon to hero section + favicon
- **nagz-web**: Added icon to login page (160px, rounded corners) + favicon (replaced Vite default)
- **nagz-ios App Store landing page** (`docs/index.html`): Added 180px rounded icon
- **Deployed to GitHub Pages**: Pushed `gh-pages` branch, verified live at nagz.online

### Integration Test Plan
- **Created `Docs/INTEGRATION_TEST_PLAN.md`** — 223 test scenarios across 15 categories
- 151 fully automatable, 55 semi-auto, 17 manual-only
- Covers auth, family, nags, connections, gamification, push, AI, guardian, safety, Siri, sync, version, cross-component, performance, privacy
- Each test has unique ID (e.g. `INT-AUTH-001`) for tracking

### Development Status Assessment
- **Code is feature-complete**: 65 API endpoints, all implemented on server + iOS + web
- **Remaining work is deployment pipeline**: custom domain, Apple Developer enrollment, TestFlight, App Store submission

### Test Results
- **All 543 tests passing** (208 + 126 + 209), up from 525
- nagzerver gained 18 tests since last session
- Updated test counts in CLAUDE.md
- Committed and pushed all 4 repos

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

*Last updated: 2026-02-24T21:15Z*
