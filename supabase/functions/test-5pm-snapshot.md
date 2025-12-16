---
title: "test-5pm-snapshot"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["test", "snapshots", "5pm", "simulation"]
version: "129"
---

# test-5pm-snapshot

Test function that simulates a 5:00 PM automated snapshot to verify snapshot creation logic.

## Overview
**Function Slug:** test-5pm-snapshot  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 129

## Purpose
Testing function that simulates a 5:00 PM snapshot execution. Verifies that snapshot automation logic works correctly at scheduled times.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/test-5pm-snapshot
```

## Request

### Headers
```
Content-Type: application/json
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/test-5pm-snapshot
```

## Response

### Success Response (200)

```json
{
  "message": "TEST: Created 15 automated snapshots for simulated 5:00 PM",
  "simulated_time": "17:00",
  "period": "PM",
  "snapshot_time": "2025-12-16T17:00:00.000Z",
  "graphic_designers": 8,
  "video_designers": 7,
  "total_snapshots": 15,
  "inserted_snapshots": 15
}
```

## Test Process

### 1. Check Settings
- Fetches automation settings
- Verifies automation is active
- Checks configured intervals

### 2. Simulate 5:00 PM
- Sets simulated time to "17:00"
- Checks if time matches configured intervals
- Proceeds if interval matches

### 3. Create Snapshots
- Fetches workload data for both teams
- Creates snapshots with PM period
- Inserts into database

## Notes

- Test/debugging function
- Simulates 5:00 PM time
- Only runs if "17:00" is in configured intervals
- Creates real snapshots in database
- Useful for testing automation logic
- Public endpoint (no auth required)
- Period set to "PM" for 17:00
- Same snapshot format as production

