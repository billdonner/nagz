# Siri & Shortcuts Integration (V2.0)

## 1. Overview

Nagz exposes key nag operations to Siri, Shortcuts, and Apple Intelligence via the **App Intents** framework (iOS 16+). Users can create nags, mark them complete, check status, and snooze — all by voice or Shortcuts automation.

**Framework choice**: App Intents (not the deprecated SiriKit/INIntent). App Intents is the only path forward for Siri integration, Apple Intelligence schemas, interactive snippets, and Control Center widgets.

**Minimum target**: iOS 26 (matches current Nagz deployment target). Interactive snippets are available on all supported devices.

## 2. Design Principles

- **No new server endpoints.** All intents use existing API surface (`/nags`, `/families`, `/preferences`).
- **Auth reuse.** Intents share the app's keychain tokens — no separate login flow.
- **Role-aware.** Intents respect `FamilyRole` permissions (children can't create nags, guardians see all nags).
- **Offline-safe.** Intents that read data try the GRDB cache first, falling back to server.
- **Privacy-first.** No nag content is sent to Apple. Entities expose only titles and categories, not descriptions or excuse text.

## 3. Entities

### 3.1 NagEntity

```swift
struct NagEntity: AppEntity {
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "Nag"
    static var defaultQuery = NagEntityQuery()

    var id: String                          // UUID string
    @Property(title: "Category") var category: String
    @Property(title: "Status") var status: String
    @Property(title: "Due") var dueAt: Date
    @Property(title: "Recipient") var recipientName: String

    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(category) — \(recipientName)",
            subtitle: "Due \(dueAt.formatted(.relative(presentation: .named)))"
        )
    }
}
```

### 3.2 FamilyMemberEntity

```swift
struct FamilyMemberEntity: AppEntity {
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "Family Member"
    static var defaultQuery = FamilyMemberQuery()

    var id: String                           // userId UUID string
    @Property(title: "Name") var displayName: String
    @Property(title: "Role") var role: String

    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(displayName)", subtitle: "\(role)")
    }
}
```

### 3.3 NagCategoryEnum

```swift
enum NagCategoryAppEnum: String, AppEnum {
    case chores, meds, homework, appointments, other

    static var typeDisplayRepresentation: TypeDisplayRepresentation = "Category"
    static var caseDisplayRepresentations: [Self: DisplayRepresentation] = [
        .chores: "Chores",
        .meds: "Meds",
        .homework: "Homework",
        .appointments: "Appointments",
        .other: "Other",
    ]
}
```

## 4. Entity Queries

### 4.1 NagEntityQuery

```swift
struct NagEntityQuery: EntityStringQuery {
    @Dependency var apiClient: APIClient

    // Resolve by ID (used by Shortcuts parameter binding)
    func entities(for identifiers: [String]) async throws -> [NagEntity] {
        // Fetch each nag by UUID, map to NagEntity
    }

    // Free-text search ("homework nags", "meds for John")
    func entities(matching string: String) async throws -> [NagEntity] {
        // Fetch open nags, filter by category/recipient name containing string
    }

    // Suggestions shown in Shortcuts editor
    func suggestedEntities() async throws -> [NagEntity] {
        // Return open nags for the current user, limit 10
    }
}
```

### 4.2 FamilyMemberQuery

```swift
struct FamilyMemberQuery: EntityStringQuery {
    @Dependency var apiClient: APIClient

    func entities(for identifiers: [String]) async throws -> [FamilyMemberEntity]
    func entities(matching string: String) async throws -> [FamilyMemberEntity]
    func suggestedEntities() async throws -> [FamilyMemberEntity]
}
```

## 5. Intents

### 5.1 Intent Summary

| Intent | Siri Phrase | Input | Output | Role |
|--------|------------|-------|--------|------|
| `CreateNagIntent` | "Create a nag in Nagz" | recipient, category, due time, description | Created nag entity + dialog | guardian, participant |
| `CompleteNagIntent` | "Mark my nag as done in Nagz" | nag (entity) | Confirmation dialog | recipient only |
| `ListNagsIntent` | "Show my nags in Nagz" | (optional) category filter | Nag list + count dialog | any member |
| `CheckOverdueIntent` | "What's overdue in Nagz" | — | Overdue nag list + dialog | any member |
| `SnoozeNagIntent` | "Snooze my nag in Nagz" | nag (entity), minutes | Confirmation dialog | recipient only |
| `FamilyStatusIntent` | "How's my family doing in Nagz" | — | Completion rates dialog | guardian |

### 5.2 CreateNagIntent

```swift
struct CreateNagIntent: AppIntent {
    static var title: LocalizedStringResource = "Create a Nag"
    static var description = IntentDescription("Create a new nag for a family member.")

    @Parameter(title: "For") var recipient: FamilyMemberEntity
    @Parameter(title: "Category") var category: NagCategoryAppEnum
    @Parameter(title: "Due") var dueAt: Date
    @Parameter(title: "Description") var description: String?

    static var parameterSummary: some ParameterSummary {
        Summary("Create a \(\.$category) nag for \(\.$recipient) due \(\.$dueAt)") {
            \.$description
        }
    }

    func perform() async throws -> some IntentResult & ProvidesDialog {
        try requireAuth()
        try requireRole([.guardian, .participant])

        let nag = NagCreate(
            familyId: currentFamilyId(),
            recipientId: UUID(uuidString: recipient.id)!,
            dueAt: dueAt,
            category: NagCategory(rawValue: category.rawValue)!,
            doneDefinition: .ackOnly,
            description: description
        )
        let created: NagResponse = try await apiClient.request(.createNag(nag))
        return .result(dialog: "Created \(category.rawValue) nag for \(recipient.displayName), due \(dueAt.formatted()).")
    }
}
```

### 5.3 CompleteNagIntent

```swift
struct CompleteNagIntent: AppIntent {
    static var title: LocalizedStringResource = "Complete a Nag"
    static var description = IntentDescription("Mark a nag as completed.")

    @Parameter(title: "Nag") var nag: NagEntity
    @Parameter(title: "Note") var note: String?

    static var parameterSummary: some ParameterSummary {
        Summary("Mark \(\.$nag) as done") {
            \.$note
        }
    }

    func perform() async throws -> some IntentResult & ProvidesDialog {
        try requireAuth()
        try await apiClient.requestVoid(
            .updateNagStatus(nagId: UUID(uuidString: nag.id)!, status: .completed, note: note)
        )
        return .result(dialog: "Done! Marked \(nag.category) nag as completed.")
    }
}
```

### 5.4 ListNagsIntent

```swift
struct ListNagsIntent: AppIntent {
    static var title: LocalizedStringResource = "Show My Nags"
    static var description = IntentDescription("List your active nags.")

    @Parameter(title: "Category") var category: NagCategoryAppEnum?

    func perform() async throws -> some IntentResult & ReturnsValue<[NagEntity]> & ProvidesDialog {
        try requireAuth()
        let nags = try await fetchOpenNags(category: category?.rawValue)
        if nags.isEmpty {
            return .result(value: [], dialog: "You have no active nags. Nice work!")
        }
        return .result(value: nags, dialog: "You have \(nags.count) active nag\(nags.count == 1 ? "" : "s").")
    }
}
```

### 5.5 CheckOverdueIntent

```swift
struct CheckOverdueIntent: AppIntent {
    static var title: LocalizedStringResource = "Check Overdue Nags"

    func perform() async throws -> some IntentResult & ProvidesDialog {
        try requireAuth()
        let overdue = try await fetchOpenNags().filter { $0.dueAt < Date() }
        if overdue.isEmpty {
            return .result(dialog: "Nothing overdue. You're all caught up!")
        }
        let summary = overdue.map { "\($0.category)" }.joined(separator: ", ")
        return .result(dialog: "\(overdue.count) overdue: \(summary).")
    }
}
```

### 5.6 SnoozeNagIntent

```swift
struct SnoozeNagIntent: AppIntent {
    static var title: LocalizedStringResource = "Snooze a Nag"

    @Parameter(title: "Nag") var nag: NagEntity
    @Parameter(title: "Minutes", default: 15) var minutes: Int

    static var parameterSummary: some ParameterSummary {
        Summary("Snooze \(\.$nag) for \(\.$minutes) minutes")
    }

    func perform() async throws -> some IntentResult & ProvidesDialog {
        try requireAuth()
        // PATCH nag due_at forward by minutes
        let newDue = nag.dueAt.addingTimeInterval(Double(minutes) * 60)
        let update = NagUpdate(dueAt: newDue)
        try await apiClient.requestVoid(.updateNag(nagId: UUID(uuidString: nag.id)!, update: update))
        return .result(dialog: "Snoozed for \(minutes) minutes. New due time: \(newDue.formatted(date: .omitted, time: .shortened)).")
    }
}
```

### 5.7 FamilyStatusIntent

```swift
struct FamilyStatusIntent: AppIntent {
    static var title: LocalizedStringResource = "Family Status"
    static var description = IntentDescription("Check how your family is doing on nags this week.")

    func perform() async throws -> some IntentResult & ProvidesDialog {
        try requireAuth()
        try requireRole([.guardian])
        let report = try await apiClient.request(.weeklyReport(familyId: currentFamilyId()))
        return .result(dialog: "This week: \(report.completionRate)% completion rate across \(report.totalNags) nags.")
    }
}
```

## 6. Shortcuts Provider

```swift
struct NagzShortcutsProvider: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: ListNagsIntent(),
            phrases: [
                "Show my nags in \(.applicationName)",
                "What are my nags in \(.applicationName)",
                "Check \(.applicationName)",
            ],
            shortTitle: "My Nags",
            systemImageName: "bell.fill"
        )
        AppShortcut(
            intent: CreateNagIntent(),
            phrases: [
                "Create a nag in \(.applicationName)",
                "New nag in \(.applicationName)",
                "Remind someone in \(.applicationName)",
            ],
            shortTitle: "New Nag",
            systemImageName: "plus.circle.fill"
        )
        AppShortcut(
            intent: CompleteNagIntent(),
            phrases: [
                "Mark my nag as done in \(.applicationName)",
                "Complete a nag in \(.applicationName)",
                "I did it in \(.applicationName)",
            ],
            shortTitle: "Complete Nag",
            systemImageName: "checkmark.circle.fill"
        )
        AppShortcut(
            intent: CheckOverdueIntent(),
            phrases: [
                "What's overdue in \(.applicationName)",
                "Am I behind in \(.applicationName)",
            ],
            shortTitle: "Overdue",
            systemImageName: "exclamationmark.triangle.fill"
        )
        AppShortcut(
            intent: FamilyStatusIntent(),
            phrases: [
                "How's my family doing in \(.applicationName)",
                "Family report in \(.applicationName)",
            ],
            shortTitle: "Family Status",
            systemImageName: "person.3.fill"
        )
    }
}
```

**Phrase count**: 14 phrases total (well under 1,000 limit).

## 7. Authentication & Authorization

### 7.1 Auth Guard

All intents call a shared `requireAuth()` helper:

```swift
func requireAuth() throws {
    guard KeychainService.shared.accessToken != nil else {
        throw NagzIntentError.notLoggedIn
    }
}

func requireRole(_ allowed: [FamilyRole]) throws {
    let myRole = // fetch from cached member list
    guard allowed.contains(myRole) else {
        throw NagzIntentError.notPermitted(myRole)
    }
}

enum NagzIntentError: Error, CustomLocalizedStringResourceConvertible {
    case notLoggedIn
    case notPermitted(FamilyRole)
    case noFamily

    var localizedStringResource: LocalizedStringResource {
        switch self {
        case .notLoggedIn:
            return "Please open Nagz and log in first."
        case .notPermitted(let role):
            return "Your role (\(role.rawValue)) doesn't have permission for this action."
        case .noFamily:
            return "No family selected. Open Nagz and join a family first."
        }
    }
}
```

### 7.2 Keychain Sharing

Intents run in the app's process (not an extension), so they share the same keychain automatically. No App Groups or keychain sharing entitlement needed.

### 7.3 Token Refresh

The existing `APIClient` actor handles 401 → token refresh transparently. Intents that call `apiClient.request()` get this for free.

## 8. Privacy & Safety

| Concern | Mitigation |
|---------|-----------|
| Nag descriptions exposed to Siri | Entities show category + recipient name only, never description or excuse text |
| Child creates nag via Siri | `requireRole([.guardian, .participant])` blocks children |
| Unauthenticated access | `requireAuth()` check in every intent |
| Lock screen access | Default `authenticationPolicy` requires device unlock |
| Shortcut sharing | Shortcuts that contain family data are not shareable (entity IDs are UUIDs) |
| Apple Intelligence training | App Intents metadata (titles, categories) is on-device only; no content sent to Apple |

## 9. Apple Intelligence Integration (iOS 18+)

When targeting iOS 18+, annotate intents with Assistant Schemas so Apple Intelligence can reason about them:

```swift
@AssistantIntent(schema: .system.search)
struct ListNagsIntent: AppIntent { ... }
```

Available schema mappings for Nagz:

| Intent | Schema | Benefit |
|--------|--------|---------|
| `ListNagsIntent` | `.system.search` | "Find my homework nags" natural language |
| `CreateNagIntent` | `.system.createEntity` | Siri proactively suggests creation |
| `CompleteNagIntent` | `.system.updateEntity` | Status update via natural language |

For iOS 26+, use the `@AppIntent(schema:)` macro instead of `@AssistantIntent`.

## 10. Interactive Snippets (iOS 26+)

When the deployment target moves to iOS 26, intents can return SwiftUI views:

```swift
struct ListNagsIntent: AppIntent {
    func perform() async throws -> some IntentResult & ShowsSnippetIntent {
        return .result(snippetIntent: NagListSnippet())
    }
}

struct NagListSnippet: SnippetIntent {
    var body: some View {
        VStack(spacing: 8) {
            ForEach(nags) { nag in
                HStack {
                    Text(nag.category)
                    Spacer()
                    Button(intent: CompleteNagIntent(nag: nag.entity)) {
                        Image(systemName: "checkmark.circle")
                    }
                }
            }
        }
    }
}
```

Users can mark nags complete directly from the Siri snippet without opening the app.

## 11. Shortcuts Automation Examples

Power users can build automations in the Shortcuts app:

| Automation | Trigger | Actions |
|-----------|---------|---------|
| Morning check | Time of Day (7:00 AM) | `ListNagsIntent` → Read aloud |
| Homework reminder | Arrive at home (location) | `CreateNagIntent(category: .homework, recipient: child)` |
| End of day report | Time of Day (9:00 PM) | `FamilyStatusIntent` → Notification |
| Auto-snooze | Focus mode (Do Not Disturb) | `SnoozeNagIntent(minutes: 30)` for all open nags |

## 12. File Structure

```
Nagz/
├── Intents/
│   ├── Entities/
│   │   ├── NagEntity.swift
│   │   ├── FamilyMemberEntity.swift
│   │   └── NagCategoryAppEnum.swift
│   ├── Queries/
│   │   ├── NagEntityQuery.swift
│   │   └── FamilyMemberQuery.swift
│   ├── Actions/
│   │   ├── CreateNagIntent.swift
│   │   ├── CompleteNagIntent.swift
│   │   ├── ListNagsIntent.swift
│   │   ├── CheckOverdueIntent.swift
│   │   ├── SnoozeNagIntent.swift
│   │   └── FamilyStatusIntent.swift
│   ├── NagzShortcutsProvider.swift
│   └── NagzIntentError.swift
```

All files live in the main app target — no extension target needed (App Intents runs in-process).

## 13. project.yml Changes

```yaml
targets:
  Nagz:
    settings:
      base:
        # Add App Intents capability (no entitlement file change needed)
        INFOPLIST_KEY_NSSupportsLiveActivities: false
    sources:
      - path: Nagz
        includes:
          - "**/*.swift"  # Already catches Intents/ subdirectory
    dependencies:
      - package: KeychainAccess
      - package: GRDB
      # No new dependencies — App Intents is a system framework
```

## 14. Implementation Order

| Step | Description | Estimate |
|------|-------------|----------|
| 1 | Create `Intents/` directory structure, entities, enums | Small |
| 2 | Implement `NagEntityQuery` + `FamilyMemberQuery` against APIClient | Small |
| 3 | Implement `ListNagsIntent` + `CheckOverdueIntent` (read-only, lowest risk) | Small |
| 4 | Implement `CompleteNagIntent` + `SnoozeNagIntent` (mutations) | Small |
| 5 | Implement `CreateNagIntent` (most complex — parameter resolution) | Medium |
| 6 | Implement `FamilyStatusIntent` (guardian-only) | Small |
| 7 | Add `NagzShortcutsProvider` with all phrases | Small |
| 8 | Add `@AssistantIntent` schemas (iOS 18+ only) | Small |
| 9 | Tests: unit test each intent's `perform()` with mock APIClient | Medium |
| 10 | Interactive snippets (iOS 26+ gate) | Deferred |

## 15. Testing Strategy

| Layer | Approach |
|-------|----------|
| Entity mapping | Unit test NagResponse → NagEntity conversion |
| Query resolution | Unit test with mock API responses |
| Intent perform() | Unit test with mock APIClient, verify correct endpoint called |
| Auth guards | Test requireAuth/requireRole throw correct errors |
| Role enforcement | Test child can't create, guardian can see family status |
| Siri phrases | Manual testing on device (no simulator support for Siri) |
| Shortcuts app | Manual testing of automation flows |

## 16. Cross-Spec Linkage

- Nag operations: `API_SURFACE.md` sections 8-9
- Role permissions: `POLICY_MATRIX.md`
- AI features exposed via Siri: `AI_ARCHITECTURE.md` (future — expose coaching/patterns as intents)
- Privacy: `SAFETY_AND_COMPLIANCE.md` section 7
- Gamification via Siri: `GAMIFICATION.md` (future — "What's my streak in Nagz")
