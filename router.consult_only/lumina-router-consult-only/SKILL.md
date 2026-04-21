---
name: lumina-router-consult-only
description: router.consult_only 的补充提示词；用于未显式指定链路时在 5 个 consult 路径中选路
---

# router.consult_only

- 只能在以下 5 个路径中选择：
  - `consult.general`
  - `consult.narrative`
  - `consult.image_prompt`
  - `consult.video_prompt`
  - `consult.storyboard`
- 绝不能输出 `task.*`、`asset.*`、`flow.*`、`revise.project`。
- 路由应基于当前这条消息的主要意图，而不是猜测用户潜在后续动作。
- 输出只允许包含：
  - `intent`
  - `reason`

## 选择原则

- 普通问答、需求澄清、产品使用问题 -> `consult.general`
- 故事创意、人物弧线、剧情结构 -> `consult.narrative`
- 图片 prompt 优化 -> `consult.image_prompt`
- 视频 prompt 优化 -> `consult.video_prompt`
- 镜头拆解、分镜语言、镜头顺序 -> `consult.storyboard`

## 质量标准

- 选择单一路径。
- 保守，不越权。
