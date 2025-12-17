---
title: "dropbox-listener"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["dropbox", "webhook", "listener", "folder-creation"]
version: "121"
---

# dropbox-listener

Dropbox webhook listener that processes folder creation events and monitors file changes.

## Overview
**Function Slug:** dropbox-listener  
**Status:** Active  
**JWT Verification:** Disabled (public webhook endpoint)  
**Version:** 121

## Purpose
Receives webhook notifications from Dropbox when files or folders change. Processes folder creation events, tracks changes using cursors, and logs detailed folder metadata. Designed for Dropbox Business team accounts.

## Endpoint

```
GET/POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/dropbox-listener
```

## Request

### GET Request (Webhook Verification)

Dropbox sends a GET request with a challenge parameter to verify the webhook endpoint.

#### Sample Request
```bash
curl -X GET \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/dropbox-listener?challenge=abc123def456"
```

#### Response (200)
```
abc123def456
```

### POST Request (Webhook Notification)

Dropbox sends POST requests when changes occur.

#### Sample Request Body
```json
{
  "list_folder": {
    "accounts": ["dbid:AABo4fq2X3uLimFPIIKWuPS3arV4l31_8fk"]
  }
}
```

#### Sample Request
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/dropbox-listener \
  -H "Content-Type: application/json" \
  -d '{
    "list_folder": {
      "accounts": ["dbid:AABo4fq2X3uLimFPIIKWuPS3arV4l31_8fk"]
    }
  }'
```

## Response

### Success Response (200)

```
Webhook processed successfully
```

### Error Responses

**400 Bad Request** - Invalid verification
```
Invalid verification request
```

**405 Method Not Allowed**
```
Method not allowed
```

**500 Internal Server Error**
```
Error processing webhook: {error_message}
```

## Webhook Flow

### 1. Initial Verification (GET)
- Dropbox sends challenge parameter
- Function echoes challenge back
- Verifies webhook endpoint is accessible

### 2. Change Notification (POST)
- Dropbox sends account IDs with changes
- Function processes each account
- Gets latest cursor for account
- Fetches recent changes
- Identifies folder creation events

### 3. Folder Detection
- Scans changes for folder entries
- Extracts folder metadata
- Logs folder creation details
- Can trigger downstream actions

## Dropbox API Integration

### Token Refresh
```typescript
// Gets new access token using refresh token
const tokenResponse = await fetch('https://api.dropbox.com/oauth2/token', {
  method: 'POST',
  body: formData // grant_type, refresh_token, client_id, client_secret
})
```

### Get Latest Cursor
```typescript
// Gets cursor for current state
const cursorResponse = await fetch('https://api.dropboxapi.com/2/files/list_folder/get_latest_cursor', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${accessToken}` },
  body: JSON.stringify({
    path: '',
    recursive: true,
    include_mounted_folders: true
  })
})
```

### Get Changes
```typescript
// Gets changes since cursor
const changesResponse = await fetch('https://api.dropboxapi.com/2/files/list_folder/continue', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${accessToken}` },
  body: JSON.stringify({ cursor })
})
```

### Get Folder Details
```typescript
// Gets detailed folder metadata
const folderResponse = await fetch('https://api.dropboxapi.com/2/files/get_metadata', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${accessToken}` },
  body: JSON.stringify({ path: folderPath })
})
```

## Folder Creation Detection

### Entry Types
- **folder** - New folder created
- **file** - New file added
- **deleted** - File/folder removed

### Folder Metadata Logged
- Folder ID
- Folder path (display)
- Folder name
- Full metadata JSON

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DROPBOX_APP_KEY` | Yes | Dropbox app key |
| `DROPBOX_APP_SECRET` | Yes | Dropbox app secret |
| `DROPBOX_REFRESH_TOKEN` | Yes | Dropbox refresh token |

## Use Cases

1. **Folder Monitoring:** Track new folder creation
2. **Automation:** Trigger workflows on folder creation
3. **Sync:** Keep external systems in sync with Dropbox
4. **Notifications:** Alert on important folder events
5. **Audit:** Log all folder creation activity

## Webhook Setup

### Dropbox Configuration
1. Go to Dropbox App Console
2. Navigate to webhooks settings
3. Add webhook URL: `https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/dropbox-listener`
4. Select events to monitor
5. Save configuration

### Verification
Dropbox will send GET request with challenge:
- Function must echo challenge back
- Verifies endpoint is accessible
- Required for webhook activation

## Processing Logic

### Multi-User Support
- Handles multiple accounts in webhook payload
- Processes each account separately
- Supports Dropbox Business team accounts

### Cursor Management
- Gets latest cursor for each account
- Uses cursor to fetch only new changes
- Prevents duplicate processing

### Error Handling
- Continues processing other accounts on error
- Logs errors for debugging
- Returns success even if some accounts fail

## Logging

### Console Logs
- Request method and URL
- Account IDs received
- Cursor operations
- Folder detection events
- Error details

### Folder Event Logs
```
FOLDER CREATION EVENT:
Folder ID: id:a4ayc_80_OEAAAAAAAAAXw
Folder Path: /Projects/ClientName/Assets
Folder Name: Assets
Folder Metadata: {...}
```

## Security Considerations

⚠️ **Public Endpoint**
- No authentication required
- Webhook secret not validated
- Anyone can send requests
- Consider adding webhook signature validation

✅ **Best Practices**
- Validate webhook source (IP whitelist)
- Add webhook secret validation
- Rate limit webhook processing
- Monitor for abuse

## Notes

- No webhook secret validation implemented
- Processes all accounts in webhook payload
- Gets latest cursor for each account
- Only processes folder creation events (files ignored)
- Detailed logging for debugging
- Supports Dropbox Business team accounts
- Token refresh handled automatically
- Cursor-based change detection prevents duplicates
- Function continues on errors (doesn't fail entire webhook)
- Folder metadata includes full Dropbox API response



