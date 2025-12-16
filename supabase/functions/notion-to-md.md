# notion-to-md

Converts Notion pages to Markdown format with support for rich text formatting and large pages.

## Overview
**Function Slug:** notion-to-md  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 106

## Purpose
Fetches Notion page content and converts it to clean Markdown format. Handles rich text annotations, custom formatting, and large pages with automatic chunking and timeout protection.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/notion-to-md
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
| `pageId` | string | Yes | Notion page ID (32 characters, with or without hyphens) |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/notion-to-md \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "pageId": "a1b2c3d4e5f67890a1b2c3d4e5f67890"
  }'
```

### JavaScript Example

```javascript
const pageId = 'a1b2c3d4-e5f6-7890-a1b2-c3d4e5f67890' // With or without hyphens

const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/notion-to-md', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ pageId })
})

const data = await response.json()
console.log(data.markdown)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "markdown": "# Page Title\n\nThis is the **converted** markdown content.\n\n## Section 1\n\nWith *proper* formatting and `code` blocks.\n\n- List item 1\n- List item 2\n\n### Subsection\n\nMore content here...",
  "debug": {
    "resultType": "string",
    "resultKeys": ["parent"],
    "blocksCount": 47
  }
}
```

### Error Responses

**400 Bad Request** - Invalid JSON
```json
{
  "success": false,
  "error": "Invalid JSON in request body"
}
```

**400 Bad Request** - Missing page ID
```json
{
  "success": false,
  "error": "pageId is required"
}
```

**400 Bad Request** - No token configured
```json
{
  "success": false,
  "error": "NOTION_API_TOKEN is not set"
}
```

**413 Payload Too Large** - Page too large
```json
{
  "success": false,
  "error": "Processing timed out - page too large"
}
```

**500 Internal Server Error** - Conversion failed
```json
{
  "success": false,
  "error": "Error converting to markdown: {...}",
  "stack": "..."
}
```

## Features

### Rich Text Support
- **Bold:** `**text**`
- **Italic:** `_text_`
- **Code:** `` `text` ``
- **Strikethrough:** `~~text~~`
- **Underline:** (no markdown equivalent, stripped)

### Content Types
- Headings (H1, H2, H3)
- Paragraphs
- Lists (bulleted, numbered)
- Code blocks
- Quotes
- Dividers

### Performance Optimizations
- **Chunking:** Processes 100 blocks at a time
- **Pagination:** Handles pages with >100 blocks
- **Timeout:** 55-second limit (just under Edge Function max)
- **Block Limit:** Stops at 1000 blocks to prevent overload

### HTML Cleanup
- Removes all HTML tags from final output
- Clean markdown without embedded HTML
- Preserves markdown formatting

## Notion API Integration

### Required Token
Integration must have access to the page:
1. Create integration at notion.so/my-integrations
2. Share page with integration
3. Copy internal integration token
4. Set as `NOTION_API_TOKEN` environment variable

### API Calls
```typescript
// Get page blocks
await notion.blocks.children.list({
  block_id: pageId,
  page_size: 100,
  start_cursor: cursor
})
```

### Rate Limits
Notion API rate limit: 3 requests per second

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `NOTION_API_TOKEN` | Yes | Notion integration token (starts with `secret_`) |

## Use Cases

1. **Documentation Export:** Convert Notion docs to Markdown
2. **Static Site Generation:** Export content for JAMstack sites
3. **Content Migration:** Move from Notion to Markdown-based systems
4. **Backup:** Archive Notion pages as Markdown files
5. **Publishing:** Export for blog platforms

## Limitations

- **Child Pages:** Not parsed (set `parseChildPages: false`)
- **Databases:** Not supported
- **Embeds:** May not convert properly
- **Tables:** Limited support
- **File Attachments:** URLs only, files not downloaded
- **Page Size:** Max ~1000 blocks
- **Timeout:** 55 seconds maximum processing time

## Testing

### Test Conversion
```bash
# Get Notion page ID from page URL
# https://notion.so/Page-Title-a1b2c3d4e5f67890a1b2c3d4e5f67890
# Page ID: a1b2c3d4e5f67890a1b2c3d4e5f67890

curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/notion-to-md \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"pageId":"a1b2c3d4e5f67890a1b2c3d4e5f67890"}'
```

## Notes

- Page ID can include or exclude hyphens
- Integration must be explicitly shared with the page
- Large pages may hit timeout (split into smaller pages)
- HTML tags are stripped from final output
- Debug info helps troubleshoot conversion issues
- Function handles pagination automatically
- Custom transformers clean HTML from headings
- Maximum processing time prevents function timeouts

