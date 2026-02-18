# Nagz Safety and Compliance (V1.0)

## 1. Purpose
Define minimum safety, privacy, and compliance requirements needed for implementation and App Store readiness.

## 2. Launch Compliance Scope
- Launch geography: United States only.
- SMS launch jurisdictions: United States only.
- Child-account handling is implemented for United States launch requirements only in V1.0.
- COPPA (Children's Online Privacy Protection Act) is in scope for US child-account handling.

## 3. Anti-Abuse Controls (V1.0)
Required user controls:
- Block user
- Mute/snooze reminders
- Report abuse

Required server controls:
- Per-actor and per-relationship rate limits
- Quiet-hour enforcement
- Daily nag cap enforcement
- Audit logging for abuse-relevant actions

Default API rate-limit profile:
- authenticated read requests: 120 requests/minute per user.
- authenticated write requests: 60 requests/minute per user.
- nag status/excuse/escalation-recompute writes: 30 requests/minute per user.
- abuse report submissions: 10 requests/hour per user.

Hard-stop defaults:
- `daily_nag_cap`: default 8 per creator per family per local day (configurable guardian policy range `1..8`).

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
- US age-gate behavior in V1.0:
  - collect date of birth at account creation.
  - treat users under 13 as child accounts requiring guardian-mediated consent.
  - block child account activation until required consent is recorded.

V1.0 consent types (authoritative list):
- `child_account_creation`
- `sms_opt_in`
- `ai_mediation`
- `gamification_participation`

Consent behavior:
- Consent records store `granted_at` and optional `revoked_at`.
- `sms_opt_in` is user-scoped consent; other V1 consent types are family-scoped.
- Revoked consent immediately blocks new actions that require that consent.
- In-flight behavior on revocation:
  - queued AI messages are canceled on `ai_mediation` revocation.
  - queued SMS deliveries are canceled on `sms_opt_in` revocation.
  - gamification snapshots remain historical; new point/streak updates stop on `gamification_participation` revocation.

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
- `gamification_events`: retain 24 months.
- `gamification_snapshots`: retain 12 months.
- `reports_snapshots`: retain 12 months.

Deleted account handling:
- Purge or anonymize deleted account data within 30 days, except required audit/legal records.

Baseline handling:
- Encrypt in transit and at rest.
- Keep sensitive content out of logs.
- Support user account deletion requests with policy-aware deletion/anonymization.

## 8. Breach Response
- Maintain incident severity classification and on-call escalation.
- Contain and assess scope before restoring affected flows.
- Notify impacted users and guardians per applicable US state breach-notification laws and contractual obligations.
- COPPA/FTC implications for child data incidents require legal review and documented decision records.
- Preserve forensic audit evidence and document remediation actions.

## 9. Third-Party Processors
- Notification delivery uses third-party processors (for example APNs/FCM and SMS providers).
- Processor usage must follow data-minimization principles and contract controls.
- Required controls:
  - documented data-processing terms
  - sub-processor inventory and review
  - least-data payload design for push/SMS providers

## 10. Medical/Medication Scope Disclaimer
- `meds` category is a reminder taxonomy label only.
- V1.0 does not provide diagnosis, dosing advice, clinical decision support, or medical-device functionality.

## 11. App Store Readiness Checklist

### Server-Side Requirements (all implemented)

| Requirement | Status | Endpoint/Feature |
|---|---|---|
| Block user | Done | `POST /blocks`, `GET /blocks`, `PATCH /blocks/{id}` |
| Report abuse | Done | `POST /abuse-reports`, `GET /abuse-reports/{id}` |
| Mute/snooze | Done | `PATCH /preferences` (interaction_controls.mute_until) |
| Account deletion | Done | `DELETE /accounts/{userId}` (soft-delete) |
| Data export | Done | `POST /accounts/export` (GDPR/CCPA) |
| Privacy policy | Done | `GET /legal/privacy-policy` (no auth) |
| Terms of service | Done | `GET /legal/terms-of-service` (no auth) |
| Age gate (COPPA) | Done | DOB at signup, under-13 → pending_guardian_consent |
| Guardian consent | Done | `POST /consents` (child_account_creation type) |
| SMS opt-in | Done | `POST /consents` (sms_opt_in type), revocable |
| Consent management | Done | 4 consent types: child_account, sms, ai_mediation, gamification |
| Audit trail | Done | Immutable audit_events table, all mutations logged |
| Rate limiting | Done | 4-tier differentiated limits |
| Quiet hours | Done | Via preferences, enforced by scheduler |
| Relationship suspension | Done | `POST /relationships/{id}/suspend` |

### Client-Side Requirements (verify before submission)
- Block/report/mute flows are user-visible and functional.
- Account deletion is accessible in-app settings.
- Privacy policy and TOS are displayed during signup and accessible from settings.
- Age gate collects date of birth before account creation.
- Guardian consent UI is presented for child accounts.
- SMS opt-in disclosure is shown before enabling SMS.
- Push notification permission is requested with clear purpose string.

### App Store Review Notes
- The "meds" category is a reminder label only; no medical advice is provided.
- AI mediation is bounded by guardian policy and does not make autonomous decisions.
- All safety controls are enforced server-side; client UI hides disallowed actions as convenience only.
- COPPA compliance: child accounts require verifiable guardian consent before activation.
