# Lumina Prompt Template Pack

这是一套按 Lumina 当前运行时真实 `domainKey` 组织的示例提示词包，可被
`scripts/import-lumina-skills.ts` 直接导入。

## 目录约束

- 目录结构必须是 `<domainKey>/<skillName>/SKILL.md`
- 顶层目录名必须和运行时 `PromptDomainKey` 完全一致
- 一个目录对应一个处理步骤，不再复用旧的“通用创作域”
- `SKILL.md` 只放补充规则，不重写系统 schema、路由和工具协议

## 当前包含的 domainKey

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

## 使用原则

- `consult.*` 只做咨询，不写项目资产或 shot。
- `task.image` / `task.video` 只读本轮对话上传素材，不读项目维护资产。
- `task.*_extract`、`asset.*_from_image`、`flow.story_to_video.*`、`revise.project`
  才参与项目结构化处理。
- `flow.story_to_video.*` 只消费项目里已有的人物、场景、道具，不自动创建新资产。

## 导入示例

```bash
pnpm tsx --env-file=.env scripts/import-lumina-skills.ts --scope=system --source=./tmp/skill-lumina
```

```bash
pnpm tsx --env-file=.env scripts/import-lumina-skills.ts --scope=user --userId=<id> --source=./tmp/skill-lumina
```

## 维护说明

- 这套目录本身就是 collector 的测试 fixture。
- 目录里不应保留 `legacy/`、嵌套 `.git/` 或其他与当前 prompt pack 无关的残留内容。
- 新增或删除 domain 时，需要同步更新：
  - `lib/shared/ai-skill-domains.ts`
  - `lib/server/ai-chat/skills/domain-catalog.ts`
  - `tmp/skill-lumina/README.md`
  - `tmp/skill-lumina/skill-authoring-guide.md`
