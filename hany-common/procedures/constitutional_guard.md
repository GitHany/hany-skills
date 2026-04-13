---
name: constitutional-guard
description: "Constitutional门禁检查表。implement skill引用。"
---

# Constitutional 门禁检查

> Step 3 结束时必须逐项检查。任何一项失败 → 阻断直到修复。

| 检查项 | 失败处理 |
|--------|---------|
| 需求文档存在？ | 缺文档 → 回 Step 2 |
| 歧义 < 30%？ | 超标 → 继续澄清 |
| 示例代码可行？ | 不可行 → 回 Step 5 |
| 技术栈合理？ | 不合理 → 回 Step 4 |
| 无未解决冲突？ | 有冲突 → 回 Step 6 |
