# SMS - Special Holiday Content

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `LNOYtAA1x5vGlTNm` |
| **Status** | Active |
| **Created** | November 10, 2025 |
| **Last Updated** | December 16, 2025 |
| **Execution Timeout** | 300 seconds (5 minutes) |
| **Error Workflow** | `38tttU6Tcl3Y8tZy` |
| **Available in MCP** | Yes |

---

## Purpose

Processes Christmas Break special holiday content submissions, creating tasks for:
- **Post Pack** - Invite Post, Merry Christmas Post, New Years Post
- **Yearly Recap Video (YRV)** - Video compilation task
- **Carousel** - Multi-image carousel post

---

## Trigger

**Type**: Webhook (POST)

| Endpoint | URL |
|----------|-----|
| **Production** | `https://prf.thesqd.com/webhook/sms-special-holiday-content` |
| **Test** | `https://prf.thesqd.com/webhook-test/sms-special-holiday-content` |

**Request Body**:
```json
{
  "formid": "<fillout_form_id>",
  "submissionid": "<submission_id>"
}
```

---

## Workflow Flow

### 1. Initial Data Gathering

- **Detail Submission** - Fetches full submission from Fillout API
- **Default Info** - Calls sub-workflow for task creation info (spaceid, folderid, listid, guest_clickup_id)
- **Default Content** - Sets up:
  - `raw_deliverables` - Selected deliverables from question `vbeA`
  - `deliverables` - Formatted deliverables list
  - `pd-header` - Submission time and email header
  - `giid` - General Information ID

### 2. Post Pack Task Creation

- **Task for Post Pack** - Creates main task:
  - Name: `{member_id} - Christmas Break Post Pack`
  - Tags: `sms-christmas-break`
  - Time Estimate: 15 minutes
  - Assignee: Guest

### 3. Content Generation

**Content for Post Pack** code node processes:

| Deliverable | Content Source |
|-------------|---------------|
| Invite Post | Calculation `0.1 Invite Post` |
| Merry Christmas Post | Calculation `0.2 Christmas Post` + Photo |
| New Years Post | Calculation `0.3 New Years Post` + Photo |

Photo handling:
- If photos uploaded → Creates markdown links
- If no photos → Uses generic template message

### 4. Additional Task Creation

**Switch** routes based on selected deliverables:

| Deliverable | Task Created |
|-------------|--------------|
| Yearly Recap Video | Task for YRV |
| Carousel | Task for Carousel |

**Task for YRV**:
- Name: `Christmas Break Social Video: Yearly Recap Video`
- Tags: `sms-christmas-break-video`
- Assignee: ID `89374776`

**Task for Carousel**:
- Name: `Christmas Break Social Post: Carousel`
- Tags: `sms-christmas-break-graphics`
- Assignee: ID `89374776`

### 5. Task Linking

- **Link** - Links YRV/Carousel tasks to Post Pack task via ClickUp API
- **Dependent Task ID** filter ensures linking happens

### 6. Content Processing

- **Task Content** - Passes through for all task types
- **Replace File Variables** - Calls sub-workflow to process file uploads
- **Cleanup Content** - Calls Content Cleanup API sub-workflow
- **Main Task Content** - Updates task description via SISX webhook

### 7. Data Persistence

**For Post Pack**:
- **Update Supabase** - Upserts to `prf_project_submissions` with project_type `124`
- **Update PD** - Updates Airtable Project Details

**For YRV/Carousel** (Filter → non-Post Pack):
- **Create PD** - Creates new submission record with UUID
- **Create PD1** - Creates Airtable Project Details record

### 8. Task Properties

- **Update Task Properties** - Calls prf-update-task-properties webhook

---

## Deliverables

Form question `vbeA` allows selection of:

| Deliverable | Description |
|-------------|-------------|
| Invite Post | Church service invite graphic |
| Merry Christmas Post | Christmas greeting post |
| New Years Post | New Year greeting post |
| Yearly Recap Video | Video compilation of year highlights |
| Carousel | Multi-image carousel post |

---

## Data Sources

### Airtable

| Base | Table | Purpose |
|------|-------|---------|
| `appXCvcntPYSBZey2` | Project Details | Project submission records |

### PostgreSQL

| Table | Purpose |
|-------|---------|
| `prf_project_submissions` | Submission tracking |
| `prf_content_history` | Content version history |

---

## Sub-Workflows Called

| Workflow | Purpose |
|----------|---------|
| Info for Task Creation | Gets default task creation info |
| Replace File Variables | Processes file upload placeholders |
| Content Cleanup API | Cleans up markdown content |

---

## Logging

Operations logged to `task_update_log`:
- `task created`
- `link task to {post_pack_id}`
- `task created with description`


