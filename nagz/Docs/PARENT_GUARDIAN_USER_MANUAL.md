# Nagz Parent and Guardian User Manual (V0.5)

## 1. Who This Is For
This manual is for guardians who create, monitor, and adjust Nagz reminders for family members.

## 2. Before You Start
- Launch region for V0.5 is United States.
- SMS reminders are US-only in V0.5.
- Child accounts must be linked to a guardian-managed family.

## 3. Core Terms
- Nag: A reminder task sent from one member to another.
- Strategy: How reminders escalate over time.
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
5. Select `friendly_reminder` strategy.
6. Select done definition type.
7. Assign optional category (chores, meds, homework, appointments).
8. Optionally attach reward/consequence policy.
9. Review and send.

## 6. Monitoring Progress
- Use dashboard to view:
  - Due soon
  - Overdue
  - Completed today
  - Active miss streaks
- Open a Nag to see timeline:
  - Created
  - Delivered
  - AI-mediated updates/excuses
  - Completion or missed outcome

## 7. AI Mediation for Guardians
- Recipients can submit status or excuses through AI.
- AI summarizes updates and shows trends.
- Guardians can select tone profile (`neutral`, `supportive`, `firm`).
- AI push-back is bounded by policy and audit logged.

## 8. Incentives and Consequences
- Configure catalog entries in guardian settings.
- Map rules to triggers (for example `miss_streak_3`).
- High-impact consequences should require guardian approval.
- Every incentive/consequence application is logged with reason.

## 9. Safety Controls
- Block member interactions when needed.
- Mute or snooze reminder streams.
- Report abuse from any screen.
- Use relationship suspension when repeated misuse occurs.

## 10. Reports and Insights
Guardian reports include:
- Completion rate
- Median response time
- Missed count
- Category performance
- Excuse frequency categories
- Strategy effectiveness
- Incentive/consequence effectiveness
- Weekly trend summaries

## 11. Notifications and SMS
- Push notifications are primary channel.
- SMS requires explicit opt-in.
- Users can stop SMS and re-enable later.
- Quiet hours and daily caps are enforced server-side.

## 12. Privacy, Retention, and Deletion
- Data is encrypted in transit and at rest.
- Retention windows:
  - Operational events: 24 months
  - Notification metadata: 12 months
  - User content: until account deletion or 24 months inactivity
  - Audit/moderation records: 36 months
- Deleted accounts are purged/anonymized within 30 days, except required audit/legal records.

## 13. Common Guardian Workflows
### A. Co-own a policy with another guardian
1. Open policy settings.
2. Add guardian co-owner.
3. Confirm ownership and approval behavior.

### B. Respond to repeated misses
1. Open recipient trend view.
2. Check miss streak and excuse history.
3. Confirm escalation history.
4. Apply or approve policy consequence if appropriate.
5. Adjust due time, strategy, or incentive plan.

### C. Handle a disputed consequence
1. Open audit timeline.
2. Review trigger and approvals.
3. Reverse or amend consequence if needed.
4. Add a note for accountability.

## 14. Troubleshooting
- Not receiving push: verify app notification permission and quiet-hour window.
- Not receiving SMS: verify opt-in status and US phone format.
- Task not escalating: check daily cap and throttles.
- Cannot view reports: only guardians have report visibility.

## 15. Best Practices
- Keep tasks specific and measurable.
- Use supportive tone first, then firm when needed.
- Keep consequence catalog limited and predictable.
- Review weekly reports and tune policies gradually.

## 16. Related Specs
- `REQUIREMENTS.md`
- `ARCHITECTURE.md`
- `PREFERENCES.md`
- `POLICY_MATRIX.md`
- `SAFETY_AND_COMPLIANCE.md`
- `AI_BEHAVIOR.md`
- `INCENTIVES.md`
- `GAMIFICATION.md`
