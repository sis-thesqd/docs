# send-slack-code

Sends a verification code to an employee's Slack account for passwordless authentication.

## Overview
**Function Slug:** send-slack-code  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 30  
**Created:** January 31, 2025  
**Last Updated:** December 16, 2025

## Purpose
This function provides passwordless authentication by sending a 6-digit verification code via Slack direct message. It's part of a two-step authentication flow where users request a code and then verify it to gain access.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-slack-code
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

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-slack-code \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com"
  }'
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-slack-code', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'john.doe@example.com'
  })
})

const data = await response.json()
console.log(data)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "Verification code sent to your Slack",
  "employeeName": "John Doe"
}
```

### Error Responses

**404 Not Found** - Email not in employees table
```json
{
  "error": "Email not found in employees list"
}
```

**400 Bad Request** - No Slack ID configured
```json
{
  "error": "No Slack ID configured for this employee"
}
```

**429 Too Many Requests** - Rate limit exceeded
```json
{
  "error": "Too many code requests. Please try again later."
}
```

**500 Internal Server Error** - Failed to generate or send code
```json
{
  "error": "Failed to generate code"
}
```

```json
{
  "error": "Failed to send Slack message: {error_details}"
}
```

## Workflow Process

### 1. Validate Email
- Checks if email exists in `employees` table
- Verifies employee has a `slack_id` configured
- Email lookup is case-insensitive and trimmed

### 2. Rate Limiting
- Maximum 3 code requests per 5 minutes per email
- Checks `auth_codes` table for recent requests
- Returns 429 error if limit exceeded

### 3. Generate Code
- Creates random 6-digit code (100000-999999)
- Sets 5-minute expiration time
- Stores in `auth_codes` table with `used: false` flag

### 4. Send via Slack
- Uses Slack `chat.postMessage` API
- Sends direct message to user's Slack account
- Message includes code and expiration notice

### 5. Return Success
- Confirms code was sent
- Returns employee name for UI personalization

## Database Tables

### Supabase
- `employees` (read) - Employee lookup by email
- `auth_codes` (read/write) - Store and track verification codes

### Table: employees
```sql
Columns used:
- email (text, indexed)
- slack_id (text, required for this function)
- name (text)
```

### Table: auth_codes
```sql
Columns:
- id (uuid, primary key)
- email (text, indexed)
- code (text, 6 digits)
- expires_at (timestamp)
- used (boolean, default false)
- created_at (timestamp)
```

## Security Features

### Rate Limiting
- 3 requests per 5 minutes per email
- Prevents code flooding attacks
- Automatic cleanup of old codes

### Code Security
- 6-digit random code (1 million combinations)
- 5-minute expiration
- One-time use (marked as used after verification)
- Cannot reuse expired or used codes

### Access Control
- Employee must exist in database
- Employee must have Slack ID configured
- No authentication required to request (allows login flow)

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key for database access |
| `SLACK_BOT_TOKEN` | Yes | Slack bot OAuth token (xoxb-*) |

## Slack Bot Setup

### Required Scopes
- `chat:write` - Send messages to users
- `users:read` - Read user information
- `users:read.email` - Read user email addresses

### Bot Configuration
1. Create Slack app at api.slack.com/apps
2. Enable "Bot Token Scopes"
3. Install app to workspace
4. Copy Bot User OAuth Token to `SLACK_BOT_TOKEN`

## Slack Message Format

```
🔐 Your login verification code is: *{CODE}*

This code will expire in 5 minutes.

If you didn't request this code, please ignore this message.
```

## Use Cases

1. **Passwordless Login:** Primary authentication method for employees
2. **Quick Access:** No password to remember or reset
3. **Secure Access:** Requires both email access (employee list) and Slack access
4. **Mobile-Friendly:** Easy to receive codes on mobile devices

## Related Functions

- [verify-slack-code](./verify-slack-code.md) - Verify the code and create session
- [send-email-code](./send-email-code.md) - Alternative email-based verification
- [verify-contractor-passcode](./verify-contractor-passcode.md) - Contractor authentication

## Error Handling

### Common Issues

**Employee Not Found**
- Ensure email is in `employees` table
- Check email spelling and case
- Email should match exactly (case-insensitive)

**No Slack ID**
- Employee record must have `slack_id` field populated
- Slack ID format: U01AB2CD3EF (starts with U)
- Update employee record with correct Slack user ID

**Slack Message Failed**
- Verify `SLACK_BOT_TOKEN` is valid
- Check bot has required scopes
- Ensure bot is installed in workspace
- User must be member of workspace

**Rate Limit Hit**
- Wait 5 minutes before requesting new code
- Check for stuck processes spamming requests
- Review `auth_codes` table for suspicious activity

## Testing

### Test Request
```bash
# Replace with actual employee email
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-slack-code \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com"}'
```

### Verify in Database
```sql
-- Check if code was created
SELECT * FROM auth_codes 
WHERE email = 'test@example.com' 
ORDER BY created_at DESC 
LIMIT 1;
```

## Notes

- Codes expire after 5 minutes
- Old expired codes should be cleaned up periodically
- Consider implementing CAPTCHA for production to prevent abuse
- Slack user ID can be found: Slack Profile > More > Copy member ID
- The function uses service role key to bypass RLS policies

