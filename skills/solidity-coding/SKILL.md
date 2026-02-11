---
name: solidity-coding
description: "[AUTO-INVOKE] MUST be invoked BEFORE writing or modifying any Solidity contract (.sol files). Covers pragma version, naming conventions, project layout, and OpenZeppelin integration. Trigger: any task involving creating, editing, or reviewing .sol source files."
---

# Solidity Coding Standards

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Coding Principles

- **Pragma**: Use `pragma solidity ^0.8.20;` — keep consistent across all files in the project
- **Dependencies**: OpenZeppelin Contracts 4.9.x, manage imports via `remappings.txt`
- **Error Handling**: Prefer custom errors over `require` strings — saves gas and is more expressive
  - Define: `error InsufficientBalance(uint256 available, uint256 required);`
  - Use: `if (balance < amount) revert InsufficientBalance(balance, amount);`
- **Documentation**: All `public` / `external` functions must have NatSpec (`@notice`, `@param`, `@return`)
- **Event Indexing**: Only add `indexed` to `address` type parameters — add comment if indexing other types
- **Special Keywords**: `immutable` / `constant` / `unchecked` / `assembly` must have inline comment explaining why

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Contract / Library | PascalCase | `MyToken`, `StakingPool` |
| Interface | `I` + PascalCase | `IMyToken`, `IStakingPool` |
| State variable / Function | lowerCamelCase | `totalSupply`, `claimDividend` |
| Constant / Immutable | UPPER_SNAKE_CASE | `MAX_SUPPLY`, `ROUTER_ADDRESS` |
| Event | PascalCase (past tense) | `TokenTransferred`, `PoolCreated` |
| Custom Error | PascalCase | `InsufficientBalance`, `Unauthorized` |
| Function parameter | prefix `_` for setter | `function setFee(uint256 _fee)` |

- **Forbidden**: Pinyin names, single-letter variables (except `i/j/k` in loops), excessive abbreviations

## Code Organization Rules

| Situation | Rule |
|-----------|------|
| Cross-contract constants | Place in `src/common/Const.sol` |
| Interface definitions | Place in `src/interfaces/I<Name>.sol`, separate from implementation |
| Simple on-chain queries | Use `cast call` or `cast send` |
| Complex multi-step operations | Use `forge script` |
| Import style | Use named imports: `import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";` |

## Project Directory Structure

```
src/              — Contract source code
  interfaces/     — Interface definitions (I*.sol)
  common/         — Shared constants, types, errors (Const.sol, Types.sol)
test/             — Test files (*.t.sol)
script/           — Deployment & interaction scripts (*.s.sol)
config/           — Network config, parameters (*.json)
deployments/      — Deployment records (latest.env)
docs/             — Documentation, changelogs
lib/              — Dependencies (managed by forge install)
```

## Configuration Management

- `config/*.json` — network RPC URLs, contract addresses, business parameters
- `deployments/latest.env` — latest deployed contract addresses, must update after each deployment
- `foundry.toml` — compiler version, optimizer settings, remappings
- Important config changes must be documented in the PR description

## Foundry Quick Reference

```bash
# Create new project
forge init <project-name>

# Install dependency
forge install OpenZeppelin/openzeppelin-contracts@v4.9.6

# Build contracts
forge build

# Format code
forge fmt

# Update remappings
forge remappings > remappings.txt
```
