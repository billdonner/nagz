# Nagz API Surface (V1.0)

## 1. Scope
This document defines the minimum V1.0 API surface beyond preferences.

## 2. Conventions
- Base path: `/api/v1`
- Auth: bearer token for all endpoints.
- Idempotency required for retryable write endpoints.
- Error envelope and canonical codes: `ARCHITECTURE.md` section 9 (authoritative).

## 3. Authorization and Consent Annotations
- `guardian_only`: endpoint callable only by guardian role in target family.
- `member`: any active family member in target family.
- `recipient_only`: current nag recipient only.
- `creator_or_guardian`: nag creator or guardian in family.
- `requires_consent(x)`: action requires active consent type `x`.

## 4. Preferences
- `GET /preferences?family_id={familyId}` (`member`)
- `PATCH /preferences?family_id={familyId}` (`member`)

## 5. Family and Membership
- `POST /families` (`guardian_only` account scope)
  - Request: `{ "name": "Smith Family" }`
  - Response: `{ "family_id": "fam_123" }`
- `POST /families/{familyId}/join` (`member` invite flow)
  - Request: `{ "invite_code": "ABCD1234" }`
- `POST /families/{familyId}/members` (`guardian_only`)
  - Request: `{ "user_id": "usr_2", "role": "child" }`
  - Requires `requires_consent(child_account_creation)` when adding a child.
- `DELETE /families/{familyId}/members/{userId}` (`guardian_only`)

## 6. Nag Management
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
- `GET /nags?family_id={familyId}&state={state}` (`member`)
  - `state` filters materialized nag status (`open`, `completed`, `missed`, `cancelled_relationship_change`).

## 7. Escalation and Delivery
- `GET /nags/{nagId}/escalation` (`creator_or_guardian`)
- `POST /nags/{nagId}/escalation/recompute` (`creator_or_guardian`)
- `GET /deliveries?nag_id={nagId}` (`creator_or_guardian`)

## 8. Policy and Co-Owner Governance
- `GET /policies/{policyId}` (`guardian_only`)
- `PATCH /policies/{policyId}` (`guardian_only`)
- `POST /policies/{policyId}/approvals` (`guardian_only` co-owner)
- `GET /policies/{policyId}/approvals` (`guardian_only`)

## 9. Incentives and Rules
- `GET /incentive-rules?family_id={familyId}` (`guardian_only`)
- `POST /incentive-rules` (`guardian_only`)
- `PATCH /incentive-rules/{ruleId}` (`guardian_only`)
- `GET /incentive-events?nag_id={nagId}` (`guardian_only`)

## 10. Gamification
- `GET /gamification/summary?family_id={familyId}` (`member`, `requires_consent(gamification_participation)`)
- `GET /gamification/leaderboard?family_id={familyId}` (`member`, `requires_consent(gamification_participation)`)
- `GET /gamification/events?user_id={userId}` (`member`, self-or-guardian)

## 11. Safety and Moderation
- `POST /abuse-reports` (`member`)
- `GET /abuse-reports/{reportId}` (`reporter_or_guardian`)
- `POST /blocks` (`member`)
- `PATCH /blocks/{blockId}` (`member` owner)
- `POST /relationships/{relationshipId}/suspend` (`guardian_only`)

## 12. Consent and Account Lifecycle
- `GET /consents?family_id={familyId}` (`member`)
- `POST /consents` (`member` or guardian depending on consent type)
- `PATCH /consents/{consentId}` (`member` or guardian depending on consent type)
- `POST /accounts` (authenticated registration flow)
- `DELETE /accounts/{userId}` (`self_or_guardian` with policy rules)

## 13. Reporting
- `GET /reports/family/weekly?family_id={familyId}` (`guardian_only`)
- `GET /reports/family/metrics?family_id={familyId}&from={date}&to={date}` (`guardian_only`)
