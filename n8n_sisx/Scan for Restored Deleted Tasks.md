---
title: "Scan for Restored Deleted Tasks"
author: "Jacob Vendramin"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["supabase"]
version: "1.0.0"
---

# Scan for Restored Deleted Tasks

> **Workflow ID:** `EHvKpkl54FMFhv2a`
> **Instance:** sisx
> **URL:** [https://sisx.thesqd.com/workflow/EHvKpkl54FMFhv2a](https://sisx.thesqd.com/workflow/EHvKpkl54FMFhv2a)

Automatically detects when deleted ClickUp tasks have been restored and cleans up the tracking records from the database.

---

## Overview

When a task is deleted in ClickUp, it gets logged in the `task_deletions` table. However, ClickUp allows users to restore deleted tasks from the trash. This workflow runs every minute to check if any recently deleted tasks have been restored, and if so, removes them from the deletion tracking table to keep the database in sync.

---

## How It Works

### Trigger

**Schedule Trigger** - Runs every minute to continuously monitor for restored tasks.

### Process Flow

1. **Fetch Recent Deletions**
   - Queries the `task_deletions` table for all records where `deleted_at` is within the last 4 hours
   - This time window ensures recently deleted tasks are checked without querying the entire history

2. **Loop Through Each Task**
   - Processes tasks one at a time using a batch loop
   - Prevents rate limiting issues with the ClickUp API

3. **Check Task Status in ClickUp**
   - Attempts to fetch each task from ClickUp using its ID
   - If the task exists, it means it was restored
   - If the task doesn't exist, it's still deleted (no action needed)

4. **Clean Up Restored Tasks**
   - When a task is found in ClickUp (restored), the corresponding record is deleted from `task_deletions`
   - Tasks that are still deleted remain in the tracking table

---

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| Schedule Trigger | `scheduleTrigger` | Triggers workflow every minute |
| Get many rows | `supabase` | Fetches task deletions from last 4 hours |
| Loop Over Items | `splitInBatches` | Processes tasks one at a time |
| Get a task | `clickUp` | Checks if task exists in ClickUp |
| Edit Fields | `set` | Adds `task_found` boolean flag |
| Merge | `merge` | Combines original data with task status |
| If | `if` | Routes based on whether task was found |
| Delete a row | `supabase` | Removes restored tasks from tracking table |

---

## Supabase Tables

### task_deletions

Tracks ClickUp tasks that have been deleted.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| `id` | text | NO | - | The ClickUp task ID |
| `deleted_at` | timestamptz | NO | `now()` | Timestamp when the task was deleted |

---

## Credentials

| Service | Credential Name |
|---------|-----------------|
| Supabase | Squad Data |
| ClickUp | TheSquad |

---

## Error Handling

- The **Get a task** node has `onError: continueRegularOutput` enabled, allowing the workflow to continue even when a task is not found (which is expected for truly deleted tasks)
- Tasks that fail to fetch are assumed to still be deleted and are skipped

---

## Related Workflows

This workflow is part of the task deletion tracking system. Related components include:

- Workflows that populate the `task_deletions` table when tasks are deleted
- Any cleanup or archival processes that depend on deletion status

---

## Maintenance Notes

- The 4-hour lookback window balances thoroughness with performance
- Running every minute ensures restored tasks are detected quickly
- No manual intervention required under normal operation
