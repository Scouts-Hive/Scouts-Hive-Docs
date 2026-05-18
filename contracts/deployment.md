# Contract Deployment Guide

## Prerequisites

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32-unknown-unknown

# Install Stellar CLI
cargo install --locked stellar-cli

# Verify
stellar --version
```

## 1. Build

```bash
cd Scouts-Hive-Contract
cargo build --target wasm32-unknown-unknown --release
stellar contract optimize --wasm target/wasm32-unknown-unknown/release/scouts_hive.wasm
```

## 2. Configure Identity

```bash
# Generate a new deployer identity (testnet)
stellar keys generate deployer --network testnet

# Fund it with testnet XLM
stellar keys fund deployer --network testnet
```

## 3. Deploy Contracts

Deploy each contract and save the returned contract ID.

```bash
# Player Registry
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/scouts_hive.optimized.wasm \
  --source deployer \
  --network testnet \
  -- --contract registry

# Scout Verification
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/scouts_hive.optimized.wasm \
  --source deployer \
  --network testnet \
  -- --contract scout

# Contact Handshake
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/scouts_hive.optimized.wasm \
  --source deployer \
  --network testnet \
  -- --contract contact

# Payment / Escrow
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/scouts_hive.optimized.wasm \
  --source deployer \
  --network testnet \
  -- --contract payment
```

## 4. Initialize Payment Contract

```bash
stellar contract invoke \
  --id <PAYMENT_CONTRACT_ID> \
  --source deployer \
  --network testnet \
  -- \
  initialize \
  --admin <ADMIN_ADDRESS> \
  --usdc_token <USDC_TOKEN_ADDRESS> \
  --contact_fee 1000000 \   # 0.1 USDC (7 decimals)
  --spotlight_fee 5000000   # 0.5 USDC
```

## 5. Generate TypeScript Bindings

Run this after every contract deployment or change.

```bash
stellar contract bindings ts \
  --contract-id <REGISTRY_CONTRACT_ID> \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/registry

stellar contract bindings ts \
  --contract-id <SCOUT_CONTRACT_ID> \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/scout

stellar contract bindings ts \
  --contract-id <CONTACT_CONTRACT_ID> \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/contact

stellar contract bindings ts \
  --contract-id <PAYMENT_CONTRACT_ID> \
  --network testnet \
  --output-dir ../Scouts-Hive-Frontend/src/contracts/payment
```

## 6. Update Environment Variables

After deployment, update `.env` in both `Scouts-Hive-Backend` and `Scouts-Hive-Frontend` with the new contract IDs. See [`ai.md`](../ai.md) for the full list of required env vars.

---

## Deployed Contract IDs

> Update this table after every deployment.

### Testnet

| Contract | Contract ID | Deployed At |
|---|---|---|
| Player Registry | `—` | — |
| Scout Verification | `—` | — |
| Contact Handshake | `—` | — |
| Payment / Escrow | `—` | — |

### Mainnet

| Contract | Contract ID | Deployed At |
|---|---|---|
| Player Registry | `—` | — |
| Scout Verification | `—` | — |
| Contact Handshake | `—` | — |
| Payment / Escrow | `—` | — |

---

## Upgrading a Contract

```bash
stellar contract install \
  --wasm target/wasm32-unknown-unknown/release/scouts_hive.optimized.wasm \
  --source deployer \
  --network testnet

stellar contract invoke \
  --id <CONTRACT_ID> \
  --source deployer \
  --network testnet \
  -- upgrade --new_wasm_hash <NEW_WASM_HASH>
```
