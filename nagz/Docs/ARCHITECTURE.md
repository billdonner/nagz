# Nagz Hybrid Architecture (V0.5)

## 1. Decision Summary
Nagz uses a hybrid architecture:
- Central backend as source of truth for policy, authorization, AI mediation, escalation, notifications, incentives, and reporting.
- Local-first clients for responsiveness and offline operation.
- Optional future peer-to-peer assist for low-latency state fanout between guardians, with server reconciliation.

## 2. Why Central Authority Is Required
Pure peer-to-peer is insufficient for V0.5 requirements:
- Push/SMS delivery requires server/provider integrations.
- Guardian-only reporting and consequence controls need centralized authorization.
- AI mediation and auditability require durable shared history.
- Abuse controls and throttles are safer with centralized enforcement.

## 3. High-Level Components
1. Client apps (iOS/Android/Web)
- Local encrypted store for tasks, events, and preference cache.
- Local scheduler for non-authoritative reminders.
- Offline mutation queue.

2. API Gateway
- AuthN/AuthZ, request validation, rate limiting.
- Versioned REST API.

3. Nag Policy Service
- Task definitions, done criteria, role enforcement.
- Relationship and ownership validation.

4. AI Mediation Service
- Excuse intake and normalization.
- Recipient-to-assigner summaries.
- Bounded push-back according to policy.
- Tone and coaching behavior from preferences.

5. Escalation Engine
- Time and behavior trigger evaluation.
- Strategy application (`friendly_reminder` in V0.5).

6. Incentives Engine
- Reward/consequence rule evaluation.
- Application events with guardian-approval checks.

7. Notification Service
- Push and SMS provider integrations.
- Delivery retries and fallback.

8. Reporting Service
- Completion, response-time, excuse, and effectiveness metrics.
- Weekly family summaries and gamification rollups.

9. Event Store + OLTP DB
- Append-only event log for auditability and model input.
- Operational store for current state.

## 4. Authoritative Data Ownership
Server-authoritative:
- Users, roles, relationships
- Task policies and assignments
- AI mediation actions and summaries
- Escalation outcomes and deliveries
- Incentive applications
- Family-level reports and audit logs

Client-temporary:
- Draft tasks not yet submitted
- Session overrides
- Offline updates pending sync

## 5. Safety Boundaries
- Clients are untrusted for policy enforcement.
- Server validates all mutations against role and safety constraints.
- Consequence application requires policy checks and optional guardian confirmation.
- AI responses are constrained by behavior policy and logging.

## 6. Sync Model
1. Client writes mutation to local queue.
2. Client sends mutation with idempotency key.
3. Server validates and commits canonical event.
4. Server broadcasts updated state to related devices.
5. Client reconciles optimistic local state with canonical state.

Conflict strategy:
- Optimistic concurrency (`etag`/version).
- Server is final arbiter of ordering and policy validity.

## 7. AI and Incentive Flow
1. Task assigned with due time and optional reward/consequence policy.
2. Recipient sends progress or excuse to AI mediator.
3. AI mediator classifies and summarizes status.
4. Escalation engine evaluates missed conditions.
5. Incentives engine applies earned rewards or policy-approved consequences.
6. Reporting service updates performance and effectiveness metrics.

## 8. Reliability Controls
- Idempotent write APIs.
- Retry with backoff for provider failures.
- Dead-letter queue for persistent delivery failures.
- Quiet-hour and daily-contact caps.
- Per-user/relationship abuse throttles.

## 9. V0.5 Delivery Recommendation
Implement now:
- Central backend policy + AI mediation + escalation + incentives + reporting
- Local-first clients with offline queue
- Push and SMS channels
- Friendly reminder strategy template

Defer:
- Full P2P synchronization
- Additional delivery channels beyond push/SMS
- Unbounded autonomous AI decisioning
- Fully automated consequences without policy controls
