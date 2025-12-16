---
title: "clickup-auth-v2"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["clickup", "oauth", "authentication", "session"]
version: "92"
---

# clickup-auth-v2

Updated ClickUp OAuth authentication flow with improved error handling, return URL support, and iframe compatibility.

## Overview
**Function Slug:** clickup-auth-v2  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 92

## Purpose
Enhanced version of ClickUp OAuth authentication with better error messages, return URL handling, and iframe support. Creates session-based authentication with 7-day expiration.

## Endpoint

```
GET/POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-auth-v2
```

## Request

### GET Request (OAuth Callback)

```
GET /functions/v1/clickup-auth-v2?code={auth_code}&isInIframe={true|false}&returnUrl={url}
```

### POST Request

#### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
Content-Type: application/json
```

#### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | OAuth authorization code from ClickUp |
| `isInIframe` | boolean | No | Whether auth is happening in iframe |
| `returnUrl` | string | No | URL to redirect after authentication |

### Sample Requests

#### GET Request (OAuth Callback)
```bash
curl -X GET \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-auth-v2?code=abc123&isInIframe=false&returnUrl=https://app.example.com/dashboard" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

#### POST Request
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-auth-v2 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "abc123def456",
    "isInIframe": false,
    "returnUrl": "https://app.example.com/dashboard"
  }'
```

### JavaScript Example

```javascript
// OAuth callback handler
const urlParams = new URLSearchParams(window.location.search)
const code = urlParams.get('code')
const returnUrl = urlParams.get('returnUrl')

const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-auth-v2', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    code,
    isInIframe: window.self !== window.top,
    returnUrl: returnUrl || window.location.origin + '/dashboard'
  })
})

const data = await response.json()

if (data.session) {
  // Store session
  localStorage.setItem('clickup_session', JSON.stringify(data.session))
  
  // Redirect
  window.location.href = data.redirectUrl
}
```

## Response

### Success Response (200)

```json
{
  "session": {
    "session_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "user": {
      "id": "123456",
      "username": "john.doe",
      "email": "john@example.com",
      "profile_picture": "https://...",
      "team_id": "1235435",
      "team_name": "My Team"
    },
    "expires_at": "2025-12-23T10:30:00.000Z"
  },
  "redirectUrl": "https://requests.thesqd.com/"
}
```

### Error Responses

**400 Bad Request** - Missing code
```json
{
  "error": "Authorization code is required"
}
```

**400 Bad Request** - Invalid JSON
```json
{
  "error": "Invalid JSON in request body"
}
```

**400 Bad Request** - Token exchange failed
```json
{
  "error": "Authorization code has expired - please try authenticating again"
}
```

**500 Internal Server Error** - Configuration error
```json
{
  "error": "Server configuration error: Missing redirect URI."
}
```

## OAuth Flow

1. **User Redirects to ClickUp**
   ```
   https://app.clickup.com/api/v2/oauth?client_id={CLIENT_ID}&redirect_uri={REDIRECT_URI}
   ```

2. **ClickUp Redirects Back**
   ```
   {REDIRECT_URI}?code={AUTHORIZATION_CODE}
   ```

3. **Exchange Code for Token**
   - Function calls ClickUp API
   - Receives access token

4. **Get User Data**
   - Fetch user profile from ClickUp
   - Get team information

5. **Create Session**
   - Generate session ID
   - Store in `clickup_sessions` table
   - Set 7-day expiration

6. **Return Redirect URL**
   - Use provided `returnUrl` if valid
   - Fallback to app origin
   - Validate hostname matches

## Return URL Handling

### Absolute URLs
```javascript
// Valid: Same hostname
returnUrl: "https://requests.thesqd.com/dashboard"
// Uses provided URL

// Invalid: Different hostname
returnUrl: "https://evil.com/steal"
// Falls back to app origin
```

### Relative URLs
```javascript
// Valid: Path only
returnUrl: "/dashboard"
// Becomes: https://requests.thesqd.com/dashboard

// Valid: Path with query
returnUrl: "/dashboard?tab=projects"
// Becomes: https://requests.thesqd.com/dashboard?tab=projects
```

### Default Behavior
```javascript
// No returnUrl provided
// Defaults to: {CLICKUP_REDIRECT_URI origin} + "/"
```

## Iframe Support

### Detection
```javascript
const isInIframe = window.self !== window.top
```

### Use Cases
- Embedded authentication widgets
- Popup authentication flows
- Cross-domain authentication

## Database Table

### Table: clickup_sessions
```sql
CREATE TABLE clickup_sessions (
  session_id UUID PRIMARY KEY,
  clickup_user_id TEXT NOT NULL,
  clickup_username TEXT,
  clickup_email TEXT,
  clickup_access_token TEXT NOT NULL,
  clickup_token_type TEXT,
  clickup_team_id TEXT,
  profile_picture TEXT,
  metadata JSONB,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_ANON_KEY` | Yes | Supabase anonymous key |
| `CLICKUP_CLIENT_ID` | Yes | ClickUp OAuth client ID |
| `CLICKUP_CLIENT_SECRET` | Yes | ClickUp OAuth client secret |
| `CLICKUP_REDIRECT_URI` | Yes | OAuth redirect URI |

## Error Handling Improvements

### Specific Error Messages
- **401:** "Invalid client credentials"
- **400 (expired):** "Authorization code has expired"
- **400 (invalid):** "Invalid authorization code"
- **500:** "Server configuration error"

### Logging
- Request method and URL logged
- Token exchange status logged
- User email logged
- Redirect URL decisions logged
- Errors logged with context

## Differences from v1

✅ **Better Error Messages** - Specific error types  
✅ **Return URL Support** - Custom redirect URLs  
✅ **Iframe Detection** - Handles embedded auth  
✅ **Hostname Validation** - Security for return URLs  
✅ **Enhanced Logging** - Better debugging  
✅ **Relative URL Support** - Flexible redirects  

## Security Features

✅ **Hostname Validation** - Prevents open redirects  
✅ **JWT Authentication** - Requires valid token  
✅ **Session Expiration** - 7-day limit  
✅ **Secure Token Storage** - Server-side only  
✅ **Error Sanitization** - No sensitive data leaked  

## Use Cases

1. **Web App Authentication:** Standard OAuth flow
2. **Embedded Widgets:** Iframe authentication
3. **Mobile Apps:** OAuth callback handling
4. **Custom Redirects:** Post-auth navigation
5. **Session Management:** Long-lived sessions

## Testing

### Test OAuth Flow
```bash
# 1. Get authorization code from ClickUp
# Visit: https://app.clickup.com/api/v2/oauth?client_id={ID}&redirect_uri={URI}

# 2. Exchange code (simulated)
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/clickup-auth-v2 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "test_code_here",
    "returnUrl": "https://requests.thesqd.com/dashboard"
  }'
```

## Notes

- Supports both GET and POST methods
- Return URL validation prevents open redirects
- Iframe mode detected automatically
- Default team ID used if none found
- Session expires after 7 days
- Access token stored in database
- Profile picture URL included
- Team information stored in metadata
- Redirect URL defaults to app origin
- Comprehensive error logging for debugging

