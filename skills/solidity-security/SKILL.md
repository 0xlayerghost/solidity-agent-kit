---
name: solidity-security
description: Solidity Security Standards - Private key protection, gas control, fundamental security practices
author: 0xlayerghost
version: 1.0.0
triggers:
  - "security"
  - "private key"
  - "gas"
  - "security check"
  - "audit"
globs:
  - "src/**/*.sol"
  - ".env*"
  - "foundry.toml"
---

# Solidity Security Standards

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Private Key Protection

- **Mandatory**: Store private keys in `.env`, load via `source .env`
- **Strictly Forbidden**: Exposing private keys in command line, logs, or screenshots
- **Template**: Provide `.env.example` for team reference
- **Strictly Forbidden**: Any real private keys in conversations, code, or commits

## Gas Control

- **Deployment**: Must set `--gas-limit` (recommended >= 3,000,000)
- **Monitoring**: Run `forge test --gas-report` regularly, watch gas changes in critical functions
- **Configuration**: Configure optimizer parameters in `foundry.toml`

## Fundamental Security Principles

- **Reentrancy Protection**: All contracts with external calls must use `ReentrancyGuard`
- **Overflow Protection**: Solidity 0.8+ has built-in checks, lower versions must use SafeMath
- **Access Control**: Sensitive functions must have access control (`onlyOwner`, `AccessControl`, etc.)
- **Input Validation**: External input parameters must be validated (zero address, zero value, boundary values)
