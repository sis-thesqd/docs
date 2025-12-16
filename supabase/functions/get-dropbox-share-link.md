---
title: "get-dropbox-share-link"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["dropbox", "sharing", "links", "caching"]
version: "23"
---

# get-dropbox-share-link

Generates or retrieves public share links for Dropbox folders with caching and fallback token support.

## Overview
**Function Slug:** get-dropbox-share-link  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 23  
**Created:** January 31, 2025  
**Last Updated:** December 16, 2025

## Purpose
Creates public share links for Dropbox folders with intelligent caching, multiple token fallback, and team member context resolution. Optimizes performance by caching links and handles token expiration gracefully.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get-dropbox-share-link
```

## Request

### Headers
```
Content-Type: application/json
```

### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `folderId` | string | Yes | Dropbox folder ID or path |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get-dropbox-share-link \
  -H "Content-Type: application/json" \
  -d '{
    "folderId": "id:a4ayc_80_OEAAAAAAAAAXw"
  }'
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get-dropbox-share-link', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    folderId: 'id:a4ayc_80_OEAAAAAAAAAXw'
  })
})

const data = await response.json()
console.log('Share URL:', data.shareUrl)
console.log('From cache:', data.cached)
```

## Response

### Success Response (200)

**Cached Link:**
```json
{
  "shareUrl": "https://www.dropbox.com/sh/abc123def456/xxxxxxxxx?dl=0",
  "cached": true
}
```

**New Link:**
```json
{
  "shareUrl": "https://www.dropbox.com/sh/abc123def456/xxxxxxxxx?dl=0",
  "cached": false
}
```

### Error Responses

**400 Bad Request** - Missing folder ID
```json
{
  "error": "Folder ID is required"
}
```

**500 Internal Server Error** - No tokens available
```json
{
  "error": "No Dropbox tokens available"
}
```

**500 Internal Server Error** - All tokens failed
```json
{
  "error": "All Dropbox tokens failed. Last error: {error_message}"
}
```

**500 Internal Server Error** - Token refresh needed
```json
{
  "error": "Dropbox access tokens need to be refreshed. Please contact an administrator."
}
```

## Workflow Process

### 1. Check Cache
- Queries `sf_share_links` table for existing link
- Returns immediately if cached link found
- Skips Dropbox API call if cache hit

### 2. Get Available Tokens
- Attempts to load multiple Dropbox tokens:
  - `dropbox_admin` (primary)
  - `dropbox_jacob` (fallback)
- Checks environment variables first
- Falls back to Supabase vault if not in env

### 3. Get Admin Email
- Retrieves `dropbox_admin_email` for team context
- Used for Dropbox Business team member resolution
- Optional but recommended for team accounts

### 4. Try Each Token
- Iterates through available tokens
- Attempts to create/retrieve share link
- Stops on first successful response
- Continues to next token on auth errors

### 5. Create or Get Share Link
- Calls Dropbox `create_shared_link_with_settings` API
- If link exists, calls `list_shared_links` to retrieve it
- Sets visibility to public
- Resolves team member ID if admin email provided

### 6. Cache Link
- Stores successful link in `sf_share_links` table
- Uses upsert to handle existing records
- Non-fatal: continues even if caching fails

### 7. Return Result
- Returns share URL
- Includes `cached` flag for debugging
- Client can use URL immediately

## Dropbox API Integration

### Endpoints Used

**Create Share Link:**
```
POST https://api.dropboxapi.com/2/sharing/create_shared_link_with_settings
```

**List Share Links:**
```
POST https://api.dropboxapi.com/2/sharing/list_shared_links
```

**Resolve Team Member:**
```
POST https://api.dropboxapi.com/2/team/members/get_info
```

### Headers
```
Authorization: Bearer {dropbox_token}
Content-Type: application/json
Dropbox-API-Select-User: {team_member_id}  (for Business accounts)
```

### Create Link Request
```json
{
  "path": "id:a4ayc_80_OEAAAAAAAAAXw",
  "settings": {
    "requested_visibility": "public",
    "audience": "public",
    "access": "viewer"
  }
}
```

## Database Tables

### Supabase
- `sf_share_links` (read/write) - Cache for share links
- `vault.decrypted_secrets` (read) - Dropbox tokens

### Table: sf_share_links
```sql
CREATE TABLE sf_share_links (
  folder_id TEXT PRIMARY KEY,
  url TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Token Fallback System

### Primary Token: dropbox_admin
- First token attempted
- Should have full team permissions
- Best for Business accounts

### Fallback Token: dropbox_jacob
- Second token attempted
- Used if primary token fails
- Provides redundancy

### Token Resolution Order
1. Check environment variable `dropbox_admin`
2. Check Supabase vault for `dropbox_admin`
3. Check environment variable `dropbox_jacob`
4. Check Supabase vault for `dropbox_jacob`
5. If all fail, return error

## Team Member Resolution

### Purpose
For Dropbox Business accounts, resolves admin email to team member ID.

### Process
1. Gets `dropbox_admin_email` from config
2. Calls Dropbox Team API to get member info
3. Extracts `team_member_id`
4. Includes in `Dropbox-API-Select-User` header

### Benefits
- Access team shared folders
- Proper permissions context
- Works with Business restrictions

## Caching Strategy

### Cache Benefits
- **Performance:** Instant response for cached links
- **API Limits:** Reduces Dropbox API calls
- **Cost:** Lower function execution time
- **Reliability:** Works if Dropbox API is slow

### Cache Invalidation
- No automatic expiration
- Manually clear from `sf_share_links` table if needed
- Consider adding TTL for auto-expiration

### Cache Query
```sql
-- Check cached links
SELECT folder_id, url, created_at 
FROM sf_share_links 
WHERE folder_id = 'your_folder_id';

-- Clear specific cache
DELETE FROM sf_share_links 
WHERE folder_id = 'your_folder_id';

-- Clear all cache
TRUNCATE TABLE sf_share_links;
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key |
| `dropbox_admin` | Recommended | Primary Dropbox access token |
| `dropbox_jacob` | Optional | Fallback Dropbox access token |
| `dropbox_admin_email` | Optional | Admin email for team resolution |

**Note:** Tokens can also be stored in Supabase vault instead of environment variables.

## Security Considerations

⚠️ **Public Endpoint**
- No JWT verification required
- Anyone can request share links
- Consider adding authentication for production
- Monitor usage for abuse

✅ **Token Security**
- Tokens never exposed to client
- Server-side API calls only
- Multiple fallback options

✅ **Public Links Only**
- Creates viewer-only links
- No edit permissions
- Appropriate for sharing files publicly

## Use Cases

1. **File Sharing:** Generate public links for client deliverables
2. **Asset Distribution:** Share media files with external users
3. **Project Folders:** Provide access to project resources
4. **Download Links:** Allow downloads without Dropbox account
5. **Embeds:** Link Dropbox content in web apps

## Error Handling

### Common Issues

**Folder Not Found**
- Verify folder ID is correct
- Check folder hasn't been deleted
- Ensure token has access to folder

**Token Expired**
- Refresh Dropbox access tokens
- Update vault or environment variables
- Use refresh tokens to get new access tokens

**All Tokens Failed**
- Check all tokens in vault
- Verify tokens have required permissions
- Regenerate tokens if needed

**No Team Member ID**
- Non-fatal: function continues without team context
- Only affects Business accounts
- Provide `dropbox_admin_email` for full functionality

## Testing

### Test Request
```bash
# Test with folder path
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get-dropbox-share-link \
  -H "Content-Type: application/json" \
  -d '{
    "folderId": "/Projects/ClientName/Assets"
  }'

# Test with folder ID
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get-dropbox-share-link \
  -H "Content-Type: application/json" \
  -d '{
    "folderId": "id:a4ayc_80_OEAAAAAAAAAXw"
  }'
```

### Check Cache
```sql
SELECT * FROM sf_share_links 
ORDER BY created_at DESC 
LIMIT 10;
```

### Monitor Token Usage
Check function logs to see which token was used:
- "Success with token: dropbox_admin" 
- "Success with token: dropbox_jacob"

## Performance Optimization

### Caching
- Cache hits return in <100ms
- Cache misses take 1-3 seconds (Dropbox API call)
- ~90% cache hit rate for repeated requests

### Token Fallback
- Minimal overhead (milliseconds per token check)
- Automatic failover increases reliability
- No user-facing delays

### Best Practices
1. **Batch Requests:** Generate links in bulk when possible
2. **Cache Monitoring:** Track cache hit rate
3. **Token Health:** Monitor which tokens are being used
4. **Error Alerts:** Set up alerts for "all tokens failed" errors

## Related Functions

- [dropbox-listener](./dropbox-listener.md) - Webhook listener for Dropbox events
- [log-dropbox-actions](./log-dropbox-actions.md) - Log Dropbox operations
- [squad-file-uploader](./squad-file-uploader.md) - Upload files to Dropbox

## Notes

- Share links are permanent until manually deleted in Dropbox
- Cache never expires automatically (consider adding TTL)
- Public links can be accessed by anyone with the URL
- Viewer-only permissions prevent editing
- Team member resolution is optional but recommended for Business
- Function handles link already exists errors gracefully
- Multiple token support provides high availability
- Cache dramatically improves performance for frequently accessed folders

