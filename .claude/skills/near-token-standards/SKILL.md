---
name: near-token-standards
description: Use when the challenge involves creating fungible tokens (NEP-141), NFTs (NEP-171), memecoins, token launches, or any token-related smart contract on NEAR.
---

# NEAR Token Standards

## NEP-141: Fungible Token

### Required Interface
- `ft_transfer(receiver_id, amount, memo?)` -- transfer tokens
- `ft_transfer_call(receiver_id, amount, memo?, msg)` -- transfer + notify receiver
- `ft_total_supply() -> U128` -- total supply
- `ft_balance_of(account_id) -> U128` -- account balance

### Metadata (NEP-148)
- `ft_metadata()` returns: spec, name, symbol, icon (data URL), decimals, reference

### Rust Implementation (recommended)
```toml
[dependencies]
near-sdk = "5.6.0"
near-contract-standards = "5.6.0"
```

```rust
use near_contract_standards::fungible_token::{FungibleToken, FungibleTokenCore};
use near_contract_standards::fungible_token::metadata::{
    FungibleTokenMetadata, FungibleTokenMetadataProvider, FT_METADATA_SPEC,
};
use near_sdk::borsh::{BorshDeserialize, BorshSerialize};
use near_sdk::collections::LazyOption;
use near_sdk::json_types::U128;
use near_sdk::{env, near_bindgen, AccountId, PanicOnDefault, PromiseOrValue, NearToken};

#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
#[borsh(crate = "near_sdk::borsh")]
pub struct TokenContract {
    token: FungibleToken,
    metadata: LazyOption<FungibleTokenMetadata>,
}

#[near_bindgen]
impl TokenContract {
    #[init]
    pub fn new(owner_id: AccountId, total_supply: U128, name: String, symbol: String, decimals: u8) -> Self {
        let metadata = FungibleTokenMetadata {
            spec: FT_METADATA_SPEC.to_string(),
            name,
            symbol,
            icon: None,
            reference: None,
            reference_hash: None,
            decimals,
        };
        let mut this = Self {
            token: FungibleToken::new(b"t"),
            metadata: LazyOption::new(b"m", Some(&metadata)),
        };
        this.token.internal_register_account(&owner_id);
        this.token.internal_deposit(&owner_id, total_supply.into());
        near_contract_standards::fungible_token::events::FtMint {
            owner_id: &owner_id,
            amount: &total_supply,
            memo: Some("Initial supply"),
        }.emit();
        this
    }

    #[payable]
    pub fn storage_deposit(&mut self, account_id: Option<AccountId>) {
        let account = account_id.unwrap_or_else(env::predecessor_account_id);
        if !self.token.accounts.contains_key(&account) {
            let deposit = env::attached_deposit();
            assert!(deposit >= NearToken::from_millinear(50), "Requires 0.05 NEAR for storage");
            self.token.internal_register_account(&account);
        }
    }
}

near_contract_standards::impl_fungible_token_core!(TokenContract, token);
near_contract_standards::impl_fungible_token_storage!(TokenContract, token);

#[near_bindgen]
impl FungibleTokenMetadataProvider for TokenContract {
    fn ft_metadata(&self) -> FungibleTokenMetadata {
        self.metadata.get().unwrap()
    }
}
```

### Deploy & Init
```bash
near contract deploy <token-account> use-file target/near/<name>/<name>.wasm \
  with-init-call new json-args '{"owner_id":"<owner>","total_supply":"1000000000000000000000000000","name":"My Token","symbol":"MTK","decimals":24}' \
  prepaid-gas '30 Tgas' attached-deposit '0 NEAR' \
  network-config testnet sign-with-keychain send
```

### Interact
```bash
# Check balance
near contract call-function as-read-only <token> ft_balance_of json-args '{"account_id":"<account>"}' network-config testnet now

# Transfer (sender must sign)
near contract call-function as-transaction <sender> <token> ft_transfer \
  json-args '{"receiver_id":"<receiver>","amount":"1000000000000000000000000"}' \
  prepaid-gas '30 Tgas' attached-deposit '1 yoctoNEAR' \
  network-config testnet sign-with-keychain send
```

---

## NEP-171: Non-Fungible Token

### JS Implementation (fastest for hackathon)
```typescript
import { NearBindgen, near, call, view, initialize, UnorderedMap, LookupMap } from "near-sdk-js";

@NearBindgen({})
class NFTContract {
  owner_id: string = "";
  token_count: number = 0;
  token_owner: LookupMap<string> = new LookupMap("to");
  token_metadata: UnorderedMap<string> = new UnorderedMap("tm");

  @initialize({})
  init({ owner_id }: { owner_id: string }) {
    this.owner_id = owner_id;
  }

  @call({ payableFunction: true })
  nft_mint({ token_id, receiver_id, token_metadata }: {
    token_id: string; receiver_id: string; token_metadata: { title: string; description: string; media?: string }
  }): { token_id: string; owner_id: string; metadata: any } {
    assert(near.predecessorAccountId() === this.owner_id, "Only owner can mint");
    assert(!this.token_owner.get(token_id), "Token already exists");
    this.token_owner.set(token_id, receiver_id);
    this.token_metadata.set(token_id, JSON.stringify(token_metadata));
    this.token_count += 1;
    // NEP-297 event
    near.log(`EVENT_JSON:${JSON.stringify({
      standard: "nep171", version: "1.2.0",
      event: "nft_mint",
      data: [{ owner_id: receiver_id, token_ids: [token_id] }]
    })}`);
    return { token_id, owner_id: receiver_id, metadata: token_metadata };
  }

  @call({ payableFunction: true })
  nft_transfer({ receiver_id, token_id }: { receiver_id: string; token_id: string }) {
    assert(near.attachedDeposit() >= BigInt(1), "Requires 1 yoctoNEAR");
    const owner = this.token_owner.get(token_id);
    assert(owner === near.predecessorAccountId(), "Not token owner");
    this.token_owner.set(token_id, receiver_id);
  }

  @view({})
  nft_token({ token_id }: { token_id: string }): any | null {
    const owner = this.token_owner.get(token_id);
    if (!owner) return null;
    return {
      token_id,
      owner_id: owner,
      metadata: JSON.parse(this.token_metadata.get(token_id) || "{}")
    };
  }

  @view({})
  nft_total_supply(): number { return this.token_count; }
}
```

---

## Memecoin Launch Checklist
1. Choose: name, symbol, decimals (24 for NEAR standard), icon (data URL or IPFS)
2. Set total supply (e.g., `"1000000000000000000000000000000"` = 1B tokens with 24 decimals)
3. Deploy FT contract with NEP-141 + NEP-148
4. Register storage for distribution accounts
5. Transfer initial allocations
6. Verify: `ft_metadata`, `ft_total_supply`, `ft_balance_of`
