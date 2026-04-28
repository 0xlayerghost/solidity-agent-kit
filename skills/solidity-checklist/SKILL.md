---
name: solidity-checklist
description: "[AUTO-INVOKE] MUST be invoked BEFORE any on-chain operation (cast send, forge script --broadcast). Systematic 6-layer verification checklist: permissions, dependencies, parameters, security, testing, and knowledge capture. Trigger: any task involving sending transactions, deploying contracts, or interacting with on-chain state."
---

# Solidity Checklist

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Why This Skill Exists

Most on-chain operation failures come from skipping systematic verification. Instead of the reactive loop:

> deploy -> fail -> paste error -> ask AI -> retry -> fail again -> ask again

This skill enforces a proactive verification flow:

> check permissions -> check dependencies -> check params -> check security -> test locally -> execute -> capture knowledge

## The 6-Layer Preflight Flow

Every on-chain operation MUST pass through these 6 layers **in order** before execution.

```
Layer 1: PERMISSIONS  — Do I have the right to do this?
Layer 2: DEPENDENCIES — Do I understand what this affects?
Layer 3: PARAMETERS   — Are my inputs correct?
Layer 4: SECURITY     — Is this safe to execute?
Layer 5: TESTING      — Have I verified locally first?
Layer 6: EXECUTE & CAPTURE — Do it, then record what I learned.
```

---

## Layer 1: Permissions Check

Before any on-chain call, verify the caller has the required access.

| Check | Command | Pass Condition |
|-------|---------|----------------|
| Contract owner | `cast call <CONTRACT> "owner()" --rpc-url <RPC>` | Returns expected deployer/admin address |
| Role-based access | `cast call <CONTRACT> "hasRole(bytes32,address)" <ROLE_HASH> <CALLER> --rpc-url <RPC>` | Returns `true` |
| Whitelist status | `cast call <CONTRACT> "isWhitelisted(address)" <CALLER> --rpc-url <RPC>` | Returns `true` (if applicable) |
| Token allowance | `cast call <TOKEN> "allowance(address,address)" <OWNER> <SPENDER> --rpc-url <RPC>` | Returns >= required amount |
| Paused state | `cast call <CONTRACT> "paused()" --rpc-url <RPC>` | Returns `false` |

### Decision Rule

| Result | Action |
|--------|--------|
| All checks pass | Proceed to Layer 2 |
| Missing permission | Fix permission first (grant role / approve / unpause), then re-check |
| Unsure which permissions are needed | Read contract source — find all `onlyOwner`, `onlyRole`, `require` in target function |

---

## Layer 2: Dependency Check

Understand the blast radius — what contracts and state does this operation touch?

| Check | How |
|-------|-----|
| Direct dependencies | Read target function source — list all external calls (`IERC20(token).transfer(...)`, etc.) |
| Upstream contracts | Which contracts hold a reference to this contract's address? |
| Downstream effects | Will this state change trigger callbacks, events, or off-chain indexers? |
| Initialization state | `cast call <CONTRACT> "initialized()" --rpc-url <RPC>` — is the contract fully set up? |
| Linked addresses | Verify all addresses stored in contract state are correct and not zero |

### Dependency Map Template

Before operating on a multi-contract system, build a mental (or written) map:

```
ContractA (owner: deployer)
  ├── references: TokenB (ERC20), RouterC
  ├── authorized callers: deployer, ContractD
  └── state deps: must be initialized, TokenB must be approved

ContractD (owner: deployer)
  ├── references: ContractA, OracleE
  └── state deps: OracleE must have price feed set
```

### Decision Rule

| Result | Action |
|--------|--------|
| Dependencies clear | Proceed to Layer 3 |
| Unknown dependency | Read contract source or ask — do NOT proceed blindly |
| Redeployment needed | Trace the full dependency graph to identify ALL contracts needing address updates |

---

## Layer 3: Parameter Validation

Verify every input before sending.

| Check | Common Pitfall |
|-------|---------------|
| Address format | Wrong network address, zero address, checksum mismatch |
| Token decimals | Using 18 decimals for a 6-decimal token (USDC/USDT) |
| Amount precision | `1e18` vs `1e6` — off by 12 orders of magnitude |
| Function selector | Calling wrong function or wrong overload |
| Enum values | Passing invalid enum index |
| Array length | Mismatched array lengths in batch operations |
| Deadline/timestamp | Expired deadline, wrong timezone, block.timestamp vs unix |

### Validation Commands

```bash
# Check token decimals before calculating amounts
cast call <TOKEN> "decimals()" --rpc-url <RPC>

# Verify address is a contract (not EOA)
cast code <ADDRESS> --rpc-url <RPC>
# Empty (0x) = EOA, non-empty = contract

# Decode your own calldata to double-check
cast calldata-decode "functionName(type1,type2)" <CALLDATA>

# Estimate gas before sending (catches obvious reverts)
cast estimate <CONTRACT> "functionName(args)" --from <CALLER> --rpc-url <RPC>
```

### Decision Rule

| Result | Action |
|--------|--------|
| All params verified | Proceed to Layer 4 |
| Decimal mismatch found | Recalculate with correct decimals |
| Address is EOA, expected contract | Wrong address — verify deployment |

---

## Layer 4: Security Quick Check

Scan for common security risks before execution.

| Risk | Check | Red Flag |
|------|-------|----------|
| Reentrancy | Does the function make external calls before updating state? | External call before state update |
| Front-running | Is this a price-sensitive operation (swap, liquidation)? | No slippage protection or deadline |
| Access control | Does the function restrict callers appropriately? | Missing `onlyOwner` / `onlyRole` on sensitive function |
| Value handling | Does the function handle msg.value correctly? | Accepts ETH but doesn't use it, or vice versa |
| Approval amount | Am I approving more than necessary? | `approve(spender, type(uint256).max)` on unknown contract |
| Private key exposure | Am I using keystore, not raw private key? | `--private-key` flag in command |

### Private Key Rule (MANDATORY)

```bash
# NEVER do this
cast send ... --private-key 0xdead...

# ALWAYS do this
cast send ... --account <KEYSTORE_NAME>

# Set up keystore if not done
cast wallet import <NAME> --interactive
```

### Decision Rule

| Result | Action |
|--------|--------|
| No red flags | Proceed to Layer 5 |
| Reentrancy risk found | Add ReentrancyGuard or fix CEI pattern before proceeding |
| Front-running risk | Add slippage/deadline params |
| Private key exposed | STOP — rotate the key, use keystore |

---

## Layer 5: Local Testing

Never send a transaction that hasn't been verified locally or on a fork.

| Method | Command | When to Use |
|--------|---------|-------------|
| Unit test | `forge test --match-test <testName> -vvvv` | New/modified contract functions |
| Fork test | `forge test --fork-url <RPC> --match-test <testName> -vvvv` | Testing against live state |
| Dry-run script | `forge script <Script> --rpc-url <RPC> -vvvv` (no `--broadcast`) | Deployment or complex operations |
| Gas estimation | `cast estimate <CONTRACT> "func(args)" --from <CALLER> --rpc-url <RPC>` | Before any cast send |
| Simulation | `cast call <CONTRACT> "func(args)" --from <CALLER> --rpc-url <RPC>` | Quick function call test |

### Testing Decision Tree

```
Is this a new or modified contract?
├── YES → Write forge test → Run test → Pass? → Proceed
│                                      → Fail? → Fix and re-test
└── NO (calling existing deployed contract)
    ├── Simple read? → cast call (Layer 3 already covers this)
    └── State-changing? → cast estimate first
        ├── Estimate succeeds → Proceed to Layer 6
        └── Estimate reverts → Debug with cast call + revert reason → Fix → Re-estimate
```

### Decision Rule

| Result | Action |
|--------|--------|
| Test passes / estimate succeeds | Proceed to Layer 6 |
| Test fails | Fix the issue — do NOT deploy broken code |
| Estimate reverts | Likely a Layer 1 (permissions) or Layer 3 (params) issue — go back |

---

## Layer 6: Execute & Capture

Execute the operation, verify success, and capture knowledge.

### Execute

```bash
# Send transaction using keystore
cast send <CONTRACT> "functionName(type1,type2)" <arg1> <arg2> \
  --account <KEYSTORE_NAME> \
  --rpc-url <RPC>

# Deploy using forge script
forge script <Script> \
  --rpc-url <RPC> \
  --account <KEYSTORE_NAME> \
  --broadcast \
  --gas-limit <LIMIT> \
  -vvvv
```

### Post-Execution Verification (MANDATORY)

| Check | Command |
|-------|---------|
| TX status | Check `status` field in receipt: 1 = success, 0 = fail |
| State change | `cast call` to verify the expected state change happened |
| Events emitted | Check `logs` in receipt for expected events |
| Balance change | `cast balance` or `cast call balanceOf` to verify token movements |

### Knowledge Capture

After every successful (or failed) operation, capture what you learned:

| What to Record | Where |
|----------------|-------|
| Successful command with parameters | Personal command handbook (`.md` file) |
| New error encountered + solution | Debug notes |
| Contract address and its role | Project architecture doc |
| Permission/role requirements discovered | Contract dependency map |
| Gas cost of common operations | Gas reference table |

### Anti-Pattern: One-Time Consumption

> The biggest leverage: **turn every AI interaction into your own knowledge**, not a one-time consumption.

If you asked AI for a command and it worked:
1. Understand WHY each parameter is what it is
2. Record it in your own words
3. Next time, write it yourself first — then verify with AI

---

## Quick Reference: Preflight by Operation Type

| Operation | Critical Layers | Most Common Failure |
|-----------|----------------|---------------------|
| `approve` | L1 (owner), L3 (amount/decimals) | Wrong decimals |
| `transfer` | L1 (balance), L3 (amount/decimals) | Insufficient balance |
| `addLiquidity` | L1 (approve both tokens), L3 (amounts/ratio), L5 (estimate) | Missing approval |
| `removeLiquidity` | L1 (LP approve), L3 (minAmounts), L4 (slippage) | Slippage too tight |
| `stake` | L1 (approve + whitelist), L2 (staking contract state), L3 (amount) | Staking not active / not whitelisted |
| `unstake` | L1 (staker status), L2 (lock period), L3 (amount) | Lock period not expired |
| `deploy` | L3 (constructor args), L4 (full security check), L5 (fork test mandatory) | Constructor arg mismatch |
| `upgrade` | L1 (proxy admin), L2 (storage layout), L4 (storage collision), L5 (fork test mandatory) | Storage layout incompatible |

---

## Preflight Summary Card

Print this mental checklist before EVERY on-chain operation:

```
[ ] L1 PERMISSIONS  — Can I call this? (owner/role/approve/whitelist/paused)
[ ] L2 DEPENDENCIES — What does this touch? (contracts/state/events)
[ ] L3 PARAMETERS   — Are inputs correct? (address/decimals/amount/selector)
[ ] L4 SECURITY     — Is this safe? (reentrancy/frontrun/key exposure)
[ ] L5 TESTING      — Did I test locally? (forge test/estimate/dry-run)
[ ] L6 EXECUTE      — Send it, verify it, record what I learned.
```

> 6 layers, 2 minutes. Saves hours of debugging and wasted gas.
