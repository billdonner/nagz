# Nagz — Claude Code Progress Log

Auto-updated by Claude Code sessions. Monitor remotely via GitHub:
`https://github.com/billdonner/nagz/blob/main/PROGRESS.md`

---

## 2026-02-28 — Session 25: Onboarding Refresh & Re-run Button

### iOS App (nagz-ios) — Build 28

**Onboarding Carousel Updated**
- Page 2 "Your Family Hub": now mentions inline AI weekly digest, member avatars, invite sharing (was just roles)
- Page 3 replaced: "Smart Escalation" → **"Connect & Nag Anyone"** — People tab, tap-to-nag, per-connection stats
- Page 4 replaced: "Earn Points & Streaks" → **"AI-Powered Insights"** — analysis, celebrity AI personalities, coaching tips
- Page 5 "Stay in the Loop": now covers push notifications + streaks + badges (was snooze/excuses)

**Re-run Onboarding from Family Tab**
- Added `isRerun` parameter to `OnboardingView` — when `true`, last page shows "Done" (dismisses sheet) instead of "Get Started" (sets `hasSeenOnboarding`)
- New "What's New in Nagz" button (sparkles icon) in Family tab, just above Log Out
- Presents onboarding carousel as a `.sheet` so users can review features anytime

### Build
- Build number bumped 27 → 28, xcodegen regenerated
- Committed and pushed to `experimental/ai-integration`

---

## 2026-02-28 — Session 24: Connection Safety, Nag Detail Layout, Tap-to-Nag

### iOS App (nagz-ios) — Builds 26–27

**Critical Bug Fix: Accidental Connection Revoke**
- **Root cause**: Inline X (revoke) button in connection rows was too easy to tap accidentally — all 3 active connections were revoked in the database
- **Database repair**: Restored revoked connections directly in Fly.io Postgres (`jamesdonner@gmail.com` and `billdonner@gmail.com` set back to `active`)
- **Revoke replaced with swipe-to-delete** (Build 26): Removed inline X button entirely; connections now require swipe-left to reveal "Remove" action
- **Confirmation alert**: "Remove Connection?" dialog names the person and warns "You'll need to re-invite them to reconnect"

**Nag Detail Layout Fix**
- **Gap eliminated**: Mark Complete, Submit Excuse, and Snooze were 3 separate single-button sections — each had insetGrouped spacing creating a huge visual gap. Consolidated into a single "Actions" section

**Tap Connection to Create Nag**
- Tapping an active connection on People tab opens New Nag sheet with that person pre-selected as recipient
- `CreateNagView` gains `preselectedConnectionId` parameter — auto-selects recipient after recipients load
- `ConnectionListView` now receives `familyId` and `currentUserId` from `AuthenticatedTabView` for proper nag creation context

### Deployment
- **TestFlight**: 1.3.0 (27) uploaded to App Store Connect
- **Deployed to both phones**: Titanic + rowboat through Build 27
- Committed and pushed to `experimental/ai-integration`

---

## 2026-02-27 — Session 23: Family Tab Overhaul, People Tab, AI Personality, Inline Digest

### iOS App (nagz-ios) — Builds 16–25

**Family Tab**
- **AI digest hex bug fixed**: `FamilyInsightsView` was showing raw UUID hex — now shows "Member" fallback text and "?" for avatar initials
- **Stale cache fix**: `ManageMembersViewModel` now invalidates `/families` cache after create/remove; family tab reloads on appear
- **Flicker fix**: Loading spinner only shown on first load (`family == nil`); subsequent refreshes update silently
- **Send Invite**: `ShareLink` in Invite Code section — opens iOS share sheet pre-filled with family name and invite code
- **Horizontal member avatars** (Build 24): Members displayed as horizontal scroll with 52pt color-coded avatar circles (blue=guardian, orange=participant, green=child), display name, role label, and star badge for current user
- **Inline AI digest** (Build 25): "This Week" card at top of family page with sparkle icon, AI summary text, Done/Open/Missed stats, completion percentage, and "Details" link to full FamilyInsightsView — only shows when there's useful data (`totalNags > 0`)

**Nagz Tab**
- **AI Analysis redesign** (Build 20): Replaced bland `.alert()` with rich `AISummarySheet` — shows overdue nags (red), due-soon nags (orange) with category icons and relative times, AI summary text, stat badges. Half-sheet with drag-to-expand
- **Sparkle button moved**: From `.automatic` (next to `+`) to `.topBarLeading` (left side). Shows spinner while generating

**People Tab**
- **Connection stats** (Build 21): Each active connection now shows Sent/Received/Open/Done stat badges with color-coded icons. Uses `TaskGroup` for concurrent per-connection API fetches
- **Tap fix** (Build 23): Added `.buttonStyle(.borderless)` to Toggle and revoke Button — fixed bug where tapping anywhere on a connection row triggered the revoke action

**AI Personality** (Build 22)
- 6 celebrity-inspired AI personalities: Standard, Simon Cowell, Susie Greene (Curb), Mr. Rogers, Gordon Ramsay, Coach
- Picker in Account settings via `@AppStorage("nagz_ai_personality")`
- Personality prompt injected into list summary, coaching, and push-back prompts

### NagzAI Package (nagz-ai)
- **`AIPersonality` enum**: 6 personalities with `displayName`, `tagline`, `promptInstruction`
- **Personality in prompts**: `ListSummaryPrompt`, `CoachingPrompt`, `PushBackPrompt` all inject personality instruction
- **Urgency emphasis**: List summary prompt rewritten to lead with what needs attention RIGHT NOW

### Deployment
- **Server**: nagzerver deployed to fly.io (bd-nagzerver.fly.dev)
- **TestFlight**: 1.3.0 (25) uploaded to App Store Connect
- **Deployed to both phones**: Titanic (iPhone 15 Pro Max) + rowboat (iPhone SE) through Build 25
- Committed and pushed to `experimental/ai-integration`

---

## 2026-02-27 — Session 22: Web Urgency Colors, Header Rename, Dep Updates & Deploy

### Web App (nagz-web)
- **Urgency-colored nag rows**: Ported iOS 5-tier urgency system to TypeScript
  - New `urgencyTier()` function in `nag-utils.ts`: calm (>24h), approaching (2–24h, blue), dueSoon (<2h, yellow), overdue (<1h past, orange), critical (>1h past, red)
  - `URGENCY_COLORS` for due-date text color, `URGENCY_BORDER` for 3px left accent bar
  - Applied to both "For Me" and "Nagz to Others:" table rows in `NagList.tsx`
- **Section header rename**: "For Others" → "Nagz to Others:" (matches iOS)
- **Test updated**: `phase3-coverage.test.tsx` assertion updated to match new header
- **Dependency patches**: axios 1.13.6, react-router-dom 7.13.1, eslint 9.39.3, typescript-eslint 8.56.1, orval 8.5.1
- **Build**: clean (272 modules), **Tests**: 126/126 passing
- Committed and pushed to `main`

### iOS App (nagz-ios)
- **Deployed to both phones**: Titanic (iPhone 15 Pro Max) + rowboat (iPhone SE)

---

## 2026-02-27 — Session 21: On-Device AI Architecture Docs & Website Update

### Documentation (nagz hub)
- **APP_STORE_SUBMISSION.md**: Expanded AI feature bullets with 7/9 on-device LLM detail; enhanced privacy section clarifying no text leaves device
- **APP_REVIEW_GUIDE.md**: Added full 9-row AI operations table (on-device/heuristic/server columns), privacy guarantee section, routing explanation to section 5b; updated section 12 data privacy verification
- **TESTFLIGHT_TEST_PLAN.md**: Expanded known limitations with per-operation on-device vs heuristic breakdown

### Server Docs (nagzerver)
- **APP_STORE_DESCRIPTION.md**: New "On-device AI intelligence" section with 7/9 detail, heuristic fallback messaging, privacy guarantee

### Website & iOS App (nagz-ios)
- **docs/index.html**: New "On-Device AI" marketing section on landing page — Apple Foundation Models, privacy messaging
- **docs/privacy.html**: New section 2g explaining on-device AI data processing and no external AI services
- **docs/support.html**: Expanded "What does the AI do?" FAQ with on-device detail; added new "Does Nagz send data to external AI?" FAQ (answer: no)
- **NagListView.swift**: Renamed section header "For Others" → "Nagz to Others:"
- All committed and pushed across 3 repos

---

## 2026-02-27 — Session 20: Smart Defaults for New Nag Sheet

### iOS App (nagz-ios)
- **Smart defaults**: New Nag form now pre-selects category and completion type based on recipient history
  - Queries local GRDB `cached_nags` table for most frequently used values per creator→recipient pair
  - Triggers on recipient selection in `onChange(of: recipientId)` — instant, no network call
  - User can still override via pickers; if no history exists, defaults stay as chores/ackOnly
- **New file**: `EnvironmentValues+Database.swift` — `DatabaseManager` environment key (follows existing AI/API pattern)
- **DatabaseManager**: Added `nagDefaults(creatorId:recipientId:)` query method using GRDB reader pool
- **CreateNagViewModel**: Added `applySmartDefaults(db:creatorId:recipientId:)` — maps raw strings to enum values
- **CreateNagView**: Reads `databaseManager` from environment; calls smart defaults after recipient context is set
- **NagzApp**: Injects `.environment(\.databaseManager, databaseManager)` into view hierarchy
- **All 202 iOS tests pass**, committed and pushed to `experimental/ai-integration`

---

## 2026-02-27 — Session 19: Urgency-Colored Nag List Cells

### iOS App (nagz-ios)
- **NagRowView**: Added private `Urgency` enum computing tier from `dueAt` vs `Date()` for `.open` nags
  - 5 tiers: calm (>24h), approaching (2–24h, blue tint), due soon (<2h, yellow bar), overdue (<1h past, orange bar), critical (>1h past, red bar)
  - Row gets background tint, 3px leading accent bar, and urgency-colored due-date text
  - Completed/missed nags unaffected (always calm)
- **ChildNagRowView**: Added matching `ChildUrgency` enum
  - Shadow color/radius intensifies with urgency (blue → orange → red, 4px → 10px)
  - Due-date text color follows same tier scheme
- **No API changes, no new files** — purely client-side computation from existing `dueAt` field
- **All 202 iOS tests pass**, committed and pushed to `experimental/ai-integration`

---

## 2026-02-27 — Session 18: Docs Update, Version Bump, Deploy to Phones

### Documentation Updates (nagz hub)
- **README.md**: Expanded AI architecture — now lists 7/9 LLM operations with heuristic fallback detail
- **APP_STORE_SUBMISSION.md**: Richer AI feature bullet ("personalized coaching tips, smart tone selection, weekly family digests, gamification nudges, and task summaries"), updated privacy section
- **APP_REVIEW_GUIDE.md**: Version → 1.3.0 (Build 14), added §5b.3 Gamification Nudges and §5b.4 List Summary sections, NagzAI dependency noted, on-device AI description expanded
- **TESTFLIGHT_TEST_PLAN.md**: Expanded AI test matrix (nudges, list summary, LLM-generated text), updated known limitations to reflect 7/9 LLM coverage

### iOS App (nagz-ios)
- **Version bump**: 1.2.0 → 1.3.0, build 13 → 14
- **README.md**: AI feature table rewritten — 9-row table showing LLM/heuristic/server split per operation
- **xcodegen regenerated** with new version
- **Deployed to both phones**: Titanic (iPhone 15 Pro Max) + rowboat (iPhone SE)

---

## 2026-02-27 — Session 17: Injectable TextGenerator for Testable LLM Paths

### NagzAI Package (nagz-ai)
- **New `TextGenerator` protocol** (`Sources/NagzAI/TextGenerator.swift`) — single-method `String → String` abstraction for LLM calls
- **New `FoundationModelsTextGenerator`** (`Providers/FoundationModelsTextGenerator.swift`) — real implementation gated behind `#if canImport(FoundationModels)`, wraps `LanguageModelSession`
- **Refactored `FoundationModelsProvider`** — added `textGenerator` property with two inits:
  - `init()` — production: wires real generator (or nil on non-FM platforms)
  - `init(textGenerator:)` — test-only: inject any mock
- **Eliminated per-method `#if canImport` gates** — all 7 LLM methods now use `guard let textGenerator else { heuristicFallback }`; the `#if canImport` gate is only in `init()` and the gated file
- **14 new tests** in `FoundationModelsProviderTests.swift` with `MockTextGenerator`:
  - Summarize excuse (LLM text + heuristic category, truncation)
  - Select tone (firm + supportive with LLM reason)
  - Coaching (LLM tip + heuristic scenario/category)
  - Digest (LLM summaryText + preserved structured fields)
  - Push back (LLM message + heuristic tone, shouldPushBack always true)
  - List summary (pure LLM text)
  - Gamification nudges (personalized, fallback on mismatch, empty passthrough)
  - Patterns + predictCompletion stay heuristic (ignore textGenerator)
- **110 tests total** (96 existing + 14 new) — all passing
- **Docs updated**: CLAUDE.md (key files, pattern note), SPEC.md (§5.5, test table, v1.2.0)

---

## 2026-02-27 — Session 16: Expand Foundation Models to All NagzAI Operations

### NagzAI Package (nagz-ai)
- **6 new prompt files**: ExcusePrompt, TonePrompt, DigestPrompt, PushBackPrompt, ListSummaryPrompt, GamificationPrompt
- **Design pattern**: Each prompt has `build()` (constructs system prompt) + `parse()` (extracts LLM text, heuristic for structured fields)
- **FoundationModelsProvider upgraded**: 7 of 9 operations now use on-device LLM via `#if canImport(FoundationModels)`
- **2 operations stay heuristic-only**: `patterns` (pure date counting), `predictCompletion` (pure math)
- **Hybrid approach**: LLM generates text fields (summary, reason, message, tip); enums, numbers, dates computed heuristically
- **For digest + gamification**: heuristic runs FIRST for structured data, then LLM overlays natural text
- **37 new tests** across 6 test files (96 total, up from 59) — all passing
- **Docs updated**: CLAUDE.md, SPEC.md (v1.1.0) with all new prompt specs

---

## 2026-02-27 — Session 15: Gamification Nudges + iOS 26 Deployment Target

### NagzAI Package (nagz-ai)
- **New types**: `GamificationContext`, `GamificationNudgeItem`, `GamificationNudgeResult` — context + results for streak/badge nudges
- **New protocol method**: `gamificationNudges(context:)` added to `NagzAIService`
- **HeuristicProvider**: Implemented streak-at-risk alerts (5 tiers: no streak → early → mid → long → legendary) and badge proximity nudges (first_completion, streak_3, streak_7, streak_30, century_club)
- **FoundationModelsProvider**: Delegates to heuristic (nudges are threshold-based)
- **ServerProvider**: Throws `.notAvailable` (on-device only)
- **Router**: Routes `gamificationNudges` to selected provider
- **8 new tests** in RouterTests: no-streak, early, mid, long, legendary, first-completion, priority-sort, custom-provider
- **59 tests pass** (up from 51)

### iOS 26 Deployment Target
- **Package.swift**: swift-tools-version 5.10 → 6.2, platforms `.iOS(.v17)/.macOS(.v14)` → `.iOS(.v26)/.macOS(.v26)`
- **project.yml**: `iOS: "17.0"` → `iOS: "26.0"`, regenerated Xcode project
- **FoundationModelsProvider simplified**: Removed `#available(iOS 26, ...)` runtime guards and `isAvailable` check fallback — Foundation Models always available with iOS 26 baseline
- **Router comment**: Updated tier description to reflect iOS 26 baseline

### iOS (nagz-ios)
- **GamificationViewModel**: Added `nudges` property, loads AI nudges after badges via `NagzAI.Router().gamificationNudges()`
- **GamificationView**: Added "Tips" section above "Your Stats" displaying icon + message rows from AI nudges

### Documentation Updates (all repos)
- Updated 12+ doc files across nagz, nagz-ai, nagz-ios replacing "iOS 17" → "iOS 26"
- Key files: APP_REVIEW_GUIDE.md, APP_STORE_SUBMISSION.md, TESTFLIGHT_TEST_PLAN.md, AI_ARCHITECTURE.md, SIRI_SHORTCUTS.md, CODE_REVIEW_PLAN.md, AGENTS.md, CLAUDE.md (all repos), README.md, SPEC.md

---

## 2026-02-26 — Session 14: AI-Powered UI Surfaces + TestFlight Build 12

### iOS (nagz-ios) — `experimental/ai-integration` branch
- **Created `EnvironmentValues+AI.swift`** — `\.aiService` environment key mirroring existing `\.apiClient` pattern
- **Wired AIService** in `NagzApp.swift` — `.environment(\.aiService, aiService)` alongside `.environment(\.apiClient, apiClient)`
- **Created `AIInsightsSection.swift`** (~160 lines) — inline AI section for NagDetailView:
  - Tone badge (color-coded capsule: blue=neutral, green=supportive, red=firm) with reason text
  - Coaching tip (lightbulb icon + tip + scenario caption)
  - Completion prediction (percentage gauge + suggested reminder time)
  - Three-tier fallback: NagzAIAdapter (GRDB cache) → ServerAIService → direct NagzAI.Router heuristics
  - Derives synthetic context from nag status/category so insights vary per nag
- **Created `FamilyInsightsView.swift`** (~145 lines) — guardian-only Family tab view:
  - Weekly digest: summary text, per-member completion stats, totals footer
  - User patterns: day-of-week miss insights from 90-day analysis
  - Pull-to-refresh + ProgressView loading state + graceful error handling
- **Modified `NagDetailView.swift`** — inserted `AIInsightsSection(nagId:nag:)` after Details section
- **Modified `AuthenticatedTabView.swift`** — added "AI Insights" section with Family Insights NavigationLink (guardian-only)
- **NagzAIAdapter** — removed cache-freshness gate on nag-specific operations (local heuristics preferred when nag is cached)
- **Debugging cycle**: fixed 403 consent error on devices without GRDB cache, fixed identical insights across nags
- **202 tests pass**, clean build, verified on both phones (Titanic + rowboat)

### TestFlight Distribution
- **Bumped** version to 1.2.0 (build 12)
- Archived, uploaded to App Store Connect
- Release notes set: AI Insights feature summary
- Auto-distributed to "family-naggers" internal beta group

### Documentation Updates (nagz hub + nagz-ios)
- **AI_ARCHITECTURE.md** — updated Section 4.3 (`OnDeviceAIService` → `NagzAIAdapter`), added Section 4.4 (iOS UI surfaces), updated implementation checklist (steps 5-8 marked DONE)
- **APP_REVIEW_GUIDE.md** — added AI Insights to Nag Detail table + new Section 5b (AI Insights testing)
- **TESTFLIGHT_TEST_PLAN.md** — added AI Insights test scenarios
- **nagz-ios/CLAUDE.md** — added AI architecture notes (environment key, adapter, views)

---

## 2026-02-26 — Session 13: Integrate nagz-ai Package into nagz-ios

### nagz-ai Package
- Standalone Swift package at `~/nagz-ai` with Router (auto-selects Foundation Models on iOS 26+ / Heuristic fallback)
- 42 tests, zero dependencies — all passing

### iOS (nagz-ios) — `experimental/ai-integration` branch
- **Added NagzAI** as local SPM dependency in `project.yml`
- **Created `NagzAIAdapter.swift`** (228 lines) — actor conforming to `AIService`, bridges GRDB context → NagzAI Router → app response types
- **Replaced `OnDeviceAIService`** entirely (deleted)
- **Coaching** — proper scenario detection (`first_miss`, `streak_3`, `repeated_time_conflict`, `high_completion_homework`) instead of random tip
- **Digest** — works locally when cache is fresh (was always server-only fallback)
- **PushBack** — works locally when cache is fresh (was always server-only fallback)
- **Foundation Models** — auto-selected on iOS 26+ via Router; heuristic fallback on iOS 17–25
- Updated `NagzApp.swift` wiring (3-line swap)
- Updated tests in `ServiceAndViewModelTests.swift` and `Phase2CoverageTests.swift` to use `NagzAIAdapter(preferHeuristic: true)`
- **202 tests pass**, clean build

---

## 2026-02-26 — Session 12b: Nagz rename, For Me / For Others sections, v1.1.11

### iOS (nagz-ios)
- **NagListView** — split into "For Me" (recipient is current user) and "For Others" sections with `.insetGrouped` style
- **AuthenticatedTabView** — tab label renamed from "Nags" to "Nagz"
- Navigation title changed to "Nagz"
- **Version bumped** to 1.1.11 (build 9)
- Deployed to both phones: Titanic (iPhone 15 Pro Max) and rowboat (iPhone SE)

### Web (nagz-web)
- **NagList** — heading changed from "All Nagz" to "Nagz", split into "For Me" and "For Others" sections
- "For Me" table shows "From" column; "For Others" table shows "To" column
- Updated `phase3-coverage.test.tsx` to test both sections (126 tests pass)

---

## 2026-02-26 — Session 12: WebSocket Real-Time Updates + APNs Smart Push

### Server — Event Bus + WebSocket Hub (nagzerver)
- **New `services/events.py`** — central event publisher via Redis pub/sub with JSON serialization
- **New `routers/ws.py`** — WebSocket endpoint `WS /api/v1/ws?token=...&family_id=...` with JWT auth, Redis pub/sub subscription, ping/pong keepalive, connection tracking
- **Nag routers** — `publish_event()` called after nag create, update, status change, excuse submit
- **Family routers** — `publish_event()` called after member add/remove
- **Smart push** — APNs notifications for `nag_created` (→ recipient) and `nag_completed` (→ creator), skipped when user has active WebSocket
- **8 new tests** for event publishing, serialization, and router integration (242 total pass)

### iOS — WebSocketService (nagz-ios)
- **New `WebSocketService` actor** — connects to `wss://server/api/v1/ws`, decodes JSON events via `AsyncStream<NagEvent>`, auto-reconnects with exponential backoff (1s → 30s), keepalive pings every 25s
- **NagListView** — subscribes to WebSocket events, reloads on nag mutations; **removed 30-second polling timer**
- **ChildNagListView** — subscribes to WebSocket events for real-time updates
- Threaded through `NagzApp → ContentView → AuthenticatedTabView/ChildTabView`

### Web — useWebSocket Hook (nagz-web)
- **New `useWebSocket` hook** — opens WS connection, auto-reconnects with exponential backoff, exposes `eventCount` for triggering refreshes
- **FamilyDashboard** — auto-refreshes nag counts on WebSocket events
- **NagList** — auto-refreshes nag list on events
- **KidView** — auto-refreshes nags/excuses on events
- All 126 web tests pass

### Test Results
| Repo | Tests | Status |
|------|-------|--------|
| nagzerver | 242 | All pass |
| nagz-web | 126 | All pass |
| nagz-ios | Build succeeds | No regressions |
| **Total** | **368+** | **All pass** |

---

## 2026-02-26 — Session 11: Child Login (Family Code + Username + PIN)

### Database & Models (nagzerver)
- **Alembic migration** adds `child_code` to families, `username`/`pin_hash` to family_memberships, new `child_settings` table
- **Models** updated: `Family.child_code`, `FamilyMembership.username/pin_hash`, new `ChildSettings` ORM class
- **Family service** generates unique 6-char uppercase alphanumeric child codes, hashes PINs with bcrypt

### Server Auth & API (nagzerver)
- **`POST /auth/child-login`** — family code + username + PIN authentication with Redis rate limiting (5 attempts/15 min)
- **`PUT /families/{id}/members/{uid}/credentials`** — guardian sets child username+PIN
- **`PATCH /families/{id}/members/{uid}/pin`** — child changes own PIN
- **`GET/PATCH /families/{id}/children/{uid}/settings`** — guardian manages per-child controls
- **Authorization enforcement** — snooze limits, excuse toggle, quiet hours checked in nag routers
- **15 new tests** (234 total, all passing), committed and pushed

### iOS Client (nagz-ios)
- **Models:** `ChildLoginRequest`, extended `AuthResponse` with `familyId`/`familyRole`, `ChildSettingsResponse/Update`, `ChildCredentialsSet`, `PinChangeRequest`
- **5 new API endpoints** in `APIEndpoint.swift`
- **AuthManager:** `isChildUser` routing, `childLogin()` method, role persistence
- **7 new views:** `ChildLoginView` (PIN pad), `ChildTabView`, `ChildNagListView`, `ChildNagRowView`, `ChildSettingsView`, `ChildControlsView`
- **ContentView** routes child→ChildTabView, adult→AuthenticatedTabView
- **LoginView** adds "I'm a Kid" button
- **ManageMembersView** adds credential management and child controls links
- Build succeeds, 215 tests passing, committed and pushed

### Web Client (nagz-web)
- **Regenerated** TypeScript API client with child login types
- **Login.tsx:** "I'm a Kid" toggle → family code + username + 4-digit PIN form
- **FamilyDashboard.tsx:** displays `child_code` alongside invite code, credential management modal, child controls link per child member
- **ChildSettings.tsx:** new component — guardian per-child snooze/excuse/quiet hours controls
- **members.tsx:** exposes `childCode` from family response
- All 126 web tests passing, committed and pushed

### Test Results
| Repo | Tests | Status |
|------|-------|--------|
| nagzerver | 234 | All passing |
| nagz-web | 126 | All passing |
| nagz-ios | 215 | All passing |
| **Total** | **575** | |

---

## 2026-02-26 — Session 10: Display Names in Web UI

### API Sync
- **Regenerated** openapi.json from nagzerver (new fields: `creator_display_name`, `recipient_display_name` on NagResponse; `other_party_email`, `other_party_display_name` on ConnectionResponse)
- **Regenerated** TypeScript client via orval in nagz-web

### Web UI Updates (nagz-web)
- **NagList.tsx:** Table rows, sort comparisons, and detail modal now show `creator_display_name` / `recipient_display_name` with fallback to member lookup
- **Connections.tsx:** All three sections (inbound invites, sent invites, active connections) now show `other_party_display_name` with fallback to `other_party_email` then `invitee_email`
- **Local interface** updated with new optional fields to match API
- All 126 web tests passing
- Committed and pushed to main

---

## 2026-02-25 — Session 9: Cross-Repo Audit, Doc Sync, Web Signup

### Deep Audit (4 Repos)
- **Launched 4 parallel audit agents** — thorough code review and doc inventory across nagzerver, nagz-web, nagz-ios, and nagz hub
- **nagzerver:** 219 tests, 68 endpoints, 23 models, 79 schemas, 77 service functions — no critical issues, code quality excellent
- **nagz-web:** 126 tests, 15 components, 14 routes, all features implemented — missing signup UI identified
- **nagz-ios:** 215 tests, 29 views, 18 ViewModels, 76 API endpoints, 80+ models — exemplary Swift 6 adoption, no issues
- **nagz hub:** 25 documentation files audited — 3 spec docs missing trusted connections feature

### Documentation Fixes
- **API_SURFACE.md:** Added Section 21 (Connections) with all 8 connection endpoints including PATCH /trust and GET /children
- **REQUIREMENTS.md:** Added trusted connections to Section 4 (Roles and Relationship Rules)
- **POLICY_MATRIX.md:** Added Section 6 (Cross-Family Trusted Connection Rules) with permission table
- **DEPLOYMENT_PLAN.md:** Updated status — marked completed items (server deployed, TestFlight builds shipped, migrations run)
- **Synced** API_SURFACE.md, REQUIREMENTS.md, POLICY_MATRIX.md from nagz hub → nagzerver/docs
- **Fixed test counts:** nagz-ios CLAUDE.md (209→215), README.md (166→215)

### Web Signup UI (Feature Parity)
- **Added signup form** to Login.tsx — toggle between Sign In / Sign Up modes
- Fields: display name (optional), email, password (8+ char validation)
- CSS: `.login-toggle` and `.login-link-btn` styles added
- All 126 web tests passing

### Fly.io Deployment
- Redeployed nagzerver to Fly.io with latest code
- Server verified healthy at `bd-nagzerver.fly.dev`
- Built and installed iOS app to iPhone Titanic

### Code Review Summary

| Repo | Grade | Critical Issues | Notes |
|------|-------|-----------------|-------|
| nagzerver | A | 0 | Well-structured, all endpoints typed, rate limiting robust |
| nagz-web | A- | 0 | Missing signup (now fixed), minor console.error statements |
| nagz-ios | A+ | 0 | Exemplary Swift 6, 100% feature parity, proper actor isolation |
| nagz hub | A- | 0 | 3 docs missing trusted connections (now fixed) |

### Test Summary

| Repo | Tests | Status |
|------|-------|--------|
| nagzerver | 219 | All passed |
| nagz-web | 126 | All passed |
| nagz-ios | 215 | All passed |
| **Total** | **560** | |

---

## 2026-02-25 — Session 8: Trusted Connections (Cross-Family Nagging)

### Feature: Trusted Flag on Connections
- **Purpose:** Allow connected adults to nag each other's children without extra setup
- Added `trusted BOOLEAN DEFAULT FALSE` to connections table (Alembic migration)
- Added `trusted: Mapped[bool]` to Connection model + `trusted` field to ConnectionResponse schema
- New schemas: `ConnectionTrustUpdate`, `TrustedConnectionChild`

### Server Changes (nagzerver)
- **Connection service:** `update_trust()` — toggle trusted flag, cancels trusted-child nags on untrust; `list_trusted_children()` — finds children in other party's guardian families; `revoke_connection()` now resets `trusted = False`
- **Nag service:** Extended connection nag path — if recipient is NOT the other party AND connection is trusted, allows nag if recipient is a child in other party's guardian family
- **New endpoints:** `PATCH /connections/{id}/trust`, `GET /connections/{id}/children`
- **11 new tests** — trust toggle, untrust cancels child nags, trust requires active/party, list trusted children, revoke resets trust, response includes trusted field, trusted/untrusted/random-user nag creation

### Web Changes (nagz-web)
- Regenerated openapi.json with all connection endpoints → orval generated TypeScript client
- **Connections.tsx:** Added "Trusted" checkbox column in Active Connections table with `handleToggleTrust`
- **CreateNag.tsx:** Loads trusted children from trusted connections, displays as optgroup in recipient picker, sends `connection_id` for trusted child nags

### iOS Changes (nagz-ios)
- **ConnectionModels.swift:** Added `trusted: Bool` to `ConnectionResponse`, new `ConnectionTrustUpdate` and `TrustedConnectionChild` structs
- **APIEndpoint.swift:** Added `updateConnectionTrust(id:trusted:)` and `listTrustedChildren(connectionId:)`
- **ConnectionListViewModel:** Added `toggleTrust()` method
- **ConnectionListView:** Added Toggle in active connection rows
- **CreateNagView:** Loads trusted children, shows in recipient picker, maps to `connection_id`
- **6 new tests:** ConnectionResponse decoding (trusted/untrusted), TrustedConnectionChild decoding, ConnectionTrustUpdate encoding, endpoint path tests

### Test Summary

| Repo | Tests | Status |
|------|-------|--------|
| nagzerver | 219 | All passed |
| nagz-web | 126 | All passed |
| nagz-ios | 215 | All passed |
| **Total** | **560** | |

---

## 2026-02-25 — Session 7: Fly.io OOM Fix

### Server Out-of-Memory Crash
- **Problem:** `bd-nagzerver` instance crashed with OOM — uvicorn killed by Linux OOM killer
- **Root cause:** Only 256 MB RAM allocated, with 2 machines running. Python baseline RSS ~120-130 MB left minimal headroom for request spikes and GC pressure
- **Fix:** Scaled RAM 256 MB → 512 MB, reduced machines 2 → 1 (net cost neutral)
- Added `--workers 1 --limit-max-requests 1000` to uvicorn CMD in Dockerfile — auto-restarts worker after 1,000 requests to prevent memory creep
- Redeployed to Fly.io, verified healthy: 107 MB RSS with 512 MB available (75% headroom)
- Load tested with 50+ rapid requests — all correct responses, no instability

---

## 2026-02-25 — Session 6: Share Sheet, Tab Persistence, Auth Fix, Web Connections

### Share Sheet for Connection Invites (iOS)
- After successfully sending an invite, success screen shows with ShareLink button
- User can share via iMessage, WhatsApp, email etc. with pre-written message + nagz.online link
- `invitedEmail` property added to `ConnectionListViewModel` to capture email before clearing

### Session Persistence (iOS)
- **Tab persistence**: Changed `selectedTab` from `@State` to `@AppStorage` — user returns to same tab on relaunch
- **Login persistence**: Already working via Keychain tokens + `restoreSession()` refresh — verified end-to-end

### Token Auth 401 Fix (Server + iOS)
- **Root cause**: Expired/invalid tokens returned 403 (`AuthzDenied`) instead of 401 — iOS never triggered token refresh, showed "You don't have permission" instead
- **Fix**: Changed all authentication failures in `deps.py`, `auth.py`, `services/auth.py` to use `AuthnFailed` (401); `AuthzDenied` (403) now reserved for true authorization failures
- 4 tests updated for new status codes, all 208 passing
- Server redeployed to Fly.io

### People Page — Outbound Invites (iOS)
- Added **"Invites You Sent"** section to People tab showing pending outbound invites with cancel button
- Now loads 3 connection sets: active, inbound pending, all pending (outbound = all - inbound)
- Friendlier duplicate invite error: "You already have a connection with this person" + orange SF Symbol

### Connections Page — Web App (NEW)
- **Created `/connections` route** with full People/Connections page (`Connections.tsx`)
- Invite form with Web Share API (falls back to clipboard copy on desktop)
- Sections: Invites for You (accept/decline), Invites You Sent (cancel), Active Connections (remove)
- Navigation links added to FamilyDashboard and NagList headers
- All 126 web tests passing, deployed to Fly.io

### Device Builds
- Built and installed directly to iPhone "Titanic" (15 Pro Max) via `xcodebuild` + `devicectl`
- Verified login persistence, tab persistence, invite flow, share sheet on physical device

---

## 2026-02-24 — Session 5 (cont'd, part 2): Auth Fixes + Server Redeploy

### Login Error Message Fix
- **Server:** Login failures returned 403 ("authorization denied") instead of 401 — created `AuthnFailed` error class (401) distinct from `AuthzDenied` (403)
- **iOS:** Added `authenticationFailed(String)` case to `APIError` — passes through server's actual message instead of generic "no permission"
- Tests updated, all passing

### Production Password Reset
- Reset password for `billdonner@gmail.com` via SSH into Fly.io production database
- Verified login works against live server

### Build 5 (1.0.0) — TestFlight
- Bumped build 4 → 5 with auth error fix
- Archived, uploaded, release notes set via App Store Connect API
- Server redeployed to Fly.io with matching 401 fix
- **Confirmed working:** login from TestFlight app succeeds

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

*Last updated: 2026-02-25T18:55Z*
