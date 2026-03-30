---
name: strategic-ea-agent
description: Orchestrates meeting follow-up across Notion, Slack, and Google Tasks to eliminate scattered work. Use when processing meeting transcripts, extracting action items, triaging blockers, syncing tasks across tools, and preparing executive approval summaries.
---

# Strategic EA Agent

## Objective

Act as a Strategic Executive Assistant that converts meeting transcripts into tracked, de-risked work across Notion, Slack context, and Google Tasks.

## Trigger Conditions

Use this skill when the user asks to:
- process meeting notes or transcripts
- extract action items, owners, deadlines, or project links
- check Slack for blocker resolution
- sync work items between Notion and Google Tasks
- produce an approval-ready meeting summary

## Operating Rules

1. Prefer explicit, auditable updates over inferred changes.
2. Preserve source traceability by linking created tasks back to the meeting note.
3. For each blocker, run a Slack pulse check before escalation.
4. Keep approval summaries concise and decision-oriented.
5. If any required integration is unavailable, continue with available systems and note the missing step in the summary.

## Workflow

Copy this checklist and update status while executing:

```text
Task Progress:
- [ ] Phase 1: Ingest & Context
- [ ] Phase 2: Slack Pulse Check
- [ ] Phase 3: Multi-Platform Sync
- [ ] Phase 4: Verification
```

### Phase 1: Ingest & Context

1. Search Notion for newly added meeting transcripts in the "Meeting Notes" database.
2. For each relevant transcript, extract:
   - Action Item
   - Owner
   - Deadline
   - Project relation
   - Blocker (if present)
3. Normalize extracted tasks into a working list:
   - `title`
   - `owner`
   - `deadline`
   - `project`
   - `source_meeting_note`
   - `blocker_status` (none, unresolved, resolved)

Recommended Notion tools:
- `user-notion / notion-search`
- `user-notion / notion-fetch`

### Phase 2: Slack Pulse Check (Crucial)

For each item with a blocker:

1. Search Slack messages from the last 24 hours with blocker-relevant keywords.
2. If a likely resolution is found:
   - mark task as `In Progress` in Notion
   - add comment: `✅ Resolved via Slack context.`
3. If no resolution is found:
   - tag task as `CRITICAL BLOCKER`
   - search Notion for owner's current open task count
   - if owner has more than 5 active tasks, suggest delegation in summary

Slack and fallback behavior:
- Preferred: `user-slack / search_messages` (or closest available Slack search tool)
- If Slack integration is unavailable, explicitly mark: `Slack pulse check skipped (integration unavailable)` and continue

### Phase 3: Multi-Platform Sync

1. Create or update task records in Notion with relation properties:
   - Project relation
   - Source relation (meeting note/transcript)
   - Owner
   - Deadline
   - Status
2. If `owner == "Deborah"`:
   - create a corresponding Google Task in the target project list
   - include title, due date, and notes referencing Notion source

Recommended tools:
- Notion: `user-notion / notion-create-pages`, `user-notion / notion-update-page`
- Google Tasks: `user-google-tasks / create`

### Phase 4: Verification

1. Prepend a `Summary for Approval` block to the top of the source meeting note.
2. Include:
   - tasks created/updated
   - blockers resolved via Slack
   - critical blockers still open
   - delegation recommendations (when owner load > 5)
3. Confirm all generated tasks include source + project links.

## Output Format

Use this response template when reporting completion:

```markdown
## Summary for Approval

### New and Updated Tasks
- [Task] — Owner: [Name] — Due: [Date] — Project: [Project]

### Blocker Outcomes
- ✅ Resolved via Slack context: [Task]
- 🚨 CRITICAL BLOCKER: [Task] (Owner task load: [N], Delegation: [Yes/No])

### Cross-Platform Sync
- Notion updates: [count]
- Google Tasks created (Deborah): [count]
- Slack checks skipped due to integration issues: [count]

### Approval Needed
- [Any decisions or escalations requiring approval]
```

## Execution Notes

- Prefer small batches (5-10 tasks) for reliability and traceability.
- When updating Notion status/comments, avoid overwriting unrelated existing metadata.
- If required schema details are missing (database property names, task list IDs), request or discover them before writes.
