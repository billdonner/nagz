# Nagz Requirements (V1.0)

## 1. Purpose
Build a family-oriented nagging system where an intermediary AI reduces partner-to-partner emotional load while improving follow-through, transparency, and safety.

## 2. Product Positioning for V1.0
- AI acts as a communication intermediary for reminders and follow-ups.
- Task creators define goals, rewards, and consequences inside policy guardrails.
- Task recipients can provide updates/excuses to AI rather than direct conflict-first negotiation.
- Guardians retain final authority over rules, reports, and consequence templates.

## 3. Terms and Naming
- Canonical terms follow `GLOSSARY.md`.
- Canonical actor term is `creator` (`assigner` is a UI synonym only).
- Canonical completion field is `done_definition`; completion enum values are called completion types.
- V1.0 uses `strategy_template` with runtime `escalation_phase` values.

## 4. Roles and Relationship Rules
- Supported roles: `guardian`, `participant`, `child`.
- Roles are family-scoped.
- The app supports bilateral relationships between two members.
- Guardians can create nags for guardians, participants, and children.
- Participants can create nags for guardians, participants, and children.
- Children cannot create nags for guardians.
- Children cannot create nags for children in V1.0.
- New members joining a family default to the `participant` role (not `guardian`).
- Self-nags are allowed only for guardians in V1.0.
- Two guardians can co-own a nag policy.
- Co-owned policy changes require dual guardian approval; if co-owners disagree, current active policy remains.
- Only guardians can view family-level reports and history.
- The authoritative allow/deny matrix is defined in `POLICY_MATRIX.md`.

## 5. Core Nag Model
- A nag has `creator`, `recipient`, `due_at`, `category`, `strategy_template`, typed `done_definition`, optional `description`, and optional `recurrence`.
- `done_definition` is selected per nag from the V1.0 completion enum.
- V1.0 recurrence values: `daily`, `weekly`, `monthly` (or none for one-off nags).
- Recurring nags create child nag instances linked via `parent_nag_id`.
- V1.0 completion types:
  - `ack_only`
  - `binary_check`
  - `binary_with_note`
- V1.0 strategy template support:
  - `friendly_reminder` only
- V1.0 `category` is a closed enum:
  - `chores`
  - `meds`
  - `homework`
  - `appointments`
  - `other`
- Each nag can attach policy references for incentives/consequences.

## 6. Escalation and Notifications
- Escalation executes inside `friendly_reminder` phases (not separate strategies).
- V1.0 phase model:
  - `phase_0_initial`: at creation/delivery.
  - `phase_1_due_soon`: reminder window before `due_at`.
  - `phase_2_overdue_soft`: first overdue reminder window.
  - `phase_3_overdue_bounded_pushback`: bounded behavior-based push-back.
  - `phase_4_guardian_review`: requires guardian intervention for unresolved misses.
- Phase transitions are blocked by hard-stop policy when required.
- V1.0 channels: push notifications and SMS.
- V1.0 mandatory hard-stop controls:
  - quiet hours
  - daily nag/contact limits
  - per-user/per-relationship throttles
- Default daily nag cap is `8` per creator per family per local day; guardian policy can lower to `1..8`.

## 7. AI Mediation Requirements
- AI collects recipient excuses/status updates.
- AI summarizes recipient responses for creators.
- AI can issue bounded push-back prompts when tasks are repeatedly missed.
- AI tone options: `neutral`, `supportive`, `firm`.
- All AI actions must be auditable and attributable.
- Authorization must be enforced before AI execution; unauthorized requests are rejected fail-closed.

## 8. Incentives and Consequences
- Reward and consequence catalogs are guardian-configurable.
- Rules map task outcomes to rewards/consequences.
- Consequences must respect safety policy and role constraints.
- The system logs when and why incentives/consequences are applied.

## 9. Gamification
- Points and streak tracking.
- Weekly score summaries.
- Optional badges and family leaderboard.
- Participation can be opt-in per user under consent and policy constraints.

## 10. Reporting and Metrics
Guardian-visible reporting includes:
- Completion rate
- Median response time
- Missed count
- Category performance (`chores`, `meds`, `homework`, `appointments`, `other`)
- Excuse categories and frequency
- Incentive/consequence effectiveness over time
- Strategy effectiveness over time
- Weekly family trends

Metric definitions:
- Completion rate denominator is all nags due in period.
- Missed count includes unresolved nags past final escalation window.
- Response time is from first delivery to completion event.

## 11. Safety and Trust
- Provide block, mute/snooze, and report-abuse controls.
- Enforce anti-harassment limits server-side.
- Keep immutable audit trail for create/edit/escalate/deliver/AI-action/complete events.
- Guardian review path exists for disputed consequences.

## 12. Compliance and Privacy
- Child-account safeguards and guardian consent model are required (US launch scope for V1.0).
- SMS opt-in and unsubscribe semantics are required (US-only launch jurisdiction in V1.0).
- Sensitive data in logs/notifications must be minimized.
- Retention/deletion behavior is policy-driven and auditable.
- Detailed requirements are defined in `SAFETY_AND_COMPLIANCE.md`.

## 13. Account and Family Lifecycle
- Account creation and role assignment must be explicit and auditable.
- Account deletion requests must be supported.
- Family leave/removal flows must be supported.
- Lifecycle edge rules:
  - removing a recipient closes open nags assigned to that recipient as `cancelled_relationship_change`.
  - removing a creator keeps historical nags immutable and blocks new nag creation.
  - if a co-owner guardian leaves, co-owned policies move to single-owner mode and consequence-expanding edits remain blocked until a new co-owner is approved.
- Required audit records are retained per policy.

## 14. Linked Specifications
- `CATALOG.md`
- `GLOSSARY.md`
- `API_SURFACE.md`
- `PREFERENCES.md`
- `ARCHITECTURE.md`
- `POLICY_MATRIX.md`
- `SAFETY_AND_COMPLIANCE.md`
- `AI_BEHAVIOR.md`
- `INCENTIVES.md`
- `GAMIFICATION.md`
- `PARENT_GUARDIAN_USER_MANUAL.md`
- `SPEC_BASELINE_CHANGELOG.md`

## 15. Out of Scope for V1.0
- Additional delivery channels beyond push/SMS
- Full peer-to-peer synchronization as primary transport
- Unbounded AI autonomy without policy constraints
- Fully automated consequences without guardian-defined templates
- Negotiated done-definition workflow
