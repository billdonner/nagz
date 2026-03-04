# Nagz Ecosystem Integration Test Plan

> Cross-component integration testing for nagzerver (API), nagz-web (React), and nagz-ios (SwiftUI).

**Last updated:** 2026-02-24

---

## Table of Contents

1. [Environment and Prerequisites](#1-environment-and-prerequisites)
2. [Test Tooling and Conventions](#2-test-tooling-and-conventions)
3. [Auth Flow](#3-auth-flow)
4. [Family Management](#4-family-management)
5. [Nag Lifecycle](#5-nag-lifecycle)
6. [Connections (Non-Family)](#6-connections-non-family)
7. [Gamification](#7-gamification)
8. [Push Notifications](#8-push-notifications)
9. [AI Mediation](#9-ai-mediation)
10. [Guardian Oversight](#10-guardian-oversight)
11. [Safety Controls](#11-safety-controls)
12. [Siri and Shortcuts](#12-siri-and-shortcuts)
13. [Sync and Offline](#13-sync-and-offline)
14. [Version Compatibility](#14-version-compatibility)
15. [Cross-Component Scenarios](#15-cross-component-scenarios)
16. [Performance and Load](#16-performance-and-load)
17. [Data Privacy and Compliance](#17-data-privacy-and-compliance)
18. [Regression Test Matrix](#18-regression-test-matrix)

---

## 1. Environment and Prerequisites

### 1.1 Infrastructure

| Component | Local Dev | Staging |
|-----------|-----------|---------|
| nagzerver | `http://localhost:9800` | `https://bd-nagzerver.fly.dev` |
| nagz-web | `http://localhost:5173` | TBD |
| nagz-ios | Xcode simulator / TestFlight device | TestFlight |
| PostgreSQL | `localhost:5433` | Fly.io managed |
| Redis | `localhost:6379` | Fly.io managed |

### 1.2 Test Accounts

Create these accounts before running integration tests:

| Account | Email | Role | Purpose |
|---------|-------|------|---------|
| Guardian A | `guardian-a@test.nagz.dev` | guardian | Primary family owner |
| Guardian B | `guardian-b@test.nagz.dev` | guardian | Co-owner, second device |
| Participant | `participant@test.nagz.dev` | participant | Standard family member |
| Child (13+) | `teen@test.nagz.dev` | child | Teen child account |
| Child (<13) | `kid@test.nagz.dev` | child | COPPA-gated child account |
| Outsider | `outsider@test.nagz.dev` | (none) | Not in any test family |

### 1.3 Prerequisites Per Component

**nagzerver:**
```bash
cd ~/nagzerver
uv run alembic upgrade head          # apply migrations
uv run pytest                         # 190 unit tests pass
uv run uvicorn nagz.server.main:app --port 9800  # server running
```

**nagz-web:**
```bash
cd ~/nagz-web
npm install
npm run api:generate                  # TS client from openapi.json
npx vitest run                        # 126 unit tests pass
npm run dev                           # Vite on port 5173
```

**nagz-ios:**
```bash
cd ~/nagz-ios
xcodegen generate                     # regenerate project
xcodebuild test -project Nagz.xcodeproj -scheme Nagz \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'
                                      # 209 unit tests pass
```

### 1.4 Test Data Seeding

For repeatable integration tests, use the server's `dev:<user-uuid>` auth tokens (development mode only) or run the seed script:

```bash
cd ~/nagzerver && uv run python -m nagz.cli.main seed-test-data
```

---

## 2. Test Tooling and Conventions

### 2.1 Automation vs Manual

| Label | Meaning |
|-------|---------|
| **AUTO** | Fully automatable via pytest/vitest/XCTest or curl scripts |
| **SEMI** | Automatable setup, but requires manual verification (UI, push, Siri) |
| **MANUAL** | Requires physical device and human interaction |

### 2.2 Test ID Convention

Tests are identified as `INT-{category}-{number}`, e.g. `INT-AUTH-001`.

### 2.3 Assertion Strategy

- **API layer:** Assert HTTP status code, response body shape, database side-effects.
- **Web layer:** Assert rendered UI state after API calls, error banners, loading states.
- **iOS layer:** Assert view model state, GRDB cache contents, navigation flow.
- **Cross-component:** Assert that a mutation on one client is observable from another.

---

## 3. Auth Flow

### 3.1 Signup

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AUTH-001 | Happy path signup | AUTO | `POST /auth/signup` with valid email, password, display_name | 201, user created, JWT pair returned |
| INT-AUTH-002 | Signup on web | SEMI | Fill signup form on nagz-web, submit | Redirected to family creation, tokens stored |
| INT-AUTH-003 | Signup on iOS | SEMI | Fill signup screen, submit | Main screen loads, keychain stores tokens |
| INT-AUTH-004 | Duplicate email | AUTO | Signup twice with same email | 409 or 422, clear error message |
| INT-AUTH-005 | Weak password | AUTO | Signup with password "123" | 422 validation error |
| INT-AUTH-006 | Missing required fields | AUTO | Signup with empty display_name | 422 validation error |
| INT-AUTH-007 | Under-13 signup (COPPA) | AUTO | Signup with DOB making user < 13 years old | Account created in `pending_guardian_consent` status |
| INT-AUTH-008 | Under-13 cannot act without consent | AUTO | Use under-13 token to call `POST /nags` | 403 AUTHZ_DENIED |

### 3.2 Login

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AUTH-010 | Happy path login | AUTO | `POST /auth/login` with valid credentials | 200, access + refresh tokens returned |
| INT-AUTH-011 | Wrong password | AUTO | Login with incorrect password | 401, generic "invalid credentials" message |
| INT-AUTH-012 | Non-existent email | AUTO | Login with unknown email | 401, same generic message (no user enumeration) |
| INT-AUTH-013 | Web login + session restore | SEMI | Login on web, refresh page | Session persists, user stays logged in |
| INT-AUTH-014 | iOS login + session restore | SEMI | Login on iOS, kill app, relaunch | Session restored from keychain, family auto-loaded |

### 3.3 Token Management

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AUTH-020 | Refresh token | AUTO | `POST /auth/refresh` with valid refresh token | New access token, old one invalidated |
| INT-AUTH-021 | Expired access token | AUTO | Wait for expiry or use a pre-expired token | 401 on API call, client auto-refreshes |
| INT-AUTH-022 | Revoked refresh token | AUTO | `POST /auth/logout`, then try refresh | 401 on refresh attempt |
| INT-AUTH-023 | iOS transparent refresh | SEMI | Let access token expire, trigger API call | APIClient auto-refreshes, request succeeds without user action |
| INT-AUTH-024 | Concurrent refresh race | AUTO | Send 5 simultaneous refresh requests | Exactly one succeeds, others get 401 or deduplicated |

### 3.4 Logout

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AUTH-030 | Server logout | AUTO | `POST /auth/logout` | Refresh token revoked |
| INT-AUTH-031 | Web logout | SEMI | Click logout in web UI | Redirected to login, tokens cleared from localStorage |
| INT-AUTH-032 | iOS logout | SEMI | Tap logout in settings | Returned to login, keychain cleared, GRDB cache deleted |

---

## 4. Family Management

### 4.1 Create Family

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-FAM-001 | Create family | AUTO | `POST /families` with name | 201, family_id + invite_code returned, creator is guardian |
| INT-FAM-002 | Create family on web | SEMI | Use family creation form | Family created, invite code displayed with copy button |
| INT-FAM-003 | Create family on iOS | SEMI | Tap "Create Family" | Family created, shown in family list |

### 4.2 Join Family

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-FAM-010 | Join via invite code | AUTO | `POST /families/join` with valid invite_code | Member added with `participant` role |
| INT-FAM-011 | Invalid invite code | AUTO | Join with wrong code | 404 NOT_FOUND |
| INT-FAM-012 | Web join flow | SEMI | Paste invite code in web UI | Joined, family dashboard loads |
| INT-FAM-013 | iOS join flow | SEMI | Enter invite code on iOS | Joined, family nags visible |
| INT-FAM-014 | Already a member | AUTO | Join same family twice | 409 or idempotent success |

### 4.3 Member Management

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-FAM-020 | List members | AUTO | `GET /families/{id}/members` | Paginated list with roles |
| INT-FAM-021 | Add member (guardian) | AUTO | `POST /families/{id}/members` as guardian | Member added with specified role |
| INT-FAM-022 | Add child member (requires consent) | AUTO | Add member with role=child | Requires `child_account_creation` consent |
| INT-FAM-023 | Create member account | AUTO | `POST /families/{id}/members/create` as guardian | New user + membership created |
| INT-FAM-024 | Remove member (guardian) | AUTO | `DELETE /families/{id}/members/{userId}` | Member removed, open nags cancelled as `cancelled_relationship_change` |
| INT-FAM-025 | Non-guardian cannot add members | AUTO | `POST /families/{id}/members` as participant | 403 AUTHZ_DENIED |
| INT-FAM-026 | Non-guardian cannot remove members | AUTO | `DELETE /families/{id}/members/{userId}` as participant | 403 AUTHZ_DENIED |
| INT-FAM-027 | Remove recipient cancels nags | AUTO | Create nag for user, then remove user from family | All open nags for that recipient have status `cancelled_relationship_change` |

### 4.4 Roles and Permissions Matrix

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-FAM-030 | Guardian sees reports | AUTO | `GET /reports/family/weekly?family_id=...` as guardian | 200 with report data |
| INT-FAM-031 | Participant cannot see reports | AUTO | Same endpoint as participant | 403 AUTHZ_DENIED |
| INT-FAM-032 | Child cannot see reports | AUTO | Same endpoint as child | 403 AUTHZ_DENIED |
| INT-FAM-033 | Child cannot create nags for guardian | AUTO | `POST /nags` with child token, recipient=guardian | 403 POLICY_FORBIDDEN |
| INT-FAM-034 | Child cannot create nags for child | AUTO | `POST /nags` with child token, recipient=another child | 403 POLICY_FORBIDDEN |
| INT-FAM-035 | Participant can nag guardian | AUTO | `POST /nags` with participant token, recipient=guardian | 201, nag created |
| INT-FAM-036 | Guardian can nag child | AUTO | `POST /nags` with guardian token, recipient=child | 201, nag created |
| INT-FAM-037 | Self-nag (guardian) | AUTO | `POST /nags` where creator = recipient, guardian role | 201, allowed |
| INT-FAM-038 | Self-nag (child denied) | AUTO | `POST /nags` where creator = recipient, child role | 403 POLICY_FORBIDDEN |

---

## 5. Nag Lifecycle

### 5.1 Create Nag

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-NAG-001 | Create basic nag | AUTO | `POST /nags` with required fields | 201, nag in `open` status |
| INT-NAG-002 | All categories | AUTO | Create nags for each: chores, meds, homework, appointments, other | All succeed, correct category stored |
| INT-NAG-003 | All completion types | AUTO | Create nags with ack_only, binary_check, binary_with_note | All succeed |
| INT-NAG-004 | Recurring daily | AUTO | Create nag with recurrence=daily | Parent nag created, child instances generated |
| INT-NAG-005 | Recurring weekly | AUTO | Create nag with recurrence=weekly | Parent nag + weekly children |
| INT-NAG-006 | Recurring monthly | AUTO | Create nag with recurrence=monthly | Parent nag + monthly children |
| INT-NAG-007 | Past due date rejected | AUTO | `POST /nags` with due_at in the past | 422 VALIDATION_ERROR |
| INT-NAG-008 | Missing required fields | AUTO | `POST /nags` without recipient_id | 422 VALIDATION_ERROR |
| INT-NAG-009 | Daily nag cap enforced | AUTO | Create 9 nags (cap=8 default) for same creator-family-day | 9th request returns 403 POLICY_FORBIDDEN |
| INT-NAG-010 | Create nag on web | SEMI | Use nag creation form in nagz-web | Nag appears in list |
| INT-NAG-011 | Create nag on iOS | SEMI | Use + button on iOS nag list | Nag appears in list with category icon |

### 5.2 Read and Filter Nags

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-NAG-020 | Get single nag | AUTO | `GET /nags/{nagId}` | 200, full nag detail |
| INT-NAG-021 | List open nags | AUTO | `GET /nags?family_id=...&state=open` | Only open nags returned |
| INT-NAG-022 | List completed nags | AUTO | `GET /nags?family_id=...&state=completed` | Only completed nags |
| INT-NAG-023 | List missed nags | AUTO | `GET /nags?family_id=...&state=missed` | Only missed nags |
| INT-NAG-024 | Pagination | AUTO | Create 60 nags, request with limit=50, offset=0, then offset=50 | First page 50 items, second page 10 items |
| INT-NAG-025 | Unauthorized nag access | AUTO | `GET /nags/{nagId}` for nag in different family | 403 or 404 |

### 5.3 Update and Complete Nag

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-NAG-030 | Complete nag (ack_only) | AUTO | `POST /nags/{id}/status` with status=completed | Nag status changes to completed |
| INT-NAG-031 | Complete nag with note | AUTO | `POST /nags/{id}/status` with status=completed, note="done" | Completed with note stored |
| INT-NAG-032 | Only recipient can complete | AUTO | Creator tries to complete recipient's nag | 403 (recipient_only) |
| INT-NAG-033 | Double-complete idempotent | AUTO | Complete same nag twice | Second request is idempotent or returns 409 |
| INT-NAG-034 | Update nag (creator) | AUTO | `PATCH /nags/{id}` to change due_at | 200, updated |
| INT-NAG-035 | Update nag (non-creator) | AUTO | Participant tries to edit guardian's nag | 403 AUTHZ_DENIED |

### 5.4 Escalation Phases

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-NAG-040 | Phase 0: initial delivery | AUTO | Create nag, check escalation state | phase_0_initial |
| INT-NAG-041 | Phase 1: due soon | AUTO | Create nag due in 30 min, wait or simulate | Transitions to phase_1_due_soon |
| INT-NAG-042 | Phase 2: overdue soft | AUTO | Let nag pass due_at without completion | Transitions to phase_2_overdue_soft |
| INT-NAG-043 | Phase 3: bounded push-back | AUTO | Continue without response past phase 2 | Transitions to phase_3_overdue_bounded_pushback |
| INT-NAG-044 | Phase 4: guardian review | AUTO | Unresolved after phase 3 | Transitions to phase_4_guardian_review |
| INT-NAG-045 | Completion cancels escalation | AUTO | Complete nag during phase 2 | Escalation stops, no further phase transitions |
| INT-NAG-046 | Recompute escalation | AUTO | `POST /nags/{id}/escalation/recompute` | Recalculates current phase |
| INT-NAG-047 | Quiet hours block transitions | AUTO | Set quiet hours, trigger escalation during quiet period | Transition deferred until quiet hours end |

### 5.5 Excuses

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-NAG-050 | Submit excuse | AUTO | `POST /nags/{id}/excuses` as recipient | 201, excuse recorded |
| INT-NAG-051 | Excuse categories | AUTO | Submit excuses with each category: forgot, time_conflict, unclear_instructions, lacking_resources, refused, other | All accepted |
| INT-NAG-052 | List excuses (creator) | AUTO | `GET /nags/{id}/excuses` as creator | Paginated excuse list |
| INT-NAG-053 | Recipient cannot list excuses | AUTO | `GET /nags/{id}/excuses` as recipient | 403 (creator_or_guardian only) |
| INT-NAG-054 | Excuse requires AI consent | AUTO | Submit excuse without `ai_mediation` consent | 403 requires_consent |

### 5.6 Deliveries

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-NAG-060 | List deliveries | AUTO | `GET /deliveries?nag_id={id}` | Paginated delivery records |
| INT-NAG-061 | Delivery includes channel and status | AUTO | Check delivery records after nag creation | Records show channel (push/sms) and status |

---

## 6. Connections (Non-Family)

### 6.1 Connection Invite Flow

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-CONN-001 | Send invite | AUTO | `POST /connections/invite` with invitee_email | 201, connection in `pending` status |
| INT-CONN-002 | Accept invite | AUTO | `POST /connections/{id}/accept` as invitee | Connection status changes to `accepted` |
| INT-CONN-003 | Decline invite | AUTO | `POST /connections/{id}/decline` as invitee | Connection status changes to `declined` |
| INT-CONN-004 | Revoke connection | AUTO | `POST /connections/{id}/revoke` by either party | Connection status changes to `revoked` |
| INT-CONN-005 | List connections | AUTO | `GET /connections` | All connections for user, paginated |
| INT-CONN-006 | List pending invites | AUTO | `GET /connections/pending` | Only pending invites TO current user |
| INT-CONN-007 | Filter by status | AUTO | `GET /connections?status=accepted` | Only accepted connections |
| INT-CONN-008 | Invite non-existent email | AUTO | Invite user who hasn't signed up | Invite created (pending registration) or appropriate error |
| INT-CONN-009 | Self-invite rejected | AUTO | Invite your own email | 422 VALIDATION_ERROR |
| INT-CONN-010 | Duplicate invite | AUTO | Send invite twice to same email | Idempotent or 409 |

### 6.2 Connection + Nag Interaction

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-CONN-020 | Nag via connection | AUTO | Create nag for a connected (non-family) user | 201, nag delivered through connection |
| INT-CONN-021 | Nag denied after revoke | AUTO | Revoke connection, then try to create nag | 403 (relationship inactive) |

---

## 7. Gamification

### 7.1 Points and Scoring

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GAM-001 | On-time completion earns points | AUTO | Complete nag before due_at | +10 points gamification event |
| INT-GAM-002 | Late completion earns reduced points | AUTO | Complete nag after due_at | +4 points gamification event |
| INT-GAM-003 | Missed nag earns 0 points | AUTO | Let nag expire to missed | 0 points, no positive gamification event |
| INT-GAM-004 | Points summary | AUTO | `GET /gamification/summary?family_id=...` | Total points, current streak, recent events |
| INT-GAM-005 | Requires gamification consent | AUTO | Request summary without `gamification_participation` consent | 403 requires_consent |

### 7.2 Streaks

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GAM-010 | Streak increments | AUTO | Complete 3 consecutive nags on time | Streak count = 3 |
| INT-GAM-011 | Streak breaks on miss | AUTO | Complete 3, then miss 1 | Streak resets to 0 |
| INT-GAM-012 | Streak survives late completion | AUTO | Complete on time, then late | Streak behavior per policy (may reset or continue) |

### 7.3 Leaderboard

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GAM-020 | Family leaderboard | AUTO | `GET /gamification/leaderboard?family_id=...` | Ranked list of family members by points |
| INT-GAM-021 | Leaderboard on web | SEMI | View leaderboard page in nagz-web | Members ranked with point totals |
| INT-GAM-022 | Leaderboard on iOS | SEMI | View Points & Streaks tab on iOS | Leaderboard with medal indicators |
| INT-GAM-023 | Only intra-family ranking | AUTO | Leaderboard does not include users from other families | No cross-family data leakage |

### 7.4 Badges

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GAM-030 | List earned badges | AUTO | `GET /gamification/badges` | List of badges earned by current user |
| INT-GAM-031 | Badge earned on milestone | AUTO | Complete enough nags to trigger a badge | Badge appears in earned list |
| INT-GAM-032 | Consent revocation stops accrual | AUTO | Revoke `gamification_participation`, then complete nag | No new gamification events; historical data retained |

### 7.5 Incentive Rules

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GAM-040 | List incentive rules | AUTO | `GET /incentive-rules?family_id=...` as guardian | Paginated rules list |
| INT-GAM-041 | Create incentive rule | AUTO | `POST /incentive-rules` as guardian | 201, rule created |
| INT-GAM-042 | Update incentive rule | AUTO | `PATCH /incentive-rules/{id}` as guardian | Rule updated |
| INT-GAM-043 | Non-guardian cannot create rules | AUTO | `POST /incentive-rules` as participant | 403 AUTHZ_DENIED |
| INT-GAM-044 | Incentive events logged | AUTO | `GET /incentive-events?nag_id=...` | Events with rule_id, action_type, timestamp |

---

## 8. Push Notifications

### 8.1 Device Registration

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PUSH-001 | Register device token | AUTO | `POST /devices` with APNs token + platform | 201, device registered |
| INT-PUSH-002 | List registered devices | AUTO | `GET /devices` | List includes registered device |
| INT-PUSH-003 | Unregister device | AUTO | `DELETE /devices/{id}` | Device removed, no further pushes sent |
| INT-PUSH-004 | Duplicate token idempotent | AUTO | Register same token twice | No duplicate entries |
| INT-PUSH-005 | iOS registers on login | SEMI | Login on iOS with notifications enabled | Device token auto-registered with server |
| INT-PUSH-006 | iOS unregisters on logout | SEMI | Logout on iOS | Device token removed from server |

### 8.2 Push Delivery

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PUSH-010 | Nag creation triggers push | MANUAL | Create nag for recipient on device A, check device B | Push notification received on recipient's device |
| INT-PUSH-011 | Escalation triggers push | MANUAL | Let nag escalate to phase_1_due_soon | Push notification received |
| INT-PUSH-012 | Tap notification opens nag | MANUAL | Tap push notification banner | App opens to nag detail view |
| INT-PUSH-013 | Quiet hours suppresses push | AUTO | Set quiet hours, trigger escalation | Delivery deferred, no push during quiet period |
| INT-PUSH-014 | Failed delivery recorded | AUTO | Send push to invalid token | Delivery record shows `failed` status |
| INT-PUSH-015 | Muted user suppresses push | AUTO | Set `mute_until` in preferences, then trigger delivery | No push sent while muted |

---

## 9. AI Mediation

### 9.1 Excuse Summarization

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AI-001 | Summarize excuse (server) | AUTO | `POST /ai/summarize-excuse` with text + nag_id | 200, summary + category + confidence |
| INT-AI-002 | Category: forgot | AUTO | Send "I forgot about it" | Category = `forgot` |
| INT-AI-003 | Category: time_conflict | AUTO | Send "I was too busy with work" | Category = `time_conflict` |
| INT-AI-004 | Category: refused | AUTO | Send "I won't do it" | Category = `refused` |
| INT-AI-005 | Requires AI consent | AUTO | Call without `ai_mediation` consent | 403 AUTHZ_DENIED |
| INT-AI-006 | Requires ai_server_enabled pref | AUTO | Disable `ai_server_enabled` in preferences, then call | 403 AUTHZ_DENIED |

### 9.2 Tone Selection

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AI-010 | Tone: neutral (default) | AUTO | `POST /ai/select-tone` for nag with 1 miss | tone = `neutral` |
| INT-AI-011 | Tone: supportive (streak) | AUTO | Select tone for nag with 0 misses + streak >= 3 | tone = `supportive` |
| INT-AI-012 | Tone: firm (repeated misses) | AUTO | Select tone for nag with >= 3 misses in 7 days | tone = `firm` |

### 9.3 Coaching

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AI-020 | Generate coaching tip | AUTO | `POST /ai/coaching` with nag_id | 200, tip + category + scenario |
| INT-AI-021 | Web displays coaching | SEMI | View nag detail on web, see coaching section | Tip displayed |
| INT-AI-022 | iOS displays coaching (on-device) | SEMI | View nag detail on iOS | On-device AI provides tip (falls back to server) |

### 9.4 Pattern Detection

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AI-030 | Detect patterns | AUTO | `GET /ai/patterns?user_id=...&family_id=...` | insights array with behavioral patterns |
| INT-AI-031 | Day-of-week pattern | AUTO | Create miss events on same weekday 3+ times | Pattern detected for that weekday |

### 9.5 Digest

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AI-040 | Weekly digest | AUTO | `GET /ai/digest?family_id=...` | period_start, period_end, summary_text, member_summaries, totals |

### 9.6 Completion Prediction

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AI-050 | Predict completion | AUTO | `GET /ai/predict-completion?nag_id=...` | likelihood (0-1), suggested_reminder_time, factors |

### 9.7 Push-Back

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AI-060 | Evaluate push-back | AUTO | `POST /ai/push-back` for nag with repeated misses | should_push_back=true, message, tone, reason |
| INT-AI-061 | Push-back mode off | AUTO | Set `pushback_mode=off` in preferences, then evaluate | should_push_back=false |
| INT-AI-062 | Push-back max attempts | AUTO | Trigger max_pushback (default 2) push-backs in window | Next evaluation returns should_push_back=false |
| INT-AI-063 | Push-back respects quiet hours | AUTO | Trigger push-back during quiet hours | should_push_back=false |
| INT-AI-064 | Push-back cooldown | AUTO | Trigger push-back, then immediately evaluate again | should_push_back=false (cooldown active) |

### 9.8 On-Device AI (iOS)

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-AI-070 | On-device excuse summary | SEMI | Submit excuse on iOS, check local AI result | OnDeviceAIService provides category without server call |
| INT-AI-071 | Fallback to server on stale cache | SEMI | Let GRDB cache go stale (>24h), trigger AI | Transparently falls back to server AI endpoints |
| INT-AI-072 | AI disabled in preferences | SEMI | Disable `ai_on_device_enabled`, check UI | On-device AI features hidden/disabled |

---

## 10. Guardian Oversight

### 10.1 Policies

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GUARD-001 | List policies | AUTO | `GET /policies?family_id=...` as guardian | Paginated policy list |
| INT-GUARD-002 | Get single policy | AUTO | `GET /policies/{id}` as guardian | Full policy detail |
| INT-GUARD-003 | Update policy | AUTO | `PATCH /policies/{id}` as guardian | Policy updated |
| INT-GUARD-004 | Non-guardian cannot access policies | AUTO | `GET /policies?family_id=...` as participant | 403 AUTHZ_DENIED |
| INT-GUARD-005 | Co-owner dual approval | AUTO | Policy owned by 2 guardians: one proposes change, other must approve | Change pending until both approve |
| INT-GUARD-006 | Co-owner disagreement | AUTO | One guardian approves, other doesn't | Current policy remains, change blocked |
| INT-GUARD-007 | Co-owner leaves family | AUTO | Remove one co-owner guardian | Policy moves to single-owner; consequence-expanding edits blocked until new co-owner |

### 10.2 Policy Approvals

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GUARD-010 | Create approval | AUTO | `POST /policies/{id}/approvals` | Approval recorded |
| INT-GUARD-011 | List approvals | AUTO | `GET /policies/{id}/approvals` | Paginated approval list |

### 10.3 Reports

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GUARD-020 | Weekly report | AUTO | `GET /reports/family/weekly?family_id=...` | Completion rate, response times, missed count, category breakdown |
| INT-GUARD-021 | Metrics report (date range) | AUTO | `GET /reports/family/metrics?family_id=...&from=...&to=...` | Metrics for specified date range |
| INT-GUARD-022 | Report on web | SEMI | Navigate to Reports tab in nagz-web as guardian | Charts and metrics displayed |
| INT-GUARD-023 | Report on iOS | SEMI | Navigate to Reports on iOS as guardian | Report data rendered |

### 10.4 Consent Management

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-GUARD-030 | List consents | AUTO | `GET /consents?family_id=...` | All consents for family |
| INT-GUARD-031 | Grant child_account_creation | AUTO | `POST /consents` with type=child_account_creation | Consent granted, child account activates |
| INT-GUARD-032 | Grant ai_mediation | AUTO | `POST /consents` with type=ai_mediation | AI features unlocked |
| INT-GUARD-033 | Revoke ai_mediation | AUTO | `PATCH /consents/{id}` with revoked_at | AI endpoints return 403, queued AI messages cancelled |
| INT-GUARD-034 | Revoke gamification | AUTO | Revoke `gamification_participation` | New point/streak updates stop; historical data kept |
| INT-GUARD-035 | SMS opt-in (user-scoped) | AUTO | `POST /consents` with type=sms_opt_in, no family_id | SMS delivery enabled |
| INT-GUARD-036 | Revoke SMS opt-in | AUTO | Revoke sms_opt_in | Queued SMS deliveries cancelled |

---

## 11. Safety Controls

### 11.1 Block User

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SAFE-001 | Block user | AUTO | `POST /blocks` with target_id | 201, block active |
| INT-SAFE-002 | List blocks | AUTO | `GET /blocks` | Paginated list of blocks created by current user |
| INT-SAFE-003 | Update block (unblock) | AUTO | `PATCH /blocks/{id}` with state=inactive | Block deactivated |
| INT-SAFE-004 | Blocked user cannot nag blocker | AUTO | Blocked user tries `POST /nags` targeting blocker | 403 POLICY_FORBIDDEN |
| INT-SAFE-005 | Block on web | SEMI | Use block UI in nagz-web | Block applied, user hidden from interaction |
| INT-SAFE-006 | Block on iOS | SEMI | Use Safety section on iOS | Block applied |

### 11.2 Report Abuse

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SAFE-010 | Submit abuse report | AUTO | `POST /abuse-reports` with reason | 201, report in `open` status |
| INT-SAFE-011 | Get report status | AUTO | `GET /abuse-reports/{id}` | Report details with current status |
| INT-SAFE-012 | Rate limit on reports | AUTO | Submit 11 reports in 1 hour | 11th returns 429 RATE_LIMITED |
| INT-SAFE-013 | Report on web | SEMI | Use report UI in nagz-web | Report submitted, acknowledgment shown |
| INT-SAFE-014 | Report on iOS | SEMI | Use Safety > Report on iOS | Report submitted |

### 11.3 Relationship Suspension

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SAFE-020 | Suspend relationship | AUTO | `POST /relationships/{id}/suspend` as guardian | Relationship suspended |
| INT-SAFE-021 | Suspended relationship blocks nags | AUTO | Try creating nag on suspended relationship | 403 POLICY_FORBIDDEN |
| INT-SAFE-022 | Non-guardian cannot suspend | AUTO | `POST /relationships/{id}/suspend` as participant | 403 AUTHZ_DENIED |

### 11.4 Mute / Snooze

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SAFE-030 | Mute reminders | AUTO | `PATCH /preferences` with `mute_until` timestamp | No deliveries until mute expires |
| INT-SAFE-031 | Mute on iOS | SEMI | Toggle mute in settings | Notifications suppressed until mute_until |

---

## 12. Siri and Shortcuts

> All Siri tests require a physical iOS device. Simulator does not support Siri.

### 12.1 Intent Functionality

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SIRI-001 | List nags via Siri | MANUAL | "Show my nags in Nagz" | Siri reads active nag count and list |
| INT-SIRI-002 | Create nag via Siri | MANUAL | "Create a nag in Nagz" | Siri prompts for recipient, category, due time; nag created |
| INT-SIRI-003 | Complete nag via Siri | MANUAL | "Mark my nag as done in Nagz" | Siri asks which nag, marks completed |
| INT-SIRI-004 | Check overdue via Siri | MANUAL | "What's overdue in Nagz" | Siri reports overdue count and categories |
| INT-SIRI-005 | Snooze nag via Siri | MANUAL | "Snooze my nag in Nagz" | Siri asks which nag + duration, reschedules |
| INT-SIRI-006 | Family status via Siri | MANUAL | "How's my family doing in Nagz" | Siri reports weekly completion rate (guardian only) |

### 12.2 Auth and Role Guards

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SIRI-010 | Not logged in | MANUAL | Invoke Siri intent without logging in | "Please open Nagz and log in first" |
| INT-SIRI-011 | Child cannot create via Siri | MANUAL | Log in as child, try create nag via Siri | "Your role doesn't have permission" |
| INT-SIRI-012 | No family selected | MANUAL | Log in without joining family, invoke intent | "No family selected" |

### 12.3 Shortcuts App

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SIRI-020 | Shortcuts discovery | MANUAL | Open Shortcuts app, search "Nagz" | All 6 shortcuts appear |
| INT-SIRI-021 | Create automation | MANUAL | Build morning check automation (7 AM -> ListNagsIntent) | Automation triggers, nags read aloud |
| INT-SIRI-022 | Shortcut parameter editing | MANUAL | Edit CreateNag shortcut to set category and recipient | Parameters populate correctly |

---

## 13. Sync and Offline

### 13.1 Incremental Sync

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SYNC-001 | Sync endpoint returns events | AUTO | `GET /sync/events?family_id=...&since=0` | Returns nags, nag_events, ai_mediation_events, gamification_events |
| INT-SYNC-002 | Incremental sync (since timestamp) | AUTO | Sync at T1, create data, sync at T2 | Only new data since T1 returned |
| INT-SYNC-003 | Sync pagination | AUTO | Create >500 events, sync | Pagination handles large result sets |
| INT-SYNC-004 | iOS SyncService periodic poll | SEMI | Open iOS app, wait 5 minutes | SyncService triggers background sync |
| INT-SYNC-005 | iOS sync on foreground | SEMI | Background the app, create nag on web, foreground iOS | New nag appears after sync |

### 13.2 GRDB Local Cache

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SYNC-010 | Cache populated on sync | SEMI | Login on iOS, wait for first sync | CachedNag, CachedNagEvent etc. populated in GRDB |
| INT-SYNC-011 | Cache survives app restart | SEMI | Sync, kill app, relaunch | Cached data still available, nag list loads from cache |
| INT-SYNC-012 | Cache cleared on logout | SEMI | Logout on iOS, check DB file | cache.sqlite cleared or deleted |
| INT-SYNC-013 | Stale cache detection | SEMI | Prevent network access for >24h, check AI | Cache marked stale, AI falls back to server |

### 13.3 Offline Behavior

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-SYNC-020 | Offline nag list (iOS) | SEMI | Enable airplane mode, open nag list | Cached nags displayed |
| INT-SYNC-021 | Offline nag creation (iOS) | SEMI | Enable airplane mode, create nag | Queued locally, synced when online |
| INT-SYNC-022 | Offline nag completion (iOS) | SEMI | Enable airplane mode, complete nag | Queued locally, synced when online |
| INT-SYNC-023 | Conflict resolution | SEMI | Complete nag offline on iOS, update same nag on web, reconnect | Server canonical state wins, client reconciles |
| INT-SYNC-024 | Network error banner (web) | SEMI | Disconnect network during web app usage | Error banner with retry option appears |
| INT-SYNC-025 | Network error banner (iOS) | SEMI | Disconnect network during iOS usage | Error banner or retry prompt shown |

---

## 14. Version Compatibility

### 14.1 Version Endpoint

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-VER-001 | Version check (unauthenticated) | AUTO | `GET /version` | 200, returns server_version, api_version, min_client_version |
| INT-VER-002 | Compatible version | AUTO | Client API matches server API major | VersionChecker returns `.compatible` |
| INT-VER-003 | Update recommended | AUTO | Client major < server major | VersionChecker returns `.updateRecommended` |
| INT-VER-004 | Update required | AUTO | Client version < min_client_version | VersionChecker returns `.updateRequired` |
| INT-VER-005 | iOS version check on launch | SEMI | Launch iOS app | Version check runs automatically |
| INT-VER-006 | iOS blocks on required update | SEMI | Set server min_client_version > app version | App shows blocking update prompt |
| INT-VER-007 | Web version check | SEMI | Load nagz-web | Version endpoint called, banner if mismatch |

### 14.2 API Compatibility

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-VER-010 | Unknown fields ignored | AUTO | Send API request with extra fields | Server ignores unknown fields, processes known ones |
| INT-VER-011 | Missing optional fields | AUTO | Send request without optional fields | Server uses defaults |

---

## 15. Cross-Component Scenarios

These tests verify that actions on one component are correctly visible on others.

### 15.1 Web-to-iOS

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-CROSS-001 | Create nag on web, see on iOS | SEMI | Login on both web and iOS. Create nag on web. Trigger sync on iOS. | Nag appears in iOS nag list |
| INT-CROSS-002 | Complete nag on iOS, see on web | SEMI | Open nag on iOS, mark complete. Refresh web. | Nag shows completed on web |
| INT-CROSS-003 | Create nag on web, push on iOS | MANUAL | Create nag targeting iOS user. | Push notification received on iOS device |
| INT-CROSS-004 | Update preferences on web, reflected on iOS | SEMI | Change tone to "firm" on web. Sync on iOS. | iOS shows "firm" tone in settings |

### 15.2 iOS-to-Web

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-CROSS-010 | Create nag on iOS, see on web | SEMI | Create nag on iOS. Refresh web. | Nag appears in web nag list |
| INT-CROSS-011 | Submit excuse on iOS, creator sees on web | SEMI | Submit excuse on iOS. Creator views on web. | Excuse visible in nag detail on web |
| INT-CROSS-012 | Block user on iOS, enforced on web | SEMI | Block user on iOS. Blocked user tries to nag on web. | Web shows 403 / block enforced |

### 15.3 Multi-Device iOS

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-CROSS-020 | Login on two iOS devices | SEMI | Login same account on two devices | Both devices have valid sessions |
| INT-CROSS-021 | Nag created, pushed to both devices | MANUAL | Create nag targeting user with 2 devices | Push received on both devices |
| INT-CROSS-022 | Complete on device A, reflected on device B | SEMI | Complete nag on device A, sync device B | Nag shows completed on device B |

### 15.4 Siri-to-Server

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-CROSS-030 | Create nag via Siri, visible on web | MANUAL | "Create a nag in Nagz" via Siri. Check web. | Nag visible on web |
| INT-CROSS-031 | Complete via Siri, reflected everywhere | MANUAL | "Complete nag in Nagz" via Siri. Check web and other iOS devices. | Completed status visible across all clients |

### 15.5 API Direct + Client

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-CROSS-040 | Create nag via curl, see on web + iOS | AUTO/SEMI | `POST /nags` via curl. Refresh web, sync iOS. | Nag visible on both clients |
| INT-CROSS-041 | Update preference via API, clients reflect | AUTO/SEMI | `PATCH /preferences` via curl. Refresh web + iOS. | New preference value shown on both clients |

---

## 16. Performance and Load

### 16.1 Rate Limiting

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PERF-001 | Read rate limit (120/min) | AUTO | Send 121 GET requests in 60s | 121st returns 429, Retry-After header present |
| INT-PERF-002 | Write rate limit (60/min) | AUTO | Send 61 POST requests in 60s | 61st returns 429 |
| INT-PERF-003 | Nag write rate limit (30/min) | AUTO | Send 31 nag status updates in 60s | 31st returns 429 |
| INT-PERF-004 | Abuse report rate limit (10/hour) | AUTO | Submit 11 abuse reports in 1 hour | 11th returns 429 |
| INT-PERF-005 | Rate limit headers present | AUTO | Any authenticated request | `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` in response |

### 16.2 Pagination Stress

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PERF-010 | Large nag list | AUTO | Create 200 nags, paginate with limit=50 | 4 pages, total=200 |
| INT-PERF-011 | Max limit enforced | AUTO | Request with limit=300 | Server caps at 200 |

### 16.3 Concurrent Access

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PERF-020 | Concurrent nag completions | AUTO | Two users complete same nag simultaneously | Only one succeeds, other gets conflict |
| INT-PERF-021 | Concurrent family joins | AUTO | Two users join family at same time | Both succeed, no duplicate memberships |

### 16.4 Metrics Endpoint

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PERF-030 | Metrics available (no auth) | AUTO | `GET /metrics` | 200, all metric keys present (uptime, rps, memory, nags, deliveries, etc.) |
| INT-PERF-031 | Metrics reflect real data | AUTO | Create nags, then check metrics | `nags_open` count increments |
| INT-PERF-032 | Sparkline history populated | AUTO | Send requests, then check metrics | `rps` sparkline_history has recent values |

---

## 17. Data Privacy and Compliance

### 17.1 Account Lifecycle

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PRIV-001 | Data export | AUTO | `POST /accounts/export` | 200, returns all personal data (GDPR/CCPA) |
| INT-PRIV-002 | Account deletion | AUTO | `DELETE /accounts/{userId}` | Account soft-deleted, data purged within 30 days |
| INT-PRIV-003 | Deletion accessible in-app (iOS) | SEMI | Navigate to Account > Delete Account | Confirmation dialog, account deleted |
| INT-PRIV-004 | Deletion accessible in-app (web) | SEMI | Navigate to Account settings > Delete | Confirmation dialog, account deleted |
| INT-PRIV-005 | Guardian deletes child account | AUTO | `DELETE /accounts/{childId}` as guardian | Child account deleted |
| INT-PRIV-006 | Audit trail survives deletion | AUTO | Delete account, check audit_events | Audit records retained (36 month retention) |

### 17.2 Legal Documents

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PRIV-010 | Privacy policy (no auth) | AUTO | `GET /legal/privacy-policy` | 200, policy text returned |
| INT-PRIV-011 | Terms of service (no auth) | AUTO | `GET /legal/terms-of-service` | 200, ToS text returned |
| INT-PRIV-012 | Legal shown during signup (iOS) | SEMI | Open signup screen | Privacy policy and ToS links visible |
| INT-PRIV-013 | Legal shown during signup (web) | SEMI | Open signup page | Privacy policy and ToS links visible |
| INT-PRIV-014 | Legal accessible from settings | SEMI | Navigate to Settings on iOS and web | Privacy Policy and ToS accessible |

### 17.3 COPPA Compliance

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PRIV-020 | Under-13 requires guardian consent | AUTO | Create under-13 account, try to use features | All actions blocked until consent granted |
| INT-PRIV-021 | Guardian grants consent | AUTO | Grant `child_account_creation` consent | Child account becomes active |
| INT-PRIV-022 | Age gate on signup (iOS) | SEMI | Enter DOB making user < 13 | Guardian consent flow presented |
| INT-PRIV-023 | Age gate on signup (web) | SEMI | Enter DOB making user < 13 | Guardian consent flow presented |

### 17.4 Preferences and Data Isolation

| ID | Scenario | Type | Steps | Expected |
|----|----------|------|-------|----------|
| INT-PRIV-030 | User cannot read other user's prefs | AUTO | `GET /preferences/{otherUserId}` as non-guardian | 403 AUTHZ_DENIED |
| INT-PRIV-031 | Guardian can read child's prefs | AUTO | `GET /preferences/{childId}` as guardian | 200, child's preferences returned |
| INT-PRIV-032 | Family data isolation | AUTO | Request nags from family user is not a member of | 403 or empty result |

---

## 18. Regression Test Matrix

Summary of test counts by category and automation level:

| Category | AUTO | SEMI | MANUAL | Total |
|----------|------|------|--------|-------|
| 3. Auth Flow | 12 | 5 | 0 | 17 |
| 4. Family Management | 15 | 3 | 0 | 18 |
| 5. Nag Lifecycle | 28 | 2 | 0 | 30 |
| 6. Connections | 12 | 0 | 0 | 12 |
| 7. Gamification | 16 | 2 | 0 | 18 |
| 8. Push Notifications | 4 | 2 | 4 | 10 |
| 9. AI Mediation | 17 | 4 | 0 | 21 |
| 10. Guardian Oversight | 13 | 2 | 0 | 15 |
| 11. Safety Controls | 8 | 5 | 0 | 13 |
| 12. Siri & Shortcuts | 0 | 0 | 9 | 9 |
| 13. Sync & Offline | 3 | 10 | 0 | 13 |
| 14. Version Compatibility | 4 | 3 | 0 | 7 |
| 15. Cross-Component | 1 | 10 | 4 | 15 |
| 16. Performance & Load | 10 | 0 | 0 | 10 |
| 17. Data Privacy | 8 | 7 | 0 | 15 |
| **Total** | **151** | **55** | **17** | **223** |

### 18.1 Priority Tiers

**P0 -- Must pass before any release:**
- INT-AUTH-001, INT-AUTH-010, INT-AUTH-020 (auth works)
- INT-FAM-001, INT-FAM-010 (family creation and joining)
- INT-NAG-001, INT-NAG-030 (create and complete nag)
- INT-VER-001 (version endpoint)
- INT-PRIV-010, INT-PRIV-011 (legal docs)
- INT-SAFE-001, INT-SAFE-010 (block and report)

**P1 -- Must pass before App Store submission:**
- All INT-FAM-03x (role/permission matrix)
- All INT-PRIV-02x (COPPA compliance)
- INT-PUSH-010, INT-PUSH-012 (push delivery and tap)
- INT-SIRI-001 through INT-SIRI-006 (all Siri intents)
- INT-CROSS-001, INT-CROSS-002 (web-iOS bidirectional)

**P2 -- Must pass before scaling:**
- All INT-PERF-00x (rate limiting)
- INT-SYNC-001 through INT-SYNC-005 (sync correctness)
- INT-PERF-020, INT-PERF-021 (concurrency)
- All INT-AI-06x (push-back safety bounds)

**P3 -- Quality and polish:**
- All INT-GAM-0xx (gamification)
- All INT-CONN-0xx (connections)
- INT-SYNC-020 through INT-SYNC-025 (offline behavior)
- INT-CROSS-020 through INT-CROSS-031 (multi-device and Siri cross-component)

### 18.2 Existing Test Coverage Mapping

The existing unit test suites already cover many of these scenarios at the unit level. Integration tests fill the gaps between components:

| Existing Suite | Tests | Covers |
|----------------|-------|--------|
| `nagzerver/tests/test_auth.py` | pytest | Auth endpoints (INT-AUTH-00x API calls) |
| `nagzerver/tests/test_families.py` | pytest | Family CRUD (INT-FAM-0xx API calls) |
| `nagzerver/tests/test_nags.py` | pytest | Nag lifecycle (INT-NAG-0xx API calls) |
| `nagzerver/tests/test_connections.py` | pytest | Connection flow (INT-CONN-0xx API calls) |
| `nagzerver/tests/test_gamification.py` | pytest | Gamification scoring (INT-GAM-0xx API calls) |
| `nagzerver/tests/test_ai.py` | pytest | AI heuristics (INT-AI-0xx server calls) |
| `nagzerver/tests/test_safety.py` | pytest | Block/report (INT-SAFE-0xx API calls) |
| `nagzerver/tests/test_escalation.py` | pytest | Escalation phases (INT-NAG-04x) |
| `nagzerver/tests/test_sync.py` | pytest | Sync endpoint (INT-SYNC-001 through 003) |
| `nagzerver/tests/test_rate_limit.py` | pytest | Rate limits (INT-PERF-001 through 005) |
| `nagzerver/tests/test_version.py` | pytest | Version endpoint (INT-VER-001) |
| `nagz-web/src/__tests__/auth.test.tsx` | vitest | Web auth flow (INT-AUTH-002, 013, 031) |
| `nagz-web/src/__tests__/version.test.ts` | vitest | Web version check (INT-VER-007) |
| `nagz-ios` XCTest suite | xcodebuild | iOS services, sync, version checker, intents |

**Gaps to fill with dedicated integration tests:**
- Cross-component scenarios (section 15) -- currently untested
- Push notification delivery end-to-end (INT-PUSH-010 through 012)
- Offline behavior and conflict resolution (INT-SYNC-020 through 025)
- Siri intent end-to-end (INT-SIRI-001 through 022)
- Multi-device sync (INT-CROSS-020 through 022)

---

## Appendix A: Test Environment Reset

Between full integration test runs, reset test state:

```bash
# Reset database to clean state
cd ~/nagzerver && uv run alembic downgrade base && uv run alembic upgrade head

# Clear Redis
redis-cli -p 6379 FLUSHDB

# Re-seed test data
cd ~/nagzerver && uv run python -m nagz.cli.main seed-test-data
```

## Appendix B: Automatable Test Runner

For the AUTO tests, a dedicated integration test script can run against a live server:

```bash
# Run all automatable integration tests against local dev
cd ~/nagzerver
NAGZ_BASE_URL=http://localhost:9800/api/v1 uv run pytest tests/ -m integration

# Run against staging
NAGZ_BASE_URL=https://bd-nagzerver.fly.dev/api/v1 uv run pytest tests/ -m integration
```

## Appendix C: Cross-Spec References

| Spec Document | Relevant Test Sections |
|---------------|----------------------|
| `API_SURFACE.md` | Sections 3-6, 10, 14, 15 |
| `ARCHITECTURE.md` | Sections 5, 13, 14, 16 |
| `REQUIREMENTS.md` | Sections 4, 5 (roles, nag model) |
| `POLICY_MATRIX.md` | Section 4.4 (role permissions) |
| `GAMIFICATION.md` | Section 7 |
| `AI_BEHAVIOR.md` | Section 9 |
| `AI_ARCHITECTURE.md` | Sections 9, 13 |
| `SAFETY_AND_COMPLIANCE.md` | Sections 11, 17 |
| `INCENTIVES.md` | Section 7.5 |
| `PREFERENCES.md` | Sections 9.1, 10, 14 |
| `SIRI_SHORTCUTS.md` | Section 12 |
