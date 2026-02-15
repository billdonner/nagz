# Nagz Requirements (V1.0)

## 1. Purpose
Build a family-oriented nagging system where an intermediary AI reduces partner-to-partner emotional load while improving follow-through, transparency, and safety.

## 2. Product Positioning for V1.0
- AI acts as a communication intermediary for reminders and follow-ups.
- Task assigners define goals, rewards, and consequences inside policy guardrails.
- Task recipients can provide updates/excuses to AI rather than direct conflict-first negotiation.
- Guardians retain final authority over rules, reports, and consequence templates.

## 3. Roles and Relationship Rules
- Supported roles: `guardian`, `child`.
- The app supports bilateral relationships between two members.
- Guardians can nag guardians and guardians can nag children.
- Children cannot nag guardians.
- Two guardians can co-own a Nag policy.
- Co-owned policy changes require dual guardian approval; if co-owners disagree, current active policy remains.
- Only guardians can view family-level reports and history.
- The authoritative allow/deny matrix is defined in `POLICY_MATRIX.md`.

## 4. Core Task Model
- A Nag has `creator`, `recipient`, `due_at`, `category`, `strategy`, and typed `done_definition`.
- `done_definition` is selected per Nag from the V1.0 completion enum.
- V1.0 completion types:
  - `ack_only`
  - `binary_check`
  - `binary_with_note`
- V1.0 default strategy template: `friendly_reminder`.
- V1.0 `category` is a closed enum: `chores`, `meds`, `homework`, `appointments`, `other`.
- Each Nag can attach reward and consequence policy references.

## 5. AI Mediation Requirements
- AI collects recipient excuses/status updates.
- AI summarizes recipient responses for assigners.
- AI can issue bounded push-back prompts when tasks are repeatedly missed.
- AI tone options: `neutral`, `supportive`, `firm`.
- All AI actions must be auditable and attributable.

## 6. Escalation and Notifications
- Time-based escalation triggers are phase transitions within `friendly_reminder`.
- Behavior-based escalation triggers are bounded push-back and cadence adjustments within `friendly_reminder`.
- V1.0 channels: push notifications and SMS.
- V1.0 includes mandatory hard-stop enforcement:
  - quiet hours
  - daily Nag/contact limits
  - per-user/per-relationship throttles

## 7. Incentives and Consequences
- Reward and consequence catalogs are guardian-configurable.
- Rules can map task outcomes to rewards/consequences.
- Consequences must respect safety policy and role constraints.
- The system logs when and why incentives/consequences are applied.

## 8. Gamification
- Points and streak tracking.
- Weekly score summaries.
- Optional badges and family leaderboard.
- Participation can be opt-in per user.

## 9. Reporting and Metrics
Guardian-visible reporting includes:
- Completion rate
- Median response time
- Missed count
- Category performance (chores, meds, homework, appointments)
- Excuse categories and frequency
- Incentive/consequence effectiveness over time
- Strategy effectiveness over time
- Weekly family trends

Metric definitions:
- Completion rate denominator is all Nags due in period.
- Missed count includes unresolved Nags past final escalation window.
- Response time is from first delivery to completion event.

## 10. Safety and Trust
- Provide block, mute/snooze, and report-abuse controls.
- Enforce anti-harassment limits server-side.
- Keep immutable audit trail for create/edit/escalate/deliver/AI-action/complete events.
- Guardian review path exists for disputed consequences.

## 11. Compliance and Privacy
- Child-account safeguards and guardian consent model are required (US launch scope for V1.0).
- SMS opt-in and unsubscribe semantics are required (US-only launch jurisdiction in V1.0).
- Sensitive data in logs/notifications must be minimized.
- Retention/deletion behavior is policy-driven and auditable.
- Detailed requirements are defined in `SAFETY_AND_COMPLIANCE.md`.

## 12. Account Lifecycle
- Account creation and role assignment must be explicit and auditable.
- Account deletion requests must be supported.
- Family leave/removal flows must be supported.
- Required audit records are retained per policy.

## 13. Linked Specifications
- `PREFERENCES.md`
- `ARCHITECTURE.md`
- `POLICY_MATRIX.md`
- `SAFETY_AND_COMPLIANCE.md`
- `AI_BEHAVIOR.md`
- `INCENTIVES.md`
- `GAMIFICATION.md`

## 14. Out of Scope for V1.0
- Additional delivery channels beyond push/SMS
- Full peer-to-peer synchronization as primary transport
- Unbounded AI autonomy without policy constraints
- Fully automated consequences without guardian-defined templates
- Negotiated done-definition workflow
