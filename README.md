# Lumina Prompt Authority Pack

这套目录是 Lumina 当前 AI Chat 新架构下的权威提示词包。它只承载每个处理步骤的行为规范，不承载程序协议。

## 核心原则

- 目录结构固定为 `<domainKey>/<skillName>/SKILL.md`。
- 每个 `domainKey` 对应一个明确处理步骤，不再复用旧的通用创作域。
- 外部 `SKILL.md` 只描述职责边界、判断原则、工作方法、质量要求和失败回退。
- 路由规则、输出协议、结构化字段、工具约束、数据库写入方式全部由程序内部控制。

## 当前 domainKey

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
- `flow.story_to_video.shot_plan`
- `flow.story_to_video.shot_prompt_package`

### Other

- `revise.project`
- `router.consult_only`

## 提示词分层

最终送给模型的提示词由三层组成：

- 程序系统指令：定义当前步骤的硬边界和运行时上下文。
- 路径技能规范：来自本目录，只补充该步骤的领域判断与质量标准。
- 程序协议约束：结构化输出协议、schema 校验和工具调用规则，由代码注入。

其中只有第二层属于可维护的外部提示词资产。

## 使用边界

- `consult.*` 只做咨询，不写项目数据。
- `project.bind_style` 只维护项目级风格，不维护人物、场景、道具资产。
- `task.image` / `task.video` 只消费本轮上传素材，不读取项目维护资产。
- `task.*_extract`、`asset.*_from_image`、`flow.story_to_video.*`、`revise.project` 才参与项目结构化处理。
- `flow.story_to_video.*` 只消费项目内已存在的人物、场景、道具，不自动创建新资产。

## 导入示例

```bash
pnpm tsx --env-file=.env scripts/import-lumina-skills.ts --scope=system --source=./tmp/lumina-skill
```

```bash
pnpm tsx --env-file=.env scripts/import-lumina-skills.ts --scope=user --userId=<id> --source=./tmp/lumina-skill
```

## 维护要求

- 这套目录本身就是 collector 的权威 fixture。
- 包内不得保留 `legacy/`、嵌套 `.git/`、无关脚本或历史残留。
- 新增或删除 domain 时，需要同步更新：
  - `lib/shared/ai-skill-domains.ts`
  - `lib/server/ai-chat/skills/domain-catalog.ts`
  - `tmp/lumina-skill/README.md`
  - `tmp/lumina-skill/skill-authoring-guide.md`
