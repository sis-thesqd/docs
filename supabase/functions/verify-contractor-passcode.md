# verify-contractor-passcode

Authenticates contractors using a passcode system and creates a Supabase auth session.

## Overview
**Function Slug:** verify-contractor-passcode  
**Status:** Active  
**JWT Verification:** Disabled (public endpoint)  
**Version:** 15

## Purpose
Provides authentication for contractor users via a simple passcode system. Validates contractor status, verifies passcode, and creates an authenticated session.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/verify-contractor-passcode
```

## Request

### Headers
```
Content-Type: application/json
```

### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Contractor's email address |
| `passcode` | string | Yes | Contractor's personal passcode |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/verify-contractor-passcode \
  -H "Content-Type: application/json" \
  -d '{
    "email": "contractor@example.com",
    "passcode": "mySecurePass123"
  }'
```

### JavaScript Example

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/verify-contractor-passcode', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'contractor@example.com',
    passcode: 'mySecurePass123'
  })
})

const data = await response.json()

if (data.access_token) {
  // Store tokens
  localStorage.setItem('access_token', data.access_token)
  localStorage.setItem('refresh_token', data.refresh_token)
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
    "email": "contractor@example.com",
    "name": "Jane Smith",
    "airtable_rec_id": "rec123456789"
  }
}
```

### Error Responses

**400 Bad Request** - Missing parameters
```json
{
  "error": "Email and passcode are required"
}
```

**401 Unauthorized** - Invalid passcode
```json
{
  "error": "Invalid passcode"
}
```

**404 Not Found** - Contractor not found
```json
{
  "error": "No contractor account found with this email"
}
```

**404 Not Found** - No passcode configured
```json
{
  "error": "No passcode configured. Contact the VP of Strategy for access."
}
```

**429 Too Many Requests** - Rate limit exceeded
```json
{
  "error": "Too many login attempts. Please try again in 15 minutes."
}
```

**500 Internal Server Error** - Session creation failed
```json
{
  "error": "Failed to create user account"
}
```

## Workflow Process

1. **Rate Limiting** - Max 5 attempts per 15 minutes per email
2. **Lookup Contractor** - Finds employee with status "Contractor"
3. **Get Passcode** - Retrieves passcode from `strategy_contractor_passcodes`
4. **Verify Passcode** - Simple string comparison
5. **Get/Create Auth User** - Checks for existing user, creates if needed
6. **Generate Session** - Creates temporary password and signs in
7. **Return Tokens** - Provides access and refresh tokens

## Database Tables

### Table: employees
```sql
Columns used:
- id (uuid)
- email (text)
- name (text)
- airtable_rec_id (text)
- status (text) -- Must be 'Contractor'
```

### Table: strategy_contractor_passcodes
```sql
CREATE TABLE strategy_contractor_passcodes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  employee_id UUID NOT NULL REFERENCES employees(id),
  passcode TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(employee_id)
);
```

### Table: auth_codes
Used for rate limiting:
```sql
Columns used:
- email (text)
- code (text) -- Set to 'contractor_attempt' for tracking
- created_at (timestamp)
- expires_at (timestamp)
- used (boolean) -- Always true for rate limit entries
```

## Security Features

### Rate Limiting
- 5 attempts per 15 minutes per email
- Uses `auth_codes` table for tracking
- Prevents brute force attacks

### Passcode Storage
- Simple string comparison (consider hashing for production)
- One passcode per contractor
- Managed by VP of Strategy

### Session Security
- Temporary passwords generated with crypto.randomUUID()
- Password immediately replaced after use
- Uses Supabase built-in JWT authentication

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key |

## Use Cases

1. **Contractor Login:** Simple authentication for external contractors
2. **Limited Access:** Contractors without company email/Slack
3. **Strategy Team:** Specific to strategy contractor workflows
4. **Quick Setup:** No complex OAuth or email verification needed
5. **Managed Access:** VP of Strategy controls passcodes

## Related Functions

- [send-slack-code](./send-slack-code.md) - Employee Slack authentication
- [verify-slack-code](./verify-slack-code.md) - Verify Slack codes
- [send-email-code](./send-email-code.md) - Employee email authentication

## Setting Up Contractor Access

### For Administrators (VP of Strategy)

1. **Add Contractor to employees table:**
```sql
INSERT INTO employees (email, name, status, airtable_rec_id)
VALUES ('contractor@example.com', 'Jane Smith', 'Contractor', 'rec123abc');
```

2. **Create passcode:**
```sql
INSERT INTO strategy_contractor_passcodes (employee_id, passcode)
VALUES (
  (SELECT id FROM employees WHERE email = 'contractor@example.com'),
  'securePassword123'
);
```

3. **Share passcode securely** with the contractor

### For Contractors

Simply log in with email and provided passcode - no registration needed.

## Notes

- Status must be exactly "Contractor" in employees table
- Passcodes are plain text (consider bcrypt for production)
- Function auto-creates auth user on first login
- Rate limiting is per email address
- Expired rate limit entries remain in auth_codes table
- Consider adding passcode expiration/rotation for security
- VP of Strategy has exclusive access to passcode management

