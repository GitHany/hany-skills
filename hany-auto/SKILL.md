---
name: hany-auto
description: "自动化编排。触发：/hany:auto。根据用户描述自动调用 hany-require → planreview → hany-implement → 智能验证路由(verify-small/verify-project)，渐进式加载+验证修复循环+错误回退。"
author: hany
version: "1.3"
---

# hany-auto — 自动化编排

**触发方式：`/hany:auto`**

> **核心原则：用户给一个描述，自动走完需求确认→实现→验证全流程。渐进式加载 skill 减少上下文膨胀。关键节点汇报，用户可随时介入。**

**快速路径**：简单任务自动判断复杂度后走 minimal 模式。具体步骤：`Step 0→1→2(require)→2.5(planreview: 简单任务跳过)→2.7确认→3(implement)→3.5确认→4(智能验证路由)→4.6`。跳过 Constitutional 门禁、计划自检、安全检查、回归验证和边界检查。

**验证路由**：`Step 4` 根据变更范围自动选择：
- `verify-project`：多文件/架构变更/新模块 → 完整验证流程
- `verify-small`：单文件/小修复 → 快速验证
- 验证失败 → 评估严重程度 → 小问题就地修 / 根本性问题回退到 Step 3(implement)

## 强制交互规则

> - **禁止**跳过 require→implement→verify 顺序；每个阶段间必须汇报+等待确认
> - 每阶段完成后质疑审查："你确定这真的对吗？"（各阶段质疑侧重点不同）
> - 发现的错误必须抽象成规则写入 `docs/hany/rules.md`
> - 用户提出需求变更 → 暂停 → 评估影响 → `AskUserQuestion` 确定处理方式

## 质量模式

从 `.hany/config.json` 的 `quality` 字段读取，默认 `standard`。

| 模式 | planreview | 验证路由 | 验证循环 | 回退上限 |
|------|-----------|---------|---------|---------|
| **strict** | 必须执行 | 默认 verify-project | 无上限 | 无上限 |
| **standard**（默认） | 必须执行 | 自动判断 | 最多 3 轮 | 最多 2 次 |
| **minimal** | 跳过 | 默认 verify-small | 最多 1 轮 | 最多 1 次 |

> **编码约束**：读取 `hany-rules/SKILL.md`，11 条规则全程生效。

## 需求文档规范

> **必读**：本文档引用的需求文档格式规范详见 `hany-auto/requirements/需求文档模板.md`。
>
> 关键规则：
> - Step 2 阶段：需求文档写入 `docs/hany/requirements/`，摘要引用路径
> - Step 4.6 归档：必须列出需求文档路径
> - 恢复检查：Step 0 前检查是否存在 `in_progress` 状态文档
> - 拆分规则：多子系统时独立需求文档命名 `YYYY-MM-DD-<主题>-<子系统>.md`

## 概览（必读）

> **[按需加载]** 本编排文件按 4 个阶段分发子 skill。执行时：
> 1. **启动阶段**：仅读"工作流程"和"Step 0→1"即可开始
> 2. **每个阶段**：先读子 skill 的 SUMMARY.md（~300 token），再按需读 SKILL.md 对应章节
> 3. **阶段完成后**：用 3 行摘要替代详细内容，释放上下文
>
> ⚠️ **铁律：禁止一次性读取子 skill 的完整 SKILL.md。** 只读 SUMMARY.md + 当前 Step。

### 快速参考卡

| 阶段 | 调用的子 skill | 加载方式 | 确认节点 |
|------|--------------|---------|---------|
| Step 0-1 | — | 直接执行 | AskUserQuestion |
| Step 2 | hany-require | SUMMARY.md → 按需读 SKILL.md | 2.5 planreview |
| Step 2.5 | hany-planreview | SUMMARY.md → 按需读 SKILL.md | 2.7 过渡确认 |
| Step 3 | hany-implement | SUMMARY.md → 按需读 SKILL.md | 3.5 过渡确认 |
| Step 4 | 智能路由：verify-small / verify-project | 按路由加载 | 验证循环+最终确认 |

### 子 skill 文件路径参考

| 子 skill | 摘要文件 | 完整手册 |
|----------|---------|---------|
| hany-require | `hany-require/SUMMARY.md` | `hany-require/SKILL.md` |
| hany-planreview | `hany-planreview/SUMMARY.md` | `hany-planreview/SKILL.md` |
| hany-implement | `hany-implement/SUMMARY.md` | `hany-implement/SKILL.md` |
| hany-verify-small | —（直接加载） | `hany-verify-small/SKILL.md` |
| hany-verify-project | `hany-verify-project/SUMMARY.md` | `hany-verify-project/SKILL.md` |
| hany-rules | —（太短，直接读） | `hany-rules/SKILL.md` |

## 恢复检查

启动时检查 `docs/hany/` 下是否有未完成的流程记录（需求文档/计划文档/验证报告）。如果有，使用 `AskUserQuestion` 询问：继续上次 / 重新开始。

## 工作流程

```
触发 → 恢复检查 → 先想想看 → 解析用户描述 → 渐进式加载skill → 阶段1：hany-require → 汇报→确认 → 阶段1.5：hany-planreview(评审需求文档) → 确认→进入实现 → 阶段2：hany-implement → 汇报→确认 → 阶段3：智能验证路由 → 验证→修复循环(含错误回退) → 最终确认
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

### 2.5 需求文档评审（hany-planreview） [进度 35%]

**需求文档创建后、进入实现前，自动执行需求评审。**

> 简单任务（minimal 模式）自动跳过此步。

#### 2.5.1 加载

```
先读 `hany-planreview/SUMMARY.md`（~300 token）了解全局。
然后用 Read(offset/limit) 跳读 `hany-planreview/SKILL.md` 对应章节。
⚠️ 禁止一次性读取整个 SKILL.md。
```

#### 2.5.2 执行

按 hany-planreview 流程执行：定位需求文档 → 读取分析 → 逐条质疑评审 → 输出评审报告。

评审维度：完整性、一致性、可行性、清晰性、合理性、可测试性。

#### 2.5.3 评审结果处理

```
评审 → 发现问题？
  ├── 高严重程度 → 用 AskUserQuestion 确认：修复需求文档 / 重新进入 hany-require / 暂停
  ├── 中/低严重程度 → 汇报 + 用 AskUserQuestion 确认：接受修改 / 暂不处理
  └── 无问题 → 自动进入 Step 2.7
```

#### 2.5.4 摘要压缩

planreview 阶段完成后，用 3 行摘要替代详细内容：

```
[planreview 摘要] 评审文档：[文档路径]。发现问题：X 个（高/Y、中/Z、低/W）。优化建议：[关键建议数]。评审结论：通过/有条件通过/不通过。
```

### 2.7 阶段过渡确认

使用 `AskUserQuestion` 汇报需求结果 + planreview 评审结果 + 示例原型。确认：进入实现 / 修改需求 / 重新评审 / 暂停。

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

## Step 4：阶段 3 — 智能验证路由 [进度 75-95%]

### 4.1 验证路由判断

**根据变更范围自动选择 verify-small 或 verify-project。**

分析 `git diff` 和变更文件清单，判断验证路径：

| 条件 | 验证路径 |
|------|---------|
| 单文件变更 + 小修复（<50行） | verify-small |
| 2-3 文件 + 同模块/同功能 | verify-small |
| 3+ 文件跨模块变更 | verify-project |
| 新增文件/模块/架构变更 | verify-project |
| hany-require 中标记为复杂任务 | verify-project |

> 需求复杂度 + 实际变更范围综合判断。不确定时默认走 verify-project。

### 4.2 渐进式加载

**verify-small 路径**：
```
直接加载 `hany-verify-small/SKILL.md`（~100 token）。
Verify 阶段完成后，用 3 行摘要替代。
```

**verify-project 路径**：
```
先读 `hany-verify-project/SUMMARY.md`（~300 token）了解全局。
然后进入 Step N 时，用 Read(offset/limit) 跳读 `hany-verify-project/SKILL.md` 对应章节。
⚠️ 禁止一次性读取整个 SKILL.md。
```

### 4.3 自动执行

按路由选择的 verify 流程完整执行。

**verify-small**：编译检查 → 功能验证 → 修复确认
**verify-project**：前置检查 → 质量清单 → 3D验证 → 多视角并行审查 → 功能验证 → 回归验证 → 边界检查 → 修复循环

### 4.4 拆分后的并行处理

> 如果触发拆分，按公共规则处理（见文件末尾"拆分并行处理规则"）。

### 4.5 验证/修复循环 + 错误回退

```
验证 → 发现问题 → 评估严重程度
  ├── 小问题（编译错误/逻辑小错/边界遗漏）→ 就地修复 → 重新验证
  ├── 根本性问题（架构偏差/需求理解错误/多模块联动错误）→ 回退到 Step 3(re-implement)
  └── 全部通过 → 质疑审查 → 进入最终汇报
```

每轮循环汇报：`[验证循环 第X轮] 路由：verify-small/verify-project。通过：X项，失败：Y项`

#### 4.5.1 错误分类标准

| 类型 | 判断标准 | 处理方式 |
|------|---------|---------|
| 小问题 | 单点修复、不影响架构、无连锁反应 | 就地修复，继续验证 |
| 根本性问题 | 需求理解偏差、架构设计不合理、多模块联动失败 | 回退到 Step 3 |
| 需求级问题 | 需求本身有误/遗漏/矛盾 | 回退到 Step 2(hany-require) |

#### 4.5.2 回退到 Step 3 (re-implement)

**触发条件**：验证发现根本性问题，小修无法解决。

1. 整理问题清单 + 根因分析，汇报给用户
2. 用 `AskUserQuestion` 确认：回退到实现 / 跳过问题 / 暂停
3. 确认回退 → 读取原计划文档 + 验证报告 → 重新进入 Step 3(hany-implement)
4. re-implement 完成后 → 重新进入 Step 4 验证

**回退限制**：
- strict 模式：无上限，必须全部解决
- standard 模式：最多回退 2 次，第 3 次用 `AskUserQuestion` 让用户决定
- minimal 模式：最多回退 1 次，之后用 `AskUserQuestion` 让用户决定

#### 4.5.3 回退到 Step 2 (re-require)

**触发条件**：验证发现需求本身有问题（不是实现问题，是需求文档本身的错误/遗漏）。

1. 整理需求问题清单
2. 用 `AskUserQuestion` 确认：重新进入需求确认 / 修改需求文档 / 暂停
3. 确认 → 重新进入 Step 2(hany-require)，从问题点开始

#### 4.5.4 验证路由升级

如果初始走 verify-small 但发现的问题超出了小修复范围：

```
verify-small → 发现根本性问题？
  ├── 是 → 自动升级到 verify-project，重新执行完整验证
  └── 否 → 继续 verify-small 循环
```

> 升级时用 `AskUserQuestion` 通知用户：verify-small 发现超出范围的问题，自动升级到 verify-project。

### 4.6 最终汇报

```markdown
## 全流程完成 [进度 100%]

### 阶段 1：需求确认 ✅
[需求摘要]

### 阶段 1.5：需求评审 ✅/⚠️
[planreview 评审摘要]

### 阶段 2：详细实现 ✅
[实现摘要]

### 阶段 3：检查测试 ✅/❌
[验证摘要，含路由：verify-small/verify-project]

### 回退记录（如有）
[回退次数、回退原因、修复方式]

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
| Step 2.5 (planreview) | SUMMARY.md（~300 token） | 按需读 SKILL.md 对应章节 | 完成后用 3 行摘要替代 |
| Step 3 (implement) | SUMMARY.md（~300 token） | 按需读 SKILL.md 对应章节 | 完成后用 3 行摘要替代 |
| Step 4 (verify-small) | 直接加载（~100 token） | 直接加载 SKILL.md | - |
| Step 4 (verify-project) | SUMMARY.md（~300 token） | 按需读 SKILL.md 对应章节 | 完成后用 3 行摘要替代 |

> **Token 优化**：每个阶段先加载 ~300 token 的摘要文件，仅在执行具体步骤时按需读取完整 SKILL.md 的对应章节。Simple 模式跳过 planreview，verify-small 路径直接加载 ~100 token，总初始加载约 700-1200 token。

## 复杂度自适应

| 复杂度 | 确认次数 | planreview | 验证路由 | 提问深度 | 并行度 | 验证循环 | 回退 |
|--------|---------|-----------|---------|---------|--------|---------|------|
| 简单 | 每阶段 1 次 | 跳过 | verify-small | 快速收敛 | 直接实现 | 1 轮 | 1 次 |
| 中等 | 每阶段 1 次 | 必须 | 自动判断 | 正常三轮 | 2-3 Agent | 3 轮 | 2 次 |
| 复杂 | 每阶段 2-3 次 | 必须+深度评审 | verify-project | 深度多轮+挑战者 | 多 Agent | 不限 | 不限 |

---

## 拆分并行处理规则（公共）

需求拆分为多个子系统后，各阶段统一遵循：

1. 每个子系统独立进入对应阶段流程
2. 无依赖的子系统用 Agent/Team **并行执行**
3. 有依赖的子系统按依赖顺序执行（被依赖方先完成）
4. 统一进度汇报：`[阶段进度] 子系统A：X% / 子系统B：Y% / 总体：Z%`
