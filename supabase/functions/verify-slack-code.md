---
title: "verify-slack-code"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["authentication", "slack", "verification", "session"]
version: "30"
---

# verify-slack-code

Verifies a Slack-delivered code and creates an authenticated session for the user.

## Overview
**Function Slug:** verify-slack-code  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 30  
**Created:** January 31, 2025  
**Last Updated:** December 16, 2025

## Purpose
This is the second step in passwordless Slack authentication. It verifies the 6-digit code sent via [send-slack-code](./send-slack-code.md), creates or retrieves the user in Supabase Auth, and returns a complete authentication session.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/verify-slack-code
```

## Request

### Headers
```
Content-Type: application/json
```

### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Employee's email address |
| `code` | string | Yes | 6-digit verification code |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/verify-slack-code \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "code": "123456"
  }'
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/verify-slack-code', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'john.doe@example.com',
    code: '123456'
  })
})

const data = await response.json()

if (data.access_token) {
  // Store tokens and user info
  localStorage.setItem('access_token', data.access_token)
  localStorage.setItem('refresh_token', data.refresh_token)
  localStorage.setItem('user', JSON.stringify(data.user))
  
  // Redirect to app
  window.location.href = '/dashboard'
}
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "v1.MUNDcHBabFlGNUQ2MGVPbmpPOUhpTXc...",
  "user": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "email": "john.doe@example.com",
    "name": "John Doe"
  }
}
```

### Error Responses

**401 Unauthorized** - Invalid or expired code
```json
{
  "error": "Invalid or expired verification code"
}
```

**401 Unauthorized** - Code expired
```json
{
  "error": "Verification code has expired"
}
```

**404 Not Found** - Employee not found
```json
{
  "error": "Employee not found"
}
```

**500 Internal Server Error** - Failed to create session
```json
{
  "error": "Failed to create user session"
}
```

```json
{
  "error": "Failed to create session"
}
```

## Workflow Process

### 1. Validate Code
- Queries `auth_codes` table for matching email and code
- Checks code hasn't been used (`used: false`)
- Verifies code hasn't expired

### 2. Mark Code as Used
- Updates code record to set `used: true`
- Prevents code reuse

### 3. Get Employee Data
- Fetches employee details from `employees` table
- Retrieves name and Airtable record ID

### 4. Create/Retrieve User
- Checks if user exists in `auth.users` table
- If exists: retrieves existing user ID
- If new: creates user with `createUser()` method
- Sets `email_confirm: true` (no email verification needed)
- Adds user metadata (name, employee_id)

### 5. Generate Session
- Creates temporary secure password (UUID)
- Updates user's password
- Signs in with password to get valid session
- Returns access_token and refresh_token

### 6. Cleanup
- Deletes expired codes from `auth_codes` table
- Keeps database clean

## Database Tables

### Supabase
- `auth_codes` (read/write) - Code verification and cleanup
- `employees` (read) - Employee information
- `auth.users` (read/write) - Supabase authentication users

### Table: auth_codes
```sql
Columns used:
- id (uuid)
- email (text, indexed)
- code (text)
- expires_at (timestamp)
- used (boolean)
- created_at (timestamp)
```

### Table: employees
```sql
Columns used:
- email (text, indexed)
- name (text)
- airtable_rec_id (text)
```

### Table: auth.users
```sql
Supabase Auth table:
- id (uuid)
- email (text)
- email_confirmed_at (timestamp)
- encrypted_password (text)
- user_metadata (jsonb)
```

## Security Features

### Code Validation
- Single-use codes (marked as used after verification)
- 5-minute expiration window
- Must match email exactly
- Prevents replay attacks

### Session Security
- Generates cryptographically random temporary password
- Immediately replaces temporary password after use
- Uses Supabase's built-in JWT authentication
- Tokens are short-lived and refreshable

### User Creation
- Email auto-confirmed (no verification email needed)
- Metadata includes employee tracking info
- Uses service role for admin operations

## Session Tokens

### Access Token
- **Type:** JWT (JSON Web Token)
- **Duration:** 1 hour (default)
- **Use:** Include in Authorization header for API calls
- **Format:** `Bearer {access_token}`

### Refresh Token
- **Duration:** 30 days (default)
- **Use:** Obtain new access token when expired
- **Storage:** Store securely (httpOnly cookie recommended)

### Using Tokens

```javascript
// API request with access token
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/rest/v1/employees', {
  headers: {
    'Authorization': `Bearer ${access_token}`,
    'apikey': 'your_anon_key'
  }
})

// Refresh session when access token expires
const { data, error } = await supabase.auth.refreshSession({
  refresh_token: refresh_token
})
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key with admin access |

## Authentication Flow

```
1. User enters email → send-slack-code
2. User receives code in Slack
3. User enters code → verify-slack-code (this function)
4. Function returns access_token + refresh_token
5. Client stores tokens
6. Client includes access_token in all API requests
7. Client refreshes token when expired
```

## Use Cases

1. **Employee Login:** Primary login method for internal employees
2. **Passwordless Auth:** No passwords to manage or reset
3. **Slack Integration:** Leverages existing Slack workspace
4. **Quick Access:** Fast authentication for frequent users

## Related Functions

- [send-slack-code](./send-slack-code.md) - Request verification code
- [send-email-code](./send-email-code.md) - Alternative email-based codes
- [verify-contractor-passcode](./verify-contractor-passcode.md) - Contractor auth

## Error Handling

### Common Issues

**Invalid Code**
- Code may have been typed incorrectly
- Code may have already been used
- Code may have expired (5 min limit)
- Request new code via send-slack-code

**User Creation Failed**
- Check `SUPABASE_SERVICE_ROLE_KEY` is valid
- Verify service role has admin permissions
- Check employee email is valid format

**Session Creation Failed**
- Verify auth is enabled in Supabase project
- Check JWT secret is configured
- Review Supabase auth logs

## Testing

### Complete Flow Test

```bash
# Step 1: Send code
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-slack-code \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com"}'

# Step 2: Check Slack for code (or check database)

# Step 3: Verify code
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/verify-slack-code \
  -H "Content-Type: application/json" \
  -d '{
    "email":"test@example.com",
    "code":"123456"
  }'
```

### Check Session in Database
```sql
-- View created auth user
SELECT id, email, email_confirmed_at, user_metadata 
FROM auth.users 
WHERE email = 'test@example.com';

-- Check code was marked as used
SELECT * FROM auth_codes 
WHERE email = 'test@example.com' 
ORDER BY created_at DESC 
LIMIT 1;
```

## Notes

- Automatically cleans up expired codes to maintain database health
- Creates new users with `email_confirm: true` (no verification email)
- Uses temporary password technique to generate valid session
- Session tokens should be stored securely (never in localStorage for production)
- Consider implementing httpOnly cookies for token storage
- Refresh tokens before expiration to maintain seamless user experience
- User metadata includes employee tracking for analytics

