# Nagz Preferences and Sync API (V1.0)

## 1. Goals
- Persist user-level settings across devices.
- Support role-aware defaults for guardians vs children.
- Configure AI mediation and gamification behavior safely.
- Allow safe partial updates.
- Keep policy controls server-authoritative.
- Keep schema evolvable with explicit migrations.

## 2. Resolution Order
Effective preferences resolve from lowest to highest precedence:
1. App defaults
2. Family defaults
3. User preferences
4. Session overrides (local only, not persisted)

## 3. Canonical Server Shape
The server stores one canonical preference document per `user_id` and `family_id`.

```json
{
  "schema_version": 1,
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
      "show_badges": true
    },
    "consent_controls": {
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

## 4. Policy vs Preference Boundary
- Policy is server-authoritative and not patchable by end users.
- Preferences are user-scoped and patchable within server rules.
- Examples of policy-only fields:
  - role/relationship permissions
  - guardian-only report access
  - global hard-stop limits
  - gamification scoring multiplier (`points_multiplier`)

## 5. Key Constraints
- `schema_version`: `1` for this release.
- `role`: `guardian` or `child`, required.
- `default_strategy`: V1.0 supports only `friendly_reminder`.
- `priority_channels`: subset of `["push", "sms"]`.
- Delivery resolution for `priority_channels`: evaluate in order and skip channels that are disabled or currently non-compliant (for example, SMS unsubscribed).
- `ai_mediation.tone`: `neutral`, `supportive`, or `firm`.
- `ai_mediation.pushback_mode`: `off` or `bounded`.
- `points_multiplier`: policy-controlled decimal `0.1..3.0` and read-only in user preference patch APIs.
- `mute_until`: RFC3339 timestamp or `null`.

## 6. Minimal Sync API
Base path: `/api/v1`

### 6.1 Get Effective Configuration
`GET /api/v1/preferences?family_id={familyId}`

Response headers:
- `ETag: "pref_v1_usr_123_18"`

Response body includes effective user prefs and read-only policy:

```json
{
  "schema_version": 1,
  "etag": "pref_v1_usr_123_18",
  "effective": {
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
        "show_badges": true
      }
    },
    "policy": {
      "report_visibility": "guardian_only",
      "daily_nag_cap": 8,
      "child_can_nag_guardian": false,
      "gamification_points_multiplier": 1.0
    }
  }
}
```

### 6.2 Patch User Preferences
`PATCH /api/v1/preferences?family_id={familyId}`

Request headers:
- `If-Match: "pref_v1_usr_123_18"` (required)

Request body:

```json
{
  "schema_version": 1,
  "set": {
    "prefs.ai_mediation.tone": "firm",
    "prefs.gamification.enabled": true,
    "prefs.notifications.sms_enabled": true
  },
  "unset": [
    "prefs.consent_controls.mute_until"
  ]
}
```

Responses:
- `200 OK` with updated effective configuration and new `etag`.
- `412 Precondition Failed` if `If-Match` is stale.
- `422 Unprocessable Entity` for invalid keys or policy-violating values.
- `403 Forbidden` for role/authorization or policy-scope violations.
- `429 Too Many Requests` for rate limiting.

### 6.3 Error Envelope
All non-2xx API responses use this envelope:

```json
{
  "error": {
    "code": "POLICY_VIOLATION",
    "message": "Field is read-only for this role.",
    "request_id": "req_abc123",
    "details": {
      "field": "prefs.gamification.points_multiplier"
    }
  }
}
```

Canonical V1 codes:
- `AUTHZ_DENIED` -> `403`
- `VALIDATION_ERROR` -> `422`
- `POLICY_VIOLATION` -> `403` or `422`
- `PRECONDITION_FAILED` -> `412`
- `RATE_LIMITED` -> `429`
- `NOT_FOUND` -> `404`

## 7. Merge and Conflict Behavior
- Partial patch updates only specified keys.
- Unspecified keys are unchanged.
- `unset` removes user override and falls back to lower precedence.
- Optimistic locking via `ETag` and `If-Match`.
- Retry flow on `412`: `GET` latest -> reapply intent -> `PATCH`.

## 8. Offline and Caching
- Cache last successful effective configuration with `etag`.
- Keep session overrides in-memory only.
- Queue patches offline and replay in order.
- Drop queued patch if schema/policy changed and request is invalid.

## 9. Migration Strategy
- Maintain server migrations forward from schema `1` for future releases.
- Run migration before merge/resolution.
- Return latest schema in API responses.

## 10. Security and Audit
- Do not store secrets in preferences.
- Emit audit events for preference changes.
- Redact sensitive preference values in logs.
