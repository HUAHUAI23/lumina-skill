# Lumina Prompt Template 编写说明

> 更新时间：2026-04-29
> 对应运行时：`lib/shared/ai-skill-domains.ts`、`lib/server/ai-chat/handlers/index.ts`、`scripts/import-lumina-skills*.ts`

## 1. 总原则

Lumina 的外部提示词模板只负责补充“这个处理步骤应该怎么做”，不负责定义“程序如何接收与消费结果”。

程序内部固定的内容包括：

- 路由决策
- 结构化输出协议
- 校验规则
- 工具调用约束
- 项目读写职责
- flow 阶段编排

外部 `SKILL.md` 只允许补充：

- 负责什么
- 不负责什么
- 判断原则
- 工作方法
- 质量标准
- 失败回退

## 2. 提示词组成

实际给模型的提示词由三部分组成：

1. 程序系统指令
2. 路径技能规范
3. 程序内部注入的协议与校验约束

可维护的权威模板包只覆盖第 2 部分。

## 3. 绝对不要写什么

外部 `SKILL.md` 不要写：

- JSON 结构
- 字段名
- 枚举值列表
- 输出模板
- schema 说明
- “只能输出哪些字段”之类的协议描述

这些都属于程序边界，不属于外部提示词资产。

## 4. 目录结构

```text
<domainKey>/
└── <skillName>/
    └── SKILL.md
```

要求：

- `domainKey` 必须与运行时枚举完全一致。
- 一个 `domainKey` 下可以有多个模板，但都必须服务同一处理步骤或同一自治路径。
- 不再使用旧的能力域抽象，只按真实运行时步骤组织。

## 5. 当前可用 domainKey

### Consult

- `consult.general`
- `consult.narrative`
- `consult.image_prompt`
- `consult.video_prompt`
- `consult.storyboard`

### Task

- `task.image`
- `task.video`
- `task.character_extract`
- `task.scene_extract`
- `task.prop_extract`

### Project

- `project.bind_style`

### Asset

- `asset.character_from_image`
- `asset.scene_from_image`
- `asset.prop_from_image`

### Flow

- `flow.story_to_video.shot_split`
- `flow.story_to_video.path.1_5_pro`
- `flow.story_to_video.path.blend_2_0`
- `flow.story_to_video.path.scene_character_closeup_2_0`

### Other

- `revise.project`
- `router.consult_only`

## 6. 推荐结构

每个 `SKILL.md` 建议按下面结构写：

- 适用场景
- 负责什么
- 不负责什么
- 判断原则
- 工作方法
- 质量标准
- 失败回退

推荐风格：

- 短
- 硬
- 可执行
- 面向该步骤本身

## 7. 通用提示词写法规范

外部 skill 面向的是可迁移的创作判断，不应绑定单一模型。写视频、图片、分镜相关规则时，默认遵守以下规范：

- 使用自然语言描述，不写 SD 标签串、权重括号、命令行参数或 provider 私有语法。
- 优先写正向目标：主体是谁、在哪里、做什么、镜头怎么运动、画面如何变化。
- 多模态引用必须明确编号：使用 `图片1`、`图片2`、`视频1`、`视频2`，不要写 `Reference image 1` 或“上图”。
- 每个参考素材都要说明职责：身份、服装、空间、道具、构图、动作、运镜或特效。
- 运镜要具体，例如固定镜头、缓慢推近、横摇、跟拍、俯拍、反打、特写切换。
- 情绪要落在可见表现上，例如停顿、眼神、手部动作、肩膀僵住、黑气翻涌、灯光变暗。
- 有台词时，要保证台词由画面触发并推动下一步，不把设定说明硬塞进角色口中。
- 字幕、音效、BGM 是否进入最终 prompt 由运行时编译器决定；skill 只说明应该保持音画同步和结构一致。
- 禁止把角色身份写错，尤其不能让已知男角色变成女角色、成年人变成儿童、现代职业装变成无关服装。
- 禁止为了丰富画面创造项目中不存在的稳定资产；如果只是镜头级临时元素，应明确它由文生图补足，不视为项目资产。

## 8. 各类 domain 的边界

- `consult.*`
  - 只做咨询，不写项目数据。
  - 不把项目资产当默认前提。
- `project.bind_style`
  - 只负责沉淀项目级风格。
  - 不绑定人物、场景、道具，不生成任务。
- `task.image` / `task.video`
  - 只整理当前轮需求与当前轮上传素材。
  - 不读取项目维护的人物、场景、道具。
  - 可以继承已有项目风格，但不把风格绑定动作混进当前步骤。
- `task.character_extract` / `task.scene_extract`
  - 只做稳定抽取，不做图片绑定。
- `asset.*_from_image`
  - 只负责把上传图片转成稳定资产锚点。
  - 必须遵守用户显式指定的图片归属语义。
- `flow.story_to_video.*`
  - 优先消费项目里已存在的人物、场景、道具。
  - 不自动创建项目资产，不越过当前阶段职责。
  - 剧情需要但项目未绑定的可见元素，可以规划镜头级文生图兜底，保证画布不断镜。
- `revise.project`
  - 只解析修改意图，不直接执行写库。
- `router.consult_only`
  - 只能在咨询路径中保守选路。

## 9. 维护约束

- 新增 domain 时，必须先更新运行时枚举，再补模板目录。
- 删除 domain 时，必须同步删掉模板目录和 README 说明。
- 示例包要始终保持 collector 可直接读取。
- 包内不应保留 `legacy`、嵌套 `.git` 或其他历史残留。
