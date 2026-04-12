---
name: hany-auto
description: "自动化编排。触发：/hany:auto。根据用户描述自动调用 hany-require → hany-implement → hany-verify-small，渐进式加载+验证修复循环。"
author: hany
version: "1.2"
---

# hany-auto — 自动化编排

**触发方式：`/hany:auto`**

> **核心原则：用户给一个描述，自动走完需求确认→实现→验证全流程。渐进式加载 skill 减少上下文膨胀。关键节点汇报，用户可随时介入。**

**快速路径**：简单任务自动判断复杂度后走 minimal 模式。具体步骤：`Step 0→1→2(require: Step 0→1简化→5→10→11)→2.5确认→3(implement: Step 0→1→3→4→7→8→10→11)→3.5确认→4(verify-small: Step 0→1→2→3→4)→4.4`。跳过 Constitutional 门禁、计划自检、安全检查、回归验证和边界检查。

## 强制交互规则

> - **禁止**跳过 require→implement→verify 顺序；每个阶段间必须汇报+等待确认
> - 每阶段完成后质疑审查："你确定这真的对吗？"（各阶段质疑侧重点不同）
> - 发现的错误必须抽象成规则写入 `docs/hany/rules.md`
> - 用户提出需求变更 → 暂停 → 评估影响 → `AskUserQuestion` 确定处理方式

## 质量模式

从 `.hany/config.json` 的 `quality` 字段读取，默认 `standard`。

| 模式 | 行为 |
|------|------|
| **strict** | 每阶段详细确认，验证循环无上限，全部质量检查 |
| **standard**（默认） | 每阶段确认一次，验证循环最多 3 轮 |
| **minimal** | 简单任务加速，验证循环最多 1 轮 |

> **编码约束**：读取 `hany-rules/SKILL.md`，11 条规则全程生效。

## 概览（必读）

> **[按需加载]** 本编排文件按 3 个阶段分发子 skill。执行时：
> 1. **启动阶段**：仅读"工作流程"和"Step 0→1"即可开始
> 2. **每个阶段**：先读子 skill 的 SUMMARY.md（~300 token），再按需读 SKILL.md 对应章节
> 3. **阶段完成后**：用 3 行摘要替代详细内容，释放上下文
>
> ⚠️ **铁律：禁止一次性读取子 skill 的完整 SKILL.md。** 只读 SUMMARY.md + 当前 Step。

### 快速参考卡

| 阶段 | 调用的子 skill | 加载方式 | 确认节点 |
|------|--------------|---------|---------|
| Step 0-1 | — | 直接执行 | AskUserQuestion |
| Step 2 | hany-require | SUMMARY.md → 按需读 SKILL.md | 2.5 过渡确认 |
| Step 3 | hany-implement | SUMMARY.md → 按需读 SKILL.md | 3.5 过渡确认 |
| Step 4 | hany-verify-small | 直接加载 | 最终确认 |

### 子 skill 文件路径参考

| 子 skill | 摘要文件 | 完整手册 |
|----------|---------|---------|
| hany-require | `hany-require/SUMMARY.md` | `hany-require/SKILL.md` |
| hany-implement | `hany-implement/SUMMARY.md` | `hany-implement/SKILL.md` |
| hany-verify-small | —（直接加载） | `hany-verify-small/SKILL.md` |
| hany-verify-project | `hany-verify-project/SUMMARY.md` | `hany-verify-project/SKILL.md` |
| hany-rules | —（太短，直接读） | `hany-rules/SKILL.md` |

## 恢复检查

启动时检查 `docs/hany/` 下是否有未完成的流程记录（需求文档/计划文档/验证报告）。如果有，使用 `AskUserQuestion` 询问：继续上次 / 重新开始。

## 工作流程

```
触发 → 恢复检查 → 先想想看 → 解析用户描述 → 渐进式加载skill → 阶段1：hany-require → 汇报→确认 → 阶段2：hany-implement → 汇报→确认 → 阶段3：hany-verify-small → 汇报 → 最终确认
```

**进度指示**：全流程和每个阶段都显示百分比进度。

---

## Step 0：先想想看 [进度 2%]

**触发后，先思考再执行。**

1. 用户给的任务描述是什么？
2. 这个任务属于什么类型（需求/实现/测试/综合）？
3. 需要调用哪些 skill？
4. 有没有上下文可以推导用户意图？

> 思考完后，输出思路再进入 Step 1。

## Step 1：解析用户描述 [进度 5%]

1. **读取描述**：理解用户要做什么
2. **判断复杂度**：
   - **简单**（单文件、小功能）→ 加速模式
   - **中等**（多文件、有依赖）→ 正常流程
   - **复杂**（新模块、架构变更）→ 详细流程
3. **探索上下文**：读取项目结构、关键文件、git 历史
4. **输出任务概要**

使用 `AskUserQuestion` 确认：确认开始 / 调整描述 / 取消。

---

## Step 2：阶段 1 — 渐进式加载 hany-require [进度 10-40%]

**渐进式加载策略**：先读摘要了解流程，需要时再读完整 SKILL.md。不一次性加载所有 skill 到上下文。

### 2.1 加载

```
先读 `hany-require/SUMMARY.md`（~300 token）了解全局。
然后进入 Step N 时，用 Read(offset/limit) 跳读 `hany-require/SKILL.md` 对应章节。
⚠️ 禁止一次性读取整个 SKILL.md。
```

### 2.2 自动执行

按 hany-require 完整流程执行，带进度汇报：
- `[require 进度 X%]` 每个 Step 完成时汇报

### 2.3 拆分后的并行处理

> 如果触发拆分，按公共规则处理（见文件末尾"拆分并行处理规则"）。

### 2.4 摘要压缩

require 阶段完成后，用 3 行摘要替代详细内容：

```
[require 摘要] 需求：[一句话描述]。需求文档：docs/hany/requirements/XXX.md。选中方案：[方案名]。歧义评分：[最终综合分]。待澄清：[剩余标记数]。
```

### 2.5 阶段过渡确认

使用 `AskUserQuestion` 汇报需求结果 + 示例原型。确认：进入实现 / 修改 / 暂停。

---

## Step 3：阶段 2 — 渐进式加载 hany-implement [进度 40-75%]

### 3.1 加载

```
先读 `hany-implement/SUMMARY.md`（~300 token）了解全局。
然后进入 Step N 时，用 Read(offset/limit) 跳读 `hany-implement/SKILL.md` 对应章节。
⚠️ 禁止一次性读取整个 SKILL.md。Implement 阶段完成后，用 3 行摘要替代。
```

### 3.2 自动执行

按 hany-implement 完整流程执行（含 Constitutional 门禁、PRD 故事拆解、Agent 分配、完成前验证门禁），带进度汇报。

### 3.3 拆分后的并行处理

> 如果触发拆分，按公共规则处理（见文件末尾"拆分并行处理规则"）。

### 3.4 摘要压缩

implement 阶段完成后，用 3 行摘要替代详细内容：

```
[implement 摘要] 变更文件：[清单]。计划文档：docs/hany/plans/XXX.md。状态：完成/部分完成。验收通过：[X/Y故事]。
```

### 3.5 阶段过渡确认

使用 `AskUserQuestion` 汇报实现结果。确认：进入验证 / 补充 / 暂停。

---

## Step 4：阶段 3 — 渐进式加载 hany-verify-small [进度 75-95%]

### 4.1 加载

```
直接加载 `hany-verify-small/SKILL.md`（~100 token）。
Verify 阶段完成后，用 3 行摘要替代。
```

### 4.2 自动执行

按 hany-verify-small 完整流程执行。

### 4.3 拆分后的并行处理

> 如果触发拆分，按公共规则处理（见文件末尾"拆分并行处理规则"）。

```
验证 → 发现问题 → 汇报（边找边报）→ 修复 → 重新验证 → 全部通过？
  ├── 是 → 进入最终汇报
  └── 否 → 继续循环
```

每轮循环汇报：`[验证循环 第X轮] 通过：X项，失败：Y项`

### 4.4 最终汇报

```markdown
## 全流程完成 [进度 100%]

### 阶段 1：需求确认 ✅
[需求摘要]

### 阶段 2：详细实现 ✅
[实现摘要]

### 阶段 3：检查测试 ✅/❌
[验证摘要]

### 发现并修复的问题
[问题列表，含根因分析]

### 优化建议
[如有]

### 归档
- 需求：`docs/hany/requirements/...`
- 计划：`docs/hany/plans/...`
- 报告：`docs/hany/archive/...`
```

---

## 渐进式加载策略

| 阶段 | 初始加载 | 执行时加载 | 压缩 |
|------|---------|-----------|------|
| Step 2 (require) | SUMMARY.md（~300 token） | 按需读 SKILL.md 对应章节 | 完成后用 3 行摘要替代 |
| Step 3 (implement) | SUMMARY.md（~300 token） | 按需读 SKILL.md 对应章节 | 完成后用 3 行摘要替代 |
| Step 4 (verify) | 直接加载（~100 token） | 直接加载 SKILL.md | - |

> **Token 优化**：每个阶段先加载 ~300 token 的摘要文件，仅在执行具体步骤时按需读取完整 SKILL.md 的对应章节。对比之前的"读取完整 SKILL.md 注入上下文"方案，初始加载从 ~9000 token 降到 ~900 token，减少约 90%。

## 复杂度自适应

| 复杂度 | 确认次数 | 提问深度 | 并行度 | 验证循环 |
|--------|---------|---------|--------|---------|
| 简单 | 每阶段 1 次 | 快速收敛 | 直接实现 | 1 轮 |
| 中等 | 每阶段 1 次 | 正常三轮 | 2-3 Agent | 3 轮 |
| 复杂 | 每阶段 2-3 次 | 深度多轮+挑战者 | 多 Agent | 不限 |

---

## 拆分并行处理规则（公共）

需求拆分为多个子系统后，三个阶段统一遵循：

1. 每个子系统独立进入对应阶段流程
2. 无依赖的子系统用 Agent/Team **并行执行**
3. 有依赖的子系统按依赖顺序执行（被依赖方先完成）
4. 统一进度汇报：`[阶段进度] 子系统A：X% / 子系统B：Y% / 总体：Z%`
