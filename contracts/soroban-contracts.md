# Soroban Contracts Reference

All contracts live in the `Scouts-Hive-Contract` repo under `src/`.

---

## Player Registry — `src/registry.rs`

Stores and manages `AspirantProfile` structs on-chain.

### Data Type

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

### Functions

| Function | Auth | Description |
|---|---|---|
| `register_profile(player, sport, position, age, video_ipfs_hash)` | player | Create a new profile |
| `update_profile(player, video_ipfs_hash)` | player | Update video highlight link |
| `get_profile(player)` | none | Read a player's profile |
| `get_profiles_by_sport(sport)` | none | List all profiles for a sport |

### Events emitted

- `profile_registered` — `{ player, sport, ipfs_hash }`
- `profile_updated` — `{ player, ipfs_hash }`

---

## Scout Verification — `src/scout.rs`

Issues and validates Scout Pass Soulbound Tokens (SBTs).

### Functions

| Function | Auth | Description |
|---|---|---|
| `issue_scout_pass(scout)` | admin | Mint a non-transferable Scout Pass to an address |
| `revoke_scout_pass(scout)` | admin | Revoke a scout's pass |
| `is_verified_scout(address)` | none | Returns `true` if address holds a valid Scout Pass |

### Events emitted

- `scout_pass_issued` — `{ scout }`
- `scout_pass_revoked` — `{ scout }`

---

## Contact Handshake — `src/contact.rs`

Records a cryptographic handshake when a scout initiates contact with a player.

### Functions

| Function | Auth | Description |
|---|---|---|
| `initiate_contact(scout, player)` | scout + Scout Pass | Write a contact record on-chain |
| `get_contact_record(scout, player)` | none | Check if a contact record exists |

### Events emitted

- `contact_initiated` — `{ scout, player, timestamp }`

---

## Payment / Escrow — `src/payment.rs`

Handles subscription fees, pay-per-contact, and premium spotlight payments in USDC or HIVE token.

### Functions

| Function | Auth | Description |
|---|---|---|
| `initialize(admin, usdc_token, contact_fee, spotlight_fee)` | — | One-time setup |
| `pay_contact_fee(scout, player)` | scout | Pay fee to unlock contact; calls `initiate_contact` internally |
| `purchase_spotlight(player, sport, duration_days)` | player | Feature profile at top of feed |
| `subscribe(scout, tier, duration_days)` | scout | Pay subscription for unlimited contacts |
| `set_contact_fee(amount)` | admin | Update pay-per-contact fee |
| `set_spotlight_fee(amount)` | admin | Update spotlight fee |
| `withdraw_fees(to)` | admin | Withdraw accumulated platform fees |
| `get_accumulated_fees()` | none | Check total fees collected |

### Events emitted

- `contact_fee_paid` — `{ scout, player, amount }`
- `spotlight_purchased` — `{ player, sport, expires_at }`
- `fees_withdrawn` — `{ to, amount }`

---

## Error Codes

| Code | Error | Description |
|---|---|---|
| 1 | `AlreadyInitialized` | Contract already initialized |
| 2 | `NotInitialized` | Contract not initialized |
| 3 | `ProfileNotFound` | Player profile does not exist |
| 4 | `ProfileAlreadyExists` | Address already has a registered profile |
| 5 | `ScoutPassRequired` | Caller does not hold a valid Scout Pass |
| 6 | `ScoutPassAlreadyIssued` | Address already holds a Scout Pass |
| 7 | `ScoutPassNotFound` | Scout Pass not found for this address |
| 8 | `ContactAlreadyInitiated` | Contact record already exists between these addresses |
| 9 | `Unauthorized` | Caller is not authorized |
| 10 | `InvalidIPFSHash` | IPFS hash is empty or malformed |
| 11 | `InvalidAge` | Age value is outside the accepted range |
| 12 | `InsufficientFee` | Payment amount is below the required fee |
| 13 | `SpotlightAlreadyActive` | Player already has an active spotlight for this sport |
| 14 | `NoFeesToWithdraw` | No accumulated fees available |
| 15 | `Overflow` | Arithmetic overflow detected |

---

## Running Tests

```bash
cd Scouts-Hive-Contract
cargo test
```

## See Also

- [`contracts/deployment.md`](deployment.md) — deployed contract IDs and deployment steps
- [`ai.md`](../ai.md) — how to regenerate TS bindings after contract changes
