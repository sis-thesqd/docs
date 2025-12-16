---
title: "get_slow_executions"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["n8n", "executions", "performance", "monitoring", "slow"]
version: "97"
---

# get_slow_executions

Fetches slow n8n workflow executions (4+ minutes) from the last 7 days and stores them in the database for performance analysis.

## Overview
**Function Slug:** get_slow_executions  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 97

## Purpose
Identifies and stores slow n8n workflow executions that took 4 or more minutes to complete. Extracts database nodes from workflows and stores execution details for performance monitoring and optimization.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get_slow_executions
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get_slow_executions \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get_slow_executions', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const data = await response.json()
console.log(`Synced ${data.details[0].executions} slow executions`)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "Successfully synced 23 slow executions (4+ minutes) from 3 n8n instances",
  "details": [
    { "instance": "sis1", "executions": 8 },
    { "instance": "sis2", "executions": 12 },
    { "instance": "sisx", "executions": 3 }
  ]
}
```

### Error Responses

**500 Internal Server Error**
```json
{
  "success": false,
  "error": "Error truncating table: ..."
}
```

## Workflow Process

### 1. Clear Existing Data
- Deletes all rows from `n8n_slow_execs_last_24h`
- Ensures fresh data on each run
- Uses delete with non-matching UUID

### 2. Fetch Executions
- Processes sis1, sis2, sisx instances
- Fetches executions with pagination
- Filters to last 7 days
- Keeps only executions ≥ 4 minutes

### 3. Extract Database Nodes
- Fetches workflow details for each execution
- Extracts database nodes from workflow
- Captures queries, functions, operations

### 4. Store Results
- Inserts execution records with DB node info
- Stores duration, timestamps, metadata
- Links to workflow and instance

## Database Table

### Table: n8n_slow_execs_last_24h
```sql
CREATE TABLE n8n_slow_execs_last_24h (
  execution_id TEXT PRIMARY KEY,
  instance_name TEXT NOT NULL,
  workflow_id TEXT,
  workflow_name TEXT,
  started_at TIMESTAMP,
  stopped_at TIMESTAMP,
  duration_minutes DECIMAL(10,2),
  mode TEXT,
  retries INTEGER,
  data JSONB,
  db_nodes JSONB
);
```

## Filtering Criteria

### Duration Filter
- **Minimum:** 4 minutes (240 seconds)
- **Calculation:** `(stoppedAt - startedAt) / 60000`
- **Precision:** 2 decimal places

### Date Filter
- **Window:** Last 7 days
- **Filter:** `startedAt >= sevenDaysAgo`
- **Format:** ISO timestamp

## Database Node Extraction

### Node Types Detected
- PostgreSQL nodes
- Supabase nodes
- Query/Insert/Update nodes

### Extracted Information
- Node ID and name
- Query text (if present)
- Function name (RPC calls)
- Operation type
- Table name

## Pagination

### Execution Pagination
- Fetches 100 executions per page
- Uses cursor-based pagination
- Maximum 20 pages (2000 executions)
- Prevents infinite loops

### Workflow Fetching
- Fetches workflow for each execution
- Extracts database nodes
- Continues on workflow fetch failure

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_ANON_KEY` | Yes | Supabase anonymous key |
| `N8N_API_KEY_SIS1` | Yes | n8n API key for sis1 |
| `N8N_API_KEY_SIS2` | Yes | n8n API key for sis2 |
| `N8N_API_KEY_SISX` | Yes | n8n API key for sisx |

## n8n Instances

Processes three instances:
- **sis1:** https://sis1.thesqd.com
- **sis2:** https://sis2.thesqd.com
- **sisx:** https://sisx.thesqd.com

## Execution Data Stored

### Execution Metadata
- Execution ID
- Workflow ID and name
- Start/stop timestamps
- Duration in minutes
- Execution mode
- Retry status
- Full execution data (JSONB)

### Database Node Data
- Array of database nodes
- Query text snippets
- Function names
- Table names
- Operations

## Use Cases

1. **Performance Monitoring:** Track slow workflows
2. **Optimization:** Identify bottlenecks
3. **Alerting:** Set up alerts for slow executions
4. **Analysis:** Analyze execution patterns
5. **Debugging:** Investigate performance issues

## Notes

- Clears table before each run (fresh data)
- Only stores executions ≥ 4 minutes
- Fetches workflows to extract DB nodes
- Handles workflow fetch failures gracefully
- Pagination limited to 20 pages max
- Duration calculated with 2 decimal precision
- Retry detection via `retryOf` field
- Full execution data stored as JSONB
- Database nodes stored as JSONB array
- Function name extraction via regex patterns
- Query extraction limited to 200 chars
- Processes all three instances in parallel
- Table name suggests 24h but actually uses 7 days

