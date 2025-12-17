# Project Details to prf_project_submissions

## Overview

| Property | Value |
|----------|-------|
| **Workflow ID** | `18NEP5p30yj1Nqai` |
| **Status** | Active |
| **Created** | October 7, 2025 |
| **Last Updated** | December 16, 2025 |
| **Available in MCP** | Yes |

---

## Purpose

Tracks project submission lifecycle by saving start and end times to `prf_project_submissions`. This workflow is called by Fillout forms to record:
- When a user starts a project form (start_time)
- When a user completes a project form (end_time + full submission data)

---

## Trigger

**Type**: Webhook (POST)

| Endpoint | URL |
|----------|-----|
| **Production** | `https://prf.thesqd.com/webhook/3922f4a8-8c9e-43bc-84db-32f0811ee9cf` |
| **Test** | `https://prf.thesqd.com/webhook-test/3922f4a8-8c9e-43bc-84db-32f0811ee9cf` |

**Response Mode**: Returns JSON from last node

**Request Body** (can be in body, body.body, or body.query):
```json
{
  "formid": "<fillout_form_id>",
  "submissionid": "<submission_id>"
}
```

---

## Workflow Flow

### 1. Fetch Submission Details

- **Get End Submission Details** - Fetches submission from Fillout API with `includeEditLink=true`

### 2. Determine Start vs End

**If Node** checks if `submission.editLink` exists:
- **Has editLink** (true) → This is a completed submission → Save End Record
- **No editLink** (false) → This is a started submission → Save Start Record

### 3a. Save Start Record

For form starts (no editLink):

- **Get Project Type Start** - Looks up project type from `prf_selection_types` by `form_id`
- **Save Project Start Record** - Upserts to `prf_project_submissions`:
  - `project_submission_id` - From submission ID
  - `start_time` - Current timestamp
  - `project_type` - From selection types lookup

### 3b. Save End Record

For completed submissions (has editLink):

- **Code in JavaScript** - Passes through submission data
- **Get Project Type End** - Looks up project type by `ptid` URL parameter
- **Save Project End Record1** - Full SQL INSERT/UPDATE with:

| Field | Value |
|-------|-------|
| `project_submission_id` | Submission ID |
| `gis_id` | From `giid` URL param or generated UUID |
| `account_id` | From `accid` URL param |
| `user_id` | From `userid` URL param |
| `start_time` | From `submission.startedAt` |
| `end_time` | Current timestamp |
| `raw_data` | Full submission JSON |
| `project_name` | First question's value |
| `project_type` | From selection types lookup |
| `edit_id` | Submission edit link |
| `editable` | false |

---

## Data Sources

### PostgreSQL

| Table | Purpose |
|-------|---------|
| `prf_selection_types` | Project type lookup by form ID |
| `prf_project_submissions` | Submission tracking (target table) |

### Fillout API

Fetches submission data including:
- Questions and answers
- URL parameters
- Edit link (for completed forms)
- Submission timestamps

---

## URL Parameters Expected

The workflow expects these URL parameters in the Fillout form:

| Parameter | Purpose |
|-----------|---------|
| `giid` | General Information ID |
| `accid` | Account ID |
| `userid` | User ID |
| `ptid` | Project Type Airtable Record ID |

---

## Usage

This workflow enables tracking:
1. **Form abandonment** - Forms started but not completed
2. **Completion time** - Duration from start to end
3. **Submission audit trail** - Full raw_data stored for reference




