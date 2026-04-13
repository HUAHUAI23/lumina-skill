# Lumina Skill 编写说明

> 更新时间：2026-04-12
> 对应运行时：`lib/server/ai-chat/skills/*`

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

## 3. 写作原则

- 把真正影响运行时行为的规则直接写进 `SKILL.md`
- 保持短、硬、可执行
- 不要重写系统定义的 schema、路由和工具协议
- 不要假设模型还能 `read_file`
- 不要把关键规则藏在资源目录里

## 4. 推荐内容

`SKILL.md` 建议包含：

- 适用场景
- 工作原则
- 处理步骤
- 质量标准
- 禁止项

## 5. 关于 legacy

`legacy/` 中保留了旧版多文件 skill 包，方便迁移时参考：

- `story` -> `narrative`
- `screenplay` -> `scriptwriting`
- `image_prompt` -> `visual_generation`
- `storyboard` -> `storyboard`

这些旧文件不再是当前导入主路径。
