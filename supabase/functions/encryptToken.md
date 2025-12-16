# encryptToken

Encrypts access tokens using AES-GCM encryption for secure storage.

## Function Details

- **Slug**: `encryptToken`
- **Status**: ACTIVE
- **JWT Verification**: Enabled (requires authentication)
- **Version**: 121

## Purpose

This function provides secure encryption for access tokens using AES-GCM (Advanced Encryption Standard - Galois/Counter Mode). It's designed to encrypt OAuth access tokens before storing them in a database or transmitting them securely.

## Endpoint

```
POST /functions/v1/encryptToken
```

## Request

### Headers

```
Authorization: Bearer YOUR_SUPABASE_JWT_TOKEN
Content-Type: application/json
```

### Body

```json
{
  "access_token": "your_plaintext_token"
}
```

### Example Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/encryptToken \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "access_token": "your_oauth_access_token"
  }'
```

## Response

### Success Response (200)

```json
{
  "encryptedToken": "base64_encrypted_token_string",
  "success": true
}
```

### Error Responses

**400 Bad Request** - Invalid or missing token

```json
{
  "error": "Invalid or missing access_token",
  "success": false
}
```

```json
{
  "error": "Cannot encrypt empty string",
  "success": false
}
```

**500 Internal Server Error** - Configuration error

```json
{
  "error": "Encryption key not configured",
  "success": false
}
```

## Implementation Details

### Encryption Algorithm

- **Algorithm**: AES-GCM (256-bit)
- **IV Length**: 12 bytes (randomly generated for each encryption)
- **Key Derivation**: SHA-256 hash of the encryption key
- **Output Format**: Base64-encoded string containing IV + encrypted data

### Encryption Process

1. Validates that the input token is a non-empty string
2. Derives a CryptoKey from the `ENCRYPTION_KEY` environment variable using SHA-256
3. Generates a random 12-byte initialization vector (IV)
4. Encrypts the token using AES-GCM
5. Combines the IV and encrypted data
6. Encodes the result as base64 for safe transmission

### Code Flow

```typescript
// 1. Derive encryption key
const keyMaterial = await crypto.subtle.digest('SHA-256', encoder.encode(ENCRYPTION_KEY))
const key = await crypto.subtle.importKey('raw', keyMaterial, { name: 'AES-GCM', length: 256 }, false, ['encrypt'])

// 2. Generate random IV
const iv = crypto.getRandomValues(new Uint8Array(12))

// 3. Encrypt the token
const encryptedData = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, key, encodedText)

// 4. Combine IV + encrypted data
const combined = new Uint8Array(iv.length + encryptedData.byteLength)
combined.set(iv)
combined.set(new Uint8Array(encryptedData), iv.length)

// 5. Encode as base64
return btoa(String.fromCharCode(...combined))
```

### Dependencies

- `deno.land/std@0.168.0/http/server.ts` - HTTP server
- Web Crypto API (built-in)

### Environment Variables

- `ENCRYPTION_KEY` - Secret key used for encryption (required)

## Security Considerations

✅ **Security Features**:
- Uses AES-GCM, providing both confidentiality and authenticity
- Random IV generated for each encryption (prevents pattern analysis)
- Requires JWT authentication
- Key derivation using SHA-256
- Proper error handling without leaking sensitive information

⚠️ **Best Practices**:
1. Store `ENCRYPTION_KEY` securely in Supabase secrets
2. Use a strong, random encryption key (minimum 32 bytes)
3. Rotate encryption keys periodically
4. Never log plaintext tokens
5. Use HTTPS for all requests
6. Store encrypted tokens securely in the database

## CORS Configuration

The function includes CORS headers for cross-origin requests:

```typescript
{
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type'
}
```

## Use Cases

- Encrypting OAuth access tokens before database storage
- Securing API keys for third-party services
- Protecting sensitive credentials in transit
- Token management in authentication flows

## Related Functions

- [decrypt-dropbox-token](./decrypt-dropbox-token.md) - Decrypt Dropbox tokens
- [encrypt-dropbox-token](./encrypt-dropbox-token.md) - Encrypt Dropbox tokens
- [get_decrypted_secret](./get_decrypted_secret.md) - Retrieve secrets from vault

## Error Handling

The function includes comprehensive validation:

- Type checking for string input
- Empty string validation
- Missing encryption key detection
- Graceful error responses

All errors are caught and returned with descriptive messages without exposing sensitive details.

## Testing

To test the function:

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/encryptToken', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    access_token: 'your_plaintext_token'
  })
})

const data = await response.json()
console.log('Encrypted token:', data.encryptedToken)
```

## Decryption

To decrypt tokens encrypted with this function, use the [decrypt-dropbox-token](./decrypt-dropbox-token.md) function with the same `ENCRYPTION_KEY`.

## Notes

- Each encryption generates a unique IV, so the same token will produce different encrypted outputs
- The encrypted output includes both the IV and the encrypted data
- The function uses TypeScript with comprehensive JSDoc comments
- Error messages are user-friendly while maintaining security

