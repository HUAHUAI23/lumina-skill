# Skill 工程最佳实践（Skill Engineering Playbook）

本文件定义如何把 `camera-director` 维护为稳定、可扩展的流程型 skill。

---

## 1. 架构原则

### 1.1 流程与知识分离
- `SKILL.md`：只放路由、SOP、执行守则
- `workflows/`：放可执行流程步骤
- `resources/`：放术语库、经验库、平台规则、模板

### 1.2 单一职责
- 一个 workflow 聚焦一个目标（创建分镜 / 优化分镜）
- 避免在单文件里混合“流程定义 + 大量知识百科”

### 1.3 可回归
- 每次流程变更必须保留检查清单
- 输出结构固定，便于做回归比对

---

## 2. 指令设计规范

### 2.1 触发描述清晰
- Skill 的 `description` 必须覆盖触发关键词
- 避免模糊定位（例如“做视频相关事情”）

### 2.2 指令可执行
- 统一使用动作词：`分析`、`拆解`、`校验`、`回退`
- 每步写输入、动作、输出、失败处理

### 2.3 结构化与渐进加载
- 主文档只保留索引和关键流程
- 需要时才读取深层资源（progressive disclosure）
- 控制上下文体积，防止信息淹没关键指令

---

## 3. Workflow 设计规范

### 3.1 标准六段式
1. 输入契约
2. 任务分支判断
3. 中间工件生成
4. 输出组装
5. 验证回路
6. 失败回退

### 3.2 反馈回路必备
- 任一关键校验失败都要回退到对应步骤修订
- 禁止“带病输出”最终结果

### 3.3 时间敏感信息治理
- 平台参数、模型能力、上传限制属于时效信息
- 这类信息必须优先放 `resources/`，并允许动态检索覆盖

---

## 4. 文档编排规范

### 4.1 `SKILL.md` 建议上限
- 主体内容保持精简（可读、可维护）
- 大段术语/示例迁移至 `resources/`

### 4.2 引用层级
- 默认只一层引用（`SKILL.md -> workflows/resources`）
- 避免多层递归引用导致上下文膨胀

### 4.3 示例策略
- 示例应覆盖“标准输入”和“失败回退”两类
- 示例目的是校验流程，不是堆砌风格文本

---

## 5. camera-director 执行检查表

```text
□ SKILL.md 是否只保留流程核心
□ workflow 是否具备输入/输出/失败处理
□ 知识性内容是否都在 resources/
□ 是否具备平台时效信息更新机制
□ 是否有验证闭环与回退机制
□ 输出模板是否可直接执行
```

---

## 6. 来源（检索日期：2026-02-24）

- OpenAI Codex Skills（仓库与文档链接）: https://github.com/openai/codex/tree/main/docs/skills
- OpenAI Model Spec（指令层级）: https://model-spec.openai.com/
- Anthropic Skills Authoring（最佳实践）: https://docs.anthropic.com/en/docs/claude-code/skills
- Anthropic Define custom slash commands and skills: https://docs.anthropic.com/en/docs/claude-code/slash-commands
