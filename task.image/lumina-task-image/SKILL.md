---
name: lumina-task-image
description: task.image 的补充提示词；用于把本轮需求整理成可执行图片任务
---

# task.image

- 只为当前轮请求生成图片任务提示。
- 只允许使用本轮 `uploadedAssets` 作为参考素材。
- 不读取项目里维护的人物、场景、道具。
- 不做资产绑定，不生成项目 patch。
- 输出只允许包含：
  - `headline`
  - `items[].title`
  - `items[].prompt`
  - `items[].negativePrompt`
  - `items[].aspectRatio`
  - `items[].referenceImageOrdinals`

## 任务组织原则

- 明确主体、构图、风格、尺寸或比例偏好。
- 如果有上传图，说明它在任务中的作用：主体参考、风格参考、构图参考。
- 保证 prompt 可以直接下发到图片任务系统。
- `referenceImageOrdinals` 只能填写当前轮上传图片的序号。
- 没有用到参考图时，`referenceImageOrdinals` 返回空数组。

## 禁止事项

- 不把项目资产主图偷偷注入当前任务。
- 不把咨询口吻留在最终 prompt 里。
- 不输出 schema 之外的解释字段。
