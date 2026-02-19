# Nagz — Agent Reference

Context file for LLMs and AI agents working on the Nagz ecosystem.

## What Is Nagz?

Nagz is a family task-reminder ("nagging") app. Guardians create nags for family members, who can complete them, submit excuses, or snooze. The system supports escalation, gamification, AI-powered suggestions, and push notifications.

## Ecosystem Layout

| Repo | Path | Stack | Purpose |
|------|------|-------|---------|
| **nagz** | `~/nagz` | Markdown | Specs, documentation, central hub |
| **nagzerver** | `~/nagzerver` | Python 3.12, FastAPI, SQLAlchemy (async), PostgreSQL, Redis | API server (source of truth) |
| **nagz-web** | `~/nagz-web` | React, TypeScript, Vite | Web client |
| **nagz-ios** | `~/nagz-ios` | SwiftUI, Swift 6, iOS 17+ | iOS client |

All repos live side-by-side under `~/`. There is no monorepo — each has its own git history.

## Server (nagzerver)

### Tech Stack
- Python 3.12, FastAPI, SQLAlchemy (async with asyncpg), PostgreSQL, Redis
- Package manager: `uv`
- Linter: `ruff`
- Tests: `pytest` (190 tests)

### API Routers

The server exposes a RESTful JSON API at `/api/v1/`. Routers:

| Router | Endpoints |
|--------|-----------|
| `auth` | signup, login, logout, token refresh |
| `accounts` | profile, data export (GDPR), account deletion |
| `families` | create, join, list members, manage roles |
| `nags` | CRUD, status transitions, filtering |
| `escalation` | escalation phase tracking, recompute |
| `ai` | AI-powered nag suggestions and excuses |
| `consents` | parental consent management (COPPA) |
| `deliveries` | push notification delivery tracking |
| `devices` | device/APNS token registration |
| `gamification` | points, streaks, leaderboards |
| `incentives` | reward management |
| `legal` | privacy policy, terms of service (no auth) |
| `policies` | role-based permission matrix |
| `preferences` | user preference CRUD |
| `reports` | nag completion reports |
| `safety` | abuse reporting, content moderation |
| `sync` | client data synchronization |
| `version` | server/API version info (no auth) |

### Key Concepts

- **Roles**: guardian, participant, child (guardian and participant can nag anyone; children cannot create nags)
- **Auth**: JWT access + refresh tokens; dev tokens use format `dev:<user-uuid>`
- **Rate limiting**: Redis-backed sliding window (reads 120/min, writes 60/min)
- **Daily nag cap**: 8 per creator per family per local day
- **Event systems**: NagEvent (nag lifecycle) + AuditEvent (system-wide)
- **Background scheduler**: checks for due nags every 60s, sends APNS push notifications
- **Soft delete**: account deletion sets status to "deleted" (recoverable)

### Database
- PostgreSQL on port 5433 (dev)
- Connection: `postgresql+asyncpg://nagz:nagz@localhost:5433/nagz`
- Migrations: Alembic (`uv run alembic upgrade head`)

### Common Commands
```bash
uv run nagz-server          # start API server (port 9800)
uv run pytest               # run tests
uv run ruff check .         # lint
uv run alembic upgrade head # run migrations
```

## Web Client (nagz-web)

### Tech Stack
- React, TypeScript, Vite
- API client auto-generated from `openapi.json` via orval
- Tests: vitest + @testing-library/react + jsdom (126 tests)

### Architecture
- `AuthProvider` wraps app, stores JWT in sessionStorage
- `VersionProvider` checks API compatibility on mount
- `ErrorBoundary` wraps entire app (React class component)
- Axios interceptor dispatches `nagz:unauthorized` on 401 → auto-logout
- 30s timeout on all API requests

### Key Files
| File | Purpose |
|------|---------|
| `src/App.tsx` | Root component, routing |
| `src/auth.tsx` | AuthProvider, login/signup |
| `src/version.tsx` | VersionProvider, CLIENT_API_VERSION |
| `src/members.tsx` | MembersProvider, family member context |
| `src/api/` | Auto-generated API client (do not edit manually) |
| `src/components/` | UI components |

### Common Commands
```bash
npm run dev           # start dev server (port 3000)
npm run build         # production build
npm run api:generate  # regenerate TS client from openapi.json
npx vitest run        # run tests
```

## iOS Client (nagz-ios)

### Tech Stack
- SwiftUI, MVVM, iOS 17+ (@Observable)
- Swift 6 strict concurrency
- Dependencies: KeychainAccess (SPM), GRDB (SPM)
- Project generated with xcodegen from `project.yml`

### Architecture

**Services (all actors — Sendable by default):**

| Service | Purpose |
|---------|---------|
| `APIClient` | Networking, in-memory cache, auto token refresh on 401 |
| `KeychainService` | Secure token storage |
| `SyncService` | Background data synchronization |
| `AIService` | Server-side AI features |
| `OnDeviceAIService` | On-device AI processing |
| `PushNotificationService` | APNS registration and handling |
| `VersionChecker` | API compatibility check on launch |

**Key patterns:**
- `AuthManager` is `@Observable @MainActor` — drives auth UI state, never use from App Intents
- No singletons — services created fresh in `NagzApp.init()`
- JSON: `convertFromSnakeCase` / `convertToSnakeCase` handles all field mapping
- Dev server: `http://127.0.0.1:9800/api/v1` (use IP, not localhost, to avoid IPv6 timeout)

**Models** (`Nagz/Models/`): AuthModels, NagModels, FamilyModels, EscalationModels, GamificationModels, PolicyModels, SafetyModels, ConsentModels, DeviceModels, PreferenceModels, AIModels, and more.

**Views** (`Nagz/Views/`): organized by feature — Auth, Nags, Family, Guardian, Safety, Gamification, Components.

### Siri & Shortcuts
- 6 App Intents: CreateNag, CompleteNag, ListNags, CheckOverdue, SnoozeNag, FamilyStatus
- `IntentServiceContainer` provides shared lazy services for intents
- User/family IDs persisted to UserDefaults for intent access

### Common Commands
```bash
xcodegen generate     # regenerate Xcode project (required after adding/removing Swift files)
xcodebuild build -project Nagz.xcodeproj -scheme Nagz -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'
xcodebuild test -project Nagz.xcodeproj -scheme Nagz -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'
```

### Swift 6 Gotchas
- Actors are already Sendable — do NOT add `nonisolated(unsafe)` to static lets of actor types
- `AppEntity.defaultQuery` must be a computed property (not stored var)
- Use `@Suite(.serialized)` in Swift Testing when tests share UserDefaults
- Use `nonisolated(unsafe)` only for non-Sendable static properties (e.g. RelativeDateTimeFormatter)

## Cross-Project Sync

After any API or model change in nagzerver:

1. Regenerate `openapi.json`:
   ```bash
   cd ~/nagzerver
   uv run python -c "from nagz.server.main import create_app; import json; app = create_app(); print(json.dumps(app.openapi(), indent=2))" > docs/openapi.json
   cp docs/openapi.json ../nagz-web/openapi.json
   ```
2. Regenerate TypeScript client:
   ```bash
   cd ~/nagz-web && npm run api:generate
   ```
3. Update `nagz-web` UI components if affected
4. Update `nagz-ios` models in `Nagz/Models/` and `Nagz/Services/APIEndpoint.swift` if affected
5. Run `xcodegen generate` if any Swift files were added/removed

## API Versioning

| Repo | File | Constants |
|------|------|-----------|
| nagzerver | `src/nagz/core/version.py` | `SERVER_VERSION`, `API_VERSION`, `MIN_CLIENT_VERSION` |
| nagz-ios | `Nagz/Services/VersionChecker.swift` | `clientAPIVersion` |
| nagz-web | `src/version.tsx` | `CLIENT_API_VERSION` |

- `GET /api/v1/version` (no auth) returns server_version, api_version, min_client_version
- Client compares its `CLIENT_API_VERSION` against `min_client_version`
- Client < min_client_version → blocks app; client major < server major → warns

## Port Registry

| Project | Port | Purpose |
|---------|------|---------|
| nagzerver | 9800 | API server |
| nagz-web | 3000 | Vite dev server |
| PostgreSQL | 5433 | Dev database |
| Redis | 6379 | Rate limiting / caching |

## Test Suites

| Repo | Tests | Command |
|------|-------|---------|
| nagzerver | 190 | `cd ~/nagzerver && uv run pytest` |
| nagz-web | 126 | `cd ~/nagz-web && npx vitest run` |
| nagz-ios | 209 | `cd ~/nagz-ios && xcodebuild test -project Nagz.xcodeproj -scheme Nagz -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'` |
| **Total** | **525** | |

## Documentation

Spec and review docs in `~/nagz/Docs/`:

| Document | Purpose |
|----------|---------|
| `APP_REVIEW_GUIDE.md` | Apple App Store reviewer guide |
| `APP_STORE_SUBMISSION.md` | Full submission checklist |
| `TESTFLIGHT_TEST_PLAN.md` | Beta tester walkthrough |
| `CODE_REVIEW_PLAN.md` | Code review methodology |
| `CODE_REVIEW_2026-02-19.md` | Review results (101 findings) |
| `DEPLOYMENT_PLAN.md` | TestFlight → App Store deployment plan |

Server-side docs in `~/nagzerver/docs/`:

| Document | Purpose |
|----------|---------|
| `API_SURFACE.md` | Endpoint inventory |
| `ARCHITECTURE.md` | Data model, error codes, rate limits |
| `REQUIREMENTS.md` | Roles, features, nag model |
| `POLICY_MATRIX.md` | Role-pair permissions |
| `PREFERENCES.md` | Preference schema |
| `SAFETY_AND_COMPLIANCE.md` | Rate limits, App Store checklist |

## GitHub

- Owner: billdonner
- Repos: `billdonner/nagzerver`, `billdonner/nagz-web`, `billdonner/nagz-ios`, `billdonner/nagz`
