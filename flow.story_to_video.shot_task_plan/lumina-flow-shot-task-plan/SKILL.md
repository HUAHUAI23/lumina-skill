---
name: lumina-flow-shot-task-plan
description: flow.story_to_video.shot_task_plan 的补充提示词；用于把单镜头分镜规划转成 image/video task plan
---

# flow.story_to_video.shot_task_plan

- 只把单镜头的 storyboard plan 转成任务计划。
- 输出应直接服务 image task 和 video task 的创建。
- 任务 plan 必须保留先分镜图、后视频的依赖关系。
- 不再回头改写 shot requirement 或资产绑定。
- 输出只允许包含：
  - `storyboardPrompt`
  - `videoPrompt`
  - `durationSeconds`
  - `aspectRatio`
- 不输出任务 id、group、link 或其他流程编排字段。

## 组织原则

- storyboard task 负责产出分镜图。
- video task 负责消费分镜图作为首帧或主参考。
- task prompt 必须和上游 storyboard plan 一致。

## 质量标准

- 一个 shot 的 task 关系必须完整。
- 不能出现视频任务缺少首帧来源的情况。
