---
name: hany-common
description: "hany skill 公共层：共享规则库、公共流程模式、公共配置。所有 hany-* skill 共享。"
author: hany
version: "1.0"
---

# hany-common — 公共层

**用途**：所有 hany-* skill 共享的规则、流程模式和质量配置。

## 目录结构

```
hany-common/
├── SKILL.md                          # 本文件：公共层索引
├── rules/
│   └── rules.md                      # 11 条编码规则（所有 skill 全程生效）
└── procedures/
    ├── quality_modes.md              # 质量模式配置（strict/standard/minimal）
    ├── split_parallel.md             # 拆分并行处理公共规则
    └── error_backoff.md              # 验证修复循环与错误回退规则
```

## 共享规则

**必读**：`rules/rules.md` — 11 条编码规则，所有 hany skill 全程生效。

引用方式：各 skill SKILL.md 中写 `> **编码约束**：读取 `hany-common/rules/rules.md`，11 条规则全程生效。`

## 共享流程模式

| 文件 | 内容 | 被引用 |
|------|------|--------|
| `procedures/quality_modes.md` | strict/standard/minimal 模式定义 | 所有 skill |
| `procedures/split_parallel.md` | 拆分后并行处理规则 | require, implement, verify-project |
| `procedures/error_backoff.md` | 验证循环+回退规则 | verify-project, auto |

## 按需加载

> **各 skill 执行时**，公共层内容只在被引用时才加载。不需要一次性加载所有公共文件。
