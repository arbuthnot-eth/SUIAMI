# SUIAMI — SUI-Authenticated Message Identity

Cryptographic proof that a wallet owner controls a SuiNS name. The proof is a signed personal message bundled into a shareable, verifiable token.

## What It Does

A SUIAMI proof answers one question: **"Does this person actually own this .sui name?"**

The wallet owner signs a structured message containing their SuiNS name, address, NFT ID, and timestamp. The signed message is encoded into a compact token (`suiami:...`) that anyone can verify on-chain without the signer being present.

## Token Format

```
suiami:<base64(message)>.<signature>
```

### Message Structure

```typescript
interface SuiamiMessage {
  suiami: string;       // "I am brando"
  datetime: string;     // Human-readable timestamp (ART timezone)
  network: 'sui';
  address: string;      // Sui address (0x...)
  ski: string;          // Profile URL (name.sui.ski)
  nftId: string;        // SuiNS NFT object ID
  timestamp: number;    // Unix timestamp (ms)
  version: 1;
}
```

## Usage

```typescript
import { buildSuiamiMessage, createSuiamiProof, parseSuiamiProof } from 'suiami';

// 1. Build the message
const message = buildSuiamiMessage('brando', '0x2b35...ee28', '0xnft...');

// 2. Sign with wallet (personal message)
const { bytes, signature } = await wallet.signPersonalMessage(
  new TextEncoder().encode(JSON.stringify(message))
);

// 3. Create shareable token
const proof = createSuiamiProof(message, bytes, signature);
console.log(proof.token); // "suiami:eyJ..."

// 4. Anyone can parse and verify
const parsed = parseSuiamiProof(proof.token);
// Verify signature against parsed.message.address on-chain
```

## Verification

To verify a SUIAMI token:
1. Parse the token to extract the message and signature
2. Reconstruct the signed bytes from the message
3. Verify the signature against the address in the message using Sui's `verifyPersonalMessageSignature`
4. Optionally check that the SuiNS NFT (`nftId`) is still owned by the address

## Integration

SUIAMI is used by [.SKI](https://github.com/arbuthnot-eth/.SKI) (sui.ski) for session authentication and identity proofs.

## License

MIT
