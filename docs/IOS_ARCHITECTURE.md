# Nagz iOS App — Architecture

> Auto-generated from codebase analysis. Last updated: 2026-02-27.

## System Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                          NagzApp (@main)                             │
│  Creates all services, injects via .environment(\.apiClient/aiService)│
└──────────┬──────────┬──────────┬──────────┬──────────┬──────────────┘
           │          │          │          │          │
    ┌──────▼──┐ ┌─────▼────┐ ┌──▼───┐ ┌───▼────┐ ┌───▼──────────┐
    │Keychain │ │APIClient │ │  DB  │ │  WS    │ │AuthManager   │
    │Service  │ │ (actor)  │ │Mgr   │ │Service │ │@Observable   │
    │ (actor) │ │ HTTP +   │ │(GRDB)│ │(actor) │ │@MainActor    │
    │         │ │ cache +  │ │      │ │        │ │              │
    │ tokens  │ │ refresh  │ │SQLite│ │wss://  │ │ login/logout │
    └────┬────┘ └──┬───┬───┘ └──┬───┘ └───┬────┘ └──────┬───────┘
         │         │   │        │         │              │
         │    ┌────┘   │   ┌────┘         │              │
         │    │        │   │              │              │
    ┌────▼────▼──┐ ┌───▼───▼──┐  ┌────────▼───┐  ┌──────▼───────┐
    │ SyncService│ │NagzAI    │  │ Push       │  │ Version     │
    │  (actor)   │ │Adapter   │  │ Service    │  │ Checker     │
    │ polls /sync│ │ (actor)  │  │ (APNs)     │  │ /version    │
    │ every 300s │ │          │  │            │  │             │
    └────────────┘ │ GRDB ctx │  └────────────┘  └─────────────┘
                   │ → Router │
                   │ → Server │
                   │ fallback │
                   └──────────┘
                        │
           ┌────────────┼────────────┐
           ▼            ▼            ▼
     NagzAI.Router  ServerAI    Direct Router
     (GRDB cache)   (HTTP)      (NagResponse)
```

## Navigation Layer

```
ContentView ─┬─ OnboardingView (first run)
             ├─ AuthFlowView (login / signup / child login)
             ├─ AuthenticatedTabView (guardian/participant)
             │   ├─ Tab 0: Nagz ─── NagListView → NagDetailView
             │   │                    └─ AIInsightsSection
             │   ├─ Tab 1: People ── ConnectionListView
             │   └─ Tab 2: Family ── FamilyTabContent
             │        ├─ Members, Settings, Consents
             │        ├─ Guardian Dashboard (Reports, Policies)
             │        ├─ AI Insights → FamilyInsightsView
             │        ├─ Gamification (Points, Incentives)
             │        └─ Safety, Account, Invite Code
             └─ ChildTabView (child)
                 └─ ChildNagListView → NagDetailView
```

## Services Layer (8 Actors + 2 @Observable)

| Service | Type | Responsibility |
|---------|------|----------------|
| KeychainService | actor | Token storage (access + refresh) |
| APIClient | actor | HTTP client, in-memory cache, auto 401 refresh |
| DatabaseManager | actor | GRDB SQLite cache (6 tables) |
| SyncService | actor | Polls `/sync/events` every 300s, upserts to GRDB |
| WebSocketService | actor | Real-time events via `wss://`, auto-reconnect |
| NagzAIAdapter | actor | Routes to NagzAI package, falls back to server |
| ServerAIService | actor | Calls `/ai/*` server endpoints |
| PushNotificationService | @Observable @MainActor | APNs registration + deep linking |
| AuthManager | @Observable @MainActor | Session state, login/logout/refresh |
| VersionChecker | @Observable @MainActor | Client/server version compatibility |

## AI Insights Data Flow (3-Tier Fallback)

```
AIInsightsSection(.task)
│
├─1─ NagzAIAdapter.selectTone(nagId)
│    ├─ Nag in GRDB cache? → NagzAI.Router (local heuristics)
│    └─ Not cached? → ServerAIService (HTTP POST /ai/select-tone)
│         ├─ Server responds? done
│         └─ 403 / error? → fall through to tier 3
│
├─2─ (same for coaching + predictCompletion)
│
└─3─ All failed? → tryDirectHeuristics()
     └─ Build AIContext from NagResponse (status-derived stats)
        └─ NagzAI.Router(preferHeuristic: true) — always works
```

## ViewModels (19 @Observable @MainActor)

| Area | ViewModels |
|------|------------|
| Auth | LoginVM, SignupVM, ChildLoginVM |
| Nags | NagListVM, NagDetailVM, CreateNagVM, EditNagVM |
| Family | FamilyVM, MemberListVM, ManageMembersVM |
| Features | GamificationVM, IncentiveRulesVM, ConnectionListVM |
| Guardian | ReportsVM, PolicyVM, DeliveryHistoryVM, PreferencesVM |
| Safety | SafetyVM, ConsentVM |

## Views (36 Files)

| Directory | Views |
|-----------|-------|
| Auth | LoginView, SignupView, ChildLoginView, ChildSettingsView |
| Nags | NagListView, ChildNagListView, NagDetailView, CreateNagView, EditNagView |
| Family | MemberListView, ManageMembersView, CreateFamilyView, JoinFamilyView, ConsentListView, PreferencesView |
| Gamification | GamificationView, IncentiveRulesView |
| Guardian | ChildControlsView, PolicyListView, PolicyDetailView, ReportsView, DeliveryHistoryView |
| AI | AIInsightsSection, FamilyInsightsView |
| Safety | SafetyView, AccountView, FeedbackMailView, LegalDocumentView |
| Connections | ConnectionListView, InviteConnectionView |
| Onboarding | OnboardingView |
| Components | NagRowView, ChildNagRowView, MemberRowView, ErrorBanner, EscalationBadge |

## Models (21 Files)

| File | Key Types |
|------|-----------|
| NagModels | NagResponse, NagCreate, NagUpdate, NagStatusUpdate |
| AuthModels | LoginRequest, SignupRequest, ChildLoginRequest, AuthResponse |
| FamilyModels | FamilyResponse, MemberResponse, MemberDetail, FamilyCreate |
| AIModels | ToneSelectResponse, CoachingResponse, PredictCompletionResponse, DigestResponse, PatternsResponse |
| ConnectionModels | ConnectionResponse, ConnectionInvite, ConnectionTypeUpdate, CaregiverConnectionChild |
| GamificationModels | GamificationEvent, LeaderboardEntry |
| EscalationModels | EscalationEvent, PhaseTransition |
| ErrorModels | APIError enum with `.isRetryable` |
| PaginatedResponse | Generic `PaginatedResponse<T>` (items, total, offset) |
| CachedRecords | CachedNag, CachedNagEvent, CachedGamificationEvent, SyncMetadata (GRDB) |

## GRDB Cache Tables

| Table | Purpose | Retention |
|-------|---------|-----------|
| cached_nags | Offline nag list | Active + 30 days completed |
| cached_nag_events | Event history for AI | 90 days rolling |
| cached_ai_mediation_events | AI coaching history | 90 days rolling |
| cached_gamification_events | Points/streaks | 90 days rolling |
| cached_preferences | User prefs snapshot | Current |
| sync_metadata | Last sync timestamp | Per entity |

## Siri & Shortcuts (13 Files)

```
IntentServiceContainer ── shared lazy KeychainService + APIClient
NagzShortcutsProvider ── 6 shortcuts, 14 Siri phrases

Actions: CreateNag, CompleteNag, ListNags, CheckOverdue, SnoozeNag, FamilyStatus
Entities: NagEntity, FamilyMemberEntity, NagCategoryAppEnum
Queries: NagEntityQuery, FamilyMemberQuery
```

## Environment Injection

```swift
.environment(\.apiClient, apiClient)   // EnvironmentValues+API.swift
.environment(\.aiService, aiService)   // EnvironmentValues+AI.swift
```

## Key Architectural Patterns

| Pattern | Implementation |
|---------|----------------|
| Actor-based concurrency | All 8 services are `actor` — thread-safe, no data races |
| @Observable for UI | 19 ViewModels + AuthManager — auto-update views |
| Environment injection | apiClient + aiService — testable, no singletons |
| Two-tier AI | Local NagzAI heuristics → server fallback → direct router |
| Dual sync | Polling (SyncService 300s) + WebSocket (real-time events) |
| Keychain tokens | Secure storage, survives reinstall, auto-refresh on 401 |
| GRDB offline cache | Local SQLite mirrors server state for AI + offline |
| Swift 6 strict | Full concurrency checking, Sendable everywhere |

## Stats

| Metric | Count |
|--------|-------|
| Swift files | ~85 |
| Lines of code | ~10,350 |
| Services (actors) | 8 |
| ViewModels | 19 |
| Views | 36 |
| Models | 21 |
| Siri intents | 6 |
| Tests | 202 |
| API endpoints wired | 76 |
