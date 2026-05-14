---
name: lumina-flow-shot-split
description: flow.story_to_video.shot_split 的补充提示词；用于把已确认分镜文本忠实结构化为连续 shot list，并做必要的镜头边界、容量和连续性整理
---

# flow.story_to_video.shot_split

## 阶段定位

这是 story_to_video 的结构化编译阶段，不是导演创作阶段。

上游 `consult.screenplay` 负责故事结构、人物动机、情节铺垫、信息差和台词；`consult.storyboard` 负责导演级分镜、镜头语言、剪辑、声音和节奏。当前阶段只把用户已经确认的分镜文本整理成后续流程可消费的连续 shot list。

## 适用场景

- 用户已经通过 `consult.storyboard` 确认一份分镜文本，需要进入 story_to_video。
- 输入可能包含：镜头编号（`### 镜头 NN：标题（约 X 秒）`）、对话块（`### 对话块 NN：标题（约 X 秒）`）、镜头块内的 `[00:00-00:05]` 时间戳硬剪辑、顶部 `## 角色锚点` + `## 连续性账本` + `## 视觉参考池` 三段跨镜手册、字段化栏目（叙事目标 / 场景/氛围 / 画面叙事 / 动作与调度 / 景别/角度/运镜 / 视线与空间 / 台词/lip-sync / 声音三子层 / 因果/物理 / 参考池引用 / 衔接 / 负向约束 / 剪辑承接 等）。
- 需要把文本里的镜头顺序、台词、声音、角色、场景、道具、时长倾向、负向约束、SDV 矢量、命名光场、参考池槽位、衔接 5W 和跨镜手册稳定保留下来。
- 兼容用户直接把剧本或故事发进来，但只做保守拆分；不要在这里承担完整分镜创作。

## 负责什么

- 忠实结构化确认分镜，不重写故事，不重排镜头，不主动改台词。
- 保留原镜头顺序、标题、分集归属、场景、人物、道具、台词、旁白、声音、时长和剪辑承接。
- 将每个镜头整理为清楚的画面、叙事目标、动作起点、直接结果、声音文本、环境声和资产引用。
- 对相邻镜头做连续性整理：上一镜结束状态、当前镜起始状态、动作承接、视线承接、声音承接和下一镜入口。
- 对明显超载的镜头做机械拆分：多个空间、多个剧情目标、台词容量严重过载、一个镜头里包含“先全景再特写”或“多年后切到现在”等情况。
- 标注台词容量是否自然、偏紧或过载。已确认分镜中不要为了容量擅自删掉关键台词。
- 从当前项目已有资产中匹配人物、场景和道具名；不确定时留空或写在画面描述里，不伪造稳定资产。
- 人物引用必须严格查漏：直接出场、说话、被旁白指代、被观看、发生动作或承担当前镜头主体功能的人物，都必须进入该 shot 的人物引用，供后续 Reference Binder 强制绑定三视图/主视图锚点。

## 不负责什么

- 不创建新人物、场景或道具资产。
- 不发明新的剧情线、人物动机、反转、铺垫、尾钩或主题物象。
- 不把原分镜改得“更好看”；发现创作问题时只做最小整理，必要时在说明里提示应回到分镜咨询修改。
- 不决定分镜图、首帧、多参考、首尾帧、溶图、视频模式或最终提示词策略。
- 不输出平台专属参数、权重语法、坐标数值、CSV 或后台协议。

## 输入模式

### 已确认分镜文本

这是主路径。

- 输入守恒优先：按原顺序处理，不新增镜头、不重排镜头、不改剧情、不改台词。
- 原文里已有的镜头编号、标题、时长、景别、运镜、声音、台词、地点、角色和道具都要尽量保留。
- 如果一个镜头文本已经清楚且容量合理，就不要为了“更电影化”继续拆碎。
- 如果一个镜头需要拆分，拆出的连续 shots 必须保留原意，并说明它们来自同一个原镜头的不同动作、反应、结果或声音段落。
- 如果原文有“第 1 集 / 第 2 集”，仍保持连续 shot list，但在标题或描述中保留分集归属。

### 原始故事或剧本文本

这是兼容路径，不是推荐路径。

- 只做保守的镜头结构拆分：建立空间、主要动作、关键反应、证据特写、结果落点。
- 不主动套用短剧公式，不重写大结构，不发明新反转。
- 如果文本明显还没经过分镜创作，给出可执行的粗粒度拆分，并在说明里提示可回到分镜咨询进一步导演化。

## 跨镜手册消费（顶部 manifest）

上游 `consult.storyboard` 在分镜文本顶部会先放两段「跨镜手册」，**只写一次，不在每个镜头块里重复**。shot_split 必须先读这两段，否则角色一致性和跨镜上下文会丢：

### `## 角色锚点`

格式形如：
```
小美：粉色泡泡袖针织衫 / 黑色短裙 / 黑色短直发齐下颌 / 右眉骨小痣 / 银色细项链
姜夜：黑色保安制服 / 短寸 / 左耳骨钉 / 右手腕黑色细绳手链
```

消费规则：

- 提取每个角色的 `角色名` 和 `视觉锚点字符串`，存到该 shot 人物引用对象的 `anchor_descriptor` 字段。
- 如果该角色在项目里有稳定资产，按现行规则使用稳定资产名；**额外**把 anchor_descriptor 也带上，给下游 Seedance prompt 拼角色 token 用。
- 如果项目里**没有**对应资产（旧规则会把这种角色丢进描述里、不进资产引用），**用 anchor_descriptor 作为临时占位**进入人物引用，让下游能消费；同时在质量说明里 flag「人物 X 缺项目资产，使用临时锚点占位」。
- 校验：每个 shot 出现的所有人物，都必须能在锚点 manifest 里找到对应字符串；缺失就在质量说明里 flag。

### `## 连续性账本`

格式形如：
```
时间线：当晚 22:00 → 22:15（连续 15 分钟，无跨日）
天气基线：小雨 → 中雨，风向南偏东
活跃道具：门禁卡（姜夜口袋）, 监控屏（墙面右上）
光线基线：保安室冷白荧光 4000K + 玻璃外门头灯红 3200K（频闪 1.5Hz）
情绪轨迹：小美 试探→警觉→恐惧；姜夜 慵懒→压制→冷漠
道具状态机：门禁卡 完整(shot1) → 被姜夜手指摩挲(shot4) → 拍在桌面(shot7) → 滑落地面(shot12)
光线连续矩阵：shot1-6 4000K 冷白 + 红 3200K 频闪；shot7 加 monitor 6500K 蓝；shot8-12 红频闪频率 1.5Hz→3Hz
声音桥接计划：shot3→4 J-cut 门禁咔哒提前 0.4s；shot7→8 L-cut 监控蜂鸣延伸 1.2s；shot11→12 sound_bridge 雨声渐强
```

消费规则：

- 原样存到 shot list 的全局元数据 `continuity_ledger`（不是单 shot 字段），供下游做跨镜核对。
- `道具状态机` / `光线连续矩阵` / `声音桥接计划` 三段是新增字段；分别存到 `continuity_ledger.prop_state_machine` / `continuity_ledger.lighting_matrix` / `continuity_ledger.sound_bridge_plan`，每段保留原文。
- 拆分镜头时**不修改账本**；只有当用户回到 `consult.storyboard` 显式改剧情时账本才会变。
- 校验：每个 shot 的时间、光线、情绪走向不能与账本冲突；冲突就在质量说明里 flag，不擅自改 shot。
- **道具状态校验**：shot 描述里出现状态机里的道具时，状态描述应与该 shot 的状态机阶段一致；不一致 flag「shot N 道具 X 状态与连续性账本不符」。
- **声音桥接校验**：账本声音桥接计划提到的 shot 对（如 shot3→4），两个 shot 的 audio.sfx[] / audio.ambient 应该呼应；缺失 flag。

### `## 视觉参考池`（新增 manifest 段，必须解析）

格式形如：
```
图像槽 (≤9):
  @char_main_xiaomei      小美三视图
  @char_main_jiangye      姜夜三视图
  @scene_baoanshi         保安室主图
  @lighting_baoanshi      命名光场参考
  @prop_keycard           门禁卡特写
视频槽 (≤3):
  @motion_pushin          标准 push-in 节奏 2s
音频槽 (≤3):
  @ambient_rain_indoor    室内雨声 5s loop
  @bgm_tension            压力 BGM 8s
```

消费规则：

- 解析每个 `@slot_name` 行，提取 `slot`（含 `@` 前缀）、`type`（image / video / audio，按所在子段）、`description`，存到全局元数据 `reference_pack[]`。
- 校验槽位上限：`image` ≤ 9 / `video` ≤ 3 / `audio` ≤ 3；超出 flag「视觉参考池超 Seedance Universal Reference 上限」。
- 校验 video / audio 总长 ≤ 15s（如果声明里有时长，例如 `5s loop` / `8s`）；超出 flag。
- 解析 `slot` 命名：`@<purpose>_<name>`，purpose 必须是 `char_main / char_outfit / char_face / scene / lighting / prop / style_palette / motion / transition / ambient / bgm / voice` 之一；非枚举值 flag「参考池槽位 purpose 非标准」但仍存入。
- **每个 shot 的"参考池引用"字段**（详见下面 §下游约束与 meta 字段透传）解析出的 `@slot` 必须能在全局 `reference_pack[]` 里找到；找不到 flag「shot N 引用未声明的参考池槽位 @xxx」。
- **顶部参考池缺失但项目里有稳定资产**时，按现行守恒模式继续，flag「缺视觉参考池 manifest，跨镜一致性回退到纯 token 模式，建议回 `consult.storyboard` 顶部补 `## 视觉参考池`」。

### Manifest 格式互斥校验

`consult.storyboard` 顶部的 `## 角色锚点` 是**多行格式**（每行一个角色，字段用 ` / ` 分隔）；姊妹 skill `consult.storyboard-script-to-shot-prompts` 用的是**单行 `角色锚点：A=...;B=...` 格式**。本 skill 只处理多行格式。

如果输入顶部命中单行 `角色锚点：A=...;B=...` 或单行 `参考池：@xxx=...;@xxx=...` 格式，**立即提示**「输入是紧凑格式 manifest，应使用姊妹 skill `lumina-flow-shot-split-scene-as-shot` 处理」并停止结构化，不要硬解析。

## 工作方法

1. 先判断输入是否为已确认分镜。命中明确镜头编号、镜头块、对话块或”已确认/按这个执行”等语义时，进入守恒模式。
2. **先读顶部跨镜手册三段**（`## 角色锚点` + `## 连续性账本` + `## 视觉参考池`），按上面的消费规则建立全局上下文。如果顶部出现的是单行紧凑格式 manifest，提示用户切姊妹 skill。
3. 按原文顺序建立镜头清单，保留分集、场景和镜头标题。
4. 对每个镜头/对话块提取已有信息：叙事目标、画面动作、景别/角度/运镜、视线/空间矢量、角色、场景、道具、台词/lip-sync、声音三子层、参考池引用、衔接 5W、负向约束、时长、转场，以及创作 meta（情绪强度、暗线、因果/物理）。
5. **识别对话块和时间戳硬剪辑**，按 §”对话块与时间戳拆分”里的规则展开成连续 sub-shot；没有对话块和时间戳的镜头按现行守恒模式直接转 shot。
6. 检查镜头边界：出现空间变化、主体变化、情绪反向、新信息揭示、动作结果落点、声音文本超载时，才进一步拆成连续 shots。
7. 整理相邻承接：人物位置、朝向、视线、手中道具、动作收尾、环境声和情绪余韵不能无原因跳变。
8. **解析每镜的"衔接"块**（5W + 衔接类型 + 重复 token + Seedance 输入模式 + 首帧参考），按 §”跨 shot 衔接 (handoff) 字段”规则原样落到结构化输出。第一镜可省略衔接；第二镜起缺衔接 flag 但不补写。
9. 匹配项目资产 + 参考池槽位：按 §”跨镜手册消费”里的人物引用规则；项目稳定资产名进 `characterRef`，锚点字符串进 `anchor_descriptor`，参考池槽位进 `reference_refs[]`，三者并存。
10. 人物查漏：每个镜头最后单独核对人物引用，不能因为人物只在台词、旁白、背影、反应或局部特写中出现就漏掉；并对照锚点 manifest 校验。
11. 做容量标注：估算台词/旁白长度、可说话时间、停顿和余韵。过载时优先拆分或标注过载，不吞掉关键句；lip-sync 台词额外按三件套（引号 + 情绪前缀 + ≤12 汉字）校验。
12. **判定 Seedance 调用模式**：每个 shot 的 `seedance_call_mode` 顶层字段从该 shot 的衔接块 `Seedance 输入模式` 字段读出；缺失时默认 `text_to_video` 并 flag「shot N 缺 Seedance 输入模式声明，回退默认 text_to_video，跨镜一致性可能下降」。
13. 透传下游约束：负向约束、声音三子层、SDV 矢量、因果/物理、参考池引用、衔接、meta 字段必须按 §”下游约束与 meta 字段透传”原样落到结构化输出，不丢、不改写。
14. 最后做守恒核对：镜头数量变化是否有必要，关键原文是否保留，顺序是否一致，资产是否未伪造，跨镜手册是否未被改动，衔接 5W 是否齐全。

## 镜头边界整理

- 一个 shot 应保持同一空间、同一主体关系、同一叙事目标。
- 可以保留一个镜头内部的起手、推进、结果和短反应，但不能跨场景、跨时间或跨主要目标。
- 原文写”先到 A 地，再到 B 地””从童年切回现在””画面先全景再微距””多年过去””突然回忆闪回”，通常应拆开。
- 原文写”他说完后沉默””她听见后愣住””众人停下看她”，如果台词或信息重要，优先拆出反应或后果。
- 原文中已存在强剪辑意图时必须保留：J-Cut、L-Cut、声音桥、匹配剪辑、跳切、遮挡转场、概念转场。

## 对话块与时间戳拆分

上游 `consult.storyboard` 在写对白和需要一次成片的多镜段时，会用两种新结构。shot_split 必须识别并按规则展开：

### `### 对话块 NN：标题（约 X 秒）`（声明式正反打）

对话块代表一段让 Seedance 2.0 自动正反打覆盖的对白段，内部典型有：双方与轴线 / 关系站位 / 覆盖期望 / 情绪轨迹 / 台词序列 / 反应锚点 / 负向约束。

- **默认拆分粒度**：按”台词序列”的台词条数 + “反应锚点”里的关键反应数，拆成连续 N 个 sub-shot。每个 sub-shot 的时长按对话块总时长平均分摊（如总长 12s、4 句台词 + 1 个反应锚点 → 5 个 sub-shot 各 2.4s）；如果”覆盖期望”里给了显式时长（如 “wide 起 3s → A 中近 3s → B 反打 3s → 回 wide 3s”），优先按显式时长。
- **共享字段**：对话块顶部的 `双方与轴线 / 关系站位 / 情绪轨迹 / 负向约束` 复制到每个 sub-shot 的对应字段；不要让这些信息只挂在第一个 sub-shot 上。
- **来源标记**：每个 sub-shot 加 `dialogue_block_origin: 对话块NN` 标记，下游 presenter 可以选择”合并回一次 Seedance 多镜调用 + `[00:00-00:04]` 时间戳”或”拆 N 次单镜调用”，但 shot_split 自己不做这个决定。
- **轴线守恒**：对话块的 `A 在画面左、B 在画面右` 这类轴线声明必须落到每个 sub-shot 的视线/空间字段，不能让模型在中间镜头悄悄越轴。

### 镜头块里的 `[00:00-00:05]` 时间戳硬剪辑

当 `### 镜头 NN` 的”动作与调度”字段里出现一组 `[00:00-00:05]` 时间戳块（Seedance 一次成片多镜的语法）时：

- **默认行为**：按时间戳数量拆成连续 N 个 sub-shot，每个 sub-shot 的 `duration` = 该时间戳块的差值，描述 = 该时间戳块的原文。
- **总时长上限校验**：N 个 sub-shot 总时长必须 ≤ 15s（Seedance 一次生成的硬上限）；超过就在质量说明里 flag「时间戳总长 X.Xs 超过 15s，需要回到分镜咨询拆段」，不擅自截断。
- **来源标记**：每个 sub-shot 加 `multi_shot_origin: 镜头NN` 标记，作用同对话块标记。
- **共享字段**：原镜头块的 `叙事目标 / 场景/氛围 / 暗线/铺垫 / 视线与空间 / 负向约束 / 因果/物理` 复制到每个 sub-shot；台词、SFX、反应锚点按时间戳归属到具体 sub-shot。
- **运镜约束守恒**：每个 sub-shot 必须只有一个主运镜；时间戳里出现 `先推近再环绕` 这种描述应在质量说明里 flag「sub-shot 单镜两个主运镜，建议回分镜咨询拆开」，但仍按原文落到结构化输出。

### 模式 B 手控正反打（仍走传统单 shot 路径）

如果用户用的是 `### 镜头 NN`（不是对话块），且没有 `[00:00-00:05]` 时间戳，就按现行守恒模式逐镜转 shot，不要再拆。手控正反打和声明式对话块在同一份分镜里可以混用，shot_split 按每个块的实际语法分别处理。

## 跨 shot 衔接 (handoff) 字段

上游 `consult.storyboard` 在每个镜头里写了"衔接"块，这是 10+ shot 长叙事跨镜不漂的最高杠杆字段。shot_split 必须解析并原样落到结构化输出。

### 输入字段格式

```
衔接：
  出帧状态：（上一镜末帧人物位置/视线/手部姿态/动作完成度/声音残响）
  入帧状态：（本镜首帧上述要素）
  衔接类型：match_on_action / eyeline_match / sound_bridge / j_cut / l_cut / graphic_match / concept_cut / hard_cut
  重复 token：@char_main_xxx + @scene_xxx + ...（共享角色/空间/声音 token，至少 1 个）
  Seedance 输入模式：text_to_video / image_to_video_first_frame / first_last_frame / ref_pack / nine_panel_grid_i2v / video_extend
  首帧参考：@last_frame_of_shotNN / @char_main_xxx / @grid_main / ...（非 text_to_video 模式必填）
```

### 落到结构化输出

每个 shot 的 `handoff` 字段：

```json
{
  "handoff": {
    "previous_shot_outframe": "shot07 末帧——小美推门走进保安室，身体前倾...",
    "current_shot_inframe":  "门把仍被小美左手握住，门已开 80°...",
    "transition_type": "match_on_action",
    "repeated_tokens": ["@char_main_xiaomei", "@char_main_jiangye", "@scene_baoanshi", "@ambient_rain_indoor"],
    "seedance_input_mode": "image_to_video_first_frame",
    "first_frame_ref": "@last_frame_of_shot07"
  }
}
```

每个 shot 顶层加 `seedance_call_mode` 字段（从 `handoff.seedance_input_mode` 提升上来，方便下游 router 直接读）：

```json
{
  "seedance_call_mode": "image_to_video_first_frame"
}
```

### 校验规则

- **第 1 个 shot 可以没有 handoff**；从第 2 个 shot 起 handoff 缺失 flag「shot N 缺衔接块，跨镜一致性回退到 hard_cut + text_to_video」，但仍按默认值落到结构化输出（`transition_type=hard_cut, seedance_input_mode=text_to_video`），不擅自补写 5W 内容。
- **`transition_type` 必须是 8 个枚举值之一**：`hard_cut / match_on_action / eyeline_match / sound_bridge / j_cut / l_cut / graphic_match / concept_cut`；非枚举值 flag「衔接类型 X 非标准」并回退到 `hard_cut`。
- **`seedance_input_mode` 必须是 6 个枚举值之一**：`text_to_video / image_to_video_first_frame / first_last_frame / ref_pack / nine_panel_grid_i2v / video_extend`；非枚举值 flag 并回退 `text_to_video`。
- **`seedance_input_mode != text_to_video` 时 `first_frame_ref` 必填**；缺失 flag「shot N 模式 X 缺首帧参考槽位 ID」。
- **`repeated_tokens` 至少含 1 个 token**（跨场景硬切允许丢空间但角色 token 必留）。空数组 flag「shot N 与 shot N-1 共享 0 token，可能跳剧」。
- **`transition_type=concept_cut` 全片计数**：超过 2 处 flag「全片 concept_cut N 处，叙事可能散架」。
- **`first_frame_ref` 引用的槽位** 必须能在全局 `reference_pack[]` 或 `@last_frame_of_shotNN`（指向已存在的 shot 编号）里解析；解析不到 flag「shot N 首帧参考 X 解析失败」。

### Seedance 调用路由建议（仅写说明，不强制行为）

下游 presenter 按 `seedance_call_mode` 选择 API 调用形式：

| 模式 | API 调用 | 必需输入 | 用途 |
|---|---|---|---|
| `text_to_video` | T2V | prompt only | 第 1 镜 / 风格化空镜 / 接受较弱一致性 |
| `image_to_video_first_frame` | I2V (first frame) | 1 张图 + prompt | 同场景同主体 + match_on_action（首帧用上一镜末帧） |
| `first_last_frame` | I2V (first + last frame) | 2 张图 + prompt | **明确 A→B 状态过渡**（化身 / reveal / 出态→入态过渡） |
| `ref_pack` | T2V/I2V + reference pack | prompt + 1-9 image + 1-3 video + 1-3 audio | 多角色 / 多场景 / 跨场景默认 hard cut；身份锚靠 `@char_main_*` turnaround sheet |
| `nine_panel_grid_i2v` | I2V (grid as input) | 1 张 9 宫格 storyboard 图 + motion prompt | **段内分镜压缩**：把 9 个紧凑节拍压成 1 段 15s 连续电影镜头（替代 6-8 次小调用）。**不是**跨段身份锚 |
| `video_extend` | Video extend | 上一段 video + prompt | **同剧情同场景延长 >15s**（模型分析整段轨迹，比 first_last_frame 更稳） |

**关键区分**：

- `nine_panel_grid_i2v` 的 `first_frame_ref` 应该是 `@grid_<段名>` 这类**段级 grid** 槽位，不是 `@char_main_*` turnaround sheet。前者是单段分镜压缩，后者是跨段身份锚。
- `image_to_video_first_frame` 与 `video_extend` 在同场景续接时优先后者（轨迹分析更全面）；只有"明确 keyframe → keyframe"过渡才用 `first_last_frame`。

shot_split 不替下游做这个调用，只把 `seedance_call_mode` 和 `handoff` 透传。

## 空间与连续性整理

- 已确认分镜怎么写，就按它的空间逻辑整理；不要重新设计机位。
- 同一场景内默认人物真实位置不变，除非文本写了移动。
- 有位移时保留起点、方向、距离感和落点，例如”从门口向病床靠近””退到桌边””看向画面外的走廊”。
- 双人对话和对峙保留原分镜的左右关系、视线方向和高度差；原文没有写时，只做保守推断，不制造复杂越轴。
- 若相邻镜头存在明显跳位、视线断裂或动作不接，只补最小承接描述，不改剧情。
- 群像镜头要保留核心视觉焦点；其他人物作为边缘、背景或反应群体处理。
- 当原文 “视线与空间” 字段出现 SDV 矢量句式 `<起点锚点> → <X+/X-/Y+/Y-/Z+/Z-> → <终点锚点>` 时，**原样存到 shot 的 `motion_vector` 字段**（string，不要改成自由描述）。这个矢量给下游 HDVP 锁深度墙用，丢了高风险位移的方向就没了。
- 当原文 “场景/氛围” 字段里写了 **明暗交界线位置** 和 **冷暖色温分布** 时，原样存到 shot 的 `lighting_geometry` 字段（string）。改写或删掉会让下游灯光锁不住。

## 台词、旁白与声音

### 台词解析（lip-sync 三件套保护）

上游 `consult.storyboard` 用以下格式写对白：

```
角色（情绪/语气）”短台词”
```

引号可以是 `”...”` / `「...」` / `『...』`，等价处理。解析规则：

- `角色名` → `dialogue.speakerRef`
- `情绪/语气` → `dialogue.tone`（新字段，原样保留，下游 Seedance 用作 lip-sync 增强信号）
- 引号内文本 → `dialogue.text`，**保留引号原样**（Seedance 把引号识别为 lip-sync 触发器，去掉就没 lip-sync 了）
- `dialogue.kind` 默认 `dialogue`；遇到 `VO / 旁白 / 内心 OS / 画外、画外音 / OS` 等关键词才切到 `voiceover / inner_monologue / offscreen`，且**这些非 dialogue 类型引号可省**。
- 输入中的台词、旁白、反问、誓言、遗言、告白、判决、辱骂、承诺、主题句和心理转折应尽量保留原貌。
- 不要把 `voiceover`、`offscreen`、`inner_monologue` 都混成普通对白；如果原文写”心里想””内心 OS””画外传来””旁白说”，必须保留这种声源差异。
- `speakerRef` 只在能稳定对应人物时填写；旁白没有明确人物声源时可以留空或使用叙述者语义，不要伪造成某个角色开口。

### `source` 与改写约束

- `source` 反映来源：原文逐字保留用 `verbatim`，为可说话容量轻微改写用 `adapted`，为连接镜头新增的短句用 `new_bridge`。
- **lip-sync 台词（kind=dialogue 且带引号）不允许 `adapted` 改写**，只能 `verbatim` 或 `new_bridge`，且 `new_bridge` 必须满足 lip-sync 三件套（引号 + 情绪前缀 + ≤12 个汉字 / 8 个英文单词）。需要改写时改成在质量说明里 flag「台词 X 容量超载，建议回分镜咨询缩短」，而不是默改。
- **任何 `source=new_bridge` 的输出必须在质量说明里强制 flag**「shot N 新增桥句台词「...」（new_bridge），未经用户确认；如需保留请在 consult.storyboard 显式确认」。new_bridge 是低频路径，不能默挂；上游 `consult.storyboard` 已经写过的台词永远走 `verbatim`。
- `voiceover / inner_monologue / offscreen` 类型可以 `adapted`，但不能吞掉核心修辞。

### 容量

- 长段原文可拆成连续旁白蒙太奇或多个相邻 shots，但不能概括吞掉核心修辞。
- 普通中文约 4-5 字/秒；重情绪、病弱、哭腔、迟疑约 3-4 字/秒；争吵和信息流快读最多约 5-6 字/秒；极快只用于特殊风格。
- 估算说话时间时要扣除动作、停顿、反应、环境音和情绪余韵。
- 已确认分镜台词过载时，不擅自删改；优先拆分、延长倾向或标注过载。
- lip-sync 台词额外校验：单条引号内文本 ≤ 12 个汉字 / 8 个英文单词；超过就在质量说明里 flag。

### 声音三子层映射

上游 `consult.storyboard` 写声音时分三个子层：

```
声音：
  SFX：（动作触发音、物件相关音、时间戳锚点）
  环境声：（场景持续 ambient）
  沉默/余韵：（沉默段长度与位置）
```

shot_split 必须把这三个子层分别映射到结构化字段：

- `SFX：` → `audio.sfx[]`（数组，可挂时间戳锚点）
- `环境声：` → `audio.ambient`（字符串，描述场景持续音）
- `沉默/余韵：` → `audio.silence[]`（数组，每项含位置 + 时长）

**禁止把三个子层合并成单字段**，下游 Seedance 原生音频路径要按通道分别消费。如果原文只笼统写了”声音”没分子层，按内容判断归类，而不是塞进单字段。

- 没有台词的镜头也要保留环境声、动作音或沉默依据，不默认添加强音乐。

## 时长与颗粒度

- 推荐单镜头时长只是拆分粒度参考，不是固定长度。不要把全片整理成机械均匀时长。
- 已确认分镜给了时长时，优先保留；没给时，按画面动作、声音文本和情绪余韵估算。
- 建立镜头、证据特写、极短反应、强快切、声音桥和转场空镜可以短于推荐时长，但必须有明确功能。
- 如果一个剧情节点在推荐时长内能清楚表达，就不要为了数量拆分。
- 如果台词容量、空间变化、情绪翻转或结果落点超过一个镜头可承载范围，应拆成连续 shots。

## 资产引用

- 只引用当前项目已有的人物、场景和道具稳定名称。
- 已绑定角色或场景出现时，优先使用项目里的稳定名称，不用自由描述替代。
- 人物引用比场景/道具更严格：任何参与当前镜头的人物都必须被引用，否则后续视频任务无法强制拿到该人物三视图/主视图。
- 项目中不存在但剧情需要的临时对象，只保留在描述中，不伪装成资产。
- 反常规状态要保留在描述里：病危、战损、荒废、枯死、烧毁、贫穷、重伤、污染、被遗弃等不能被默认美化。
- **顶部 `## 角色锚点` manifest 与项目资产并存**：项目里有该角色稳定资产时，资产名进 `characterRef`，锚点字符串进 `anchor_descriptor`，两者一起带给下游；项目里没有时，**用 anchor_descriptor 作为临时占位进入 characterRef**，让下游 Seedance 仍能拿到一致性 token，同时在质量说明里 flag「人物 X 缺项目资产，使用临时锚点占位」。
- 不要因为"项目里没这个角色资产"就把角色从镜头里丢掉——锚点占位的优先级高于"严格资产匹配"。

## 下游约束与 meta 字段透传

上游 `consult.storyboard` 写到镜头里的 `负向约束` / `因果/物理` / `情绪强度` / `暗线/铺垫` 这几个字段，旧 shot_split 没有对应处理。新规则：

| 来源字段 | 落到结构化输出的字段 | 必须性 |
|---|---|---|
| `负向约束` | `negative_constraints`（string 数组，按 `/` 或逗号拆条，全角 `｜` 与半角 `\|` 都识别） | **必须保留**，下游 Seedance prompt 会拼到 negative slot |
| `因果/物理` | `meta.causal_physics`（string，原样） | 高物理交互镜头**必须保留**，下游 Seedance 用作 WHY 信号 |
| `情绪强度` | `meta.emotion_intensity`（string，原样） | 保留即可，无下游强消费 |
| `暗线/铺垫` | `meta.subtext`（string，原样） | 保留即可，给后续修订阶段参考 |
| `视线与空间` 里的 SDV 矢量 | `motion_vector`（string，原样） | 高风险位移场**必须保留** |
| `场景/氛围` 里的明暗交界线 + 冷暖色温分布 | `lighting_geometry`（string，原样） | 命名光场**必须保留** |
| `参考池引用` | `reference_refs[]`（数组，每项 `{slot, type, purpose}`） | 项目里有稳定资产的角色/场景**必须保留**，下游 Seedance 路由到 ref 槽 |
| `衔接` 整块 | `handoff`（object，详见 §跨 shot 衔接字段） | 第二镜起**必须保留**，缺失 flag 不补 |
| `衔接` 里 `Seedance 输入模式` | `seedance_call_mode`（string，顶层字段，方便下游 router 直接读） | **必须保留** |

**禁止改写、合并、压缩**这些字段。如果原文这些字段为空，结构化输出对应字段也留空，不要硬编出值。

### `fast` 关键词 flag 作用域（限定）

shot_split 检查 `fast` 关键词时**只在以下两个字段范围里 flag**：
- `景别/角度/运镜` 字段全文
- `动作与调度` 字段里描述运镜的部分（`相机 fast push-in`、`fast tracking shot` 等）

**不在以下范围 flag**（这些是合法的剧情/生理描述，不影响 Seedance 运镜）：
- 角色描述（`他比上次跑得更快`、`心跳加快`）
- 旁白 / 台词内容（`"快进来"` 这种 lip-sync 台词）
- 因果/物理（`fast-flowing water` 描述水流物理属性）

flag 时给具体出现位置，不要全文级 flag。

### 主体前置 (Subject-First) flag

shot_split 不改写原分镜句序，但要在质量说明里 flag 触发 Seedance 2.0 "人物闪现 / 拍间融合" 高发反模式的写法。检查 `画面叙事` / `动作与调度` 字段的首句：

- **环境前置**：首句以"明亮的 X / 阳光 / 冷光 / 暖光 / 整间 / 一片"等环境短语开头，接"镜头推近到 X / 镜头摇过到 X"，flag「shot N 首句环境前置，Seedance 2.0 易出现人物闪现，建议回 consult.storyboard 改为主体前置（先写主体位置 + 动作，再写环境）」。
- **被动揭示语**：出现"揭示出 X / 最终落在 X 身上 / 原来 X 站在 / 突然出现 X / 忽然闪现 X / 画面里多了 X"，flag「shot N 出现被动揭示语，主体可能延迟揭示」。
- **carry-over 角色缺位**：上一镜（已绑定 character_refs）的同一角色在本镜首句没有显式出现（无位置/状态描述），flag「shot N carry-over 角色 X 未在首句重提位置/状态，跨镜可能融合」。
- **新进角色无入画方向**：本镜首次出现的角色（不在上一镜 character_refs 里）描述里没有方向/起始位置（"从画面左侧入画" / "在画面右下角"），flag「shot N 新进角色 X 缺入画方向/起始位置」。

shot_split **不擅自改写**这些字段，按原文落到 description；只在质量说明里 flag，让用户决定回 consult.storyboard 修。

### 焦距 jargon flag

`画面叙事` / `动作与调度` / `景别/角度/运镜` 字段里出现 `85mm / f/1.4 / 35mm 焦段 / shallow DOF / focal plane / 浅景深 / 焦距 / 光圈 / 景深` 等技术 jargon 时 flag「shot N 出现焦距 jargon，Seedance 不响应甚至 degrade，建议改 `广角空间感 / 自然视角 / 人像压缩感 / 微距质感 + 画面占比`」。原文落到 description 不改写。

## 质量标准

- 不漏用户确认的镜头、对话块、台词、旁白、声音、场景、角色和关键道具。
- 镜头顺序与确认文本一致；任何拆分都只是把一个过载镜头/对话块/时间戳块拆成连续子镜头，不改变含义。
- 每个 shot 都有清楚的画面动作、叙事目标、直接结果和下一镜承接点。
- 相邻 shots 没有人物突然换位置、视线突然反向、台词突然开始、情绪突然跳变或空间突然断裂。
- 台词/旁白容量与时长倾向匹配；过载时有明确标注或合理拆分。
- lip-sync 台词（kind=dialogue + 引号）：引号保留原样、tone 字段已填、单条 ≤ 12 个汉字 / 8 英文单词、source 不是 `adapted`；`source=new_bridge` 已强 flag。
- 资产引用真实存在，不伪造项目内不存在的稳定名称；缺项目资产的角色用顶部 anchor_descriptor 占位并已 flag。
- 跨镜手册三段：顶部 `角色锚点` 的所有角色都被某个 shot 引用过；`连续性账本` 已存到 `continuity_ledger` 全局元数据（含 prop_state_machine / lighting_matrix / sound_bridge_plan 三新字段），且没有 shot 与账本冲突；`视觉参考池` 已存到 `reference_pack[]` 全局元数据，槽位上限 9 image / 3 video / 3 audio 已校验。
- 输入是单行紧凑 manifest 格式（`角色锚点：A=...;B=...` / `参考池：@xxx=...`），已**提示切姊妹 skill** 而不是硬解析。
- 对话块和时间戳硬剪辑：对话块按台词序列 + 反应锚点拆 sub-shot，时间戳按时间戳数拆 sub-shot；sub-shot 都带 `dialogue_block_origin` / `multi_shot_origin` 标记；时间戳总长 ≤ 15s。
- **跨 shot 衔接 (handoff)**：第二镜起每个 shot 都有 `handoff` 块；`transition_type` / `seedance_input_mode` 都是合法枚举值；`repeated_tokens` 至少 1 个；非 `text_to_video` 模式的 `first_frame_ref` 已填且能解析；`concept_cut` 全片 ≤ 2 处；`seedance_call_mode` 顶层字段已从 handoff 提升。
- 透传字段：`negative_constraints` / `motion_vector` / `lighting_geometry` / `meta.causal_physics` / `meta.emotion_intensity` / `meta.subtext` / `reference_refs[]` / `handoff` / `seedance_call_mode` 全部按原文落到结构化输出，没改写、没合并、没压缩。
- `fast` flag 作用域已限定在 `景别/角度/运镜` 字段和 `动作与调度` 字段的运镜描述部分；剧情/生理/台词里的 fast 用法没有误 flag。
- 声音：SFX / ambient / silence 三子层分别落到 `audio.sfx[]` / `audio.ambient` / `audio.silence[]`，没合并成单字段。
- 输出能被后续分镜图、视频路径和提示词包阶段继续消费，不需要再猜原分镜意思。

## 失败回退

- 输入不像确认分镜时，给粗粒度结构化拆分，并提示应先到分镜咨询完善导演表达。
- 原文空间关系不清时，保持简单：建立镜头 -> 中景动作 -> 特写证据或反应 -> 结果落点。
- 原文台词太长而时长不足时，拆成连续旁白或对话 shots，不吞掉关键句。
- 资产匹配不确定时留空，不要为了完整性编造资产名。
- 顶部跨镜手册缺失时（角色锚点 / 连续性账本 / 视觉参考池任一缺失），按现行守恒模式继续处理，但在质量说明里 flag「缺跨镜手册 X，跨镜一致性可能下降，建议回 consult.storyboard 补」。三段都缺时同时 flag，并提示「这是 Tier 1 ≤3 shot 短片段以外的场景必装项」。
- 输入顶部出现单行 `角色锚点：A=...;B=...` 或单行 `参考池：@xxx=...;@xxx=...` 紧凑格式 manifest 时，**立即停止**结构化，提示「输入是紧凑格式 manifest，应使用姊妹 skill `lumina-flow-shot-split-scene-as-shot` 处理」。
- 时间戳总长 > 15s、对话块"覆盖期望"语义不清、单镜两个主运镜、lip-sync 台词超过 12 字等情况，**不擅自修复**，按原文结构化输出 + 在质量说明里 flag，让用户回 consult.storyboard 决定怎么改。
- 衔接块缺失（第二镜起）/ 衔接类型非枚举值 / Seedance 输入模式非枚举值 / 非 text_to_video 模式缺首帧参考 / 重复 token 为空 / first_frame_ref 解析失败时，**不擅自修复**，按默认值（`hard_cut + text_to_video`）落结构化输出 + 在质量说明里 flag，让用户回 consult.storyboard 决定。
- 长叙事 ≥10 shot 但顶部参考池没有 `@char_main_*` + `@scene_*` 两个最低槽位时，按 Tier 1 现行守恒模式继续，flag「10+ shot 长叙事缺 Tier 3 参考池基线，跨镜一致性预期低于 60%，强烈建议回 consult.storyboard 补 9-panel grid 计划与参考池槽位」。
