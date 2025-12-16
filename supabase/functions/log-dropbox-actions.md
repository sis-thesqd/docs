---
title: "log-dropbox-actions"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["dropbox", "logging", "audit", "team-log"]
version: "105"
---

# log-dropbox-actions

Fetches Dropbox team log events and syncs file operations to the database for auditing and tracking.

## Overview
**Function Slug:** log-dropbox-actions  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 105

## Purpose
Retrieves file operation events from Dropbox Team Log API for the past 20 minutes and stores them in the `dropbox_log` table. Extracts account numbers from file paths and tracks user actions.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/log-dropbox-actions
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/log-dropbox-actions \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/log-dropbox-actions', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const data = await response.json()
console.log(`Processed ${data.eventsProcessed} events`)
console.log(`Upserted ${data.rowsUpserted} rows`)
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

### Error Responses

**500 Internal Server Error** - Vault error
```json
{
  "success": false,
  "error": "Failed to retrieve Dropbox API key from vault: ..."
}
```

**500 Internal Server Error** - Dropbox API error
```json
{
  "success": false,
  "error": "Dropbox API error: 401 Unauthorized - Invalid access token"
}
```

**500 Internal Server Error** - Database error
```json
{
  "success": false,
  "error": "Failed to upsert data: ..."
}
```

## Workflow Process

### 1. Get Dropbox Token
- Retrieves access token from Supabase vault
- Uses vault ID: `2b5d8b48-77b6-4d55-8753-c3dc344f1c9b`
- Accesses `vault.decrypted_secrets` table

### 2. Fetch Team Log Events
- Queries Dropbox Team Log API
- Category: `file_operations`
- Time range: Last 20 minutes
- Uses cursor-based pagination

### 3. Process Events
- Filters events with assets
- Extracts file metadata
- Maps to database schema
- Extracts account numbers from paths

### 4. Upsert to Database
- Inserts/updates `dropbox_log` table
- Conflict resolution on `id` field
- Sorts by timestamp (descending)

## Dropbox Team Log API

### Endpoint
```
POST https://api.dropboxapi.com/2/team_log/get_events
```

### Request Body
```json
{
  "category": "file_operations",
  "time": {
    "start_time": "2025-12-16T10:10:00Z"
  }
}
```

### Continue Endpoint
```
POST https://api.dropboxapi.com/2/team_log/get_events/continue
```

### Request Body
```json
{
  "cursor": "cursor_string_from_previous_response"
}
```

## Event Processing

### Event Structure
```typescript
{
  type: string,           // File operation type
  path: string,          // File/folder path
  id: string,            // File ID
  name: string,          // Display name
  user_email: string,    // User email
  user_id: string,       // Team member ID
  timestamp: string      // ISO timestamp
}
```

### Account Extraction
Extracts account number from paths like:
```
/2. Client Accounts/3570 - Church Name/Folder
```

Returns: `3570`

## Database Table

### Table: dropbox_log
```sql
CREATE TABLE dropbox_log (
  id TEXT PRIMARY KEY,              -- File ID from Dropbox
  type TEXT NOT NULL,               -- Operation type
  name TEXT,                        -- File/folder name
  created_by_email TEXT,             -- User email
  created_by TEXT,                   -- Team member ID
  created_at TIMESTAMP NOT NULL,    -- Event timestamp
  path TEXT,                        -- File path
  account INTEGER                   -- Extracted account number
);
```

### Upsert Logic
```typescript
{
  onConflict: 'id',
  ignoreDuplicates: false
}
```

## Time Range

### Default Window
- **Start Time:** 20 minutes ago
- **End Time:** Current time
- **Format:** ISO 8601 without milliseconds

### Calculation
```typescript
const startTime = new Date()
startTime.setMinutes(startTime.getMinutes() - 20)
const formattedTime = startTime.toISOString().split('.')[0] + 'Z'
```

## Account Number Extraction

### Path Pattern
```
/2. Client Accounts/{account_number} - {name}/...
```

### Extraction Logic
1. Check if path contains "2. Client Accounts"
2. Split path after "2. Client Accounts"
3. Extract first segment
4. Match digits with regex
5. Parse as integer
6. Return if > 0, else null

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key |
| Vault secret ID: `2b5d8b48-77b6-4d55-8753-c3dc344f1c9b` | Yes | Dropbox API key |

## Use Cases

1. **Audit Trail:** Track all file operations
2. **Account Tracking:** Link actions to client accounts
3. **User Activity:** Monitor team member actions
4. **Compliance:** Maintain operation logs
5. **Analytics:** Analyze file operation patterns

## Pagination

### Cursor-Based
- Initial request returns cursor
- Continue requests use cursor
- Processes all events until `has_more: false`
- Handles large event sets automatically

### Event Limits
- No explicit limit on events processed
- Continues until all events fetched
- May take time for large datasets

## Error Handling

### Vault Errors
- Logs error details
- Returns descriptive error message
- Fails fast if token unavailable

### API Errors
- Logs status and response body
- Continues processing if possible
- Returns error with context

### Database Errors
- Logs upsert error details
- Returns error message
- Partial success not supported

## Performance

### Processing Time
- ~2-5 seconds for typical runs
- Depends on number of events
- Cursor pagination adds overhead

### Optimization
- Processes events in batches
- Single upsert for all events
- Sorts before upsert for consistency

## Notes

- Fetches last 20 minutes of events
- Requires Dropbox Business account
- Team Log API requires admin permissions
- Account extraction only works for specific path structure
- Events sorted by timestamp (newest first)
- Duplicate events handled via upsert
- Only processes events with assets
- User email and ID extracted from event context
- File ID used as primary key
- Path stored as contextual path from Dropbox
- Function designed for scheduled execution (every 20 minutes)

