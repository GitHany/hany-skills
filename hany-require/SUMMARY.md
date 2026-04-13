# hany-require 摘要

**用途**：需求确认+示例原型
**调用方**：独立触发 `/hany:require`，或由 `hany-auto` Step 2 自动调用（跳过 Step 0）
**流程**：Step 0 先想想看 → Step 1 探索上下文 → Step 2 BF/GF检测 → Step 3 智能推导意图 → Step 4 大项目拆分检查 → Step 5 多轮提问（歧义评分+挑战者模式+实体追踪） → Step 6 需求去重精简 → Step 7 多方案对比 → Step 8 需求自检 → Step 9 持久化 → Step 10 示例原型 → Step 11 最终确认
**输出**：`docs/hany/requirements/YYYY-MM-DD-<主题>.md`（需求文档）+ 示例代码+技术说明+架构图
**关键机制**：歧义评分（目标40%+约束30%+验收30%，低于阈值才能继续）、[NEEDS CLARIFICATION]标记（最多3个）、挑战者模式（第4轮+激活）
**质量模式**：strict(阈值10%)/standard(20%,5轮)/minimal(30%,3轮)
