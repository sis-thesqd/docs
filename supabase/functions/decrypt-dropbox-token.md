# decrypt-dropbox-token

Decrypts Dropbox refresh tokens using AES-GCM encryption.

## Function Details

- **Slug**: `decrypt-dropbox-token`
- **Status**: ACTIVE
- **JWT Verification**: Enabled (requires authentication)
- **Version**: 150

## Purpose

This function securely decrypts Dropbox refresh tokens that were previously encrypted using the companion `encrypt-dropbox-token` function. It uses AES-GCM (Advanced Encryption Standard - Galois/Counter Mode) for secure decryption.

## Endpoint

```
POST /functions/v1/decrypt-dropbox-token
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
  "refreshToken": "encrypted_token_string"
}
```

### Example Request

```bash
curl -X POST \
  https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/decrypt-dropbox-token \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "base64_encrypted_token"
  }'
```

## Response

### Success Response (200)

```json
{
  "decryptedToken": "your_decrypted_refresh_token",
  "success": true
}
```

### Error Responses

**400 Bad Request** - Invalid input or decryption failure

```json
{
  "error": "No encrypted token provided",
  "success": false
}
```

```json
{
  "error": "Decryption failed: {error_details}",
  "success": false
}
```

**500 Internal Server Error** - Missing encryption key

```json
{
  "error": "Encryption key not configured",
  "success": false
}
```

## Implementation Details

### Encryption Algorithm

- **Algorithm**: AES-GCM (256-bit)
- **IV Length**: 12 bytes
- **Key Derivation**: SHA-256 hash of the encryption key

### Decryption Process

1. Derives a CryptoKey from the `ENCRYPTION_KEY` environment variable using SHA-256
2. Decodes the base64-encoded encrypted token
3. Extracts the 12-byte IV from the beginning of the data
4. Decrypts the remaining data using AES-GCM
5. Returns the decrypted token as a UTF-8 string

### Code Flow

```typescript
// 1. Derive key from password
const keyMaterial = await crypto.subtle.digest('SHA-256', encoder.encode(password))
const key = await crypto.subtle.importKey('raw', keyMaterial, { name: 'AES-GCM', length: 256 }, false, ['decrypt'])

// 2. Extract IV and encrypted data
const combined = new Uint8Array(atob(ciphertext).split('').map(char => char.charCodeAt(0)))
const iv = combined.slice(0, 12)
const encryptedData = combined.slice(12)

// 3. Decrypt
const decryptedData = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, key, encryptedData)
const decrypted = new TextDecoder().decode(decryptedData)
```

### Dependencies

- `deno.land/std@0.168.0/http/server.ts` - HTTP server
- Web Crypto API (built-in)

### Environment Variables

- `ENCRYPTION_KEY` - Secret key used for encryption/decryption (required)

## Security Considerations

✅ **Security Features**:
- Uses AES-GCM, a secure authenticated encryption mode
- Requires JWT authentication
- Key derivation using SHA-256
- CORS headers properly configured

⚠️ **Best Practices**:
1. Store `ENCRYPTION_KEY` securely in Supabase secrets
2. Use strong, random encryption keys (minimum 32 bytes)
3. Rotate encryption keys periodically
4. Never log decrypted tokens
5. Use HTTPS for all requests

## CORS Configuration

The function includes CORS headers for cross-origin requests:

```typescript
{
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type'
}
```

## Use Cases

- Decrypting Dropbox refresh tokens for API calls
- Securely retrieving stored OAuth credentials
- Token management in authentication flows

## Related Functions

- [encrypt-dropbox-token](./encrypt-dropbox-token.md) - Encrypt Dropbox tokens
- [encryptToken](./encryptToken.md) - General purpose token encryption
- [get_decrypted_secret](./get_decrypted_secret.md) - Retrieve secrets from vault

## Error Handling

The function includes comprehensive error handling:

- Empty string validation
- Base64 decoding errors
- Decryption failures
- Missing configuration

All errors are caught and returned with descriptive messages.

## Testing

To test the function:

```javascript
const response = await fetch('https://wttgwoxlezqoyzmesekt.supabase.co/functions/v1/decrypt-dropbox-token', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${supabaseToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    refreshToken: 'your_encrypted_token'
  })
})

const data = await response.json()
console.log(data.decryptedToken)
```

