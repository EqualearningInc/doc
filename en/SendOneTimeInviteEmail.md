# Send One-Time Invite Email API Documentation

## 1. API Path

**Request Method**: `POST`  
**Endpoint**: `/api/invite/sendOneTimeInviteEmail`

## 2. Request Data Format

```json
{
  "email": "student@example.com",
  "userDisplayName": "John Doe",
  "schoolId": "00000000-0000-0000-0000-000000000000",
  "classId": "11111111-1111-1111-1111-111111111111"
}
```

## 3. Field Description

| Field Name | Type | Required | Description |
| ---------- | ---- | -------- | ----------- |
| email | string | Yes | Recipient email address (e.g., example@qq.com) |
| userDisplayName | string | No | Display name of the user to be invited |
| schoolId | Guid | Yes | School ID (UUID format) |
| classId | Guid | Yes | Class ID (UUID format) |

## 4. Authentication

### Request Headers

When a school has configured an API Key, the following headers are required for signature authentication:

* **`e-timestamp`**: Current Unix timestamp (in seconds)
* **`e-signature`**: Signature generated using the following rules:

### Signature Generation Rules

1. Construct the signature data string in the following format (all lowercase):
   ```
   method=post&path=/api/invite/sendonetimeinviteemail&timestamp={timestamp}&apikey={apiKey}
   ```
   
   Where:
   - `method`: HTTP method in lowercase (e.g., "post")
   - `path`: API path in lowercase (e.g., "/api/invite/sendonetimeinviteemail")
   - `timestamp`: Unix timestamp in seconds (must match the value in `e-timestamp` header)
   - `apikey`: The API key configured for the school

2. Use `HMAC-SHA256` algorithm to encrypt the signature data string with the `apiKey` as the key

3. Convert the result to hexadecimal string

### JavaScript Example Code

```javascript
// Define API Key
const apiKey = "sk-863b8cb0c7b24dc49299a3a7a7f8e57b";

// Generate Unix timestamp (in seconds)
const unixTimestamp = Math.floor(Date.now() / 1000);

// Set request header
xhr.setRequestHeader("e-timestamp", unixTimestamp);

// Construct signature data (all lowercase)
const requestData = `method=post&path=/api/invite/sendonetimeinviteemail&timestamp=${unixTimestamp}&apikey=${apiKey}`.toLowerCase();

// Calculate HMAC-SHA256 signature
const signature = CryptoJS.HmacSHA256(
  CryptoJS.enc.Utf8.parse(requestData),
  CryptoJS.enc.Utf8.parse(apiKey)
).toString(CryptoJS.enc.Hex);

// Add signature request header
xhr.setRequestHeader("e-signature", signature);
```

### Authentication Notes

- **Timestamp Validation**: The timestamp must be within 10 minutes (600 seconds) of the server's current time. Requests with expired timestamps will be rejected.
- **Optional Authentication**: If the school has not configured an API Key, signature authentication is not required.
- **Header Priority**: The system will check for `e-timestamp` and `e-signature` in request headers first, then fall back to query string parameters if not found in headers.

## 5. Response Format

### Success Response (200 OK)

```json
{
  "id": "22222222-2222-2222-2222-222222222222",
  "inviteCode": "ABC12345",
  "schoolId": "00000000-0000-0000-0000-000000000000",
  "classId": "11111111-1111-1111-1111-111111111111",
  "inviteLink": "https://example.com/Account/SignUp?code=ABC12345",
  "inviteType": 2,
  "count": 1,
  "dateAdded": "2024-01-15T10:30:00Z"
}
```

### Response Field Description

| Field Name | Type | Description |
| ---------- | ---- | ----------- |
| id | Guid | Invite record ID |
| inviteCode | string | Generated unique invite code (8 uppercase alphanumeric characters) |
| schoolId | Guid | School ID |
| classId | Guid | Class ID |
| inviteLink | string | Complete invite link URL |
| inviteType | int | Invite type (2 = Class) |
| count | int | Usage count (-1 = unlimited, 0 = unavailable, >0 = remaining uses). For this API, it's always 1 (single use) |
| dateAdded | DateTime | Creation timestamp |

### Error Response (400/500)

```json
{
  "message": "Error message description"
}
```

## 6. Error Scenarios

- **400 Bad Request**: Invalid request data (missing required fields, invalid GUID format, etc.)
- **401 Unauthorized**: Invalid or missing signature when API Key is configured
- **500 Internal Server Error**: Server-side errors (unable to generate unique code, email sending failure, etc.)

## 7. Example cURL Request

```bash
curl -X POST "https://api.example.com/api/invite/sendOneTimeInviteEmail" \
  -H "Content-Type: application/json" \
  -H "e-timestamp: 1705312200" \
  -H "e-signature: abc123def456..." \
  -d '{
    "email": "student@example.com",
    "userDisplayName": "John Doe",
    "schoolId": "00000000-0000-0000-0000-000000000000",
    "classId": "11111111-1111-1111-1111-111111111111"
  }'
```
