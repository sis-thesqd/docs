---
title: "update-workflow-execution"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["n8n", "workflows", "executions", "sync"]
version: "84"
---

# update-workflow-execution

Fetches the latest successful execution for a workflow from n8n and updates the workflow record in Supabase with sample execution data.

## Overview
**Function Slug:** update-workflow-execution  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 84

## Purpose
Retrieves the most recent successful execution for a specific n8n workflow and stores it as sample execution data in the `n8n_workflows` table. Used for workflow documentation and testing.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/update-workflow-execution
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
| `workflow_id` | string | Yes | n8n workflow ID |
| `server_name` | string | No | n8n instance (sis1/sis2/sisx/cloud) |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/update-workflow-execution \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "workflow_id": "123456",
    "server_name": "sis1"
  }'
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/update-workflow-execution', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    workflow_id: '123456',
    server_name: 'sis1'
  })
})

const data = await response.json()
console.log(`Updated workflow ${data.workflow_id}`)
console.log(`Execution ID: ${data.execution_id}`)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "workflow_id": "123456",
  "server_name": "sis1",
  "execution_id": "789012",
  "execution_date": "2025-12-16T10:30:00.000Z",
  "message": "Workflow execution data updated successfully"
}
```

### Error Responses

**400 Bad Request** - Missing workflow_id
```json
{
  "error": "workflow_id is required"
}
```

**404 Not Found** - Workflow not found
```json
{
  "error": "Workflow not found",
  "details": "..."
}
```

**404 Not Found** - No executions
```json
{
  "error": "No successful executions found for this workflow",
  "workflow_id": "123456"
}
```

**500 Internal Server Error** - n8n API error
```json
{
  "error": "Failed to fetch execution data from n8n",
  "status": 401,
  "statusText": "Unauthorized"
}
```

## Workflow Process

### 1. Lookup Workflow
- Queries `n8n_workflows` table
- Gets `server_name` if not provided
- Validates workflow exists

### 2. Determine Server
- Uses provided `server_name` or from DB
- Validates against allowed instances
- Gets API key for server

### 3. Fetch Latest Execution
- Calls n8n API for executions
- Filters to successful executions only
- Limits to 1 result (latest)
- Includes execution data

### 4. Update Database
- Updates `sample_execution` field
- Sets `updated_at` timestamp
- Stores full execution JSON

## n8n API Integration

### Endpoint
```
GET {baseUrl}/api/v1/executions
```

### Query Parameters
- `includeData=true` - Include execution data
- `status=success` - Only successful executions
- `workflowId={id}` - Filter by workflow
- `limit=1` - Get latest only

### Base URLs
- **sis1:** https://sis1.n8n.co
- **sis2:** https://sis2.n8n.co
- **sisx:** https://sisx.n8n.co
- **cloud:** https://cloud.n8n.co

## Database Table

### Table: n8n_workflows
```sql
UPDATE n8n_workflows
SET 
  sample_execution = {...},
  updated_at = NOW()
WHERE workflow_id = {workflow_id}
```

### Sample Execution Field
Stores full execution object:
- Execution ID
- Start/stop timestamps
- Duration
- Node execution data
- Input/output data
- Status and mode

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_ANON_KEY` | Yes | Supabase anonymous key |
| `N8N_API_KEY_SIS1` | Yes | n8n API key for sis1 |
| `N8N_API_KEY_SIS2` | Yes | n8n API key for sis2 |
| `N8N_API_KEY_SISX` | Yes | n8n API key for sisx |
| `N8N_API_KEY_CLOUD` | Yes | n8n API key for cloud |

## Use Cases

1. **Documentation:** Store sample executions for reference
2. **Testing:** Use sample data for testing workflows
3. **Debugging:** Analyze execution patterns
4. **Monitoring:** Track workflow execution history
5. **Onboarding:** Show example executions to new users

## Notes

- Only fetches successful executions
- Gets most recent execution (limit=1)
- Includes full execution data
- Server name can be inferred from DB
- Updates timestamp on each run
- Handles missing server_name gracefully
- Validates server_name against allowed list
- Returns execution ID and date for confirmation
- Full execution stored as JSONB
- Used for workflow documentation purposes



