# Nagz Policy Matrix (V1.0)

## 1. Scope
This document defines the authoritative allow/deny rules for who can create nags for whom in V1.0.

## 2. Role Pair Permissions
| Creator Role | Recipient Role | Allowed | Notes |
|---|---|---|---|
| guardian | guardian | Yes | Includes bilateral nagging between guardians. |
| guardian | participant | Yes | Guardian-to-participant allowed when relationship is active. |
| guardian | child | Yes | Guardian-to-child allowed when relationship is active. |
| participant | child | Yes | Participant-to-child allowed when relationship is active. |
| participant | guardian | Yes | Participant-to-guardian allowed when relationship is active. |
| participant | participant | Yes | Participant-to-participant allowed when relationship is active. |
| child | guardian | No | Explicitly denied in V1.0. |
| child | participant | No | Denied in V1.0. |
| child | child | No | Denied in V1.0 to reduce abuse and moderation risk. |

Self-nag rules:
- guardian -> guardian (self): allowed.
- participant -> participant (self): allowed.
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
| participant | No access to family reports/history |
| child | No access to family reports/history |

## 5. Enforcement Rules
- Server enforces all matrix rules on every write request.
- Client UI may hide disallowed actions, but server enforcement is authoritative.
- Violating requests return authorization/policy errors and are audit-logged.
- Errors follow shared API codes from `ARCHITECTURE.md` and `PREFERENCES.md`.

## 6. Cross-Family Trusted Connection Rules

When two adults are connected and the connection is marked as **trusted**, additional nag creation paths open:

| Creator | Recipient | Allowed | Condition |
|---|---|---|---|
| Connected adult (guardian/participant) | Other party's child | Yes | Connection active AND trusted |
| Connected adult (guardian/participant) | Other party's child | No | Connection active but NOT trusted |
| Connected adult (guardian/participant) | Unrelated user | No | Recipient must be other party or their child |

Trusted connection nags use `connection_id` (not `family_id`) and follow the existing XOR constraint on the nag table. Untrusting a connection cancels all open trusted-child nags. Revoking a connection resets `trusted = false`.

## 7. Policy Change Governance
- Any future change to role pair permissions requires:
  - Product review
  - Safety/compliance review
  - Updated App Review notes and test coverage
