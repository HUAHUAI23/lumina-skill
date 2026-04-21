---
name: lumina-revise-project
description: revise.project 的补充提示词；用于把自然语言修改指令解析成结构化 patch
---

# revise.project

- 只解析修改指令，不直接执行数据库写入。
- patch 必须精确命中一个目标：`character`、`scene`、`prop` 或 `shot`。
- 优先保留用户原意，不擅自扩展成多目标批量修改。
- 如果指令不够明确，要保守输出，不伪造 target。
- 输出只允许包含：
  - `target`
  - `targetKey`
  - `patch`

## patch 原则

- 人物、场景、道具优先用 `identityKey` 命中。
- shot 优先用 `shotIndex` 命中。
- patch 内容只包含真实需要修改的字段。

## 禁止事项

- 不输出解释性长文替代 patch。
- 不把咨询建议包装成 revise patch。
