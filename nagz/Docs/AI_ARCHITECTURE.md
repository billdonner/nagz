# AI Architecture (V1.0)

## 1. Overview

Nagz uses a split AI architecture: on-device intelligence for privacy-sensitive, latency-critical features and server-side AI for coordination, policy enforcement, and cross-user analysis. Both paths conform to a shared AI service interface so behavior is consistent regardless of execution location.

## 2. Design Principles

- **Privacy first.** Excuse text, behavioral patterns, and coaching stay on-device when possible. The server sees structured events (category, status, timestamps), not raw AI outputs.
- **Server is source of truth.** Policy enforcement, safety controls, and the canonical audit trail are server-authoritative. Clients are untrusted for policy decisions.
- **Graceful degradation.** On-device AI is an enhancement. All AI features must work via server fallback for web clients and older iOS devices.
- **Heuristics first, LLM later.** Ship v1 with rule-based heuristics. The interface is designed so a real model (OpenAI, Claude, Apple Foundation Models) can be swapped in without changing the API contract.

## 3. AI Service Interface

All AI features are defined as operations on this shared interface. Each operation has an on-device implementation (iOS 18.1+) and a server implementation (web + fallback).

| Operation | Input | Output |
|-----------|-------|--------|
| `summarizeExcuse` | excuse text, nag context | structured summary + category |
| `selectTone` | event history, preferences | tone mode (neutral / supportive / firm) |
| `detectPatterns` | event history for user + family | list of behavioral insights |
| `generateCoaching` | current nag, event history | motivational tip text |
| `evaluatePushBack` | nag, event history, policy | push-back decision + message |
| `generateDigest` | week's events for family | weekly summary text |
| `predictCompletion` | nag, event history | likelihood + suggested reminder time |

## 4. Execution Model

```
┌──────────────────────────────────────────────────┐
│              AI Service Interface                 │
│  (shared operation signatures, same results)      │
└────────┬─────────────────────────┬────────────────┘
         │                         │
  ┌──────┴───────┐         ┌──────┴──────┐
  │  On-Device   │         │   Server    │
  │  (iOS 18.1+) │         │  (all clients) │
  │              │         │              │
  │  Foundation  │         │  Heuristic   │
  │  Models      │         │  engine      │
  │  + SwiftData │         │  (upgradable │
  │  event cache │         │   to LLM)   │
  └──────────────┘         └──────────────┘
```

### 4.1 iOS path (on-device preferred)

1. Device maintains a local event cache in SwiftData, synced from server via `GET /events?since=<timestamp>`.
2. AI operations run against the local cache using Apple Foundation Models framework.
3. Results are used locally for UX (tone, coaching, pattern alerts).
4. Structured outcomes (excuse category, completion status) are sent to server as canonical events.
5. Falls back to server AI endpoints when:
   - Device is pre-iOS 18.1
   - Foundation Models are unavailable (unsupported hardware)
   - Device is offline and cache is stale beyond threshold

### 4.2 Web path (server always)

1. Web client sends raw input to server AI endpoints.
2. Server runs the operation against the canonical event store.
3. Server returns the result; web client displays it.
4. No local model, no local event cache beyond session state.

### 4.3 Fallback path (iOS without Apple Intelligence)

Same as web path. The iOS `AIService` protocol detects availability at runtime:

```swift
protocol AIService {
    func summarizeExcuse(_ text: String, context: NagContext) async -> ExcuseSummary
    func selectTone(for nag: Nag, history: [NagEvent]) async -> ToneMode
    func detectPatterns(userId: String, events: [NagEvent]) async -> [Insight]
    func generateCoaching(for nag: Nag, history: [NagEvent]) async -> String
    func evaluatePushBack(for nag: Nag, history: [NagEvent], policy: Policy) async -> PushBackDecision
}

// Two implementations:
// - OnDeviceAIService (Foundation Models + SwiftData)
// - ServerAIService (calls /api/v1/ai/* endpoints)
```

## 5. Server AI Endpoints

New endpoints for web clients and iOS fallback:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/ai/summarize-excuse` | POST | Summarize excuse text, assign category |
| `/api/v1/ai/select-tone` | POST | Choose tone based on nag + event history |
| `/api/v1/ai/coaching` | POST | Generate motivational tip for a nag |
| `/api/v1/ai/patterns` | GET | Detect behavioral patterns for a user |
| `/api/v1/ai/digest` | GET | Generate weekly family digest |
| `/api/v1/ai/predict-completion` | GET | Predict completion likelihood + optimal reminder time |
| `/api/v1/ai/push-back` | POST | Evaluate and generate push-back message |

All endpoints require authentication. Policy and consent checks apply (see `AI_BEHAVIOR.md` section 3).

## 6. Local Event Cache (iOS)

### 6.1 What is cached

| Entity | Sync direction | Retention |
|--------|---------------|-----------|
| `nag_events` for user's nags | Server → device | 90 days rolling |
| `ai_mediation_events` for user | Server → device | 90 days rolling |
| `gamification_events` for user | Server → device | 90 days rolling |
| `nags` for user's families | Server → device | Active + 30 days completed |
| User preferences | Server → device | Current snapshot |

### 6.2 Sync protocol

1. Device stores a `last_sync_at` timestamp per entity type.
2. On app foreground / every 5 minutes while active, device calls `GET /api/v1/events?since=<last_sync_at>&types=nag_event,mediation,gamification`.
3. Server returns new events in chronological order.
4. Device appends to SwiftData and updates `last_sync_at`.
5. If `last_sync_at` is older than 24 hours, device marks cache as stale and falls back to server AI until re-synced.

### 6.3 Storage

- SwiftData with on-device encryption (iOS Data Protection).
- Estimated size: ~50KB per user per month of active use.
- Cache is deleted on logout and account deletion.

## 7. Heuristic Engine (Server v1)

The server AI starts as deterministic heuristics, not an LLM. This ships fast and provides consistent, testable behavior.

### 7.1 Tone selection

```
if miss_count_last_7_days >= 3:
    return "firm"
elif miss_count_last_7_days == 0 and streak >= 3:
    return "supportive"
else:
    return "neutral"
```

### 7.2 Excuse summarization

```
summary = text[:200]
category = classify_by_keywords(text)
    "forgot" / "remember" → forgot
    "busy" / "time" / "schedule" → time_conflict
    "don't understand" / "confused" → unclear_instructions
    "don't have" / "need" → lacking_resources
    "no" / "won't" / "refuse" → refused
    else → other
```

### 7.3 Pattern detection

```sql
-- Day-of-week miss pattern
SELECT day_of_week, COUNT(*) as misses
FROM nag_events
WHERE event_type = 'missed' AND user_id = ?
GROUP BY day_of_week
HAVING misses >= 3
```

### 7.4 Coaching tips

Static tip pool keyed by category and miss/completion ratio:

| Scenario | Example tip |
|----------|------------|
| First miss | "Everyone forgets sometimes. Try setting a phone alarm for next time." |
| Streak of 3+ | "Great streak! Keep it up — you're building a solid habit." |
| Repeated time_conflict | "Looks like timing is tricky. Would an earlier reminder help?" |
| Homework category, high completion | "You're crushing homework this week!" |

### 7.5 Push-back evaluation

```
if pushback_mode == "off": return no_pushback
if pushback_count_this_window >= max_pushback: return no_pushback
if in_quiet_hours(): return no_pushback
if minutes_since_last_pushback < cooldown: return no_pushback
return generate_pushback(tone, nag)
```

## 8. LLM Upgrade Path

When ready to upgrade from heuristics to a real model:

### 8.1 Server-side

- Add an LLM provider (OpenAI, Claude API, or self-hosted) behind the same endpoint interface.
- The heuristic engine becomes the fallback when the LLM provider is unavailable.
- Prompt templates are stored in config, not code.
- All LLM calls are logged to the audit trail with `actor: ai_mediator`.

### 8.2 On-device (iOS)

- Apple Foundation Models framework provides the on-device LLM.
- Prompt construction uses the same templates as the server.
- Model availability is checked at runtime via `FoundationModels.isAvailable`.
- Results are validated against policy bounds before use (e.g., push-back limits).

## 9. Privacy Implications

| Data | On-device | Server |
|------|-----------|--------|
| Raw excuse text | Stays on device (processed locally) | Only receives category + summary |
| Behavioral patterns | Computed locally | Computed from canonical events |
| Coaching text | Generated locally | Generated from event history |
| Tone decisions | Made locally | Made from event history |
| Push-back messages | Generated locally | Generated from event history |
| Audit trail | Local cache (encrypted) | Canonical copy (encrypted at rest) |

This architecture supports a strong App Store privacy story: "AI coaching and excuse processing happen on your device. Only structured data is shared with the server."

## 10. Consent Requirements

All AI features require the `ai_mediation` consent (see `SAFETY_AND_COMPLIANCE.md`). Without consent:

- Server AI endpoints return `403 AUTHZ_DENIED`.
- On-device AI features are disabled in the UI.
- The app falls back to non-AI behavior (raw excuse text stored as-is, no tone adjustment, no coaching).

## 11. Testing Strategy

| Layer | Approach |
|-------|----------|
| Heuristic engine | Unit tests with deterministic inputs/outputs |
| Server AI endpoints | Integration tests via pytest |
| On-device AI | XCTest with mock event history, skip on CI (no Foundation Models) |
| Fallback behavior | Test iOS AIService with `ServerAIService` forced |
| Consent gating | Test 403 responses without consent |

## 12. Implementation Order

1. **Server heuristic engine** — implement tone selection, excuse categorization, pattern detection as pure functions.
2. **Server AI endpoints** — expose heuristics via `/api/v1/ai/*` endpoints.
3. **Web integration** — call server AI endpoints from nagz-web components.
4. **iOS AIService protocol** — define the shared interface with `ServerAIService` calling the endpoints.
5. **iOS event cache** — SwiftData sync for local event history.
6. **iOS OnDeviceAIService** — Foundation Models implementation behind `@available(iOS 18.1, *)`.
7. **LLM upgrade** — swap heuristics for real model when ready.
