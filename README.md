# nagz

Nagz is a family-oriented AI-mediated nagging/reminder app with **Apple Intelligence** integration.

## AI Architecture

Nagz uses a split AI architecture:

- **On-device** (iOS) — Apple Foundation Models for privacy-sensitive processing (excuse summarization, coaching, pattern detection). Data stays on device.
- **Server-side** (all clients) — Deterministic heuristic engine, upgradable to LLM. 7 AI endpoints for web clients and iOS fallback.
- **Siri & Shortcuts** (V2.0) — App Intents for voice control and automation ("Show my nags", "Mark my homework as done").

See `nagz/Docs/AI_ARCHITECTURE.md` for the full design.

## Documentation
- `nagz/Docs/REQUIREMENTS.md`
- `nagz/Docs/ARCHITECTURE.md`
- `nagz/Docs/PREFERENCES.md`
- `nagz/Docs/POLICY_MATRIX.md`
- `nagz/Docs/SAFETY_AND_COMPLIANCE.md`
- `nagz/Docs/GLOSSARY.md`
- `nagz/Docs/API_SURFACE.md`
- `nagz/Docs/AI_BEHAVIOR.md`
- `nagz/Docs/AI_ARCHITECTURE.md`
- `nagz/Docs/INCENTIVES.md`
- `nagz/Docs/GAMIFICATION.md`
- `nagz/Docs/SIRI_SHORTCUTS.md`
- `nagz/Docs/PARENT_GUARDIAN_USER_MANUAL.md`
- `nagz/Docs/SPEC_BASELINE_CHANGELOG.md`
- `nagz/Docs/web-samples/parent-guardian-manual.html`
- `nagz/Docs/web-samples/guardian-portal.html`

## Ecosystem

| Repo | Description | Tests |
|------|-------------|-------|
| [nagzerver](https://github.com/billdonner/nagzerver) | Python API server (source of truth) | 175 |
| [nagz-ios](https://github.com/billdonner/nagz-ios) | SwiftUI iOS client + Apple Intelligence | 166 |
| [nagz-web](https://github.com/billdonner/nagz-web) | React/TypeScript web client | 105 |

Docs index: `nagz/Docs/CATALOG.md` (with `nagz/Docs/README.md` as shim).
