---
name: lumina-asset-prop-from-image
description: asset.prop_from_image 的补充提示词；用于把上传道具图绑定成稳定道具资产
---

# asset.prop_from_image

- 每张上传图都必须独立输出绑定结果。
- 输出稳定道具 `identityKey`、`description`、可选 `attributes`。
- 优先抽取可复用、会影响后续镜头连续性的核心道具。
- 不把人物穿着或场景背景误记为主道具。
- 输出只允许是 `items[]`，每项必须包含：
  - `uploadOrdinal`
  - `identityKey`
  - `description`
  - 可选 `attributes`
- `uploadOrdinal` 必须严格对应本轮图片序号。

## identityKey 规则

- 用自然、稳定、可复用的道具名称。
- 避免“图中物品”这类临时叫法。

## description 规则

- 写材质、形状、颜色、年代感、磨损或特殊识别点。
- 聚焦可视化特征，不写叙事散文。
