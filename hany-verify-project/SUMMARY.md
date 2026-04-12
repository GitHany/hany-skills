# hany-verify-project 摘要

**用途**：项目级完整验证（大功能/架构调整/多文件改动）
**流程**：前置检查 → Agent独立验证 → 质量清单 → 3D验证(完整性/正确性/一致性) → 多视角并行审查(架构/安全/代码质量) → 功能验证 → 回归验证(Red-Green) → 边界检查 → 修复循环+根因分析 → 错误→规则抽象 → 优化反馈 → 报告 → 归档
**输出**：验证报告（质量清单+3D验证+多视角审查+问题修复记录+优化建议）+ 归档到 docs/hany/archive/
**关键机制**：3 Agent并行审查（Architect/Security/Reviewer）、Red-Green回归测试、根因分析表、反复错误强制记录、质疑审查（每轮验证后）
**质量模式**：strict(无上限)/standard(3轮,含边界检查)/minimal(1轮,跳过安全+回归)
**前置条件**：docs/hany/plans/ 和 docs/hany/requirements/ 下存在已确认文档
