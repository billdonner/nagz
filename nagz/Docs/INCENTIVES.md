# Incentives and Consequences Spec (V1.0)

## 1. Purpose
Define how rewards and consequences are configured, evaluated, and applied to nag outcomes.

## 2. Model
- Reward: positive outcome after completion or streak achievement.
- Consequence: predefined outcome for misses under guardian policy.
- Rule: condition + action mapping tied to nag/policy templates.
- Rule persistence: rules are stored as `incentive_rules` (see `ARCHITECTURE.md`).

## 3. Preference and Policy Interaction
- User preference toggles in `PREFERENCES.md` gate whether rewards/consequences are displayed and applied for that user.
- Family/guardian policy remains authoritative for allowed consequence actions.
- Co-owned policy edits follow dual-approval rules from `POLICY_MATRIX.md`.

## 4. Example Rule Types
- `on_time_completion` -> add points, unlock badge progress.
- `late_completion` -> reduced points.
- `missed_once` -> warning only.
- `miss_streak_3` -> apply policy consequence if guardian-approved.

## 5. Guardrails
- Consequences must come from guardian-defined catalog.
- No free-form AI-generated punishments.
- Child accounts cannot configure consequence templates.
- High-impact consequences require guardian confirmation.

## 6. Audit Requirements
Record:
- rule id and version
- triggering event(s)
- applied action
- approval source (auto vs guardian-confirmed)
- timestamp

## 7. Initial V1.0 Reward Types
- points
- streak bonus
- badge progress
- optional privilege unlock marker

## 8. Initial V1.0 Consequence Types
- point deduction
- streak reset
- require guardian review message

## 9. Out of Scope for V1.0
- Monetary transfers
- Fully automatic real-world punishments
- Cross-family public ranking consequences
