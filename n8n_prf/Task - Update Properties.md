# Task - Update Properties

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `fv3CIc8W0jkk1vdU` |
| **Status** | Active |
| **Created** | May 28, 2025 |
| **Last Updated** | December 16, 2025 |
| **Execution Timeout** | 300 seconds (5 minutes) |
| **Error Workflow** | `38tttU6Tcl3Y8tZy` |
| **Available in MCP** | Yes |

---

## Purpose

Updates ClickUp task properties after task creation, including:
- Department custom field
- Project tags based on content count
- Time estimates
- "All-In" tags for premium members
- CSS Rep assignment
- Trial account tags
- Submission link custom field

**Called From**: Project Details Processor, Brand Guide Edit workflows

---

## Trigger

**Type**: Webhook (POST)

| Endpoint | URL |
|----------|-----|
| **Production** | `https://prf.thesqd.com/webhook/prf-update-task-properties` |
| **Test** | `https://prf.thesqd.com/webhook-test/prf-update-task-properties` |

**Request Body**:
```json
{
  "task_id": "<clickup_task_id>",
  "member_id": "<member_number>",
  "pd_submission_id": "<submission_id>",
  "ptid": "<project_type_airtable_id>",
  "time_estimate": "<minutes>0",
  "follow_brand": true/false,
  "formid": "<optional_fillout_form_id>"
}
```

---

## Workflow Flow

### 1. Data Gathering

- **Project Type** - Fetches from `prf_selection_types` table (department, tag, tag_suffix, filter_categories)
- **Detail Submission** - Fetches submission from Fillout API
- **Get Task** - Gets task details from ClickUp
- **Strategy** - Checks if member is in "All-In Members" view in Strategy base
- **Brains** - Fetches account data from Master Brain base

### 2. Set Department

Sets the Department custom field (`423fe36b-7833-4b75-8685-51143b675dbe`) based on project type:

| Department | Custom Field Value |
|------------|-------------------|
| Design | `3fbc274e-f59f-4ffa-a035-ed6b03f81378` |
| Video | `1fb2955d-3a0a-4c0b-b12d-4ebd237a12b1` |
| Socials | `c2d8a589-ef2d-4bfe-a944-90440ab00f2e` |
| Branding | `7aa5c657-be88-43fd-8a2b-4c3741be7ea5` |

### 3. Set Submission Link

Sets the custom field (`20772b3c-def8-4e9d-bc6a-31a35ec29926`) to:
```
https://pv.thesqd.com/project/{task_id}
```

### 4. Tag Assignment

**Get Tag** code node calculates tags based on:
- Base tag from project type
- Content count (determines suffix like `1-3`, `4-6`, `7-9`, `10`)
- For tasks with >10 content, parses title for range (e.g., "(11-20)")
- `follow_brand` flag adds "branded" tag

**Split Tags** then adds each tag individually to the task.

### 5. Subtask Processing

- Waits briefly for subtasks to be created
- **Get Subtasks** - Fetches subtasks via ClickUp API
- **Split Out** - Processes each subtask
- **Code Fields** - Applies same tag logic to subtasks

### 6. Time Estimate

If `time_estimate` is provided and not "0":
- Updates task time estimate in ClickUp

### 7. Premium Member Tags

**All-In Tag**: If member found in Strategy "All-In Members" view:
- Adds `all-in` tag to task

**Trial Tag**: If account status is "Trial" in Brains:
- Adds `trial` tag to task

### 8. CSS Rep Assignment

If account has a CSS Rep assigned in Brains:
- Sets CSS Rep custom field (`12bb27b3-7942-4208-8c1c-21a568a47610`)

---

## Data Sources

### PostgreSQL

| Table | Purpose |
|-------|---------|
| `prf_selection_types` | Project type configuration |

### Airtable

| Base | Table | Purpose |
|------|-------|---------|
| `appjHSW7sGtitxoHf` (Strategy) | 🧩 Accounts | All-In member check |
| `app6LbfcLAbwsfobY` (Brains) | Accounts | CSS Rep, Trial status |

---

## Logging

All operations logged to `task_update_log`:
- `custom field - set https://pv.thesqd.com`
- `custom field - set department`
- `properties - add "{tag}" task tag`
- `properties - update time estimate`
- `properties - add "all-in" task tag`
- `properties - add "trial" task tag`
- `custom field - update CSS Rep`




