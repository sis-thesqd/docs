---
title: "execute_database_script"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["sql", "database", "script", "execute"]
version: "94"
---

# execute_database_script

Executes SQL scripts via RPC function `execute_query`. Wrapper around database script execution.

## Overview
**Function Slug:** execute_database_script  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 94

## Purpose
Executes SQL scripts by calling the `execute_query` RPC function. Provides a simple interface for running database scripts.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/execute_database_script
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
| `script_text` | string | Yes | SQL script to execute |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/execute_database_script \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "script_text": "SELECT COUNT(*) FROM employees WHERE status = '\''Active'\'';"
  }'
```

## Response

### Success Response (200)
```json
{}
```

### Error Responses

**400 Bad Request**
```json
{
  "error": "Missing script_text parameter"
}
```

**500 Internal Server Error**
```json
{
  "error": "..."
}
```

## Notes

- Calls `execute_query` RPC function internally
- Uses service role key for execution
- Returns empty object on success
- Similar to `execute-sql` but uses different RPC
- Part of database utility function suite

