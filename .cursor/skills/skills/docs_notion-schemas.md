# Notion Database Schemas — Startup OS

This document defines the 9 relational Notion databases that form the Startup OS workspace.

## 1. Community Tracker
**Linked skill:** `/find-community`

| Property | Type | Notes |
|----------|------|-------|
| Community Name | Title | e.g. "Indie Hackers" |
| My Role | Select | Member / Contributor / Leader |
| Problem Observed | Text | Specific recurring pain point |
| Pain Level | Select | Mild / Moderate / Hair-on-fire |
| Can Reach Them? | Checkbox | Do I have direct access? |
| → Linked Idea | Relation | → Idea Validation DB |

## 2. Idea Validation Tracker
**Linked skill:** `/validate-idea`

| Property | Type | Notes |
|----------|------|-------|
| Idea | Title | One-sentence description |
| Target Customer | Text | Specific person |
| Would They Pay? | Select | Yes / Maybe / No |
| Conversations Had | Number | Count of interviews |
| Verdict | Select | Validated / Needs More / Pivot |
| → Community | Relation | → Community Tracker |
| → MVP | Relation | → MVP Builder |

*(... and so on for all 9 databases)*