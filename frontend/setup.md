# Frontend Setup

Repo: `Scouts-Hive-Frontend` — Next.js dashboard for players and scouts.

## Prerequisites

- Node.js 20+
- [Freighter wallet](https://www.freighter.app/) browser extension installed

## 1. Install Dependencies

```bash
cd Scouts-Hive-Frontend
npm install
```

## 2. Configure Environment

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```env
NEXT_PUBLIC_STELLAR_NETWORK=testnet
NEXT_PUBLIC_RPC_URL=https://soroban-testnet.stellar.org
NEXT_PUBLIC_CONTRACT_REGISTRY_ID=<from contracts/deployment.md>
NEXT_PUBLIC_CONTRACT_SCOUT_ID=<from contracts/deployment.md>
NEXT_PUBLIC_CONTRACT_CONTACT_ID=<from contracts/deployment.md>
NEXT_PUBLIC_CONTRACT_PAYMENT_ID=<from contracts/deployment.md>
NEXT_PUBLIC_API_URL=http://localhost:3001/api/v1
```

## 3. Add Soroban TS Bindings

Bindings are auto-generated from the deployed contracts. If `src/contracts/` is empty, run from the contract repo:

```bash
# From Scouts-Hive-Contract root
stellar contract bindings ts \
  --contract-id $CONTRACT_REGISTRY_ID \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/registry
```

See [`contracts/deployment.md`](../contracts/deployment.md) for all four contracts.

## 4. Run Dev Server

```bash
npm run dev
# → http://localhost:3000
```

## 5. Build for Production

```bash
npm run build
npm start
```

---

## Wallet Integration

The frontend uses `@stellar/freighter-api` for wallet connection and transaction signing.

### Connect wallet

```typescript
import { isConnected, getPublicKey } from "@stellar/freighter-api";

export async function connectWallet(): Promise<string | null> {
  if (!(await isConnected())) return null;
  return getPublicKey();
}
```

### Sign and submit a contract call

```typescript
import { signTransaction } from "@stellar/freighter-api";
import { SorobanRpc, TransactionBuilder } from "@stellar/stellar-sdk";

export async function submitTx(xdr: string, network: string) {
  const signed = await signTransaction(xdr, { network });
  const server = new SorobanRpc.Server(process.env.NEXT_PUBLIC_RPC_URL!);
  return server.sendTransaction(
    TransactionBuilder.fromXDR(signed, network)
  );
}
```

### Call a contract function (using generated bindings)

```typescript
import { Client as RegistryClient } from "@/contracts/registry";

const client = new RegistryClient({
  contractId: process.env.NEXT_PUBLIC_CONTRACT_REGISTRY_ID!,
  networkPassphrase: Networks.TESTNET,
  rpcUrl: process.env.NEXT_PUBLIC_RPC_URL!,
});

const profile = await client.get_profile({ player: walletAddress });
```

---

## Key Pages

| Route | Description |
|---|---|
| `/` | Landing page |
| `/player/register` | Aspirant profile creation form |
| `/player/[address]` | Public player profile + video |
| `/scout/discover` | Scout browse feed (incognito) |
| `/scout/dashboard` | Scout's contact history |
| `/admin` | Admin panel — issue/revoke Scout Passes |

## See Also

- [`README.md`](../README.md) — env vars, branch conventions, cross-repo flow
- [`backend/setup.md`](../backend/setup.md) — start the API the frontend depends on
