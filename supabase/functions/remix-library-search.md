# remix-library-search

Searches the remix library with advanced filtering, pagination, and sorting capabilities.

## Overview
**Function Slug:** remix-library-search  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 1

## Purpose
Provides comprehensive search functionality for the remix design library with support for text search, keyword filtering, category/color filtering, and pagination. Returns structured results with metadata.

## Endpoint

```
GET/POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/remix-library-search
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
Content-Type: application/json
```

### Query Parameters (GET) or Body (POST)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | No | General text search (name, description, file_name) |
| `name` | string | No | Search by name specifically |
| `keywords` | array | No | Filter by keywords |
| `categories` | array | No | Filter by categories |
| `colors` | array | No | Filter by colors |
| `wcid` | number | No | Filter by wcid |
| `limit` | number | No | Results per page (default 50, max 100) |
| `offset` | number | No | Pagination offset (default 0) |
| `sort_by` | string | No | Sort field (created_at, name, file_name, wcid) |
| `sort_order` | string | No | Sort order (asc, desc) |

### Sample Requests

#### GET Request - Simple Search
```bash
curl -X GET \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/remix-library-search?q=christmas&limit=10" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

#### GET Request - Advanced Filtering
```bash
curl -X GET \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/remix-library-search?keywords=sermon,social&categories=motion&colors=red,blue&limit=20&sort_by=name&sort_order=asc" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

#### POST Request
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/remix-library-search \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "q": "christmas",
    "keywords": ["sermon", "social"],
    "limit": 20,
    "offset": 0
  }'
```

### JavaScript Example

```javascript
// Simple search
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/remix-library-search?q=christmas', {
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const data = await response.json()
console.log(`Found ${data.pagination.total} results`)
console.log(data.data)

// Advanced search with POST
const searchParams = {
  keywords: ['sermon', 'announcement'],
  categories: ['motion', 'graphic'],
  colors: ['blue', 'red'],
  limit: 25,
  sort_by: 'created_at',
  sort_order: 'desc'
}

const advancedResponse = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/remix-library-search', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(searchParams)
})

const results = await advancedResponse.json()
```

## Response

### Success Response (200)

```json
{
  "data": [
    {
      "id": 123,
      "name": "Christmas Announcement",
      "description": "Festive Christmas announcement design",
      "file_name": "christmas-announcement.mp4",
      "file_path": "/remix/motion/christmas-announcement.mp4",
      "keywords": ["christmas", "announcement", "festive"],
      "categories": ["motion", "announcement"],
      "colors": ["red", "green", "gold"],
      "wcid": 5678,
      "created_at": "2024-12-01T10:00:00Z",
      "desc_md": "# Christmas Announcement\n\nA beautiful festive design..."
    }
  ],
  "pagination": {
    "total": 147,
    "limit": 50,
    "offset": 0,
    "has_more": true
  }
}
```

### Error Responses

**405 Method Not Allowed**
```json
{
  "error": "Method not allowed"
}
```

**500 Internal Server Error**
```json
{
  "error": "Query error message"
}
```

## Search Features

### Text Search (q parameter)
Searches across multiple fields using case-insensitive pattern matching:
- `name`
- `description`
- `file_name`
- `desc_md`

### Array Filtering
Uses PostgreSQL array overlap operator for:
- **keywords**: Matches if any keyword is present
- **categories**: Matches if any category is present
- **colors**: Matches if any color is present

### Sorting
Valid sort fields:
- `created_at` (default)
- `name`
- `file_name`
- `wcid`

Sort order:
- `desc` (default for created_at)
- `asc`

### Pagination
- Default limit: 50 results
- Maximum limit: 100 results
- Use `offset` to paginate through results
- `has_more` flag indicates additional pages

## Database Table

### Table: remix_library
```sql
Columns:
- id (integer, primary key)
- name (text)
- description (text)
- file_name (text)
- file_path (text)
- keywords (text[])
- categories (text[])
- colors (text[])
- wcid (integer)
- created_at (timestamp)
- desc_md (text)
```

## Use Cases

1. **Design Discovery:** Find designs by keyword or theme
2. **Category Browsing:** Filter by design type
3. **Color Matching:** Find designs with specific colors
4. **Content Management:** Search and organize remix assets
5. **Client Selection:** Help clients find appropriate designs

## Performance Notes

- Text search uses `ilike` for case-insensitive matching
- Array filters use efficient `overlaps` operator
- Results are limited to prevent large data transfers
- Pagination recommended for large result sets
- Sorting adds minimal overhead

## Examples

### Find All Christmas Designs
```bash
curl -X GET \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/remix-library-search?q=christmas" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Find Motion Graphics with Red Color
```bash
curl -X GET \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/remix-library-search?categories=motion&colors=red" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Paginate Through Results
```bash
# Page 1
curl "...?limit=50&offset=0"

# Page 2
curl "...?limit=50&offset=50"

# Page 3
curl "...?limit=50&offset=100"
```

## Notes

- Both GET and POST methods supported
- GET is better for simple searches (URL parameters)
- POST is better for complex filters (JSON body)
- Array parameters in GET use comma separation: `keywords=a,b,c`
- Maximum 100 results per request
- Use `has_more` flag to determine if pagination needed
- Invalid sort fields fall back to `created_at desc`

