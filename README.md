# 🗂️ Strategic Task Tracker

> **Notion AI Challenge submission — Major League Hacking**

An AI-powered Strategic Executive Assistant that turns meeting transcripts and notes into tracked, cross-platform action items — automatically. It reads from Notion, checks Slack for blocker resolution, creates tasks in both Notion and Google Tasks, and writes an approval-ready summary back to your meeting note.

---

## What It Does

You drop a meeting transcript or notes into a Notion database. The agent reads it, extracts every action item, checks Slack for context on blockers, syncs tasks across Notion and Google Tasks, and writes a structured summary back to the source note for your approval.

### The 4-Phase Workflow

```
Phase 1: INGEST & CONTEXT
  → Reads meeting transcripts from your Notion "Meeting Notes" database
  → Extracts: action items, owners, deadlines, project links, blockers

Phase 2: SLACK PULSE CHECK
  → For each blocker, searches Slack messages from the last 24 hours
  → If resolved → marks task "In Progress" in Notion with a ✅ comment
  → If unresolved → tags as CRITICAL BLOCKER, checks owner task load,
    suggests delegation if owner has 5+ active tasks

Phase 3: MULTI-PLATFORM SYNC
  → Creates/updates task records in Notion with full relation properties
  → For tasks owned by Deborah → also creates a Google Task with
    title, due date, and a link back to the Notion source

Phase 4: VERIFICATION
  → Prepends a "Summary for Approval" block to the source meeting note
  → Lists all tasks created, blockers resolved, critical blockers open,
    and delegation recommendations
```

### Example Output Written to Notion

```markdown
## Summary for Approval

### New and Updated Tasks
- Design landing page — Owner: Deborah — Due: 2026-04-05 — Project: Launch Q2

### Blocker Outcomes
- ✅ Resolved via Slack context: API integration delay
- 🚨 CRITICAL BLOCKER: Budget approval (Owner task load: 7, Delegation: Yes)

### Cross-Platform Sync
- Notion updates: 4
- Google Tasks created (Deborah): 2
- Slack checks skipped due to integration issues: 0

### Approval Needed
- Delegate budget approval from Deborah to Finance lead
```

---

## Technologies Used

| Technology | Purpose |
|---|---|
| **[Notion MCP](https://mcp.notion.com)** | Read meeting notes, create & update task records, write approval summaries |
| **[Slack MCP](https://github.com/modelcontextprotocol/server-slack)** (`@modelcontextprotocol/server-slack`) | Search messages for blocker resolution context |
| **[Google Tasks MCP](https://github.com/alvincrave/gtasks-mcp)** (`@alvincrave/gtasks-mcp`) | Create Google Tasks for assigned owners |
| **Claude (via MCP)** | Orchestrates the full 4-phase workflow using the SKILL.md agent definition |
| **`mcp.json`** | MCP server configuration connecting all three integrations |

---

## Getting Started

### Prerequisites

- A [Notion](https://notion.so) account with:
  - A **"Meeting Notes"** database (with transcript/notes pages)
  - A **Tasks** database with properties: Title, Owner, Deadline, Status, Project (relation), Source (relation)
- A [Slack](https://slack.com) workspace with a Bot Token
- A [Google account](https://tasks.google.com) with Google Tasks API enabled
- [Claude Desktop](https://claude.ai/download) or any MCP-compatible AI client

---

### Step 1 — Clone the repo

```bash
git clone https://github.com/Zubbee18/strategic-task-tracker.git
cd strategic-task-tracker
```

---

### Step 2 — Get your API credentials

#### Notion Integration Token
1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click **"New integration"** → name it → **Save**
3. Copy the **Internal Integration Token** (starts with `secret_...`)
4. Connect the integration to both your Meeting Notes and Tasks databases:
   - Open each database → `...` top right → **Connections** → select your integration

#### Slack Bot Token
1. Go to [api.slack.com/apps](https://api.slack.com/apps) → **Create New App**
2. Add OAuth scopes: `channels:history`, `search:read`, `channels:read`
3. Install to workspace → copy the **Bot User OAuth Token** (`xoxb-...`)
4. Copy your **Team ID** from workspace settings (starts with `T...`)

#### Google Tasks Credentials
1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a project → enable the **Google Tasks API**
3. Create **OAuth 2.0 credentials** (Desktop app type)
4. Download the credentials and run the OAuth flow to get a refresh token

---

### Step 3 — Configure `mcp.json`

Copy the example and fill in your credentials:

```bash
cp mcp.json.example mcp.json
```

Edit `mcp.json`:

```json
{
  "mcpServers": {
    "notion": {
      "url": "https://mcp.notion.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_NOTION_TOKEN"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-token",
        "SLACK_TEAM_ID": "T0123456789"
      }
    },
    "google-tasks": {
      "command": "npx",
      "args": ["-y", "@alvincrave/gtasks-mcp"],
      "env": {
        "GOOGLE_CLIENT_ID": "your-client-id",
        "GOOGLE_CLIENT_SECRET": "your-client-secret",
        "GOOGLE_REFRESH_TOKEN": "your-refresh-token"
      }
    }
  }
}
```

> ⚠️ Never commit `mcp.json` — it is already in `.gitignore`

---

### Step 4 — Load the skill in Claude

Point your MCP client (Claude Desktop) to the `mcp.json` config. Then in a new conversation, the `strategic-ea-agent` skill defined in `SKILL.md` will be available.

Trigger it by saying something like:

> *"Process the latest meeting transcript in my Notion Meeting Notes database and sync action items."*

---

## Notion Database Schema

### Meeting Notes Database
| Property | Type | Notes |
|---|---|---|
| Title | Title | Meeting name |
| Date | Date | Meeting date |
| Transcript | Rich text / Page content | Paste full notes here |

### Tasks Database
| Property | Type | Notes |
|---|---|---|
| Title | Title | Task name |
| Owner | Text / Person | Who owns it |
| Deadline | Date | Due date |
| Status | Select | `Not Started`, `In Progress`, `Done`, `CRITICAL BLOCKER` |
| Project | Relation | Links to a Projects database |
| Source | Relation | Links back to the Meeting Note |

---

## Project Structure

```
strategic-task-tracker/
├── SKILL.md              # Agent skill definition (the brain of the system)
├── mcp.json.example      # MCP server config template (safe to commit)
├── mcp.json              # Your real config with credentials (gitignored)
├── README.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
└── LICENSE
```

---

## How the Skill Works

The `SKILL.md` file defines the `strategic-ea-agent` — a structured prompt that instructs Claude exactly how to orchestrate the 4-phase workflow using MCP tools. It includes:

- **Trigger conditions** — when to activate the skill
- **Operating rules** — how to handle missing integrations, traceability, and escalation
- **Phase-by-phase instructions** — exactly which MCP tools to call and when
- **Output format** — the exact approval summary template written back to Notion

---

## License

MIT