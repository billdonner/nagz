# App Store Submission Plan

> Step-by-step checklist to get Nagz from TestFlight to the App Store.

---

## Phase 1: Pre-Submission Requirements

### 1.1 Apple Developer Account

| Item | Status | Notes |
|------|--------|-------|
| Apple Developer Program enrollment | TODO | $99/year, requires DUNS if organization |
| App ID registered in developer portal | TODO | com.nagz.app |
| Push notification certificates (production) | TODO | APNs production certificate or key |
| Provisioning profiles (App Store distribution) | TODO | Automatic signing recommended |

### 1.2 App Store Connect Setup

| Item | Status | Notes |
|------|--------|-------|
| Create app record in App Store Connect | TODO | Bundle ID: com.nagz.app |
| Set primary language | TODO | English (U.S.) |
| Set primary category | TODO | Lifestyle |
| Set secondary category | TODO | Productivity |
| Set pricing | TODO | Free |
| Set availability | TODO | United States only (V1.0) |

### 1.3 Production Server

| Item | Status | Notes |
|------|--------|-------|
| Production API server deployed | DONE | bd-nagzerver.fly.dev (Fly.io) |
| SSL/TLS certificate | DONE | Fly.io auto-managed TLS |
| Update AppEnvironment.swift production URL | TODO | Replace placeholder with production domain |
| APNs production push configured | DONE | JWT-based ES256, server sends via httpx HTTP/2 |
| SMS provider configured (Twilio/etc.) | DEFERRED | Not in V1.3 — SMS bridge deferred |
| Database backups configured | TODO | Automated daily backups |
| Monitoring/alerting (Sentry, DataDog) | TODO | Error tracking, uptime |
| Rate limiting verified | DONE | 120 read/min, 60 write/min per user |
| Load testing completed | TODO | Simulate 1000+ concurrent users |
| WebSocket service running | DONE | Real-time events for nag/connection updates |
| Notification scheduler running | DONE | Background loop with escalation phases + repeat notifications |

---

## Phase 2: App Store Metadata

### 2.1 App Information

| Field | Content |
|-------|---------|
| **App Name** | Nagz |
| **Subtitle** | AI-Powered Family Reminders |
| **Promotional Text** | Talk to your tasks. Create, complete, and manage family reminders by chatting with on-device AI. No data leaves your phone. |

### 2.2 Description (4000 char max)

```
Nagz helps families stay organized with AI-powered conversational reminders.

TALK TO YOUR TASKS
Just type what you need: "Remind me to call the dentist tomorrow" or "What's overdue?" The AI chat creates, completes, reschedules, and manages your nags — all conversationally. Choose from 6 AI personalities: a motivational coach, Gordon Ramsay, Mr. Rogers, and more.

KEY FEATURES
• AI Chat — create, complete, and manage tasks by chatting naturally
• Per-nag conversations — talk to any specific task for context-aware help
• Connections — nag friends and family outside your household
• Create reminders with categories, due dates, and recurrence
• Track completion with points, streaks, and family leaderboards
• Escalating reminders that adapt over time
• 7 Siri Shortcuts — "Remind me in Nagz" for quick self-reminders
• Role-based access: guardians manage, participants and children complete
• Safety tools: block, report, and mute family members
• Push notifications with escalation phases and repeat reminders
• Self-reminders separated into "My Reminders" for easy tracking

ON-DEVICE AI — PRIVATE BY DESIGN
All AI features run entirely on your iPhone using Apple Foundation Models:
• Conversational chat with 6 tools (list, create, complete, reschedule, status, excuse)
• Excuse summarization, tone selection, coaching tips, weekly digests
• Push-back reminders, task list summaries, gamification nudges
• No text ever leaves your device — only structured data syncs to the server
• Works offline with heuristic fallbacks when AI is unavailable

BUILT FOR FAMILIES
• Guardian controls for policies, consents, and incentive rules
• COPPA-compliant child accounts with guardian consent
• Quiet hours to silence reminders at bedtime
• Notification preferences: control who gets notified and when
• People tab with at-a-glance overdue digest per connection

PRIVACY FIRST
• No ads, no tracking, no data selling
• All AI processing on-device — zero external AI services
• Minimal data collection: just email and family task data
• Account deletion available anytime

Start your family's productivity journey with Nagz.
```

### 2.3 Keywords (100 char max)

```
family,reminders,chores,tasks,kids,parenting,nag,AI,chat,streaks
```

### 2.4 Screenshots (Required)

| Device | Size | Quantity |
|--------|------|----------|
| iPhone 6.9" (16 Pro Max) | 1320 x 2868 | 4-8 screenshots |
| iPhone 6.7" (15 Plus) | 1290 x 2796 | 4-8 screenshots |
| iPad 13" (optional) | 2064 x 2752 | 0-8 screenshots |

**Recommended screenshots:**
1. AI Chat — conversational task creation (hero shot)
2. AI Chat — listing overdue nags with personality
3. Nag list with "My Reminders" section and overdue indicators
4. Per-nag chat with context-aware AI response
5. People tab with overdue digest badges per connection
6. Nag detail with AI Insights and escalation phase
7. Points & streaks gamification view
8. Siri "Remind me in Nagz" quick add

### 2.5 App Icon

| Size | Requirement |
|------|-------------|
| 1024 x 1024 px | Single-layer, no transparency, no rounded corners |
| Format | PNG |

### 2.6 App Preview Video (Optional)

- 15-30 seconds
- Show: AI chat creating nag → notification → complete via chat → earn points → Siri "Remind me"
- Highlight: conversational flow, personality selection, overdue digest
- No device bezels required

---

## Phase 3: Legal & Compliance

### 3.1 Privacy Policy

| Item | Status | Notes |
|------|--------|-------|
| Privacy policy URL | TODO | Must be publicly accessible (server endpoint `GET /legal/privacy-policy` exists; needs public hosting URL) |
| Data types collected documented | TODO | Email, DOB, device tokens, task data |
| Data sharing disclosed | TODO | No third-party sharing |
| Data retention periods stated | TODO | 24 months operational, 36 months audit |
| Account deletion process described | TODO | 30-day purge after soft delete |
| COPPA compliance statement | TODO | Guardian consent for under-13 |
| California (CCPA) compliance | TODO | If applicable |

### 3.2 Terms of Service

| Item | Status | Notes |
|------|--------|-------|
| Terms of Service URL | TODO | Must be publicly accessible (server endpoint `GET /legal/terms-of-service` exists; needs public hosting URL) |
| Acceptable use policy | TODO | No harassment, abuse |
| Content moderation policy | TODO | 4h critical, 24h standard SLA |
| Age requirements stated | TODO | 13+ or guardian consent |

### 3.3 App Privacy Labels

Fill out in App Store Connect → App Privacy:

| Category | Data Type | Collected | Linked to Identity | Tracking |
|----------|-----------|-----------|-------------------|----------|
| Contact Info | Email Address | Yes | Yes | No |
| Contact Info | Name | Yes (optional) | Yes | No |
| Identifiers | User ID | Yes | Yes | No |
| Usage Data | Product Interaction | Yes | Yes | No |
| Sensitive Info | None | No | — | — |
| Health & Fitness | None | No | — | — |
| Financial Info | None | No | — | — |
| Location | None | No | — | — |
| Contacts | None | No | — | — |
| Browsing History | None | No | — | — |
| Diagnostics | None | No | — | — |

**Tracking declaration: NO** (no ATT, no IDFA, no ad networks)

### 3.4 Age Rating

Answer "No" to all content descriptors:
- Cartoon/Fantasy Violence: No
- Realistic Violence: No
- Sexual Content: No
- Profanity: No
- Drug/Alcohol/Tobacco: No
- Simulated Gambling: No
- Horror/Fear Themes: No
- Medical/Treatment Info: No (meds category is reminder-only, no medical advice)
- Mature/Suggestive Themes: No
- Contests: No

**Expected rating: 4+**

### 3.5 Medical Disclaimer

The "meds" category is a **reminder label only**. Nagz does not:
- Provide medical advice, diagnosis, or treatment
- Dispense or track medication dosage
- Interface with health records or medical devices
- Replace professional medical guidance

Include in App Store description and/or in-app About screen.

---

## Phase 4: Technical Preparation

### 4.1 Build Configuration

| Item | Action |
|------|--------|
| Set build configuration to Release | Xcode → Edit Scheme → Release |
| Update MARKETING_VERSION | Increment in project.yml |
| Update CURRENT_PROJECT_VERSION | Increment in project.yml |
| Verify deployment target | iOS 26.0 |
| Switch to production API URL | AppEnvironment.swift |
| Remove development NSAllowsLocalNetworking | Info.plist cleanup |
| Enable bitcode (if required) | Xcode build settings |
| Code signing: automatic (App Store) | Xcode signing settings |

### 4.2 Archive & Upload

```bash
# 1. Clean build folder
xcodebuild clean -project Nagz.xcodeproj -scheme Nagz

# 2. Archive
xcodebuild archive \
  -project Nagz.xcodeproj \
  -scheme Nagz \
  -archivePath ~/NagzArchive.xcarchive \
  -destination 'generic/platform=iOS'

# 3. Export for App Store (or use Xcode Organizer)
xcodebuild -exportArchive \
  -archivePath ~/NagzArchive.xcarchive \
  -exportPath ~/NagzExport \
  -exportOptionsPlist ExportOptions.plist

# 4. Upload via Xcode Organizer or:
xcrun altool --upload-app -f ~/NagzExport/Nagz.ipa \
  --type ios --apiKey KEY --apiIssuer ISSUER
```

Or use **Xcode → Product → Archive → Distribute App → App Store Connect**.

### 4.3 TestFlight Beta Testing

| Step | Action |
|------|--------|
| Upload build to App Store Connect | Via Xcode or altool |
| Add internal testers | App Store Connect → TestFlight → Internal |
| Add external testers | Requires Beta App Review (1-2 days) |
| Set beta app description | Explain what to test |
| Collect feedback | TestFlight feedback or shake-to-report |
| Fix critical bugs | Iterate builds |
| Run for minimum 2 weeks | Gather stability data |

### 4.4 Pre-Submission Checklist

| Check | Verify |
|-------|--------|
| All API endpoints reachable from production | curl each endpoint |
| Push notifications working in production | Test with production APNs cert |
| Token refresh works | Kill app, wait, relaunch |
| No debug logging in release build | Check console output |
| No development URLs in release build | grep for 127.0.0.1, localhost |
| App launches in < 3 seconds | Time cold launch |
| No crashes on common devices | Test on 3+ device types |
| All test suites passing | /test-all (644 tests: 254 + 126 + 264) |
| Version checker working | Verify /version endpoint live |
| AI chat works on AI device | Test on iPhone 15 Pro or newer |
| AI chat hidden on non-AI device | Verify Chat tab absent on iPhone SE/older |
| Chat persistence survives restart | Close app, reopen, check per-nag history |

---

## Phase 5: App Review Submission

### 5.1 Review Information

| Field | Content |
|-------|---------|
| **Demo account (guardian)** | reviewer-guardian@nagz.app / [password] |
| **Demo account (child)** | reviewer-child@nagz.app / [password] |
| **Contact info** | Name, email, phone for reviewer questions |
| **Notes for reviewer** | See APP_REVIEW_GUIDE.md for full walkthrough |

### 5.2 Review Notes Template

```
Nagz is a family task-reminder app with AI-powered conversational
management. Users chat naturally to create, complete, and manage
reminders. All AI runs on-device via Apple Foundation Models.

DEMO ACCOUNTS:
Guardian: reviewer-guardian@nagz.app / [password]
Child: reviewer-child@nagz.app / [password]
Both accounts are in the same family with sample nags.

TO TEST:
1. Log in as guardian → Chat tab → type "What's overdue?"
2. In Chat: "Remind me to call the dentist tomorrow" → creates nag
3. In Chat: "I took out the trash" → completes matching nag
4. Open a nag detail → tap sparkle icon → per-nag AI chat
5. People tab → see overdue digest per connection
6. Try Siri: "Remind me in Nagz"
7. View Points & Streaks for gamification

AI CHAT NOTES:
- Requires Apple Intelligence (iPhone 15 Pro or newer)
- Chat tab hidden on non-AI devices — all other features work
- 6 AI personalities available (Standard, Gordon Ramsay, Mr. Rogers, etc.)
- All text generation on-device — no external AI services
- Context window is limited; long conversations show friendly error

OTHER NOTES:
- "Meds" category is a reminder label only, no medical advice
- Child accounts (< 13) require guardian consent per COPPA
- No in-app purchases, ads, or tracking
- Push notifications used for reminders only
```

### 5.3 Common Rejection Reasons to Pre-Address

| Guideline | Risk | Mitigation |
|-----------|------|------------|
| **1.2 User-Generated Content** | Family members create text (nag descriptions, excuses) | Block + Report + moderation workflow implemented |
| **1.3 Kids Category** | App involves children | COPPA consent flow, guardian controls, age-gating |
| **2.1 App Completeness** | Server down during review | Ensure production server uptime, provide demo accounts |
| **2.3 Accurate Metadata** | "Meds" could imply medical | Disclaimer: reminder label only, no medical advice |
| **2.3 Accurate Metadata** | "AI" claims | All AI is on-device Apple Foundation Models — no external AI services, clearly disclosed |
| **3.1.1 In-App Purchase** | No monetization | Confirm no hidden paywalls or future-gating |
| **4.0 Design** | Push notification purpose | Only used for nag reminders, clear opt-in, creator notifications off by default |
| **4.0 Design** | AI features on older devices | Chat tab gracefully hidden via `#if canImport(FoundationModels)` + runtime check |
| **5.1.1 Data Collection** | Privacy labels accuracy | Verified: email, name, user ID, usage data only. AI chat stays on-device. |
| **5.1.2 Data Use and Sharing** | Third-party sharing | None — no analytics, no ads, no data brokers, no external AI |
| **5.6 App Intents** | Siri phrases work | Tested all 7 shortcuts with proper error handling |

---

## Phase 6: Post-Submission

### 6.1 During Review (1-3 days typical)

| Action | Notes |
|--------|-------|
| Monitor App Store Connect status | Check daily |
| Keep production server running | Don't deploy breaking changes |
| Be available for reviewer questions | Check Apple email |
| If rejected, read rejection reason carefully | Address specific guideline cited |

### 6.2 After Approval

| Action | Notes |
|--------|-------|
| Set release date | Immediate or scheduled |
| Verify app appears in App Store | Search "Nagz" |
| Monitor crash reports | Xcode Organizer → Crashes |
| Monitor user reviews | App Store Connect → Ratings |
| Plan V1.1 | Address initial feedback |

### 6.3 Ongoing Maintenance

| Cadence | Action |
|---------|--------|
| Weekly | Check crash reports, user reviews |
| Monthly | Update dependencies (KeychainAccess, GRDB) |
| Quarterly | iOS version compatibility check |
| Per Apple release | Test on new iOS betas |
| As needed | Server API version bumps (update CLIENT_API_VERSION) |

---

## Timeline Estimate

| Phase | Duration | Dependencies |
|-------|----------|-------------|
| Pre-submission (infra, legal, metadata) | 1-2 weeks | Production server, legal review |
| TestFlight beta | 2-4 weeks | Beta testers recruited |
| Bug fixes from beta | 1-2 weeks | Based on feedback volume |
| App Review submission | 1-3 days review | All metadata and builds ready |
| Post-approval launch | Same day or scheduled | Marketing readiness |
| **Total** | **5-9 weeks** | |

---

## File Checklist

| File | Purpose | Location |
|------|---------|----------|
| TESTFLIGHT_TEST_PLAN.md | Beta tester instructions | nagz/Docs/ |
| APP_REVIEW_GUIDE.md | Detailed review walkthrough | nagz/Docs/ |
| APP_STORE_SUBMISSION.md | This document | nagz/Docs/ |
| Privacy Policy | Legal (server-hosted) | Server endpoint: `GET /api/v1/legal/privacy-policy` (needs production URL) |
| Terms of Service | Legal (server-hosted) | Server endpoint: `GET /api/v1/legal/terms-of-service` (needs production URL) |
| ExportOptions.plist | Archive export config | TODO: create in nagz-ios |
| Screenshots (6-8) | App Store listing | TODO: capture |
| App Icon (1024x1024) | App Store listing | TODO: design |
