# Nagz Code Review Plan

> Use this document as a prompt for AI-assisted code review or as a checklist for human reviewers. It covers all three repos in the Nagz ecosystem.

---

## Scope

| Repo | Path | Stack | Files | Lines |
|------|------|-------|-------|-------|
| nagzerver | ~/nagzerver | Python 3.12, FastAPI, SQLAlchemy | 77 | ~7,200 |
| nagz-web | ~/nagz-web | React 18, TypeScript, Vite | 174 | ~8,800 |
| nagz-ios | ~/nagz-ios | SwiftUI, Swift 6, iOS 17+ | 95 | ~7,100 |

---

## 1. Edge Cases

Review each repo for unhandled or poorly handled edge cases.

### Data Boundaries

- [ ] Empty collections: What happens when a family has zero nags, zero members, or zero events? Do list views show a meaningful empty state?
- [ ] Single-item collections: Does pagination logic work correctly with exactly 1 result? Does the "has more" flag behave?
- [ ] Maximum lengths: Are there server-enforced limits on nag title, description, excuse text, display name? Are those limits mirrored in client-side validation?
- [ ] Unicode and emoji: Can a nag title or excuse contain emoji, RTL text, or special characters without breaking layout or encoding?
- [ ] Null vs missing fields: Are optional API fields handled consistently? Does the iOS app crash if an expected optional is nil? Does the web app render `undefined` as visible text?

### Temporal Edge Cases

- [ ] Past due dates: Creating a nag with a due date in the past — is this allowed, warned, or blocked? Is behavior consistent across all three clients?
- [ ] Timezone handling: Are due dates stored as UTC on the server? Do clients convert correctly? What happens near midnight when local day rolls over?
- [ ] Daily nag cap: The cap is 8 per creator per family per local day. What defines "local day"? Is it the creator's timezone or UTC? What happens at the boundary?
- [ ] Token expiry during long sessions: If the user leaves the app open for hours, does the 401 → refresh → retry flow work without data loss?
- [ ] Concurrent edits: Two guardians editing the same nag simultaneously — who wins? Is there last-write-wins, conflict detection, or silent overwrite?

### Authentication & Authorization

- [ ] Expired refresh tokens: Does the app gracefully redirect to login, or does it spin in a retry loop?
- [ ] Role escalation: Can a child or participant craft a request that bypasses role checks? Are role checks server-enforced, not just client-gated?
- [ ] Deleted accounts: If a user is soft-deleted, can they still authenticate? Can other users still see their nags or family membership?
- [ ] Family switching: If a user leaves a family and joins another, is all old family data properly inaccessible?

### Network & Offline

- [ ] No network on launch: Does the iOS app show cached data from GRDB, or a blank screen?
- [ ] Network drop mid-mutation: Creating a nag, completing a nag, or submitting an excuse when the network drops — is the action retried, lost, or duplicated?
- [ ] Server returning unexpected HTTP codes (502, 503, rate-limited 429): Are all error paths covered, or do some fall through to a generic crash?
- [ ] Slow responses: Is there a timeout? Does the UI show a loading state, or does it appear frozen?

---

## 2. Redundant & Dead Code

Look for code that can be removed or consolidated.

### Cross-Repo Duplication

- [ ] Legal documents: Privacy policy and TOS exist as both HTML in `nagzerver/website/` and as Python text in `nagzerver/src/nagz/server/routers/legal.py`. Are they in sync? Should one be the source of truth?
- [ ] Model definitions: Nag, Family, User models are defined independently in Python (SQLAlchemy), TypeScript (generated client), and Swift (manual Codable structs). Are there fields present in one but missing in another?
- [ ] Enum values: NagCategory, NagStatus, FamilyRole, DoneDefinition — are all enum cases identical across all three codebases? A missing case in one client means silent decoding failures.
- [ ] Error codes: Does the web app handle every error code the server can return? Does the iOS app?

### Within Each Repo

- [ ] Unused imports: Scan for imported modules, types, or functions that are never referenced.
- [ ] Unreachable code: Look for `return` statements followed by more code, dead branches in `if/else` or `switch/case`, and catch blocks that can never trigger.
- [ ] Duplicate logic: Are there utility functions that do the same thing? For example, date formatting, error message extraction, or API URL construction duplicated across files.
- [ ] Commented-out code: Remove it. Version control is the archive.
- [ ] Unused API endpoints: Are there server endpoints that no client calls? Are there client API methods that are defined but never invoked?
- [ ] Test helpers: Are there test utilities or fixtures that are no longer used by any test?

---

## 3. Consistency & Correctness

### API Contract

- [ ] Snake case vs camel case: The server uses snake_case. iOS uses `convertFromSnakeCase`. The web client uses a generated TypeScript client. Are there any manual JSON keys that don't follow the convention?
- [ ] HTTP methods: Are all mutations POST/PUT/PATCH/DELETE? Are all reads GET? Are there any GETs with side effects?
- [ ] Response envelopes: Does every endpoint return a consistent shape? Are error responses always `{ "detail": "..." }` or `{ "error_code": "...", "message": "..." }`?
- [ ] Pagination: Do all list endpoints support `offset` and `limit`? Do all clients use them? Are there list endpoints that return unbounded results?

### Error Handling

- [ ] Silent failures: Are there `try/except: pass` blocks (Python), empty `.catch(() => {})` (TypeScript), or `try? method()` with discarded results (Swift) that swallow errors?
- [ ] User-facing error messages: Are error messages helpful and non-technical? Does the user ever see a raw stack trace, JSON blob, or "undefined"?
- [ ] Retry logic: Where retries exist, is there a maximum count and backoff? Can retries cause duplicate side effects (double nag creation, double point award)?

### Security

- [ ] Input validation: Are all user inputs validated on the server, not just the client? Pay special attention to: nag titles, excuse text, display names, family names.
- [ ] SQL injection: Are all database queries parameterized? Any raw string interpolation in SQLAlchemy queries?
- [ ] XSS: Does the web app render any user-provided text with `dangerouslySetInnerHTML` or equivalent?
- [ ] Rate limiting: Are rate limits enforced on all mutation endpoints? Can they be bypassed by rotating tokens?
- [ ] Secrets in code: Are there hardcoded API keys, passwords, or tokens anywhere in the codebase? Check for `.env` files committed to git.

---

## 4. Architecture & Maintainability

### Separation of Concerns

- [ ] Views doing business logic: Do any SwiftUI views or React components contain API calls, data transformation, or business rules that should be in a ViewModel/hook/service?
- [ ] Fat controllers: Do any FastAPI route handlers contain business logic that should be in a service layer?
- [ ] God objects: Are there classes or modules that do too many things? Look for files over 300 lines.

### Swift 6 Concurrency (iOS Only)

- [ ] Actor isolation: Are all `@MainActor` annotations correct? Is any UI code running off the main actor?
- [ ] Sendable conformance: Are all types passed across actor boundaries `Sendable`? Look for compiler warnings.
- [ ] Data races: Are there any shared mutable state accessed without synchronization? Check static properties and singletons.
- [ ] AuthManager usage from intents: AuthManager is `@MainActor`. App Intents run on arbitrary threads. Is `IntentServiceContainer` properly avoiding AuthManager?

### TypeScript (Web Only)

- [ ] Any types: Are there `any` casts that bypass type safety? Every `as any` is a potential runtime bug.
- [ ] Missing error boundaries: Can a component crash take down the entire app, or are errors caught per-route?
- [ ] Stale closures: In useEffect or event handlers, are there references to stale state that should use refs or dependency arrays?

### Python (Server Only)

- [ ] Async correctness: Are there blocking calls (`time.sleep`, synchronous I/O) inside async route handlers that would block the event loop?
- [ ] Database session management: Are sessions properly closed/committed/rolled back in all code paths, including error paths?
- [ ] Missing migrations: Are there model changes that don't have corresponding Alembic migrations?

---

## 5. Test Quality

- [ ] Test coverage gaps: Are there endpoints, views, or services with zero tests? Prioritize: auth flows, payment/incentive logic, role-based access, and data mutations.
- [ ] Brittle tests: Do tests depend on execution order, hardcoded timestamps, or external services? Look for flaky tests that pass sometimes.
- [ ] Missing negative tests: Are there tests for invalid input, unauthorized access, and error responses — not just happy paths?
- [ ] Test data leakage: Do tests clean up after themselves? Can one test's state leak into another? (Especially relevant for iOS tests using UserDefaults.)
- [ ] Mocking vs integration: Are external services (APNS, Redis, database) properly mocked in unit tests? Are there integration tests that verify real connections?

---

## 6. Performance

- [ ] N+1 queries: Does the server make one query per item in a list? Look at endpoints that return nags with related data (family members, events, excuses).
- [ ] Unbounded fetches: Are there API calls that fetch all records without pagination? This will break as data grows.
- [ ] Memory leaks: In the iOS app, are there strong reference cycles in closures, delegates, or observation? In the web app, are there effects that don't clean up subscriptions?
- [ ] Cache invalidation: After creating, updating, or deleting a nag, is the iOS GRDB cache and the APIClient in-memory cache properly invalidated?
- [ ] Large payloads: Are there API responses that could grow unbounded (e.g., all events for a family ever)?

---

## How to Use This Document

### As an AI Prompt

```
You are reviewing the Nagz codebase — a family task-reminder app with three repos:
- nagzerver (Python FastAPI API server)
- nagz-web (React TypeScript web client)
- nagz-ios (SwiftUI iOS app, Swift 6)

Using the checklist in CODE_REVIEW_PLAN.md, systematically examine each
category. For each issue found, report:
1. File path and line number
2. Category (edge case, redundancy, consistency, security, performance, test gap)
3. Severity (critical, major, minor, nit)
4. Recommended fix

Start with security and edge cases. Prioritize issues that could cause data
loss, crashes, or unauthorized access.
```

### As a Human Checklist

Work through the sections in order. Check off items as you verify them. For each issue found, file it with severity and a suggested fix. Focus on:

1. **Critical first**: Security holes, data loss risks, crashes
2. **Then correctness**: Wrong behavior, inconsistent cross-repo contracts
3. **Then quality**: Dead code, missing tests, performance
4. **Last, nits**: Style, naming, minor improvements

---

## Review Schedule

| Phase | Focus | Estimated Effort |
|-------|-------|-----------------|
| 1 | Security & auth edge cases | 2-3 hours |
| 2 | API contract consistency across repos | 2-3 hours |
| 3 | Redundant code & dead code cleanup | 1-2 hours |
| 4 | Test coverage gaps | 2-3 hours |
| 5 | Performance & scalability | 1-2 hours |
| 6 | Swift 6 concurrency audit (iOS) | 1-2 hours |

---

*Generated 2026-02-18. Covers nagzerver, nagz-web, and nagz-ios.*
