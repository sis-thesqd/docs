---
title: "tus"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["tus", "upload", "resumable", "s3", "protocol"]
version: "122"
---

# tus

TUS protocol implementation for resumable file uploads to S3 with chunked transfer support.

## Overview
**Function Slug:** tus  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 122

## Purpose
Implements the TUS (Resumable Upload Protocol) v1.0.0 for reliable large file uploads. Supports chunked uploads, resumable transfers, and upload progress tracking. Files are stored in S3 with metadata tracking.

## Endpoint

```
POST/HEAD/PATCH https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/tus
```

## TUS Protocol Methods

### POST - Create Upload

Creates a new upload session and returns a location URL.

#### Headers
```
Upload-Length: {file_size_in_bytes}
Upload-Metadata: {base64_encoded_metadata}
```

#### Sample Request
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/tus \
  -H "Upload-Length: 10485760" \
  -H "Upload-Metadata: filename dGVzdC5wZGY=,content-type YXBwbGljYXRpb24vcGRm"
```

#### Response (201 Created)
```
Location: /files/{file_id}
Tus-Resumable: 1.0.0
```

### HEAD - Get Upload Status

Retrieves current upload offset and metadata.

#### Sample Request
```bash
curl -X HEAD \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/tus/files/{file_id} \
  -H "Tus-Resumable: 1.0.0"
```

#### Response (200 OK)
```
Upload-Offset: 5242880
Upload-Length: 10485760
Tus-Resumable: 1.0.0
Upload-Metadata: filename dGVzdC5wZGY=
```

### PATCH - Upload Chunk

Uploads a chunk of data at a specific offset.

#### Headers
```
Upload-Offset: {current_offset}
Content-Type: application/offset+octet-stream
Content-Length: {chunk_size}
```

#### Sample Request
```bash
curl -X PATCH \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/tus/files/{file_id} \
  -H "Upload-Offset: 0" \
  -H "Content-Type: application/offset+octet-stream" \
  -H "Content-Length: 5242880" \
  --data-binary @chunk1.bin
```

#### Response (204 No Content)
```
Upload-Offset: 5242880
Tus-Resumable: 1.0.0
```

### OPTIONS - Protocol Discovery

Returns supported TUS protocol features.

#### Sample Request
```bash
curl -X OPTIONS \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/tus
```

#### Response (200 OK)
```
Tus-Version: 1.0.0
Tus-Resumable: 1.0.0
Tus-Extension: creation,creation-with-upload,termination,checksum,expiration
Tus-Max-Size: 1073741824
```

## JavaScript Example

```javascript
// Using @tus/client library
import * as tus from '@tus/client'

const file = document.querySelector('input[type="file"]').files[0]

const upload = new tus.Upload(file, {
  endpoint: 'https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/tus',
  retryDelays: [0, 3000, 5000, 10000, 20000],
  metadata: {
    filename: file.name,
    filetype: file.type
  },
  onError: (error) => {
    console.error('Upload failed:', error)
  },
  onProgress: (bytesUploaded, bytesTotal) => {
    const percentage = ((bytesUploaded / bytesTotal) * 100).toFixed(2)
    console.log(`Upload progress: ${percentage}%`)
  },
  onSuccess: () => {
    console.log('Upload finished:', upload.url)
  }
})

// Start upload
upload.start()

// Resume upload (if interrupted)
upload.resume()
```

## TUS Protocol Features

### Supported Extensions
- **creation** - Create uploads
- **creation-with-upload** - Create and upload in one request
- **termination** - Delete incomplete uploads
- **checksum** - Verify upload integrity
- **expiration** - Automatic cleanup

### Limits
- **Max File Size:** 1GB (1,073,741,824 bytes)
- **Protocol Version:** 1.0.0
- **Chunk Size:** Client-determined

## Upload Metadata

### Encoding
Metadata is base64-encoded key-value pairs:
```
filename dGVzdC5wZGY=,content-type YXBwbGljYXRpb24vcGRm
```

### Decoding
```javascript
// Decode metadata
const metadata = "filename dGVzdC5wZGY="
const [key, value] = metadata.split(' ')
const decoded = atob(value) // "test.pdf"
```

## S3 Storage Structure

### File Organization
```
s3://{bucket}/
  uploads/
    {file_id}/
      chunks/
        0          (first chunk)
        5242880    (second chunk)
        10485760   (third chunk)
      {filename}   (final file, if assembled)
    {file_id}.metadata  (upload metadata)
```

### Metadata Format
```json
{
  "uploadLength": 10485760,
  "metadata": {
    "filename": "test.pdf",
    "content-type": "application/pdf"
  },
  "offset": 5242880,
  "createdAt": "2025-12-16T10:30:00.000Z"
}
```

## Resumable Upload Flow

1. **Create Upload** (POST)
   - Client sends file size and metadata
   - Server creates upload session
   - Returns file ID and location

2. **Upload Chunks** (PATCH)
   - Client uploads chunks sequentially
   - Server tracks offset
   - Can resume from any offset

3. **Check Progress** (HEAD)
   - Client queries current offset
   - Determines if resume needed
   - Verifies upload completion

4. **Complete Upload**
   - All chunks uploaded
   - Server assembles final file
   - File available at S3 URL

## Error Responses

**400 Bad Request** - Missing metadata
```
Missing upload metadata
```

**413 Payload Too Large** - File too large
```
Invalid upload length
```

**404 Not Found** - Invalid file ID
```
File not found
```

**500 Internal Server Error** - S3 error
```
S3 bucket not configured
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `S3_BUCKET` | Yes | S3 bucket name |
| `S3_REGION` | Yes | AWS region |
| `S3_ACCESS_KEY` | Yes | AWS access key |
| `S3_SECRET_KEY` | Yes | AWS secret key |

## Use Cases

1. **Large File Uploads:** Handle files >100MB reliably
2. **Unstable Connections:** Resume interrupted uploads
3. **Progress Tracking:** Show real-time upload progress
4. **Mobile Uploads:** Handle network interruptions
5. **Video Uploads:** Large media file transfers

## Advantages Over Standard Upload

✅ **Resumable:** Continue from last offset  
✅ **Reliable:** Handles network failures  
✅ **Progress:** Real-time offset tracking  
✅ **Efficient:** No re-upload on failure  
✅ **Standard:** TUS protocol compliance  

## CORS Configuration

Full CORS support for browser uploads:
- `Access-Control-Allow-Origin: *`
- `Access-Control-Allow-Methods: GET, POST, HEAD, PATCH, DELETE, OPTIONS`
- `Access-Control-Allow-Headers: *`
- `Access-Control-Expose-Headers: Upload-Offset, Location, Upload-Length, Tus-Version`

## Notes

- File ID is UUID generated server-side
- Chunks stored separately until upload complete
- Metadata file tracks upload progress
- No automatic file assembly (chunks remain separate)
- Consider implementing final file assembly step
- TUS protocol is industry standard for resumable uploads
- Compatible with tus-js-client and other TUS libraries
- Max size enforced at creation (not per chunk)
- Offset must match exactly (no gaps allowed)



