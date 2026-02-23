---
name: near-frontend-bos
description: Use when the challenge requires building a frontend, BOS (Blockchain Operating System) component, widget, or any user-facing interface that interacts with NEAR.
---

# NEAR Frontend & BOS Components

## Option A: BOS Widget (fastest -- no build step)

BOS widgets are JSX components stored on-chain, rendered by near.org gateway.

### Widget Template
```jsx
const accountId = context.accountId;
const contractId = props.contractId || "contract.testnet";

const [inputValue, setInputValue] = useState("");
const [result, setResult] = useState(null);

// Read from contract (auto-refreshes)
const data = Near.view(contractId, "get_value", { key: "main" });

// Write to contract
const handleSubmit = () => {
  Near.call(contractId, "set_value", {
    key: "main",
    value: inputValue
  }, "30000000000000", "0");
};

return (
  <div className="container p-3">
    <h2>NEAR dApp</h2>
    <p>Connected: <b>{accountId || "Not connected"}</b></p>
    <p>Current value: <b>{data || "Loading..."}</b></p>
    <div className="d-flex gap-2 mt-3">
      <input
        className="form-control"
        placeholder="Enter value"
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
      />
      <button className="btn btn-primary" onClick={handleSubmit}>
        Submit
      </button>
    </div>
  </div>
);
```

### Deploy Widget
```bash
# Deploy to SocialDB via near-cli
WIDGET_CODE=$(cat widget.jsx | jq -Rs .)
near contract call-function as-transaction <account-id> social.near set \
  json-args "{\"data\":{\"<account-id>\":{\"widget\":{\"MyWidget\":{\"\":$WIDGET_CODE}}}}}" \
  prepaid-gas '100 Tgas' attached-deposit '0.1 NEAR' \
  network-config mainnet sign-with-keychain send
```

### View Widget
- URL: `https://near.org/<account-id>/widget/MyWidget`
- Sandbox: `https://near.org/sandbox` (paste code to test)

### Key BOS APIs
| API | Purpose |
|-----|---------|
| `Near.view(contractId, method, args)` | Read contract state |
| `Near.call(contractId, method, args, gas, deposit)` | Call contract method |
| `Social.get(patterns)` | Read from SocialDB |
| `Social.set(data)` | Write to SocialDB |
| `State.update(obj)` | Update component state |
| `Storage.get(key)` / `Storage.set(key, val)` | Local storage |
| `<Widget src="account/widget/Name" props={...} />` | Embed other widgets |

---

## Option B: Next.js + near-api-js (richer UI)

```bash
npx create-next-app@latest my-near-app --typescript --tailwind --app
cd my-near-app
npm install near-api-js
```

### Connection Setup
```typescript
// lib/near.ts
import { connect, keyStores, Contract, WalletConnection } from "near-api-js";

const config = {
  networkId: "testnet",
  keyStore: new keyStores.BrowserLocalStorageKeyStore(),
  nodeUrl: "https://rpc.testnet.near.org",
  walletUrl: "https://testnet.mynearwallet.com/",
  helperUrl: "https://helper.testnet.near.org",
};

export async function initNear() {
  const near = await connect(config);
  const wallet = new WalletConnection(near, "my-app");
  return { near, wallet };
}

export async function viewMethod(contractId: string, method: string, args: any = {}) {
  const { near } = await initNear();
  const account = await near.account("dontcare");
  return account.viewFunction({ contractId, methodName: method, args });
}
```

### React Component
```typescript
"use client";
import { useState, useEffect } from "react";
import { viewMethod } from "@/lib/near";

export default function ContractView() {
  const [data, setData] = useState<string | null>(null);

  useEffect(() => {
    viewMethod("contract.testnet", "get_value", { key: "main" })
      .then(setData)
      .catch(console.error);
  }, []);

  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold">NEAR Contract Data</h1>
      <p className="mt-2">Value: {data ?? "Loading..."}</p>
    </div>
  );
}
```

## Hackathon Strategy
1. Default to **BOS widget** (zero build step, instant deploy, works immediately)
2. Use **Next.js** only if challenge needs a full web app with routing/auth
3. For BOS: test at `https://near.org/sandbox` before deploying
4. BOS has Bootstrap CSS built in -- use `className="btn btn-primary"` etc.
