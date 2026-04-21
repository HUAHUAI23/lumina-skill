---
name: lumina-flow-shot-storyboard-requirement
description: flow.story_to_video.shot_storyboard_requirement 的补充提示词；用于分析单镜头需要哪些参考图和缺失资产
---

# flow.story_to_video.shot_storyboard_requirement

- 只分析单个 shot 的分镜图需求。
- 输入会带项目资产摘要；只从这些人物、场景、道具里选。
- 需要明确：
  - 这个镜头是否需要参考图
  - 需要人物图、场景图、道具图中的哪些
  - 缺了哪些 identityKey
- 如果关键身份缺失，要把缺失内容准确写入 `missingIdentityKeys`。
- 输出只允许包含：
  - `primaryGoal`
  - `mustUseKeys`
  - `optionalKeys`
  - `missingIdentityKeys`
  - `framing`
  - `cameraMove`
  - `emotion`
  - `action`
  - `notes`
- 不输出 `blocked`、`status` 或其他 schema 外字段。

## 判断原则

- 角色外观连续性强时，优先需要人物图。
- 空间连续性强时，优先需要场景图。
- 情节依赖特定物件时，明确需要道具图。
- 没有合适参考图时，才允许退回纯文生分镜图。
