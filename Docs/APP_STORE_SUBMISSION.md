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
| Production API server deployed | TODO | api.nagz.app or equivalent |
| SSL/TLS certificate | TODO | Valid cert for HTTPS |
| Update AppEnvironment.swift production URL | TODO | Replace placeholder |
| APNs production push configured | TODO | Server-side push sending |
| SMS provider configured (Twilio/etc.) | TODO | US-only, STOP/HELP support |
| Database backups configured | TODO | Automated daily backups |
| Monitoring/alerting (Sentry, DataDog) | TODO | Error tracking, uptime |
| Rate limiting verified | TODO | 120 read/min, 60 write/min per user |
| Load testing completed | TODO | Simulate 1000+ concurrent users |

---

## Phase 2: App Store Metadata

### 2.1 App Information

| Field | Content |
|-------|---------|
| **App Name** | Nagz |
| **Subtitle** | Family Task Reminders |
| **Promotional Text** | Keep your family on track with smart reminders, gamification, and Siri integration. |

### 2.2 Description (4000 char max)

```
Nagz helps families stay organized with smart, escalating reminders.

WHAT IT DOES
Guardians create "nags" — task reminders for chores, meds, homework, and appointments. Family members complete them, earn points, and build streaks. If a nag is missed, it escalates — gently at first, then more firmly.

KEY FEATURES
• Create reminders with categories, due dates, and recurrence
• Track completion with points, streaks, and family leaderboards
• Escalating reminders that adapt over time
• Siri Shortcuts — "Show my nags" or "Create a nag" by voice
• Role-based access: guardians manage, participants and children complete
• Safety tools: block, report, and mute family members
• On-device AI coaching and behavioral insights
• Push notification reminders at every escalation phase

BUILT FOR FAMILIES
• Guardian controls for policies, consents, and incentive rules
• COPPA-compliant child accounts with guardian consent
• Quiet hours to silence reminders at bedtime
• Gamification that motivates without pressuring

PRIVACY FIRST
• No ads, no tracking, no data selling
• On-device AI processing when possible
• Minimal data collection: just email and family task data
• Account deletion available anytime

Start your family's productivity journey with Nagz.
```

### 2.3 Keywords (100 char max)

```
family,reminders,chores,tasks,kids,parenting,nag,homework,meds,streaks
```

### 2.4 Screenshots (Required)

| Device | Size | Quantity |
|--------|------|----------|
| iPhone 6.9" (16 Pro Max) | 1320 x 2868 | 4-8 screenshots |
| iPhone 6.7" (15 Plus) | 1290 x 2796 | 4-8 screenshots |
| iPad 13" (optional) | 2064 x 2752 | 0-8 screenshots |

**Recommended screenshots:**
1. Nag list with open reminders (hero shot)
2. Create nag form with category picker
3. Nag detail with escalation phase
4. Family members with roles
5. Points & streaks gamification view
6. Siri Shortcuts in action
7. Reports / completion rate
8. Safety features (block/report)

### 2.5 App Icon

| Size | Requirement |
|------|-------------|
| 1024 x 1024 px | Single-layer, no transparency, no rounded corners |
| Format | PNG |

### 2.6 App Preview Video (Optional)

- 15-30 seconds
- Show: create nag → notification → complete → earn points → Siri shortcut
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
| Verify deployment target | iOS 17.0 |
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
| All test suites passing | /test-all |
| Version checker working | Verify /version endpoint live |

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
Nagz is a family task-reminder app where guardians create reminders
("nags") for family members. Members complete tasks, earn points,
and build streaks.

DEMO ACCOUNTS:
Guardian: reviewer-guardian@nagz.app / [password]
Child: reviewer-child@nagz.app / [password]
Both accounts are in the same family with sample nags.

TO TEST:
1. Log in as guardian → view nag list → create a nag
2. Log in as child → view assigned nags → complete a nag
3. Try Siri: "Show my nags in Nagz"
4. View Points & Streaks for gamification

NOTES:
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
| **3.1.1 In-App Purchase** | No monetization | Confirm no hidden paywalls or future-gating |
| **4.0 Design** | Push notification purpose | Only used for nag reminders, clear opt-in |
| **5.1.1 Data Collection** | Privacy labels accuracy | Verified: email, name, user ID, usage data only |
| **5.1.2 Data Use and Sharing** | Third-party sharing | None — no analytics, no ads, no data brokers |
| **5.6 App Intents** | Siri phrases work | Tested all 14 phrases, proper error handling |

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
