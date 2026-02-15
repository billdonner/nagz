# Nagz Policy Matrix (V1.0)

## 1. Scope
This document defines the authoritative allow/deny rules for who can create Nags for whom in V1.0.

## 2. Role Pair Permissions
| Creator Role | Recipient Role | Allowed | Notes |
|---|---|---|---|
| guardian | guardian | Yes | Includes bilateral nagging between guardians. |
| guardian | child | Yes | Guardian-to-child allowed when relationship is active. |
| child | guardian | No | Explicitly denied in V1.0. |
| child | child | No | Denied in V1.0 to reduce abuse and moderation risk. |

## 3. Relationship State Requirements
- Nag creation requires an active bilateral relationship record.
- Suspended or revoked relationships cannot create or receive Nags.
- A co-owned Nag policy is valid only when both guardian owners have active status.

## 4. Report and History Visibility
| Viewer Role | Access |
|---|---|
| guardian | Full family history/reports for authorized family scope |
| child | No access to family reports/history |

## 5. Enforcement Rules
- Server enforces all matrix rules on every write request.
- Client UI may hide disallowed actions, but server enforcement is authoritative.
- Violating requests return authorization error and are audit-logged.

## 6. Policy Change Governance
- Any future change to role pair permissions requires:
  - Product review
  - Safety/compliance review
  - Updated App Review notes and test coverage
