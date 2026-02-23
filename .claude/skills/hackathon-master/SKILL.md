---
name: hackathon-master
description: Use when starting a hackathon challenge, receiving challenge instructions, or needing to orchestrate autonomous challenge completion at NEARCON Agent Wars hackathon.
---

# NEARCON Agent Wars -- Master Orchestration

## CRITICAL CONSTRAINTS
- Human CANNOT write/edit code, make architecture decisions, or debug
- Human CAN provide API keys, answer Y/N, monitor
- 30-60 minutes per challenge -- move FAST
- Every decision is yours. Do not ask the human for architectural guidance.

## Challenge Intake Workflow

### Step 1: Fetch & Parse Challenge Instructions
- If given a URL: fetch it with WebFetch or curl and extract instructions
- If given pasted text: parse directly
- Extract: objective, deliverables, constraints, evaluation criteria, job_id

### Step 2: Classify Challenge Type

| Pattern in Instructions | Type | Skill to Invoke |
|------------------------|------|-----------------|
| "smart contract", "deploy contract", "Rust" | Contract (Rust) | /near-smart-contract-rust |
| "quick prototype", "JavaScript contract", "TypeScript" | Contract (JS) | /near-smart-contract-js |
| "token", "fungible", "NFT", "NEP-141", "NEP-171", "memecoin" | Token/NFT | /near-token-standards |
| "DeFi", "swap", "liquidity", "portfolio", "yield", "AMM" | DeFi | /near-defi-operations |
| "frontend", "UI", "widget", "BOS", "component", "dApp" | Frontend | /near-frontend-bos |
| "research", "analysis", "report", "data", "investigate" | Research | /near-research-analysis |
| "cross-chain", "chain signature", "intent", "bridge", "multi-chain" | Cross-chain | /near-cross-chain |
| "API", "integration", "marketplace", "agent" | API/Agent | /near-marketplace |

If unclear, default to JS contract approach (fastest).

### Step 3: Time Budget
- 0-5 min: Parse challenge, classify, plan approach
- 5-10 min: Environment check (invoke /near-env-setup if needed)
- 10-40 min: Build solution (invoke domain skill)
- 40-50 min: Test and verify
- 50-60 min: Submit (invoke /near-marketplace)

### Step 4: Execute
1. Create a subdirectory for this challenge: `mkdir -p /Users/nikita/PycharmProjects/near/challenge-N`
2. Invoke the domain skill
3. Build and test the solution
4. Git commit the solution

### Step 5: Submit
1. Push to GitHub: `git push`
2. Compute deliverable hash: `git rev-parse HEAD`
3. Invoke /near-marketplace to submit:
   - deliverable_url = GitHub repo URL (with path to challenge dir)
   - deliverable_hash = git commit SHA or sha256 of primary file

## Completion Checklist
- [ ] Challenge parsed and classified
- [ ] Environment verified
- [ ] Solution directory created
- [ ] Solution built and locally tested
- [ ] Solution deployed (if contract/frontend)
- [ ] Git committed and pushed
- [ ] Submitted via marketplace API
- [ ] Submission confirmed (check response)

## If Stuck
- Time-box investigation to 5 minutes max
- If Rust approach fails, pivot to JS/TS (faster iteration)
- If deployment fails, submit code repo URL as deliverable
- Use web search for any NEAR API questions
- If totally blocked, submit partial work -- partial > nothing
