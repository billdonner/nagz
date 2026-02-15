# Nagz Preferences and Sync API (V1)

## 1. Goals
- Persist user-level settings across devices.
- Support role-aware defaults for guardians vs children.
- Allow safe, partial updates from clients.
- Keep schema evolvable with explicit versioning and migrations.

## 2. Preference Resolution Order
Effective preferences are resolved in this order (lowest to highest precedence):
1. App defaults
2. Family/workspace defaults
3. User preferences
4. Session overrides (local only, not persisted)

## 3. Canonical Server Document
The server stores one canonical document per user and family workspace.

```json
{
  "schema_version": 1,
  "user_id": "usr_123",
  "family_id": "fam_abc",
  "role": "guardian",
  "updated_at": "2026-02-15T17:12:00Z",
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
- `default_strategy`: V1 only supports `friendly_reminder`.
- `daily_nag_limit`: integer `0..100`.
- `start_local` and `end_local`: `HH:mm` 24-hour format.
- `priority_channels`: subset of `["push", "sms"]` for V1.
- `mute_until`: RFC3339 timestamp or `null`.

## 5. Role-Aware Rules
- Child accounts cannot enable settings that permit child-to-guardian nagging.
- Guardian-only reporting visibility is controlled by server authorization, not only preference values.
- Server enforces final policy constraints even if client sends invalid combinations.

## 6. Minimal Sync API
Base path: `/api/v1`

### 6.1 Get Effective Preferences
`GET /api/v1/preferences?family_id={familyId}`

Response headers:
- `ETag: "pref_v1_usr_123_17"`

Response body:
```json
{
  "schema_version": 1,
  "etag": "pref_v1_usr_123_17",
  "effective": {
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
  },
  "sources": {
    "notifications.daily_nag_limit": "user",
    "nag_defaults.default_strategy": "app_default"
  }
}
```

### 6.2 Patch User Preferences
`PATCH /api/v1/preferences?family_id={familyId}`

Request headers:
- `If-Match: "pref_v1_usr_123_17"` (required)

Request body:
```json
{
  "schema_version": 1,
  "set": {
    "notifications.sms_enabled": true,
    "notifications.daily_nag_limit": 6,
    "ui.time_format": "24h"
  },
  "unset": [
    "consent_controls.mute_until"
  ]
}
```

Success response:
- `200 OK` with updated effective document and a new `etag`.

Conflict response:
- `412 Precondition Failed` if `If-Match` is stale.

Validation response:
- `422 Unprocessable Entity` with invalid keys and reasons.

## 7. Merge and Conflict Behavior
- Partial patch updates only specified paths.
- Unspecified keys are unchanged.
- `unset` removes user override and falls back to lower-precedence value.
- Concurrency control uses optimistic locking via `ETag` and `If-Match`.
- Client retry flow on `412`:
  1. `GET` latest
  2. Reapply local intent
  3. `PATCH` with new `If-Match`

## 8. Client Caching and Offline
- Cache last successful effective preferences locally with `etag`.
- Apply session overrides in-memory only.
- Queue patches offline and replay in order when online.
- Drop queued patch if validation rules changed and payload is invalid.

## 9. Migration Strategy
- Keep migration functions on server from `schema_version N` to `N+1`.
- Run migration on read before merge/resolution.
- Return latest schema in API response.
- Example migration v1 -> v2:
  - Rename `ui.week_start` to `ui.calendar_week_start`.
  - Preserve old key read compatibility for one release window.

## 10. Security and Audit
- Do not store secrets in this preference document.
- Emit audit events for preference changes:
  - `user_id`, `family_id`, changed keys, timestamp, request id
- Redact preference values in logs for sensitive keys (future use).
