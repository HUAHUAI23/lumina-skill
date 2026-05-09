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
  - 场景标题 `场景X 名称 X秒`（默认 12-15s/场景）
  - 场景下若干镜头行 `**镜头X X.Xs 时间: ... [景别/运镜]** 描述`
  - 镜头编号在整篇内连续递增，**不在每个场景里重置**
  - 镜头行可能内嵌 `角色（情绪、动作）"短台词"`，行尾可选 `｜避免:无字幕条/无后景人/...`
- 用户目标是直接走 Seedance 2.0 一次成片路径。

## 不适用 / 走姊妹 skill

如果输入命中以下任一特征，**应该走 `lumina-flow-shot-split` 而不是本 skill**：

- 出现字段化栏目（`叙事目标：` / `画面叙事：` / `动作与调度：` / `视线与空间：` / `声音：` 等多行字段）。
- 出现 `### 镜头 NN：标题（约 X 秒）` 三级 heading 形式的镜头小节。
- 出现 `### 对话块 NN：标题（约 X 秒）`。
- 出现 `## 角色锚点` + 多行展开的 manifest 块。

发现这些特征时，提示用户「输入是导演级长稿格式，应使用 `lumina-flow-shot-split` 处理」，不要在本 skill 里硬拆。

## 核心映射规则

### 顶层结构

| 来源 | 目标 | 备注 |
|---|---|---|
| 顶部 `角色锚点：A=...;B=...` 单行块 | `continuity_ledger.character_anchors` map | 每个角色的视觉锚点字符串原样存 |
| 每个 `场景X 名称 X秒` | 1 个 shot | shot.title = 场景名称、shot.scene_index = X、shot.declared_duration = X秒 |
| 场景内所有镜头时长之和 | shot.duration | 与 declared_duration 不一致时以镜头之和为准并 flag |
| 镜头行的 `时间: 白天/夜晚` | shot.time_of_day | 同场景所有镜头必须一致；不一致 flag |
| 场景内所有镜头行尾的 `｜避免:...` | shot.negative_constraints[] | 按 `/` 拆条、聚合到 shot 级、去重 |

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

合成为一个 shot 的 `timeline_blocks`：

```json
{
  "duration": 12.0,
  "scene_name": "保安室-玻璃后的微笑",
  "time_of_day": "夜晚",
  "timeline_blocks": [
    { "start": "00:00.0", "end": "00:02.0", "framing": "中景", "movement": "固定", "description": "保安室内冷白色灯管微微闪烁,姜夜坐在值班桌后,玻璃窗外隐约映出小美站在门口的身影。" },
    { "start": "00:02.0", "end": "00:03.5", "framing": "特写",   "movement": "固定", "description": "玻璃反光遮住姜夜半边脸,他嘴角慢慢上扬,露出意味不明的微笑。", "dialogue": { "speakerRef": "姜夜", "tone": "玩味、轻声", "text": "\"奖励？\"", "kind": "dialogue", "source": "verbatim" } },
    { "start": "00:03.5", "end": "00:05.0", "framing": "反应镜头", "movement": "固定", "description": "玻璃外的小美听见这两个字,笑容更甜,微微歪头,眼神带着得逞的意味,像以为姜夜已经上钩。" },
    { "start": "00:05.0", "end": "00:07.0", "framing": "中近景", "movement": "推镜", "description": "镜头缓缓推向姜夜,他靠在椅背上嘿嘿一笑,眼神从慵懒变兴奋,伸手按向桌边开门按钮。", "dialogue": { "speakerRef": "姜夜", "tone": "兴奋、急切", "text": "\"那还等什么？快进来！\"", "kind": "dialogue", "source": "verbatim" } },
    { "start": "00:07.0", "end": "00:08.0", "framing": "插入镜头", "movement": "固定", "description": "姜夜手指按下门禁开关,按钮亮起绿光,保安室门锁发出\"咔哒\"一声轻响。", "sfx": ["门禁绿光提示音", "门锁咔哒"] },
    { "start": "00:08.0", "end": "00:10.0", "framing": "中景",   "movement": "跟拍", "description": "小美推门走进来,脸上还挂着甜腻笑容,身体轻轻前倾,像迫不及待要靠近姜夜。" },
    { "start": "00:10.0", "end": "00:12.0", "framing": "快切",   "movement": "手持晃动", "description": "姜夜突然起身,动作快得带出残影,一只手猛地伸出,瞬间扣住小美的咽喉,将她压制在墙边。" }
  ]
}
```

时间戳规则：

- 第一个块从 `00:00.0` 开始。
- 后续块的 `start` = 上一块的 `end`，无空隙、无重叠。
- 时间戳用 `MM:SS.S`（毫秒级单位 0.5s 精度即可），与 Seedance 的 `[00:00-00:05]` 语法等价。
- shot.duration = 最后一个块的 `end`。
- 单 shot duration **必须 ≤ 15s**（Seedance 2.0 一次生成硬上限）；超出 flag 不擅自截断。

### 镜头行解析

镜头行格式：

```
**镜头X X.Xs 时间: 白天。** [景别/运镜] 描述...角色（情绪/语气）"短台词"...｜避免:负向约束1/负向约束2
```

逐字段解析：

- `**镜头X` → block.shot_index_in_full（整篇连续编号，跨场景不重置；本 skill 仅记录，不做强消费）
- ` X.Xs ` → block.duration（用于累加生成 start/end）
- `时间: XX。` → 校验与 shot.time_of_day 一致
- `[景别/运镜]` → block.framing + block.movement（按 `/` 拆）
  - 单镜**只允许一个主运镜**（push-in / pull-out / pan / tilt / tracking / orbit / aerial / handheld / locked-off / 固定），出现 `先推近再环绕` flag
  - 出现 `fast` 关键词立即 flag（Seedance 官方 degrade 词）
- 引号外的描述句 → block.description（保留原貌）
- 引号 + 情绪前缀的台词 → block.dialogue（按 lip-sync 三件套保护规则解析，下面单列）
- 行尾 `｜避免:...` → 聚合到 shot.negative_constraints[]，按 `/` 拆条

### 台词解析（lip-sync 三件套保护）

格式：`角色（情绪/语气）"短台词"`（引号可以是 `"..."` / `「...」` / `『...』`，等价处理）

- `角色名` → `dialogue.speakerRef`
- `情绪/语气` → `dialogue.tone`（原样保留）
- 引号内文本 → `dialogue.text`，**保留引号原样**（Seedance 把引号识别为 lip-sync 触发器）
- `dialogue.kind` 默认 `dialogue`
- 出现 `VO / 旁白 / 内心 OS / 画外、画外音 / OS` 关键词时切到 `voiceover / inner_monologue / offscreen`，且这些类型引号可省。
- `dialogue.source`：原文逐字 `verbatim`；为时长容量轻微改写 `adapted`（**lip-sync dialogue 不允许 adapted**，超载就 flag）；shot_split 自己加的桥句 `new_bridge`（必须仍满足三件套）。
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

## 工作方法

1. 先识别输入格式。命中"场景X 名称 X秒"标题 + "**镜头X X.Xs 时间: ... [景别/运镜]**" 行 → 进入本 skill 守恒模式；否则提示走姊妹 skill。
2. 解析顶部可选的 `角色锚点：` 单行块，存入 `continuity_ledger.character_anchors`。
3. 按 `场景X` 切分输入，每个场景独立处理为一个 shot。
4. 解析场景标题：提取场景编号、场景名称、声明时长。
5. 遍历场景内的镜头行：
   - 累加镜头时长生成时间戳 `start / end`
   - 拆 `[景别/运镜]`
   - 提取描述、台词、SFX、负向约束
   - 校验单镜单主运镜、禁 `fast`、禁焦距 jargon
6. 校验场景级约束：
   - 总时长 = 镜头时长之和；与声明时长不一致 flag
   - 总时长 ≤ 15s（Seedance 上限）；超出 flag
   - 总时长 ∈ [12, 15]（用户另行指定时除外）；不在范围 flag
   - time_of_day 同场景所有镜头一致；不一致 flag
7. 把场景内出场角色（描述里直接出现的、台词的 speakerRef、被反应镜头观看的）落到 shot.character_refs；按角色锚点占位规则处理资产匹配。
8. 聚合行尾负向约束到 shot.negative_constraints[]，去重。
9. 生成最终 shot list，按场景顺序。
10. 输出质量说明：列出所有 flag、占位说明、容量警告、上下限警告。

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

- 输入格式判定正确：紧凑格式走本 skill，导演级长稿提示走姊妹 skill，没有混判。
- 每个场景都被转成一个 shot；场景顺序、场景名称、场景编号一致。
- shot.duration = 内部镜头时长之和；与声明时长不一致已 flag。
- 每个 shot.duration ≤ 15s（Seedance 一次生成硬上限）；超出已 flag。
- 每个 shot 的 timeline_blocks 时间戳无空隙、无重叠，从 00:00.0 起，最后一块 end = shot.duration。
- 每个 timeline_block 单镜只有一个主运镜；出现两个或 `fast` 关键词已 flag。
- 没有出现焦距/光圈/景深/85mm/f/1.4 这种技术 jargon（Seedance 不响应）；出现已 flag。
- lip-sync 台词（kind=dialogue + 引号）：引号保留原样、tone 字段已填、单条 ≤ 12 个汉字 / 8 英文单词、source 不是 `adapted`。
- 角色锚点 manifest 已解析；所有 shot 出场角色都在锚点里能找到；缺项目资产的角色已用 anchor_descriptor 占位 + flag。
- 行尾负向约束已聚合到 shot.negative_constraints[]，去重，没有任何条目被丢。
- 描述里的 SFX、命名光场、微表情链、眼神锚已落到对应字段，没合并、没压缩。
- shot list 顺序与原分镜场景顺序一致。
- 输出能直接喂给下游 Seedance 一次成片调用，不需要再猜原意。

## 失败回退

- 输入不像紧凑格式时（看到字段化栏目 / `### 镜头 NN：` / `### 对话块 NN：` / `## 角色锚点` 多行块），提示「输入是导演级长稿格式，应使用 `lumina-flow-shot-split` 处理」，不在本 skill 里硬转。
- 场景声明时长 > 15s 时，flag「场景 X 时长 X.Xs 超过 Seedance 一次生成上限 15s，建议回 `consult.storyboard-script-to-shot-prompts` 拆成两个场景」，仍按原文落到结构化输出（shot.duration 用镜头之和），不擅自截断。
- 场景声明时长 < 12s 且用户没有显式指定短时长时，flag「场景 X 时长偏短，可能是上游格式漂移」，仍按原文输出。
- 单镜时长 > 5s 时，flag「场景 X 镜头 N 时长 X.Xs 偏长，时间戳合成意义不大，建议回 consult 拆开或考虑改用姊妹 skill」，仍按原文输出。
- lip-sync 台词超 12 字时，flag「场景 X 镜头 N 台词超载，建议回 consult 缩短」，**不擅自 adapt**。
- 顶部缺角色锚点 manifest 时，按现行守恒模式继续处理，flag「缺角色锚点 manifest，跨镜一致性可能下降，建议回 `consult.storyboard-script-to-shot-prompts` 顶部补」。
- 镜头编号跳号/重号时，flag「场景 X 镜头编号不连续：1, 2, 4」，按原文顺序处理，不擅自补/改编号。
