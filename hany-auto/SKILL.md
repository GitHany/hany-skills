---
name: hany-auto
description: "自动化编排。触发：/hany:auto。根据用户描述自动调用 hany-require → planreview → hany-implement → 智能验证路由(verify-small/verify-project)，渐进式加载+验证修复循环+错误回退。"
author: hany
version: "1.3"
---

# hany-auto — 自动化编排

**触发方式：`/hany:auto`**

> **核心原则：用户给一个描述，自动走完需求确认→实现→验证全流程。渐进式加载 skill 减少上下文膨胀。关键节点汇报，用户可随时介入。**

**快速路径**：简单任务自动判断复杂度后走 minimal 模式。具体步骤：`Step 0→1→2(require)→2.5(planreview: 简单任务跳过)→2.7(任务分配检查: 可选)→2.8(过渡确认)→3(implement)→3.5(过渡确认)→4(智能验证路由)→4.6(最终汇报)`。跳过 Constitutional 门禁、计划自检、安全检查、回归验证和边界检查。

## 强制交互规则

> - **禁止**跳过 require→implement→verify 顺序；每个阶段间必须汇报+等待确认
> - 每阶段完成后质疑审查："你确定这真的对吗？"（各阶段质疑侧重点不同）
> - 发现的错误必须抽象成规则写入 `docs/hany/rules.md`
> - 用户提出需求变更 → 暂停 → 评估影响 → `AskUserQuestion` 确定处理方式

## 质量模式

> 详见 `hany-common/procedures/quality_modes.md`

> **编码约束**：读取 `hany-common/rules/rules.md`，11 条规则全程生效。

## 概览（必读）

> **[按需加载]** 本编排文件按 4 个阶段分发子 skill。执行时：
> 1. **启动阶段**：仅读"工作流程"和"Step 0→1"即可开始
> 2. **每个阶段**：先读子 skill 的 SUMMARY.md（~300 token），再按需读 SKILL.md 对应章节
> 3. **阶段完成后**：用 3 行摘要替代详细内容，释放上下文
>
> ⚠️ **铁律：禁止一次性读取子 skill 的完整 SKILL.md。** 只读 SUMMARY.md + 当前 Step。

### 标准加载策略

> 各阶段加载子 skill 时统一执行：
> 1. 先读 `<子skill>/SUMMARY.md`（~300 token）了解全局
> 2. 进入具体 Step 时，用 Read(offset/limit) 跳读 `<子skill>/SKILL.md` 对应章节
> 3. ⚠️ 禁止一次性读取整个 SKILL.md
>
> 下文各阶段以「按标准加载策略加载 XXX」引用此规则。

### 快速参考卡

| 阶段 | 调用的子 skill | 加载方式 | 确认节点 |
|------|--------------|---------|---------|
| Step 0-1 | — | 直接执行 | AskUserQuestion |
| Step 1.5 | hany-question（可选） | 直接加载 | 1.6 过渡确认 |
| Step 2 | hany-require | SUMMARY.md → 按需读 SKILL.md | 2.5 planreview |
| Step 2.5 | hany-planreview | SUMMARY.md → 按需读 SKILL.md | 2.7 task 检查 |
| Step 2.7 | hany-task（可选） | SUMMARY.md → 按需读 SKILL.md | 2.8 过渡确认 |
| Step 2.8 | — | — | 过渡确认（含 task 结果） |
| Step 3 | hany-implement | SUMMARY.md → 按需读 SKILL.md | 3.5 过渡确认 |
| Step 4 | 智能路由：verify-small / verify-project | 按路由加载 | 验证循环+最终确认 |

### 子 skill 文件路径参考

| 子 skill | 摘要文件 | 完整手册 |
|----------|---------|---------|
| hany-require | `hany-require/SUMMARY.md` | `hany-require/SKILL.md` |
| hany-question | `hany-question/SUMMARY.md` | `hany-question/SKILL.md` |
| hany-planreview | `hany-planreview/SUMMARY.md` | `hany-planreview/SKILL.md` |
| hany-implement | `hany-implement/SUMMARY.md` | `hany-implement/SKILL.md` |
| hany-verify-small | `hany-verify-small/SUMMARY.md` | `hany-verify-small/SKILL.md` |
| hany-verify-project | `hany-verify-project/SUMMARY.md` | `hany-verify-project/SKILL.md` |
| hany-task | `hany-task/SUMMARY.md` | `hany-task/SKILL.md` |
| hany-rules | `hany-common/rules/rules.md` | — |

## 概览工作流程

见上方快速参考卡（阶段列即流程）。**进度指示**：全流程和每个阶段都显示百分比进度。

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

## Step 1.5：问题澄清（可选）[进度 7%]

**仅在用户描述模糊时触发**（< 10 字或含"处理一下"/"优化一下"/"改改"/"差不多"）。

1. 加载 `hany-question/SKILL.md`，按质量模式执行问题澄清流程
2. 歧义评分达标后生成问题草稿
3. 用 `AskUserQuestion` 确认：草稿准确 → 进入 Step 2 / 需要修改 → 继续追问 / 跳过 → 直接进入 Step 2

**不触发条件**：用户描述已包含明确的目标、输入、输出 → 直接进入 Step 2。

## Step 2：阶段 1 — 渐进式加载 hany-require [进度 10-40%]

### 2.1 加载

按标准加载策略加载 `hany-require`。

### 2.2 自动执行

按 hany-require 完整流程执行，带进度汇报：
- `[require 进度 X%]` 每个 Step 完成时汇报

### 2.3 拆分后的并行处理

> 如果触发拆分，按公共规则处理（见 `hany-common/procedures/split_parallel.md`）。

### 2.4 摘要压缩

require 阶段完成后，用 3 行摘要替代详细内容：

```
[require 摘要] 需求：[一句话描述]。需求文档：docs/hany/requirements/XXX.md。选中方案：[方案名]。歧义评分：[最终综合分]。待澄清：[剩余标记数]。
```

## Step 2.5：需求文档评审（hany-planreview） [进度 42%]

**需求文档创建后、进入实现前，自动执行需求评审。**

> 简单任务（minimal 模式）自动跳过此步。

#### 2.5.1 加载

按标准加载策略加载 `hany-planreview`。

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

```
[planreview 摘要] 评审文档：[文档路径]。发现问题：X 个（高/Y、中/Z、低/W）。优化建议：[关键建议数]。评审结论：通过/有条件通过/不通过。
```

## Step 2.7：任务分配检查（可选）[进度 45%]

**仅在涉及多 Agent 协作或任务分配场景时触发**（需求中包含"分配"、"协作"、"多Agent"、"任务调度"等关键词）。

### 2.7.1 加载

先读 `hany-task/SUMMARY.md`（~150 token），再按需读 `hany-task/SKILL.md` 对应章节。

### 2.7.2 执行

按 hany-task 流程执行任务分配检查：
1. 收集 Agent 列表、任务列表、任务属性
2. 工作负载分析（极差、方差）
3. 依赖关系与时序分析（关键路径、阻塞点）
4. 优先级评估
5. 生成优化建议

### 2.7.3 检查结果处理

```
检查 → 发现问题？
  ├── 严重失衡/阻塞 → 用 AskUserQuestion 确认：优化分配方案 / 重新设计任务结构
  ├── 轻微问题 → 汇报 + 用 AskUserQuestion 确认：接受当前方案 / 调整优化
  └── 无问题 → 自动进入 Step 2.8
```

### 2.7.4 摘要压缩

```
[hany-task 摘要] 检查 Agent 数：N。任务数：M。负载均衡：{均衡/轻微失衡/严重失衡}。关键路径：{路径描述}。优化建议：{建议数}。
```

---

## Step 2.8：阶段过渡确认 [进度 48%]

使用 `AskUserQuestion` 汇报需求结果 + planreview 评审结果 + task 检查结果（如有）+ 示例原型。确认：进入实现 / 修改需求 / 重新评审 / 暂停。

---

## Step 3：阶段 2 — 渐进式加载 hany-implement [进度 50-80%]

### 3.1 加载

按标准加载策略加载 `hany-implement`。阶段完成后，用 3 行摘要替代。

### 3.2 自动执行

按 hany-implement 完整流程执行（含 Constitutional 门禁、PRD 故事拆解、Agent 分配、完成前验证门禁），带进度汇报。

### 3.3 摘要压缩

```
[implement 摘要] 变更文件：[清单]。计划文档：docs/hany/plans/XXX.md。状态：完成/部分完成。验收通过：[X/Y故事]。
```

### 3.5 阶段过渡确认

使用 `AskUserQuestion` 汇报实现结果。确认：进入验证 / 补充 / 暂停。

---

## Step 4：阶段 3 — 智能验证路由 [进度 80-98%]

### 4.1 验证路由判断

**根据变更范围自动选择 verify-small 或 verify-project。**

| 条件 | 验证路径 |
|------|---------|
| 单文件变更 + 小修复（<50行） | verify-small |
| 2-3 文件 + 同模块/同功能 | verify-small |
| 3+ 文件跨模块变更 | verify-project |
| 新增文件/模块/架构变更 | verify-project |
| hany-require 中标记为复杂任务 | verify-project |

> 需求复杂度 + 实际变更范围综合判断。不确定时默认走 verify-project。

### 4.2 渐进式加载

**verify-small 路径**：直接加载 `hany-verify-small/SKILL.md`（~100 token）。

**verify-project 路径**：按标准加载策略加载 `hany-verify-project`。

### 4.3 自动执行

按路由选择的 verify 流程完整执行。

**verify-small**：编译检查 → 功能验证 → 修复确认
**verify-project**：前置检查 → 质量清单 → 3D验证 → 多视角并行审查 → 功能验证 → 回归验证 → 边界检查 → 修复循环

### 4.4 拆分后的并行处理

> 如果触发拆分，按公共规则处理（见 `hany-common/procedures/split_parallel.md`）。

### 4.5 验证/修复循环 + 错误回退

> 详见 `hany-common/procedures/error_backoff.md`

每轮循环汇报：`[验证循环 第X轮] 路由：verify-small/verify-project。通过：X项，失败：Y项`

### 4.6 最终汇报

```markdown
## 全流程完成 [进度 100%]

### 阶段 1：需求确认 ✅
[需求摘要]

### 阶段 1.5：问题澄清 ✅/⏭️（可选）
[question 草稿摘要，仅在触发时显示]

### 阶段 2：需求评审 ✅/⚠️
[planreview 评审摘要]

### 阶段 2.5：任务分配检查 ✅/⏭️（可选）
[hany-task 检查摘要，仅在触发时显示]

### 阶段 3：详细实现 ✅
[实现摘要]

### 阶段 4：检查测试 ✅/❌
[验证摘要，含路由：verify-small/verify-project]

### 回退记录（如有）
[回退次数、回退原因、修复方式]

### 发现并修复的问题
[问题列表，含根因分析]

### 归档
- 需求：`docs/hany/requirements/...`
- 计划：`docs/hany/plans/...`
- 报告：`docs/hany/archive/...`
```

## 渐进式加载策略

> Token 优化：每个阶段先加载 ~300 token 的摘要文件，仅在执行具体步骤时按需读取完整 SKILL.md 的对应章节。Simple 模式跳过 planreview，verify-small 路径直接加载 ~100 token。
