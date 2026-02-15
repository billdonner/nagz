# Nagz Glossary (V1.0)

## 1. Canonical Naming
- Canonical object name: `nag` (lowercase in prose unless sentence-start, lowercase in schema names).
- Canonical actor term: `creator`.
- `assigner` is an allowed synonym in user-facing copy, but backend and specs should use `creator`.
- Canonical completion field: `done_definition`.
- `completion type` refers to the enum value stored in `done_definition`.

## 2. Core Domain Terms
- `nag`: A reminder task from a `creator` to a `recipient`.
- `recipient`: The user expected to complete/update a nag.
- `strategy_template`: Named escalation/message family (V1.0 supports `friendly_reminder` only).
- `escalation_phase`: A runtime stage within a strategy template.
- `done_definition`: Completion schema enum for a nag (`ack_only`, `binary_check`, `binary_with_note`).
- `incentive_rule`: Condition + action mapping for reward/consequence logic.
- `hard-stop policy`: Safety constraints enforced server-side (quiet hours, caps, throttles).

## 3. Role Semantics
- Roles are family-scoped through `family_memberships.role`.
- A user can have one role per family in V1.0.
- V1.0 does not allow a dual-role membership for the same user in the same family.

## 4. Authority
- Client UI may hide options but never authoritatively enforces policy.
- Server policy and authorization checks are authoritative on all writes.
