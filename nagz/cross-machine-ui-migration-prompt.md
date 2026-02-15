# Prompt for External AI/Codex: Spec-Driven UI Proposal + SwiftUI Paging Plan

You are working in the `nagz` repository on https://github.com/billdonner/nagz

## Mandatory workflow
1. Read these files fully before proposing anything:
- `README.md`
- Every document listed in `nagz/Docs/CATALOG.md` (treat the catalog as authoritative and complete it fully before proceeding)

2. Use this explicit checklist to confirm coverage:
- `README.md`
- `nagz/Docs/CATALOG.md`
- `nagz/Docs/README.md`
- `nagz/Docs/REQUIREMENTS.md`
- `nagz/Docs/ARCHITECTURE.md`
- `nagz/Docs/PREFERENCES.md`
- `nagz/Docs/POLICY_MATRIX.md`
- `nagz/Docs/SAFETY_AND_COMPLIANCE.md`
- `nagz/Docs/GLOSSARY.md`
- `nagz/Docs/API_SURFACE.md`
- `nagz/Docs/AI_BEHAVIOR.md`
- `nagz/Docs/INCENTIVES.md`
- `nagz/Docs/GAMIFICATION.md`
- `nagz/Docs/PARENT_GUARDIAN_USER_MANUAL.md`
- `nagz/Docs/SPEC_BASELINE_CHANGELOG.md`
- `nagz/Docs/web-samples/parent-guardian-manual.html`
- `nagz/Docs/web-samples/guardian-portal.html`

3. Do **not** edit code yet.
4. Ask clarifying questions first.
5. Wait for answers.
6. Then provide a proposal only.

## Hard constraints
- This is a **proposal-only task**. Do not modify files, do not run write operations, do not generate patches.
- For web UI outputs, provide **static mockups only** (no requirement for running app code).
- Swift app direction must target **SwiftUI `TabView` paging** for a gallery experience.

## Objectives
1. Summarize key product and policy constraints that the UI must reflect.
2. Suggest a web-based UI direction aligned to the specs.
3. Provide sample screen concepts (static mockups) for the major flows.
4. Propose how the current Swift app should be adapted into a paged gallery of all web samples using SwiftUI `TabView`.

## What to produce (after Q&A)
### A) Spec-aligned UX summary
- 8-12 bullets linking UI choices to requirements/policy constraints.

### B) Web UI concept
- Information architecture (main sections and navigation model).
- Visual direction (tone, typography, color approach, component style).
- Accessibility notes (contrast, text size, touch targets, motion preferences).

### C) Static mockup set
Provide at least these screen concepts with concise annotations:
1. Guardian dashboard
2. Create nag flow
3. Recipient task detail + excuse submission
4. AI mediation summary view
5. Incentives/consequences settings view
6. Gamification summary view
7. Safety controls (block/mute/report)
8. Family reports overview

For each screen include:
- Purpose
- Primary actions
- Key fields/components
- Policy/safety constraints shown in UI

### D) SwiftUI migration proposal (no edits)
- Proposed app structure for a paged gallery using `TabView` with `.page` style.
- Proposed model for gallery items (title, description, source mockup reference).
- Proposed file/module layout changes.
- Proposed navigation and state handling.
- Risks and edge cases (loading failures, unavailable samples, offline behavior).

### E) Execution plan (proposal only)
- Step-by-step implementation plan another engineer could execute.
- Include test plan: unit/UI tests and manual validation checklist.

## Clarifying questions to ask before proposing
Ask these first, then pause:
1. Which user journeys should be prioritized first for the mockups?
2. Should mockups reflect existing brand/style, or define a new temporary visual language?
3. Do you want low-fidelity wireframes or mid-fidelity polished static comps?
4. Should the Swift gallery page include captions/spec callouts per sample?
5. Are there existing Swift files/views that must be preserved exactly?

## Output format
- Sectioned markdown.
- Be explicit about assumptions.
- Clearly label any inferred decisions.
- Include a short “Open Questions” section if anything remains unresolved.
