# Spec Baseline Changelog

Date: 2026-02-15
Baseline: V1.0 spec alignment pass

## Summary
- Resolved cross-document inconsistencies and underspecified policy behavior.
- Standardized preference schema versioning and API error semantics.
- Clarified escalation, co-owner governance, consent gating, and gamification policy boundaries.

## Changes
- `PREFERENCES.md`
  - Standardized `schema_version` to `1`.
  - Defined `priority_channels` runtime resolution when channels are disabled or non-compliant.
  - Moved `points_multiplier` to policy-controlled semantics.
  - Added shared API error envelope and canonical error code mapping.
- `REQUIREMENTS.md`
  - Clarified `done_definition` as enum-selected per Nag.
  - Defined V1 category as closed enum: `chores`, `meds`, `homework`, `appointments`, `other`.
  - Defined escalation behavior as `friendly_reminder` phases (time-based and behavior-based).
  - Added co-owner dual-approval/disagreement rule.
- `ARCHITECTURE.md`
  - Added common API error contract and canonical codes.
  - Added dual-approval evaluation in policy service.
  - Clarified escalation as parameterized behavior within `friendly_reminder`.
  - Added V1 consent type list.
- `AI_BEHAVIOR.md`
  - Added explicit authorization boundary: role/relationship enforcement is upstream.
- `POLICY_MATRIX.md`
  - Added co-owner dual-approval and disagreement handling.
  - Linked enforcement to shared API error codes.
- `SAFETY_AND_COMPLIANCE.md`
  - Added explicit V1 consent types and revoke behavior.
  - Classified gamification events/snapshots under retention rules.
- `GAMIFICATION.md`
  - Added consent gating for participation.
  - Marked `points_multiplier` as guardian policy control (not user-patchable).
  - Added retention note aligned with compliance policy.
- `PARENT_GUARDIAN_USER_MANUAL.md`
  - Synced strategy, category, and co-owner approval language with core specs.
- `web-samples/parent-guardian-manual.html`
  - Synced rendered manual sample with updated strategy/category/co-owner rules.
- `web-samples/guardian-portal.html`
  - Updated title label to `V1.0`.

## Notes
- This changelog reflects spec decisions only; no product implementation changes are included.
