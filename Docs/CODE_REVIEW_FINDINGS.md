# Code Review Findings — 2026-02-18

Comprehensive review across all 3 Nagz repos using the [Code Review Plan](CODE_REVIEW_PLAN.md).

## Summary

| Repo | Critical | Major | Minor | Nit | Test Gaps | Total |
|------|----------|-------|-------|-----|-----------|-------|
| nagzerver | 5 | 9 | 15 | 7 | 10 | 51 |
| nagz-web | 3 | 6 | 12 | 4 | 8 | 33 |
| nagz-ios | 3 | 7 | 12 | 3 | 5 | 29 |
| **Total** | **11** | **22** | **39** | **14** | **23** | **113** |

---

## Critical Issues (Must Fix Before Ship)

### nagzerver

| # | File | Issue | Status |
|---|------|-------|--------|
| S1 | `services/nags.py:77-79` | Participant can nag guardians — role check missing | FIXED |
| S2 | `core/config.py:15` | JWT secret defaults to `"change-me-in-production"` — no startup guard | FIXED |
| S3 | `services/apns.py:47` | Synchronous `open()` blocks event loop in async APNS path | FIXED |
| S4 | `schemas/auth.py:12-16` | No email format validation on signup/login | FIXED |
| S5 | `schemas/auth.py:14` | No password strength validation — empty passwords allowed | FIXED |

### nagz-web

| # | File | Issue | Status |
|---|------|-------|--------|
| W1 | `components/Login.tsx:3-11` | Hardcoded dev credentials and family ID shipped to browser | FIXED |
| W2 | `auth.tsx:19,25` + `axios-instance.ts:9` | Auth token in localStorage — XSS exfiltration risk | FIXED |
| W3 | `axios-instance.ts:17-26` | 401 interceptor bypasses React state, causes full page reload | FIXED |

### nagz-ios

| # | File | Issue | Status |
|---|------|-------|--------|
| I1 | `NagzApp.swift:25` | `try!` on DatabaseManager init — crashes on failure | FIXED |
| I2 | `Models/ReportModels.swift:9-19` | `ReportMetrics` CodingKeys conflict with `convertFromSnakeCase` — production decode failure | FIXED |
| I3 | `Models/AIModels.swift:44` | `ToneSelectResponse.missCount7d` mismatches `convertFromSnakeCase` → `missCount7D` | FIXED |

---

## Major Issues

### nagzerver (9)

| # | File | Issue |
|---|------|-------|
| S6 | `services/nags.py:82` | Daily nag cap uses UTC midnight, not user's local day |
| S7 | `services/nags.py:54-126` | No validation that `due_at` is in the future |
| S8 | `services/nags.py:54-126` | No check preventing self-nagging (`creator_id == recipient_id`) |
| S9 | `routers/preferences.py:29,61` | `db.commit()` on GET endpoints (side effect on read) |
| S10 | `routers/sync.py:51` | N+1 subquery pattern in sync endpoint |
| S11 | `routers/accounts.py:38-46` | `POST /accounts/export` should be GET (pure read) |
| S12 | `routers/ai.py:43,63,83` | Calls private `ai_svc._get_nag()` instead of `nag_svc.get_nag()` |
| S13 | `services/scheduler.py:68-85` | Quiet hours check uses UTC, not user timezone |
| S14 | `schemas/families.py:9` | No length validation on free-text fields (name, reason, description) |

### nagz-web (6)

| # | File | Issue |
|---|------|-------|
| W4 | `axios-instance.ts:52-59` | Deprecated `CancelToken` API (use `AbortController`) |
| W5 | Multiple components | `useEffect` hooks with missing dependency arrays / stale closures |
| W6 | All data-fetching components | No cleanup / abort on unmount in async useEffects |
| W7 | 10 components | Non-null assertion (`!`) on `userId` throughout codebase |
| W8 | 10 components | `familyId` read from localStorage on every render, not in React state |
| W9 | `api/endpoints/*` | Generated API endpoint functions are never used (all calls are manual) |

### nagz-ios (7)

| # | File | Issue |
|---|------|-------|
| I4 | `AuthManager.swift:56` | Force-unwrap after TOCTOU nil check across `await` boundary |
| I5 | `APIClient.swift:203-217` | `CacheEntry.data: Any` is not Sendable |
| I6 | `NagCategoryAppEnum.swift:23` | Force-unwrap in `nagCategory` computed property |
| I7 | `AuthenticatedTabView.swift:19-21` | `UUID()` fallback for nil `currentUser` generates random ID |
| I8 | SyncService + NagzApp | GRDB sync never started; AI service created but unused |
| I9 | `PushNotificationService.swift:34-39` | Device token registration errors silently swallowed |
| I10 | `PolicyModels.swift:11-15` | Custom CodingKeys with `convertFromSnakeCase` is confusing |

---

## Minor Issues (39 total)

### nagzerver (15)
- Unused imports (`update`, `date`, `Date`)
- Duplicate `_get_nag()` function in `ai.py`
- `get_escalation` and `recompute_escalation` are identical
- Report/gamification endpoints lack `response_model`
- `suspend_relationship` returns hand-built dict
- `submit_excuse` returns hand-built dict
- Hardcoded version `"0.2.0"` instead of `SERVER_VERSION`
- `list_excuses` returns dict instead of `PaginatedResponse`
- Daily nag cap test creates 50 nags but cap is 8
- `recurrence` field accepted but never implemented
- Rate limit adds request before checking count
- Database session has no explicit rollback on error
- Lazy imports inside function bodies
- Guardian auth check duplicated inline
- Manual ORM-to-dict conversion instead of Pydantic `from_attributes`

### nagz-web (12)
- Duplicate `formatPhase` and `statusLabel` functions
- Unused `STATUS_COLORS` in NagList
- `as string[]` type assertion on `owners` in Policies
- Swallowed errors in 6+ catch blocks
- Unreachable catch block in Reports (`Promise.allSettled`)
- `void resp` unused variable workaround
- Gamification shows `+` prefix even for negative delta
- `navigator.clipboard.writeText` without error handling
- Single ErrorBoundary for entire app
- `DONE_DEFS` and `CATEGORIES` duplicated between components
- `loadNags` doesn't reset error on retry
- Mid-file import in CreateNag

### nagz-ios (12)
- SyncService weak self captured only once
- `ConsentViewModel.hasConsent` doesn't check revocation
- No `.missed` filter option
- Date validation race between render and submit
- New formatter created on every call in list views
- `EnvironmentValues.apiClient` injected but never read
- `ExcuseModels` custom CodingKeys fragile with `convertFromSnakeCase`
- Unconditional reload on sheet dismiss
- Synchronous write in actor (should be async)
- `apiClient` should be private in ViewModels
- Property `error` shadows catch variable
- No client-side email format validation

---

## Test Gaps (23 total)

### nagzerver (10)
- T1: No test that participant CANNOT nag guardians (test actually asserts success!)
- T2: No test creating nag with past due date
- T3: No test for self-nagging
- T4: No test for deleted user auth rejection
- T5: No test for empty family boundary
- T6: No concurrent nag status update test
- T7: No refresh token replay attack test
- T8: No unauthorized report access test
- T9: No data export structure validation test
- T10: No guardian auth check for `suspend_relationship`

### nagz-web (8)
- T11: No test for MemberSettings save flow
- T12: No test for Policies approval flow
- T13: No test for Safety actions (report, block, delete)
- T14: No test for NagList edit flow
- T15: No test for KidView markComplete/snooze/excuse flows
- T16: No test for Gamification 403 / consent path
- T17: No test for FamilyDashboard add/remove member
- T18: No network error / timeout component tests

### nagz-ios (5)
- T19: No APIClient tests (refresh, cache, error mapping)
- T20: No SyncService integration tests
- T21: No AuthManager state machine tests
- T22: No negative ViewModel tests (API failures)
- T23: No OnDeviceAIService cache-dependent method tests

---

## Priority Fix Order

1. **iOS CodingKeys conflicts** (I2, I3) — will cause production decode failures
2. **iOS `try!` crash** (I1) — app won't launch if DB init fails
3. **Server role check** (S1) — core business rule violation
4. **Server auth validation** (S4, S5) — allows garbage accounts
5. **Server JWT default** (S2) — auth bypass in production
6. **Web dev credentials** (W1) — ships secrets to browser
7. **iOS force-unwraps** (I4, I6, I7) — potential crashes
8. **Web 401 handling** (W3) — poor UX, state inconsistency
