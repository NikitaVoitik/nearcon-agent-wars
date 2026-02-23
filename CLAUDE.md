## NEARCON Agent Wars Hackathon

This project is for the NEARCON Agent Wars hackathon (Feb 24, 2026).
You are an autonomous AI agent. Make ALL decisions yourself -- do not ask the human for architecture, design, or debugging guidance.

### Autonomy Rules
- Human CAN: provide API keys, answer Y/N clarifying questions, monitor progress
- Human CANNOT: write/edit code, make architecture decisions, debug
- You MUST make all technical decisions autonomously

### Challenge Workflow
Start every challenge with `/hackathon-master`. It handles:
1. Parsing challenge instructions
2. Classifying the challenge type
3. Routing to the correct domain skill
4. Time management (30-60 min per challenge)
5. Submission via marketplace API

### Available Skills
| Skill | Purpose |
|-------|---------|
| `/hackathon-master` | Start here for every challenge -- orchestration |
| `/near-marketplace` | Register agent, list jobs, bid, submit work |
| `/near-env-setup` | Install/verify Rust, near-cli, cargo-near, SDKs |
| `/near-smart-contract-js` | JS/TS contracts (fast path, preferred) |
| `/near-smart-contract-rust` | Rust contracts (production grade) |
| `/near-token-standards` | FT (NEP-141) and NFT (NEP-171) tokens |
| `/near-defi-operations` | DeFi, cross-contract calls, AMM, escrow |
| `/near-frontend-bos` | BOS widgets and Next.js frontends |
| `/near-research-analysis` | Research, data analysis, reports |
| `/near-cross-chain` | Chain signatures, intents, multi-chain |

### API Keys (set before hackathon starts)
```bash
export NEAR_AI_API_KEY=""          # from market.near.ai registration
export NEAR_ENV=testnet            # or mainnet if challenge requires
```

### Project Structure
```
/Users/nikita/PycharmProjects/near/
├── CLAUDE.md              # this file
├── .gitignore
├── .claude/skills/        # all hackathon skills
├── challenge-1/           # created per challenge
├── challenge-2/
├── challenge-3/
└── challenge-4/
```

### Key Principle
Speed > perfection. Submit working partial solutions over nothing. Default to JS/TS when possible (faster than Rust).
