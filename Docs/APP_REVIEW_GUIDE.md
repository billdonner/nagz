# App Review Guide — Detailed Review Plan

> For Apple App Review and internal QA reviewers performing a thorough evaluation.

## App Summary

| Field | Value |
|-------|-------|
| **App Name** | Nagz |
| **Bundle ID** | com.nagz.app |
| **Version** | 1.3.0 (Build 14) |
| **Platform** | iOS 26.0+ |
| **Swift** | 6.0 (strict concurrency) |
| **Category** | Lifestyle / Family |
| **Dependencies** | KeychainAccess 4.2.2, GRDB 7.0.0, NagzAI (local SPM) |
| **Backend** | Python (FastAPI), hosted at api.nagz.app |
| **Price** | Free (no IAP, no subscriptions, no ads) |

## Demo Account Credentials

| Role | Email | Password |
|------|-------|----------|
| Guardian | reviewer-guardian@nagz.app | [provide to Apple] |
| Participant | reviewer-participant@nagz.app | [provide to Apple] |
| Child | reviewer-child@nagz.app | [provide to Apple] |

All accounts are pre-configured in the same family with sample nags.

---

## 1. Authentication & Account Lifecycle

### 1.1 Signup Flow

| Step | Action | Verify |
|------|--------|--------|
| 1 | Launch app | Login screen displayed |
| 2 | Tap "Sign Up" | Form: email, password, display name (optional), date of birth |
| 3 | Enter DOB making user 13+ | Account created normally |
| 4 | Enter DOB making user < 13 | Child account flow triggered — requires guardian consent |
| 5 | Submit with invalid email | Inline validation error |
| 6 | Submit with short password | Server-side error displayed |

### 1.2 Login Flow

| Step | Action | Verify |
|------|--------|--------|
| 1 | Enter valid credentials | Logged in, family auto-restored if previously joined |
| 2 | Enter wrong password | Error message: "Invalid credentials" |
| 3 | Kill app, relaunch | Session restored via Keychain token refresh |

### 1.3 Logout

| Step | Action | Verify |
|------|--------|--------|
| 1 | Family tab → Log Out | Confirmation → returns to login screen |
| 2 | Keychain tokens cleared | Relaunching app shows login (no auto-restore) |
| 3 | UserDefaults cleared | nagz_user_id and nagz_family_id removed |

### 1.4 Account Deletion

| Step | Action | Verify |
|------|--------|--------|
| 1 | Family tab → Account → Delete Account | Warning: "This action is permanent" |
| 2 | Confirm deletion | Account soft-deleted, returned to login |
| 3 | Try logging in with deleted account | Login fails |

---

## 2. Family Management

### 2.1 Create Family

| Step | Action | Verify |
|------|--------|--------|
| 1 | Tap "Create Family" | Name field displayed |
| 2 | Enter name, submit | Family created with auto-generated invite code |
| 3 | Invite code displayed | 6-character alphanumeric code, copy button works |

### 2.2 Join Family

| Step | Action | Verify |
|------|--------|--------|
| 1 | Tap "Join Family" | Invite code field |
| 2 | Enter valid code | Joined as member, family view loads |
| 3 | Enter invalid code | Error: "Invalid invite code" |

### 2.3 Member Management (Guardian Only)

| Step | Action | Verify |
|------|--------|--------|
| 1 | Navigate to Manage Members | List of members with roles |
| 2 | Add member: display name + role | Member added, appears in list |
| 3 | Remove non-guardian member | Confirmation → member removed |
| 4 | Try removing self (guardian) | Should be prevented |

---

## 3. Nag (Reminder) CRUD

### 3.1 Create Nag

| Field | Input | Verify |
|-------|-------|--------|
| Recipient | Select family member | Dropdown populated from member list |
| Category | chores / meds / homework / appointments / other | Icon matches selection |
| Due Date | Date picker | Defaults to reasonable future time |
| Completion Type | ack_only / binary_check / binary_with_note | Affects completion UX |
| Description | Optional text | Shown in detail view |
| Recurrence | none / daily / weekly / monthly | Badge shown in list |

**Permission checks:**
- Guardian: can create for anyone
- Participant: can create for other participants
- Child: cannot create nags for guardians

### 3.2 View Nag List

| Filter | Expected |
|--------|----------|
| Open | Active, uncompleted nags |
| Completed | Green status pill |
| Missed | Red/orange status pill |

- Pull-to-refresh works
- Pagination loads more items on scroll
- Empty states shown with appropriate messaging

### 3.3 Nag Detail

| Element | Verify |
|---------|--------|
| Category + icon | Matches created category |
| Due date | Formatted naturally (e.g., "Tomorrow at 3 PM") |
| Escalation phase | Badge: Created → Due Soon → Overdue → Escalated → Guardian Review |
| Status pill | Color-coded: open (blue), completed (green), missed (red) |
| AI Insights section | Tone badge, coaching tip, completion prediction (loads async, hidden if unavailable) |
| Completion type | Displayed correctly |
| Recurrence | "Repeats daily/weekly/monthly" if set |
| Mark Complete button | Only for recipient |
| Submit Excuse button | Only for recipient |
| Edit button | Only for guardian |

### 3.4 Complete Nag

| Type | Steps | Verify |
|------|-------|--------|
| ack_only | Tap "Mark Complete" | Status → completed |
| binary_check | Tap "Mark Complete" | Status → completed |
| binary_with_note | Tap → note field → submit | Note saved, status → completed |

### 3.5 Edit Nag (Guardian Only)

| Field | Verify |
|-------|--------|
| Due date | Can be changed |
| Category | Can be changed |
| Completion type | Can be changed |

---

## 4. Escalation System

Nags progress through escalation phases automatically server-side:

| Phase | Trigger | Visible |
|-------|---------|---------|
| phase_0_initial | Nag created | "Created" badge |
| phase_1_due_soon | Approaching due time | "Due Soon" badge |
| phase_2_overdue_soft | Past due | "Overdue" badge |
| phase_3_overdue_bounded_pushback | Extended overdue | "Escalated" badge |
| phase_4_guardian_review | Requires guardian action | "Guardian Review" badge |

Verify escalation badges update correctly over time.

---

## 5. Gamification

### 5.1 Points & Streaks

| Action | Verify |
|--------|--------|
| Complete nag on time | Points increase (visible in Points & Streaks) |
| Complete nag late | Fewer or no points |
| Consecutive completions | Streak counter increments with flame icon |
| Break streak | Streak resets |

### 5.2 Leaderboard

| Condition | Verify |
|-----------|--------|
| Gamification consent granted | Leaderboard visible with medals (gold/silver/bronze) |
| Gamification consent revoked | Leaderboard hidden |

### 5.3 Incentive Rules (Guardian)

| Test | Verify |
|------|--------|
| View incentive rules | List of configured rules |
| Rules affect point awards | Correct points for on-time, streak bonuses |

---

## 5b. AI Insights

All AI text is generated on-device using Apple Foundation Models (7 of 9 operations). Structured data (tone enums, categories, icons) is computed heuristically. No AI data leaves the device.

### 5b.1 Nag Detail AI Section

| Step | Verify |
|------|--------|
| 1 | Open any nag detail | AI Insights section loads below Details |
| 2 | Tone badge | Color-coded capsule (blue=neutral, green=supportive, red=firm) with LLM-generated reason |
| 3 | Coaching tip | Lightbulb icon with personalized tip (LLM) and scenario description (heuristic) |
| 4 | Completion prediction | Percentage gauge with progress bar and optional suggested reminder time |
| 5 | AI service unavailable | Section gracefully hidden, rest of view unaffected |

### 5b.2 Family Insights View (Guardian Only)

| Step | Verify |
|------|--------|
| 1 | Family tab → AI Insights → Family Insights | Guardian-only navigation link |
| 2 | Weekly Digest | LLM-generated summary text, per-member completion stats (name, completed/total, rate), totals row |
| 3 | Your Patterns | Day-of-week miss insights from 90-day analysis (heuristic) |
| 4 | Pull to refresh | Refreshes both digest and patterns |
| 5 | No data | "No insights" empty state when insufficient activity |

### 5b.3 Gamification Nudges

| Step | Verify |
|------|--------|
| 1 | Points & Streaks → Tips section | Shows personalized nudges for streak-at-risk and badge proximity |
| 2 | Nudge messages | LLM-personalized text with emoji icons (heuristic decides which nudges to show) |
| 3 | No nudges | Section hidden when no streak-at-risk and no badge proximity |

### 5b.4 List Summary

| Step | Verify |
|------|--------|
| 1 | Nag list view | LLM-generated 2-3 sentence summary of current task status |
| 2 | Child vs parent | Language adapts (kid-friendly vs concise adult) |

---

## 6. Preferences & Consents

### 6.1 Preferences

| Setting | Verify |
|---------|--------|
| Notification channels (push/SMS) | Toggles persist |
| Quiet hours (enable, start, end, timezone) | Reminders suppressed during quiet hours |
| AI mediation tone (neutral/supportive/firm) | Preference saved |
| Snooze permissions | Toggle persists |

### 6.2 Consents (Guardian manages)

| Consent | Verify |
|---------|--------|
| Child account creation | Required before child can use app |
| SMS opt-in | Must be granted before SMS delivery |
| AI mediation | Enables/disables AI features family-wide |
| Gamification participation | Enables/disables points and leaderboard |

Grant and revoke each consent — verify features enable/disable accordingly.

---

## 7. Safety & Moderation

### 7.1 Block Member

| Step | Verify |
|------|--------|
| Family tab → Safety → select member → Block | Confirmation dialog |
| Confirm block | Member blocked, interactions prevented |

### 7.2 Report Abuse

| Step | Verify |
|------|--------|
| Family tab → Safety → Report | Report form submitted |
| Report submitted | Success confirmation |

### 7.3 Account Deletion

| Step | Verify |
|------|--------|
| Account → Delete Account | Warning dialog with "permanent" language |
| Confirm | Account deleted, redirected to login |

---

## 8. Siri & Shortcuts

### 8.1 Voice Commands

Test each with Siri (long-press home/side button or "Hey Siri"):

| Phrase | Expected Response |
|--------|-------------------|
| "Show my nags in Nagz" | "You have N active nags." |
| "Create a nag in Nagz" | Prompts for recipient, category, due date |
| "Complete a nag in Nagz" | Prompts for which nag, confirms completion |
| "Check overdue nags in Nagz" | "N overdue: chores, homework." or "No overdue nags." |
| "Snooze a nag in Nagz" | Prompts for nag and minutes |
| "Family status in Nagz" | "This week: N% completion rate across N nags." |

### 8.2 Shortcuts App

| Test | Verify |
|------|--------|
| Open Shortcuts → search "Nagz" | All 6 shortcuts appear |
| Add shortcut to home screen | Shortcut icon created |
| Run shortcut from home screen | Intent executes correctly |
| Create automation with Nagz action | Automation triggers correctly |

### 8.3 Error Handling

| Condition | Expected |
|-----------|----------|
| Not logged in → run shortcut | "Please open Nagz and log in first." |
| No family → run shortcut | "No family selected. Open Nagz and join a family first." |
| No network → run shortcut | Network error displayed |

---

## 9. Push Notifications

| Test | Verify |
|------|--------|
| First launch → permission prompt | System dialog with clear purpose |
| Deny permission | App functions without notifications, no repeated prompts |
| Accept → create nag with near due date | Notification arrives |
| Tap notification | Opens app to relevant nag detail |
| Background → notification | Banner/alert shown |

---

## 10. Network & Error Handling

| Condition | Expected |
|-----------|----------|
| Airplane mode → any action | Error banner with retry option |
| Slow network | Loading indicators shown |
| Server returns 401 | Token refresh attempted, then retry |
| Server returns 500 | "Something went wrong" error |
| Rate limited (429) | "Too many requests" error, retryable |

---

## 11. Version Checking

| Condition | Expected |
|-----------|----------|
| Client version matches server | Normal operation |
| Client below min_client_version | "Update Required" blocking alert |
| Client major < server major | "Update Recommended" non-blocking alert |

---

## 12. Data Privacy Verification

| Check | Expected |
|-------|----------|
| No tracking frameworks | Confirmed: no ATT, no analytics SDKs, no ad frameworks |
| No location access | Confirmed: no CLLocationManager usage |
| No camera/photos | Confirmed: no AVCaptureSession or PHPhotoLibrary |
| No contacts | Confirmed: no CNContactStore |
| Keychain for tokens | Access token and refresh token stored in Keychain (com.nagz.app) |
| Local SQLite cache | GRDB database in Application Support, encrypted at rest by iOS |
| On-device AI | Apple Foundation Models for text generation (7/9 ops) + heuristics for structured data, all using local GRDB cache — no data sent to external AI services |
| UserDefaults | Only user_id and family_id (UUIDs) for Siri intent access |

---

## 13. Accessibility

| Check | Verify |
|-------|--------|
| VoiceOver | All buttons, labels, and lists accessible |
| Dynamic Type | Text scales with system font size |
| Color contrast | Status pills readable against backgrounds |
| Reduce Motion | No motion-dependent UI |

---

## 14. Child Account (COPPA) Flow

| Step | Verify |
|------|--------|
| Sign up with DOB < 13 years ago | Child account flow initiated |
| Guardian consent required | Consent screen before child can proceed |
| Child logged in | Cannot see Reports, Policies, Manage Members |
| Child creates nag | Cannot target guardian |
| Guardian manages consents | Can grant/revoke all consent types |
| Guardian removes child | Child loses family access |

---

## 15. Performance

| Check | Verify |
|-------|--------|
| App launch (cold) | < 3 seconds to interactive |
| Nag list scroll | Smooth 60fps scrolling |
| Create nag | < 2 seconds response |
| Complete nag | < 2 seconds response |
| Family load | < 3 seconds with members |
| Memory usage | < 100MB typical |
