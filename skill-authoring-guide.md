# Lumina Prompt Template 编写说明

> 更新时间：2026-04-21
> 对应运行时：`lib/shared/ai-skill-domains.ts`、`lib/server/ai-chat/handlers/index.ts`、`scripts/import-lumina-skills*.ts`

## 1. 当前模型是什么

当前 Prompt Template 不是“按创作能力分组”的老 skill 包，而是“按处理路径和处理步骤分组”的补充提示词。

系统已经固定：

- 路由
- 输出 schema
- 工具约束
- 项目读写职责
- flow 的阶段拆分

模板只负责补充：

- 该步骤的判断原则
- 该步骤的领域术语
- 该步骤的质量标准
- 该步骤禁止做什么

## 2. 目录结构

```text
<domainKey>/
└── <skillName>/
    └── SKILL.md
```

要求：

- `domainKey` 必须与运行时枚举完全一致
- 一个 `domainKey` 可以有多个模板，但每个模板都必须服务同一个步骤
- 不再使用 `general / narrative / storyboard / visual_generation` 这类旧抽象域

## 3. 当前可用 domainKey

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

## 4. 各类 domain 的边界

- `consult.*`
  - 只做咨询，不写项目。
  - 不读取项目资产做答案主依据。
- `task.image` / `task.video`
  - 只整理当前轮需求与当前轮上传素材。
  - 不读取项目维护的人物、场景、道具。
- `task.character_extract` / `task.scene_extract`
  - 输出稳定 identityKey + description。
  - 不负责生成任务图，不负责绑定图片。
- `asset.*_from_image`
  - 一张上传图对应一条绑定结果。
  - 必须遵守用户显式写的“图1是角色A”这类语义。
- `flow.story_to_video.*`
  - 只消费项目里已存在的人物、场景、道具。
  - 不自动创建资产，不跳出当前 flow 阶段职责。
- `revise.project`
  - 只产出结构化 patch。
- `router.consult_only`
  - 只能在 5 个 `consult.*` 中选路。

## 5. 推荐写法

每个 `SKILL.md` 建议包含：

- 这个步骤负责什么
- 这个步骤绝不能做什么
- 输入里哪些信号最重要
- 输出时优先保证什么
- 失败时如何保守返回

推荐风格：

- 短
- 硬
- 可执行
- 少背景散文

## 6. 维护约束

- 新增 domain 时，必须先更新运行时枚举，再补模板目录。
- 删除 domain 时，必须同步删掉模板目录和 README 中的说明。
- 示例包要始终保持“collector 可直接读取”的状态，不能只写文档不落目录。
- 示例包不应保留旧 `legacy` 目录、嵌套仓库 `.git` 或其他历史残留。
