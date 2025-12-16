---
title: "godaddy-proxy"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["proxy", "godaddy", "dns", "api", "rate-limiting"]
version: "102"
---

# godaddy-proxy

Secure proxy for GoDaddy API requests, protecting API credentials from client exposure.

## Overview
**Function Slug:** godaddy-proxy  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 102  
**Created:** January 17, 2025  
**Last Updated:** January 17, 2025

## Purpose
Provides a secure intermediary for GoDaddy API requests, keeping API keys server-side while allowing authenticated clients to manage DNS records, domains, and other GoDaddy services. Includes rate limiting and request logging.

## Endpoint

```
GET/POST/PUT/DELETE https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/godaddy-proxy
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
Content-Type: application/json
X-HTTP-Method: {actual_method}  (optional, for method override)
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `endpoint` | string | Yes | GoDaddy API endpoint path (e.g., `/v1/domains/thesqd.com/records`) |

### Sample Requests

#### Get DNS Records
```bash
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/godaddy-proxy?endpoint=/v1/domains/thesqd.com/records" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -H "X-HTTP-Method: GET"
```

#### Create DNS Record
```bash
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/godaddy-proxy?endpoint=/v1/domains/thesqd.com/records" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -H "X-HTTP-Method: POST" \
  -d '[
    {
      "data": "192.168.1.1",
      "name": "subdomain",
      "ttl": 600,
      "type": "A"
    }
  ]'
```

#### Update DNS Record
```bash
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/godaddy-proxy?endpoint=/v1/domains/thesqd.com/records/A/subdomain" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -H "X-HTTP-Method: PUT" \
  -d '[
    {
      "data": "192.168.1.2",
      "ttl": 3600
    }
  ]'
```

#### Delete DNS Record
```bash
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/godaddy-proxy?endpoint=/v1/domains/thesqd.com/records/A/subdomain" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -H "X-HTTP-Method: DELETE"
```

### JavaScript Example

```javascript
// Helper function to call GoDaddy API
async function callGoDaddyAPI(endpoint, method = 'GET', body = null) {
  const url = `https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/godaddy-proxy?endpoint=${encodeURIComponent(endpoint)}`
  
  const options = {
    method: 'POST', // Always POST to the proxy
    headers: {
      'Authorization': `Bearer ${supabaseToken}`,
      'Content-Type': 'application/json',
      'X-HTTP-Method': method // Actual method
    }
  }
  
  if (body) {
    options.body = JSON.stringify(body)
  }
  
  const response = await fetch(url, options)
  return response.json()
}

// Get all DNS records
const records = await callGoDaddyAPI('/v1/domains/thesqd.com/records', 'GET')

// Create A record
const newRecord = await callGoDaddyAPI('/v1/domains/thesqd.com/records', 'POST', [
  {
    data: '192.168.1.1',
    name: 'api',
    ttl: 600,
    type: 'A'
  }
])
```

## Response

### Success Response (200)

**Get DNS Records:**
```json
[
  {
    "data": "192.168.1.1",
    "name": "@",
    "ttl": 600,
    "type": "A"
  },
  {
    "data": "ns1.domaincontrol.com",
    "name": "@",
    "ttl": 3600,
    "type": "NS"
  }
]
```

**Create/Update Record (204 No Content):**
```
null
```

### Error Responses

**400 Bad Request** - Missing endpoint parameter
```json
{
  "error": "Missing endpoint parameter"
}
```

**429 Too Many Requests** - Rate limit exceeded
```json
{
  "error": "Rate limit exceeded. Try again later."
}
```

**500 Internal Server Error** - Missing credentials
```json
{
  "error": "Missing API credentials. Make sure to set GODADDY_API_KEY and GODADDY_API_SECRET environment variables."
}
```

**500 Internal Server Error** - General error
```json
{
  "error": "Internal server error",
  "details": "Error message details"
}
```

## Workflow Process

### 1. Handle CORS
- Responds to OPTIONS preflight requests
- Enables cross-origin requests

### 2. Rate Limiting
- Checks client IP address
- Allows maximum 10 requests per minute per IP
- Returns 429 if limit exceeded
- Automatically cleans old request history

### 3. Parse Request
- Extracts `endpoint` query parameter
- Gets actual HTTP method from `X-HTTP-Method` header
- Falls back to request method if header not present

### 4. Validate Credentials
- Checks `GODADDY_API_KEY` environment variable
- Checks `GODADDY_API_SECRET` environment variable
- Returns error if either is missing

### 5. Forward Request
- Constructs full GoDaddy API URL
- Adds authentication header
- Includes request body for POST/PUT/PATCH
- Makes request to GoDaddy API

### 6. Return Response
- Returns GoDaddy API response as-is
- Handles 204 No Content responses
- Includes CORS headers in response

## GoDaddy API Integration

### Domain Used
```
thesqd.com
```

### Base URL
```
https://api.godaddy.com/v1
```

### Authentication
```
Authorization: sso-key {GODADDY_API_KEY}:{GODADDY_API_SECRET}
```

### Common Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/domains/{domain}/records` | GET | List all DNS records |
| `/v1/domains/{domain}/records` | POST | Create DNS records |
| `/v1/domains/{domain}/records/{type}/{name}` | GET | Get specific record |
| `/v1/domains/{domain}/records/{type}/{name}` | PUT | Update specific record |
| `/v1/domains/{domain}/records/{type}/{name}` | DELETE | Delete specific record |
| `/v1/domains/{domain}` | GET | Get domain info |
| `/v1/domains/available` | GET | Check domain availability |

## DNS Record Types

| Type | Description | Example Data |
|------|-------------|--------------|
| A | IPv4 address | `192.168.1.1` |
| AAAA | IPv6 address | `2001:0db8::1` |
| CNAME | Canonical name | `example.com` |
| MX | Mail exchange | `mail.example.com` |
| TXT | Text record | `v=spf1 include:_spf.google.com ~all` |
| NS | Name server | `ns1.example.com` |
| SRV | Service record | `10 5 5060 server.example.com` |

## Rate Limiting

### Limits
- **10 requests per minute** per IP address
- Tracked in-memory (resets on function cold start)
- Rolling window (not fixed interval)

### Rate Limit Logic
```typescript
const RATE_LIMIT = 10 // requests per minute
const requestLog: { [key: string]: number[] } = {}

function checkRateLimit(clientIp: string): boolean {
  const now = Date.now()
  const minuteAgo = now - 60000
  
  // Clean old requests
  requestLog[clientIp] = (requestLog[clientIp] || []).filter(time => time > minuteAgo)
  
  // Check limit
  if (requestLog[clientIp].length < RATE_LIMIT) {
    requestLog[clientIp].push(now)
    return true
  }
  
  return false
}
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GODADDY_API_KEY` | Yes | GoDaddy API key (Production or OTE) |
| `GODADDY_API_SECRET` | Yes | GoDaddy API secret |

## GoDaddy API Setup

### Getting API Credentials
1. Log into GoDaddy Developer Portal: developer.godaddy.com
2. Create API key (Production or OTE environment)
3. Copy Key and Secret
4. Add to Supabase secrets:
   - `GODADDY_API_KEY`
   - `GODADDY_API_SECRET`

### Testing with OTE (Optional)
- OTE = Operational Test Environment
- Use for testing without affecting production
- Separate API keys for OTE vs Production
- Base URL: `https://api.ote-godaddy.com/v1`

## Security Features

✅ **Credential Protection**
- API keys never exposed to client
- Server-side authentication only
- Credentials stored in environment variables

✅ **Rate Limiting**
- 10 requests/minute per IP
- Prevents abuse and API quota exhaustion
- Automatic cleanup of old requests

✅ **JWT Authentication**
- Requires valid Supabase auth token
- Only authenticated users can access
- User identity tracked via IP

✅ **CORS Protection**
- Controlled cross-origin access
- Validates request origins
- Proper preflight handling

## Use Cases

1. **DNS Management:** Update DNS records from web interface
2. **Domain Info:** Check domain status and configuration
3. **Subdomain Creation:** Automatically create subdomains for users
4. **Domain Availability:** Check if domains are available
5. **SSL Certificate Validation:** Manage TXT records for SSL

## Common Operations

### Check Domain Availability
```javascript
const result = await callGoDaddyAPI(
  '/v1/domains/available?domain=example.com',
  'GET'
)
console.log(result.available) // true or false
```

### Get Domain Info
```javascript
const domain = await callGoDaddyAPI(
  '/v1/domains/thesqd.com',
  'GET'
)
console.log(domain.status) // ACTIVE
```

### Create Subdomain
```javascript
await callGoDaddyAPI('/v1/domains/thesqd.com/records', 'POST', [
  {
    type: 'CNAME',
    name: 'app',
    data: 'myapp.vercel.app',
    ttl: 600
  }
])
// Creates app.thesqd.com → myapp.vercel.app
```

## Error Handling

### Common Issues

**Missing Endpoint**
- Always include `endpoint` query parameter
- Start endpoint with `/v1/`
- URL-encode the endpoint value

**Rate Limit Exceeded**
- Wait 1 minute before retrying
- Batch requests when possible
- Consider caching results

**Invalid Credentials**
- Verify API key and secret in Supabase
- Check keys are for correct environment (Production vs OTE)
- Regenerate keys if compromised

**GoDaddy API Errors**
- Check GoDaddy API documentation
- Verify endpoint path is correct
- Ensure request body format matches API spec

## Testing

### Test DNS Record Operations
```bash
# Get all records
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/godaddy-proxy?endpoint=/v1/domains/thesqd.com/records" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "X-HTTP-Method: GET"

# Create TXT record for domain verification
curl -X POST \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/godaddy-proxy?endpoint=/v1/domains/thesqd.com/records" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -H "X-HTTP-Method: POST" \
  -d '[
    {
      "type": "TXT",
      "name": "_verification",
      "data": "verification-code-here",
      "ttl": 600
    }
  ]'
```

## Notes

- Always use POST to the proxy, specify actual method in `X-HTTP-Method` header
- Rate limit is per IP, shared across all users from same IP
- Rate limit resets on function cold start
- GoDaddy API has its own rate limits (varies by plan)
- DNS changes can take time to propagate (TTL dependent)
- Use lower TTL (600) for records that change frequently
- Use higher TTL (3600+) for stable records to reduce lookups
- The function is designed specifically for `thesqd.com` domain
- To support multiple domains, modify the DOMAIN constant

