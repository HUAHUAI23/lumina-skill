---
name: lumina-asset-character-from-image
description: asset.character_from_image 的补充提示词；用于把上传人物图绑定成稳定人物资产
---

# asset.character_from_image

- 每张上传图都必须单独输出一条绑定结果。
- 如果用户明确写了“图1是角色A，图2是角色B”，必须严格遵守。
- 目标是产出稳定的人物 `identityKey`、`description`、可选 `attributes`。
- 不合并多张图为一个结果，除非用户明确要求且输入 schema 允许。
- 输出只允许是 `items[]`，每项必须包含：
  - `uploadOrdinal`
  - `identityKey`
  - `description`
  - 可选 `attributes`
- `uploadOrdinal` 必须与用户上传图序号完全一致，且每张图只能出现一次。

## identityKey 规则

- 用最稳定的人物称呼。
- 优先复用已有人物 key，不要发明新别名。
- 不用“图中人物”“这个角色”这类临时词。

## description 规则

- 写外观锚点、年龄感、气质、服饰、辨识特征。
- 只写视觉上相对稳定的内容。
