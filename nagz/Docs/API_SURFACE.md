# Nagz API Surface (V1.0)

## 1. Scope
This document defines the V1.0 API surface.

## 2. Conventions
- Base path: `/api/v1`
- Auth: bearer token (JWT or `dev:<user-uuid>`) for all endpoints except version and auth.
- Idempotency required for retryable write endpoints.
- Error envelope and canonical codes: `ARCHITECTURE.md` section 9 (authoritative).
- All list endpoints return paginated responses: `{ "items": [...], "total": N, "limit": N, "offset": N }`.
- Default `limit` is 50, maximum `limit` is 200.
- Pagination query params: `?limit={limit}&offset={offset}`.

## 3. Authorization and Consent Annotations
- `guardian_only`: endpoint callable only by guardian role in target family.
- `member`: any active family member in target family.
- `participant`: family member with participant role (can create nags for children only).
- `recipient_only`: current nag recipient only.
- `creator_or_guardian`: nag creator or guardian in family.
- `self_or_guardian`: the user themselves or a guardian in a shared family.
- `requires_consent(x)`: action requires active consent type `x`.

## 4. Version
- `GET /version` (unauthenticated) — returns server version, API version, and minimum client version.

## 5. Authentication
- `POST /auth/signup` (unauthenticated) — create account with email, password, display_name.
- `POST /auth/login` (unauthenticated) — returns access and refresh tokens.
- `POST /auth/refresh` (unauthenticated) — exchange refresh token for new access token.
- `POST /auth/logout` (authenticated) — revoke refresh token.

## 6. Preferences
- `GET /preferences?family_id={familyId}` (`member`)
- `PATCH /preferences?family_id={familyId}` (`member`)
- `GET /preferences/{userId}` (`self_or_guardian`)
- `PATCH /preferences/{userId}` (`self_or_guardian`)

## 7. Family and Membership
- `POST /families` (`guardian_only` account scope)
  - Request: `{ "name": "Smith Family" }`
  - Response includes `family_id` and `invite_code`.
- `GET /families/{familyId}` (`member`)
- `POST /families/join` (`member` invite flow)
  - Request: `{ "invite_code": "ABCD1234" }`
- `POST /families/{familyId}/join` (`member` invite flow, legacy)
  - Request: `{ "invite_code": "ABCD1234" }`
- `POST /families/{familyId}/members` (`guardian_only`)
  - Request: `{ "user_id": "usr_2", "role": "child" }`
  - Requires `requires_consent(child_account_creation)` when adding a child.
- `POST /families/{familyId}/members/create` (`guardian_only`)
  - Creates a new user account and adds them as a family member.
- `GET /families/{familyId}/members` (`member`, paginated)
- `DELETE /families/{familyId}/members/{userId}` (`guardian_only`)

## 8. Nag Management
- `POST /nags` (`creator_or_guardian`)
- `GET /nags/{nagId}` (`creator_or_guardian` or recipient)
- `PATCH /nags/{nagId}` (`creator_or_guardian`)
- `POST /nags/{nagId}/status` (`recipient_only`)
  - Purpose: update nag completion/progress state.
  - Request: `{ "status": "completed", "note": "done" }`
- `POST /nags/{nagId}/excuses` (`recipient_only`, `requires_consent(ai_mediation)`)
  - Purpose: send excuse/blocked-context text to AI mediation intake.
  - Request: `{ "text": "missed bus", "category": "time_conflict" }`
  - Does not directly set nag terminal status.
- `GET /nags/{nagId}/excuses` (`creator_or_guardian`, paginated)
- `GET /nags?family_id={familyId}&state={state}` (`member`, paginated)
  - `state` filters materialized nag status (`open`, `completed`, `missed`, `cancelled_relationship_change`).

## 9. Escalation and Delivery
- `GET /nags/{nagId}/escalation` (`creator_or_guardian`)
- `POST /nags/{nagId}/escalation/recompute` (`creator_or_guardian`)
- `GET /deliveries?nag_id={nagId}` (`creator_or_guardian`, paginated)

## 10. Policy and Co-Owner Governance
- `GET /policies?family_id={familyId}` (`guardian_only`, paginated)
- `GET /policies/{policyId}` (`guardian_only`)
- `PATCH /policies/{policyId}` (`guardian_only`)
- `POST /policies/{policyId}/approvals` (`guardian_only` co-owner)
- `GET /policies/{policyId}/approvals` (`guardian_only`, paginated)

## 11. Incentives and Rules
- `GET /incentive-rules?family_id={familyId}` (`guardian_only`, paginated)
- `POST /incentive-rules` (`guardian_only`)
- `PATCH /incentive-rules/{ruleId}` (`guardian_only`)
- `GET /incentive-events?nag_id={nagId}` (`guardian_only`, paginated)

## 12. Gamification
- `GET /gamification/summary?family_id={familyId}` (`member`, `requires_consent(gamification_participation)`)
- `GET /gamification/leaderboard?family_id={familyId}` (`member`, `requires_consent(gamification_participation)`)
- `GET /gamification/events?user_id={userId}` (`member`, self-or-guardian, paginated)

## 13. Safety and Moderation
- `POST /abuse-reports` (`member`)
- `GET /abuse-reports/{reportId}` (`reporter_or_guardian`)
- `GET /blocks` (`member`, paginated) — list blocks created by current user.
- `POST /blocks` (`member`)
- `PATCH /blocks/{blockId}` (`member` owner)
- `POST /relationships/{relationshipId}/suspend` (`guardian_only`)

## 14. Consent and Account Lifecycle
- `GET /consents?family_id={familyId}` (`member`, paginated)
- `POST /consents` (`member` or guardian depending on consent type)
- `PATCH /consents/{consentId}` (`member` or guardian depending on consent type)
- `POST /accounts` (authenticated registration flow)
- `POST /accounts/export` (authenticated) — export all personal data for the current user (GDPR/CCPA).
- `DELETE /accounts/{userId}` (`self_or_guardian` with policy rules)

## 15. Device Token Management
- `POST /devices` (authenticated) — register device token for push notifications.
- `GET /devices` (authenticated, paginated) — list registered device tokens.
- `DELETE /devices/{deviceId}` (authenticated) — unregister device token.

## 16. Reporting
- `GET /reports/family/weekly?family_id={familyId}` (`guardian_only`)
- `GET /reports/family/metrics?family_id={familyId}&from={date}&to={date}` (`guardian_only`)

## 17. Legal Documents
- `GET /legal/privacy-policy` (unauthenticated) — returns the current privacy policy.
- `GET /legal/terms-of-service` (unauthenticated) — returns the current terms of service.

## 18. AI (Server-Side Heuristics)
All AI endpoints require `requires_consent(ai_mediation)` and the `ai_server_enabled` user preference. V1 uses deterministic heuristics; a real model can be swapped in later without changing the API contract.

- `POST /ai/summarize-excuse` (`member`) — summarize and categorize excuse text.
  - Request: `{ "text": "...", "nag_id": "..." }`
  - Response: `{ "nag_id", "summary", "category", "confidence" }`
- `POST /ai/select-tone` (`member`) — select escalation tone based on recent miss history.
  - Request: `{ "nag_id": "..." }`
  - Response: `{ "nag_id", "tone", "miss_count_7d", "streak", "reason" }`
- `POST /ai/coaching` (`member`) — generate coaching tip for a scenario.
  - Request: `{ "nag_id": "..." }`
  - Response: `{ "nag_id", "tip", "category", "scenario" }`
- `GET /ai/patterns?user_id={userId}&family_id={familyId}` (`member`) — detect behavioral patterns (e.g. missed days).
  - Response: `{ "user_id", "family_id", "insights[]", "analyzed_at" }`
- `GET /ai/digest?family_id={familyId}` (`member`) — weekly family digest with completion rates.
  - Response: `{ "family_id", "period_start", "period_end", "summary_text", "member_summaries[]", "totals" }`
- `GET /ai/predict-completion?nag_id={nagId}` (`member`) — predict completion likelihood.
  - Response: `{ "nag_id", "likelihood", "suggested_reminder_time", "factors" }`
- `POST /ai/push-back` (`member`) — evaluate whether to push back on an excuse.
  - Request: `{ "nag_id": "..." }`
  - Response: `{ "nag_id", "should_push_back", "message", "tone", "reason" }`

## 19. Sync
- `GET /sync/events?family_id={familyId}&since={timestamp}` (`member`, paginated) — incremental sync of events since a given timestamp.
