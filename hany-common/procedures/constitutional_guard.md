---
name: constitutional-guard
description: "Constitutional门禁检查表。implement skill引用。"
---

# Constitutional 门禁检查

> 实现前必须依照此表逐项检查。任何一项失败 → 阻断直到修复。

| 检查项 | 失败处理 |
|--------|---------|
| 需求文档存在？ | 缺文档 → 提示用户执行 `/hany:require` |
| 歧义评分达标？ | 仍有歧义 → 回到 `hany-require` 阶段继续澄清 |
| 示例代码可行？ | 代码不可行 → 要求重新生成或回到 `hany-require` 修改方案 |
| 技术栈合理？ | 不合理 → 修正技术方案或与用户沟通 |
| 无待澄清项？ | 仍有 `[NEEDS CLARIFICATION]` → 必须在实现前全部消除 |
