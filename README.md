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

**One-command install via [skills.sh](https://skills.sh):**

```bash
npx skills add CharlexH/codex-code-review
```

**Or install manually:**

```bash
mkdir -p ~/.claude/skills/codex-code-review
curl -fsSL https://raw.githubusercontent.com/CharlexH/codex-code-review/main/SKILL.md \
  -o ~/.claude/skills/codex-code-review/SKILL.md
```

## Usage

Trigger the skill in Claude Code with any of these phrases:

| English | 中文 |
|---------|------|
| `codex review` | `codex 审查` |
| `codex check` | `用 codex 审` |
| `let codex check` | `让 codex 看看` |
| `review before push` | `push 前让 codex 审一下` |
| `run codex before push` | `提交前让 codex 过一遍` |

<details>
<summary>All supported triggers (20+)</summary>

**English:** codex review, codex code review, codex check, let codex check, code review with codex, review before push, pre-push review, review and fix with codex, run codex before push, codex audit loop

**中文:** codex 审查, codex 检查, 用 codex 审, 让 codex 看看, 写完让 codex 检查, push 前让 codex 审一下, 提交前让 codex 过一遍, 修完再让 codex 看一遍, 让 codex 复审一下, 帮我走一遍 codex 审查流程
</details>

## Default Config

| Setting | Default |
|---------|---------|
| Model | gpt-5.4 |
| Reasoning Effort | high |

Confirm once at the start of each session. Change anytime by asking.

## License

MIT
