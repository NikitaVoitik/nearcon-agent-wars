---
name: near-defi-operations
description: Use when the challenge involves DeFi operations -- token swaps, liquidity provision, portfolio management, cross-contract calls, or building DeFi protocols on NEAR.
---

# NEAR DeFi Operations

## Cross-Contract Calls

### Rust Promise Pattern
```rust
use near_sdk::{ext_contract, Promise, Gas, NearToken, AccountId};
use near_sdk::json_types::U128;

#[ext_contract(ext_ft)]
trait ExtFungibleToken {
    fn ft_transfer(&mut self, receiver_id: AccountId, amount: U128, memo: Option<String>);
    fn ft_balance_of(&self, account_id: AccountId) -> U128;
    fn ft_transfer_call(&mut self, receiver_id: AccountId, amount: U128, memo: Option<String>, msg: String) -> U128;
}

// Usage in contract:
pub fn transfer_tokens(&self, token_id: AccountId, receiver: AccountId, amount: U128) -> Promise {
    ext_ft::ext(token_id)
        .with_attached_deposit(NearToken::from_yoctonear(1))
        .with_static_gas(Gas::from_tgas(10))
        .ft_transfer(receiver, amount, None)
}

pub fn check_balance(&self, token_id: AccountId, account: AccountId) -> Promise {
    ext_ft::ext(token_id)
        .with_static_gas(Gas::from_tgas(5))
        .ft_balance_of(account)
        .then(
            Self::ext(env::current_account_id())
                .with_static_gas(Gas::from_tgas(10))
                .on_balance_check()
        )
}

#[private]
pub fn on_balance_check(&self, #[callback_result] result: Result<U128, PromiseError>) -> U128 {
    result.unwrap_or(U128(0))
}
```

### JS Promise Pattern
```typescript
@call({})
transfer_tokens({ token_id, receiver, amount }: { token_id: string; receiver: string; amount: string }) {
    return NearPromise.new(token_id)
        .functionCall(
            "ft_transfer",
            JSON.stringify({ receiver_id: receiver, amount, memo: null }),
            BigInt(1),  // 1 yoctoNEAR
            BigInt(10_000_000_000_000)  // 10 Tgas
        );
}
```

## DeFi Patterns

### Simple AMM (Constant Product x*y=k)
```rust
pub fn swap(&mut self, token_in: AccountId, amount_in: U128) -> U128 {
    let reserve_in = self.reserves.get(&token_in).expect("Unknown token");
    let other_token = self.get_other_token(&token_in);
    let reserve_out = self.reserves.get(&other_token).unwrap();

    // x * y = k formula with 0.3% fee
    let amount_in_with_fee = amount_in.0 * 997;
    let numerator = amount_in_with_fee * reserve_out;
    let denominator = reserve_in * 1000 + amount_in_with_fee;
    let amount_out = numerator / denominator;

    // Update reserves
    self.reserves.insert(token_in, reserve_in + amount_in.0);
    self.reserves.insert(other_token, reserve_out - amount_out);

    U128(amount_out)
}
```

### Escrow Pattern
```rust
pub struct Escrow {
    depositor: AccountId,
    beneficiary: AccountId,
    amount: NearToken,
    released: bool,
    deadline: u64,  // nanoseconds
}

#[payable]
pub fn create_escrow(&mut self, beneficiary: AccountId, deadline_seconds: u64) -> u64 {
    let id = self.next_id;
    self.next_id += 1;
    self.escrows.insert(id, Escrow {
        depositor: env::predecessor_account_id(),
        beneficiary,
        amount: env::attached_deposit(),
        released: false,
        deadline: env::block_timestamp() + deadline_seconds * 1_000_000_000,
    });
    id
}

pub fn release(&mut self, escrow_id: u64) -> Promise {
    let escrow = self.escrows.get_mut(&escrow_id).expect("Not found");
    assert!(!escrow.released, "Already released");
    assert_eq!(env::predecessor_account_id(), escrow.depositor, "Only depositor");
    escrow.released = true;
    Promise::new(escrow.beneficiary.clone()).transfer(escrow.amount)
}
```

### Portfolio Tracker
```typescript
@call({})
add_holding({ token_id, amount }: { token_id: string; amount: string }) {
    const sender = near.predecessorAccountId();
    const key = `${sender}:${token_id}`;
    const current = BigInt(this.holdings.get(key) || "0");
    this.holdings.set(key, (current + BigInt(amount)).toString());
}

@view({})
get_portfolio({ account_id }: { account_id: string }): Array<{token: string; amount: string}> {
    // Iterate holdings for this account
    const result: Array<{token: string; amount: string}> = [];
    // ... iterate and filter by account prefix
    return result;
}
```

## Ref Finance (Testnet)
Contract: `ref-finance-101.testnet`
```bash
# View pools
near contract call-function as-read-only ref-finance-101.testnet get_pools \
  json-args '{"from_index":0,"limit":10}' network-config testnet now

# Swap via ft_transfer_call
near contract call-function as-transaction <signer> <token-in> ft_transfer_call \
  json-args '{"receiver_id":"ref-finance-101.testnet","amount":"1000000","msg":"{\"force\":0,\"actions\":[{\"pool_id\":0,\"token_in\":\"<token-in>\",\"token_out\":\"<token-out>\",\"min_amount_out\":\"1\"}]}"}' \
  prepaid-gas '100 Tgas' attached-deposit '1 yoctoNEAR' \
  network-config testnet sign-with-keychain send
```

## Gas Budgeting
| Operation | Gas |
|-----------|-----|
| Simple view call | 5 Tgas |
| ft_transfer | 10 Tgas |
| ft_transfer_call | 30 Tgas |
| Cross-contract (2 hops) | 50 Tgas |
| Complex DeFi tx | 100-200 Tgas |
| Max per transaction | 300 Tgas |
