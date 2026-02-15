# Nagging Systems - Requirements (V1)

## 1. Purpose
Build a family-oriented **Nagging System** where members can assign reminders ("Nags") to each other, with escalation and reporting to improve follow-through.

## 2. Relationship and Roles
- The system supports **bilateral relationships** between two people.
- In a bilateral relationship, **either party can create a Nag** for the other party.
- **Children cannot nag parents/guardians**.
- **Parents/guardians can nag each other**.
- A Nag policy can be **co-owned by two guardians**.
- **Only guardians** can view history and reports.

## 3. Core Nag Model
- A Nag is created by one party and delivered to the other party.
- "Done" criteria are **Nag-specific** (not globally fixed).
- Completion definition may be **negotiated between the two parties** in future versions.

## 4. Escalation Requirements
- Escalation must support both:
  - **Time-based triggers** (e.g., intervals after due time)
  - **Behavior-based triggers** (e.g., repeated misses)
- Initial required channels:
  - **Push notifications**
  - **SMS**
- Architecture should remain open for future channels (email, voice, etc.).
- Include **hard-stop rules** (quiet hours, daily limits, similar guardrails) in a later phase.

## 5. Strategy Templates (V1)
- Start with a single template:
  - **Friendly Reminder**

## 6. Reporting and Metrics
Support guardian-visible reports including:
- Completion rate
- Median response time
- Missed count
- Performance by Nag category (chores, meds, homework, appointments)
- Strategy effectiveness over time
- Weekly family summary trends

## 7. Safety and Trust
- Include anti-abuse guardrails.
- Support consent-aware controls (mute/snooze/limits).
- Keep auditable activity history for accountability.
- Enforce privacy boundaries within family accounts.

## 8. Out of Scope for V1 (but planned)
- Expanded strategy templates beyond Friendly Reminder
- Full negotiation workflow for "Done" criteria
- Additional escalation channels beyond Push/SMS
- Full hard-stop policy engine

## 9. Preferences and Sync (V1)
Detailed schema, precedence, and API contract are defined in `PREFERENCES.md`.
