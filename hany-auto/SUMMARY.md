# hany-auto 摘要

**用途**：自动化编排
**流程**：解析描述 → require → planreview → implement → 智能验证路由
**输出**：完整的需求文档、计划文档、验证报告
**关键机制**：渐进式加载子 skill（每阶段只读 SUMMARY.md + 当前 Step）、智能验证路由（verify-small/verify-project）、验证修复循环+错误回退
**质量模式**：strict/standard(默认)/minimal 三档，影响 planreview/验证路由/循环次数/回退上限
