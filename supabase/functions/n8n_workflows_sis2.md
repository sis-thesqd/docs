---
title: "n8n_workflows_sis2"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["n8n", "workflows", "sis2", "database"]
version: "92"
---

# n8n_workflows_sis2

Syncs n8n workflows with database nodes from the sis2 instance. Identical to `n8n_db_workflows` but processes only sis2.

## Overview
**Function Slug:** n8n_workflows_sis2  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 92

## Purpose
Same functionality as `n8n_db_workflows` but specifically processes workflows from the sis2 n8n instance. Extracts database nodes and syncs metadata to Supabase.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/n8n_workflows_sis2
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/n8n_workflows_sis2 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "Processed 156 workflows and found 42 workflows with 78 database nodes across 1 n8n instances",
  "details": [
    {
      "instance": "sis2",
      "totalWorkflows": 156,
      "dbNodeWorkflows": 42,
      "totalDbNodes": 78,
      "success": true
    }
  ]
}
```

## Notes

- Processes only sis2 instance
- Same functionality as `n8n_db_workflows`
- Uses sis2 API endpoint: `https://sis2.thesqd.com/api/v1`
- Requires `N8N_API_KEY_SIS2` environment variable
- See [n8n_db_workflows](./n8n_db_workflows.md) for full documentation



