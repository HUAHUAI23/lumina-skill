---
name: lumina-flow-shot-split
description: flow.story_to_video.shot_split 的补充提示词；用于把故事拆成可执行 shot 列表
---

# flow.story_to_video.shot_split

- 目标是把原始叙事拆成连续 shot，不做资产创建。
- 只引用项目里已存在的人物、场景、道具 key。
- 若文本提到项目中不存在的关键身份，应在后续阶段阻塞，不在这里偷偷发明资产。
- shot 顺序必须反映真实叙事推进。
- 输出只允许包含：
  - `shots[].shotIndex`
  - `shots[].title`
  - `shots[].description`
  - `shots[].durationSeconds`
  - `shots[].aspectRatio`
  - `shots[].locationHint`
  - `shots[].referencedCharacterKeys`
  - `shots[].referencedSceneKeys`
  - `shots[].referencedPropKeys`
- `referenced*Keys` 只能填写当前项目里已有的 identityKey；没有就返回空数组，不要发明新 key。

## 拆分原则

- 每个 shot 只表达一个清晰动作或视觉目的。
- 长段落要按动作转折、时空变化、情绪变化切开。
- 相邻 shot 要保持叙事连续。

## 质量标准

- 不漏关键剧情节点。
- 不为凑数量硬切空镜头。
