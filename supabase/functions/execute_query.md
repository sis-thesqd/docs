---
title: "execute_query"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["sql", "query", "database", "execute"]
version: "94"
---

# execute_query

Executes SQL queries with automatic LIMIT 100 and JSON aggregation. Wrapper around `execute_database_script` RPC.

## Overview
**Function Slug:** execute_query  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 94

## Purpose
Executes SQL queries with automatic result limiting (100 rows) and JSON aggregation. Uses `execute_database_script` RPC internally.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/execute_query
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
| `query_text` | string | Yes | SQL query to execute |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/execute_query \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "query_text": "SELECT id, name, email FROM employees"
  }'
```

## Response

### Success Response (200)
```json
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  }
]
```

### Error Responses

**400 Bad Request**
```json
{
  "error": "Missing query_text parameter"
}
```

## Query Processing

### Automatic LIMIT
Queries are automatically wrapped with LIMIT 100:
```sql
SELECT jsonb_agg(row_to_json(t)) 
FROM (YOUR_QUERY LIMIT 100) t
```

### Result Format
- Returns array of JSON objects
- Each row becomes a JSON object
- Limited to 100 rows maximum

## Notes

- Automatically limits results to 100 rows
- Uses `execute_database_script` RPC internally
- Results aggregated as JSON array
- Part of database utility function suite
- Similar to `execute-sql` but with automatic limiting



