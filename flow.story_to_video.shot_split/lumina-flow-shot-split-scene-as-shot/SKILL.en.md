---
name: lumina-flow-shot-split-scene-as-shot
description: A derived version of flow.story_to_video.shot_split; specifically converts compact storyboard text in the form of "scene + several shots" into a structured shot list with "1 scene = 1 shot", where multiple shots within a scene are compressed into timestamped hard-cut blocks like `[00:00-00:02]` inside the shot, directly matching Seedance 2.0's ability to generate multiple shots in a single pass.
---

# flow.story_to_video.shot_split / scene-as-shot
## Stage Positioning

This is the structured compilation stage of story_to_video, used in parallel with and mutually exclusively to its sister skill `lumina-flow-shot-split`.

- **Sister skill `lumina-flow-shot-split`**: Handles director-level long-form drafts output by `consult.storyboard` (each shot is a fielded subsection in the format `### Shot NN: Title (about X seconds)`) → **1 shot = 1 shot**.
- **This skill `lumina-flow-shot-split-scene-as-shot`**: Handles the compact "scene + shot" format output by `consult.storyboard-script-to-shot-prompts` → **1 scene = 1 shot**, with multiple shots within a scene compressed into timestamped hard-cut blocks like `[00:00-00:02]` inside the shot.

The two produce different output structures, and their downstream consumption paths also differ: the output of this skill connects directly to Seedance 2.0 single-pass multi-shot generation calls (one Seedance call per shot, with all internal shots in the shot fed to the model at once as timestamp blocks).
## Applicable Scenarios

- The input uses a compact "scene + multiple shots" format:
  - Optional top-level `# Storyboard Text` main title
  - Optional single-line `Character Anchors: A=...;B=...` block at the top
  - Optional single-line `Reference Pool: @slot=description;@slot=description...` block at the top (Universal Reference 12-slot declaration)
  - Scene title `Scene X Name Xs` or `Scene X Name Xs Transition:type(token1+token2)` (inter-scene transition; strongly recommended starting from the second scene)
  - Several shot lines under each scene: `**Shot X X.Xs Time: ... [Shot Size/Camera Movement]** Description`
  - Shot descriptions may inline reference pool slot references such as `(@char_main_xxx)` / `(@scene_xxx)`
  - Shot numbering increases continuously across the entire document and **does not reset within each scene**
  - A shot line may embed `Character (emotion, action) "short dialogue"`, and may optionally end with `｜Avoid:no subtitles/no background people/...` (both full-width `｜` and half-width `|` are recognized)
  - A shot line may be followed by a line like `Transition:type Mode:xxx First Frame:@xxx Repeat:xxx` (strongly recommended for every shot in Tier 2/3 long-form narratives)
- The user's goal is to go directly through the Seedance 2.0 one-pass final video generation path.
## Not Applicable / Route to Sister Skill

If the input matches any of the following characteristics, **it should use `lumina-flow-shot-split` instead of this skill**:

- Fielded sections appear (`Narrative Goal:` / `Visual Narrative:` / `Action & Blocking:` / `Eyeline & Space:` / `Sound:` / `Reference Pool Citations:` / `Transition:` and other multi-line fields).
- Shot subsections appear in the form of level-3 headings like `### Shot NN: Title (about X seconds)`.
- `### Dialogue Block NN: Title (about X seconds)` appears.
- `## Character Anchors` appears along with a multi-line expanded manifest block.
- `## Visual Reference Pool` appears along with a multi-line expanded slot list (image slots / video slots / audio slot subsegments).
- `## Continuity Ledger` appears containing multi-field sections such as `Prop State Machine` / `Lighting Continuity Matrix` / `Sound Bridging Plan`.

When these characteristics are detected, inform the user: "The input is in director-level long-form format and should be processed with `lumina-flow-shot-split`." Do not force a split within this skill.
## Core Mapping Rules
### Top-Level Structure

| Source | Target | Notes |
|---|---|---|
| Top single-line block `Character anchors: A=...;B=...` | `continuity_ledger.character_anchors` map | Store each character's visual anchor string as-is |
| Top single-line block `Reference pool: @slot=description;@slot=description...` | `reference_pack[]` global metadata | Each item is `{slot, type, purpose, description}`; `type` is inferred from `purpose` (`char_*`/`scene`/`lighting`/`prop`/`style_palette` → image, `motion`/`transition` → video, `ambient`/`bgm`/`voice` → audio) |
| Each `Scene X Name Xs` | 1 shot | `shot.title` = scene name, `shot.scene_index` = X, `shot.declared_duration` = Xs |
| Appended after the scene title: `Handoff:type(token1+token2)` | `shot.handoff` (only the two fields `transition_type` + `repeated_tokens`) | The first scene does not have this; if missing starting from the second scene, flag "Scene N has no inter-scene handoff; cross-scene consistency may decrease" |
| Sum of all shot durations within the scene | `shot.duration` | If inconsistent with `declared_duration`, use the sum of shot durations and flag it |
| `Time: day/night` in the shot line | `shot.time_of_day` | Must be consistent across all shots in the same scene; if inconsistent, flag it |
| Inline references `(@char_main_xxx)` / `(@scene_xxx)` in the shot description | `timeline_blocks[i].reference_refs[]` | Extract the `@xxx` slot IDs; the shot-level `reference_refs[]` is the deduplicated union of all block references |
| Next line after the shot line: `Handoff:type Pattern:xxx First frame:@xxx Repeat:xxx` | `timeline_blocks[i].handoff` (intra-block handoff) | A hard cut written between shots is mapped to block-level fields |
| `｜Avoid:...` at the end of all shot lines within the scene | `shot.negative_constraints[]` | Split items by `/`, aggregate to the shot level, and deduplicate; **both full-width `｜` and half-width `\|` are recognized** |
### Intra-shot timestamp composition (core)

Convert each camera shot within a scene into a shot-level `timeline_blocks[]` array, generating timestamps by cumulative duration:

```
Shot 1 2.0s [Medium shot/Static] Description...
Shot 2 1.5s [Close-up/Static] Description...
Shot 3 1.5s [Reaction shot/Static] Description...
Shot 4 2.0s [Medium close-up/Push-in] Description...
Shot 5 1.0s [Insert shot/Static] Description...
Shot 6 2.0s [Medium shot/Tracking] Description...
Shot 7 2.0s [Quick cut/Handheld shake] Description...
```

Compose them into a single shot’s `timeline_blocks` (including `reference_refs` and `handoff` examples):

```json
{
  "duration": 12.0,
  "scene_name": "Security Room - The Smile Behind the Glass",
  "time_of_day": "Night",
  "scene_handoff": null,
  "reference_refs": ["@char_main_xiaomei", "@char_main_jiangye", "@scene_baoanshi", "@lighting_baoanshi", "@prop_keycard"],
  "timeline_blocks": [
    {
      "start": "00:00.0", "end": "00:02.0", "framing": "Medium shot", "movement": "Static",
      "description": "Inside the security room (@scene_baoanshi), cold white fluorescent tubes flicker faintly. Jiang Ye (@char_main_jiangye) sits behind the duty desk, while Xiaomei's (@char_main_xiaomei) figure can be vaguely seen reflected outside the glass window at the doorway.",
      "reference_refs": ["@char_main_xiaomei", "@char_main_jiangye", "@scene_baoanshi"]
    },
    {
      "start": "00:02.0", "end": "00:03.5", "framing": "Close-up", "movement": "Static",
      "description": "The glare on the glass obscures half of Jiang Ye's (@char_main_jiangye) face. The corner of his mouth slowly lifts into an ambiguous smile.",
      "dialogue": { "speakerRef": "Jiang Ye", "tone": "playful, soft", "text": "\"Reward\"", "kind": "dialogue", "source": "verbatim" },
      "reference_refs": ["@char_main_jiangye"],
      "handoff": { "transition_type": "match_on_action", "seedance_input_mode": "text_to_video", "first_frame_ref": null, "repeated_tokens": ["@char_main_jiangye", "@scene_baoanshi"] }
    },
    {
      "start": "00:03.5", "end": "00:05.0", "framing": "Reaction shot", "movement": "Static",
      "description": "Outside the glass, Xiaomei (@char_main_xiaomei) hears those two words. Her smile grows sweeter; she tilts her head slightly, her eyes carrying a hint of triumph.",
      "reference_refs": ["@char_main_xiaomei"],
      "handoff": { "transition_type": "eyeline_match", "seedance_input_mode": "text_to_video", "first_frame_ref": null, "repeated_tokens": ["@char_main_xiaomei"] }
    },
    {
      "start": "00:05.0", "end": "00:07.0", "framing": "Medium close-up", "movement": "Push-in",
      "description": "The camera slowly pushes toward Jiang Ye (@char_main_jiangye). Leaning back in his chair, he lets out a chuckle; his gaze shifts from languid to excited as he reaches toward the door-release button by the desk.",
      "dialogue": { "speakerRef": "Jiang Ye", "tone": "excited, eager", "text": "\"Then what are we waiting for\"", "kind": "dialogue", "source": "verbatim" },
      "reference_refs": ["@char_main_jiangye"]
    },
    {
      "start": "00:07.0", "end": "00:08.0", "framing": "Insert shot", "movement": "Static",
      "description": "Jiang Ye's finger presses the access control switch (@prop_keycard). The button lights up green, and the security room door lock lets out a soft \"click.\"",
      "sfx": ["Access control green-light prompt", "Door lock click"],
      "reference_refs": ["@prop_keycard"]
    },
    {
      "start": "00:08.0", "end": "00:10.0", "framing": "Medium shot", "movement": "Tracking",
      "description": "Xiaomei (@char_main_xiaomei) pushes the door open and walks in, still wearing that cloyingly sweet smile, her body leaning slightly forward.",
      "reference_refs": ["@char_main_xiaomei", "@scene_baoanshi"]
    },
    {
      "start": "00:10.0", "end": "00:12.0", "framing": "Quick cut", "movement": "Handheld",
      "description": "Jiang Ye (@char_main_jiangye) suddenly rises. The force of his movement leaves a blur as one hand shoots out and instantly clamps around Xiaomei's (@char_main_xiaomei) throat, pinning her against the wall.",
      "reference_refs": ["@char_main_xiaomei", "@char_main_jiangye"],
      "handoff": { "transition_type": "match_on_action", "seedance_input_mode": "text_to_video", "first_frame_ref": null, "repeated_tokens": ["@char_main_xiaomei", "@char_main_jiangye"] }
    }
  ]
}
```

Timestamp rules:

- The first block starts at `00:00.0`.
- For each subsequent block, `start` = the previous block’s `end`, with no gaps and no overlaps.
- Use the `MM:SS.S` timestamp format (**0.1s precision**, matching the `X.Xs` duration precision in the source shot lines; do not round to 0.5s, or cumulative timing will drift), equivalent to Seedance’s `[00:00-00:05]` syntax.
- `shot.duration` = the last block’s `end`.
- A single shot’s duration **must be ≤ 15s** (the hard upper limit for one Seedance 2.0 generation pass); if it exceeds this, flag it rather than truncating it on your own.
### Shot Line Parsing

Shot line format:

```
**Shot X X.Xs Time: Daytime.** [Framing/Camera movement] Description...Character (emotion/tone) "Short dialogue"...｜Avoid: negative constraint 1/negative constraint 2
```

Field-by-field parsing:

- `**镜头X` → block.shot_index_in_full (continuous numbering across the full document; does not reset across scenes; this skill only records it and does not consume it heavily)
- ` X.Xs ` → block.duration (used to accumulate and generate start/end, with 0.1s precision)
- `时间: XX。` → validate that it matches shot.time_of_day
- `[景别/运镜]` → block.framing + block.movement (split by `/`)
  - A single shot line **allows only one primary camera movement** (push-in / pull-out / pan / tilt / tracking / orbit / aerial / handheld / locked-off / 固定); if `先推近再环绕` appears, flag it
  - The **scope of the `fast` flag is limited to the full `[景别/运镜]` field plus the parts of the description sentence that describe camera movement** (such as `相机 fast push-in` / `fast tracking`); do not flag plot/physiological phrases like `心跳加快` / `他比上次跑得更快` / dialogue like `"快进来"`
- The description sentence outside quotation marks → block.description (preserve exactly as written)
- **Inline slot references `(@xxx)` inside the description sentence** → extract all tokens in the form `@<purpose>_<name>`, deduplicate them, and store them in block.reference_refs[]; this shot-level reference_refs[] = the deduplicated union of all block references
  - Each extracted `@xxx` must be found in the global `reference_pack[]`; if not found, flag `shot N block M 引用未声明的参考池槽位 @xxx`
- Dialogue with quotation marks + emotion prefix → block.dialogue (parse according to the lip-sync trio protection rules, listed separately below)
- Line-ending `｜避免:...` or `|避免:...` (both full-width and half-width separators are recognized) → aggregate into shot.negative_constraints[], split entries by `/`
### Handoff line parsing (handoff within a block, optional)

If the line immediately after a shot line appears in the single-line format `衔接:类型 模式:xxx 首帧:@xxx 重复:xxx`, map the following fields to the block's `handoff`:

```
衔接:match_on_action 模式:image_to_video_first_frame 首帧:@last_frame_of_shot07 重复:@char_main_xiaomei+@scene_baoanshi
```

This maps to `timeline_blocks[i].handoff`:

```json
{
  "transition_type": "match_on_action",
  "seedance_input_mode": "image_to_video_first_frame",
  "first_frame_ref": "@last_frame_of_shot07",
  "repeated_tokens": ["@char_main_xiaomei", "@scene_baoanshi"]
}
```

Validation:

- `transition_type` must be one of `hard_cut / match_on_action / eyeline_match / sound_bridge / j_cut / l_cut / graphic_match / concept_cut`; if not in the enum, flag it and fall back to `hard_cut`.
- `seedance_input_mode` must be one of `text_to_video / image_to_video_first_frame / first_last_frame / ref_pack / nine_panel_grid_i2v / video_extend`; if not in the enum, flag it and fall back to `text_to_video`.
- If the mode is not `text_to_video`, `首帧:` is required; if missing, flag it.
- After splitting `重复:`, there must be at least 1 token; if the array is empty, flag: "scene N block M shares 0 tokens with the previous block".
- If the handoff line is **missing**, do **not** flag the first block (the first shot within a scene defaults to `hard_cut+text_to_video`); if a subsequent block within the same shot is missing it, flag: "block M is missing handoff, falling back to default".
### Scene Title Handoff Parsing (inter-scene handoff)

A scene title line may include a suffix: `场景X 名称 X秒 衔接:类型(token1+token2)`

Example: `场景2 保安室-身份反转 13秒 衔接:sound_bridge(@ambient_rain+雨声延续)`

Map it to `shot.scene_handoff` (note: this is at the shot level, not the block level; it expresses the handoff from "the previous shot as a whole → this shot as a whole"):

```json
{
  "scene_handoff": {
    "transition_type": "sound_bridge",
    "repeated_tokens": ["@ambient_rain", "雨声延续"]
  }
}
```

Validation:

- `transition_type` uses the same 8 enum values as above; if not in the enum, flag it and fall back to `hard_cut`.
- The first scene has no previous scene, so `scene_handoff` is not required; starting from the second scene, if it is missing, flag: "Scene N is missing an inter-scene handoff (`scene_handoff`); cross-scene consistency may decrease. Recommend returning to consult.storyboard-script-to-shot-prompts to fill it in."
- If the total count of `concept_cut` across the entire piece is >2, flag: "There are N `concept_cut` instances in the full piece; the narrative may fall apart."
### Dialogue Parsing (protecting the lip-sync trio)

Format: `Character (emotion/tone) "short line"` (quotation marks may be `"..."` / `「...」` / `『...』`, and are treated equivalently)

- `角色名` → `dialogue.speakerRef`
- `情绪/语气` → `dialogue.tone` (preserve as-is)
- Text inside quotation marks → `dialogue.text`, **preserve the quotation marks exactly as-is** (Seedance recognizes quotation marks as lip-sync triggers)
- `dialogue.kind` defaults to `dialogue`
- When keywords such as `VO / narration / inner OS / off-screen, off-screen voice / OS` appear, switch to `voiceover / inner_monologue / offscreen`, and for these types the quotation marks may be omitted.
- `dialogue.source`: original text verbatim `verbatim`; lightly rewritten to fit duration capacity `adapted` (**lip-sync dialogue does not allow adapted**; if overloaded, flag it); bridge lines added by shot_split itself `new_bridge` (must still satisfy the trio). **Any `new_bridge` must be force-flagged**: `shot N added a new bridge line new_bridge without user confirmation`.
- **Leading `说:` / `说道:` recognition**: upstream consult rules forbid the `说:` lead-in (quotation marks are the lip-sync trigger), but historical input is still supported. If a shot contains `Character (emotion) 说:「line」`, parse it as `verbatim` (preserve the quotation marks exactly as-is in `dialogue.text`), but also flag: `shot N contains the leading \`说:\`; recommend returning to consult.storyboard-script-to-shot-prompts to remove it so Seedance does not treat it as noise`.
- Validation: within the quotation marks of a single `dialogue.text`, ≤ 12 Chinese characters / 8 English words; if exceeded, flag it.
### Character Anchor Manifest Consumption

The compact format has only one line:

```
Character anchors: Xiaomei=pink puff-sleeve knit top/black short skirt/black short straight hair to jawline/small mole on right brow bone/thin silver necklace; Jiang Ye=black security uniform/buzz cut/helix stud on left ear/thin black cord bracelet on right wrist
```

Parse as:

```json
{
  "continuity_ledger": {
    "character_anchors": {
      "小美": "pink puff-sleeve knit top/black short skirt/black short straight hair to jawline/small mole on right brow bone/thin silver necklace",
      "姜夜": "black security uniform/buzz cut/helix stud on left ear/thin black cord bracelet on right wrist"
    }
  }
}
```

Character reference rules are the same as `lumina-flow-shot-split`:

- If the project has a stable asset for the character → use the stable asset name for `characterRef` + also include `anchor_descriptor`.
- If the project does not have one → **use `anchor_descriptor` as a temporary placeholder for `characterRef`**, so downstream Seedance receives a consistency token, and flag in the quality notes: "Character X is missing a project asset; using a temporary anchor placeholder."
- Validation: all appearing characters in each shot must have a corresponding string in the anchor manifest; flag if missing.
### Reference Pool Single-Line Manifest Consumption

The compact format has only one line:

```
Reference Pool: @char_main_xiaomei=Xiaomei three-view sheet;@char_main_jiangye=Jiangye three-view sheet;@scene_baoanshi=security room main image;@lighting_baoanshi=cool white + red strobe lighting setup;@prop_keycard=keycard close-up;@ambient_rain=indoor rain sound 5s loop;@bgm_tension=tension BGM 8s
```

Parsed as:

```json
{
  "reference_pack": [
    { "slot": "@char_main_xiaomei", "type": "image", "purpose": "char_main", "name": "xiaomei", "description": "Xiaomei three-view sheet" },
    { "slot": "@char_main_jiangye", "type": "image", "purpose": "char_main", "name": "jiangye", "description": "Jiangye three-view sheet" },
    { "slot": "@scene_baoanshi",    "type": "image", "purpose": "scene",     "name": "baoanshi", "description": "security room main image" },
    { "slot": "@lighting_baoanshi", "type": "image", "purpose": "lighting",  "name": "baoanshi", "description": "cool white + red strobe lighting setup" },
    { "slot": "@prop_keycard",      "type": "image", "purpose": "prop",      "name": "keycard", "description": "keycard close-up" },
    { "slot": "@ambient_rain",      "type": "audio", "purpose": "ambient",   "name": "rain",    "description": "indoor rain sound 5s loop" },
    { "slot": "@bgm_tension",       "type": "audio", "purpose": "bgm",       "name": "tension", "description": "tension BGM 8s" }
  ]
}
```

`type` is inferred from `purpose`:
- `char_main / char_outfit / char_face / scene / lighting / prop / style_palette` → `image`
- `motion / transition` → `video`
- `ambient / bgm / voice` → `audio`

Validation:

- `image` slots ≤ 9, `video` ≤ 3, `audio` ≤ 3; if exceeded, flag "reference pool exceeds the Seedance Universal Reference limit".
- `purpose` must be one of the 12 enum values; if not, flag "reference pool slot purpose is non-standard" but still keep it.
- Any `(@xxx)` reference appearing in a shot description must be found in reference_pack[]; if not found, flag "shot N references undeclared reference pool slot @xxx".
- If there are ≥4 shots but no reference pool single-line manifest at the top, continue using the current conservation mode and flag "≥4 shots but missing reference pool manifest; cross-shot consistency falls back to pure token mode; recommended to go back to the top of consult.storyboard-script-to-shot-prompts and add it".
## Workflow

1. First identify the input format. If it matches the `Scene X Name Xs` title + `**Shot X X.Xs Time: ... [Shot Size/Camera Movement]**` line pattern → enter this skill's conservation mode; otherwise, prompt the user to use a sister skill instead (see §Not applicable / Characteristics for routing to sister skills).
2. Parse the optional top manifest (up to three sections):
   - Single-line `Character anchors: A=...;B=...` → `continuity_ledger.character_anchors`
   - Single-line `Reference pool: @xxx=...;@xxx=...` → `reference_pack[]` (consume according to the single-line reference pool manifest rules in §Reference pool single-line manifest above)
3. Split the input by `Scene X`, and process each scene independently as one shot.
4. Parse the scene title: extract the scene number, scene name, and declared duration; if it includes a `Handoff:type(token)` suffix, write it to `shot.scene_handoff`.
5. Iterate through the shot lines within the scene:
   - Accumulate shot durations (with 0.1s precision) to generate timestamps `start / end`
   - Split `[Shot Size/Camera Movement]`
   - Extract description, dialogue, SFX, and negative constraints (recognize both full-width and half-width separators)
   - Extract inline slot references `(@xxx)` in the description and validate them against `reference_pack`
   - Validate single primary camera movement per shot, ensure `fast` appears only as a flag in the camera movement field, and forbid focal-length jargon
   - Parse the `Handoff:` line on the line following the shot line (if any), and write it to `timeline_blocks[i].handoff`
6. Validate scene-level constraints:
   - Total duration = sum of shot durations; flag if inconsistent with the declared duration
   - Total duration ≤ 15s (Seedance limit); flag if exceeded
   - Total duration ∈ [12, 15] (unless otherwise specified by the user); flag if out of range
   - `time_of_day` must be consistent across all shots in the same scene; flag if inconsistent
7. Write all characters appearing in the scene (those directly mentioned in the description, `speakerRef` in dialogue, and those being looked at in reaction shots) to `shot.character_refs`; handle asset matching according to the character anchor placeholder rules; also write the corresponding `@char_main_*` slots for appearing characters to `shot.reference_refs[]` (if declared in the reference pool).
8. Aggregate trailing negative constraints into `shot.negative_constraints[]` and deduplicate them.
9. Starting from the second scene, validate the presence of `shot.scene_handoff` + enum validity + non-empty token; flag if missing.
10. Generate the final shot list in scene order.
11. Output quality notes: list all flags, placeholder notes, capacity warnings, upper/lower bound warnings, missing handoff warnings, undeclared reference pool slot reference warnings, and `说:` prefix compatibility warnings.
## Shot Boundaries (Within a Scene)

This skill **does not split shots within a scene** — the number and order of shots within a scene are determined by the upstream director `consult.storyboard-script-to-shot-prompts`, and shot_split preserves them.

However, the following cases should be flagged:

- Single-shot duration > 5s: beyond this threshold, timestamp composition has limited value (one camera shot is almost equivalent to one shot); recommend going back to consult to split it.
- Two consecutive shots within the same scene with the same shot size and the same primary camera movement appear twice (for example, `[Medium Close-Up/Static] → [Medium Close-Up/Static]`): this may be a duplicate from the director; flag it for the user to review.
- Shot numbering within a scene does not increase continuously (skipped numbers, duplicate numbers): the source text contains an error; flag it and do not fill in or correct it on your own.
## Scene Boundaries (across shots)

- By default, transitions between scenes are hard cuts (the scene title itself constitutes a scene boundary). The downstream presenter determines whether to add J-Cuts, L-Cuts, or audio bridges between scenes.
- Character blocking, eyelines, and handheld props are not required to remain continuous across scenes: jump cuts are allowed between scenes (when users write separate scenes, it implies that space/time may change).
- Across scenes, same-named characters in character_refs still refer to the same asset/anchor; do not treat them as new characters just because the scene has changed.
## Duration and Granularity

**A "scene" = 1 shot = 1 Seedance call = what the user calls "1 video"**, and is an aggregation unit above individual camera shots.

- Each scene’s declared duration **defaults to 12-15s** (aligned with Seedance’s hard single-generation upper limit of 15s to maximize output per generation).
- **When the user does not explicitly specify a duration, validate against the default 12-15s**; if it deviates, flag: "Scene X duration Y.Ys deviates from the default 12-15s; confirm whether the user explicitly specified another duration".
- **When the user explicitly specifies a duration** (for example, writing in the prompt "one video is recommended to be 8-10s" / "each segment is 6 seconds" / "each scene should not exceed 10s" / "each scene is 12s"), validate against the user-provided value and no longer apply the 12-15s flag.
- Seedance hard limit per generation is ≤15s: if any scene is declared as > 15s, flag: "Scene X duration Y.Ys exceeds the Seedance single-generation limit of 15s", but still output according to the original text (use the sum of the camera shots for `shot.duration`) and **do not truncate on your own**.
- Typical single-shot durations: establishing/environment shot 1.5-2.5s / expression close-up 1.0-1.5s / simple insert 0.8-1.2s / dialogue line 2.0-3.5s / sudden action 0.5-1.2s / reversal/closing beat 1.5-2.5s.
- Do not force durations within a scene to be evenly distributed; the upstream process has already allocated rhythm.
- If the declared scene duration and the sum of shot durations do not match → use the sum of the shots as `shot.duration`, and flag in the quality notes: "Scene X declared as 12s, actual sum of shots is 11.5s/14.5s".

| User Prompt | Validation Range |
|---|---|
| Duration not mentioned | **Default 12-15s** |
| "One video is 8-10 seconds" | 8-10s |
| "Each segment is 6 seconds" | 6s |
| "Each video should not exceed 10s" | ≤10s (treat as 8-10s) |
| "Each scene is 12s" | 12s |
## Asset References

- Same as `lumina-flow-shot-split` §Asset Reference Rules: only reference stable project names; if an asset is missing, use `anchor_descriptor` as a placeholder + flag.
- The compact format has no explicit prop field. Extract obvious props from the shot description (access card, surveillance monitor, glass door, coffee table, etc.), and include them in `prop_refs` only if the project has stable assets for them; otherwise, keep them in `description`.
- The scene name itself may be a scene asset (for example, in "Security Room - Smile Behind the Glass," "Security Room"). If the project has a corresponding scene asset → `scene_ref`; otherwise, keep it only in `shot.scene_name`.
## Downstream Constraints and meta Field Pass-through

The compact format has relatively few fields, but there are still several that must be passed through:

| Source | Target | Notes |
|---|---|---|
| End-of-line `｜避免:...` | shot.negative_constraints[] (aggregate at the shot level, deduplicate) | **Must be preserved**; downstream Seedance prompts will append it to the negative slot |
| Implicit SFX signals in the shot description (such as "a click sounds" or "the fluorescent tube flickers twice") | timeline_blocks[i].sfx[] | Preserve; downstream can route to Seedance native audio SFX channels |
| Chiaroscuro boundary / color temperature information in the shot description (such as "5600K cool white fluorescent top light" or "the light-shadow boundary splits diagonally") | shot.lighting_geometry | Named light fields must be preserved |
| Micro-expression chains in the shot description (such as "blinks once first → Adam's apple moves slightly → corners of the mouth press downward for 0.5 seconds") | timeline_blocks[i].micro_expression_chain (array) | Key signals for expression-focused scenes; preserve exactly as written |
| Gaze anchors in the shot description (such as "gaze locked on B's left eye for 1.6s") | timeline_blocks[i].gaze_anchor | Key signals for expression-focused scenes |

**Do not rewrite, merge, or compress** these fields. If the original text does not contain them, leave them empty; do not fabricate them.
## Subject-First flag

scene-as-shot does not rewrite the original shot-line descriptions, but it must flag phrasings in the quality notes that trigger the Seedance 2.0 anti-patterns of "character flicker / inter-shot blending." Check the first sentence of each `block.description`:

- **Environment-first**: The first sentence begins with environmental phrases such as "bright X / sunlight / cool light / warm light / the whole room / a stretch of," followed by "the camera pushes in toward X / the camera pans over to X." Flag: "Scene X Shot N has an environment-first opening sentence; Seedance 2.0 is prone to character flicker. Recommend returning to consult.storyboard-script-to-shot-prompts and changing it to subject-first (write the subject's position + action first, then the environment)."
- **Passive reveal phrasing**: Phrases such as "revealing X / finally settles on X / it turns out X is standing / X suddenly appears / X flashes into view / X is now in the frame" appear. Flag: "Scene X Shot N contains passive reveal phrasing; the subject may be revealed too late."
- **Missing carry-over character**: The same character from the previous shot (in the same scene or the last shot of the previous scene) does not explicitly appear in the first sentence of this shot (no position/state description). Flag: "Scene X Shot N carry-over character X is not reintroduced with position/state in the first sentence."
- **New entering character lacks entry direction**: A character appearing for the first time in this shot has no direction/starting position specified (e.g. "enters from the left side of the frame"). Flag: "Scene X Shot N new entering character X is missing entry direction."

scene-as-shot **must not rewrite** `block.description` on its own; output it as-is from the source text. Only flag issues in the quality notes, and let the user decide whether to go back to the consult layer to fix them.
## Focal Length Jargon Flag

When technical jargon such as `85mm / f/1.4 / 35mm 焦段 / shallow DOF / focal plane / 浅景深 / 焦距 / 光圈 / 景深` appears in the `[景别/运镜]` field or in block.description, flag it as: "Scene X Shot N contains focal length jargon, which Seedance does not respond to and may even degrade on. It is recommended to replace it with `广角空间感 / 自然视角 / 人像压缩感 / 微距质感 + 画面占比`."
## Quality Standards

- Input format is classified correctly: compact format goes through this skill; director-level long-form drafts (multi-line `## Character Anchors` / `## Visual Reference Pool` / fielded sections) should be routed to the sister skill, with no misclassification.
- Every scene is converted into exactly one shot; scene order, scene names, and scene numbering remain consistent.
- shot.duration = the sum of internal camera segment durations (0.1s precision, no rounding drift); any mismatch with the declared duration is flagged.
- Each shot.duration ≤ 15s (the hard upper limit for a single Seedance generation); any excess is flagged.
- Each shot.duration falls within the **default 12–15s** range (when the user did not specify) or the **user’s explicitly specified range** (for example, “one video 8–10s”); any deviation is flagged. If the user did not specify a range, always use the default and do not reinterpret it on your own.
- Timestamps in each shot’s timeline_blocks have no gaps and no overlaps, start from 00:00.0, and the final block end = shot.duration.
- Each timeline_block single shot has only one primary camera movement; if two appear, flag it.
- The `fast` flag scope is restricted to the full text of the `[Shot Size/Camera Movement]` field plus camera-movement fragments in the description; uses of fast in plot/physiology/dialogue are not falsely flagged.
- Technical jargon such as focal length/aperture/depth of field/85mm/f/1.4 must not appear (Seedance does not respond to them); if present, flag it.
- For lip-sync dialogue (kind=dialogue + quotation marks): preserve quotation marks exactly as written, fill the tone field, keep each line ≤ 12 Chinese characters / 8 English words, source must not be `adapted`; `new_bridge` must be strongly flagged; leading `说:` is compatibly parsed + flagged.
- The character anchor manifest has been parsed; every character appearing in any shot can be found in the anchors; characters missing project assets use anchor_descriptor as a placeholder + are flagged.
- **The reference-pool single-line manifest has been parsed** (when there are ≥4 shots); the upper limits for the three reference_pack[] slot types are compliant (image ≤9 / video ≤3 / audio ≤3); all `(@xxx)` references in shot descriptions can be found in reference_pack, and any missing reference is flagged.
- **Inter-scene handoff has been parsed**: starting from the second scene, the existence of scene_handoff, the validity of the transition_type enum, and non-empty repeated_tokens have all been validated; any missing item is flagged.
- **In-block transition lines have been parsed** (if present): transition_type / seedance_input_mode enums are valid, first_frame_ref is filled for non-text_to_video modes, and repeated_tokens is non-empty.
- The total count of `concept_cut` across the full video is ≤2; any excess is flagged.
- End-of-line negative constraints are aggregated into shot.negative_constraints[], deduplicated, with both full-width and half-width separators recognized, and no item dropped.
- SFX, named lighting setups, micro-expression chains, and gaze anchors in descriptions have been mapped into their corresponding fields, with no merging and no compression.
- The shot list order matches the original storyboard scene order.
- The output can be fed directly into a downstream Seedance one-pass final video generation call, with no need to guess the original intent.
## Failure Fallback

- When the input does not look like the compact format (i.e., it contains fielded sections / `### Shot NN:` / `### Dialogue Block NN:` / a multi-line `## Character Anchor` block / a multi-line `## Visual Reference Pool` block / a multi-line `## Continuity Ledger` block), prompt: "The input is in director-level long-form format and should be processed with `lumina-flow-shot-split`; do not forcibly convert it within this skill."
- When the declared scene duration is > 15s, flag: "Scene X duration X.Xs exceeds Seedance's 15s single-generation limit; recommend going back to `consult.storyboard-script-to-shot-prompts` to split it into two scenes", and still emit structured output based on the original text (`shot.duration` should use the sum of the shots), without truncating on its own.
- When the declared scene duration deviates from the default 12-15s **and the user has not explicitly specified another duration** (if the user did not specify one, always use the default), flag: "Scene X duration Y.Ys deviates from the default 12-15s; this may indicate upstream format drift or an undeclared special duration requirement from the user", and still output according to the original text.
- If the user explicitly specifies another duration in the prompt (e.g. "one video 8-10s"), validate against the user-specified value and no longer apply the 12-15s flag.
- When a single shot duration is > 5s, flag: "Scene X Shot N duration X.Xs is long; timestamp composition is of limited value; recommend going back to consult to split it or consider using the sister skill instead", and still output according to the original text.
- When lip-sync dialogue exceeds 12 characters, flag: "Scene X Shot N dialogue overload; recommend going back to consult to shorten it", **do not adapt it on its own**.
- When the top-level character anchor manifest is missing, continue processing under the current conservation mode, and flag: "Character anchor manifest missing; cross-shot consistency may degrade; recommend going back to `consult.storyboard-script-to-shot-prompts` to add it at the top."
- When there are ≥4 shots but the reference pool manifest is missing, continue under the current conservation mode, and flag: "≥4 shots but missing the single-line reference pool manifest; cross-shot consistency falls back to pure token mode (practical upper limit around 6 shots); for long narratives of 10+ shots, adding it is strongly recommended."
- When `scene_handoff` is missing starting from the second scene, flag: "Scene N has no inter-scene transition (`scene_handoff`); cross-scene consistency may degrade", and emit output using the default `hard_cut` value without inventing tokens.
- If parsing the `Transition:` line immediately below a shot line fails (invalid enum value / mode is not `text_to_video` but first frame is missing / token is empty), emit output using the default values (`hard_cut + text_to_video`) + a flag, without repairing it on its own.
- When shot numbering skips or duplicates, flag: "Scene X shot numbering is non-continuous: 1, 2, 4", and process in original order without adding/changing numbering on its own.

### Failure Mode Notes for the scene-as-shot Path

This skill compresses a scene into a single Seedance multi-shot call. It has several path-specific failure modes that users should be warned about:

- **drift within a scene**: middle shots within a single generated clip are prone to face drift / wardrobe jumps / scene geometry changes. This is harder to control for mid-sequence consistency than shot-as-shot (independent per-shot calls). Recommendation: keep shots per scene ≤6 + use a strong reference pool + strongly repeat tokens.
- **hard cuts at scene boundaries**: each scene is an independent Seedance call, so transitions between scenes are naturally hard cuts. If the story requires continuity like match_on_action / sound_bridge, it must be declared in `scene_handoff`, and downstream must use `image_to_video_first_frame` (first frame = last frame of the previous scene) to lock continuity.
- **the 15s hard wall cannot be broken**: a total scene duration > 15s is a hard limit for a single Seedance generation, **and cannot be bypassed through prompt engineering**; the scene can only be split.

When users hit these constraints in the prompt, this skill does not repair them on its own; it emits output according to the original text + a flag, and prompts the user to return to the consult layer to decide the path.