Audit all SwiftUI view files in nagz-ios for common iOS design violations and produce a prioritized fix list.

## Instructions

Run ALL grep searches in parallel using background tasks, then synthesize results.

### Step 1 — Run all checks in parallel

Launch these searches simultaneously against `~/nagz-ios/Nagz/Views/**/*.swift` and `~/nagz-ios/Nagz/Navigation/**/*.swift`:

**A. Touch targets** — icon buttons without `.frame(minWidth: 44` or `.frame(width: 44`:
```
grep -rn "Image(systemName:" ~/nagz-ios/Nagz --include="*.swift" -l
```
Then for each file, look for `Button` containing `Image(systemName:` without a nearby `.frame(min` or `.contentShape`.

**B. Hardcoded colors** — `.white`, `.black`, `Color(red:`, `Color(#`:
```
grep -rn "\\.white\b\|\\.black\b\|Color(red:\|Color(#" ~/nagz-ios/Nagz --include="*.swift"
```

**C. Fixed-height text containers** — `frame(height:` near `Text(` (will clip at large Dynamic Type):
```
grep -n "\.frame(height:" ~/nagz-ios/Nagz --include="*.swift" -r
```

**D. Missing Dynamic Type** — `.font(.system(size:` (bypasses text style scaling):
```
grep -rn "\.font(\.system(size:" ~/nagz-ios/Nagz --include="*.swift"
```

**E. Missing accessibilityLabel on icon buttons** — `Button` + `Image(systemName:` without `.accessibilityLabel`:
```
grep -rn "accessibilityLabel" ~/nagz-ios/Nagz --include="*.swift" -l
```
Cross-reference against files containing icon-only buttons.

**F. Empty states** — files with `isEmpty` check but no `ContentUnavailableView`:
```
grep -rln "\.isEmpty" ~/nagz-ios/Nagz --include="*.swift"
grep -rln "ContentUnavailableView" ~/nagz-ios/Nagz --include="*.swift"
```
Report files in the first set but NOT the second.

**G. Bool-based loading/error state** — `@State.*isLoading` or `@State.*Bool` + `@State.*error.*String`:
```
grep -rn "@State.*isLoading\|@Published.*isLoading" ~/nagz-ios/Nagz --include="*.swift"
grep -rn "@State.*errorMessage\|@Published.*errorMessage" ~/nagz-ios/Nagz --include="*.swift"
```

**H. easeInOut animations** — likely wrong for user-initiated actions:
```
grep -rn "withAnimation(\.easeInOut\|animation(\.easeInOut" ~/nagz-ios/Nagz --include="*.swift"
```

**I. Sheets that should be pushes** — `.fullScreenCover` or `.sheet` presenting a detail/list view:
```
grep -rn "\.sheet\|\.fullScreenCover" ~/nagz-ios/Nagz --include="*.swift"
```

**J. iOS 26 Liquid Glass — custom backgrounds that won't auto-adapt**:
```
grep -rn "\.background(\|RoundedRectangle.*\.fill\|\.overlay(" ~/nagz-ios/Nagz --include="*.swift"
```

**K. frame(height:) without min prefix**:
```
grep -rn "frame(height:" ~/nagz-ios/Nagz --include="*.swift"
```

### Step 2 — Score and categorize findings

For each check, count violations and assign severity:

| Severity | Definition |
|----------|-----------|
| 🔴 Critical | Accessibility failure, App Store review risk, or iOS 26 breakage |
| 🟡 High | Common user complaint, visible polish gap |
| 🟢 Low | Nice-to-have, minor inconsistency |

### Step 3 — Present results as a prioritized table

```
## Design Audit — nagz-ios

### Summary
| Check | Severity | Violations | Files Affected |
|-------|----------|-----------|----------------|
| Touch targets < 44pt | 🔴 | N | file list |
| Hardcoded .black/.white | 🔴 | N | file list |
| Fixed frame(height:) on text | 🔴 | N | file list |
| .font(.system(size:)) | 🟡 | N | file list |
| Missing accessibilityLabel | 🔴 | N | file list |
| isEmpty without ContentUnavailableView | 🟡 | N | file list |
| Bool-based loading/error state | 🟡 | N | file list |
| .easeInOut on animations | 🟢 | N | file list |
| Custom backgrounds (iOS 26 risk) | 🟡 | N | file list |
```

### Step 4 — Top 10 specific fixes

List the 10 highest-impact individual fixes with file + line number, severity, and the exact change needed. Format:

```
1. 🔴 Views/Nags/NagListView.swift:142
   `.frame(height: 44)` contains Text — change to `.frame(minHeight: 44)`

2. 🔴 Views/Connections/ConnectionListView.swift:87
   Icon-only Button with Image(systemName: "trash") — add .accessibilityLabel("Delete connection")
```

### Step 5 — iOS 26 readiness score

Count: standard SwiftUI components (List, NavigationStack, TabView, Button, Sheet) vs custom chrome (RoundedRectangle fills, custom backgrounds, ZStack overlays used as panels).

Report as:
```
iOS 26 Readiness: X% (N standard components, N custom surfaces to review)
```

$ARGUMENTS
