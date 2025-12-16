---
title: "n8n_workflows_sisx"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["n8n", "workflows", "sisx", "database"]
version: "92"
---

# n8n_workflows_sisx

Syncs n8n workflows with database nodes from the sisx instance. Identical to `n8n_db_workflows` but processes only sisx.

## Overview
**Function Slug:** n8n_workflows_sisx  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 92

## Purpose
Same functionality as `n8n_db_workflows` but specifically processes workflows from the sisx n8n instance. Extracts database nodes and syncs metadata to Supabase.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/n8n_workflows_sisx
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/n8n_workflows_sisx \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "Processed 89 workflows and found 23 workflows with 45 database nodes across 1 n8n instances",
  "details": [
    {
      "instance": "sisx",
      "totalWorkflows": 89,
      "dbNodeWorkflows": 23,
      "totalDbNodes": 45,
      "success": true
    }
  ]
}
```

## Notes

- Processes only sisx instance
- Same functionality as `n8n_db_workflows`
- Uses sisx API endpoint: `https://sisx.thesqd.com/api/v1`
- Requires `N8N_API_KEY_SISX` environment variable
- See [n8n_db_workflows](./n8n_db_workflows.md) for full documentation

