# Lumina Skill Pack

基于 Lumina 当前 AI Chat skill 运行时整理的示例技能包。

## 当前结构

```text
skill-lumina/
├── general/
│   └── lumina-general/
├── narrative/
│   └── lumina-narrative/
├── scriptwriting/
│   └── lumina-scriptwriting/
├── storyboard/
│   └── lumina-storyboard/
├── visual_generation/
│   └── lumina-visual-generation/
├── legacy/
└── skill-authoring-guide.md
```

说明：

- 新运行时推荐使用 `<domainKey>/<skillName>/SKILL.md`。
- 目录名必须与 `domainKey` 完全一致，不做别名映射。
- `SKILL.md` 是主路径，真正会进入运行时的关键规则应直接写在这里。
- `legacy/` 保存旧版多文件 skill 包，仅作迁移参考，不参与当前导入规范。

## 域映射

| 新 domainKey | 用途 |
| --- | --- |
| `general` | 通用问答、澄清、转单引导 |
| `narrative` | 故事结构、人物弧线、叙事组织 |
| `scriptwriting` | 场景、动作、对白、剧本格式 |
| `storyboard` | 镜头设计、分镜拆解、连续性 |
| `visual_generation` | 图像/视频提示词、构图、风格、强约束 |

## 功能结构

| 功能分层 | 典型 pipeline / 用途 | 推荐主 domain |
| --- | --- | --- |
| `consult` | `consult.general` 通用问答与需求澄清 | `general` |
| `consult` | `consult.narrative` 故事与创意咨询 | `narrative` |
| `task` | `task.image` / `task.video` 直接生成任务 | `visual_generation` |
| `task` | `task.character_extract` / `task.scene_extract` 抽离角色或场景资产 | `narrative` + `visual_generation` |
| `flow` | `flow.story_to_video` 故事到视频完整流程 | `narrative` / `scriptwriting` / `storyboard` / `visual_generation` |
| `revise` | `revise.project` 项目内增量修订 | 按目标选择 `narrative` / `scriptwriting` / `storyboard` / `visual_generation` |

- `scriptwriting` 是创作前置结构域，主要服务剧本化整理、场景化表达和向 `storyboard` 的下游衔接。
- 当前导入脚本按这 5 个新 `domainKey` 工作；`legacy/` 仅作迁移参考，不会被自动导入。

## 导入

```bash
pnpm tsx --env-file=.env scripts/import-lumina-skills.ts --scope=system --repo=https://github.com/<you>/skill-lumina.git
```

如果要导入到用户域：

```bash
pnpm tsx --env-file=.env scripts/import-lumina-skills.ts --scope=user --userId=<id> --repo=https://github.com/<you>/skill-lumina.git
```

本地目录导入仍然支持，但需要显式传 `--source=/path/to/skill-lumina`。
