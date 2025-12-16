---
title: "backfill-v2"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["backfill", "clickup", "tasks", "v2", "improved"]
version: "94"
---

# backfill-v2

Improved version of ClickUp task backfill with better logging, batch processing, and error handling.

## Overview
**Function Slug:** backfill-v2  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 94

## Purpose
Enhanced version of `backfill-tasks` with improved logging, batch processing of existing task checks, and better error handling. Imports ClickUp tasks and their history into Supabase.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/backfill-v2
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
Content-Type: application/json
```

### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `created_at_gt` | string | Yes | Start date (ISO format) |
| `created_at_lt` | string | Yes | End date (ISO format) |
| `limit` | number | No | Maximum tasks to process |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/backfill-v2 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "created_at_gt": "2024-01-01",
    "created_at_lt": "2024-12-31",
    "limit": 1000
  }'
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "total_tasks_found": 847,
  "tasks_processed": 824,
  "tasks_success": 824,
  "tasks_error": 0,
  "existing_tasks": 23,
  "date_range": {
    "start": "2024-01-01T00:00:00.000Z",
    "end": "2024-12-31T23:59:59.999Z"
  },
  "pages_fetched": 9
}
```

## Improvements Over v1

### Enhanced Logging
- Timestamped log messages
- Log prefix: `[ClickUp Backfill]`
- Detailed progress tracking
- Error logging per task

### Batch Processing
- Checks existing tasks in batches of 500
- Handles large task sets efficiently
- Prevents Postgres IN clause limits

### Better Error Handling
- Continues processing on individual task errors
- Tracks success/error counts separately
- Detailed error messages

### Collection Strategy
- Collects all tasks first
- Then processes in batch
- More efficient than page-by-page processing

## Differences from backfill-tasks

| Feature | backfill-tasks | backfill-v2 |
|---------|----------------|-------------|
| Logging | Basic | Enhanced with timestamps |
| Existing Check | Per page | Batch (500 at a time) |
| Processing | Page-by-page | Collect then process |
| Error Tracking | Basic | Detailed counts |
| Task Collection | Immediate process | Collect all first |

## Notes

- Same core functionality as `backfill-tasks`
- Improved logging and error handling
- Batch processing for better performance
- Skips existing tasks automatically
- Processes same tables as v1
- Same date range and limit support
- Better suited for large backfills

