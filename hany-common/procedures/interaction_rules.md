---
name: interaction-rules
description: "强制交互规则统一版本。各skill直接引用。"
---

# 强制交互规则

> 所有确认步骤必须使用 `AskUserQuestion` 或 `EnterPlanMode`。禁止在文本中列选项等待回复，禁止跳过确认，禁止合并多个确认为一次交互。违反视为流程失败。
