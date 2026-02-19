# Comprehensive Code Review — 2026-02-19

Full ecosystem review across all 3 repos. **101 total findings.**

---

## Summary

| Repo | Critical | High | Medium | Low | Cross-Repo | Total |
|------|----------|------|--------|-----|------------|-------|
| nagzerver | 3 | 8 | 16 | 11 | 2 | 40 |
| nagz-web | 3 | 5 | 10 | 10 | — | 28 |
| nagz-ios | 3 | 7 | 12 | 11 | — | 33 |
| **Total** | **9** | **20** | **38** | **32** | **2** | **101** |

---

# NAGZERVER (Python/FastAPI)

## Critical

| # | File | Description |
|---|------|-------------|
| C1 | `core/config.py:15` | **Hardcoded JWT secret** `"change-me-in-production"` used in dev mode. If a dev server is exposed, attacker can forge JWTs. |
| C2 | `schemas/auth.py:14` | **No password max_length.** Allows multi-MB passwords causing bcrypt DoS. Add `max_length=128`. |
| C3 | `server/middleware.py:107-169` | **Rate limit TOCTOU race.** Check-then-record allows burst traffic to exceed limits. Use atomic Redis pipeline or Lua script. |

## High

| # | File | Description |
|---|------|-------------|
| H1 | `server/routers/sync.py:51-54` | Sync loads ALL nag IDs (up to 5000) into memory for `IN(...)` clause. Use subquery instead. |
| H2 | `models/tables.py` | **Missing indexes** on Nag.family_id, Nag.status, NagEvent.nag_id, FamilyMembership.family_id, GamificationEvent columns, Consent.user_id. |
| H3 | `services/scheduler.py:183-186` | Scheduler fetches ALL open nags every 60s with no limit or index. Full table scan at scale. |
| H4 | `services/accounts.py:90-237` | `accounts/export` fetches ALL user data with no pagination or size cap. OOM risk. |
| H5 | `models/tables.py:38` | `date_of_birth` uses `DateTime` instead of `Date`. Stores unnecessary precision. |
| H6 | `services/families.py:88` | **Joiners default to guardian role.** Anyone with invite code gets full admin. Default to `participant`. |
| H7 | `services/nags.py` (create_nag) | **No block enforcement** on nag creation. Blocked user can still nag their blocker. |
| H8 | `schemas/incentives.py:9-13` | `condition` and `action` are unvalidated bare `dict`. Arbitrary JSON accepted. |

## Medium

| # | File | Description |
|---|------|-------------|
| M1 | `services/gamification.py:32-38` | Returns `str(family_id)` but schema expects `uuid.UUID`. Works via coercion but fragile. |
| M2 | `services/gamification.py:56-80` | Same UUID string/object mismatch in leaderboard. |
| M3 | `services/reports.py:29-32` | Same UUID string/object mismatch in reports. |
| M4 | `server/routers/accounts.py:38` | Export endpoint is GET but CLAUDE.md says POST. Documentation mismatch. |
| M5 | `server/routers/accounts.py:38` | Missing `response_model` on export endpoint. No OpenAPI documentation. |
| M6 | `server/routers/accounts.py:14` | Unused import: `require_self_or_guardian`. |
| M7 | `services/accounts.py:240-246` | `_age_from_dob` doesn't handle timezone-naive dates consistently. |
| M8 | `services/nags.py:243` | `state` filter is raw `str`, not `NagStatus` enum. Invalid values silently return empty. |
| M9 | `services/preferences.py:86-91` | Preferences accepts arbitrary keys with no validation of names or value types. |
| M10 | `schemas/accounts.py:14` | `date_of_birth` typed as `datetime` instead of `date`. |
| M11 | `services/ai.py:405` | Compares enum column to string literal instead of using `MembershipStatus.active`. |
| M12 | `services/nags.py:389` | Excuse `prompt_type` mismatch between submit ("excuse") and AI summarize ("excuse_summary"). |
| M13 | `schemas/gamification.py:44` | `badge_type` returns `str` instead of `BadgeType` enum. OpenAPI loses enum info. |
| M14 | `services/badges.py:37-43` | `perfect_week` badge defined in enum but never checked/awarded. |
| M15 | `services/policies.py:57-58` | Policy owners stored as JSON strings, compared as UUIDs. Fragile conversion. |
| M16 | `services/nags.py:279-280` | **Streak never resets on miss.** `streak_delta=0` means streak is a running sum, not consecutive. |

## Low

| # | File | Description |
|---|------|-------------|
| L1 | `models/tables.py:4` | Unused import: `Date`. |
| L2 | `services/nags.py:313,355` | `Recurrence` imported inside functions instead of at module level. |
| L3 | `server/routers/legal.py:206` | TOS contains placeholder `[Your State]` for governing law. |
| L4 | `services/families.py:29` | Invite code is 48-bit entropy (12 hex chars). Could be brute-forced. Use 24+ chars. |
| L5 | `models/enums.py:96-98` | `PushbackMode` enum defined but never used in any table. |
| L6 | Multiple routers | Inconsistent lazy vs top-level error imports across files. |
| L7 | `services/reports.py:89-96` | Dual-type handling for SQLite vs PostgreSQL enum comparisons. Test environment doesn't mirror prod. |
| L8 | `models/tables.py:282` | `UserPreference.updated_at` missing `onupdate=func.now()`. Always shows creation time. |
| L9 | Tests | Missing tests for: block enforcement, concurrent rate limiting, scheduler integration, preference validation, large data export, `perfect_week` badge, monthly recurrence edge cases. |
| L10 | `schemas/safety.py:56-63` | `ConsentResponse` missing `granted_at` and `revoked_at` fields. |
| L11 | `models/tables.py:40,53` | `User.status` and `Family.status` are free-form strings, not enums. |

## Cross-Repo Sync

| # | Description |
|---|-------------|
| X1 | `SyncedNag` schema missing `strategy_template`, `recurrence`, `parent_nag_id`, `policy_id` that exist on `NagResponse`. iOS sync gets reduced data. |
| X2 | `ExcuseCreate.category` typed as `str` but server AI uses `ExcuseCategory` enum. Clients can send invalid values. |

---

# NAGZ-WEB (React/TypeScript)

## Critical

| # | File | Description |
|---|------|-------------|
| C1 | `components/Login.tsx:7-14` | **Dev credentials in prod bundle.** `DEV_FAMILY_ID` and `DEV_USERS` included in production JS. Gate with `import.meta.env.DEV` or dynamic import. |
| C2 | `auth.tsx:37-39` | **Family ID not cleared on logout.** `nagz_family_id` persists in localStorage. Next user could inherit previous family context. |
| C3 | `App.tsx:27-44` | **GuardianRoute allows access during loading.** While members load, `role` is undefined, so children can briefly see guardian pages. Show loading spinner instead. |

## High

| # | Files | Description |
|---|-------|-------------|
| H1 | All components | `familyId` read from `localStorage` independently in every component. Not reactive. Should be centralized in context. |
| H2 | Multiple | **Missing useEffect dependencies** causing stale closures: FamilyDashboard, NagList, Policies, Reports, KidView. |
| H3 | `components/KidView.tsx:93-102` | `eslint-disable` hides real dependency issue. `cancelled` flag never actually prevents stale state updates. |
| H4 | `components/KidView.tsx:39-91` | **N+1 API calls.** Fetches all family nags, filters client-side, then N requests for excuses + N for escalations. No cancellation. |
| H5 | `api/axios-instance.ts:5` | Default API URL is `localhost:8001` but port registry says **9800**. Local dev won't work without `VITE_API_URL`. |

## Medium

| # | File | Description |
|---|------|-------------|
| M1 | `auth.tsx:30` | **userId null in production.** Only extracts from `dev:` tokens. JWT mode returns null userId, breaking role checks everywhere. |
| M2 | Multiple | **No pagination support.** All list components show only server default page size. No "load more" or page controls. |
| M3 | `api/axios-instance.ts` | No CSRF protection. Verify server doesn't use cookies for auth. |
| M4 | Multiple | Error state persists across navigation/actions. Not cleared consistently. |
| M5 | `components/NagList.tsx:209` | Detail modal shows stale data from list cache. Not refetched from server. |
| M6 | FamilyDashboard, Safety | Uses native `confirm()`/`alert()` dialogs. Blocks main thread, looks unprofessional. |
| M7 | `components/Deliveries.tsx:15-20` | `STATUS_COLORS` redefined locally with different keys than shared version. Confusing. |
| M8 | Multiple | No client-side length validation on text inputs. Long text breaks layout. |
| M9 | `version.tsx:87-88,103` | Brittle type assertions in JSX. Use discriminated union narrowing instead. |
| M10 | `components/Safety.tsx:61-76` | `setState` after `logout()` may execute on unmounted component. |

## Low

| # | File | Description |
|---|------|-------------|
| L1 | `KidView.tsx:93` | `cancelled` flag declared but never used to prevent state updates. |
| L2 | `CreateNag.tsx:2` | Minor: unused `Link` import in modal context. |
| L3 | Multiple | Inline styles throughout. Defeats CSS caching. |
| L4 | Multiple modals | No keyboard/Escape handling. Accessibility gap. |
| L5 | All components | Almost zero ARIA attributes. Only 1 `aria-label` in entire codebase. |
| L6 | `NagList.tsx:156-166` | Sort computed on every render. Wrap in `useMemo`. |
| L7 | `Gamification.tsx:6` | Imports both `axios` and `customInstance`. Use `extractErrorMessage` for consistency. |
| L8 | `NagList.tsx:37-53` | No loading indicator during filter change. Old data visible until refetch completes. |
| L9 | `IncentiveRules.tsx:82-92` | Unsafe type casts from `unknown` without validation. Could produce `undefined`/`NaN`. |
| L10 | `App.tsx:160` | `path="*"` silently redirects to home. No 404 feedback. |

---

# NAGZ-IOS (SwiftUI)

## Critical

| # | File | Description |
|---|------|-------------|
| C1 | `NagzApp.swift:63` | **Force-unwrap** of `syncService!`. Safe in current flow but latent fragility. Use guard with fallback. |
| C2 | `AppDelegate.swift:4,32` | `@unchecked Sendable` with mutable `var pushService`. Theoretical data race. Use `@MainActor` isolation. |
| C3 | `Config/AppEnvironment.swift:18` | **Dev URL port 8001 vs registry 9800.** All local API calls fail. |

## High

| # | File | Description |
|---|------|-------------|
| H1 | `Extensions/Date+Formatting.swift:10` | `shortFormatter` (DateFormatter) missing `nonisolated(unsafe)`. Not Sendable-safe. |
| H2 | `Services/APIClient.swift:10` | **In-memory cache grows unbounded.** No max size or LRU eviction. Memory leak risk. |
| H3 | `Services/APIClient.swift:82-208` | `performRequestWithData` and `performRequest` are near-duplicate methods (~60 lines each). DRY violation. |
| H4 | `Services/SyncService.swift:20-29` | `try?` on Task.sleep swallows CancellationError. `stopSync()` only takes effect after sleep completes. |
| H5 | `Views/Nags/NagListView.swift:77-81` | Sheet onDismiss uses fire-and-forget Task with no error handling. |
| H6 | `Models/PolicyModels.swift:11-15` | **PolicyResponse CodingKeys conflict** with `convertFromSnakeCase` decoder. Double-conversion mismatch may cause runtime decoding failures. |
| H7 | `Services/APIClient.swift:123-131` | No distinction between "refresh token expired" and "network error during refresh" in 401 handling. `try?` swallows actual error. |

## Medium

| # | File | Description |
|---|------|-------------|
| M1 | `Services/PushNotificationService.swift:16-22` | Uses old-style completion handler. Could use async/await on iOS 15+. |
| M2 | `Services/AuthManager.swift:32-38` | Init fires unstructured Task to set handler. Could complete after first API call. |
| M3 | `Services/APIEndpoint.swift:334,337` | Creates new `ISO8601DateFormatter` each call. Expensive. Use shared constant. |
| M4 | `Views/Nags/NagDetailView.swift:113-125` | Snooze and mark-complete share `isUpdating` state. Both buttons show loading simultaneously. |
| M5 | `ViewModels/CreateNagViewModel.swift:26` | `dueAt > Date()` re-evaluates on each check. Form may invalidate while user is typing. |
| M6 | `Models/PreferenceModels.swift:17-59` | `AnyCodableValue` doesn't handle arrays or nested objects. Complex prefs decode as `.null`. |
| M7 | Multiple views | Missing accessibility labels on StatusDot, StatusPill, EscalationBadge, icon buttons. |
| M8 | `Views/Safety/AccountView.swift:118-132` | `deleteAccount` may leave app in logged-in state if logout fails after server deletion. |
| M9 | `ViewModels/NagDetailViewModel.swift:95` | **Snooze adds to original dueAt, not current time.** Overdue nag snoozed 15m stays in the past. |
| M10 | `Intents/Actions/SnoozeNagIntent.swift:20` | Same snooze-from-dueAt bug as M9. |
| M11 | `Intents/Actions/CompleteNagIntent.swift:17` | Invalid UUID throws `NagzIntentError.notLoggedIn`. Misleading error message. |
| M12 | `Services/AuthManager.swift:79-86` | **Logout doesn't clear local sync cache.** Next user could see stale GRDB data from previous user. |

## Low

| # | File | Description |
|---|------|-------------|
| L1 | `Extensions/EnvironmentValues+API.swift` | `@Environment(\.apiClient)` defined but never used. All views take apiClient as init param. |
| L2 | `Views/Nags/NagListView.swift:4` | `@State var viewModel` missing `private` access control. |
| L3 | `ViewModels/ManageMembersViewModel.swift:22-23` | `apiClient` and `familyId` are `let` instead of `private let`. |
| L4 | `Views/Gamification/GamificationView.swift:122-156` | Badge type magic strings. Should reference enum or documented constants. |
| L5 | `Views/Family/PreferencesView.swift:23-30` | Quiet hours uses plain TextField. No time format validation. Use DatePicker. |
| L6 | `Services/SyncService.swift` | No retry/backoff on sync failures. Fixed interval regardless of consecutive failures. |
| L7 | `Views/Family/ConsentListView.swift` | No loading indicator on initial consent load. |
| L8 | `Database/DatabaseManager.swift:28-31` | `inMemory()` creates temp SQLite files never cleaned up. |
| L9 | `Views/Guardian/PolicyListView.swift:36` | Uses `UUID.self` for navigationDestination. Could collide with nag navigation if stacks merge. |
| L10 | Various | Inconsistent access control across ViewModels. |
| L11 | `Config/Constants.swift:7` + `Services/VersionChecker.swift:16` | `clientAPIVersion` duplicated in two places. Maintenance risk. |

---

# Cross-Ecosystem Issues

| # | Severity | Description |
|---|----------|-------------|
| 1 | **High** | **Port mismatch across all 3 repos.** iOS uses 8001, web uses 8001, port registry says 9800. All local dev broken without env overrides. |
| 2 | **Medium** | **Snooze behavior inconsistent.** Web (KidView) and iOS both add minutes to original dueAt instead of current time. Server has no validation preventing past due dates on update (fixed in earlier session for create, but not for snooze). |
| 3 | **Medium** | **Sync schema drift.** `SyncedNag` missing fields (`recurrence`, `parent_nag_id`, `strategy_template`, `policy_id`) that `NagResponse` has. iOS offline mode shows incomplete data. |
| 4 | **Medium** | **Logout doesn't clear all state.** Web: localStorage family_id persists. iOS: GRDB cache persists. Both risk data leakage between users on shared devices. |
| 5 | **Low** | **Streak calculation broken.** Misses don't reset streak (server). All clients display inflated streak values. |

---

# Priority Fix Order

1. **Port alignment** (C3-ios, H5-web, port registry) — blocks all local dev
2. **GuardianRoute auth bypass during loading** (C3-web) — security
3. **Family ID / sync cache not cleared on logout** (C2-web, M12-ios) — data leakage
4. **Password max_length** (C2-server) — DoS vector
5. **Rate limit race condition** (C3-server) — security
6. **Joiners default to guardian** (H6-server) — privilege escalation
7. **Missing database indexes** (H2-server) — performance at scale
8. **Block enforcement on nag creation** (H7-server) — feature gap
9. **Snooze from current time, not dueAt** (M9/M10-ios, web KidView) — UX bug
10. **Streak reset on miss** (M16-server) — gamification correctness
