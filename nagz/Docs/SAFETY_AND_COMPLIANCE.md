# Nagz Safety and Compliance (V1.0)

## 1. Purpose
Define minimum safety, privacy, and compliance requirements needed for implementation and App Store readiness.

## 2. Launch Compliance Scope
- Launch geography: United States only.
- SMS launch jurisdictions: United States only.
- Child-account handling is implemented for United States launch requirements only in V1.0.

## 3. Anti-Abuse Controls (V1.0)
Required user controls:
- Block user
- Mute/snooze reminders
- Report abuse

Required server controls:
- Per-actor and per-relationship rate limits
- Quiet-hour enforcement
- Daily Nag cap enforcement
- Audit logging for abuse-relevant actions

## 4. Moderation Workflow and SLA (V1.0)
- Abuse reports create tracked moderation records.
- Each report has status: `open`, `investigating`, `resolved`, `dismissed`.
- Enforcement actions include warning, temporary restriction, and relationship suspension.
- Moderator actions are audit logged.

SLA targets:
- Critical safety reports: first review within 4 hours.
- Standard abuse reports: first review within 24 hours.
- Low-severity complaints: first review within 72 hours.
- User receives immediate in-app acknowledgment on report submission.

## 5. Child and Guardian Compliance
- Child accounts must be attached to a guardian-managed family.
- Child-to-guardian nagging is blocked by policy.
- Child account creation requires guardian-mediated consent flow.
- Child data collection must be minimized to what is operationally necessary.
- V1.0 age thresholds and region logic are limited to United States launch requirements.

V1.0 consent types and behavior:
- `child_account_creation`
- `sms_opt_in`
- `ai_mediation`
- `gamification_participation`
- Consent records store `granted_at` and optional `revoked_at`.
- Revoked consent immediately blocks new actions that require that consent.

## 6. SMS Compliance Baseline
- Explicit SMS opt-in required before first SMS delivery.
- Clear in-app disclosure of SMS purpose and frequency characteristics.
- STOP/HELP semantics supported at provider integration layer.
- Unsubscribed recipients must not receive further SMS until re-opt-in.

## 7. Data Handling and Retention
Data classes and retention:
- Operational events: retain 24 months.
- Notification metadata: retain 12 months.
- User-generated content (nag text, notes, excuses): retain until account deletion request or 24 months of inactivity.
- Audit/moderation records: retain 36 months.
- Gamification events (points/streak/badge changes): operational events retention (24 months).
- Gamification leaderboard/snapshot aggregates: notification/report snapshot retention (12 months).

Deleted account handling:
- Purge or anonymize deleted account data within 30 days, except required audit/legal records.

Baseline handling:
- Encrypt in transit and at rest.
- Keep sensitive content out of logs.
- Support user account deletion requests with policy-aware deletion/anonymization.

## 8. App Store Readiness Checklist
Before submission, verify:
- Block/report/mute flows are user-visible and functional.
- Child safeguards are implemented and test-covered.
- Account deletion flow is implemented in-app.
- Privacy disclosures match actual data usage.
- SMS opt-in and unsubscribe behavior are implemented and tested.
