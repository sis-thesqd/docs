---
title: "squad-file-uploader"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["upload", "s3", "dropbox", "files", "multipart"]
version: "117"
---

# squad-file-uploader

Uploads files to either S3 or Dropbox with support for large files and chunked uploads.

## Overview
**Function Slug:** squad-file-uploader  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 117

## Purpose
Unified file upload solution supporting both S3 and Dropbox destinations. Handles files from URLs or base64 data, with automatic chunking for large files (>20MB) and Dropbox rate limiting.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/squad-file-uploader
```

## Request

### Headers
```
Content-Type: application/json
```

### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `destination` | string | Yes | Either "s3" or "dropbox" |
| `uploadPath` | string | Yes | Destination path (S3: "bucket/path", Dropbox: "/path") |
| `fileURL` | string | Conditional | URL to fetch file from (if not using `file`) |
| `file` | string | Conditional | Base64 encoded file data (if not using `fileURL`) |
| `filename` | string | Conditional | Required when using `file` parameter |
| `awsRegion` | string | Conditional | Required when destination is "s3" (e.g., "us-east-1") |

### Sample Requests

#### Upload from URL to Dropbox
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/squad-file-uploader \
  -H "Content-Type: application/json" \
  -d '{
    "destination": "dropbox",
    "uploadPath": "/Projects/ClientName",
    "fileURL": "https://example.com/file.pdf"
  }'
```

#### Upload Base64 to S3
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/squad-file-uploader \
  -H "Content-Type: application/json" \
  -d '{
    "destination": "s3",
    "uploadPath": "my-bucket/uploads",
    "file": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...",
    "filename": "image.png",
    "awsRegion": "us-east-1"
  }'
```

### JavaScript Example

```javascript
// Upload file from URL
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/squad-file-uploader', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    destination: 'dropbox',
    uploadPath: '/Projects/Assets',
    fileURL: 'https://example.com/video.mp4'
  })
})

const data = await response.json()
console.log('File URL:', data.url)
console.log('Dropbox ID:', data.fileId)
```

## Response

### Success Response (200)

**Dropbox:**
```json
{
  "url": "https://www.dropbox.com/s/abc123/video.mp4?dl=1",
  "path": "/Projects/Assets/video.mp4",
  "fileId": "id:a4ayc_80_OEAAAAAAAAAXw"
}
```

**S3:**
```json
{
  "url": "https://my-bucket.s3.us-east-1.amazonaws.com/uploads/file.pdf",
  "path": "uploads/file.pdf",
  "fileId": "abc123def456789"
}
```

### Error Responses

**400 Bad Request**
```json
{
  "error": "Missing required parameters"
}
```

**400 Bad Request**
```json
{
  "error": "Filename is required when uploading base64 data"
}
```

**400 Bad Request**
```json
{
  "error": "Destination must be either \"s3\" or \"dropbox\""
}
```

**400 Bad Request**
```json
{
  "error": "AWS region is required when uploading to S3"
}
```

**500 Internal Server Error**
```json
{
  "error": "Failed to upload file to Dropbox: Rate limited"
}
```

## Features

### Large File Support
- **Chunk Size:** 20MB per chunk
- **Simple Upload:** Files ≤20MB
- **Session Upload:** Files >20MB (chunked)
- **Progress:** Automatic chunking and retries

### Dropbox Rate Limiting
- Automatic retry on 429 errors
- 5-second wait between retries
- Maximum 3 retry attempts
- Applies to all Dropbox API calls

### Dropbox Team Support
- Uses hardcoded team member ID
- Hardcoded path root for team account
- Specific to SQD Dropbox Business account

## Dropbox Configuration

### Headers
```
Dropbox-API-Select-User: dbmid:AABo4fq2X3uLimFPIIKWuPS3arV4l31_8fk
Dropbox-API-Path-Root: {".tag": "root", "root": "2564080211"}
```

### Share Link Settings
```json
{
  "requested_visibility": "public"
}
```

## S3 Configuration

### Upload Method
Uses Deno S3 library with automatic chunking

### URL Format
```
https://{bucket}.s3.{region}.amazonaws.com/{key}
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DROPBOX_APP_KEY` | Conditional | Dropbox app key (for Dropbox uploads) |
| `DROPBOX_APP_SECRET` | Conditional | Dropbox app secret |
| `DROPBOX_REFRESH_TOKEN` | Conditional | Dropbox refresh token |
| `AWS_ACCESS_KEY_ID` | Conditional | AWS access key (for S3 uploads) |
| `AWS_SECRET_ACCESS_KEY` | Conditional | AWS secret key |
| `PROJECT_URL` | Optional | Supabase URL for token updates |
| `SERVICE_ROLE_KEY` | Optional | Supabase service key for token updates |

## Use Cases

1. **Client Deliverables:** Upload final files to Dropbox
2. **Asset Storage:** Store media files in S3
3. **Backup:** Copy files from external URLs
4. **File Migration:** Transfer between storage systems
5. **Automated Workflows:** Upload files from n8n/Make

## Notes

- Dropbox team ID and path root are hardcoded
- Dropbox access token is auto-updated in Supabase vault
- S3 uploads don't set ACL (relies on bucket policy)
- Rate limiting only applies to Dropbox
- File type detection from URL or base64 prefix
- Dropbox share links use `?dl=1` for direct download
- Large files automatically use session-based upload

