---
title: "automated-snapshots"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["snapshots", "workload", "employees", "tasks", "monitoring"]
version: "136"
---

# automated-snapshots

Creates automated snapshots of employee workload by capturing task counts for graphic and video designers at a specific point in time.

## Overview
**Function Slug:** automated-snapshots  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 136

## Purpose
Generates workload snapshots for all employees in Graphic Design and Video departments. Captures task counts by status (received, deliverables needed, needs update) and out-of-office tasks. Stores snapshots in `workload_snapshots` table for historical tracking and analysis.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/automated-snapshots
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
Content-Type: application/json
```

### Body Parameters (Optional)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `source` | string | No | Source of snapshot (default: "manual") |
| `target_timezone` | string | No | Target timezone for snapshot |
| `scheduled_time` | string | No | Scheduled execution time |
| `time_period` | string | No | Time period identifier (e.g., "morning", "afternoon") |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/automated-snapshots \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "source": "scheduled",
    "time_period": "morning",
    "target_timezone": "America/New_York"
  }'
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/automated-snapshots', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    source: 'scheduled',
    time_period: 'morning',
    target_timezone: 'America/New_York'
  })
})

const data = await response.json()
console.log(`Created ${data.snapshot_count} snapshots`)
console.log(`Total tasks: ${data.total_tasks}`)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "snapshot_count": 15,
  "graphic_designers": 8,
  "video_designers": 7,
  "total_tasks": 234,
  "snapshot_time": "2025-12-16T10:30:00.000Z",
  "source": "scheduled",
  "time_period": "morning",
  "target_timezone": "America/New_York"
}
```

### Error Responses

**500 Internal Server Error** - No employees
```json
{
  "success": false,
  "message": "No employees found"
}
```

**500 Internal Server Error** - Database error
```json
{
  "success": false,
  "error": "...",
  "stack": "..."
}
```

## Workflow Process

### 1. Fetch Employees
- Queries `employees` table
- Filters by department (Graphic Design, Video)
- Gets employee metadata

### 2. Process Graphic Designers
- Calls `get_designer_tasks_by_status_v3` for each status
- Calls `get_designer_ooo_tasks_v8` for OOO tasks
- Calculates total task counts
- Checks out-of-office status

### 3. Process Video Designers
- Calls `get_video_designer_tasks_by_status_v3` for each status
- Calls `get_video_designer_ooo_tasks_v8` for OOO tasks
- Calculates total task counts
- Checks out-of-office status

### 4. Store Snapshots
- Inserts all snapshot records
- Single batch insert for efficiency
- Stores timestamp and metadata

## Database Functions Used

### Graphic Designers
- `get_designer_tasks_by_status_v3(status, designer_id)`
- `get_designer_ooo_tasks_v8(designer_id)`

### Video Designers
- `get_video_designer_tasks_by_status_v3(status, designer_id)`
- `get_video_designer_ooo_tasks_v8(designer_id)`

### Task Statuses Tracked
- **received** - Tasks received by designer
- **deliverables needed** - Tasks needing deliverables
- **needs an update** - Tasks requiring updates

## Database Table

### Table: workload_snapshots
```sql
CREATE TABLE workload_snapshots (
  employee_id TEXT NOT NULL,
  employee_name TEXT,
  total_tasks INTEGER,
  received_tasks INTEGER,
  deliverables_tasks INTEGER,
  needs_update_tasks INTEGER,
  ooo_received_tasks INTEGER,
  ooo_deliverables_tasks INTEGER,
  ooo_needs_update_tasks INTEGER,
  snapshot_time TIMESTAMP NOT NULL,
  time_period TEXT,
  snapshot_type TEXT DEFAULT 'automated',
  team_type TEXT, -- 'graphic' or 'video'
  out_of_office BOOLEAN,
  employee_timezone TEXT,
  local_time_period TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Snapshot Data Structure

### Per Employee Snapshot
```json
{
  "employee_id": "123456",
  "employee_name": "John Doe",
  "total_tasks": 15,
  "received_tasks": 5,
  "deliverables_tasks": 7,
  "needs_update_tasks": 3,
  "ooo_received_tasks": 2,
  "ooo_deliverables_tasks": 1,
  "ooo_needs_update_tasks": 0,
  "snapshot_time": "2025-12-16T10:30:00.000Z",
  "time_period": "morning",
  "snapshot_type": "automated",
  "team_type": "graphic",
  "out_of_office": false,
  "employee_timezone": "America/New_York",
  "local_time_period": "morning"
}
```

## Task Count Calculations

### Total Tasks
```
total_tasks = received_tasks + deliverables_tasks + needs_update_tasks
```

### OOO Tasks
Separate counts for tasks assigned while out of office:
- `ooo_received_tasks`
- `ooo_deliverables_tasks`
- `ooo_needs_update_tasks`

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key |

## Use Cases

1. **Workload Monitoring:** Track employee workload over time
2. **Capacity Planning:** Analyze task distribution
3. **Performance Analysis:** Identify workload trends
4. **Scheduling:** Optimize task assignments
5. **Reporting:** Generate workload reports

## Scheduled Execution

### Recommended Schedule
- Morning snapshot (e.g., 9:00 AM)
- Afternoon snapshot (e.g., 2:00 PM)
- End of day snapshot (e.g., 5:00 PM)

### Time Period Values
- `morning` - Early day snapshot
- `afternoon` - Mid-day snapshot
- `evening` - End of day snapshot
- `night` - Late day snapshot

## Notes

- Processes all employees in Graphic Design and Video departments
- Uses same RPC functions as main application
- Parallel processing for task fetching (Promise.all)
- Handles errors per employee (continues on failure)
- Snapshot time set at function start (consistent timestamp)
- Out-of-office status checked from employee record
- Timezone information stored for each employee
- Batch insert for all snapshots (single transaction)
- Summary logged with counts and totals
- Source tracking for manual vs scheduled executions
- Time period can be customized per execution
- OOO tasks tracked separately from regular tasks
- Total tasks excludes OOO tasks from main count

