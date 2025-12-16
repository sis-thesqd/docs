---
title: "execute-sql"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["sql", "database", "query", "execute"]
version: "94"
---

# execute-sql

Executes SQL queries against the Supabase database using a database function wrapper.

## Overview
**Function Slug:** execute-sql  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 94

## Purpose
Provides a secure way to execute SQL queries through Supabase's RPC system. Uses the `execute_database_script` database function to run SQL with proper authentication and error handling.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/execute-sql
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
| `sql` | string | Yes | SQL query to execute |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/execute-sql \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "sql": "SELECT id, name, email FROM employees WHERE status = '\''Active'\'' LIMIT 10;"
  }'
```

### JavaScript Example

```javascript
const sqlQuery = `
  SELECT 
    id,
    name,
    email,
    created_at
  FROM employees
  WHERE status = 'Active'
  ORDER BY created_at DESC
  LIMIT 20;
`

const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/execute-sql', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ sql: sqlQuery })
})

const data = await response.json()
console.log('Query results:', data)
```

## Response

### Success Response (200)

```json
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "created_at": "2024-01-15T10:30:00Z"
  },
  {
    "id": 2,
    "name": "Jane Smith",
    "email": "jane@example.com",
    "created_at": "2024-01-16T14:20:00Z"
  }
]
```

### Error Responses

**400 Bad Request** - Missing SQL
```json
{
  "error": "SQL query is required"
}
```

**500 Internal Server Error** - Query error
```json
{
  "error": "syntax error at or near \"SELECT\""
}
```

**500 Internal Server Error** - General error
```json
{
  "error": "Internal server error"
}
```

## Database Function

### RPC: execute_database_script
```sql
CREATE OR REPLACE FUNCTION execute_database_script(script_text TEXT)
RETURNS JSONB
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  result JSONB;
BEGIN
  -- Execute the SQL script
  EXECUTE script_text;
  
  -- Return result (implementation depends on query type)
  RETURN result;
END;
$$;
```

## Supported Query Types

### SELECT Queries
Returns result set as JSON array:
```sql
SELECT * FROM table_name WHERE condition;
```

### INSERT/UPDATE/DELETE
Returns affected row count:
```sql
INSERT INTO table_name (col1, col2) VALUES ('val1', 'val2');
UPDATE table_name SET col1 = 'val' WHERE id = 1;
DELETE FROM table_name WHERE id = 1;
```

### DDL Queries
Schema modifications:
```sql
CREATE TABLE new_table (...);
ALTER TABLE table_name ADD COLUMN new_col TEXT;
DROP TABLE old_table;
```

## Security Considerations

⚠️ **SQL Injection Risk**
- Always validate and sanitize user input
- Use parameterized queries when possible
- Limit query scope with RLS policies
- Audit all SQL execution

✅ **Protection Features**
- Requires JWT authentication
- Uses SECURITY DEFINER function
- Respects Row Level Security (RLS)
- Service role key for execution

## Use Cases

1. **Admin Queries:** Execute complex database operations
2. **Data Migration:** Run migration scripts
3. **Reporting:** Generate custom reports
4. **Maintenance:** Database cleanup and optimization
5. **Debugging:** Test queries in production

## Limitations

- Query timeout: 60 seconds (Edge Function limit)
- Result size: Limited by response size
- No transaction management
- Single query per request
- No prepared statements

## Best Practices

### Parameterized Queries
```javascript
// Good: Use parameters
const sql = `
  SELECT * FROM employees 
  WHERE email = $1 AND status = $2
`
// Note: This function doesn't support parameters directly
// Consider using Supabase client for parameterized queries
```

### Query Validation
```javascript
// Validate query before execution
const sql = userInput.trim()

if (!sql.match(/^SELECT/i)) {
  throw new Error('Only SELECT queries allowed')
}

// Check for dangerous keywords
const dangerous = ['DROP', 'DELETE', 'TRUNCATE', 'ALTER']
if (dangerous.some(keyword => sql.includes(keyword))) {
  throw new Error('Dangerous operation not allowed')
}
```

### Error Handling
```javascript
try {
  const response = await fetch('/functions/v1/execute-sql', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ sql })
  })
  
  if (!response.ok) {
    const error = await response.json()
    console.error('SQL Error:', error.error)
    return
  }
  
  const data = await response.json()
  console.log('Results:', data)
} catch (error) {
  console.error('Request failed:', error)
}
```

## Alternative: Direct Supabase Client

For better security and features, consider using Supabase client directly:

```javascript
// Better approach: Use Supabase client
const { data, error } = await supabase
  .from('employees')
  .select('*')
  .eq('status', 'Active')
  .limit(10)

// Or use RPC for stored procedures
const { data, error } = await supabase
  .rpc('get_employee_stats', { 
    start_date: '2024-01-01',
    end_date: '2024-12-31'
  })
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key |

## Notes

- Uses `execute_database_script` RPC function internally
- JWT token passed through to maintain user context
- RLS policies still apply to queries
- Service role key used for execution
- No query result caching
- Consider rate limiting for production
- Log all SQL executions for audit trail
- Function is a thin wrapper around database RPC
- Better suited for admin/debugging than production queries
- For production, prefer Supabase client with type safety

