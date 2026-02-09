---
name: solidity-deploy
description: Solidity Deployment Workflow - Pre-deployment checks, deployment rules, post-deployment operations
author: 0xlayerghost
version: 1.0.0
triggers:
  - "deploy"
  - "deployment"
  - "go live"
  - "forge script"
  - "forge create"
  - "verify contract"
globs:
  - "script/**/*.sol"
  - "script/**/*.s.sol"
  - "deployments/**"
  - "config/**/*.json"
---

# Deployment Workflow

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Pre-deployment Checks (all must pass)

1. Read development standards
2. Run `forge fmt` to format code
3. Run `forge test` to ensure all tests pass
4. Verify `config/*.json` parameter correctness
5. Dry-run deployment: `forge script <Script> --fork-url <RPC> -vvvv`
6. Confirm deployer account has sufficient gas balance
7. Deployment command must include `--gas-limit`

## Deployment Rules

- **No verification by default**: Do not use `--verify` flag. Contracts are not verified on block explorers by default. Only add `--verify` when explicitly requested
- **Deployment command must include `--gas-limit`**

## Post-deployment Operations (all must be completed)

1. Update contract addresses in `config/*.json` and `deployments/latest.env`
2. Test critical functionality (manually call core functions to confirm)
3. Record changes in `docs/CHANGELOG.md`
4. Submit PR with deployment transaction link
5. If verification is needed, execute `forge verify-contract` separately

## Deployment Command Templates

```bash
# Load environment variables
source .env

# Standard deployment (no verification by default)
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --gas-limit 5000000 \
  -vvvv

# Only add --verify when explicitly requested
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --verify \
  --gas-limit 5000000 \
  -vvvv

# Verify contract separately (when needed)
forge verify-contract <ADDRESS> <CONTRACT> \
  --chain-id <CHAIN_ID> \
  --constructor-args $(cast abi-encode "constructor(address)" <ARG>)
```
