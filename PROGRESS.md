# Claude Code Progress Log

Auto-updated by Claude Code sessions working across repos.

---

## 2026-02-19 — Alities: TestFlight & Server Deployment

### Engine (alities-engine) — commit `93ce3ad`
- Added configurable `--host` bind address to ControlServer
- Added `GET /health` endpoint for container health probes
- Added `GET /gamedata` endpoint (full GameDataOutput from SQLite)
- Added Bearer token auth on destructive POST endpoints (`CONTROL_API_KEY`)
- Added CORS headers + OPTIONS preflight on all responses
- Created `Dockerfile` (multi-stage swift:5.9-jammy → ubuntu:22.04)
- Created `docker-compose.yml` (engine + postgres:16)
- Created `fly.toml` (Fly.io config with auto-TLS, persistent volumes)
- Tests: 101/101 passed

### Mobile (alities-mobile) — commit `5c59a3c`
- Created `project.yml` (xcodegen spec for iOS 17 app target)
- Added conditional `#if APP_TARGET @main` for dual SPM/Xcode build paths
- Added debug/release server URL switching in GameService
- Added `fetchGameData()` method for `/gamedata` endpoint
- Created `Assets.xcassets` with 1024x1024 placeholder app icon
- Created `PrivacyInfo.xcprivacy` (iOS 17+ privacy manifest)
- Added `*.xcodeproj` to `.gitignore`
- Tests: 19/19 passed
- Xcode build: **SUCCEEDED** (iPhone 16 Pro Max Simulator)

### Remaining Manual Steps
- Phase C: Deploy engine to Fly.io (`flyctl auth login` → `flyctl deploy`)
- Phase D: Xcode Archive → TestFlight upload
- Phase E: App Store metadata (screenshots, privacy policy, description)
