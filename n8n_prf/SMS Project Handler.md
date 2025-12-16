# SMS Project Handler

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `3oY3dGhM5SuYghYV` |
| **Status** | Active |
| **Created** | June 18, 2025 |
| **Last Updated** | December 16, 2025 |
| **Execution Timeout** | 300 seconds (5 minutes) |
| **Error Workflow** | `38tttU6Tcl3Y8tZy` |
| **Available in MCP** | Yes |

---

## Trigger

**Type**: Webhook (POST)

| Endpoint | URL |
|----------|-----|
| **Production** | `https://prf.thesqd.com/webhook/sms-project-handler` |
| **Test** | `https://prf.thesqd.com/webhook-test/sms-project-handler` |

**Additional Trigger**: Execute Workflow Trigger (`WebSermon Pack`) - Can be called from other workflows

---

## Purpose

Handles Social Media Strategy (SMS) project submissions including:
- Sermon Series plans
- Coaching Calls
- Sermon Recap Posts
- Holiday content
- Event content
- General social media plans

---

## Workflow Flow

### 1. Initial Data Gathering

- **Detail Submission** - Fetches full submission from Fillout API
- **Default Info** - Calls sub-workflow to get task creation info (spaceid, folderid, listid, etc.)
- **Get Comms Rep** - Queries PostgreSQL for account's communications rep (with fallback to Amber Pankey)
- **Default Fields** - Sets member ID, church name, submission ID, project type ID

### 2. Project Type Routing

**Switch Node** routes based on form ID:

| Form ID | Project Type | Output |
|---------|--------------|--------|
| `hbd1fTXmnFus` | Sermon Series | Sermon Series fields |
| `7kmZ3uFFuous` | Coaching Call | Coaching Call fields |
| `v4sJ2Zokf4us` | Sermon Recap | Sermon Recap fields |
| Other | Holiday/Event/General | Code node processing |

### 3. Task Field Configuration

Each project type sets specific fields:
- **task title** - Generated title with member ID and project name
- **tag** - Project-specific tags (e.g., `sms-sermon-recap`, `sms-coaching-call`)
- **status** - Initial status
- **time estimate** - Minutes for the task
- **assignee id** - Guest and/or comms rep ClickUp IDs
- **subtask assignee** - Comms rep for subtasks
- **due date** - Calculated based on submission time and project type
- **subtask condition** - Airtable formula for filtering subtasks

### 4. Content Processing

- **General Info** - Fetches GI record from Airtable
- **rawContent** - Calls Replace File Variables sub-workflow
- **Fix Content** - JavaScript code that:
  - Replaces template variables
  - Processes deliverables into formatted list
  - Cleans up markdown formatting
  - Removes empty sections

### 5. ClickUp Task Creation

- **Create Main Task** - Creates task with configured fields
- **PR Subtask** - Fetches subtask templates from Airtable `SMS Subtasks` table
- **Check Deliverables** - Filters subtasks based on selected deliverables
- **Fix Subtask Details** - Processes subtask titles, due dates, and content
- **Create Subtask** - Creates each subtask in ClickUp

### 6. Sermon Recap Video (SRP) Handling

For Sermon Recap submissions:
- **SR Due Date** - Calculates due date based on submission day/time
- **Create SRP Video** - Creates video task in dedicated list (901705255352)
- **SRP Video Content** - Updates task description
- **Link SRP Video** - Creates dependency between main task and SRP video
- **Update Status** - Sets main task to "Dependent"

### 7. Data Persistence

- **Upsert Submission** - Updates `prf_project_submissions` table
- **Update Content** - Saves task content to Supabase
- **Create Project Details** - Creates Airtable record if needed
- **Update PD** - Updates existing Project Details record
- **strategy_sermon_data** - Saves sermon data to PostgreSQL table

### 8. Notifications

- **Slack Notif** - Posts to project-specific Slack channel
- **Trial Notif** - Special notification for trial accounts
- **SRP Comment** - Adds beautify comment for early SRP submissions (1st or 2nd)

---

## Project Type Details

### Sermon Series (`hbd1fTXmnFus`)

| Field | Value |
|-------|-------|
| Tags | `sms-series-plan` |
| Status | Open |
| Due Date | Submission + 2 days |
| Assignees | Comms Rep + Member |

### Coaching Call (`7kmZ3uFFuous`)

| Field | Value |
|-------|-------|
| Tags | `sms-coaching-call` |
| Status | Upcoming Call |
| List ID | From calculation |
| Assignees | Comms Rep + Member (trial: Member only) |

### Sermon Recap (`v4sJ2Zokf4us`)

| Field | Value |
|-------|-------|
| Tags | `sms-sermon-recap` or `sms-social-trial` |
| Status | Open |
| Time Estimate | 15 minutes |
| Due Date | Calculated based on submission day |

---

## Data Sources

### Airtable

| Base | Table | Purpose |
|------|-------|---------|
| `appXCvcntPYSBZey2` | Project Types | Project type configuration |
| `appXCvcntPYSBZey2` | SMS Subtasks | Subtask templates |
| `appXCvcntPYSBZey2` | Project Details | Project submission records |
| `appXCvcntPYSBZey2` | General Information | GI submissions |
| `appXCvcntPYSBZey2` | Project Selection | Project selection updates |

### PostgreSQL

| Table | Purpose |
|-------|---------|
| `employees` | Comms rep lookup |
| `accounts` | Church name lookup |
| `prf_project_submissions` | Project submission tracking |
| `strategy_sermon_data` | Sermon data storage |
| `prf_content_history` | Content version history |

---

## Sub-Workflows Called

| Workflow | Purpose |
|----------|---------|
| Replace File Variables | Processes file upload placeholders |
| Info for Task Creation | Gets default task creation info |

---

## Logging

All task operations logged to `task_update_log` with:
- `task created`
- `set description`
- `create subtask - parent: {id}`
- `link tasks and update status`

