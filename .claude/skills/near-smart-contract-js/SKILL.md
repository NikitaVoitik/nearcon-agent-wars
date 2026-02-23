---
name: near-smart-contract-js
description: Use when building NEAR smart contracts in JavaScript or TypeScript using near-sdk-js. Preferred for rapid prototyping when time is critical.
---

# NEAR Smart Contract Development (JS/TS)

Fastest path for hackathon. Node.js already available, no Rust needed.

## 1. Scaffold Project
```bash
mkdir <contract-name> && cd <contract-name>
npm init -y
npm install near-sdk-js
```

### package.json scripts
```json
{
  "scripts": {
    "build": "near-sdk-js build src/contract.ts build/contract.wasm",
    "test": "ava"
  }
}
```

### tsconfig.json
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "target": "es2020",
    "noEmit": true
  }
}
```

## 2. Contract Template

```typescript
// src/contract.ts
import { NearBindgen, near, call, view, initialize, LookupMap, UnorderedMap, NearPromise } from "near-sdk-js";

@NearBindgen({})
class Contract {
  owner_id: string = "";
  data: LookupMap<string> = new LookupMap<string>("d");

  @initialize({})
  init({ owner_id }: { owner_id: string }) {
    this.owner_id = owner_id;
  }

  @view({})
  get_value({ key }: { key: string }): string | null {
    return this.data.get(key);
  }

  @call({ payableFunction: true })
  set_value({ key, value }: { key: string; value: string }) {
    const sender = near.predecessorAccountId();
    this.data.set(key, value);
    near.log(`${sender} set ${key} = ${value}`);
  }

  @view({})
  get_owner(): string {
    return this.owner_id;
  }
}
```

## 3. Build
```bash
npx near-sdk-js build src/contract.ts build/contract.wasm
```

## 4. Test (with near-workspaces)
```bash
npm install --save-dev near-workspaces ava
```
```typescript
// test/contract.ava.ts
import { Worker, NearAccount } from "near-workspaces";
import test from "ava";

let worker: Worker;
let root: NearAccount;
let contract: NearAccount;

test.before(async (t) => {
  worker = await Worker.init();
  root = worker.rootAccount;
  contract = await root.devDeploy("build/contract.wasm");
  await contract.call(contract, "init", { owner_id: root.accountId });
});

test.after.always(async () => { await worker.tearDown(); });

test("set and get value", async (t) => {
  await root.call(contract, "set_value", { key: "test", value: "42" });
  const result = await contract.view("get_value", { key: "test" });
  t.is(result, "42");
});
```
```bash
npx ava
```

## 5. Deploy
```bash
# Create account if needed
near account create-account fund-later use-auto-generation save-to-folder ~/.near-credentials/testnet/

# Deploy with init
near contract deploy <account-id> use-file build/contract.wasm \
  with-init-call init json-args '{"owner_id":"<owner>"}' \
  prepaid-gas '30 Tgas' attached-deposit '0 NEAR' \
  network-config testnet sign-with-keychain send
```

## 6. Interact
```bash
# View call (free)
near contract call-function as-read-only <contract-id> get_value \
  json-args '{"key":"test"}' network-config testnet now

# Change call (costs gas)
near contract call-function as-transaction <signer-id> <contract-id> set_value \
  json-args '{"key":"hello","value":"world"}' \
  prepaid-gas '30 Tgas' attached-deposit '0 NEAR' \
  network-config testnet sign-with-keychain send
```

## Key Decorators
| Decorator | Purpose |
|-----------|---------|
| `@NearBindgen({})` | Marks class as NEAR contract |
| `@initialize({})` | Init function (call once after deploy) |
| `@view({})` | Read-only, no gas cost |
| `@call({})` | State-changing, costs gas |
| `@call({ payableFunction: true })` | Accepts NEAR deposit |
| `@call({ privateFunction: true })` | Only callable by contract itself |

## Collections
| Type | Use Case |
|------|----------|
| `LookupMap<V>` | Key-value, no iteration |
| `UnorderedMap<V>` | Key-value with iteration |
| `Vector<V>` | Indexed array |
| `UnorderedSet<V>` | Unique values with iteration |
| `LookupSet` | Unique values, no iteration |

## Common Patterns
- Access control: `assert(near.predecessorAccountId() === this.owner_id, "Unauthorized")`
- Deposit check: `assert(near.attachedDeposit() > BigInt(0), "Requires deposit")`
- Cross-contract: `NearPromise.new("other.near").functionCall("method", args, deposit, gas)`
- Events: `near.log(JSON.stringify({ standard: "nep297", event: "...", data: [...] }))`
