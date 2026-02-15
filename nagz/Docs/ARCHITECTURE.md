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
- Recipient-to-assigner summaries
- Bounded push-back prompts and tone controls

5. Escalation Engine
- Time-based and behavior-based trigger evaluation
- Strategy execution (`friendly_reminder` in V1.0) where escalation is parameterized behavior, not a separate strategy family

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
1. Nag created with due time, strategy, and typed `done_definition`.
2. Recipient updates status or submits excuse through AI mediation.
3. AI summarizes and records attributable mediation events.
4. Escalation checkpoints evaluate completion status, behavior signals, and hard-stop policy inside `friendly_reminder` phases.
5. Notification intents emit to push/SMS channels.
6. Incentives engine applies reward/consequence events under policy.
7. Completion cancels pending escalation intents.

## 8. Core Data Model
- `users` (id, role)
- `family_memberships` (user_id, family_id, role)
- `relationships` (party_a, party_b, status)
- `nag_policies` (id, owners, strategy, constraints)
- `nags` (id, creator_id, recipient_id, due_at, done_definition)
- `nag_events` (id, nag_id, event_type, actor_id, at, payload)
- `ai_mediation_events` (id, nag_id, prompt_type, tone, summary, at)
- `deliveries` (id, nag_event_id, channel, status, provider_ref)
- `incentive_events` (id, nag_id, rule_id, action_type, approved_by, at)
- `abuse_reports` (id, reporter_id, target_id, reason, status)
- `blocks` (id, actor_id, target_id, state)
- `consents` (id, user_id, consent_type, granted_at, revoked_at)
- `reports_snapshots` (family_id, period_start, metrics_json)

Consent types in V1.0:
- `child_account_creation`
- `sms_opt_in`
- `ai_mediation`
- `gamification_participation`

## 9. API Error Contract
All API errors use a shared envelope with:
- `code`
- `message`
- `request_id`
- optional `details`

Canonical V1 codes:
- `AUTHZ_DENIED`
- `VALIDATION_ERROR`
- `POLICY_VIOLATION`
- `PRECONDITION_FAILED`
- `RATE_LIMITED`
- `NOT_FOUND`

## 10. Reliability and Safety Controls
- Idempotent write APIs
- Retry with exponential backoff for provider failures
- Dead-letter handling for repeated send failures
- Policy-based throttles per actor, relationship, and channel
- Auditability for all user-visible state transitions

## 11. Deployment Pattern
- Start as centralized multi-tenant deployment.
- Use managed queue/scheduler for escalation work.
- Keep service boundaries clean for scale-out.

## 12. V1.0 vs Later
Implement in V1.0:
- Central backend authority
- Local-first client sync queue
- Push + SMS channels (US-only for SMS in V1.0)
- Friendly reminder strategy
- AI mediation with bounded behavior controls
- Minimum hard-stop and safety controls

Defer:
- Full P2P state synchronization
- Additional delivery channels
- Rich strategy marketplace
- Full end-to-end encrypted payload model
