# Nagz Requirements (V0.5)

## 1. Purpose
Build a family-oriented nagging system where an intermediary AI reduces partner-to-partner emotional load while improving follow-through, transparency, and safety.

## 2. Product Positioning for V0.5
- AI acts as the communication intermediary for reminders and follow-ups.
- Task assigners define goals, rewards, and consequences within guardrails.
- Task recipients can provide updates and excuses to the AI instead of directly negotiating in the moment.
- Guardians retain final authority over rules, reporting, and consequences.

## 3. Roles and Relationship Rules
- Supports bilateral relationships between two people.
- Parents/guardians can nag each other.
- Children cannot nag parents/guardians.
- Only guardians can view family-level reports and configure consequence templates.
- A nag policy can be co-owned by two guardians.

## 4. Core Task Model
- A Nag has creator, recipient, due time, done criteria, and category.
- Done criteria are nag-specific.
- Each Nag can attach a reward policy and consequence policy.
- V0.5 supports one default strategy template: `friendly_reminder`.

## 5. AI Mediation Requirements
- AI collects excuses/status updates from recipients.
- AI summarizes recipient responses for assigners.
- AI can issue bounded push-back prompts when tasks are repeatedly missed.
- AI tone options: neutral, supportive, firm.
- All AI actions must be auditable and attributable.

## 6. Escalation and Notifications
- Time-based triggers (after due time).
- Behavior-based triggers (miss streaks, repeated delays).
- V0.5 channels: push notifications and SMS.
- Hard-stop and anti-spam controls:
  - quiet hours
  - daily contact limits
  - per-user throttles

## 7. Incentives and Consequences
- Reward and consequence catalogs are guardian-configurable.
- Rules can map task outcomes to rewards/consequences.
- Consequences must respect safety policy and role constraints.
- The system logs when and why incentives are applied.

## 8. Gamification
- Points and streak tracking.
- Weekly score summaries.
- Optional badges and family leaderboard.
- Gamification participation can be opt-in per user.

## 9. Reporting and Metrics
Guardian-visible reporting includes:
- Completion rate
- Median response time
- Missed count
- Excuse categories and frequency
- Incentive effectiveness over time
- Strategy effectiveness over time
- Weekly family trends

## 10. Safety and Trust
- Consent-aware controls: mute, snooze, limits.
- Anti-abuse guardrails and escalation caps.
- Explainability log for AI decisions and policy actions.
- Audit trail for assignments, reminders, excuses, and outcomes.
- Guardian review path for disputed consequences.

## 11. Linked Specifications
- Preferences and sync API: `PREFERENCES.md`
- Architecture: `ARCHITECTURE.md`
- AI behavior policy: `AI_BEHAVIOR.md`
- Incentive model: `INCENTIVES.md`
- Gamification model: `GAMIFICATION.md`

## 12. Out of Scope for V0.5
- Voice or email escalation channels
- Open-ended AI autonomy without policy boundaries
- Fully automated punishment without guardian-defined templates
- Full negotiation workflow for changing done criteria
