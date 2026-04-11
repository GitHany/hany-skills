# hany-implement 摘要

**用途**：详细实现
**流程**：检查前置 → Constitutional门禁 → 文件结构映射 → PRD故事拆解+Agent分配 → 计划自检 → 结构化计划文档 → 确认 → 并行实现 → 完成前验证门禁（含质疑审查） → 提交协议 → 完成确认
**输出**：`docs/hany/plans/YYYY-MM-DD-<主题>.md`（计划文档）+ 实际代码变更
**关键机制**：Constitutional门禁（5项检查）、并行组分配（同一文件同一Agent）、Agent返回后独立验证、红旗词检测、错误→规则抽象
**前置条件**：docs/hany/requirements/ 下存在已确认的需求文档
**质量模式**：strict(TDD)/standard(正常验证)/minimal(快速实现)
