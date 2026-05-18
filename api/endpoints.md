# API Reference

Base URL: `http://localhost:3001/api/v1`  
Production: `https://api.scoutshive.io/api/v1`

All responses are JSON. Errors follow `{ "error": "<message>", "code": <number> }`.

---

## Profiles

### `GET /profiles`

List player profiles. Optionally filter by sport.

**Query params**

| Param | Type | Required | Description |
|---|---|---|---|
| `sport` | string | no | Filter by sport (e.g. `football`, `basketball`) |
| `page` | number | no | Page number, default `1` |
| `limit` | number | no | Results per page, default `20`, max `100` |

**Response `200`**
```json
{
  "data": [
    {
      "player_address": "G...",
      "sport": "football",
      "position": "striker",
      "age": 22,
      "video_ipfs_hash": "Qm..."
    }
  ],
  "page": 1,
  "total": 142
}
```

---

### `GET /profiles/:address`

Get a single player profile by Stellar address.

**Response `200`**
```json
{
  "player_address": "G...",
  "sport": "football",
  "position": "striker",
  "age": 22,
  "video_ipfs_hash": "Qm..."
}
```

**Response `404`**
```json
{ "error": "Profile not found", "code": 3 }
```

---

### `POST /upload`

Upload a video file to IPFS. Returns the IPFS hash to pass to `register_profile`.

**Request** — `multipart/form-data`

| Field | Type | Required |
|---|---|---|
| `video` | file | yes |

**Response `200`**
```json
{ "ipfs_hash": "QmXoypizjW3WknFiJnKLwHCnL72vedxjQkDDP1mXWo6uco" }
```

---

## Scouts

### `POST /scouts/verify`

Submit KYC documents for scout verification. Admin reviews and issues Scout Pass on-chain.

**Request body**
```json
{
  "scout_address": "G...",
  "full_name": "Jane Smith",
  "organisation": "FC Example",
  "kyc_document_url": "https://..."
}
```

**Response `202`**
```json
{ "status": "pending_review", "scout_address": "G..." }
```

---

### `GET /scouts/:address/contacts`

List a scout's contact history (requires scout wallet signature via `Authorization` header).

**Response `200`**
```json
{
  "data": [
    {
      "contact_record_id": "abc123",
      "player_address": "G...",
      "initiated_at": "2026-05-18T20:00:00Z"
    }
  ]
}
```

---

## Admin

All admin endpoints require `Authorization: Bearer <admin_jwt>`.

### `POST /admin/scouts/:address/pass`

Issue a Scout Pass SBT to a verified scout.

**Response `200`**
```json
{ "scout_address": "G...", "status": "verified", "tx_hash": "..." }
```

---

### `DELETE /admin/scouts/:address/pass`

Revoke a scout's pass.

**Response `200`**
```json
{ "scout_address": "G...", "status": "revoked", "tx_hash": "..." }
```

---

## Health

### `GET /health`

```json
{ "status": "ok", "network": "testnet", "db": "connected" }
```

---

## Error Codes

| Code | Meaning |
|---|---|
| 3 | Profile not found |
| 5 | Scout Pass required |
| 9 | Unauthorized |
| 10 | Invalid IPFS hash |
| 12 | Insufficient fee |
| 500 | Internal server error |
