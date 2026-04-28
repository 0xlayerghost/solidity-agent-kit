<div align="center">

# solidity-agent-kit

**Battle-tested AI agent skills for Solidity smart contract development**

*Equip your coding agent with professional-grade Solidity practices — from coding standards to DeFi security.*

<br/>

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat&logo=opensourceinitiative&logoColor=white)](https://opensource.org/licenses/MIT)
[![Solidity](https://img.shields.io/badge/Solidity-%5E0.8.20-363636?style=flat&logo=solidity&logoColor=white)](https://soliditylang.org)
[![Foundry](https://img.shields.io/badge/built%20with-Foundry-orange?style=flat&logo=ethereum&logoColor=white)](https://getfoundry.sh)
[![Skills](https://img.shields.io/badge/10%20skills-blueviolet?style=flat&logo=bookstack&logoColor=white)](https://skills.sh/0xlayerghost/solidity-agent-kit)
[![Agents](https://img.shields.io/badge/8%2B%20agents-informational?style=flat&logo=probot&logoColor=white)](#supported-agents)

<br/>

**EVM Compatible**

[![Ethereum](https://img.shields.io/badge/Ethereum-3C3C3D?style=flat&logo=ethereum&logoColor=white)](https://ethereum.org)
[![Polygon](https://img.shields.io/badge/Polygon-8247E5?style=flat&logo=polygon&logoColor=white)](https://polygon.technology)
[![Arbitrum](https://img.shields.io/badge/Arbitrum-28A0F0?style=flat&logo=ethereum&logoColor=white)](https://arbitrum.io)
[![Optimism](https://img.shields.io/badge/Optimism-FF0420?style=flat&logo=optimism&logoColor=white)](https://optimism.io)
[![Base](https://img.shields.io/badge/Base-0052FF?style=flat&logo=coinbase&logoColor=white)](https://base.org)
[![Avalanche](https://img.shields.io/badge/Avalanche-E84142?style=flat&logo=avalanche&logoColor=white)](https://avax.network)
[![BNB Chain](https://img.shields.io/badge/BNB%20Chain-F0B90B?style=flat&logo=binance&logoColor=black)](https://bnbchain.org)
[![zkSync](https://img.shields.io/badge/zkSync-1E69FF?style=flat&logo=ethereum&logoColor=white)](https://zksync.io)
[![Linea](https://img.shields.io/badge/Linea-121212?style=flat&logo=consensys&logoColor=white)](https://linea.build)

</div>

---

## What is solidity-agent-kit?

`solidity-agent-kit` is a **collection of AI agent skills** that supercharge your coding agent with professional-grade Solidity development knowledge. Instead of writing one-off prompts or hoping your AI agent knows best practices, you install these skills once and get consistent, battle-tested behaviors across every project.

> Equip your AI coding agent with battle-tested Solidity development practices — from coding standards to DeFi security.

```
Without solidity-agent-kit          With solidity-agent-kit
─────────────────────────           ─────────────────────────
AI agent writes code...             AI agent invokes /solidity-coding
  ❌ inconsistent style               ✅ enforces NatSpec, custom errors
  ❌ misses security checks           ✅ runs /solidity-security rules
  ❌ no deployment verification       ✅ pre-deploy checklist enforced
  ❌ weak test coverage               ✅ fuzz tests, 100% branch coverage
  ❌ DeFi vulnerabilities             ✅ anti-flash-loan, anti-whale guards
```

---

## Table of Contents

- [Install](#install)
- [Skills Included](#skills-included)
- [EVM Chain Support](#evm-chain-support)
- [Architecture](#architecture)
- [Supported Agents](#supported-agents)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [License](#license)

---

## Install

### Quick Install

```bash
# Interactive — choose which skills to install
npx skills add 0xlayerghost/solidity-agent-kit

# Install all skills at once (no prompts)
npx skills add 0xlayerghost/solidity-agent-kit -y
```

Auto-detects all AI agents on your machine and installs for each one. Creates `.agents/`, `.claude/`, `.cursor/`, `.windsurf/` folders — `.agents/` holds the actual files, others are symlinks.

### Install for a Specific Agent

```bash
# Claude Code
npx skills add 0xlayerghost/solidity-agent-kit -y --agent claude-code

# Cursor
npx skills add 0xlayerghost/solidity-agent-kit -y --agent cursor

# Windsurf
npx skills add 0xlayerghost/solidity-agent-kit -y --agent windsurf
```

### After Install

Copy the `CLAUDE.md` template to your project root so skills are auto-invoked — no `/slash-commands` needed:

```bash
curl -sL https://raw.githubusercontent.com/0xlayerghost/solidity-agent-kit/main/CLAUDE.md.template -o CLAUDE.md
```

Or manually copy [`CLAUDE.md.template`](./CLAUDE.md.template) → `CLAUDE.md` in your project root.

> `CLAUDE.md` is loaded at the start of every Claude Code session. It tells your agent **when** to invoke each skill automatically.

Add to `.gitignore` to keep your repo clean:

```gitignore
# Agent skills (installed locally, not committed)
.agents/
.claude/
.cursor/
.trae/
.windsurf/
```

### Uninstall

```bash
npx skills remove 0xlayerghost/solidity-agent-kit
```

---

## Skills Included

| # | Skill | Trigger | Description | Link |
|---|-------|---------|-------------|------|
| 1 | **solidity-coding** | Writing / editing `.sol` files | Pragma, naming, NatSpec, custom errors, anti-patterns, OZ imports | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-coding) |
| 2 | **solidity-security** | Writing / editing any contract | Private key protection, access control, reentrancy guards, gas limits | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-security) |
| 3 | **solidity-testing** | Writing / editing `*.t.sol` | Test layout, coverage ≥90%, fuzz testing, invariant tests with Foundry | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-testing) |
| 4 | **solidity-deploy** | Writing `*.s.sol` / deploying | Pre-deploy checklist, env vars, constructor args, post-deploy workflow | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-deploy) |
| 5 | **solidity-checklist** | Before any on-chain op (`cast send`, `--broadcast`) | 6-layer verification: permissions, dependencies, params, security, testing, knowledge capture | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-checklist) |
| 6 | **solidity-debug** | Debugging failed txs | Cast calldata decoding, revert analysis, gas diagnosis, trace reading | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-debug) |
| 7 | **solidity-audit** | Security reviews | Manual audit checklist: storage, access control, arithmetic, ERC compatibility | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-audit) |
| 8 | **defi-security** | DeFi protocol work | Anti-whale, anti-flash-loan, launch checklist, emergency pause, MEV defense | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/defi-security) |
| 9 | **git-workflow** | Before commits / PRs | Conventional commits, branch naming, PR templates for Solidity projects | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/git-workflow) |
| 10 | **claude-code-usage** | After every `/clear` | Context recovery, subagent strategy, Plan Mode, TodoWrite patterns | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/claude-code-usage) |

---

## EVM Chain Support

Skills are designed for **EVM-compatible chains** and work out of the box with Foundry's multi-chain tooling:

```
Mainnet / L2s                 Testnets
──────────────                ──────────────
Ethereum (ETH)                Sepolia
Polygon (MATIC)               Mumbai
Arbitrum One                  Arbitrum Sepolia
Optimism                      OP Sepolia
Base                          Base Sepolia
Avalanche C-Chain             Fuji
BNB Smart Chain               BSC Testnet
zkSync Era                    zkSync Sepolia
Linea                         Linea Sepolia
```

Chain-specific `foundry.toml` configuration, RPC URL management, and `cast` commands are all covered in the deploy and debug skills.

---

## Architecture

```
Your Project
    │
    ├── CLAUDE.md ◄── skill auto-invoke rules
    │
    └── .agents/
        └── 0xlayerghost/
            └── solidity-agent-kit/
                ├── solidity-coding/    ──► Enforces code style & NatSpec
                ├── solidity-security/  ──► Guards access control & reentrancy
                ├── solidity-testing/   ──► Fuzz + invariant test coverage
                ├── solidity-checklist/ ──► 6-layer on-chain op verification
                ├── solidity-deploy/    ──► Pre/post deploy checklist
                ├── solidity-debug/     ──► Cast-based tx analysis
                ├── solidity-audit/     ──► Manual security review
                ├── defi-security/      ──► DeFi-specific attack vectors
                ├── git-workflow/       ──► Conventional commits & PRs
                └── claude-code-usage/  ──► Agent context strategies


AI Agent Workflow:
─────────────────────────────────────────────────────────────
 User Request ──► CLAUDE.md detects task type
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
    Writing .sol   Testing .t.sol  Deploying .s.sol
          │            │            │
   /solidity-coding  /solidity-   /solidity-deploy
   /solidity-security  testing    /defi-security
          │            │            │
          └────────────┴────────────┘
                       │
                  forge build
                  forge test
                       │
                  ✅ Verified
```

---

## Supported Agents

| Agent | Status | Notes |
|-------|--------|-------|
| **Claude Code** | ✅ Full support | Best experience — CLAUDE.md auto-invoke |
| **Cursor** | ✅ Full support | `.cursor/rules` integration |
| **Windsurf** | ✅ Full support | `.windsurf/rules` integration |
| **Trae** | ✅ Supported | `.trae` integration |
| **OpenCode** | ✅ Supported | — |
| **Gemini CLI** | ✅ Supported | — |
| **GitHub Copilot** | 🚧 Partial | Manual invocation |
| **Codex CLI** | ✅ Supported | — |

---

## Tech Stack

| Tool | Version | Purpose |
|------|---------|---------|
| **Solidity** | `^0.8.20` | Smart contract language |
| **Foundry** | latest | Build, test, deploy (`forge`, `cast`, `anvil`) |
| **OpenZeppelin** | `4.9.x` | Secure contract standards (ERC20/721/1155/Governor) |
| **Slither** | latest | Static analysis (90+ vulnerability detectors) |
| **skills.sh** | latest | Skill package management & distribution |

---

## Project Structure

```
solidity-agent-kit/
├── CLAUDE.md.template              ← Copy to project root as CLAUDE.md
├── skills/
│   ├── solidity-coding/            ← Coding standards & naming conventions
│   │   └── SKILL.md
│   ├── solidity-security/          ← Security best practices
│   │   └── SKILL.md
│   ├── solidity-testing/           ← Test organization & coverage
│   │   └── SKILL.md
│   ├── solidity-checklist/         ← 6-layer checklist for on-chain ops
│   │   └── SKILL.md
│   ├── solidity-deploy/            ← Deployment workflow & checks
│   │   └── SKILL.md
│   ├── solidity-debug/             ← On-chain transaction debugging
│   │   └── SKILL.md
│   ├── solidity-audit/             ← Manual security audit checklist
│   │   └── SKILL.md
│   ├── defi-security/              ← DeFi-specific attack prevention
│   │   └── SKILL.md
│   ├── git-workflow/               ← Git conventions for Solidity projects
│   │   └── SKILL.md
│   └── claude-code-usage/          ← AI agent context strategies
│       └── SKILL.md
├── openai/
├── LICENSE
└── README.md
```

---

## Who Is This For

- Solidity developers using **Foundry** (`forge`, `cast`, `anvil`)
- Teams building **DeFi protocols**, NFT projects, and DAO governance
- Developers using **AI coding agents** who want consistent, secure output
- Security engineers running **automated audits** with Slither + manual checklists
- Anyone deploying on **EVM-compatible chains**

---

## License

MIT — see [LICENSE](./LICENSE)

---

<div align="center">

**Built by [0xlayerghost](https://github.com/0xlayerghost)**

*Blockchain & Solidity Engineer*

[![GitHub](https://img.shields.io/badge/GitHub-0xlayerghost-181717?style=flat&logo=github)](https://github.com/0xlayerghost)
[![skills.sh](https://img.shields.io/badge/skills.sh-profile-blue?style=flat&logo=bookstack&logoColor=white)](https://skills.sh/0xlayerghost)

*If this helped you ship safer contracts, give it a ⭐*

</div>
