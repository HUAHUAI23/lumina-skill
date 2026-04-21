---
name: lumina-task-scene-extract
description: task.scene_extract 的补充提示词；用于从文本中抽取稳定场景资产
---

# task.scene_extract

- 只抽取稳定场景，不抽取一次性镜头背景。
- 每个场景都要产出可复用的 `identityKey` 和 `description`。
- 优先抽故事里反复出现或对视觉连续性关键的地点。
- 不输出人物和道具。
- 输出只允许是 `items[]`，每项只包含 `identityKey`、`description`、可选 `attributes`。
- 同一地点或同一稳定空间不要重复拆多条结果。

## identityKey 规则

- 使用地点名或最稳定的场景称呼。
- 同一场景不要拆成多个轻微变体 key。

## description 规则

- 写清空间类型、时代气质、关键陈设和环境氛围。
- 只保留对后续视觉生成有用的信息。
