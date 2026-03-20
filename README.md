[English](README.md) | [中文](README.zh-CN.md)

# codex-code-review

A Claude Code skill that orchestrates an **Opus writes → Codex audits → Opus fixes → Codex confirms → push** loop for Codex-verified deployments.

## How It Works

```
Phase 1        Phase 2         Phase 3        Phase 4         Phase 5
Opus writes → Codex audits → Opus fixes → Codex re-audits → Push
  + commit      (review)      + commit       (confirm)       (ask first)
```

1. **Opus** implements the feature or fix and commits
2. **Codex** reviews the diff (`codex review --base <branch>`) for bugs, security, performance, and style issues
3. **Opus** fixes all findings and commits again
4. **Codex** re-audits to confirm the fixes are clean (max 3 rounds)
5. Code is pushed after user confirmation

## Audit Priority

| Priority | Category | Action |
|----------|----------|--------|
| 1 | Bugs / Logic errors | Must fix |
| 2 | Security issues | Must fix |
| 3 | Performance problems | Fix if reasonable |
| 4 | Style / conventions | Fix at discretion |

## Features

- Automatic audit loop with max 3 rounds to prevent infinite cycles
- Priority-ordered review: Bugs > Security > Performance > Style
- Resume Codex sessions for deeper discussion on ambiguous findings
- Critical evaluation — Opus treats Codex as a colleague, not an authority
- Bilingual triggers (English + Chinese)

## Requirements

- [Claude Code](https://claude.com/claude-code)
- [Codex CLI](https://github.com/openai/codex) (`npm i -g @openai/codex`)
- Git repository

## Installation

Copy the skill to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/codex-code-review
cp SKILL.md ~/.claude/skills/codex-code-review/SKILL.md
```

## Usage

Trigger the skill in Claude Code with any of these phrases:

| English | 中文 |
|---------|------|
| `codex review` | `codex 审查` |
| `codex code review` | `codex 检查` |
| `codex check` | `用 codex 审` |
| `let codex check` | `让 codex 看看` |
| `write and audit code` | `写完让 codex 检查` |
| `code review with codex` | `帮我写完代码然后审一下` |
| `dual AI review` | |
| `write then push with review` | |

## Default Config

| Setting | Default |
|---------|---------|
| Model | gpt-5.4 |
| Reasoning Effort | high |

Confirm once at the start of each session. Change anytime by asking.

## License

MIT
