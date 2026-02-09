# solidity-agent-kit

A comprehensive agent skills toolkit for Solidity smart contract development with Foundry.

> Equip your AI coding agent with battle-tested Solidity development practices — from coding standards to DeFi security.

## Install

```bash
npx skills add 0xlayerghost/solidity-agent-kit
```

Works with **Claude Code**, **Cursor**, **GitHub Copilot**, **Aider**, and more.

## Skills Included

| Skill | Description |
|-------|-------------|
| **solidity-coding** | Coding standards, naming conventions, project structure for Foundry |
| **solidity-security** | Private key protection, gas control, security best practices |
| **solidity-deploy** | Pre-deployment checks, deployment rules, post-deployment workflow |
| **solidity-testing** | Test organization, coverage requirements, fuzz testing with Foundry |
| **defi-security** | Anti-whale, anti-flash-loan, launch checklist, emergency response |
| **git-workflow** | Conventional commits, PR requirements, code review for Solidity projects |
| **claude-code-usage** | Context management, task strategies, prompt techniques |

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
