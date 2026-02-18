Compare specs in nagz/Docs/ against actual implementations across all 3 repos.

## Instructions

1. **Read the spec catalog**: `~/nagz/Docs/CATALOG.md` to get the list of all spec documents.

2. **For each major spec area**, check implementation status across repos:

   **API Endpoints**: Compare endpoints listed in specs against:
   - `~/nagzerver/src/nagz/server/routers/*.py` (server implementation)
   - `~/nagz-ios/Nagz/Services/APIEndpoint.swift` (iOS client)
   - `~/nagz-web/src/` (web client API calls)

   **Models**: Compare model definitions in specs against:
   - `~/nagzerver/src/nagz/models/*.py`
   - `~/nagz-ios/Nagz/Models/*.swift`
   - `~/nagz-web/src/` (TypeScript types)

   **Features**: Check if spec'd features are implemented (look for relevant views, components, routes).

3. **Output a gap analysis table** for each spec area:

| Feature / Endpoint | Spec | Server | iOS | Web | Notes |
|--------------------|------|--------|-----|-----|-------|

Use these status markers:
- `done` — fully implemented
- `partial` — partially implemented
- `missing` — not implemented
- `n/a` — not applicable to this repo

4. **Summary**: At the end, provide a prioritized list of the top gaps to address.

$ARGUMENTS
