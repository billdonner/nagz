Run all tests, commit all dirty repos, and push everything.

## Instructions

1. **Check git status** of all 4 repos in parallel:
   - `~/nagz`
   - `~/nagzerver`
   - `~/nagz-web`
   - `~/nagz-ios`

2. **Run all test suites in parallel** (same as /test-all):
   - `cd ~/nagzerver && uv run pytest`
   - `cd ~/nagz-web && npx vitest run`
   - `cd ~/nagz-ios && xcodebuild test -project Nagz.xcodeproj -scheme Nagz -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'`

3. **If any tests fail, STOP.** Report failures and do not commit or push anything.

4. **If all tests pass**, for each repo that has uncommitted changes:
   - Stage all modified/new files (but skip .env, credentials, secrets)
   - Generate a concise commit message describing the changes
   - Commit with `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`
   - Push to origin

5. **Report final status table**:

| Repo | Tests | Committed | Pushed | Details |
|------|-------|-----------|--------|---------|

6. If user provides a message as argument, use it as the commit message prefix for all repos.

$ARGUMENTS
