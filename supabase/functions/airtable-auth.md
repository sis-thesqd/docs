---
title: "airtable-auth"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["authentication", "airtable", "salary", "executive"]
version: "123"
---

# airtable-auth

Authenticates users against Airtable employee directory and provides salary data access for authorized executives.

## Overview
**Function Slug:** airtable-auth  
**Status:** Active  
**JWT Verification:** Disabled (uses Airtable-based authentication)  
**Version:** 123

## Purpose
Two-step authentication system: verifies user exists in employee directory, checks department privileges (C-Suite or Exec Squad only), and optionally fetches salary data if authorized.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/airtable-auth
```

## Request

### Headers
```
Content-Type: application/json
```

### Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Employee email address |
| `action` | string | Yes | 'authenticate' or 'getSalaryData' |

### Sample Requests

#### Authenticate User
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/airtable-auth \
  -H "Content-Type: application/json" \
  -d '{
    "email": "executive@example.com",
    "action": "authenticate"
  }'
```

#### Get Salary Data
```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/airtable-auth \
  -H "Content-Type: application/json" \
  -d '{
    "email": "executive@example.com",
    "action": "getSalaryData"
  }'
```

### JavaScript Example

```javascript
// Step 1: Authenticate
const authResponse = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/airtable-auth', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'executive@example.com',
    action: 'authenticate'
  })
})

const authData = await authResponse.json()

if (authData.success) {
  console.log(`Welcome ${authData.user.name}`)
  
  // Step 2: Fetch salary data
  const salaryResponse = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/airtable-auth', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      email: 'executive@example.com',
      action: 'getSalaryData'
    })
  })
  
  const salaryData = await salaryResponse.json()
  console.log(salaryData.data)
}
```

## Response

### authenticate Success (200)

```json
{
  "success": true,
  "user": {
    "name": "John Executive",
    "email": "executive@example.com",
    "department": "C-Suite"
  }
}
```

### getSalaryData Success (200)

```json
{
  "success": true,
  "data": {
    "employeeName": "John Executive",
    "email": "executive@example.com",
    "baseSalary": 150000,
    "performanceMultipliers": [
      {
        "category": "Performance",
        "name": "Exceeds Expectations",
        "description": "Consistently delivers exceptional results",
        "multiplier": 1.15
      }
    ],
    "locationMultipliers": [
      {
        "country": "US",
        "countryName": "United States",
        "costOfLivingIndex": 100,
        "classification": "Tier 1",
        "multiplier": 1.0,
        "rank": 1
      }
    ]
  }
}
```

### Error Responses

**401 Unauthorized** - Employee not found
```json
{
  "success": false,
  "error": "Employee not found"
}
```

**403 Forbidden** - Insufficient privileges
```json
{
  "success": false,
  "error": "Access denied. Insufficient privileges."
}
```

**500 Internal Server Error**
```json
{
  "success": false,
  "error": "Missing Airtable configuration"
}
```

## Access Control

### Required Departments
Only these departments can authenticate:
- **C-Suite** - Full access
- **Exec Squad** - Full access

All other departments are denied access.

### Authentication Flow
1. **Lookup Employee** - Search by email in employee directory
2. **Check Department** - Verify user is C-Suite or Exec Squad
3. **Grant/Deny Access** - Allow if authorized, deny otherwise

## Airtable Configuration

### Bases
- Employee Base ID: From `AIRTABLE_EMPLOYEE_BASE_ID`
- Salary Base ID: From `AIRTABLE_SALARY_BASE_ID`

### Tables & Fields

**Employee Directory (Base 2)**
- Table: `tbluMUf3ijGVOCdIK`
- Email Field: `fldvTPj396J0PBmKM`
- Name Field: `fldoUOr5fiJO2xaIF`
- Department Field: `fldtJKrZg4JTWD8AQ`

**Salary Info (Base 1)**
- Table: `tblnBeOB2YqmWqui2`
- Employee Name: `fldfnEGFyi7cOcB6s`
- Email: `fldAaZmBrxhAkUri5`
- Base Salary: `fldZxqmWYYgBUS9pN`

**Performance Matrix (Base 1)**
- Table: `tbldqJriilrcDIGyY`
- Category: `fldSr3Zkz90pFTH2S`
- Name: `fldMfpwHEFfZmiOrR`
- Description: `fldszKwFSkSfbpgsG`
- Multiplier: `fldsf46SxCuNfZY62`

**Location Multipliers (Base 1)**
- Table: `tblukcywY2AWtS49a`
- Country: `fldNl4iDVAZorg23t`
- Country Name: `fldNcvFFNH2HqpzNB`
- Cost Index: `fld3ozMSSlz35LuDP`
- Classification: `fldAAhzUj963rM4xx`
- Multiplier: `fldxhFw37LdSJUnU6`
- Rank: `fld7TtTw0hYRpZvRP`

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `AIRTABLE_API_KEY` | Yes | Airtable Personal Access Token |
| `AIRTABLE_EMPLOYEE_BASE_ID` | Yes | Base ID for employee directory |
| `AIRTABLE_SALARY_BASE_ID` | Yes | Base ID for salary data |

## Use Cases

1. **Executive Dashboard:** Authenticate executives for salary tools
2. **Salary Management:** Provide data for salary calculators
3. **Access Control:** Ensure only authorized users access sensitive data
4. **Performance Reviews:** Load performance multiplier matrices
5. **Cost Adjustments:** Access location-based salary multipliers

## Data Fetching

### Parallel Requests
`getSalaryData` uses `Promise.all` to fetch three datasets simultaneously:
- Salary information
- Performance multipliers
- Location multipliers

This reduces total request time by ~66%.

### Email Matching
Case-insensitive email comparison using Airtable formula:
```
LOWER({email_field})=LOWER("user@example.com")
```

## Security Considerations

⚠️ **No JWT Authentication**
- Function is public but enforces Airtable-based auth
- All requests require valid employee email
- Department check prevents unauthorized access

✅ **Security Features**
- Email verified against employee directory
- Department privileges checked server-side
- API keys never exposed to client
- Case-insensitive email matching prevents bypass
- Detailed logging for access attempts

## Testing

### Test Authentication
```bash
# Should succeed for C-Suite/Exec Squad
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/airtable-auth \
  -H "Content-Type: application/json" \
  -d '{
    "email": "ceo@example.com",
    "action": "authenticate"
  }'

# Should fail for other departments
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/airtable-auth \
  -H "Content-Type: application/json" \
  -d '{
    "email": "designer@example.com",
    "action": "authenticate"
  }'
```

## Notes

- Two separate Airtable bases (employee vs salary data)
- Field IDs are hardcoded (not field names)
- Email lookup uses formula filter for case-insensitivity
- Only C-Suite and Exec Squad have access
- Access denials are logged with department info
- Salary data includes user's own salary if available
- Performance and location data is global (not user-specific)
- Missing salary data defaults to 0
- All three data types fetched in parallel for performance

