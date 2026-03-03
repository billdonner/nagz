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
| **Create a nag** | Nagz tab → + button → fill form → save | Nag appears in list with correct category icon |
| **Set due date** | Create nag with due date 5 min from now | Nag shows countdown in list |
| **Add description** | Create nag with optional description | Description visible in detail view |
| **Set recurrence** | Create daily/weekly/monthly nag | Recurrence badge shown |
| **Complete a nag** | Open nag → "Mark Complete" | Status changes to completed (green) |
| **Complete with note** | Create nag with "Check Off + Note" type → complete | Note field appears, saved with completion |
| **Filter by status** | Use segmented picker: Open / Completed / Missed | List filters correctly |
| **Pull to refresh** | Pull down on nag list | List refreshes from server |
| **Collapsible sections** | Tap section header ("From X", "To X") | Section collapses/expands with chevron animation |
| **Default collapse** | Open filter tab | "From" sections expanded, "To" and "My Reminders" collapsed by default |
| **Direction indicators** | View nag rows | Arrow icons (↙ received, ↗ sent, ↺ self) with colored sidebars |
| **Urgency sparklines** | View section headers | Small colored bars showing urgency distribution of nags in each group |
| **Schedule view** | Tap calendar icon in toolbar | Switches to timeline view sorted by due date |
| **Self-nag warning** | Create nag → select yourself as recipient | Orange warning: "This will remind you, not someone else"; button says "Remind Myself" |
| **Overdue banner** | Open Chat tab | Shows "X overdue for you" (only YOUR tasks, not tasks sent to others) |

### Family Management

| Test | Steps | Expected |
|------|-------|----------|
| **View members** | Settings tab → Family → Members | All members listed with roles |
| **Add member** | Manage Members → Add → enter name + role | New member appears |
| **Remove member** | Manage Members → Remove (non-guardian) | Confirmation dialog → member removed |
| **Invite code** | Settings tab → Family → copy invite code | Code works when pasted on another device |

### Roles & Permissions

| Test | Steps | Expected |
|------|-------|----------|
| **Guardian powers** | Log in as guardian | Can see Reports, Policies, Manage Members, Incentive Rules via Settings → Family |
| **Participant** | Log in as participant | Can create nags, view own nags, no admin features |
| **Child restrictions** | Log in as child | Cannot create nags for guardians, cannot see reports |

### Gamification

| Test | Steps | Expected |
|------|-------|----------|
| **View points** | Settings tab → Points & Streaks | Total points, streak, recent activity shown |
| **Leaderboard** | Enable gamification consent → view leaderboard | Top members with medal emojis |
| **Earn points** | Complete a nag on time | Points increase, event logged |

### AI Chat — "Talk to Nagz"

The Chat tab (first tab, requires Apple Intelligence device) lets you manage nags conversationally.

| Test | Steps | Expected |
|------|-------|----------|
| **Chat tab visible** | Open app on iPhone 15 Pro or newer | Chat tab appears as first tab with message icon |
| **Chat tab hidden** | Open app on older device without Apple Intelligence | Chat tab does not appear |
| **Greeting** | Open Chat tab | Personality-appropriate greeting message appears |
| **List nags** | Type "What's overdue?" or "Show my tasks" | AI lists your open nags with overdue indicators |
| **Create nag** | Type "Remind me to call the dentist tomorrow" | AI creates nag, confirms recipient and due time |
| **Create nag for someone** | Type "Nag Cookie to do her homework" | AI finds the person and creates nag for them |
| **Complete nag** | Type "I took out the trash" | AI finds matching nag and marks it done |
| **Reschedule nag** | Type "Push back the homework to tomorrow" | AI reschedules the matching nag |
| **Submit excuse** | Type "Send excuses for my overdue nags — I'm sick" | AI submits excuse on overdue nags (does NOT create new nags) |
| **Status check** | Type "How am I doing?" | AI summarizes YOUR tasks separately from tasks you're monitoring for others |
| **Self-nag** | Type "Remind me to pick up groceries" | Creates nag assigned to yourself, appears in "My Reminders" |
| **Keyboard dismiss** | Tap outside text field, or tap "Done" above keyboard | Keyboard dismisses, tab bar accessible |
| **Long conversation** | Send 10+ messages | Error message if context window exceeded, suggesting fresh start |
| **Personality** | Change personality in settings → reopen Chat | Greeting and tone match selected personality |

### Per-Nag AI Chat

| Test | Steps | Expected |
|------|-------|----------|
| **Open nag chat** | Open any nag detail → tap chat bubble icon | Chat sheet opens with personality greeting and nag context |
| **Reschedule via chat** | In nag chat: "Can I do this tomorrow?" | AI pushes back, then reschedules if you insist |
| **Complete via chat** | In nag chat: "I finished it" | AI marks nag done and congratulates |
| **Submit excuse** | In nag chat: "I'm not feeling well" | AI submits excuse for review |
| **Chat persistence** | Close and reopen nag chat | Prior messages visible as read-only history |

### AI Insights

| Test | Steps | Expected |
|------|-------|----------|
| **Nag AI section** | Open any nag → scroll to "AI Insights" | Tone badge, coaching tip, and completion prediction shown |
| **Tone indicator** | Check AI Insights section | Color-coded capsule (blue/green/red) with LLM-generated explanation |
| **Coaching tip** | Check AI Insights section | Lightbulb icon with personalized tip text |
| **Completion gauge** | Check AI Insights section | Percentage bar showing likelihood + optional reminder time |
| **Family Insights** | Settings tab → Family → AI Insights (guardian only) | Weekly digest with LLM summary, per-member stats, and miss patterns |
| **List summary** | View nag list header | 2-3 sentence summary of your task list |
| **Gamification nudges** | Points & Streaks → Tips section | Personalized streak/badge nudges with emoji icons |
| **Pull to refresh** | Family Insights → pull down | Data refreshes |

### Safety

| Test | Steps | Expected |
|------|-------|----------|
| **Block member** | Settings tab → Family → Safety → Block | Confirmation dialog → member blocked |
| **Report member** | Settings tab → Family → Safety → Report | Abuse report submitted |
| **Delete account** | Settings tab → Account → Delete Account | Warning dialog → account deleted, returned to login |

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
| **"Remind me in Nagz"** | Prompts for description and time, creates self-reminder (default 60 min, category "other") |

Also test in Shortcuts app:
- Open Shortcuts → search "Nagz" → all 7 shortcuts should appear (including "Quick Remind")
- Create an automation (e.g., morning nag check at 7 AM)

---

## Push Notifications

| Test | Steps | Expected |
|------|-------|----------|
| **Permission prompt** | First launch after login | System dialog asks for notification permission |
| **Nag reminder** | Create nag with near-future due date | Push notification arrives near due time |
| **Tap notification** | Tap the push notification | App opens to the relevant nag detail (Nagz tab) |
| **No creator alerts (default)** | Create nag for someone else, let it go overdue | You do NOT get a push notification (default off) |
| **Opt-in creator alerts** | Family → Preferences → enable "Notify when sent nags are overdue" | You now get alerts when nags you sent go overdue |
| **Quiet hours** | Set quiet hours in Preferences | No pushes during quiet window |

## People Tab — Overdue Digest

| Test | Steps | Expected |
|------|-------|----------|
| **Overdue badge** | Send nags to connections, let some go overdue → open People tab | Red "Overdue" count next to connections with past-due nags |
| **Stats per connection** | Open People tab | Each connection shows Sent, Received, Open, Done counts |
| **Self-nag section** | Create nags for yourself | They appear in "My Reminders" section at bottom of Nagz tab |

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

## Known Limitations (V1.3 Beta)

- Server URL may be development instance (slower responses)
- SMS delivery not enabled in beta
- AI Chat and per-nag chat require Apple Intelligence (iPhone 15 Pro or newer). On older devices, the Chat tab and chat buttons are hidden — all other features work normally.
- On-device LLM has a small context window — very long chat conversations will hit a limit. Switch tabs and come back to start fresh.
- The on-device model may occasionally refuse certain requests (content guardrails). Rephrase and try again.
- AI features use Apple Foundation Models (on-device LLM) — all text generation happens on your iPhone:
  - On-device LLM: AI chat, excuse summaries, tone selection, coaching tips, weekly digests, push-back reminders, list summaries, gamification nudges
  - Local heuristics only (no LLM needed): pattern detection, completion prediction
  - No user text ever leaves your device — only structured data syncs to the server
  - If the LLM is unavailable, heuristic fallbacks ensure every feature still works
- Only "friendly_reminder" strategy template available
- US-only launch scope
