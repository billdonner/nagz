# Nagz Preferences and Sync API (V1.0)

## 1. Goals
- Persist user-level settings across devices.
- Support role-aware defaults for guardians vs children.
- Configure AI mediation and gamification behavior safely.
- Allow safe partial updates.
- Keep policy controls server-authoritative.
- Keep schema evolvable with explicit migrations.

## 2. Resolution Order
Effective configuration resolves from lowest to highest precedence:
1. App defaults
2. Family policy defaults
3. User preferences
4. Session overrides (local only, not persisted)

## 3. Canonical Server Shape
The server stores one canonical preference document per `user_id` and `family_id`.

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
      "priority_channels": ["push", "sms"]
    },
    "nag_defaults": {
      "default_strategy_template": "friendly_reminder",
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
    "interaction_controls": {
      "allow_snooze": true,
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
- Policy-only examples:
  - role/relationship permissions
  - guardian-only report access
  - global hard-stop limits
  - `daily_nag_cap`
  - `max_snooze_minutes`
  - gamification scoring multiplier (`points_multiplier`)
  - AI push-back attempt/cooldown bounds

## 5. Key Constraints
- `schema_version`: `2` (current).
- `role`: `guardian`, `participant`, or `child` (derived from family membership; read-only).
- `default_strategy_template`: V1.0 supports only `friendly_reminder`.
- Legacy alias `default_strategy` may be accepted for backward compatibility, but responses use `default_strategy_template`.
- `priority_channels`: subset of `["push", "sms"]`.
- Delivery resolution for `priority_channels`: evaluate in order and skip channels that are disabled or currently non-compliant (for example, SMS unsubscribed).
- `ai_mediation.tone`: `neutral`, `supportive`, or `firm`.
- `ai_mediation.pushback_mode`: `off` or `bounded`.
- `interaction_controls.allow_snooze`: patchable boolean, constrained by policy.
- `interaction_controls.mute_until`: RFC3339 timestamp or `null`.
- `points_multiplier`: policy-controlled decimal `0.1..3.0` and read-only in user preference patch APIs.

## 5b. AI Opt-Out Preferences
Two top-level boolean keys in `prefs_json` control AI features independently:

| Key | Default | Description |
|-----|---------|-------------|
| `ai_server_enabled` | `true` | Enables server-side AI heuristic endpoints (summarize, tone, coaching, etc.) |
| `ai_on_device_enabled` | `true` | Enables on-device AI features (iOS Apple Intelligence) |

Both sit under the existing `ai_mediation` consent gate. Guardians can set these for children via `PATCH /preferences/{userId}`. The server AI endpoints check both the consent and the `ai_server_enabled` preference — if either is missing, 403 `AUTHZ_DENIED`.

## 6. Cross-Spec Linkage
- AI fields map to `AI_BEHAVIOR.md`.
- Incentive preference toggles map to `INCENTIVES.md`.
- Gamification preference toggles map to `GAMIFICATION.md`.
- Hard-stop and consent constraints map to `SAFETY_AND_COMPLIANCE.md`.

## 7. Minimal Sync API
Base path: `/api/v1`

### 7.1 Get Effective Configuration
`GET /api/v1/preferences?family_id={familyId}`

Response headers:
- `ETag: "pref_v1_usr_123_18"`

Response body includes effective user prefs and read-only policy:

```json
{
  "schema_version": 2,
  "etag": "pref_v2_usr_123_18",
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
        "default_strategy_template": "friendly_reminder",
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
      "interaction_controls": {
        "allow_snooze": true,
        "mute_until": null
      }
    },
    "policy": {
      "report_visibility": "guardian_only",
      "daily_nag_cap": 8,
      "max_snooze_minutes": 30,
      "child_can_nag_guardian": false,
      "gamification_points_multiplier": 1.0,
      "ai_pushback_max_attempts_per_window": 2,
      "ai_pushback_cooldown_minutes": 60
    }
  }
}
```

### 7.2 Patch User Preferences
`PATCH /api/v1/preferences?family_id={familyId}`

V1.0 implementation uses a simplified patch format:

```json
{
  "prefs_json": {
    "tone": "firm",
    "gamification_enabled": true,
    "notification_frequency": "always"
  }
}
```

The `prefs_json` dict is shallow-merged with existing preferences. Keys present in the request overwrite existing values; keys not present are unchanged.

Responses:
- `200 OK` with updated preferences, `schema_version`, and `etag`.
- `422 Unprocessable Entity` for invalid values.
- `403 Forbidden` for authorization or disallowed policy scope.
- `429 Too Many Requests` for rate limiting.

Future versions may add `set`/`unset` semantics and `If-Match`/`ETag` optimistic concurrency (412 Precondition Failed).

### 7.3 Error Envelope
All non-2xx API responses use this envelope:

```json
{
  "error": {
    "code": "POLICY_FORBIDDEN",
    "message": "Field is read-only for this role.",
    "request_id": "req_abc123",
    "details": {
      "field": "prefs.gamification.points_multiplier"
    }
  }
}
```

Canonical error codes are authoritative in `ARCHITECTURE.md` section 9.

## 8. Merge and Conflict Behavior
- Partial patch updates only specified keys.
- Unspecified keys are unchanged.
- `unset` removes user override and falls back to lower precedence.
- Optimistic locking via `ETag` and `If-Match`.
- Retry flow on `412`: `GET` latest -> reapply intent -> `PATCH`.

## 9. Offline and Caching
- Cache last successful effective configuration with `etag`.
- Keep session overrides in-memory only.
- Queue patches offline and replay in order.
- Drop queued patch if schema/policy changed and request is invalid.

## 10. Snooze Semantics (Normative)
- `snooze` defers next eligible reminder delivery only; it does not mark completion.
- Snooze does not cancel the nag.
- Default `max_snooze_minutes` is `30`, configurable guardian policy range `5..120`.
- Snooze cannot extend beyond `max_snooze_minutes` policy bound.
- A snooze during overdue phases delays the next reminder but does not reset miss history.
- If quiet hours begin during a snooze, delivery waits until both snooze and quiet-hour constraints are cleared.

## 11. Migration Strategy
- Maintain server migrations forward from schema `1` for future releases.
- Run migration before merge/resolution.
- Return latest schema in API responses.

## 12. Security and Audit
- Do not store secrets in preferences.
- Emit audit events for preference changes.
- Redact sensitive preference values in logs.
