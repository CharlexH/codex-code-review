[English](README.md) | [中文](README.zh-CN.md)

# codex-code-review

一个 Claude Code 技能，编排 **Opus 写码 → Codex 审计 → Opus 修复 → Codex 确认 → 推送** 的完整循环，确保代码经过 Codex 验证后再部署。

## 工作流程

```
阶段 1         阶段 2          阶段 3         阶段 4          阶段 5
Opus 写码 → Codex 审计 → Opus 修复 → Codex 再次审计 → 推送
  + 提交       (review)      + 提交        (确认)         (征求同意)
```

1. **Opus** 实现功能或修复，提交代码
2. **Codex** 审查 diff（`codex review --base <分支>`），检查 Bug、安全、性能和代码风格
3. **Opus** 修复所有发现的问题，再次提交
4. **Codex** 再次审计，确认修复无误（最多 3 轮）
5. 经用户确认后推送代码

## 审计优先级

| 优先级 | 类别 | 处理方式 |
|--------|------|----------|
| 1 | Bug / 逻辑错误 | 必须修复 |
| 2 | 安全问题 | 必须修复 |
| 3 | 性能问题 | 合理范围内修复 |
| 4 | 代码风格 / 规范 | 酌情修复 |

## 特性

- 自动审计循环，最多 3 轮，防止无限循环
- 按优先级审查：Bug > 安全 > 性能 > 风格
- 可恢复 Codex 会话，对模糊发现进行深入讨论
- 批判性评估 — Opus 将 Codex 视为同事而非权威，不盲从
- 双语触发词支持（中文 + 英文）

## 前置要求

- [Claude Code](https://claude.com/claude-code)
- [Codex CLI](https://github.com/openai/codex)（`npm i -g @openai/codex`）
- Git 仓库

## 安装

将技能复制到 Claude Code 技能目录：

```bash
mkdir -p ~/.claude/skills/codex-code-review
cp SKILL.md ~/.claude/skills/codex-code-review/SKILL.md
```

## 使用方法

在 Claude Code 中用以下任意短语触发：

| 中文 | English |
|------|---------|
| `codex 审查` | `codex review` |
| `codex 检查` | `codex code review` |
| `用 codex 审` | `codex check` |
| `让 codex 看看` | `let codex check` |
| `写完让 codex 检查` | `write and audit code` |
| `帮我写完代码然后审一下` | `code review with codex` |
| | `dual AI review` |
| | `write then push with review` |

## 默认配置

| 设置 | 默认值 |
|------|--------|
| 模型 | gpt-5.4 |
| 推理强度 | high |

每次会话开始时确认一次，随时可以要求更改。

## 许可证

MIT
