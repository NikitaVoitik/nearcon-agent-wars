---
name: near-smart-contract-rust
description: Use when building, testing, or deploying a NEAR smart contract in Rust using near-sdk. Use for production-grade contracts or when challenge specifically requires Rust.
---

# NEAR Smart Contract Development (Rust)

## Prerequisites
Invoke /near-env-setup if Rust or cargo-near not available.

## 1. Scaffold Project
```bash
cargo near new <contract-name>
cd <contract-name>
```
Or manually create Cargo.toml:
```toml
[package]
name = "contract-name"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
near-sdk = "5.6.0"

[dev-dependencies]
near-sdk = { version = "5.6.0", features = ["unit-testing"] }

[profile.release]
codegen-units = 1
opt-level = "z"
lto = true
debug = false
panic = "abort"
overflow-checks = true
```

## 2. Contract Template
```rust
// src/lib.rs
use near_sdk::borsh::{BorshDeserialize, BorshSerialize};
use near_sdk::{env, near_bindgen, AccountId, NearToken, PanicOnDefault};
use near_sdk::store::LookupMap;

#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
#[borsh(crate = "near_sdk::borsh")]
pub struct Contract {
    owner_id: AccountId,
    data: LookupMap<String, String>,
}

#[near_bindgen]
impl Contract {
    #[init]
    pub fn new(owner_id: AccountId) -> Self {
        Self {
            owner_id,
            data: LookupMap::new(b"d"),
        }
    }

    pub fn get_value(&self, key: String) -> Option<&String> {
        self.data.get(&key)
    }

    #[payable]
    pub fn set_value(&mut self, key: String, value: String) {
        let sender = env::predecessor_account_id();
        env::log_str(&format!("{} set {} = {}", sender, key, value));
        self.data.insert(key, value);
    }

    pub fn get_owner(&self) -> &AccountId {
        &self.owner_id
    }
}
```

## 3. Build
```bash
cargo near build non-reproducible-wasm
# Output: target/near/<contract-name>/<contract_name>.wasm
```

## 4. Test
```rust
#[cfg(test)]
mod tests {
    use super::*;
    use near_sdk::test_utils::VMContextBuilder;
    use near_sdk::testing_env;

    fn get_context(predecessor: AccountId) -> VMContextBuilder {
        let mut builder = VMContextBuilder::new();
        builder.predecessor_account_id(predecessor);
        builder
    }

    #[test]
    fn test_new() {
        let context = get_context("alice.near".parse().unwrap());
        testing_env!(context.build());
        let contract = Contract::new("alice.near".parse().unwrap());
        assert_eq!(contract.get_owner().as_str(), "alice.near");
    }

    #[test]
    fn test_set_get() {
        let context = get_context("alice.near".parse().unwrap());
        testing_env!(context.build());
        let mut contract = Contract::new("alice.near".parse().unwrap());
        contract.set_value("key1".to_string(), "val1".to_string());
        assert_eq!(contract.get_value("key1".to_string()), Some(&"val1".to_string()));
    }
}
```
```bash
cargo test
```

## 5. Deploy
```bash
near contract deploy <account-id> use-file target/near/<name>/<name>.wasm \
  with-init-call new json-args '{"owner_id":"<owner>"}' \
  prepaid-gas '30 Tgas' attached-deposit '0 NEAR' \
  network-config testnet sign-with-keychain send
```

## 6. Interact
```bash
# View (free)
near contract call-function as-read-only <contract-id> get_value \
  json-args '{"key":"test"}' network-config testnet now

# Change (costs gas)
near contract call-function as-transaction <signer> <contract-id> set_value \
  json-args '{"key":"k","value":"v"}' prepaid-gas '30 Tgas' attached-deposit '0 NEAR' \
  network-config testnet sign-with-keychain send
```

## Cross-Contract Calls
```rust
use near_sdk::{ext_contract, Promise, Gas};

#[ext_contract(ext_other)]
trait OtherContract {
    fn some_method(&self, arg: String) -> String;
}

// In your contract:
pub fn call_other(&self, contract_id: AccountId, arg: String) -> Promise {
    ext_other::ext(contract_id)
        .with_static_gas(Gas::from_tgas(10))
        .some_method(arg)
        .then(
            Self::ext(env::current_account_id())
                .with_static_gas(Gas::from_tgas(5))
                .on_callback()
        )
}

#[private]
pub fn on_callback(&self, #[callback_result] result: Result<String, near_sdk::PromiseError>) -> String {
    match result {
        Ok(val) => val,
        Err(_) => "Call failed".to_string(),
    }
}
```

## Pitfalls
| Issue | Fix |
|-------|-----|
| Missing wasm target | `rustup target add wasm32-unknown-unknown` |
| Contract too large (>4MB) | Enable LTO, `opt-level = "z"` |
| Borsh version mismatch | Add `#[borsh(crate = "near_sdk::borsh")]` |
| "State doesn't exist" | Must call init function after deploy |
| Panic on arithmetic | Use `checked_add`, `checked_sub` |
