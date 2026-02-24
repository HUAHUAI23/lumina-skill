# Skill 工程最佳实践（System Prompt Pack）

本文件用于系统提示词一次性注入，不放在 `resources/`。

## 1. 架构原则
- 流程与知识分离：
  - `./SKILL.md` + `./workflows/`：只放路由、SOP、质量闸门。
  - `./system-prompts/`：放高频、稳定、必须常驻的规则和知识。
  - `./resources/`：仅保留“体积大/低频/按需读取”的知识，不做默认依赖。
- 目录语义遵循运行时约定：
  - `./resources/`、`./knowledge/`、`./templates/`、`./examples/`、`./refs/`、`./data/` 为按需读取目录。
  - 其他目录内容默认可进入系统提示词上下文。
- 单一职责：一个 workflow 聚焦一种交付目标。
- 可回归：每次改动都要保留可复测样例与检查清单。

## 2. 指令规范
- 路由钩子清晰：`name + description` 必须能支持稳定匹配。
- 每步写全：输入、动作、输出、失败处理、回退点。
- 默认最小上下文：先跑主流程，必要时再扩展材料。

## 3. Workflow 规范
1. 输入契约
2. 分支判断
3. 中间工件
4. 输出组装
5. 验证回路
6. 失败回退

## 4. 工具与安全设计
- 最小工具集：工具越多，决策复杂度和失败面越大。
- 最小权限：禁止默认开放高风险能力（如无限网络与文件写入）。
- 技能文件不存敏感信息：Skill 内容会进入模型上下文，应视为公开提示文本。
- 外部能力通过受控接口暴露（如 MCP），并给出清晰用途边界。

## 5. 时间敏感治理
- 平台限制和能力变化属于时效信息（高频变更）。
- 对“最新/上限/规则”类请求，允许动态检索覆盖本地默认值。
- 每次关键平台升级后，回归测试核心工作流（A/B/B-Review/B-Ad）。

## 6. 成本与稳定性
- 把稳定长文本固定在系统提示前缀，配合提示词缓存降本。
- 版本固定（pinning）+ 回归样例，保证迭代可控。
- 变更先小步发布，观察失败样例再扩展规则。

## 7. 路径规范（相对路径）
- 统一相对路径，不使用绝对路径。
- skill 根文件引用子目录：`./...`
- 子目录文件回引根目录或兄弟目录：`../...`
- 文档中的路径示例必须与仓库实际结构一致。

## 8. 检查表
```text
□ system-prompts 覆盖所需知识与规则
□ resources 仅存按需读取内容（非默认依赖）
□ workflow 可执行且有回路
□ 输出格式可直接交付
□ 关键场景有回归样例
□ 路径均为相对路径且可解析
```

## 9. 参考来源（用于维护时）
- OpenAI Agents SDK（Skills / Tools / Safety / Prompt Caching / Evals）: https://openai.github.io/openai-agents-python/
- OpenAI Skills 官方文档: https://platform.openai.com/docs/guides/agents/skills
- Anthropic Agent Skills: https://docs.anthropic.com/en/docs/claude-code/sub-agents-and-skills/agent-skills
- Model Context Protocol（Resources / Prompts / Tools）: https://modelcontextprotocol.io/specification
