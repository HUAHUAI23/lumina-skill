# 单镜头工作流程 (模式 A)

本工作流用于生成单个视频片段的专业运镜提示词。

---

## 触发条件

- 图片数量 ≤ 2
- 用户描述单一场景
- 关键词: 这张图、这个画面、单个镜头

---

## 工作流程概览

```
用户输入（文本 + 图片 + 台词）
         ↓
┌─────────────────────────────────────┐
│  Step 1: 分析与约束判断              │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  Step 2: 运镜推荐生成               │
└─────────────────────────────────────┘
         ↓
    用户选择（或使用推荐的首选）
         ↓
┌─────────────────────────────────────┐
│  Step 3: 提示词生成 + 验证          │
└─────────────────────────────────────┘
         ↓
    输出: 单段视频提示词
```

---

## Step 1: 分析与约束判断

### 输入
```
- 用户描述文本
- 参考图片 (0-2 张)
- 台词/对话 (可选)
```

### 处理逻辑

#### 1.1 意图识别

```
IF 用户明确指定运镜类型:
   - 关键词: 推进、拉远、环绕、横摇、dolly、pan、orbit
   → intent_type = "explicit"
   → 锁定该运镜，优化修饰语
   
ELSE:
   → intent_type = "implicit"
   → 从知识库推荐 3-4 种方案
```

#### 1.2 语言检测

```
检测优先级: 台词语言 > 描述语言

IF 包含中文字符:
   → language_code = "ZH_CN"
ELIF 包含日文字符:
   → language_code = "JA"
ELSE:
   → language_code = "EN"

重要: language_code 决定台词输出语言
```

#### 1.3 视觉约束评估 (图生视频)

```
分析图片特征，确定约束:

IF 极近景/大特写:
   → blocked: ["Fast Dolly In", "Rapid Push In"]
   → warning: "⚠️ 避免面部畸变"

IF 主体占满边缘:
   → blocked: ["Wide Pan", "Fast Tilt"]
   → warning: "⚠️ 防止背景幻觉"

IF 2D 插画/平面:
   → blocked: ["Complex Orbit", "3D Arc"]
   → warning: "⚠️ 保持平面艺术感"
```

#### 1.4 图片编号映射

```
FOR i, image IN enumerate(images, start=1):
   mapping[f"img_{i:03d}"] = f"Image {i}"

示例输出:
{
  "img_001": "Image 1",
  "img_002": "Image 2"
}
```

### 输出

```json
{
  "intent_type": "implicit",
  "language_code": "ZH_CN",
  "visual_constraints": {
    "allowed": ["Tracking Shot", "Dolly In"],
    "blocked": ["Fast Push In"],
    "warnings": ["⚠️ 动作场景建议动态运镜"]
  },
  "image_mapping": {
    "img_001": "Image 1"
  },
  "detected_dialogues": [
    {"speaker": "角色A", "text": "快跑！"}
  ]
}
```

---

## Step 2: 运镜推荐生成

### 输入
```
- Step 1 的分析结果
- 用户描述
- 系统提示词知识包（`../system-prompts/knowledge-base.md`）
```

### 处理逻辑

#### 2.1 推荐策略

```
IF intent_type == "explicit":
   # 用户已指定运镜，生成 1-2 个优化变体
   → 优化速度修饰词
   → 添加风格建议
   
ELSE (implicit):
   # 从知识库匹配 3-4 种不同维度方案
   → 按主体类型匹配
   → 按情感氛围匹配
   → 应用视觉约束过滤
   → 按适配度排序
```

#### 2.2 匹配逻辑

```
# 提取场景特征
scene_features = {
  "subject_type": "人物全身/动作",
  "mood": "紧张",
  "environment": "街道",
  "action": "奔跑"
}

# 从匹配矩阵获取候选
candidates = get_from_matrix(scene_features)

# 过滤受约束的运镜
candidates = filter_blocked(candidates, visual_constraints)

# 计算适配度
FOR candidate IN candidates:
   candidate.suitability = calculate_score(candidate, scene_features)

# 排序返回前 3-4 个
return sorted(candidates)[:4]
```

### 输出

```json
{
  "analysis_summary": "场景为赛博朋克追逐，推荐动态跟随运镜增强临场感",
  "recommended_modes": [
    {
      "category": "复合运镜",
      "en_term": "Tracking Shot",
      "cn_label": "跟拍镜头",
      "suitability": "High",
      "reason": "最适合表现奔跑动作，持续跟随主体"
    },
    {
      "category": "复合运镜",
      "en_term": "FPV",
      "cn_label": "主观视角",
      "suitability": "High",
      "reason": "第一人称视角强化代入感"
    },
    {
      "category": "基础运镜",
      "en_term": "Fast Truck",
      "cn_label": "快速平移",
      "suitability": "Medium",
      "reason": "横向展现环境，速度感强"
    }
  ]
}
```

---

## Step 3: 提示词生成 + 验证

### 输入
```
- 用户选择的运镜 (或使用首选)
- 分析结果 (language_code, image_mapping)
- 用户描述和台词
```

### 处理逻辑

#### 3.1 构建 4 段式结构

```
# 1. Header (电影感标签)
header = build_header(
  movement=selected_movement,
  scene_type=detect_scene_type(description),
  has_dialogue=bool(dialogues)
)
# 示例: "Cyberpunk thriller, tracking shot, neon-lit street, urgent dialogue."

# 2. Camera Movement
camera = build_camera_movement(
  movement=selected_movement,
  speed=speed_modifier,
  image_mapping=image_mapping,
  description=description
)
# 示例: "Camera rapidly tracks alongside Image 1 as they sprint through the rain."

# 3. Subject & Action (关键: 保留原语言台词!)
subject = build_subject_action(
  dialogues=dialogues,
  image_mapping=image_mapping,
  language_code=language_code,
  description=description
)
# 示例: "Image 1 runs desperately, shouting "快跑!" in panic."

# 4. Environment & Mood
environment = build_environment(
  description=description,
  movement=selected_movement
)
# 示例: "Wet pavement reflects neon signs. Heavy rain. Dark, moody atmosphere."
```

#### 3.2 组装提示词

```
final_prompt = f"""{header}

Camera Movement: {camera}

Subject & Action: {subject}

Environment & Mood: {environment}"""
```

#### 3.3 严格验证

```
validation_checks = []

# 检查 1: Image 编号
IF "Subject" in prompt OR "Character" in prompt:
   checks["image_numbering"] = "FAIL"
ELSE:
   checks["image_numbering"] = "PASS"

# 检查 2: 台词语言
FOR dialogue IN original_dialogues:
   IF dialogue.text NOT IN prompt:
      checks["dialogue_language"] = "FAIL"

# 检查 3: 无创意补充
user_elements = extract_elements(description)
prompt_elements = extract_elements(prompt)
IF has_new_elements(prompt_elements - user_elements):
   checks["no_creative_supplement"] = "FAIL"

# 检查 4: 结构
required = ["Camera Movement:", "Subject & Action:", "Environment"]
IF NOT all(r in prompt for r in required):
   checks["structure"] = "FAIL"

# 检查 5: 长度
IF NOT (5 <= line_count <= 8):
   checks["line_count"] = "FAIL"

passed = all(v == "PASS" for v in checks.values())
```

### 输出

```json
{
  "final_prompt": "Cyberpunk thriller, tracking shot, neon-lit rainy street, urgent dialogue.\n\nCamera Movement: Camera rapidly tracks alongside Image 1 as they sprint through the rain-soaked cyberpunk street, handheld style adding urgency.\n\nSubject & Action: Image 1 runs desperately through puddles, shouting \"快跑!\" in panic, rain streaming down their face.\n\nEnvironment & Mood: Wet pavement reflects vibrant neon signs in pink and blue. Heavy rain, steam rising from vents. Dark, moody atmosphere with high contrast lighting.",
  
  "validation": {
    "passed": true,
    "checks": {
      "image_numbering": "PASS",
      "dialogue_language": "PASS",
      "no_creative_supplement": "PASS",
      "structure": "PASS",
      "line_count": "PASS"
    }
  }
}
```

---

## 完整示例

### 输入

```
用户描述: "赛博朋克街道，霓虹灯闪烁，主角在雨中奔跑"
图片: [cyberpunk_street.jpg]
台词: [{"speaker": "角色A", "text": "快跑！"}]
```

### Step 1 输出

```
意图类型: implicit (未指定具体运镜)
语言: ZH_CN
约束: 无特殊约束
图片映射: {"img_001": "Image 1"}
```

### Step 2 输出

```
📊 分析: 动作场景，推荐动态运镜

推荐 1: Tracking Shot (跟拍) - High
推荐 2: FPV (主观视角) - High  
推荐 3: Fast Truck (快速平移) - Medium
推荐 4: Whip Pan (甩镜头) - Medium
```

### Step 3 输出 (选择 Tracking Shot)

```
Cyberpunk thriller, tracking shot, neon-lit rainy street, urgent dialogue.

Camera Movement: Camera rapidly tracks alongside Image 1 as they sprint through the rain-soaked cyberpunk street, handheld style adding urgency.

Subject & Action: Image 1 runs desperately through puddles, shouting "快跑!" in panic, rain streaming down their face.

Environment & Mood: Wet pavement reflects vibrant neon signs in pink and blue. Heavy rain, steam rising from vents. Dark, moody atmosphere with high contrast lighting.

---
✅ 验证通过: 5/5 项检查全部通过
```

---

## 系统提示词依赖（非 resources）

- `../system-prompts/knowledge-base.md`
- `../system-prompts/platform-prompts.md`
- `../system-prompts/validation-rules.md`
