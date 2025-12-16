---
title: "n8n_db_workflows"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["n8n", "workflows", "database", "sync", "monitoring"]
version: "95"
---

# n8n_db_workflows

Syncs n8n workflows with database nodes to Supabase, tracking database queries, functions, and execution metrics.

## Overview
**Function Slug:** n8n_db_workflows  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 95

## Purpose
Scans n8n workflows for database nodes (PostgreSQL, Supabase), extracts query details, and syncs metadata including execution metrics to the `n8n_db_workflows` table. Only processes active workflows.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/n8n_db_workflows
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/n8n_db_workflows \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/n8n_db_workflows', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const data = await response.json()
console.log(`Processed ${data.details[0].totalWorkflows} workflows`)
console.log(`Found ${data.details[0].dbNodeWorkflows} workflows with DB nodes`)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "Processed 247 workflows and found 89 workflows with 156 database nodes across 1 n8n instances",
  "details": [
    {
      "instance": "sis1",
      "totalWorkflows": 247,
      "dbNodeWorkflows": 89,
      "totalDbNodes": 156,
      "success": true
    }
  ]
}
```

### Error Responses

**500 Internal Server Error**
```json
{
  "success": false,
  "error": "Failed to fetch workflows: 401 Unauthorized"
}
```

## Workflow Process

### 1. Fetch All Workflows
- Uses n8n API with pagination
- Fetches up to 250 workflows per page
- Continues until all workflows retrieved
- Only processes active workflows

### 2. Extract Database Nodes
- Scans workflow nodes for DB types
- Extracts queries, functions, operations
- Identifies tables and columns accessed
- Captures credential information

### 3. Get Execution Metrics
- Fetches last 24 hours of executions
- Calculates average/min/max execution times
- Counts executions in time window
- Gets most recent execution timestamp

### 4. Upsert to Database
- Updates `n8n_db_workflows` table
- Conflict resolution on composite key
- Stores all extracted metadata

## Database Node Types Detected

### Supported Node Types
- `n8n-nodes-base.postgres`
- `n8n-nodes-base.postgresql`
- `n8n-nodes-base.supabase`
- `n8n-nodes-base.pgquery`
- `n8n-nodes-base.pginsert`
- `n8n-nodes-base.pgupdate`
- `n8n-nodes-base.supabasequery`

## Extracted Information

### Query Extraction
- Direct SQL queries from `query`/`sql` parameters
- SQL snippets from code nodes
- Function calls from RPC patterns
- Table names from operations

### Column Extraction
- SELECT column lists
- INSERT column lists
- UPDATE column lists
- Handles aliases and table prefixes

### Function Name Patterns
```typescript
// RPC call pattern
.rpc("function_name")

// Function name pattern
function: "function_name"
```

## Database Table

### Table: n8n_db_workflows
```sql
CREATE TABLE n8n_db_workflows (
  instance_name TEXT NOT NULL,
  workflow_id TEXT NOT NULL,
  workflow_name TEXT,
  workflow_active BOOLEAN,
  node_id TEXT NOT NULL,
  node_name TEXT,
  node_type TEXT,
  query TEXT,
  function_name TEXT,
  operation TEXT,
  table_name TEXT,
  columns_accessed TEXT[],
  credentials_name TEXT,
  credentials_type TEXT,
  execution_count INTEGER,
  avg_execution_minutes DECIMAL,
  min_execution_minutes DECIMAL,
  max_execution_minutes DECIMAL,
  last_execution TIMESTAMP,
  updated_at TIMESTAMP,
  PRIMARY KEY (instance_name, workflow_id, node_id)
);
```

## Execution Metrics

### Time Window
- Last 24 hours from current time
- Filters executions by `startedAt`
- Calculates duration from `startedAt` to `stoppedAt`

### Metrics Calculated
- **Count:** Executions in last 24 hours
- **Average:** Mean execution time in minutes
- **Min:** Shortest execution time
- **Max:** Longest execution time
- **Last Execution:** Most recent timestamp

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_ANON_KEY` | Yes | Supabase anonymous key |
| `N8N_API_KEY_SIS1` | Yes | n8n API key for sis1 instance |
| `N8N_API_KEY_SIS2` | No | n8n API key for sis2 instance |
| `N8N_API_KEY_SISX` | No | n8n API key for sisx instance |

## n8n Instances Processed

Currently processes:
- **sis1** (required)

Future support:
- sis2
- sisx

## Pagination

### Workflow Pagination
- Uses cursor-based pagination
- Fetches 250 workflows per page
- Continues until `nextCursor` is null
- 100ms delay between pages

### Execution Pagination
- Fetches up to 250 executions
- Filters to last 24 hours client-side
- No pagination for executions

## Use Cases

1. **Database Audit:** Track all DB operations in workflows
2. **Performance Monitoring:** Identify slow queries
3. **Security Review:** Audit database access patterns
4. **Documentation:** Auto-generate DB usage docs
5. **Optimization:** Find inefficient queries

## Column Extraction Logic

### SELECT Queries
```sql
SELECT col1, col2, col3 FROM table
-- Extracts: ["col1", "col2", "col3"]
```

### INSERT Queries
```sql
INSERT INTO table (col1, col2) VALUES (...)
-- Extracts: ["col1", "col2"]
```

### UPDATE Queries
```sql
UPDATE table SET col1 = val1, col2 = val2 WHERE ...
-- Extracts: ["col1", "col2"]
```

### Handles Aliases
```sql
SELECT table.column AS alias FROM table
-- Extracts: ["column"]
```

## Notes

- Only processes active workflows
- Skips inactive/archived workflows
- Extracts SQL from code nodes if present
- Handles multiple database node types
- Credential names/types captured if available
- Execution metrics limited to 24-hour window
- Pagination respects API rate limits
- Composite key prevents duplicates
- Updated timestamp tracks last sync
- Function extracts queries up to 500 chars
- Column extraction handles complex SQL
- RPC function calls identified via regex
- Code node SQL extraction limited to 500 chars

