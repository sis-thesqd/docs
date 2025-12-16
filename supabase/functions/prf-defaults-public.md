---
title: "prf-defaults-public"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["prf", "defaults", "projects", "public", "configuration"]
version: "10"
---

# prf-defaults-public

Public endpoint to retrieve PRF project default settings in array format.

## Overview
**Function Slug:** prf-defaults-public  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 10

## Purpose
Similar to `get-project-defaults` but returns selections in array format instead of object format. Designed for public-facing applications that need project default configurations.

## Endpoint

```
GET https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/prf-defaults-public?account_id={account_id}&project_type={project_type}
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account/church identifier |
| `project_type` | number | Yes | Project type ID |

### Sample Request

```bash
curl -X GET \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/prf-defaults-public?account_id=rec123abc&project_type=5" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### JavaScript Example

```javascript
const accountId = 'rec123abc'
const projectType = 5

const response = await fetch(`https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/prf-defaults-public?account_id=${accountId}&project_type=${projectType}`, {
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const data = await response.json()
// Returns array format: [{ field: 'designer', value: 'John Doe' }, ...]
console.log(data.selections)
```

## Response

### Success Response (200) - With Defaults

```json
{
  "selections": [
    {
      "field": "designer",
      "value": "John Doe"
    },
    {
      "field": "style",
      "value": "modern"
    },
    {
      "field": "colorScheme",
      "value": ["#FF5733", "#C70039"]
    },
    {
      "field": "deliveryMethod",
      "value": "email"
    }
  ]
}
```

### Success Response (200) - No Defaults Found

```json
{
  "selections": []
}
```

### Error Responses

**400 Bad Request** - Missing parameters
```json
{
  "error": "Missing required parameters: account_id and project_type"
}
```

**500 Internal Server Error**
```json
{
  "error": "Internal server error",
  "details": "..."
}
```

## Format Comparison

### Object Format (get-project-defaults)
```json
{
  "designer": "John Doe",
  "style": "modern",
  "colorScheme": ["#FF5733", "#C70039"]
}
```

### Array Format (prf-defaults-public)
```json
{
  "selections": [
    { "field": "designer", "value": "John Doe" },
    { "field": "style", "value": "modern" },
    { "field": "colorScheme", "value": ["#FF5733", "#C70039"] }
  ]
}
```

## Use Cases

1. **Form Prefill:** Load defaults into form fields
2. **Public Forms:** Access defaults without object manipulation
3. **Iteration:** Loop through selections easily
4. **Display:** Show defaults in lists/tables
5. **Compatibility:** Work with systems expecting array format

## Database Table

### Table: prf_project_defaults
```sql
CREATE TABLE prf_project_defaults (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  account_id TEXT NOT NULL,
  project_type INTEGER NOT NULL,
  selections JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(account_id, project_type)
);
```

### Conversion Logic
```typescript
// Object to Array conversion
const selectionsList = Object.entries(data.selections).map(([key, value]) => ({
  field: key,
  value: value
}))
```

## Advantages of Array Format

✅ **Iteration:** Easy to loop through selections  
✅ **Form Binding:** Direct mapping to form fields  
✅ **Display:** Simple to render in lists  
✅ **Validation:** Easier to validate each field  
✅ **Transformation:** Simple to transform for different formats  

## Related Functions

- [get-project-defaults](./get-project-defaults.md) - Returns object format
- Both query same table with different output formats

## Notes

- Returns empty array `[]` if no defaults exist (not null)
- Array format easier for form libraries
- Object format better for direct property access
- Choose format based on use case
- Both functions use same database table
- Same authentication requirements
- Same query parameters
- Conversion happens server-side
- No performance difference between formats
- Consider caching for frequently accessed defaults

