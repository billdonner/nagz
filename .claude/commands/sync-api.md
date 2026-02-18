Regenerate the API client after nagzerver changes and check for iOS drift.

## Instructions

1. **Regenerate OpenAPI spec** from nagzerver:
   - `cd ~/nagzerver && uv run python -m nagz.server.openapi_export`
   - If that command doesn't exist, check for alternative generation methods in nagzerver/CLAUDE.md

2. **Copy to nagz-web**:
   - `cp ~/nagzerver/openapi.json ~/nagz-web/openapi.json`

3. **Regenerate TypeScript client**:
   - `cd ~/nagz-web && npm run api:generate`

4. **Check for iOS model drift**:
   - Compare the endpoint paths and request/response models in `openapi.json` against:
     - `~/nagz-ios/Nagz/Services/APIEndpoint.swift`
     - `~/nagz-ios/Nagz/Models/*.swift`
   - Report any mismatches as a table:

| Endpoint/Model | OpenAPI | iOS | Status |
|----------------|---------|-----|--------|

5. **Run nagz-web tests** to verify the regenerated client:
   - `cd ~/nagz-web && npx vitest run`

6. If there are iOS mismatches, list the specific changes needed but do NOT auto-apply them (the user will decide).

7. Commit and push any changes in nagzerver and nagz-web if files changed.

$ARGUMENTS
