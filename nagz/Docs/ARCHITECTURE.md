# Nagz Hybrid Architecture (V1)

## 1. Decision Summary
Nagz should use a **hybrid architecture**:
- **Central backend** as source of truth for policy, authorization, escalation scheduling, notifications, and reporting.
- **Local-first clients** for responsiveness and offline operation.
- **Optional peer-to-peer assist** for low-latency state fanout between guardians, with server reconciliation.

This approach preserves reliability and guardrails while keeping a path open for stronger privacy and partial decentralization later.

## 2. Why Not Pure P2P
Pure peer-to-peer is not sufficient for V1 requirements:
- Push and SMS delivery require server/provider integrations.
- Guardian-only reporting/history requires trusted centralized authorization.
- Behavior-based escalation needs durable cross-device event history.
- Abuse controls (rate limits, quiet-hours policy enforcement, auditability) are safer with centralized enforcement.

## 3. High-Level Components
1. Client apps (iOS/Android/Web)
- Local encrypted store for nags, events, and preferences cache.
- Local scheduler for non-authoritative reminders.
- Offline queue for mutations.

2. API Gateway
- AuthN/AuthZ, request validation, rate limiting.
- Versioned REST/GraphQL endpoints.

3. Nag Policy Service
- Creates and validates nag definitions.
- Enforces role constraints (e.g., child cannot nag guardian).
- Manages co-owned guardian policies.

4. Escalation Engine
- Evaluates time-based and behavior-based triggers.
- Applies strategy templates (V1: friendly reminder).
- Produces escalation intents.

5. Notification Service
- Sends push and SMS via providers.
- Channel fallback and retry policies.

6. Reporting Service
- Computes guardian-visible metrics and trends.
- Materialized weekly summaries.

7. Event Store + OLTP DB
- Append-only event log for audit and behavior analytics.
- Relational/document store for current state.

8. Optional P2P Relay Layer (Future-Friendly)
- WebRTC/secure relay for direct guardian-to-guardian state hints.
- Server remains authoritative on conflict and policy.

## 4. Authoritative Data Ownership
Server-authoritative:
- Family relationships and roles
- Nag policy definitions
- Escalation schedules/outcomes
- Delivery attempts and statuses
- Guardian-visible reports and audit log

Client-authoritative (temporary/local):
- Draft nags not yet submitted
- Session overrides and UI settings
- Offline completion actions pending sync

## 5. Trust Boundaries and Security
- Clients are untrusted for policy enforcement.
- Server validates every mutation against relationship and role rules.
- Require TLS in transit and encryption at rest.
- Keep immutable audit events for creation, edit, escalation, notification, and completion.
- Optional E2EE path:
  - Encrypt message bodies client-side.
  - Keep metadata (routing, timestamps, policy ids) server-readable for operations.

## 6. Sync Model
1. Client writes mutation to local queue.
2. Client sends mutation with idempotency key.
3. Server validates, applies, emits canonical event.
4. Server pushes state updates to other devices.
5. Client reconciles local optimistic state with canonical state.

Conflict strategy:
- Use optimistic concurrency (`etag`/version).
- Server resolves final order by event timestamp + logical version.
- Client retries on conflict with latest snapshot.

## 7. Escalation Execution Flow
1. Nag created with due time, strategy, and done criteria.
2. Escalation Engine schedules checkpoints.
3. On checkpoint, engine evaluates:
- completion status
- recent behavior signals (miss streak, latency profile)
- hard-stop constraints (quiet hours, daily cap)
4. Engine emits delivery intent to Notification Service.
5. Delivery outcomes logged as events.
6. Completion cancels pending escalation intents.

## 8. Data Model (Core)
- `users` (id, role)
- `family_memberships` (user_id, family_id, role)
- `relationships` (party_a, party_b, type)
- `nag_policies` (id, owners, strategy, constraints)
- `nags` (id, creator_id, recipient_id, due_at, done_definition)
- `nag_events` (id, nag_id, event_type, actor_id, at, payload)
- `deliveries` (id, nag_event_id, channel, status, provider_ref)
- `reports_snapshots` (family_id, period_start, metrics_json)

## 9. Reliability and Safety Controls
- Idempotent write APIs for mobile retries.
- Retry with backoff for provider failures.
- Dead-letter queue for repeated notification failures.
- Policy-based max nags per day and quiet hours.
- Abuse throttles per actor, relationship, and channel.

## 10. Deployment Pattern
- Start as centralized multi-tenant cloud deployment.
- Use managed queue + scheduler for escalation jobs.
- Keep services modular to allow future split by workload.
- Add region-aware data partitioning as usage grows.

## 11. V1 Recommendation
Implement now:
- Central backend for policy, escalation, notifications, reporting
- Local-first clients with offline queue
- Push + SMS channels
- Friendly reminder strategy template

Defer:
- Full P2P synchronization
- Additional channels beyond push/SMS
- Rich strategy marketplace
- Full end-to-end encrypted payload model
