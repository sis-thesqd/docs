---
title: "test"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["test", "webhook", "debugging"]
version: "94"
---

# test

Simple test function for sending webhook notifications when task status changes.

## Overview
**Function Slug:** test  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 94

## Purpose
Test/debugging function that sends webhook notifications to a test webhook URL when task status changes. Used for testing webhook integrations.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/test
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
| `task_id` | string | Yes | Task ID |
| `status_after` | string | Yes | New status |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/test \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "task_id": "123456",
    "status_after": "completed"
  }'
```

## Response

### Success Response (200)

```
Webhook sent successfully
```

### Error Responses

**405 Method Not Allowed**
```
Method not allowed
```

**500 Internal Server Error**
```
Webhook failed
```

## Webhook Payload

```json
{
  "task_id": "123456",
  "status_after": "completed",
  "message": "Task 123456 status changed to completed"
}
```

## Notes

- Test/debugging function
- Sends webhook to test URL
- Webhook URL hardcoded in function
- Simple status change notification
- Used for testing webhook integrations
- Replace webhook URL for production use
- Not intended for production use



