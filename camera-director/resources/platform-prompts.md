# AI 视频生成平台提示词指南

本文档包含主流 AI 视频生成平台的提示词结构和最佳实践。

> **动态学习提示**: 用户提到特定平台时，应搜索 `[平台名] prompt guide [当前年份]` 获取最新指南。

---

## OpenAI Sora

### 提示词结构（6 层框架）

```
[Scene Summary] + [Camera Shot/Angle] + [Camera Movement] + 
[Subject & Action] + [Lighting & Color] + [Style & Mood]
```

### 关键词参考

#### 镜头类型
- `close-up`, `extreme close-up`, `medium shot`, `wide shot`
- `establishing shot`, `over-the-shoulder`, `insert shot`
- `full body shot`, `long shot`

#### 机位角度
- `low angle` - 使主体显得强大
- `high angle` - 使主体显得渺小
- `bird's eye view` - 正上方俯视
- `dutch angle` - 倾斜构图，营造不安
- `eye-level` - 中立视角
- `POV` / `first person` - 主观视角

#### 运动关键词
- `pan left/right`, `tilt up/down`
- `dolly in/out`, `tracking shot`
- `crane up/down`, `jib shot`
- `zoom in/out`, `handheld`, `steadicam`
- `drone shot`, `aerial view`

#### 特殊效果
- `slow-motion`, `time-lapse`
- `motion blur`, `shallow depth of field`

### 示例提示词

```
A confident news anchor stands in a high-tech studio. 
Medium shot, low angle. Camera slowly dollies in as she 
delivers breaking news about emerging AI trends. 
Soft ambient lighting with blue accent tones. 
Cinematic documentary style, professional atmosphere.
```

### 最佳实践
1. **简洁概览开头** - 先用一句话概括整个场景
2. **单一动作原则** - 每个镜头只描述一个主要动作
3. **时间节拍** - 复杂场景按时间段拆分 `[0-2s]`, `[2-4s]`
4. **120 词以内** - 保持简洁，避免过度复杂

---

## Runway Gen-3 / Gen-4

### 提示词结构

```
[Subject Motion] + [Scene Motion] + [Camera Motion] + [Style Descriptor]
```

### Gen-3 关键词

#### 镜头角度
- `low angle`, `high angle`, `overhead`
- `FPV`, `handheld`, `wide angle`
- `close up`, `macro cinematography`
- `static shot`, `establishing wide`
- `over the shoulder`

#### 运动控制
- `tracking`, `panning`
- `horizontal movement` (left/right)
- `vertical movement` (up/down)
- `pan` (horizontal rotation)
- `tilt` (vertical rotation)
- `zoom` (in/out)
- `dynamic`, `smooth`

#### 速度/效果
- `dynamic motion`, `slow motion`
- `hyperspeed`, `timelapse`
- `grows`, `emerges`, `explodes`
- `transforms`, `ripples`, `shatters`

### Gen-4 增强功能

#### 核心组件
- **Subject Motion**: 主体动作 (`runs`, `dances`, `jumps`, `walks`, `glances`)
- **Camera Motion**: 镜头运动 (`tracking`, `panning`, `dolly`, `orbit`)
- **Scene Motion**: 环境动态 (`wind rustles`, `lights flicker`)
- **Style**: 风格 (`cinematic`, `vintage`, `handheld`)

#### 速度控制
- `low speed` - 边缘更清晰
- `high speed` - 可能产生抖动

#### 方向提示
- `negative space` - 留白方向
- `open foreground` - 开放前景
- `layered background` - 层次背景

### 示例提示词

**文本生视频:**
```
[camera movement]: [establishing scene]. [additional details]

Tracking shot: A cyclist on a coastal road, cliffs and ocean in background.
Golden hour lighting, wind in hair. Motion: Truck Right, low speed. 
Cinematic, handheld stability.
```

**图片+文本:**
```
Camera slowly dollies forward as the subject turns to look at camera.
Gentle wind movement in hair and clothes. Cinematic warm lighting.
```

### 参数控制
- `-gs [1-100]` - 引导强度 (guidance scale)
- `-neg [text]` - 负面提示词
- `-ar [16:9/9:16/1:1]` - 宽高比
- `-seed [number]` - 一致性种子

---

## Pika Labs

### 核心技巧

1. **高度具体化** - 使用丰富描述词"画画面"
2. **负面提示词** - 主动用 `-neg` 排除不想要的元素
3. **参考图片** - 结合图片引导生成更精确
4. **参数精调** - 利用 `-gs`, `-ar`, `-seed` 控制

### 相机控制

#### 运动参数
- `zoom in` / `zoom out`
- `pan left` / `pan right`
- `tilt up` / `tilt down`
- `rotate clockwise` / `rotate counterclockwise`

#### motion_control 参数
- 用于精确动画控制
- 可指定运动幅度和速度

### 示例提示词

```
A majestic eagle soaring through dramatic mountain clouds at sunset.
Camera slowly follows from behind, golden light illuminating wings.
Cinematic, wildlife documentary style.
-neg blurry, distorted, low quality
-gs 12
-ar 16:9
```

### 高级功能
- **Video-to-Video**: 转换现有视频风格
- **Expand Canvas**: 外扩画面边界
- **Modify Region**: 局部区域修改

---

## 通用最佳实践

### 结构化方法

```
1. 主体 (Subject): 谁/什么
2. 动作 (Action): 在做什么
3. 景别 (Shot): 镜头距离
4. 运镜 (Movement): 镜头如何移动
5. 光影 (Lighting): 光线质感
6. 风格 (Style): 整体美学
```

### 写作原则

| 原则         | 说明                         | 示例                                         |
| ------------ | ---------------------------- | -------------------------------------------- |
| **正面描述** | 描述想要什么，而非不想要什么 | ✅ "clear sky" ❌ "no clouds"                  |
| **具体明确** | 避免模糊词汇                 | ✅ "golden hour sunset" ❌ "nice lighting"     |
| **单一运动** | 每镜头一个主运动             | ✅ "dolly in" ❌ "dolly in while panning left" |
| **专业术语** | 使用电影语言                 | ✅ "tracking shot" ❌ "camera follows"         |
| **迭代优化** | 从简单开始，逐步添加         | 先测试核心元素，再加细节                     |

### 常见错误

| 错误         | 问题              | 修正                  |
| ------------ | ----------------- | --------------------- |
| 过长的提示词 | 模型难以处理      | 控制在 100-150 词     |
| 对话式语言   | AI 不是聊天机器人 | 使用陈述句，非请求句  |
| 冲突的指令   | 运动矛盾          | 确保指令一致          |
| 忽略时间     | 复杂场景混乱      | 使用时间节拍 `[0-3s]` |
