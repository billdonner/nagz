# Nagz

Nagz is a family-oriented AI-mediated nagging/reminder app with **Apple Intelligence** integration.

## AI Architecture

Nagz uses a split AI architecture:

- **On-device** (iOS) — Apple Foundation Models for privacy-sensitive processing (excuse summarization, coaching, pattern detection). Data stays on device.
- **Server-side** (all clients) — Deterministic heuristic engine, upgradable to LLM. 7 AI endpoints for web clients and iOS fallback.
- **Siri & Shortcuts** (V2.0) — App Intents for voice control and automation ("Show my nags", "Mark my homework as done").

See `nagz/Docs/AI_ARCHITECTURE.md` for the full design.

## Documentation

See `nagz/Docs/CATALOG.md` for the full spec index, including:

- Requirements, Architecture, Preferences, Policy Matrix
- Safety & Compliance, Glossary, API Surface
- AI Behavior, AI Architecture, Incentives, Gamification
- Siri & Shortcuts, Parent/Guardian Manual

## All Projects

### Nagz — AI-mediated nagging/reminder app

| Repo | Description | Port |
|------|-------------|------|
| [nagz](https://github.com/billdonner/nagz) | **This repo** — specs, docs, orchestration hub | — |
| [nagzerver](https://github.com/billdonner/nagzerver) | Python API server (source of truth) | 9800 |
| [nagz-web](https://github.com/billdonner/nagz-web) | TypeScript/React web app | 5173 |
| [nagz-ios](https://github.com/billdonner/nagz-ios) | SwiftUI iOS app + Apple Intelligence | — |

### OBO — Flashcard learning app

| Repo | Description | Port |
|------|-------------|------|
| [obo](https://github.com/billdonner/obo) | Hub — specs, docs, orchestration | — |
| [obo-server](https://github.com/billdonner/obo-server) | Python/FastAPI deck API | 9810 |
| [obo-gen](https://github.com/billdonner/obo-gen) | Swift CLI deck generator | — |
| [obo-ios](https://github.com/billdonner/obo-ios) | SwiftUI iOS flashcard app | — |

### Alities — Trivia game platform

| Repo | Description | Port |
|------|-------------|------|
| [alities](https://github.com/billdonner/alities) | Hub — specs, docs, orchestration | — |
| [alities-engine](https://github.com/billdonner/alities-engine) | Swift trivia engine daemon | 9847 |
| [alities-studio](https://github.com/billdonner/alities-studio) | React/TypeScript game designer | 9850 |
| [alities-mobile](https://github.com/billdonner/alities-mobile) | SwiftUI iOS game player | — |
| [alities-trivwalk](https://github.com/billdonner/alities-trivwalk) | Python TrivWalk trivia game | — |

### Server Monitor — Multi-frontend server dashboard

| Repo | Description | Port |
|------|-------------|------|
| [monitor](https://github.com/billdonner/monitor) | Hub — specs, docs, orchestration | — |
| [server-monitor](https://github.com/billdonner/server-monitor) | Python web dashboard + Terminal TUI | 9860 |
| [server-monitor-ios](https://github.com/billdonner/server-monitor-ios) | SwiftUI iOS + WidgetKit companion | — |

### Standalone Tools

| Repo | Description |
|------|-------------|
| [claude-cli](https://github.com/billdonner/claude-cli) | Swift CLI for the Claude API |
| [Flyz](https://github.com/billdonner/Flyz) | Fly.io deployment configs for all servers |
