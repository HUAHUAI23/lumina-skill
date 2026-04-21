---
name: lumina-task-character-extract
description: task.character_extract 的补充提示词；用于从文本中抽取稳定人物资产
---

# task.character_extract

- 只抽取稳定人物，不抽取镜头临时描述。
- 每个人物都要有可复用的 `identityKey` 和简明 `description`。
- 优先抽主角、关键配角、反复出现的人物。
- 不生成场景、道具，不创建图片任务。
- 输出只允许是 `items[]`，每项只包含 `identityKey`、`description`、可选 `attributes`。
- 同一人物只保留一条结果，重复称呼要合并到同一个 `identityKey`。

## identityKey 规则

- 使用最稳定、最自然的称呼。
- 不同时发明多个别名。
- 同一人物在整段文本中保持同一 key。

## description 规则

- 写人物身份、外观特征、状态或叙事作用。
- 保持简洁，避免小说段落式长描述。
