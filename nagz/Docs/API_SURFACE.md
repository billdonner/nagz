# Nagz API Surface (V1.0)

## 1. Scope
This document defines the minimum V1.0 API surface beyond preferences.

## 2. Conventions
- Base path: `/api/v1`
- Auth: bearer token for all endpoints.
- Idempotency required for write endpoints that can be retried.
- Error envelope and codes: see `PREFERENCES.md` and `ARCHITECTURE.md`.

## 3. Preferences
- `GET /preferences?family_id={familyId}`
- `PATCH /preferences?family_id={familyId}`

## 4. Nag Management
- `POST /nags`
- `GET /nags/{nagId}`
- `PATCH /nags/{nagId}`
- `POST /nags/{nagId}/status`
- `POST /nags/{nagId}/excuses`
- `GET /nags?family_id={familyId}&state={state}`

## 5. Escalation and Delivery
- `GET /nags/{nagId}/escalation`
- `POST /nags/{nagId}/escalation/recompute`
- `GET /deliveries?nag_id={nagId}`

## 6. Policy and Co-Owner Governance
- `GET /policies/{policyId}`
- `PATCH /policies/{policyId}`
- `POST /policies/{policyId}/approvals`
- `GET /policies/{policyId}/approvals`

## 7. Incentives and Rules
- `GET /incentive-rules?family_id={familyId}`
- `POST /incentive-rules`
- `PATCH /incentive-rules/{ruleId}`
- `GET /incentive-events?nag_id={nagId}`

## 8. Gamification
- `GET /gamification/summary?family_id={familyId}`
- `GET /gamification/leaderboard?family_id={familyId}`
- `GET /gamification/events?user_id={userId}`

## 9. Safety and Moderation
- `POST /abuse-reports`
- `GET /abuse-reports/{reportId}`
- `POST /blocks`
- `PATCH /blocks/{blockId}`
- `POST /relationships/{relationshipId}/suspend`

## 10. Consent and Account Lifecycle
- `GET /consents?family_id={familyId}`
- `POST /consents`
- `PATCH /consents/{consentId}`
- `POST /accounts`
- `DELETE /accounts/{userId}`
- `POST /families/{familyId}/members`
- `DELETE /families/{familyId}/members/{userId}`

## 11. Reporting
- `GET /reports/family/weekly?family_id={familyId}`
- `GET /reports/family/metrics?family_id={familyId}&from={date}&to={date}`
