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

## 导入

```bash
pnpm tsx --env-file=.env scripts/import-lumina-skills.ts --scope=system --repo=https://github.com/<you>/skill-lumina.git
```

如果要导入到用户域：

```bash
pnpm tsx --env-file=.env scripts/import-lumina-skills.ts --scope=user --userId=<id> --repo=https://github.com/<you>/skill-lumina.git
```

本地目录导入仍然支持，但需要显式传 `--source=/path/to/skill-lumina`。
