---
name: solidity-security
description: Enforce smart contract security standards in Foundry projects. Use when writing, reviewing, or auditing Solidity contracts — covers private key handling, access control patterns, reentrancy prevention, gas safety, and pre-audit checklists.
---

# Solidity Security Standards

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Private Key Protection

- Store private keys in `.env`, load via `source .env` — never pass keys as CLI arguments
- Never expose private keys in logs, screenshots, conversations, or commits
- Provide `.env.example` with placeholder values for team reference
- Add `.env` to `.gitignore` — verify with `git status` before every commit

## Security Decision Rules

When writing or reviewing Solidity code, apply these rules:

| Situation | Required Action |
|-----------|----------------|
| External ETH/token transfer | Use `ReentrancyGuard` + Checks-Effects-Interactions (CEI) pattern |
| Owner-only function | Inherit `Ownable` from `@openzeppelin/contracts/access/Ownable.sol` (OZ 4.9.x) |
| Multi-role access | Use `AccessControl` from `@openzeppelin/contracts/access/AccessControl.sol` |
| Token approval | Reset allowance to 0 before setting new value, or use `safeIncreaseAllowance` |
| Price data needed | Use Chainlink oracle or TWAP — never use spot pool price directly |
| Upgradeable contract | Prefer UUPS (`UUPSUpgradeable`) over TransparentProxy for gas efficiency |
| Solidity version < 0.8.0 | Must use `SafeMath` — but strongly prefer upgrading to 0.8.20+ |
| Emergency scenario | Inherit `Pausable`, implement `pause()` / `unpause()` with `onlyOwner` |

## Reentrancy Protection

- All contracts with external calls: inherit `ReentrancyGuard`, add `nonReentrant` modifier
  - Import: `@openzeppelin/contracts/security/ReentrancyGuard.sol` (OZ 4.9.x)
- Always apply CEI pattern even with `ReentrancyGuard`:
  1. **Checks** — validate all conditions (`require`)
  2. **Effects** — update state variables
  3. **Interactions** — external calls last

## Input Validation

- Reject `address(0)` for all address parameters
- Reject zero amounts for fund transfers
- Validate array lengths match when processing paired arrays
- Bound numeric inputs to reasonable ranges (prevent dust attacks, gas griefing)

## Gas Control

- Deployment commands must include `--gas-limit` (recommended >= 3,000,000)
- Monitor gas with `forge test --gas-report` — review before every PR
- Configure optimizer in `foundry.toml`: `optimizer = true`, `optimizer_runs = 200`
- Avoid unbounded loops over dynamic arrays — use pagination or pull patterns

## Pre-Audit Checklist

Before submitting code for review or audit, verify:

- [ ] All external/public functions have `nonReentrant` where applicable
- [ ] No `tx.origin` used for authentication (use `msg.sender`)
- [ ] No `delegatecall` to untrusted addresses
- [ ] All `external call` return values checked
- [ ] Events emitted for every state change
- [ ] No hardcoded addresses — use config or constructor params
- [ ] `.env` is in `.gitignore`
- [ ] `forge test` passes with zero failures
- [ ] `forge coverage` shows adequate coverage on security-critical paths

## Security Verification Commands

```bash
# Run all tests with gas report
forge test --gas-report

# Fuzz testing with higher runs for critical functions
forge test --fuzz-runs 10000

# Check test coverage
forge coverage

# Dry-run deployment to verify no runtime errors
forge script script/Deploy.s.sol --fork-url $RPC_URL -vvvv

# Static analysis (if slither installed)
slither src/
```
