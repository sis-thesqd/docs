# Task - Update File Size Content

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `jDlV6KGcm41RVeb4` |
| **Status** | Active |
| **Created** | May 28, 2025 |
| **Last Updated** | December 16, 2025 |
| **Execution Timeout** | 300 seconds (5 minutes) |
| **Error Workflow** | `38tttU6Tcl3Y8tZy` |
| **Available in MCP** | Yes |

---

## Purpose

Creates file size deliverables for design tasks:
- Adds checklist items for standard file sizes
- Creates subtasks for large format file sizes (requiring vector builds)
- Updates `task_deliverable_id` in Supabase

---

## Trigger

**Type**: Webhook (POST)

| Endpoint | URL |
|----------|-----|
| **Production** | `https://prf.thesqd.com/webhook/update-file-size-content` |
| **Test** | `https://prf.thesqd.com/webhook-test/update-file-size-content` |

**Request Body**:
```json
{
  "pdid": "<project_details_airtable_id>",
  "fsid": ["<file_size_id_1>", "<file_size_id_2>"],
  "spaceid": "<clickup_space_id>",
  "folderid": "<clickup_folder_id>",
  "listid": "<clickup_list_id>",
  "parentid": "<parent_task_id>"
}
```

---

## Workflow Flow

### 1. Initial Wait & Validation

- **Wait** - 10 second delay to allow Project Details to be fully processed
- **2 Get PD** - Fetches Project Details record from Airtable
- **Switch** - Checks if "File Size Template Checked?" is "No"
  - If "Needs Processing" → Stop and Error
  - Otherwise → Continue processing

### 2. File Size Processing

- **task_deliverable_id** - Checks if task already has a deliverable ID in `prf_project_submissions`
- **If** - Routes based on whether checklist already exists:
  - No deliverable ID → Create new Checklist
  - Has deliverable ID → Use existing

- **Split Out1** - Splits file sizes array for individual processing
- **2 Get File Size** - Fetches each file size details from Airtable

### 3. Categorization Logic

**Code in JavaScript** determines whether each file size becomes a checklist item or subtask:

| Department | File Size Count | Big Dimensions | Result |
|------------|-----------------|----------------|--------|
| Non-Design | Any | - | All checklist items |
| Design | 1 | - | Checklist item |
| Design | 2 | No | Checklist + Subtask |
| Design | 2+ | Mixed | Big → Subtask, Standard → Checklist |

### 4. Checklist Item Creation

For standard file sizes:

**Checklist Name Format**:
```
{File Size Label} · {Template provided} · {Bleed Info} · {DPI} DPI
```

Examples:
- `1080x1080 Instagram Post · Template provided`
- `11x17 Poster · 0.125" Bleed · Template provided · 300 DPI`

### 5. Subtask Creation

For large format/big dimension file sizes:

**Task Name**: `{File Size Label} for {Initial Task Title}`

**Content**: Links to SOP with vector build requirements

**Tags**: `largeformat`

### 6. Data Updates

- **task_deliverable_id** - Updates `prf_project_submissions` with checklist ID
- **task_deliverable_id1** - Creates new record if checklist was just created

---

## Data Sources

### Airtable

| Base | Table | Purpose |
|------|-------|---------|
| `appXCvcntPYSBZey2` | Project Details | Project submission data |
| `appXCvcntPYSBZey2` | File Sizes | File size specifications |

### PostgreSQL

| Table | Purpose |
|-------|---------|
| `prf_project_submissions` | Task deliverable ID storage |

---

## File Size Fields Used

| Field | Purpose |
|-------|---------|
| `File Size Label` | Display name |
| `File Size Folder` | Indicates if template exists |
| `Print or Digital` | Determines if bleed info needed |
| `Bleed Info` | Print bleed specifications |
| `DPI` | Resolution requirement |
| `Big Dimensions?` | Determines checklist vs subtask |

---

## Logging

Operations logged to `task_update_log`:
- `file size - create checklist`
- `create subtask - parent: {id}`

---

## Error Handling

If "File Size Template Checked?" is "No", the workflow throws an error with instructions to:
1. Try re-running the workflow
2. Check for errors in workflow `AsfOk8sSDZhGhetk` at the same time




