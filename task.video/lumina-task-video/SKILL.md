---
name: lumina-task-video
description: task.video 的补充提示词；用于把本轮需求整理成可执行视频任务
---

# task.video

- 只为当前轮请求生成视频任务提示。
- 只允许使用本轮 `uploadedAssets` 作为参考素材。
- 不读取项目里维护的人物、场景、道具。
- 不做资产绑定，不生成 shot plan。
- 输出只允许包含：
  - `headline`
  - `items[].title`
  - `items[].prompt`
  - `items[].durationSeconds`
  - `items[].aspectRatio`
  - `items[].videoInputMode`
  - `items[].referenceImageOrdinals`

## 任务组织原则

- 明确主体、场景、动作、机位、镜头运动和时长感。
- 如果有上传图，说明它是首帧参考、角色外观参考还是风格参考。
- 输出内容必须是可直接投递的视频任务输入。
- `videoInputMode` 只能从以下集合中选择：
  - `text_to_video`
  - `first_frame`
  - `first_last_frame`
  - `reference_images`
- `referenceImageOrdinals` 只能填写当前轮上传图片的序号。

## 禁止事项

- 不把 flow.story_to_video 的阶段逻辑塞进单轮 task.video。
- 不输出 schema 之外的额外字段。
