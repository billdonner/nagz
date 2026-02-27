# Nagz Hybrid Architecture (V1.0)

> Updated 2026-02-27 to match actual implementation across all repos.

## 1. Decision Summary
Nagz uses a hybrid architecture:
- Central backend is source of truth for authorization, policy, AI mediation, escalation, notifications, incentives, and reporting.
- Local-first clients provide responsive UX and offline operation.
- Real-time WebSocket push supplements periodic sync for low-latency updates.

## 2. Why Central Authority Is Required
Pure P2P is insufficient for V1.0 because:
- Push and SMS require provider-backed server integrations.
- Guardian-only reporting and consequence controls need centralized authorization.
- AI mediation and auditability need durable shared history.
- Abuse controls are safest when enforced centrally.

## 3. Core Components

### 3.1 Client Apps (iOS, Web)

| Client | Stack | Capabilities |
|--------|-------|-------------|
| iOS | SwiftUI, Swift 6, GRDB | Local encrypted cache, offline AI heuristics, Siri Shortcuts, push notifications |
| Web | React, TypeScript, Vite | Full feature parity, WebSocket real-time, generated API client (Orval) |

Common client capabilities:
- Local encrypted cache for nags/events/preferences (iOS: GRDB SQLite, Web: session state)
- WebSocket subscription for real-time family events
- Token-based auth with automatic refresh on 401

### 3.2 API Gateway
- AuthN (JWT bearer tokens, child PIN auth) / AuthZ (role + relationship checks)
- Validation and rate limiting (Redis-backed, per-endpoint tiers)
- Versioned API surface (`/api/v1/`) with `GET /version` compatibility check
- Common error envelope with 8 canonical error codes

### 3.3 Auth Service
- Email + password signup/login (bcrypt hashed)
- Child login via family code + username + PIN (rate-limited: 5 attempts / 15 min)
- JWT access tokens (short-lived) + refresh tokens (long-lived)
- Token refresh endpoint, logout (token invalidation)

### 3.4 Policy Service
- Role and relationship enforcement
- Co-owner guardian policy rules with approval workflow
- Hard-stop policy enforcement (quiet hours, caps, throttles)
- Dual-approval evaluation for co-owned policy changes

### 3.5 AI Mediation Service
- Excuse intake, summarization, and categorization
- Tone selection (neutral / supportive / firm) based on behavior history
- Coaching tips keyed by scenario (first miss, streak, time conflicts, homework)
- Completion prediction (category + overall weighted rates)
- Bounded push-back prompts with cooldown and quiet-hour enforcement
- Weekly family digest generation
- Pattern detection (day-of-week miss analysis, 90-day window)
- Fail-closed reject behavior for unauthorized requests (requires `ai_mediation` consent)

### 3.6 Escalation Engine
- Time-based and behavior-based trigger evaluation
- 5-phase `friendly_reminder` escalation: `phase_0_initial` → `phase_1_due_soon` → `phase_2_overdue_soft` → `phase_3_overdue_bounded_pushback` → `phase_4_guardian_review`
- Recompute endpoint for manual guardian intervention

### 3.7 Incentives Engine
- Reward/consequence rule evaluation (condition/action JSON)
- Guardian-policy checks before consequence application
- Approval modes: `auto` or `guardian_confirmed`

### 3.8 Gamification Engine
- Points calculation on nag completion (on-time bonus, late penalty)
- Streak tracking with streak delta events
- 6 badge types: `first_completion`, `streak_3`, `streak_7`, `streak_30`, `perfect_week`, `century_club`
- Family leaderboard snapshots (periodic)

### 3.9 Notification Service
- APNs push delivery (Bearer token auth, JSON payload)
- Smart push: skips APNs when recipient has active WebSocket connection
- Delivery tracking (pending → sent → delivered → failed)
- Provider reference storage (APNs message IDs)

### 3.10 WebSocket Hub
- Endpoint: `WS /api/v1/ws?token={jwt}&family_id={uuid}`
- JWT authentication + family membership verification
- Redis Pub/Sub subscription to `nagz:family:{family_id}` channel
- Bidirectional: client can send `ping`, server responds `pong`
- Events broadcast as JSON: `{ event, family_id, actor_id, data, ts }`
- Auto-reconnect on client side (exponential backoff 1s → 30s)

### 3.11 Event Bus (Redis Pub/Sub)
- Central `publish_event()` called by nag routers, family routers, gamification
- Events: `nag_created`, `nag_updated`, `nag_status_changed`, `excuse_submitted`, `member_added`, `member_removed`, `gamification_event`
- Smart push integration: checks active WebSocket connections before sending APNs

### 3.12 Connections Service
- One-way invite: inviter → invitee by email
- Status lifecycle: `pending` → `active` / `declined` / `revoked`
- Trusted flag: allows cross-family nagging of children
- `GET /connections/{id}/children` lists children of trusted connection's families
- Connection revocation cascels trusted-child nags and resets trust flag

### 3.13 Reporting Service
- Weekly family completion metrics (completion %, streaks, point deltas)
- Per-member breakdowns with historical snapshots
- Metrics JSON stored in `reports_snapshots` table

### 3.14 Safety Service
- Abuse reporting intake and status tracking (open → investigating → resolved / dismissed)
- User-to-user blocking (active / lifted states)
- Relationship suspension and revocation

### 3.15 Metrics Service
- Unauthenticated `GET /metrics` endpoint for system health monitoring
- Uptime, database counts (users, families, nags), delivery stats, sync request count

### 3.16 Scheduler Service
- Background job runner: escalation recomputation, delivery retry, report generation
- Scheduler-driven recurrent nag creation
- Redis-based job queuing and locking

### 3.17 Event Store + OLTP DB
- PostgreSQL (Fly.io managed) with immutable event log for audit and analytics
- Canonical operational state
- 24 tables (see Section 8)

## 4. Authoritative Ownership

**Server-authoritative:**
- User roles and relationships
- Policy and hard-stop constraints
- AI mediation actions and summaries
- Escalation state and delivery outcomes
- Incentive/consequence applications
- Reporting and audit records
- Connection trust status
- Child account settings

**Client-local:**
- Draft nags not yet submitted
- Session-only UI overrides
- Offline actions pending sync
- AI heuristic results (when GRDB cache is fresh)

## 5. Security and Trust Boundaries
- Clients are untrusted for policy enforcement.
- Server validates every mutation against policy.
- TLS in transit and encryption at rest are required.
- Notification payloads and logs should avoid sensitive details.
- Child login rate-limited (5 attempts / 15 min per family_code + username).
- On-device AI excuse processing stays local — server receives only structured category + summary.
- Keychain storage for tokens on iOS (encrypted at rest by iOS).
- GRDB cache in Application Support with iOS Data Protection.

## 6. Sync and Conflict Model

### 6.1 Real-time (WebSocket)
1. Client connects: `WS /ws?token={jwt}&family_id={uuid}`
2. Server subscribes to Redis Pub/Sub channel `nagz:family:{family_id}`
3. On nag/family/gamification events, server pushes JSON to all connected clients
4. Client invalidates cache and refreshes affected views

### 6.2 Periodic (Polling)
1. iOS `SyncService` polls `GET /sync/events?family_id={id}&since={last_sync_at}` every 300 seconds
2. Server returns new nags, nag_events, ai_mediation_events, gamification_events (max 500 per entity)
3. Client upserts into GRDB and updates `last_sync_at` in `sync_metadata` table
4. If `last_sync_at` > 24 hours, on-device AI falls back to server endpoints

### 6.3 Conflict Resolution
- Use optimistic concurrency (`etag`/version) for preferences
- Server resolves canonical order for events
- Client retries on conflict with latest snapshot

## 7. Task and AI Flow
1. Nag created with due time, strategy template, and typed `done_definition`.
2. Nag may target a family member (`family_id`) or a trusted connection's child (`connection_id`) — XOR constraint.
3. Recipient updates status or submits excuse through AI mediation.
4. AI summarizes excuse, selects tone, provides coaching, and records attributable mediation events.
5. Escalation checkpoints evaluate completion status, behavior signals, and hard-stop policy inside `friendly_reminder` phases.
6. Notification intents emit to push channel (APNs), skipping if WebSocket active.
7. Incentives engine applies reward/consequence events under policy.
8. Gamification engine awards points, updates streaks, checks badge eligibility.
9. Completion cancels pending escalation intents and creates `nag_completed` event.

## 8. Core Data Model (24 Tables)

### Identity & Access
- `users` (id, email, password_hash, display_name, date_of_birth, created_at, status)
- `families` (id, name, created_by, invite_code, child_code, created_at, status)
- `family_memberships` (user_id, family_id, role, status, username, pin_hash, joined_at, left_at)
- `connections` (id, inviter_id, invitee_id, invitee_email, status, trusted, created_at, responded_at)
- `relationships` (id, party_a, party_b, status)
- `consents` (id, user_id, family_id_nullable, consent_type, granted_at, revoked_at)
- `child_settings` (child_user_id, family_id, can_snooze, max_snoozes_per_day, can_submit_excuses, quiet_hours_start, quiet_hours_end)

### Nag Domain
- `nags` (id, family_id, connection_id, policy_id, strategy_template, creator_id, recipient_id, due_at, category, done_definition, description, recurrence, parent_nag_id, last_notified_phase, status, created_at, completed_at)
- `nag_events` (id, nag_id, event_type, actor_id, at, payload)
- `nag_policies` (id, family_id, owners, strategy_template, constraints, status)
- `policy_approvals` (id, policy_id, approver_id, approved_at, comment)

### AI & Delivery
- `ai_mediation_events` (id, nag_id, prompt_type, tone, summary, at)
- `deliveries` (id, nag_event_id, channel, status, provider_ref)
- `device_tokens` (id, user_id, platform, token, created_at, last_used_at)

### Gamification & Incentives
- `incentive_rules` (id, family_id, version, condition, action, approval_mode, status)
- `incentive_events` (id, nag_id, rule_id, action_type, approved_by, at)
- `gamification_events` (id, family_id, user_id, event_type, delta_points, streak_delta, at)
- `gamification_snapshots` (family_id, period_start, leaderboard_json)
- `badges` (id, user_id, family_id, badge_type, earned_at)

### Safety & Audit
- `abuse_reports` (id, reporter_id, target_id, reason, status)
- `blocks` (id, actor_id, target_id, state)
- `audit_events` (id, event_type, actor_id, resource_type, resource_id, payload, request_id, at)

### Preferences & Reports
- `user_preferences` (user_id, family_id, prefs_json, schema_version, etag, updated_at)
- `reports_snapshots` (family_id, period_start, metrics_json)

### Enums

| Enum | Values |
|------|--------|
| FamilyRole | guardian, participant, child |
| NagCategory | chores, meds, homework, appointments, other |
| DoneDefinition | ack_only, binary_check, binary_with_note |
| NagStatus | open, completed, missed, cancelled_relationship_change |
| EscalationPhase | phase_0_initial, phase_1_due_soon, phase_2_overdue_soft, phase_3_overdue_bounded_pushback, phase_4_guardian_review |
| ConnectionStatus | pending, active, declined, revoked |
| ConsentType | child_account_creation, sms_opt_in, ai_mediation, gamification_participation |
| BadgeType | first_completion, streak_3, streak_7, streak_30, perfect_week, century_club |
| Recurrence | every_5_minutes, every_15_minutes, every_30_minutes, hourly, daily, weekly, monthly |
| AITone | neutral, supportive, firm |
| ExcuseCategory | forgot, time_conflict, unclear_instructions, lacking_resources, refused, other |

**Consent scope:**
- User-level consents (e.g. `sms_opt_in`) use `family_id_nullable = null`.
- Family-scoped consents (e.g. `child_account_creation`, `ai_mediation`, `gamification_participation`) use a concrete `family_id`.

**Nag `status` semantics:**
- Status is a materialized read-model value derived from canonical `nag_events`.
- Allowed status values: `open`, `completed`, `missed`, `cancelled_relationship_change`.

**Nag ownership constraint:**
- Each nag belongs to either `family_id` (family-internal) or `connection_id` (trusted person) — XOR, never both.

## 9. API Error Contract
All API errors use a shared envelope:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "request_id": "uuid",
    "details": { }
  }
}
```

Canonical V1 codes and HTTP mappings:

| Code | HTTP | Use |
|------|------|-----|
| `AUTHN_FAILED` | 401 | Invalid credentials, expired token |
| `AUTHZ_DENIED` | 403 | Insufficient role or relationship |
| `POLICY_FORBIDDEN` | 403 | Action blocked by family policy |
| `VALIDATION_ERROR` | 422 | Invalid request body/params |
| `POLICY_INVALID_VALUE` | 422 | Policy constraint violation |
| `PRECONDITION_FAILED` | 412 | ETag mismatch, stale state |
| `RATE_LIMITED` | 429 | Too many requests |
| `NOT_FOUND` | 404 | Resource does not exist |

## 10. API Surface (88 Endpoints)

| Group | Count | Key Endpoints |
|-------|-------|---------------|
| Auth | 5 | signup, login, child-login, refresh, logout |
| Accounts | 3 | create, GDPR export, delete |
| Families | 12 | CRUD, join, members, child credentials, child settings |
| Nags | 7 | CRUD, status, excuses, list (paginated) |
| Connections | 8 | invite, accept/decline/revoke, trust toggle, list trusted children |
| Escalation | 2 | get phase, recompute |
| AI | 7 | summarize-excuse, select-tone, coaching, patterns, digest, predict-completion, push-back |
| Gamification | 4 | summary, leaderboard, events, badges |
| Incentives | 4 | rules CRUD, events |
| Policies | 4 | CRUD, approvals |
| Preferences | 4 | own get/set, per-user get/set |
| Consents | 3 | list, grant, update |
| Devices | 3 | register, list, delete |
| Deliveries | 1 | list (paginated) |
| Safety | 5 | abuse reports, blocks, relationship suspension |
| Reports | 2 | weekly, metrics |
| Sync | 1 | incremental event sync for iOS |
| Legal | 2 | privacy policy, terms of service (no auth) |
| Version | 1 | server/API/min-client version (no auth) |
| Metrics | 1 | system health (no auth) |
| WebSocket | 1 | real-time family events |

Full endpoint details in `API_SURFACE.md`.

## 11. Reliability and Safety Controls
- Idempotent write APIs
- Retry with exponential backoff for provider failures (APNs, Redis)
- Dead-letter handling for repeated send failures
- Policy-based throttles per actor, relationship, and channel
- Auditability for all user-visible state transitions
- Default API throttle profile (Redis-backed, differentiated by endpoint tier):
  - Authenticated read requests (GET): 120 requests/minute per user
  - Authenticated write requests (POST/PATCH/DELETE): 60 requests/minute per user
  - Nag status/excuse/escalation-recompute writes: 30 requests/minute per user
  - Abuse report submissions: 10 requests/hour per user
  - Child login: 5 attempts/15 minutes per family_code + username
- Rate limit response headers on every response: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
- Rate limited responses include `Retry-After` header
- Password max length 128 (bcrypt DoS prevention)

## 12. Deployment

| Component | Host | Details |
|-----------|------|---------|
| nagzerver (API + web) | Fly.io (`bd-nagzerver.fly.dev`) | 1 machine, 512MB RAM, uvicorn + 1 worker |
| PostgreSQL | Fly.io managed | Single instance, encrypted at rest |
| Redis | Fly.io managed | Pub/sub for WebSocket events + rate limiting |
| nagz-web | Bundled with nagzerver | Vite-built SPA served by FastAPI static files |
| nagz-ios | App Store Connect | TestFlight (internal beta group: "family-naggers") |

## 13. Test Coverage

| Repo | Tests | Command |
|------|-------|---------|
| nagzerver | 234+ | `uv run pytest` |
| nagz-web | 126 | `npx vitest run` |
| nagz-ios | 202 | `xcodebuild test` |
| nagz-ai | 42 | `swift test` |
| **Total** | **604+** | |

## 14. V1.0 Implementation Status

**Implemented:**
- Central backend authority (FastAPI + PostgreSQL + Redis)
- Local-first iOS client with GRDB cache and periodic sync
- Real-time WebSocket events (replaces polling for connected clients)
- Push notifications (APNs) with smart push (skip if WebSocket active)
- `friendly_reminder` strategy template with 5-phase escalation
- AI mediation: 7 operations, on-device heuristics + server fallback + direct router fallback
- AI UI surfaces: tone/coaching/prediction in NagDetailView, family digest + patterns
- Gamification: points, streaks, 6 badge types, leaderboard
- Child accounts: family code + username + PIN auth, per-child settings
- Trusted connections: cross-family nagging of children
- Safety: abuse reports, blocking, relationship suspension
- Siri Shortcuts: 6 intents, 14 phrases
- Version compatibility checking (client ↔ server)
- System metrics endpoint
- GDPR data export + account deletion

**Deferred:**
- SMS delivery channel (infrastructure ready, provider not configured)
- Full P2P state synchronization
- Additional strategy templates beyond `friendly_reminder`
- Full end-to-end encrypted payload model
- Android client
- Apple Foundation Models integration (awaiting iOS 26)
