---
title: "get-project-defaults"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["projects", "defaults", "configuration", "prf"]
version: "13"
---

# get-project-defaults

Retrieves saved default selections for a specific account and project type combination.

## Overview
**Function Slug:** get-project-defaults  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 13

## Purpose
Fetches saved project default selections from `prf_project_defaults` table, allowing users to quickly populate project forms with their preferred settings.

## Endpoint

```
GET https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get-project-defaults?account_id={account_id}&project_type={project_type}
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
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get-project-defaults?account_id=rec123abc&project_type=5" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### JavaScript Example

```javascript
const accountId = 'rec123abc'
const projectType = 5

const response = await fetch(`https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get-project-defaults?account_id=${accountId}&project_type=${projectType}`, {
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const defaults = await response.json()
// Returns the selections object directly
console.log(defaults)
```

## Response

### Success Response (200) - With Defaults

```json
{
  "designer": "John Doe",
  "style": "modern",
  "colorScheme": ["#FF5733", "#C70039"],
  "deliveryMethod": "email",
  "customField1": "value1"
}
```

### Success Response (200) - No Defaults Found

```json
{}
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

### selections Column Structure

The `selections` JSONB column can contain any project-specific fields:

```json
{
  "designer": "string",
  "motionDesigner": "string",
  "style": "string",
  "fonts": ["array"],
  "colors": ["array"],
  "deliveryFormat": "string",
  "customFields": {...}
}
```

## Use Cases

1. **Form Prefill:** Load user's saved defaults when starting new project
2. **Quick Start:** Allow repeat projects with same settings
3. **Templates:** Save commonly used project configurations
4. **User Preferences:** Remember user's typical choices
5. **Time Savings:** Reduce form filling for frequent users

## Related Endpoints

To save defaults, you would typically use Supabase client:

```javascript
// Save defaults
const { error } = await supabase
  .from('prf_project_defaults')
  .upsert({
    account_id: 'rec123abc',
    project_type: 5,
    selections: {
      designer: 'John Doe',
      style: 'modern',
      ...
    }
  }, {
    onConflict: 'account_id,project_type'
  })
```

## Notes

- Returns empty object `{}` if no defaults exist (not 404)
- The `selections` field is returned directly, not wrapped in an object
- Uses unique constraint on `(account_id, project_type)` for upserts
- JSONB column allows flexible schema per project type
- Function uses service role for database access

