---
name: git-workflow
description: Git Collaboration Standards - Commit conventions, PR requirements, Code Review for Solidity projects
author: 0xlayerghost
version: 1.0.0
triggers:
  - "commit"
  - "git"
  - "PR"
  - "pull request"
  - "code review"
---

# Git Collaboration Standards

## Language Rule

- **Always respond in the same language the user is using.** If the user asks in Chinese, respond in Chinese. If in English, respond in English.

## Commit Conventions

Use Conventional Commits:

| Prefix | Usage |
|--------|-------|
| `feat:` | New feature |
| `fix:` | Bug fix |
| `refactor:` | Refactoring (no functionality change) |
| `test:` | Test related |
| `docs:` | Documentation update |
| `chore:` | Build/toolchain changes |

- Run `git diff` before committing to confirm changes
- Only commit, do not push unless explicitly requested
- Never push directly to main/master branch

## PR Requirements

Every PR must include:
- **Change description**: What was done and why
- **Test results**: `forge test` output
- **Deployment impact**: Whether it affects deployed contracts
- **Review focus**: Areas that need special attention

## Code Review

- At least 1 maintainer must approve
- Security-related changes require 2 maintainer approvals
- AI-generated code must pass manual review

## AI Assistance Rules

- AI-generated code must pass `forge test` before committing
- Output should include code snippets, file references, and test cases
