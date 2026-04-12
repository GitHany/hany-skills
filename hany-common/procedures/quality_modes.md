---
name: quality-modes
description: "质量模式配置。strict/standard/minimal 三档，被所有 skill 引用。"
---

# 质量模式配置

从 `.hany/config.json` 的 `quality` 字段读取，默认 `standard`。

## 各 Skill 质量模式对照

### hany-require

| 模式 | 歧义阈值 | 提问轮次 | 行为 |
|------|---------|---------|------|
| **strict** | 10% | 无上限 | 全部步骤严格执行 |
| **standard**（默认） | 20% | 最多 5 轮 | 正常流程 |
| **minimal** | 30% | 最多 3 轮 | 快速收敛 |

### hany-implement

| 模式 | 行为 |
|------|------|
| **strict** | 每步 TDD、完整验证、详细汇报 |
| **standard**（默认） | 正常验证、标准汇报 |
| **minimal** | 快速实现、最小验证 |

### hany-auto

| 模式 | planreview | 验证路由 | 验证循环 | 回退上限 |
|------|-----------|---------|---------|---------|
| **strict** | 必须执行 | 默认 verify-project | 无上限 | 无上限 |
| **standard**（默认） | 必须执行 | 自动判断 | 最多 3 轮 | 最多 2 次 |
| **minimal** | 跳过 | 默认 verify-small | 最多 1 轮 | 最多 1 次 |

### hany-verify-project

| 模式 | 验证循环上限 | 行为 |
|------|------------|------|
| **strict** | 无上限 | 全部步骤严格执行 |
| **standard**（默认） | 最多 3 轮 | 跳过多视角审查和回归验证，含边界检查 |
| **minimal** | 最多 1 轮 | 跳过安全检查和回归验证 |

### hany-rules

| 模式 | 规则执行力度 |
|------|------------|
| **strict** | 全部 11 条强制检查，违规即中断 |
| **standard**（默认） | 全部 11 条检查，违规需修正后继续 |
| **minimal** | 核心规则（1/2/6/9）强制，其余建议遵守 |
