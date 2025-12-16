# Task - Link Primary x Non Primary

## Overview
**Workflow ID:** qmSPsrgFvHlXPkb5  
**Status:** Active  
**Created:** May 29, 2025  
**Last Updated:** December 16, 2025

## Purpose
Establishes dependencies between primary and non-primary tasks in multi-project submissions, ensuring proper task sequencing and designer coordination.

## Triggers
**Type:** Webhook + Schedule

### Webhook Trigger
**Method:** POST  
**Path:** `/webhook/link-projects`  
**URL:** https://prf.thesqd.com/webhook/link-projects

### Schedule Trigger
**Frequency:** Every 15 minutes

## Workflow Process

### 1. Identify Unprocessed Submissions
- **From Webhook:** Uses provided `gis_id`
- **From Schedule:** Queries for unprocessed general submissions
  - Within last 24 hours
  - Excludes demo account (rec5xR7FXVuNGoeFs)
  - Where `is_project_details_processed` = FALSE

### 2. Determine Project Count
Switch based on Project Details count:
- **No PD:** Resets processing flag to NULL
- **1 PD:** Marks as processed, updates to TRUE
- **> 1 PD:** Continues to linking process

### 3. Identify Primary Task
Queries for:
- Project submission with `is_primary` = TRUE
- Gets department (Web or Brand determines special handling)

### 4. Validate Primary Task
- Checks if primary task exists
- Verifies task status (not closed/done)
- If not valid, either skips or throws error

### 5. Find Non-Primary Tasks
Queries for:
- Tasks with `is_primary` = FALSE
- Has ClickUp ID assigned
- Project type not marked as `never_dependent`
- Not already dependent (`is_task_dependent` check)

### 6. Create Dependencies
For each non-primary task:
- **Set Custom Field:** Adds Primary Task URL to custom field
- **Add Dependency:** Links task as dependent of primary
- **Update Status:** Sets status to "dependent", removes due date
- **Add Tag:** Applies "dependent" tag
- **Modify Description:** Adds dependency note to description
- **Update Database:** Sets `is_task_dependent` = TRUE

### 7. Add ClickUp Comments
For active non-primary tasks:
- Creates task summary with all related task links
- Posts formatted comment with:
  - Primary task link and title
  - List of all related tasks (excluding current)
  - Instructions to use primary designs as influence

### 8. Finalize Processing
- Marks general submission as processed
- Updates `is_project_details_processed` = TRUE

## Key Features
- **Smart Routing:** Different handling for Web/Brand vs other departments
- **Active Task Detection:** Only processes non-closed tasks
- **Comment Generation:** Automatic markdown-to-ClickUp formatting
- **Comprehensive Logging:** All actions logged to task_update_log

## Database Operations

### Supabase
- `prf_general_submissions` (read/update)
- `prf_project_submissions` (read/update)
- `prf_selection_types` (read)
- `task_update_log` (write)

## Comment Format
```markdown
📌 **NOTES FOR DESIGNER**

This task has been marked as dependent of [Primary Task](URL).

Here are the updated tasks linked to it:
- [Task 1](URL)
- [Task 2](URL)

Please use the designs from the other tasks to influence this project request.
```

## Error Handling
- **Error Workflow:** 38tttU6Tcl3Y8tZy
- **Timeout:** 600 seconds (10 minutes)
- **Continue on Error:** Enabled for API calls

## Task Update Log Entries
1. Custom field - set Primary Task field
2. Add task dependency
3. Update due date and status to dependent
4. Properties - add "dependent" task tag

## Sub-Workflow Integration
- **Task - Add Dependency on Description** (TBqBivbv59NRsQjF)
  - Adds formatted dependency note to task description
  - Source marked as "non-primary"

## Notes
- Waits 15 seconds after webhook trigger for data propagation
- Processes in batches to handle multiple non-primary tasks
- Excludes projects marked with `never_dependent` flag
- Properly formats task links and references in comments


