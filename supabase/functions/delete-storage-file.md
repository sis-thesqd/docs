---
title: "delete-storage-file"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["storage", "delete", "files", "cleanup"]
version: "6"
---

# delete-storage-file

Deletes files from Supabase Storage buckets using the file's public URL.

## Overview
**Function Slug:** delete-storage-file  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 6

## Purpose
Provides a convenient way to delete files from Supabase Storage by passing the public URL instead of manually parsing bucket and path information.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/delete-storage-file
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
| `url` | string | Yes | Full Supabase Storage URL of the file to delete |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/delete-storage-file \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/public/dropbox-srp-staging/1759791730626ext-check-db-path.mp4"
  }'
```

### JavaScript Example

```javascript
const fileUrl = 'https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/public/dropbox-srp-staging/sample.mp4'

const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/delete-storage-file', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ url: fileUrl })
})

const data = await response.json()
if (data.success) {
  console.log('File deleted successfully')
}
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "File deleted successfully",
  "bucket": "dropbox-srp-staging",
  "path": "1759791730626ext-check-db-path.mp4",
  "data": [
    {
      "name": "1759791730626ext-check-db-path.mp4"
    }
  ]
}
```

### Error Responses

**400 Bad Request** - Missing URL
```json
{
  "error": "URL is required"
}
```

**400 Bad Request** - Invalid URL format
```json
{
  "error": "Invalid storage URL format"
}
```

**500 Internal Server Error** - Delete failed
```json
{
  "error": "Failed to delete file",
  "details": "File not found or insufficient permissions"
}
```

## URL Format

### Expected URL Pattern

```
https://{project_ref}.supabase.co/storage/v1/object/public/{bucket_name}/{file_path}
```

### Examples

**Valid URLs:**
- `https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/public/avatars/user123.jpg`
- `https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/public/documents/folder/file.pdf`
- `https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/public/images/2024/12/photo.png`

**Invalid URLs:**
- Missing `/storage/v1/object/public/` path
- Private bucket URLs (not supported)
- Authenticated URLs with tokens

## Workflow Process

1. **Parse URL** - Extracts bucket name and file path using regex
2. **Validate Format** - Ensures URL matches expected pattern
3. **Initialize Client** - Creates Supabase client with service role
4. **Delete File** - Calls Storage API to remove file
5. **Return Result** - Confirms deletion with bucket and path info

## Storage API

### Internal Call
```typescript
const { data, error } = await supabase.storage
  .from(bucket)
  .remove([filePath])
```

### Permissions
Uses service role key to bypass RLS policies and delete any file.

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key (for admin access) |

## Use Cases

1. **Cleanup:** Remove temporary or outdated files
2. **User Actions:** Delete user-uploaded files
3. **Storage Management:** Automated file lifecycle management
4. **Error Recovery:** Remove corrupted or failed uploads
5. **GDPR Compliance:** Delete user data on request

## Security Considerations

⚠️ **Service Role Access**
- Function uses service role key
- Bypasses Row Level Security (RLS)
- Can delete ANY file in ANY bucket
- Ensure proper JWT authentication

✅ **Best Practices**
1. Verify user has permission to delete the file
2. Log deletions for audit trail
3. Consider soft deletes for important files
4. Implement additional authorization checks
5. Rate limit delete operations

## Common Issues

### File Not Found
- Verify URL is correct
- Check if file was already deleted
- Ensure bucket name is spelled correctly

### Invalid URL Format
- Must include `/storage/v1/object/public/`
- URL must be from Supabase Storage
- Private bucket URLs won't match pattern

### Permission Denied
- Service role key must be valid
- Bucket must exist
- Check bucket policies

## Testing

### Test Delete
```bash
# First, upload a test file
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/test-bucket/test.txt" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: text/plain" \
  -d "test content"

# Then delete it
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/delete-storage-file \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/public/test-bucket/test.txt"
  }'
```

## Notes

- Function only works with public bucket URLs
- Private/authenticated URLs require different handling
- Deletion is permanent - no recycle bin
- Service role bypasses all RLS policies
- URL regex: `/\/storage\/v1\/object\/public\/([^\/]+)\/(.+)$/`
- Returns array of deleted files in `data` field
- Function does not verify user owns the file
- Consider adding ownership checks before deletion

