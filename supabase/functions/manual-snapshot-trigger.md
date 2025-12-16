---
title: "manual-snapshot-trigger"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["snapshots", "manual", "workload", "trigger"]
version: "132"
---

# manual-snapshot-trigger

Manually triggers workload snapshots for graphic and video designers, bypassing scheduled automation.

## Overview
**Function Slug:** manual-snapshot-trigger  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 132

## Purpose
Creates workload snapshots on-demand without waiting for scheduled automation. Useful for testing, debugging, or creating snapshots outside normal schedule.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/manual-snapshot-trigger
```

## Request

### Headers
```
Content-Type: application/json
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/manual-snapshot-trigger
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "Manual snapshot created with 15 snapshots",
  "debug_info": {
    "current_time": "14:30",
    "intervals": ["09:00", "17:00"],
    "automation_active": true,
    "should_run_now": false,
    "graphic_designers": 8,
    "video_designers": 7,
    "total_snapshots": 15
  }
}
```

## Workflow Process

### 1. Check Settings
- Fetches `snapshot_automation_settings`
- Checks if automation is active
- Logs current time and intervals

### 2. Force Run Snapshot
- Bypasses time-based checks
- Always creates snapshots regardless of time
- Uses same RPC functions as frontend

### 3. Create Snapshots
- Calls `get_creative_team_workload` for graphic team
- Calls `get_video_team_workload` for video team
- Creates snapshots with comprehensive data
- Inserts into `workload_snapshots` table

## Database Functions Used

- `get_creative_team_workload()` - Graphic designers
- `get_video_team_workload()` - Video designers

## Snapshot Data

### Fields Captured
- Employee ID and name
- Total tasks
- Received tasks
- Deliverables tasks
- Needs update tasks
- OOO task counts (received, deliverables, needs update)
- Snapshot time
- Period (AM/PM)
- Team type (graphic/video)

## Notes

- Public endpoint (no JWT required)
- Forces snapshot creation regardless of time
- Uses same RPC functions as frontend
- Comprehensive debug logging
- Checks cron job history
- Period determined by hour (<17 = AM, >=17 = PM)
- Same snapshot format as automated snapshots
- Useful for testing and debugging

