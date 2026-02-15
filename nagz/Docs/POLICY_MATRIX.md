# Nagz Policy Matrix (V1.0)

## 1. Scope
This document defines the authoritative allow/deny rules for who can create nags for whom in V1.0.

## 2. Role Pair Permissions
| Creator Role | Recipient Role | Allowed | Notes |
|---|---|---|---|
| guardian | guardian | Yes | Includes bilateral nagging between guardians. |
| guardian | child | Yes | Guardian-to-child allowed when relationship is active. |
| child | guardian | No | Explicitly denied in V1.0. |
| child | child | No | Denied in V1.0 to reduce abuse and moderation risk. |

Self-nag rules:
- guardian -> guardian (self): allowed.
- child -> child (self): denied in V1.0.

Role cardinality rule:
- A user can hold only one role in a given family membership in V1.0.

## 3. Relationship and Co-Owner Requirements
- nag creation requires an active bilateral relationship record.
- Suspended or revoked relationships cannot create or receive nags.
- A co-owned nag policy is valid only when both guardian owners have active status.
- Co-owned policy edits require both guardian owners to approve.
- On co-owner disagreement, keep the current active policy and deny consequence-expanding changes.

## 4. Report and History Visibility
| Viewer Role | Access |
|---|---|
| guardian | Full family history/reports for authorized family scope |
| child | No access to family reports/history |

## 5. Enforcement Rules
- Server enforces all matrix rules on every write request.
- Client UI may hide disallowed actions, but server enforcement is authoritative.
- Violating requests return authorization/policy errors and are audit-logged.
- Errors follow shared API codes from `ARCHITECTURE.md` and `PREFERENCES.md`.

## 6. Policy Change Governance
- Any future change to role pair permissions requires:
  - Product review
  - Safety/compliance review
  - Updated App Review notes and test coverage
