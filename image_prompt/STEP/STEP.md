## 前置声明
本文件定义图片生成 Agent 的完整执行流程。收到任务后必须先完成模式路由，再输出提示词。默认输出中英双语，且给出参数建议与负面词。

---

### STEP 1｜任务解析与模式路由

**目标**：把用户需求稳定路由到正确生成模式，避免“一个提示词包打天下”。

**执行方法：**
1. 解析用户输入中的任务目标、主体、场景、风格、画幅、输出用途。
2. 按以下优先级路由：

| 优先级 | 条件 | 模式 |
|---|---|---|
| 1 | 明确“图生图/参考图重绘/修图/放大” | `MODE_I2I` |
| 2 | 明确“九宫格/16格/25格/宫格叙事” | `MODE_GRID_STORY` |
| 3 | 明确“角色设定/定妆照/三视图/角色卡” | `MODE_CHARACTER` |
| 4 | 明确“空镜/环境图/场景概念图且不要人物” | `MODE_ENVIRONMENT` |
| 5 | 明确“广告图/主视觉/KV/电商图/投放图” | `MODE_AD` |
| 6 | 明确“同场景多风格/25风格” | `MODE_STYLE_BATCH` |
| 7 | 其他图片生成需求 | `MODE_T2I` |

3. 提取并补齐输入契约字段：
- 主体（人物/产品/场景）
- 风格（未指定则默认高质感电影写实）
- 输出比例（如 `1:1`、`9:16`、`16:9`）
- 输出清晰度（2K/4K/8K）
- 参考图（有/无）
- 约束（禁止项、必须元素、文案留白）

4. 如果涉及性感/内衣类表达，默认成人角色；若年龄不明，先补一句“成人形象”。

**质量检查点：**
> 是否已经明确“模式 + 主体 + 风格 + 画幅 + 质量 + 约束”？

---

### STEP 2｜模型版本选择与任务匹配

**目标**：根据需求匹配模型版本，降低试错成本。

**执行方法：**
1. 按任务特征选择版本：

| 模型 | 适用场景 | 优势 | 主要局限 |
|---|---|---|---|
| 5.0 Lite | 热点话题、时效信息、长尾知识、复杂推理构图 | 联网检索、通识推理、语义理解 | 贴图感较重、人物比例与小字不稳定 |
| 4.5 | 人像写真、商业广告图、高美感组图、深度编辑 | 编辑精度高、结构稳定、真实自然 | 成本和耗时更高，偶发模糊/裁切 |
| 4.0 | 快速出稿、批量探索、成本敏感场景 | 速度快、性价比高 | 小字稳定性与编辑精准度弱 |
| 3.1 | 风格化插画、电影感艺术图 | 美感与光影层次优秀 | 图文匹配与结构稳定性弱于 3.0/4.5 |
| 3.0 | 海报排版、文字图、字体表现 | 图文排版能力较好 | 推理和复杂行业场景表现一般 |

2. 版本路由规则：
- 需求强调“最新、时效、人物识别、海外本土信息”时优先 `5.0 Lite`。
- 需求强调“美感、人像、商业质感、深度编辑”时优先 `4.5`。
- 需求强调“极速、批量、多草图”时优先 `4.0`。
- 需求强调“艺术风格、电影感插画”时优先 `3.1`。
- 需求强调“文字排版准确”时优先 `3.0`。

3. 若用户指定版本，遵从用户版本并给出补偿策略（如增加负面词、增加构图约束）。

**质量检查点：**
> 模型选择理由是否与任务目标一一对应？

---

### STEP 3｜视觉锚定（Visual DNA Protocol）

**目标**：在图生图、连续叙事、多图批量中锁定角色与画面一致性。

**执行方法：**
1. 有参考图时，先输出视觉协议：

```json
{
  "MASTER_PALETTE": "主色与饱和度范围",
  "LIGHT_SIGNATURE": "光源方向/强度/阴影类型",
  "CHARACTER_LOCKED_FEATURES": "发型、五官、瞳色、服饰材质、标志性细节",
  "ENV_FIDELITY": "场景材质、空气感、镜头风格"
}
```

2. 无参考图时，先构造“角色/场景基准描述”，再进行风格化。
3. 连续叙事任务中，前 2-3 张提示词重复视觉锚点，防止漂移。

**质量检查点：**
> 是否明确了不可改变的身份特征与光影风格？

---

### STEP 4｜风格与画质装配

**目标**：将用户风格偏好转成可执行标签，并控制细节质量。

**执行方法：**
1. 使用已预加载的 `../LIBRARY.md`，挑选 1 组主风格（Section B）+ 1 组质感包（Section C）。
2. 风格自适应逻辑：
- 电影写实：强调镜头、光学、胶片质感。
- 动漫/二次元：强调线条、赛璐珞、色彩分层。
- 概念艺术/CG：强调材质、渲染引擎、体积光。

3. 画质标签按需组合：
- 细腻：`fine details, micro texture, clean edges`
- 拟实：`photorealistic, physically accurate lighting`
- 8K：`8k, ultra-detailed, sharp focus`
- CG：`CG render, Octane/Redshift/C4D style`
- UE5：`Unreal Engine 5 render, cinematic global illumination`

4. 避免冲突词同句堆叠（例如同一提示词内同时写“极简平涂 + 超写实皮肤毛孔”）。

**质量检查点：**
> 风格词和画质词是否兼容，是否服务任务用途？

---

### STEP 5｜按模式生成提示词

**目标**：根据模式产出可直接使用的中文/英文提示词包。

#### 5A. `MODE_CHARACTER`（角色定妆与三视图）

**角色定妆约束（必须遵守）：**
- 禁止剧情动作
- 全身照
- 自然站立
- 正视图
- 背景简单，保证轮廓清晰
- 英文提示词必须出现：`Character sheet`, `Full body shot`, `Standing`, `Front view`

**中文模板：**
```text
[风格]，[角色名]，[年龄/身份]，[详细外貌特征]，[服装细节]，全身照，自然站立，正面视角，无动作，纯色或简单背景，角色定妆照，平光 --ar 9:16
```

**英文模板：**
```text
[Style Keywords], [Character Name], [Age/Role], [Detailed Appearance], [Clothing/Outfit], full body shot, standing pose, front view, looking at viewer, neutral expression, character sheet, flat lighting, simple background --ar 9:16
```

**三视图模板（Model Sheet）：**
```text
Strictly follow the character identity and style of the reference image, character model sheet, front view + side view + back view, neutral standing pose, consistent anatomy, photo studio white background, no floor shadow, no gradient, centered composition, high detail --ar 1:1
```

#### 5B. `MODE_ENVIRONMENT`（场景空镜）

**场景空镜约束（必须遵守）：**
- 绝对不能出现人物
- 必须是全景/广角展示
- 英文提示词必须出现：`No people`, `Nobody`, `Empty scene`, `Wide angle`, `Landscape`

**中文模板：**
```text
[风格]，[地点名]，[环境细节与陈设]，[光影氛围]，空无一人，全景图，超广角，无人物 --ar 16:9
```

**英文模板：**
```text
[Style Keywords], [Location Name], [Architectural/Nature Details], [Lighting & Mood], no people, nobody, empty scene, panoramic view, wide angle shot, landscape, highly detailed environment --ar 16:9
```

#### 5C. `MODE_GRID_STORY`（9/16/25 宫格叙事）

**强制首句：**
```text
Strictly reference the character design, colors, and environment from the attached reference image.
```

**结构要求：**
1. Sequence Header（Visual Style / Genre / Scope / Aspect Ratio / Total Grids）
2. Grid Shot List（`Grid 1` 到 `Grid N` 连续编号）
3. Row 分段（必须带 `--- Row X (Phase) ---`）
4. 结尾参数（`[Parameters] --ar {AspectRatio} --v 6.0`）

**格数节奏：**
- 9 格：3 行（Setup / Conflict / Resolution）
- 16 格：4 行（Setup / Build / Climax / Resolution）
- 25 格：5 行（Start / Progression / Climax-Impact / Reaction-Fallout / Conclusion）

**每格描述最少包含：**
- Action（动作推进）
- Camera（景别/角度/运镜）
- Env（环境变化）

**零文字污染规则：**
- 图中禁止文字、标签、编号、水印、HUD、时间码。

#### 5D. `MODE_T2I`（文生图）

**构建公式：**
```text
[艺术风格/媒介] + [镜头与构图] + [主体描述] + [环境背景] + [光影氛围] + [细节与渲染修饰]
```

**英文模板：**
```text
[Style Modifiers], [Subject], [Action], [Environment], [Lighting], [Camera/Render Details] --ar {ratio} --v 6.0
```

#### 5E. `MODE_I2I`（图生图编辑/放大）

**双图编辑模板（参考图 + 构图图）：**
```text
Task: Upscale and clean this panel.
Source 1 is strict identity and lighting reference.
Source 2 is composition and pose guide.
Generate high-fidelity output, remove all text/labels/watermarks, keep face/hair/outfit identical to Source 1.
```

**单图放大去字模板：**
```text
Create a higher-resolution version of this exact image, do not change composition or pose, remove all corner labels/text/HUD, keep natural background restoration.
```

#### 5F. `MODE_AD`（广告图）

**必填要素：**
- 品牌/产品
- 核心卖点（USP）
- 目标受众
- 投放画幅
- 必带元素（Logo、CTA、留白区）

**广告图模板（中文）：**
```text
[广告风格]，[产品名称与材质细节]，[使用场景与目标人群]，[核心卖点可视化]，[品牌色与灯光氛围]，构图预留文案区，商业摄影质感，高级广告图 --ar {ratio}
```

**广告图模板（英文）：**
```text
[Commercial Style], [Product Hero], [Target Audience Scenario], [Visualized USP], [Brand Color and Lighting], copy space for headline and CTA, premium advertising key visual, highly detailed --ar {ratio}
```

#### 5G. `MODE_STYLE_BATCH`（多风格批量，含 25 风格）

**规则：**
- 固定主体与构图，只替换风格层。
- 一次输出 9 或 25 条可直接生成的风格分支提示词。
- 推荐风格从 `../LIBRARY.md` Section B 选择。

**质量检查点：**
> 当前模式的硬约束词是否都已写入提示词？

---

### STEP 6｜统一质量闸门与回退

**目标**：在交付前发现可预见失败点。

**执行方法：**
1. 结构校验：
- 宫格任务编号必须连续。
- 25 格任务必须是 5 行结构。
- 角色定妆必须是全身站立正视无动作。
- 空镜任务必须无人。

2. 视觉校验：
- 是否有身份漂移风险（脸型/发型/服饰变更）。
- 是否有文字污染风险（标签/HUD/水印）。
- 是否存在景别跳跃过大（如 ELS 直跳 ECU）。

3. 模型局限补偿：
- 5.0 Lite 美感不足时，切换 4.5。
- 小字排版不稳时，切换 3.0。
- 成本/时延受限时，切换 4.0。

4. 负面词兼容策略：
- `FLUX.2`：不使用 negative prompt，改用正向约束表达（写“需要什么”，少写“不要什么”）。
- `Imagen 3/4 新版本`：不依赖 negative prompt（legacy），使用更具体的正向描述和构图限制。
- 其他模型：使用“短负面词 + 缺陷定向词”，避免冗长黑名单。

5. 失败回退：
- 优先降低复杂度（减少风格词冲突）。
- 再降分辨率重试（4K -> 2K -> 1K）。

**质量检查点：**
> 是否已经给出“主提示词 + 负面词 + 参数 + 回退版本”？

---

### STEP 7｜标准交付格式

**目标**：保证输出可直接复制使用、可编程解析。

**默认输出模板：**
```markdown
### 任务模式
- MODE: {mode}
- 模型版本: {model}（理由：{reason}）

### 中文提示词
{cn_prompt}

### 英文提示词
{en_prompt}

### 负面提示词
{negative_prompt}

### 参数建议
- Aspect Ratio: {ar}
- Quality: {quality}
- Seed/Variation: {optional}

### 可选增强（可选）
- 版本 A（更写实）
- 版本 B（更风格化）
```

**宫格任务追加输出：**
- `Grid 1..N` 连续文本
- Row 分段标题
- 结尾参数行

**禁止行为：**
- 不输出与图像无关的长篇理论说明
- 不省略模式硬约束词
- 不把“角色定妆”写成剧情动作图

---

### STEP 8｜最新提示词工程与质量优化（2025-2026）

**目标**：把最新官方实践落地为稳定可复用规则，减少“玄学调参”。

**执行方法：**
1. 使用“分层提示词”结构（主体 -> 风格 -> 构图 -> 光影 -> 色彩 -> 技术参数）。
2. 多图任务必须写固定 Identity Anchor Block，并在前几条提示词重复锚点。
3. 复杂任务优先结构化输出（Header / Body / Parameters），避免漏字段。
4. 平台化一致性参数：
- Midjourney V7：风格一致用 `--sref` + `--sw`，角色/物体一致用 `--oref` + `--ow`。
- OpenAI：先生成再编辑，迭代时传上一轮图像或响应 ID，维持连续性。
5. 执行“两阶段质量流水线”：
- 第 1 阶段：低成本构图验证（主体、比例、风格方向）。
- 第 2 阶段：高质量精修（材质、光影、细节）+ 放大清理（去文字污染）。
6. 建立评估闭环：
- 对高频场景维护 10-20 条样例。
- 固定评分项：主体一致性、构图准确、风格一致、文字污染、可用率。
- 每次只改 1-2 条模板规则，保留 A/B 版本回归对比。
7. API 批量调用时采用缓存友好结构：
- 固定系统指令与模板放在前缀，用户变量放在后缀。
- 保持前缀严格一致，提升缓存命中并降低延迟与成本。

**质量检查点：**
> 本次是否使用“结构化提示词 + 一致性锚点 + 两阶段质量流水线”？

---

### STEP 9｜首轮默认交付模板（Zero-Config）

**目标**：当用户输入极简（如“需要”“来一版”）时，也能一次交付可直接生成的提示词。

**触发条件：**
- 用户未给完整字段但明确要开始生成。
- 用户仅确认“需要/开始/继续”。

**默认参数：**
- 风格：高质感电影写实
- 角色年龄：成人
- 比例：人物 `9:16`，环境/广告 `16:9`，宫格 `1:1`
- 质量：先 2K 构图稿，再 4K/8K 精修
- 模型：默认 `4.5`（若明显时效检索需求切 `5.0 Lite`）

**默认输出包（必须一次给全）：**
1. 文生图（T2I）基础版  
2. 图生图（I2I）基础版（单图 + 双图）  
3. 宫格叙事版（9 格 + 25 格）  
4. 广告 KV 版（商业图）  
5. 每套都附：负面词 + 参数建议 + 可选增强版 A/B
6. 禁止输出占位符（如 `{cn_t2i_prompt}`）；必须输出可直接复制的真实提示词。

**缺省场景兜底（信息不足时启用）：**
- 人物类：成年女性角色，夜雨霓虹城市街道，电影写实风格。
- 广告类：轻量消费品（如防晒喷雾/饮品）主视觉，留出标题和 CTA 文案区。
- 叙事类：从建立环境 -> 动作推进 -> 收束情绪的完整小闭环。

**首轮输出模板：**
```markdown
### 1) 文生图（T2I）
- 中文提示词:
  {cn_t2i_prompt}
- 英文提示词:
  {en_t2i_prompt}
- 负面词:
  {negative_prompt}
- 参数:
  --ar {ratio}, quality {2k|4k|8k}

### 2) 图生图（I2I）
- 单图修复版:
  {i2i_single_prompt}
- 双图一致性版:
  {i2i_dual_prompt}
- 参数:
  {i2i_params}

### 3) 宫格叙事（Grid）
- 9 格模板:
  {grid9_prompt}
- 25 格模板:
  {grid25_prompt}
- 参数:
  --ar 1:1, no text/hud/labels

### 4) 广告 KV
- 中文提示词:
  {cn_ad_prompt}
- 英文提示词:
  {en_ad_prompt}
- 参数:
  --ar 16:9

### 5) 模型与回退
- 首选模型: {model}
- 回退策略: {fallback}
```

**质量检查点：**
> 是否已在首轮输出中覆盖“文生图 + 图生图 + 9/25 宫格 + 广告KV”四类场景？

---

## 常见错误与修正

| 错误 | 表现 | 修正 |
|---|---|---|
| 模式混淆 | 把空镜写成人物海报 | 重新路由到 `MODE_ENVIRONMENT`，强制加入 `No people` |
| 角色漂移 | 同角色多图脸型变化 | 增加 `CHARACTER_LOCKED_FEATURES` 并前几帧重复 |
| 宫格不可解析 | 编号跳号/缺少 Row 标题 | 回写固定模板并强制 `Grid X:` |
| 文字污染 | 图中出现 KF/编号/HUD | 加强 `NO TEXT / NO LABELS / CLEAN IMAGE` 约束 |
| 风格冲突 | 动漫词与超写实词混用 | 主风格只保留一组，质感词单独控制 |
