# Lumina Prompt Template 编写说明

> 更新时间：2026-04-21
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
- 一个 `domainKey` 下可以有多个模板，但都必须服务同一处理步骤。
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

### Asset

- `asset.character_from_image`
- `asset.scene_from_image`
- `asset.prop_from_image`

### Flow

- `flow.story_to_video.shot_split`
- `flow.story_to_video.shot_storyboard_requirement`
- `flow.story_to_video.shot_storyboard_plan`
- `flow.story_to_video.shot_task_plan`

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

## 7. 各类 domain 的边界

- `consult.*`
  - 只做咨询，不写项目数据。
  - 不把项目资产当默认前提。
- `task.image` / `task.video`
  - 只整理当前轮需求与当前轮上传素材。
  - 不读取项目维护的人物、场景、道具。
- `task.character_extract` / `task.scene_extract`
  - 只做稳定抽取，不做图片绑定。
- `asset.*_from_image`
  - 只负责把上传图片转成稳定资产锚点。
  - 必须遵守用户显式指定的图片归属语义。
- `flow.story_to_video.*`
  - 只消费项目里已存在的人物、场景、道具。
  - 不自动创建资产，不越过当前阶段职责。
- `revise.project`
  - 只解析修改意图，不直接执行写库。
- `router.consult_only`
  - 只能在咨询路径中保守选路。

## 8. 维护约束

- 新增 domain 时，必须先更新运行时枚举，再补模板目录。
- 删除 domain 时，必须同步删掉模板目录和 README 说明。
- 示例包要始终保持 collector 可直接读取。
- 包内不应保留 `legacy`、嵌套 `.git` 或其他历史残留。
