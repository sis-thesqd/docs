---
title: "clickup-auth"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["clickup", "oauth", "authentication", "legacy"]
version: "140"
---

# clickup-auth

Legacy ClickUp OAuth authentication flow. Superseded by `clickup-auth-v2` with improved features.

## Overview
**Function Slug:** clickup-auth  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 140

## Purpose
Original ClickUp OAuth authentication implementation. Creates session-based authentication with 7-day expiration. Consider using `clickup-auth-v2` for new implementations.

## Endpoint

```
GET/POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-auth
```

## Differences from clickup-auth-v2

| Feature | clickup-auth | clickup-auth-v2 |
|---------|--------------|-----------------|
| Return URL Support | Basic origin detection | Full return URL support |
| Error Messages | Generic | Specific error types |
| Iframe Support | Basic | Enhanced |
| Hostname Validation | No | Yes |
| Logging | Basic | Comprehensive |

## Notes

- Legacy version maintained for compatibility
- Same core OAuth flow as v2
- Uses same session table (`clickup_sessions`)
- 7-day session expiration
- Basic redirect URL handling
- Consider migrating to `clickup-auth-v2`
- See [clickup-auth-v2](./clickup-auth-v2.md) for full documentation

