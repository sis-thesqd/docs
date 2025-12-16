# Cleanup Incomplete Project Submissions

## Overview
**Workflow ID:** mRxfT7WtUj9ZQ5Cq  
**Status:** Active  
**Created:** June 26, 2025  
**Last Updated:** December 16, 2025

## Purpose
Automatically removes stale, incomplete project submissions that haven't been completed within 4 days, preventing database clutter.

## Trigger
**Type:** Schedule  
**Frequency:** Daily (default interval)

## Workflow Process

### 1. Find Incomplete Submissions
Queries `prf_project_submissions` for records where:
- `start_time` is older than 4 days
- `end_time` IS NULL (not completed)
- `account_id` IS NULL (not assigned)
- `clickup_id` IS NULL (no task created)

Returns:
- project_submission_id
- gis_id
- project_name
- project_type
- start_time
- created_at
- dropbox_folder_path
- days_old (calculated)

Ordered by start_time ascending (oldest first)

### 2. Delete Old Records
- Deletes each identified record from `prf_project_submissions`
- Uses project_submission_id as identifier

## Key Features
- **Automatic Cleanup:** Runs daily without intervention
- **Safety Window:** 4-day grace period
- **Selective Deletion:** Only removes truly incomplete submissions
- **Ordered Processing:** Handles oldest submissions first

## Deletion Criteria
A submission is deleted if ALL of these are true:
- ✅ Started more than 4 days ago
- ✅ No end_time (incomplete)
- ✅ No account_id (not assigned)
- ✅ No clickup_id (no task created)

## Database Tables

### Supabase
- `prf_project_submissions` (read/delete)

## Query Logic
```sql
WHERE start_time < NOW() - INTERVAL '4 days'
  AND end_time IS NULL
  AND account_id IS NULL 
  AND clickup_id IS NULL
ORDER BY start_time ASC
```

## Use Cases
- Prevent database bloat
- Remove abandoned submissions
- Clean up failed form submissions
- Maintain database performance
- Remove test submissions

## Safety Features
- **4-Day Window:** Ensures legitimate submissions aren't deleted prematurely
- **Multiple Criteria:** Requires ALL conditions to prevent accidental deletion
- **No Active Data:** Only removes submissions that never progressed

## Data Retrieved Before Deletion
Provides visibility into what's being deleted:
- Submission identifiers
- Project type and name
- Timestamps
- Age calculation
- Folder paths (if any)

## Notes
- Conservative approach with 4-day window
- Won't delete submissions that have:
  - Been assigned to account
  - Had tasks created
  - Been marked as complete
- Useful for cleaning up:
  - Form testing
  - Abandoned submissions
  - Failed webhook payloads
  - Incomplete integrations
- Runs daily to maintain database health

