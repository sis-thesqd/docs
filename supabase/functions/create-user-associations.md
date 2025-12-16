---
title: "create-user-associations"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["users", "associations", "clickup", "accounts", "folders"]
version: "91"
---

# create-user-associations

Creates associations between ClickUp users and account folders/lists for access control and permissions.

## Overview
**Function Slug:** create-user-associations  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 91

## Purpose
Links a ClickUp user to an account's folder and all associated lists. Creates entries in `user_associations` table to grant user access to specific ClickUp objects. Prevents duplicate associations.

## Endpoint

```
GET https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/create-user-associations?email={email}&account={account_number}
```

## Request

### Headers
```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User's email address |
| `account` | number | Yes | Account number (e.g., 3570) |

### Sample Request

```bash
curl -X GET \
  "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/create-user-associations?email=user@example.com&account=3570" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### JavaScript Example

```javascript
const email = 'user@example.com'
const account = 3570

const response = await fetch(`https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/create-user-associations?email=${email}&account=${account}`, {
  headers: {
    'Authorization': `Bearer ${supabaseToken}`
  }
})

const data = await response.json()
console.log(`Created ${data.data.total_associations} associations`)
console.log(`Folders: ${data.data.folders_added}, Lists: ${data.data.lists_added}`)
```

## Response

### Success Response (200)

```json
{
  "success": true,
  "message": "User associations created successfully",
  "data": {
    "user_id": "123456",
    "email": "user@example.com",
    "account": 3570,
    "folders_added": 1,
    "lists_added": 5,
    "total_associations": 6,
    "existing_associations": 0
  }
}
```

### Error Responses

**400 Bad Request** - Missing parameters
```json
{
  "error": "Missing required parameters",
  "message": "Both email and account parameters are required",
  "usage": "?email=user@example.com&account=3570"
}
```

**400 Bad Request** - Invalid account
```json
{
  "error": "Invalid account parameter",
  "message": "Account must be a valid number"
}
```

**404 Not Found** - User not found
```json
{
  "error": "User not found",
  "message": "No user found with email: user@example.com"
}
```

**404 Not Found** - Account not found
```json
{
  "error": "Account not found",
  "message": "No account found with ID: 3570"
}
```

**500 Internal Server Error** - Database error
```json
{
  "error": "Failed to create associations",
  "message": "duplicate key value violates unique constraint"
}
```

## Workflow Process

### 1. Validate Parameters
- Checks email and account provided
- Validates account is numeric
- Returns helpful error messages

### 2. Lookup User
- Queries `clickup_users` table by email
- Verifies user exists
- Gets ClickUp user ID

### 3. Lookup Account
- Queries `accounts` table by account number
- Verifies account exists
- Gets folder_id if available

### 4. Get Account Lists
- Queries `clickup_lists` table
- Filters by account number
- Gets all list IDs

### 5. Check Existing Associations
- Queries `user_associations` table
- Finds already-linked objects
- Prevents duplicates

### 6. Create Associations
- Inserts folder association (if exists)
- Inserts list associations (if new)
- Uses UUID for row_id
- Tracks counts

## Database Tables

### Table: clickup_users
```sql
Columns used:
- clickup_id (primary key)
- email (indexed)
```

### Table: accounts
```sql
Columns used:
- account (primary key)
- folder_id (ClickUp folder ID)
```

### Table: clickup_lists
```sql
Columns used:
- id (ClickUp list ID)
- account (foreign key)
```

### Table: user_associations
```sql
CREATE TABLE user_associations (
  row_id UUID PRIMARY KEY,
  type TEXT NOT NULL,              -- 'folder' or 'list'
  user_id TEXT NOT NULL,           -- ClickUp user ID
  object_id TEXT NOT NULL,         -- Folder or list ID
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Association Types

### Folder Association
```typescript
{
  row_id: uuid,
  type: 'folder',
  user_id: clickup_user_id,
  object_id: folder_id,
  created_at: timestamp
}
```

### List Association
```typescript
{
  row_id: uuid,
  type: 'list',
  user_id: clickup_user_id,
  object_id: list_id,
  created_at: timestamp
}
```

## Duplicate Prevention

### Existing Check
```typescript
// Get existing associations for user
const existing = await supabase
  .from('user_associations')
  .select('object_id')
  .eq('user_id', user.clickup_id)
  .in('object_id', [folder_id, ...list_ids])

// Filter out existing
const existingIds = new Set(existing.map(e => e.object_id))
const newAssociations = associations.filter(a => !existingIds.has(a.object_id))
```

## Use Cases

1. **Access Control:** Grant users access to account folders/lists
2. **Onboarding:** Set up user permissions for new accounts
3. **Bulk Assignment:** Link multiple users to accounts
4. **Permission Management:** Control ClickUp object access
5. **Account Setup:** Initialize user associations for accounts

## Response Details

### Counts Returned
- **folders_added:** Number of folder associations created
- **lists_added:** Number of list associations created
- **total_associations:** Total new associations
- **existing_associations:** Already-existing associations

### User Information
- **user_id:** ClickUp user ID
- **email:** User email address
- **account:** Account number

## Error Handling

### Validation Errors
- Missing parameters: 400 with usage example
- Invalid account: 400 with clear message
- Helpful error messages guide usage

### Lookup Errors
- User not found: 404 with email
- Account not found: 404 with account number
- Lists fetch error: 500 with details

### Database Errors
- Insert failures logged
- Returns error with message
- Partial success not supported

## Notes

- Only creates new associations (skips existing)
- Folder association optional (if account has folder_id)
- List associations created for all account lists
- UUID generated for each association row_id
- Created_at timestamp set to current time
- Returns detailed counts for verification
- GET method used (no request body)
- Query parameters must be URL-encoded
- Account number must be numeric
- Email lookup is case-sensitive
- Associations are permanent (no expiration)
- Consider adding delete endpoint for removal
- Function idempotent (safe to call multiple times)

