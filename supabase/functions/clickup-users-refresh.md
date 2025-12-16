---
title: "clickup-users-refresh"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["clickup", "sync", "users", "database"]
version: "93"
---

# clickup-users-refresh

Synchronizes ClickUp team members with the local database, updating user information and deactivating removed users.

## Overview
**Function Slug:** clickup-users-refresh  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 93

## Purpose
Keeps the `clickup_users` table in sync with ClickUp by fetching all team members, upserting active users, and marking inactive users who have been removed from the ClickUp team.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-users-refresh
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-users-refresh \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-users-refresh', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const data = await response.json()
console.log(`Upserted ${data.results.upserted} users`)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "results": {
    "totalClickUpUsers": 45,
    "upserted": 45,
    "deactivated": 3,
    "errors": []
  },
  "timestamp": "2025-12-16T10:30:00.000Z"
}
```

### Partial Success (200)

```json
{
  "success": false,
  "results": {
    "totalClickUpUsers": 45,
    "upserted": 43,
    "deactivated": 2,
    "errors": [
      "Failed to deactivate user 12345: ...",
      "Upsert error: ..."
    ]
  },
  "timestamp": "2025-12-16T10:30:00.000Z"
}
```

### Error Response (500)

```json
{
  "success": false,
  "error": "CLICKUP_API_KEY not configured"
}
```

## Workflow Process

1. **Fetch ClickUp Users** - Retrieves all members from team ID 1235435
2. **Get Existing Users** - Queries local `clickup_users` table
3. **Prepare Upserts** - Formats user data with converted timestamps
4. **Identify Inactive** - Finds users in DB but not in ClickUp
5. **Upsert Active Users** - Updates or inserts current team members
6. **Deactivate Removed** - Sets `active: false` for removed users
7. **Return Results** - Reports success/errors for each operation

## ClickUp API

### Team Endpoint
```
GET https://api.clickup.com/api/v2/team/1235435
```

### Headers
```
Authorization: {CLICKUP_API_KEY}
Content-Type: application/json
```

### Response Format
```json
{
  "team": {
    "members": [
      {
        "user": {
          "id": 123456,
          "username": "john.doe",
          "email": "john@example.com",
          "profilePicture": "https://...",
          "date_invited": "1640000000000",
          "date_joined": "1640100000000",
          "last_active": "1734350000000"
        }
      }
    ]
  }
}
```

## Database Table

### Table: clickup_users

```sql
CREATE TABLE clickup_users (
  clickup_id BIGINT PRIMARY KEY,
  email TEXT,
  username TEXT,
  profile_picture TEXT,
  date_invited TIMESTAMP,
  date_joined TIMESTAMP,
  last_active TIMESTAMP,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key |
| `CLICKUP_API_KEY` | Yes | ClickUp API key |

## Use Cases

1. **User Sync:** Keep local user list current
2. **Scheduled Jobs:** Run daily to catch team changes
3. **Onboarding:** Update new team member information
4. **Offboarding:** Mark departed members as inactive
5. **Profile Updates:** Sync profile picture and username changes

## Notes

- Team ID is hardcoded to `1235435`
- Timestamps are converted from ClickUp milliseconds to ISO format
- Invalid timestamps are logged as warnings but don't stop the sync
- Users removed from ClickUp are marked inactive, not deleted
- Upsert uses `clickup_id` as conflict resolution
- Function returns partial success if some operations fail

