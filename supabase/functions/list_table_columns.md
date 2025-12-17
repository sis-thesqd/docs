---
title: "list_table_columns"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["database", "columns", "metadata", "schema"]
version: "94"
---

# list_table_columns

Lists all table columns and their metadata from the Supabase database.

## Overview
**Function Slug:** list_table_columns  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 94

## Purpose
Retrieves metadata about all table columns in the database including column names, data types, and constraints.

## Endpoint

```
GET/POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/list_table_columns
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X GET \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/list_table_columns \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Response

### Success Response (200)
```json
[
  {
    "table_name": "employees",
    "column_name": "id",
    "data_type": "uuid",
    "is_nullable": "NO"
  },
  {
    "table_name": "employees",
    "column_name": "name",
    "data_type": "text",
    "is_nullable": "YES"
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

- Calls `list_table_columns` RPC function
- Returns array of column metadata
- Includes table name, column name, data type
- Part of database utility function suite
- Useful for schema discovery and documentation



