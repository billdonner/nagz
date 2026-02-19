# Nagz Deployment Plan — TestFlight to App Store

## Overview

This document covers the full path from current state to App Store release, including server deployment choices, TestFlight beta, and App Store submission.

**Current state:** 3 repos, 516+ tests passing, code review fixes applied, all features implemented.

---

## Phase 1: Server Deployment (Week 1-2)

The iOS app connects to `https://api.nagz.app/api/v1` in production. The server needs to be running before TestFlight or App Store.

### Deployment Platform Options

| Platform | Cost | Pros | Cons | Recommendation |
|----------|------|------|------|----------------|
| **Fly.io** | ~$5-15/mo | Easy Postgres & Redis add-ons, auto-TLS, global edge, `fly deploy` | Newer platform, less enterprise support | **Best for launch** |
| **Railway** | ~$5-20/mo | One-click Postgres/Redis, GitHub deploy, simple UI | Less control over networking | Good alternative |
| **Render** | ~$7-25/mo | Free tier available, managed Postgres, auto-deploy from GitHub | Cold starts on free tier | Good for cost-sensitive |
| **DigitalOcean App Platform** | ~$12-24/mo | Managed Postgres ($15), familiar UI, good docs | More expensive for what you get | Solid but pricier |
| **AWS (ECS/Fargate + RDS)** | ~$30-80/mo | Full control, enterprise-grade, auto-scaling | Complex setup, overkill for launch | Defer to post-traction |
| **Hetzner + Docker** | ~$5-10/mo | Cheapest, full control | Manual setup, no managed services | Only if budget-constrained |

### Recommended: Fly.io

```
# One-time setup
fly launch --name nagzerver --region iad
fly postgres create --name nagz-db
fly redis create --name nagz-redis
fly secrets set NAGZ_AUTH_MODE=jwt
fly secrets set NAGZ_JWT_SECRET=$(openssl rand -hex 32)
fly secrets set NAGZ_APNS_KEY_ID=...
fly secrets set NAGZ_APNS_TEAM_ID=...
```

### Server Deployment Checklist

- [ ] Choose platform (Fly.io recommended)
- [ ] Set up PostgreSQL (managed, daily backups)
- [ ] Set up Redis (for rate limiting + cache)
- [ ] Configure DNS: `api.nagz.app` → server
- [ ] Configure TLS/SSL (auto via platform)
- [ ] Set environment variables:
  - `NAGZ_AUTH_MODE=jwt`
  - `NAGZ_JWT_SECRET=<strong random value>`
  - `NAGZ_DATABASE_URL=<production postgres URL>`
  - `NAGZ_REDIS_URL=<production redis URL>`
  - `NAGZ_APNS_KEY_ID`, `NAGZ_APNS_TEAM_ID`, `NAGZ_APNS_KEY_PATH`, `NAGZ_APNS_BUNDLE_ID`
  - `NAGZ_APNS_USE_SANDBOX=false`
  - `NAGZ_SCHEDULER_ENABLED=true`
  - `NAGZ_CORS_ORIGINS=https://nagz.online`
- [ ] Run `alembic upgrade head` for database migrations
- [ ] Create demo reviewer accounts (see Phase 4)
- [ ] Seed demo family with sample nags
- [ ] Verify `GET /api/v1/version` returns correctly
- [ ] Load test: 100 concurrent users, all endpoints
- [ ] Set up monitoring (Sentry for errors, uptime check for `api.nagz.app/api/v1/version`)

### Dockerfile (create in nagzerver root)

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen --no-dev
COPY src/ src/
COPY alembic/ alembic/
COPY alembic.ini .
EXPOSE 9800
CMD ["uv", "run", "nagz-server"]
```

---

## Phase 2: Web App Deployment (Week 1-2, parallel with server)

### Options

| Platform | Cost | Notes |
|----------|------|-------|
| **Vercel** | Free tier | Best for React/Vite, auto-deploy from GitHub |
| **Netlify** | Free tier | Similar to Vercel, slightly different build config |
| **Fly.io static** | Bundled | Keep everything on one platform |

### Recommended: Vercel (free)

```bash
cd ~/nagz-web
npx vercel --prod
```

Set `VITE_API_URL=https://api.nagz.app` as environment variable in Vercel dashboard.

---

## Phase 3: Apple Developer Setup (Week 1)

### Prerequisites

- [ ] Apple Developer Program enrollment ($99/year) — https://developer.apple.com/programs/
- [ ] App ID registered: `com.nagz.app`
- [ ] Push notification capability enabled (production)
- [ ] App Store Connect record created
- [ ] APNS production key generated (.p8 file)
  - Copy to server and set `NAGZ_APNS_KEY_PATH`

### Certificates & Profiles

- [ ] Distribution certificate (Automatic Signing handles this)
- [ ] App Store provisioning profile (Automatic Signing handles this)
- [ ] Entitlements: change `aps-environment` from `development` to `production`

### App Store Connect Setup

- [ ] Create app: "Nagz" / Bundle ID: `com.nagz.app`
- [ ] Primary category: **Productivity**
- [ ] Secondary category: **Lifestyle**
- [ ] Price: **Free**
- [ ] Availability: **United States** (US-only launch per SAFETY_AND_COMPLIANCE.md)

---

## Phase 4: TestFlight Beta (Week 2-3)

### Build & Upload

```bash
cd ~/nagz-ios

# 1. Update entitlements for production
# Edit Nagz.entitlements: aps-environment → production

# 2. Archive
xcodebuild archive \
  -project Nagz.xcodeproj \
  -scheme Nagz \
  -archivePath build/Nagz.xcarchive \
  -destination 'generic/platform=iOS'

# 3. Export for App Store
xcodebuild -exportArchive \
  -archivePath build/Nagz.xcarchive \
  -exportPath build/export \
  -exportOptionsPlist ExportOptions.plist

# 4. Upload via Xcode Organizer or xcrun altool
```

Or simply: **Xcode → Product → Archive → Distribute App → App Store Connect**

### TestFlight Configuration

- [ ] Upload build to App Store Connect
- [ ] Wait for processing (~15 min)
- [ ] Add internal testers (your team, up to 100)
- [ ] Write beta app description using `TESTFLIGHT_TEST_PLAN.md`
- [ ] Set "What to Test" notes
- [ ] Enable automatic distribution to internal testers
- [ ] After internal testing: add external testers (up to 10,000)
  - External requires Beta App Review (~24-48 hours)

### TestFlight Test Matrix

| Area | Test Cases | Priority |
|------|-----------|----------|
| Auth | Login/signup/logout, session persistence | P0 |
| Family | Create, join via code, add/remove members | P0 |
| Nags | Create, complete, miss, snooze, excuse | P0 |
| Escalation | Phase progression, recompute | P1 |
| Gamification | Points, streaks, badges, leaderboard | P1 |
| Push | Notification delivery, tap-to-open | P0 |
| Siri | All 6 shortcuts via voice | P1 |
| Offline | GRDB cache, sync on reconnect | P2 |
| Safety | Block, report, consent management | P1 |
| Version | Compatibility check on launch | P2 |

---

## Phase 5: App Store Metadata (Week 3)

### Required Assets

| Asset | Spec | Status |
|-------|------|--------|
| App icon | 1024x1024 PNG, no alpha | TODO |
| iPhone 6.7" screenshots | 1290x2796 (iPhone 15 Pro Max) x6-8 | TODO |
| iPhone 6.1" screenshots | 1179x2556 (iPhone 15 Pro) x6-8 | TODO |
| App preview video | 15-30s, optional | Optional |

### Screenshot Sequence (recommended)

1. **Hero:** Kid's nag list with cards and escalation badges
2. **Create:** Create nag form with category picker
3. **Family:** Family dashboard with member roles
4. **Gamification:** Leaderboard with points and streaks
5. **Detail:** Nag detail with snooze, excuse, and coaching tip
6. **Siri:** Shortcuts in action

### App Store Description

```
Name: Nagz
Subtitle: Family Task Reminders

Description:
Nagz helps families stay on top of tasks, chores, and responsibilities.
Parents create "nags" — friendly reminders with due dates and categories.
Kids see their tasks, mark them complete, or submit excuses. Everyone
earns points and competes on the family leaderboard.

Features:
• Create task reminders with categories (homework, chores, meds, etc.)
• Automatic escalation when tasks are overdue
• Points, streaks, and badges for completing tasks
• Family leaderboard to motivate everyone
• AI coaching tips for encouragement
• Siri Shortcuts for hands-free nag management
• Push notifications for due and overdue tasks
• Safety controls: blocking, reporting, quiet hours

Keywords: family,reminders,chores,tasks,kids,parenting,nag,homework,
          meds,streaks,leaderboard,productivity
```

### Privacy & Legal

| Item | URL | Status |
|------|-----|--------|
| Privacy Policy | `https://api.nagz.app/api/v1/legal/privacy-policy` | Implemented |
| Terms of Service | `https://api.nagz.app/api/v1/legal/terms-of-service` | Implemented |
| Support URL | `https://nagz.online/support.html` | Implemented |
| Marketing URL | `https://nagz.online` | Implemented |

### App Privacy Labels

| Data Type | Collected | Linked to Identity | Used for Tracking |
|-----------|-----------|-------------------|-------------------|
| Email Address | Yes | Yes | No |
| Name | Yes | Yes | No |
| User ID | Yes | Yes | No |
| Product Interaction | Yes | Yes | No |
| App Functionality | Yes | Yes | No |

**No tracking. No third-party analytics. No advertising.**

### Age Rating

- Rating: **4+**
- All content descriptors: **None**
- Kids category: No (to avoid additional COPPA review complexity)
- Medical disclaimer note in review notes: "Meds category is a reminder label only"

---

## Phase 6: App Review Submission (Week 3-4)

### Demo Account Setup

Create on production server before submission:

```
Guardian: reviewer-guardian@nagz.app / ReviewPass123!
Child: reviewer-child@nagz.app / ReviewPass123!
```

Both in the same family ("Review Family") with:
- 5-10 sample nags in various states (open, completed, missed)
- Some with excuses and escalation
- Gamification data (points, badges)
- Consents granted

### Review Notes Template

```
Demo Accounts:
Guardian: reviewer-guardian@nagz.app / ReviewPass123!
Child: reviewer-child@nagz.app / ReviewPass123!

Both accounts are in the "Review Family" with pre-populated nags.

The "Meds" category in nag creation is a reminder label only.
This app does not provide medical advice or track medication dosages.

Siri Shortcuts: Say "Show my nags in Nagz" after granting Siri access.

Push Notifications: Grant permission when prompted. Notifications are sent
when nags are due or overdue. The background scheduler runs every 60 seconds.

This app requires an active internet connection for all features except
viewing previously-synced nag data.
```

### Common Rejection Reasons to Avoid

| Risk | Mitigation |
|------|------------|
| Guideline 5.1.1 - Data collection | Privacy labels accurate, privacy policy URL set |
| Guideline 2.1 - App completeness | All features functional, no placeholder content |
| Guideline 4.0 - Design | Native SwiftUI, follows HIG patterns |
| Guideline 1.2 - User-generated content | Block/report/mute features implemented |
| Guideline 5.2.1 - Legal | Terms of Service and Privacy Policy hosted |
| Guideline 2.3 - Accurate metadata | Screenshots match actual app behavior |

---

## Phase 7: Post-Launch (Week 4+)

### Monitoring

- [ ] Sentry for crash reporting (iOS + server)
- [ ] Uptime monitoring for `api.nagz.app`
- [ ] App Store Connect crash reports review
- [ ] Customer review monitoring

### Maintenance

- [ ] Weekly server dependency updates
- [ ] iOS SDK updates as Xcode releases
- [ ] Database backup verification
- [ ] Rate limit monitoring via Redis

---

## Timeline Summary

| Week | Milestone |
|------|-----------|
| 1 | Server deployed, DNS configured, Apple Developer enrolled |
| 2 | TestFlight internal build uploaded, web app deployed |
| 3 | External TestFlight, App Store metadata complete |
| 4 | App Review submission, demo accounts ready |
| 4-5 | App Review (1-3 day typical turnaround) |
| 5 | App Store release |

---

## Cost Estimate (Monthly)

| Service | Cost |
|---------|------|
| Apple Developer Program | $8.25/mo ($99/year) |
| Fly.io (server + Postgres + Redis) | $10-20/mo |
| Vercel (web hosting) | Free |
| Domain (nagz.online + nagz.app) | ~$3/mo |
| Sentry (error monitoring) | Free tier |
| **Total** | **~$22-32/month** |
