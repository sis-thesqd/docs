# Project Details Processor

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `aLvfX2XhNgBcZuhX` |
| **Status** | Active |
| **Created** | June 10, 2025 |
| **Last Updated** | December 16, 2025 |
| **Execution Timeout** | 300 seconds (5 minutes) |
| **Error Workflow** | `38tttU6Tcl3Y8tZy` |
| **Available in MCP** | Yes |

---

## Trigger

**Type**: Webhook (POST)

| Endpoint | URL |
|----------|-----|
| **Production** | `https://prf.thesqd.com/webhook/project-detail-processor` |
| **Test** | `https://prf.thesqd.com/webhook-test/project-detail-processor` |

**Request Body**:
```json
{
  "formid": "<fillout_form_id>",
  "submissionid": "<fillout_submission_id>",
  "retry": false
}
```

---

## Workflow Sections

### 1. Core Data

The initial section handles data gathering and validation:

- **prf_project_submission** - Checks if ClickUp ID already exists for this submission
- **Detail Submission** - Fetches full submission data from Fillout API
- **Filter out demo submissions** - Excludes test user `demo-rec-301`
- **Guest** - Retrieves guest/user information from Airtable
- **Project Type** - Fetches project type configuration from Airtable
- **ClickUp List** - Determines the correct ClickUp list for task creation
- **GI (General Information)** - Checks for existing General Information record

**Error Handling**:
- If ClickUp ID already exists → Flow stops (no duplicate tasks)
- If submission ID is missing → Throws error
- If no project details found and already retried → Throws error with edit link
- If no project details found (first attempt) → Waits 1 minute and auto-retries

---

### 2. Folder Code

Generates and assigns sequential folder codes for project organization:

- **Project Code** - Generates incremental project counter per account
- **Seq Folder Number** - Updates general submission with folder number
- **Folder Code** - Sets the folder code for downstream use

---

### 3. Content Variables & Task Fields

Processes form content and builds task description:

- **General Info** - Fetches submission data from `prf_general_submissions` table
- **Creative Direction** - Gets design style and creative preferences from Airtable
- **Project Details** - Searches for project detail record in Airtable
- **Content Variables** - Retrieves template variables from `prf_content_variables` table
- **TCP (Aggregate)** - Aggregates content variables
- **rawContent** - Processes and replaces template variables in content
- **Task Fields** - Builds final task title and content sections:
  - `title` - Project title with project type
  - `pd-header` - Administrative header with folder code and go-live date
  - `pd-audience` - Audience information
  - `pd-description` - Project description
  - `pd-filesizeinfo` - File size specifications
  - `cd-imagery` - Vision/imagery notes
  - `cd-designstyle` - Design style and brand guide references

---

### 4. Content Creation

Handles content formatting based on content count:

**Content Count Switch**:
- **None** (pd-multiplecontent not present) → Fix Content → Single content item
- **> 10** (pd-multiplecontent present) → Get Subtask Count → Split Content Count → Multiple subtasks

**Fix Content** - Cleans up content:
- Removes empty headers
- Adds proper markdown formatting
- Handles sermon clips cleanup
- Encodes URLs
- Removes excess whitespace

**Fix Multiple Content** - For multi-content projects:
- Main task gets full content
- Subtasks get link back to main task

---

### 5. ClickUp Task Creation

**If1 Switch** - Routes based on form type:
- **Standard forms** → Create Task directly
- **Subbrand form (uwYq54sL7rus)** → Uses Subbrand template via HTTP request

**Create Task** - Creates main ClickUp task with:
- Task name from Task Fields
- Assignee from guest's ClickUp User ID
- Proper space/folder/list placement

**Create Main Task Filter** - Confirms task creation before proceeding

---

### 6. Checklist Creation

Creates deliverable checklists when Action Items are present:

- **Manual Checklist** - Filters for submissions with action items
- **Checklist** - Creates "Deliverables" checklist on the task
- **Checklist Items** - Parses action items (newline-separated)
- **Split Out** - Creates individual checklist items
- **Action Item** - Adds each item to the checklist
- **task_deliverable_id** - Updates Supabase with checklist ID

---

### 7. Subtask & Dependency

For projects with more than 10 content items:

- **Subtask** - Creates subtasks named `{title} ({range})` (e.g., "Project - Social Graphics (11-20)")
- **Create Subtask Details** - Creates Airtable record for each subtask
- **Subtask Content** - Updates subtask description via SISX webhook
- **Dependency** - Sets main task as dependency for subtasks

---

### 8. Sync Task & Add Properties

Final synchronization and property updates:

- **Content** - Consolidates final content
- **Update Task Link** - Updates Airtable with ClickUp task link and content
- **Update Supabase** - Updates `prf_project_submissions` with:
  - ClickUp ID
  - General Information Submission ID
  - Task content
  - Editable flag (false)
- **Main Task Content** - Updates task description via SISX webhook
- **Filter** - Checks for file sizes
- **Update File Size Content** - Creates file size subtasks if applicable
- **Update Task Properties** - Sets additional task properties via webhook
- **content_history** - Logs content to `prf_content_history` table

---

## Special Cases

### Sermon Series Carousel Template

For "All In" members submitting sermon series forms (`hbd1fTXmnFus`):
- Checks account plan includes "All In"
- Verifies ministry type is "General Audience"
- Calls **Sermon Series Carousel Template** workflow (`lnQdPMrGLYdJmTz1`)

### SMS Forms Processing

When Switch detects sermon series with social media plan:
- Calls **Process SMS Forms** webhook at `https://prf.thesqd.com/webhook/sms-project-handler`

### Subbrand Processing

For subbrand form (`uwYq54sL7rus`):
- Uses ClickUp task template `t-86dx12vwz`
- Calls **Subbrand Processor** workflow (`P7CDE0AUv1ReEkCk`)

---

## Data Sources

### Airtable

| Base | Table | Purpose |
|------|-------|---------|
| `appXCvcntPYSBZey2` | Guests (`tblSpD6Sw8VVnr0Wh`) | User/member information |
| `appXCvcntPYSBZey2` | Project Types (`tblY94xLE8XmHqqq7`) | Project type configuration |
| `appXCvcntPYSBZey2` | Project Details (`tblUbcbEnRP1pLvaD`) | Project submission records |
| `appXCvcntPYSBZey2` | General Information (`tblbHuKhDjUCwebAK`) | General info submissions |

### PostgreSQL (Supabase)

| Table | Purpose |
|-------|---------|
| `prf_account_project_counter` | Sequential project numbering per account |
| `prf_general_submissions` | General information submission data |
| `prf_content_variables` | Content template variables |
| `prf_project_submissions` | Project submission tracking |
| `prf_content_history` | Content version history |

### Supabase (Direct)

| Table | Purpose |
|-------|---------|
| `task_update_log` | Logs all task operations for auditing |

---

## External Webhooks Called

| Webhook | Purpose |
|---------|---------|
| `prf.thesqd.com/webhook/clickup-list-checker` | Determines correct ClickUp list |
| `prf.thesqd.com/webhook/fix-missing-info` | Creates missing GI records |
| `prf.thesqd.com/webhook/update-file-size-content` | Creates file size subtasks |
| `prf.thesqd.com/webhook/prf-update-task-properties` | Sets task custom fields |
| `prf.thesqd.com/webhook/sms-project-handler` | Processes social media plans |
| `sisx.thesqd.com/webhook/jWZxgFKCMaqXvwfm-update-desc` | Updates main task description |
| `sis1.thesqd.com/webhook/desc-updated-new` | Updates subtask descriptions |
| `churchmediasquad.app.n8n.cloud/webhook/check-airtable-record` | Checks Airtable records |

---

## Logging

All task operations are logged to `task_update_log` table with:
- `task_id` - ClickUp task ID
- `narrative` - Description of operation
- `workflow_id` - This workflow's ID
- `server` - "prf"

Logged events:
- `task created`
- `create subtask - parent: {id}`
- `create dependency - parent: {id}`
- `create checklist`
- `add checklist item "{item}"`
- `set description`
