---
title: "send-email-code"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["authentication", "email", "verification", "passwordless"]
version: "17"
---

# send-email-code

Sends a verification code to an employee's email address for passwordless authentication.

## Overview
**Function Slug:** send-email-code  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 17  
**Created:** January 26, 2025  
**Last Updated:** December 16, 2025

## Purpose
Alternative to Slack-based authentication. Sends a beautifully formatted 6-digit verification code via email using Resend API. Part of a two-step passwordless authentication flow.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-email-code
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
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-email-code \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com"
  }'
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-email-code', {
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
  "employeeName": "John Doe",
  "message": "Verification code sent to your email"
}
```

### Error Responses

**400 Bad Request** - Missing email
```json
{
  "error": "Email is required"
}
```

**404 Not Found** - Email not in employees table
```json
{
  "error": "Email not found in our system"
}
```

**429 Too Many Requests** - Rate limit exceeded
```json
{
  "error": "Please wait a minute before requesting another code"
}
```

**500 Internal Server Error** - Failed to generate or send code
```json
{
  "error": "Failed to generate verification code"
}
```

```json
{
  "error": "Failed to send verification email. Please try again."
}
```

## Workflow Process

### 1. Validate Input
- Checks email parameter exists
- Returns 400 if missing

### 2. Lookup Employee
- Queries `employees` table by email (case-insensitive)
- Verifies employee exists in system

### 3. Rate Limiting
- Maximum 1 code request per minute per email
- Prevents spam and abuse
- Checks last code sent time

### 4. Generate Code
- Creates random 6-digit code (100000-999999)
- Sets 10-minute expiration time
- Stores in `auth_codes` table with `used: false` flag

### 5. Send Email via Resend
- Uses Resend API for email delivery
- Sends beautifully formatted HTML email
- Includes code, employee name, and expiration notice

### 6. Return Success
- Confirms email was sent
- Returns employee name for UI personalization

## Email Template

### Subject
```
Your Login Verification Code
```

### HTML Email Content

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
  </head>
  <body>
    <div class="header">
      <h1>🔐 Verification Code</h1>
    </div>
    
    <p>Hello {employee_name},</p>
    
    <p>You requested a login verification code for your Church Media Squad account.</p>
    
    <div class="code-container">
      <p>Your verification code is:</p>
      <div class="code">{6-digit-code}</div>
      <p>This code will expire in 10 minutes.</p>
    </div>
    
    <p>Enter this code on the login page to access your account.</p>
    
    <p><strong>Important:</strong> If you didn't request this code, please ignore this email.</p>
    
    <div class="footer">
      <p>This is an automated message from Church Media Squad.<br>
      Please do not reply to this email.</p>
    </div>
  </body>
</html>
```

### Email Styling
- Professional, modern design
- Large, easy-to-read code display
- Mobile-responsive layout
- Branded colors and typography

## Database Tables

### Supabase
- `employees` (read) - Employee lookup
- `auth_codes` (read/write) - Store and track verification codes

### Table: employees
```sql
Columns used:
- email (text, indexed)
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
- 1 request per minute per email
- Stricter than Slack (which allows 3 per 5 min)
- Prevents email flooding
- Automatic cleanup of old codes

### Code Security
- 6-digit random code (1 million combinations)
- 10-minute expiration (longer than Slack's 5 min)
- One-time use only
- Cannot reuse expired or used codes

### Email Security
- Uses professional email service (Resend)
- SPF/DKIM authentication
- From address: Church Media Squad
- Clear warning about unsolicited codes

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key for database access |
| `RESEND_API_KEY` | Yes | Resend API key for sending emails |

## Resend Setup

### Getting Resend API Key
1. Sign up at resend.com
2. Verify your domain (or use onboarding@resend.dev for testing)
3. Create API key
4. Add key to Supabase secrets as `RESEND_API_KEY`

### Email Sending Limits
- **Free Tier:** 100 emails/day
- **Paid Plans:** Higher limits available
- Monitor usage at resend.com/dashboard

### Domain Configuration
For production, configure custom domain:
1. Add domain in Resend dashboard
2. Add DNS records (SPF, DKIM, DMARC)
3. Wait for verification
4. Update `from` address in code

## Comparison with Slack Method

| Feature | Email | Slack |
|---------|-------|-------|
| Rate Limit | 1/min | 3/5min |
| Expiration | 10 min | 5 min |
| Setup Required | Email service | Slack bot |
| User Requirement | Email address | Slack account |
| Best For | External users | Internal team |

## Use Cases

1. **Employee Login:** Alternative to Slack authentication
2. **External Contractors:** Users without Slack access
3. **Mobile Access:** Easy to access email on mobile
4. **Backup Method:** If Slack is down or unavailable
5. **Email-Preferred Users:** Some users prefer email

## Related Functions

- [verify-contractor-passcode](./verify-contractor-passcode.md) - Verify email codes
- [send-slack-code](./send-slack-code.md) - Alternative Slack-based codes
- [verify-slack-code](./verify-slack-code.md) - Verify Slack codes

## Error Handling

### Common Issues

**Employee Not Found**
- Ensure email is in `employees` table
- Check email spelling
- Email lookup is case-insensitive

**Rate Limit Hit**
- Wait 60 seconds before requesting new code
- Check for processes spamming requests
- Review recent codes in database

**Email Not Received**
- Check spam/junk folder
- Verify Resend API key is valid
- Check Resend dashboard for delivery status
- Verify domain is properly configured

**Failed to Send**
- Check Resend API key
- Verify email quota not exceeded
- Check Resend service status
- Review Resend logs for errors

## Testing

### Test Request
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/send-email-code \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com"}'
```

### Check Database
```sql
-- View generated code
SELECT email, code, expires_at, used, created_at 
FROM auth_codes 
WHERE email = 'test@example.com' 
ORDER BY created_at DESC 
LIMIT 1;
```

### Check Resend Dashboard
1. Log into resend.com
2. Go to "Logs" section
3. Find email by recipient address
4. Check delivery status

## Email Deliverability Tips

1. **Use Custom Domain:** Better deliverability than resend.dev
2. **Configure SPF/DKIM:** Required for custom domains
3. **Add DMARC:** Improves email reputation
4. **Monitor Bounces:** Remove invalid emails
5. **Check Spam Reports:** Address issues quickly
6. **Warm Up Domain:** Gradually increase sending volume

## Notes

- Longer expiration (10 min) vs Slack (5 min) due to email delivery delays
- Stricter rate limit (1/min) to prevent email abuse
- Professional HTML email improves user experience
- Code is prominently displayed for easy reading
- Mobile-responsive design for smartphone access
- Consider implementing CAPTCHA for public-facing endpoints
- Resend provides excellent deliverability and analytics
- Development uses `onboarding@resend.dev` - replace for production

