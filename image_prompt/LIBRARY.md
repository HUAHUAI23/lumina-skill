# Image Prompt Library（Preloaded）

本文件为系统预加载指令，不依赖工具读取。用于快速选择模型、风格、模板与质量优化策略。

---

## A. 模型版本速查（5.0 Lite / 4.5 / 4.0 / 3.1 / 3.0）

| 版本 | 定位 | 推荐任务 | 优势 | 主要局限 |
|---|---|---|---|---|
| 5.0 Lite | 知识推理 + 实时检索 | 热点图、长尾词、复杂语义场景 | 联网与推理能力强 | 贴图感、比例、小字稳定性偏弱 |
| 4.5 | 深度编辑 + 高美感 | 人像、广告图、产品图、组图一致性 | 编辑准确、结构稳定、质感好 | 成本/时延更高 |
| 4.0 | 高效低耗 | 批量探索、草图阶段、低成本产出 | 速度快、性价比高 | 小字和复杂编辑较弱 |
| 3.1 | 艺术美感 | 风格化插画、电影艺术图 | 光影层次和风格表达强 | 图文匹配与结构稳定下降 |
| 3.0 | 图文排版 | 海报文字、字体布局 | 文字和排版能力稳定 | 复杂推理与写实细节一般 |

**默认路由：**
1. 时效/检索/长尾知识 -> 5.0 Lite
2. 商业质感/高精编辑 -> 4.5
3. 低成本批量 -> 4.0
4. 强风格艺术化 -> 3.1
5. 文字排版任务 -> 3.0

**回退：**
- 5.0 Lite 质感不够 -> 4.5
- 4.5 太慢/太贵 -> 4.0
- 文字不稳 -> 3.0
- 风格表现不够 -> 3.1

---

## B. 风格库（25 Style Packs）

固定主体和构图，仅切换风格包可实现 9/25 风格批量图。

| # | 风格 | 中文关键词 | 英文关键词 |
|---|---|---|---|
| 01 | 电影写实 | 电影感、真实光影 | cinematic, photorealistic, film still |
| 02 | 商业棚拍 | 高级商业摄影、干净背景 | commercial studio, premium lighting |
| 03 | 新海诚风格 | 清透天空、逆光色彩 | Makoto Shinkai style, luminous sky |
| 04 | 动漫赛璐珞 | 清晰线稿、赛璐珞上色 | anime key visual, cel shading |
| 05 | 吉卜力风 | 温暖手绘、自然氛围 | Ghibli-inspired, hand-painted warmth |
| 06 | 鬼刀风格 | 唯美插画、戏剧光效 | GuDao-inspired illustration, dramatic light |
| 07 | 赛博朋克 | 霓虹雨夜、高对比 | cyberpunk, neon rain, high contrast |
| 08 | 蒸汽波 | 复古霓虹、梦幻渐变 | vaporwave, retro neon gradients |
| 09 | 黑白胶片 | 黑白颗粒、纪实质感 | monochrome film, grainy documentary |
| 10 | 复古 VHS | 扫描线、噪点 | 1980s VHS, noisy texture |
| 11 | 巴洛克油画 | 厚涂、强明暗 | baroque oil painting, chiaroscuro |
| 12 | 水彩插画 | 柔和笔触、纸张肌理 | watercolor illustration, paper texture |
| 13 | 国风水墨 | 留白、墨韵 | ink wash painting, oriental composition |
| 14 | 古风写意 | 古典服饰、诗意构图 | Chinese classical aesthetic, poetic frame |
| 15 | 像素艺术 | 像素块、复古游戏感 | pixel art, retro game style |
| 16 | 低多边形 | 几何切面 | low-poly 3D style |
| 17 | 粘土动画 | 手作质感 | claymation, handcrafted texture |
| 18 | 皮克斯感3D | 友好光影、动画电影感 | Pixar-like 3D render |
| 19 | Unreal Engine 5 | 次世代实时渲染 | Unreal Engine 5 render, global illumination |
| 20 | Octane 渲染 | 高反射材质、电影级渲染 | Octane render, hyper-detailed materials |
| 21 | C4D 产品图 | 几何构成、品牌视觉 | C4D product render, design-forward |
| 22 | 建筑可视化 | 结构真实、透视准确 | architectural visualization, realistic scale |
| 23 | 科幻概念图 | 巨构场景、体积光 | sci-fi concept art, volumetric lighting |
| 24 | 奇幻概念图 | 史诗场景、魔法光效 | fantasy concept art, epic atmosphere |
| 25 | 极简平面 | 几何、留白、品牌配色 | minimal graphic design, clean layout |

---

## C. 画质包（Quality Packs）

| 类型 | 中文关键词 | 英文关键词 |
|---|---|---|
| 细腻 | 细节丰富、材质清晰 | fine details, micro texture, crisp edges |
| 拟实 | 真实皮肤、物理光照 | photorealistic skin, physically accurate lighting |
| 8K | 超清、锐利对焦 | 8k, ultra-detailed, sharp focus |
| CG | 全局光照、体积雾 | CG render, global illumination, volumetric fog |
| 广告商业 | 高级灯光、商业精修 | premium commercial lighting, polished look |

---

## D. 人物描述词库

- 年龄段：青年 / 轻熟 / 成熟（默认成人）
- 区域特征：亚洲女性、欧美女性、拉美风格、混血气质
- 气质：优雅、冷艳、甜美、知性、力量感、未来感
- 服装：礼服、职业装、休闲装、运动装、战术服、内衣风造型
- 材质：丝绸、皮革、棉麻、金属、透明硬纱、亮片
- 配饰：耳饰、项链、戒指、眼镜、头饰、手套、高跟鞋

**边界：**
- 涉及性感或内衣风时必须明确成人（adult woman/adult model）。
- 不输出未成年人性感化或露骨性内容。

---

## E. 模板速查

### E1 文生图（T2I）
```text
[风格]，[主体]，[动作/状态]，[环境背景]，[光影氛围]，[构图镜头]，[画质修饰] --ar {ratio}
```
```text
[Style], [Subject], [Action/State], [Environment], [Lighting], [Composition/Camera], [Quality] --ar {ratio}
```

### E2 图生图（I2I）
```text
Use image A as strict identity reference and image B as composition guide. Keep face/hair/outfit consistent with image A, follow pose/framing from image B, remove all text/labels/watermarks, output high-fidelity clean result.
```

### E3 角色定妆（无动作）
```text
[Style], [Character Name], [Age/Role], [Appearance], [Outfit], full body shot, standing pose, front view, neutral expression, character sheet, flat lighting, simple background --ar 9:16
```

### E4 场景空镜（无人）
```text
[Style], [Location], [Environment Details], [Lighting Mood], no people, nobody, empty scene, panoramic view, wide angle, landscape --ar 16:9
```

### E5 宫格叙事（9/16/25）
```text
Strictly reference the character design, colors, and environment from the attached reference image.
[Sequence Header + Grid 1..N + Row sections + Parameters]
```

### E6 广告图（KV）
```text
[Commercial Style], [Product Hero with USP], [Audience Scenario], [Brand Color and Lighting], copy space for headline and CTA, premium advertising key visual --ar {ratio}
```

---

## F. 最新优化规则（2025-2026）

### F1 结构化提示词优先
- 先写“任务目标 + 硬约束 + 可变风格 + 输出参数”，再写细节描述。
- 复杂任务优先结构化模板（Header/Body/Parameters），降低漏项。

### F2 分层描述法（BFL 官方实践）
按层写：
1. Subject（主体）
2. Style（风格）
3. Composition（构图）
4. Lighting（光影）
5. Color（色彩）
6. Technical（镜头/质量）

### F3 一致性优先于堆词
- 使用固定“Identity Anchor Block”在多图任务中逐条复用。
- 先锁主体身份，再扩展动作/镜头变化。

### F4 负面词兼容矩阵
- `FLUX.2`：官方建议不使用 negative prompt，改用正向约束。
- `Imagen 3/4 新版`：Google 标注 negative prompt 为 legacy/不再支持新版本。
- 其余模型：只用“目标缺陷定向负面词”，避免过长黑名单。

### F5 平台一致性参数
- Midjourney V7：
  - 风格一致：`--sref` + `--sw`
  - 角色/物体一致：`--oref` + `--ow`
- OpenAI：先生成再编辑，迭代时传前一轮图像 ID 提升连续性。

### F6 质量流水线
1. 低成本构图稿（先验证主体/构图/风格方向）
2. 质量精修稿（加材质与光影细节）
3. 放大与清理（去文字污染/水印/HUD）

### F7 评估闭环
- 对高频场景建立 10-20 条评测样本。
- 固定评分项：主体一致性、构图准确、风格一致、文字污染、可用率。
- 小步迭代提示词模板并保留 A/B 版本。

### F8 缓存友好编排
- 固定模板和系统规则作为前缀，用户变量置于后缀。
- 同类请求保持前缀稳定，提升 API 提示词缓存命中率。

---

## G. 通用负面词

```text
lowres, blurry, deformed anatomy, bad hands, broken fingers, extra limbs, bad proportions, duplicated face, text, watermark, logo, caption, subtitle, HUD, overexposed, oversaturated
```

---

## H. 规则来源索引（2026-02）

- OpenAI Image Generation Guide（生成与编辑 API、参数与流程）
- OpenAI Prompt Optimization Guide（提示词迭代与评估闭环）
- OpenAI Prompt Caching Guide（静态前缀缓存思路）
- Black Forest Labs Prompting Guide（分层写法、HEX 颜色锚定、FLUX.2 负面词建议）
- Midjourney Docs: Aspect Ratio / Style Reference / Omni Reference（`--ar`、`--sref`、`--sw`、`--oref`、`--ow`）
- Google Vertex Imagen Prompt Guide + Negative Prompt Note（新版 negative prompt 兼容性）
- CVPR 2025 Prompting Patterns for Text-to-Image Generation（结构化提示词模式）
- ICLR 2025 STEPS（基于语义分解的提示词增强）

---

## I. 首轮默认预设（用户仅确认“需要”时）

- 默认模型：`4.5`
- 默认风格：电影写实 + 广告商业质感
- 默认输出：`T2I + I2I + Grid9/25 + Ad KV`
- 默认回退：`4.5 -> 4.0 -> 3.1 -> 3.0`（按速度/风格/文字稳定需求选择）

---

## J. 首轮四套提示词包（真实示例）

### J1 文生图（T2I）示例

- 中文提示词：  
  电影写实风格，成年亚洲女性，黑色短风衣与高筒靴，站在夜雨霓虹街道中央，地面有倒影与薄雾，远处车灯形成散景，冷暖对比光，三分法构图，中景镜头，细腻皮肤与服装材质，商业级画质 --ar 9:16

- 英文提示词：  
  cinematic photorealistic style, adult Asian woman, black short trench coat and knee-high boots, standing in a neon rainy street at night, reflective wet ground and light fog, distant car-light bokeh, teal-orange contrast lighting, rule-of-thirds composition, medium shot, fine skin and fabric texture, premium commercial quality --ar 9:16

- 负面词：  
  lowres, blurry, deformed anatomy, bad hands, extra fingers, bad proportions, duplicated face, text, watermark, logo, HUD

- 参数建议：  
  model `4.5`, first pass `2K`, final `4K/8K`

### J2 图生图（I2I）示例

- 单图修复版：  
  Upscale this exact image while preserving composition and pose. Keep the character identity unchanged, restore fine texture on face and clothing, remove all text labels, watermark, and HUD overlays, keep natural lighting and background details.

- 双图一致性版：  
  Use image A as strict identity and lighting reference, and image B as composition and pose guide. Keep the same face structure, hair style, eye color, and outfit details from image A. Recreate framing from image B. Remove any text, labels, numbers, and watermark. Output clean high-fidelity photorealistic result.

- 参数建议：  
  model `4.5`, denoise low to medium, preserve identity priority high

### J3 宫格叙事（Grid9 / Grid25）示例

- 9 格开头模板：  
  Strictly reference the character design, colors, and environment from the attached reference image.  
  [Sequence Header] Visual Style: Cinematic Realism; Genre: Urban Chase; Scope: Short Action Beat; Aspect Ratio: 1:1; Total Grids: 9  
  --- Row 1 (Setup) ---  
  Grid 1: Wide establishing shot, rainy neon street, protagonist appears at frame left, camera slightly high angle, city ambience building.  
  Grid 2: Medium shot, protagonist turns and notices movement behind, shoulder tension rises, neon reflections intensify.  
  Grid 3: Close-up, determined eyes and rain droplets, shallow depth of field, background lights blur.  
  [continue Grid 4-9 with continuous action]  
  [Parameters] --ar 1:1 --v 6.0

- 25 格结构说明：  
  使用 5 行相位：Start / Progression / Climax-Impact / Reaction-Fallout / Conclusion；每格都包含 Action + Camera + Env，并严格连续编号 `Grid 1..25`。

- 统一约束：  
  no text, no labels, no numbers, no HUD, clean image only.

### J4 广告 KV（Ad）示例

- 中文提示词：  
  高级商业广告风格，防晒喷雾产品主视觉，磨砂半透明瓶身悬浮于清爽水雾场景，背景为浅蓝渐变与柔和阳光斑，前景有微小水珠高光，突出清透防晒与轻薄不黏腻卖点，右上角预留标题区，底部预留 CTA 区域，商业摄影级精修质感 --ar 16:9

- 英文提示词：  
  premium commercial advertising style, sunscreen spray product hero, frosted semi-transparent bottle floating in a fresh misty scene, soft blue gradient background with gentle sunlight highlights, micro water droplets in foreground, emphasize lightweight non-sticky UV protection benefit, copy space on top-right for headline and bottom area for CTA, high-end retouched product photography --ar 16:9

- 负面词：  
  blurry text, distorted bottle shape, messy reflections, oversaturated highlights, watermark, logo artifacts

- 参数建议：  
  model `4.5`, product texture priority high, highlight control medium
