# hany-skills

Claude Code 自定义技能集，实现结构化的多阶段软件开发工作流。

## 工作流程

```
用户描述任务
    │
/hany:auto（编排器）
    │
    ├── /hany:require   → 需求文档 + 原型
    ├── /hany:implement → 计划文档 + 代码实现
    └── /hany:verify    → 验证报告 + 修复
```

每个阶段之间有确认门控，产出物存档至 `docs/hany/`。

## 技能说明

| 技能 | 触发方式 | 说明 |
|------|---------|------|
| **hany-auto** | `/hany:auto` | 主编排器，串联 require → implement → verify 三阶段，支持渐进式加载和断点续跑 |
| **hany-require** | `/hany:require` | 需求确认 + 原型。多轮深度访谈式提问，歧义评分，输出需求文档和可运行原型 |
| **hany-implement** | `/hany:implement` | 详细实现。拆解任务为用户故事，并行 Agent 执行，完成后验证并提交 |
| **hany-verify** | `/hany:verify` | 多视角验证。架构/安全/代码质量并行审查，verify→fix 循环直到全部通过 |
| **hany-rules** | 自动加载 | 11 条共享编码规则，贯穿所有阶段 |

## 配置

`.hany/config.json` 控制质量模式：

```json
{
  "quality": "standard",
  "version": "1.1"
}
```

- `strict` — 严格模式，完整验证流程
- `standard` — 标准模式（默认）
- `minimal` — 最小模式，跳过非关键步骤

## 安装

将项目目录添加为 Claude Code 的 skills 目录即可使用。

## License

MIT
