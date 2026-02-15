# Gamification Spec (V1)

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
- Miss streak penalty and bonuses are policy-driven.

## 4. Participation and Consent
- Gamification can be enabled/disabled per user.
- Leaderboard visibility is optional.
- Guardians can require non-public mode for child accounts.

## 5. Anti-Gaming Controls
- Only canonical server events can change score.
- Duplicate/offline replay protection via idempotency keys.
- Suspicious activity flagged for guardian review.

## 6. Reporting Outputs
- Weekly score delta
- Current streaks and streak breaks
- Badge progress
- Correlation between gamification and completion rate

## 7. V1 Constraints
- Keep formulas explainable and deterministic.
- Avoid hidden multipliers.
- No cross-family global leaderboard.
