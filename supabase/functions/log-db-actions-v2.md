---
title: "log-db-actions-v2"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["dropbox", "logging", "v2", "improved"]
version: "97"
---

# log-db-actions-v2

Improved version of Dropbox team log sync with individual event processing to avoid duplicate key conflicts.

## Overview
**Function Slug:** log-db-actions-v2  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 97

## Purpose
Enhanced version of `log-dropbox-actions` that processes events one-by-one instead of batch upsert to prevent duplicate key conflicts. Fetches Dropbox team log events and syncs to database.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/log-db-actions-v2
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/log-db-actions-v2 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "eventsProcessed": 47,
  "rowsUpserted": 47,
  "message": "Successfully processed 47 events and upserted 47 rows"
}
```

## Improvements Over v1

### Individual Event Processing
- Processes events one-by-one instead of batch
- Uses `.single()` for each upsert
- Prevents duplicate key conflicts
- Better error handling per event

### Error Handling
- Continues processing on individual failures
- Logs errors per event
- Tracks successful upserts separately
- More resilient to data issues

## Differences from log-dropbox-actions

| Feature | log-dropbox-actions | log-db-actions-v2 |
|---------|-------------------|-------------------|
| Upsert Strategy | Batch insert | One-by-one |
| Error Handling | Fails entire batch | Continues per event |
| Conflict Handling | Batch upsert | Individual `.single()` |
| Resilience | Lower | Higher |

## Notes

- Same core functionality as v1
- Processes events individually for better reliability
- Handles duplicate key conflicts gracefully
- Same time window (20 minutes)
- Same account extraction logic
- Better suited for production use
- More resilient to data conflicts



