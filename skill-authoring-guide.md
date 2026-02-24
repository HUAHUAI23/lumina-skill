# Skill 编写指南

> 本文档描述 **2026-02** 版本引入的新 Skill 架构。
> 旧版依赖 LLM 主动调用 `read_file` 读取指令的写法已废弃。
> 对应代码：`lib/server/ai-chat/agent/deepagents.ts` → `buildPreloadedSkillContent()`

---

## 一、核心设计：两类文件

Skill 的文件分为两类，系统处理方式完全不同：

| 类型 | 路径规则 | 处理方式 | 适合存放 |
|------|---------|---------|---------|
| **指令文件** | 路径**不以**知识目录前缀开头 | 启动前**自动注入**到 `<instructions>` | SOP 步骤、输出模板、质量标准 |
| **知识文件** | 路径以知识目录前缀开头 | 仅列出路径，LLM **按需** `read_file` | 行业规范、示例库、素材库 |

**知识目录前缀（任一匹配即为知识文件）：**

```
resources/
knowledge/
templates/
examples/
refs/
data/
```

代码来源：`deepagents.ts` `KNOWLEDGE_DIR_PREFIXES`

---

## 二、系统架构流程

```
每次 Agent 请求
       │
       ▼
buildPreloadedSkillContent()      ← deepagents.ts
       │
       ├─ 读取 Skill 文件列表（DB + 缓存，TTL 60s）
       │
       ├─ 指令文件 → 注入到 <active_skill name="...">
       │    <instructions>
       │      <file path="SKILL.md">...</file>
       │      <file path="STEP/STEP.md">...</file>
       │    </instructions>
       │
       └─ 知识文件 → 仅列路径
            <resources>
              <resource path="/skills/{agentKey}/{skillName}/resources/xxx.md" />
            </resources>
```

**指令文件排序规则（影响 LLM 读取顺序）：**
- `SKILL.md` 永远排第一
- 其余指令文件按路径字母序排列
- STEP/STEP.md 在 STEP/ 目录下，字母序通常紧随 SKILL.md

---

## 三、Skill 优先级与范围

系统支持两种 Skill 来源，优先级如下：

```
用户 Skill（isActive=true）> 系统 Skill > Seed Skill（兜底）
```

- **用户 Skill**：用户在技能编辑器中创建并激活的 Skill，**一个 agentKey 最多一个激活**
- **系统 Skill**：管理员创建的 `scope=system` Skill，当用户没有激活 Skill 时使用
- **Seed Skill**：代码内置兜底（`lib/server/ai-chat/skills/system.ts`），当 DB 中没有系统 Skill 时使用

> 实际选用逻辑见 `filesystem.ts`：
> - 有激活用户 Skill → 只加载该用户 Skill，忽略系统 Skill
> - 无用户 Skill → 加载所有系统 Skill
> - 无系统 Skill → 加载 Seed Skill

---

## 四、标准目录结构

```
<agentKey>/
└── <skillName>/
    ├── SKILL.md              # 必须。技能入口文件（指令文件）
    ├── STEP/
    │   ├── STEP.md           # 所有步骤汇总（指令文件）
    │   ├── STEP1.md          # 步骤 1 详细说明（指令文件）
    │   ├── STEP2.md          # 步骤 2 详细说明（指令文件）
    │   └── ...
    ├── resources/            # 知识文件目录（按需读取）
    │   ├── platform-specs.md
    │   └── style-guide.md
    ├── examples/             # 示例文件目录（按需读取）
    │   └── output-samples.md
    └── templates/            # 模板目录（按需读取）
        └── script-template.md
```

**agentKey 可选值：** `general` / `screenplay` / `storyboard` / `story` / `image_prompt`

**文件路径格式（存入 DB 时）：**
- 相对于 skill 根目录
- 不需要前导 `/`（系统会自动 normalize）
- 示例：`SKILL.md`、`STEP/STEP.md`、`resources/douyin-specs.md`

---

## 五、指令文件写法

### 5.1 SKILL.md（必须）

每个 Skill 的入口文件。LLM **直接从 `<instructions>` 中读取**，无需调用 `read_file`。

**格式规范：**

```markdown
---
name: <技能标识符，与目录名一致>
description: |
  一句话描述：本技能做什么。
  适用场景：用户何时触发此技能。
  输出物：产出什么格式的内容。
version: 1.0
---

## 技能概述

简要说明本技能的定位和核心能力（3-5 行）。

## 执行前置条件（Pre-flight Checklist）

在输出任何内容之前，必须明确的信息：

- **信息项 1**：用于什么目的
- **信息项 2**：用于什么目的

## 执行步骤概览

按顺序列出步骤名称（详细内容在 STEP/STEP.md）：

```
STEP 1 → 步骤名
STEP 2 → 步骤名
...
STEP N → 步骤名
```

所有步骤的完整规范已预加载在系统提示词中，直接按步骤执行。
```

**关键原则：**
- `SKILL.md` **不需要**再写 "请先 read_file STEP/STEP.md" 这类指令
- STEP 文件已预注入，直接引用步骤名称即可
- 保持简洁，50-100 行以内（控制 token 消耗）

---

### 5.2 STEP/STEP.md（推荐）

所有执行步骤的完整规范，是 LLM 执行任务的核心依据。

**格式规范：**

```markdown
## 前置声明

本文件包含 N 个执行步骤，必须全部理解后才能开始工作。

---

### STEP 1｜步骤名称

**目标**：这一步要达成什么。

**执行方法：**
1. 具体操作 1
2. 具体操作 2

**质量检查点：**
> 完成本步骤后，问自己：[检查问题]

---

### STEP 2｜步骤名称

...

---

### STEP N｜输出

按以下标准格式输出：

\`\`\`
[输出模板，使用具体占位符]
\`\`\`

---

## 常见错误 & 避免方法

| 错误类型 | 表现 | 修正方法 |
|---------|------|---------|
| ...     | ...  | ...     |
```

**文件大小限制：**
- 建议 100-400 行
- 超过 400 行考虑拆分成 STEP1.md、STEP2.md 等独立文件
- 所有指令文件**合计**建议不超过 2000 行（约 8000 tokens）

---

### 5.3 拆分为多个 STEP 文件（可选）

当步骤复杂时，可将 STEP.md 按步骤编号拆分：

```
STEP/
├── STEP.md     # 总览，包含所有步骤的简短描述
├── STEP1.md    # STEP 1 的完整规范
├── STEP2.md    # STEP 2 的完整规范
└── ...
```

**注意：** 拆分后**所有文件都是指令文件**，全部预注入系统提示词。拆分是为了组织清晰，不影响性能。

---

## 六、知识文件写法

知识文件**不预注入**，LLM 根据任务需要自行决定是否读取。

### 6.1 适合放入知识文件的内容

| 目录 | 存放内容 | 示例文件名 |
|-----|---------|---------|
| `resources/` | 平台规范、行业标准、技术参数 | `douyin-spec.md`, `video-formats.md` |
| `knowledge/` | 领域知识、背景资料 | `motion-comic-theory.md` |
| `templates/` | 输出模板、格式样板 | `script-template.md`, `shot-list-template.md` |
| `examples/` | 优质示例、参考案例 | `excellent-output-samples.md` |
| `refs/` | 参考资料、外部链接汇总 | `style-references.md` |
| `data/` | 数据表、参数列表 | `pricing-data.md`, `keyword-list.md` |

### 6.2 关键限制：read_file 行数限制

LLM 调用 `read_file` 读取知识文件时有行数限制：

| 参数 | 默认值 | 最大值 |
|-----|--------|--------|
| 默认读取行数 | **100 行** | — |
| LLM 可指定最大行数 | — | **2000 行** |

**影响：**
- 如果知识文件超过 100 行，LLM 需要明确指定 `limit` 参数，否则只读前 100 行
- 超过 2000 行时，系统会显示 WARNING 提示剩余未读行
- **建议单个知识文件不超过 500 行**，避免 LLM 多次分段读取

> 代码来源：`backend.ts` `DEFAULT_READ_LINE_LIMIT = 100`、`MAX_READ_LINE_LIMIT = 2000`

### 6.3 知识文件格式原则

LLM 主要通过**文件名**判断是否需要读取，通过 **grep** 定位内容。因此：

**原则 1：文件名要见名知义**

```
# 好的命名
resources/douyin-vertical-video-specs.md    # 一看就知道是抖音竖屏规范
resources/tvc-30s-format-guide.md          # TVC 30秒格式指南
examples/award-winning-scripts-2024.md     # 2024获奖剧本示例

# 差的命名
resources/doc1.md
resources/notes.md
resources/misc.md
```

**原则 2：在文件顶部写摘要（帮助 LLM 判断相关性）**

```markdown
<!--
摘要：本文件包含抖音平台短视频的技术规格、内容审核标准和流量推荐算法要点。
适用场景：创作抖音短视频脚本时参考。
关键词：抖音、短视频、竖屏、算法、流量
-->

# 正文内容...
```

**原则 3：使用清晰的 Markdown 标题层级**

```markdown
# 抖音短视频规范（2024）

## 基础参数

| 参数 | 规格 |
|-----|------|
| 分辨率 | 1080×1920（9:16） |
| 时长   | 15s / 60s / 3min / 10min |

## 内容规范

### 禁止内容
...
```

**原则 4：适当长度**
- 单个知识文件建议 **50-300 行**（LLM 一次可完整读取）
- 超过 300 行考虑按主题拆分
- 太短（< 20 行）直接合并到 SKILL.md 的附录里

---

## 七、LLM 访问路径参考

LLM 调用 `read_file` 时使用的路径格式：

```
/skills/<agentKey>/<skillName>/<relativePath>
```

**示例：**

```bash
# 读取知识文件
read_file("/skills/screenplay/xiaoshuo2juben/resources/douyin-specs.md")
read_file("/skills/screenplay/xiaoshuo2juben/examples/sample-outputs.md")

# 指定读取行数（默认只读 100 行）
read_file("/skills/screenplay/xiaoshuo2juben/resources/douyin-specs.md", limit=500)

# 列出 resources 目录
ls("/skills/screenplay/xiaoshuo2juben/resources/")

# 在所有知识文件中搜索关键词
grep("抖音", "/skills/screenplay/xiaoshuo2juben/resources/")
```

**注意：**
- **指令文件（SKILL.md, STEP/*.md）不需要 `read_file`** — 内容已在 `<instructions>` 中
- Skills 后端是**只读**的，LLM 不能通过 `write_file` 修改 Skill 内容
- 缓存 TTL 为 60 秒，编辑 Skill 后缓存自动失效

---

## 八、完整示例：`screenplay/xiaoshuo2juben`

```
screenplay/
└── xiaoshuo2juben/
    ├── SKILL.md                         # 技能入口，~80 行
    ├── STEP/
    │   ├── STEP.md                      # 9步流程总览，~400 行
    │   ├── STEP1.md                     # 原文解构详细说明，~40 行
    │   └── ...（STEP2-9.md）
    ├── resources/
    │   ├── douyin-motion-comic-specs.md # 抖音动态漫规范，<300 行
    │   └── bgm-style-guide.md           # 配乐风格指南，<100 行
    └── examples/
        └── excellent-episodes.md        # 优质集数参考，<300 行
```

**token 预算：**

| 文件 | 行数 | 预估 tokens | 加载方式 |
|-----|------|------------|---------|
| SKILL.md | 80 行 | ~400 | 预注入 |
| STEP/STEP.md | 320 行 | ~1600 | 预注入 |
| STEP/STEP1-9.md | 各 ~40 行 | ~1800 | 预注入 |
| **指令文件合计** | **~500 行** | **~2500** | 每次请求固定消耗 |
| resources/douyin-motion-comic-specs.md | <300 行 | 按需 | LLM 决定是否读 |
| resources/bgm-style-guide.md | <100 行 | 按需 | LLM 决定是否读 |
| examples/excellent-episodes.md | <300 行 | 按需 | LLM 决定是否读 |

---

## 九、常见错误

### ❌ 错误 1：在 SKILL.md 里要求读文件

```markdown
# 错误写法
运行前必须先 read_file("STEP/STEP.md") 完整读取，否则不工作。
```

```markdown
# 正确写法
所有步骤已在系统提示词 <instructions> 中预加载，直接按步骤执行。
```

### ❌ 错误 2：把大量静态知识塞进 SKILL.md

```markdown
# 错误写法（SKILL.md 里放 300 行行业知识）
## 行业背景知识
中国短视频平台发展历程...（100行）
各平台算法对比...（100行）
竖屏视觉设计原理...（100行）
```

```markdown
# 正确写法
## 参考资料（按需读取）
- resources/platform-comparison.md — 各平台对比
- resources/vertical-design-principles.md — 竖屏设计原理
```

### ❌ 错误 3：知识文件命名模糊

```
resources/doc.md        # ❌ 不知道是什么
resources/notes.md      # ❌ 太宽泛
resources/info.md       # ❌ 无意义
```

```
resources/douyin-content-policy-2024.md   # ✅
resources/tvc-production-standards.md    # ✅
resources/voice-over-style-examples.md   # ✅
```

### ❌ 错误 4：知识文件过长（超过 300 行）

```
examples/all-samples.md    # 1500 行 — LLM 默认只读前 100 行，大量内容被截断
```

```
examples/drama-samples.md       # 300 行 — 按剧类型拆分
examples/comedy-samples.md      # 250 行
examples/romance-samples.md     # 200 行
```

### ❌ 错误 5：步骤指令过于抽象

```markdown
# 错误写法
STEP 3: 分析故事结构，理解情节。
```

```markdown
# 正确写法
### STEP 3｜情节结构规划

执行以下分析：
1. 识别三幕结构（铺垫/对抗/决议），标记每幕的起止节点
2. 找出诱发事件（打破平衡的转折点）
3. 确定情绪最高点（最适合视觉冲击的场景）

> 检查点：能否用一句话描述"本集最大的戏剧冲突是什么"？
```

---

## 十、新旧架构对比

| | 旧版（LLM 主动读取） | 新版（预注入） |
|--|---------------------|--------------|
| 指令获取方式 | LLM 调用 `read_file` | 自动注入 `<instructions>` |
| 截断风险 | 有（LLM 可能 limit 不足） | 无（完整内容预注入） |
| 合规依赖 | 依赖 LLM 遵守指令 | 系统保证，零合规风险 |
| 知识文件 | 同样需要 LLM 主动读取 | 路径列在 `<resources>`，按需读取 |
| token 成本 | 不可预测 | 固定（指令文件总量） |
| 指令文件大小限制 | 受 read_file limit 约束 | 无限制（预注入） |
| 知识文件大小建议 | — | **≤ 300 行**（单次读完） |

---

*文档版本：2026-02 | 对应代码：`lib/server/ai-chat/agent/deepagents.ts` · `lib/server/ai-chat/skills/backend.ts`*
