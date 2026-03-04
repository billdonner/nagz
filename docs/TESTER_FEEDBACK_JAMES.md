# Tester Feedback: James (Session 1)

> Internal tester. James is a recipient — his spouse Angela creates nags for him.
> Date: 2026-03-02

---

## Context

James and Angela are a couple. Angela assigns tasks to James. James wants to manage those tasks on his own schedule without being nagged until he's actually late on HIS timeline, not hers.

James is using Connections (not Family) to interact with Angela.

---

## Raw Feedback (paraphrased from iMessage conversation)

### Recipient Experience

- "Build the view where I can track my tasks"
- "I need that. And I want to rearrange when I'm going to get them done"
- "So Angela doesn't bother me. She just checks the app."
- "I need a drag and drop interface where I see when she assigned stuff and then I schedule it"
- "As customer I need a drag and drop interface where I can see my schedule of due dates and I can schedule when my nags will be done"
- "I don't want to be bothered by her unless I miss my stated deadline"
- "There's her due dates but then there are my planned time to do it"

### Family vs Connections Confusion

- "I need an invite code for a family. Am I 1 or 2 families? We have me and Angela and you guys."
- "So need multiple. For Siri. It said do a family."
- "Ok well Siri says join a family. And then I already have 1 family."
- "So you'll need multiple families. I think that you should just have 1 object. You have relationship groups. Like iMessage. You have single chats or group."
- "Well what is a family? Right now I'm confused between doing family or single relationship. Idk if I'm missing features if I don't do family."
- "And now I have 2 families but I can only have 1"

### General Attitude

- "Idk what you mean I'm the customer I'm always right"
- "You can't talk to your customers that way" (re: app pushback on rescheduling)
- "No do it. Send Claude this chat."

---

## Feature Requests Extracted

| # | Feature | James's Words | Priority |
|---|---------|--------------|----------|
| 1 | **Recipient task dashboard** | "Build the view where I can track my tasks" | High |
| 2 | **Commit time ("I'll do it by")** | "There's her due dates but then there are my planned time to do it" | High |
| 3 | **Quiet until commit time missed** | "I don't want to be bothered by her unless I miss my stated deadline" | High |
| 4 | **Reorder/schedule tasks** | "I want to rearrange when I'm going to get them done" | Medium |
| 5 | **Simplify couple onboarding** | "I'm confused between doing family or single relationship" | Medium |
| 6 | **Multiple families** | "We have me and Angela and you guys" | Low (V2) |
| 7 | **Drag and drop** | "I need a drag and drop interface" | Low (nice-to-have) |

---

## Analysis

### Core Insight

The app is currently **guardian/creator-centric**: create nags, assign them, track compliance. James reveals the **recipient experience** is underserved. Recipients need:

1. A view of "what do I owe people" — not mixed with nags they created
2. The ability to commit to their own timeline
3. Escalation that respects their commitment, not just the creator's deadline
4. Minimal interruption until they're actually late

### Design Implications

- **Commit time** is a per-nag recipient action: "I'll do this by Saturday 10am"
- **Quiet until missed** is a global preference (default OFF) that shifts escalation to the recipient's commit time
- **"My Plan" view** is a recipient-focused sort of the nag list: received nags, ordered by commit time or due date, with clear visual distinction between creator's deadline and recipient's plan
- **Family vs Connections confusion** is an onboarding problem, not a data model problem. Connections already handle the James/Angela use case. The app just doesn't make that clear enough.

### What NOT to build (yet)

- Full drag-and-drop reordering — complex, low ROI for V1
- Multiple families — server schema change, defer to V2
- Relationship groups / iMessage-style model — fundamental rearchitecture, not needed

---

## Proposed Implementation

See plan for "Commit Time & Recipient Experience" feature.

| Change | Type | Effort |
|--------|------|--------|
| `committed_at` nullable field on nags | Server schema + API | Small |
| "I'll do it by" action on NagDetailView | iOS UI | Small |
| "Only escalate after my commit time" preference | Server + iOS | Small |
| Scheduler respects commit time for escalation | Server logic | Medium |
| "My Plan" section in nag list (received nags, sorted by commit/due) | iOS UI | Medium |
