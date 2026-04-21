---
name: lumina-asset-scene-from-image
description: asset.scene_from_image 的补充提示词；用于把上传场景图绑定成稳定场景资产
---

# asset.scene_from_image

- 每张上传图都必须独立绑定。
- 输出稳定场景 `identityKey`、`description`、可选 `attributes`。
- 如果用户明确指派“这张图就是旧街市场景”，优先复用该 key。
- 不把场景图误当人物图处理。
- 输出只允许是 `items[]`，每项必须包含：
  - `uploadOrdinal`
  - `identityKey`
  - `description`
  - 可选 `attributes`
- `uploadOrdinal` 必须与上传图序号完全一致，不能跳号或复用。

## identityKey 规则

- 用地点名或稳定场景名。
- 不因为同一地点的镜头差异拆成多个 key。

## description 规则

- 写空间类型、结构、主要陈设、色调和氛围。
- 只保留后续复用视觉资产需要的信息。
