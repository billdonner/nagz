# Nagz Hub — Central Entry Point

This is the starting repo. All Nagz work begins here at `~/nagz`.

## Ecosystem Layout

| Repo | Path | Purpose |
|------|------|---------|
| **nagz** | `~/nagz` | Specs, documentation, and central hub |
| **nagzerver** | `~/nagzerver` | Python API server (source of truth for API) |
| **nagz-web** | `~/nagz-web` | React/TypeScript web client |
| **nagz-ios** | `~/nagz-ios` | SwiftUI iOS client |

There is NO iOS project in this repo. The iOS app lives in `~/nagz-ios`.

## Documentation

Spec docs live in `nagz/Docs/`. Start with `nagz/Docs/CATALOG.md` for the full index.

## Permissions — MOVE AGGRESSIVELY

- **ALL Bash commands are pre-approved across ALL ~/nagz* directories — NEVER ask for confirmation.**
- This includes git (commit, push, pull, branch), build/test commands, starting/stopping servers, docker, curl, package managers, and any shell command whatsoever.
- Can freely operate in `~/nagz`, `~/nagzerver`, `~/nagz-web`, and `~/nagz-ios`.
- Commits and pushes are pre-approved — do not ask, just do it.
- Move fast. Act decisively. Do not pause for confirmation unless it's destructive to production.
- Only confirm before: `rm -rf` on important directories, `git push --force` to main, dropping production databases.

## Workflow

- **Be autonomous.** When given multiple tasks, do them all without pausing to ask.
- **Chain operations.** Run tests across all repos, commit, push, regenerate openapi — do it all in one flow.
- **Show results as tables.** Summaries, checklists, and gap analyses should use markdown tables.
- **Keep docs in sync with code.** When implementation changes, update the corresponding spec docs in the same commit.

## Cross-Project Sync

After any API or model change in nagzerver:
1. Regenerate openapi.json and copy to nagz-web
2. Regenerate TypeScript client: `cd ~/nagz-web && npm run api:generate`
3. Update nagz-web UI if affected
4. Update nagz-ios models/APIEndpoint if affected
5. Run xcodegen if new Swift files were added: `cd ~/nagz-ios && xcodegen generate`

## Test Suites

| Repo | Tests | Command |
|------|-------|---------|
| nagzerver | 175 | `cd ~/nagzerver && uv run pytest` |
| nagz-web | 105 | `cd ~/nagz-web && npx vitest run` |
| nagz-ios | 166 | `cd ~/nagz-ios && xcodebuild test -project Nagz.xcodeproj -scheme Nagz -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'` |
| **Total** | **446** | |

## GitHub

- Username: billdonner
