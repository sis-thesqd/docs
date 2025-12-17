---
title: "list_database_functions"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["database", "functions", "metadata", "list"]
version: "94"
---

# list_database_functions

Lists all database functions (RPC functions) in the Supabase database.

## Overview
**Function Slug:** list_database_functions  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 94

## Purpose
Retrieves a list of all database functions (stored procedures/RPC functions) available in the Supabase database.

## Endpoint

```
GET/POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/list_database_functions
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X GET \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/list_database_functions \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Response

### Success Response (200)
```json
[
  {
    "function_name": "get_designer_tasks_by_status_v3",
    "schema": "public",
    "return_type": "TABLE"
  },
  {
    "function_name": "execute_query",
    "schema": "public",
    "return_type": "jsonb"
  }
]
```

### Error Responses

**500 Internal Server Error**
```json
{
  "error": "..."
}
```

## Notes

- Calls `list_database_functions` RPC function
- Returns array of function metadata
- Includes function name, schema, return type
- Part of database utility function suite
- Useful for discovering available RPC functions



