# SiS - Workflow Database (PRF)

## Overview
**Workflow ID:** QGEy8ZLl1iQGrThV  
**Status:** Active  
**Created:** May 27, 2025  
**Last Updated:** December 16, 2025

## Purpose
Maintains a comprehensive database of all n8n workflows on the PRF server, tracking ownership, status, tags, and syncing with GitHub repository for documentation.

## Triggers
**Type:** Multiple (Webhook + Schedule)

### Webhook Triggers

#### 1. New Record Webhook
**Method:** POST  
**Path:** `/webhook/sis-sync-wf-new-record`  
**Purpose:** Sync individual workflow when created/updated

#### 2. Cursor Pagination Webhook
**Method:** POST  
**Path:** `/webhook/sis-sync-wf-cursor`  
**Purpose:** Handle paginated results for full sync

#### 3. Delete Workflow Record
**Method:** GET  
**Path:** `/webhook/sis-sync-wf-delete`  
**Purpose:** Remove deleted workflows from database

### Schedule Trigger
**Frequency:** Daily at 8:05 PM (20:05)  
**Purpose:** Full synchronization of all workflows

## Workflow Process

### Full Sync Flow (Scheduled)

#### 1. Initialize Sync
- Fetches all workflows from database
- Sorts by last_synced_at (DESC) and ID

#### 2. Check Last Sync
- Filters for workflows where:
  - `last_synced_at` is NULL, OR
  - `last_synced_at` is before yesterday

#### 3. Process in Batches
- Loops through workflows needing sync
- For each workflow:
  - Fetches latest data from n8n API
  - Checks if workflow still exists (error = deleted)

#### 4. Handle Deleted Workflows
- If error returned: Deletes from database
- If successful: Updates last_synced_at

#### 5. Cursor-Based Pagination
- Fetches workflows from n8n API (20 at a time)
- Checks for pagination cursor
- If cursor exists: Triggers cursor webhook recursively
- If webhook response: Returns cursor response

### Single Record Sync Flow (New Record Webhook)

#### 1. Receive Workflow ID
- Gets workflow_id from webhook payload
- Sets server to "prf.thesqd.com"

#### 2. Fetch Workflow Details
- Retrieves full workflow JSON from n8n API

#### 3. Extract Wing Tag
Custom code determines "wing" from tags:
- **SaRW:** Social and Remote Work
- **DW:** Digital Work
- **NULL:** No wing assigned

#### 4. Get Employee Owner
- Queries `employees` table
- Matches by email from workflow's shared project relations

#### 5. Upsert Workflow Record
Inserts or updates in `sis_workflows`:
- **id:** Workflow ID (primary key)
- **name:** Workflow name
- **platform:** "n8n"
- **server:** "prf.thesqd.com"
- **link:** Direct workflow URL
- **created_at:** Creation timestamp
- **wing:** Extracted wing tag
- **owner_email:** Employee email
- **owner_cu_id:** ClickUp ID of owner
- **active:** Workflow active status
- **last_udpated_at:** Current time
- **last_synced_at:** Current time

#### 6. GitHub Sync (Disabled)
- Check if file exists in GitHub
- Create or edit workflow JSON file
- Commit to sis-workflows repository

### Delete Flow (Delete Webhook)

#### 1. Load Database Records
- Fetches all workflows from `sis_workflows`

#### 2. Check Sync Status
- Filters for workflows needing verification

#### 3. Process Deletions
- Loops through workflows
- Attempts to fetch from n8n API
- If error (deleted): Removes from database
- If successful: Updates last_synced_at

## Key Features
- **Multi-Trigger Design:** Handles new, updated, and deleted workflows
- **Pagination Support:** Processes large workflow lists via cursor
- **Wing Classification:** Categorizes workflows by team
- **Owner Tracking:** Links workflows to employee owners
- **GitHub Integration:** Optionally syncs to version control (disabled)
- **Self-Healing:** Removes deleted workflows automatically

## Database Tables

### Supabase
- `sis_workflows` (read/write)
  - Workflow metadata and tracking
- `employees` (read)
  - Owner information lookup

### GitHub (Disabled)
- Repository: aceesquad/sis-workflows
- Path: sis2/{workflow_id}.json

## sis_workflows Schema
```sql
id TEXT PRIMARY KEY,
name TEXT,
platform TEXT,
server TEXT,
wing TEXT,
link TEXT,
created_at TIMESTAMP,
owner_email TEXT,
owner_cu_id INTEGER,
active TEXT,
last_udpated_at TIMESTAMP,
last_synced_at TIMESTAMP,
description TEXT,
tags ARRAY,
documentation TEXT
```

## Wing Extraction Logic
```javascript
tags.includes('sarw') → 'SaRW'
tags.includes('dw') → 'DW'
else → null
```

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Timeout:** 300 seconds
- **Continue on Error:** n8n API calls
- **Always Output Data:** Ensures processing continues

## Sync Strategies

### Daily Full Sync
- Processes all workflows needing update
- Verifies workflow still exists
- Updates sync timestamp

### Real-Time Sync
- Triggered on workflow save
- Immediate database update
- No pagination needed

### Pagination Sync
- Handles large workflow lists
- Recursive cursor following
- 20 workflows per page

## Notes
- Critical for workflow inventory management
- Enables workflow analytics and reporting
- GitHub sync disabled (feature flag for future use)
- Replace empty strings enabled on upsert
- Tracks workflow lifecycle (creation, updates, deletion)
- Wing tags help organize by team/department
- Owner tracking enables accountability




