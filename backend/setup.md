# Backend Setup

Repo: `Scouts-Hive-Backend` — REST API, Stellar event listener, IPFS integration, KYC service.

## Prerequisites

- Node.js 20+
- PostgreSQL 15+
- A [Pinata](https://pinata.cloud) account for IPFS pinning

## 1. Install Dependencies

```bash
cd Scouts-Hive-Backend
npm install
```

## 2. Configure Environment

```bash
cp .env.example .env
```

Edit `.env`:

```env
PORT=3001
DATABASE_URL=postgresql://user:pass@localhost:5432/scouts_hive

STELLAR_NETWORK=testnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org

CONTRACT_REGISTRY_ID=<from contracts/deployment.md>
CONTRACT_SCOUT_ID=<from contracts/deployment.md>
CONTRACT_CONTACT_ID=<from contracts/deployment.md>
CONTRACT_PAYMENT_ID=<from contracts/deployment.md>

PINATA_API_KEY=<your_pinata_api_key>
PINATA_SECRET_KEY=<your_pinata_secret_key>

ADMIN_SECRET_KEY=<stellar_secret_key_for_admin_operations>
```

## 3. Set Up the Database

```bash
npm run db:migrate
```

## 4. Run Dev Server

```bash
npm run dev
# → http://localhost:3001
```

## 5. Run Tests

```bash
npm test
```

---

## API Overview

Base URL: `http://localhost:3001/api/v1`

Full reference: [`api/endpoints.md`](../api/endpoints.md)

### Profiles

| Method | Path | Description |
|---|---|---|
| `GET` | `/profiles` | List profiles (filterable by `?sport=`) |
| `GET` | `/profiles/:address` | Get a single player profile |
| `POST` | `/upload` | Upload video to IPFS, returns `ipfs_hash` |

### Scouts

| Method | Path | Description |
|---|---|---|
| `POST` | `/scouts/verify` | Submit KYC docs for scout verification |
| `GET` | `/scouts/:address/contacts` | List a scout's contact history |

### Admin

| Method | Path | Description |
|---|---|---|
| `POST` | `/admin/scouts/:address/pass` | Issue Scout Pass (admin auth required) |
| `DELETE` | `/admin/scouts/:address/pass` | Revoke Scout Pass (admin auth required) |

---

## Event Listener

The backend subscribes to Stellar ledger events and syncs the PostgreSQL database automatically. No manual polling needed.

Watched events:

| Event | Action |
|---|---|
| `profile_registered` | Insert player row |
| `profile_updated` | Update video hash |
| `contact_initiated` | Insert contact record |
| `scout_pass_issued` | Mark scout as verified |
| `scout_pass_revoked` | Mark scout as unverified |

The listener starts automatically with `npm run dev`. To run it standalone:

```bash
npm run listener
```

---

## IPFS Upload Flow

```
POST /api/v1/upload  { video: <multipart file> }
  → Backend pins file to Pinata
  → Returns { ipfs_hash: "Qm..." }
  → Frontend passes ipfs_hash to register_profile contract call
```

## See Also

- [`ai.md`](../ai.md) — full env var reference and cross-repo conventions
- [`api/endpoints.md`](../api/endpoints.md) — complete API reference
