# Scouts-Hive-Docs

📚 Central documentation for the Scouts Hive platform — architecture, contract references, API docs, and contribution guides.

> **AI agents (Copilot, Cursor, Kiro, etc.):** This README is your single source of truth for working across the Scouts Hive organisation. Read it before touching any repo.

---

## Organisation Repos

| Repo | Description | Primary Language |
|---|---|---|
| [`Scouts-Hive-Contract`](https://github.com/Scouts-Hive/Scouts-Hive-Contract) | Soroban smart contracts — player registry, scout verification, contact handshake, payments | Rust |
| [`Scouts-Hive-Backend`](https://github.com/Scouts-Hive/Scouts-Hive-Backend) | REST API, event listener, KYC service, IPFS integration, database | TypeScript / Node.js |
| [`Scouts-Hive-Frontend`](https://github.com/Scouts-Hive/Scouts-Hive-Frontend) | Next.js dashboard for players and scouts, Freighter wallet integration | TypeScript / Next.js |
| [`Scouts-Hive-Docs`](https://github.com/Scouts-Hive/Scouts-Hive-Docs) | This repo — central documentation | Markdown |

---

## Docs Structure

```
Scouts-Hive-Docs/
├── README.md
│
├── architecture/
│   ├── overview.md                # System architecture + Mermaid diagrams
│   └── data-flow.md               # End-to-end data flows between repos
│
├── contracts/
│   ├── soroban-contracts.md       # Contract function reference + error codes
│   └── deployment.md              # Deployment steps + live contract IDs
│
├── frontend/
│   └── setup.md                   # Frontend setup + Freighter wallet integration
│
├── backend/
│   └── setup.md                   # Backend setup + event listener overview
│
├── api/
│   └── endpoints.md               # REST API reference
│
└── guides/
    └── contributing.md            # Contribution guide + cross-repo workflow
```

## Quick Links

- **New to the project?** → [`architecture/overview.md`](architecture/overview.md)
- **Setting up locally?** → [`frontend/setup.md`](frontend/setup.md) · [`backend/setup.md`](backend/setup.md)
- **Deploying contracts?** → [`contracts/deployment.md`](contracts/deployment.md)
- **Contributing?** → [`guides/contributing.md`](guides/contributing.md)

---

## How the Repos Connect

```
Scouts-Hive-Contract  (Soroban / Rust)
        │
        │  stellar contract bindings ts
        ▼
Scouts-Hive-Frontend  (Next.js / TypeScript)
        │                        │
        │  REST API calls        │  direct contract calls (TS bindings)
        ▼                        ▼
Scouts-Hive-Backend      Scouts-Hive-Contract  (on Stellar Ledger)
        │                        │
        │  Stellar SDK            │  IPFS hash stored on-chain
        │  event listener         ▼
        └──────────────▶  IPFS / Pinata  (off-chain video storage)
```

---

## Contract Repo — `Scouts-Hive-Contract`

**What it owns:** All on-chain state and business logic.

### Key contracts

| Contract | File | Responsibility |
|---|---|---|
| Player Registry | `src/registry.rs` | Store and update `AspirantProfile` structs |
| Scout Verification | `src/scout.rs` | Issue / revoke Scout Pass SBTs |
| Contact Handshake | `src/contact.rs` | Record `initiate_contact` between scout and player wallets |
| Payment / Escrow | `src/payment.rs` | Handle fees, subscriptions, spotlight payments |

### Core data type

```rust
#[contracttype]
pub struct AspirantProfile {
    pub player_address: Address,
    pub sport: Symbol,
    pub position: Symbol,
    pub age: u32,
    pub video_ipfs_hash: String,
}
```

### Environment variables

```env
STELLAR_NETWORK=testnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
CONTRACT_REGISTRY_ID=<deployed_contract_id>
CONTRACT_SCOUT_ID=<deployed_contract_id>
CONTRACT_CONTACT_ID=<deployed_contract_id>
CONTRACT_PAYMENT_ID=<deployed_contract_id>
DEPLOYER_SECRET_KEY=<stellar_secret_key>
```

### Generate TypeScript bindings (run after every contract change)

```bash
stellar contract bindings ts \
  --contract-id $CONTRACT_REGISTRY_ID \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/registry

stellar contract bindings ts \
  --contract-id $CONTRACT_SCOUT_ID \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/scout

stellar contract bindings ts \
  --contract-id $CONTRACT_CONTACT_ID \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/contact

stellar contract bindings ts \
  --contract-id $CONTRACT_PAYMENT_ID \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/payment
```

---

## Backend Repo — `Scouts-Hive-Backend`

**What it owns:** Off-chain services — REST API, Stellar event listener, IPFS uploads, database, KYC.

### Key modules

| Module | Path | Responsibility |
|---|---|---|
| REST API | `src/api/` | Expose endpoints consumed by the frontend |
| Event Listener | `src/listener/` | Watch Stellar ledger for contract events |
| IPFS Service | `src/ipfs/` | Upload videos to Pinata, return IPFS hash |
| KYC Service | `src/kyc/` | Verify scout identity before pass issuance |
| Database | `src/db/` | PostgreSQL — cache profiles, contact records, events |

### Environment variables

```env
PORT=3001
DATABASE_URL=postgresql://user:pass@localhost:5432/scouts_hive
STELLAR_NETWORK=testnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
CONTRACT_REGISTRY_ID=<same_as_contract_repo>
CONTRACT_SCOUT_ID=<same_as_contract_repo>
CONTRACT_CONTACT_ID=<same_as_contract_repo>
CONTRACT_PAYMENT_ID=<same_as_contract_repo>
PINATA_API_KEY=<pinata_api_key>
PINATA_SECRET_KEY=<pinata_secret_key>
ADMIN_SECRET_KEY=<stellar_secret_key>
```

### API base URL consumed by frontend

```
Development:  http://localhost:3001/api/v1
Production:   https://api.scoutshive.io/api/v1
```

---

## Frontend Repo — `Scouts-Hive-Frontend`

**What it owns:** Next.js UI for players and scouts. Talks to both the backend API and directly to Soroban contracts via generated TS bindings.

### Key directories

| Path | Responsibility |
|---|---|
| `src/contracts/` | Auto-generated Soroban TS bindings (do not edit manually) |
| `src/components/` | Shared UI components |
| `src/app/` | Next.js App Router pages |
| `src/hooks/` | Wallet connection, contract call hooks |
| `src/lib/` | Stellar SDK helpers, IPFS utilities |

### Environment variables

```env
NEXT_PUBLIC_STELLAR_NETWORK=testnet
NEXT_PUBLIC_CONTRACT_REGISTRY_ID=<deployed_contract_id>
NEXT_PUBLIC_CONTRACT_SCOUT_ID=<deployed_contract_id>
NEXT_PUBLIC_CONTRACT_CONTACT_ID=<deployed_contract_id>
NEXT_PUBLIC_API_URL=http://localhost:3001/api/v1
```

### Wallet integration pattern

```typescript
import { getPublicKey, signTransaction } from "@stellar/freighter-api";

// Connect wallet
const publicKey = await getPublicKey();

// Sign and submit a contract call
const signed = await signTransaction(xdr, { network: "TESTNET" });
```

---

## Shared Conventions

### Branch naming

```
feat/<repo-prefix>/<short-description>
fix/<repo-prefix>/<short-description>

Examples:
  feat/contract/add-spotlight-payment
  feat/frontend/profile-upload-form
  fix/backend/ipfs-upload-timeout
```

### Contract ID sharing

Contract IDs deployed to testnet/mainnet are stored in [`contracts/deployment.md`](contracts/deployment.md).  
All repos read from that file as the canonical source — never hardcode IDs in application code.

### IPFS hash flow

1. Backend uploads video to Pinata → receives `ipfs_hash`
2. Backend returns `ipfs_hash` to frontend
3. Frontend calls `register_profile(...)` on the Registry contract with `video_ipfs_hash = ipfs_hash`
4. Hash is now immutably anchored on-chain

### Event-driven updates

The backend event listener watches for these contract events and syncs the database:

- `profile_registered` → insert player row
- `profile_updated` → update video hash
- `contact_initiated` → insert contact record
- `scout_pass_issued` → mark scout as verified
- `scout_pass_revoked` → mark scout as unverified

---

## Cross-Repo Change Checklist

When making a change that spans repos, follow this order:

1. **Contract** — update Rust contract, run `cargo test`, deploy to testnet
2. **Bindings** — regenerate TS bindings and copy to `Scouts-Hive-Frontend/src/contracts/`
3. **Backend** — update event listener or API if contract events/functions changed
4. **Frontend** — update UI to use new bindings or API endpoints
5. **Docs** — update `contracts/deployment.md` with new contract IDs, update relevant docs

---

## Useful Commands

```bash
# Contract
cargo test
cargo build --target wasm32-unknown-unknown --release

# Backend
npm run dev
npm test

# Frontend
npm run dev
npm run build
```
