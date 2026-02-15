# Nagz Preferences and Sync API (V0.5)

## 1. Goals
- Persist user-level settings across devices.
- Support role-aware defaults for guardians vs children.
- Configure AI mediation behavior safely.
- Configure gamification participation and notification intensity.
- Keep schema evolvable with explicit versioning and migrations.

## 2. Preference Resolution Order
Effective preferences resolve from lowest to highest precedence:
1. App defaults
2. Family/workspace defaults
3. User preferences
4. Session overrides (local only, not persisted)

## 3. Canonical Server Document
```json
{
  "schema_version": 2,
  "user_id": "usr_123",
  "family_id": "fam_abc",
  "role": "guardian",
  "updated_at": "2026-02-15T18:00:00Z",
  "prefs": {
    "notifications": {
      "push_enabled": true,
      "sms_enabled": false,
      "quiet_hours": {
        "enabled": true,
        "start_local": "21:00",
        "end_local": "07:00",
        "timezone": "America/Los_Angeles"
      },
      "daily_nag_limit": 8,
      "priority_channels": ["push", "sms"]
    },
    "nag_defaults": {
      "default_strategy": "friendly_reminder",
      "default_due_offset_minutes": 60,
      "allow_behavior_escalation": true,
      "allow_time_escalation": true
    },
    "ai_mediation": {
      "enabled": true,
      "tone": "supportive",
      "pushback_mode": "bounded",
      "summary_frequency": "daily",
      "tips_enabled": true
    },
    "incentives": {
      "rewards_enabled": true,
      "consequences_enabled": true,
      "guardian_approval_required": true
    },
    "gamification": {
      "enabled": true,
      "show_leaderboard": false,
      "show_badges": true,
      "points_multiplier": 1.0
    },
    "consent_controls": {
      "allow_guardian_reports": true,
      "allow_snooze": true,
      "max_snooze_minutes": 30,
      "mute_until": null
    },
    "ui": {
      "locale": "en-US",
      "time_format": "12h",
      "week_start": "monday"
    }
  }
}
```

## 4. Key Constraints
- `schema_version`: integer; required.
- `role`: `guardian` or `child`; required.
- `default_strategy`: V0.5 only supports `friendly_reminder`.
- `daily_nag_limit`: integer `0..100`.
- `priority_channels`: subset of `["push", "sms"]`.
- `ai_mediation.tone`: `neutral`, `supportive`, or `firm`.
- `ai_mediation.pushback_mode`: `off` or `bounded`.
- `gamification.points_multiplier`: decimal `0.1..3.0`.
- `mute_until`: RFC3339 timestamp or `null`.

## 5. Role-Aware Rules
- Child accounts cannot enable policies that allow child-to-guardian nagging.
- Only guardians can set defaults that affect consequences and reporting.
- Server enforces policy and role constraints even if client payload is invalid.

## 6. Minimal Sync API
Base path: `/api/v1`

### 6.1 Get Effective Preferences
`GET /api/v1/preferences?family_id={familyId}`

Response headers:
- `ETag: "pref_v2_usr_123_18"`

Response body includes:
- `schema_version`
- `etag`
- `effective` preferences
- optional `sources` map (which layer won for selected keys)

### 6.2 Patch User Preferences
`PATCH /api/v1/preferences?family_id={familyId}`

Request headers:
- `If-Match: "pref_v2_usr_123_18"` (required)

Request body:
```json
{
  "schema_version": 2,
  "set": {
    "ai_mediation.tone": "firm",
    "gamification.enabled": true,
    "notifications.daily_nag_limit": 6
  },
  "unset": [
    "consent_controls.mute_until"
  ]
}
```

Responses:
- `200 OK` updated effective preferences and new `etag`
- `412 Precondition Failed` stale `If-Match`
- `422 Unprocessable Entity` invalid keys/values

## 7. Merge and Conflict Behavior
- Patch updates only specified paths.
- Unspecified keys remain unchanged.
- `unset` removes the user override and falls back to lower precedence.
- Use optimistic locking with `ETag` and `If-Match`.
- Retry flow on `412`: `GET` latest -> reapply intent -> `PATCH`.

## 8. Client Caching and Offline
- Cache last successful effective preferences and `etag` locally.
- Apply session overrides in-memory only.
- Queue patches offline and replay in order once online.
- Drop queued operations that fail against changed validation constraints.

## 9. Migration Strategy
- Keep server migrations from `schema_version N` to `N+1`.
- Migrate on read before merge/resolution.
- Return latest schema in API responses.
- Example migration v1 -> v2:
  - Add `ai_mediation`, `incentives`, and `gamification` blocks with safe defaults.

## 10. Security and Audit
- Do not store secrets in preference documents.
- Emit audit events for preference changes with user, family, keys, and timestamp.
- Redact sensitive values in logs where applicable.
