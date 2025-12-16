---
title: "backfill-tasks"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["backfill", "clickup", "tasks", "migration", "history"]
version: "116"
---

# backfill-tasks

Bulk imports ClickUp tasks and their history into Supabase database tables with comprehensive data processing.

## Overview
**Function Slug:** backfill-tasks  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 116

## Purpose
Fetches tasks from ClickUp API within a date range and imports them into multiple Supabase tables including tasks, status_history, assignee_history, tag_history, due_date_history, time_estimate_history, time_tracking_history, and go_live_dates.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/backfill-tasks
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
| `created_at_gt` | string | Yes | Start date (ISO format or parseable date) |
| `created_at_lt` | string | Yes | End date (ISO format or parseable date) |
| `limit` | number | No | Maximum tasks to process (default: all) |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/backfill-tasks \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "created_at_gt": "2024-01-01",
    "created_at_lt": "2024-12-31",
    "limit": 1000
  }'
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/backfill-tasks', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    created_at_gt: '2024-01-01T00:00:00Z',
    created_at_lt: '2024-12-31T23:59:59Z',
    limit: 500
  })
})

const data = await response.json()
console.log(`Processed ${data.tasks_processed} tasks`)
console.log(`Skipped ${data.tasks_skipped} existing tasks`)
console.log(`Results:`, data.results)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "tasks_processed": 847,
  "tasks_skipped": 23,
  "pages_processed": 9,
  "date_range": {
    "start": "2024-01-01",
    "end": "2024-12-31",
    "unix_start": 1704067200000,
    "unix_end": 1735689599000
  },
  "results": {
    "tasks": 847,
    "status_history": 847,
    "assignee_history": 1234,
    "tag_history": 2156,
    "due_date_history": 623,
    "time_estimate_history": 445,
    "time_tracking_history": 189,
    "go_live_dates": 156,
    "errors": []
  }
}
```

### Error Responses

**405 Method Not Allowed**
```json
{
  "error": "Method not allowed. Use POST."
}
```

**400 Bad Request** - Missing dates
```json
{
  "error": "Missing required parameters. Please provide created_at_gt and created_at_lt date strings."
}
```

**400 Bad Request** - Invalid date format
```json
{
  "error": "Invalid date format. Please provide dates in a standard format (e.g., \"2023-01-01\" or \"2023-01-01T00:00:00Z\")."
}
```

**500 Internal Server Error** - API key missing
```json
{
  "error": "ClickUp API key is not configured. Please set the CLICKUP_API_KEY environment variable."
}
```

## ClickUp API Integration

### Endpoint
```
GET https://api.clickup.com/api/v2/team/1235435/task
```

### Query Parameters
- `page` - Page number (0-indexed)
- `limit` - Tasks per page (max 100)
- `space_ids[]` - Filter by space IDs
- `date_created_gt` - Start timestamp (milliseconds)
- `date_created_lt` - End timestamp (milliseconds)
- `include_closed` - Include archived tasks

### Allowed Spaces
- `1306092`
- `1301552`

## Database Tables Updated

### tasks
```sql
INSERT INTO tasks (
  task_id,
  name,
  list_id,
  created_at,
  row_created,
  task_archived,
  linked_tasks
)
```

### status_history
```sql
INSERT INTO status_history (
  task_id,
  status_after,
  status_before,
  changed_at,
  row_id,
  changed_by
)
```

### assignee_history
```sql
INSERT INTO assignee_history (
  task_id,
  assignee,
  change_type,
  changed_at,
  row_id,
  updated_by
)
```

### tag_history
```sql
INSERT INTO tag_history (
  task_id,
  tag_after,
  tag_before,
  created_at,
  row_id
)
```

### due_date_history
```sql
INSERT INTO due_date_history (
  task_id,
  due_date_after,
  due_date_before,
  created_at,
  row_id
)
```

### time_estimate_history
```sql
INSERT INTO time_estimate_history (
  task_id,
  estimate_mins_after,
  estimate_mins_before,
  created_at,
  row_id
)
```

### time_tracking_history
```sql
INSERT INTO time_tracking_history (
  task_id,
  total_minutes,
  created_at,
  start_time,
  end_time,
  row_id,
  tracking_id
)
```

### go_live_dates
```sql
INSERT INTO go_live_dates (
  task_id,
  go_live_date,
  created_at,
  id
)
```

## Go Live Date Parsing

### Pattern Matching
Extracts dates from task descriptions:
```
GoLive Date: Friday — 04.04.2025
```

### Regex Pattern
```typescript
/GoLive Date:\s*(.*?)(?:\n|$)/i
```

### Date Format
- Input: `04.04.2025` (DD.MM.YYYY)
- Output: `2025-04-04` (YYYY-MM-DD)

## Pagination

### Page-by-Page Processing
- Processes 100 tasks per page
- Continues until limit reached or no more tasks
- Tracks total processed vs limit
- Skips existing tasks efficiently

### Limit Handling
- If limit provided: processes up to limit
- If no limit: processes all tasks
- Respects limit across all pages
- Stops when limit reached

## Duplicate Prevention

### Existing Task Check
```typescript
// Batch check for existing tasks
const existingTaskIds = await getExistingTasks(taskIds)

// Skip if already exists
if (existingTaskIds.includes(task.id)) {
  results.skipped++
  continue
}
```

### Upsert Strategy
All tables use upsert with conflict resolution:
- `tasks`: `onConflict: 'task_id'`
- History tables: `onConflict: 'row_id'`
- Prevents duplicate inserts

## Data Transformations

### Timestamps
- ClickUp: Milliseconds since epoch
- Database: ISO 8601 strings
- Conversion: `new Date(timestamp).toISOString()`

### Time Estimates
- ClickUp: Microseconds
- Database: Minutes
- Conversion: `Math.round(microseconds / 60000)`

### Time Tracking
- ClickUp: Microseconds total
- Database: Minutes
- Assumes 1-hour session before creation
- Conversion: `microseconds / 60000`

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key |
| `CLICKUP_API_KEY` | Yes | ClickUp API key |

## Use Cases

1. **Initial Migration:** Import all historical tasks
2. **Date Range Import:** Import tasks for specific period
3. **Incremental Sync:** Backfill missing tasks
4. **Data Recovery:** Restore deleted tasks
5. **Analytics Setup:** Populate history tables

## Performance Considerations

### Batch Processing
- Processes 100 tasks per page
- Checks existing tasks in batch
- Upserts multiple records efficiently

### Time Estimates
- Large date ranges may take minutes
- Limit parameter helps control execution time
- Edge Function timeout: 60 seconds

### Optimization Tips
- Use smaller date ranges for large imports
- Set limit to prevent timeouts
- Run multiple smaller batches
- Monitor execution logs

## Error Handling

### Per-Task Errors
- Errors logged but don't stop processing
- Continues with next task
- Errors collected in results array

### Validation Errors
- Date format validation
- Date range validation (auto-adjusts)
- API key validation
- Method validation

## Notes

- Only processes tasks from allowed spaces
- Includes archived/closed tasks
- Skips tasks that already exist
- Processes all history tables for each task
- Go live dates extracted from descriptions
- Time tracking assumes 1-hour sessions
- Default user ID: `4463869` for changes
- Linked tasks stored as array
- Pagination handles large datasets
- Date range auto-adjusts if reversed
- Limit of 0 returns success with no processing
- Comprehensive error collection per task
- Results include detailed breakdown by table

