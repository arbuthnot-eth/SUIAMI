# SUIAMI — SUI-AUTH-MSG-ID

Cryptographic proof that a wallet owner controls a SuiNS name, with cross-chain address attestation via IKA dWallets.

```
npm install suiami
```

## Entry Points

### `suiami` — Build & Parse Proofs

```typescript
import { buildMessage, createProof, parseProof, extractName } from 'suiami';

// Build message with cross-chain addresses
const message = buildMessage('brando', '0x2b35...ee28', '0xnft...', {
  btc: 'bc1q...',
  eth: '0x...',
  sol: '5Kz...',
});

// Sign with wallet
const msgBytes = new TextEncoder().encode(JSON.stringify(message, null, 2));
const { bytes, signature } = await wallet.signPersonalMessage(msgBytes);

// Create shareable token
const proof = createProof(message, bytes, signature);
console.log(proof.token); // "suiami:eyJ..."

// Parse back
const parsed = parseProof(proof.token);
console.log(extractName(parsed.message)); // "brando"
```

### `suiami/verify` — Server-Side Verification

Works in any JS runtime (Node, Deno, Bun, Cloudflare Workers).

```typescript
import { verify } from 'suiami/verify';

const result = await verify(token);
// { valid: true, ownershipVerified: true, nameVerified: true, address: '0x...', nftId: '0x...' }
```

Checks:
- Token format and timestamp freshness (5 min window)
- NFT ownership on-chain via GraphQL
- NFT `domain_name` matches claimed name
- SuinsRegistration or SubDomainRegistration type

### `suiami/roster` — On-Chain Identity Roster

```typescript
import { readByName, readByAddress, readByChain, buildSetIdentityArgs } from 'suiami/roster';

// Lookup by SuiNS name
const record = await readByName('brando');
// { name: 'brando', sui_address: '0x...', chains: { sui: '0x...' }, walrus_blob_id: '...', verified: true }

// Lookup by Sui address
const record2 = await readByAddress('0x2b35...');

// Reverse lookup by chain address
const record3 = await readByChain('btc', 'bc1q...');

// Build PTB args for set_identity
const args = buildSetIdentityArgs('brando', { sui: '0x...', btc: 'bc1q...' });
```

## Architecture

```
On-chain roster          Walrus blob (Seal-encrypted)
┌─────────────────┐      ┌──────────────────────┐
│ SUI address      │      │ btc: "bc1q..."       │
│ walrus_blob_id ──┼─────>│ eth: "0x..."         │
│ seal_nonce       │      │ sol: "5Kz..."        │
│ verified: bool   │      │ dwallet_caps: [...]   │
└─────────────────┘      └──────────────────────┘
```

- **SUI address** stored plaintext on-chain (already public from tx sender)
- **Cross-chain addresses** stored only in Seal-encrypted Walrus blobs (privacy)
- **Storm membership** gates Seal decryption
- **Reciprocal**: reading someone's addresses requires your own SUIAMI proof
- **Viral**: every read auto-writes the reader's roster entry

## Contracts (Sui Mainnet)

| Contract | Address |
|----------|---------|
| Roster Package | `0x2c1d63b3b314f9b6e96c33e9a3bca4faaa79a69a5729e5d2e8ac09d70e1052fa` |
| Roster Object | `0x30b45c51a34b20b5ab99e8c493a82c332e9502e5f4380d1be6cc79e712eaab1d` |

### Seal Policy

`seal_approve_roster_reader` — approves Seal decryption if:
1. Caller has a roster entry (`has_address`)
2. Caller's entry has a non-empty `walrus_blob_id` (reciprocity)

### Move Source

Included in `contracts/` — `roster.move` (identity records + triple-indexed lookups) and `seal_roster.move` (Seal decryption policy).

## Token Format

```
suiami:<base64(message)>.<signature>
```

```typescript
interface SuiamiMessage {
  suiami: string;       // "I am brando"
  datetime: string;     // Timestamp (ART timezone)
  chains: string;       // "sui  0x2b35…ee28\nbtc  bc1qtx…hw23"
  ski: string;          // "brando.sui.ski"
  sui: string;          // Full Sui address
  btc?: string;         // Full BTC address
  eth?: string;         // Full ETH address
  sol?: string;         // Full SOL address
  nftId: string;        // SuiNS NFT object ID
  timestamp: number;    // Unix ms
  version: 2;
}
```

## Integration

Used by [.SKI](https://github.com/arbuthnot-eth/.SKI) (sui.ski) for identity proofs, reciprocal roster exchange, and Storm-gated encrypted messaging.

## License

MIT — brando.sui
