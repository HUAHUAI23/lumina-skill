---
name: lumina-flow-shot-split-scene-as-shot
description: flow.story_to_video.shot_split 的派生版本；专门把"场景+若干镜头"紧凑分镜文本转成「1 场景 = 1 shot」的结构化 shot list，场景内多个镜头被压缩为 shot 内部的 `[00:00-00:02]` 时间戳硬剪辑块，直接对接 Seedance 2.0 一次成片多镜的能力。
---

# flow.story_to_video.shot_split / scene-as-shot

## 阶段定位

这是 story_to_video 的结构化编译阶段，与姊妹 `lumina-flow-shot-split` 并列、互斥使用。

- **姊妹 skill `lumina-flow-shot-split`**：处理 `consult.storyboard` 输出的导演级长稿（每镜一个 `### 镜头 NN：标题（约 X 秒）` 字段化小节）→ **1 镜头 = 1 shot**。
- **本 skill `lumina-flow-shot-split-scene-as-shot`**：处理 `consult.storyboard-script-to-shot-prompts` 输出的紧凑「场景+镜头」格式 → **1 场景 = 1 shot**，场景内多个镜头压缩为 shot 内部的 `[00:00-00:02]` 时间戳硬剪辑块。

两者输出结构不同，下游消费路径也不同：本 skill 的输出直接对接 Seedance 2.0 一次成片多镜调用（每个 shot 一次 Seedance 调用，shot 内部所有镜头作为 timestamp blocks 一次喂给模型）。

## 适用场景

- 输入是紧凑「场景 + 若干镜头」格式：
  - 顶部可选 `# 分镜文本` 总标题
  - 顶部可选单行 `角色锚点：A=...;B=...` 块
  - 顶部可选单行 `参考池：@slot=描述;@slot=描述...` 块（Universal Reference 12 槽位声明）
  - 场景标题 `场景X 名称 X秒` 或 `场景X 名称 X秒 衔接:类型(token1+token2)`（场景间衔接，第二个场景起强烈建议）
  - 场景下若干镜头行 `**镜头X X.Xs 时间: ... [景别/运镜]** 描述`
  - 镜头描述里可能内联 `(@char_main_xxx)` / `(@scene_xxx)` 参考池槽位引用
  - 镜头编号在整篇内连续递增，**不在每个场景里重置**
  - 镜头行可能内嵌 `角色（情绪、动作）"短台词"`，行尾可选 `｜避免:无字幕条/无后景人/...`（全角 `｜` 与半角 `|` 都识别）
  - 镜头行可能跟一行 `衔接:类型 模式:xxx 首帧:@xxx 重复:xxx`（Tier 2/3 长叙事强烈建议每镜都写）
- 用户目标是直接走 Seedance 2.0 一次成片路径。

## 不适用 / 走姊妹 skill

如果输入命中以下任一特征，**应该走 `lumina-flow-shot-split` 而不是本 skill**：

- 出现字段化栏目（`叙事目标：` / `画面叙事：` / `动作与调度：` / `视线与空间：` / `声音：` / `参考池引用：` / `衔接：` 等多行字段）。
- 出现 `### 镜头 NN：标题（约 X 秒）` 三级 heading 形式的镜头小节。
- 出现 `### 对话块 NN：标题（约 X 秒）`。
- 出现 `## 角色锚点` + 多行展开的 manifest 块。
- 出现 `## 视觉参考池` + 多行展开的槽位列表（图像槽 / 视频槽 / 音频槽分子段）。
- 出现 `## 连续性账本` 含 `道具状态机` / `光线连续矩阵` / `声音桥接计划` 多字段段。

发现这些特征时，提示用户「输入是导演级长稿格式，应使用 `lumina-flow-shot-split` 处理」，不要在本 skill 里硬拆。

## 核心映射规则

### 顶层结构

| 来源 | 目标 | 备注 |
|---|---|---|
| 顶部 `角色锚点：A=...;B=...` 单行块 | `continuity_ledger.character_anchors` map | 每个角色的视觉锚点字符串原样存 |
| 顶部 `参考池：@slot=描述;@slot=描述...` 单行块 | `reference_pack[]` 全局元数据 | 每项 `{slot, type, purpose, description}`；type 由 purpose 推断（char_*/scene/lighting/prop/style_palette → image，motion/transition → video，ambient/bgm/voice → audio） |
| 每个 `场景X 名称 X秒` | 1 个 shot | shot.title = 场景名称、shot.scene_index = X、shot.declared_duration = X秒 |
| 场景标题后追加的 `衔接:类型(token1+token2)` | shot.handoff（仅 transition_type + repeated_tokens 两字段） | 第一个场景没有；第二个场景起缺失 flag「场景 N 无场景间衔接，跨场景一致性可能下降」 |
| 场景内所有镜头时长之和 | shot.duration | 与 declared_duration 不一致时以镜头之和为准并 flag |
| 镜头行的 `时间: 白天/夜晚` | shot.time_of_day | 同场景所有镜头必须一致；不一致 flag |
| 镜头描述里内联的 `(@char_main_xxx)` / `(@scene_xxx)` 引用 | timeline_blocks[i].reference_refs[] | 提取 `@xxx` 槽位 ID；该 shot 级 reference_refs[] 为所有 block 引用的并集去重 |
| 镜头行下一行的 `衔接:类型 模式:xxx 首帧:@xxx 重复:xxx` | timeline_blocks[i].handoff（block 内衔接） | 写在镜头之间的硬切，按 block 级落字段 |
| 场景内所有镜头行尾的 `｜避免:...` | shot.negative_constraints[] | 按 `/` 拆条、聚合到 shot 级、去重；**全角 `｜` 与半角 `\|` 都识别** |

### shot 内部时间戳合成（核心）

把场景内每个镜头转成 shot 的 `timeline_blocks[]` 数组，按累计时间生成时间戳：

```
镜头1 2.0s [中景/固定] 描述...
镜头2 1.5s [特写/固定] 描述...
镜头3 1.5s [反应镜头/固定] 描述...
镜头4 2.0s [中近景/推镜] 描述...
镜头5 1.0s [插入镜头/固定] 描述...
镜头6 2.0s [中景/跟拍] 描述...
镜头7 2.0s [快切/手持晃动] 描述...
```

合成为一个 shot 的 `timeline_blocks`(含 reference_refs 与 handoff 示意):

```json
{
  "duration": 12.0,
  "scene_name": "保安室-玻璃后的微笑",
  "time_of_day": "夜晚",
  "scene_handoff": null,
  "reference_refs": ["@char_main_xiaomei", "@char_main_jiangye", "@scene_baoanshi", "@lighting_baoanshi", "@prop_keycard"],
  "timeline_blocks": [
    {
      "start": "00:00.0", "end": "00:02.0", "framing": "中景", "movement": "固定",
      "description": "保安室(@scene_baoanshi)内冷白色灯管微微闪烁,姜夜(@char_main_jiangye)坐在值班桌后,玻璃窗外隐约映出小美(@char_main_xiaomei)站在门口的身影。",
      "reference_refs": ["@char_main_xiaomei", "@char_main_jiangye", "@scene_baoanshi"]
    },
    {
      "start": "00:02.0", "end": "00:03.5", "framing": "特写", "movement": "固定",
      "description": "玻璃反光遮住姜夜(@char_main_jiangye)半边脸,他嘴角慢慢上扬,露出意味不明的微笑。",
      "dialogue": { "speakerRef": "姜夜", "tone": "玩味、轻声", "text": "\"奖励\"", "kind": "dialogue", "source": "verbatim" },
      "reference_refs": ["@char_main_jiangye"],
      "handoff": { "transition_type": "match_on_action", "seedance_input_mode": "text_to_video", "first_frame_ref": null, "repeated_tokens": ["@char_main_jiangye", "@scene_baoanshi"] }
    },
    {
      "start": "00:03.5", "end": "00:05.0", "framing": "反应镜头", "movement": "固定",
      "description": "玻璃外的小美(@char_main_xiaomei)听见这两个字,笑容更甜,微微歪头,眼神带着得逞的意味。",
      "reference_refs": ["@char_main_xiaomei"],
      "handoff": { "transition_type": "eyeline_match", "seedance_input_mode": "text_to_video", "first_frame_ref": null, "repeated_tokens": ["@char_main_xiaomei"] }
    },
    {
      "start": "00:05.0", "end": "00:07.0", "framing": "中近景", "movement": "推镜",
      "description": "镜头缓缓推向姜夜(@char_main_jiangye),他靠在椅背上嘿嘿一笑,眼神从慵懒变兴奋,伸手按向桌边开门按钮。",
      "dialogue": { "speakerRef": "姜夜", "tone": "兴奋、急切", "text": "\"那还等什么\"", "kind": "dialogue", "source": "verbatim" },
      "reference_refs": ["@char_main_jiangye"]
    },
    {
      "start": "00:07.0", "end": "00:08.0", "framing": "插入镜头", "movement": "固定",
      "description": "姜夜手指按下门禁(@prop_keycard)开关,按钮亮起绿光,保安室门锁发出\"咔哒\"一声轻响。",
      "sfx": ["门禁绿光提示音", "门锁咔哒"],
      "reference_refs": ["@prop_keycard"]
    },
    {
      "start": "00:08.0", "end": "00:10.0", "framing": "中景", "movement": "跟拍",
      "description": "小美(@char_main_xiaomei)推门走进来,脸上还挂着甜腻笑容,身体轻轻前倾。",
      "reference_refs": ["@char_main_xiaomei", "@scene_baoanshi"]
    },
    {
      "start": "00:10.0", "end": "00:12.0", "framing": "快切", "movement": "手持",
      "description": "姜夜(@char_main_jiangye)突然起身,动作带出残影,一只手猛地伸出,瞬间扣住小美(@char_main_xiaomei)的咽喉,将她压制在墙边。",
      "reference_refs": ["@char_main_xiaomei", "@char_main_jiangye"],
      "handoff": { "transition_type": "match_on_action", "seedance_input_mode": "text_to_video", "first_frame_ref": null, "repeated_tokens": ["@char_main_xiaomei", "@char_main_jiangye"] }
    }
  ]
}
```

时间戳规则：

- 第一个块从 `00:00.0` 开始。
- 后续块的 `start` = 上一块的 `end`，无空隙、无重叠。
- 时间戳用 `MM:SS.S`（**0.1s 精度**，与源镜头行 `X.Xs` 时长精度匹配；不要四舍五入到 0.5s 否则累加会 drift），与 Seedance 的 `[00:00-00:05]` 语法等价。
- shot.duration = 最后一个块的 `end`。
- 单 shot duration **必须 ≤ 15s**（Seedance 2.0 一次生成硬上限）；超出 flag 不擅自截断。

### 镜头行解析

镜头行格式：

```
**镜头X X.Xs 时间: 白天。** [景别/运镜] 描述...角色（情绪/语气）"短台词"...｜避免:负向约束1/负向约束2
```

逐字段解析：

- `**镜头X` → block.shot_index_in_full（整篇连续编号，跨场景不重置；本 skill 仅记录，不做强消费）
- ` X.Xs ` → block.duration（用于累加生成 start/end，0.1s 精度）
- `时间: XX。` → 校验与 shot.time_of_day 一致
- `[景别/运镜]` → block.framing + block.movement（按 `/` 拆）
  - 单镜**只允许一个主运镜**（push-in / pull-out / pan / tilt / tracking / orbit / aerial / handheld / locked-off / 固定），出现 `先推近再环绕` flag
  - **`fast` flag 作用域限定在 `[景别/运镜]` 字段全文 + 描述句里描述运镜的部分**（如 `相机 fast push-in` / `fast tracking`）；剧情/生理 `心跳加快` / `他比上次跑得更快` / 台词 `"快进来"` 不 flag
- 引号外的描述句 → block.description（保留原貌）
- **描述句里内联的 `(@xxx)` 槽位引用** → 提取所有 `@<purpose>_<name>` 形式 token，去重后存到 block.reference_refs[]；该 shot 级 reference_refs[] = 所有 block 引用并集去重
  - 提取的 `@xxx` 必须能在全局 `reference_pack[]` 找到；找不到 flag「shot N block M 引用未声明的参考池槽位 @xxx」
- 引号 + 情绪前缀的台词 → block.dialogue（按 lip-sync 三件套保护规则解析，下面单列）
- 行尾 `｜避免:...` 或 `|避免:...`（全角/半角分隔符都识别） → 聚合到 shot.negative_constraints[]，按 `/` 拆条

### 衔接行解析（block 内衔接，可选）

镜头行下一行如果出现 `衔接:类型 模式:xxx 首帧:@xxx 重复:xxx` 单行格式，按以下字段映射到该 block 的 handoff：

```
衔接:match_on_action 模式:image_to_video_first_frame 首帧:@last_frame_of_shot07 重复:@char_main_xiaomei+@scene_baoanshi
```

落到 `timeline_blocks[i].handoff`：

```json
{
  "transition_type": "match_on_action",
  "seedance_input_mode": "image_to_video_first_frame",
  "first_frame_ref": "@last_frame_of_shot07",
  "repeated_tokens": ["@char_main_xiaomei", "@scene_baoanshi"]
}
```

校验：

- `transition_type` 必须是 `hard_cut / match_on_action / eyeline_match / sound_bridge / j_cut / l_cut / graphic_match / concept_cut` 之一；非枚举 flag + 回退 `hard_cut`。
- `seedance_input_mode` 必须是 `text_to_video / image_to_video_first_frame / first_last_frame / ref_pack / nine_panel_grid_i2v / video_extend` 之一；非枚举 flag + 回退 `text_to_video`。
- 模式非 `text_to_video` 时 `首帧:` 必填，缺失 flag。
- `重复:` 拆 token 后必须 ≥1 个；空数组 flag「场景 N block M 与上一 block 共享 0 token」。
- 衔接行**缺失时第一个 block 不 flag**（场景内首镜默认 hard_cut+text_to_video）；同 shot 内后续 block 缺失 flag「block M 缺衔接，回退默认」。

### 场景标题衔接解析（场景间 handoff）

场景标题行可能带后缀：`场景X 名称 X秒 衔接:类型(token1+token2)`

例：`场景2 保安室-身份反转 13秒 衔接:sound_bridge(@ambient_rain+雨声延续)`

落到 `shot.scene_handoff`（注意是 shot 级，不是 block 级；表达"上一个 shot 整体 → 本 shot 整体"的衔接）：

```json
{
  "scene_handoff": {
    "transition_type": "sound_bridge",
    "repeated_tokens": ["@ambient_rain", "雨声延续"]
  }
}
```

校验：

- `transition_type` 同上 8 枚举值；非枚举 flag + 回退 `hard_cut`。
- 第一个场景没有上一场景，不要求 scene_handoff；第二个场景起缺失 flag「场景 N 无场景间衔接（scene_handoff），跨场景一致性可能下降，建议回 consult.storyboard-script-to-shot-prompts 补」。
- `concept_cut` 全片计数 >2 处 flag「全片 concept_cut N 处，叙事可能散架」。

### 台词解析（lip-sync 三件套保护）

格式：`角色（情绪/语气）"短台词"`（引号可以是 `"..."` / `「...」` / `『...』`，等价处理）

- `角色名` → `dialogue.speakerRef`
- `情绪/语气` → `dialogue.tone`（原样保留）
- 引号内文本 → `dialogue.text`，**保留引号原样**（Seedance 把引号识别为 lip-sync 触发器）
- `dialogue.kind` 默认 `dialogue`
- 出现 `VO / 旁白 / 内心 OS / 画外、画外音 / OS` 关键词时切到 `voiceover / inner_monologue / offscreen`，且这些类型引号可省。
- `dialogue.source`：原文逐字 `verbatim`；为时长容量轻微改写 `adapted`（**lip-sync dialogue 不允许 adapted**，超载就 flag）；shot_split 自己加的桥句 `new_bridge`（必须仍满足三件套）。**任何 `new_bridge` 都强制 flag**「shot N 新增桥句 new_bridge，未经用户确认」。
- **`说:` / `说道:` 前导识别**：上游 consult 规则禁用 `说:` 前导（引号才是 lip-sync 触发器），但兼容历史输入。如果镜头里出现 `角色（情绪）说:「台词」`，按 `verbatim` 解析（保留引号原样进 dialogue.text），但同时 flag「shot N 出现 `说:` 前导，建议回 consult.storyboard-script-to-shot-prompts 删除以避免 Seedance 把它当噪声」。
- 校验：单条 dialogue.text 引号内 ≤ 12 个汉字 / 8 个英文单词；超过 flag。

### 角色锚点 manifest 消费

紧凑格式只有一行：

```
角色锚点：小美=粉色泡泡袖针织衫/黑色短裙/黑色短直发齐下颌/右眉骨小痣/银色细项链;姜夜=黑色保安制服/短寸/左耳骨钉/右手腕黑色细绳手链
```

解析为：

```json
{
  "continuity_ledger": {
    "character_anchors": {
      "小美": "粉色泡泡袖针织衫/黑色短裙/黑色短直发齐下颌/右眉骨小痣/银色细项链",
      "姜夜": "黑色保安制服/短寸/左耳骨钉/右手腕黑色细绳手链"
    }
  }
}
```

人物引用规则同 `lumina-flow-shot-split`：

- 项目里有该角色稳定资产 → `characterRef` 用稳定资产名 + 同时带上 `anchor_descriptor`。
- 项目里没有 → **用 anchor_descriptor 作为 characterRef 临时占位**，让下游 Seedance 拿到一致性 token，并在质量说明里 flag「人物 X 缺项目资产，使用临时锚点占位」。
- 校验：每个 shot 的所有出场角色都必须在锚点 manifest 里有对应字符串；缺失 flag。

### 参考池单行 manifest 消费

紧凑格式只有一行：

```
参考池：@char_main_xiaomei=小美三视图;@char_main_jiangye=姜夜三视图;@scene_baoanshi=保安室主图;@lighting_baoanshi=冷白+红频闪光场;@prop_keycard=门禁卡特写;@ambient_rain=室内雨声 5s loop;@bgm_tension=压力 BGM 8s
```

解析为：

```json
{
  "reference_pack": [
    { "slot": "@char_main_xiaomei", "type": "image", "purpose": "char_main", "name": "xiaomei", "description": "小美三视图" },
    { "slot": "@char_main_jiangye", "type": "image", "purpose": "char_main", "name": "jiangye", "description": "姜夜三视图" },
    { "slot": "@scene_baoanshi",    "type": "image", "purpose": "scene",     "name": "baoanshi", "description": "保安室主图" },
    { "slot": "@lighting_baoanshi", "type": "image", "purpose": "lighting",  "name": "baoanshi", "description": "冷白+红频闪光场" },
    { "slot": "@prop_keycard",      "type": "image", "purpose": "prop",      "name": "keycard", "description": "门禁卡特写" },
    { "slot": "@ambient_rain",      "type": "audio", "purpose": "ambient",   "name": "rain",    "description": "室内雨声 5s loop" },
    { "slot": "@bgm_tension",       "type": "audio", "purpose": "bgm",       "name": "tension", "description": "压力 BGM 8s" }
  ]
}
```

`type` 由 `purpose` 推断:
- `char_main / char_outfit / char_face / scene / lighting / prop / style_palette` → `image`
- `motion / transition` → `video`
- `ambient / bgm / voice` → `audio`

校验:

- `image` 槽位 ≤ 9，`video` ≤ 3，`audio` ≤ 3；超出 flag「参考池超 Seedance Universal Reference 上限」。
- `purpose` 必须是 12 枚举值之一；非枚举 flag「参考池槽位 purpose 非标准」但仍存。
- 每个镜头描述里出现的 `(@xxx)` 引用必须能在 reference_pack[] 找到；找不到 flag「shot N 引用未声明的参考池槽位 @xxx」。
- ≥4 shot 但顶部没参考池单行 manifest 时按现行守恒模式继续，flag「≥4 shot 但缺参考池 manifest，跨镜一致性回退到纯 token 模式，建议回 consult.storyboard-script-to-shot-prompts 顶部补」。

## 工作方法

1. 先识别输入格式。命中"场景X 名称 X秒"标题 + "**镜头X X.Xs 时间: ... [景别/运镜]**" 行 → 进入本 skill 守恒模式；否则提示走姊妹 skill（详见 §不适用 / 走姊妹 skill 的特征列表）。
2. 解析顶部可选 manifest（最多三段）：
   - `角色锚点：A=...;B=...` 单行 → `continuity_ledger.character_anchors`
   - `参考池：@xxx=...;@xxx=...` 单行 → `reference_pack[]`（按上面 §参考池单行 manifest 消费规则）
3. 按 `场景X` 切分输入，每个场景独立处理为一个 shot。
4. 解析场景标题：提取场景编号、场景名称、声明时长；如果带 `衔接:类型(token)` 后缀，落 `shot.scene_handoff`。
5. 遍历场景内的镜头行：
   - 累加镜头时长（0.1s 精度）生成时间戳 `start / end`
   - 拆 `[景别/运镜]`
   - 提取描述、台词、SFX、负向约束（全/半角分隔符都识别）
   - 提取描述里内联的 `(@xxx)` 槽位引用，校验在 reference_pack 里
   - 校验单镜单主运镜、`fast` 仅在运镜字段 flag、禁焦距 jargon
   - 解析镜头行下一行的 `衔接:` 行（如有），落 `timeline_blocks[i].handoff`
6. 校验场景级约束：
   - 总时长 = 镜头时长之和；与声明时长不一致 flag
   - 总时长 ≤ 15s（Seedance 上限）；超出 flag
   - 总时长 ∈ [12, 15]（用户另行指定时除外）；不在范围 flag
   - time_of_day 同场景所有镜头一致；不一致 flag
7. 把场景内出场角色（描述里直接出现的、台词的 speakerRef、被反应镜头观看的）落到 shot.character_refs；按角色锚点占位规则处理资产匹配；同时把出场角色对应的 `@char_main_*` 槽位落到 shot.reference_refs[]（如果参考池里声明过）。
8. 聚合行尾负向约束到 shot.negative_constraints[]，去重。
9. 第二个场景起校验 `shot.scene_handoff` 存在性 + 枚举合法性 + token 非空；缺失 flag。
10. 生成最终 shot list，按场景顺序。
11. 输出质量说明：列出所有 flag、占位说明、容量警告、上下限警告、衔接缺失警告、参考池槽位引用未声明警告、`说:` 前导兼容警告。

## 镜头边界（场景内）

本 skill **不在场景内做镜头拆分** —— 场景内的镜头数和顺序是上游 `consult.storyboard-script-to-shot-prompts` 的导演决定，shot_split 守恒。

但下面情况要 flag：

- 单镜时长 > 5s：超过这个值时间戳合成意义不大（一个镜头几乎等于一个 shot），建议回 consult 拆开。
- 同场景出现两次相同景别相同主运镜的连续镜头（如 `[中近景/固定] → [中近景/固定]`）：可能是导演重复，flag 让用户检查。
- 场景内镜头编号不连续递增（跳号、重号）：原文有错，flag 不擅自补。

## 场景边界（跨 shot）

- 场景之间默认是硬切（场景标题本身就是一次 scene boundary）。下游 presenter 决定是否在场景间加 J-Cut/L-Cut/声音桥。
- 跨场景的人物站位、视线、手中道具不强制连续：场景之间允许跳位（用户写场景就意味着空间/时间可能变化）。
- 跨场景同名角色在 character_refs 里仍指同一资产/锚点，不要因为场景换了就当成新人物。

## 时长与颗粒度

- 每个场景声明时长应在 12-15s（默认）或用户指定值（写在用户提示里）。
- 单镜典型时长：环境镜头 1.5-2.5s / 表情特写 1.0-1.5s / 简单插入 0.8-1.2s / 对白行 2.0-3.5s / 突发动作 0.5-1.2s / 反转收口 1.5-2.5s。
- 不强制场景内时长平均；上游已经按节奏分配。
- 场景声明时长 vs 镜头之和不一致 → 以镜头之和为 shot.duration，并在质量说明 flag「场景 X 声明 12s 实际镜头之和 11.5s/14.5s」。

## 资产引用

- 同 `lumina-flow-shot-split` §资产引用规则：只引用项目稳定名称；缺资产时用 anchor_descriptor 占位 + flag。
- 紧凑格式没有显式道具字段，从镜头描述里抽取明显道具（门禁卡、监控屏、玻璃门、咖啡桌等），仅当项目里有稳定资产时才进 prop_refs；否则保留在 description 里。
- 场景名本身可能是一个场景资产（如「保安室-玻璃后的微笑」中的"保安室"）；项目里有对应场景资产 → scene_ref，否则只保留在 shot.scene_name。

## 下游约束与 meta 字段透传

紧凑格式的字段相对少，但仍有几个必须透传的：

| 来源 | 目标 | 备注 |
|---|---|---|
| 行尾 `｜避免:...` | shot.negative_constraints[]（聚合到 shot 级、去重） | **必须保留**，下游 Seedance prompt 会拼到 negative slot |
| 镜头描述里的 SFX 隐含信号（如"咔哒一声"、"灯管闪烁两下"） | timeline_blocks[i].sfx[] | 保留，下游可路由到 Seedance 原生音频 SFX 通道 |
| 镜头描述里的明暗交界线/色温信息（如"5600K 冷白荧光顶光"、"明暗交界线沿对角线分割"） | shot.lighting_geometry | 命名光场必须保留 |
| 镜头描述里的微表情链（如"先眨一下眼 → 喉结轻动 → 嘴角下压 0.5 秒"） | timeline_blocks[i].micro_expression_chain（数组） | 表情戏的关键信号，原样保留 |
| 镜头描述里的眼神锚（如"视线锁定 B 的左眼 1.6s"） | timeline_blocks[i].gaze_anchor | 表情戏的关键信号 |

**禁止改写、合并、压缩**这些字段。原文没有就留空，不硬编。

## 质量标准

- 输入格式判定正确：紧凑格式走本 skill，导演级长稿（多行 `## 角色锚点` / `## 视觉参考池` / 字段化栏目）提示走姊妹 skill，没有混判。
- 每个场景都被转成一个 shot；场景顺序、场景名称、场景编号一致。
- shot.duration = 内部镜头时长之和（0.1s 精度，无四舍五入 drift）；与声明时长不一致已 flag。
- 每个 shot.duration ≤ 15s（Seedance 一次生成硬上限）；超出已 flag。
- 每个 shot 的 timeline_blocks 时间戳无空隙、无重叠，从 00:00.0 起，最后一块 end = shot.duration。
- 每个 timeline_block 单镜只有一个主运镜；出现两个已 flag。
- `fast` flag 作用域已限定在 `[景别/运镜]` 字段全文 + 描述里运镜片段；剧情/生理/台词的 fast 用法没有误 flag。
- 没有出现焦距/光圈/景深/85mm/f/1.4 这种技术 jargon（Seedance 不响应）；出现已 flag。
- lip-sync 台词（kind=dialogue + 引号）：引号保留原样、tone 字段已填、单条 ≤ 12 个汉字 / 8 英文单词、source 不是 `adapted`；`new_bridge` 已强 flag；`说:` 前导已兼容解析 + flag。
- 角色锚点 manifest 已解析；所有 shot 出场角色都在锚点里能找到；缺项目资产的角色已用 anchor_descriptor 占位 + flag。
- **参考池单行 manifest 已解析**（≥4 shot 时）；reference_pack[] 三类槽位上限合规（image ≤9 / video ≤3 / audio ≤3）；镜头描述里 `(@xxx)` 引用都能在 reference_pack 找到，找不到已 flag。
- **场景间 handoff 已解析**：第二个场景起 scene_handoff 存在性 + transition_type 枚举合法性 + repeated_tokens 非空都已校验；缺失已 flag。
- **block 内衔接行已解析**（如有）：transition_type / seedance_input_mode 枚举合法、非 text_to_video 模式 first_frame_ref 已填、repeated_tokens 非空。
- `concept_cut` 全片计数 ≤2 处，超出已 flag。
- 行尾负向约束已聚合到 shot.negative_constraints[]，去重，全/半角分隔符都识别，没有任何条目被丢。
- 描述里的 SFX、命名光场、微表情链、眼神锚已落到对应字段，没合并、没压缩。
- shot list 顺序与原分镜场景顺序一致。
- 输出能直接喂给下游 Seedance 一次成片调用，不需要再猜原意。

## 失败回退

- 输入不像紧凑格式时（看到字段化栏目 / `### 镜头 NN：` / `### 对话块 NN：` / `## 角色锚点` 多行块 / `## 视觉参考池` 多行块 / `## 连续性账本` 多行块），提示「输入是导演级长稿格式，应使用 `lumina-flow-shot-split` 处理」，不在本 skill 里硬转。
- 场景声明时长 > 15s 时，flag「场景 X 时长 X.Xs 超过 Seedance 一次生成上限 15s，建议回 `consult.storyboard-script-to-shot-prompts` 拆成两个场景」，仍按原文落到结构化输出（shot.duration 用镜头之和），不擅自截断。
- 场景声明时长 < 12s 且用户没有显式指定短时长时，flag「场景 X 时长偏短，可能是上游格式漂移」，仍按原文输出。
- 单镜时长 > 5s 时，flag「场景 X 镜头 N 时长 X.Xs 偏长，时间戳合成意义不大，建议回 consult 拆开或考虑改用姊妹 skill」，仍按原文输出。
- lip-sync 台词超 12 字时，flag「场景 X 镜头 N 台词超载，建议回 consult 缩短」，**不擅自 adapt**。
- 顶部缺角色锚点 manifest 时，按现行守恒模式继续处理，flag「缺角色锚点 manifest，跨镜一致性可能下降，建议回 `consult.storyboard-script-to-shot-prompts` 顶部补」。
- ≥4 shot 但缺参考池 manifest 时，按现行守恒模式继续，flag「≥4 shot 但缺参考池单行 manifest，跨镜一致性回退到纯 token 模式（适用上限约 6 shot），10+ shot 长叙事强烈建议补」。
- 第二个场景起缺 scene_handoff 时，flag「场景 N 无场景间衔接（scene_handoff），跨场景一致性可能下降」，按 `hard_cut` 默认值落输出，不擅自补 token。
- 镜头行下一行的 `衔接:` 行解析失败（枚举值非法 / 模式非 text_to_video 但缺首帧 / token 为空）时，按默认值（`hard_cut + text_to_video`）落输出 + flag，不擅自修复。
- 镜头编号跳号/重号时，flag「场景 X 镜头编号不连续：1, 2, 4」，按原文顺序处理，不擅自补/改编号。

### scene-as-shot 路径的失败模式提示

本 skill 把场景压成单次 Seedance 多镜调用，特有的失败模式（用户应该被提醒）：

- **scene 内 drift**：单次成片内的中间镜头容易脸飘 / 服装跳 / 场景几何变。比 shot-as-shot（每镜独立调用）更难控制中段一致性。建议:scene 内镜头数 ≤6 + 强参考池 + 强 token 重复。
- **scene 边界硬切**：每个 scene 都是一次独立 Seedance 调用，scene 之间天然是硬切。如果剧情上需要 match_on_action / sound_bridge 衔接，必须在 scene_handoff 里声明，并配合下游用 `image_to_video_first_frame`（首帧 = 上一 scene 末帧）锁连续性。
- **15s 硬墙不可破**：scene 内总时长 > 15s 是 Seedance 单次生成的硬限，**不能靠 prompt 工程绕开**，只能拆 scene。

用户在 prompt 里碰到这些限制时，本 skill 不擅自修复，按原文落输出 + flag，提示用户回 consult 层决定路径。
