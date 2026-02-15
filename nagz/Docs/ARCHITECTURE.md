# Nagz Hybrid Architecture (V1.0)

## 1. Decision Summary
Nagz uses a hybrid architecture:
- Central backend is source of truth for authorization, policy, AI mediation, escalation, notifications, incentives, and reporting.
- Local-first clients provide responsive UX and offline operation.
- Optional peer-assisted sync may be added later, with server reconciliation.

## 2. Why Central Authority Is Required
Pure P2P is insufficient for V1.0 because:
- Push and SMS require provider-backed server integrations.
- Guardian-only reporting and consequence controls need centralized authorization.
- AI mediation and auditability need durable shared history.
- Abuse controls are safest when enforced centrally.

## 3. Core Components
1. Client apps (iOS/Android/Web)
- Local encrypted cache for nags/events/preferences
- Offline mutation queue
- Non-authoritative local reminder scheduler

2. API Gateway
- AuthN/AuthZ
- Validation and rate limiting
- Versioned API surface
- Common error envelope and canonical error codes

3. Policy Service
- Role and relationship enforcement
- Co-owner guardian policy rules
- Hard-stop policy enforcement (quiet hours, caps, throttles)
- Dual-approval evaluation for co-owned policy changes

4. AI Mediation Service
- Excuse intake and normalization
- Recipient-to-creator summaries
- Bounded push-back prompts and tone controls
- Fail-closed reject behavior for unauthorized requests

5. Escalation Engine
- Time-based and behavior-based trigger evaluation
- `friendly_reminder` phase execution (normative phase definitions in `REQUIREMENTS.md` section 6)

6. Incentives Engine
- Reward/consequence rule evaluation
- Guardian-policy checks before consequence application

7. Notification Service
- Push and SMS dispatch
- Retry/backoff and provider error handling

8. Reporting Service
- Completion, response-time, excuse, and effectiveness metrics
- Weekly family and gamification summaries

9. Safety Service
- Abuse reporting intake and tracking
- Blocking/muting state and enforcement

10. Event Store + OLTP DB
- Immutable event log for audit and analytics
- Canonical operational state

## 4. Authoritative Ownership
Server-authoritative:
- User roles and relationships
- Policy and hard-stop constraints
- AI mediation actions and summaries
- Escalation state and delivery outcomes
- Incentive/consequence applications
- Reporting and audit records

Client-local:
- Draft nags not yet submitted
- Session-only UI overrides
- Offline actions pending sync

## 5. Security and Trust Boundaries
- Clients are untrusted for policy enforcement.
- Server validates every mutation against policy.
- TLS in transit and encryption at rest are required.
- Notification payloads and logs should avoid sensitive details.
- Optional future E2EE may encrypt message bodies while keeping routing metadata server-readable.

## 6. Sync and Conflict Model
1. Client enqueues mutation locally.
2. Client sends mutation with idempotency key.
3. Server validates and writes canonical event.
4. Server pushes updates to related clients.
5. Clients reconcile optimistic state with canonical state.

Conflict rules:
- Use optimistic concurrency (`etag`/version).
- Server resolves canonical order.
- Client retries on conflict with latest snapshot.

## 7. Task and AI Flow
1. Nag created with due time, strategy template, and typed `done_definition`.
2. Recipient updates status or submits excuse through AI mediation.
3. AI summarizes and records attributable mediation events.
4. Escalation checkpoints evaluate completion status, behavior signals, and hard-stop policy inside `friendly_reminder` phases defined in `REQUIREMENTS.md` section 6.
5. Notification intents emit to push/SMS channels.
6. Incentives engine applies reward/consequence events under policy.
7. Completion cancels pending escalation intents.

## 8. Core Data Model
- `users` (id, created_at, status)
- `families` (id, name, created_by, created_at, status)
- `family_memberships` (user_id, family_id, role, status, joined_at, left_at)
- `relationships` (party_a, party_b, status)
- `nag_policies` (id, family_id, owners, strategy_template, constraints, status)
- `nags` (id, family_id, policy_id, strategy_template, creator_id, recipient_id, due_at, category, done_definition, status)
- `nag_events` (id, nag_id, event_type, actor_id, at, payload)
- `ai_mediation_events` (id, nag_id, prompt_type, tone, summary, at)
- `deliveries` (id, nag_event_id, channel, status, provider_ref)
- `incentive_rules` (id, family_id, version, condition, action, approval_mode, status)
- `incentive_events` (id, nag_id, rule_id, action_type, approved_by, at)
- `gamification_events` (id, family_id, user_id, event_type, delta_points, streak_delta, at)
- `gamification_snapshots` (family_id, period_start, leaderboard_json)
- `abuse_reports` (id, reporter_id, target_id, reason, status)
- `blocks` (id, actor_id, target_id, state)
- `consents` (id, user_id, family_id_nullable, consent_type, granted_at, revoked_at)
- `reports_snapshots` (family_id, period_start, metrics_json)

Consent type values are authoritative in `SAFETY_AND_COMPLIANCE.md`.
Consent scope:
- user-level consents (for example `sms_opt_in`) use `family_id_nullable = null`.
- family-scoped consents (for example `child_account_creation`, `ai_mediation`, `gamification_participation`) use a concrete `family_id`.

nag `status` semantics:
- status is a materialized read-model value derived from canonical `nag_events`.
- allowed status values in V1.0: `open`, `completed`, `missed`, `cancelled_relationship_change`.

## 9. API Error Contract
All API errors use a shared envelope with:
- `code`
- `message`
- `request_id`
- optional `details`

Canonical V1 codes and HTTP mappings (authoritative list):
- `AUTHZ_DENIED` -> `403`
- `VALIDATION_ERROR` -> `422`
- `POLICY_FORBIDDEN` -> `403`
- `POLICY_INVALID_VALUE` -> `422`
- `PRECONDITION_FAILED` -> `412`
- `RATE_LIMITED` -> `429`
- `NOT_FOUND` -> `404`

## 10. API Surface Coverage
Minimum endpoint coverage is defined in `API_SURFACE.md`.

## 11. Reliability and Safety Controls
- Idempotent write APIs
- Retry with exponential backoff for provider failures
- Dead-letter handling for repeated send failures
- Policy-based throttles per actor, relationship, and channel
- Auditability for all user-visible state transitions
- Default API throttle profile:
  - authenticated read requests: 120 requests/minute per user
  - authenticated write requests: 60 requests/minute per user
  - nag status/excuse writes: 30 requests/minute per user
  - abuse report submissions: 10 requests/hour per user

## 12. Deployment Pattern
- Start as centralized multi-tenant deployment.
- Use managed queue/scheduler for escalation work.
- Keep service boundaries clean for scale-out.

## 13. V1.0 vs Later
Implement in V1.0:
- Central backend authority
- Local-first client sync queue
- Push + SMS channels (US-only for SMS in V1.0)
- `friendly_reminder` strategy template with phases defined in `REQUIREMENTS.md` section 6
- AI mediation with bounded behavior controls
- Minimum hard-stop and safety controls

Defer:
- Full P2P state synchronization
- Additional delivery channels
- Additional strategy templates beyond `friendly_reminder`
- Full end-to-end encrypted payload model
