# Nagz Parent and Guardian User Manual (V1.0)

## 1. Who This Is For
This manual is for guardians who create, monitor, and adjust nags for family members.

## 2. Before You Start
- Launch region for V1.0 is United States.
- SMS reminders are US-only in V1.0.
- Child accounts must be linked to a guardian-managed family.

## 3. Core Terms
- Nag: A reminder task sent from one member (`creator`) to another (`recipient`).
- Strategy template: How reminders escalate over time. In V1.0 this is `friendly_reminder` with time-based and behavior-based phases.
- Done definition: How completion is recorded (`ack_only`, `binary_check`, `binary_with_note`).
- Incentive: Reward or consequence tied to outcome.
- AI mediation: AI-assisted status/excuse handling and summaries.

## 4. First-Time Setup
1. Create guardian account.
2. Create or join family.
3. Add second guardian (optional co-owner).
4. Add child members.
5. Configure notification preferences (push/SMS).
6. Set quiet hours and daily limits.

## 5. Creating a Nag
1. Open `New Nag`.
2. Choose recipient.
3. Enter task name and optional details.
4. Set due date/time.
5. Select `friendly_reminder` strategy template.
6. Select done definition type.
7. Assign optional category (`chores`, `meds`, `homework`, `appointments`, `other`).
8. Optionally attach reward/consequence policy.
9. Review and send.

## 6. Escalation Phases
`friendly_reminder` uses these phases:
- `phase_0_initial`
- `phase_1_due_soon`
- `phase_2_overdue_soft`
- `phase_3_overdue_bounded_pushback`
- `phase_4_guardian_review`

## 7. Monitoring Progress
- Use dashboard to view:
  - Due soon
  - Overdue
  - Completed today
  - Active miss streaks
- Open a nag to see timeline:
  - Created
  - Delivered
  - AI-mediated updates/excuses
  - Completion or missed outcome

## 8. AI Mediation for Guardians
- Recipients can submit status or excuses through AI.
- AI summarizes updates and shows trends.
- Guardians can select tone profile (`neutral`, `supportive`, `firm`).
- AI push-back is bounded by policy and audit logged.

## 9. Incentives and Consequences
- Configure catalog entries in guardian settings.
- Map rules to triggers (for example `miss_streak_3`).
- High-impact consequences require guardian approval.
- Every incentive/consequence application is logged with reason.

## 10. Safety Controls
- Block member interactions when needed.
- Mute or snooze reminder streams.
- Report abuse from any screen.
- Use relationship suspension when repeated misuse occurs.

## 11. Snooze Behavior
- Snooze delays the next reminder delivery and does not complete a nag.
- Snooze is limited by family policy (`max_snooze_minutes`).
- Snooze does not reset miss history or escalation state.

## 12. Reports and Insights
Guardian reports include:
- Completion rate
- Median response time
- Missed count
- Category performance
- Excuse frequency categories
- Strategy effectiveness
- Incentive/consequence effectiveness
- Weekly trend summaries

## 13. Notifications and SMS
- Push notifications are primary channel.
- SMS requires explicit opt-in.
- Users can stop SMS and re-enable later.
- Quiet hours and daily caps are enforced server-side.

## 14. Privacy, Retention, and Deletion
- Data is encrypted in transit and at rest.
- Retention windows are defined in `SAFETY_AND_COMPLIANCE.md`.
- Deleted accounts are purged/anonymized within 30 days, except required audit/legal records.

## 15. Common Guardian Workflows
### A. Co-own a policy with another guardian
1. Open policy settings.
2. Add guardian co-owner.
3. Confirm dual-approval behavior for co-owned policy changes.
4. If co-owners disagree on a change, current policy remains active.

### B. Respond to repeated misses
1. Open recipient trend view.
2. Check miss streak and excuse history.
3. Confirm escalation history.
4. Apply or approve policy consequence if appropriate.
5. Adjust due time, strategy template, or incentive plan.

### C. Handle a disputed consequence
1. Open audit timeline.
2. Review trigger and approvals.
3. Reverse or amend consequence if needed.
4. Add a note for accountability.

## 16. Troubleshooting
- Not receiving push: verify app notification permission and quiet-hour window.
- Not receiving SMS: verify opt-in status and US phone format.
- Task not escalating: check daily cap and throttles.
- Cannot view reports: only guardians have report visibility.

## 17. Best Practices
- Keep tasks specific and measurable.
- Use supportive tone first, then firm when needed.
- Keep consequence catalog limited and predictable.
- Review weekly reports and tune policies gradually.

## 18. Related Specs
- `CATALOG.md`
- `GLOSSARY.md`
- `API_SURFACE.md`
- `REQUIREMENTS.md`
- `ARCHITECTURE.md`
- `PREFERENCES.md`
- `POLICY_MATRIX.md`
- `SAFETY_AND_COMPLIANCE.md`
- `AI_BEHAVIOR.md`
- `INCENTIVES.md`
- `GAMIFICATION.md`
- `SPEC_BASELINE_CHANGELOG.md`
