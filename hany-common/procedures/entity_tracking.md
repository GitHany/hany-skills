---
name: entity-tracking
description: 实体追踪公共模块。被 hany-require 和 hany-question 引用。
---

# 实体追踪公共模块

## 提取规则

从用户回答中提取：
- 大写开头的术语（如 TaskManager、UserService）
- 代码标识符（如 getUserById、api/v1/users）
- 技术名词（如 React、PostgreSQL、JWT）

## 分类

| 类型 | 判断标准 | 示例 |
|------|---------|------|
| core | 被多次提及、是问题/需求的主体 | Task、User、Order |
| supporting | 辅助概念 | Email、Token、Config |
| external | 外部系统/服务 | GitHub API、Stripe |

## 稳定性计算

- **稳定实体**：连续 2 轮出现
- **新实体**：本轮首次出现
- **漂移实体**：前一轮有、本轮消失
- **稳定率** = 稳定实体 / 总实体数

## 终止条件增强

歧义度 ≤ 阈值 **且** 稳定率 ≥ 70% → 可以终止追问
歧义度 ≤ 阈值 **但** 稳定率 < 70% → 再追 1 轮确认核心概念

## 展示格式

```
实体：{count} 个 | 稳定率：{ratio}% | 新：{new} | 漂移：{drift}
```
