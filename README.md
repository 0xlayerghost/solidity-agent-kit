# solidity-agent-kit

A comprehensive agent skills toolkit for Solidity smart contract development with Foundry.

> Equip your AI coding agent with battle-tested Solidity development practices — from coding standards to DeFi security.

## Install

### Quick Install (all agents)

```bash
# Interactive — choose which skills to install
npx skills add 0xlayerghost/solidity-agent-kit

# Install all 7 skills at once
npx skills add 0xlayerghost/solidity-agent-kit -y
```

This auto-detects all AI agents on your machine and installs for each one. It may create multiple folders (`.agents/`, `.claude/`, `.cursor/`, `.windsurf/`, etc.) — this is normal. The `.agents/` folder holds the actual files; others are just symlinks.

### Install for a Specific Agent

If you only use one agent, specify it with `--agent`:

```bash
# Claude Code only
npx skills add 0xlayerghost/solidity-agent-kit -y --agent claude-code

# Cursor only
npx skills add 0xlayerghost/solidity-agent-kit -y --agent cursor

# Windsurf only
npx skills add 0xlayerghost/solidity-agent-kit -y --agent windsurf
```

### After Install

The CLI auto-detects all agents on your machine and creates folders for each (`.agents/`, `.claude/`, `.cursor/`, etc.). The `.agents/` folder holds the actual files; others are just **symlinks** (no extra disk space).

Add these to your project's `.gitignore` to keep your repo clean:

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

### Supported Agents

Claude Code, Cursor, Windsurf, Trae, Codex, Gemini CLI, OpenCode, and more.

## Skills Included

| Skill | Description | skills.sh |
|-------|-------------|-----------|
| **solidity-coding** | Coding standards, naming conventions, project structure for Foundry | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-coding) |
| **solidity-security** | Private key protection, gas control, security best practices | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-security) |
| **solidity-deploy** | Pre-deployment checks, deployment rules, post-deployment workflow | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-deploy) |
| **solidity-testing** | Test organization, coverage requirements, fuzz testing with Foundry | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/solidity-testing) |
| **defi-security** | Anti-whale, anti-flash-loan, launch checklist, emergency response | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/defi-security) |
| **git-workflow** | Conventional commits, PR requirements, code review for Solidity projects | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/git-workflow) |
| **claude-code-usage** | Context management, task strategies, prompt techniques | [View](https://skills.sh/0xlayerghost/solidity-agent-kit/claude-code-usage) |

## Who Is This For

- Solidity developers using **Foundry** (forge, cast, anvil)
- Teams building **DeFi protocols** and smart contracts
- Developers using **AI coding agents** for Solidity development

## Tech Stack

- **Solidity** ^0.8.20
- **Foundry** (forge, cast, anvil)
- **OpenZeppelin** Contracts 4.9.x

## Project Structure

```
solidity-agent-kit/
├── skills/
│   ├── solidity-coding/      # Coding standards
│   │   └── SKILL.md
│   ├── solidity-security/    # Security practices
│   │   └── SKILL.md
│   ├── solidity-deploy/      # Deployment workflow
│   │   └── SKILL.md
│   ├── solidity-testing/     # Testing standards
│   │   └── SKILL.md
│   ├── defi-security/        # DeFi-specific security
│   │   └── SKILL.md
│   ├── git-workflow/         # Git collaboration
│   │   └── SKILL.md
│   └── claude-code-usage/    # AI agent best practices
│       └── SKILL.md
└── README.md
```

## License

MIT

## Author

**0xlayerghost** — Blockchain & Solidity Engineer

- GitHub: [@0xlayerghost](https://github.com/0xlayerghost)
