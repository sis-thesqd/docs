---
title: "upload-to-sb-storage"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["upload", "storage", "supabase", "remix", "library"]
version: "100"
---

# upload-to-sb-storage

Uploads files from URLs to Supabase Storage and syncs metadata to remix_library table.

## Overview
**Function Slug:** upload-to-sb-storage  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 100

## Purpose
Downloads files from external URLs, uploads them to Supabase Storage, and creates/updates records in the remix_library table. Handles filename extraction, content type detection, and automatic extension detection.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/upload-to-sb-storage
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
| `fileUrl` | string | Yes | URL of file to download and upload |
| `bucketName` | string | Yes | Supabase Storage bucket name |
| `fileName` | string | No | Override filename (auto-detected if not provided) |
| `contentType` | string | No | MIME type (auto-detected if not provided) |
| `data` | object | Yes | Remix library metadata object |

### Remix Data Object

```typescript
{
  id: number;              // WCID
  name: string;            // Design name
  file_name: string;       // Original filename
  file_url: string;        // Original file URL
  download: string;        // Download URL
  link: string;            // Page URL
  description: string;     // Text description
  desc_md: string;         // Markdown description
  categories: string[];     // Category tags
  colors: string[];        // Color tags
  keywords: string[];      // Keyword tags
}
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/upload-to-sb-storage \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fileUrl": "https://example.com/designs/christmas-announcement.mp4",
    "bucketName": "remix-library",
    "data": {
      "id": 12345,
      "name": "Christmas Announcement",
      "file_name": "christmas-announcement.mp4",
      "file_url": "https://example.com/designs/christmas-announcement.mp4",
      "download": "https://example.com/download/christmas-announcement.mp4",
      "link": "https://remix.com/designs/christmas-announcement",
      "description": "Festive Christmas announcement design",
      "desc_md": "# Christmas Announcement\n\nFestive design...",
      "categories": ["motion", "announcement"],
      "colors": ["red", "green", "gold"],
      "keywords": ["christmas", "holiday", "festive"]
    }
  }'
```

### JavaScript Example

```javascript
const remixData = {
  id: 12345,
  name: "Christmas Announcement",
  file_name: "christmas-announcement.mp4",
  file_url: "https://example.com/designs/christmas-announcement.mp4",
  download: "https://example.com/download/christmas-announcement.mp4",
  link: "https://remix.com/designs/christmas-announcement",
  description: "Festive Christmas announcement design",
  desc_md: "# Christmas Announcement\n\nFestive design...",
  categories: ["motion", "announcement"],
  colors: ["red", "green", "gold"],
  keywords: ["christmas", "holiday", "festive"]
}

const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/upload-to-sb-storage', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    fileUrl: 'https://example.com/designs/christmas-announcement.mp4',
    bucketName: 'remix-library',
    data: remixData
  })
})

const result = await response.json()
console.log('Uploaded to:', result.supabaseUrl)
console.log('Database entry:', result.databaseEntry)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "originalUrl": "https://example.com/designs/christmas-announcement.mp4",
  "supabaseUrl": "https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/public/remix-library/christmas-announcement.mp4",
  "fileName": "christmas-announcement.mp4",
  "path": "christmas-announcement.mp4",
  "databaseEntry": [
    {
      "id": 123,
      "name": "Christmas Announcement",
      "file_name": "christmas-announcement.mp4",
      "public_url": "https://wttgwoxlezqoyzmesekt.supabase.co/storage/v1/object/public/remix-library/christmas-announcement.mp4",
      ...
    }
  ]
}
```

### Error Responses

**400 Bad Request** - Missing required fields
```json
{
  "error": "fileUrl, bucketName, and data are required"
}
```

**500 Internal Server Error** - Download failed
```json
{
  "error": "Failed to fetch file from URL: Not Found"
}
```

**500 Internal Server Error** - Storage error
```json
{
  "error": "The resource already exists"
}
```

**500 Internal Server Error** - Database error
```json
{
  "error": "Database error: duplicate key value violates unique constraint"
}
```

## Filename Detection Logic

### Priority Order
1. **Provided fileName** - If explicitly set
2. **data.file_name** - From remix data object
3. **URL pathname** - Extract from URL
4. **Generated** - `file_{timestamp}.{extension}`

### Extension Detection
1. Extract from URL path
2. Validate against allowed extensions
3. Fallback to Content-Type header
4. Default to no extension if unknown

### Allowed Extensions
```
jpg, jpeg, png, gif, webp, svg, pdf, mp4, mp3, zip
```

### Filename Sanitization
- Removes special characters: `/[^a-zA-Z0-9_\-\.]/g`
- Replaces with underscores
- Preserves alphanumeric, hyphens, dots

## Content Type Detection

### Priority Order
1. **Provided contentType** - If explicitly set
2. **Blob.type** - From downloaded file
3. **Default** - `application/octet-stream`

## Database Upsert

### Table: remix_library
```sql
CREATE TABLE remix_library (
  id SERIAL PRIMARY KEY,
  name TEXT,
  file_name TEXT,
  file_url TEXT UNIQUE,
  download_url TEXT,
  page_url TEXT,
  description TEXT,
  desc_md TEXT,
  categories TEXT[],
  colors TEXT[],
  keywords TEXT[],
  wcid INTEGER,
  public_url TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Conflict Resolution
Uses `onConflict: 'file_url'` to update existing records:
- If `file_url` exists, updates all fields
- If new, creates new record
- Prevents duplicates based on original URL

## Storage Configuration

### Bucket Requirements
- Bucket must exist in Supabase Storage
- Service role key required for uploads
- Public bucket recommended for remix library

### Upload Options
```typescript
{
  contentType: fileContentType,
  upsert: true  // Overwrites if exists
}
```

## Use Cases

1. **Remix Library Sync:** Import designs from external sources
2. **Asset Migration:** Move files to Supabase Storage
3. **Content Import:** Bulk import design assets
4. **Backup:** Create copies of external files
5. **CDN Migration:** Move to Supabase CDN

## Workflow Process

1. **Validate Input** - Check required fields
2. **Download File** - Fetch from external URL
3. **Detect Filename** - Extract or generate name
4. **Detect Content Type** - Determine MIME type
5. **Upload to Storage** - Store in Supabase bucket
6. **Get Public URL** - Generate public access URL
7. **Upsert Database** - Create/update remix_library record
8. **Return Results** - Provide URLs and database entry

## Error Handling

### Download Failures
- 404 Not Found
- Network timeouts
- Invalid URLs
- Access denied

### Storage Failures
- Bucket doesn't exist
- Insufficient permissions
- File size limits
- Storage quota exceeded

### Database Failures
- Constraint violations
- Connection errors
- Invalid data types

## Performance Considerations

- Downloads entire file to Edge Function memory
- Limited by Edge Function timeout (60 seconds)
- Large files may fail (use tus for >50MB)
- Consider streaming for very large files

## Security

⚠️ **Public URLs**
- Files uploaded to public bucket
- Accessible via public URL
- No authentication required for access

✅ **Upload Protection**
- Requires JWT authentication
- Service role key for storage access
- Validates file URLs before download

## Notes

- Function downloads file to memory (not streaming)
- Large files may hit Edge Function memory limits
- Filename sanitization prevents path traversal
- Extension validation prevents malicious uploads
- Upsert prevents duplicate entries
- Public URL generated automatically
- Original URL preserved in database
- WCID links to external system ID
- Categories, colors, keywords stored as arrays
- Markdown description stored separately from plain text

