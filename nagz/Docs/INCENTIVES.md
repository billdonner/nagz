# Incentives and Consequences Spec (V0.5)

## 1. Purpose
Define how rewards and consequences are configured, evaluated, and applied to task outcomes.

## 2. Model
- Reward: positive outcome after completion or streak achievement.
- Consequence: predefined outcome for misses under guardian policy.
- Rule: condition + action mapping tied to task or policy template.

## 3. Example Rule Types
- `on_time_completion` -> add points, unlock badge progress.
- `late_completion` -> reduced points.
- `missed_once` -> warning only.
- `miss_streak_3` -> apply policy consequence if guardian-approved.

## 4. Guardrails
- Consequences must come from guardian-defined catalog.
- No free-form AI-generated punishments.
- Child accounts cannot configure consequence templates.
- High-impact consequences require guardian confirmation.

## 5. Audit Requirements
Record:
- rule id and version
- triggering event(s)
- applied action
- approval source (auto vs guardian-confirmed)
- timestamp

## 6. Initial V0.5 Reward Types
- points
- streak bonus
- badge progress
- optional privilege unlock marker

## 7. Initial V0.5 Consequence Types
- point deduction
- streak reset
- require guardian review message

## 8. Out of Scope for V0.5
- Monetary transfers
- Fully automatic real-world punishments
- Cross-family public ranking consequences
