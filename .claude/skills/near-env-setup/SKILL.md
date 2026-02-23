---
name: near-env-setup
description: Use when setting up or verifying the NEAR development environment -- installing Rust, near-cli, cargo-near, near-sdk-js, py-near, or any NEAR toolchain dependencies.
---

# NEAR Development Environment Setup

## Quick Check (run first)
```bash
echo "=== Node ===" && node --version && npm --version && \
echo "=== Rust ===" && (rustc --version 2>/dev/null || echo "NOT INSTALLED") && \
echo "=== WASM ===" && (rustup target list --installed 2>/dev/null | grep wasm32 || echo "NOT INSTALLED") && \
echo "=== NEAR CLI ===" && (near --version 2>/dev/null || echo "NOT INSTALLED") && \
echo "=== cargo-near ===" && (cargo near --version 2>/dev/null || echo "NOT INSTALLED") && \
echo "=== Python ===" && python3 --version
```

## Install Missing Tools

### Rust + WASM target
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup target add wasm32-unknown-unknown
```

### near-cli-rs
```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/near/near-cli-rs/releases/latest/download/near-cli-rs-installer.sh | sh
```

### cargo-near
```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/near/cargo-near/releases/latest/download/cargo-near-installer.sh | sh
```

### near-sdk-js (per project, when needed)
```bash
npm init -y && npm install near-sdk-js
```

### py-near (when needed)
```bash
pip3 install py-near
```

## NEAR Account Setup
```bash
# Create testnet account with faucet
near account create-account fund-later use-auto-generation save-to-folder ~/.near-credentials/testnet/

# Import existing account
near account import-account using-seed-phrase "<SEED_PHRASE>" --networkId testnet
```

## Environment Variables
```bash
export NEAR_ENV=testnet
export NEAR_AI_API_KEY=""  # get from human at hackathon start
```

## Speed Optimization
- If challenge is JS-only: skip Rust installation entirely
- If challenge is research-only: skip all SDK installs
- Always check what's installed before installing
- Rust install takes ~2-3 min; plan accordingly
