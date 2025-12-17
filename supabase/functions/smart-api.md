---
title: "smart-api"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["n8n", "workflows", "executions", "sync"]
version: "84"
---

# smart-api

**Note:** This function is identical to `update-workflow-execution`. See [update-workflow-execution](./update-workflow-execution.md) for full documentation.

## Overview
**Function Slug:** smart-api  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 84

## Purpose
Fetches the latest successful execution for a workflow from n8n and updates the workflow record in Supabase with sample execution data. This is an alias/duplicate of `update-workflow-execution`.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/smart-api
```

## Quick Reference

See [update-workflow-execution.md](./update-workflow-execution.md) for complete documentation including:
- Request/response examples
- Workflow process details
- n8n API integration
- Database schema
- Error handling

## Differences

None - this function is identical to `update-workflow-execution` in functionality.

## Notes

- Duplicate of `update-workflow-execution`
- Same codebase and version
- Consider consolidating to single endpoint
- May be deprecated in favor of `update-workflow-execution`



