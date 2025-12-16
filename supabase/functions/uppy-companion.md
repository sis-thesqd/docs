# uppy-companion

Companion server for Uppy file uploader supporting URL imports and S3 multipart uploads.

## Overview
**Function Slug:** uppy-companion  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 123

## Purpose
Backend companion for Uppy.js file uploader. Handles URL-to-S3 uploads, generates presigned URLs for direct uploads, and provides infrastructure for multipart uploads.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/uppy-companion
```

## Request

### Supported Paths

| Path | Method | Purpose |
|------|--------|---------|
| `/url` | POST | Upload file from URL to S3 |
| `/s3/multipart` | POST | Initialize S3 multipart upload |
| `/dropbox/*` | Any | Placeholder (not implemented) |

### Upload from URL

#### Request Body
```json
{
  "url": "https://example.com/file.pdf",
  "meta": {
    "destinationPath": "uploads"
  }
}
```

#### Sample Request
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/uppy-companion/url \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/large-video.mp4",
    "meta": {
      "destinationPath": "videos"
    }
  }'
```

### S3 Multipart Upload

#### Request Body
```json
{
  "filename": "large-file.zip",
  "contentType": "application/zip",
  "metadata": {
    "destinationPath": "uploads"
  }
}
```

#### Sample Request
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/uppy-companion/s3/multipart \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "large-file.zip",
    "contentType": "application/zip",
    "metadata": {
      "destinationPath": "documents"
    }
  }'
```

### JavaScript Example with Uppy

```javascript
import Uppy from '@uppy/core'
import AwsS3 from '@uppy/aws-s3'

const uppy = new Uppy()
  .use(AwsS3, {
    companionUrl: 'https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/uppy-companion',
    companionHeaders: {
      // No auth headers needed
    }
  })

// Upload from URL
uppy.addFile({
  name: 'remote-file.pdf',
  type: 'application/pdf',
  data: null,
  remote: {
    url: 'https://example.com/file.pdf',
    meta: {
      destinationPath: 'imports'
    }
  }
})
```

## Response

### URL Upload Success (200)

```json
{
  "successful": true,
  "uploadURL": "https://my-bucket.s3.us-east-1.amazonaws.com/uploads/file.pdf"
}
```

### S3 Multipart Success (200)

```json
{
  "url": "https://my-bucket.s3.us-east-1.amazonaws.com/uploads/large-file.zip?X-Amz-Algorithm=...",
  "method": "PUT"
}
```

### Error Responses

**405 Method Not Allowed**
```json
"Method not allowed"
```

**501 Not Implemented** - Dropbox
```json
{
  "error": "Dropbox integration requires additional setup with the Dropbox API"
}
```

**404 Not Found** - Invalid path
```json
{
  "error": "Not implemented",
  "path": "/invalid/path"
}
```

**500 Internal Server Error**
```json
{
  "error": "Failed to upload to S3: Access Denied"
}
```

## Features

### URL Import
1. Downloads file from provided URL
2. Uploads to S3 using presigned URL
3. Returns public S3 URL
4. Supports any file type

### S3 Multipart
1. Generates presigned PUT URL
2. 1-hour expiration
3. Supports large files
4. Returns URL and method for client

### CORS Support
Full CORS headers for cross-origin requests:
- `Access-Control-Allow-Origin: *`
- `Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS`
- `Access-Control-Allow-Headers: Content-Type, Authorization, X-Requested-With`

## S3 Configuration

### AWS SDK Setup
```typescript
const s3Client = new S3Client({
  region: Deno.env.get("S3_REGION") || "us-east-1",
  credentials: {
    accessKeyId: Deno.env.get("S3_ACCESS_KEY"),
    secretAccessKey: Deno.env.get("S3_SECRET_KEY")
  }
})
```

### Presigned URL Generation
- **Expiration:** 3600 seconds (1 hour)
- **Method:** PUT
- **Content-Type:** Set per file
- **ACL:** Not set (relies on bucket policy)

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `S3_BUCKET` | Yes | S3 bucket name |
| `S3_REGION` | Yes | AWS region (e.g., us-east-1) |
| `S3_ACCESS_KEY` | Yes | AWS access key ID |
| `S3_SECRET_KEY` | Yes | AWS secret access key |

## File Path Structure

```
{S3_BUCKET}/
  {destinationPath}/
    {filename}
```

Example:
```
my-bucket/
  uploads/
    file.pdf
  videos/
    video.mp4
```

## Use Cases

1. **Remote File Import:** Upload files from external URLs
2. **Large File Uploads:** Direct browser-to-S3 multipart uploads
3. **Uppy Integration:** Backend for Uppy.js file uploader
4. **Drag-and-Drop:** Support for drag-drop file uploads
5. **Progress Tracking:** Client-side progress with presigned URLs

## Limitations

### Not Implemented
- **Dropbox Integration:** Returns 501 error
- **Multipart Completion:** Simplified implementation
- **Upload Tracking:** No progress callbacks
- **Authentication:** No access control

### Known Issues
- No ACL setting (bucket must allow public read if needed)
- Simplified multipart (no part management)
- No upload validation
- No file size limits

## Security Considerations

⚠️ **Public Endpoint**
- No authentication required
- Anyone can upload if they know the URL
- Consider adding JWT auth for production
- Rate limiting recommended

⚠️ **AWS Credentials**
- Credentials exposed server-side only
- Presigned URLs expire after 1 hour
- URLs grant temporary upload access
- Bucket policies should restrict access

## Testing

### Test URL Upload
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/uppy-companion/url \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://httpbin.org/image/png",
    "meta": {
      "destinationPath": "test"
    }
  }'
```

### Test Multipart Init
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/uppy-companion/s3/multipart \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "test.txt",
    "contentType": "text/plain",
    "metadata": {
      "destinationPath": "test"
    }
  }'
```

## Integration with Uppy

### Basic Setup
```javascript
import Uppy from '@uppy/core'
import Dashboard from '@uppy/dashboard'
import AwsS3 from '@uppy/aws-s3'

const uppy = new Uppy({
  restrictions: {
    maxFileSize: 1000000000, // 1GB
    allowedFileTypes: ['image/*', 'video/*', 'application/pdf']
  }
})
  .use(Dashboard, {
    trigger: '#upload-button',
    inline: true
  })
  .use(AwsS3, {
    companionUrl: 'https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/uppy-companion'
  })

uppy.on('complete', (result) => {
  console.log('Upload complete:', result.successful)
})
```

## Notes

- Presigned URLs expire in 1 hour
- URL imports download entire file to Edge Function first
- Large files may timeout (Edge Function 60s limit)
- For files >50MB, use direct multipart upload
- Dropbox support is placeholder only
- Content-Type defaults to application/octet-stream
- Filename extracted from URL path for URL imports
- Bucket must exist and credentials must have write access
- No file virus scanning implemented
- Consider implementing upload webhooks for production

