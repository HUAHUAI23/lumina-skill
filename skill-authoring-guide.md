# Lumina Skill 编写说明

> 更新时间：2026-04-17
> 对应运行时：`lib/server/ai-chat/skills/*`、`lib/shared/ai-workflow-types.ts`、`scripts/import-lumina-skills*.ts`

## 1. 先理解当前运行时

当前 skill 的本质是“补充提示词”，不是可供模型自由浏览的文件系统。

系统已经负责：

- agent 身份
- workflow phase 指令
- output contract
- 工具边界

skill 只应该补充：

- 领域术语
- 处理 SOP
- 风格偏好
- 常见限制
- 质量标准

## 2. 推荐结构

```text
<domainKey>/
└── <skillName>/
    └── SKILL.md
```

当前可用 `domainKey`：

- `general`
- `narrative`
- `scriptwriting`
- `storyboard`
- `visual_generation`

目录约束：

- 目录名必须与 `domainKey` 完全一致
- 不再维护 `general_chat / image_prompt / video_prompt / shot_decomposition / shot_frame_planning` 这类旧目录别名

## 2.1 最新功能结构

- `consult.general`：通用咨询与需求澄清，优先使用 `general`
- `consult.narrative`：故事、创意、结构咨询，优先使用 `narrative`
- `task.image` / `task.video`：直接生成任务，优先使用 `visual_generation`
- `task.character_extract` / `task.scene_extract`：抽离角色或场景资产，通常由 `narrative` 负责语义整理，`visual_generation` 负责视觉提示词
- `flow.story_to_video`：完整故事到视频流程，组合使用 `narrative`、`scriptwriting`、`storyboard`、`visual_generation`
- `revise.project`：项目内增量修订，按修订目标选择对应 domain

- `scriptwriting` 不是默认 consult pipeline 的独立入口，而是创作结构层，用于把故事整理成可拍、可拆分镜的剧本化中间稿。
- 当前导入脚本只识别上述 5 个新 domainKey；`legacy/` 目录不会自动导入。

## 3. 写作原则

- 把真正影响运行时行为的规则直接写进 `SKILL.md`
- 保持短、硬、可执行
- 不要重写系统定义的 schema、路由和工具协议
- 不要假设模型还能 `read_file`
- 不要把关键规则藏在资源目录里

## 4. 推荐内容

`SKILL.md` 建议包含：

- 定位
- 适用场景
- 核心硬规则
- 输入契约
- 域边界与交接
- 标准输出骨架
- 处理步骤
- 失败回退
- 质量闸门 / 质量标准
- 项目连续性专则

优化原则：

- 先写“这个 domain 负责什么、不负责什么”，避免 skill 互相越权
- 先写标准输出骨架，再写长篇规则，保证运行时拿到的是可直接拼接的协议
- 失败回退要可执行，不能只写“重试、优化一下”
- 项目内场景要明确继承 canon、资产来源和是否为局部修订
- 文案尽量短、硬、可判定，少写解释性散文

## 5. 关于 legacy

`legacy/` 中保留了旧版多文件 skill 包，方便迁移时参考：

- `story` -> `narrative`
- `screenplay` -> `scriptwriting`
- `image_prompt` -> `visual_generation`
- `storyboard` -> `storyboard`

这些旧文件不再是当前导入主路径，也不会被 `scripts/import-lumina-skills*.ts` 自动导入。
