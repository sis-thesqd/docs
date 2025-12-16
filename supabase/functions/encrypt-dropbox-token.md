---
title: "encrypt-dropbox-token"
author: "Josh Sorenson"
date: "2025-12-16"
last_updated: "2025-12-16"
tags: ["dropbox", "decryption", "token", "security"]
version: "125"
---

# encrypt-dropbox-token

**Note:** Despite the name, this function decrypts Dropbox tokens. Decrypts encrypted Dropbox refresh tokens using AES-GCM encryption.

## Overview
**Function Slug:** encrypt-dropbox-token  
**Status:** Active  
**JWT Verification:** Enabled (requires authentication)  
**Version:** 125

## Purpose
Decrypts encrypted Dropbox refresh tokens that were previously encrypted. Uses AES-GCM encryption with SHA-256 key derivation.

## Endpoint

```
POST https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/encrypt-dropbox-token
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
| `refreshToken` | string | Yes | Encrypted token to decrypt |

### Sample Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/encrypt-dropbox-token \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "base64_encrypted_token_here"
  }'
```

## Response

### Success Response (200)

```json
{
  "decryptedToken": "actual_dropbox_refresh_token",
  "success": true
}
```

### Error Responses

**400 Bad Request** - Missing token
```json
{
  "error": "No encrypted token provided",
  "success": false
}
```

**400 Bad Request** - Decryption failed
```json
{
  "error": "Decryption failed: ...",
  "success": false
}
```

## Encryption Details

### Algorithm
- **Encryption:** AES-GCM
- **Key Size:** 256 bits
- **Key Derivation:** SHA-256
- **IV Size:** 12 bytes

### Decryption Process
1. Derive key from `ENCRYPTION_KEY` using SHA-256
2. Extract IV from first 12 bytes
3. Decrypt remaining data using AES-GCM
4. Return decrypted token

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ENCRYPTION_KEY` | Yes | Encryption key for decryption |

## Notes

- Function name is misleading (decrypts, not encrypts)
- Uses Web Crypto API for decryption
- Base64 encoded input expected
- IV extracted from ciphertext
- Validates non-empty decrypted result
- Part of Dropbox token management system
- Used with `decrypt-dropbox-token` function
- See [decrypt-dropbox-token](./decrypt-dropbox-token.md) for encryption

