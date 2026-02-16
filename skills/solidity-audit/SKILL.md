---
name: solidity-audit
description: "Security audit and code review checklist. Covers 30+ vulnerability types with real-world exploit cases (2024-2026). Use when conducting security audits, code reviews, or pre-deployment security assessments."
---

# Solidity Security Audit Checklist

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

> **Usage**: This skill is for security audits and code reviews. It is NOT auto-invoked — call `/solidity-audit` when reviewing contracts for vulnerabilities.

## Contract-Level Vulnerabilities

### 1. Reentrancy

| Variant | Description | Check |
|---------|-------------|-------|
| Same-function | Attacker re-enters the same function via fallback/receive | All external calls after state updates (CEI pattern)? |
| Cross-function | Attacker re-enters a different function sharing state | All functions touching shared state protected by `nonReentrant`? |
| Cross-contract | Attacker re-enters through a different contract that reads stale state | External contracts cannot read intermediate state? |
| Read-only | View function returns stale data during mid-execution state | No critical view functions used as oracle during state transitions? |

**Case**: [GMX v1 (Jul 2025, $42M)](https://www.halborn.com/blog/post/explained-the-gmx-hack-july-2025) — reentrancy in GLP pool on Arbitrum, attacker looped withdrawals to drain liquidity.

### 2. Access Control

| Check | Detail |
|-------|--------|
| Missing modifier | Every state-changing function has explicit access control? |
| Modifier logic | Modifier actually reverts on failure (not just empty check)? |
| State flag | Access-once patterns properly update storage after each user? |
| Admin privilege scope | Owner powers are minimal and time-limited? |

**Case**: [Bybit (Feb 2025, $1.4B)](https://www.halborn.com/blog/post/explained-the-bybit-hack-february-2025) — Safe{Wallet} UI injected with malicious JS, hijacked signing process. Not a contract flaw, but access control at the infrastructure layer.

### 3. Input Validation

| Check | Detail |
|-------|--------|
| Zero address | All address params reject `address(0)`? |
| Zero amount | Fund transfers reject zero amounts? |
| Array bounds | Paired arrays validated for matching length? |
| Arbitrary call | No unvalidated `address.call(data)` where attacker controls `data`? |
| Numeric bounds | Inputs bounded to prevent dust attacks or gas griefing? |

### 4. Flash Loan Attacks

| Variant | Mechanism | Defense |
|---------|-----------|---------|
| Price manipulation | Flash-borrow → swap to move price → exploit price-dependent logic → repay | TWAP oracle with min-liquidity check |
| Governance | Flash-borrow governance tokens → vote → repay in same block | Snapshot voting + minimum holding period + timelock ≥ 48h |
| Liquidation | Flash-borrow → manipulate collateral value → trigger liquidation | Multi-oracle price verification + circuit breaker |
| Combo (rounding) | Flash-borrow → manipulate pool → micro-withdrawals exploit rounding → repay | Minimum withdrawal amount + virtual shares |

**Cases**:
- [Cream Finance (Oct 2021, $130M)](https://rekt.news/cream-rekt-2/) — flash loan + yUSD oracle manipulation + missing reentrancy guard
- [Abracadabra (Mar 2025, $13M)](https://www.halborn.com/blog/post/explained-the-abracadabra-money-hack-march-2025) — state tracking error in cauldron, self-liquidation + bad loan
- [Bunni (Sep 2025, $8.4M)](https://www.theblock.co/post/369564/bunni-smart-contract-rounding-error) — flash loan + pool price manipulation + rounding error micro-withdrawals

### 5. Oracle & Price

| Check | Detail |
|-------|--------|
| Single oracle dependency | Using multiple independent price sources? |
| Stale price | Checking `updatedAt` timestamp and rejecting old data? |
| Spot price usage | Never using raw AMM reserves for pricing? |
| Minimum liquidity | Oracle reverts if pool reserves below threshold? |
| Price deviation | Circuit breaker if price moves beyond threshold vs last known? |
| Chainlink round completeness | Checking `answeredInRound >= roundId`? |

**Case**: [Cream Finance (Oct 2021, $130M)](https://rekt.news/cream-rekt-2/) — attacker manipulated yUSD vault price by reducing supply, then used inflated collateral to drain all lending pools.

### 6. Numerical Issues

| Type | Description | Defense |
|------|-------------|---------|
| Primitive overflow | `uint256 a = uint8(b) + 1` — reverts if b=255 on Solidity ≥0.8 | Use consistent types, avoid implicit narrowing |
| Truncation | `int8(int256Value)` — silently overflows even on ≥0.8 | Use `SafeCast` library for all type narrowing |
| Rounding / precision loss | `usdcAmount / 1e12` always rounds to 0 for small amounts | Multiply before divide; check for zero result |
| Division before multiplication | `(a / b) * c` loses precision | Always `(a * c) / b` |

**Case**: [Bunni (Sep 2025, $8.4M)](https://www.halborn.com/blog/post/explained-the-bunni-hack-september-2025) — rounding errors in micro-withdrawals exploited via flash loan.

### 7. Signature Issues

| Type | Description | Defense |
|------|-------------|---------|
| ecrecover returns address(0) | Invalid sig returns `address(0)`, not revert | Always check `recovered != address(0)` |
| Replay attack | Same signature reused across txs/chains | Include `chainId` + `nonce` + `deadline` in signed data |
| Signature malleability | ECDSA has two valid (s, v) pairs per signature | Use OpenZeppelin `ECDSA.recover` (enforces low-s) |
| Empty loop bypass | Signature verification in for-loop, attacker sends empty array | Check `signatures.length >= requiredCount` before loop |
| Missing msg.sender binding | Proof/signature not bound to caller | Always include `msg.sender` in signed/proven data |

### 8. ERC20 Compatibility

| Issue | Description | Defense |
|-------|-------------|---------|
| Fee-on-transfer | `transfer(100)` may deliver <100 tokens | Check balance before/after, use actual received amount |
| Rebase tokens | Token balances change without transfers | Never cache external balances; always read live |
| No bool return | Some tokens (USDT) don't return bool on transfer | Use `SafeERC20.safeTransfer` |
| ERC777 hooks | Transfer hooks can trigger reentrancy | Use `ReentrancyGuard` on all token-receiving functions |
| Zero-amount transfer | `transferFrom(A, B, 0)` — address poisoning | Reject zero-amount transfers |
| Approval race | Changing allowance from N to M allows spending N+M | Use `safeIncreaseAllowance` / `safeDecreaseAllowance` |

### 9. MEV / Front-Running

| Type | Description | Defense |
|------|-------------|---------|
| Sandwich attack | Attacker front-runs buy + back-runs sell around victim | Slippage protection + deadline parameter |
| ERC4626 inflation | First depositor donates to inflate share price, rounding out later depositors | Minimum first deposit or virtual shares (ERC4626 with offset) |
| Approval front-run | Attacker spends old allowance before new allowance tx confirms | Use `increaseAllowance` not `approve` |
| Unrestricted withdrawal | Attacker monitors mempool for withdraw tx, front-runs with own | Require commit-reveal or auth binding |

### 10. Storage & Low-Level

| Issue | Description |
|-------|-------------|
| Storage pointer | `Foo storage foo = arr[0]; foo = arr[1];` — does NOT update arr[0] |
| Nested delete | `delete structWithMapping` — inner mapping data persists |
| Private variables | All contract storage is publicly readable via `eth_getStorageAt` |
| Unsafe delegatecall | Delegatecall to untrusted contract can `selfdestruct` the caller |
| Proxy storage collision | Upgrade changes parent order → variables overwrite each other (use storage gaps) |
| msg.value in loop | msg.value doesn't decrease in loop — enables double-spend |

### 11. Contract Detection Bypass

| Method | How it works |
|--------|-------------|
| Constructor call | Attack from constructor — `extcodesize == 0` during deployment |
| CREATE2 pre-compute | Pre-calculate contract address, use as EOA before deploying |

## Infrastructure-Level Vulnerabilities

### 12. Frontend / UI Injection

Attackers inject malicious code into the dApp frontend or signing interface.

**Defense**: Verify transaction calldata matches expected function selector and parameters before signing. Use hardware wallet with on-device transaction preview. Audit all frontend dependencies regularly.

**Case**: [Bybit (Feb 2025, $1.4B)](https://www.nccgroup.com/research-blog/in-depth-technical-analysis-of-the-bybit-hack/) — malicious JavaScript injected into Safe{Wallet} UI, tampered with transaction data during signing.

### 13. Private Key & Social Engineering

Compromised keys remain the #1 loss source in 2025-2026.

**Defense**: Store keys in HSM or hardware wallet. Use multisig (≥ 3/5) for all treasury and admin operations. Never share seed phrases with any "support" contact. Conduct regular social engineering awareness training.

**Case**: [Step Finance (Jan 2026, $30M)](https://www.halborn.com/blog/post/explained-the-step-finance-hack-january-2026) — treasury wallet private keys compromised via device breach.

### 14. Cross-Chain Bridge

| Check | Detail |
|-------|--------|
| Inherited code | Audit all bridge logic inherited from third-party frameworks |
| Message verification | Cross-chain messages validated with proper signatures and replay protection? |
| Liquidity isolation | Bridge funds separated from protocol treasury? |

**Case**: [SagaEVM (Jan 2026, $7M)](https://www.theblock.co/post/386638/sagaevm-suffers-exploit) — inherited vulnerable EVM precompile bridge logic from Ethermint.

### 15. Legacy / Deprecated Contracts

Old contracts with known bugs remain callable on-chain forever.

**Defense**: Permanently `pause` or migrate funds from deprecated contracts. Monitor old contract addresses for unexpected activity. Remove mint/admin functions before deprecation.

**Case**: [Truebit (Jan 2026, $26.4M)](https://www.coindesk.com/markets/2026/01/09/truebit-token-tru-crashes-99-9-after-usd26-6m-exploit-drains-8-535-eth) — Solidity 0.6.10 contract lacked overflow protection, attacker minted tokens at near-zero cost.

## Audit Execution Checklist

When conducting a security audit, check each item:

**Reentrancy:**
- [ ] All functions with external calls use `nonReentrant`
- [ ] CEI pattern followed — no state reads after external calls
- [ ] View functions not used as oracle during state transitions

**Access Control:**
- [ ] Every state-changing function has explicit access modifier
- [ ] Modifiers actually revert (not silently pass)
- [ ] Admin privileges are minimal and documented

**Input & Logic:**
- [ ] No unvalidated arbitrary `call` / `delegatecall`
- [ ] No `tx.origin` for authentication
- [ ] Array lengths validated for paired inputs
- [ ] No division-before-multiplication precision loss

**Token Handling:**
- [ ] All ERC20 ops use `SafeERC20`
- [ ] Fee-on-transfer tokens handled (balance diff check)
- [ ] Rebase token balances not cached
- [ ] Zero-amount transfers rejected

**Price & Oracle:**
- [ ] No raw spot price usage
- [ ] Stale price check (`updatedAt` / `answeredInRound`)
- [ ] Minimum liquidity threshold enforced
- [ ] Price deviation circuit breaker

**Signature & Crypto:**
- [ ] `ecrecover` result checked against `address(0)`
- [ ] Signed data includes `chainId`, `nonce`, `msg.sender`, `deadline`
- [ ] Using OZ `ECDSA` (low-s enforced)
- [ ] MerkleProof leaves bound to `msg.sender`

**Flash Loan Defense:**
- [ ] Governance: snapshot voting + holding period + timelock
- [ ] Price: TWAP or multi-oracle, not single-block spot
- [ ] Vault: minimum first deposit or virtual shares (ERC4626)

**Infrastructure:**
- [ ] Third-party dependencies audited (bridge code, inherited contracts)
- [ ] No deprecated contracts still callable with admin/mint functions
- [ ] Multisig on all treasury and admin wallets
- [ ] Frontend transaction verification (calldata matches expected)

## 2021-2026 Incident Quick Reference

| Date | Project | Loss | Attack Type | Root Cause | Source |
|------|---------|------|-------------|------------|--------|
| Oct 2021 | Cream Finance | $130M | Flash loan + oracle | yUSD vault price manipulation via supply reduction | [rekt.news](https://rekt.news/cream-rekt-2/) |
| Feb 2025 | Bybit | $1.4B | UI injection / supply chain | Safe{Wallet} JS tampered via compromised dev machine | [NCC Group](https://www.nccgroup.com/research-blog/in-depth-technical-analysis-of-the-bybit-hack/) |
| Mar 2025 | Abracadabra | $13M | Logic flaw | State tracking error in cauldron liquidation | [Halborn](https://www.halborn.com/blog/post/explained-the-abracadabra-money-hack-march-2025) |
| Jul 2025 | GMX v1 | $42M | Reentrancy | GLP pool cross-contract reentrancy on Arbitrum | [Halborn](https://www.halborn.com/blog/post/explained-the-gmx-hack-july-2025) |
| Sep 2025 | Bunni | $8.4M | Flash loan + rounding | Rounding direction error in withdraw, 44 micro-withdrawals | [The Block](https://www.theblock.co/post/369564/bunni-smart-contract-rounding-error) |
| Oct 2025 | Abracadabra #2 | $1.8M | Logic flaw | cook() validation flag reset, uncollateralized MIM borrow | [Halborn](https://www.halborn.com/blog/post/explained-the-abracadabra-hack-october-2025) |
| Jan 2026 | Step Finance | $30M | Key compromise | Treasury wallet private keys stolen via device breach | [Halborn](https://www.halborn.com/blog/post/explained-the-step-finance-hack-january-2026) |
| Jan 2026 | Truebit | $26.4M | Legacy contract | Solidity 0.6.10 integer overflow in mint pricing | [CoinDesk](https://www.coindesk.com/markets/2026/01/09/truebit-token-tru-crashes-99-9-after-usd26-6m-exploit-drains-8-535-eth) |
| Jan 2026 | SagaEVM | $7M | Supply chain / bridge | Inherited Ethermint precompile bridge vulnerability | [The Block](https://www.theblock.co/post/386638/sagaevm-suffers-exploit) |
