# Nagz Safety and Compliance (V1)

## 1. Purpose
Define minimum safety, privacy, and compliance requirements needed for implementation and App Store readiness.

## 2. Anti-Abuse Controls (V1)
Required user controls:
- Block user
- Mute/snooze reminders
- Report abuse

Required server controls:
- Per-actor and per-relationship rate limits
- Quiet-hour enforcement
- Daily Nag cap enforcement
- Audit logging for abuse-relevant actions

## 3. Moderation Workflow (V1)
- Abuse reports create tracked moderation records.
- Each report has status: `open`, `investigating`, `resolved`, `dismissed`.
- Enforcement actions include warning, temporary restriction, and relationship suspension.
- Moderator actions are audit logged.

## 4. Child and Guardian Compliance
- Child accounts must be attached to a guardian-managed family.
- Child-to-guardian nagging is blocked by policy.
- Child account creation requires guardian-mediated consent flow.
- Child data collection must be minimized to what is operationally necessary.

## 5. SMS Compliance Baseline
- Explicit SMS opt-in required before first SMS delivery.
- Clear in-app disclosure of SMS purpose and frequency characteristics.
- STOP/HELP semantics supported at provider integration layer.
- Unsubscribed recipients must not receive further SMS until re-opt-in.

## 6. Data Handling and Retention
Data classes:
- Operational events
- Notification metadata
- User-generated content (nag text, optional notes)
- Audit/moderation records

Baseline handling:
- Encrypt in transit and at rest.
- Keep sensitive content out of logs.
- Define retention windows per data class.
- Support user account deletion requests with policy-aware deletion/anonymization.

## 7. App Store Readiness Checklist
Before submission, verify:
- Block/report/mute flows are user-visible and functional.
- Child safeguards are implemented and test-covered.
- Account deletion flow is implemented in-app.
- Privacy disclosures match actual data usage.
- SMS opt-in and unsubscribe behavior are implemented and tested.

## 8. Open Compliance Decisions to Confirm
These need explicit product/legal confirmation before production launch:
1. Exact age thresholds and region handling for child accounts.
2. Retention durations per data class.
3. Moderation response-time targets.
4. Jurisdictions supported for SMS at launch.
