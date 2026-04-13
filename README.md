# hany-skills

结构化的多阶段软件开发技能集，让 AI 遵循"需求确认 → 详细实现 → 验证修复"的顺序工作，减少返工和遗漏。

## 快速开始

```bash
# 1. 描述你的任务
你：帮我加一个用户登录功能

# 2. 触发编排器，自动走完全流程
/hany:auto

# 3. 每个阶段结束后确认，AI 逐步推进
```

## 整体架构

```
用户描述任务
        │
   /hany:auto
        │
        ├─→ /hany:question（可选，描述模糊时）
        │
        ├─→ /hany:require  ──→ 需求文档 + 示例原型
        │                        docs/hany/requirements/YYYY-MM-DD-<主题>.md
        │
        ├─→ /hany:planreview ──→ 需求文档评审报告
        │
        ├─→ /hany:implement ──→ 计划文档 + 代码实现
        │                        docs/hany/plans/YYYY-MM-DD-<主题>.md
        │
        └─→ /hany:verify-*  ──→ 验证报告 + 修复
                                 docs/hany/archive/YYYY-MM-DD-<主题>.md
```

## 技能一览

| 技能 | 触发方式 | 说明 | 输出物 |
|------|---------|------|--------|
| **hany-auto** | `/hany:auto` | 主编排器，串联全流程。渐进式加载子 skill，关键节点暂停确认 | — |
| **hany-question** | `/hany:question` | 问题澄清。引导用户形成结构化技术问题描述，也可由 auto 自动触发 | `docs/hany/questions/*.md` |
| **hany-require** | `/hany:require` | 需求确认。多轮提问 + 歧义评分，输出可行方案 | `docs/hany/requirements/*.md` |
| **hany-planreview** | `/hany:planreview` | 需求评审。批判性审视需求文档，识别漏洞并优化 | — |
| **hany-implement** | `/hany:implement` | 详细实现。PRD 拆解 + Agent 并行，验证后提交 | `docs/hany/plans/*.md` + 代码 |
| **hany-verify-small** | `/hany:verify-small` | 小改动验证。编译 + 功能 + 修复确认 | `docs/hany/archive/*.md` |
| **hany-verify-project** | `/hany:verify-project` | 项目级验证。多视角并行审查 + 回归测试 | `docs/hany/archive/*.md` |
| **hany-rules** | 自动加载 | 11 条共享编码规则，全程生效（实体在 hany-common/rules/） | — |
| **hany-common** | 自动加载 | 公共层：规则库 + 流程模式 + 质量配置 + 模板 | — |

## 适用场景

| 适合 | 不适合 |
|------|--------|
| 新功能开发（从小功能到多子系统） | 纯搜索/研究任务（无代码产出） |
| Bug 修复（有验证闭环） | 单次问答（直接问就行） |
| 文档/配置改进（有产出物） | 紧急 hotfix（直接改，跳流程） |
| 重构（有计划有验证） | |

## 质量模式

`.hany/config.json` 控制严格程度：

```json
{ "quality": "standard" }
```

| 模式 | 歧义容忍 | 验证轮次 | 适用 |
|------|---------|---------|------|
| **minimal** | 30% | 1 轮 | 简单改动、快速验证 |
| **standard**（默认） | 20% | 3 轮 | 大部分场景 |
| **strict** | 10% | 不限 | 重要功能、架构变更 |

## 目录结构

```
hany-skills/
├── hany-auto/            # 主编排器
├── hany-question/        # 问题澄清（可选，描述模糊时触发）
├── hany-require/         # 需求阶段
├── hany-planreview/      # 需求评审（require 后、implement 前）
├── hany-implement/       # 实现阶段
├── hany-verify-small/    # 小改动验证
├── hany-verify-project/  # 项目级验证
├── hany-rules/           # 编码规则索引（实体在 hany-common/rules/）
├── hany-common/          # 公共层：规则 + 流程模式 + 质量配置 + 模板
└── README.md
```

## 产出物路径

所有产出物统一在项目根目录的 `docs/hany/` 下：

```
docs/hany/
├── requirements/        # 需求文档 + 示例原型
├── plans/               # 计划文档 + 任务拆解
├── questions/           # 结构化问题草稿
├── rules.md             # 项目专属编码规则（错误抽象积累）
└── archive/             # 验证报告 + 问题记录
```

## 安装

将本目录添加为 Claude Code 的 skills 目录：

```bash
# 在 claude_code_settings.json 中添加
"skills": {
  "dirs": ["path/to/hany-skills"]
}
```

## License

MIT
