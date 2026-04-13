# AI 视频平台提示词参考

本文件仅保留各平台的推荐写法与术语映射。默认中文说明，英文只作为平台增强词。

---

## 1. OpenAI Sora

### 中文推荐结构
```text
场景概述 + 镜头类型/角度 + 镜头运动 + 主体动作 + 光影与色彩 + 风格氛围
```

### 使用建议
- 先描述整体场景，再补镜头与动作细节。
- 复杂动作建议按时间段拆开。
- 要写“变化过程”，不要只写静态结果。

---

## 2. Runway Gen-4

### 中文推荐结构
```text
主体动作 + 场景动作 + 镜头动作 + 风格说明
```

### 使用建议
- 直接、具体、少绕弯。
- 图生视频时，不重复图片里已经固定的静态信息。
- 优先强调运动变化，而不是堆风格词。

---

## 3. Seedance 2.0

### 中文推荐结构
```text
素材映射 + 任务类型 + 时长与画幅 + 中文主提示词 + 可选英文增强词
```

### 使用建议
- 先说明素材用途。
- 再写镜头主线和时序。
- 延长或编辑任务必须标明修改区间或新增秒数。

---

## 4. 通用中文术语与英文增强词

| 中文术语 | 英文增强词 |
| --- | --- |
| 远景 | `wide shot` |
| 中景 | `medium shot` |
| 近景 | `close-up` |
| 大特写 | `extreme close-up` |
| 平视 | `eye level` |
| 仰拍 | `low angle` |
| 俯拍 | `high angle` |
| 横摇 | `pan` |
| 纵摇 | `tilt` |
| 推镜 | `dolly in` |
| 拉镜 | `dolly out` |
| 跟拍 | `tracking shot` |
| 手持感 | `handheld` |
| 过肩镜头 | `over-the-shoulder` |
| 主观视角 | `POV` |

---

## 5. 中文主稿示例

```text
场景概述：一名女主播站在高科技演播室中，准备播报突发新闻。
镜头类型与角度：中景，轻微仰拍。
镜头运动：镜头缓慢推进。
主体动作：她看向镜头，短暂停顿后开始播报。
光影与色彩：环境光偏冷蓝，屏幕边缘有柔和高光。
风格氛围：专业、克制、纪录片式真实。
```

### 可选英文增强版示例

```text
Scene summary: A female news anchor stands in a high-tech studio, preparing to deliver breaking news.
Shot and angle: medium shot, slight low angle.
Camera motion: slow dolly in.
Subject motion: she looks into the camera, pauses briefly, then begins speaking.
Lighting and color: cool blue ambient light with soft edge highlights from the screens.
Mood: professional, restrained, documentary realism.
```
