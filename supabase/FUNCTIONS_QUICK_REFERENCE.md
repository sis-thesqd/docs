# Supabase Functions Quick Reference

Quick reference guide for all 51 deployed Edge Functions.

## Base URL
```
https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/
```

## Function Categories

### 🔐 Authentication & Authorization (9 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `clickup-auth` | POST | No | ClickUp OAuth flow |
| `clickup-auth-v2` | POST | Yes | Updated ClickUp OAuth |
| `airtable-auth` | GET | No | Airtable OAuth flow |
| `decrypt-dropbox-token` | POST | Yes | Decrypt Dropbox tokens |
| `encrypt-dropbox-token` | POST | Yes | Encrypt Dropbox tokens |
| `encryptToken` | POST | Yes | General token encryption |
| `get_decrypted_secret` | GET | No | Retrieve vault secrets |
| `send-slack-code` | POST | No | Send Slack verification code |
| `send-email-code` | POST | No | Send email verification code |
| `verify-slack-code` | POST | No | Verify Slack code & create session |
| `verify-contractor-passcode` | POST | No | Contractor authentication |

### 📁 File Management (5 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `squad-file-uploader` | POST | No | Upload to S3 or Dropbox |
| `uppy-companion` | POST | No | Uppy file upload companion |
| `tus` | POST/HEAD/PATCH | No | TUS resumable uploads |
| `upload-to-sb-storage` | POST | Yes | Upload to Supabase Storage |
| `delete-storage-file` | POST | Yes | Delete from Supabase Storage |

### 📦 Dropbox Integration (3 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `dropbox-listener` | GET/POST | No | Dropbox webhook handler |
| `log-dropbox-actions` | POST | Yes | Log Dropbox operations |
| `get-dropbox-share-link` | POST | No | Generate Dropbox share links |

### 💾 Database Operations (6 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `execute-sql` | POST | Yes | Execute SQL queries |
| `execute_database_script` | POST | Yes | Run database scripts |
| `execute_query` | POST | Yes | Execute DB queries |
| `list_database_functions` | GET | Yes | List DB functions |
| `list_table_columns` | GET | Yes | List table columns |
| `log-db-actions-v2` | POST | Yes | Log database actions |

### 🔄 n8n Workflow Integration (7 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `n8n_workflows_sis1` | GET | Yes | SIS1 workflow database |
| `n8n_workflows_sis2` | GET | Yes | SIS2 workflow database |
| `n8n_workflows_sisx` | GET | Yes | SISX workflow database |
| `get_all_workflows_n8n` | GET | Yes | Retrieve all n8n workflows |
| `smart-api` | GET | Yes | Get n8n workflow execution |
| `update-workflow-execution` | POST | Yes | Update execution status |
| `get_slow_executions` | GET | Yes | Find slow executions |

### 🔄 Data Backfill & Migration (3 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `backfill-tasks` | POST | Yes | Backfill task data |
| `backfill-v2` | POST | Yes | Updated backfill |
| `create-user-associations` | POST | Yes | Create user associations |

### 📸 Snapshots & Backups (4 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `automated-snapshots` | POST | Yes | Auto database snapshots |
| `manual-snapshot-trigger` | POST | No | Manual snapshot trigger |
| `test-5pm-snapshot` | POST | No | Test 5PM snapshots |
| `test-automated-snapshot` | POST | Yes | Test automated snapshots |

### 👥 User Management (2 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `create-user-associations` | POST | Yes | Create user links |
| `clickup-users-refresh` | POST | Yes | Sync ClickUp users |

### 📊 Analytics & Reporting (2 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `task-analytics` | GET | Yes | Task performance metrics |
| `analyze-transactions` | POST | Yes | Transaction analysis |

### 🌐 External API Proxies (3 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `godaddy-proxy` | POST | Yes | Proxy GoDaddy API requests |
| `notion-to-md` | POST | Yes | Convert Notion to Markdown |
| `airtable-auth` | GET | No | Airtable OAuth |

### 📋 Project Management (4 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `get-project-defaults` | GET | Yes | Get project default settings |
| `prf-defaults-public` | GET | Yes | Public PRF defaults |
| `remix-library-search` | POST | Yes | Search remix library |
| `match-designs` | POST | Yes | Match design assets |

### 🛠️ Utilities (3 functions)
| Function | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `salary-calculator` | POST | No | Calculate salary info |
| `test` | POST | Yes | Test function |

## Quick Examples

### Authenticated Request
```bash
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/{function-name}" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"param": "value"}'
```

### Public Request
```bash
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/{function-name}" \
  -H "Content-Type: application/json" \
  -d '{"param": "value"}'
```

## Authentication

Functions marked "Auth Required: Yes" need:
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

Get JWT from Supabase auth:
```javascript
const { data: { session } } = await supabase.auth.getSession()
const token = session?.access_token
```

## Common Patterns

### File Upload Pattern
```json
{
  "destination": "s3" | "dropbox",
  "uploadPath": "/path/to/destination",
  "fileURL": "https://..." | "file": "base64...",
  "filename": "file.ext"
}
```

### Query Pattern
```json
{
  "query": "SELECT * FROM table WHERE id = $1",
  "params": [123]
}
```

### Pagination Pattern
```
?limit=50&offset=0
```

## Error Handling

All functions return errors in this format:
```json
{
  "error": "Error message",
  "details": "Additional information"
}
```

Common status codes:
- `400` - Bad Request (missing/invalid parameters)
- `401` - Unauthorized (missing/invalid auth)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `429` - Too Many Requests (rate limited)
- `500` - Internal Server Error

## Rate Limits

Functions with rate limiting:
- `godaddy-proxy`: 10 req/min per IP
- `send-slack-code`: 3 req/5min per email
- `send-email-code`: 1 req/min per email
- `verify-contractor-passcode`: 5 req/15min per email

## Support

For detailed documentation on each function, see individual function docs in this directory.

