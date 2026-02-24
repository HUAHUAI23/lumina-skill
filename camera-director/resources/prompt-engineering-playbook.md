# 提示词工程最佳实践（Prompt Engineering Playbook）

本文件沉淀可复用的提示词工程经验，服务于 `camera-director` 的流程执行。

---

## 1. 通用 Prompt 设计原则

### 1.1 指令优先、上下文分层
- 先放核心任务，再放约束、输入、输出格式
- 复杂任务使用明确分段：`目标 / 输入 / 规则 / 输出`
- 使用结构化标记（Markdown 标题、XML 标签、字段名）降低歧义

### 1.2 具体、可验证、可执行
- 指令中明确“要做什么”，不要只写“不要做什么”
- 输出必须给出固定字段，避免自由散文
- 先给最小可行版本，再迭代细化

### 1.3 复杂任务分阶段
- 大任务拆成中间工件：场次表 -> 镜头表 -> Prompt 包 -> 验证报告
- 每阶段都可检查通过/失败，避免一次性长输出失控

### 1.4 迭代策略
- `zero-shot` 先跑通
- 失败时补 `few-shot` 示例
- 若高频重复任务仍不稳，再考虑更强约束（模板/脚本）

---

## 2. API/Agent 侧工程化建议

### 2.1 消息角色治理
- `developer` 层定义长期规则
- `user` 层定义本次任务
- 生成前先解析优先级冲突（安全规则 > 开发规则 > 用户偏好）

### 2.2 Prompt 版本化与评测
- 对生产提示词做版本号
- 每次修改都用固定样例回归测试
- 避免未评测的 prompt 直接上线

### 2.3 结构化输出优先
- 能用 JSON/表格就不用自由文本
- 对关键字段做 schema 约束（例如镜头字段、时长、转场）
- 用自动校验替代人工肉眼检查

---

## 3. 视频生成场景专项策略

### 3.1 Sora 风格建议
- 先给清晰场景摘要，再写镜头、动作、光影与风格
- 描述要具体，避免抽象词堆叠
- 复杂动作用时间节拍拆分（如 `0-3s`、`3-6s`）

### 3.2 Runway 风格建议
- 优先写清楚三类运动：主体、场景、摄像机
- 语句简洁，减少互相冲突的动作词
- 机位与运镜用标准电影术语，避免口语化

### 3.3 多模态平台（含 Seedance）
- 明确每个素材的用途（首帧、动作参考、运镜参考、音频节奏）
- 同一提示词中避免“素材身份”混用
- 长镜头任务先写运动主线，再加风格细节

---

## 4. camera-director 标准 Prompt 骨架

```text
[Header / 风格与目标]

Camera Movement: [运镜路径 + 速度 + 稳定性 + 机位关系]

Subject & Action: [Image/角色映射 + 动作 + 台词(原语言)]

Environment & Mood: [光影 + 氛围 + 空间层次 + 声音线索]
```

如需平台化导出，可在主稿后附：
- `CN 解析`
- `EN Prompt`（运镜关键词前置）

---

## 5. 常见反模式

- 一段提示词塞入多个互斥运镜
- 只有风格词，没有可执行动作
- 台词被翻译导致语义偏移
- 未拆分时序，复杂动作挤在单句
- 只给“禁止项”，不给“替代执行路径”

---

## 6. 来源（检索日期：2026-02-24）

- OpenAI Prompt Engineering Guide: https://platform.openai.com/docs/guides/prompt-engineering
- OpenAI Prompting Guide: https://cookbook.openai.com/examples/gpt4-1_prompting_guide
- OpenAI Help: Best practices for prompt engineering: https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-openai-api
- OpenAI Sora Prompting Guide: https://help.openai.com/en/articles/9957612-generating-videos-on-sora/
- Runway Gen-4 Prompting Guide: https://help.runwayml.com/hc/en-us/articles/39789879462419-Gen-4-Prompting-Guide
