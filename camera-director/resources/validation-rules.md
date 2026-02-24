# 验证规则详情

本文档定义了提示词输出的严格验证规则，确保输出质量和一致性。

---

## 核心验证规则 (5 项必检)

### 规则 1: Image 编号正确

**规则说明:**
- 所有人物/角色必须使用 `Image N` 格式标记
- 禁止使用 `Subject`, `Character`, `Person`, `Figure` 等通用词
- 编号必须与用户图片顺序对应

**正确示例:**
```
✅ Image 1 runs through the rain, shouting for help.
✅ Camera tracks alongside Image 1 as Image 2 pursues from behind.
✅ Image 1 and Image 2 embrace in the final shot.
```

**错误示例:**
```
❌ The subject runs through the rain...
❌ Character A pursues Character B...
❌ The man runs while the woman watches...
❌ A person stands alone on the platform...
```

**验证方法:**
```python
forbidden_terms = ["subject", "character", "person", "figure", "man", "woman", "protagonist"]
if any(term in prompt.lower() for term in forbidden_terms):
    return FAIL
if not re.search(r"Image \d+", prompt):
    return FAIL if has_human_figures else PASS
```

---

### 规则 2: 台词语言保留

**规则说明:**
- 用户输入的台词/对话必须保留原始语言
- 禁止翻译、意译或语言转换
- 可以添加语言标注说明

**正确示例:**
```
用户输入: 角色说 "快跑！"

✅ Image 1 shouts "快跑!" in panic.
✅ Image 1 exclaims "快跑!" (Chinese: Run!), urgency in voice.
✅ ...saying "快跑!" with desperation.
```

**错误示例:**
```
❌ Image 1 shouts "Run!" in panic. (台词被翻译)
❌ Image 1 says "Hurry!" (意译替换)
❌ Image 1 yells an urgent command. (台词被省略)
```

**验证方法:**
```python
original_dialogues = extract_dialogues(user_input)
output_dialogues = extract_dialogues(prompt)

for orig in original_dialogues:
    if orig not in prompt:
        return FAIL
```

---

### 规则 3: 无创意补充

**规则说明:**
- 所有内容必须来自用户输入
- 禁止添加用户未提及的元素
- 禁止推断、隐喻转换或"脑补"

**正确示例:**
```
用户输入: 赛博朋克街道，主角在雨中奔跑

✅ Cyberpunk street, neon lights, Image 1 running in rain.
✅ Rain-soaked cyberpunk alley, Image 1 sprints through puddles.
```

**错误示例:**
```
用户输入: 赛博朋克街道，主角在雨中奔跑

❌ Flying cars hover overhead... (用户未提及飞行汽车)
❌ Holographic advertisements flicker... (用户未提及全息广告)
❌ Image 1 runs from pursuing robots... (用户未提及机器人)
❌ Her cybernetic implants glow... (用户未提及植入物)
```

**允许的扩展:**
```
✅ 基于用户描述的合理视觉细节:
   - "雨中" → rain, puddles, wet pavement (合理)
   - "赛博朋克" → neon lights, urban setting (风格特征)
   - "奔跑" → running, sprinting, movement (动作描述)

❌ 不允许添加的新元素:
   - 新角色/人物
   - 新物体/道具
   - 新动作/事件
   - 具体剧情发展
```

**验证方法:**
```python
user_elements = extract_key_elements(user_input)
prompt_elements = extract_key_elements(prompt)

added_elements = set(prompt_elements) - set(user_elements)
# 过滤允许的扩展（视觉细节、风格特征）
if has_disallowed_additions(added_elements):
    return FAIL
```

---

### 规则 4: 结构正确

**规则说明:**
提示词必须遵循 4 段式结构:

```
[Header/电影感标签]

Camera Movement: [运镜描述]

Subject & Action: [主体动作 + 台词]

Environment & Mood: [环境氛围]
```

**正确结构:**
```
✅ Cyberpunk thriller, tracking shot, neon-lit rainy street, urgent dialogue.

Camera Movement: Camera rapidly tracks alongside Image 1 as they sprint through the rain.

Subject & Action: Image 1 runs desperately through puddles, shouting "快跑!" in panic.

Environment & Mood: Wet pavement reflects neon signs in pink and blue. Heavy rain. Dark, moody atmosphere.
```

**错误结构:**
```
❌ 缺少段落:
A tracking shot of someone running in the rain in a cyberpunk city.

❌ 格式混乱:
The camera follows Image 1 running. Neon lights everywhere. She shouts something.

❌ 段落顺序错误:
Environment first, then camera, then subject...
```

**验证方法:**
```python
required_sections = ["Camera Movement:", "Subject & Action:", "Environment & Mood:"]
for section in required_sections:
    if section not in prompt:
        return FAIL
```

---

### 规则 5: 长度限制

**规则说明:**
- 单镜头提示词: 5-8 行
- 分镜脚本每镜头: 4-6 行

**正确长度:**
```
✅ 6 行:
Cyberpunk thriller, tracking shot, neon-lit rainy street.

Camera Movement: Camera rapidly tracks alongside Image 1 through the rain.

Subject & Action: Image 1 runs desperately, shouting "快跑!" in panic.

Environment & Mood: Wet pavement, neon reflections, dark atmosphere.
```

**错误长度:**
```
❌ 过长 (12 行):
[过多的细节描述，冗余信息...]

❌ 过短 (2 行):
Tracking shot of person running in rain. Cyberpunk city.
```

**验证方法:**
```python
line_count = len(prompt.strip().split('\n'))
if mode == "single_shot":
    return PASS if 5 <= line_count <= 8 else FAIL
if mode == "storyboard":
    return PASS if 4 <= line_count <= 6 else FAIL
```

---

## 验证检查清单

在输出任何提示词之前，逐项检查:

```
□ 1. Image 编号正确
    □ 所有人物使用 Image N 格式
    □ 无 Subject/Character 等通用词
    □ 编号与用户图片对应

□ 2. 台词语言保留
    □ 所有台词为用户原始语言
    □ 无翻译或意译

□ 3. 无创意补充
    □ 所有内容来自用户输入
    □ 无新增人物/物体/事件
    □ 只有合理的视觉细节扩展

□ 4. 结构正确
    □ 包含 Header
    □ 包含 Camera Movement
    □ 包含 Subject & Action
    □ 包含 Environment & Mood
    □ 段落顺序正确

□ 5. 长度适当
    □ 单镜头: 5-8 行
    □ 分镜每镜头: 4-6 行
```

---

## 常见错误速查

| 错误类型     | 示例                             | 修正方法            |
| ------------ | -------------------------------- | ------------------- |
| 使用 Subject | "The subject runs..."            | → "Image 1 runs..." |
| 翻译台词     | "shouts 'Run!'" (原为中文)       | → "shouts '快跑!'"  |
| 添加新元素   | "flying cars hover" (用户未提及) | → 删除该内容        |
| 缺少段落     | 只有动作描述                     | → 补充 4 段式结构   |
| 过长         | 15 行详细描述                    | → 精简至 8 行内     |

---

## 自动修复建议

如果检测到问题，建议修复方案:

| 问题            | 自动修复                 |
| --------------- | ------------------------ |
| Subject →       | 替换为 Image 1           |
| Character A/B → | 替换为 Image 1/Image 2   |
| 翻译的台词      | 恢复原语言               |
| 结构缺失        | 补充缺失段落             |
| 过长            | 提示用户简化，或自动精简 |
| 创意补充        | 标记并请求用户确认       |
