# TestFlight Test Plan

> For beta testers evaluating the Nagz app before public release.

## What Is Nagz?

Nagz is a family task-reminder app. Guardians create "nags" (reminders) for family members — chores, meds, homework, appointments — and track completion. The app includes escalation, gamification, AI-powered coaching, and Siri Shortcuts.

---

## Getting Started

### 1. Create an Account

| Step | Expected Result |
|------|-----------------|
| Open the app | Login screen appears |
| Tap "Sign Up" | Signup form: email, password, display name, date of birth |
| Enter valid info and submit | Account created, you're logged in |
| Try signing up with age < 13 | Should require guardian consent flow |

### 2. Create or Join a Family

| Step | Expected Result |
|------|-----------------|
| Tap "Create Family" | Enter family name → family created with invite code |
| Copy invite code | Code copied to clipboard |
| (On second device) Tap "Join Family" | Enter invite code → joined as member |

---

## Core Features to Test

### Nags (Reminders)

| Test | Steps | Expected |
|------|-------|----------|
| **Create a nag** | Family tab → Nags → + button → fill form → save | Nag appears in list with correct category icon |
| **Set due date** | Create nag with due date 5 min from now | Nag shows countdown in list |
| **Add description** | Create nag with optional description | Description visible in detail view |
| **Set recurrence** | Create daily/weekly/monthly nag | Recurrence badge shown |
| **Complete a nag** | Open nag → "Mark Complete" | Status changes to completed (green) |
| **Complete with note** | Create nag with "Check Off + Note" type → complete | Note field appears, saved with completion |
| **Filter by status** | Use segmented picker: Open / Completed / Missed | List filters correctly |
| **Pull to refresh** | Pull down on nag list | List refreshes from server |

### Family Management

| Test | Steps | Expected |
|------|-------|----------|
| **View members** | Family tab → Members | All members listed with roles |
| **Add member** | Manage Members → Add → enter name + role | New member appears |
| **Remove member** | Manage Members → Remove (non-guardian) | Confirmation dialog → member removed |
| **Invite code** | Family tab → copy invite code | Code works when pasted on another device |

### Roles & Permissions

| Test | Steps | Expected |
|------|-------|----------|
| **Guardian powers** | Log in as guardian | Can see Reports, Policies, Manage Members, Incentive Rules |
| **Participant** | Log in as participant | Can create nags, view own nags, no admin features |
| **Child restrictions** | Log in as child | Cannot create nags for guardians, cannot see reports |

### Gamification

| Test | Steps | Expected |
|------|-------|----------|
| **View points** | Family tab → Points & Streaks | Total points, streak, recent activity shown |
| **Leaderboard** | Enable gamification consent → view leaderboard | Top members with medal emojis |
| **Earn points** | Complete a nag on time | Points increase, event logged |

### AI Insights

| Test | Steps | Expected |
|------|-------|----------|
| **Nag AI section** | Open any nag → scroll to "AI Insights" | Tone badge, coaching tip, and completion prediction shown |
| **Tone indicator** | Check AI Insights section | Color-coded capsule (blue/green/red) with LLM-generated explanation |
| **Coaching tip** | Check AI Insights section | Lightbulb icon with personalized tip text |
| **Completion gauge** | Check AI Insights section | Percentage bar showing likelihood + optional reminder time |
| **Family Insights** | Family tab → AI Insights (guardian only) | Weekly digest with LLM summary, per-member stats, and miss patterns |
| **List summary** | View nag list header | 2-3 sentence summary of your task list |
| **Gamification nudges** | Points & Streaks → Tips section | Personalized streak/badge nudges with emoji icons |
| **Pull to refresh** | Family Insights → pull down | Data refreshes |

### Safety

| Test | Steps | Expected |
|------|-------|----------|
| **Block member** | Family tab → Safety → Block | Confirmation dialog → member blocked |
| **Report member** | Family tab → Safety → Report | Abuse report submitted |
| **Delete account** | Family tab → Account → Delete Account | Warning dialog → account deleted, returned to login |

---

## Siri & Shortcuts

Test these by voice or in the Shortcuts app:

| Phrase | Expected |
|--------|----------|
| "Show my nags in Nagz" | Lists active nags with count |
| "Create a nag in Nagz" | Prompts for recipient, category, due date |
| "Complete a nag in Nagz" | Prompts for which nag, marks done |
| "Check overdue nags in Nagz" | Reports overdue count and categories |
| "Snooze a nag in Nagz" | Prompts for nag and minutes, reschedules |
| "Family status in Nagz" | Reports weekly completion rate |

Also test in Shortcuts app:
- Open Shortcuts → search "Nagz" → all 6 shortcuts should appear
- Create an automation (e.g., morning nag check at 7 AM)

---

## Push Notifications

| Test | Steps | Expected |
|------|-------|----------|
| **Permission prompt** | First launch after login | System dialog asks for notification permission |
| **Nag reminder** | Create nag with near-future due date | Push notification arrives near due time |
| **Tap notification** | Tap the push notification | App opens to the relevant nag detail |

---

## Edge Cases

| Test | Expected |
|------|----------|
| Kill app and relaunch | Session restored, family auto-loaded |
| Lose network mid-action | Error banner with retry option |
| Create nag with past due date | Should be rejected with validation error (due date must be in the future) |
| Very long description text | Text wraps properly, no truncation |
| Logout and login as different user | Clean state, different family |
| Rapid tap "Complete" multiple times | Only processes once, no double-completion |

---

## What to Report

When filing feedback, please include:
- **Device**: iPhone model and iOS version
- **Steps to reproduce**: Exact steps that led to the issue
- **Expected vs actual**: What should have happened vs what did
- **Screenshot or screen recording**: If visual

Use the TestFlight feedback button (shake device or screenshot → "Send Beta Feedback").

---

## Known Limitations (V1.0 Beta)

- Server URL may be development instance (slower responses)
- SMS delivery not enabled in beta
- AI features use Foundation Models (on-device LLM) for 7 of 9 operations (excuse summaries, tone, coaching, digest, push-back, list summaries, gamification nudges); patterns and prediction use heuristics
- Only "friendly_reminder" strategy template available
- US-only launch scope
