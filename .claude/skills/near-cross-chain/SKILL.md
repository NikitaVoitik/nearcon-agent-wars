---
name: near-cross-chain
description: Use when the challenge involves cross-chain operations, chain signatures, NEAR intents, multi-chain interactions, or bridging assets between NEAR and other blockchains.
---

# NEAR Cross-Chain Operations

## Chain Signatures (MPC)

NEAR's MPC network allows a NEAR contract to sign transactions for other chains (Ethereum, Bitcoin, Solana, etc.) without bridges.

### How It Works
1. MPC validators hold key shares -- no single entity has the full key
2. NEAR contract requests a signature from the MPC contract
3. MPC network cooperatively produces the signature
4. Signature is valid on the target chain

### Signer Contract
- Testnet: `v1.signer-prod.testnet`
- Mainnet: `v1.signer.near`

### Derive Address (get your cross-chain address)
```bash
near contract call-function as-read-only v1.signer-prod.testnet derived_public_key \
  json-args '{"path":"ethereum-1","predecessor":"<your-account.testnet>"}' \
  network-config testnet now
```

### Request Signature (in a Rust contract)
```rust
use near_sdk::{Promise, Gas, NearToken, env};

pub fn sign_payload(&self, payload: Vec<u8>, path: String) -> Promise {
    let signer: AccountId = "v1.signer-prod.testnet".parse().unwrap();
    Promise::new(signer).function_call(
        "sign".to_string(),
        serde_json::json!({
            "request": {
                "payload": payload,
                "path": path,
                "key_version": 0
            }
        }).to_string().into_bytes(),
        NearToken::from_near(1),  // deposit for MPC fee
        Gas::from_tgas(250),       // needs significant gas
    )
}
```

### Use Cases
- Sign Ethereum transactions from NEAR
- Control Bitcoin UTXOs from NEAR
- Multi-chain asset management from a single NEAR account

---

## NEAR Intents (NEP-514)

Declarative approach: express WHAT you want, not HOW to execute it.

### Intent Structure
```json
{
  "intent_type": "swap",
  "source": {
    "chain": "near",
    "token": "wrap.near",
    "amount": "1000000000000000000000000"
  },
  "target": {
    "chain": "ethereum",
    "token": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
    "min_amount": "500000000000000000"
  },
  "recipient": "0x1234...",
  "deadline": 1708900000
}
```

### Solver Pattern
1. User/agent submits intent to NEAR contract
2. Solvers (market makers, AI agents) compete to fulfill it
3. Best execution wins (lowest slippage, fastest)
4. Settlement happens on both chains via chain signatures

### Building an Intent Contract
```rust
#[near_bindgen]
impl IntentContract {
    #[payable]
    pub fn submit_intent(&mut self, intent: Intent) -> u64 {
        let id = self.next_id;
        self.next_id += 1;
        self.intents.insert(id, IntentRecord {
            creator: env::predecessor_account_id(),
            intent,
            status: IntentStatus::Open,
            deposit: env::attached_deposit(),
            created_at: env::block_timestamp(),
        });
        id
    }

    pub fn fulfill_intent(&mut self, intent_id: u64, proof: FulfillmentProof) -> Promise {
        let record = self.intents.get_mut(&intent_id).expect("Not found");
        assert_eq!(record.status, IntentStatus::Open, "Not open");
        // Verify proof of cross-chain execution
        record.status = IntentStatus::Fulfilled;
        // Release deposit to solver
        Promise::new(env::predecessor_account_id()).transfer(record.deposit)
    }
}
```

---

## Practical Hackathon Approach

### If challenge is cross-chain:
1. Build the NEAR-side contract (intent submission, signature requests)
2. Use chain signatures testnet for signing payloads
3. If full cross-chain execution is too complex in 30 min:
   - Build NEAR contract that constructs the cross-chain transaction
   - Derive the target chain address
   - Submit a signed payload (even if not broadcast to target chain)
   - Document the full intended flow in README

### If challenge is multi-chain agent:
1. Use near-api-js to interact with NEAR
2. Use ethers.js/web3.js for Ethereum side
3. Chain signatures bridge the two

### Key Testnet Resources
| Resource | Address |
|----------|---------|
| MPC Signer | `v1.signer-prod.testnet` |
| Wrapped NEAR | `wrap.testnet` |
| NEAR Testnet RPC | `https://rpc.testnet.near.org` |
| NEAR Explorer | `https://testnet.nearblocks.io` |
