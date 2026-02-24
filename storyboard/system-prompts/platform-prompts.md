# AI 视频平台提示词参考（Knowledge Only）

本文件仅保留平台能力、术语、结构与参数参考，不承载执行流程规则。

---

## 1. OpenAI Sora

### 1.1 常见结构
```text
[Scene Summary] + [Camera Shot/Angle] + [Camera Movement] +
[Subject & Action] + [Lighting & Color] + [Style & Mood]
```

### 1.2 术语参考
- 镜头类型：`close-up`, `extreme close-up`, `medium shot`, `wide shot`, `establishing shot`, `over-the-shoulder`
- 机位角度：`low angle`, `high angle`, `bird's eye view`, `dutch angle`, `eye-level`, `POV`
- 运动关键词：`pan`, `tilt`, `dolly in/out`, `tracking shot`, `crane`, `zoom`, `handheld`, `steadicam`, `drone shot`
- 运动效果：`slow-motion`, `time-lapse`, `motion blur`, `shallow depth of field`

### 1.3 示例
```text
A confident news anchor stands in a high-tech studio.
Medium shot, low angle. Camera slowly dollies in as she delivers breaking news.
Soft ambient lighting with blue accent tones.
Cinematic documentary style.
```

---

## 2. Runway Gen-3 / Gen-4

### 2.1 常见结构
```text
[Subject Motion] + [Scene Motion] + [Camera Motion] + [Style Descriptor]
```

### 2.2 术语与组件
- 角度：`low angle`, `high angle`, `overhead`, `FPV`, `handheld`, `wide angle`, `close up`
- 运动：`tracking`, `panning`, `horizontal movement`, `vertical movement`, `tilt`, `zoom`
- Gen-4 组件：
  - `Subject Motion`
  - `Camera Motion`
  - `Scene Motion`
  - `Style`

### 2.3 方向与速度提示
- 速度：`low speed`, `high speed`
- 构图：`negative space`, `open foreground`, `layered background`

### 2.4 示例
```text
Tracking shot: A cyclist on a coastal road, cliffs and ocean in background.
Golden hour lighting, wind in hair. Motion: Truck Right, low speed.
Cinematic, handheld stability.
```

### 2.5 常见参数
- `-gs [1-100]`: guidance scale
- `-neg [text]`: negative prompt
- `-ar [16:9/9:16/1:1]`: aspect ratio
- `-seed [number]`: reproducibility seed

---

## 3. Pika

### 3.1 常见控制词
- 相机：`zoom in/out`, `pan left/right`, `tilt up/down`, `rotate clockwise/counterclockwise`
- 质量控制：`-neg`, `-gs`, `-ar`, `-seed`

### 3.2 功能特性
- `Video-to-Video`
- `Expand Canvas`
- `Modify Region`

### 3.3 示例
```text
A majestic eagle soaring through dramatic mountain clouds at sunset.
Camera slowly follows from behind, golden light illuminating wings.
Cinematic, wildlife documentary style.
-neg blurry, distorted, low quality
-gs 12
-ar 16:9
```
