# 提示词工程最佳实践（System Prompt Pack）

本文件用于系统提示词一次性注入，不放在 `resources/`。

## 1. 核心写法（高优先）
- 指令前置：先写目标，再写约束，再写输出格式。
- 清晰具体：避免抽象词，直接写动作、主体、顺序、时长。
- 分隔语义块：使用稳定标题或标签（例如 `Camera Movement`、`Subject & Action`）。
- 正向约束：多写“必须包含什么”，少写“不要做什么”。
- 先零样本、再少样本：仅在歧义高时补 1-2 条示例，避免示例污染。

## 2. 结构化输出与校验
- 能用结构就不用散文：优先表格/JSON。
- 严格字段：关键任务定义必填字段，缺项即失败回路重试。
- 对外可机读：如需稳定解析，优先 `strict JSON schema`（平台支持时）。
- 输出后自检：编号一致、台词原文保留、时长约束、镜头物理逻辑。

## 3. 复杂任务拆解
- 复杂镜头按时序拆段：`[0-2s] -> [2-5s] -> [5-8s]`。
- 一镜一主运动：避免同段互斥指令（如同时 `dolly in` 和 `dolly out`）。
- 先中间工件：场次表/镜头计划表通过后再产终稿。

## 4. 成本与稳定性
- 静态长前缀放系统提示：结合提示词缓存降低重复 token 成本。
- 版本化与可回归：关键 prompt 编号，配套回归样例。
- 评测驱动迭代：每次改规则先定义“通过/失败”样例再更新。
- 对生产任务固定模型版本（snapshot/pin），减少行为漂移。

## 5. 视频任务专项
- Sora：先整体场景，再镜头与动作细节。
- Runway：主体动作 + 场景动作 + 镜头动作三段表达。
- Seedance：先写 `@素材` 用途映射，再写时序与镜头主线。

## 6. 语义标记与分隔
- 长提示词优先使用稳定分隔（标题块或 XML/标签块）减少歧义。
- 多约束场景按“目标 > 硬约束 > 软约束 > 输出格式”顺序排列。
- 指令冲突时以“硬约束优先”并触发失败回路重写。

## 7. camera-director 标准骨架
```text
[Header]
Camera Movement: ...
Subject & Action: ...
Environment & Mood: ...
```

## 8. 路径与模板规范
- 所有路径统一使用相对路径。
- 在 skill 根文件中使用 `./...`；在 `./workflows/` 内引用上级目录用 `../...`。
- 禁止绝对路径与环境耦合路径。

## 9. 常见反模式
- 互斥运镜同时出现
- 只有风格词，没有动作路径
- 台词被翻译导致语义偏移
- 复杂动作不拆时序

## 10. 参考来源（用于维护时）
- OpenAI Prompting Guide: https://platform.openai.com/docs/guides/prompt-engineering
- OpenAI Agents Guide（含 Prompting / Evals / Caching）: https://openai.github.io/openai-agents-python/
- OpenAI Function Calling & Structured Outputs: https://platform.openai.com/docs/guides/function-calling
- Anthropic Prompt Engineering Overview: https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
