---
title: "get_all_workflows_n8n"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["n8n", "workflows", "sync", "all-instances"]
version: "91"
---

# get_all_workflows_n8n

Fetches all workflows from all n8n instances (sis1, sis2, sisx, cloud) and stores them in the `n8n_workflows` table.

## Overview
**Function Slug:** get_all_workflows_n8n  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 91

## Purpose
Syncs all workflows from all n8n instances to the database. Stores workflow metadata without extracting database nodes. Simpler version focused on workflow inventory.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get_all_workflows_n8n
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get_all_workflows_n8n \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "Processed 492 workflows across 4 n8n instances",
  "details": [
    {
      "instance": "sis1",
      "totalWorkflows": 247,
      "success": true
    },
    {
      "instance": "sis2",
      "totalWorkflows": 156,
      "success": true
    },
    {
      "instance": "sisx",
      "totalWorkflows": 89,
      "success": true
    },
    {
      "instance": "cloud",
      "totalWorkflows": 0,
      "success": true
    }
  ]
}
```

## Workflow Process

### 1. Fetch All Workflows
- Processes sis1, sis2, sisx, cloud instances
- Uses pagination to fetch all workflows
- Continues until all workflows retrieved

### 2. Store Workflows
- Upserts to `n8n_workflows` table
- Stores workflow ID, name, server, status
- Includes full raw workflow data
- Updates timestamp on each sync

## Database Table

### Table: n8n_workflows
```sql
CREATE TABLE n8n_workflows (
  workflow_id TEXT PRIMARY KEY,
  name TEXT,
  server_name TEXT,
  status TEXT, -- 'active' or 'inactive'
  raw_data JSONB,
  updated_at TIMESTAMP
);
```

## n8n Instances

Processes four instances:
- **sis1:** https://sis1.thesqd.com/api/v1
- **sis2:** https://sis2.thesqd.com/api/v1
- **sisx:** https://sisx.thesqd.com/api/v1
- **cloud:** https://churchmediasquad.app.n8n.cloud/api/v1

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_ANON_KEY` | Yes | Supabase anonymous key |
| `N8N_API_KEY_SIS1` | Yes | n8n API key for sis1 |
| `N8N_API_KEY_SIS2` | Yes | n8n API key for sis2 |
| `N8N_API_KEY_SISX` | Yes | n8n API key for sisx |
| `N8N_API_KEY_CLOUD` | Yes | n8n API key for cloud |

## Differences from n8n_db_workflows

| Feature | n8n_db_workflows | get_all_workflows_n8n |
|---------|------------------|----------------------|
| Focus | Database nodes | All workflows |
| Processing | Extracts DB nodes | Stores workflow data |
| Table | n8n_db_workflows | n8n_workflows |
| Instances | Single instance | All instances |
| Complexity | Higher | Lower |

## Use Cases

1. **Workflow Inventory:** Track all workflows across instances
2. **Discovery:** Find workflows by name or server
3. **Monitoring:** Track workflow status changes
4. **Documentation:** Generate workflow catalogs
5. **Migration:** Prepare for workflow migrations

## Notes

- Processes all four n8n instances
- Stores full workflow data as JSONB
- No database node extraction
- Simpler than `n8n_db_workflows`
- Useful for workflow inventory management
- Cloud instance uses different base URL
- Status derived from `active` field
- Updates timestamp on each sync
- Conflict resolution on `workflow_id`

