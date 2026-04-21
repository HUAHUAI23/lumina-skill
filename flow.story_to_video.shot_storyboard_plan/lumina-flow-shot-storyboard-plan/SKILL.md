---
name: lumina-flow-shot-storyboard-plan
description: flow.story_to_video.shot_storyboard_plan 的补充提示词；用于规划单镜头分镜图策略、融图方式与 prompt
---

# flow.story_to_video.shot_storyboard_plan

- 只为单个 shot 规划分镜图方案，不生成任务 graph。
- 基于 requirement 阶段选出的资产和主参考图决定策略。
- 优先选择最少但足够的输入图组合。
- 输出必须说明使用什么策略，例如：
  - `text_to_image`
  - `character_only`
  - `scene_only`
  - `prop_only`
  - `character_scene`
  - `character_prop`
  - `scene_prop`
  - `character_scene_prop`
- 输出只允许包含：
  - `strategy`
  - `reasoning`
  - `prompt`
  - `negativePrompt`
  - `inputKeyRoles`
- `inputKeyRoles[].role` 只能从以下集合中选择：
  - `character`
  - `scene`
  - `prop`
  - `style`
- `inputKeyRoles[].identityKey` 必须引用 requirement 和项目上下文里真实存在的 key。

## 规划原则

- 先保证人物和场景连续性，再考虑装饰性输入图。
- prompt 要描述这个 shot 的视觉目标，而不是复述整个故事。
- 如果输入图不足以支持复杂融图，主动收敛方案。

## 禁止事项

- 不创建新资产。
- 不在这里分配任务 id。
