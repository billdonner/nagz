Run all 3 Nagz test suites in parallel and report results.

## Instructions

1. Run all 3 test suites **in parallel** using background tasks:
   - `cd ~/nagzerver && uv run pytest`
   - `cd ~/nagz-web && npx vitest run`
   - `cd ~/nagz-ios && xcodebuild test -project Nagz.xcodeproj -scheme Nagz -destination 'platform=iOS Simulator,name=iPhone 16 Pro Max,OS=18.5'`

2. Wait for all 3 to complete, then parse each output for:
   - Total tests run
   - Tests passed
   - Tests failed
   - Duration

3. Present results as a markdown table:

| Repo | Tests | Passed | Failed | Duration | Status |
|------|-------|--------|--------|----------|--------|

4. If any suite failed, show the first 5 failure messages below the table.

5. Update test counts in `~/nagz/CLAUDE.md` and `~/nagz-ios/CLAUDE.md` if they changed.

$ARGUMENTS
