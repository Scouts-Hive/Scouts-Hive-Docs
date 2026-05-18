# Data Flow

End-to-end data flows for the three primary user journeys.

---

## 1. Player Registers a Profile

```mermaid
sequenceDiagram
    actor Player
    participant Frontend
    participant Backend
    participant IPFS as IPFS / Pinata
    participant Contract as Registry Contract
    participant Ledger as Stellar Ledger

    Player->>Frontend: fills profile form + uploads video
    Frontend->>Backend: POST /api/v1/upload  { video_file }
    Backend->>IPFS: pin video file
    IPFS-->>Backend: ipfs_hash
    Backend-->>Frontend: { ipfs_hash }
    Player->>Frontend: confirms transaction in Freighter
    Frontend->>Contract: register_profile(player, sport, position, age, ipfs_hash)
    Contract->>Ledger: write AspirantProfile
    Ledger-->>Contract: tx confirmed
    Contract-->>Frontend: ok
    Ledger->>Backend: emit profile_registered event
    Backend->>Backend: insert player row in PostgreSQL
```

---

## 2. Scout Discovers and Contacts a Player

```mermaid
sequenceDiagram
    actor Scout
    participant Frontend
    participant Backend
    participant Contract as Contact Contract
    participant Ledger as Stellar Ledger

    Scout->>Frontend: browse profiles (no wallet signature)
    Frontend->>Backend: GET /api/v1/profiles?sport=football
    Backend-->>Frontend: [ AspirantProfile[] ] from PostgreSQL cache

    Note over Scout,Frontend: Scout decides to initiate contact
    Scout->>Frontend: clicks "Connect" on player profile
    Frontend->>Contract: initiate_contact(scout, player)
    Note over Contract: requires Scout Pass SBT
    Contract->>Ledger: write contact record
    Ledger-->>Contract: tx confirmed
    Contract-->>Frontend: contact_record_id
    Ledger->>Backend: emit contact_initiated event
    Backend->>Backend: insert contact record in PostgreSQL
    Backend->>Backend: notify player (email / push)
```

---

## 3. Admin Issues a Scout Pass

```mermaid
sequenceDiagram
    actor Admin
    participant Backend
    participant KYC as KYC Service
    participant Contract as Scout Contract
    participant Ledger as Stellar Ledger

    Admin->>Backend: POST /api/v1/scouts/verify  { scout_address, kyc_docs }
    Backend->>KYC: verify identity documents
    KYC-->>Backend: approved
    Backend->>Contract: issue_scout_pass(scout_address)
    Contract->>Ledger: mint Scout Pass SBT to scout_address
    Ledger-->>Contract: tx confirmed
    Ledger->>Backend: emit scout_pass_issued event
    Backend->>Backend: mark scout as verified in PostgreSQL
    Backend-->>Admin: { scout_address, status: "verified" }
```

---

## 4. IPFS Hash Integrity Check

When the frontend loads a player profile:

```
1. Frontend calls GET /api/v1/profiles/:address  →  Backend returns cached profile (fast)
2. Frontend also calls Registry Contract get_profile(address)  →  reads ipfs_hash from chain
3. Frontend compares the two hashes
4. If they match  →  render video from IPFS gateway
5. If they differ →  show warning: "Profile data may be stale, on-chain hash is authoritative"
```

The on-chain hash is always the source of truth.

---

## 5. Fee Payment Flow

```mermaid
sequenceDiagram
    actor Scout
    participant Frontend
    participant Contract as Payment Contract
    participant USDC as USDC Token
    participant Ledger as Stellar Ledger

    Scout->>Frontend: clicks "Unlock Contact Info"
    Frontend->>USDC: approve(payment_contract, contact_fee)
    Frontend->>Contract: initiate_contact(scout, player)
    Contract->>USDC: transfer(scout → platform_escrow, fee)
    USDC->>Ledger: record transfer
    Contract->>Ledger: write contact record
    Ledger-->>Contract: tx confirmed
    Contract-->>Frontend: contact_record_id
```

---

## Cross-Repo Data Dependencies

| Data | Produced by | Consumed by |
|---|---|---|
| `ipfs_hash` | Backend (IPFS upload) | Frontend → Contract |
| `AspirantProfile` | Contract (on-chain) | Backend (event listener) → PostgreSQL → Frontend API |
| `contact_record` | Contract (on-chain) | Backend (event listener) → PostgreSQL |
| `scout_verified` flag | Contract (Scout Pass) | Frontend (gate `initiate_contact` button) |
| Contract IDs | Contract repo (deployment) | Backend `.env`, Frontend `.env` |
| TS bindings | Contract repo (`stellar contract bindings ts`) | Frontend `src/contracts/` |
