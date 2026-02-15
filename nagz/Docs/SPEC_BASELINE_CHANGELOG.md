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

---

Date: 2026-02-15
Baseline: V1.0 spec remediation pass (review-driven)

## Summary
- Normalized naming and cross-doc references.
- Completed missing data model and API surface definitions.
- Added explicit escalation, snooze, consent-revocation, and compliance behavior.

## Changes
- Added `GLOSSARY.md` as canonical terminology and role semantics source.
- Added `API_SURFACE.md` to cover non-preferences endpoint scope.
- Updated `REQUIREMENTS.md` with:
  - canonical naming rules (`creator`, `strategy_template`, `done_definition`)
  - explicit escalation phases
  - explicit daily cap default/range
  - lifecycle edge handling rules
  - expanded linked-spec list
- Updated `ARCHITECTURE.md` with:
  - complete core data model (`families`, `incentive_rules`, `policy_id` linkage, `gamification_events`, `gamification_snapshots`)
  - API error code split (`POLICY_FORBIDDEN` vs `POLICY_INVALID_VALUE`)
  - API surface linkage
- Updated `PREFERENCES.md` with:
  - renamed `consent_controls` -> `interaction_controls`
  - explicit policy bounds (`max_snooze_minutes`, push-back bounds)
  - unambiguous error code mapping
  - normative snooze semantics
  - explicit links to AI/incentives/gamification specs
- Updated `AI_BEHAVIOR.md` with:
  - configuration source reference to preferences
  - fail-closed unauthorized behavior
  - concrete push-back defaults/ranges
- Updated `INCENTIVES.md` and `GAMIFICATION.md` with explicit preference/policy linkage.
- Updated `POLICY_MATRIX.md` with self-nag and single-role-per-family rules.
- Updated `SAFETY_AND_COMPLIANCE.md` with:
  - explicit COPPA reference
  - age-gate behavior
  - consent revocation in-flight handling
  - breach response requirements
  - explicit snapshot/event retention entries by table class
- Updated `PARENT_GUARDIAN_USER_MANUAL.md` for terminology and phase/snooze clarity.
- Updated `CATALOG.md` and root `README.md` to include new docs.
