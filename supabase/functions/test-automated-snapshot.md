---
title: "test-automated-snapshot"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["test", "snapshots", "automated", "wrapper"]
version: "120"
---

# test-automated-snapshot

Test wrapper function that calls `automated-snapshots` with test parameters to verify functionality.

## Overview
**Function Slug:** test-automated-snapshot  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 120

## Purpose
Testing function that invokes `automated-snapshots` with simulated parameters. Used to test snapshot automation without waiting for scheduled execution.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/test-automated-snapshot
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/test-automated-snapshot \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Response

### Success Response (200)

```json
{
  "message": "Test completed successfully",
  "automated_snapshots_response": {
    "success": true,
    "snapshot_count": 15,
    "graphic_designers": 8,
    "video_designers": 7,
    "total_tasks": 234,
    "snapshot_time": "2025-12-16T09:00:00.000Z",
    "source": "manual_test",
    "time_period": "morning"
  }
}
```

## Test Parameters

### Simulated Data
```json
{
  "source": "manual_test",
  "scheduled_time": "09:00"
}
```

## Notes

- Wrapper around `automated-snapshots` function
- Calls function with test parameters
- Simulates 9:00 AM scheduled time
- Returns response from automated-snapshots
- Useful for testing without cron
- Requires JWT authentication
- See [automated-snapshots](./automated-snapshots.md) for details



