# Gamification Spec (V1.0)

## 1. Purpose
Increase consistency and motivation through simple, transparent game mechanics.

## 2. Core Mechanics
- Points: earned from completion behavior.
- Streaks: consecutive completions across configured periods.
- Badges: milestone achievements.
- Leaderboard: optional family ranking view.

## 3. Initial Scoring Rules
- On-time completion: +10 points
- Late completion: +4 points
- Missed task: 0 points
- Miss streak penalties and bonuses are policy-driven by guardian-managed rules.

## 4. Policy and Preference Linkage
- Participation is gated by `gamification_participation` consent state.
- User toggles and visibility preferences are defined in `PREFERENCES.md`.
- `points_multiplier` is guardian-controlled family policy (`0.1..3.0`) and is not user-patchable.
- Role/visibility constraints are enforced per `POLICY_MATRIX.md`.

## 5. Anti-Gaming Controls
- Only canonical server events can change score.
- Duplicate/offline replay protection via idempotency keys.
- Suspicious activity flagged for guardian review.

## 6. Reporting Outputs
- Weekly score delta
- Current streaks and streak breaks
- Badge progress
- Correlation between gamification and completion rate

## 7. Data and Retention
- Event storage is modeled as `gamification_events` and `gamification_snapshots` in `ARCHITECTURE.md`.
- Retention policy is authoritative in `SAFETY_AND_COMPLIANCE.md`.

## 8. V1.0 Constraints
- Keep formulas explainable and deterministic.
- Avoid hidden multipliers.
- No cross-family global leaderboard.
