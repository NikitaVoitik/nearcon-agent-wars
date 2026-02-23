---
name: near-research-analysis
description: Use when the challenge involves research, data analysis, chain analytics, report generation, or any non-code deliverable about the NEAR ecosystem.
---

# NEAR Research & Analysis

## Workflow
1. Parse the research question/topic from challenge instructions
2. Gather data from multiple sources (on-chain, web, docs)
3. Analyze and synthesize findings
4. Produce deliverable in requested format

## Data Sources

### On-chain Data via RPC
```bash
# Account info
curl -s -X POST https://rpc.testnet.near.org -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":"1","method":"query","params":{"request_type":"view_account","finality":"final","account_id":"<ACCOUNT>"}}' | jq .

# View contract state
curl -s -X POST https://rpc.testnet.near.org -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":"1","method":"query","params":{"request_type":"view_state","finality":"final","account_id":"<CONTRACT>","prefix_base64":""}}' | jq .

# Call view function
curl -s -X POST https://rpc.testnet.near.org -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":"1","method":"query","params":{"request_type":"call_function","finality":"final","account_id":"<CONTRACT>","method_name":"<METHOD>","args_base64":"<BASE64_ARGS>"}}' | jq .

# Transaction status
curl -s -X POST https://rpc.testnet.near.org -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":"1","method":"tx","params":{"tx_hash":"<HASH>","sender_account_id":"<SENDER>","wait_until":"EXECUTED"}}' | jq .

# Latest block
curl -s -X POST https://rpc.testnet.near.org -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":"1","method":"block","params":{"finality":"final"}}' | jq .

# Validators
curl -s -X POST https://rpc.testnet.near.org -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":"1","method":"validators","params":[null]}' | jq .
```

**Mainnet RPC**: Replace `testnet` with `mainnet` in URL: `https://rpc.mainnet.near.org`

### Encode args for RPC
```bash
echo -n '{"account_id":"alice.near"}' | base64
```

### Web Research
Use WebSearch tool or gemini CLI for current information:
```bash
gemini "Research: <specific NEAR topic>"
```

### Documentation
- Protocol docs: https://docs.near.org
- Protocol spec: https://nomicon.io
- Explorer: https://testnet.nearblocks.io (testnet) / https://nearblocks.io (mainnet)

## Report Template
```markdown
# [Report Title]

## Executive Summary
[2-3 sentence overview of findings]

## Methodology
- Data sources used
- Time period analyzed
- Tools and methods

## Key Findings

### 1. [Finding Title]
[Data, analysis, charts/tables]

### 2. [Finding Title]
[Data, analysis, charts/tables]

### 3. [Finding Title]
[Data, analysis, charts/tables]

## Conclusions & Recommendations
[Actionable takeaways]

## Data Appendix
[Raw data, additional tables, references]
```

## Deliverable Formats
| Format | When to Use | How to Submit |
|--------|-------------|---------------|
| Markdown report | Default for research | Commit .md file to git repo |
| JSON dataset | Data analysis challenges | Commit .json with schema |
| On-chain proof | Verification tasks | Deploy contract, submit tx hash |
| Code + report | Mixed challenges | Code in repo + report in README |

## Speed Tips
- Use multiple web searches in parallel for different sub-topics
- Use RPC directly with curl + jq (no SDK setup needed)
- Structure report sections as you research, don't wait until end
- Tables and bullet points > long paragraphs
- Include raw data in appendix for credibility
