---
title: "get_decrypted_secret"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["security", "vault", "secrets", "decryption"]
version: "130"
---

# get_decrypted_secret

Retrieves decrypted secrets from the Supabase vault.

## Function Details

- **Slug**: `get_decrypted_secret`
- **Status**: ACTIVE
- **JWT Verification**: Disabled (public access)
- **Version**: 130

## Purpose

This function provides secure access to decrypted secrets stored in the `vault.decrypted_secrets` table. It queries the vault using a secret ID and returns the decrypted value.

## Endpoint

```
GET /functions/v1/get_decrypted_secret?id={secret_id}
```

## Request

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | The ID of the secret to retrieve |

### Example Request

```bash
curl "https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/get_decrypted_secret?id=your-secret-id"
```

## Response

### Success Response (200)

```json
{
  "decrypted_secret": "your-secret-value"
}
```

### Error Responses

**400 Bad Request** - Missing ID parameter
```json
"ID parameter is required"
```

**500 Internal Server Error** - Database error
```json
"Error fetching data: {error_message}"
```

## Implementation Details

### Database Query

The function queries the `vault.decrypted_secrets` table:

```sql
SELECT decrypted_secret 
FROM vault.decrypted_secrets 
WHERE id = {id}
LIMIT 1
```

### Dependencies

- `@supabase/supabase-js@1.35.6` - Supabase client library
- `deno.land/x/sift@0.6.0` - HTTP server framework

### Environment Variables

- `SUPABASE_URL` - Supabase project URL
- `SUPABASE_ANON_KEY` - Supabase anonymous key

## Security Considerations

⚠️ **Warning**: This function does not require JWT verification (`verify_jwt: false`). Access is controlled only by knowledge of the secret ID. Ensure:

1. Secret IDs are not exposed in client-side code
2. IDs are sufficiently random/unpredictable
3. Consider implementing additional authentication if needed
4. Monitor access logs for suspicious activity

## Use Cases

- Retrieving API keys stored securely in the vault
- Accessing encrypted credentials for third-party services
- Fetching configuration secrets at runtime

## Related Functions

- [encryptToken](./encryptToken.md) - Encrypt tokens before storage
- [decrypt-dropbox-token](./decrypt-dropbox-token.md) - Decrypt Dropbox tokens
- [encrypt-dropbox-token](./encrypt-dropbox-token.md) - Encrypt Dropbox tokens

## Notes

- The function uses an older version of the Supabase client (`@1.35.6`)
- Consider updating to the latest version for improved security and features
- The vault table must have appropriate RLS policies configured

